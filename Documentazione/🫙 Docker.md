E' la popolare piattaforma per la gestione dei [[container]]. Permette di creare, distribuire ed eseguire applicazioni in container.

**Processo**:
	Dockerfile → 🔨 Build → Immagine → ▶️ Run → Container

**Dockerfile** è un **file di testo** con le **istruzioni** per costruire una [[Container#^113a42 | immagine]] personalizzata.

**Dockerhub** è un **registro pubblico** dove si trovano migliaia  di immagini uffuciali (Ubuntu, MySQL, Redis...) già pronte da scaricare e usare e offre anche repository privati.

Dato che i container sono temporanei, Docker offre soluzioni per **persistere i dati**.

Quando si hanno **molti container** da gestire su più macchine, Docker da solo non basta. Entra in gioco **[[Kubernetes]]**.

## INSTALLAZIONE
Per installarlo, vai [[Installazione del Docker|qui]]