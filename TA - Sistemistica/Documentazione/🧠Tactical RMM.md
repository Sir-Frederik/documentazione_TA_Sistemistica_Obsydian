

> **Cos'è**: Strumento open source di **Remote Monitoring & Management (RMM)** per la gestione centralizzata di endpoint Windows, Linux e macOS. Alternativa gratuita a soluzioni commerciali come NinjaRMM, ConnectWise o Datto. Ideale per MSP e team IT interni con flotte di macchine da monitorare e gestire da remoto.

---
Le sue operazioni sono simili a [[🏦 SAMBA]] e [[FreeIPA]], ma non necessita di Windows Pro sui client.
## Architettura

Tactical RMM è composto da diversi componenti che lavorano insieme:

|Componente|Ruolo|
|---|---|
|**Django (backend)**|Cuore del sistema — serve le API per frontend e agenti|
|**Vue.js (frontend)**|Dashboard web accessibile via browser|
|**PostgreSQL**|Database principale|
|**Redis**|Cache e message broker per Celery|
|**Celery**|Gestione task asincroni (check, alert, patch)|
|**NATS**|Comunicazione in tempo reale server ↔ agenti|
|**MeshCentral**|Remote desktop, shell remota, file browser|
|**Nginx**|Reverse proxy + TLS termination|

> ℹ️ La comunicazione tra server e agenti avviene ==tramite **NATS**. ==Gli agenti online ricevono i comandi immediatamente; quelli ==offline li eseguono alla successiva connessione.
==
---

## Prerequisiti installazione

- VM dedicata (Ubuntu 22.04 LTS o Debian 11/12)
    
- Minimo 4 GB RAM
    
- Architettura x64
    
- Tre sottodomini DNS puntati all'IP pubblico del server
    

|Sottodominio|Uso|
|---|---|
|`rmm.azienda.com`|Dashboard web + comunicazione agenti|
|`api.azienda.com`|Backend API|
|`mesh.azienda.com`|MeshCentral|

> ⚠️ I tre sottodomini devono essere allo stesso livello DNS.

### Porte firewall

|Porta|Uso|
|---|---|
|`443/TCP`|Dashboard, API, MeshCentral|
|`22/TCP`|SSH|

---

## Installazione server

```bash
wget -O install.sh https://raw.githubusercontent.com/amidaware/tacticalrmm/master/install.sh
chmod +x install.sh
./install.sh
```

> ⚠️ Lo script va eseguito una sola volta su una VM pulita.

---

## Aggiornamento

```bash
wget -N https://raw.githubusercontent.com/amidaware/tacticalrmm/master/update.sh
chmod +x update.sh
./update.sh
```

---

## Backup

```bash
wget -N https://raw.githubusercontent.com/amidaware/tacticalrmm/master/backup.sh
chmod +x backup.sh
./backup.sh
```

Abilitazione backup automatici:

```bash
./backup.sh --schedule
```

Retention automatica:

- Giornalieri: 2 settimane
    
- Settimanali: 2 mesi
    
- Mensili: 1 anno
    

---

## Funzionalità principali

### Script

Supporta:

- PowerShell
    
- Batch
    
- Python
    
- Bash
    
- Nushell
    
- Deno
    

Modalità di esecuzione:

|Modalità|Comportamento|
|---|---|
|Wait for Output|Attende il completamento|
|Fire and Forget|Esegue senza attendere|
|Email Output|Invia il risultato via email|

---

### Check

Monitoraggio di:

- CPU
    
- RAM
    
- Disco
    
- Servizi Windows
    
- Event Log
    
- Script personalizzati
    

---

### Task

Esecuzione pianificata di script tramite scheduler.

---

### Patch Management

Gestione centralizzata degli aggiornamenti Windows.

---

### Alert

Notifiche tramite:

- Email
    
- SMS
    
- Webhook
    

---

### Remote Access (MeshCentral)

- Desktop remoto
    
- Shell remota
    
- File browser
    
- Registry editor
    
- Event viewer
    
- Gestione servizi
    

---

### Software (Chocolatey)

```powershell
choco install googlechrome -y
choco upgrade all -y
```

---

# Installazione Technology Advising

## Infrastruttura

|Componente|URL|
|---|---|
|Pannello Web|`https://ta-tactical-rmm.duckdns.org`|
|API|`https://ta-tactical-api.duckdns.org`|
|MeshCentral|`https://ta-tactical-mesh.duckdns.org`|
|VM Server|OVH Frankfurt — Ubuntu 22.04 LTS — `213.32.30.52`|

Client configurato:

- **Technology Advising**
    
- Site: **Sede Principale**
    
- Timezone: **Europe/Rome**
    

---

## Problema TLS riscontrato

### Errore

```text
tls: failed to verify certificate:
certificate is valid for *.ta-tactical.duckdns.org,
not ta-tactical-api.duckdns.org
```

### Causa

Il certificato installato copriva:

```text
*.ta-tactical.duckdns.org
```

mentre Tactical RMM utilizzava:

```text
ta-tactical-rmm.duckdns.org
ta-tactical-api.duckdns.org
ta-tactical-mesh.duckdns.org
```

I nomi non coincidevano.

### Risoluzione

```bash
sudo apt install python3-certbot-nginx -y

sudo certbot --nginx \
  -d ta-tactical-rmm.duckdns.org \
  -d ta-tactical-api.duckdns.org \
  -d ta-tactical-mesh.duckdns.org

sudo systemctl reload nginx
```

---

## Installazione agente Windows

1. Dashboard → **Agents → Install Agent**
    
2. Selezionare:
    
    - Client: Technology Advising
        
    - Site: Sede Principale
        
    - Type: Workstation
        
    - Architecture: 64-bit
        
3. Copiare il PowerShell one-liner generato
    
4. Eseguirlo come amministratore
    
5. Attendere la registrazione dell'agente
    

---

# Script Operativi

## Verifica utenti locali

```powershell
Get-LocalUser | Select Name, Enabled
```

Verifica amministratori locali:

```powershell
Get-LocalGroupMember -Group "Administrators"
```

---

## Disabilitazione account utente

Disabilita l'account e termina eventuali sessioni attive.

```powershell
param($username)

Disable-LocalUser -Name $username

$session = query user $username 2>$null | Select-String $username

if ($session) {
    $sessionId = ($session -split '\s+')[2]
    logoff $sessionId /server:localhost
}

Write-Output "Account '$username' disabilitato e sessione terminata."
```

Riabilitazione:

```powershell
Enable-LocalUser -Name <username>
```

---

## Rimozione privilegi amministrativi

```powershell
param($username)

Remove-LocalGroupMember `
    -Group "Administrators" `
    -Member $username

Write-Output "Utente '$username' rimosso dagli Administrators."
```

---

## Blocco installazioni e disinstallazioni

Obiettivi:

- Rimuovere privilegi amministrativi
    
- Bloccare MSI
    
- Bloccare esecuzione EXE dalle cartelle utente
    
- Bloccare disinstallazione software
    

```powershell
# Script completo TRMM
# (contenuto invariato)
```

> ⚠️ Richiede riavvio per l'attivazione delle Software Restriction Policy.

---

## Windows 11 Home e [[🔒BitLocker]]

> ℹ️ Windows 11 Home non supporta BitLocker completo ma include **Device Encryption** automatica quando l'hardware soddisfa i requisiti Microsoft.

---

## Comandi utili

### Stato servizi

```bash
sudo systemctl status \
    rmm celery celerybeat \
    nginx nats meshcentral
```

### Riavvio servizi

```bash
sudo systemctl restart \
    rmm celery celerybeat
```

### Verifica MeshCentral

```bash
python manage.py check_mesh
```

### Log errori

```bash
tail -f \
/rmm/api/tacticalrmm/tacticalrmm/private/log/error.log
```

### Log accessi

```bash
tail -f \
/rmm/api/tacticalrmm/tacticalrmm/private/log/access.log
```

---

## Troubleshooting

|Problema|Possibile causa|Soluzione|
|---|---|---|
|Agente non si connette|DNS o firewall|Verificare porta 443|
|Dashboard non raggiungibile|Servizi down|Riavviare rmm/nginx|
|Remote Desktop non funziona|Problema MeshCentral|`check_mesh`|
|Script in timeout|Script troppo lungo|Return code 98|
|Check non ricevuti|NATS fermo|Riavviare NATS|

---

## Note operative

- Utilizzare certificati multi-dominio quando si impiegano host DuckDNS distinti.
    
- Testare sempre il primo agente prima del rollout.
    
- Gli script di restrizione software possono impedire il funzionamento di applicazioni installate in `%APPDATA%` o `%LOCALAPPDATA%`.
    
- Per applicazioni aziendali è preferibile l'installazione in `C:\Program Files`.
    
- Dopo modifiche alle Software Restriction Policy è consigliato un riavvio del sistema.