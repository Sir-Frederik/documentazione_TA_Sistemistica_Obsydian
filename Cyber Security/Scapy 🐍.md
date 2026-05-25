E' un ==modulo python== per la ==**manipolazione** dei pacchetti ==di rete.
Opera sia a **livello 2** che a **livello 3** e permette di fare delle operazioni molto complesse.

###  Concetto di Layering
Scapy si basa sul concetto di **Layering**, cioè di ==fusione tra livelli del [[Modello ISO - OSI]]==
puoi costruire un pacchetto di rete **strato per strato**, esattamente come funziona nella realtà il modello OSI/TCP-IP

In Scapy gli strati si "impilano" con la barra `/`:



```python
pacchetto = Ethernet() / IP() / TCP() / "payload"
```

Ogni `/` aggiunge un layer sopra il precedente.

**Esempi**:

```python
IP() / ICMP()
```

Significa: _"un pacchetto IP che trasporta dentro un messaggio ICMP"_ — esattamente un ping.


```python
IP() / TCP()
```

Un pacchetto IP con dentro TCP — la base di quasi tutto il traffico web.

#### I layer corrispondono ai livelli reali

|Layer Scapy|Livello di rete|
|---|---|
|`Ethernet()`|Livello 2 (Data Link)|
|`IP()`|Livello 3 (Network)|
|`TCP()` / `UDP()` / `ICMP()`|Livello 4 (Transport)|
|`"testo"` / `Raw()`|Payload / Dati|
### Avvio
Per avviarlo, fai partire Python, poi:
```python
from scapy.all import *
```

Da quel momento si aprirà il prompt di Scapy. Per uscire scrivi:
``` python
exit()
```

### Invio di un pacchetto ICMP
In Scapy, `ICMP` è una classe che rappresenta il [[Protocollo ICMP]] (lo stesso usato dal comando `ping`).

Non possiamo solo usare il protocollo ICMP, ma anche quello IP ([[Scapy 🐍#Concetto di Layering|Layering]]).
Facciamo un *esempio* in cui inviamo al **localhost**.

``` python
send(IP(dst="127.0.0.1") / ICMP() / "ciao Mondo")
```
Ecco come lo vediamo su wireshark se lo sintonizzi su "*loopback*"
![[Pasted image 20260525164502.png]]