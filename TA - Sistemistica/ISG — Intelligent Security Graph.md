

**ISG** è un servizio cloud Microsoft che assegna un punteggio di reputazione agli eseguibili basandosi su telemetria aggregata da miliardi di endpoint Windows nel mondo. Non è un antivirus — non analizza il comportamento del file in isolamento, ma valuta quanto è diffuso e con quale storico di esecuzioni sicure.

---

## Come funziona

Ogni file eseguibile ha una firma digitale e un hash. Quando un eseguibile viene avviato, Windows invia l'hash al servizio ISG e riceve una risposta: reputazione alta, bassa o sconosciuta. La valutazione si basa principalmente su:

- **Diffusione** — quante macchine nel mondo hanno eseguito quel file
- **Firma** — se il file è firmato da un publisher noto e attendibile
- **Storico** — se quel file è stato associato a comportamenti malevoli

---

## Relazione con [[📘Guida  personale a TRMM#WDAC — Windows Defender Application Control|WDAC]]

In una policy WDAC standard, tutto ciò che non è esplicitamente in whitelist viene bloccato. Abilitando la regola `Enabled:Intelligent Security Graph Authorization`, si aggiunge un terzo canale di autorizzazione: un file può passare se ISG lo considera sufficientemente affidabile, anche senza essere in whitelist.

Questo risolve un problema pratico: i driver e i componenti di sistema di Windows sono troppi e troppo variabili tra versioni per poterli coprire tutti esplicitamente in una policy. Senza ISG, ogni aggiornamento di Windows o driver rischia di causare un boot failure perché un file legittimo non è in whitelist.

---

## Il trade-off

Il vantaggio — nessun boot failure per driver di sistema non coperti — ha un costo: software commerciale legittimo, firmato e molto diffuso (Firefox, VLC, Sumatra PDF, ecc.) passa automaticamente anche senza essere esplicitamente autorizzato. ISG non distingue tra "software che voglio far girare" e "software che mi fido non sia malware" — se un file è reputato sicuro, entra.

Per contesti con requisiti di controllo molto stretti questo è inaccettabile. Per il nostro caso — flotta aziendale con utenti standard, WDAC come layer principale su una difesa in profondità — è un compromesso accettabile.

---

## ISG non è adatto quando

- Si vuole un controllo assoluto su ogni singolo eseguibile autorizzato (es. ambienti regolamentati, air-gapped)
- Gli endpoint non hanno accesso a internet (ISG richiede connettività al cloud Microsoft)
- Si gestisce software molto di nicchia o sviluppato internamente — ISG potrebbe non avere reputazione su di esso e bloccarlo comunque

---

## Riferimento nel progetto

Nella policy WDAC di Technology Advising, ISG è abilitato dalla versione iniziale. La scelta è documentata nella [[📘Guida  personale a TRMM]] come trade-off accettato.