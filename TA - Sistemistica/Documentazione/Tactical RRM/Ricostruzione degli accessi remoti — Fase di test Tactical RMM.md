

**Documento di supporto al verbale di chiusura della fase di test.** Tutti i dati riportati provengono dai registri (log) conservati sul server e sono verificabili in qualunque momento. Nulla di quanto segue si basa sulla sola memoria dell'operatore, salvo dove esplicitamente indicato.

---

## 1. Stato attuale: la piattaforma è ferma

Da **venerdì 24 luglio 2026** la piattaforma non raccoglie più alcun dato dalle macchine:

- tutti i controlli automatici periodici sono stati disattivati;
- le chiavi di accesso all'API (i "codici" che permettono agli script di comunicare con il server) sono state revocate;
- nessuno script viene più eseguito sugli endpoint.

Il server è stato mantenuto acceso e intatto, come richiesto, esclusivamente per conservare i registri delle attività passate — che sono la fonte di questo documento.

---

## 2. Come leggere questo documento: cosa registra il sistema

Per interpretare correttamente i dati serve sapere cosa i registri contengono e cosa no.

**Il sistema di accesso remoto (MeshCentral) distingue tre tipi di sessione:**

|Tipo|Cosa significa in pratica|
|---|---|
|**Desktop**|L'operatore vede lo schermo della macchina, come se fosse seduto davanti. È il tipo di accesso più invasivo.|
|**Terminal**|L'operatore apre una riga di comando sulla macchina. Non vede lo schermo né i programmi aperti dall'utente.|
|**File**|L'operatore apre una finestra di gestione file per copiare file da o verso la macchina. Non vede lo schermo.|

Per ogni sessione i registri riportano: data e ora di inizio e fine, durata in secondi e volume di dati trasferiti (in byte). **Non riportano i nomi dei singoli file trasferiti**: questo è un limite del software, e le affermazioni su _quali_ file siano stati copiati restano dichiarazioni dell'operatore, indicate come tali alla Sezione 6.

**Il volume di dati aiuta a capire la natura di una sessione**: un file di log di testo pesa pochi kilobyte (KB); un documento, una foto o un archivio pesano da centinaia di KB a molti megabyte (MB). Sessioni con traffico di pochi KB sono compatibili solo con file di testo di piccole dimensioni.

---

## 3. La richiesta di consenso all'utente

MeshCentral dispone di una funzione (`userConsentFlags`) che, quando attiva, **mostra una finestra sullo schermo dell'utente ogni volta che un operatore tenta di accedere** ("l'operatore X richiede l'accesso al desktop: accetti?"). L'utente deve accettare esplicitamente, altrimenti la sessione non parte.

**Cosa è successo nel nostro caso:** nella configurazione predefinita del software questa richiesta di consenso è **disabilitata**. Può essere attivata in due modi diversi:

- dall'**interfaccia grafica** di amministrazione — dove però resta un'impostazione modificabile in ogni momento da chiunque disponga di un accesso amministrativo alla console;
- nel **file di configurazione del server**, dove diventa invece un'impostazione stabile, che nessun operatore può disattivare dalla console.

Nella prima fase del test la configurazione era quella predefinita (consenso non attivo). Nel corso del progetto, in previsione di estendere l'accesso alla console ad altri operatori, è stata ricercata e individuata la seconda modalità — quella a livello di file di configurazione — con l'obiettivo di garantire che nessun futuro operatore potesse accedere alle postazioni senza avviso all'utente. La richiesta di consenso è stata quindi introdotta nella configurazione del server; è stata successivamente sospesa in via temporanea per la durata delle attività tecniche di raccolta dei log (una conferma manuale a ogni trasferimento avrebbe richiesto di interrompere ripetutamente i colleghi per operazioni di pochi KB), ed è stata **attivata in via definitiva il 22 luglio 2026** (13:50, con riavvio del servizio alle 13:53). Da quel momento ogni accesso richiede il consenso esplicito dell'utente.

**Conseguenza da dichiarare con trasparenza:** le sessioni sulle postazioni dei colleghi elencate alla Sezione 4 sono avvenute **prima** di questa attivazione definitiva, quindi senza che comparisse la finestra di consenso. Erano tecnicamente possibili accessi senza avviso; i registri (Sezione 4) documentano quali accessi siano effettivamente avvenuti e di che tipo — esclusivamente trasferimenti di file e una sessione a riga di comando, nessuna visualizzazione dello schermo.

---

## 4. Accessi alle postazioni dei colleghi: i numeri

Dai registri completi del periodo, le sessioni verso postazioni assegnate a dipendenti sono state le seguenti.

### Riepilogo

|Postazione|Sessioni desktop (visione schermo)|Sessioni terminal|Sessioni file|
|---|---|---|---|
|Ilaria|**0**|0|7|
|Luca|**0**|0|5|
|Anna|**0**|1 (29 secondi)|1|
|Giovanni|**0**|0|0|

**Il dato principale: nessuna sessione di visualizzazione dello schermo è mai stata aperta su alcuna postazione assegnata a un dipendente.** Gli accessi si sono limitati al trasferimento di file e a una singola sessione a riga di comando di 29 secondi.

### Dettaglio delle sessioni

|Data|Postazione|Tipo|Durata|Dati trasferiti|Nota|
|---|---|---|---|---|---|
|09/07|Ilaria|file|12 min|~16 KB||
|09/07|Ilaria|file|2 h 49 min|~90 KB|Finestra rimasta aperta mentre l'operatore lavorava ad altro: il volume (90 KB in quasi 3 ore) mostra che non c'è stata attività continuativa|
|09/07|Ilaria|file|7 min|~450 KB||
|09/07|Ilaria|file|14 min|~180 KB||
|09/07|Ilaria|file|28 min|~15 KB||
|13/07|Ilaria|file|2 min|~4 KB||
|20/07|Ilaria|file|21 min|~12 KB||
|09/07|Luca|file|2 min|~5 KB||
|09/07|Luca|file|9 min|~7 KB||
|13/07|Luca|file|27 min|~11 KB||
|16/07|Luca|file|7 min|~8 KB||
|16/07|Luca|file|1 min|~3 KB||
|09/07|Anna|file|1 min|~4 KB||
|16/07|Anna|terminal|29 sec|~2 KB|Unica sessione a riga di comando su postazione assegnata|

Tutti i volumi sono nell'ordine dei kilobyte: compatibili con file di log di testo, non con documenti, immagini o archivi.

### Il caso di Giovanni

Il registro della console TRMM riporta due richieste di sessione remota verso la postazione di Giovanni (1 e 22 luglio). Tuttavia, **i registri di MeshCentral — che tracciano le sessioni effettivamente stabilite — non contengono alcuna sessione verso quella postazione**, di nessun tipo: gli unici eventi presenti sono le accensioni del dispositivo.

La spiegazione tecnica è documentata: fino al 22 luglio la funzione "Take Control" della console **non era funzionante** — restituiva un errore di autenticazione su qualunque browser, per un problema legato al vecchio dominio DNS gratuito allora in uso, diagnosticato il 2 luglio e non risolvibile via configurazione. Il problema è stato sanato solo con la migrazione al dominio aziendale, completata il 22 luglio. Le due richieste risultano quindi **tentativi non andati a buon fine**: cliccati dalla console, mai tradotti in una sessione reale.

---

## 5. Macchine di laboratorio (fuori dal perimetro)

La grande maggioranza delle sessioni registrate — incluse tutte le sessioni desktop e i trasferimenti di grandi dimensioni (157, 271 e 541 MB) — riguarda due macchine **non assegnate ad alcun dipendente**:

- **TA-TEST**: macchina di laboratorio (Windows 11 Pro) usata per compilare e testare le configurazioni;
- **Federico**: la postazione dell'operatore stesso.

Su queste macchine l'attività è stata intensa perché è lì che si svolgeva il lavoro tecnico: nessun dato di alcun dipendente vi era presente.

---

## 6. Cosa è stato trasferito dalle postazioni dei colleghi

Come indicato alla Sezione 2, i registri non riportano i nomi dei file. **Per dichiarazione dell'operatore**, i trasferimenti dalle postazioni dei colleghi hanno riguardato esclusivamente i file di log tecnici generati dallo strumento di analisi delle applicazioni (salvati nella cartella `C:\WDAC\` di ciascuna macchina), raccolti per costruire e correggere la configurazione di sicurezza. I volumi registrati (pochi KB per sessione) sono coerenti con questa dichiarazione.

Questi file di log contengono l'elenco dei programmi eseguiti sulla macchina — un dato tecnico, raccolto al solo scopo di definire l'elenco del software da autorizzare. Sono tuttora presenti sulle macchine; la loro eventuale rimozione è tra le decisioni in attesa.

---

## 7. Chi ha operato: attribuzione degli accessi

- Tutte le operazioni risultano eseguite dall'**unico account amministrativo** della console, le cui credenziali sono in possesso del solo operatore (Federico).
- Gli indirizzi IP di provenienza registrati (`188.12.216.152`, `130.25.1.104`, `176.245.174.243`) corrispondono alle connessioni abituali dell'operatore (ufficio, abitazione, rete mobile) e sono stati da lui riconosciuti.
- Nei registri compaiono anche **tentativi di accesso falliti** con nomi utente estranei (`tactical`, `sd` e altri), provenienti da indirizzi IP esterni sempre diversi. Si tratta dei consueti tentativi automatici di intrusione che qualunque servizio esposto su internet riceve quotidianamente: **nessuno è andato a buon fine** e nessun accesso da soggetti terzi risulta nei registri.
- A ulteriore protezione, è stato installato sul server **fail2ban**, un meccanismo che blocca automaticamente e in modo permanente gli indirizzi IP che tentano accessi non autorizzati, riducendo l'esposizione a tentativi futuri.

---

## 8. In sintesi

1. La piattaforma è **ferma dal 24 luglio**: nessun dato viene più raccolto.
2. **Nessuno ha mai visto lo schermo di un collega**: zero sessioni desktop sulle postazioni assegnate.
3. Gli accessi effettuati sono stati **trasferimenti di file di log tecnici** (volumi di pochi KB) e una sessione a riga di comando di 29 secondi.
4. Sulla postazione di Giovanni **non è mai stata stabilita alcuna sessione**.
5. La finestra di consenso all'utente era inattiva per un errore di configurazione, **corretto il 22 luglio**; la circostanza è dichiarata apertamente e i registri documentano cosa è effettivamente avvenuto in quel periodo.
6. Tutti gli accessi sono **attribuibili al solo operatore**; i tentativi di intrusione esterni sono falliti.

---

_I file di export a supporto (registri completi in formato CSV/TXT) sono conservati sul server nella cartella evidenze e disponibili per verifica. Le modalità di condivisione con i consulenti sono da concordare, trattandosi di dati che contengono nomi e indirizzi IP._