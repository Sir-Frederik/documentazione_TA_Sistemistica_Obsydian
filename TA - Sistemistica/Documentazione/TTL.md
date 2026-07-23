Il **TTL** (_Time To Live_) è un numero, in ==secondi==, attaccato a ogni record DNS. Dice ai resolver di tutto il mondo: _"questa risposta puoi tenerla in cache per X secondi — poi buttala e richiedimela"_.

Si vede nella seconda colonna dell'output di `dig`:

```
rmm.tactical.talabservices.it.  900  IN  A  213.32.30.52
                                ^^^
                                TTL: 900 secondi = 15 minuti
```

## ⚙️ Come funziona in pratica

Quando un PC chiede "che IP ha `rmm.tactical.talabservices.it`?", la richiesta non arriva quasi mai direttamente al nameserver di Register. In mezzo c'è una catena di cache: il resolver del sistema operativo, quello del router, quello dell'ISP (o 1.1.1.1/8.8.8.8). Ognuno di questi, la prima volta, chiede al nameserver autoritativo e poi ==memorizza la risposta per TTL secondi==. Tutte le richieste successive, dentro quella finestra, vengono servite dalla cache senza disturbare nessuno.

Conseguenza pratica: se cambio l'IP di un record, il mondo ==non se ne accorge subito==. Chi ha la risposta vecchia in cache continua a usarla finché il suo timer non scade. Con TTL 900, nel caso peggiore un client vede il vecchio IP per 15 minuti. È questo che la gente chiama (impropriamente) "propagazione DNS": non è il record che viaggia, sono le ==cache che scadono==, ognuna per conto suo.

> 💡 Il valore in `dig` non è fisso: interrogando una cache si vede il ==countdown== che scala verso zero. Per vedere il TTL _configurato_ bisogna chiedere al nameserver autoritativo: `dig @ns1.register.it nome.dominio.it`

## ⚖️ Il trade-off

	||TTL basso (60-300s)|TTL alto (3600s+)|
	|---|---|---|
	|Cambio record|Effettivo in minuti ✅|Effettivo in ore ❌|
	|Carico sul nameserver|Più query ❌|Poche query ✅|
	|Se il nameserver è giù|Il nome muore in fretta ❌|Le cache reggono a lungo ✅|

|                       | \|TTL basso (60-300s)     | TTL alto (3600s+)          |
| --------------------- | ------------------------- | -------------------------- |
| Cambio record         | Più query ❌               | Poche query ✅              |
| Carico sul nameserver | Il nome muore in fretta ❌ | Le cache reggono a lungo ✅ |

La regola operativa che ne esce:

- **Prima di una migrazione**: abbassare il TTL (300) ==in anticipo==, almeno un vecchio-TTL prima del cambio — così quando si cambia il record, la correzione di un eventuale errore si propaga in minuti, non in ore
- **A regime**: rialzarlo. 900-3600 è la fascia normale per record di produzione
- **Il nostro caso**: i tre record `*.tactical.talabservices.it` stanno a ==900 (default Register)== — valore ordinario, va bene così, non toccato

## 🪤 La trappola vissuta

Durante la migrazione il `dig` dalla VM rispondeva `127.0.1.1` per i nomi DuckDNS e sembrava un disastro DNS. Non c'entrava niente né il TTL né la cache: era una riga in ==`/etc/hosts`== che scavalcava tutto (il file hosts vince sempre sul DNS, non ha TTL, non scade mai). Lezione doppia:

1. Diagnosi DNS ==sempre anche contro resolver esterni==: `dig @1.1.1.1 nome` bypassa hosts e cache locali
2. Quando due fonti danno risposte diverse, la catena da controllare è: `/etc/hosts` → cache resolver locale → cache ISP → nameserver autoritativo. Il colpevole è quasi sempre nel primo anello che nessuno guarda