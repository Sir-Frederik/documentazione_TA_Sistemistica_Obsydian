**ELK** è un acronimo che indica 3 prodotti Open Source sviluppati da **Elastic**
1. **Elasticsearch**
2. **Logstash**
3. **Kibana**

Recentemente si è aggiunto anche **Filebeat** e il nuvo nome dello stack è ==**Elastic Stack**.==

```
🖥️ Server 1                🖥️ Server 2
    ↓                           ↓
📦 Filebeat              📦 Filebeat
    ↓                           ↓
    └──────────┬────────────────┘
               ↓
          ⚙️ Logstash
          (elabora e struttura)
               ↓
       🔍 Elasticsearch
       (salva e indicizza)
               ↓
          📊 Kibana
       (visualizza e analizza)
```


## 📤Filebeat
E' un agente leggero che gira sulle macchine e che ha il compito di raccogliere e **inviare** i **file di Log** verso un sistema centralizzato per l'analisi.
Ogni volta che viene aggiunta una nuova riga, la inoltre a **Logstash** o direttamente a **Elasticsearch**
Ogni volta che i file di log relativi aggiungono nuove righe, le inoltrano alla destinazione configurata.

## 👨‍🍳Logstash
È sostanzialmente un **motore di elaborazione dei log** (dati)
Raccoglie i Log da **Filebeat** li **trasforma e li normalizza**, poi li invia a **Elasticsearch**.
La fase in cui trasforma i dati si chiama **Filter** e può fare diverse operazioni:

|Operazione|Descrizione|
|---|---|
|**Parsing**|Analizza il testo del log e lo struttura|
|**Grok**|Estrae informazioni da testo non strutturato|
|**Mutate**|Modifica, rinomina o rimuove campi|
|**GeoIP**|Aggiunge informazioni geografiche agli indirizzi IP|
|**Date**|Normalizza i formati delle date|
Esempio:
```
Log grezzo:
"192.168.1.1 - GET /index.html 200"

Dopo Logstash:
{
  "ip": "192.168.1.1",
  "metodo": "GET",
  "pagina": "/index.html",
  "codice": "200"
}
```

## 🤓Elasticsearch
E' il cuore del sistema: è un **database di ricerca e analisi**.
Riceve i dati da **Logstash** e li salva in modo da poterli cercare e analizzare molto velocemente.

Elasticsearch usa una tecnica chiamata ==**Indice Invertito**==:

Invece di cercare documento per documento, crea un **indice** di tutte le parole con i documenti in cui appaiono. Questo lo rende **estremamente veloce** nelle ricerche.

Ricerca in tutto il testo dei documenti e li mette nel suo DB con l'indice. Può inoltre calcolare **statistiche** sui dati.
Viene interrogato trami semplici richiesta [[HTTPS e HTTP | HTTP]] cioè con **API Rest**

## 📊Kibana
E' l'==**interfaccia web**== del'' ELK Stack. Permette di visualizzare e analizzare o dato salvati in Elasticsearch.
Kibana usa un linguaggio di ricerca **KQL** (Kibana Query Language) per cercare nei log.

## Formati di dati supportati in input:
- **JSON**, **CSV**, **XML**, **Syslog**, **Plain text**, **Multiline**, **Grok patterns**, **CEF**, **GELF**, e molti altri tramite i plugin di Logstash o Beats.
  Per sapere come integrare i dati, vai [[Integrazione in ELK Stack di Zabbix, Prometheus e Falco.|qui]]
