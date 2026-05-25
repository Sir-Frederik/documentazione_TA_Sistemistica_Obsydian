
È la più popolare piattaforma per la creazione e gestione dei [[container]]. Permette di pacchettizzare, distribuire ed eseguire applicazioni in ambienti isolati, eliminando per sempre il problema *"sul mio computer funzionava"*.

A differenza delle macchine virtuali, Docker non emula un intero sistema operativo, ma condivide il kernel della macchina host (il server fisico), risultando estremamente **leggero, veloce da avviare e poco avido di risorse (RAM/CPU)**.

###  Il Processo Fondamentale
Il ciclo di vita di un'applicazione in Docker si riassume in questo flusso:
> **Dockerfile → 🔨 Build → Immagine → ▶️ Run → Container**

*   **Dockerfile:** È un **file di testo** con le istruzioni riga per riga per costruire un'immagine (es. "parti da Ubuntu, installa Java, copia il mio codice").
*   **Immagine (Image):** È un "pacchetto" di sola lettura, un modello congelato che contiene il codice, le librerie e le dipendenze. ^113a42
*   **Container:** È l'istanza *in esecuzione* di un'immagine. È temporaneo e isolato. Dalla stessa Immagine posso avviare decine di Container identici.

###  Persistenza dei Dati
Dato che i container sono temporanei (se si rompono o si aggiornano, vengono cancellati), i dati generati al loro interno andrebbero persi. Docker risolve il problema con i **Volumes (Volumi)**: sono aree di archiviazione gestite da Docker e "agganciate" fuori dal container. Se il container muore, il Volume con i dati (es. il database) rimane intatto.

---

##  Docker Registry e Docker Hub

Per spostare le immagini dal PC dello sviluppatore ai server di produzione, serve un sistema di archiviazione e distribuzione centralizzato.

*   **Docker Hub:** È il **registro pubblico** ufficiale di Docker. Contiene migliaia di immagini ufficiali già pronte all'uso (Ubuntu, MySQL, Redis, Nginx). Chiunque può scaricarle. Offre anche repository privati, ma con dei limiti.
*   **Docker Registry(Privato):** In ambito aziendale, non si carica MAI il codice sorgente proprietario su Docker Hub. Le aziende installano un **Docker Registry privato** sui propri server. È un'applicazione a sé stante che fa da **"magazzino centrale chiuso"**. L'infrastruttura segue una gerarchia precisa: *Registry* (il server) → *Repository* (l'applicazione) → *Tag* (la versione, es. `v2.1`). Usare un registry interno garantisce **sicurezza assoluta** e **velocità di download** massimizzata, poiché le immagini pesanti viaggiano solo sulla rete locale.

---

## 🧩 Altri Componenti Docker Degni di Nota

Oltre al motore base, l'ecosistema Docker comprende strumenti fondamentali per gestire scenari complessi:

### 🐙 Docker Compose
Se un'applicazione è composta da più servizi (es. un server web + un database + una cache Redis), avviarli uno ad uno a mano è impensabile. **Docker Compose** è uno strumento che permette di definire un'intera applicazione multi-container tramite un singolo file `docker-compose.yml`. Con un solo comando (`docker-compose up`), avvia e collega insieme tutti i container necessari.

### 🕸️ Docker Network
È il sistema di networking interno di Docker. Permette di creare **reti virtuali isolate**. I container che si trovano sulla stessa "Docker Network" possono comunicare tra loro usando semplicemente i loro nomi (es. il container web chiama direttamente il container `db`), rimanendo invisibili e irraggiungibili dall'esterno per questioni di sicurezza.

### 🐝 Docker Swarm (Menzione storica/alternativa)
È lo strumento nativo di Docker per raggruppare più server fisici e gestirli come un unico grande cluster. Oggigiorno è usato raramente nei grandi progetti, poiché è stato di fatto soppiantato dallo standard di mercato: **[[Kubernetes]]**.

---

## 🚀 Orchestrazione su Larga Scala
Quando si hanno **molti container** da gestire su decine o centinaia di macchine (cluster), si ha bisogno di bilanciamento del carico, auto-riavvio se un container cade e aggiornamenti senza disservizi. In questi scenari enterprise, Docker da solo non basta più. Entra in gioco **[[Kubernetes]]** (K8s), che usa Docker (o Containerd) come motore, ma si occupa di orchestrare tutto il sistema dall'alto.

---

## 🛠️ INSTALLAZIONE
Per installare e configurare Docker sul sistema, vai a [[Installazione del Docker]].