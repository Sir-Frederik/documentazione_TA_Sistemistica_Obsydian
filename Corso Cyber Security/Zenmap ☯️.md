---
tags:
  - Strumento
---
E' l'interfaccia grafica del tool **Nmap**. Costui è un tool di ==scansione della rete== e permette di capire quali servizi sono in ascolto su una certa macchina.

Si può anche effettuare la **[[Protocollo TCP|TCP]] Connect Scan**. E' una scansione completa, in cui viene completato il **3-way handshake** e s instaura la comunicazione.

==**Zenmap** è uno **strumento  attivo** (esplora e interroga), mentre [[Wireshark 🦈]] è uno **strumento  passivo** (ascolta e analizza).==

Innanzitutto **verifica** che il server sia raggiungibile con il comando `ping`.
Poi, per fare la **Connect Scan**, su Zenmap si digita il comando
```
nmap -sT  *IPV4*
```

Per analizzare una porta specifica invece, si scrive così:

```
nmap -sT -p *PORTA* *IPV4*
```