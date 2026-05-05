Si fa tramite [[Putty]]. SI prendono gli ip pubblici da [[OVH Cloud]].  Nell'Host name si mette
*nomeutente@indirizzoip*.
Putty si può connettere al sever grazie al protocollo **SSH** che cripta i dati e le password.
Nel caso dei sistemi ubuntu il nome utente è "ubuntu", in quelli "Centos" è "centos" e in quelli "Debian  " è "debian".

In "Connection", nel menù laterale metti "**60 secondi**" come timer.
In **SSH - Auth - Credential** devi uplodare la **chiave  SSH** fornita specifica per quel server.
Se salvi la configurazione, non dovereai più rifarlo.

Se ti connetti al server, si apre il terminale e lì devi inserire la password del server.
Quando vuoi uscire dalla sessione, scrivi "**Exit**".
Per conoscere l'ip del server digita
```
ip a
```

Adesso puoi anche vedere i [[Vedere i  Servizi del server|servizi]], i [[Crontab]] e le regole dei [[Firewall]]