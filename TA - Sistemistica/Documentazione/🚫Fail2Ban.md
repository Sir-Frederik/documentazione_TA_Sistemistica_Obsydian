E' uno strumento di sicurezza che monitora i log di sistema e blocca in automatico gli IP che mostrano comportamenti sospetti. Il servizio è **Open Source**.

Analizza i log del sistema alla ricerca di **troppi tentativi di accesso falliti** e blocca temporaneamente l'IP responsabile tramite le regole del **firewall**.

Il Fail2Ban può proteggere molti servizi tra cui L'[[🔐 SSH]] dai [[bruteforce]], [[🌐 Apache HTTP Server|Apache]]/ [[Ngix]]  dagli attacchi ai webserver, l' [[🚢FTP]] dagli accessi non autorizzati, il mail server dallo spam e **qualsiasi servizio con un log**

```
Log del servizio → Fail2Ban analizza
        ↓
Troppi tentativi rilevati
        ↓
Fail2Ban comunica con il Firewall
        ↓
🚫 IP bloccato
```


### Jail
È la **configurazione per ogni servizio** da proteggere. Definisce:

- Quale **log** monitorare
- Quanti **tentativi** sono ammessi
- Per quanto tempo **bloccare** l'IP
  
### Filter
È la **regola** che definisce cosa cercare nei log, tramite espressioni regolari.

### Action
È l'**azione da eseguire** quando si supera la soglia, solitamente il blocco tramite firewall.

## INSTALLAZIONE
Per installare il fail2ban, vai [[Installazione del FAIL2BAN|qui]].