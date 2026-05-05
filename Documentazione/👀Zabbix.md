E' una **piattaforma di monitoraggio**  enterprise completa e  open source.
A differenza di [[👀 Prometheus]] che è più moderno e orientato ai microservizi, Zabbix è una soluzione **tutto in uno**.

```
Dispositivi da monitorare
(Server, Router, Switch, DB...)
           ↓
    Zabbix Agent          ← Software di monitoraggio
           ↓
    Zabbix Proxy          ← Opzionale, per reti remote
           ↓
    Zabbix Server         ← Cuore del sistema
           ↓
       Database           ← MySQL/PostgreSQL
           ↓
    Zabbix Frontend       ← Interfaccia web
           ↓
       Notifiche          ← Email, Slack, SMS...
```

### Zabbix Server
E' il cuore del sistema e si occupa di:
- **Raccogliere** i dati degli Agent e dei Proxy
- **Elaborare** le metriche ricevute
- **Generare alert** se nota problemi
- **Salvare** tutto nel DB
- **Inviare notifiche** agli amministratori.

### Zabbix Agent
E' il software installato su ogni server da monitorare. Esistono due versioni:
1. **Agent:** Versione classica
2. **Agent 2** Versione moderna, più performante e con più plugin.
   Può lavorare sia in modalità **attiva**, in cui manda i dati al server, oppure **passiva** in cui il server gli chiede periodicamente i dati.

### Zabbix Proxy
E' un componente **opzionale** che si mette come intermezzo tra gli Agent e il Server.
E' utile per alleggerire il carico sul Server centrale, oppure quando i server da monitorare sono in reti remote.
```
Rete Remota:                    Sede Centrale:
Z.Agent 1 ─┐
Z.Agent 2 ──→ Zabbix Proxy ────→ Zabbix Server
Z.Agent 3 ─┘
```

### Altri servizi
Zabbix ha anche un **database relazionale** in cui salva le informazioni e un **Frontend** con cui l'amministratore interagisce.

### Sistema di alerting

Zabbix ha un sistema di alerting molto avanzato con livelli di severità:

|Livello|Descrizione|
|---|---|
|🔵 **Not classified**|Problema non classificato|
|⚪ **Information**|Solo informativo|
|🟡 **Warning**|Attenzione richiesta|
|🟠 **Average**|Problema moderato|
|🔴 **High**|Problema grave|
|🔴 **Disaster**|Situazione critica|

