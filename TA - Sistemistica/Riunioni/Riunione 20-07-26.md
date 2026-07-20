### Da chiedere 

1. **Autorizzazione all'acquisto OVH Object Storage** — è il blocco principale. Costo trascurabile (parliamo di centesimi/mese per ~15 MB per backup, anche con retention lunga). Chi firma e con che tempi?
2. **Policy di retention aziendale** — quante copie/per quanto tempo conservare? Serve a dimensionare il bucket e configurare la rotazione.
3. **Chi custodisce le credenziali S3** e dove (password manager aziendale?). Rilevante perché il backup contiene il DB con le **chiavi di recovery BitLocker** — è un artefatto ad alta sensibilità.
4. **Vincoli su dove risiedono i dati** (stesso datacenter Frankfurt o region diversa?) e se il team security ha requisiti di cifratura a riposo oltre a quella nativa.
5. **Finestra per il crash test** — serve concordare quando simulare il disastro, ed eventualmente autorizzare una VM temporanea per il restore.

### Da presentare 

**Fatto:**

- **Backup**: primo backup completo eseguito e verificato (archivio integro, 27 componenti — DB TRMM, DB MeshCentral, certificati, config nginx/systemd, MeshCentral). Attualmente risiede solo sulla VM: è il gap che l'Object Storage chiude.
- **WDAC v17** in enforcement sulle due macchine di test, resto del pilota in audit mode.
- **Raccolta audit automatizzata**: policy TRMM che esegue l'export dei log CodeIntegrity su tutti gli agenti pilota, con recupero automatico per le macchine offline. Validata oggi, produce output.
- **BitLocker**: risolto su Ilaria, chiave sincronizzata nel custom field TRMM.
- **Defender**: rimosso Avira da Ilaria, risolto il passive mode che disattivava ASR, Behavior Monitoring e CFA. Proposta di standardizzazione: Defender come unico AV primario sulla flotta.
- **MeshCentral**: consenso utente irrinunciabile (`userConsentFlags: 56`).

**In corso / bloccato:**

- Validazione audit v19 → in attesa dei log di **Giovanni** (in ferie).
- Crash test → **bloccato** sul backup off-site.
- Migrazione da DuckDNS a dominio di proprietà (fix definitivo per il Take Control in iframe).
- Documentazione tecnica consolidata.

**Prossimi passi:** completamento pilota → enforcement fleet-wide sui 46 endpoint.