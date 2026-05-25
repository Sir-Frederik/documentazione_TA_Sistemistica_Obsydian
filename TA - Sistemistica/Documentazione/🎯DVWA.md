
**DVWA** (Damn Vulnerable Web Application) è un'applicazione web ==volutamente piena di vulnerabilità==, progettata per essere attaccata. È usata come **target di test** per verificare che strumenti e pipeline di sicurezza funzionino correttamente, senza dover attaccare sistemi reali.

---

## Accesso

L'url del DVWA di Openvas è : `http://51.75.196.150:8081`



> ⚠️ Al primo accesso andare su `http://51.75.196.150:8081/setup.php` e cliccare **Create/Reset Database**.

---

## A Cosa Serve

Invece di attaccare un sistema reale — illegale senza autorizzazione scritta — si attacca DVWA che è costruito apposta per questo. Nel nostro caso è il bersaglio su cui girano **Nikto** e **ZAP** per verificare che la [[🦸🏻‍♂️OWASP|pipeline OWASP]] funzioni correttamente prima di usarla su target reali.

---

## Vulnerabilità Presenti

DVWA contiene le vulnerabilità più comuni della [[🦸🏻‍♂️OWASP#OWASP Top 10|OWASP Top 10]]:

- **SQL Injection** — campi di input che accettano query SQL malevole
- **XSS** (Cross-Site Scripting) — campi che eseguono JavaScript iniettato
- **Command Injection** — input che passa comandi al sistema operativo
- **File Upload** — upload di file eseguibili malevoli
- **Brute Force** — login senza protezione contro tentativi ripetuti

---

## Livelli di Difficoltà

DVWA permette di regolare il livello di sicurezza dal pannello **DVWA Security**:

|Livello|Comportamento|
|---|---|
|**Low**|Nessuna protezione — vulnerabilità facilmente sfruttabili|
|**Medium**|Protezioni parziali — aggirabili con tecniche semplici|
|**High**|Protezioni più robuste — richiedono tecniche avanzate|
|**Impossible**|Codice sicuro — usato come riferimento per il codice corretto|

---

## Nota sui Risultati degli Scan

Con scan **non autenticato** (senza credenziali) i finding si fermano a Medium/Low — ZAP e Nikto vedono solo la superficie pubblica. Con scan **autenticato** i finding High emergono ovunque — SQLi, XSS e Command Injection sono presenti in tutte le sezioni. È per questo che i risultati del test dell'11/05/2026 riportano **0 High** pur su un'applicazione notoriamente vulnerabile.

---

## Gestione Container

```bash
# Avvio
docker run --rm -d -p 8081:80 vulnerables/web-dvwa

# Verifica
curl -s -o /dev/null -w "%{http_code}" http://localhost:8081  # atteso: 302

# Stop
docker stop $(docker ps -q --filter ancestor=vulnerables/web-dvwa)

# Restart
docker restart $(docker ps -q --filter ancestor=vulnerables/web-dvwa)
```

> ℹ️ Nel flusso normale non serve gestire il container manualmente — ci pensa [[Script di Routine OWASP#01-start-env.sh — Avvio Ambiente|01-start-env.sh]].

---

[🦸🏻‍♂️OWASP] | [[Script di Routine OWASP]] | [[OWASP — Troubleshooting]]