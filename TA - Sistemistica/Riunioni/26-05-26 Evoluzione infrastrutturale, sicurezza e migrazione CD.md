

## Obiettivo generale

Strutturare i processi tecnici, infrastrutturali e di sicurezza dell’azienda in modo:

- standardizzato
    
- documentato
    
- replicabile
    
- monitorabile
    
- conforme ai requisiti ISO 27001
    

L’obiettivo è costruire procedure operative reali e utilizzabili internamente.

---

# 1. Creazione CD e sanitizzazione sorgenti (massima priorità)

## Attività

- Effettuare export dei progetti da SVN
    
- Creare una cartella unica contenente tutti i sorgenti
    
- Rinominare riferimenti e naming applicativi:
    
    - es. “Gaia” → “Etere”
        
- Rimuovere:
    
    - password
        
    - username
        
    - puntamenti DB
        
    - riferimenti sensibili
        
- Inserire placeholder al posto delle credenziali
    
- Verificare eventuali loghi o riferimenti grafici residuali
    

## Note operative

- Utilizzare export SVN e non working copy completa
    
- Utilizzare sostituzioni massive (es. Notepad++)
    
- Attività da completare entro una settimana
    

---

# 2. ISO 27001 e documentazione operativa

## Obiettivo

Costruire documentazione reale, utile e mantenibile.

## Attività immediate

- Recuperare elenco documenti ISO 27001 già esistente
    
- Creare Excel centralizzato con:
    
    - documenti presenti
        
    - documenti mancanti
        
    - documenti da revisionare
        
    - stato avanzamento
        
    - assegnazione responsabili
        

## Metodo di lavoro

Classificare documenti:

- Verde → completo
    
- Giallo → parziale/incompleto
    
- Rosso → assente
    

## Requisiti fondamentali

- I documenti devono descrivere procedure reali
    
- L’uso di LLM è consentito solo come supporto
    
- Le procedure devono essere:
    
    - verificabili
        
    - ripetibili
        
    - utilizzabili operativamente
        

## Concetto chiave emerso

La documentazione non è “burocrazia”:  
serve a definire come il team lavora realmente.

---

# 3. Procedure operative da costruire

## Vulnerability Management

Definire:

- identificazione vulnerabilità
    
- workflow di gestione
    
- migrazione applicazioni in ambienti di test
    
- verifica remediation
    

## Capacity Management

Creare metodologia per:

- valutazione carico piattaforma
    
- dimensionamento cluster
    
- espansione nodi Kubernetes
    
- gestione risorse infrastrutturali
    

## Backup Management

Definire:

- backup persistenti
    
- retention
    
- replica geografica
    
- gestione restore
    

## Disaster Recovery

Da avviare almeno su un progetto pilota entro fine anno.

### Il DR deve includere:

- scenari di disastro
    
- procedure operative
    
- tempistiche
    
- ruoli coinvolti
    
- test periodici
    
- infrastruttura remota
    

## Nota importante

Un Disaster Recovery reale richiede:

- ambienti separati
    
- distanza geografica
    
- test periodici verificabili
    

---

# 4. Governance team e modalità operative

## Regola introdotta

Una task non è “finita” quando una persona dice di averla completata.

Una task è conclusa solo se:

- verificata
    
- validata
    
- accettata dal team
    

## Necessità evidenziate

- maggiore comunicazione interna
    
- utilizzo strutturato di Jira
    
- assegnazione trasparente task
    
- tracciamento avanzamento lavori
    

---

# 5. LLM locali, automazione e AI interna

## Strategia

Iniziare a costruire strumenti AI locali specializzati.

## Motivazioni

- riduzione costi API cloud
    
- controllo infrastrutturale
    
- sicurezza
    
- indipendenza da servizi terzi
    

## Caso d’uso iniziale

Prodotto relativo ai calciatori.

---

## Agenti da sviluppare

### Agente sicurezza repository

Funzioni:

- scansione repository Git
    
- identificazione problematiche sicurezza
    
- supporto aggiuntivo a SonarQube
    

### Agente documentazione

Funzioni:

- generazione documentazione tecnica
    
- produzione template
    
- supporto documentazione pipeline
    

---

## Concetto architetturale importante

Le pipeline AI devono essere:

- modulari
    
- sostituibili
    
- versionabili
    
- controllabili internamente
    

---

# 6. Jenkins, SonarQube e pipeline CI/CD

## Stato attuale

Nuove installazioni:

- Jenkins
    
- SonarQube
    

Migrazione prevista:

- luglio
    

---

## Strategia migrazione

- iniziare dai pacchetti meno critici
    
- migrazione graduale
    
- possibile export configurazioni Jenkins
    

---

## SonarQube

Definire quality gate personalizzati:

- inizialmente permissivi
    
- successivamente più restrittivi
    

Necessario adattarli ai software aziendali.

---

# 7. Problematiche CI/CD microservizi e patching

## Problema emerso

Con architettura a microservizi:

- le patch dipendono dalla revisione specifica dell’ambiente
    
- non è sufficiente rebuildare l’ultima versione
    

## Necessità

Creare:

- procedura patching strutturata
    
- namespace dedicati fix/patch
    
- gestione versioni Helm chart
    
- tracciamento revisioni
    

## Tema centrale

Separare:

- sviluppo
    
- collaudo
    
- produzione
    
- patch temporanee
    

---

# 8. Docker Registry

## Situazione attuale

Uso di DockerHub come registry.

## Problema

Necessità di:

- controllo storage
    
- retention immagini
    
- policy tag/versioning
    
- indipendenza futura
    

## Possibile soluzione

Registry locale interno.

## Da definire

- retention immagini
    
- gestione tag storici
    
- policy immagini produzione
    

---

# 9. Penetration testing

## Priorità alta

Creare procedura completa di penetration testing.

## La procedura deve includere

- migrazione ambiente test
    
- replica DB/schema
    
- versionamento applicativo
    
- ambiente Kubernetes dedicato
    
- workflow operativo
    

---

# 10. Elasticsearch e logging

## Decisione architetturale

Elasticsearch non deve stare nella rete interna principale.

## Motivazione

La raccolta log deve sopravvivere anche a problemi della rete interna.

## Possibile soluzione

Macchina dedicata separata.

---

# 11. Automazione infrastrutturale

## Ansible

Studiare Ansible per:

- automazione deploy
    
- migrazioni
    
- provisioning
    
- gestione server
    

---

# 12. Città Metropolitana di Napoli

## Stato

In attesa validazione finale e produzione.

## Problemi emersi

- modifiche manuali lato cliente
    
- configurazioni alterate
    
- difficoltà operative sulla VPN
    

## Azione

Definire deadline e avvio produzione.

---

# Priorità  ⚠️

1. Export/CD sorgenti sanitizzati
    
2. Procedura penetration testing
    
3. ISO 27001 + Excel documentazione
    
4. Migrazione Jenkins/SonarQube
    

---

# Visione strategica emersa dalla riunione

La direzione tecnica si sta spostando verso:

- DevSecOps strutturato
    
- standardizzazione processi
    
- governance infrastrutturale
    
- automazione operativa
    
- AI interna specializzata
    
- compliance reale e non solo documentale
    
- gestione enterprise di CI/CD e microservizi