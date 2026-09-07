

## Il quadro vero

- Il driver non è solo PRINCE2 ma la **ISO 27001**. I documenti attuali (piano di risk assessment + policy) esistono dall'anno scorso e sono stati fatti per accontentare il certificatore.
- Obiettivo di Adriano quest'anno: **usare davvero il modello di gestione del rischio**, non produrre carta. "A me non interessa che abbiamo il documento per la 27001, a me interessa che iniziamo a gestire i rischi."
- Restano comunque **due documenti**: (1) come gestiamo il rischio, (2) il modello di matrice / regole di valutazione. Il documento fisso descrive le regole, Jira è dove i rischi vivono.

## Decisioni prese

### Pulizia dei rischi esistenti

- Molti ticket attuali **non sono rischi**: "accesso tramite credenziali compromesse", "errata configurazione" sono errori di sviluppo o fatti infrastrutturali, non rischi di progetto.
- Un rischio vero: probabilità sensata + impatto sul progetto. Esempio buono: know-how concentrato su una sola persona (impatto alto, probabilità bassa, rischio medio, mitigazione = knowledge transfer).
- Primo passo: **cernita dei rischi veri**, progetto per progetto. La faccio io con i gruppi di lavoro.

### Struttura su Jira

- Epic = contenitore rischi. **Story = rischio. Task/subtask = azioni di mitigazione** (assegnabili come attività).
- Adriano NON vuole toccare gli stereotipi (issue type) esistenti: li usano tutti i gruppi. "Se sfondiamo qualcosa passiamo un guaio."
- Strada preferita: creare un **nuovo tipo di ticket "Rischio"** che si comporta esattamente come una Story ma si chiama Rischio e ha i campi in più. Se esiste già un tipo simile, riusarlo.
- I campi vanno aggiunti **per tipo di ticket, non per progetto**.
- Piano B se non fattibile: matrice nella descrizione o etichette + riga su Excel.

### Matrice del rischio

- Adriano vuole **semplificarla molto** rispetto a quella nei documenti attuali.
- Parametri che contano: **severità, probabilità, detection**.
- Detection: un rischio basso ma difficile da identificare sale di livello; uno facilmente identificabile può scendere.
- Il punteggio finisce nel campo **Priority** di Jira (o Priority Risk): severità e probabilità generano il risk score, il risk score ordina il backlog.
- Manca una colonna **data censimento del rischio**.

### Processo

- Censimento continuo: i rischi emergono durante il progetto, non solo all'inizio.
- **Riunione del lunedì**: il team censisce i rischi emergenti. Attività ricorrente.
- Quando un rischio si materializza: si segna la data e **passa alla gestione issue**. Lo storico delle date è prezioso.
- Per il certificatore serve poter **estrarre documenti da Jira** (export). Lo stavo già studiando, ne parlo con Rita.
- Il registro rischi attuale sono fogli Excel; l'attuazione del monitoraggio sta dentro Jira con le attività.

### Esempio di rischio emerso citato da Adriano

- Progetto Maiesit: rischio che vengano chieste attività di configurazione durante il periodo di chiusura aziendale (riunione infrastrutture di domani). Va censito su Jira.

## Cose da chiarire con Rita 

- [ ] Si può creare un tipo di ticket "Rischio" identico alla Story come comportamento, con campi aggiuntivi, senza toccare i tipi esistenti?
- [ ] In alternativa esiste già un tipo riusabile?
- [ ] Campi da aggiungere sul tipo: severità, probabilità, detection (+ data censimento)
- [ ] Rita ha accennato a un vincolo: si può aggiungere / togliere qualcosa ma non cambiare la denominazione. Audio pessimo, da farsi rispiegare
- [ ] Export/report da Jira in formato documento per il certificatore
- [ ] Il campo Priority esistente basta per il risk score o serve un campo dedicato?

## Azioni

- [ ] **Adriano**: manda domani una checklist del lavoro sui documenti
- [ ] **Adriano**: fa una modifica di base ai documenti, poi io li sistemo
- [ ] **Federico**: call con Rita domani mattina (punti sopra)
- [ ] **Federico**: cernita dei rischi veri con i gruppi di lavoro
- [ ] **Federico**: preparare slide "cos'è un rischio / cosa non lo è" per i gruppi
- [ ] **Federico**: verificare export documentale da Jira
- [ ] Da istituire: riunione ricorrente del lunedì per censimento rischi

## Note a margine

- La 27001 potrebbe richiedere anche un **documento verbalizzato con versioni datate** (stato del registro alla data X). Da tenere presente per l'export da Jira: le versioni datate servono all'audit.
- Adriano interviene lui direttamente sui documenti base; il mio lavoro parte dopo, sulla sistemazione.