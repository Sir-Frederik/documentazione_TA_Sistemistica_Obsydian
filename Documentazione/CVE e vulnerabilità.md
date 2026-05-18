Vulnerabilità ed Esposizioni Comuni. E' un catalogo pubblico di vulnerabilità di sicurezza delle informazioni gestito dalla MITRE Corp.
E' simile ad un **dizionario**, con un nome ed una descrizione.

Una **vulnerabilità** è una debolezza logica computazione presente nei software o hardware che se sfruttata potrebbe creare dei danni.

Quando [[Dependency-Check]] trova una vulnerabilità, la riporta con un **codice CVE** tipo *CVE-2023-12345*.
E' un identificatore univoco universale per ogni vulnerabilità nota.

Ogni CVE ha un ==punteggio **CVSS**== (*Common Vulnerability Scoring System*) che indica la gravità:

| Punteggio  | Gravità  |
| ---------- | -------- |
| 0.0        | None     |
| 0.1 – 3.9  | Low      |
| 4.0 – 6.9  | Medium   |
| 7.0 – 8.9  | High     |
| 9.0 – 10.0 | Critical |
Nel nostro `config.env` di `TA-Server Open Vas` c'è la variabile `CVSS_THRESHOLD` impostata a `7.0`. Quindi la pipeline ci segnala solo le vulnerabilità più pericolose.
##### Approfondimenti:
https://www.ibm.com/it-it/think/topics/cve


