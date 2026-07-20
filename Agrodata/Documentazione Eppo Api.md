

## Cos'è EPPO

**EPPO** = _European and Mediterranean Plant Protection Organization_, organizzazione intergovernativa fondata nel 1951 che coordina la protezione delle piante nell'area euro-mediterranea. È l'ente che definisce quali organismi nocivi sono soggetti a quarantena e a regolamentazione nell'UE.

Il **Global Database** è la sua banca dati pubblica: copre circa 98.000 specie tra piante (coltivate e spontanee) e organismi nocivi, con dati di dettaglio su circa 1.900 organismi di interesse regolatorio.

> [!warning] Attenzione all'omonimia Esiste una società statunitense chiamata **Eppo, Inc.** (`eppo.cloud`, `geteppo.com`) che si occupa di _feature flagging_ e _A/B testing_. Non ha nulla a che vedere con noi, ma domina i risultati di ricerca su Google. **I nostri domini sono solo `*.eppo.int`.** Se nella documentazione trovate `X-Eppo-Token`, `/experiments` o `/feature-flags`, siete sul sito sbagliato.

---

## Cosa contiene (e cosa NON contiene)

### ✅ Cosa possiamo ottenere

- **Anagrafica dell'organismo nocivo**: nomi scientifici, sinonimi, nomi comuni multilingua, tassonomia
- **Piante ospiti** (`hosts`): quali specie vegetali un dato organismo può attaccare
- **Organismi nocivi di una pianta** (`pests`): la relazione inversa
- **Presenza geografica**: quali organismi risultano presenti in un dato Paese, con stato e storico
- **Categorizzazione**: stato di quarantena / regolamentazione per Paese
- **Vettori** e **agenti di controllo biologico** (BCA)
- **Reporting Service**: bollettino mensile EPPO sugli eventi di rilevanza fitosanitaria (nuove segnalazioni, specie aliene, focolai)
- Foto, documenti, standard EPPO

### ❌ Cosa NON possiamo ottenere

> [!important] Due limiti da tenere bene a mente **1. Nessuna granularità regionale per l'Italia.** EPPO ragiona a livello di **Paese**. Per l'Italia esistono solo tre entità: _Italia continentale_, _Sardegna_, _Sicilia_ (le isole maggiori sono considerate entità epidemiologiche separate). **La Campania non esiste come unità geografica in EPPO.** → Per il livello regionale/comunale serve il **Servizio Fitosanitario Regionale** (vedi SIMFito).
> 
> **2. `hosts` è una relazione biologica, non geografica.** L'endpoint `hosts` risponde a _"quali piante questo organismo PUÒ attaccare, nel mondo"_, **non** a _"quali colture ci sono in questo territorio"_. Nessun endpoint EPPO dice cosa si coltiva in un dato luogo: quel dato deve arrivare da noi (input dell'agricoltore o mappatura colturale).

---

## L'EPPO code: la chiave primaria

Ogni specie (pianta o organismo nocivo) ha un **EPPO code**: un identificatore alfanumerico stabile e universale.

|EPPO code|Specie|
|---|---|
|`XYLEFA`|_Xylella fastidiosa_|
|`HALYHA`|_Halyomorpha halys_ (cimice asiatica)|
|`CERTCA`|_Ceratitis capitata_ (mosca mediterranea della frutta)|
|`BEMITA`|_Bemisia tabaci_|

> [!tip] Decisione architetturale **L'EPPO code va usato come chiave di join su ogni entità del nostro modello dati** (colture, organismi nocivi, disciplinari, bollettini). È il campo che permette di far combaciare fonti eterogenee — EPPO, bollettini regionali, disciplinari — senza dover fare matching sui nomi (che sono ambigui, tradotti e pieni di sinonimi).

---

## Accesso all'API

### 1. Ottenere la API key

1. Registrarsi su **https://data.eppo.int** (gratuito)
2. Accettare la **Open Data Licence**
3. Generare il token dalla dashboard

> [!note] `data.eppo.int` NON è l'API `data.eppo.int` è solo il **portale** dove si crea l'account e si genera il token, oltre a scaricare i dump del database. Le chiamate API vanno fatte su un dominio diverso: **`api.eppo.int`**.

### 2. Base URL

```
https://api.eppo.int/gd/v2
```

Esiste anche un server di staging (`api2025.eppo.dev:6443`) — **da non usare in produzione**.

### 3. Autenticazione

Il token va passato come **header HTTP**, non come query parameter:

```
X-Api-Key: <il_tuo_token>
```

---

## Limiti e regole

> [!caution] Rate limiting (in vigore dal 22/05/2026)
> 
> - Massimo **2000 richieste per indirizzo IP** in una finestra scorrevole di **10 secondi**
> - Le richieste parallele sono permesse, ma devono rispettare il limite complessivo
> - Il superamento restituisce **HTTP 429** (_Too Many Requests_)

**Licenza**: Open Data Licence (da accettare in fase di registrazione). Va prevista la citazione della fonte nei prodotti derivati.

**Migrazione in corso**: la vecchia API (`data.eppo.int/api/rest/1.0`, autenticazione via `?authtoken=`) è **deprecata** e resta attiva fino al **1° settembre 2026**. Qualsiasi nuova integrazione deve partire direttamente sulla v2 documentata qui. Attenzione a tutorial e librerie di terze parti trovati online (es. il pacchetto R `pestr`): puntano quasi tutti alla versione legacy.

**Health check**: `GET /status` — utile per il monitoring, non richiede autenticazione.

---

## Documentazione ufficiale

La documentazione completa è consultabile dalla dashboard su **https://data.eppo.int** ed è scaricabile come **specifica OpenAPI 3.0 in formato YAML**.

> [!tip] Workflow consigliato per gli sviluppatori
> 
> 1. Scaricare lo spec YAML dalla dashboard EPPO
> 2. In **Postman**: `Import` → trascinare il file `.yaml`
> 3. Postman genera automaticamente una **collection con tutti i 36 endpoint**, già configurati con parametri e schemi
> 4. Impostare la variabile d'ambiente `baseUrl` = `https://api.eppo.int/gd/v2` e l'header `X-Api-Key`
> 
> Da lì si possono provare tutte le chiamate senza scrivere una riga di codice. La collection esportata è il modo più rapido per passare il contesto al team.

---

## Endpoint — esempi

L'API espone **36 endpoint**. Di seguito solo i più rilevanti per il nostro caso d'uso; per l'elenco completo fare riferimento allo spec YAML.

### Dato geografico

```http
GET /country/{ISOCODE}/presence
```

Restituisce tutti gli organismi nocivi registrati in un Paese.

Esempio — `GET /country/IT/presence`:

```json
{
    "state_id": null,
    "eppocode": "ACANOB",
    "prefname": "Acanthoscelides obtectus",
    "peststatus": "X",
    "yr_situation": "2001",
    "yr_introd": null,
    "yr_erad": null
}
```

|Campo|Significato|
|---|---|
|`state_id`|Sotto-unità geografica. **Per l'Italia è quasi sempre `null`**; valorizzato solo per Sardegna e Sicilia|
|`eppocode`|Chiave primaria dell'organismo|
|`prefname`|Nome scientifico preferito|
|`peststatus`|Stato di presenza (vedi tabella sotto)|
|`yr_situation`|Anno a cui si riferisce il dato|
|`yr_introd` / `yr_erad`|Anno di introduzione / di eradicazione|

### Anagrafica dell'organismo

Tutti nella forma `GET /taxons/taxon/{EPPOCODE}/...`

|Endpoint|Restituisce|
|---|---|
|`/overview`|Scheda sintetica|
|`/names`|Nomi scientifici, sinonimi, nomi comuni per lingua|
|`/taxonomy`|Posizione tassonomica|
|`/hosts`|Piante ospiti|
|`/pests`|(input = pianta) Organismi nocivi di quell'ospite|
|`/distribution`|Distribuzione geografica mondiale|
|`/categorization`|Stato di quarantena / regolamentazione|
|`/vectors`|Vettori dell'organismo|

### Risoluzione nomi → codice

```http
GET /tools/name2codes?name=Halyomorpha halys
GET /tools/search?keyword=cimice
```

Servono per agganciare una lista di colture o parassiti (in italiano, o con nomi commerciali) ai rispettivi EPPO code.

### Alert e segnalazioni

```http
GET /reportings/list
GET /reportings/reporting/{reporting_id}
GET /reportings/article/{article_id}
```

Accesso all'**EPPO Reporting Service**, il bollettino mensile su eventi di rilevanza fitosanitaria. È il canale in cui vengono pubblicate le nuove segnalazioni e i ritrovamenti di specie aliene.

### Tabelle di riferimento

Da caricare una volta sola e usare come lookup locali:

```http
GET /references/distributionStatus
GET /references/countries
GET /references/countriesStates
GET /references/qList
GET /references/pestHostClassification
GET /references/vectorClassification
```

---

## Stati di presenza (`peststatus`)

Fonte: `GET /references/distributionStatus`

|Codice|Etichetta EPPO|Traduzione|
|---|---|---|
|`A`|Present, widespread|Presente, diffuso|
|`B`|Present, restricted distribution|Presente, distribuzione ristretta|
|`C`|Present, few occurrences|Presente, poche segnalazioni|
|`X`|Present, no details|Presente, senza dettagli|
|`T`|Transient|Transitorio|
|`D`|Absent, pest no longer present|Assente, non più presente|
|`E`|Absent, pest eradicated|Assente, eradicato|
|`F`|Absent, intercepted only|Assente, solo intercettato|
|`G`|Absent, no pest record|Assente, nessuna segnalazione|
|`H`|Absent, confirmed by survey|Assente, confermato da indagine|
|`J`|Absent, invalid record|Assente, segnalazione non valida|
|`K`|Absent, unreliable record|Assente, segnalazione inattendibile|

> [!warning] Non collassare gli stati in un booleano I 12 codici si raggruppano in tre famiglie — **Presente** (`A`, `B`, `C`, `X`), **Assente** (`D`–`K`), **Transitorio** (`T`) — ma appiattirli su un semplice sì/no fa perdere informazione critica:
> 
> - **`E` (eradicato)** e **`G` (mai segnalato)** hanno lo stesso effetto pratico ma un profilo di rischio molto diverso: dove un organismo è stato eradicato, può rientrare.
> - **`F` (solo intercettato)** è probabilmente lo stato **più prezioso per un sistema di early warning**: significa che l'organismo è stato fermato ai controlli ma non è (ancora) entrato nel territorio.
> - **`J` e `K`** indicano segnalazioni non affidabili: non sono un'assenza accertata, sono un'assenza di dato valido.

---

## Posizionamento nell'architettura

|Livello|Fonte|Tipo di dato|Aggiornamento|
|---|---|---|---|
|**Nazionale / biologico**|**EPPO API**|Anagrafica, ospiti, tassonomia, quarantena, presenza nel Paese, storico|Dato "freddo" — sincronizzazione mensile|
|**Regionale / operativo**|**SIMFito – SFR Campania**|Aree delimitate, buffer, focolai comunali, bollettini, obblighi di lotta|Dato "caldo" — settimanale in stagione|

Il **join tra i due livelli avviene sull'`eppocode`**.

> [!note] Dove sta il nostro valore aggiunto EPPO ci dice che un dato organismo è un problema _per l'Italia_. Il nostro software deve dire all'agricoltore se il _suo_ campo è dentro un'area delimitata. Quel secondo pezzo non esiste in nessuna API — è lì che sta il prodotto.

---

## Alternativa: dump del database

Dalla dashboard `data.eppo.int` sono scaricabili i **dump completi** del database (XML, SQL, JSON, oltre al file SQLite dei codici EPPO).

Per un dato che cambia raramente come l'anagrafica degli organismi nocivi, **il dump periodico è probabilmente più robusto della chiamata API in tempo reale**: evita rate limit, gestione del token in produzione e dipendenza dalla disponibilità del servizio. L'API resterebbe utile per i delta e per il Reporting Service.

→ Da valutare con il team in fase di design.