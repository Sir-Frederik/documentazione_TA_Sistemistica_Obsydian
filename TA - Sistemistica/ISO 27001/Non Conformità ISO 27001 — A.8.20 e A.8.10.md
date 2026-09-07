

Nota di tracciamento: da dove nascono i ticket Jira CER-36 e CER-37, e stato della remediation. Ultimo aggiornamento: 24/08/2026.

## Fonte del rilievo

Entrambe le Non Conformità provengono dallo stesso documento:

- **Rapporto di audit**: `F05_IT.2025.ISM.W047 TECHN ADV Stg 2 2025-signed.pdf` (Verifica di Stage 2).
- **Riferimento pratica**: `IT.2025.ISM.W047`.
- **Ente / auditor**: Lead Assessor **Massimiliano Aiello**. I rapporti controfirmati sono stati trasmessi tramite **Resource Consulting** (Dott. Ing. Pasquale Mele).
- **Date verifica**: 26-27.09.2025.
- **Perimetro dichiarato nel report**: solo la sede fisica (Technology Advising S.r.l., Centro Direzionale Isola G1, 80143 Napoli). Il report **non elenca IP né hostname** dei sistemi affetti.

Percorso di arrivo dell'informazione: Resource Consulting → Carmela Ceraudo (03/10/2025) → inoltrata da Adriano Caligiuri (CTO) a Federico Brunetti il 21/07/2026 ("fyi"), con in copia Genny Scognamiglio e Alessandro Crasto. I ticket Jira sono stati poi creati manualmente da Federico (non assegnati da terzi).

Nota formale: l'intestazione del report cita "ISO IEC 27001:2013", ma i controlli sono numerati secondo l'Annex A del 2022 (A.8.20, A.8.10). Incongruenza del documento, da chiarire con l'ente solo se emergesse in sorveglianza.

## Le due Non Conformità (stessa area di rete, stesso scan)

### NC #1 — A.8.20 (Sicurezza delle reti) → ticket CER-36

Testo originale del rilievo:

> Non risultano adeguati controlli per limitare accesso a servizi critici da reti non autorizzate (Scan Nmap e Kube-hunter: porta 111/tcp (rpcbind) e porta 10250/tcp (Kubelet API) accessibili da rete pubblica su tutti gli host).

- Classificazione: Non Conformità minore.
- Causa: configurazione di rete con regole permissive (assenza di default deny).
- Azione correttiva accettata: configurare firewall/ACL per consentire accesso alle porte 111 e 10250 solo da subnet autorizzate.

### NC #2 — A.8.10 → ticket CER-37

> Esposizione critica di informazioni di sistema (uptime, clock skew) — Report OpenVAS: ICMP Timestamp Reply e TCP Timestamps attivi su tutti i nodi.

- Classificazione: Non Conformità minore.
- Azione correttiva accettata: aggiornare le linee guida interne di hardening per disabilitare i protocolli/opzioni che espongono queste informazioni.

Nota: il ticket **CER-34** ("A.8.20 - Hardening accesso rete") risulta un sottotask gemello vuoto e non assegnato, verosimilmente ridondante rispetto a CER-36. Da verificare ed eventualmente unire per non duplicare il lavoro.

## Verifica prossimo audit

Tutte accettate dall'ente, **da verificare nel prossimo audit (Sorveglianza 1, 09/2026)**.

## Ambito tecnico e approccio alla remediation

- Le porte appartengono a due mondi distinti: **111/tcp (rpcbind)** tipico di host Linux con NFS; **10250/tcp (Kubelet API)** presente solo sui nodi Kubernetes.
- L'infrastruttura coinvolta è mista: **alcuni nodi Kubernetes, altre VM su OVH cloud**. Non è la flotta di endpoint Windows gestita da Tactical RMM — **TRMM non ha ruolo in questa remediation**.
- Su Kubernetes la 10250 va bloccata a livello di **firewall di nodo / security group**, non con NetworkPolicy K8s (che non intercetta le chiamate dirette all'IP del nodo). Su VM OVH dipende dal firewall OVH disponibile o da iptables/ufw locale.

### Passi operativi

1. **Costruire l'inventario dei bersagli**: raccogliere gli IP pubblici dei sistemi in perimetro dal pannello OVH (VM) e dagli IP pubblici dei nodi Kubernetes → `targets.txt` (un IP per riga). Il report di audit non fornisce questa lista, va ricostruita.
2. **Scan "before"** da postazione esterna (per riprodurre la vista pubblica dell'auditor): `nmap -Pn -p 111,10250 -oA nmap_before_A820 -iL targets.txt`
3. Applicare firewall/ACL per limitare 111 e 10250 alle sole subnet autorizzate (metodo per host secondo l'ambiente: security group/firewall di nodo per K8s; firewall OVH o iptables per le VM).
4. **Scan "after"** con lo stesso comando (`-oA nmap_after_A820`) come evidenza di chiusura, da allegare al ticket.
5. Affrontare A.8.10 (CER-37) sullo stesso giro di macchine, essendo gli stessi host/nodi.

## Stato

- CER-36 (A.8.20): **Da fare**. Inventario host ancora da costruire (scan Nmap non ancora eseguito alla data di questa nota).
- CER-37 (A.8.10): **Da fare**.