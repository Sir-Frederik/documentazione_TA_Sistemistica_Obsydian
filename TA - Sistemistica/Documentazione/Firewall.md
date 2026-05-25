\Il firewall è un "cancello" che decide quali connessioni di rete **possono entrare o uscire** dalla VM. Sapere le sue regole ti serve per documentare quali porte sono **effettivamente raggiungibili dall'esterno**, indipendentemente da quello che ti dice `ss` o `lsof`.

> In pratica: una porta può essere in ascolto ma il firewall la blocca comunque. Oppure una porta è aperta nel firewall ma nessun servizio ci sta girando dietro.
> 
> 
>

 Sul tuo server Ubuntu/Debian, esegui questi tre comandi in ordine:
**1 — Controlla UFW (il più probabile su Ubuntu):**


```
sudo ufw status verbose
```
**2 — Controlla iptables (il livello sottostante):**

```
sudo iptables -L -n -v
```

**3 — Controlla firewall-cmd (improbabile su Ubuntu, ma per sicurezza):**



```
sudo firewall-cmd --state
```

## Come interpretare i risultati 

- Se `ufw status` risponde `Status: active` → UFW è attivo, leggi le sue regole
- Se risponde `Status: inactive` → UFW è spento, le regole sono solo in iptables
- Se `firewall-cmd` risponde `not running` o `command not found` → firewalld non è presente, normale su Ubuntu