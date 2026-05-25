Ti serve per accedere ai server e per interagirci, sfruttando il protocollo [[🔐 SSH]]. 
Esiste anche la versione **puttygen** per generare le chiavi.

E' possibile salvare le impostazioni di accesso per ogni server, così non devi farlo ogni volta. Per sapere  come si accede, vai [[Accedere ai Server|qui]].

Se ti connetti al server, si apre il terminale e lì devi inserire la password del server.

Se vuoi ritentare la connessione dal terminale, vai con tasto destro nella barra in alto e clicca su "restart session".

Per **copiare del testo** basta selezionarlo. Per incollarlo, basta usare il tasto destro. Se metti le password, non si vedono i caratteri.

I **package manager** possono essere di tue tipologie:
### Famiglia Debian/Ubuntu:

- **APT** (Advanced Package Tool)
- Comandi: apt, apt-get
- Pacchetti in formato **.deb**
  
### Famiglia RHEL (CentOS/AlmaLinux):

- **YUM** (Yellowdog Updater Modified) → versione vecchia
- **DNF** (Dandified YUM) → versione moderna, ha sostituito YUM
- Pacchetti in formato **.rpm**

(sudo, nano, # per commentare eccetera)

Per scorrere le pagine, premi *pggiù* o *pgsù*.
Per tornare al prompt premi **Q**
PEr salvare premi ctrl + o. Ti chiederà il nome del file, eventualmente accettalo con invio.

Quando vuoi uscire dalla sessione, scrivi "**Exit**"
Per conoscere l'ip del server digita
```
ip a
```