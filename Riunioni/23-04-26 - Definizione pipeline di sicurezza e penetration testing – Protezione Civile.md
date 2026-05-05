## Penetration Testing – Protezione Civile

### Obiettivo

Effettuare attività di **penetration testing** entro 10 giorni, definendo una catena di attività chiara e automatizzata.

---

## Obiettivi Finali

    
1. Analisi dei codici  con **SonarQube**
    
2. Verifica sicurezza pacchetti con **Trivy**
    
3. Controllo infrastruttura e aggiornamento pacchetti dell’[[Container#^113a42 | immagine]] /container
    
4. **Penetration testing** finale con strumenti come [[🛡️OpenVAS (GSAD, GVMD, ospd, Notus Scanner)|OpenVas]]
    

---

## Analisi del codice – SonarQube

SonarQube non è un tool di penetration testing, ma analizza la qualità del codice e individua vulnerabilità di sicurezza. Deve essere conforme agli standard **OWASP**.

### Configurazione

L’attivazione avviene tramite **Jenkinsfile**, dove è già presente del codice commentato. Va abilitato nel processo di build.  
È necessaria una modifica al file pom.xml, aggiungendo la proprietà sonar.java tramite plugin standard.  
Attualmente è attivo solo lato server e deve essere aggiornato all’ultima versione.

### Metriche principali

- Security (priorità massima: deve essere pari a zero)
    
- Reliability
    
- Manutenibilità
    
- Hotspots reviewed
    

### Quality Gate

Deve risultare **passed**.  
La sezione *Overall* Code mostra i problemi generali.  
La sezione *Duplications* evidenzia codice duplicato.  
Il codice non coperto dai test è evidenziato separatamente.

---

## Testing

Si utilizza **JUnit** come framework Java.  
L’obiettivo è raggiungere almeno il **60%** di copertura dei test.

---

## Scan vulnerabilità – Trivy

Trivy viene eseguito da riga di comando e può analizzare immagini Docker e registry.

L’attenzione è sulle vulnerabilità di livello **critical** e **high**.  
Le vulnerabilità sono identificate tramite *codici CVE*.  
È possibile intervenire aggiornando le dipendenze o applicando patch.

---

## Altri strumenti

OpenVAS per il penetration testing.  
**Tool di automazione** e discovery per scansioni di routine.  
**Ansible** per automatizzare operazioni su infrastruttura e deployment.

---

## Ambiente e sicurezza

L’ambiente di testing deve essere isolato, trattandosi di attività potenzialmente distruttive.  
Si può valutare l’utilizzo di namespace e il partizionamento del database per maggiore sicurezza.

---

## Concetti di sicurezza applicativa

Le vulnerabilità di tipo injection consistono nell’inserimento di input malevolo, ad esempio comandi concatenati.  
Se il codice è scritto correttamente, grazie a tipizzazione e validazione degli input, queste situazioni vengono prevenute.  
**SonarQube** è in grado di individuare questo tipo di problemi.

---

## Documentazione richiesta

Va prodotto un documento che descriva:

- Pipeline completa su [[🕴️Jenkins]]
    
- Flusso delle attività
    
- Integrazione tra SonarQube, Trivy e OpenVAS
    
- Linee guida per aggiornamento pacchetti e best practice di sicurezza
    

---

## Organizzazione del lavoro

Le attività devono essere gestite su **Jira** per pianificazione e tracciamento.  
È possibile lavorare senza sprint, ma è necessario mantenere un tracciamento coordinato.

---

## Note operative

I pacchetti buildati devono essere:

1. Scansionati con Trivy
    
2. Verificati a livello infrastrutturale
    
3. Aggiornati se necessario
    
4. Sottoposti a penetration testing
    

---

## Automazione e pipeline

Trivy può essere integrato nel Jenkinsfile.  
Va considerato che, se configurato in modo bloccante, può interrompere la pipeline.

Si può valutare l’esecuzione in modalità non bloccante oppure su job separato.  
L’esecuzione in background è possibile ma va gestita con attenzione, perché Jenkins potrebbe non gestire correttamente i processi separati.

### Quindi? I Prossimi passi da fare:

- Organizzare le attività su **Jira**, assegnando responsabilità e tracciando l’avanzamento.
    
-  Definire un **documento** in cui si ipotizza la **pipeline** su Jenkins, includendo le fasi di build, analisi con SonarQube, scan con Trivy e test finali. Fare i test sul pacchetto in GIt del **mesit - validazione Dati**
    
- Completare la configurazione di SonarQube e verificare il corretto funzionamento del quality gate.
    
- **Integrare Trivy** nel processo, stabilendo modalità e criteri di esecuzione.
    
- Predisporre **un ambiente isolato** per i test, definendo le modalità di separazione e protezione dei dati.
    
- Verificare e **aggiornare** le immagini e le dipendenze prima dell’esecuzione dei test.
    
- Definire l’approccio al **penetration testing**, inclusi strumenti e ambito delle verifiche.
    
- Cercare di **automatizzare tutto**, affinché uno script invochi automaticamente Trivy e cerchi le vulnerabilità e invii un'email a chi ha buildato. Deve effettuare anche dei check per verificare che le criticità siano state sistemate.

