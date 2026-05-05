Sono dei certificati forniti  da **Let's Encrypt**, un'autorità certificatrice **gratuita e open source** che garantisce che i siti abbiamo il protocollo ***[[HTTPS e HTTP | HTTPS]]***, ovvero **protocolli di crittografia**.
Lo strumento per averli è [[🎖️Certbot Renew | Certbot]].
I certificati scadono dopo 90 giorni e il sito potrebbe non risultare sicuro agli occhi del browser.

### SSL
Il protocollo originale, oggi **obsoleto e insicuro**

### TLS
Il successore moderno di SSL, **quello usato oggi** (anche se per abitudine si continua a dire SSL).

La crittografia del TSL si basa su una **coppia di chiavi**: la **Pubblica** e la **Privata**.
La **Pubblica** è condivisa con tutti e serve per **Cifrare** i dati.
La **Privata** è conosciuta solo dal Server, e serve a **Decifrare** i dati.

```
Client                          Server
  │                               │
  │── Richiesta connessione ─────>│
  │<─ Chiave Pubblica ────────────│
  │── Dati cifrati con chiave ───>│
  │   pubblica                    │
  │                 Decifra con   │
  │                 chiave privata│
  │<─ Risposta cifrata ───────────│
```