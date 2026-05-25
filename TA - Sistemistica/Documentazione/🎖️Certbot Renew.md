Cerbot è uno strumento che gestisce i [[Certificati SSL E TLS| Certificati SSL/TLS]] per i siti web che permettono di usare **[[HTTPS e HTTP|HTTPS]]**.
Ottiene i certificati grazie a [[Certificati SSL E TLS|Let's Encrypt]].
Cerbot Renew si occupa di **rinnovare automaticamente** i certificati prima che scadano.
```
# Rinnova tutti i certificati in scadenza
sudo certbot renew

# Simula il rinnovo senza farlo davvero (utile per testare!)
sudo certbot renew --dry-run

# Vedi tutti i certificati installati e la loro scadenza
sudo certbot certificates
```
Durante il rinnovo succede:
1. 📋 Certbot **controlla** tutti i certificati installati
2. ⏱️ Verifica se **stanno per scadere**
3. 🌐 Contatta [[Certificati SSL E TLS|Let's Encrypt]] per il rinnovo
4. 🔑 Ottiene il **nuovo certificato**
5. 🔄 **Riavvia** automaticamente il web server (es. Apache)
6. ✅ Il sito è di nuovo **sicuro per altri 90 giorni**