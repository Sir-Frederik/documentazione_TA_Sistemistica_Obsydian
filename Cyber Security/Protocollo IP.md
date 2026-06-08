---
tags:
  - Protocollo
---
L'Internet Protocol (IP) è un protocollo di rete fondamentale che opera al [[Modello ISO - OSI|livello di Rete]] del modello ISO-OSI. È il meccanismo base che permette di **indirizzare e instradare** i pacchetti di dati attraverso reti diverse, fino a raggiungere il destinatario.
![[Pasted image 20260521165853.png]]
È un protocollo **non orientato alla connessione** e **non affidabile**: non garantisce che i pacchetti arrivino a destinazione, nell'ordine corretto, o che arrivino del tutto. Questa responsabilità viene delegata ai protocolli di livello superiore, come il [[Protocollo TCP|TCP]].

### Caratteristiche principali

- **Indirizzamento**: assegna a ogni dispositivo un indirizzo IP univoco (sorgente e destinazione) che permette di identificarlo sulla rete.
- **Instradamento (Routing)**: i pacchetti vengono instradati hop-by-hop attraverso i router, ognuno dei quali decide il percorso migliore verso la destinazione.
- **Frammentazione**: se un pacchetto è troppo grande per la rete che deve attraversare, IP lo frammenta in pezzi più piccoli e li riassembla a destinazione.
- **TTL (Time To Live)**: ogni pacchetto ha un contatore che si decrementa ad ogni hop. Quando arriva a 0, il pacchetto viene scartato — questo evita che i pacchetti girino all'infinito sulla rete. Quando il TTL scade, il router invia un messaggio [[Protocollo ICMP|ICMP]] di tipo Time Exceeded al mittente.

### Versioni

Esistono due versioni del protocollo attualmente in uso:

|Versione|Lunghezza indirizzo|Esempio|
|---|---|---|
|IPv4|32 bit|`192.168.1.1`|
|IPv6|128 bit|`fe80::1`|

IPv4 è ancora il più diffuso, ma gli indirizzi disponibili sono esauriti, motivo per cui IPv6 è in progressiva adozione.

### Struttura del pacchetto

L'header IP contiene, tra le informazioni principali:

|Campo|Descrizione|
|---|---|
|**Version**|Versione del protocollo (4 o 6)|
|**TTL**|Time To Live|
|**Protocol**|Protocollo del livello superiore (es. TCP, UDP, ICMP)|
|**Source IP**|Indirizzo IP sorgente|
|**Destination IP**|Indirizzo IP destinazione|

Tutti i protocolli di trasporto come [[Protocollo TCP|TCP]], [[Protocollo UDP|UDP]] e [[Protocollo ICMP|ICMP]] si appoggiano sull'IP per essere veicolati sulla rete. Per analizzare il traffico IP si può usare [[Wireshark 🦈]] filtrando con `ip`, oppure costruire pacchetti personalizzati con [[Scapy 🐍]]