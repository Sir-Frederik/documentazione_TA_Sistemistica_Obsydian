
Lo script `06-analyze-vpn.sh` invia i findings di un report OWASP a [[🦙OLLAMA]] — un motore AI self-hosted che gira su una macchina PGX raggiungibile via **WireGuard VPN**. Il modello produce un report HTML strutturato con sommario in italiano e dettaglio tecnico in inglese.

---

## Prerequisiti

Prima di lanciare lo script verificare che:

- La VPN WireGuard sia attiva verso la PGX
- Ollama sia in esecuzione sulla PGX con `OLLAMA_HOST=0.0.0.0`
- Il modello scelto sia già scaricato sulla PGX
- Almeno un report sia presente in `~/owasp-scan/reports/`

```bash
# Verifica VPN attiva
sudo wg show
ping -c 2 10.0.0.1

# Se VPN non attiva
sudo wg-quick up wg0

# Verifica Ollama raggiungibile e modelli disponibili
curl -s http://10.0.0.1:11434/api/tags | python3 -c "import sys,json; [print(m['name']) for m in json.load(sys.stdin).get('models',[])]"
```

---

## Modelli Consigliati

|Modello|Dimensione|Qualità|Velocità|Note|
|---|---|---|---|---|
|`qwen2.5:72b`|72B|Eccellente|Lenta (5-10 min)|Prima scelta|
|`qwen2.5:32b`|32B|Ottima|Media (2-5 min)|Buon compromesso|
|`llama3.3:70b`|70B|Ottima|Media (4-8 min)|Alternativa valida|
|`qwen3.6:35b`|35B|Ottima|Lenta (10+ min)|Thinking mode — sconsigliato|

> ⚠️ I modelli con **thinking mode** (`qwen3.x`, `deepseek-r1`) producono migliaia di token interni prima di rispondere — tempi molto più lunghi e spesso risposta vuota. Preferire sempre `qwen2.5` per questo caso d'uso.

---

## Utilizzo

```bash
# aggiungi ~/owasp-scan
./06-analyze-vpn.sh                                      # lista report e chiede quale analizzare
./06-analyze-vpn.sh --latest                             # analizza l'ultimo report
./06-analyze-vpn.sh --model qwen2.5:72b --latest         # usa modello specifico
./06-analyze-vpn.sh --model qwen2.5:32b --latest
./06-analyze-vpn.sh --report ~/owasp-scan/reports/20260518_144320/
```

---

## Struttura del Report Generato

Il report viene salvato come `ai-analysis.html` nella directory del report OWASP e visualizzabile tramite [[Script di Routine OWASP#04-view-report.sh — Visualizzazione Report|04-view-report.sh]].

|Sezione|Lingua|Contenuto|
|---|---|---|
|Sommario Esecutivo|Italiano|Rischio generale, finding principali, azioni urgenti|
|Risk Level|Italiano|CRITICO / ALTO / MEDIO / BASSO|
|Top 10 Vulnerabilities|Inglese|Prioritizzate per exploitability con CWE e flag falsi positivi|
|Probable False Positives|Inglese|Alert che probabilmente non sono vulnerabilità reali|
|Remediation Plan|Inglese|Azioni ordinate per priorità con effort e owner|
|Technical Details|Inglese|Per ogni finding critico: scenario di attacco e fix con codice|
|What Was Not Found|Inglese|Classi di vulnerabilità non coperte dallo scan|

---

## Flusso di Esecuzione

1. Verifica VPN WireGuard attiva
2. Verifica Ollama raggiungibile su `10.0.0.1:11434`
3. Verifica modello disponibile sulla PGX
4. Seleziona report da analizzare
5. Estrae findings da ZAP, Nikto, ASTF (max 50 alert)
6. Sanitizza caratteri speciali dal JSON
7. Costruisce prompt strutturato e lo salva su file
8. Invia a Ollama via `curl` con streaming
9. Raccoglie i token della risposta
10. Genera `ai-analysis.html` nella directory del report

---

[🦸🏻‍♂️OWASP] | [[🦙OLLAMA]] | [[Script di Routine OWASP]] | [[config.env — Guida alle Variabili]] | [[📎OWASP — Troubleshooting]]