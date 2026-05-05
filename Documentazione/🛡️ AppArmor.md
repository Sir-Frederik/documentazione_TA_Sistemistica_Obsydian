E' un sistema di **sicurezza** per Linux che limita quello che i programmi possono fare sul sistema. E' tipo un "regolamento" che impone delle regole ai programmi.
Così, se un programma venisse hackerato o si comporta male, può fare molti meno danni.

AppArmor usa dei file chiamati "**Profili**" che definiscono le regole di ogni programma.

Ha due modalità principali:
1. Enforce: applica le regole e blocca tutto ciò che non è permesso
2. Complain: Non blocca nulla, ma registra tutte le violazioni (utile per i test)
   
```
# Controlla lo stato di AppArmor
sudo apparmor_status

# Metti un profilo in modalità enforce
sudo aa-enforce /etc/apparmor.d/nome-profilo

# Metti un profilo in modalità complain
sudo aa-complain /etc/apparmor.d/nome-profilo
```
