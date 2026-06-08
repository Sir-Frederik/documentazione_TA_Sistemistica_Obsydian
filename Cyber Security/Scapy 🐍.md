---
tags:
  - Strumento
---
E' un ==modulo python== per la ==**manipolazione** dei pacchetti ==di rete.
Opera sia a **livello 2** che a **livello 3** e permette di fare delle operazioni molto complesse.

##  Concetto di Layering
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

### I layer corrispondono ai livelli reali

|Layer Scapy|Livello di rete|
|---|---|
|`Ethernet()`|Livello 2 (Data Link)|
|`IP()`|Livello 3 (Network)|
|`TCP()` / `UDP()` / `ICMP()`|Livello 4 (Transport)|
|`"testo"` / `Raw()`|Payload / Dati|
## Avvio
Per avviarlo, ==fai partire Python==, poi:
```python
from scapy.all import *
```

Da quel momento si aprirà il prompt di Scapy. Per uscire scrivi:
``` python
exit()
```

## Invio del pacchetto ICMP
### Invio di  un messaggio
In Scapy, `ICMP` è una classe che rappresenta il [[Protocollo ICMP]] (lo stesso usato dal comando `ping`).

Non possiamo solo usare il protocollo ICMP, ma anche quello IP ([[Scapy 🐍#Concetto di Layering|Layering]]).
Facciamo un *esempio* in cui inviamo al **localhost** e mettiamo il **payload** "*Ciao Mondo*"

``` python
send(IP(dst="127.0.0.1") / ICMP() / "Ciao Mondo")
```
Ecco come lo vediamo su wireshark se lo sintonizzi su "*loopback*"
![[Pasted image 20260525164502.png]]

#### Specifiche del tipo
Ora modifichiamo il [[Protocollo ICMP#Tipi di messaggio principali|Tipo]] dei messaggi ICMP. Per **default**, come abbiamo fatto prima, c'è il *tipo 8*, "Echo Request".

Inviamo il tipo **3** (*Destination Unreachable*)e mettiamo come ==source== un ==IP che non esiste== ([[Spoofing|IP Spoofing]] ) per vedere come possiamo manipolare il pacchetto (**packet crafting**) falsificando un ip e inviando manualmente un messaggio (tipo 3) che normalmente viene generato in automatico da un router.

``` python
send(IP(src="192.168.1.10", dst="127.0.0.1") / ICMP(type=3) / "Come Stai?")
```
![[Pasted image 20260526120833.png]]
![[Pasted image 20260526120749.png]]

### Ricezione dei messaggi
Useremo 3 metodi messi a disposizione da Scapy, ognuno in grado di gestire [[Modello ISO - OSI|Livelli]] diversi.
- ==**sr()**==
	Permette di ==**Inviare e Ricevere** ==pacchetti di **livello ==3==**
- **==sr1()==**
	Permette di  **==Ricevere==** una risposta relativamente ad un pacchetto di **livello ==3==** inviato.
- ==**srp()**==
	Permette di ==**Inviare e Ricevere** ==pacchetti di **livello ==2==**

%% sr = Send Receive %%
#### sr1()
Vogliamo ottenere un solo pacchetto di risposta.
Definiamolo con  una **variabile**:
``` python
pacchettoICMP = sr1(IP(dst="127.0.0.1")/ICMP())
```
Non c'è bisogno di scrivere il `type` in questo caso.
Mandiamo un **ICMP Echo Request**  (`type 0`) e se la macchina di desitnazione può ricevere questo tipo di pacchetti, ci aspettiamo che ci dia un **ICMP Echo Reply** (`type 8`) che ==cercheremo di catturare== col metodo **sr1()**.
