I container sono dei pacchetti leggere che contengono un'applicazione e tutto ciò di cui ha bisogno per funzionare, isolata dal resto del sistema.

I container vengono creati a partire da una **Immagine**. 
L'**immagine** è È un **modello di sola lettura** che contiene tutto il necessario per eseguire un'applicazione. Sistema operativo, dipendenze, configurazioni, codice.
Le immagini sono costruite a **strati (layers)**, e ogni strato è di sola lettura. Ogni volta che il [[📦 Docker|dockerfile]] avvia un'istruzione, genera un nuovo strato.
Se due immagini usano lo stesso strato, viene scaricato una sola volta.^113a42

L'esecuzione stessa dell'immagine è un **container** e se ne possono creare diversi dalla stessa  immagine.

Vengono costruiti da [[🫙Containerd]] e/o da [[🫙 Docker]].

Quando un container viene avviato da un'immagine:

- Viene aggiunto un **strato scrivibile** sopra agli strati dell'immagine
- Tutte le modifiche fatte nel container vanno in questo strato
- Quando il container viene **eliminato**, questo strato viene eliminato
- L'immagine originale rimane **intatta**
  
  **Ciclo di un Container**
  Creato → Avviato → In esecuzione → Fermato → Eliminato
           ↑              ↓
           └──── Riavviato ┘