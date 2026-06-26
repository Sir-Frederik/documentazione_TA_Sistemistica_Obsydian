
> **Cos'è**: VPN moderna, leggera e ad alte prestazioni basata su crittografia a chiave pubblica. Integrata nel kernel Linux dalla versione 5.6. Usata per connettere client remoti a reti private (es. per raggiungere un [[Samba]] AD DC dall'esterno).

---

## Concetti base

| Termine | Significato |
|---|---|
| **Interface** | La scheda di rete virtuale VPN (es. `wg0`) |
| **Peer** | Un nodo che partecipa al tunnel (server o client) |
| **PrivateKey / PublicKey** | Coppia di chiavi asimmetriche per autenticazione e cifratura |
| **AllowedIPs** | Subnet il cui traffico viene instradato nel tunnel |
| **Endpoint** | IP:porta del server WireGuard raggiungibile dall'esterno |
| **PersistentKeepalive** | Intervallo in secondi per mantenere vivo il tunnel (utile dietro NAT) |

---

## Installazione (Ubuntu amd64)

```bash
sudo apt update && sudo apt install wireguard
```

> ℹ️ WireGuard è già incluso nel kernel Ubuntu ≥ 20.04. Non servono moduli aggiuntivi.

---

## Configurazione server

### 1. Generazione chiavi

```bash
# Chiavi server
wg genkey | sudo tee /etc/wireguard/server_private.key | wg pubkey | sudo tee /etc/wireguard/server_public.key
sudo chmod 600 /etc/wireguard/server_private.key

# Chiavi client
wg genkey | sudo tee /etc/wireguard/user_private.key | wg pubkey | sudo tee /etc/wireguard/user_public.key
sudo chmod 600 /etc/wireguard/user_private.key
```

---

### 2. Abilitare IP forwarding

```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

### 3. File di configurazione server `/etc/wireguard/wg0.conf`

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <contenuto di server_private.key>

# Sostituire ens3 con l'interfaccia di rete reale (verificare con: ip a)
PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE

[Peer]
# Client
PublicKey  = <contenuto di user_public.key>
AllowedIPs = 10.0.0.2/32
```

> ⚠️ Verificare il nome dell'interfaccia con `ip a` — su OVH è tipicamente `ens3` o `ens4`, non `eth0`.

---

### 4. Avvio e abilitazione al boot

```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

---

## Configurazione client

### File `/etc/wireguard/user.conf`

```ini
[Interface]
Address    = 10.0.0.2/24
PrivateKey = <contenuto di user_private.key>
DNS        = 10.0.0.1, 8.8.8.8

[Peer]
PublicKey           = <contenuto di server_public.key>
Endpoint            = <IP_pubblico_server>:51820
AllowedIPs          = 10.0.0.0/24
PersistentKeepalive = 25
```

> ℹ️ `AllowedIPs = 10.0.0.0/24` instrada nel tunnel solo il traffico verso la subnet VPN (split tunnel). Per far passare tutto il traffico usare `0.0.0.0/0` (full tunnel).
> ⚠️ Il campo `DNS` deve puntare al DC se si usa [[Samba]] AD — il client deve risolvere i nomi di dominio tramite il DNS di Samba.

---

### Esportare la config su Windows (via SCP)

Da PowerShell o CMD sul client Windows:

```powershell
scp ubuntu@<IP_server>:/etc/wireguard/user.conf C:\Users\TuoUtente\Desktop\user.conf
```

Su Windows: installare **WireGuard per Windows** da [wireguard.com/install](https://www.wireguard.com/install/), poi **Import tunnel(s) from file** e selezionare `user.conf`.

---

## Firewall (UFW)

```bash
sudo ufw allow 51820/udp
sudo ufw reload
```

---

## Comandi utili — Cheatsheet

```bash
# Stato tunnel e peer connessi
sudo wg show

# Attiva / disattiva tunnel manualmente
sudo wg-quick up wg0
sudo wg-quick down wg0

# Gestione servizio systemd
sudo systemctl start   wg-quick@wg0
sudo systemctl stop    wg-quick@wg0
sudo systemctl restart wg-quick@wg0
sudo systemctl status  wg-quick@wg0

# Lettura chiavi
sudo cat /etc/wireguard/server_private.key
sudo cat /etc/wireguard/server_public.key
sudo cat /etc/wireguard/user_private.key
sudo cat /etc/wireguard/user_public.key

# Verifica IP forwarding attivo
sysctl net.ipv4.ip_forward
```

---

## Troubleshooting

| Sintomo | Causa probabile | Fix |
|---|---|---|
| Handshake non avviene | Porta 51820/UDP bloccata | `sudo ufw allow 51820/udp` |
| Tunnel attivo ma nomi dominio non risolvono | DNS client punta a 8.8.8.8 invece del DC | Impostare `DNS = <IP_DC>` nel `user.conf` |
| DC raggiunto via IP pubblico invece che VPN | `AllowedIPs` non include la subnet del DC | Aggiungere la subnet corretta in `AllowedIPs` |
| Nessuna navigazione internet con tunnel attivo | `AllowedIPs = 0.0.0.0/0` senza DNS pubblico | Aggiungere DNS pubblico: `DNS = 10.0.0.1, 8.8.8.8` |
| `apt install wireguard` fallisce con DNS error | DNS forwarder Samba non raggiungibile | Vedi [[Samba]] § Troubleshooting |

---

##### Fonti

- WireGuard — [wireguard.com](https://www.wireguard.com/)
- WireGuard Quick Start — [wireguard.com/quickstart](https://www.wireguard.com/quickstart/)