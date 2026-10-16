---
data: 2026-09-28
autore: Federico Brunetti
---


## Scopo e riferimenti

Le due non conformità minori dello Stage 2 non si possono risolvere con l'azione correttiva accettata dall'ente: riguardano i nodi di un cluster Kubernetes gestito da OVH, su cui non abbiamo controllo. Questo documento ricostruisce le verifiche fatte, spiega il vincolo del fornitore e propone un trattamento alternativo.


| Riferimento         | Dettaglio                                                                        |
| ------------------- | -------------------------------------------------------------------------------- |
| Verifica di origine | Stage 2, rif. IT.2025.ISM.W047, 26-27/09/2025, lead assessor Massimiliano Aiello |
| NC1                 | A.8.20, porte 111 e 10250 esposte (Jira CER-36, CER-34)                          |
| NC2                 | A.8.10, TCP e ICMP timestamp attivi (Jira CER-37)                                |
| Task Jira           | CER-29 (risoluzione NC), CER-123 (questo documento), story CER-170               |


## NC1 · A.8.20 · Porte 111 e 10250 esposte

Al *28/09/2026* le porte **111** e **10250** sono ancora raggiungibili da Internet sui tre nodi del cluster, ma sono requisiti obbligatori del servizio OVH e non possono essere chiuse.

**Rilievo.** Le scansioni Nmap e Kube-hunter dello Stage 2 hanno trovato la porta 111/tcp (rpcbind) e la 10250/tcp (Kubelet API) accessibili dalla rete pubblica su tutti gli host. Causa indicata: regole di rete permissive, senza "default deny". Azione correttiva accettata dall'ente: firewall o ACL che limitino le due porte alle sole subnet autorizzate.

**Verifiche.**

|   |   |   |   |
|---|---|---|---|
|Data|Strumento|Host|Esito|
|28/09/2026|nmap da rete esterna|Nodi 51.91.210.225, 51.91.211.52, 51.91.208.205|111 e 10250 aperte su tutti e tre|
|28/09/2026|nmap da rete esterna|VM gitlab 57.128.27.71|111 e 10250 chiuse, nessun servizio in ascolto|
|27/05/2026|OpenVAS "PenTest Cluster"|I tre nodi|111 tcp e udp esposta|

**Perché l'azione correttiva non è applicabile.** I tre host sono nodi di OVH Managed Kubernetes (MKS). Secondo la [documentazione OVH](https://docs.ovhcloud.com/it/guides/public-cloud/containers-orchestration/managed-kubernetes/known-limits), la 10250 serve alla comunicazione tra il control plane e i nodi, la 111 serve ai client NFS, e bloccarle può rompere il cluster. OVH chiede anche di non modificare il security group di default dei nodi. Usiamo NFS sul cluster, quindi entrambe le porte sono necessarie. Confermato da Alessandro il 28/09/2026.

**Trattamento proposto.** Controlli compensativi e accettazione formale del rischio residuo, descritti nelle sezioni seguenti.

## NC2 · A.8.10 · TCP e ICMP timestamp attivi

I timestamp sono stati disattivati sulle VM indipendenti il 28/09/2026, ma sui tre nodi del cluster restano attivi perché il loro sistema operativo è gestito da OVH.

**Rilievo.** Il report OpenVAS dello Stage 2 ha rilevato ICMP Timestamp Reply e TCP Timestamps attivi su tutti i nodi, con esposizione di informazioni di sistema come uptime e scarto dell'orologio. Causa indicata: configurazioni di default mai modificate dopo l'installazione. Trattamento richiesto: aggiornare le linee guida interne di hardening. Azione correttiva accettata: impostare `net.ipv4.tcp_timestamps = 0` dove non necessari.

**Verifiche.**

|   |   |   |   |
|---|---|---|---|
|Data|Strumento|Host|Esito|
|28/09/2026 15:18|nmap -sn -PP da rete esterna|Nodi 51.91.210.225, 51.91.211.52, 51.91.208.205|Tutti e tre rispondono al timestamp|
|28/09/2026 15:12|nmap -sn -PP da rete esterna|VM gitlab 57.128.27.71|Nessuna risposta al timestamp (dopo l'intervento)|
|28/09/2026 14:58|nmap -sn -PP da rete esterna|VM gitlab 57.128.27.71|Risponde al timestamp (prima dell'intervento)|
|27/05/2026|OpenVAS "PenTest Cluster"|I tre nodi|ICMP e TCP timestamp attivi|

**Intervento sulla VM gitlab.** Il parametro `net.ipv4.tcp_timestamps` è passato da 1 a 0, reso permanente con il file `/etc/sysctl.d/99-hardening.conf`. Le richieste e risposte ICMP timestamp (type 13 e 14) sono bloccate con due regole iptables, reinserite a ogni avvio dal servizio systemd `icmp-timestamp-block.service`.

**Perché sui nodi non è applicabile.** MKS è un servizio completamente gestito: non c'è accesso SSH ai nodi e sistema operativo e componenti sono aggiornati da OVH ([documentazione OVH](https://docs.ovhcloud.com/it/guides/public-cloud/containers-orchestration/managed-kubernetes/known-limits)). Nessuno in azienda può quindi modificare questi parametri. Confermato da Alessandro il 28/09/2026.

**Trattamento proposto.** Linee guida di hardening per le macchine che gestiamo direttamente, più accettazione del rischio residuo sui nodi MKS.

## Controlli compensativi proposti

Non potendo chiudere le porte, riduciamo il rischio verificando che chi le raggiunge non possa usarle e controlliamo periodicamente che la situazione non peggiori. Le verifiche sulla Kubelet API e su NFS sono state completate il 29/09/2026.


| Controllo                                                                                                        | Rischio che riduce                                | Chi                    | Stato                                                          |
| ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | ---------------------- | -------------------------------------------------------------- |
| Verificare che la Kubelet API (10250) rifiuti le richieste anonime (risposta 401 o 403)                          | Uso della 10250 da parte di chi non è autenticato | Federico Brunetti      | Verificato il 29/09/2026: "Unauthorized" su tutti e tre i nodi |
| Verificare che le condivisioni NFS siano accessibili solo dalla rete del cluster                                 | Accesso ai dati condivisi tramite la 111          | Genny Scognamiglio     | Confermato da Genny Scognamiglio il 29/09/2026                 |
| Scansione esterna periodica dei nodi (nmap porte e -PP), con confronto rispetto alle sole porte obbligatorie OVH | Apertura di porte nuove non previste              | Federico Brunetti      | Da pianificare                                                 |
| Aggiornamenti di sistema e componenti dei nodi garantiti da OVH                                                  | Vulnerabilità del sistema operativo dei nodi      | OVH (servizio gestito) | Attivo per contratto                                           |

**Esito sulla porta 111 (29/09/2026).** Interrogato con `nmap` e lo script `rpcinfo`, rpcbind su tutti e tre i nodi annuncia solo sé stesso (programma 100000) e nessun altro servizio RPC, nemmeno quelli di NFS: i nodi usano NFS come client e non hanno servizi propri da esporre. Chi raggiunge la porta da Internet ottiene quindi solo l'informazione che rpcbind è attivo.

## Proposta di accettazione del rischio residuo

Si propone di accettare il rischio residuo sui tre nodi MKS, perché la correzione non è tecnicamente possibile senza cambiare servizio. La decisione spetta al Responsabile del SGSI.


| Voce                    | Contenuto                                                                                   |
| ----------------------- | ------------------------------------------------------------------------------------------- |
| Rischio residuo         | Porte 111 e 10250 raggiungibili da Internet e timestamp attivi sui nodi del cluster OVH MKS |
| Motivazione             | Requisiti obbligatori del fornitore; nodi senza accesso al sistema operativo                |
| Misure in essere        | Controlli compensativi della sezione precedente                                             |
| Valutazione del rischio | Da calcolare con il metodo aziendale (Severità × Probabilità × Detection)                   |
| Riesame                 | A ogni cambio di piano o fornitore del cluster, e comunque una volta l'anno                 |
| Approvazione            | Adriano Caligiuri, 29/09/2026                                                               |

Una volta approvato, il rischio va riportato nel registro dei rischi aziendale.

## Linee guida di hardening per le macchine gestite internamente

In azienda non esisteva un documento di hardening: queste regole minime valgono per tutte le macchine Linux che gestiamo direttamente, come le VM OVH, e non per i nodi MKS, gestiti dal fornitore.

1. Disattivare i TCP timestamp con `net.ipv4.tcp_timestamps = 0` in `/etc/sysctl.d/99-hardening.conf`.
2. Bloccare richieste e risposte ICMP timestamp (type 13 e 14), con regole rese permanenti al riavvio.
3. Non lasciare servizi in ascolto se non servono: verificare con `ss -tulpn` e disattivare rpcbind dove non si usa NFS.
4. Attivare un firewall sulla macchina con blocco predefinito in ingresso, aprendo solo le porte necessarie al servizio.
5. Proteggere SSH dai tentativi di accesso ripetuti con fail2ban.
6. Dopo ogni nuova installazione o modifica di rete, ripetere una scansione esterna di porte e timestamp e conservarne l'esito.

Sulla VM gitlab i punti 1, 2, 3 e 5 risultano applicati al 28/09/2026. Il punto 4 no: il firewall ufw è inattivo e la policy in ingresso è ACCEPT, da concordare con Genny prima di intervenire.

## Evidenze e stato


| Evidenza                                                                                                  | Dove si trova                                                                                                                  |
| --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Rapporto Stage 2 (F05_IT.2025.ISM.W047)                                                                   | Tagitlab/quality/docs/Gestione_Non _Conformità                                                                                 |
| Report OpenVAS "PenTest Cluster" del 27/05/2026                                                           | Tagitlab/quality/docs/Gestione_Non _Conformità                                                                                 |
| Output nmap del 28/09/2026 (nodi e VM gitlab) e verifiche del 29/09/2026 (Kubelet API e rpcinfo sui nodi) | Tagitlab/quality/docs/Gestione_Non _Conformità                                                                                 |
| Esiti e dettagli tecnici                                                                                  | Commenti Jira su CER-36 e CER-37                                                                                               |
| Requisiti di porte e gestione dei nodi MKS                                                                | [OVH, Known limits](https://docs.ovhcloud.com/it/guides/public-cloud/containers-orchestration/managed-kubernetes/known-limits) |
