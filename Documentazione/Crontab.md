I **[[crontab]]** (cron table) sono file di configurazione in sistemi Unix/Linux che pianificano l'esecuzione automatica di comandi o script a intervalli regolari (**Batch**). Sono gestiti dal demone cron e permettono di automatizzare attività ripetitive.

Un **cron job** è invece la singola istruzione o il compito specifico pianificato all'interno di tale file. In sintesi: ==_crontab_ contiene l'agenda, il _cron job_ è l'evento segnato in agenda==

Ogni riga segue il formato: `m h dom mon dow command` (es. `0 23 * * * /path/to/script.sh` esegue lo script ogni giorno alle 23:00).

Per vedere i **[[Crontab]]** ci sono questi comandi:

**Cron dell'utente corrente:**
```
crontab -l
```


**Cron di root**
```
 sudo crontab -l
```
 
**Cron di sistema**
```
cat /etc/crontab
```

**Script schedulati**
```
ls /etc/cron.*
```

I backup possono essere anche in queste cartelle:
```
ls /etc/crontab
```
```
 ls /etc/cron.daily/
```
```
 ls /etc/cron.weekly/
```
  
  
Fatto questo puoi indagare le sulle regole del [[Firewall]]