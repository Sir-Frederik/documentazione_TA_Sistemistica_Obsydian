## Stato attuale

- Jenkins gestisce la pipeline
    
- SAST: SonarQube
    
- Container scan: Trivy
    
    - copre anche parte SCA, ma non analizza tutte le dipendenze in profondità
        

Mancano ancora alcuni elementi per completare la catena di sicurezza.

---

## Componenti della sicurezza

### SAST – Analisi statica

- Strumento: SonarQube
    
- Attualmente:
    
    - supporto presente per JavaScript
        
    - da estendere a TypeScript
        
- Necessario:
    
    - integrare plugin Angular per SonarQube
        
    - evitare impatti significativi sui tempi di build
        
- Nota operativa:
    
    - per il frontend è necessario integrare il sonar-scanner come dipendenza e configurarlo
        
    - attività da coordinare con lo sviluppatore frontend (tramite Genny)
        

---

### SCA – Software Composition Analysis

- Obiettivo:
    
    - analisi delle dipendenze nelle fasi iniziali
        
    - individuazione vulnerabilità nelle librerie
        
    - controllo eventuali credenziali nel pacchetto buildato
        
- Situazione:
    
    - Trivy copre solo parzialmente questo aspetto
        
    - serve un tool dedicato
        
- Requisiti:
    
    - open source
        
    - non SaaS (installabile su macchina interna)
        
    - analisi del codice proveniente da Git
        
- Note operative:
    
    - gli sviluppatori dovranno aggiornare le dipendenze
        
    - le versioni da adottare devono essere definite dal team sicurezza
        
    - individuare uno sviluppatore “pilota” per testare gli aggiornamenti prima della diffusione
        
    - mantenere compatibilità con Spring versione 3
        

---

### DAST – Analisi dinamica

- Obiettivo:
    
    - simulare attacchi reali alla piattaforma
        
    - identificare vulnerabilità runtime
        
- Strumento:
    
    - OpenVAS
        
- Modalità di utilizzo:
    
    - non eseguito su Jenkins
        
    - ambiente isolato e protetto
        
    - deploy della piattaforma in versione replica
        
    - ambiente non produttivo
        
- Attività:
    
    - test iniziale sul progetto MESIT
        
    - valutazione di alternative open source installabili localmente
        
    - creazione di un ambiente dedicato per i test di penetrazione
        

---

### Container Scan

- Strumento: Trivy
    
- Funzione:
    
    - analisi vulnerabilità delle immagini container
        
- Focus:
    
    - vulnerabilità HIGH e CRITICAL
        
- Limiti:
    
    - copertura parziale delle dipendenze annidate
        

---

### Monitoraggio e runtime security

- Falco
    
    - da analizzare nel dettaglio
        
    - possibile utilizzo per monitoraggio runtime e rilevazione comportamenti anomali
        
- Prometheus e Zabbix
    
    - raccolta metriche infrastrutturali
        

---

### Logging e analisi centralizzata

- Stack ELK (Elasticsearch, Logstash, Kibana)
    
- Stato attuale:
    
    - presenti dashboard di sicurezza con alert e detection rules
        
    - monitoraggio eventi (es. tentativi di accesso)
        
- Necessità:
    
    - dashboard unica per tutti i progetti
        
    - integrazione metriche applicative e di sicurezza
        
- Attività:
    
    - invio dati da Zabbix e Prometheus verso ELK in formato JSON
        
    - valutazione della mole di dati gestita
        
    - individuazione delle metriche più rilevanti
        

---

## Gestione output e report

- Situazione attuale:
    
    - output Trivy salvato localmente e non persistente
        
    - report inviati via email
        
- Problemi:
    
    - perdita dati
        
    - mancanza di centralizzazione
        
- Obiettivo:
    
    - inviare output Trivy su ELK
        
    - mantenere storico consultabile
        

---

## Attività operative

- Costruire una catena completa di sicurezza integrata nella pipeline
    
- Applicare la catena al progetto MESIT
    

### Azioni immediate

- Integrare Trivy e coordinare la risoluzione vulnerabilità con gli sviluppatori
    
- Effettuare uno scanning iniziale con OpenVAS su MESIT
    
- Ricercare strumenti DAST open source alternativi
    

---

## Assegnazioni

- Genny
    
    - gestione Trivy e SCA
        
    - realizzazione Helm chart
        
    - coordinamento configurazione SonarQube frontend
        
    - creazione ambiente dedicato per test DAST
        
- Alessandro
    
    - gestione OpenVAS
        
    - gestione Trivy e SCA
        
    - configurazione sistema di invio email senza utilizzare funzionalità avanzate ELK
        
    - installazione di un nuovo Jenkins su macchina dedicata (>= 32 GB RAM)
        
    - integrazione SonarQube scanner nel nuovo Jenkins
        
- Federico e Vincenzo
    
    - [ ] gestione metriche e logging su ELK
        
    - [ ] creazione dashboard centralizzata per tutti i progetti
        
    - [ ] integrazione dati da Zabbix e Prometheus
        
    - [x] analisi e valutazione [[🚨Falco]] (funzionamento e impatto)
        
    - [x] analisi volume dati e sostenibilità ELK
        

---

## Direzione generale

Costruire una catena di sicurezza completa che includa:

- analisi statica (SAST)
    
- analisi dipendenze (SCA)
    
- analisi dinamica (DAST)
    
- scanning container
    
- monitoraggio runtime
    
- raccolta e visualizzazione centralizzata delle metriche
    

con l’obiettivo di garantire controllo, tracciabilità e sicurezza lungo tutto il ciclo di vita dell’applicazione.