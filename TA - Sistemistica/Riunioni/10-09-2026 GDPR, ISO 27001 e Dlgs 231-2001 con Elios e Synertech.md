---
data: 2026-09-10
tipo: riunione
titolo: 10-09-2026 GDPR, ISO 27001 e Dlgs 231-2001 con Elios e Synertech
argomento: GDPR, ISO 27001 e Dlgs 231/2001
fornitori:
  - Synertech
  - Elios
tags:
  - riunione
  - gdpr
  - dlgs231
  - iso27001
  - nis2
  - sicurezza
---


Incontro con due aziende distinte: [[Synertech]] si occupa del ==**Dlgs 231/2001**== ed è quella con cui abbiamo il **contratto**, mentre [[Elios]] segue ==**GDPR e [[NIS2]]**==. Ci affiancano nel percorso verso la **[[ISO 27001]]** e la conformità **GDPR**.

La **prossima riunione** ==non è ancora stata decisa==.

## Stato attuale

Il termine per l'**attuazione delle misure di sicurezza** è fissato al ==**31 ottobre 2026**== e riguarda i soggetti inseriti nell'**elenco [[NIS]] del 2025**.

## Accountability e [[NIS2]]

Il GDPR ha introdotto l'**accountability**, cioè dichiarare quello che si fa e quello che non si fa. In passato l'adempimento era **solo documentale**, ma con la [[NIS2]] la situazione è cambiata: l'**autorità è tecnica** e analizza sia la parte documentale sia quella effettiva. Di conseguenza ==ogni cosa deve essere dimostrabile==.

## Disaster recovery vs continuità operativa

Il [[Disaster Recovery]] **non coincide** con il piano di ==[[Continuità operativa]]==, distinzione spiegata nel nuovo GDPR. Il piano di continuità operativa consiste nel **piano di subentro in caso di emergenze**.

## Documentazione dei progetti

*Adriano* segnala che, avendo **tanti progetti in continua evoluzione**, scriverli e revisionarli costantemente richiede **troppo lavoro**, e chiede quindi quale sia la ==best practice== per superare questo limite.

## Gestione incidenti e notifica entro 72 ore

Secondo *Pasquale*, anche **non essendo soggetto [[NIS2]]**, dobbiamo essere pronti ad aiutare il cliente entro ==**72 ore**==, e resta da capire quanto la [[ISO 27001]] ci abbia spinto a organizzarci per **presidiare la dichiarazione di incidenti**.

*Adriano* riferisce che finora **non ci sono stati incidenti rilevanti**, solo incidenti minori, e che **non esiste una figura specifica** dedicata a questo ambito: se ne occupano lui o *Genny*.

*Pasquale* sottolinea che è importante ==avere figure specifiche==, sia per il GDPR sia per la ISO, e che servirebbero **procedure operative comprensibili anche dai clienti**. Cita come esempio il mondo della sanità, dove è frequente l'**uso scorretto del CCN o del CC nelle email**, che costituisce una ==violazione privacy==: scrivere una procedura da condividere con loro, anche se dovrebbe essere compito dell'**ASL**, faciliterebbe il nostro rapporto.

## Informazioni aziendali richieste

Viene chiesto che l'ente, quando avvia una nuova tecnologia o procedura, esegua l'==**AIP** (Analisi impatto privacy)== con un elenco delle cose che comportano un **rischio elevato per la privacy**, e ci viene richiesto di fornire una ==**DPA di prodotto**==. *Adriano* risponde che noi ce l'abbiamo **per fornitore, non per singolo progetto**. Fornendo una DPA saremmo comunque pronti qualora ce la chiedessero, ad esempio perché il cliente riceve un'**ispezione dalla Guardia di Finanza**.

*Adriano* aggiunge che alcuni clienti, come le **ASL**, hanno procedure interne di gestione del GDPR, tipo l'uso di intranet, che però **non hanno a che fare con il progetto**.

## Sicurezza a livello architetturale

A livello architetturale stiamo implementando **best practice per la sicurezza**. Siamo passati da una gestione di build e delivery a un processo ==**CI/CD**==, con una **catena di controllo del codice**, **CVE in automatico**, **allerta di vulnerabilità note** e simili, e abbiamo **già documentato** il tutto con gli ambiti di sicurezza.

Esistono inoltre parametri che vengono **sottovalutati**, come gli ==**SLA**==, che sono una cosa seria e vanno definiti **in base all'effettiva capacità dell'azienda**.

## Analisi e gestione dei rischi

Il GDPR, nell'analisi dei rischi, vede come rischio l'==**interessato**==.

Sulla gestione dei rischi, *Adriano* dice che li stiamo gestendo con ==**PRINCE2 Agile**==. Bisogna definire il ==**risk appetite**==, cioè il **rischio massimo che si può accettare**, e queste cose **devono essere registrate**. È importante anche mettere per iscritto la **pianificazione della gestione del rischio**, qualora mancasse.

## Occhiali Meta e policy sugli strumenti

La questione degli **occhiali Meta** pone un problema di **GDPR e sicurezza aziendale**, e ==non sappiamo bene come procedere==. Serve una ==**policy sull'utilizzo degli strumenti**== e sulle dotazioni che si adoperano: è vero che gli occhiali sono graduati, ma **non si deve avere uno strumento in grado di registrare**.

Il GDPR è stato scritto da poche persone che ne capivano, in maniera **semplice ed essenziale**. Va applicato il ==**principio di minimizzazione (articolo 5)**==: c'è davvero bisogno di poter riprendere il proprio lavoro? Usare gli occhiali è stata una **scelta personale**, quindi occorre fare il **bilanciamento degli interessi**. Il ==**rischio privacy è altissimo**== con gli occhiali Meta.

Per **mitigare il problema** si sta aggiornando la ==**privacy policy aziendale**==, lavoro **già in corso**. Ma se il dipendente non volesse firmare? In quel caso **si può fare un richiamo**. Come esempio viene citata la scuola: non si possono mettere **telecamere** a scuola e si possono accendere solo **fuori dall'orario scolastico**, quindi anche gli occhiali Meta **non possono essere indossati da un professore**.

## Smart working

Dobbiamo regolare anche la **postazione dello smart working**. Ufficialmente noi **non facciamo smart working**, però è ==essenziale== regolarlo.

## Ruoli e responsabilità

Nell'ambito della [[ISO 27001]] l'aspetto diventa **anche amministrativo**, e viene fatto gestire anche dal **team amministrazione** (*Marco* e *Nunzia*). *Pasquale* ribadisce che è ==molto importante definire i ruoli==, sia per la 27001 sia per il GDPR.

## [[Dlgs 231-2001]]

Il percorso sul Dlgs è seguito da [[Synertech]], con cui abbiamo il **contratto**, e *Pasquale* è il **dipendente [[Elios]]** che se ne occupa. Per l'**adeguamento** abbiamo ==**una cinquantina di giorni**==, in linea con la scadenza del **31/10/2026**.

Il **Dlgs 231/2001** è un **modello di organizzazione, gestione e controllo**: se adottato in maniera efficace, ==anche qualora si verificassero dei reati non avremmo responsabilità==. È un requisito **sempre più richiesto dalle aziende** come garanzia ed è ottimo anche **per ottenere appalti**.

Il percorso si articola in nove fasi: si parte dalla **raccolta documentale e analisi** (organigramma dipendenti, tipi di contratti senza le parti sensibili, procedure esistenti e incarichi in corso), poi la **mappatura dei processi sensibili**, il ==**risk assessment**== e la ==**gap analysis**==, cioè il confronto tra i presidi di controllo già in essere e quelli necessari a mitigare i rischi individuati. Seguono la **redazione della parte generale**, con la costruzione della struttura del modello (principi, ruoli), e la **redazione della parte speciale e dei protocolli**.

Si passa quindi alla **costituzione dell'Organismo di Vigilanza**, che è l'organo di controllo al quale devono essere inviati **sistematicamente i flussi organizzativi**, e al ==**sistema disciplinare e whistleblowing**==, cioè le sanzioni nel caso in cui i dipendenti non rispettino le regole aziendali: in quest'ambito produrremo anche il **codice etico** che dovranno seguire tutti i destinatari del modello. L'ultima fase è **formazione e comunicazione**, cioè l'aggiornamento dei dipendenti.

## Questionario e verifiche

Ci è stato preparato un **questionario** per capire la situazione attuale; **possiamo anche non rispondere** per motivi di privacy. I documenti richiesti (questionario, **AIP**, **DPA** e simili) vanno inviati **entro due giorni**, quindi entro il ==**12/09/2026**==.

Tra le verifiche si farà quella del **sito**, che è la ==finestra sul mondo==. Per esempio la **Guardia di Finanza, reparto privacy**, guarda prima il sito, e al di là della conformità GDPR si trovano spesso anche **vulnerabilità**. La conformità del sito deve valere nei confronti del GDPR e dell'**Agenzia delle Entrate**, dato che ==le autorità sono diverse==.

Avendo **progetti finanziati** (è il nostro caso), in parte dobbiamo rispettare la ==**trasparenza**==, quindi dobbiamo capire cosa vada pubblicato sul sito a questo titolo. *Adriano* osserva però che **il nostro sito è morto**, ed è in **WordPress**. Bisogna guardare anche all'==**accessibilità del sito**== da parte di dipendenti o clienti con **disabilità**.

I **test** verranno fatti ==solo sul perimetro esterno==: [[Elios]] analizza **il perimetro esterno**, incluso il **sito internet**, perché è lì che ha **gli strumenti per farlo**. Dobbiamo inoltre creare un ==**Drive in condivisione**== per mettere i documenti a loro disposizione.

## Decisioni

_(nessuna decisione formalizzata durante la riunione)_

## Action item

### Entro il ==12/09/2026== (due giorni)

- [ ] Compilare e inviare il **questionario** ricevuto, decidendo quali risposte dare
- [ ] Inviare l'==**AIP**== (Analisi impatto privacy)
- [ ] Fornire la ==**DPA di prodotto**==
- [ ] Creare il **Drive condiviso** per lo scambio dei documenti

### Entro il ==31/10/2026==

- [ ] Verificare **attuazione misure di sicurezza**
- [ ] Completare l'**adeguamento al Dlgs 231/2001** con [[Synertech]] (circa **50 giorni** di tempo)

### Senza scadenza definita

- [ ] Individuare una **best practice** per la stesura e revisione della documentazione dei progetti
- [ ] Definire **ruoli e figure specifiche** per GDPR e ISO 27001, compresa la gestione incidenti
- [ ] Scrivere **procedure operative** comprensibili anche dai clienti (es. uso corretto di CC e CCN nelle email)
- [ ] Definire e registrare il **risk appetite** e mettere per iscritto la pianificazione della gestione del rischio
- [ ] Completare l'aggiornamento della **privacy policy aziendale** (caso occhiali Meta) e la **policy sull'utilizzo di strumenti e dotazioni**
- [ ] Regolare la **postazione di smart working**
- [ ] Definire cosa pubblicare sul sito in materia di **trasparenza** (progetti finanziati)
- [ ] Verificare **conformità, vulnerabilità e accessibilità** del sito
- [ ] Fissare la **data della prossima riunione**

## Appunti
