È un **sistema di vulnerability scanning**, ovvero scansiona qualsiasi host in rete alla ricerca di **vulnerabilità di sicurezza**. 
In origine era solo un motore di scansione Open Source, mentre oggi il prodotto completo si chiama ==**Greenbone Vulnerability Management**== ***GVM***, anche se nel linguaggio comune continua ad essere chiamato "OpenVAS"


È composto da più componenti che lavorano assieme:

| Componente                                                | Ruolo                                                                                                                                                                                 |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **GSA** (*Greenbone Security Assistant*)                  | Interfaccia web                                                                                                                                                                       |
| **gvmd** ( *Greenbone Vulnerability Management Daemon*👹) | E' il "cervello": riceve le richieste dall'interfaccia web, coordina lo scanner, salva configurazioni e risultati nel database (da non confondere con GVM, il nome dell'intera suite) |
| **GMP** (*Greenbone Management Protocol*)                 | Protocollo tra GSA e gvmd                                                                                                                                                             |
| **ospd** (*Open Scanner Protocol Daemon* 👹)              | E' il demone che implementa l'OSP, ovvero il Protocollo di comunicazione tra *gvmd* ed uno *Scanner*                                                                                  |
| **openvas-scanner**                                       | Il motore di scansione attiva vero e proprio: esegue gli script NASL del feed contro i target                                                                                         |
| **Notus Scanner**                                         | Scanner più recente e specializzato nei controlli locali: verifica le versioni dei pacchetti installati confrontandole con quelle vulnerabili not                                     |
| **Feed**                                                  | Database dei test di vulnerabilità (NVT) e dei dati correlati (CVE, CPE...), aggiornato da Greenbone                                                                                  |


I componenti visti finora restano invisibili durante l'uso quotidiano: non parli mai direttamente con gvmd o con ospd, ma lavori sempre attraverso gli oggetti che l'interfaccia ti mette a disposizione.

Sono questi oggetti il vero vocabolario operativo di OpenVAS.
Un **Target** definisce cosa scansionare, una **Scan Config** definisce come farlo (quali NVT del feed vengono usati), un **Task** unisce i due elementi in un lavoro eseguibile, e ogni volta che lo lanci il sistema produce un Report pieno di Result, ciascuno con la sua **Severity** calcolata dal **CVSS** del **CVE** corrispondente.

 È lo stesso flusso feed, scanner, manager presente  nella prima tabella, solo osservato dal punto di vista di chi usa lo strumento. 
 Con l'accumularsi dei report, il sistema popola anche l'**Asset inventory**, cioè l'elenco di host, sistemi operativi e certificati che ha scoperto scansionando.


| Termine                            | Significato                                                                                                                   |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| NVT (*Network Vulnerability Test*) | Singolo test di vulnerabilità contenuto nel feed                                                                              |
| **CVE**                            | Identificativo standard di una vulnerabilità nota                                                                             |
| **CVSS**                           | Punteggio numerico di gravità di una vulnerabilità, da cui deriva la Severity                                                 |
| **Target**                         | Host o range di IP da scansionare                                                                                             |
| **Scan Config**                    | Definisce quali NVT (quali famiglie di test) vengono eseguiti durante la scansione, es. "Full and fast" oppure solo discovery |
| **Task**                           | Il lavoro di scansione vero e proprio: unisce un Target, una Scan Config e uno scanner. Quando lo lanci produce un Report     |
| **Report**                         | L'esito di un'esecuzione di un Task; contiene i Result                                                                        |
| **Result**                         | Una singola vulnerabilità trovata, collegata a un NVT e spesso a uno o più CVE, con una Severity                              |
| **Severity Class**                 | Fascia di gravità in cui viene raggruppato il CVSS: Log, Low, Medium, High, Critical                                          |
| **Asset**                          | Inventario costruito automaticamente dai risultati delle scansioni: host, sistemi operativi, certificati TLS rilevati         |
## Le tab di un Report

Quando si apre un report di una scansione completata, i dati vengono presentati attraverso diverse tab in alto, che sono modi diversi di guardare le stesse informazioni raccolte durante lo scan, non scansioni separate.

La tab Information è la scheda anagrafica dello scan: riporta quale Task lo ha generato, l'orario di inizio e fine, la durata, lo stato, il numero di host scansionati e il filtro applicato alla visualizzazione dei risultati.

La tab **Results** elenca i singoli finding trovati, cioè le vulnerabilità individuate durante lo scan, ordinabili per *severità, host, data di rilevamento e QoD* (Quality of Detection, l'affidabilità del rilevamento).

Le tab successive raggruppano gli stessi dati per oggetto invece che per vulnerabilità: **Hosts** elenca gli host coinvolti nello scan, **Ports** le porte trovate aperte, **Applications** i software e servizi identificati tramite banner o versione, **Operating Systems** i sistemi operativi rilevati sugli host.

La tab **CVEs** elenca gli identificativi CVE univoci citati nei risultati del report. La tab **Closed CVEs** invece è particolarmente utile in ottica di audit, perché mostra i CVE che erano stati trovati in scansioni precedenti sugli stessi host e che ora non compaiono più: sono quindi vulnerabilità già risolte, una prova concreta di remediation nel tempo.

La tab **TLS Certificates** raccoglie i certificati trovati sui servizi che espongono HTTPS/TLS. 
La tab **Error Messages** elenca i test che non sono riusciti a completarsi correttamente durante lo scan, utile per capire cosa non è stato verificato. 
La tab **User Tags** infine mostra le etichette manuali applicate agli elementi del report, quando presenti.

## Il filtro dei risultati

Il numero mostrato in cima alla tab Results, ad esempio "90 of 2614", indica che i==l filtro attivo nasconde parte dei dati grezzi raccolti dallo scan.==
Il filtro di default, visibile nel campo Filter della tab Information, ha tipicamente questa forma: 
```
apply_overrides=0 levels=hml min_qod=70.
```

`apply_overrides=0` significa che non vengono applicate eventuali override manuali sulla severità di un finding. Un **override** serve a riclassificare manualmente un risultato, ad esempio per segnalarlo come falso positivo o per correggerne la gravità rispetto a quella assegnata automaticamente.

`levels=hml `limita la visualizzazione ai soli risultati di livello High, Medium e Low, escludendo quelli di livello Log, che sono informazioni raccolte durante lo scan ma non vere e proprie vulnerabilità.

`min_qod=70` nasconde i rilevamenti con una **Quality of Detection** *QoD* inferiore al 70%, cioè quelli meno affidabili, per ridurre il rumore e concentrare l'attenzione sui finding con maggiore certezza di essere reali.

Questo spiega perché il totale grezzo (2614) e quello mostrato dopo il filtro (90) sono così diversi: non sono dati diversi, ma la stessa scansione osservata con un livello di rumore diverso.

## Anatomia di un finding (Result)

Ogni singola vulnerabilità elencata nella tab Results apre una pagina di dettaglio con una struttura fissa, che si ripete identica per qualsiasi finding: cambia solo il contenuto specifico, non l'organizzazione delle sezioni.

Il testo introduttivo descrive in astratto cosa verifica quel test, senza riferimento all'host specifico: è lo stesso identico testo su ogni macchina dove il finding compare, perché descrive il controllo, non il risultato. Ad esempio, per un test sulle cipher suite TLS, l'introduzione si limita a spiegare che il controllo verifica quali cipher suite vengono accettate da un servizio HTTPS.

Il **Detection Result** è invece la prova concreta raccolta su quello specifico host: cosa ha effettivamente trovato lo scanner. È la sezione da citare come evidenza in un audit, perché documenta il dato osservato, non la teoria generale del test.

Il **Product Detection Result** indica cosa è stato identificato come oggetto del controllo. Può essere un software specifico con la sua versione, oppure, quando il test riguarda un protocollo o uno standard piuttosto che un prodotto, un riferimento CPE generico che descrive lo standard verificato invece di un'applicazione concreta.

L'**Insight** spiega la logica con cui il finding viene giudicato vulnerabile, cioè la regola tecnica alla base della valutazione. Nel caso delle cipher suite 3DES, ad esempio, l'insight chiarisce che si tratta di un cifrario a blocchi da 64 bit vulnerabile all'attacco SWEET32, con il relativo CVE di riferimento.

Il **Detection Method** descrive come lo scanner è arrivato a quella conclusione, se tramite una verifica attiva oppure riutilizzando dati raccolti da un test precedente, e riporta l'OID, l'identificativo univoco dello script NVT nel feed, utile per rintracciare il test esatto nella documentazione o citarlo in un report.

**Affected Software/OS** descrive lo scope generale del problema, cioè quali tipi di sistemi o servizi sono potenzialmente esposti a quella classe di vulnerabilità, mentre Impact ne spiega la conseguenza pratica nel caso venga sfruttata da un attaccante.

**Solution** è probabilmente la sezione più rilevante ai fini della gestione delle vulnerabilità: indica il tipo di intervento richiesto (mitigazione, workaround, aggiornamento del vendor...) e descrive la correzione concreta. Alcune vulnerabilità si risolvono con una semplice riconfigurazione, altre richiedono un intervento strutturale più ampio, ed è proprio questa differenza a determinare priorità e tempistiche di remediation.

Infine **References** raccoglie i riferimenti esterni, tipicamente CVE e bollettini CERT, utili per documentare le fonti e per approfondire la vulnerabilità quando serve.