Qui vedremo come integrare i dati di [[👀Zabbix]] e [[👀 Prometheus]] in [[ELK STACK - Elastic Stack]].
Vedremo anche il confronto tra due approcci di monitoraggio in tempo reale:  [[🚨Falco]] e "Wazuh + Tetragon".
```
Prometheus   ──→ ┐
Zabbix Agent ──→ ├──→  Filebeat  ──→  Elasticsearch  ──→  Kibana (alert)
Falco        ──→ ┘
```
Il punto centrale è **Logstash** o ancora meglio [[ELK STACK - Elastic Stack#📤Filebeat| Filebeat]] che fa da collettore e normalizzatore prioma di inviare tutto ad [[ELK STACK - Elastic Stack#🤓Elasticsearch| Elasticsearch]] 
## [[👀 Prometheus]]

Prometheus non produce log, ma **==metriche==**. Filebeat non sa fare scraping delle metriche di Pr.
Quindi la strategia consiste nell'usare **Metricbeat** oppure **Prometheus Alertmanager**.

### Tramite Metricbeat
Metricbeat è un agente della suite **Elastic** che fa scraping diretto dell'endpoint `/metrics` di Prometheus e li manda a Elasticsearch.

In pratica Met. si installa sulla macchina oppure come pod di Kubernetes e interroga periodicamente Prometheus tramite HTTP.  Legge semplicemente quello che Prometheus già espone.
Coesiste tranquillamente con Filebeat e necessita un **Endpoint Prometheus** raggiungibile: (es. `http://<ip>:9090`).
Nel file `metricbeat.yml` si abilita il **modulo** Prometheus:

```yaml
metricbeat.modules:
  - module: prometheus
    metricsets: ["collector"]
    period: 30s #ogni quanto fare scraping
    hosts: ["http://<ip-prometheus>:9090"]
    metrics_path: /metrics

output.elasticsearch:
  hosts: ["http://<ip-elasticsearch>:9200"]
```

Su Elasticsearch arrivano ==tutte le metriche==  esposte di Prometheus in formato compatibile con KIbana.
Per il contesto della sicurezza, le più rilevanti sono:
- `kube_pod_container_status_restarts_total` — pod in crash loop
- `container_cpu_usage_seconds_total` — spike anomali di CPU
- `kube_node_status_condition` — stato dei nodi
#### Limiti
Metricbeat manda **tutte** le metriche, che possono essere migliaia. Il ==volume dati== può diventare ==significativamente alto==: da valutare con attenzione prima di andare in produzione.
### Tramite Alertmanager.
Alertmanager è un componente di Prometheus che gestisce gli **Alert** già definiti tramite regole (**`PrometheusRule`**).
Manda solo gli eventi che Prometheus ha già identificato come **Problemi**.
Ergo, necessita di un ==**volume di dati** più **basso**== rispetto a Metricbeat.
Per le nostre esigenze, questa via è più **idonea**

```
Prometheus (valuta le regole) → Alertmanager (riceve l'alert) → webhook → Elasticsearch
```

Bisogna configurare le **regole di alerting**, `PrometeusRole` in Kubernetes e, Elasticsearch deve poter essere raggiungibile da Alertmanager.
Per configurarlo, nel file `alertmanager.yml` si configura un *receiver* di tipo "*webhook*"

```bash
route:
  receiver: "elasticsearch"

receivers:
  - name: "elasticsearch"
    webhook_configs:
      - url: "http://<ip-elasticsearch>:9200/prometheus-alerts/_doc"
        send_resolved: true
```

In questo modo Alertmanager manda una *POST* JSON direttamente a Elasticsearch ogni volta che scatta un alert.
##### Struttura JSON - Esempio
```
{
  "receiver": "elasticsearch",
  "status": "firing",
  "alerts": [
    {
      "status": "firing",
      "labels": {
        "alertname": "PodCrashLooping",
        "namespace": "production",
        "severity": "critical"
      },
      "annotations": {
        "summary": "Pod in crash loop da 5 minuti"
      },
      "startsAt": "2026-04-30T10:00:00Z"
    }
  ]
}
```
## [[👀Zabbix]]

**Zabbix Agent** è installato sui nodi e raccoglie le metriche di sistema e le invia a **Zabbix Server**, dove le valuta e genera degli "*alert*" o "*trigger*".
Abbiamo due approcci:

### Tramite Filebeat
Come detto prima, Zabbix deve inviare le informazioni a [[ELK STACK - Elastic Stack#📤Filebeat |Filebeat]]. Quest'ultimo, a differenza di Logstash non può ricevere dei Webhook, quindi bisogna passare per un **file intermedio**.

Sul Zabbix Server ==si configura un **Media Type**== di tipo **"script"** che ad ==ogni alert==, ==scrive una riga JSON su un file di log== dedicato.

``` bash
# /usr/lib/zabbix/alertscripts/to_elk.sh
echo "{\"timestamp\":\"$1\",\"alert\":\"$2\",\"severity\":\"$3\",\"host\":\"$4\"}" >> /var/log/zabbix/elk-alerts.log
```
 Filebeat deve monitorare quel file:
 
``` yaml
filebeat.inputs:
  - type: log
    paths:
      - /var/log/zabbix/elk-alerts.log
    json.keys_under_root: true

```

### Webhook diretto a Elasticsearch
Dato che Zbx. Server supporta i Media Type di tipo **webhook HTTP** si può configurare per mandare il JSON dell'alert direttamente a Elasticsearch (bypassando Filebeat).

Anche se è una via più semplice, si perde la centralizzazione su Filebeat e la pipeline è quindi meno ordinata.

## Confronto tra Falco e la combo Wazuh + Tetragon

Gli strumenti Prometheus e Zabbix  coprono il **monitoraggio infrastrutturale**: metriche di sistema, stato dei nodi, disponibilità dei servizi. Non sono però in grado di rilevare **comportamenti anomali a runtime**, ovvero quello che accade _dentro_ i container mentre l'applicazione è in esecuzione.

È qui che si inserisce il **runtime security monitoring**: uno strato di sicurezza che ==osserva in tempo reale== le syscall, i processi, gli accessi ai file e il traffico di rete, ==rilevando attività sospette== come l'apertura di una shell in un container, una privilege escalation o l'accesso a file di sistema sensibili.

Questo tipo di minacce non è intercettabile né da SonarQube (che analizza il codice statico), né da Trivy (che scansiona l'immagine prima del deploy), né da Prometheus o Zabbix. Si manifestano **solo a runtime**, spesso come conseguenza di una vulnerabilità non nota o di un attacco in corso.

Per coprire questo gap esistono due approcci principali nel nostro contesto Kubernetes: **Falco** e la combo **Wazuh + Tetragon**.
### [[🚨Falco]]
Falco è un progetto **CNCF** ( Cloud Native Computing Foundation) specializzato nel ==runtime security per i [[Container| container]] e [[Kubernetes]]==. 
Monitora le syscall del kernel tramite **eBPF** (o kernel module) e quindi ==genera alert quando rileva comportamenti anomali rispetto alle **regole predefinite**==.

##### Punti di forza:
- Nativo per Kubernetes, conosce il concetto di pod/container/namespace
- Regole già pronte per i casi più comuni (shell in container, accesso a file sensibili, privilege escalation)
- Output JSON strutturato, integrazione con ELK immediata
- **Leggero**, pensato per non impattare le performance
- Matura e molto adottato (CNCF graduated project)

##### Limiti:
- Solo **osservazione**, non può bloccare/enforcement
- Focalizzato su K8s/container, copertura limitata fuori da quel perimetro
- Non è un SIEM completo
### Combo Wazuh + Tetragon  
#### Wazuh
E' una piattaforma **SIEM/XDR** completa e va ben ==oltre il runtime security==:
copre **log analysis, file integrity monitoring (FIM), vulnerability detection e compliance (GDPR, PCI, DSS, ecc...)**.
Presenta un ==**agente** da installare in ogni nodo==.

##### Punti di forza:
- **Copertura molto ampia**: log, integrità file, vulnerabilità, compliance
- Ha una **propria dashboard** (basata su OpenSearch/Kibana)
- Può mandare dati anche verso Elasticsearch esterno
- Copre sia ambienti container che VM tradizionali
##### Limiti:
- **Pesante** da installare e gestire (server Wazuh + agenti su tutti i nodi)
- **Overlap** con ELK che già abbiamo — rischio di duplicare infrastruttura
- Non specifico per Kubernetes come Falco

#### Tetragon
E' un progetto Cillium basato su **eBPF** (o kernel module), focalizato sulla ==osservabilità e **enforcement** a livello kernel==.
E' più recente e più basso livello rispetto a Falco.
##### Punti di forza:
- **Visibilità profondissima**: syscall, esecuzione processi, traffico di rete, accesso file — tutto tracciato a livello kernel
- **Enforcement attivo**: può bloccare syscall, killare processi, droppare pacchetti in tempo reale (non solo osservare)
- Integrato nativamente con Cilium (se lo usassimo come CNI nel cluster)
- Output JSON strutturato
##### Limiti:
- Più complesso da configurare rispetto a Falco
- Progetto più giovane, meno maturo e meno documentato
- Richiede kernel recente con supporto eBPF completo

#### Punti di forza e limiti della Combo
L'idea è di coprire due livelli diversi:

- **Tetragon** → runtime enforcement a livello kernel nel cluster K8s
- **Wazuh** → SIEM completo per log analysis, FIM e compliance sull'intera infrastruttura

##### Punti di forza:
- Copertura molto ampia, dal kernel al SIEM
- Tetragon copre il gap di enforcement che Falco non ha
- Wazuh copre l'infrastruttura fuori da Kubernetes
##### Limiti:
- Complessità molto elevata: due stack da installare, configurare e mantenere
- Wazuh ha una propria UI che si sovrappone alELK già esistente
- Rischio di duplicazione di dati e alert
- Costo operativo significativo per un team quale il nostro.

### Confronto finale (Falco vs Wazu + Tetragon)
Per il nostro contesto, che ha un progetto su Kubernetes, ELK già esistente e abbiamo attualmente un team con risorse limitate, ==**Falco è la scelta più pragmatica**.==

La combo Wz + Tr ha senso qualora in futuro volessimo estendere il monitoraggio anche alle VM fuori dal cluster e se avessimo bisogno di un enforcerment attivo. Ma attualmente aggiunge una complessità difficile in questa fase.