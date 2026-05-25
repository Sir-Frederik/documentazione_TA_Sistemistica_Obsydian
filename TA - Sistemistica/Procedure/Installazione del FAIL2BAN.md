L'aggiunta del [[🚫Fail2Ban]] cambia in base al **SO**.

Prima **verifica** che non sia presente

```bash
sudo systemctl status fail2ban
```
### Installazione su Ubuntu e Debian:
Prima aggiorna tutti i repository:
```bash
sudo apt update
```
E poi installa Fail2Ban:

```bash
sudo apt install fail2ban -y
```

### Installazione su Centos e Almalinux:
Prima aggiorna tutti i repository:
Sostituisci "yum" a "dnf" se la versione del sistema è vecchia.

```bash
sudo dnf update -y  
``` 
```  
sudo yum update -y   #vecchia alternativa 
```
Abilita il repository **EPEL**:
```bash
sudo dnf install epel-release -y
```
```
sudo yum install epel-release -y #vecchia alternativa 
```

Se Centos ti dà problemi perchè non trova le risorse, fai così:
``` bash
# Sostituisci i mirror non più disponibili con il vault
sudo sed -i 's/mirror.centos.org/vault.centos.org/g' /etc/yum.repos.d/CentOS-Base.repo
sudo sed -i 's/^#baseurl/baseurl/g' /etc/yum.repos.d/CentOS-Base.repo
sudo sed -i 's/^mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-Base.repo
sudo yum-config-manager --disable ius  #disabilita la vecchia repository ius
sudo yum clean all
sudo yum makecache
sudo yum install epel-release -y
```


E poi installa Fail2Ban:

```bash
sudo dnf install fail2ban -y
```
```
sudo yum install fail2ban -y  #vecchia alternativa 
```


### Configurazione base
Devi verificare che sia configurato per [[🔐 SSH]] e devi anche poter personalizzare le impostazioni. Quindi creiamo un **Jail** locale copiato da quello del sistema (da non toccare mai!).

**Crea** il file di **configurazione locale:**
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
Se ti da errore perchè la cartella è vuota, crea un file pulito così:
``` bash
sudo nano /etc/fail2ban/jail.local
```

**modificalo** con:
```bash
sudo nano /etc/fail2ban/jail.local
```


Se stai usando Debian, potresti avere problemi con l'ssdh.
Assicurati che ci sia:
```
backend = systemd
```
Assicurati che il tuo file abbia almeno questo:
``` bash
[DEFAULT]
allowipv6 = auto
bantime   = 7d
findtime  = 10m
maxretry  = 3
ignoreip  = 127.0.0.1/8 ::1  79.41.50.45

[sshd]
enabled = true 
backend = systemd
```

puoi anche **cancellarlo** con:

``` bash
sudo rm /etc/fail2ban/jail.local
```

se sotto default metti:
```
 maxretry = 3
```
In questo modo blocchi l'ip dopo 3 tentativi.
Se come bantime metti:
```ini
bantime = 604800  
```
Oppure
```ini
bantime = 7d 
```
L'ip resta bloccato per 7 giorni.
Se metti "-1"  diventa perpetuo.

Puoi anche mettere come sicuro **l'ip aziendale** in **[DEFAULT]** con:
```
ignoreip = 127.0.0.1/8 ::1 79.41.50.45
```
```
 79.41.50.45
```
Assicurati che **non** ci siano i **#** o il codice viene commentato.

Controlla che **[sshd]** abbia questo:
```
enabled = true 
```

## Avvio 
``` bash
sudo systemctl start fail2ban
```
``` bash
sudo systemctl enable fail2ban
```
# Verifica
Controlla lo stato:
```bash
sudo systemctl status fail2ban
```
Controlla i ban attivi:
```bash
sudo fail2ban-client status sshd
```

Controlla **tutti i dati** di un fail2ban:

``` bash
sudo fail2ban-client -d
```

**Modifica** il file jail con:

```bash
sudo nano /etc/fail2ban/jail.local
``` 
Se vuoi cercare un valore specifico nel documento, puoi usare un **grep**:
``` bash
sudo grep -n "bantime" /etc/fail2ban/jail.local

```
	oppure trova il numero di riga con
``` bash
grep -n "^\[sshd\]" /etc/fail2ban/jail.local
```
Se vuoi andare alla riga scrivi:
```bash
sudo nano +274 /etc/fail2ban/jail.local

```

Se vuoi cercare una sezione specifica, seleziona l'intervallo con:
``` bash
sudo sed -n '95,105p' /etc/fail2ban/jail.local
```

Se fai delle modifiche devi **riavviare** il Fail2ban così:
``` bash
sudo systemctl restart fail2ban
```
# Cancellazione
Per cancellare tutto metti:
``` bash
# Rimozione completa incluse le config
sudo apt purge fail2ban
sudo apt install fail2ban
```

Per reinstallarlo lasciando i file di configurazione metti:

``` bash
sudo apt reinstall fail2ban
```
Torna [[🚫Fail2Ban#INSTALLAZIONE|su]] 