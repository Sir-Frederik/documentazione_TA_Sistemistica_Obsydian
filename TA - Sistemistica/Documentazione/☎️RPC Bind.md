È un servizio che funziona da **"centralino"** per le comunicazioni **RPC (Remote Procedure Call)**. 
Le **RPC** sono un meccanismo che permette a un programma di **eseguire funzioni su un altro computer** in rete come se fossero locali.

Ogni servizio che usa RPC si registra su **rpcbind** con un numero di porta. Quando un client vuole comunicare con quel servizio, chiede prima a rpcbind: