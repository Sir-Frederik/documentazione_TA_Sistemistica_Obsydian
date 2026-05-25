Il **Transmission Control Protocol (TCP)** ==è un protocollo di rete a pacchetto di livello di trasporto fondamentale per il funzionamento di Internet==. Esso trasforma il servizio di base e "instabile" del protocollo IP in una comunicazione totalmente **affidabile e orientata alla connessione** tra mittente e destinatario.

E' più **lento** rispetto all' [[Protocollo UDP|UDP]], ma è molto più **affidabile**.
![[Pasted image 20260521165853.png]]
Si trova nel livello  [[Modello ISO - OSI|Trasporto]] del modello ISO-OSI.
Questo protocollo prima di comunicare vuole essere ==sicuro che dall'altra parte ci sia qualcuno pronto a ricevere== la comunicazione.
Di conseguenza effettua il **3-Way Handshake**.
### 3-Way Handshake
 La "*Stretta di mano a 3 vie*" avviene prima di scambiare dati. Il client e il server stabiliscono una connessione tramite ==tre passaggi== (invio del pacchetto **SYN**, risposta **SYN-ACK**, e conferma finale **ACK**).

![[Pasted image 20260525122427.png]]

Il client invia un pacchetto SYN, il destinatario risponde col SYN-ACK confermando di essere pronta, allorché il client conferma con una altro ACK e poi parte la comunicazione regolare.

### Struttura del pacchetto

![[Pasted image 20260525122950.png]]

Il protocollo TCP si appoggia sull' [[Protocollo IP|IP]].
Oltre al protocollo IP, abbiamo altre informazioni, come il **Source Port** e il **Destination Port**, e i **TCP FLAGS 🏴**.
Questi permettono di impostare i FLAG aggiuntivi all'interno del pacchetto, e in questa categoria appartengono i  **SYN** e **ACK**.
Per visualizzarli si può usare [[Zenmap ☯️]].

Innanzitutto **verifica** che il server sia raggiungibile con il comando `ping`.
Poi, per fare la **Connect Scan**, su Zenmap si digita il comando
```
nmap -sT  *IPV4*
```
Mettiti in ascolta con [[Wireshark 🦈]] sulla rete desiderata e filtra con "TCP", oppure cn "HTTP" (si basa comunque su TCP).

![[Pasted image 20260525151447.png]]

Cliccando sul pacchetto, è possibile ottenere le i sue informazioni:

![[Pasted image 20260525151733.png]]

