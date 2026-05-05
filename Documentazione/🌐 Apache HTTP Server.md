E' un **web server** molto utilizzato. Riceve le richieste degli utenti e risponde inviando le pagine web dal server. 
E' **open source** e **gratuito** e supporta php, https e altro.
Apache spessopuò funzionare  come un **Reverse Proxy** cioè prende le richieste dall'esterno e le passa ad altri servizi (ex. [[☕WildFly]]).
 Funziona anche da **Load Balancer***, cioè se ci sono più server applicativi identici, Apache distribuisce il traffico tra di loro.
Anche [[🚦HAProxy]] fa il lavoro da Reverse Proxy  e da Load Balancer ed è più specializzato nel farlo.


Usa il modello **process - based**:
- Crea un **nuovo processo o thread** per ogni richiesta
- Con molte richieste simultanee consuma **molta RAM**
  
```
Apache:
1000 richieste → 1000 processi/thread

Nginx:
1000 richieste → 1 processo, gestione asincrona
```