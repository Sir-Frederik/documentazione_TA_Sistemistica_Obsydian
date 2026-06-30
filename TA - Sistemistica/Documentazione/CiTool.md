
`CiTool` è lo strumento da riga di comando di Windows per gestire le policy [[📘Guida  personale a TRMM#WDAC — Windows Defender Application Control|WDAC]] (Windows Defender Application Control) direttamente sul sistema locale. È integrato in Windows 11 — non richiede installazione.

---

## Perché esiste 🤔

La compilazione di una policy WDAC richiede il modulo PowerShell `ConfigCI`, disponibile solo su Windows Pro/Enterprise. Il **deploy** invece è un'operazione separata, e Microsoft ha scelto di renderlo disponibile su tutte le edizioni tramite CiTool — Home inclusa.

Questo è esattamente il motivo per cui il nostro workflow funziona: si compila sul PC Pro (`TA-TEST`) e si deploya sul PC Home tramite CiTool.

---

## Cosa fa in pratica ⚙️

I comandi che usiamo sono essenzialmente due:

**Deploy di una policy:**

```
CiTool --update-policy <percorso_al_file.p7b>
```

Carica il file `.p7b` compilato, lo converte in `.cip` e lo copia nella cartella attiva di CodeIntegrity. La policy è effettiva al riavvio successivo.

**Lista delle policy attive:**

```
CiTool --list-policies
```

Mostra le policy attualmente caricate sul sistema, con GUID e versione. Utile per verificare cosa è effettivamente attivo.

---

## Il file .cip 📄

Quando CiTool deploya una policy, non usa il `.p7b` direttamente — lo converte in un file `.cip` (Compiled Integrity Policy) e lo copia in:

```
C:\Windows\System32\CodeIntegrity\CiPolicies\Active\
```

Il nome del file è il GUID della policy, es:

```
{966d1f08-bcea-48c4-bc3a-6651c21e4090}.cip
```

Questo è il file che Windows legge all'avvio. In caso di recovery, è questo il file da eliminare — non il `.p7b` originale.

---

## Cosa NON fa ⛔

CiTool non compila policy, non legge XML, non gestisce regole. È solo il meccanismo di deploy e interrogazione. Tutto il lavoro di costruzione della policy (scansione file, merge, compilazione) avviene con il modulo `ConfigCI` su TA-TEST.

---

## Riferimenti nel progetto 🔗

- Usato dallo script `WDAC: Deploy Policy` nella [[📘Guida personale a TRMM]]
- La cartella `CiPolicies\Active\` è il punto di riferimento anche per la procedura di recovery in WinRE
- Correlato: [[ISG — Intelligent Security Graph]]