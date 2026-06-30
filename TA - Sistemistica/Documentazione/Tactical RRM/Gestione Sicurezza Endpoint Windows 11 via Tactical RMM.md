
Questo documento descrive l'infrastruttura di gestione endpoint basata su **Tactical RMM**, l'architettura di sicurezza implementata, le motivazioni delle scelte tecniche e le procedure operative — incluse quelle di errore e recovery.


---

## 1. Obiettivo del progetto

L'obiettivo è dotare una flotta di circa **46 endpoint Windows 11** (mix di edizioni Home e Pro) di un'infrastruttura di sicurezza a più livelli, gestita centralmente e a costo zero in termini di licenze. I requisiti principali:

- Impedire l'installazione e l'esecuzione di software non autorizzato da parte degli utenti standard
- Controllo remoto degli account (abilitazione, disabilitazione, privilegi)
- Cifratura dei dischi con escrow centralizzato delle chiavi di ripristino
- Possibilità di lockdown di dispositivi smarriti o rubati
- Monitoraggio continuo della conformità

Il vincolo più rilevante è la presenza di **Windows 11 Home** su parte della flotta: questa edizione non supporta né l'aggiunta a un dominio Active Directory, né alcune tecnologie di sicurezza enterprise. Le scelte architetturali derivano in larga parte da questo vincolo.

---

## 2. Tactical RMM — la piattaforma di gestione

### Cos'è

**Tactical RMM (TRMM)** è una piattaforma open source di **Remote Monitoring & Management** — categoria di strumenti che permette a un team IT di monitorare e amministrare da remoto una flotta di macchine. È l'alternativa **gratuita** e **self-hosted** a prodotti commerciali come NinjaRMM o Datto.

Concretamente, TRMM permette di eseguire script da remoto su tutti gli endpoint, monitorarne lo stato, gestire gli aggiornamenti e accedere ai desktop. È il punto di controllo centrale da cui passa tutto il progetto.

*Dashboard TRMM con due Agenti*
![[Pasted image 20260630122709.png]]

> ❓ **Perché non un dominio Active Directory?** Sarebbe la scelta naturale in un ambiente Windows, ma richiede Windows Pro su tutti i client. Con macchine Home in flotta non è praticabile. TRMM lavora tramite un ==agente installato su ogni macchina==, indipendentemente dall'edizione, e non richiede join a dominio.


### Architettura

TRMM è un insieme di componenti che girano su un'unica VM server:

| Componente      | Ruolo                                                  |
| --------------- | ------------------------------------------------------ |
| **Django**      | Backend — espone le API per frontend e agenti          |
| **Vue.js**      | Frontend — la dashboard web                            |
| **PostgreSQL**  | Database principale                                    |
| **Redis**       | Cache e broker di messaggi                             |
| **Celery**      | Esecuzione task asincroni (check, alert, patch)        |
| **NATS**        | Canale di comunicazione in tempo reale server ↔ agenti |
| **MeshCentral** | Accesso remoto: desktop, shell, file browser           |
| **Nginx**       | Reverse proxy e terminazione TLS                       |
*Esempi schermata di Mesch Central:*
![[Pasted image 20260630123705.png]]
![[Pasted image 20260630123818.png]]

La comunicazione **server ↔ agente** avviene tramite ==**NATS**==, un sistema di messaggistica a bassa latenza. Gli agenti online ricevono i comandi immediatamente; quelli offline li eseguono alla successiva riconnessione. 
Questo significa che un comando inviato a una macchina spenta non va perso — viene accodato.

### L'agente

Su ogni endpoint è installato l'**agente TRMM**, un piccolo servizio che mantiene la connessione con il server, esegue gli script richiesti e invia i dati di monitoraggio. Si installa tramite un comando PowerShell generato dalla dashboard, che associa la macchina a un client e a un sito.
![[Pasted image 20260630124050.png]]

### Nostra installazione di Test

|Componente|Valore|
|---|---|
|Server|OVH Frankfurt — Ubuntu 22.04 LTS — `213.32.30.52`|
|Pannello web|`https://ta-tactical-rmm.duckdns.org`|
|API|`https://ta-tactical-api.duckdns.org`|
|MeshCentral|`https://ta-tactical-mesh.duckdns.org`|
|Client / Site|Technology Advising / Sede Principale|

> ⚠️ **Nota sui certificati TLS.** I tre sottodomini sono host **DuckDNS** distinti, non un wildcard (caratteri jolly). Durante il setup si è verificato un errore di validazione del certificato perché quello iniziale copriva `*.ta-tactical.duckdns.org`, che non corrisponde ai nomi effettivi (`ta-tactical-rmm`, `ta-tactical-api`, `ta-tactical-mesh`). La soluzione è stata emettere un certificato multi-dominio con *Let's Encrypt* che elenca esplicitamente i tre nomi. Da ricordare in caso di rinnovo o migrazione.

---

## 3. Architettura di sicurezza — visione d'insieme

La sicurezza degli endpoint è costruita a **strati** (defense in depth): se un livello viene aggirato, gli altri restano attivi. 
I cinque strati:

| 🛡️Strato                      | Tecnologia                   | Funzione                                             |
| ------------------------------ | ---------------------------- | ---------------------------------------------------- |
| Controllo applicazioni         | **WDAC**                     | Blocca l'esecuzione di software non autorizzato      |
| Riduzione superficie d'attacco | **ASR**                      | Blocca vettori di attacco noti                       |
| Protezione cartelle            | **Controlled Folder Access** | Impedisce modifiche non autorizzate a file sensibili |
| Cifratura disco                | **BitLocker**                | Protegge i dati su dispositivi persi o rubati        |
| Controllo accessi              | **Account & privilegi**      | Gestione remota di utenti e diritti amministrativi   |

Ogni strato è descritto in dettaglio nelle sezioni seguenti, con la spiegazione del perché è stato scelto e come si comporta.

> ❓ **Questi strati sono tutti nella stessa policy?** **No**. ==Solo *WDAC* vive in un file di policy dedicato==. *ASR* e *Controlled Folder Access* sono impostazioni di Windows Defender. *BitLocker* è una funzionalità di sistema. Gli account sono gestiti via script. Sono cinque meccanismi indipendenti, configurati separatamente e che persistono separatamente.

---

## 4. WDAC — Controllo delle applicazioni

### Cos'è

**WDAC (Windows Defender Application Control)** è la tecnologia Microsoft che permette di definire esattamente quali applicazioni possono essere eseguite su una macchina. ==Tutto ciò che non è **esplicitamente autorizzato** dalla policy viene bloccato a livello di sistema operativo== — prima ancora che il programma parta.

È il cuore del progetto: è lo strato che impedisce a un utente di eseguire software arbitrario, installer non approvati o malware.

> ❓ **Perché WDAC e non AppLocker?** **AppLocker** è la tecnologia di application control più conosciuta, ma le sue regole non vengono applicate su Windows Home — sarebbe inerte su parte della flotta. WDAC invece funziona su tutte le edizioni di Windows 11, comprese le Home. Questo lo rende l'unica scelta che garantisce un comportamento uniforme su tutto il parco macchine.

### Audit mode ed Enforcement mode

WDAC opera in due modalità, ed è essenziale capirne la differenza:

- **Audit mode** 📝 — la policy non blocca nulla. Ogni volta che un'applicazione _sarebbe_ stata bloccata, WDAC si limita a ==registrare un evento== nel **log** di sistema. Serve per costruire e testare la policy senza interrompere il lavoro degli utenti.
- **Enforcement mode** 🛡️ — la policy ==blocca attivamente== tutto ciò che non è autorizzato. È la modalità di produzione.

La **strategia** di lavoro consiste nel partire sempre in audit mode, raccogliere per un periodo gli eventi registrati per capire quale software legittimo viene usato sulla macchina, aggiungere quel software alla policy, e solo allora passare in enforcement. Questo evita di bloccare applicazioni necessarie.

> **Stato attuale**: la policy è in **enforcement mode** sulle macchine di test, alla versione `10.0.0.14`, verificata sia su account amministrativo sia su account standard.

### ISG — la deroga basata sulla reputazione 🪪

**ISG (Intelligent Security Graph)** è un servizio cloud Microsoft che assegna agli eseguibili un punteggio di reputazione basato sulla loro diffusione mondiale e sul loro storico. Nella nostra policy ISG è **abilitato**, il che introduce un terzo canale di autorizzazione: oltre a "in whitelist" e "bloccato", ==un file può passare se ISG lo considera affidabile== (firmato e molto diffuso) anche senza essere esplicitamente autorizzato.

**Perché è abilitato**: senza ISG, i numerosi driver e componenti di sistema non esplicitamente coperti dalla policy causerebbero blocchi a livello di driver, fino al **boot failure** (la macchina non si avvia). ISG li lascia passare automaticamente.

**Il trade-off**: come effetto collaterale, software commerciale legittimo, firmato e diffuso (es. Firefox, VLC) passa anche senza autorizzazione esplicita. Per il nostro contesto — flotta aziendale con utenti standard e WDAC come uno strato tra molti — è un compromesso accettato consapevolmente. In un ambiente ad altissima sicurezza (air-gapped, regolamentato) ISG andrebbe invece disabilitato.

> ❓ **Dove si configura ISG?** . È una regola scritta nell'*XML* della **policy** (`Enabled:Intelligent Security Graph Authorization`), presente fin dalla versione iniziale e propagata a tutte le versioni successive tramite il merge.

### Il vincolo della compilazione

Creare e aggiornare una policy WDAC richiede il modulo PowerShell **ConfigCI** (`New-CIPolicy`, `Merge-CIPolicy`, `ConvertFrom-CIPolicy`), che è disponibile **solo su Windows Pro/Enterprise**.
 Le macchine Home **non** possono compilare policy.

Il _deploy_ invece avviene tramite **CiTool**, uno strumento da riga di comando integrato in Windows 11 in tutte le edizioni, Home compresa.

Questa asimmetria definisce l'intero **workflow**: ==si compila su una macchina Pro dedicata e si distribuisce il risultato alle macchine Home==.

|Macchina|Ruolo|Edizione|
|---|---|---|
|`TA-TEST`|Compilazione policy|Windows 11 Pro|
|Lenovo V15 G4 AMN|Test / cavia|Windows 11 Home|

### Anatomia dei file di policy

Durante il lavoro si incontrano tre estensioni, ed è utile sapere cosa rappresentano:

|Estensione|Cos'è|Dove|
|---|---|---|
|`.xml`|La policy in forma leggibile e modificabile|Su TA-TEST, in `C:\WDAC\`|
|`.p7b`|La policy compilata in formato binario, pronta per il deploy|Compilata su TA-TEST, trasferita al target|
|`.cip`|La policy installata e attiva, letta da Windows all'avvio|`C:\Windows\System32\CodeIntegrity\CiPolicies\Active\`|

Il flusso è: si modifica l'`.xml`, lo si compila in `.p7b`, e CiTool lo installa convertendolo in `.cip`. Il nome del file `.cip` è il **GUID** (identificativo univoco) della policy.

> 💡 Il GUID della nostra policy è `{966d1f08-bcea-48c4-bc3a-6651c21e4090}` e **non cambia** tra versioni: è scritto nel campo `PolicyID` dell'XML e si propaga immutato. Cambierebbe solo creando una policy da zero — scenario che non rientra nel workflow normale.

### Il Workflow di aggiornamento policy 🔃

Ogni volta che si deve aggiungere software alla policy, si segue questo ciclo. 
Le fasi che usano `ConfigCI` (1, 2, 3) avvengono su ***TA-TEST***; il deploy (5, 6) sulla **macchina target**.

**Fase 0 — Ricerca file (se app Store)** 🔍 Se il software da aggiungere è un'app Microsoft Store, i suoi file vanno individuati in `C:\Program Files\WindowsApps\`. Attenzione: la ricerca deve filtrare esplicitamente la variante `x64`, perché le varianti `neutral_split` non contengono le DLL effettive e produrrebbero regole inutili.

**Fase 1 — Lettura log audit** 📋 Si esegue lo script [[Libreria Script TRMM#WDAC Export Audit Log|WDAC: Export Audit Log]], che estrae dal log di sistema gli eventi di blocco (Event ID 3076) e li salva in un file leggibile. Questo dice esattamente quale software è stato usato ma non è ancora autorizzato.

**Fase 2 — Raccolta file** 📂 I file da autorizzare vengono raccolti in `C:\WDAC\AppxScan\` sulla macchina di test.

**Fase 3 — Trasferimento al compilatore** 📤 I file vengono trasferiti a TA-TEST via **MeshCentral** (il modulo di accesso remoto e file transfer di TRMM), eventualmente compressi in RAR.

**Fase 4 — Compilazione** ⚙️ Su TA-TEST si eseguono in sequenza, manualmente in PowerShell, `New-CIPolicy` (scansione dei nuovi file) e `Merge-CIPolicy` (unione con la policy esistente). Si crea sempre un file di versione nuova senza sovrascrivere la precedente, così da mantenere un backup. Infine si esegue lo script [[Libreria Script TRMM#WDAC Compile Enforcement Policy|WDAC: Compile Enforcement Policy]], che rimuove la regola di audit mode, incrementa la versione e produce il `.p7b`.

> ❓ **Perché `New-CIPolicy` e `Merge-CIPolicy` non sono in uno script?** Perché richiedono decisioni diverse a ogni esecuzione (quali file scansionare, quali versioni unire, come nominare l'output). Automatizzarli non darebbe vantaggi: bisognerebbe comunque modificare i parametri ogni volta. Lo script copre solo l'ultimo passo, che è invece sempre identico.

**Fase 5 — Trasferimento al target** 📥 Solo il `.p7b` compilato viene trasferito alla macchina di destinazione via MeshCentral.

**Fase 6 — Deploy** 🚀 Sulla macchina target si esegue lo script [[Libreria Script TRMM#WDAC Deploy Policy|WDAC: Deploy Policy]], che installa la policy con CiTool, verifica che il `.cip` sia stato creato e registra lo stato del deploy in un file locale. Serve un riavvio perché la policy diventi effettiva.

**Fase 7 — Verifica** ✅ Si verifica il funzionamento su account amministrativo e standard.

---

## 5. ASR — Attack Surface Reduction

### Cos'è

**ASR (Attack Surface Reduction)** è un insieme di regole di **Windows Defender** — l'antivirus integrato di Windows — che bloccano comportamenti tipicamente associati ad attacchi, indipendentemente dal file specifico. Invece di chiedersi "questo file è malevolo?", le regole ASR si chiedono "questa azione è sospetta?".

Esempi di comportamenti bloccati: una macro di Office che lancia un eseguibile, un tentativo di furto di credenziali dalla memoria, l'esecuzione di script offuscati, l'avvio di programmi da una chiavetta USB.

Sulla flotta sono attive **9 regole ASR** in modalità di blocco.

### Modalità operative

Lo script [[Libreria Script TRMM#Defender Deploy ASR Rules|Defender: Deploy ASR Rules]] gestisce tre modalità:

|Modalità|Comportamento|Quando|
|---|---|---|
|`Enable`|Blocca attivamente|Produzione (default)|
|`AuditMode`|Registra senza bloccare|Diagnosi di un falso positivo|
|`Disable`|Rimuove le regole|Disattivazione mirata|

Anche ASR ha quindi un concetto di audit, analogo a WDAC: se si sospetta che una regola stia bloccando un'applicazione legittima, si passa temporaneamente in `AuditMode`, si osservano i log e si decide se aggiungere un'eccezione. A differenza di WDAC, il cambio di modalità è immediato e non richiede riavvio né ricompilazione.

---

## 6. Controlled Folder Access

### Cos'è

**Controlled Folder Access (CFA)** è una funzione di Windows Defender che protegge cartelle sensibili (Documenti, Desktop, ecc.) impedendo che applicazioni non autorizzate vi scrivano. È pensata principalmente come difesa contro i ransomware, che cifrano i file dell'utente.

> ❗ **Differenza importante rispetto a WDAC.** CFA controlla la _scrittura_ nelle cartelle protette, non l'_esecuzione_ dei programmi. Un'app può essere autorizzata a girare da WDAC ma comunque bloccata da CFA se tenta di scrivere in una cartella protetta senza esserne autorizzata. Sono due controlli ortogonali.

### Gestione della whitelist

Lo script [[Libreria Script TRMM#Defender Enable Controlled Folder Access|Defender: Enable Controlled Folder Access]] si limita ad **abilitare** la protezione. Non autorizza alcuna applicazione.

Quando un'app legittima viene bloccata da CFA (es. un gestionale che non riesce a salvare un documento), va autorizzata manualmente indicando il **percorso esatto dell'eseguibile** — non sono ammessi caratteri jolly (wildcard). L'autorizzazione si dà con `Add-MpPreference -ControlledFolderAccessAllowedApplications` seguito dal path completo del `.exe`.

> ❓ **La whitelist è vuota, è un problema?** No. Attualmente nessuna applicazione è in whitelist su nessuna macchina, ed è lo stato atteso: significa semplicemente che finora nessun software legittimo è stato bloccato da CFA. La whitelist va popolata solo quando e se si presenta un blocco effettivo.

---

## 7. BitLocker — Cifratura disco

### Cos'è

**BitLocker** è la tecnologia di cifratura disco di Windows. Cifra l'intero volume di sistema, rendendo i dati illeggibili senza la chiave corretta. Protegge contro l'accesso fisico ai dati in caso di furto o smarrimento del dispositivo.

In caso di problemi di avvio o cambio hardware, BitLocker richiede una **Recovery Key** — una password numerica di ripristino. Se questa chiave viene persa, i dati sono irrecuperabili: per questo la gestione centralizzata delle chiavi è critica.

### Specificità di Windows 11 Home

Windows 11 Home non espone BitLocker nell'interfaccia grafica e usa il nome **Device Encryption** per una versione semplificata. Tuttavia, i comandi PowerShell di BitLocker funzionano anche su Home, ed è tramite questi che gli script operano. In pratica, la cifratura è gestibile via TRMM indipendentemente dall'edizione.

### Escrow delle chiavi su TRMM

Le Recovery Key non vanno mai lasciate solo sulla macchina. Vengono salvate centralmente su TRMM in un **Custom Field** — un campo dati personalizzato associato a ciascun agente.

|Campo|Valore|
|---|---|
|Nome|`bitlocker_recovery_key`|
|ID|`1`|
|Tipo|Testo|

In questo modo, se una macchina richiede la Recovery Key (es. durante una procedura di recovery WDAC), questa è recuperabile dalla dashboard TRMM senza dipendere dalla macchina stessa.

### Script BitLocker

|Script|Funzione|
|---|---|
|[[Libreria Script TRMM#BitLocker Check Status\|BitLocker: Check Status]]|Verifica lo stato, sola lettura|
|[[Libreria Script TRMM#BitLocker Enable\|BitLocker: Enable]]|Attiva la cifratura e salva la chiave su TRMM|
|[[Libreria Script TRMM#BitLocker Disable\|BitLocker: Disable]]|Disattiva e decifra (solo manutenzione)|
|[[Libreria Script TRMM#BitLocker Store Recovery Key\|BitLocker: Store Recovery Key]]|Sincronizza su TRMM una chiave già esistente|

> ❓ **Quando serve `Store Recovery Key` se `Enable` salva già la chiave?** Per le macchine già cifrate prima dell'installazione dell'agente TRMM, la cui chiave non è ancora su TRMM. Lo script la estrae e la salva senza toccare la cifratura esistente.

> 💡 **Nota tecnica.** Se su un volume esistono più chiavi di ripristino (più "protector"), gli script selezionano sempre la prima. Questo evita un errore noto nella chiamata API, che rifiuta un valore non testuale se gli viene passato un elenco di chiavi invece di una singola.

---

## 8. AppLocker — stato e ruolo residuo

**AppLocker** è la tecnologia di application control precedente a WDAC, funzionante solo su Windows Pro/Enterprise. Nel progetto **non è uno strato attivo di sicurezza**: dove WDAC è attivo, AppLocker è ridondante, e su Home è comunque inerte.

Sulla sola macchina `TA-TEST` (Pro) AppLocker è configurato in **AuditOnly** (registra senza bloccare). Questo stato è il risultato della risoluzione di un problema: una precedente configurazione di AppLocker in enforcement aveva bloccato componenti della shell di Windows, rendendo inutilizzabili Menu Start e Impostazioni. Riportandolo in AuditOnly il problema è stato risolto.

Gli script [[Libreria Script TRMM#AppLocker Set Enforcement|AppLocker: Set Enforcement]] e [[Libreria Script TRMM#AppLocker Set AuditOnly|AppLocker: Set AuditOnly]] restano in libreria per gestione di TA-TEST, ma non fanno parte del deploy standard sulla flotta.

> 💡 Il servizio `AppIDSvc`, da cui AppLocker dipende, non sempre si arresta completamente da TRMM per via di dipendenze attive. Non è un problema: in AuditOnly il servizio è di fatto innocuo.

---

## 9. Gestione account e privilegi

Quattro script coprono il controllo remoto degli account locali. Sono lo strumento per il lockdown di dispositivi e per la standardizzazione dei diritti utente.

|Script|Funzione|
|---|---|
|[[Libreria Script TRMM#User Enable Account\|User: Enable Account]]|Abilita un account locale|
|[[Libreria Script TRMM#User Disable Account\|User: Disable Account]]|Disabilita un account e chiude la sessione attiva|
|[[Libreria Script TRMM#User Grant Admin\|User: Grant Admin]]|Concede privilegi amministrativi|
|[[Libreria Script TRMM#User Revoke Admin\|User: Revoke Admin]]|Revoca i privilegi e applica restrizioni|

Lo script `User: Revoke Admin` fa più della semplice rimozione dal gruppo Administrators: applica anche il blocco dell'installer MSI per gli utenti non amministratori, disabilita l'autorun dei supporti rimovibili e blocca la disinstallazione dei programmi. Il controllo dell'esecuzione degli eseguibili è invece delegato interamente a WDAC.

> 💡 La versione originale di questo script conteneva una sezione basata su **SRP (Software Restriction Policy)**, una tecnologia di controllo applicazioni ormai deprecata. È stata rimossa perché ridondante con WDAC e potenzialmente conflittuale.

---

## 10. Monitoraggio della conformità

Tre script sono progettati per girare periodicamente su tutta la flotta tramite **Automation Policy** (la funzione di TRMM che pianifica l'esecuzione automatica di script su gruppi di macchine). Seguono una convenzione comune: terminano con codice di uscita `0` se la macchina è conforme, `1` se non lo è. TRMM interpreta il codice `1` come condizione di alert.

|Script|Verifica|
|---|---|
|[[Libreria Script TRMM#Monitor Store Apps\|Monitor: Store Apps]]|App Microsoft Store non autorizzate|
|[[Libreria Script TRMM#Monitor BitLocker Compliance\|Monitor: BitLocker Compliance]]|BitLocker attivo e disco cifrato|
|[[Libreria Script TRMM#Monitor WDAC Policy Status\|Monitor: WDAC Policy Status]]|Policy WDAC presente e alla versione attesa|

`Monitor: WDAC Policy Status` opera in due passaggi: verifica prima la presenza del file `.cip`, poi confronta la versione registrata localmente al momento del deploy con quella attesa. Se il file di stato locale manca — perché la macchina è stata aggiornata manualmente senza usare lo script di deploy standard — la macchina viene segnalata come non conforme.

> ⚠️ **Prima di attivare questo monitor sulla flotta**: ogni macchina deve aver eseguito almeno una volta `WDAC: Deploy Policy`, che crea il file di stato. Le macchine deployate manualmente in passato risulteranno non conformi al primo controllo. È il comportamento atteso e si risolve con un deploy pulito tramite lo script.

---

## 11. Casistiche di errore e recovery

Questa sezione raccoglie le situazioni anomale, come riconoscerle e come affrontarle.

### 11.1 Boot failure dopo deploy WDAC 🚨

**Sintomo**: dopo il deploy di una nuova policy e il riavvio, la macchina non completa l'avvio. Causa tipica: la policy blocca un driver o un componente di sistema necessario al boot.

**Riconoscimento**: la macchina si blocca o entra in loop di avvio subito dopo un deploy WDAC.

**Risoluzione** — procedura manuale via **WinRE** (Windows Recovery Environment, l'ambiente di ripristino di Windows):

1. Accedere a WinRE (tenere Shift premuto e selezionare Riavvia, oppure interrompere il boot tre volte consecutive)
2. Aprire **Opzioni avanzate → Prompt dei comandi**
3. Se richiesta, inserire la Recovery Key di BitLocker, recuperandola dal Custom Field `bitlocker_recovery_key` su TRMM
4. Eliminare il file della policy attiva con il comando `del`, puntando al percorso `C:\Windows\System32\CodeIntegrity\CiPolicies\Active\{966d1f08-bcea-48c4-bc3a-6651c21e4090}.cip`
5. Riavviare con `wpeutil reboot`

Al riavvio la macchina parte senza WDAC. Si analizzano i log, si corregge la policy e si ripete il deploy.

> ⚠️ **Due insidie in WinRE.** Primo: la lettera del disco Windows in WinRE può non essere `C:` — va verificata (es. con `dir C:\Windows`) prima di eseguire il comando di eliminazione. Secondo: lo strumento `wevtutil` per leggere i log degli eventi non funziona in WinRE; se servono i log, i file `.evtx` vanno copiati con `copy` su un supporto esterno e analizzati altrove.

### 11.2 Policy problematica con macchina ancora avviata

**Sintomo**: una policy causa malfunzionamenti (app legittime bloccate) ma la macchina si avvia e resta raggiungibile da TRMM.

**Risoluzione**: in questo caso non serve WinRE. Si esegue da remoto lo script [[Libreria Script TRMM#WDAC Remove Active Policy|WDAC: Remove Active Policy]], che rimuove il file `.cip`. La rimozione diventa effettiva al riavvio successivo, lasciando il tempo di valutare prima di riavviare.

> ⚠️ **Bug storico, ora risolto.** La versione originale di questo script cercava di rimuovere un file con estensione `.p7b`, mentre la policy attiva ha estensione `.cip`. Lo script era quindi completamente inerte: sembrava funzionare ma non rimuoveva nulla. La versione attuale punta al file `.cip` corretto tramite GUID esplicito. Da tenere presente se si incontrano riferimenti alla vecchia versione.

### 11.3 App Store bloccata anche dopo l'aggiunta alla policy

**Sintomo**: si è aggiunta un'app Microsoft Store alla policy, ma continua a essere bloccata dopo il deploy.

**Causa**: in fase di raccolta file sono state scansionate le cartelle `neutral_split` del pacchetto invece della variante `x64`. Le prime non contengono le DLL effettivamente eseguite, quindi la policy autorizza file che non vengono mai usati.

**Risoluzione**: ripetere la raccolta file filtrando esplicitamente la variante `x64` della cartella dell'app in `WindowsApps`, quindi ricompilare e rideployare.

### 11.4 Errori di parser PowerShell negli script TRMM

**Sintomo**: uno script che dovrebbe essere corretto fallisce in TRMM con errori di sintassi PowerShell.

**Causa**: le virgolette tipografiche "curve" (`" "`), introdotte automaticamente da molti editor di testo durante il copia-incolla, non sono valide in PowerShell, che richiede virgolette dritte (`"`).

**Risoluzione**: prima di salvare uno script in TRMM, sostituire tutte le virgolette curve con virgolette dritte. In fase di standardizzazione sono stati anche rimossi i commenti inline con stringhe interpolate, più soggetti a questo problema.

### 11.5 Falso positivo sulla versione di Windows

**Sintomo**: il comando `Get-ComputerInfo` riporta "Windows 10 Home" su una macchina che esegue Windows 11 Home.

**Causa**: bug noto del registro su alcune build di Windows 11. Non riflette la versione reale del sistema.

**Risoluzione**: nessuna — è un errore di reporting, non un problema reale. Da ignorare, ma da conoscere per non essere fuorviati durante le diagnosi.

### 11.6 Problemi di connessione e servizi TRMM

|Sintomo|Causa probabile|Azione|
|---|---|---|
|Agente non si connette|DNS o firewall|Verificare raggiungibilità della porta 443|
|Dashboard non raggiungibile|Servizi server fermi|Riavviare i servizi `rmm` e `nginx` sul server|
|Accesso remoto non funziona|Problema MeshCentral|Eseguire `check_mesh` sul server|
|Script termina con codice 98|Timeout di esecuzione|Non è un errore dello script: aumentare il timeout o suddividerlo|
|Check non ricevuti dal server|Servizio NATS fermo|Riavviare NATS sul server|

---

## 12. Deploy sulla flotta — strategia

Il rollout sui ~46 endpoint è la fase più delicata e va affrontato con cautela, perché **il software installato varia da macchina a macchina**. Una policy perfettamente funzionante sulla macchina di test può bloccare applicazioni presenti solo su altri endpoint.

### Approccio raccomandato

Non si distribuisce la policy in enforcement direttamente su tutte le macchine. Il principio è far passare ogni macchina attraverso lo stesso ciclo audit → enforcement già usato in fase di test:

1. Distribuire la policy in **audit mode** su un gruppo ridotto di macchine
2. Lasciar passare un periodo di osservazione, raccogliendo gli eventi di blocco con `WDAC: Export Audit Log`
3. Integrare nella policy il software legittimo emerso dai log
4. Passare quel gruppo in enforcement solo dopo aver verificato che i log siano puliti
5. Procedere per gruppi successivi, usando `Monitor: WDAC Policy Status` come verifica continua

### Sequenza di hardening per un nuovo endpoint

Per portare un endpoint dallo stato iniziale a quello protetto, gli script vanno eseguiti in quest'ordine:

|Ordine|Script|Note|
|---|---|---|
|1|`Defender: Deploy ASR Rules`|Modalità `Enable`|
|2|`Defender: Enable Controlled Folder Access`|—|
|3|`BitLocker: Enable`|Richiede la API Key|
|4|`User: Revoke Admin`|Sull'account dell'utente standard|
|5|`WDAC: Deploy Policy`|Dopo il ciclo audit/compilazione|

> ❓ **Cos'è la "API Key" richiesta da alcuni script?** È una chiave di autenticazione generata da TRMM (in _Settings → API Keys_) che autorizza uno script a comunicare con l'API del server — per esempio per scrivere la Recovery Key di BitLocker in un Custom Field. Va passata allo script come parametro al momento dell'esecuzione e **non va mai scritta dentro il codice**, così da non finire in versioni salvate o condivise.

---

## 13. Riepilogo dei riferimenti

|Elemento|Valore|
|---|---|
|Policy GUID WDAC|`{966d1f08-bcea-48c4-bc3a-6651c21e4090}`|
|Versione policy attuale|`10.0.0.14`|
|Cartella policy attiva|`C:\Windows\System32\CodeIntegrity\CiPolicies\Active\`|
|Cartella di lavoro WDAC|`C:\WDAC\`|
|Custom Field BitLocker|`bitlocker_recovery_key` (ID 1)|
|Macchina di compilazione|`TA-TEST` (Windows 11 Pro)|
|Event ID blocco WDAC|3076|
|Codice uscita conforme (monitor)|0|
|Codice uscita non conforme (monitor)|1|
|Codice timeout script TRMM|98|

---

## Documenti collegati

- [[Libreria Script TRMM]] — codice completo e parametri di ogni script
- [[Tactical RMM]] — riferimento generale sulla piattaforma
- [[ISG — Intelligent Security Graph]] — approfondimento sul servizio di reputazione
- [[CiTool]] — approfondimento sullo strumento di deploy