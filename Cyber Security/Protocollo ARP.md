---
tags:
  - Protocollo
---
L'**Address Resolution Protocol** ==è un protocollo fondamentale di rete utilizzato per mappare un indirizzo IP logico su un indirizzo MAC fisico==. Permette ai dispositivi in una rete locale (LAN) di comunicare traducendo gli indirizzi IP a livello di rete in indirizzi MAC a livello di collegamento dati.
![[Pasted image 20260521165853.png]]
Si trova tra il livello [[Modello ISO - OSI|Data]] e il livello [[Modello ISO - OSI|Rete]]


![[Screenshot 2026-04-02 170131.png]]


Nell' esempio mostrato c'è la prima delle tre macchine collegate ad una rete locale (ognuna col proprio ip e indirizzo Mac) che fa una richiesta ARP. Questa richiesta ARP chiede **chi sia** quello con l'IP *158.108.2.4*, e per farlo manda una richiesta **Broadcast** a tutti.
Solo il computer con l'IP giusto accetterà il pacchetto, manda una **ARP Reply** inviandogli il proprio indirizzo MAC.
Da questo momento i due pc ==possono comunicare direttamente.==

Usando il comando Dos in modalità admin ```
```
netsh interface ip delete arpcache
```
 si andrà a pulire la cache dell'ARP.
DIgitando:
```
 arp -a
```
è possibile vedere tutta la memoria ARP.

In questo modo noi possiamo vedere esattamente le connessioni che vedremmo andando su *"Pannello di controllo -> Connessioni di rete"*


Per iniziare la comunicazione, mettiti in ascolto con [[Wireshark 🦈]]

digita il comando (inserendo il vero ip)
```
ping  *IPV4*
```
Su wireshark,  seleziona quell'ip (dalle impostazioni ⚙️) filtra per "arp" e vedrai le richieste broadcast.
ex:
![[Pasted image 20260525114307.png]]
