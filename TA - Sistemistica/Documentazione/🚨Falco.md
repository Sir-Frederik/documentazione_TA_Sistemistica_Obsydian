E' un progetto  **CNCF** ( Cloud Native Computing Foundation) designato per trovare un comportamento anomalo e da lì ti danno dei consigli di sicurezza in tempo reale. In pratica ti dice cosa succede mentre l'app è in esecuzione.
E' specializzato nel **runtime security per container e Kubernetes**. Monitora le syscall del kernel tramite **eBPF** (o kernel module) e genera alert quando rileva comportamenti anomali rispetto a regole predefinite.
Può essere installato solo su ubuntu e su kubernetes.
Si installa come **daemonset**, cioè gira su ogni nodo del cluster e monitora tutti i container che passano da lì.


### Che impatto ha?

 Ha circa 100 mb di dimensioni. Con il ruleset di default i valori reali sono spesso intorno a **200–400m CPU** e **200–400Mi RAM** per nodo sotto carico moderato.


[[Integrazione in ELK Stack di Zabbix, Prometheus e Falco]]
