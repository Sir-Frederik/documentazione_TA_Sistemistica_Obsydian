

Procedura completa dalla compilazione del `.cip` su TA-TEST fino all'applicazione sugli agent TRMM. Vale per ogni nuova versione della policy: cambia solo il numero di versione.

---

## Riferimenti fissi

|Elemento|Valore|
|---|---|
|GUID policy|`{966d1f08-bcea-48c4-bc3a-6651c21e4090}`|
|Macchina di compilazione|**TA-TEST** (Win 11 Pro — unica con modulo ConfigCI)|
|Cartella di lavoro|`C:\WDAC\`|
|Server|`ubuntu@213.32.30.52` (OVH Frankfurt)|
|Cartella pubblicata|`/srv/wdac/`|
|URL policy|`https://ta-tactical-rmm.duckdns.org/wdac/%7B966d1f08-bcea-48c4-bc3a-6651c21e4090%7D.cip`|

> [!note] Perché `%7B` e `%7D` Le graffe non sono caratteri ammessi negli URL e vanno codificate (==URL encoding==): `{` → `%7B`, `}` → `%7D`. Il trattino invece è sicuro e resta invariato.

---

## Prerequisito — il file `.cip` mantiene sempre lo stesso nome

Il binario si chiama sempre `{966d1f08-...}.cip`, a ogni versione. È ==voluto==:

- sovrascrive il file precedente sul server, quindi l'URL non cambia mai
- lo script di deploy funziona senza modifiche
- CiTool aggiorna la policy esistente invece di affiancarne una seconda

La versione si distingue dal contenuto (campo `Version` nell'XML), non dal nome file.

---

## Passo 1 — Compilazione del `.cip` (su TA-TEST)

Da eseguire via TRMM (gli script girano come SYSTEM, quindi niente problemi di permessi admin).

```powershell
ConvertFrom-CIPolicy -XmlFilePath "C:\WDAC\policy_final_vNN.xml" -BinaryFilePath "C:\WDAC\{966d1f08-bcea-48c4-bc3a-6651c21e4090}.cip"
Get-Item "C:\WDAC\{966d1f08-bcea-48c4-bc3a-6651c21e4090}.cip" | Select-Object Name, Length, LastWriteTime
```

Sostituire `vNN` con la versione corrente. Controllare che `LastWriteTime` sia di adesso: se è vecchia, la compilazione non ha sovrascritto.

---

## Passo 2 — TA-TEST → PC locale

MeshCentral → TA-TEST → tab **Files** → `C:\WDAC\` → scarica il `.cip`.

> [!warning] Il terminale Mesh non accetta input interattivo Non si può digitare `yes` né incollare testo. Quindi ==niente `scp` dal terminale Mesh==: per spostare file si usa la tab Files.

---

## Passo 3 — PC locale → server (`pscp`)

Da **PowerShell sul proprio PC** (non sul server, non da Mesh).

```powershell
& "C:\Program Files\PuTTY\pscp.exe" -i "C:\Users\feder\Desktop\Putty\TA_FreeIPA.ppk" "C:\WDAC\{966d1f08-bcea-48c4-bc3a-6651c21e4090}.cip" ubuntu@213.32.30.52:/tmp/
```

Poi inserire la ==passphrase== della chiave quando richiesta (il prompt può non essere evidente: se sembra bloccato, probabilmente sta aspettando lì).

> [!bug] Errori tipici
> 
> - **`AmpersandNotAllowed`** → è stato copiato anche il prompt `PS C:\...>` insieme al comando. Incollare solo il comando.
> - **`Permission denied (publickey)`** → si sta usando `scp` di Windows invece di `pscp`: il server accetta ==solo chiave==, e la `.ppk` la legge solo PuTTY. Non serve avere una sessione PuTTY aperta, `pscp` apre la sua.
> - **`Could not resolve hostname c`** → `scp`/`pscp` interpreta tutto ciò che precede i `:` come host, e legge `C:` come hostname. Usare un percorso relativo dopo un `cd`.

---

## Passo 4 — Pubblicazione sul server (PuTTY)

```bash
sudo mv /tmp/{966d1f08-bcea-48c4-bc3a-6651c21e4090}.cip /srv/wdac/
sudo chown www-data:www-data /srv/wdac/*.cip
sudo chmod 644 /srv/wdac/*.cip
curl -I https://ta-tactical-rmm.duckdns.org/wdac/%7B966d1f08-bcea-48c4-bc3a-6651c21e4090%7D.cip
```

Nel `curl` verificare:

- `HTTP/1.1 200 OK`
- `Content-Type: application/octet-stream` (se esce `text/html` è una pagina di errore, non il file)
- ==`Content-Length` uguale alla dimensione del `.cip` appena compilato== — è la prova che la sovrascrittura è avvenuta e non si sta servendo la versione vecchia

---

## Passo 5 — Deploy sugli agent

Script TRMM: **[[Libreria Script TRMM#WDAC Deploy Policy from URL|WDAC: Deploy Policy from URL]]**

Script Arguments: ==lasciare vuoto== (l'URL della policy corrente è il default nel `param()`). Compilare `-PolicyUrl` solo per una policy con GUID diverso.

Lo script: scarica il `.cip` → controlla che sia > 1000 byte (scarta pagine di errore HTML) → applica con `CiTool --update-policy`. ==Nessun reboot richiesto== per l'aggiornamento di una policy già presente.

---

## Passo 6 — Verifica

Script TRMM: **[[Libreria Script TRMM#WDAC Check Policy Version|WDAC: Check Policy Version]]** su ogni agent.

Atteso: `Versione : 10.0.0.NN` corrispondente a quella appena deployata.

> [!info] `IsEnforced: True` non significa enforcement Nel linguaggio di CiTool quel campo indica solo "policy ==attiva sul sistema==". Per sapere se blocca davvero servono gli eventi:
> 
> - **3076** = audit (segnala, non blocca)
> - **3077** = block (enforcement reale)
> 
> ```powershell
> Get-WinEvent -LogName "Microsoft-Windows-CodeIntegrity/Operational" -MaxEvents 20 |
>     Where-Object { $_.Id -in 3076, 3077 } | Select-Object TimeCreated, Id
> ```

---

## Ciclo completo (contesto)

Questa pagina copre i passi 4–6 del workflow. Il ciclo intero:

1. **Audit** — la policy in audit mode registra eventi 3076
2. **Analisi** — [[Libreria Script TRMM#WDAC Export Audit Log Details|WDAC: Export Audit Log Details]] estrae file, hash, publisher
3. **Raccolta file** — se serve una regola Hash: [[Libreria Script TRMM#Tools Robocopy and Zip Locked Folder|Tools: Robocopy and Zip Locked Folder]] sull'agent, poi trasferimento su TA-TEST
4. **Regole + merge** — `New-CIPolicy` / `New-CIPolicyRule`, poi `Merge-CIPolicy` (solo su TA-TEST)
5. **Deploy** — ==questa pagina==
6. Ritorno al punto 1 finché gli audit non sono puliti → poi enforcement

---

## Errori già commessi (da non ripetere)

> [!danger] Verificare SEMPRE il merge prima di compilare Nella **v16** sono entrati XML ==vuoti o mancanti== senza che `Merge-CIPolicy` desse errore: la policy è stata deployata senza le regole di YourPhone e mscorsvw, e gli audit continuavano a segnalarli. Causa: alcuni XML erano stati cancellati per liberare spazio, altri erano gusci vuoti da ~1285 byte (scansione su cartella vuota → `New-CIPolicy` non dà errore).
> 
> **Controlli obbligatori prima di `ConvertFrom-CIPolicy`:**
> 
> - ogni XML sorgente esiste e ha una dimensione sensata (un XML con regole reali pesa ==decine o centinaia di KB==; ~1285 byte = vuoto)
> - dopo il merge, elencare le regole effettivamente presenti:
>     
>     ```powershell
>     Select-String -Path "C:\WDAC\policy_final_vNN.xml" -Pattern 'FilePath="' -SimpleMatch | ForEach-Object {    if ($_.Line -match 'FilePath="([^"]+)"') { $matches[1] }}
>     ```
>     
> - verificare a campione gli hash attesi con `Select-String -SimpleMatch -Quiet`

> [!danger] `New-CIPolicy` non sovrascrive in modo affidabile Rigenerando `policy_filepath_apps.xml` da 2 a 5 regole, il file è rimasto a 2 pur non dando errori — e il merge ha usato la versione vecchia. **Fare sempre `Remove-Item` del file prima di rigenerarlo.**

> [!warning] Virgolette curve Il copia-incolla da documenti può introdurre virgolette tipografiche che rompono il parser PowerShell in TRMM. Usare sempre virgolette dritte.

---

## Recovery da boot failure

Se una policy in enforcement impedisce l'avvio: procedura manuale via **WinRE**. Verificare la lettera del disco (in WinRE può essere diversa da `C:`), poi eliminare il `.cip` con `del` da `Windows\System32\CodeIntegrity\CiPolicies\Active\`. ==`wevtutil` non funziona in WinRE.==

---

## Collegamenti

- [[Libreria Script TRMM]]
- [[WDAC - Compilazione e Merge Policy]]
- [[Test Cavie TRMM - Anomalie]]