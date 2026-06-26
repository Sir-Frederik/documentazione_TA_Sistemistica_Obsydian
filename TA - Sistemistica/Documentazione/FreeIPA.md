
> **Cos'è**: Soluzione Open Source  per la ==**gestione centralizzata** di identità, **autenticazione** e policy per server Linux/Unix, con supporto per client Windows e macOS== .
> Il nome sta per **Free Identity, Policy, and Audit**. È il progetto upstream di **Red Hat Identity Management**.

---

### È gratis?

Sì, completamente. È open source, mantenuto da Red Hat e dalla community. Non richiede licenze.

> ℹ️ Red Hat offre una versione commerciale chiamata **Red Hat Identity Management (IdM)**, ma FreeIPA standalone è gratuito al 100%.

---

### Cosa integra nativamente

Invece di configurare e far comunicare tra loro più strumenti separati (come fa [[🏦 SAMBA]] + *OpenLDAP*), FreeIPA include tutto in un unico pacchetto:

| Componente               | Ruolo                                                      |     |
| ------------------------ | ---------------------------------------------------------- | --- |
| **389 Directory Server** | Backend [[LDAP]] per la gestione utenti/gruppi             |     |
| **MIT Kerberos**         | Autenticazione sicura                                      |     |
| **Dogtag CA**            | Certificate Authority interna per gestione certificati TLS |     |
| **BIND DNS**             | Risoluzione nomi integrata                                 |     |
| **SSSD**                 | Consente ai client Linux di autenticarsi contro FreeIPA    |     |
| **Web UI**               | Interfaccia grafica per gestione utenti, gruppi, policy    |     |

---

### Compatibilità con la nostra infrastruttura

Su AlmaLinux il pacchetto FreeIPA è disponibile nei repository standard senza moduli aggiuntivi o subscription, il che lo rende uno degli 
**identity server** più rapidi da installare su una distribuzione RHEL-based. [ComputingForGeeks](https://computingforgeeks.com/install-freeipa-server-on-rocky-almalinux/)

|Sistema|Supporto|
|---|---|
|**AlmaLinux 8/9/10**|✅ Nativo|
|**Rocky Linux**|✅ Nativo|
|**CentOS**|✅ Supportato|
|**Client Windows**|✅ Tramite trust con Active Directory|
|**Client MacBook**|✅ Tramite SSSD o profilo di configurazione|

---

### Requisiti minimi del server

FreeIPA richiede almeno **2 vCPU e 4 GB di RAM**. Con meno di 1.5 GB il processo Dogtag (CA) va in timeout durante l'installazione. È consigliato installarlo su una VM dedicata, senza altri servizi che usino le stesse porte (DNS, LDAP, porta 443). [ComputingForGeeks](https://computingforgeeks.com/install-freeipa-server-on-rocky-almalinux/)

---

### Installazione su AlmaLinux (panoramica)

L'installazione avviene tramite `dnf` e un wizard interattivo:



```bash
# 1. Abilitare il modulo IDM
sudo dnf module enable idm:DL1 -y

# 2. Installare FreeIPA server
sudo dnf install freeipa-server ipa-server-dns bind-dyndb-ldap -y

# 3. Avviare il wizard di configurazione
sudo ipa-server-install
```

Il wizard chiede interattivamente: nome del dominio (es. `azienda.local`), realm Kerberos, password admin, e se integrare il DNS interno.

> ⚠️ Il hostname del server deve essere **fully qualified** (FQDN) e risolvibile via DNS prima di avviare l'installazione.

---

### Aggiungere un client (es. altra VM AlmaLinux)


```bash
# Sul client da aggiungere al dominio FreeIPA
sudo dnf install freeipa-client -y
sudo ipa-client-install --server=ipa.azienda.local --domain=azienda.local
```

Da quel momento il client autentica gli utenti tramite FreeIPA.

---

### Porte di rete necessarie

|Porta|Protocollo|Uso|
|---|---|---|
|`88`|Kerberos|Autenticazione|
|`389`|LDAP|Directory|
|`636`|LDAPS|Directory cifrata|
|`443`|HTTPS|Web UI|
|`464`|Kerberos|Cambio password|
|`53`|DNS|Risoluzione nomi|

---

### Sicurezza

- Autenticazione tramite **Kerberos** (nessuna password in chiaro sulla rete)
- Certificati TLS gestiti dalla **CA interna** con rotazione automatica
- Supporto **2FA** (Two Factor Authentication)
- **HBAC rules** (Host-Based Access Control) — definisci quale utente può accedere a quale server
- Audit degli accessi integrato
- Password policy centralizzata (lunghezza, scadenza, complessità)

---

### Web UI

FreeIPA include una Web UI accessibile via browser su `https://<ip-server>/ipa/ui` per gestire utenti, gruppi, host e policy senza usare la CLI.