> **Cos'è**: Protocollo di rete sviluppato da Microsoft per la condivisione di file, stampanti e risorse tra computer in rete locale. Sta per **Server Message Block**.

---

### Versioni

| Versione  | Anno         | Note                                             |
| --------- | ------------ | ------------------------------------------------ |
| **SMBv1** | 1983         | Obsoleto e **insicuro** – da disabilitare sempre |
| **SMBv2** | 2006 (Vista) | Prestazioni migliori, meno verbose               |
| **SMBv3** | 2012 (Win8)  | Aggiunge crittografia end-to-end                 |

> ⚠️ **SMBv1 non va mai usato.** È la causa di EternalBlue, WannaCry e SambaCry.

---

### Porte

|Porta|Uso|
|---|---|
|`139/TCP`|SMB su NetBIOS (legacy)|
|`445/TCP`|SMB diretto – versione moderna|

---

### Come funziona (semplificato)

1. Il client invia una **richiesta SMB** al server (es. "voglio aprire il file X")
2. Il server **autentica** il client (tramite [[NTLM]] o [[Kerberos]])
3. Il server risponde e gestisce lettura/scrittura sul file
4. Tutto avviene in modo trasparente → l'utente vede la cartella remota come se fosse locale

---

### Relazioni

- [[🏦 SAMBA]] – implementazione open source di SMB per Linux/Unix
- [[NTLM]] – sistema di autenticazione usato da SMB
- [[Kerberos]] – autenticazione più sicura usata in ambienti [[Active Directory]]
- [[EternalBlue]] – exploit critico su SMBv1