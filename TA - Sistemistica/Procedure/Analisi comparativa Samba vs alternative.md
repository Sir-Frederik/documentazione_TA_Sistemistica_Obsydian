#### Cos'è [[🏦 SAMBA]]

Samba è un software open source che permette la ==comunicazione tra ambienti Unix/Linux e ambienti Windows== per la **condivisione** di file, stampanti e **autenticazione** di rete.

In pratica adempie a due compiti principali:

- **Condivisione di file in rete locale** (File Server) tramite il protocollo [[SMB]]/CIFS
- **Autenticazione degli utenti** tramite [[Active Directory]], potendo fungere da _Domain Controller_ completo

---

#### Il contesto della nostra infrastruttura

I server dell'infrastruttura girano su CentOS (in migrazione verso AlmaLinux), mentre i client sono per la quasi totalità laptop Windows e una piccola parte di MacBook. Il documento "_SECSE_" propone un'architettura basata su **Samba + OpenLDAP** per la gestione centralizzata di utenti e autenticazione.

L'obiettivo di questa analisi è valutare se tale architettura sia la scelta più adeguata, o se esistano alternative open source più efficienti e moderne.

---

#### Architettura proposta nel documento interno: Samba + OpenLDAP

Il documento descrive un setup in cui ==[[SAMBA]] gestisce la **condivisione** file e l'**autenticazione** SMB==, mentre ==_OpenLDAP_ funge da **backend** per la gestione centralizzata== di utenti e credenziali. Le componenti di sicurezza previste includono:

- Cifratura SMB con AES-128-GCM (SMB 3.0)
- LDAPS o StartTLS per le connessioni al server LDAP
- _Kerberos_ per l'autenticazione sicura
- Gestione manuale dei certificati TLS e rotazione periodica

Questa architettura è tecnicamente valida, ma presenta una **complessità di configurazione e manutenzione elevata**, in quanto richiede di integrare e mantenere tre componenti separati (_Samba, OpenLDAP, Kerberos_) che devono essere configurati per comunicare correttamente tra loro.

> ⚠️ **Nota**: Lo stack _Samba + OpenLDAP_ descritto nel documento è considerato **obsoleto**. Era lo standard prima di Samba 4 (2012). Oggi esistono soluzioni più moderne e integrate che sostituiscono questo approccio.

---

#### Samba AD DC — Lo stato attuale di Samba

Con il rilascio di **Samba 4**, il progetto ha integrato in un unico prodotto tutto ciò che prima richiedeva Samba + OpenLDAP separati. La versione moderna si chiama **Samba AD DC** (Active Directory Domain Controller) e include nativamente:

- Active Directory completo
- LDAP integrato
- Kerberos integrato
- Supporto nativo ai protocolli SMB e CIFS

Samba AD DC è open source, gratuito, e rappresenta un'alternativa diretta a Windows Server AD in ambienti misti Linux/Windows. [1](#user-content-fn-1)

> ⚠️ **Limitazione critica per la nostra infrastruttura**: Samba AD DC richiede che i client Windows siano in edizione **Pro o superiore** per poter effettuare il _domain join_. I client con **Windows 11 Home** — che costituiscono la maggior parte della nostra flotta — **non supportano l'aggiunta al dominio**. Questo rende Samba AD DC inutilizzabile come soluzione di gestione centralizzata per il nostro caso d'uso specifico.

---

#### Alternativa identificata: [[FreeIPA]]

_[[FreeIPA]]_ (noto anche come _Red Hat Identity Management_) è una soluzione **open source** che ==integra in un unico prodotto== LDAP, Kerberos, una _Certificate Authority_ interna e una _Web UI_ di gestione.

È progettato specificamente per ambienti Linux enterprise e risulta **nativamente compatibile con AlmaLinux**, essendo un progetto sponsorizzato da Red Hat. [2](#user-content-fn-4)

Sebbene FreeIPA sia installato su un server Linux, ==supporta client Windows e macOS==: i laptop Windows possono autenticarsi tramite un trust con un'eventuale [[Active Directory]] esistente, oppure direttamente tramite [[SSSD]]. I MacBook si integrano tramite SSSD o profilo di configurazione.

> ⚠️ **Limitazione importante**: FreeIPA è principalmente orientato ad ambienti Linux. L'integrazione con client Windows, pur possibile, richiede configurazione aggiuntiva e non è nativa come in Samba AD DC. Soffre inoltre della stessa limitazione di Samba AD DC rispetto a Windows Home.

##### Cosa integra nativamente

| Componente | Ruolo |
|---|---|
| **389 Directory Server** | Backend LDAP per la gestione utenti/gruppi |
| **MIT Kerberos** | Autenticazione sicura |
| **Dogtag CA** | Certificate Authority interna per gestione certificati TLS |
| **BIND DNS** | Risoluzione nomi integrata |
| **SSSD** | Consente ai client di verificare le proprie credenziali tramite il server FreeIPA |
| **Web UI** | Interfaccia grafica per gestione utenti, gruppi, policy |

##### Requisiti minimi del server

FreeIPA richiede almeno **2 vCPU e 4 GB di RAM**. Con meno di 1.5 GB il processo Dogtag (CA) va in timeout durante l'installazione. È consigliato installarlo su una VM dedicata, senza altri servizi che usino le stesse porte (DNS, LDAP, porta 443). [3](#user-content-fn-5)

---

#### Alternativa adottata: [[🧠Tactical RMM]]

[[🧠Tactical RMM]] (TRMM) è uno strumento open source di **Remote Monitoring & Management** che consente la gestione centralizzata di endpoint Windows **senza richiedere il domain join**. È la soluzione che meglio si adatta alla nostra infrastruttura reale.

A differenza di Samba AD DC e FreeIPA — che richiedono che i client si uniscano a un dominio (funzionalità non disponibile su Windows Home) — Tactical RMM opera tramite un **agente leggero** installato su ogni client, che comunica con il server centrale via HTTPS/NATS.

##### Funzionalità rilevanti per il nostro caso d'uso

| Funzionalità | Descrizione |
|---|---|
| **Script remoti** | Esecuzione di PowerShell, Batch, Python su uno o più endpoint |
| **Gestione utenti locali** | Tramite script: disabilitazione account, rimozione da gruppi |
| **Blocco installazioni software** | Policy via Windows Installer, SRP, chiavi di registro — deployabili via script |
| **Patch Management** | Gestione Windows Update centralizzata con policy per severità |
| **Check e alert** | Monitoraggio CPU, RAM, disco, servizi con notifiche email/SMS/webhook |
| **Remote Desktop / Shell** | Accesso remoto tramite integrazione MeshCentral |
| **Automazione** | Task pianificati e policy di automazione per gruppo di client |
| **Installazione software** | Deploy silenzioso di applicazioni tramite script PowerShell |

> ℹ️ TRMM è compatibile con **Windows 7, 8.1, 10, 11** in tutte le edizioni — inclusa **Home** — e con Windows Server dalla versione 2008 R2 in poi.

---

#### Confronto finale: tutte le soluzioni

| Caratteristica                | Samba + OpenLDAP  | Samba AD DC           | FreeIPA               | Tactical RMM             |
| ----------------------------- | ----------------- | --------------------- | --------------------- | ------------------------ |
| **Ambiente ideale**           | Legacy / obsoleto | Misto Linux + Windows | Prevalentemente Linux | Qualsiasi flotta Windows |
| **Funziona con Windows Home** | ❌                 | ❌                     | ❌                     | ✅                        |
| **Richiede domain join**      | ✅                 | ✅                     | ✅                     | ❌                        |
| **Gestione utenti locali**    | No                | Tramite GPO           | No                    | Tramite script           |
| **Blocco installazioni**      | No                | GPO                   | No                    | Script + policy          |
| **Remote Desktop**            | No                | No                    | No                    | ✅ MeshCentral            |
| **Patch Management**          | No                | Parziale (WSUS)       | No                    | ✅ Integrato              |
| **Interfaccia di gestione**   | CLI               | CLI                   | Web UI + CLI          | Web UI + CLI             |
| **Complessità setup**         | Alta              | Media                 | Media                 | Bassa                    |
| **Gratuito**                  | Sì                | Sì                    | Sì                    | Sì                       |

---

#### Nota sulla separazione dei ruoli (DC vs File Server)

Samba può funzionare sia come _Domain Controller_ che come _File Server_, ma la wiki ufficiale di Samba ==sconsiglia esplicitamente di usare il DC anche come file server==. [1](#user-content-fn-1)

DC e file server hanno cicli di aggiornamento diversi e mescolarli introduce ==rischi di stabilità== e limitazioni tecniche sulle _Access Control List_. Questo significa che, se si scegliesse Samba, andrebbero previsti ==almeno due server distinti==: uno dedicato al _DC_ e uno al _file sharing_.

---

#### Conclusioni e raccomandazione

**[[🧠Tactical RMM]]** è la soluzione raccomandata per la nostra infrastruttura.

Samba AD DC, pur essendo tecnicamente superiore per ambienti con dominio Windows, si è rivelato **inutilizzabile** nel nostro contesto: la quasi totalità dei client monta **Windows 11 Home**, che non supporta il _domain join_. Lo stesso vincolo esclude FreeIPA.

Tactical RMM supera questo limite strutturale operando tramite **agente**, senza alcun requisito sull'edizione del sistema operativo. Consente di raggiungere gli obiettivi originali del progetto — controllo centralizzato degli account locali, blocco delle installazioni non autorizzate, monitoraggio e gestione remota — su tutta la flotta esistente, senza modifiche infrastrutturali ai client.

---

### Fonti

1. Samba Wiki ufficiale — _Setting up Samba as an Active Directory Domain Controller_: [https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller) [↩](#user-content-fnref-1) [↩2](#user-content-fnref-1-2)

2. StackShare — _FreeIPA vs OpenLDAP_: [https://stackshare.io/stackups/freeipa-vs-openldap](https://stackshare.io/stackups/freeipa-vs-openldap) [↩](#user-content-fnref-4)

3. ComputingForGeeks — _Install FreeIPA on Rocky/AlmaLinux_: [https://computingforgeeks.com/install-freeipa-server-on-rocky-almalinux/](https://computingforgeeks.com/install-freeipa-server-on-rocky-almalinux/) [↩](#user-content-fnref-5)

4. Red Hat — Documentazione FreeIPA / Identity Management: [https://docs.redhat.com/it/documentation/red_hat_enterprise_linux/9/html/installing_identity_management](https://docs.redhat.com/it/documentation/red_hat_enterprise_linux/9/html/installing_identity_management) [↩](#user-content-fnref-3)