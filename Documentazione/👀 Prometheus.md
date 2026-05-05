E' un sistema di **monitoraggio** Open Source che raccoglie dati (CPU, memoria, disco, tempi di risposta delle app, rete...) forniti da [[📊 Node Exporter]] e li memorizza sullo stato dei server e nelle applicazioni.
Funziona con un meccanismo chiamato "**scraping"**, cioè va a raccogliere i dati (*metrics*) esposti dal server periodicamente e lì salva nel suo **database**.
Se qualcosa va storto, lo segnala ad [[🔔Alertmanager]].

Prometheus ha un linguaggio chiamato **PromQl**.
Per funzionare, Prometheus richiede **Grafana**.