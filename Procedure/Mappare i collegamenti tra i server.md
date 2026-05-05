Oltre [[Collegare i Progetti ai Server|a sapere quali progetti girano su un server]], è utile capire **come i server comunicano tra loro** — quali connessioni sono attive, verso quali altri server puntano e attraverso quali porte.

---

### Connessioni attive

Mostra tutte le connessioni di rete **aperte in questo momento**:


```bash
ss -tnp | grep ESTAB
```

`ESTAB` significa _established_ — connessione stabilita e attiva. È diverso dal datasource che mostra le connessioni _configurate_: questo mostra quelle _realmente aperte_.

Come interpretare l'output:

```
146.59.240.195:22  ←  79.41.50.45:54883
```

Connessione **in entrata** sulla porta 22 — qualcuno è collegato via [[🔐 SSH]] (PuTTY).

```
146.59.240.195:53332  →  130.61.39.98:3306  (java, pid=523)
```

Connessione **in uscita** verso un server [[⛃ MySQL]] sulla porta 3306 — è [[☕WildFly]] che comunica con il database. Normale trovarne più di una: WildFly mantiene un **pool di connessioni** aperte contemporaneamente.

> ℹ️ L'IP di destinazione nella connessione in uscita è il tuo **DB Server** — annotalo nella documentazione se non lo conosci già.

---

### Configurazione Apache (reverse proxy)

Apache può funzionare da **reverse proxy** — riceve le richieste degli utenti e le smista verso altri server o servizi interni. Leggendo la sua configurazione capisci verso quali altri server punta. Lo stesso ruolo è svolto da [[🚦HAProxy]].

**Su CentOS/AlmaLinux:**



```bash
cat /etc/httpd/conf.d/*
```

**Su Ubuntu/Debian:**



```bash
cat /etc/apache2/sites-enabled/*
```

Cerca righe con `ProxyPass` o `ProxyPassReverse` — indicano verso quale server interno Apache sta smistando il traffico.

---

### Connessioni nei file properties

I file `.properties` contengono configurazioni applicative, inclusi URL di database e servizi esterni. Questo comando li cerca su tutto il server e filtra le righe con connessioni:



```bash
grep -rn "jdbc:\|http://\|https://\|host=\|server=" \
  $(find / -name "*.properties" 2>/dev/null | grep -v "/proc\|/sys") 2>/dev/null
```

> ℹ️ Questo comando può essere lento su server con molti file. Se non trovi nulla di rilevante, le connessioni sono già tutte nel `standalone.xml`.