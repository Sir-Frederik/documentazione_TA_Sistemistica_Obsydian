
Appunti operativi personali sul progetto sicurezza endpoint di Technology Advising. Tutto quello che ho fatto, perché l'ho fatto, e come ripeterlo senza dover ricordare tutto a memoria.

---

## 🧩 Contesto e decisioni architetturali

**Il problema di partenza**: gestire ~46 endpoint con un mix di Windows 11 Home e Pro in modo centralizzato, applicando controllo delle applicazioni e cifratura disco, senza spendere soldi in licenze extra.

Le strade esplorate e scartate:

| Opzione                           | Perché no                                               |
| --------------------------------- | ------------------------------------------------------- |
| Samba AD DC + domain join         | Windows 11 Home non supporta l'aggiunta a un dominio AD |
| Upgrade a Windows 11 Pro          | Costo per ~46 licenze                                   |
| AppLocker su Home                 | Le regole non vengono applicate su Home — è inerte      |
| SRP (Software Restriction Policy) | Tecnologia superata, incompatibile con WDAC attivo      |
|                                   |                                                         |

**La soluzione**: ==WDAC (Windows Defender Application Control) via [[CiTool]]== , disponibile su tutte le edizioni di Windows 11. Si compila la policy sul PC Pro con il modulo `ConfigCI`, e si deploya su qualsiasi macchina — Home inclusa — tramite CiTool. Elegante.

---

## 🖥️ Infrastruttura

|Componente|Dettaglio|
|---|---|
|TRMM Server|OVH Frankfurt — Ubuntu 22.04 LTS — `213.32.30.52`|
|Pannello|`https://ta-tactical-rmm.duckdns.org`|
|API|`https://ta-tactical-api.duckdns.org`|
|MeshCentral|`https://ta-tactical-mesh.duckdns.org`|
|PC Home (cavia)|Lenovo V15 G4 AMN — Windows 11 Home — utente `testf`|
|PC Pro (compilazione)|`TA-TEST` — Windows 11 Pro — utente `testf`|
|Cartella policy|`C:\WDAC\` su entrambe le macchine|

### Riferimenti WDAC:

| Campo            | Valore                                                                   |
| ---------------- | ------------------------------------------------------------------------ |
| Policy GUID      | `{966d1f08-bcea-48c4-bc3a-6651c21e4090}`                                 |
| Versione attuale | `10.0.0.14` (v14)                                                        |
| File .cip attivo | `C:\Windows\System32\CodeIntegrity\CiPolicies\Active\{966d1f08-...}.cip` |
| Stato attuale    | ==Enforcement mode ✅==                                                   |

### Riferimenti TRMM:

| Campo                  | Valore                             |
| ---------------------- | ---------------------------------- |
| Custom Field BitLocker | `bitlocker_recovery_key` — ID: `1` |

#### 🔑 API Key

Alcuni script (`BitLocker: Enable`, `BitLocker: Store Recovery Key`) devono
comunicare con l'API di TRMM per salvare dati nei Custom Field — ad esempio
la recovery key di BitLocker. Per farlo serve una **API Key** generata da
TRMM, passata come parametro obbligatorio `ApiKey` allo script.

==Non va mai scritta dentro lo script== — si passa come parametro al momento
dell'esecuzione, così resta fuori dal codice salvato e non finisce per
sbaglio in una versione condivisa o esportata.
La trovi nelle **impostazioni** di TRMM
La chiave si genera da: **TRMM → Settings → API Keys**.
---

## 🛡️ Stack di sicurezza attivo

### WDAC

Il layer principale. Blocca tutto ciò che non è esplicitamente autorizzato dalla policy o riconosciuto da [[ISG — Intelligent Security Graph]] come software legittimo firmato. Attivo in ==enforcement mode== sulla flotta.

La policy è stata costruita iterativamente: prima in **audit mode** per raccogliere i file legittimi presenti sulle macchine, poi ==compilata in enforcement== sul PC Pro e deployata con [[CiTool]].

> 💡 **Trade-off ISG**: la regola `Enabled:Intelligent Security Graph Authorization` è necessaria per evitare boot failure da driver non coperti. Come effetto collaterale, software firmati molto diffusi (Firefox, Sumatra PDF…) passano anche senza essere in whitelist. 
> Questa regola è **dentro l'XML della policy WDAC**

### ASR — Attack Surface Reduction Rules

9 regole di Windows Defender che bloccano vettori di attacco specifici: macro Office, credential stealing (LSASS), script offuscati, eseguibili da USB, processi figlio di Adobe Reader… Attive in modalità `Enable` sulla flotta. 
Impostata dallo script [[Libreria Script TRMM#Defender Deploy ASR Rules|Defender: Deploy ASR Rules]] via `Add-MpPreference.`

### Controlled Folder Access

Protegge le cartelle di sistema e documenti da modifiche non autorizzate. Le app legittime vanno aggiunte alla whitelist con il ==path eseguibile specifico== — no wildcard di directory.
Impostata dallo script [[Libreria Script TRMM#Defender Enable Controlled Folder Access|Defender: Enable Controlled Folder Access]] via `Set-MpPreferenc`
### BitLocker / Device Encryption

Cifratura disco attiva sugli endpoint. Su Windows 11 Home non c'è BitLocker nell'UI ma i cmdlet PowerShell funzionano e Device Encryption è disponibile. Le ==recovery key sono archiviate nel Custom Field TRMM== di ogni agente.
Attivata  dallo script [[Libreria Script TRMM#BitLocker Enable| BitLocker: Enable]] 

### AppLocker (solo TA-TEST)

Su TA-TEST è in **AuditOnly** — lo stato in cui è finito per risolvere un problema di Start Menu/Settings rotto da una configurazione precedente. Di fatto inerte. Con WDAC attivo, AppLocker è comunque ==superfluo==.

---

## 🔄 Workflow WDAC — ciclo completo

Questo è il processo ogni volta che si aggiorna la policy. Nuove app da aggiungere, nuova versione da rilasciare.

### Fase 0 — Ricerca file app Store 🔍 (PC Home)

Se l'app da aggiungere è un'app Microsoft Store, i suoi file si trovano in
`C:\Program Files\WindowsApps\` — ma la cartella giusta va cercata
esplicitamente, altrimenti `Get-Item` restituisce le varianti `neutral_split`
che non contengono DLL effettive.

```powershell
# Trovare la cartella corretta — filtrare su *x64*
Get-Item "C:\Program Files\WindowsApps\*NomeApp*x64*"
```

Una volta trovata, copiare il contenuto in `C:\WDAC\AppxScan\` e procedere
con la Fase 1.

> ⚠️ Senza il filtro `*x64*` il wildcard restituisce prima i pacchetti
> `neutral_split` — file parziali che WDAC non incontrerà mai durante
> l'esecuzione. La scansione produrrebbe regole inutili e l'app rimarrebbe
> bloccata.
### Fase 1 — Raccolta file 📂 (PC Home)

Raccogliere i file da aggiungere in `C:\WDAC\AppxScan\` sul PC Home — eseguibili, DLL, package Store.

### Fase 2 — Lettura log audit 🔍 (PC Home)

Prima di raccogliere file, leggere i log WDAC per capire cosa è stato bloccato:

Eseguire: [[Libreria Script TRMM#WDAC Export Audit Log|WDAC:Export Audit Log]]
```
Output: C:\WDAC\audit_newapps.txt
Event ID rilevante: 3076
```

### Fase 3 — Trasferimento al Pro 📤 (MeshCentral)

Spostare i file da aggiungere dal PC Home a `C:\WDAC\` su TA-TEST via **MeshCentral**.  MeshCentral non vuole cartelle, ma file compressi.

### Fase 4 — Compilazione policy ⚙️ (TA-TEST)

Questa fase si svolge in 3 passaggi distinti, tutti su TA-TEST.

##### Passaggio 1 — Scansione nuovi file _(PowerShell manuale)_

Scansiona i file in `AppxScan` e genera un XML parziale con le nuove regole:

```powershell
New-CIPolicy -Level Publisher -Fallback Hash -ScanPath C:\WDAC\AppxScan\ -FilePath C:\WDAC\new_rules.xml
```

##### Passaggio 2 — Merge con la policy esistente _(PowerShell manuale)_

Unisce le nuove regole con la policy corrente e produce un nuovo XML.
==La versione precedente non viene mai sovrascritta — si crea sempre un file nuovo==:

```powershell
Merge-CIPolicy -PolicyPaths C:\WDAC\policy_final_v14.xml, C:\WDAC\new_rules.xml -OutputFilePath C:\WDAC\policy_final_v15.xml
```

##### Passaggio 3 — Compilazione in enforcement _(script TRMM)_

Su TRMM, eseguire lo script `WDAC: Compile Enforcement Policy` con questi parametri:

| Parametro | Valore |
|---|---|
| SourceXml | `C:\WDAC\policy_final_v15.xml` |
| OutputXml | `C:\WDAC\policy_final_v15_enforced.xml` |
| OutputP7b | `C:\WDAC\policy_final_v15.p7b` |
| NewVersion | `10.0.0.15` |

Lo script rimuove la regola `Enabled:Audit Mode` dall'XML, aggiorna la versione e compila il `.p7b` pronto per il deploy.

> ℹ️ `ConfigCI` è disponibile ==solo su Windows Pro/Enterprise== — tutta la compilazione avviene su TA-TEST.

### Fase 5 — Trasferimento al Home 📥 (MeshCentral)

Trasferire solo il file `.p7b` compilato da TA-TEST al PC Home in `C:\WDAC\`. 

### Fase 6 — Deploy 🚀 (PC Home, via TRMM)


Eseguire: 
[[Libreria Script TRMM#WDAC Deploy Policy|WDAC: Deploy Policy]]
```
  PolicyPath    = C:\WDAC\policy_final_v15.p7b
  PolicyVersion = 10.0.0.15
  Reboot        = (switch facoltativo)
```

Riavviare. La policy è effettiva dopo il riavvio.

### Fase 7 — Verifica ✅

Testare su account admin e account standard. Per la flotta: ==sempre su una macchina sola prima di procedere con le altre==.

---

## 🚨 Recovery WDAC — boot failure

Se dopo un deploy la macchina non si avvia, WDAC ha bloccato qualcosa di critico. Niente panico — si risolve.

**1. Entrare in WinRE** Tenere Shift premuto e selezionare Riavvia, oppure interrompere il boot 3 volte consecutive per forzarlo.

**2. Opzioni avanzate → Prompt dei comandi**

**3. Recovery Key BitLocker** (se richiesta) Recuperarla dal Custom Field `bitlocker_recovery_key` dell'agente TRMM.

**4. Eliminare il file .cip**

``` cmd
del "C:\Windows\System32\CodeIntegrity\CiPolicies\Active\{966d1f08-bcea-48c4-bc3a-6651c21e4090}.cip"
```

> ⚠️ In WinRE `wevtutil` non funziona. Per analizzare i log, copiarli con `copy` su una chiavetta USB e leggerli offline.

**5. Riavviare**

```
wpeutil reboot
```

La macchina si avvia senza WDAC. Analizzare i log, correggere la policy, ripetere il deploy.

---

## 📜 Script Library

Tutti gli script sono salvati e categorizzati in TRMM. Qui sotto trovi per ognuno: a cosa serve, quando usarlo e le cose a cui stare attento.

---

### 💾 BitLocker

#### [[Libreria Script TRMM#BitLocker Check Status|BitLocker: Check Status]]

Legge lo stato di BitLocker su `C:` e suggerisce l'azione correttiva. Non salva nulla su TRMM — è solo lettura locale.

**Quando**: primo controllo su una macchina nuova, o verifica rapida prima di un intervento.

---

#### [[Libreria Script TRMM#BitLocker Enable|BitLocker: Enable]]

Attiva BitLocker con TPM + Recovery Password e ==salva/aggiorna automaticamente la chiave nel Custom Field TRMM==. 
Gestisce ==**quattro scenari**==: ==già **attivo**== (sincronizza comunque la chiave), ==cifratura in corso== (nessuna nuova azione di abilitazione), ==protezione sospesa== (resume), ==non attivo== (**attivazione completa**).

Garantisce inoltre un solo protector RecoveryPassword attivo, rimuovendo automaticamente eventuali duplicati residui da tentativi precedenti.  

**Quando**: primo setup su un endpoint nuovo, per riattivare dopo una sospensione, o per risincronizzare la chiave su TRMM se sospetti che sia disallineata. 
**Parametri**: `ApiKey` (obbligatorio)
ex: 
```
-ApiKey YNV################
```


---

#### [[Libreria Script TRMM#BitLocker Disable|BitLocker: Disable]]

Disabilita BitLocker e avvia la decifrazione. Idempotente: se già disabilitato, esce senza fare nulla.

**Quando**: solo per manutenzione straordinaria — non in produzione senza una buona ragione.

---

#### [[Libreria Script TRMM#BitLocker Store Recovery Key|BitLocker: Store Recovery Key]]

Estrae la Recovery Key già esistente e la sincronizza nel Custom Field TRMM, senza toccare BitLocker.

**Quando**: macchine già cifrate prima del deploy dell'agente TRMM, o se la chiave in TRMM manca o è sbagliata. **Parametri**: `ApiKey` _(obbligatorio)_

> ⚠️ Se esistono più `RecoveryPassword` protector sullo stesso volume, lo script prende il primo con `Select-Object -First 1` — necessario per evitare errori di tipo nella chiamata API.

---

### 🛡️ Defender

#### [[Libreria Script TRMM#Defender Enable Controlled Folder Access|Defender: Enable Controlled Folder Access]]

Abilita *CFA* su Windows Defender. ==Non aggiunge app alla whitelist== — manualmente tramite   `Add-MpPreference` `-ControlledFolderAccessAllowedApplications`   con path eseguibile specifico, senza *wildcard* (Cioè gli i caratteri jolly come gli asterischi.)
ex:
``` powershell
Add-MpPreference -ControlledFolderAccessAllowedApplications "C:\Program Files\Mozilla Firefox\firefox.exe"
```
Questo comando dice a Windows Defender: "Firefox può scrivere nelle cartelle protette da Controlled Folder Access (Documenti, Desktop, ecc.) anche se normalmente non gliel'avresti permesso."

Controllo:
``` powershell
(Get-MpPreference).ControlledFolderAccessAllowedApplications
```

**Quando**: setup iniziale di un endpoint.

---

#### [[Libreria Script TRMM#Defender Deploy ASR Rules|Defender: Deploy ASR Rules]]

Configura le **9 ASR rules** (Attack Surface Reduction) di Windows Defender —
regole che bloccano vettori di attacco specifici, come macro Office
malevole, credential stealing o esecuzione di script offuscati.

Lo script supporta tre modalità, da passare nel parametro `Mode`:

| Modalità    | Effetto                                  | Quando usarla                                            |
| ----------- | ---------------------------------------- | -------------------------------------------------------- |
| `Enable`    | Blocca attivamente — modalità produzione | Default, uso normale                                     |
| `AuditMode` | Registra ma non blocca                   | Per testare nuove regole senza rischio di falsi positivi |
| `Disable`   | Rimuove le regole                        | Solo se serve disattivare ASR su una macchina specifica  |

Dopo l'esecuzione, lo stato configurato viene salvato in
`C:\ProgramData\TacticalRMM\asr_status.json` — utile per verificare
rapidamente quale modalità è attiva senza dover rileggere tutte le regole.

**Quando usarlo**: al setup iniziale di un endpoint, oppure per cambiare
modalità su una macchina specifica (es. passare in `AuditMode` per
diagnosticare un falso positivo).

**Parametri**: `Mode` _(facoltativo, default `Enable`)_

> ⚠️ Il GUID `01443614-...-2ECDE92B5EBE` (regola che blocca eseguibili da
> USB) aveva un carattere extra nella versione originale dello script —
> la regola non veniva mai applicata. Corretta nella libreria attuale.

---

### 🔒 WDAC

Quattro script che coprono l'intero ciclo di vita della policy: dalla
diagnosi di cosa manca, alla compilazione, al deploy, fino al recovery
in caso di problemi.

---

#### [[Libreria Script TRMM#WDAC Export Audit Log|WDAC: Export Audit Log]]

Quando WDAC è in **audit mode**, non blocca i file sconosciuti — li lascia
passare ma registra l'evento nel log di sistema con **Event ID 3076**. Questo
script legge quegli eventi e li salva in un file di testo leggibile:
`C:\WDAC\audit_newapps.txt`.

È il modo per scoprire **cosa la policy attuale non copre ancora** — ogni
eseguibile, DLL o script che è stato eseguito sulla macchina ma non è
esplicitamente autorizzato. È il primo passo da fare prima di costruire una
nuova versione della policy.

**È il passo 2 del workflow.**

**Quando usarlo**: prima di raccogliere file per una nuova versione della
policy, per sapere esattamente cosa manca invece di indovinare.

---

#### [[Libreria Script TRMM#WDAC Compile Enforcement Policy|WDAC: Compile Enforcement Policy]]

Una volta che l'XML della nuova policy è pronto (dopo `New-CIPolicy` e
`Merge-CIPolicy`, eseguiti manualmente — vedi [[📘Guida personale a TRMM#Fase 4 — Compilazione policy ⚙️ (TA-TEST)|Fase 4]]),
questo script fa l'ultimo miglio: rimuove la regola `Enabled:Audit Mode`
dall'XML — trasformando la policy da "registra soltanto" a "blocca
attivamente" — aggiorna il numero di versione e compila il file binario
`.p7b`, l'unico formato che `CiTool` sa effettivamente deployare.

**È il passo 4 del workflow.**

**Quando usarlo**: ogni volta che si rilascia una nuova versione della
policy, dopo aver già preparato l'XML mergiato.

**Parametri**: `SourceXml`, `OutputXml`, `OutputP7b`, `NewVersion`
_(tutti obbligatori)_

> ⚠️ ==Eseguibile solo su TA-TEST== — il modulo `ConfigCI`, da cui dipende
> la compilazione, non è disponibile su Windows Home.

---

#### [[Libreria Script TRMM#WDAC Deploy Policy|WDAC: Deploy Policy]]

Il binario `.p7b` compilato su TA-TEST non serve a nulla finché non viene
installato sulla macchina di destinazione. Questo script fa esattamente
questo: chiama `CiTool --update-policy` per caricare la policy, poi verifica
che il file `.cip` risultante sia effettivamente comparso nella cartella
`CiPolicies\Active` — non si fida solo del codice di uscita di CiTool.

Se la verifica va a buon fine, scrive un piccolo file di stato in
`C:\ProgramData\TacticalRMM\wdac_state.json` con versione e data di deploy.
Questo file è quello che [[Libreria Script TRMM#Monitor WDAC Policy Status|Monitor: WDAC Policy Status]]
legge per sapere se la macchina è aggiornata.

**È il passo 6 del workflow.**

**Quando usarlo**: subito dopo aver trasferito il `.p7b` compilato sulla
macchina target via MeshCentral.

**Parametri**: `PolicyPath`, `PolicyVersion` _(obbligatori)_ —
`Reboot` _(switch facoltativo, per riavviare automaticamente a fine deploy)_

> ⚠️ Lo script non trasferisce file — il `.p7b` deve essere ==già presente==
> sulla macchina (via MeshCentral) prima di lanciarlo.

---

#### [[Libreria Script TRMM#WDAC Remove Active Policy|WDAC: Remove Active Policy]]

La via di emergenza quando una policy deployata causa problemi ma la
macchina è ancora accesa e raggiungibile da TRMM. Rimuove il file `.cip`
dalla cartella Active — la rimozione diventa effettiva solo al riavvio
successivo, quindi c'è tempo per valutare prima di riavviare.

**Quando usarlo**: la macchina è accesa, ma qualcosa nella policy sta
causando malfunzionamenti (app legittime bloccate, comportamenti anomali) e
serve toglierla rapidamente da remoto. Se invece la macchina **non si avvia
più**, questo script non è raggiungibile — serve la procedura manuale via
WinRE (vedi [[📘Guida personale a TRMM#🚨 Recovery WDAC — boot failure|Recovery WDAC]]).

> ⚠️ Il file da rimuovere è ==`.cip`==, non `.p7b`. La versione originale
> dello script cercava il file sbagliato (`.p7b`) ed era completamente
> inerte — non rimuoveva mai nulla. La versione attuale punta al GUID
> esplicito della policy e funziona correttamente.
### 🔐 AppLocker

> ℹ️ Questi script si applicano solo a TA-TEST (Windows Pro). Su Home AppLocker è inerte anche se il servizio gira. Con WDAC attivo è comunque superfluo ovunque.

#### [[Libreria Script TRMM#AppLocker Set Enforcement|AppLocker: Set Enforcement]]

Avvia `AppIdSvc` e applica una policy di base: exe consentiti solo da `%WINDIR%` e `%PROGRAMFILES%` per tutti, tutto libero per gli Administrators.

**Quando**: raramente — solo se si vuole AppLocker attivo su TA-TEST per un test specifico.

---

#### [[Libreria Script TRMM#AppLocker Set AuditOnly|AppLocker: Set AuditOnly]]

Porta AppLocker in AuditOnly e tenta di fermare `AppIdSvc`. ==È lo stato attuale di TA-TEST==, ed è quello che ha risolto il problema di Start Menu/Settings rotto.

**Quando**: se AppLocker su TA-TEST causa problemi operativi.

---

### 👤 User

#### [[Libreria Script TRMM#User Enable Account|User: Enable Account]]

Abilita un account utente locale disabilitato. **Parametri**: `Username` _(obbligatorio)_

---

#### [[Libreria Script TRMM#User Disable Account|User: Disable Account]]

Disabilita un account utente locale e termina la sessione attiva se presente. **Parametri**: `Username` _(obbligatorio)_

---

#### [[Libreria Script TRMM#User Grant Admin|User: Grant Admin]]

Aggiunge un utente agli Administrators. Idempotente: se già membro, esce senza errori. **Parametri**: `Username` _(obbligatorio)_

---

#### [[Libreria Script TRMM#User Revoke Admin|User: Revoke Admin]]

Rimuove un utente dagli Administrators e applica restrizioni aggiuntive: ==blocco MSI per non-admin==, Autorun/Autoplay disabilitato, blocco disinstallazione programmi. Il controllo degli eseguibili è delegato a WDAC.

**Quando**: standardizzare un endpoint aziendale, o dopo che un utente ha avuto i privilegi admin temporaneamente. **Parametri**: `Username` _(obbligatorio)_

> ℹ️ La versione originale conteneva una sezione SRP — rimossa perché superata da WDAC e potenzialmente conflittuale.

---

### 📊 Monitor

Questi script girano via **Automation Policy** su tutta la flotta. Escono con `0` se tutto è OK, con `1` se c'è qualcosa che non va — TRMM genera un alert.

#### [[Libreria Script TRMM#Monitor Store Apps|Monitor: Store Apps]]

Confronta le app Microsoft Store installate con la lista autorizzata. Segnala qualsiasi app non in lista.

**Lista attuale**: Calculator, Notepad, Photos, Paint, Store, DesktopAppInstaller, Terminal, SecHealthUI, SoundRecorder, OpenAI.ChatGPT, Anthropic.Claude.

> ⚠️ I nomi `OpenAI.ChatGPT` e `Anthropic.Claude` sono da verificare con `Get-AppxPackage` — potrebbero non corrispondere ai nomi esatti dei pacchetti Store installati.

---

#### [[Libreria Script TRMM#Monitor BitLocker Compliance|Monitor: BitLocker Compliance]]

Verifica che BitLocker sia attivo e completamente cifrato su `C:`. Distingue nei messaggi i tre casi di non conformità per facilitare il triage da TRMM senza aprire la macchina.

---

#### [[Libreria Script TRMM#Monitor WDAC Policy Status|Monitor: WDAC Policy Status]]

Verifica in cascata: prima che il file `.cip` sia presente nella cartella Active, poi che la versione corrisponda a quella attesa leggendo il file di stato locale.

**Parametri**: `ExpectedVersion` _(obbligatorio — es. `10.0.0.14`)_

> ⚠️ Prima di usare questo monitor sulla flotta, eseguire `WDAC: Deploy Policy` almeno una volta su ogni macchina per creare il file di stato. Le macchine deployate manualmente risulteranno non conformi al primo run — normale.

---

## 🚀 Fleet deployment — strategia

Il rollout sui ~46 endpoint va fatto con cautela: il software installato varia tra macchina e macchina. Una policy perfetta sul PC di test può bloccare qualcosa su un'altra macchina.

**Approccio consigliato:**

1. Su ogni macchina, partire con un periodo in ==audit mode== per raccogliere i file locali non coperti dalla policy base
2. Aggiungere i file, incrementare la versione, ricompilare
3. Passare in enforcement solo dopo che i log audit sono puliti
4. Usare `Monitor: WDAC Policy Status` come check continuativo post-rollout

**Sequenza script per ogni endpoint nuovo:**

|Ordine|Script|Note|
|---|---|---|
|1|`Defender: Deploy ASR Rules`|Mode = Enable|
|2|`Defender: Enable Controlled Folder Access`||
|3|`BitLocker: Enable`|Richiede ApiKey|
|4|`User: Revoke Admin`|Per l'utente standard|
|5|`WDAC: Deploy Policy`|Dopo audit e compilazione|

---

## 🪲 Gotcha e learnings

Le cose che ho imparato a mie spese — meglio non doverle riscoprire.

- **`wevtutil` non funziona in WinRE** — per analizzare i log da WinRE, copiare il file `.evtx` su USB con `copy` e analizzarlo offline
- **Virgolette in TRMM** — le virgolette curve (`"`) copiate da qualsiasi editor causano errori di parser PowerShell in TRMM. Usare sempre virgolette dritte, rimuovere i commenti inline nelle stringhe interpolate
- **`Get-Item` e AppxScan** — il wildcard match seleziona i package `neutral_split` prima di quelli `x64`. Filtrare esplicitamente con `*x64*`
- **`Get-ComputerInfo` su Home** — può riportare "Windows 10 Home" su alcune build. È un bug del registry noto, non un problema reale di versione OS
- **Policy GUID fisso** — il GUID `{966d1f08-...}` è scritto nel campo `PolicyID` dell'XML e ==non cambia tra versioni==. Cambierebbe solo ripartendo da zero con `New-CIPolicy` su un XML vuoto — scenario improbabile
- **BitLocker multiple protector** — se esistono più `RecoveryPassword` protector sullo stesso volume, il cast `[string]($keyRaw | Select-Object -First 1)` è necessario per evitare errori di tipo nella chiamata API TRMM