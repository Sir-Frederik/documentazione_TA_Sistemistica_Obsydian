
Gli script di routine automatizzano le operazioni più frequenti del testing di sicurezza sul server `ubuntu@ta-server-openvas`. Sono progettati per essere usati **in sequenza** — ogni script copre una fase specifica del workflow.

Tutti gli script si trovano in `~/owasp-scan/` e vanno resi eseguibili una volta sola:

```bash
chmod +x ~/owasp-scan/*.sh
```

---

## Panoramica

| Script               | Fase         | Scopo                                      | Durata      |
| -------------------- | ------------ | ------------------------------------------ | ----------- |
| `01-start-env.sh`    | Preparazione | Avvia DVWA, ZAP daemon, Falco              | ~45 secondi |
| `02-check-status.sh` | Verifica     | Controlla che tutti i servizi siano attivi | ~5 secondi  |
| `03-run-scan.sh`     | Esecuzione   | Lancia la pipeline `owasp-scan.sh`         | Variabile   |
| `04-view-report.sh`  | Analisi      | Avvia server HTTP per i report             | Istantaneo  |
| `05-cleanup.sh`      | Chiusura     | Ferma tutti i servizi                      | ~10 secondi |
| `06-analyze-vpn.sh`  | AI Analysis  | Invia i report a Ollama via VPN            | 2-5 minuti  |

---

## Sequenza Operativa

==Seguire sempre questa sequenza per una sessione di test completa e affidabile.==

```bash
# STEP 1 — Avvia l'ambiente
~/owasp-scan/01-start-env.sh

# STEP 2 — Verifica che tutto sia pronto
~/owasp-scan/02-check-status.sh

# STEP 3 — Lancia il test (scegli la modalità)
~/owasp-scan/03-run-scan.sh                # DAST rapido (default)
~/owasp-scan/03-run-scan.sh --dast-only   # solo Nikto + ZAP
~/owasp-scan/03-run-scan.sh --dast-api    # web + API
~/owasp-scan/03-run-scan.sh --full        # pipeline completa

# STEP 4 — Visualizza il report
~/owasp-scan/04-view-report.sh --latest

# STEP 5 — Analisi AI (opzionale, richiede VPN attiva)
~/owasp-scan/06-analyze-vpn.sh --latest

# STEP 6 — Chiudi l'ambiente
~/owasp-scan/05-cleanup.sh
```

---

## Gli Script in Dettaglio

### 01-start-env.sh — Avvio Ambiente

Prepara l'ambiente di test avviando tutti i servizi in sequenza. Va eseguito **all'inizio di ogni sessione**. Controlla automaticamente se i servizi sono già attivi prima di riavviarli.

**Cosa fa:**

- Monta il BPF filesystem (necessario per Falco su KVM)
- Carica il kernel module Falco e avvia il processo di monitoring
- Avvia il container Docker [[🎯DVWA]] sulla porta `8081`
- Avvia ZAP in modalità ==daemon== sulla porta `8080` (non webswing)
- Verifica che tutti i servizi rispondano prima di terminare

```bash
./01-start-env.sh           # avvia tutto
./01-start-env.sh --no-falco   # salta Falco
./01-start-env.sh --no-zap     # salta ZAP
./01-start-env.sh --no-dvwa    # salta DVWA
```

**Output atteso:**

```
[OK] BPF filesystem già montato
[OK] Falco avviato (PID: 2161232)
[OK] DVWA avviato sulla porta 8081 (HTTP 302)
[OK] ZAP daemon avviato (versione: 2.17.0, atteso 30s)
```

> ⚠️ ZAP impiega 20-30 secondi ad avviarsi. Lo script attende automaticamente fino a 60 secondi prima di segnalare un errore.

---

### 02-check-status.sh — Verifica Stato

Esegue una verifica completa di tutti i componenti prima di lanciare un test. Produce un riepilogo con **PASS**, **WARN** e **FAIL** per ogni controllo. Se ci sono FAIL critici esce con `exit code 1`.

```bash
./02-check-status.sh           # verifica completa
./02-check-status.sh --quiet   # mostra solo FAIL e WARN
```

**Cosa controlla:**

|Controllo|Critico|
|---|---|
|Docker — versione e daemon attivo|Sì|
|DVWA — container attivo e risposta HTTP 302|Sì|
|ZAP — API REST in modalità daemon|Sì|
|Falco — processo, BPF, kernel module|Sì|
|ASTF — JAR presente e Java 21|Sì|
|`owasp-scan.sh` — presente, eseguibile, sintassi corretta|Sì|
|`config.env` — presente con `TARGET_URL` configurato|Sì|
|WuppieFuzz, Semgrep, GitLeaks, Dep-Check, Nikto|No|

**Come interpretare il risultato:**

- `21 pass, 0 fail` → ambiente pronto, si può procedere
- `FAIL` presenti → eseguire `01-start-env.sh` per avviare i servizi mancanti
- Solo `WARN` → ambiente funzionante ma con limitazioni — valutare prima di procedere

---

### 03-run-scan.sh — Lancio Pipeline

Lancia `owasp-scan.sh` con la modalità scelta. Verifica che DVWA e ZAP siano raggiungibili prima di partire. Monitora l'output in tempo reale e segnala il percorso del report al termine.

> ⚠️ Prima di lanciare verificare sempre che ZAP sia in modalità **daemon**. Se è in modalità webswing lo script fallisce silenziosamente. Usare `02-check-status.sh` per verificare.

**Modalità disponibili:**

|Flag|Strumenti|Durata|Quando usarla|
|---|---|---|---|
|(nessuno)|Nikto + ZAP|~15 secondi|Verifica rapida quotidiana|
|`--dast-only`|Nikto + ZAP|~15 secondi|Solo test web|
|`--api-only`|ASTF + WuppieFuzz|5-15 minuti|Solo test API REST|
|`--dast-api`|Nikto + ZAP + ASTF|10-20 minuti|Test web + API completo|
|`--full`|Tutti gli strumenti|30-60 minuti|Pentest trimestrale|
|`--tool NOME`|Uno strumento specifico|Variabile|Debug o test singolo|

```bash
./03-run-scan.sh                 # DAST rapido (default)
./03-run-scan.sh --dast-only
./03-run-scan.sh --dast-api
./03-run-scan.sh --full
./03-run-scan.sh --tool nikto    # strumenti: nikto | zap | astf | wuppiefuzz
./03-run-scan.sh --tool zap      #             semgrep | gitleaks | depcheck | openvas
```

---

### 04-view-report.sh — Visualizzazione Report

Avvia un server HTTP Python nella directory del report selezionato. Elenca tutti i report disponibili con data, numero di file e alert ZAP trovati.
##### Digressione: avvio del server manualmente
Se preferisci avviarlo manualmente, posizionati nella directory del report e lancia:



```bash
cd ~/owasp-scan/reports/TIMESTAMP/
python3 -m http.server 9999
```

Il server gira sulla VPS e serve i file di quella directory — raggiungibile dal browser del tuo PC su `http://51.75.196.150:9999`. Si ferma quando chiudi il terminale o premi `Ctrl+C`. Per tenerlo attivo in background:

```bash
nohup python3 -m http.server 9999 &
```

#### Comandi dello Script


```bash
./04-view-report.sh              # elenca e chiede quale aprire
./04-view-report.sh --latest     # apre direttamente l'ultimo report
./04-view-report.sh --list       # elenca senza avviare il server
./04-view-report.sh --port 8888  # usa porta diversa da 9999
```

**URL disponibili dopo l'avvio:**

```
http://51.75.196.150:9999/index.html        # report unificato
http://51.75.196.150:9999/ai-analysis.html  # analisi AI
http://51.75.196.150:9999/nikto-report.html # report Nikto
http://51.75.196.150:9999/zap-alerts.json   # alert ZAP raw
```

Premere `Ctrl+C` per fermare il server quando si ha finito.

---

### 05-cleanup.sh — Pulizia Ambiente

Ferma in modo ordinato tutti i servizi e processi avviati durante la sessione.

**Cosa ferma:** container ZAP e DVWA, processo Falco, processi ASTF e WuppieFuzz residui, `owasp-scan.sh` se in esecuzione, server HTTP report, immagini Docker dangling.

```bash
./05-cleanup.sh            # ferma servizi, mantieni i report
./05-cleanup.sh --reports  # ferma tutto + cancella report > 7 giorni
./05-cleanup.sh --all      # ferma tutto + cancella TUTTI i report
./05-cleanup.sh --no-falco # ferma tutto tranne Falco
```

> 💡 Usare `--reports` periodicamente per liberare spazio disco — ogni scan genera 10-50 MB di report.

---

### 06-analyze-vpn.sh — Analisi AI

Invia i findings del report a [[Analisi AI con Ollama|Ollama sulla PGX via WireGuard VPN]] per un'analisi approfondita. Vedi la pagina dedicata per i dettagli.

---

## Casi d'Uso Comuni

### Test rapido quotidiano (~1 minuto)

```bash
./02-check-status.sh --quiet
./03-run-scan.sh --dast-only
./04-view-report.sh --latest
```

### Test completo con analisi AI (~15-30 minuti)

```bash
./01-start-env.sh
./02-check-status.sh
./owasp-scan.sh --skip-api --skip-openvas
./06-analyze-vpn.sh --latest
./04-view-report.sh --latest
./05-cleanup.sh
```

### Pentest trimestrale completo

```bash
# 1. Prepara il sorgente e aggiorna config.env
git clone https://github.com/REPO/app.git ~/source
nano ~/owasp-scan/config.env   # TARGET_URL, SOURCE_DIR, API_TOKEN, OPENVAS_HOST

# 2. Pipeline completa
./01-start-env.sh
./02-check-status.sh
./owasp-scan.sh
./06-analyze-vpn.sh --model qwen2.5:72b --latest
./04-view-report.sh --latest
./05-cleanup.sh --reports
```

---

[[🦸🏻‍♂️OWASP]] | [[config.env — Guida alle Variabili]] | [[Analisi AI con Ollama]] | [[📎OWASP — Troubleshooting]]