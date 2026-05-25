### Gitaly
È un **servizio di GitLab** che si occupa di gestire tutte le operazioni sui **repository Git**. È sostanzialmente il componente che fa da **intermediario** tra GitLab e i dati Git salvati sul disco.

### GitLab Exporter
È l'**exporter [[👀 Prometheus]]** specifico per GitLab. Come abbiamo visto per [[🗣️ Redis Exporter]] il suo compito è **tradurre le metriche di GitLab** in un formato che Prometheus può raccogliere.

## GitLab KAS (Kubernetes Agent Server)
È il componente che gestisce la **comunicazione tra GitLab e i cluster Kubernetes**. Permette di collegare GitLab a Kubernetes per automatizzare il deploy delle applicazioni direttamente dai repository.

## GitLab Rails - Puma

**GitLab Rails** è il **cuore dell'applicazione GitLab**, scritto con il framework Ruby on Rails. Gestisce tutta la logica applicativa: utenti, permessi, merge request, issues...

**Puma** è il **web server** che manda in esecuzione GitLab Rails. Gestisce le richieste in arrivo e le smista all'applicazione

## GitLab Workhorse

È un **proxy** che si posiziona davanti a GitLab Rails e gestisce le **operazioni** **pesanti** per non sovraccaricare Puma e Rails. Operazioni tipo: download, upload e operazioni Git.

```
Utente
  ↓
GitLab Workhorse (operazioni pesanti)
  ↓
Puma → GitLab Rails (logica applicativa)
  ↓
Gitaly (operazioni sui repository)
```