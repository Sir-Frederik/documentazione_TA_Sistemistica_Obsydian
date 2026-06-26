
## Cos’è

**BitLocker** è la tecnologia di cifratura disco di Windows (presente su edizioni Pro, Enterprise ed Education) che protegge i dati presenti su un’unità rendendoli illeggibili senza le corrette chiavi di accesso.

Serve principalmente a mitigare il rischio di **furto fisico del dispositivo o del disco**.

---

## Obiettivo

- Proteggere i dati in caso di furto o smarrimento del PC
    
- Impedire l’accesso ai dati anche rimuovendo l’SSD/HDD
    
- Garantire compliance di sicurezza (es. ISO 27001)
    

---

## Come funziona

BitLocker cifra l’intero disco e richiede una o più **chiavi di sblocco (protectors)** per accedere ai dati.

Tipologie comuni di protectors:

- **TPM (Trusted Platform Module)**  
    Sblocco automatico legato all’hardware del PC
    
- **PIN all’avvio (TPM + PIN)**  
    Richiede un codice prima del boot
    
- **Recovery Key (48 cifre)**  
    Chiave di emergenza per recupero accesso
    
- **Password / USB key** (meno comune in contesti moderni)
    

---

## TPM e BitLocker

Il TPM è un chip hardware che permette a BitLocker di:

- verificare che il sistema non sia stato manomesso
    
- sbloccare automaticamente il disco all’avvio
    
- legare la cifratura alla macchina specifica
    

Senza TPM, BitLocker può funzionare ma con limitazioni operative.

---

## Stati di BitLocker

|Stato|Significato|
|---|---|
|Attivo|Cifratura e protezione completamente operative|
|Sospeso|Disco cifrato ma protezione temporaneamente disattivata|
|Parzialmente configurato|Protectors mancanti o non coerenti|
|Disattivo|Disco non cifrato|

---

## Comandi utili

### Stato BitLocker

```powershell
manage-bde -status
```

### Visualizzare protectors

```powershell
manage-bde -protectors -get C:
```

### Abilitare protezione

```powershell
Resume-BitLocker -MountPoint "C:"
```

### Abilitare TPM protector

```powershell
Add-BitLockerKeyProtector -MountPoint "C:" -TpmProtector
```

---

## Recupero chiavi 🔑

Le recovery key sono fondamentali:

- vengono generate al momento dell’attivazione
    
- possono essere salvate su account Microsoft, AD o file
    
- sono l’unico modo per recuperare il disco se TPM o password non funzionano
 
 Per visualizzarla, esegui questo [[🔒BitLocker#Visualizzare protectors|comando]].    

---

## BitLocker e sicurezza aziendale

In contesti aziendali BitLocker è essenziale perché:

- riduce drasticamente il rischio di data breach da furto fisico
    
- è richiesto da molte policy di sicurezza (es. ISO 27001)
    
- può essere centralizzato via Active Directory o Entra ID
    

---

## Limitazioni

- Non protegge da malware o attacchi logici mentre il sistema è acceso
    
- Richiede configurazione corretta per essere efficace
    
- Se mal configurato può entrare in stato “sospeso”
    

---

## Nota operativa

Un sistema BitLocker “sospeso” o senza TPM protector attivo:

- rimane cifrato
    
- ma non è in uno stato di enforcement ottimale
    
- può indicare provisioning incompleto o manutenzione non conclusa