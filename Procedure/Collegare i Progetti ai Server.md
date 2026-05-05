Un **progetto** è un'applicazione software sviluppata dal team. Può essere:

- Un **applicativo web** (portale, gestionale, ecc.)
- Un **servizio REST** (API che espone dati)
- Una **batch** (processo automatico in background, gestita dai [[Crontab]])

Ogni server può ospitare più progetti, un progetto gira su più server (sviluppo, collaudo, produzione) e può essere collegato a più DB. A loro volta i DB possono essere condivisi tra più progetti. Tutte le relazioni sono quindi **n:n**.

---

### Il ruolo di [[☕WildFly]]

WildFly è il motore che esegue i progetti Java (application server). Quando un progetto viene deployato (messo in esecuzione), il suo file `.WAR` o `.EAR` viene copiato in una cartella specifica di WildFly chiamata `deployments`.


```bash
ls /opt/wildfly-14.0.1.Final/standalone/deployments/
```

> ⚠️ Il percorso può variare in base alla versione installata. Se non esiste, cercalo con:

``` bash
find / -name "deployments" -type d 2>/dev/null
```

I file con estensione `.ear.deployed` o `.war.deployed` confermano che il progetto è **attivo e in esecuzione**.

`standalone.xml` è il file di configurazione principale di WildFly in [[☕WildFly#Standalone Mode |modalità standalone]]. Al suo interno sono dichiarati i **datasource**, ovvero le connessioni ai DB. Per ogni connessione trovi:

- il tipo di DB (MySQL, MongoDB, PostgreSQL)
- l'indirizzo del server DB
- il nome dello schema/database
- l'utente di connessione

---

### Ricerca delle Batch

Una **batch** è un piccolo programma di elaborazione dati che gira in background. È tipicamente composta da un file `.jar` (il processo), uno script `.sh` (che lo lancia) e un file `.properties` (la configurazione).

Prima capisci la struttura della home:


```bash
ls /home/
```

Ti usciranno =tutti gli utenti =collegati al server, tra cui quello con cui sei connesso. Accedi alla home del tuo utente:

```bash
#ex: centos
ls -la /home/centos
```

Esplora le cartelle presenti con `ls -la /home/NOME_UTENTE/NOME_CARTELLA/`.

Come interpretare quello che trovi:

- Cartella con `.jar`, `.sh`, `.properties` → probabilmente una **batch**
- Cartella con solo `.js`, `.css`, `.png`, `.jpg` → è un **frontend statico**, ignorala
- Script `.sh` isolati → potrebbero essere script di manutenzione infrastrutturale, non batch applicative

> ⚠️ Puoi accedere solo alla home dell'utente con cui sei loggato. Le altre daranno `Permission denied` — è normale.

---

### Ricerca di JAR e script ovunque

Per sicurezza, cerca tutti i file `.jar` e `.sh` nella home:

bash

```bash
find /home/centos -name "*.jar" 2>/dev/null
find /home/centos -name "*.sh" 2>/dev/null
```

Se non trovi nulla di rilevante, significa che su questo server non girano batch applicative per quell'utente.

---

### Ricerca dei [[Crontab | Cron Job]]

I cron job sono processi schedulati che si avviano automaticamente a orari prestabiliti. Spesso lanciano le batch trovate al passo precedente.


```bash
crontab -l
cat /etc/crontab
ls /etc/cron.d/
```

---

### Connessioni al DB

Estrai il blocco `<datasources>` da `standalone.xml` per capire a quali database si connettono i progetti:

bash

```bash
sed -n '/<datasources>/,/<\/datasources>/p' $(find / -name "standalone.xml" 2>/dev/null | head -1)
```

Per ogni datasource trovi:

- **jndi-name** — il nome logico con cui WildFly identifica la connessione internamente
- **connection-url** — l'indirizzo del server DB, la porta e il nome dello schema
- **driver** — il tipo di database (com.mysql, h2, postgresql...)
- **user-name** — l'utente di connessione

> ⚠️ Ignora il datasource **ExampleDS** — è creato da WildFly di default, non è usato da nessun progetto reale.

> ⚠️ Il server DB nella `connection-url` può essere una macchina diversa da quella su cui gira WildFly — annotalo come **DB Server** separato nella documentazione.

---

