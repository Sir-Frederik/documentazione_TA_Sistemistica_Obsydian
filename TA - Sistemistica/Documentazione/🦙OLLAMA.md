
**Ollama** è uno strumento open source che permette di scaricare ed eseguire **modelli AI in locale**, direttamente su una macchina propria, senza inviare dati a servizi cloud esterni come OpenAI o Anthropic.

In pratica è un motore che:

- scarica e gestisce i modelli (come un package manager, ma per LLM)
- li esegue sulla macchina host sfruttando CPU o GPU
- espone un'**API REST locale** compatibile con lo standard OpenAI, raggiungibile da altri programmi

---

## Perché usarlo

Il vantaggio principale è la **privacy** — i dati non escono mai dalla rete interna. Per un contesto come il nostro, dove i report di sicurezza contengono informazioni sensibili su vulnerabilità e infrastruttura, inviare tutto a un servizio cloud esterno sarebbe un rischio. Con Ollama tutto rimane sulla PGX.

Altri vantaggi:

- Nessun costo per token o abbonamento
- Funziona offline
- Supporta decine di modelli diversi intercambiabili

---

## Come Funziona

Ollama espone un server HTTP sulla porta `11434`. Qualsiasi programma che conosce quell'indirizzo può interrogarlo — nel nostro caso è `06-analyze-vpn.sh` che ci invia i findings OWASP via [[Analisi AI con Ollama|VPN WireGuard]].

Il flusso è:

```
ta-server-openvas  →  WireGuard VPN  →  PGX ThinkStation
  (findings OWASP)                       (Ollama :11434)
                                              ↓
                                        modello AI
                                              ↓
                                       ai-analysis.html
```

---

## I Modelli

Un **modello** è il cervello — il file addestrato che elabora il testo. Ollama li gestisce come pacchetti:

```bash
ollama pull qwen2.5:32b    # scarica un modello
ollama list                # lista modelli disponibili
ollama rm qwen2.5:32b      # rimuove un modello
```

I modelli si distinguono per:

- **Dimensione** (7B, 32B, 72B...) — il numero indica i miliardi di parametri. Più è grande, più è capace ma più lento e pesante
- **Famiglia** — architetture diverse con punti di forza diversi (ragionamento, codice, lingue...)
- **Thinking mode** — alcuni modelli "pensano ad alta voce" prima di rispondere, generando migliaia di token interni. Utile per problemi matematici complessi, ma controproducente per analisi di report — produce latenza altissima e a volte risposta vuota

### Modelli che usiamo

|Modello|Parametri|Thinking|Note|
|---|---|---|---|
|`qwen2.5:72b`|72B|No|Prima scelta per qualità|
|`qwen2.5:32b`|32B|No|Buon compromesso velocità/qualità|
|`llama3.3:70b`|70B|No|Alternativa valida|
|`qwen3.6:35b`|35B|==Sì==|Sconsigliato per i nostri report|

---

## API REST

Ollama espone endpoint compatibili con lo standard OpenAI. Il principale è `/api/generate` per le richieste singole e `/api/chat` per le conversazioni multi-turno.

```bash
# Esempio richiesta base
curl http://10.0.0.1:11434/api/generate -d '{
  "model": "qwen2.5:32b",
  "prompt": "Analizza questa vulnerabilità...",
  "stream": false
}'

# Lista modelli disponibili
curl http://10.0.0.1:11434/api/tags
```

> ℹ️ Nella nostra configurazione Ollama è avviato con `OLLAMA_HOST=0.0.0.0` per essere raggiungibile dalla VPN. Di default accetta connessioni solo da localhost.

---

## Differenza con i Servizi Cloud

||Ollama (locale)|ChatGPT / Claude (cloud)|

|                   | Ollama (locale)                   | ChatGPT / Claude (cloud)           |
| ----------------- | --------------------------------- | ---------------------------------- |
| **Privacy**       | I dati restano in rete interna    | I dati vengono inviati al provider |
| **Costo**         | Solo hardware                     | A consumo o abbonamento            |
| **Qualità**       | Dipende dall'hardware disponibile | Modelli più grandi e aggiornati    |
| **Connessione**   | Funziona offline                  | Richiede internet                  |
| **Aggiornamenti** | Manuali                           | Automatici                         |
| **Privacy**       | I dati restano in rete interna    | I dati vengono inviati al provider |
| **Costo**         | Solo hardware                     | A consumo o abbonamento            |
| **Qualità**       | Dipende dall'hardware disponibile | Modelli più grandi e aggiornati    |
| **Connessione**   | Funziona offline                  | Richiede internet                  |
| **Aggiornamenti** | Manuali                           | Automatici                         |


---

[🦸🏻‍♂️OWASP] | [[Analisi AI con Ollama]] | [[📎OWASP — Troubleshooting]]