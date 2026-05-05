E' un **Application server Java** Open Source. Esegue applicazioni sviluppare in Java EE/Jakarta EE. E' il server su cui vengono **deployate ed eseguite ** le applicazoni Java Enterprise.

```
Utente browser
      │
      ▼
   Apache          ← riceve la richiesta HTTP, fa da "portiere"
      │
      ▼
   WildFly         ← esegue l'applicazione Java (.war o .ear)
      │
      ▼
  Database         ← WildFly legge/scrive i dati

```
#### Application Server
Oltre ad essere un web server, cioè gestire le richieste HTTP, fornisce una serie di **servizi pronti all'uso** per le applicazioni:

| Servizio                 | Descrizione                                            |
| ------------------------ | ------------------------------------------------------ |
| **Gestione transazioni** | Garantisce che le operazioni sul database siano sicure |
| **Connection Pooling**   | Gestisce un pool di connessioni al database            |
| **Sicurezza**            | Autenticazione e autorizzazione degli utenti           |
| **Messaging**            | Comunicazione asincrona tra componenti                 |
| **Dependency Injection** | Gestione delle dipendenze tra i componenti             |

##  Cosa si "deploya" su WildFly?

Le applicazioni Java Enterprise si impacchettano in formati specifici:

|Formato|Descrizione|
|---|---|
|**WAR** (Web Archive)|Applicazione web|
|**EAR** (Enterprise Archive)|Applicazione enterprise complessa|
|**JAR** (Java Archive)|Libreria o microservizio|

## Architettura di WildFly

WildFly può funzionare in due modalità:

### Standalone Mode

- Un **singolo server** indipendente
- Più semplice da gestire
- Adatto per ambienti piccoli o sviluppo
In ***standalone.xml*** è possibile trovare i dati di configurazione, e dentro ci sono i **datasource ** che ti dicono con quale DB Wildfly è connesso. 

### Domain Mode

- **Più server** gestiti da un controller centrale
- Permette di gestire configurazioni e deploy su tutti i server da un unico punto
- Adatto per ambienti enterprise

text

```text
Domain Mode:
Domain Controller
      ↓
┌─────┼─────┐
Server1  Server2  Server3
```

## Come riavviare wildlfly
1. Trova i processi wildfly attivi: 
```
   ps aux | grep wildfly
```
Segna i **PID** (ex: 2454089)
2. Uccidi il processo:
```
   kill PID
```
(ex: kill 2454089)
verifica che sia fermo:
```
ps aux | grep wildfly
```
Deve uscire solo questa riga:
`
```
centos  12345  0.0  0.0 112812  980 pts/0  S+  00:00  0:00 grep --color=auto wildfly`
```
Se ci sono ancora dei processi attivi, forza con:
```
kill -9 PID
```
3. Riavvia Wildfly:
```
   JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java)))) nohup /opt/wildfly-14.0.1.Final/bin/standalone.sh -b 0.0.0.0 &
```
E aspetta 30 secondi:
4. Controlla che funzioni:
```
   tail -50 /opt/wildfly-14.0.1.Final/standalone/log/server.log | grep -i "bound\|started\|error\|denied"
```

| Cosa Cercare          | Significato   |
| --------------------- | ------------- |
| **Bound data source** | ✅ DB connesso |
| **Started**           |               |
