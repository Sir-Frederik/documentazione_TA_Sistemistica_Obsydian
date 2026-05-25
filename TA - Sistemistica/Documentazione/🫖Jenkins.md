È uno strumento Open Source per l'**automazione del ciclo di sviluppo del software**.
È il più diffuso strumento di **CI/CD** (Continuous Integration / Continuous Delivery).

- **CI**: Ogni **modifica** al codice viene automaticamente **Testata**
- **CD**: Il codice testato viene automaticamente **Distribuito**
Ogni volta che uno sviluppatore modifica il codice, Jenkins può automaticamente scaricare il nuovo codice dal Repository, compilarlo, eseguire test automatici, creare pacchetti o [[Container]], distribuire l'applicazione sui server (**deploy**).

### Pipeline

È la **sequenza di operazioni** automatizzate che Jenkins esegue. Si definisce tramite un file chiamato **Jenkinsfile**.

### Job

È una **singola attività** configurata in Jenkins, ad esempio eseguire i test o fare il deploy.

### Plugin

Jenkins è estendibile tramite **plugin**. Ne esistono migliaia per integrarsi con quasi qualsiasi strumento.