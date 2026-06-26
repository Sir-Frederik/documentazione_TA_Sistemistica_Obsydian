

## 1. Migrazione Data Warehouse UNISA (Questionari Ambur)

### Contesto

Attualmente i questionari Ambur sono utilizzati in due contesti distinti:

- ambiente finanziato direttamente dall'Università;
- ambiente gestito tramite PowerHouse/Otumousse all'interno dell'Università.

L'obiettivo non è intervenire sugli ambienti esistenti, ma predisporre una nuova infrastruttura da utilizzare come base per una futura migrazione del Data Warehouse.

Prima della migrazione è necessario effettuare una ricognizione dell'infrastruttura esistente, eliminando o dismettendo le macchine non più utilizzate e verificando la documentazione tecnica già disponibile.

### Attività preliminari

- Analizzare la documentazione esistente prodotta da Federico presente in `\soc\documenti`
- Verificare le macchine attualmente coinvolte nel processo
- Identificare eventuali sistemi non più utilizzati e valutarne la dismissione
- Analizzare i puntamenti dei batch e i flussi di esecuzione attuali
- Definire la strategia di ripuntamento dei batch verso il nuovo ambiente
- Assegnare a Federico microattività di analisi e censimento dell'infrastruttura coinvolta

### Attività di migrazione

- Creare un nuovo Data Warehouse sul cloud aziendale
- Valutare l'utilizzo di una macchina di test su Aruba (NoCPU) oppure altra infrastruttura equivalente
- Migrare la struttura tabellare dei questionari Ambur nel nuovo ambiente
- Eseguire test funzionali e verifiche di consistenza dei dati
- Validare il corretto funzionamento dei processi batch
- Dopo il collaudo, spostare i puntamenti degli ambienti di produzione verso il nuovo Data Warehouse

### Attività successive

- Aggiornamento dei puntamenti dei dataset
- Attività a carico di Giuseppe

### Tempistica

- Obiettivo: completamento entro una settimana
    

---

## 2. Evoluzione architettura ASL e Data Warehouse multitenant

### Obiettivo

Progettare un'architettura multitenant che permetta di servire più clienti utilizzando la stessa infrastruttura, riducendo costi e complessità operativa.

### Situazione attuale

- L'OAC trasporta i dati verso un database locale
    
- Il database utilizzato dall'OAC è separato dal Data Warehouse
    
- I dati vengono acquisiti tramite file CSV provenienti da una macchina IaaS
    

### Problema

L'approccio ipotizzato inizialmente prevedeva:

- un'istanza OAC dedicata per ogni cliente
    

Secondo Adriano questa soluzione non è sostenibile:

- aumenta inutilmente i costi
    
- complica la gestione
    
- non è giustificata dai carichi reali
    

### Direzione proposta

Realizzare una soluzione multitenant basata su:

- unico Data Warehouse condiviso
    
- unico OAC condiviso
    
- personalizzazione delle interfacce in base al cliente
    
- identificazione del cliente tramite parametri o ID di query
    
- configurazioni differenziate senza duplicare l'infrastruttura
    

### Benefici attesi

- riduzione dei costi annuali
    
- migliore utilizzo delle risorse hardware
    
- maggiore scalabilità
    
- gestione centralizzata
    

### Attività di studio

Partire dalla realizzazione del Data Warehouse di test e valutare:

- fattibilità del modello multitenant
    
- gestione di più clienti sullo stesso ambiente
    
- impatto sulle prestazioni
    
- modalità di segregazione dei dati
    

### Scenario di test previsto

Entro giugno:

- predisporre tre scenari distinti (tre clienti simulati)
    
- utilizzare un unico OAC
    
- verificare la capacità di cambiare contesto cliente tramite configurazione o parametro
    

### Alternative infrastrutturali da valutare

- Data Warehouse dedicato sul cloud attuale
    
- soluzione su OVH
    
- macchina IaaS dedicata con Oracle installato su OVH
    

L'obiettivo è confrontare costi, semplicità gestionale e scalabilità.

---

## 3. Certificazioni

### Contesto

Adriano sta approfondendo alcune certificazioni professionali e organizzative.

Non si tratta della ISO 27001, ma di certificazioni differenti che potrebbero essere richieste o favorire la collaborazione con la VSN.

### Attività

- Identificare le certificazioni di interesse
    
- Studiare requisiti e modalità di ottenimento
    
- Valutare costi, tempi e benefici
    
- Definire un percorso di certificazione aziendale
    



### Obiettivo

Comprendere quali certificazioni possano rappresentare un vantaggio competitivo e quali siano necessarie per operare in determinati contesti o collaborazioni.

---

## Priorità operative


Le attività prioritarie sono ==la realizzazione del nuovo Data Warehouse per UNISA== ==e la migrazione della struttura dati dei questionari Ambur==, da completare entro una settimana.

Prima della migrazione del Data Warehouse è necessario ==completare la ricognizione dell'infrastruttura esistente==, verificando documentazione, batch e macchine coinvolte, così da definire correttamente il piano di migrazione e i successivi ripuntamenti dei dataset.

In parallelo occorre avviare lo ==studio dell'architettura multitenant per le ASL==, utilizzando il nuovo Data Warehouse come ambiente di sperimentazione e valutando la possibilità di servire più clienti tramite un'unica istanza OAC. 

Infine, è necessario raccogliere ==informazioni sulle certificazioni== di interesse per l'azienda e valutarne l'adozione in funzione delle future collaborazioni e opportunità di mercato.