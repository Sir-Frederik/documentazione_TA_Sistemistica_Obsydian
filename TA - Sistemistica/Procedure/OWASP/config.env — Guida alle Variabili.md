**`config.env`** è ==l'unico file da modificare== per configurare la pipeline OWASP. Lo script `owasp-scan.sh` non va mai toccato direttamente.

```bash
nano ~/owasp-scan/config.env
```

> ⚠️ Non usare mai credenziali di produzione. Usare sempre utenti di test dedicati con permessi minimi.

---

## Variabili Obbligatorie

Sono solo 3 — senza di queste la pipeline non parte:

|Variabile|Scopo|Esempio|
|---|---|---|
|`TARGET_URL`|URL completo dell'app web da testare|`http://172.17.0.1:8081`|
|`ZAP_PORT`|Porta API REST di ZAP|`8080`|
|`ZAP_API_KEY`|API key configurata nel docker run di ZAP|`dallaradallara20262026`|

---

## Tutte le Variabili

### Progetto

|Variabile|Descrizione|Esempio|
|---|---|---|
|`PROJECT_NAME`|Nome del progetto — usato nel titolo del report HTML e nei log. Non influenza il comportamento dello script.|`"myapp"`|

---

### Target

|Variabile|Descrizione|Esempio|Obbligatoria|
|---|---|---|---|
|`TARGET_URL`|URL completo dell'app web con schema e porta. Usato da Nikto e ZAP. Deve essere raggiungibile dalla VPS.|`"http://172.17.0.1:8081"`|==Sì==|
|`TARGET_API_URL`|URL base delle API REST per ASTF e WuppieFuzz. Se vuoto usa `TARGET_URL` come fallback.|`"http://172.17.0.1:8082/api"`|No|
|`TARGET_IP`|IP del target senza porta. Usato da OpenVAS per creare il target di scan.|`"51.75.196.150"`|Solo per OpenVAS|

---

### Codice Sorgente

> ⚠️ Se `SOURCE_DIR` è vuoto, le fasi GitLeaks, Semgrep e Dependency-Check vengono **saltate automaticamente**.

|Variabile|Descrizione|Esempio|
|---|---|---|
|`SOURCE_DIR`|Path assoluto alla directory con il codice sorgente. Deve essere una directory git inizializzata per GitLeaks.|`"/home/ubuntu/myapp"`|

---

### Credenziali API

|Variabile|Descrizione|Esempio|
|---|---|---|
|`API_TOKEN`|Bearer token JWT/OAuth per ASTF. Ha priorità su `API_USERNAME` se entrambi sono configurati. Includere il prefisso `Bearer`.|`"Bearer eyJhbGci..."`|
|`API_USERNAME`|Username per Basic Authentication. Ignorato se `API_TOKEN` è configurato.|`"admin"`|
|`API_PASSWORD`|Password per Basic Authentication. Usato con `API_USERNAME`.|`"password"`|

---

### ZAP

> ⚠️ ZAP deve essere in modalità **daemon** per essere usato dallo script. Verificare sempre prima di lanciare. Vedi [[Script di Routine OWASP#01-start-env.sh — Avvio Ambiente|01-start-env.sh]].

|Variabile|Descrizione|Esempio|Obbligatoria|
|---|---|---|---|
|`ZAP_PORT`|Porta API REST di ZAP. Deve corrispondere alla porta del `docker run`.|`"8080"`|==Sì==|
|`ZAP_API_KEY`|API key configurata nel `docker run` con `-config api.key=VALORE`. Deve corrispondere esattamente.|`"dallaradallara20262026"`|==Sì==|

---

### Dependency-Check

|Variabile|Descrizione|Esempio|
|---|---|---|
|`NVD_API_KEY`|API key gratuita NVD per scaricare il database CVE senza timeout. Registrarsi su `nvd.nist.gov`.|`"ef5e1703-..."`|
|`CVSS_THRESHOLD`|Soglia CVSS (0-10) per segnalare findings. `7.0` = High + Critical. `9.0` = solo Critical.|`"7.0"`|

---

### ASTF

|Variabile|Descrizione|Esempio|
|---|---|---|
|`ASTF_TIMEOUT`|Timeout scansione in minuti. Su target locali 5-10 min, su target remoti 20-30. Se scade il report non viene generato.|`"10"`|
|`ASTF_THREADS`|Thread paralleli. Max 8 su VPS con 4 core. Su target pubblici usare 3 per evitare rate limiting.|`"8"`|

---

### WuppieFuzz

|Variabile|Descrizione|Esempio|
|---|---|---|
|`OPENAPI_SPEC`|Path assoluto alla specifica OpenAPI. Deve contenere il campo `servers`. Se vuoto WuppieFuzz viene saltato.|`"/home/ubuntu/openapi.json"`|
|`WUPPIE_TIMEOUT`|Timeout fuzzing in secondi. 120s per test rapidi, 300s+ per fuzzing approfondito.|`"120"`|
|`WUPPIE_AUTH_FILE`|Path a file YAML con configurazione autenticazione. Supporta bearer, oauth, basic.|`"/home/ubuntu/auth.yaml"`|

---

### OpenVAS

> ℹ️ OpenVAS viene avviato in background. Il report va recuperato manualmente dall'interfaccia Greenbone su `http://IP-OPENVAS:9392`.

|Variabile|Descrizione|Esempio|
|---|---|---|
|`OPENVAS_HOST`|IP della macchina Greenbone. Se vuoto la fase OpenVAS viene saltata.|`"192.168.1.100"`|
|`OPENVAS_PORT`|Porta Greenbone Web Interface. Default `9392`.|`"9392"`|
|`OPENVAS_USER`|Username Greenbone.|`"admin"`|
|`OPENVAS_PASS`|Password Greenbone.|`"password"`|

---

### Report

|Variabile|Descrizione|Esempio|
|---|---|---|
|`AUTO_SERVE`|Se `true` avvia server HTTP Python alla fine della pipeline. Usare `false` per nohup o CI/CD.|`"false"`|
|`SERVE_PORT`|Porta server HTTP report. Verificare che non sia in uso.|`"9999"`|

---

## Esempio Completo

```bash
PROJECT_NAME="myapp-backend"
TARGET_URL="http://staging.example.com:8080"
TARGET_API_URL="http://staging.example.com:8080/api/v1"
TARGET_IP="10.0.1.50"
SOURCE_DIR="/home/ubuntu/myapp"
API_TOKEN="Bearer eyJhbGciOiJIUzI1NiJ9..."
ZAP_PORT="8080"
ZAP_API_KEY="dallaradallara20262026"
NVD_API_KEY="ef5e1703-c84b-4df3-902b-15926c8d7ff6"
CVSS_THRESHOLD="7.0"
ASTF_TIMEOUT="10"
ASTF_THREADS="8"
OPENAPI_SPEC="/home/ubuntu/myapp/openapi.json"
WUPPIE_TIMEOUT="120"
WUPPIE_AUTH_FILE="/home/ubuntu/wuppie-auth.yaml"
OPENVAS_HOST="192.168.1.100"
OPENVAS_PORT="9392"
OPENVAS_USER="admin"
OPENVAS_PASS="password"
AUTO_SERVE="false"
SERVE_PORT="9999"
```

---

[[🦸🏻‍♂️OWASP]] | [[Script di Routine OWASP]] | [[Analisi AI con Ollama]] | [[📎OWASP — Troubleshooting]]