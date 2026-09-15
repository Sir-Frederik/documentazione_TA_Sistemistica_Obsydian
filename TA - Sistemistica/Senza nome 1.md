# SECBC - Business Continuity

Ecco tutte le domande aperte sul Business Continuity, numerate. Rispondi come ti viene, anche solo con una riga per punto, e alla fine revisiono il documento.

Il file che mi hai caricato è già la revisione 6, quella con i quattro segnaposto gialli e il capitolo 11 vuoto.

---

#### Perimetro e scope

**BC-01** · § 1 Scopo  
Il documento limita tutto ai progetti del cliente ASL Napoli 3, ma il certificato copre consulenza e assistenza informatica più progettazione e sviluppo software, su tre siti (Isola G1, Isola C2, San Teodoro SS).  
👉 _Come riscrivo lo scopo? Includo tutti e tre i siti o il piano copre solo la sede principale?
**Non si può far sì che il documento riguardi ogni progetto di ogni cliente? bisogna per forza scrivere perogni progetto di ogni sede operativa?**

**BC-11** · nuovo, dal report di audit  
Il sito di **San Teodoro (SS)** non compare da nessuna parte nel documento.  
👉 _Che ruolo ha rispetto alla continuità operativa? Ha infrastruttura propria, è solo postazioni di lavoro, o è fuori dal piano?
**il sito di san teodoro è stato dismesso**

---

### Classificazione dei servizi

**BC-03** · § 8.3 · **il punto più importante**  
I tre livelli (Tier 3 = essenziali con RTO 3 giorni, Tier 1 = importanti con RTO 20 giorni, sola protezione dati) non elencano nessun servizio.  
👉 _Quali servizi stanno in ciascun livello? Anche un elenco grezzo va bene, poi lo formatto io.
**Priorità uno: protezione civile, priorità due Asl napoli 3 e priorità tre tutto il resto**


**BC-02** · § 8  
La frase "per quanto riguarda tutti i servizi con fornitura DR ed in particolare per i software:" si interrompe.  
👉 _Quali software e servizi sono coperti dalla fornitura DR?_
**non fare menzione ai software.**

---

### Infrastruttura

**BC-07** · Appendice II  
La gestione patch descrive solo Oracle Cloud, ma il § 8.2 dice che la primaria è OVH.  
👉 _Come funziona il patching su OVH? Posso riusare quanto scritto nel SECSM, oppure è diverso?_
**Parla solo di oracle cloud e non parlare di Ovh**

**BC-08** · Appendice III  
Le policy di backup citano Veeam, Acronis, Commvault, AWS S3, Azure Blob come esempi generici.  
👉 _Cosa usate davvero? Se il backup è quello descritto nel SECBM, rimando lì invece di riscriverlo._
**rimuovi i riferimenti ai programmi**

**BC-09** · Appendice IV  
L'appendice contiene tutta la configurazione Disaster Recovery su Oracle, che è materia del SECDR di Alessandro.  
👉 _La sposto nel SECDR e qui lascio un rimando, oppure resta?_
**Rimanda al documento di alessandro, penso non ci sia bisogno nemmeno di avereuna appendice,ma diretamente un punto del documento che riporta al documento**

**BC-10** · Appendice V  
L'analisi settimanale dei log non nomina ELK Stack, che è lo strumento dichiarato nel master e descritto nel SECLM.  
👉 _Sostituisco la descrizione con un rimando al SECLM?
**eLK STACK è IMPLICITO**


---

### Struttura decisionale

**BC-04** · § 9.2  
Il documento rimanda all'"Allegato 1 - Struttura decisionale" con nomi e contatti, ma l'allegato non esiste.  
👉 _Chi compone la struttura decisionale? Mi servono i ruoli, i nominativi e come si raggiungono in emergenza. In alternativa rimando a SECRU_Ruoli.xlsx e lascio solo i ruoli._

Crisis manager è Genny Scognamiglio

Disaster recovery Alessandro

Resp comunicazione marco Giorgino

referentte sicureffa fisica Marco Caligiuri

responsabile del personale Laura del Pezzo

Responsabile finanziario Nunzia Cappabianca

---

### Il capitolo 11, quello vuoto

**BC-05** · nuovo capitolo 11 "Manutenzione, archiviazione e verifica del piano"  
Il § 7 lo annunciava ma non c'era. Ho messo il titolo, il contenuto è da scrivere. È anche l'**osservazione 5 del report di Aiello** (punto norma 8.1, esercitazioni DR/BCP e registrazione evidenze).

Mi servono quattro cose:

- 👉 _Ogni quanto si riesamina il piano?_
- 👉 _Ogni quanto si fanno le esercitazioni, e di che tipo (table-top, prova di ripristino reale, entrambe)?_
- 👉 _Chi redige il verbale e dove si archivia?_
- 👉 _Dove si conserva il piano e chi ne detiene le copie, anche offline?_
Rimuovi il riferimento a Manutenzione, archiviazione e verifica del piano"   e rimuovi il capitolo 11
---

### Firme e formalità

**BC-06** · tabella firme  
Ho proposto 03/09/2026 come data del redattore, evidenziata in giallo.  
👉 _Confermi la data? E chi mette come revisore e approvatori?_
|   |   |   |   |
|---|---|---|---|
|Redatto da:|Adriano Caligiuri|03/08/2026||
|Revisionato da:|Federico Brunetti|1/09/2026||

**BC-12** · nuovo  
Nella copertina manca la riga "Procedura:" che c'è negli altri documenti. Questo lo correggo comunque, non serve risposta.

---

### Due punti che emergono dal confronto con gli altri documenti

**BC-13** · § 10.1  
Avevo sostituito il riferimento ad A2A con "il personale reperibile di Technology Advising".  
👉 _Esiste davvero una reperibilità? Se sì, come è organizzata? Se no, riscrivo diversamente._
**E' ok**

**BC-14** · terminologia  
Il documento alterna "CED primario" e "Hosting Primario" per la stessa cosa.  
👉 _Quale dei due uso?_
**Hosting Primario*
