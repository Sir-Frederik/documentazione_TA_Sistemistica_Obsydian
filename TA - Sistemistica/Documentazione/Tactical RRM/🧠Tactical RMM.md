
**Tactical RMM** è una piattaforma open source di **Remote Monitoring & Management (RMM)** per la gestione centralizzata di endpoint Windows, Linux e macOS. Rappresenta un'alternativa gratuita a soluzioni commerciali come NinjaRMM, ConnectWise o Datto, ed è pensata per MSP e team IT interni che gestiscono flotte di macchine da remoto.

A differenza di soluzioni come [[🏦 SAMBA]] o [[FreeIPA]], non richiede che i client siano aggiunti a un dominio, né impone requisiti specifici sull'edizione del sistema operativo.

[[📘Guida  personale a TRMM|Qui]] puoi trovare la guida per l'installazione di TRMM, mentre [[Libreria Script TRMM|qui]] puoi trovare la libreria degi script.

---

## Architettura

Tactical RMM è composto da diversi componenti che cooperano in un'unica installazione server:

|Componente|Ruolo|
|---|---|
|**Django**|Backend — espone le API per il frontend e per gli agenti|
|**Vue.js**|Frontend — dashboard web accessibile via browser|
|**PostgreSQL**|Database principale|
|**Redis**|Cache e message broker per Celery|
|**Celery**|Gestione task asincroni (check, alert, patch)|
|**NATS**|Canale di comunicazione in tempo reale tra server e agenti|
|**MeshCentral**|Accesso remoto — desktop, shell, file browser|
|**Nginx**|Reverse proxy con terminazione TLS|

La comunicazione tra server e agenti avviene tramite **NATS**: gli agenti online ricevono i comandi in tempo reale; quelli offline li eseguono alla successiva connessione.

---

## Prerequisiti

Il server richiede una VM dedicata con le seguenti caratteristiche minime:

- Sistema operativo: **Ubuntu 22.04 LTS** o Debian 11/12
- RAM: almeno **4 GB**
- Architettura: **x64**
- Tre sottodomini DNS puntati all'IP pubblico del server

|Sottodominio|Uso|
|---|---|
|`rmm.azienda.com`|Dashboard web e comunicazione agenti|
|`api.azienda.com`|Backend API|
|`mesh.azienda.com`|MeshCentral|

> ⚠️ I tre sottodomini devono essere allo stesso livello DNS. Non sono supportate strutture wildcard che li raggruppino.

Le porte necessarie sul firewall sono **443/TCP** (dashboard, API, MeshCentral) e **22/TCP** (SSH per amministrazione server).

---

## Installazione e manutenzione

L'installazione avviene tramite uno script ufficiale da eseguire una sola volta su una VM pulita. Il processo è automatizzato e configura tutti i componenti elencati nell'architettura.

Gli aggiornamenti seguono la stessa logica: uno script dedicato aggiorna i componenti preservando la configurazione esistente. È buona pratica eseguire un backup prima di ogni aggiornamento.

Il backup include database, configurazioni e certificati. Può essere pianificato automaticamente con le seguenti retention di default:

- **Giornalieri**: 2 settimane
- **Settimanali**: 2 mesi
- **Mensili**: 1 anno

---

## Funzionalità principali

### [[Libreria Script TRMM|Script]]

Supporta l'esecuzione remota di script in PowerShell, Batch, Python, Bash, Nushell e Deno. Tre modalità di esecuzione disponibili:

|Modalità|Comportamento|
|---|---|
|Wait for Output|Attende il completamento e restituisce l'output|
|Fire and Forget|Esegue senza attendere il risultato|
|Email Output|Invia il risultato via email al completamento|

### Check

Monitoraggio continuo di CPU, RAM, disco, servizi Windows, Event Log e script personalizzati. I check possono generare alert se superano soglie configurabili.

### Task

Esecuzione pianificata di script tramite scheduler interno, indipendente dal Task Scheduler Windows.

### Patch Management

Gestione centralizzata degli aggiornamenti Windows su tutti gli endpoint della flotta.

### Alert

Notifiche configurabili tramite email, SMS o webhook verso sistemi esterni.

### Accesso remoto (MeshCentral)

Accesso completo alla macchina remota tramite desktop remoto, shell interattiva, file browser, registry editor, event viewer e gestione servizi. MeshCentral è integrato nativamente nella dashboard.

### Software (Chocolatey)

Installazione e aggiornamento di software su endpoint Windows tramite integrazione con il package manager Chocolatey.

---

## Installazione agente

L'agente viene installato su ogni endpoint tramite un PowerShell one-liner generato dalla dashboard. Il processo assegna l'agente a un client, un sito e un tipo di macchina (workstation o server). Una volta registrato, l'agente è visibile nella dashboard e inizia a inviare dati di monitoraggio.

---

## Note operative

- Utilizzare certificati multi-dominio quando i tre sottodomini sono gestiti come host DuckDNS distinti — un certificato wildcard di primo livello non li copre.
- Testare sempre il deploy su un singolo agente prima del rollout sull'intera flotta.
- Gli script che modificano restrizioni software (SRP, WDAC, AppLocker) possono impattare applicazioni installate in percorsi non standard — verificare su macchine di test prima del deploy.
- Per applicazioni aziendali è preferibile l'installazione in `C:\Program Files` rispetto alle cartelle utente, che sono spesso soggette a restrizioni di esecuzione.
- Script con timeout superiore al limite configurato restituiscono codice 98 — non indica un errore dello script ma un timeout di TRMM.

---

## Troubleshooting

|Problema|Possibile causa|Soluzione|
|---|---|---|
|Agente non si connette|DNS o firewall|Verificare raggiungibilità porta 443|
|Dashboard non raggiungibile|Servizi down|Riavviare rmm e nginx|
|Remote desktop non funziona|Problema MeshCentral|Eseguire check_mesh dal server|
|Script in timeout|Esecuzione troppo lunga|Codice 98 — aumentare timeout o suddividere lo script|
|Check non ricevuti|NATS fermo|Riavviare il servizio NATS|