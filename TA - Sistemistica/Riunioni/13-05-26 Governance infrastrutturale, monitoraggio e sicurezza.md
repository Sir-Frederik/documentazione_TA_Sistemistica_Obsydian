
## Obiettivo generale

Costruire un’infrastruttura:

- monitorata
    
- documentata
    
- sicura
    
- prevedibile
    
- facilmente gestibile
    

L’obiettivo finale è anticipare problemi, vulnerabilità e saturazioni prima che impattino i servizi.

---

# Priorità operative

## Fase 1 — Monitoraggio e stabilizzazione infrastruttura

### Obiettivo

Ottenere visibilità completa su server, database e servizi.

### Attività

- Configurare monitoraggio centralizzato dell’infrastruttura
    
- Ricevere email automatiche dagli stati dei server
    
- Aggiungere Federico agli alert Kubernetes
    
- Configurare alert automatici su:
    
    %% - CPU
        
    - RAM %%
        
    - spazio disco
        
    - servizi critici
        
    - database
        
    - stato dei pod Kubernetes
        

### Zabbix

Configurare Zabbix per:

- monitoraggio infrastrutturale
    
- gestione alert
    
- monitoraggio macchine Kubernetes
    
- monitoraggio macchine standalone/non Kubernetes
    

### Logging e gestione informazioni 

- Centralizzare e archiviare report operativi
    
- Gestire report relativi a:
    
    - server
        
    - database
        
    - vulnerabilità
        
    - anomalie infrastrutturali
        
- Definire procedure per anticipare e risolvere criticità
    

---

# Analisi criticità attuali

## Sistema Big Data

Sono presenti problemi sul sistema Big Data e bisogna investigare le cause.

### Verifiche da effettuare

- utilizzo CPU
    
- consumo RAM
    
- spazio disco
    
- log applicativi
    
- stato Kubernetes
    
- stato database
    
- traffico e carico dei servizi
    

---

# Capacity management e Kubernetes

## Obiettivi

Evitare saturazioni e crash dell’infrastruttura.

### Attività

- Analizzare il consumo risorse delle applicazioni:
    
    - CPU
        
    - RAM
        
    - storage
        
- Verificare quanto occupano le applicazioni
    
- Liberare Kubernetes dai progetti non più utili
    
- Effettuare capacity planning dell’infrastruttura
    

---

# Architettura e censimento infrastrutturale

## Documentazione architetturale

Realizzare:

- schema a blocchi tra i due server
    
- schema a blocchi del dominio applicativo
    
- schema delle macchine Parthenope:
    
    - suddivisione tra OVH e Cloud
        
    - correlazione con i puntamenti database
        

## Censimento risorse

### Tipologie da censire

- IaaS
    
- PaaS
    
- SaaS (in tabella separata)
    

### Informazioni richieste

Per ogni macchina/server:

- sistema operativo
    
- versione sistema operativo
    
- versione software installati:
    
    - Java
        
    - WildFly
        
    - Apache
        
    - HAProxy
        
    - PostgreSQL
        
    - MySQL
        
- ruolo della macchina
    
- relazioni e dipendenze
    

### Note

- PaaS e SaaS: non serve censire le porte
    
- IaaS: censimento completo
    

---

# Analisi connessioni e dipendenze

## Attività

- Verificare se le VM sono sullo stesso host fisico
    
- Analizzare comunicazioni tra server:
    
    - Apache
        
    - proxy/reverse proxy
        
- Individuare:
    
    - puntamenti a database esterni
        
    - comunicazioni tra servizi
        

## Dove cercare

- datasource di WildFly
    
- file `.properties`
    
- configurazioni applicative
    

---

# Sicurezza infrastrutturale

## Obiettivi

- Nessuna porta pubblica esposta
    
- VPN su ogni macchina
    
- Aggiornamento sistemi non supportati
    
- Riduzione superficie di attacco
    

## Attività

- Installare Fail2Ban su tutte le macchine
    
- Aggiornare sistemi operativi a versioni non deprecate
    
- Creare documento con:
    
    - vulnerabilità
        
    - componenti obsoleti
        
    - remediation plan
        

## Piano di aggiornamento

Definire migrazione e aggiornamenti per ogni macchina:

- sistema operativo
    
- middleware
    
- web server
    
- database
    

---

# Sicurezza applicativa e vulnerability management

## Trivy

Integrare Trivy in tutti i progetti importanti.

### Attività

- scansione vulnerabilità container
    
- scansione immagini Docker
    
- standardizzazione report
    

### Compliance

Documentare Trivy in ottica ISO 27001.

---

# Catena di sicurezza applicativa

## Obiettivo

Documentare e standardizzare la pipeline di sicurezza applicativa.

## Da documentare

- pipeline CI/CD
    
- scansioni sicurezza
    
- build
    
- analisi codice
    
- vulnerability assessment
    
- output e report prodotti
    

## Gestione report

Definire:

- archiviazione risultati
    
- storicizzazione report
    
- tracciamento vulnerabilità
    

---

# SonarQube e quality gate

## SonarQube

Studiare integrazione tra frontend e SonarQube.

## Verifiche

- comportamento build su progetti legacy
    
- rilevamento modifiche incrementalmente
    
- analisi statistica e trend del codice
    

## Pipeline

Definire documento ufficiale della pipeline da adottare:

- build
    
- analisi statica
    
- scansioni sicurezza
    
- reportistica
    

---

# Database e log analysis

## MySQL

Sul database MySQL di default sono presenti numerosi log.

### Attività

Analizzare:

- errori
    
- warning
    
- query lente
    
- accessi
    
- riconnessioni
    
- anomalie operative
    

---

# Ambiente di test sicurezza

## Attività

Studiare la creazione di un ambiente dedicato a:

- penetration test
    
- simulazioni di attacco
    
- test vulnerabilità
    

---

# Disaster Recovery e continuità operativa

## Attività

Definire:

- strategie di backup
    
- procedure di restore
    
- disaster recovery
    
- monitoraggio continuità operativa
    

## Verifiche

- stato servizi
    
- corretto funzionamento sistemi
    
- monitoraggio pacchetti e processi
    

---

# Automazione

## Ansible

Utilizzare Ansible per:

- automazione configurazioni
    
- deploy
    
- aggiornamenti
    
- standardizzazione setup server
    

---

# Logging centralizzato

## ELK Stack

Utilizzare Elastic Stack per:

- centralizzazione log
    
- analisi traffico
    
- correlazione eventi
    
- alert automatici
    

---

# Documentazione finale

## Wiki

Pubblicare tutta la documentazione tecnica su wiki aziendale.

## Documenti da produrre

- censimento infrastrutturale
    
- schema architetturale
    
- report vulnerabilità
    
- pipeline sicurezza
    
- procedure operative
    
- roadmap aggiornamenti
    
- piano disaster recovery
    

---

# Pianificazione

## Da definire

- roadmap attività
    
- priorità operative
    
- tempistiche implementazione
    
- milestone tecniche e sicurezza infrastrutturale

[[23-04-26 - Definizione pipeline di sicurezza e penetration testing – Protezione Civile]]
[[27-04-26 - Organizzazione del lavoro per Penetration Testing]]
[[28-04-26 Aggiornamento – Catena di sicurezza applicativa]]