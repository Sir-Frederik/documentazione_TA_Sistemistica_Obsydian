L'Internet Control Message Protocol (ICMP) è un protocollo di rete di livello di rete utilizzato principalmente per la **diagnostica e il controllo** delle comunicazioni IP. A differenza di [[Protocollo TCP]] e [[Protocollo UDP]] non serve a trasportare dati applicativi, ma a scambiare **messaggi di servizio** tra dispositivi di rete.

Si trova nel livello [[Modello ISO - OSI|Rete]] del modello ISO-OSI, e si appoggia direttamente sull' [[Protocollo IP |IP]]

![[Pasted image 20260521165853.png]]

È il protocollo alla base del comando `ping`: quando si vuole verificare se un host è raggiungibile, si invia un pacchetto ICMP di tipo **Echo Request**, e il destinatario risponde con un **Echo Reply**.

Non è orientato alla connessione e non effettua alcun handshake: i messaggi vengono inviati e ricevuti in modo autonomo, senza stabilire una sessione.

### Caratteristiche principali

- **Nessuna connessione**: non esiste una fase di sincronizzazione tra mittente e destinatario.
- **Solo diagnostica**: non trasporta dati utente, ma informazioni sul funzionamento della rete.
- **Messaggi di errore**: notifica problemi come host irraggiungibile, TTL scaduto, o rete inesistente.
- **Base del ping**: il tipo di messaggio più comune è l'Echo Request/Reply, usato per testare la raggiungibilità di un host.

### Tipi di messaggio principali

|Tipo|Descrizione|
|---|---|
|`0`|Echo Reply (risposta al ping)|
|`3`|Destination Unreachable (host/rete irraggiungibile)|
|`8`|Echo Request (richiesta ping)|
|`11`|Time Exceeded (TTL scaduto)|

### Struttura del pacchetto

Il pacchetto ICMP si appoggia sull' [[Protocollo IP |IP]] Oltre all'header IP, contiene un campo **Type**, un campo **Code** (che specifica il sottotipo del messaggio), e un **Checksum** per la verifica dell'integrità.

Per visualizzare il traffico ICMP in tempo reale si può usare [[Wireshark 🦈]], filtrando con `icmp`. Con [[Scapy 🐍]]è possibile costruire e inviare pacchetti ICMP personalizzati.