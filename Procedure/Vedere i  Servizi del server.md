Per avere l'elenco di tutti i servizi **attivi** si scrive:
```
systemctl list-units --type=service --state=running
```


Per vedere un servizio **specifico**:

```
systemctl status nome-servizio
```

Per vedere i servizi **inattivi** si scrive:
```
systemctl list-units --type=service --state=failed
```

Per vedere **tutti i servizi** si scrive:
```
systemctl list-units --type=service --no-pager
```



Per vedere **tutte le porte** in ascolto:
``` 
ss -tuln
```

per cercare tutti i servizi **java**, si usa il comando grep:
```
ps aux | grep java | grep -v grep
```

In questo modo si cercano tutti i servizi che hanno "java" nel nome, ed esclude il file grep stesso, quindi se non c'è nessun servizio, vedrai un elenco vuoto.


Per vedere il journal di tutte le interazioni , si scrive:
`journalctl`

## Versioni dei servizi
Pere vedere le versioni servizi segui questi comandi:
#### Java
```bash
java -version
```
#### Wildfly
```bash
# Versione
find / -name "wildfly*" -o -name "jboss* " 2>/dev/null | head -20
# Stato servizio
systemctl status wildfly
```

#### Apache
```bash
apache2 -v          # Ubuntu/Debian
httpd -v            # AlmaLinux/CentOS

systemctl status apache2    # Ubuntu/Debian
systemctl status httpd       #Almalinux/CentOS
```

#### Nginx
```bash
nginx -v        

```

#### HAProxy
```bash
haproxy -v
systemctl status haproxy
```

#### MySQL/MariaDB
```bash
mysql --version
systemctl status mysql
systemctl status mariadb
```
#### Postgress
```bash

psql --version
systemctl status postgresql
```

#### MongoDB
```bash

mongod --version
systemctl status mongod
```


#### Comando Unico
```bash
echo "=== OS ===" && cat /etc/os-release | grep PRETTY

echo "=== JAVA ===" && java -version 2>&1

echo "=== WILDFLY ===" && (
  WILDFLY_DIR=$(find /opt -maxdepth 3 -type d -name "wildfly-*" 2>/dev/null | head -1) && \
  echo "Path: $WILDFLY_DIR" && \
  echo "Versione: $(echo $WILDFLY_DIR | sed 's/.*wildfly-//')" || \
  echo "non trovato"
)

echo "=== APACHE ===" && (apache2 -v 2>/dev/null || httpd -v 2>/dev/null || echo "non trovato")

echo "=== DB ===" && (mysql --version 2>/dev/null || psql --version 2>/dev/null || echo "non trovato")

echo "=== MONGODB ===" && (mongod --version 2>/dev/null || echo "non trovato")
echo "=== HAPROXY ===" && (haproxy -v 2>&1 | head -1 || echo "non trovato")

```

Fatto questo puoi vedere in [[Crontab]].
Per avere un elenco dei servizi vai [[🛠️SERVIZI USATI🛠️|qui]]



