
Raccolta dei problemi più comuni con causa e soluzione. Se un problema non è presente qui, controllare il log della pipeline:

```bash
tail -50 ~/owasp-scan/pipeline.log
```

---

## ZAP

**ZAP API non risponde su `/JSON/`**

- **Causa:** ZAP è in modalità webswing invece che daemon
- **Soluzione:** Riavviare in modalità daemon

```bash
docker stop zap && docker rm zap
docker run -d --name zap -p 8080:8080 \
  ghcr.io/zaproxy/zaproxy:stable zap.sh -daemon \
  -host 0.0.0.0 -port 8080 \
  -config api.addrs.addr.name=.* \
  -config api.addrs.addr.regex=true \
  -config api.key=dallaradallara20262026
```

**ZAP API 404 sulla porta 8080**

- **Causa:** Webswing intercetta le richieste sulla stessa porta
- **Soluzione:** Usare esclusivamente la modalità daemon — vedi [[Script di Routine OWASP#01-start-env.sh — Avvio Ambiente|01-start-env.sh]]

**ZAP si ferma durante lo scan**

- **Causa:** Container crashato durante la scansione
- **Soluzione:**

```bash
docker ps | grep zap   # verifica che sia ancora attivo
```

---

## Falco

**`Trying to open engine` all'avvio**

- **Causa:** BPF filesystem non montato — comune dopo un riavvio della VPS
- **Soluzione:**

```bash
sudo mount -t bpf bpf /sys/fs/bpf
```

> ℹ️ Il mount è già aggiunto a `/etc/fstab` — dovrebbe montarsi automaticamente ad ogni riavvio. Se il problema si ripresenta verificare che la riga sia presente:

```bash
grep bpf /etc/fstab
```

---

## owasp-scan.sh

**`unbound variable` durante l'esecuzione**

- **Causa:** Bug array bash in heredoc Python — versione dello script non aggiornata
- **Soluzione:** Aggiornare lo script alla versione corrente

**Pipeline termina con `exit code 1` ma il report è stato generato**

- **Causa:** Tool saltati (es. gitleaks senza `SOURCE_DIR`) interpretati come errore nel riepilogo finale
- **Soluzione:** Ignorabile — il report è valido. Verificare comunque il log per escludere altri errori

```bash
tail -20 ~/owasp-scan/pipeline.log
```

---

## ASTF

**`invalid target release: 21` — Maven usa Java 17**

- **Causa:** `JAVA_HOME` punta a una versione precedente
- **Soluzione:**

```bash
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
```

**Report non generato — processo terminato prima della fine**

- **Causa:** Timeout scaduto o processo killato
- **Soluzione:** Aumentare `ASTF_TIMEOUT` in `config.env` oppure lanciare con `nohup`

```bash
nohup ./owasp-scan.sh > ~/owasp-scan/pipeline.log 2>&1 &
```

---

## WuppieFuzz

**`command not found`**

- **Causa:** PATH cargo non caricato nella sessione corrente
- **Soluzione:**

```bash
source $HOME/.cargo/env
```

---

## Dependency-Check

**Timeout durante il download del database NVD**

- **Causa:** Rate limit API NVD senza chiave
- **Soluzione:** Aggiungere `--nvdApiDelay 6000` e verificare che `NVD_API_KEY` sia configurata in `config.env`

---

## Analisi AI

**Risposta vuota da Ollama**

- **Causa:** Modello con thinking mode attivo (`qwen3.x`, `deepseek-r1`)
- **Soluzione:** Usare `qwen2.5:32b` o `qwen2.5:72b` — vedi [[Analisi AI con Ollama#Modelli Consigliati|modelli consigliati]]

**Timeout durante l'analisi**

- **Causa:** Modello troppo lento o VPN instabile
- **Soluzione:** Usare un modello più piccolo oppure aumentare `TIMEOUT_SEC` nello script

**`VPN non trovata`**

- **Causa:** WireGuard non attivo
- **Soluzione:**

```bash
sudo wg-quick up wg0
sudo wg show   # verifica handshake recente
```

---

## OpenVAS

**Scan bloccato a 0% per ore**

- **Causa:** Port scan TCP+UDP troppo lento
- **Soluzione:** Nell'interfaccia Greenbone cambiare la Port List in `All TCP` — esclude UDP e velocizza drasticamente lo scan

---

[🦸🏻‍♂️OWASP] | [[Script di Routine OWASP]] | [[config.env — Guida alle Variabili]] | [[Analisi AI con Ollama]]