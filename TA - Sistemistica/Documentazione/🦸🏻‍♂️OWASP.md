La *Open Web Application Security Project*  è una ==fondazione no Profit ==completamente aperta e **gratuita**.
Produce **Documentazioni, strumenti e standard** che chiunque può usare.

Il suo **obiettivo** è ==rendere il software più **sicuro**==, mettendo a disposizione della collettività delle conoscenze che prima erano elitarie.

## I Prodotti Principali
#### OWASP Top 10
E' una **==lista==** aggiornata ogni 3-4 anni delle ==10 vulnerabilità== più critiche e diffuse nelle web application ed è un **riferimento universale**.

L' ultima versione è del *2021* e include vulnerabilità come:
- **Broken Access Control**: un utente accede ai dati che non dovrebbe vedere
- **Injection**: in SQL o Comandi, viene iniettato codice malevolo.
- **Cryptographic Failures**: dati sensibili cifrati male o affatto.
- **Security Misconfiguration**: server configurati con impostazioni di default insicure.
  
Esiste anche una versione dedicata alle **API**, la ==**OWASP API Top 10**== che è quella che noi usiamo con [[ASTF]] E [[WuppieFuzz]] nel server di test.

#### OWASP Testing Guide
E' una **==Guida==** che spiega metodologie e tecniche per testare manualmente  la sicurezza di un'applicazione.
E' il manuale di ==**riferimento** per i **penetration tester**.==


#### OWASP ASVS
Acronimo di *Application Security Verification Standard* è un **==Lista di Requisiti==** di **sicurezza** che un'applicazione dovrebbe soddisfare.
E' usata spesso come **==checklist==** durante gli **audit**

#### Strumenti Open Source
OWASP sviluppa e mantiene anche **tool** pratici, come ==[[ZAP]]== (*Zed Attack Proxy*) che è uno dei  **DAST** (*Dynamic Application Security Testing*) più usati al mondo.



## Nel Nostro Progetto
La pipeline OWASP gira sul server `ubuntu@ta-server-openvas` (`51.75.196.150`) e copre tutti i [[Tipi di Analisi di Sicurezza]] — dal codice sorgente fino al runtime.


| Pagina                                | Contenuto                                                            |
| ------------------------------------- | -------------------------------------------------------------------- |
| [[Script di Routine OWASP]]           | I 6 script operativi in sequenza — da usare ad ogni sessione di test |
| [[config.env — Guida alle Variabili]] | Tutte le variabili configurabili per cambiare target e credenziali   |
| [[Analisi AI con Ollama]]             | Invio dei report a Ollama via VPN WireGuard per analisi automatica   |
| [[📎OWASP — Troubleshooting]]           | Problemi noti con causa e soluzione                                  |

---

