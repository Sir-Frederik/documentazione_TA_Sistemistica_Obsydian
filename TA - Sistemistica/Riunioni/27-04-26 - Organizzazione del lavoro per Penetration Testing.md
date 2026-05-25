
Collegamento a [[23-04-26 - Definizione pipeline di sicurezza e penetration testing – Protezione Civile]]


---

## Aggiornamento attività – Pipeline sicurezza e penetration testing

Collegamento a: _Definizione pipeline di sicurezza e penetration testing – Protezione Civile_

### Attività assegnate

- **Genny**  
    Conversione e gestione dei file di report generati da Trivy.
    
- **Alessandro**  
    Aggiornamento di SonarQube e distribuzione su tutti gli ambienti.
    
- **Vincenzo**  
    Redazione di un report che descriva la procedura completa con:
    
    - SonarQube
        
    - Trivy
        
    - Jenkins
        

---

## Architettura attuale (sintesi)

- Presenza di:
    
    - applicativi
        
    - servizi REST
        
    - processi batch
        
- I batch:
    
    - interrogano database
        
    - consumano dati interni ed esterni
        
- Alcuni servizi espongono e aggiornano dati
    
- In Kubernetes:
    
    - è presente una porta di ingresso (entry point)
        
    - e una di uscita
        

Obiettivo: testare la sicurezza provando a “forzare” la porta di ingresso in un ambiente controllato.

---

## Attività infrastrutturali

- Necessità di:
    
    - trasferire i progetti su un nuovo sistema operativo
        
    - oppure portarli su Kubernetes
      [[Collegare i Progetti ai Server]]
        
- Tracciamento attività:
    
    - utilizzo di strumenti come Azure o Samba per monitorare operazioni e accessi
        
- Gestione immagini:
    
    - abbandono di Docker Hub
        
    - migrazione verso un Docker Registry interno
        
- Infrastruttura:
    
    - creazione di una macchina dedicata su OVH
        
    - la macchina deve poter ospitare e gestire tutte le immagini utilizzate
        

---

## Attività Federico (infrastruttura e immagini)

### Obiettivo generale

Gestire e mettere in sicurezza la parte infrastrutturale legata alle immagini container e al loro utilizzo in pipeline.

### Attività operative

- **Controllo infrastruttura**
    
    - Verifica dello stato attuale delle macchine
        
    - Analisi di dove risiedono i progetti e come vengono eseguiti
        
- **Aggiornamento pacchetti**
    
    - Aggiornare i pacchetti delle immagini container
        
    - Verificare la presenza di vulnerabilità note (anche tramite Trivy)
        
- **Migrazione immagini**
    
    - Configurare un **Docker Registry interno**
        
    - Spostare le immagini da Docker Hub al nuovo registry
        
    - Aggiornare i riferimenti alle immagini nei progetti
        
- **Configurazione nuova macchina (OVH)**
    
    - Creazione macchina da zero
        
    - Installazione ambiente necessario (Docker/Kubernetes se richiesto)
        
    - Verifica che riesca a:
        
        - buildare immagini
            
        - salvarle nel registry
            
        - renderle disponibili per il deploy
            
- **Integrazione con Jenkins**
    
    - Modificare le pipeline per:
        
        - usare il nuovo Docker Registry
            
        - non dipendere più da Docker Hub
            
    - Verificare che il pull delle immagini funzioni correttamente
        
- **Supporto alla sicurezza**
    
    - Garantire che le immagini utilizzate siano aggiornate
        
    - Ridurre la superficie di attacco lato infrastruttura
        
    - Preparare l’ambiente per i test di penetration (isolamento incluso)
        

---

Così è più chiaro anche per te: non è solo “aggiornare pacchetti”, stai praticamente gestendo tutta la parte **registry + immagini + base infrastrutturale della pipeline**.