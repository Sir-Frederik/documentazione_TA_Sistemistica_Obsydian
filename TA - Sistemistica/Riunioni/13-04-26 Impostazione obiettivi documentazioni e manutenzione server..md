Riunione con Adriano, Genny e Alessandro.
# **1. Documentazione architetturale**

###  Schema infrastrutturale

- Schema a blocchi tra i due server
- Schema a blocchi del dominio applicativo
- Schema delle macchine _Parthenope_:
    - Suddivisione tra **OVH** e **Cloud**
    - Evidenziare correlazione con i **puntamenti ai database**

---

#  **2. Censimento risorse**

###  Tipologie da censire

- **IaaS (Infrastructure as a Service)**
    - Macchine gestite direttamente (già fatto)
- **PaaS (Platform as a Service)**
    - Servizi di piattaforma (es. DB gestiti, runtime)
- **SaaS (Software as a Service)**
    - Es. Google Drive
    - Inserire in **tabella separata**

###  Dettagli da raccogliere

Per ogni risorsa/macchina:

- Versione del sistema operativo
- Versioni software:
    - Java
    - Web server (WildFly)
    - Apache / HAProxy
    - Database (PostgreSQL, MySQL)
- Ruolo della macchina
- Relazioni con altri sistemi

Nota:

- **PaaS e SaaS** → non serve censire le porte
- **IaaS** → censimento completo già fatto.

---

#  **3. Analisi connessioni e dipendenze**

###  Verifiche da fare

- Verificare se le VM sono sullo stesso host fisico
- Analizzare comunicazioni tra server:
    - Bilanciamento **Apache**
    - Proxy / reverse proxy
- Individuare:
    - Puntamenti a DB esterni
    - Comunicazioni tra servizi

###  Dove cercare

- Configurazioni **WildFly (datasource)**
- File `.properties` delle applicazioni

---

#  **4. Sicurezza e hardening**

###  Obiettivi

- Nessuna porta pubblica esposta
- VPN su ogni macchina

###  Azioni

- Installare **Fail2Ban** su tutte le macchine
- Analizzare vulnerabilità e componenti obsoleti
- Creare documento con:
    - Vulnerabilità
    - Software deprecati

###  Piano

- Definire **migrazione di aggiornamento** per ogni macchina:
    - Sistema operativo → versione non deprecata
    - Software → versioni aggiornate

---

#  **5. Automazione**

- **Ansible**
    - Tool per automatizzare operazioni:
        - Configurazioni
        - Deploy
        - Aggiornamenti

---

#  **6. Monitoraggio e logging**

###  Strumenti

- **ELK Stack**
    - Centralizzazione log
    - Analisi traffico
    - Alert su soglie
- **Zabbix**
    - Monitoraggio rete e infrastruttura
- **SonarQube**
    - Analisi qualità del codice durante le build

---

#  **7. Output finale**

- Documentazione completa su **Wiki**
- Tabelle separate per:
    - IaaS
    - PaaS
    - SaaS
- Documento dedicato a:
    - Vulnerabilità
    - Obsolescenze
    - Piano di aggiornamento