È uno strumento che funziona da **Load Balancer** e  **Reverse Proxy**, ovvero distribuisce il traffico di rete tra più server.

HAProxy decide come distribuire il traffico tramite diversi algoritmi:

|Algoritmo|Descrizione|
|---|---|
|**Round Robin**|Le richieste vengono distribuite in ordine ciclico|
|**Least Connections**|La richiesta va al server meno occupato|
|**Source**|Le richieste dello stesso IP vanno sempre allo stesso server|

## Health Check
HAProxy controlla periodicamente se i server sono **raggiungibili e funzionanti**. Se un server non risponde, viene **escluso automaticamente** dal bilanciamento.

### Differenza con [[🌐 Apache HTTP Server]]]
Apache nasce come **web server** ma può anche funzionare da Proxy, tuttavia ha una capacità di load balancer limitato ed una performance inferiore rispetto ad HA Proxy.
### Differenza con [[🌐 Nginx]]
Anche Nginx può lavorare come **Load Balancer** ma è più adatto nel bilanciare un contenuto statico, come HTML, immagini, video ecc.
Il contenuto statico non richiede elaborazione, solo **lettura dal disco e invio**. Nginx è ottimizzato esattamente per questo:

- Legge il file
- Lo invia direttamente al client
- Nessuna elaborazione necessaria

Questo lo rende estremamente veloce e leggero per questo tipo di operazioni.
