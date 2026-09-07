

**Società:** Technology Advising S.r.l.
**Data di redazione:** 27 luglio 2026
**Redattore:** Federico Brunetti (IT)
**Destinatari:** Adriano Caligiuri (CTO), Segreteria/Compliance, Consulente del lavoro / DPO

---

## 1. Contesto e finalità del progetto

Il progetto nasce da una richiesta aziendale di dotare le postazioni di lavoro di misure tecniche di sicurezza, nell'ambito del percorso di adeguamento alla norma ISO/IEC 27001. Gli obiettivi indicati erano:

- impedire l'installazione e l'esecuzione di software non autorizzato sulle postazioni;
- monitorare lo stato di sicurezza delle macchine (aggiornamenti, antivirus, anomalie tecniche);
- attivare e gestire centralmente la cifratura dei dischi, con custodia delle chiavi di ripristino.

La selezione dello strumento è avvenuta per fasi successive: una prima valutazione basata su Samba Active Directory (a partire da documentazione aziendale pregressa relativa al percorso ISO 27001), scartata per incompatibilità con le edizioni Windows Home presenti in flotta; una seconda valutazione su FreeIPA; infine l'adozione, per la fase di test, della piattaforma open source **Tactical RMM** con il componente di accesso remoto **MeshCentral**, installata su server dedicato (VM Ubuntu 22.04, OVH Francoforte).

La fase di test aveva lo scopo di validare tecnicamente strumenti, configurazioni e procedure su un gruppo ristretto di macchine, prima di qualsiasi estensione alla flotta aziendale (~46 endpoint). L'estensione alla flotta **non è mai avvenuta**.

## 2. Perimetro della fase di test

Macchine coinvolte, con date di installazione dell'agente risultanti dal database del server:

| Macchina | Tipo | Edizione Windows | Agente installato il |
|---|---|---|---|
| TA-Test | Macchina di laboratorio (non assegnata) | 11 Pro | 15/06/2026 |
| Federico | Postazione del redattore | 11 Home | 16/06/2026 |
| ilaria | Postazione assegnata (pilota) | 11 Home | 30/06/2026 |
| Giovanni_Foglia | Postazione assegnata (pilota) | 11 Pro | 30/06/2026 |
| Luca_Caligiuri | Postazione assegnata (pilota) | 11 Pro | 01/07/2026 |
| NB-ANNA_ALFIERI | Postazione assegnata (pilota) | 11 Pro | 02/07/2026 |

I quattro colleghi coinvolti come postazioni pilota **erano stati informati** dell'installazione dell'agente e della natura sperimentale dell'attività, sia dal redattore sia dal CTO. [*Se esiste traccia scritta di tale comunicazione — messaggio, email, chat di gruppo — citarla qui con data.*]

Periodo di attività della piattaforma: dal 15/06/2026 al 24/07/2026, data di sospensione (Sezione 6).

## 3. Attività tecniche svolte

Sulle macchine del perimetro sono state configurate, tramite script eseguiti dalla piattaforma, le seguenti misure di sicurezza:

- **Cifratura del disco (BitLocker / Device Encryption)**, con custodia centralizzata delle chiavi di ripristino;
- **Regole di riduzione della superficie d'attacco (ASR)** e **Controlled Folder Access** di Windows Defender;
- **Policy di controllo delle applicazioni (WDAC)**: sulle postazioni assegnate ai colleghi la policy è stata mantenuta in **modalità di sola registrazione** (audit), che rileva senza mai bloccare; la modalità di blocco effettivo (enforcement) è stata attivata esclusivamente su macchine di laboratorio e del redattore;
- Script di verifica e diagnostica dello stato di sicurezza (stato BitLocker, stato Defender, versione policy).

L'elenco completo delle esecuzioni di script — con data, macchina di destinazione e operatore — è disponibile nell'export del registro attività della piattaforma (Allegato C).

**Verifica a posteriori sulla policy WDAC** (24/07/2026): sulle postazioni verificabili (Luca, Anna) i registri di sistema confermano che la policy di test non ha mai bloccato alcun programma; gli unici eventi di blocco presenti provengono da una policy di sistema Microsoft preinstallata, estranea al progetto. L'esito è documentato negli output conservati agli atti.

## 4. Accessi remoti alle postazioni

La ricostruzione completa degli accessi remoti: fonti, numeri, durate, volumi e configurazione del consenso utente, è oggetto del documento dedicato (**Allegato A**), di cui si riportano le conclusioni:

- **nessuna sessione di visualizzazione dello schermo** è mai stata aperta su postazioni assegnate a dipendenti;
- gli accessi si sono limitati a **trasferimenti di file di log tecnici** (volumi dell'ordine dei kilobyte) e a una sessione a riga di comando di 29 secondi;
- sulla postazione di Giovanni **nessuna sessione è mai stata stabilita**;
- la richiesta di consenso all'utente è stata **attivata in via definitiva il 22/07/2026** a livello di configurazione del server; per il periodo precedente, la circostanza e gli accessi effettivamente avvenuti sono dichiarati e documentati nell'Allegato A;
- tutti gli accessi risultano **attribuibili al solo redattore**; i tentativi di intrusione esterni registrati non sono mai andati a buon fine.

## 5. Dati raccolti e loro conservazione

| Dato | Natura | Dove è conservato |
|---|---|---|
| Log tecnici WDAC (elenco programmi eseguiti, per costruzione della whitelist) | Tecnico, finalità di sicurezza | Sulle singole macchine (`C:\WDAC\`); estratti trasferiti sulla macchina di laboratorio |
| Chiavi di ripristino BitLocker | Tecnico, necessario al recupero dei dischi cifrati | Campo dedicato della piattaforma + copia in archivio cifrato (KeePass), verificata il 27/07 |
| Inventario hardware/software e stato di sicurezza | Tecnico | Database della piattaforma |
| Registri delle attività (audit log TRMM, eventi MeshCentral) | Tecnico, tracciabilità operatore | Database del server; export in cartella evidenze |

Non sono mai stati raccolti: screenshot, registrazioni di sessione, dati di navigazione, metriche di inattività o produttività, contenuti di file personali.

## 6. Sospensione della piattaforma e verifiche di chiusura

A seguito delle valutazioni di conformità avviate internamente (Art. 4 L. 300/1970 e GDPR), in data **24/07/2026** su disposizione del CTO:

1. sono state **disattivate** tutte le automation policy e i controlli periodici;
2. sono state **revocate** le chiavi API della piattaforma;
3. è stato eseguito un **backup integrale** del server ed è stata avviata l'**esportazione delle evidenze** (registri, inventario, ricostruzione accessi);
4. lo stato reale di ogni postazione raggiungibile è stato **verificato dai registri** (policy WDAC, privilegi amministrativi locali).

Da tale data la piattaforma **non raccoglie più alcun dato**. Il server è mantenuto attivo al solo scopo di conservare i registri a supporto della valutazione in corso.

## 7. Misure adottate in sede di chiusura

- Attivazione definitiva della richiesta di **consenso utente** per ogni accesso remoto, a livello di configurazione del server (22/07/2026);
- installazione di **fail2ban** sul server, con blocco permanente automatico degli IP responsabili di tentativi di accesso non autorizzati;
- predisposizione della **rimozione automatica della policy WDAC** dalle postazioni non raggiungibili al momento delle verifiche, con esecuzione alla riconnessione;
- verifica di corrispondenza tra le chiavi di ripristino BitLocker custodite in piattaforma e l'archivio cifrato esterno.

## 8. Elementi in attesa di decisione

Alla data di redazione restano da definire, su decisione della direzione e sentito il consulente:

1. la **rimozione dei file di log tecnici residui** dalle postazioni pilota (con elenco verbalizzato di quanto rimosso), ovvero la loro conservazione fino al completamento della valutazione;
2. il completamento della **verifica sulle postazioni** di Ilaria e Giovanni [*aggiornare con esito del Check Policy Version, ora che risultano riconnesse*];
3. la **disinstallazione degli agent** dalle macchine client, con registrazione di data e ora per ciascuna (il server resta attivo);
4. la **comunicazione al gruppo di lavoro** della conclusione della fase di test;
5. le **modalità di consegna delle evidenze** al consulente (i file di export contengono nominativi e indirizzi IP).

## 9. Allegati

- **Allegato A**:  Ricostruzione degli accessi remoti (documento divulgativo con tabelle di dettaglio)
- **Allegato B**:  Tabella delle sessioni MeshCentral per postazione
- **Allegato C**:  Export dei registri (inventario agenti, audit log TRMM, riepiloghi per agente)  conservati sul server, condivisione da concordare
- **Allegato D**:  Output delle verifiche WDAC per postazione (policy attive, eventi di blocco)

---

**Il redattore**
Federico Brunetti

**Per presa visione**
Adriano Caligiuri (CTO)
