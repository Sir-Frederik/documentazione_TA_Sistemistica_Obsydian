### 1. Introduzione SonarQube sui pacchetti

- Attivare **SonarQube su tutti i pacchetti**.
- Abilitazione tramite decommento di:
    - `pom.xml`
    - Jenkinsfile
- Responsabili: Alessandro, Federico, Genny.
- Obiettivo: iniziare su ambiente legacy e poi migrare progressivamente.

---

### 2. Gestione vulnerabilità (Trivy e Security Gate)

- Le vulnerabilità rilevate da **Trivy devono essere eliminate**.
- Il requisito di sicurezza è che la **Quality Gate security sia “A”**.
- Regole del gate:
    - Una singola vulnerabilità di sicurezza deve **bloccare la build**.
    - Le altre categorie di issue **non devono bloccare la build**.
- Da valutare la configurazione di SonarQube per:
    - escludere dai blocchi i gate non legati alla security
    - mantenere focus esclusivo sulla sicurezza

---

### 3. Integrazione Jenkins – SonarQube

- Verificare nel tool la chiamata da Jenkins verso SonarQube.
- Controllare se il risultato della scansione viene validato correttamente.
- Possibilità futura:
    - invio automatico via email del link al report SonarQube.

---

### 4. Migrazione MESIT e infrastruttura sicurezza

- Migrare i pacchetti **MESIT sulla nuova infrastruttura**.
- Includere:
    - build per **OpenVAS**
    - test con **Zed Attack Proxy (ZAP)**
- Obiettivo: standardizzare le pipeline di sicurezza.

---

### 5. Standardizzazione e documentazione

- Rendere tutti i processi **standard aziendali**.
- Creare documentazione operativa per tutte le attività svolte.

---

### 6. Gestione vulnerabilità “assorbite”

- Alcuni pacchetti (es. eventi MESIT) non mostrano vulnerabilità perché “assorbite”.
- È necessario:
    - controllo da parte di uno sviluppatore backend
    - verificare corretta propagazione delle vulnerabilità tra pacchetti

⚠️ Tutte le vulnerabilità critiche devono essere eliminate immediatamente.

---

### 7. Sicurezza immagini Docker e dipendenze

- Le immagini Docker utilizzate per il runtime devono essere:
    - prive di vulnerabilità note
- Anche le dipendenze (es. Maven `pom`) devono essere controllate e sicure.

---

### 8. Obiettivo finale (audit e clienti)

- Dimostrare ai clienti capacità di sicurezza tramite:
    - report **Trivy**
    - report **SonarQube**
    - report **OpenVAS**

---

### 9. Testing

- Implementare **unit test sui pacchetti aziendali**.
- Attualmente si usa Postman.
- Arianna e Saverio hanno competenze già presenti sul tema.