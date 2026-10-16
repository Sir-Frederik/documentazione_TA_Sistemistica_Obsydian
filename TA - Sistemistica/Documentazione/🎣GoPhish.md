---
title: GoPhish - Simulazione Phishing TA
tags: [cybersecurity, iso27001, phishing, gophish, infrastruttura]
stato: in-corso
creato: 2026-10-13
aggiornato: 2026-10-13
---

# GoPhish - Simulazione Phishing TA

> [!info] Stato
> Infrastruttura pronta e in HTTPS. Mancano: Sending Profile (SMTP, con Gianluca), email template, landing page, lancio campagna. Documento vivo, da aggiornare a fine setup.

## Obiettivo e contesto

Simulazione di phishing interna autorizzata, a campione/esteso sui dipendenti, come evidenza di formazione ed efficacia per la certificazione [[ISO 27001]] (osservazione Stage 2 **CER-39**). Fa seguito al corso di Cyber Hygiene su [[learning.talabservices.it]].

- **Autorizzazione**: OK di Adriano (13/10/2026).
- **Privacy**: risultati usati solo a fini formativi e in forma aggregata, senza conseguenze disciplinari (art. 4 Statuto dei Lavoratori + GDPR). La landing page registra *che* la persona ha inserito le credenziali, **senza salvare la password reale**. Da verificare con il referente privacy aziendale.

## Architettura

```
Internet
   |  HTTPS (443)
   v
nginx (reverse proxy, TLS Let's Encrypt)
   |  HTTP  ->  127.0.0.1:8080
   v
GoPhish phish_server  (pagina esca / tracciamento)

GoPhish admin_server  ->  127.0.0.1:3333  (solo via tunnel SSH)
```

| Elemento | Valore |
|---|---|
| VM | `ta-tactical` (OVH) |
| OS | Ubuntu 22.04 (Jammy) |
| IP pubblico | 213.32.30.52 |
| Sottodominio esca | `tasevrices-portale.duckdns.org` (refuso "tasevrices" voluto, typosquatting) |
| Reverse proxy | nginx (`/etc/nginx/conf.d/gophish.conf`) |
| Certificato | Let's Encrypt via certbot (plugin nginx), rinnovo automatico |
| GoPhish | `/opt/gophish`, servizio systemd `gophish` |
| Porta admin | `127.0.0.1:3333` (TLS autofirmato, non esposto a internet) |
| Porta esca | `127.0.0.1:8080` (dietro nginx) |

## Installazione (sintesi)

1. **Aggiornamento VM**: `sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y`
2. **Download GoPhish** in `/opt/gophish` (release linux-64bit da GitHub), unzip, `chmod +x gophish`.
3. **Config** (`config.json`): admin già su `127.0.0.1:3333`; phish spostato da `0.0.0.0:80` a `127.0.0.1:8080`.
4. **Servizio systemd** `gophish` (`/etc/systemd/system/gophish.service`), `enable --now`.
5. **Reverse proxy nginx** per `tasevrices-portale.duckdns.org` verso `127.0.0.1:8080` (header `X-Forwarded-For` ecc.).
6. **HTTPS**: `sudo certbot --nginx -d tasevrices-portale.duckdns.org` con redirect HTTP->HTTPS.

## Accesso al pannello admin

Il pannello non è pubblico: si raggiunge con un tunnel SSH.

**PuTTY**: carica la sessione della VM -> Connection > SSH > Tunnels -> Source port `3333`, Destination `127.0.0.1:3333`, Local + Auto -> Add -> Save -> Open.

**CLI** (equivalente): `ssh -L 3333:127.0.0.1:3333 ubuntu@213.32.30.52`

Poi nel browser: `https://127.0.0.1:3333` (accettare il certificato autofirmato). Utente `admin`, password in **KeePass** (quella iniziale è stata cambiata al primo accesso).

## Componenti GoPhish

- [ ] **Sending Profile** (SMTP Aruba + mittente) — dipende da Gianluca
- [ ] **Users & Groups** — importare i 45 destinatari (CSV: First Name, Last Name, Email, Position)
- [ ] **Email Template** — email esca
- [ ] **Landing Page** — finta pagina di login servizi TA (no salvataggio password)
- [ ] **Campaign** — invio e tracciamento

## Scenario scelto

**Verifica / scadenza credenziali del portale.** Email che avvisa che la password del portale/posta sta per scadere e invita a riconnettersi; il link porta alla finta pagina di login su `tasevrices-portale.duckdns.org`. La pagina cattura il dato di invio ma non la password reale.

## TODO / prossimi passi

- [ ] Con **Gianluca**: credenziali SMTP Aruba, dominio mittente, record **SPF / DKIM / DMARC**, whitelist sul filtro antispam aziendale verso le caselle interne.
- [ ] Preparare CSV dei 45 e importarlo in Users & Groups.
- [ ] Costruire email template e landing page coerenti con lo scenario.
- [ ] Invio di prova a se stesso prima del lancio.
- [ ] Pianificare l'invio (orari sfalsati, 2-3 varianti di modello).
- [ ] A fine campagna: esportare i risultati come evidenza per l'audit (aggregati).

## Comandi utili

```bash
# stato ed eventuale riavvio del servizio
sudo systemctl status gophish
sudo systemctl restart gophish

# log (anche password iniziale al primo avvio)
sudo journalctl -u gophish --no-pager | less

# porte in ascolto
sudo ss -ltnp | grep -E ':3333 |:8080 '

# certificato: stato e test rinnovo
sudo certbot certificates
sudo certbot renew --dry-run

# nginx: test config e reload
sudo nginx -t && sudo systemctl reload nginx
```

## Note di sicurezza

- Pannello admin mai esposto a internet: solo loopback + tunnel SSH.
- Password admin in KeePass, mai in chiaro nelle note o in chat.
- La landing page non deve conservare le password inserite.
- A test concluso, valutare lo spegnimento della VM o la disattivazione del sottodominio DuckDNS.

## Changelog

- **2026-10-13** — Setup infrastruttura: VM aggiornata, GoPhish installato come servizio, reverse proxy nginx, HTTPS con Let's Encrypt, accesso admin via tunnel SSH. Scenario deciso. Componenti campagna da costruire.

## Collegamenti

- [[ISO 27001]]
- [[learning.talabservices.it]]
- [[OpenVAS]]
