### SAST - **Static Application Security Testing**
==Analizza il codice sorgente== senza eseguirlo. Cerca dei **pattern pericolosi** nel codice, tipo una funzione che accetta input utente e lo passa direttamente ad una query SQL senza sanitizzazione. Lo strumento principale sulla pipeline Jenkins è [[SonarQube]]. [[Semgrep]] è usato nella pipeline [[🦸🏻‍♂️OWASP]] dedicata ai test di sicurezza (`ta-server-openvas`).

### DAST - Dynamic Application Security Testing
Al contrario del SAST, costui ==attacca l'applicazione== mentre è in esecuzione, dall'esterno, come farebbe un attaccante reale. Non vede il codice, ==vede quello che vede un client HTTP== o un browser.
Per le Web App lo fanno [[Nikto]] e [[ZAP]], per le API [[WuppieFuzz]] e [[ASTF]]. [[🕵️‍♀️ OpenVAS]] è lo strumento secondario, ma non gira su Jenkins — viene usato in un ambiente isolato su una replica non produttiva della piattaforma.

### Container Scan
==**Analizza** le immagini **Docker**== prima del deploy, cercando vulnerabilità gravi. Strumento : [[Trivy]].

### SCA - Software Composition Analysis
==Analizza le dipendenze== del progetto (*librerie, pacchetti npm, jar...*) e ==confronta con database di vulnerabilità note [[CVE e vulnerabilità|CVE]] ==. Se usi una libreria con una falla di sicurezza conosciuta, te lo segnala. Sulla pipeline [[🦸🏻‍♂️OWASP]] se ne occupa [[Dependency-Check]]. [[Trivy]] copre parzialmente questa categoria (fa anche da **Container Scan**) ma non analizza tutte le dipendenze in profondità. Si sta [[28-04-26 Aggiornamento – Catena di sicurezza applicativa|cercando]] un tool dedicato con questi requisiti: open source, installabile su macchina interna e che analizzi il codice da Git.

### Secrets Detection
Cerca nel codice e nella storia del repository git dati come: credenziali, API key, password, token e altri dati che non dovrebbero mai finire in un repository.
Se ne occupa [[GitLeaks]].

### Runtime Security
==**Monitora** quello che succede **in produzione**==, dopo il deploy, in tempo reale.
Intercetta comportamenti sospetti a livello di sistema operativo. Lo fa [[🚨Falco]], unito a [[👀 Prometheus]] e [[👀Zabbix]] per le metriche infrastrutturali.

## Perchè si usa una pipeline?
Nessuno strumento riesce a coprire tutto. Ogni analisi ha punti ciechi:
- Il SAST non sa come si comporta l'app a runtime,
- Il DAAST non vede il codice sorgente e le logiche interne.
- Nessuno dei due vede le vulnerabilità nelle librerie di terze parti
- Nessuno monitora cosa succede dopo il deploy.