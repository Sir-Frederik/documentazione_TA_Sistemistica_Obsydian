Un **progetto** è un'applicazione software sviluppata dal team. Può essere:

- Un **applicativo web** (portale, gestionale, ecc.)
- Un **servizio REST** (API che espone dati)
- Una **batch** (processo automatico in background, gestita dai [[Crontab]])

Ogni server può ospitare più progetti, un progetto gira su più server (sviluppo, collaudo, produzione) e può essere collegato a più DB. A loro volta i DB possono essere condivisi tra più progetti. Tutte le relazioni sono quindi **n:n**.

---
### Passo 0 — Panoramica rapida
Prima di qualsiasi altra analisi, esegui:

```bash
ls /opt/
```

`/opt/` è la directory standard di Linux dove vengono installati i software opzionali di terze parti. Ti dà immediatamente un'idea del tipo di server e di cosa gira sopra.

##### Come interpretare quello che trovi:

- `wildfly-*` → server applicativo Java → procedi con la ricerca dei deployments
- `containerd` / `docker` → server container
- `trivy` → scanner vulnerabilità installato
- `jboss` → predecessore di WildFly, probabilmente un link simbolico
- `graphhopper` → libreria di routing geografico, non è un progetto del team
- **Cartelle** con nomi applicativi (es. `ExportDati`, `MailSender`, `SitCloudBatchSystem`) → **batch o progetti installati direttamente in `/opt/`** — esplorale e interpreta così:
    -    `.jar` + `.sh` + `.properties` → **Batch**
	- `.war` o `.ear` → **WildFly** se lanciato da WildFly, altrimenti **Batch**
	- **JAR senza `.war`/`.ear`** → potrebbe essere una **batch** o un **servizio standalone**. Distinguili così:
		- In crontab + log datati → **Batch**
		- Systemd unit + porta aperta + processo attivo → **Servizio standalone**
		  
Usa il comando 
``` bash
ps aux | grep java
```
Per capire se ci sono **processi** java attivi oltre a wildfly.
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

`standalone.xml` è il file di configurazione principale di WildFly in [[☕WildFly#Standalone Mode |modalità standalone]]. Al suo interno sono dichiarati i [[Collegare i Progetti ai Server#Connessioni al DB|datasource]], ovvero le connessioni ai DB. Per ogni connessione trovi: datasource

- il tipo di DB (MySQL, MongoDB, PostgreSQL)
- l'indirizzo del server DB
- il nome dello schema/database
- l'utente di connessione

### Il ruolo di [[🕴️Jenkins]]
Jenkins è il server di **CI/CD** (Continuous Integration / Continuous Deployment) — si occupa di prendere il codice sorgente dai repository Git, compilarlo e deployarlo sui server. Non esegue i progetti direttamente, ma li **builda e distribuisce**.

Se un server ospita Jenkins, nella **home** dell'utente troverai **cartelle Git clonate** — una per ogni progetto o microservizio gestito dalla pipeline. Non sono progetti in esecuzione, ma sorgenti in fase di build.
Jenkins ==non si connette ai database== dei progetti, li **builda** soltanto.
Jenkins ==gestisce tutti gli ambient==i:  build per sviluppo, collaudo e produzione.

##### Come riconoscere un server Jenkins:

- Nel `ps aux` trovi un processo tipo: `java -jar /usr/share/java/jenkins.war --httpPort=8080`
```
ps aux | grep -v grep | grep -v bash | grep -v sshd
```
- Nella **home** trovi decine di cartelle con nomi tipo `nomeprogetto-microservizio`
- In `/opt/` trovi [[🫙Containerd]] o [[🫙 Docker]] ma non wildfly.
- Le connessioni attive (`ss -tnp`) mostrano traffico sulla porta `8080` **in locale** (`127.0.0.1:8080`) — Apache riceve le richieste esterne sulla porta `80` e le inoltra internamente a Jenkins
```
 ss -tnp | grep ESTAB
```
  
##### Come interpretare le cartelle nella home:

- Le cartelle sono raggruppate per **progetto** tramite prefisso: `protciv-*`, `agata-*`, `gaia-*`
- Il **nome del progetto** è il prefisso comune (es. `protciv`)
- Il **nome del microservizio** è il suffisso (es. `validazione-dati`)
- Cartella `trivy_report` → contiene i report di scansione vulnerabilità Trivy — ignorala
- Cartella `.jenkins` → configurazione locale Jenkins — ignorala
- Cartella `.kube` → configurazione Kubernetes — ignorala
- File `jenkins_backup_*.tar.gz` → backup di Jenkins — ignoralo
---
### Il Ruolo di [[🦊Ecosistema Git| Gitlab]]

GitLab è la piattaforma **SCM** (Source Code Management) del team — ospita i repository Git del codice sorgente. Come [[🕴️Jenkins]], **non esegue i progetti direttamente**: li archivia e li versiona.

GitLab ==non si connette ai database== dei progetti. Nella documentazione Excel va trattato esattamente come Jenkins: **una riga per progetto/repository**, con `Ruolo Server = GitLab` e colonne DB sempre `N/A`.

###### Come riconoscere un server GitLab:

- In `/opt/` trovi solo la cartella `gitlab` — installazione **Omnibus** (tutto incluso)
- Non c'è WildFly, non ci sono batch applicative del team
- Il comando `sudo gitlab-ctl status` è disponibile a livello di sistema

###### Come documentarlo:

**1. Recupera il nome host del server:**



```bash
hostname
```

**2. Lista tutti i repository ospitati:**


```bash
sudo gitlab-rails runner "Project.all.each { |p| puts p.full_path }" 2>/dev/null
```

Se il runner non è disponibile:


```bash
sudo ls /var/opt/gitlab/git-data/repositories/
```


| Progetto | Ambiente | Server           | Ruolo Server | Nome DB | Tipo DB | IPv4 DB Server | Note                       |
| -------- | -------- | ---------------- | ------------ | ------- | ------- | -------------- | -------------------------- |
| protciv  | Tutti    | TA_Server_GitLab | GitLab       | N/A     | N/A     | N/A            | Multipli microservizi: ... |

> ⚠️ GitLab usa internamente PostgreSQL e Redis, ma sono componenti propri del prodotto — **non vanno documentati come DB applicativi**.

---


### Ricerca delle Batch

Una **batch** è un piccolo programma di elaborazione dati che gira in background. È tipicamente composta da un file `.jar` (il processo), uno script `.sh` (che lo lancia) e un file `.properties` (la configurazione).

Prima capisci la struttura della home:


```bash
ls -la /home/
```

Ti usciranno ==tutti gli utenti== collegati al server, tra cui quello con cui sei connesso. Accedi alla home del tuo utente:

```bash
#ex: centos
ls -la /home/centos
```

Esplora le cartelle presenti con `ls -la /home/NOME_UTENTE/NOME_CARTELLA/`.

##### Come interpretare quello che trovi:

- Cartella con `.jar`, `.sh`, `.properties` → probabilmente una **batch applicativa**
- Cartella con  `.sh` ma non `.jar`: molto probabilmente richiama un file .jar nello script, esaminalo.
- Cartella con solo `.js`, `.css`, `.png`, `.jpg`, `.woff`, `.woff2`, `.svg` → è un **frontend statico**, ignorala
- Cartella chiamata `back_...` → è un **backup del frontend**, ignorala
- Script `.sh` isolati → potrebbero essere **script di manutenzione infrastrutturale**, non batch applicative
- Cartella `certificati` o `CheckCertificateScript` → contiene [[Certificati SSL E TLS]] o script per gestirli, non sono batch applicative
- Cartella con nome di uno strumento noto (es. `nagios-plugins-...`, `nrpe-...`) → sono **sorgenti o pacchetti di monitoring** scaricati e compilati manualmente, ignorali
- File `.tar.gz` → archivio compresso, probabilmente il sorgente di qualcosa già installato, ignoralo
- File `.bash_history` → file nascosto (inizia con il punto) che registra automaticamente tutti i comandi digitati dall'utente nel terminale. Utile per trovare percorsi o script non standard. Consultalo con `cat ~/.bash_history`
- Cartella `.ssh` → contiene le chiavi di accesso SSH dell'utente, non è rilevante per i progetti
- Cartella `.pki` → certificati personali dell'utente, ignorala
- File `.sql`backup del database.


> ⚠️ Puoi accedere solo alla home dell'utente con cui sei loggato. Le altre daranno `Permission denied` — è normale.

---

### Ricerca di JAR e script ovunque

Per sicurezza, cerca tutti i file `.jar`(applicativi java) e `.sh` (shell script) nella home:


```bash
find /home/almalinux -name "*.jar" 2>/dev/null
```
``` bash
find /home/ubuntu -name "*.sh" 2>/dev/null
```

Gli script `.sh` che usano strumenti di sistema o Oracle (epmautomate, vpn, certificati) **sono batch valide** se implementano logica applicativa custom del team. Vanno ignorati solo gli script di pura manutenzione infrastrutturale.
[[Collegare i Progetti ai Server#Ricerca delle Batch|Come interpretare quello che trovi]]
Se non trovi nulla di rilevante, significa che su questo server non girano batch applicative per quell'utente.


---

### Ricerca dei [[Crontab ]]

I cron job sono processi schedulati che si avviano automaticamente a orari prestabiliti. Spesso lanciano le batch trovate al passo precedente.


```bash
crontab -l
cat /etc/crontab
ls /etc/cron.d/
```

---

### Connessioni al DB
#### Con Wildfly
Se ci sono ==batch o progetti di Wildfly== Estrai il blocco ==`<datasources>`== da `standalone.xml` per capire a quali database si connettono i progetti:

```bash
sed -n '/<datasources>/,/<\/datasources>/p' $(find / -name "standalone.xml" 2>/dev/null | head -1)
```

Per ogni datasource trovi:

- **jndi-name** — il nome logico con cui WildFly identifica la connessione internamente
- Il **nome** dello schema, o database, si trova qui: `jdbc:TIPO://IP:PORTA/NOME_DB:?...`
- **connection-url** — l'indirizzo del server DB, la porta e il nome dello schema
- **driver** — il tipo di database (com.mysql, h2, postgresql...)
- **user-name** — l'utente di connessione

> ⚠️ Ignora il datasource **ExampleDS** — è creato da WildFly di default, non è usato da nessun progetto reale.

> ⚠️ Il server DB nella `connection-url` può essere una macchina diversa da quella su cui gira WildFly — annotalo come **DB Server** separato nella documentazione.

#### ==Properties== delle batch indipendenti
Per controllare le connessioni ai DB delle ==batch non Wildfly==, usa il comando:

``` bash
find /opt /home/centos -name "*.properties" 2>/dev/null | grep -v "_lib" |
grep -vi "wildfly\|fonts"
```

**Evita** quelli che si trovano dentro `/opt/wildfly-1` perché sono appunto di Wildfly.
Puoi leggerli uno alla volta oppure   **concatenarli**  usando `cat` oppure `grep -H ""` che ti fornisce sempre il percorso del file.

 *Esempio*:
 ``` bash
 grep -H "" /opt/NOME_CARTELLA/mysql.properties \     /opt/NOME_CARTELLA/mongodb.properties \ 
 /opt/NOME_CARTELLA/config.properties
 ```
 
 Oppure ==direttamente:
 
```bash
find /opt /home/centos -name "*.properties" 2>/dev/null \
| grep -v "_lib" \
| grep -vi "wildfly\|fonts" \
| while read f; do
    matches=$(grep -niE "jdbc:[a-z]+:|mongodb://|mongo\.uri" "$f")
    
    if [ -n "$matches" ]; then
        echo -e "\n===================="
        echo -e "FILE: $f"
        echo -e "======================"
        echo "$matches"
    fi
done
```

 Per scovare i DB, vale  la regola:
 ```
 jdbc:TIPO://IP:PORTA/NOME_DB:
 ```
Oppure **variazioni** come nel caso di `# MongoDB Config`
```
mongo.uri=mongodb://UTENTE:PASSWORD@IP:Porta/NOMEDB?
```

---

[[Collegare i Progetti ai Server#Passo 0 — Panoramica rapida| TORNA SU]]