E' un **WebServer** ad alte prestazioni che può svolgere anche il ruolo di **reverse proxy, load balancer e cache**.

Di fatto fornisce contenuto statico (HTML, CSS, immagini, video...).
A differenza di [[🌐 Apache HTTP Server]] usa il modello **event-driven** :

- Un **singolo processo** gestisce migliaia di richieste contemporaneamente
- Non crea nuovi processi, gestisce tutto tramite **eventi asincroni**
- Consuma **molta meno memoria**
```
Apache:
1000 richieste → 1000 processi/thread

Nginx:
1000 richieste → 1 processo, gestione asincrona
```

Può lavorare anche come [[🚦HAProxy#Differenza con 🌐Nginx| Load - Balancer]].

Nginx viene spesso usato insieme a  [[🦊Ecosistema Git#GitLab Workhorse|GitLab Workhorse]]: Nginx gestisce le connessioni in ingresso e SSL, poi passa tutto a Workhorse.

```
Client → Nginx → GitLab Workhorse → Puma/Rails
```
## Reverse Proxy
E' un proxy che si posiziona **davanti ai Server Applicativi** (tipo Node.js) per motivi di **sicurezza** e/o **praticità.**
Infatti in questo modo il server applicativo non è esposto direttamente.  Il reverse proxy può anche gestire l'[[HTTPS e HTTP | HTTPS]] ed inviare l'HTTP al backend (*[[Certificati SSL E TLS | SSL]]Termination*), può mettere in Cache le risposte del backend, oppure comprimerle prima di inviarle al client.