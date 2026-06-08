---
tags:
  - Protocollo
---
Il **User Datagram Protocol (UDP)** ==è un protocollo di trasporto di rete== che punta tutto sulla **velocità e sulla leggerezza**, sacrificando l'affidabilità.

A differenza del [[Protocollo TCP|TCP]], l'UDP è un protocollo **non orientato alla connessione** (*connectionless*): invia i dati direttamente al destinatario senza verificare se questo sia online o pronto a riceverli, esattamente come una normale cartolina postale.

Viene utilizzata  nei casi in cui la possibilità di perdere un pacchetto non è così rilevante come lo è la velocità (come nello *streaming video*)
![[Pasted image 20260521165853.png]]
Si trova nel livello  [[Modello ISO - OSI|Trasporto]] del modello ISO-OSI.

L'UDP ==riduce al minimo== le operazioni di gestione per trasmettere i dati il più velocemente possibile:

- **Nessun handshake**: Non esiste alcuna fase di sincronizzazione iniziale tra mittente e destinatario.
- **Invio "a colpo sicuro"**: I dati vengono impacchettati in blocchi chiamati _datagrammi_ e spediti immediatamente.
- **Nessun riscontro (ACK)**: Il mittente non riceve alcuna conferma dell'avvenuta ricezione del messaggio.
- **Nessun controllo dell'ordine**: Se i pacchetti arrivano in disordine, l'UDP non li riorganizza e non richiede il rinvio di quelli persi.
  
  


### Struttura del pacchetto

![[Pasted image 20260525123304.png]]
Il protocollo UDP si appoggia sull' [[Protocollo IP|IP]]