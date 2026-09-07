# ISO 27001 - Registro delle domande aperte

> Elenco dei punti da chiarire con il team prima dell'audit di sorveglianza (Sorveglianza 1, settembre 2026, auditor Pasquale Mele).
> Ogni voce nasce dalla revisione dei documenti SGSI e corrisponde a un segnaposto evidenziato in giallo nel documento di riferimento.

**Ultimo aggiornamento:** 07/09/2026
**Legenda stato:** `[ ]` da chiedere · `[~]` chiesto, in attesa · `[x]` risolto

---

## Come si usa

1. Ogni domanda ha un ID stabile (es. `BC-03`) che non cambia mai, anche quando la voce viene chiusa.
2. Quando una domanda viene risolta, si spunta la casella e si scrive la risposta nella riga `Esito`.
3. Le voci risolte restano nel documento: servono come tracciabilità in sede di audit.
4. Aggiungendo nuovi documenti alla revisione, si crea una nuova sezione con il prefisso del codice documento.

---

## SECBC - Business Continuity

Stato documento: revisionato il 03/09/2026 (rev. 6). Frequenza annuale.

- [ ] **BC-01** · § 1 Scopo · **A chi:** Adriano
  Il perimetro è limitato ai progetti del cliente ASL Napoli 3. Corrisponde ancora allo scope del certificato ISO/IEC 27001:2022 in vigore?
  *Esito:*

- [ ] **BC-02** · § 8 · **A chi:** team infrastruttura
  Manca l'elenco dei software e servizi coperti dalla fornitura DR. La frase si interrompe a metà.
  *Esito:*

- [ ] **BC-03** · § 8.3 · **A chi:** Adriano + team infrastruttura
  I tre livelli di priorità (Tier 3, Tier 1, sola protezione dati) non elencano alcun servizio. È una lacuna sostanziale: un BCP senza classificazione dei servizi non è verificabile. Blocca anche CM-02.
  *Esito:*

- [ ] **BC-04** · § 9.2 · **A chi:** Adriano
  Il documento rimanda all'Allegato 1 "Struttura decisionale" con nomi e contatti, ma l'allegato non esiste.
  *Esito:*

- [ ] **BC-05** · nuovo cap. 11 · **A chi:** Adriano
  Il § 7 annuncia un capitolo su manutenzione, archiviazione e verifica del piano che non era presente. Il titolo è stato inserito, il contenuto è da scrivere. Collegato alle osservazioni dello Stage 2 sulle esercitazioni DR/BCP e alle task CER-27 / CER-28.
  *Esito:*

- [ ] **BC-06** · tabella firme · **A chi:** Adriano
  Data del redattore da confermare (proposta 03/09/2026). Mancano revisore e approvatori.
  *Esito:*

- [ ] **BC-07** · Appendice II · **A chi:** team infrastruttura
  La gestione patch descrive solo Oracle Cloud, mentre il § 8.2 indica OVH come infrastruttura primaria. Manca tutta la parte OVH.
  *Esito:*

- [ ] **BC-08** · Appendice III · **A chi:** team infrastruttura
  Le policy di backup citano strumenti a titolo di esempio (Veeam, Acronis, Commvault, AWS S3, Azure Blob) che non sono la configurazione reale. Vanno sostituiti con quelli effettivamente in uso.
  *Esito:*

- [ ] **BC-09** · Appendice IV · **A chi:** Adriano
  L'appendice contiene la configurazione Disaster Recovery su Oracle, materia di `SECDR_Disaster_Recovery.docx`. Valutare lo spostamento per evitare due documenti che dicono cose diverse.
  *Esito:*

- [ ] **BC-10** · Appendice V · **A chi:** team infrastruttura
  L'analisi settimanale dei log non nomina ELK Stack, che nel master è lo strumento dichiarato. Da collegare, anche perché il logging centralizzato è un'osservazione dello Stage 2.
  *Esito:*

---

## SECSE - Security Encryption

Stato documento: revisionato per intero (frontespizio + corpo). Rev. 3 del 21/07/2026.

- [ ] **SE-01** · tabella Revisioni · **A chi:** Adriano
  Le colonne sono Versione / Data revisione / Note / Autore, mentre il canovaccio usa Agg. / Data / Approvato da / Note. Si uniformano tutti i documenti o si tiene la colonna Autore?
  *Esito:*

- [ ] **SE-02** · piè di pagina · **A chi:** Adriano
  Riporta "Revisione 3 / Pagina N" invece dell'indirizzo Technology Advising e "Doc. Version" come gli altri documenti SGSI.
  *Esito:*

- [ ] **SE-03** · tabella firme · **A chi:** Adriano
  Data del redattore da confermare (proposta 21/07/2026). Mancano revisore e approvatori.
  *Esito:*

- [x] **SE-04** · corpo · **A chi:** nessuno, attività interna
  Revisione del corpo del documento.
  *Esito:* completata il 07/09/2026.

- [ ] **SE-05** · § MySQL, Ulteriori procedure · **A chi:** team sviluppo · **priorità alta**
  Il progetto Brain Assistant usa `aes-128-ecb`. La modalità ECB non nasconde i pattern nei dati cifrati ed è l'unico ambito del documento con chiavi a 128 bit invece di 256. Confermare se corrisponde ancora alla configurazione reale o aggiornare l'algoritmo.
  *Esito:*

- [ ] **SE-06** · § MySQL · **A chi:** team infrastruttura
  Il documento dichiara MySQL 5.7, fuori supporto da ottobre 2023, e la tabella delle funzioni include diverse voci deprecate. Confermare la versione in uso.
  *Esito:*

- [ ] **SE-07** · tutti gli ambiti · **A chi:** Adriano + team infrastruttura
  La rotazione delle chiavi è richiamata in cinque ambiti senza mai indicarne la frequenza, mentre SECBM dichiara "ogni 6 mesi". Fissare una periodicità verificabile e uniformarla tra i documenti.
  *Esito:*

- [ ] **SE-08** · § Tactical RMM · **A chi:** Adriano
  Opportunità di riportare in chiaro il dominio aziendale `talabservices.it` e i nomi dei tre virtual host. Stessa valutazione di BM-06.
  *Esito:*

- [ ] **SE-09** · § Stato di applicazione · **A chi:** team infrastruttura · **priorità alta**
  Il rollout dei controlli crittografici sugli endpoint è dichiarato in completamento "entro settembre 2026", cioè il mese dell'audit. Aggiornare con lo stato effettivo e allegare le evidenze prima della verifica.
  *Esito:*

- [ ] **SE-10** · master riga 17 · **A chi:** Adriano
  La copertina dichiarava frequenza Annuale mentre il master indica Semestrale. Il documento è stato allineato al master (Semestrale): confermare che sia la periodicità corretta.
  *Esito:*

---

## SECBM - Backup Management

Stato documento: revisionato. Sigla cambiata da SECCM a SECBM. Data documento 08/08/2025, **fuori ciclo annuale**.

- [ ] **BM-01** · § 1 Ambito · **A chi:** Adriano · **priorità alta**
  Il documento copre il solo repository SVN, ma `SECCM_Capacity_Management` vi rimanda per le policy di backup di database, indici Elasticsearch e storage. Si allarga l'ambito o si correggono i rimandi? Orientamento attuale: integrare i sistemi in questo documento.
  *Esito:*

- [ ] **BM-02** · § 5.1 · **A chi:** team infrastruttura
  Orario esatto di avvio del backup completo notturno.
  *Esito:*

- [ ] **BM-03** · § 6.1 · **A chi:** team infrastruttura
  Periodi di conservazione mancanti per backup completi, differenziali e mensili su supporti rimovibili. È indicato solo il dato sugli incrementali (7 giorni).
  *Esito:*

- [ ] **BM-04** · § 3 · **A chi:** team infrastruttura
  RPO dichiarato 24 ore ma il § 4.2 prevede incrementali ogni 4 ore. Inoltre "4 ore per progetti critici" non specifica quali siano i progetti critici.
  *Esito:*

- [ ] **BM-05** · § 7.2 e 7.3 · **A chi:** team infrastruttura · **priorità alta**
  I paragrafi sono formulati come azioni da compiere ("sostituzione delle credenziali FTP hardcoded", "eliminazione dell'FTP non crittografato"). Se le attività sono concluse, vanno riscritte al presente come policy in vigore; altrimenti sono non conformità autodichiarate e vanno aperte le azioni correttive.
  *Esito:*

- [ ] **BM-06** · § 1 · **A chi:** Adriano
  Opportunità di riportare in chiaro l'IP del NAS (192.168.1.90) e il path `/media/Archive/svn` in un documento distribuibile.
  *Esito:*

- [ ] **BM-07** · § 8.3 e § 13.2 · **A chi:** team infrastruttura
  Il documento dichiara test settimanali, mensili e annuali più controlli mensili, trimestrali, semestrali e annuali. Verificare che esistano evidenze per ciascuna scadenza, altrimenti ridurre le frequenze dichiarate.
  *Esito:*

- [ ] **BM-08** · tabella Revisioni · **A chi:** Adriano
  Lo storico delle revisioni è completamente vuoto.
  *Esito:*

- [ ] **BM-09** · tabella firme · **A chi:** Adriano
  Le firme riportano 15/01/2021 mentre il documento è datato 08/08/2025. Quale data è corretta?
  *Esito:*

- [ ] **BM-10** · copertina · **A chi:** Adriano
  Frequenza annuale con ultimo aggiornamento 08/08/2025: documento fuori ciclo. Va bumpato prima dell'audit.
  *Esito:*

---

## SECCM - Capacity Management

Stato documento: revisionato. Data 01/09/2025, ciclo annuale in scadenza.

- [ ] **CM-01** · § 4.2 · **A chi:** team sviluppo
  Il documento dichiara l'uso di algoritmi di machine learning per l'analisi delle tendenze. Vengono effettivamente impiegati? In caso contrario va rimosso il riferimento, perché in audit ne verrebbero chieste le evidenze.
  *Esito:*

- [ ] **CM-02** · § 4.3 · **A chi:** Adriano · **dipende da BC-03**
  Il rimando a SECBC e SECDR per RTO/RPO resta privo di riferimento operativo finché la classificazione Tier del Business Continuity non è compilata.
  *Esito:*

- [ ] **CM-03** · § 4.5 · **A chi:** team infrastruttura
  Il documento dichiara report mensili (§ 4.1), analisi trimestrali (§ 4.2), pianificazione semestrale (§ 4.3), test semestrali e annuali (§ 4.5), audit trimestrali e annuali (§ 7). Confermare le evidenze per ciascuna scadenza o ridurre le frequenze.
  *Esito:*

- [ ] **CM-04** · § 5, e anche § 4.4, 7, 8 · **A chi:** Adriano
  La figura "Responsabile Qualità (QUA)" verifica la conformità ISO/IEC 27001. In un documento SGSI il ruolo dovrebbe essere del responsabile SGSI. È la stessa persona con due cappelli o è un residuo del precedente sistema qualità?
  *Esito:*

- [ ] **CM-05** · § 5 · **A chi:** Adriano
  I nominativi sono elencati nel documento. È stato aggiunto il rimando a `SECRU_Ruoli.xlsx`: si rimuovono i nomi da qui, così non serve revisionare il documento a ogni variazione organizzativa?
  *Esito:*

- [ ] **CM-06** · tabella Revisioni · **A chi:** Adriano
  Stesso punto di SE-01: colonne diverse dal canovaccio.
  *Esito:*

- [ ] **CM-07** · copertina · **A chi:** Adriano
  Frequenza annuale con ultimo aggiornamento 01/09/2025: ciclo in scadenza, conviene bumpare prima dell'audit.
  *Esito:*

---

## SECSD - SDLC

Stato documento: revisionato per intero. Rev. 2 del 07/09/2026. Riclassificato da SGQPRO06bis a SECSD.

- [ ] **SD-01** · § 3 · **A chi:** Adriano · **collegato a CM-04**
  La figura "Responsabile Sistema Gestione Qualità (QUA)" è stata riscritta come "Responsabile del SGSI", coerentemente con la natura del documento. Se le due figure restano distinte in azienda, va ripristinata la denominazione originaria.
  *Esito:*

- [ ] **SD-02** · § 4.5 · **A chi:** team sviluppo
  Gli standard di riferimento sono datati: ANSI/IEEE 1074 è stato ritirato e SEI/CMM è superato da CMMI. Valutare l'aggiornamento.
  *Esito:*

- [ ] **SD-03** · § 4.4 · **A chi:** team sviluppo
  L'elenco degli acronimi non definisce "Deep Dive", citato tre volte al capitolo 8 come strumento di analisi del pacchetto, né le sigle dei ruoli introdotte al capitolo 3.
  *Esito:*

- [ ] **SD-04** · § 7 · **A chi:** Adriano
  Il capitolo descrive Scrum come metodologia di sviluppo. Esplicitare come si raccorda con PRINCE2 Agile adottato per la gestione di progetto, così da rendere evidente in audit quale metodologia governa quale ambito.
  *Esito:*

- [ ] **SD-05** · § 8.3 · **A chi:** team sviluppo
  Il documento descrive il solo Subversion, mentre SECCM_Capacity_Management cita GitLab, DockerHub e "SVN legacy" e il master associa il controllo documentale a GIT/SVN. Allineare gli strumenti di versionamento tra i documenti.
  *Esito:*

- [ ] **SD-06** · § 8.3 · **A chi:** Adriano · **stesso punto di TR-03**
  Nel master il documento copre due voci distinte, "Ciclo di sviluppo" (riga 14) e "Release management" (riga 15), ma il rilascio è trattato solo dentro il § 8.3. Valutare se strutturare un capitolo dedicato o separare i due documenti.
  *Esito:*

- [ ] **SD-07** · tabella firme · **A chi:** Adriano
  Le firme sono ferme al 11/01/2021. La revisione 2 non ha redattore, revisore né approvatori.
  *Esito:*

- [x] **SD-08** · tabella VERSIONI PRECEDENTI · **A chi:** nessuno, attività interna
  La riga della revisione 0 è attraversata da una linea di bordo. Difetto grafico da correggere a mano in Word (Layout tabella → Proprietà → Riga, oppure rimuovendo il bordo inferiore della riga di intestazione).
  *Esito:*

- [ ] **SD-09** · frontespizio · **A chi:** nessuno, attività interna
  La riga "Data di aggiornamento: 07/09/2026" è presente nel file ma non si vede: la casella di testo che contiene il frontespizio è troppo bassa e la taglia. Difetto già presente nell'originale.
  *Esito:*

---

## SECSM - Security Update Maintenance

Stato documento: revisionato per intero, incluso il passaggio dell'intero testo dal consuntivo al presente. Data 23/01/2025, **fuori ciclo da quattro periodi**.

- [ ] **SM-01** · copertina · **A chi:** Adriano · **priorità alta**
  Frequenza dichiarata semestrale con ultimo aggiornamento 23/01/2025: sono quattro cicli saltati, ed è il documento più scaduto del set. Va bumpato prima dell'audit.
  *Esito:*

- [ ] **SM-02** · tabella Revisioni · **A chi:** Adriano
  La tabella delle revisioni non esisteva ed è stata creata. Manca il motivo della revisione del 23/01/2025 (riga Agg. 1).
  *Esito:*

- [ ] **SM-03** · tabella firme · **A chi:** Adriano
  Le firme riportano 18-19/01/2021 mentre il documento è datato 23/01/2025. Stessa situazione di BM-09.
  *Esito:*

- [ ] **SM-04** · Premessa · **A chi:** team infrastruttura
  Il perimetro è indicato genericamente come "sistemi in ambiente IaaS". Va sostituito con l'elenco dei sistemi e degli ambienti coperti, coerente con SECBC (OVH primaria, Oracle Cloud per il DR).
  *Esito:*

- [ ] **SM-05** · § 2 e §§ 3-8 · **A chi:** team infrastruttura · **priorità alta**
  Manca la procedura di patching per l'ambiente Windows: dal § 3 al § 8 ci sono solo comandi Linux, e l'unico comando Windows del § 2 (`wmic`) è deprecato nelle versioni recenti. SECSE dichiara endpoint Windows 11 gestiti tramite Tactical RMM.
  *Esito:*

- [ ] **SM-06** · § 9 · **A chi:** team infrastruttura
  L'elenco degli strumenti è generico ("strumenti di orchestrazione", "servizi di gestione del sistema operativo", "soluzioni di monitoraggio e alerting") e non nomina alcun prodotto, mentre il § 4 cita OpenVAS e il § 8 lynis.
  *Esito:*

- [ ] **SM-07** · § 10 · **A chi:** team infrastruttura
  Le tempistiche dichiarate (patch critiche entro 24-48 ore, patch di routine mensili) sono l'impegno più verificabile del documento. Servono le evidenze.
  *Esito:*

- [ ] **SM-08** · nuovo cap. 11 · **A chi:** Adriano
  Il capitolo "Ruoli e responsabilità" è stato creato ma è vuoto. Da definire chi esegue il patching, chi autorizza le patch critiche fuori finestra, chi verifica l'esito e chi conserva le evidenze.
  *Esito:*

- [ ] **SM-09** · nome file · **A chi:** Adriano · **stesso tipo di TR-01**
  Refuso "Maintenence" invece di "Maintenance", sia nel titolo interno sia nel nome file del master (riga 16).
  *Esito:*

---

## Trasversali e master

Riguardano il foglio master (`1ys2dncWVbYua25vPyCl5vbOp7BhdI7K_`) o più documenti insieme.

- [ ] **TR-01** · master riga 18 · **A chi:** Adriano
  Refuso nel nome file: `SECVA_Vulnerabilty_Assessment.docx`. I documenti che lo citano scrivono correttamente "Vulnerability". Va corretto il master o rinominato il file.
  *Esito:*

- [ ] **TR-02** · master riga 12 · **A chi:** Adriano
  Il registro non conformità è marcato "Non presente in quanto non certificati", ma la certificazione è attiva e ci sono due non conformità minori dallo Stage 2 in remediation (task CER-36 → CER-40).
  *Esito:*

- [ ] **TR-03** · master righe 14 e 15 · **A chi:** Adriano
  `SECSD_SDLC.docx` copre sia "Ciclo di sviluppo" sia "Release management". È corretto che un solo documento copra due voci, o vanno separati?
  *Esito:*

- [ ] **TR-04** · rinomina SECBM · **A chi:** nessuno, attività interna
  Dopo il passaggio SECCM → SECBM vanno controllati i riferimenti incrociati in `SECLM_Log_Management`, `SECVM_Version_Management`, `SECDR_Disaster_Recovery` e `SECBC_Business_Continuity`. Il Capacity Management è già stato aggiornato.
  *Esito:*

- [ ] **TR-05** · master righe 41 e 42 · **A chi:** Adriano
  Le voci "Log file management" e "Incidenti informatici" riportano ancora i periodi vecchi (15/09/2025 → 23/09/2025). La mail dell'auditor del 28/07/2026 indica 01/09/2026 → 15/09/2026 per i log e 01/09/2025 → 01/09/2026 per gli incidenti.
  *Esito:*

- [ ] **TR-06** · controlli ISO nei documenti · **A chi:** nessuno, attività interna
  I riferimenti ai controlli della 27001:2013 vanno sostituiti con quelli della 27001:2022 in tutti i documenti ancora da revisionare. Corrispondenze già applicate: A.12.1 → A.8.6, A.17.2 → A.5.30, backup → A.8.13, continuità → A.5.29 / A.5.30, crittografia → A.8.24 / A.5.33 / A.8.5 / A.8.12, sviluppo → A.8.25 / A.8.26 / A.8.27 / A.8.28 / A.8.29 / A.8.31 / A.8.32 / A.8.33, patching → A.8.8 / A.8.19 / A.8.31 / A.8.32.
  *Esito:*

---

## Documenti ancora da revisionare

Da aprire con lo stesso metodo (frontespizio sul canovaccio, poi revisione del corpo):

- [x] `SECRA_RiskAssessment_Piano.docx`
- [ ] `SECDR_Disaster_Recovery.docx`: **di Alessandro**
- [ ] `SECVA_Vulnerabilty_Assessment.docx`
- [ ] `SECVM_Version_Management.docx`
- [ ] `SECUS_Gestione_Utenti.docx`
- [ ] `SECLM_Log_Management.docx`
- [ ] `SECVS_Violazioni_Sicurezza.docx`
