Per prima cosa devi sapere se Il registry deve essere raggiungibile **solo dall'interno** (team) o anche da CI/CD esterni tipo [[🫖Jenkins]].
La prassi di installazione del [[🫙 Docker]] è la stessa, solo che nel caso servisse non esposto, bisogna limitare la ==porta *443*== col firewall.

#### 1 - Aggiorna e installa le dipendenze
``` bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ca-certificates curl gnupg
```
#### 2 - Aggiungi la repository ufficiale Docker
```bash
sudo install -m 0755 -d /etc/apt/keyrings curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \ 
	sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg sudo chmod a+r /etc/apt/keyrings/docker.gpg
	
	echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \ https://download.docker.com/linux/ubuntu \ $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \ sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Se dovesse chiederti `File '/etc/apt/keyrings/docker.gpg' exists. Overwrite? (y/N)` tu sovrascrivilo, per sicurezza.

#### 3 - Installa Docker Engine + [[🫙Containerd]]
```bash
sudo apt update 
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

#### 4 - Verifica
```bash
sudo systemctl is-active docker 
sudo systemctl is-active containerd 
sudo docker run hello-world
```

#### 5 - Preparate la struttura del Docker Registry
Prepara la struttura:
```bash
sudo mkdir -p /opt/docker-registry/{data,certs,auth}
```

#### 6 - [[Certificati SSL E TLS|Certificato TLS]] 
Per un server di ==**test** senza dominio==, si usa un certificato self-signed sull'IP pubblico (sostituisci il tuo ip corretto.)
```bash
sudo openssl req -newkey rsa:4096 -nodes -sha256 \
  -keyout /opt/docker-registry/certs/domain.key \
  -x509 -days 365 \
  -out /opt/docker-registry/certs/domain.crt \
  -subj "/CN=IP_DEL_SERVER" \
  -addext "subjectAltName=IP:IP_DEL_SERVER"
```
Per produzione con ==dominio reale,== si usa invece [[Let's Encrypt]]:
```bash
sudo apt install -y certbot
sudo certbot certonly --standalone -d registry.tuodominio.it
```

7 - Credenziali di accesso (htpasswd)
``` bash
sudo apt install -y apache2-utils
sudo htpasswd -Bbn tuoutente tuapassword | sudo tee /opt/docker-registry/auth/htpasswd
```
Salva tutto su KeePass.


8 - Avvio e verifica
Avvia il container del Registry: 
``` bash
sudo docker run -d \
  --name docker-registry \
  --restart=always \
  -p 443:5000 \
  -v /opt/docker-registry/data:/var/lib/registry \
  -v /opt/docker-registry/certs:/certs \
  -v /opt/docker-registry/auth:/auth \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -e REGISTRY_AUTH=htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2
```
e verifica che sia su:

``` bash
sudo docker ps | grep registry
```

**Testa ** il login sulla tua stessa macchina (modifica ip e credenziali):
``` bash
echo "tuapassword" | docker login https://51.68.84.145 -u tuoutente --password-stdin
```

**Se il login restituisce `x509: certificate signed by unknown authority`**, il client Docker non riconosce il certificato self-signed. Aggiungilo manualmente ai certificati fidati:
``` bash
sudo mkdir -p /etc/docker/certs.d/IP_DEL_SERVER
sudo cp /opt/docker/certs/domain.crt /etc/docker/certs.d/IP_DEL_SERVER/ca.crt
sudo systemctl restart docker
```
Questo va fatto su **ogni macchina** che deve connettersi al registry (non solo il server stesso).