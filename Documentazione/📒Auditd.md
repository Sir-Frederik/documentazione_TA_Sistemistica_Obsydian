E' un sistema di **auditing**  di Linux, cioè uno strumento che **registra** tutto quello che succedere sul sistema in un apposito **file di log**.

```
# Controlla lo stato di Auditd
sudo systemctl status auditd

# Visualizza le regole attive
sudo auditctl -l

# Cerca nei log un utente specifico
sudo ausearch -ua nome-utente

# Genera un report generale
sudo aureport
```
