---
tags:
  - cybersecurity
  - SOC
  - corso-soc-junior
  - udemy
corso: SOC Analyst Junior
fonte: Udemy
---

## Definizione

> [!note] Cos'è un SOC
> Il **SOC (Security Operations Center)** è il centro da cui vengono erogati i servizi che hanno come scopo finale quello di garantire la **sicurezza** e la **difesa** di una rete informatica.

## Modalità di erogazione

Il SOC può operare su:

- una **rete interna**, cioè quella dell'organizzazione stessa;
- **da remoto**, offrendo il servizio a **diversi clienti**.

## I componenti del SOC

Il SOC si basa su 6 elementi fondamentali:

![[Pasted image 20261020162814.png]]

1. **SIEM**
2. **Collezione dei log e loro correlazione**
3. **Monitoraggio e risposta agli incidenti informatici**
4. **Reporting e ticketing**
5. **Ricerca e sviluppo**
6. **Threat intelligence**

### SIEM

> [!important] Il componente più importante
> Il **SIEM** (*Security Information and Event Management*) è il tool principale usato nel SOC: è un software che si occupa di **collezionare i log e correlarli tra loro**.

Le regole con cui il SIEM analizza i log sono **totalmente personalizzabili**, oltre a quelle già preconfigurate, che nei SIEM più comuni sono generalmente adeguate.

### Collezione dei log e loro correlazione

È l'attività svolta concretamente dal SIEM, che viene configurato per ricevere i log da **apparati di rete**, sistemi, server e workstation, fino al portatile del singolo dipendente.

Il flusso di lavoro è questo:

1. il SIEM **riceve** i log;
2. li **memorizza**;
3. tramite i **motori delle regole** (*rule engine*) i log vengono **analizzati, parsati e correlati** tra loro;
4. in caso di anomalie vengono **triggerati degli allarmi**, che avvisano l'analista che sta monitorando la console che è stato rilevato qualcosa di anomalo.

### Monitoraggio e (eventuale) risposta agli incidenti

Dopo la raccolta, il parsing e la correlazione dei log, entra in gioco il **monitoraggio e l'eventuale risposta agli incidenti informatici**.

L'**analista SOC di primo livello** ha come compito principale quello di monitorare gli eventi, la console e gli allarmi provenienti dal SIEM.

### Reporting e ticketing

Un altro strumento fondamentale sono i **sistemi di reporting e ticketing**. Per il SOC è importantissimo tracciare, su un sistema di ticketing o comunque in modo strutturato, tutto ciò che viene svolto: monitoraggio, ricerca, rilevazione e risposta agli incidenti.

> [!tip] Perché è così importante
> Se in futuro si ripresenta una situazione già accaduta, la si ritrova nel sistema di ticketing ed è molto più facile intervenire. Questo porta grossi vantaggi in termini di **tempo ed energie** per l'analisi e la risposta all'incidente.

### Ricerca e sviluppo

Insieme alla threat intelligence, è un'attività di approfondimento solo apparentemente collaterale al SOC.

- **Ricerca:** l'attività del SOC è in continua evoluzione, quindi è fondamentale rimanere costantemente informati e cercare sempre nuove informazioni e aggiornamenti.
- **Sviluppo:** alcuni SOC hanno al loro interno degli sviluppatori che realizzano tool specifici per il SOC, ad esempio per automatizzare la risposta agli incidenti (vedi i ==**SOAR**, *Security Orchestration, Automation and Response*==, che verranno affrontati più avanti nel corso).

### Threat Intelligence

La **threat intelligence** è un'attività di **terzo livello** del SOC: si occupa di collezionare e correlare le informazioni di pubblico dominio per conoscere i cosiddetti **attaccanti** (o cyber criminali).

L'obiettivo è scoprire come si muovono gli attaccanti e quali mezzi, tool e strategie usano. Questo serve sia a individuare chi sta svolgendo attività illecite e segnalarlo alle forze dell'ordine, sia (lato SOC e lato **Blue Team**) a difendersi nel migliore dei modi: **conoscere l'attaccante permette di difendersi meglio**.

> [!question] Cos'è il Blue Team?
> È il team che si occupa della **difesa**: protegge i sistemi, monitora, rileva e risponde agli attacchi. Il SOC è un tipico esempio di Blue Team. Si contrappone al **Red Team**, che simula gli attacchi (come farebbe un vero attaccante) per mettere alla prova le difese.

## Le fasi operative del SOC

Le attività del SOC si possono dividere, sommariamente, in 5 fasi:

![[Pasted image 20261020164826.png]]

1. **Information gathering (raccolta delle informazioni):** è la raccolta vera e propria dei log da parte di tutti gli apparati di rete che devono essere monitorati.
2. **Memorizzazione e archiviazione:** viene effettuata dal SIEM; solitamente è la macchina stessa del SIEM a collezionare e memorizzare i log.
3. **Monitoraggio**
4. **Rilevazione**
5. **Reazione**

Le ultime tre fasi (monitoraggio, rilevazione e reazione) rappresentano la **parte tecnica e umana**, svolta dagli analisti SOC in quanto personale specializzato.

---

## APPUNTI

