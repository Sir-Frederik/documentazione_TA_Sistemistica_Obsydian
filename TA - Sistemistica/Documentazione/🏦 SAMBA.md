

> **Cos'è**: Implementazione open source del protocollo [[SMB|SMB/CIFS]] che permette l'interoperabilità tra sistemi [[Linux]]/[[Unix]] e [[Windows]] per la condivisione di file, stampanti e autenticazione di rete.

---

## Info

### Versioni

| Versione           | Anno          | Caratteristiche                                         |
| ------------------ | ------------- | ------------------------------------------------------- |
| **Samba 3**        | Pre-2012      | File sharing + richiedeva [[OpenLDAP]] separato per AD  |
| **Samba 4 (AD DC)**| 2012 → oggi   | LDAP + Kerberos + AD integrati in un unico prodotto     |

> ⚠️ Lo stack **Samba 3 + OpenLDAP** è considerato **obsoleto**. La versione moderna è **Samba AD DC** (Samba 4).

---

### Funzioni principali

- **File Server** — condivisione cartelle in rete locale tramite protocollo ==[[SMB]]/CIFS==
- **Active Directory Domain Controller** — gestione centralizzata di utenti, gruppi e autenticazione

---

### Protocollo [[SMB]] / CIFS

| Termine      | Significato                                                     |
| ------------ | --------------------------------------------------------------- |
| **SMB**      | Server Message Block – protocollo di comunicazione Microsoft    |
| **CIFS**     | Common Internet File System – versione pubblica/standard di SMB |
| **SMBv1**    | Versione obsoleta e **insicura** – va disabilitata sempre       |
| **SMBv2/v3** | Versioni moderne e sicure                                       |

> ⚠️ **SMBv1 è deprecato** e responsabile di vulnerabilità critiche storiche (es. EternalBlue/WannaCry). Non usarlo mai.

---

### Porte di rete

| Porta         | Protocollo      | Uso                      |
| ------------- | --------------- | ------------------------ |
| `53/TCP-UDP`  | DNS             | Risoluzione nomi AD      |
| `88/TCP-UDP`  | Kerberos        | Autenticazione           |
| `135/TCP`     | RPC Endpoint    | DCOM/RPC                 |
| `139/TCP`     | NetBIOS Session | SMB su NetBIOS (legacy)  |
| `389/TCP`     | LDAP            | Interrogazioni directory |
| `445/TCP`     | SMB diretto     | SMB moderno (principale) |
| `464/TCP-UDP` | Kerberos        | Cambio password Kerberos |
| `636/TCP`     | LDAPS           | LDAP over TLS            |
| `3268/TCP`    | Global Catalog  | Ricerche forest-wide     |

> 🔒 Queste porte **non devono essere esposte su Internet**. Per accesso remoto usare [[🚇WireGuard]] o equivalente.

---

### Samba AD DC vs [[FreeIPA]]

| Caratteristica           | Samba AD DC            | [[FreeIPA]]                          |
| ------------------------ | ---------------------- | ------------------------------------ |
| **Ambiente ideale**      | Misto Linux + Windows  | Prevalentemente Linux                |
| **Client Windows**       | Nativo, perfetto       | Possibile, con configurazione extra  |
| **Client Linux**         | Funziona               | Nativo, perfetto                     |
| **Interfaccia gestione** | Solo CLI               | Web UI + CLI                         |
| **Kerberos**             | Integrato              | Integrato                            |
| **Certificati TLS / CA** | Da gestire manualmente | CA interna integrata                 |
| **Protocollo SMB**       | Sì, nativo             | No                                   |
| **Gratuito**             | Sì                     | Sì                                   |

---

### Nota sulla separazione dei ruoli

Samba AD DC è in grado di fornire anche ==share di file==, ma il team di Samba **non raccomanda** di usare un DC anche come file server. I motivi principali sono:

- DC e file server hanno cicli di aggiornamento diversi
- Si introducono rischi di stabilità
- Limitazioni tecniche sulle [[ACL|Access Control List]]

> ℹ️ Se si usa Samba, si raccomandano **almeno due VM distinte**: una per il DC e una per il file sharing.

---

## Installazione Samba AD DC su Ubuntu

### Prerequisiti

- VM **dedicata** con Ubuntu (nessun altro servizio installato)
- IP statico configurato
- Hostname **FQDN** impostato correttamente **prima** dell'installazione (es. `dc1.azienda.lan`)
- Hostname ≤ 15 caratteri (limite NetBIOS)

---

### 1. Installazione pacchetti

```bash
sudo apt install samba-ad-dc krb5-user dnsutils
```

> ℹ️ Durante l'installazione di `krb5-user` è possibile accettare i valori di default premendo INVIO.

---

### 2. Disabilitare Samba standalone

I servizi classici Samba (`smbd`, `nmbd`, `winbind`) non devono essere attivi in modalità AD DC.

```bash
sudo systemctl disable --now smbd nmbd winbind
sudo systemctl mask smbd nmbd winbind
```

Abilitare il servizio AD DC:

```bash
sudo systemctl unmask samba-ad-dc
sudo systemctl enable samba-ad-dc
```

---

### 3. Backup configurazione esistente

Il provisioning genera automaticamente `smb.conf`, quindi eventuali file esistenti vanno rimossi o spostati:

```bash
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.orig
```

---

### 4. Provisioning Domain Controller

```bash
sudo samba-tool domain provision \
    --domain AZIENDA \
    --realm=AZIENDA.LAN \
    --server-role=dc \
    --dns-backend=SAMBA_INTERNAL \
    --use-rfc2307 \
    --adminpass='PasswordSicura!'
```

> ⚠️ Se non viene specificata la password admin, ne verrà generata una casuale.

Output atteso:
````

Server Role: active directory domain controller Hostname: dc1 DNS Domain: azienda.lan Domain SID: S-1-5-21-XXXX

````

---

## 5. Configurazione post-installazione

### 5.1 DNS forwarder (FONDAMENTALE)

Samba usa il DNS interno per AD, ma necessita di un forwarder per risolvere internet.

Nel file `/etc/samba/smb.conf`:

```ini
[global]
    dns forwarder = 1.1.1.1 8.8.8.8
```

> ℹ️ È possibile specificare più forwarder separati da spazio. Samba li usa in ordine: se il primo non risponde, prova il successivo.
> ⚠️ Verificare sempre che i forwarder impostati siano raggiungibili dalla VM — un forwarder irraggiungibile causa timeout su ogni query DNS.

---

### 5.2 Configurazione resolv.conf

Per forzare l'uso del DNS Samba (il DC deve usare se stesso come resolver):

```bash
sudo unlink /etc/resolv.conf
sudo bash -c 'cat > /etc/resolv.conf << EOF
nameserver 127.0.0.1
search azienda.lan
EOF'
```

> ⚠️ Su Ubuntu moderno con `systemd-resolved`, questo file può essere rigenerato al riavvio. Per rendere la modifica persistente, disabilitare systemd-resolved:
> ```bash
> sudo systemctl disable --now systemd-resolved
> ```

---

### 5.3 Kerberos config

Samba genera automaticamente la configurazione Kerberos:

```bash
sudo cp -f /var/lib/samba/private/krb5.conf /etc/krb5.conf
```

---

## 6. Avvio e gestione servizio

```bash
# Avvio
sudo systemctl start samba-ad-dc

# Riavvio
sudo systemctl restart samba-ad-dc

# Stato
sudo systemctl status samba-ad-dc

# Log in tempo reale
sudo journalctl -u samba-ad-dc -f
```

---

## 7. Firewall (UFW)

```bash
sudo ufw allow 53/tcp
sudo ufw allow 53/udp
sudo ufw allow 88/tcp
sudo ufw allow 88/udp
sudo ufw allow 135/tcp
sudo ufw allow 389/tcp
sudo ufw allow 389/udp
sudo ufw allow 445/tcp
sudo ufw allow 464/tcp
sudo ufw allow 464/udp
sudo ufw allow 636/tcp
sudo ufw allow 3268/tcp
sudo ufw reload
```

---

## 8. Verifica installazione

### 8.1 Kerberos test

```bash
kinit Administrator
klist
```

Se funziona → ticket valido con scadenza visibile.

---

### 8.2 DNS test (SRV records AD)

```bash
hostname
hostname -f

for srv in _ldap._tcp _kerberos._tcp _kerberos._udp _kpasswd._udp; do
    dig @127.0.0.1 +short -t SRV ${srv}.azienda.lan
done
```

Query completa zona:

```bash
sudo samba-tool dns query 127.0.0.1 azienda.lan @ ALL -U administrator
```

---

### 8.3 Test livello dominio

```bash
sudo samba-tool domain level show
```

---

### 8.4 Verifica configurazione

```bash
samba --version
testparm
```

---

## 9. Gestione DNS record

Utile per correggere IP errati nel DNS di Samba (es. dopo cambio IP o configurazione VPN).

```bash
# Visualizza tutti i record di una zona
sudo samba-tool dns query 127.0.0.1 azienda.lan @ ALL -U administrator

# Aggiunge un record A
sudo samba-tool dns add 127.0.0.1 azienda.lan <hostname> A <IP> -U administrator

# Rimuove un record A
sudo samba-tool dns delete 127.0.0.1 azienda.lan <hostname> A <IP> -U administrator

# Verifica record specifico
sudo samba-tool dns query 127.0.0.1 azienda.lan <hostname> ALL -U administrator
```

> ⚠️ Se il DC ha più record A (es. IP privato, IP pubblico e IP VPN), i client Windows potrebbero tentare di connettersi a IP errati. Tenere **un solo record A** per ogni hostname, corrispondente all'IP raggiungibile dai client.

---

## 10. Gestione utenti

```bash
# Crea utente
sudo samba-tool user add nomeutente

# Lista utenti
sudo samba-tool user list

#vedi info utente
sudo samba-tool user show nomeutente

# Disabilita utente
sudo samba-tool user disable nomeutente

# Abilita utente
sudo samba-tool user enable nomeutente

# Reset password
sudo samba-tool user setpassword nomeutente
```

---

## 11. Password policy

```bash
# Visualizza policy attuale
sudo samba-tool domain passwordsettings show

# Modifica lunghezza minima password
sudo samba-tool domain passwordsettings set --min-pwd-length=12

# Disabilita scadenza password (lab/test)
sudo samba-tool domain passwordsettings set --max-pwd-age=0
```

---

## 12. GPO (Group Policy)

```bash
# Lista tutte le GPO
sudo samba-tool gpo listall

# Crea nuova GPO
sudo samba-tool gpo create "NomePolicy" -U Administrator

# Collega GPO a una OU
sudo samba-tool gpo setlink "OU=Users,DC=azienda,DC=lan" <GUID_GPO> -U Administrator

# Visualizza GPO collegate a una OU
sudo samba-tool gpo listcontainers <GUID_GPO> -U Administrator
```

> ⚠️ Le GPO si applicano normalmente a OU specifiche, non direttamente al dominio root.
> ℹ️ La configurazione dettagliata delle GPO avviene da Windows con **Group Policy Management Console (GPMC)**.

---

## 13. Join client Windows al dominio

### Prerequisiti lato client

-  DNS impostato **esclusivamente** all'IP del DC (es. `10.0.0.1`)
-  Ora sincronizzata (max 5 minuti di scarto — requisito Kerberos)
-  Connettività al DC verificata (`ping`, `Test-NetConnection`)
-  SRV record visibili (`nslookup -type=SRV _ldap._tcp.azienda.lan <IP_DC>`)

> ℹ️ Se il DC è raggiungibile solo tramite VPN, configurare prima il tunnel [[🚇WireGuard]] e impostare il DNS della scheda VPN all'IP del DC.

### Verifica pre-join (da PowerShell)

```powershell
# Verifica DNS
nslookup -type=A AZIENDA.LAN <IP_DC>

# Verifica raggiungibilità porte critiche
Test-NetConnection -ComputerName <IP_DC> -Port 389
Test-NetConnection -ComputerName <IP_DC> -Port 88
Test-NetConnection -ComputerName <IP_DC> -Port 445

# Verifica che il DC sia individuato
nltest /dsgetdc:AZIENDA.LAN
```

### Esecuzione join

```powershell
Add-Computer -DomainName AZIENDA.LAN -Credential AZIENDA\Administrator -Restart
```

Oppure via GUI: **Impostazioni → Sistema → Informazioni → Rinomina PC (avanzate) → Cambia → Dominio**.

### Verifica post-join

```powershell
# Deve restituire il DC con IP corretto
nltest /dsgetdc:AZIENDA.LAN
```

---

## 14. Troubleshooting

| Sintomo | Causa probabile | Fix |
|---|---|---|
| `apt install` fallisce con DNS error | DNS forwarder irraggiungibile | Verificare con `dig @<forwarder> google.com` |
| `nltest` → `ERROR_NO_SUCH_DOMAIN` | Client usa DNS sbagliato | Controllare `ipconfig /all`, impostare DNS = IP DC |
| DC risponde con IP errato | Record A multipli in Samba DNS | `samba-tool dns delete` per rimuovere IP vecchi |
| Join fallisce con errore credenziali | Ora non sincronizzata o DNS errato | `w32tm /query /status`, verificare DNS |
| `kinit` fallisce | `/etc/krb5.conf` mancante o errato | `sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf` |

---

## Comandi rapidi — Cheatsheet

```bash
samba --version                              # Versione Samba
testparm                                     # Verifica smb.conf
sudo systemctl status samba-ad-dc            # Stato servizio
sudo journalctl -u samba-ad-dc -f            # Log live
sudo samba-tool domain level show            # Livello funzionale dominio
samba-tool dns serverinfo 127.0.0.1          # Info DNS server
sudo samba-tool user list                    # Lista utenti dominio
sudo samba-tool gpo listall                  # Lista GPO
```

---

##### Fonti

- Samba Wiki ufficiale — [Setting up Samba as an Active Directory Domain Controller](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller)
- Documentazione ufficiale Ubuntu — [Install and configure Samba](https://ubuntu.com/tutorials/install-and-configure-samba#2-installing-samba)
