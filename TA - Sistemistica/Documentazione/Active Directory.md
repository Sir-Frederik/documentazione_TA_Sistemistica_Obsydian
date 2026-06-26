
> **Cos'è**: ==Servizio di directory== sviluppato da Microsoft, introdotto con Windows Server 2000. 
> Gestisce in modo ==centralizzato **identità, autenticazione e autorizzazioni**== di utenti, computer e risorse in una rete aziendale.

---

## Concetto base 

Immagina un'azienda con 200 computer e 150 dipendenti. Senza AD, ogni computer avrebbe i suoi utenti locali da gestire separatamente. Con AD:

- Ogni dipendente ha **un solo account** valido su tutta la rete
- L'amministratore gestisce tutto da un punto centrale
- Un utente si logga su **qualsiasi PC** dell'azienda con le stesse credenziali

---

## Componenti principali

|Componente|Cos'è|
|---|---|
|**Domain Controller (DC)**|Il server che esegue AD e gestisce autenticazione/autorizzazioni|
|**Domain**|L'unità base di AD — es. `azienda.local`|
|**Forest**|Insieme di più domain che si fidano l'uno dell'altro|
|**Tree**|Gerarchia di domain con namespace condiviso|
|**OU (Organizational Unit)**|"Cartelle" dentro il domain per organizzare utenti/computer|
|**GPO (Group Policy Object)**|Regole e configurazioni applicate automaticamente a utenti/computer|
|**LDAP**|Protocollo usato da AD per interrogare la directory|
|**Kerberos**|Protocollo di autenticazione principale usato da AD|
|**DNS**|AD dipende fortemente da DNS per funzionare|

---

## Come funziona l'autenticazione (Kerberos)

```
1. L'utente inserisce username e password sul PC
2. Il PC contatta il DC e chiede un "biglietto" (TGT - Ticket Granting Ticket)
3. Il DC verifica le credenziali e rilascia il TGT
4. Quando l'utente accede a una risorsa (es. una share), 
   presenta il TGT per ottenere un ticket specifico per quella risorsa
5. La risorsa verifica il ticket → accesso consentito
```

> ℹ️ La password **non viaggia mai in chiaro** sulla rete con Kerberos. È uno dei motivi per cui è preferito a [[NTLM]].

---

## NTLM vs Kerberos

| |NTLM|Kerberos|
|---|---|---|
|**Tipo**|Challenge-response|Ticket-based|
|**Sicurezza**|Più debole|Più forte|
|**Usato quando**|Fallback, IP diretti, workgroup|Default in ambienti AD|
|**Vulnerabilità**|Pass-the-Hash, NTLM Relay|Golden Ticket, Pass-the-Ticket|
|**Richiede DC?**|No (può essere locale)|Sì|

---

## Struttura gerarchica

```
Forest: azienda.com
│
├── Tree: azienda.com  (root domain)
│   ├── OU: Direzione
│   │   └── Utenti, Computer, GPO
│   ├── OU: IT
│   │   └── Utenti, Computer, GPO
│   └── OU: Risorse Umane
│
└── Child Domain: filiale.azienda.com
```

---

## GPO – Group Policy Object

Permettono di applicare configurazioni automaticamente a interi gruppi di utenti o computer. Esempi pratici:

- Bloccare l'accesso al pannello di controllo per certi utenti
- Impostare lo screensaver con lock dopo 5 minuti su tutti i PC
- Distribuire software automaticamente
- Configurare le impostazioni del firewall su tutti i computer del dominio

---

## AD in ambienti Linux

Su Linux, AD non è nativo. Le soluzioni per integrare Linux in un dominio AD sono:

| Soluzione         | Descrizione                                                           |
| ----------------- | --------------------------------------------------------------------- |
| **[[🏦 SAMBA]]**  | Può agire da Domain Controller AD completo (open source)              |
| **SSSD**          | Permette a un client Linux di autenticarsi tramite un DC AD esistente |
| **Winbind**       | Componente di Samba per integrare Linux in un dominio Windows         |
| **realm / adcli** | Tool moderni per aggiungere una macchina Linux a un dominio AD        |

---

## Sicurezza – Attacchi comuni su AD

|Attacco|Descrizione|
|---|---|
|**Pass-the-Hash**|Riutilizzo di hash [[NTLM]] per autenticarsi senza la password|
|**Pass-the-Ticket**|Riutilizzo di ticket Kerberos rubati|
|**Golden Ticket**|Forgiatura di un TGT usando l'hash dell'account `krbtgt` — accesso illimitato al dominio|
|**Silver Ticket**|Forgiatura di un ticket per un servizio specifico|
|**Kerberoasting**|Richiesta di ticket per service account e crack offline dell'hash|
|**DCSync**|Simulazione di un DC per ottenere hash di tutti gli utenti tramite replica AD|
|**BloodHound**|Tool di enumerazione dei percorsi di privilege escalation in AD|

> ⚠️ **Il Golden Ticket è l'attacco più critico**: chi lo possiede ha accesso completo al dominio anche dopo il reset delle password, finché non si resetta l'account `krbtgt` due volte.

---

## Porte di rete rilevanti

|Porta|Protocollo|Uso|
|---|---|---|
|`53`|DNS|Risoluzione nomi AD|
|`88`|Kerberos|Autenticazione|
|`135`|RPC|Comunicazione DC|
|`389`|LDAP|Query directory|
|`445`|SMB|File sharing, GPO|
|`636`|LDAPS|LDAP su TLS|
|`3268/3269`|Global Catalog|Ricerca in tutto il forest|
