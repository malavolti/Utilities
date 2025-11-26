# Docker

## Indice

1. [Le Basi](./Docker-Notes.md#le-basi)
2. [Comandi](./Docker-Notes.md#comandi-ctrlc-per-terminare)
3. [Dockerfile -  1 linea = 1 nuovo livello/layer](./Docker-Notes.md#dockerfile----1-linea--1-nuovo-livellolayer)
   1. [Istruzioni Dockerfile](./Docker-Notes.md#istruzioni-dockerfile)
   2. [HowTo - Creare una nuova immagine custom per un container](./Docker-Notes.md#howto---creare-una-nuova-immagine-custom-per-un-container)
   3. [HowTo - Caricare la propria immagine custom su Docker Hub](./Docker-Notes.md#howto---caricare-la-propria-immagine-custom-su-docker-hub)
4. [Docker Compose - Come installare un application stack](./Docker-Notes.md#docker-compose---come-installare-un-application-stack)
   1. [Installazione (su Debian)](./Docker-Notes.md#installazione-su-debian)
   2. [Utilizzo](./Docker-Notes.md#utilizzo)
5. [Docker Registry](./Docker-Notes.md#docker-registry)
6. [Docker Storage](./Docker-Notes.md#docker-storage)
7. [Docker Network](./Docker-Notes.md#docker-network)
8. [Container Orchestration](./Docker-Notes.md#container-orchestration)
9. [Docker Swarm](./Docker-Notes.md#docker-swarm)
10. [Kubernetes](./Docker-Notes.md#kubernetes)

## Le Basi

* La portabilità è una delle caratteristiche di spicco di Docker.
* Un container Docker è un'immagine Docker in esecuzione.
* Un'immagine Docker si può creare con un Dockerfile, un file di testo che elenca una serie di operazioni.
* Il nome delle immagini non può contenere lettere MAIUSCOLE.
* Docker si può installare con il [Convenience script](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script), ma è meglio installarlo attraverso repository APT/PKG.
* Per poter eseguire i comandi senza essere utenti ROOT, seguire i [post-install steps](https://docs.docker.com/engine/install/linux-postinstall/).
* [https://hub.docker.com](https://hub.docker.com) contiene le immagini Docker già create da altri e pronte all'uso.
* Un container vive per il tempo di vita dei processi che in esso girano, ossia vive per il tempo necessario a svolgere il compito richiesto.
* Per uscire da un container bloccato bisogna trovare il suo `<CONTAINER-ID>` attraverso il comando `docker ps` ed eseguire `docker stop <CONTAINER-ID>`
* Ogni nuova immagine Docker si deve basare su una già esistente, quindi \
  ogni Dockerfile DEVE INIZIARE con l'istruzione `FROM <IMAGE-NAME>`, \
  dove `<IMAGE-NAME>` è l'immagine base recuperata dal Docker Hub o da un altro repository di immagini docker.
* Quando creiamo delle immagini personali/custom, dobbiamo nominarle come `<username-docker-hub>/<my-image-name>`.
* Docker Host o Docker Engine: è la macchina fisica su cui viene installato ed eseguito docker
* Ogni container, per default, ottiene un IP locale raggiungibile solo dal Docker Host. \
  Per permettere agli esterni di utilizzare l'applicazione web che gira all'interno del container \
  è necessario mappare la porta utilizzata dall'applicazione con una libera del Docker Host:

  `docker run -p 443:8443 <IMAGE-NAME>`

  questo comando crea un container da `<IMAGE-NAME>`, che usa la porta **8443** per il suo servizio, \
  e lo rende disponibile a tutti / al Docker Host attraverso la porta **443**. \
  _Le porte devono essere libere per essere usate._

* Ogni container mantiene i dati in esso memorizzati fino alla sua rimozione con: `docker rm <CONTAINER-ID o CONTAINER-NAME>`
* Il processo di generazione delle proprie immagini è rapido grazie alla cache che mantiene le operazioni già eseguite.
* Non è possibile eseguire 2 container con lo stesso nome.
* Docker ottimizza l'uso dei layer ri-utilizzando quanto già presente. In questo modo il build di un'immagine è sempre performante.
* Il comando `docker build` serve per creare un'immagine con cui si può creare un container. \
  Qualsiasi nuovo container che usa la medesima immagine condividerà la cache per la sua creazione rendendo più veloce il tutto. \
  Tutto quello che viene scritto in un container, viene cancellato alla sua rimozione o morte.
* Per connettersi via rete ad un container a partire da un altro è necessario usare il suo **nome** \
  in quanto l'indirizzo IP della rete privata _bridge_ non è garantito che rimanga lo stesso al riavvio.
* Cgroups (Control Groups): sono il modo con cui il kernel Linux gestisce l'utilizzo delle risorse della macchina da parte di un gruppo specifico di processi. \
  Grazie ai cgroups è possibile limitare la quantità di risorse utilizzate da uno o più processi. \
  Docker usa i Cgroups per limitare le risorse alle applicazioni da lui create.
* Namespaces: Hanno il compito di garantire l'isolamento dei processi in esecuzione in container differenti. \
  Un processo in esecuzione in un container non può accedere direttamente alla macchina ospite o ad altri container. \
  Assegnare dei namespace differenti ad ogni container significa quindi creare delle sandbox in grado di isolare i processi del container dal resto della macchina.

## Comandi (CTRL+C per terminare)

* `docker run <IMAGE-NAME or IMAGE-ID>`: crea un container dall'immagine `<IMAGE-NAME or IMAGE-ID>` (che viene scaricata se non presente).
    * `docker run -it debian bash` \
       crea il container docker dall'ultima versione dell'immagine `debian` \
       e lo esegue esponendo la shell bash del container stesso. Bash è il servizio esposto dall'immagine.

    * `docker run debian:8.0` \
       crea il container docker dal TAG `8.0` dell'immagine `debian`. \
       Questo permette di scegliere quale versione dell'immagine utilizzare.

    * `docker run -V <PATH-OUTSIDE-DIR>:<PATH-INSIDE-DIR> <IMAGE-NAME or IMAGE-ID>` \
      crea un container in cui la cartella al suo interno `<PATH-INSIDE-DIR>` rimane sincronizzata nel contenuto, \
      con quella presente esternamente sul Docker Host `<PATH-OUTSIDE-DIR>` da cui è partito il comando.

    * `docker run --name <CONTAINER-NAME> <IMAGE-NAME or IMAGE-ID>` \
      crea il container docker da un'immagine e lo nomina `<CONTAINER-NAME>`

    * `sudo docker run -e CONSTANT-NAME=value <IMAGE-NAME or IMAGE-ID>` \
      crea il container docker dall'immagine passando anche la variabile di ambiente **CONSTANT-NAME**

    * `docker run -p 443:8443 <IMAGE-NAME>` \
     	crea un container da `<IMAGE-NAME>` e rende disponibile sulla porta `443` l'applicazione in  \
	    esecuzione sul container e che usa la porta `8443`

    * `docker run -d <IMAGE-NAME or IMAGE-ID>` \
      esegue il container in background lasciando libero il terminale per eseguire altre operazioni. \
      Per vedere tutti i container in background(detached mode) usare `docker ps` \
      e per riportare i container in foreground usare `docker attach <CONTAINER-ID or CONTAINER-NAME>`

    * `docker run -d -p 443:8443` `<IMAGE-NAME or IMAGE-ID>` \
      esegue il container in background(detached mode) esponendndo l'applicazione, che nel container usa la porta 8443, sulla porta 443.
  
    * `docker run -v <VOLUME-PATH>:<CONTAINER-DIRPATH-FOR-VOLUME> <IMAGE-NAME or IMAGE-ID>`: \
      lega un volume persistente ad una cartella del container. \
      Il contenuto della cartella `<CONTAINER-DIRPATH-FOR-VOLUME> \
      viene scritto nel volume `<VOLUME-PATH>` del Docker Host che non viene distrutto con la morte del container.

      esempio:

      `docker run -v /data/mysql_volume:/var/lib/mysql mysql`

      oppure:

      `docker run --mount type=bind,source=/data/mysql_volume,target=/var/lib/mysql mysql`

* `docker ps`: elenca tutti i container attivi e utili informazioni
* `docker ps -a`: elenca tutti i container, anche quelli non più attivi
* `docker restart <CONTAINER-ID o CONTAINER-NAME>`: riavvia un container
* `docker stop <CONTAINER-ID o CONTAINER-NAME>`: disattiva un container. \
  E' possibile fermare più container inserendo le prime 2 lettere di ogni CONTAINER-ID separate da uno spazio
* `docker rm <CONTAINER-ID o CONTAINER-NAME>`: rimuove un container non attivo
* `docker rm $(docker ps -qa)`: rimuove tutti i container non in esecuzione
* `docker images`: elenca tutte le immagini disponibili
* `docker rmi <IMAGE-NAME or IMAGE-ID>`: rimuove l'ultima versione dell'immagine indicata solo se non è usata da un container
* `docker rmi -f <IMAGE-NAME or IMAGE-ID>`: rimuove l'immagine indicata forzatamente
* `docker pull <IMAGE-NAME or IMAGE-ID>`: scarica l'immagine dal Docker hub
* `docker exec <CONTAINER-ID o CONTAINER-NAME> <COMMAND-TO-RUN-IN-THE-CONTAINER>`: esegue un comando nel container attivo
* `docker inspect <CONTAINER-ID o CONTAINER-NAME>`: restituisce informazioni sul container, comprese le variabili d'ambiente (ENV)
* `docker logs <CONTAINER-ID o CONTAINER-NAME>`: restituisce lo standard output del container
* `docker build /path/Dockerfile -t <username-docker-hub>/my-image`: crea l'immagine a partire dal Dockerfile
* `docker history <IMAGE-NAME or IMAGE-ID>`: vedo quanto occupa ogni singolo layer del container creato
* `docker volume create <VOLUME-NAME>` (facoltativo, non necessario): crea un volume persistente che non viene rimosso con la rimozione del container
* `docker info`: rivela alcune informazioni sul docker engine installato
* `docker system df`: mostra lo spazio occupato sul Docker Host dalle immagini, dai container e dai volumi. Se appendo il parametro `-v` mostra maggiori dettagli.
* `docker network create --driver <bridge, none or host> --subnet <XXX.YYY.0.0/16> --gateway <WWW.ZZZ.0.1> <NETWORK-NAME>:` crea una nuova rete
* `docker network ls`: elenca tutte le reti disponibili

# Dockerfile -  1 linea = 1 nuovo livello/layer

[https://docs.docker.com/develop/develop-images/dockerfile_best-practices/](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## Istruzioni Dockerfile

1. `FROM <IMAGE-NAME>`: indica da quale immagine partire, quale sarà l'immagine base da espandere
2. `RUN <COMMAND>`: indica il comando da eseguire
3. `COPY <DOCKER-HOST-PATH> <DOCKER-CONTAINER-PATH>`: copia il contenuto di <DOCKER-HOST-PATH> in <DOCKER-CONTAINER-PATH>.
4. `ENTRYPOINT <COMMAND>`: specifica il comando da eseguire per mantenere attivo il container.

## HowTo - Creare una nuova immagine custom per un container

Suggerimento 1: Creare una nuova cartella da condividere con il container per i file di cui non si possono perdere le modifiche.

1. Creare un nuovo container con `docker run -it debian bash`
2. Installare e configurare normalmente tutto il software che si desidera avere nella propria istanza
3. Controllare la `history` dei comandi eseguiti. \
   Questo determinerà le istruzioni da inserire nel Dockerfile sottoforma di istruzioni docker (FROM, RUN, COPY, ENTRYPOINT, …): \

   ```bash
   FROM debian:latest
   USER root
   RUN apt update
   RUN apt install apache2
   ```
   
4. Una volta scritto il Dockerfile lanciare il build con i giusti TAG: \
   `docker build . -t <MY-IMAGE-NAME>:<TAG2> …`
   
5. Creare il proprio container con `docker run <MY-IMAGE-NAME or MY-IMAGE-ID>` o  \
   `docker run -it <MY-IMAGE-NAME or MY-IMAGE-ID> bash` per ottenere il terminale "bash" del container

## HowTo - Caricare la propria immagine custom su Docker Hub

1. Creare il proprio account su Docker Hub ([https://hub.docker.com/](https://hub.docker.com/)) 
2. Spostarsi nella cartella contenente il Dockerfile e lanciare `docker login`
3. Creare l'immagine con `docker build . -t <username>/<my-image>:<TAG1>`
4. Caricare l'immagine sul Docker Hub con `docker push <username>/my-image`

## Docker Compose - Come installare un application stack

Il docker compose consente di comporre un'applicazione unendo diverse immagini docker insieme grazie a un file YAML.

Per poter interagira tra loro i container devono essere dotati di un nome (--name), non potranno raggiungersi.

Linkare 2 container insieme si traduce nell'inserire un record in /etc/hosts avente l'indirizzo IP privato del container e il nome del container come hostname. (?)

La versione di Docker Composer (v1, v2, v3) è specificabile alla prima linea del file `docker-compose.yml`: `version: 3`

Con Docker si deve sempre pensare di dover raggiungere un host esterno, un server esterno e non il _localhost_.

## Installazione (su Debian)

https://docs.docker.com/compose/install/#scenario-two-install-the-docker-compose-plugin-linux-only

## Utilizzo

DOC: [https://docs.docker.com/compose/gettingstarted/](https://docs.docker.com/compose/gettingstarted/)

1. Creare una cartella dove inserire il file `docker-compose.yml`
2. Completare il `docker-compose.yml` con: \
   ```yaml
   services: \
     <nome-servizio-1>: \
       image: xxxxxx \
     <nome-servizio-2>: \
       image: yyyyyy \
       ports:  \
         - 5000:80`
       environment:
         CONSTANT: value
       links:
         - <CONTAINER NAME TO LINK TO>
   ``` 
    
3. Lanciare `docker-compose up` per eseguire l'application stack

## Docker Registry

Luogo in cui vengono depositate e gestite le immagini Docker. Può essere pubblico (docker hub) o privato (installato per conto proprio)

## Docker Storage

Dove e Come Docker memorizza i dati e come gestisce i file system dei containers.

`/var/lib/docker` - Qui Docker memorizza le immagini, containers, …

## Docker Network

Una volta installato docker si hanno a disposizione 3 reti:

1. Bridge (rete privata): `docker run debian`
2. None (nessuna rete): `docker run debian --network=none`
3. Host (rete pubblica): `docker run debian --network=host`

Se viene usata la rete `host` non c'è bisogno del mapping delle porte ottenuto con `-p` nel comando \
perché vengono usate le stesse porte sia dentro che fuori dal container senza alcun isolamento. \
Se un'app usa la porta 5000 nel container, la stessa porta sarà quella esposta pubblicamente se si usa `host`.

Le porte vengono condivise tra tutti i containers e, essendo occupate, non possono essere utilizzate da altri.

## Container Orchestration

Un container orchestration gestisce tutta quella miriade di aspetti legati all'hosting di container in ambiente di produzione, dalla replica alla disponibilità in caso di guasto. \
Un esempio tipico è ospitare container su più Docker Host in modo che se un'applicazione non risulta più raggiungibile da un docker host, \
può esser resa raggiungibile da un altro Docker Host.

Esistono 3 Container Orchestration: Docker Swarm, Kubernetes e Mesos

## Docker Swarm

Docker Swarm permette di combinare più Docker Host in un solo Cluster prendendosi cura della distribuzione delle applicazioni e del loro bilanciamento di carico e disponibilità.

Docker Swarm mette a disposizione degli strumenti per la gestione: \
- della ridondanza (quante “copie” di un container devono essere eseguite e su quali nodi)
- del bilanciamento del carico
- del fail-over (se un nodo del cluster non funziona, i container che eseguiva verranno automaticamente migrati su altri nodi), ecc..

Per usare Docker Swarm si devono avere più Docker Host con installato `docker` al loro interno \
e battezzare chi sarà lo SWARM Master con il `docker swarm init`. 
Il comando fornirà poi cosa lanciare su tutti gli `slave` che si vogliono unire al Master.

Grazie al comando `docker service create --replica=3 my-idp` (eseguito sul Master) \
io vado a creare 3 `my-idp` su 3 differenti docker host e Docker Swarm provvede a distribuirne il carico e la disponibilità.

## Kubernetes

Altro orchestratore oltre a Docker Swarm, ma più potente.
