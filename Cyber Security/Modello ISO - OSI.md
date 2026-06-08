---
tags:
  - Protocollo
---
Il Modello ISO-OSI (**Open Systems Interconnection**) è un modello concettuale formato da ==**7 livelli**== che descrive come i dati viaggiano attraverso una rete, dall'hardware fisico fino all'applicazione che li utilizza. Permette di capire all'interno di quale flusso di scambio di pacchetti ci troviamo in un certo contesto di rete.

Ogni livello ha un compito preciso e **comunica solo con il livello immediatamente sopra e sotto di sé**. Quando un dato viene inviato, scende dal livello 7 al livello 1 aggiungendo informazioni ad ogni passaggio (**incapsulamento**). Quando viene ricevuto, risale dal livello 1 al livello 7 rimuovendole (**decapsulamento**).
### I 7 Livelli

![[Pasted image 20260402165824.png]]

#### 1. Fisico

Si occupa della trasmissione dei **bit grezzi** sul mezzo fisico (cavi, fibre ottiche, onde radio). Non conosce il significato dei dati, li tratta solo come segnali elettrici o luminosi. _Esempi: cavi Ethernet, WiFi, fibra ottica_

#### 2. Data Link

Gestisce la comunicazione tra due dispositivi **direttamente collegati** sulla stessa rete locale. Introduce gli indirizzi **MAC** per identificare i dispositivi fisici e si occupa del rilevamento degli errori di trasmissione. _Esempi: Ethernet, Switch_

#### 3. Rete

Si occupa dell'**indirizzamento logico e dell'instradamento** dei pacchetti attraverso reti diverse, fino alla destinazione finale. È il livello del [[Protocollo IP|]] e dei router. _Esempi: [[Protocollo IP|IP]], [[Protocollo ICMP|ICMP]], Router_

#### 4. Trasporto

Gestisce la **comunicazione end-to-end** tra mittente e destinatario, occupandosi dell'affidabilità, dell'ordine dei pacchetti e del controllo del flusso. È il livello di [[Protocollo TCP|TCP]] e [[Protocollo UDP|UDP]]. _Esempi: [[Protocollo TCP|TCP]], [[Protocollo UDP|UDP]]_

#### 5. Sessione

Gestisce l'apertura, il mantenimento e la chiusura delle **sessioni di comunicazione** tra due applicazioni. Si occupa anche della sincronizzazione in caso di interruzioni. _Esempi: NetBIOS, RPC_

#### 6. Presentazione

Si occupa della **traduzione, compressione e cifratura** dei dati. Converte i dati dal formato dell'applicazione a un formato standard comprensibile dalla rete, e viceversa. _Esempi: SSL/TLS, JPEG, ASCII, UTF-8_

#### 7. Applicazione

È il livello più vicino all'utente. Fornisce i **servizi di rete direttamente alle applicazioni**, come la navigazione web, la posta elettronica o il trasferimento file. _Esempi: HTTP, HTTPS, FTP, DNS, SMTP_

### Schema riepilogativo
![[Pasted image 20260521165853.png]]

|#|Livello|Unità di dati|Protocolli/Dispositivi|
|---|---|---|---|
|7|Applicazione|Dati|HTTP, FTP, DNS|
|6|Presentazione|Dati|SSL/TLS, JPEG|
|5|Sessione|Dati|NetBIOS, RPC|
|4|Trasporto|Segmenti|[[Protocollo TCP\|TCP]], [[Protocollo UDP\|UDP]]|
|3|Rete|Pacchetti|[[Protocollo IP\|IP]], [[Protocollo ICMP\|ICMP]]|
|2|Data Link|Frame|Ethernet, Switch|
|1|Fisico|Bit|Cavi, WiFi, Fibra|



