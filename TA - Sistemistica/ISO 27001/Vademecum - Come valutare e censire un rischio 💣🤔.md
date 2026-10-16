---
tags:
  - iso27001
  - risk-assessment
  - jira
  - prince2agile
  - reference
documento-fonte: SECRA_RiskAssessment_Policy
aggiornato: 2026-10-01
nota: Per gli Owner dei progetti
autore: Federico Brunetti
---


> Promemoria operativo tratto dal documento aziendale  *SECRA Risk Assessment Policy.*
> Puoi trovare la guida ufficiale qui: https://wikita.talabservices.it/it/procedure/gestione-rischi


---

## Prima domanda: è davvero un rischio? 🤔

Un rischio è un **evento futuro e incerto** che, se accadesse, danneggerebbe il progetto o la sicurezza delle informazioni.

Servono tutte e tre le condizioni:

1. **Incertezza** → non è ancora successo e non è detto che succeda
2. **Probabilità plausibile** → può concretamente accadere *su questo progetto*
3. **Impatto** → se accade, colpisce tempi, costi, qualità, continuità o conformità

### Non sono rischi ❌

| Cosa | Perché no | Dove va |
|---|---|---|
| Configurazione errata già presente | è un fatto accertato, non un evento incerto | gestione difetti |
| Caratteristica nota dell'infrastruttura | è una condizione stabile, non un evento | può essere la *causa* di un rischio |
| Attività pianificata | è un elemento di piano | il rischio semmai è che non venga completata |
| Evento già accaduto | non è più incerto | diventa una **issue** |
| Formulazione generica | non è valutabile | riformulare o scartare |

### Come si formula bene ✔️

Sempre **causa → evento → effetto**.

> "La conoscenza operativa di un sistema critico è concentrata su una sola risorsa; in caso di sua indisponibilità prolungata, gli interventi correttivi subirebbero ritardi significativi."

Se non riesci a scrivere la frase in questa forma, probabilmente non è un rischio.

---

## La sequenza di valutazione 

1. Formulare il rischio (causa, evento, effetto)
2. Determinare la **Severità**
3. Determinare la **Probabilità**
4. Ricavare il **Punteggio di rischio** (matrice 1)
5. Determinare la **Detection**
6. Ricavare la **Priorità** (matrice 2) → va nel campo Priorità del ticket
7. Scegliere il **trattamento** e creare i sottotask

> [!tip] In caso di dubbio tra due livelli contigui, si sceglie il più cautelativo.

---

## Le scale 🪜

### Severità (quanto fa male) ☠️

- **Alta** → impatto su riservatezza/integrità/disponibilità, violazione normativa o contrattuale, interruzione servizi ai clienti, danno reputazionale
- **Media** → impatto significativo ma circoscritto su tempi/costi/qualità, degrado prestazioni senza interruzione, interventi correttivi non pianificati
- **Bassa** → impatto marginale, assorbibile nelle normali attività

### Probabilità (quanto è verosimile) 🎲

- **Alta** → atteso con ragionevole certezza: soluzioni non consolidate, molti punti di integrazione, dipendenze esterne non governate, nessun controllo specifico
- **Media** → può capitare occasionalmente: intervento manuale, configurazioni non standard, complessità apprezzabile
- **Bassa** → improbabile: soluzione consolidata, usata senza modifiche, con controlli già attivi

### Detection (quanto presto ce ne accorgiamo) 🚨

> [!warning] Verso della scala
> Misura la **capacità di accorgersene**. `Immediata` è la situazione **migliore**, `Nessuna` la **peggiore**.
> Non confondere con "alta difficoltà di rilevamento".

- **Immediata** → allarme automatico, nessun intervento umano
- **Alta** → rilevato al primo utilizzo della funzione, controlli automatici
- **Media** → individuabile ma non subito, emerge quando la funzione viene usata
- **Bassa** → difficile da vedere in anticipo, dipende da verifiche manuali non sistematiche
- **Nessuna** → si scopre solo a danno avvenuto o per segnalazione esterna

---

## Matrice 1: Punteggio di rischio 🏆

==Severità × Probabilità==

| **Punteggio Rischio** | Prob. Bassa | Prob. Media | Prob. Alta |
| --------------------- | ----------- | ----------- | ---------- |
| **Severità Alta**     | 2           | 3           | 3          |
| **Severità Media**    | 1           | 2           | 3          |
| **Severità Bassa**    | 1           | 1           | 2          |

**3 = peggiore**, 1 = migliore. Il numero cresce insieme alla gravità.

**Scorciatoia per non guardare la tabella:** *assegna Bassa=1, Media=2, Alta=3 e somma*:
Se ==il valore è uguale a 4==, allora il rischio ha un ==punteggio 2==.

Negli altri due casi,
Se il valore è minore di 4, il punteggio è 1
Se il valore è maggiore di 4, il punteggio è 3

| Somma del valore Prob + Severità | Punteggio Rischio |
| -------------------------------- | ----------------- |
| <4                               | 1                 |
| 4                                | 2                 |
| 4>                               | 3                 |

*Esempio:* Severità Alta (3) + Probabilità Bassa (1) = somma 4 → **punteggio 2**.

---

## Matrice 2: Priorità del rischio

**==Punteggio × Detection==** → **Priorità**

|                 | Immediata | Alta  | Media | Bassa | Nessuna |     |
| --------------- | --------- | ----- | ----- | ----- | ------- | --- |
| **Punteggio 1** | Bassa     | Bassa | Bassa | Media | Media   |     |
| **Punteggio 2** | Bassa     | Bassa | Media | Alta  | Alta    |     |
| **Punteggio 3** | Media     | Media | Alta  | Alta  | Alta    |     |

### Esempio di un rischio 🧮
Probabilità: Media (2)
Severità: Alta (3) 
Punteggio rischio:  2 +3 = 5.  % è maggiore di 5, quindi il punteggio è **3**,
Detection: Alta 
**Priorità:**  Punteggio x Detection = 3 x Alta = **MEDIA**

![[Pasted image 20261001130640.png|82]]



---

## Trattamento

- **Riduzione** → controlli che abbassano severità e/o probabilità *(opzione ordinaria)*
- **Trasferimento** → assicurazione, clausole contrattuali
- **Evitamento** → si elimina la causa cambiando approccio
- **Accettazione** → si decide di non agire, mantenendo il monitoraggio

> [!important] Un rischio con trattamento diverso da "accettazione" e **senza sottotask associati** è da considerarsi **non trattato**.

### Soglia di tolleranza

| Priorità | Chi decide |
|---|---|
| Bassa | accettabile senza azioni, solo monitoraggio |
| Media | accettabile con consenso esplicito del Risk Owner, motivazione nel ticket |
| Alta | **supera la soglia**, trattamento obbligatorio, accettazione solo con autorizzazione della Direzione |

Mai accettabili, a prescindere dalla priorità: violazioni normative o contrattuali, compromissione dei dati personali, interruzione dei servizi ai clienti.

> L'accettazione **è** una decisione di trattamento e va registrata. Serve a distinguere un rischio accettato consapevolmente da uno mai valutato.

---

## Come Censire su Jira

- **Epic contenitore** per progetto (uno per i rischi, uno per le issue)
- **Ticket tipo Rischio** → la chiave del ticket è l'ID rischio
- **Sottotask** → una per ogni azione di mitigazione, con assegnatario e stima

### Campi  della task da compilare 

 `Probabilità` · `Severità`  · `Detection` · `Data censimento` · `Priorità`


![[Pasted image 20261001125854.png]]


La **Data censimento** si mette al momento dell'identificazione e **non si tocca più**. Serve a ricostruire l'evoluzione del registro nel tempo.

### Ruoli

- **Risk Owner** = ISMS Manager, unico per l'organizzazione. Risponde della valutazione e delle decisioni.
- **Risk actionee** = assegnatario del sottotask. Esegue, non risponde del rischio.

---

## Quando un rischio si materializza

1. Registrare la data di accadimento
2. La gestione prosegue come **issue**
3. Mantenere il collegamento con il rischio di origine

Il confronto tra rischi censiti e issue effettivamente accadute misura quanto bene stiamo valutando.

---

## Cadenza

**Riunione del lunedì**, con i gruppi di lavoro. Si verificano:

- rischi di nuova identificazione
- avanzamento delle mitigazioni in corso
- variazioni di valutazione su rischi già censiti
- rischi materializzati → diventati issue
- rischi non più pertinenti → chiusi con motivazione

Riesame straordinario in caso di: incidenti di sicurezza, modifiche architetturali o organizzative rilevanti, variazioni normative, audit.

---
