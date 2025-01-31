# Docker

## Indice

1. [Le Basi](./Docker-Notes.md#le-basi)
2. [Comandi](./Docker-Notes.md#comandi-ctrlc-per-terminare)
3. [Dockerfile -  1 linea = 1 nuovo livello/layer](./Docker-Notes.md#dockerfile----1-linea--1-nuovo-livellolayer)
   1. [Istruzioni Dockerfile](./Docker-Notes.md#istruzioni-dockerfile)

## Le Basi

* Un container Docker è l’istanziazione di un’immagine Docker.
* Un’immagine Docker è generabile attraverso un Dockerfile: file di testo che permette di eseguire operazioni.
* Docker è installabile attraverso il Convenience script, ma è preferibile installarlo via repository APT/PKG.
* [https://hub.docker.com](https://hub.docker.com) contiene le immagini Docker già create da altri e pronte ad essere usate.
* Un container vive per il tempo di vita dei processi che in esso girano, ossia vive per il tempo necessario a svolgere il compito richiesto.
* Per uscire da un container bloccato bisogna trovare il suo **<code>&lt;CONTAINER-ID></code></strong> attraverso il comando “<strong><code>docker ps</code></strong>” ed eseguire “<strong><code>docker stop &lt;CONTAINER-ID></code></strong>”
* Ogni nuova immagine Docker si deve basare su una già esistente, quindi ogni Dockerfile DEVE INIZIARE con l’istruzione “<strong><code>FROM &lt;IMAGE-NAME></code></strong>”, dove &lt;IMAGE-NAME> è l’immagine base recuperabile dal Docker Hub.
* Quando creiamo delle immagini personali/custom, dobbiamo nominarle come “<code>&lt;username-docker-hub>/&lt;my-image-name></code>”
* Docker Host o Docker Engine: è la macchina fisica su cui viene installato ed eseguito docker
* Ogni container, per default, ottiene un IP locale raggiungibile solo dal Docker Host. Per permettere agli esterni di utilizzare l’applicazione web che gira all’interno del container è necessario mappare la porta utilizzata dall’applicazione con una libera del Docker Host:

                ```
                sudo docker run -p 443:8443 <IMAGE-NAME>
                ```



    questo comando crea un container da **<code>&lt;IMAGE-NAME></code></strong> e rende disponibile al Docker Host sulla porta <strong><code>443</code></strong> l’applicazione in esecuzione sul container occupante la porta <strong><code>8443</code></strong>.  \
<em>Le porte devono essere libere per essere usate.</em>

* Ogni container mantiene i dati in esso memorizzati fino alla sua rimozione con \
“**<code>sudo docker rm &lt;CONTAINER-ID o CONTAINER-NAME></code></strong>”.
* Il processo di generazione delle proprie immagini è rapido grazie alla cache che mantiene le operazioni già eseguite.
* Non è possibile eseguire 2 container con lo stesso nome.
* Docker ottimizza l’uso dei layer ri-utilizzando il riutilizzabile. In questo modo il build di un’immagine è sempre performante.
* Il comando “<strong><code>docker build</code></strong>” serve per creare un’immagine con cui si può istanziare un container. Qualsiasi nuovo container che usa la medesima immagine, condividerà la cache per la sua istanziazione rendendo più veloce il tutto. Tutto quello che viene scritto in un container, viene cancellato alla sua rimozione.
* Per connettersi via rete ad un container a partire da un altro è necessario usare il suo “<strong>nome</strong>” in quanto l’indirizzo IP della rete privata “bridge” non è garantito che rimanga lo stesso al riavvio.


## Comandi (CTRL+C per terminare)



* **<code>docker run &lt;IMAGE-NAME or IMAGE-ID></code></strong>: crea un container dall’immagine <strong><code>&lt;IMAGE-NAME or IMAGE-ID></code></strong> (che viene scaricata se non presente).
    * <strong><code>sudo docker run -it debian bash</code></strong> \
crea il container docker dall’ultima versione dell’immagine “debian” e apre una shell bash sul container da poter usare sfruttando lo standard input della propria macchina.
    * <strong><code>sudo docker run debian:8.0</code></strong> \
crea il container docker dal TAG “8.0” dell’immagine “debian”
    * <strong><code>sudo docker run -V &lt;PATH-OUTSIDE-DIR>:&lt;PATH-INSIDE-DIR> &lt;IMAGE-NAME or IMAGE-ID></code></strong>

        crea un container in cui la cartella al suo interno “&lt;PATH-INSIDE-DIR>” rimane sincronizzata, nel contenuto, con quella presente sul Docker Host “&lt;PATH-OUTSIDE-DIR>”

    * **<code>sudo docker run --name &lt;CONTAINER-NAME> &lt;IMAGE-NAME or IMAGE-ID> \
</code></strong>crea il container docker da un’immagine e lo nomina “&lt;CONTAINER-NAME>”
    * <strong><code>sudo docker run -e CONSTANT-NAME=value &lt;IMAGE-NAME or IMAGE-ID> \
</code></strong>crea il container docker dall’immagine passando anche la variabile di ambiente CONSTANT-NAME
    * <strong><code>sudo docker run -p 443:8443 &lt;IMAGE-NAME></code></strong>

     	crea un container da <strong><code>&lt;IMAGE-NAME></code></strong> e rende disponibile al Docker Host sulla porta <strong><code>443</code></strong> l’applicazione in  \
	esecuzione sul container che usa la porta <strong><code>8443</code></strong>.

    * **<code>docker run -d &lt;IMAGE-NAME or IMAGE-ID></code></strong> \
esegue il container in background lasciando libero il terminale per eseguire altre operazioni. Per vedere tutti i container in background usare “<strong><code>docker ps</code></strong>” e per riportare i container in foreground usare “<strong><code>docker attach &lt;CONTAINER-ID or CONTAINER-NAME></code></strong>”
    * <strong><code>sudo docker run -d -p 443:8443</code></strong> <strong><code>&lt;IMAGE-NAME or IMAGE-ID> \
</code></strong>esegue il container in background(detached mode) rendendo disponibile sulla 443 l’applicazione che nel container usa la porta 8443
    * <strong><code>docker run -v &lt;VOLUME-PATH>:&lt;CONTAINER-DIRPATH-FOR-VOLUME> &lt;IMAGE-NAME or IMAGE-ID></code></strong>: lega un volume persistente ad una cartella del container. Il contenuto della cartella viene scritto nel volume che non viene distrutto con la distruzione del container.<strong><code> \
 \
docker run -v /data/mysql_volume:/var/lib/mysql mysql \
</code></strong>Questo comando crea un volume persistente legato alla cartella <code>/var/lib/mysql</code> i cui dati<strong> </strong>permangono anche dopo la distruzione del container perché scritti nel volume. \
 \
New style: \
<strong><code>docker run --mount type=bind,source=/data/mysql_volume,target=/var/lib/mysql mysql</code></strong>
* <strong><code>docker ps</code></strong>: elenca tutti i container attivi e utili informazioni
* <strong><code>docker ps -a</code></strong>: elenca tutti i container, anche quelli non più attivi
* <strong><code>docker restart &lt;CONTAINER-ID o CONTAINER-NAME></code></strong>: riavvia un container.
* <strong><code>docker stop &lt;CONTAINER-ID o CONTAINER-NAME></code></strong>: disattiva un container. E’ possibile fermare più container inserendo le prime 2 lettere di ogni CONTAINER-ID separate da uno spazio.
* <strong><code>docker rm &lt;CONTAINER-ID o CONTAINER-NAME></code></strong>: rimuove un container disattivato
* <strong><code>docker images</code></strong>: elenca tutte le immagini disponibili
* <strong><code>docker rmi &lt;IMAGE-NAME or IMAGE-ID></code></strong>: rimuove l’immagine indicata solo se non è usata da un container.
* <strong><code>docker rmi -f &lt;IMAGE-NAME or IMAGE-ID></code></strong>: rimuove l’immagine indicata forzatamente.
* <strong><code>docker pull &lt;IMAGE-NAME or IMAGE-ID></code></strong>: scarica l’immagine dal Docker hub
* <strong><code>docker exec &lt;CONTAINER-ID o CONTAINER-NAME> &lt;COMMAND-TO-RUN-IN-THE-CONTAINER></code></strong>: esegue un comando nel container attivo indicato da <strong><code>&lt;CONTAINER-ID or CONTAINER-NAME></code></strong> individuabile con “<strong><code>docker ps</code></strong>”
* <strong><code>docker inspect &lt;CONTAINER-ID o CONTAINER-NAME></code></strong>: restituisce informazioni sul container, ENV variables
* <strong><code>docker logs &lt;CONTAINER-ID o CONTAINER-NAME></code></strong>: restituisce lo standard output del container
* <strong><code>docker build path/Dockerfile -t &lt;username-docker-hub>/myImage</code></strong>: crea l’immagine a partire dal Dockerfile.
* <strong><code>docker history &lt;IMAGE-NAME or IMAGE-ID></code></strong>: vedo quanto occupa ogni singolo layer del container creato.
* <strong><code>docker volume create &lt;VOLUME-NAME></code></strong> (facoltativo, non necessario): crea un volume persistente che non viene rimosso con la rimozione del container. 
* <strong><code>docker info</code></strong>: rivela alcune informazioni sul docker installato nel Docker Host.
* <strong><code>docker system df</code></strong>: mostra lo spazio occupato sul Docker Host dalle immagini, dai container e dai volumi. Se appendo il parametro “-v” mostra maggiori dettagli.
* <strong><code>docker network create --driver &lt;bridge, none or host> --subnet &lt;XXX.YYY.0.0/16> --gateway &lt;WWW.ZZZ.0.1> &lt;NETWORK-NAME>:</code></strong> crea una nuova rete
* <strong><code>docker network ls</code></strong>: elenca tutte le reti disponibili


# Dockerfile -  1 linea = 1 nuovo livello/layer

[https://docs.docker.com/develop/develop-images/dockerfile_best-practices/](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)


## Istruzioni Dockerfile



1. **<code>FROM &lt;IMAGE-NAME></code></strong>: indica da quale immagine partire, l’immagine base
2. <strong><code>RUN &lt;COMMAND></code></strong>: indica il comando da eseguire
3. <strong><code>COPY &lt;DOCKER-HOST-PATH> &lt;DOCKER-CONTAINER-PATH></code></strong>: copia il contenuto di &lt;DOCKER-HOST-PATH> in &lt;DOCKER-CONTAINER-PATH>.
4. <strong><code>ENTRYPOINT &lt;COMMAND></code></strong>: specifica il comando da eseguire quando l’immagine diventerà container.


## HowTo - Creare una nuova immagine custom per un container

<span style="text-decoration:underline;">Suggerimento 1:</span> Creare una nuova cartella da condividere con il container per i file di cui non si possono perdere le modifiche.

**ATTENZIONE:** il nome delle immagini non possono contenere lettere MAIUSCOLE



1. Creare un nuovo container con “**<code>sudo docker run -it debian bash</code></strong>”
2. Installare e configurare normalmente tutto il software che si desidera avere nella propria istanza
3. Controllare la “<strong><code>history</code></strong>” dei comandi eseguiti. Questo determina le istruzioni da riportare nel Dockerfile con le istruzioni docker (FROM, RUN, COPY, ENTRYPOINT, …): \
FROM debian:latest \
USER root \
RUN apt update \
RUN apt install apache2
4. Una volta scritto il Dockerfile lanciare il build con i giusti TAG:  \
<strong><code>docker build . -t &lt;MY-IMAGE-NAME>:&lt;TAG2> …</code></strong>
5. Creare il proprio container con “<strong><code>docker run &lt;MY-IMAGE-NAME or MY-IMAGE-ID></code></strong>” o  \
“<strong><code>docker run -it &lt;MY-IMAGE-NAME or MY-IMAGE-ID> bash</code></strong>”


## HowTo - Caricare la propria immagine custom su Docker Hub



1. Creare il proprio account su Docker Hub ([https://hub.docker.com/](https://hub.docker.com/)) 
2. Spostarsi nella cartella contenente il DockerFile e lanciare “**<code>docker login</code></strong>”
3. Creare l’immagine con: “docker build . -t &lt;username>/&lt;my-image>:&lt;TAG1>”
4. Caricare l’immagine sul Docker Hub con “docker push &lt;username>/my-image”


# Docker Compose - Come installare un application stack

Il docker compose consente di comporre un applicazione unendo diverse immagini docker insieme grazie a un file YAML.

Per poter comunicare tra loro, i container devono essere “linkati” altrimenti non sanno con chi parlare e per essere linkati ad ogni container deve essere dato un nome (--name)

Linkare 2 container insieme si traduce nell’inserire un record in /etc/hosts avente l’indirizzo IP privato del container e il nome del container come hostname.

Usando il Docker Compose v2 non è necessario indicare i link ai vari container ed è possibile indicare quale container deve essere attivo per poterne usare un altro.

La versione di Docker Composer (v1, v2, v3) è specificabile alla prima linea del file “docker-compose.yml”: “version: 3”

Con Docker si deve sempre pensare di dover raggiungere un host esterno, un server esterno e non il Localhost.


## Installazione (su Debian) - [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)



1. `sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
2. `sudo chmod +x /usr/local/bin/docker-compose`
3. `cd /etc/bash_completion.d/`
4. `sudo curl -L https://raw.githubusercontent.com/docker/compose/1.26.2/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose`


## Utilizzo - [https://docs.docker.com/compose/gettingstarted/](https://docs.docker.com/compose/gettingstarted/)



1. Creare una cartella dove inserire il “**<code>docker-compose.yml"</code></strong>
2. Completare il “<strong><code>docker-compose.yml</code></strong>” con: \
<strong><code>version: '3.0' \
services: \
  &lt;nome-servizio-1>: \
    image: xxxxxx \
  &lt;nome-servizio-2>: \
    image: yyyyyy \
    ports:  \
      - 5000:80</code></strong>

    ```
        environment:
          CONSTANT: value
        links:
          - <CONTAINER NAME TO LINK TO>
    ```


3. Lanciare “**<code>docker-compose up</code></strong>” per installare l’application stack


# Docker Registry

Luogo in cui vengono depositate e gestite le immagini Docker. Può essere pubblico (docker hub) o privato (installato per conto proprio)


# Docker Storage

Dove e Come Docker memorizza i dati e come gestisce i file system dei containers.

`/var/lib/docker` - Qui Docker memorizza le immagini, containers, …


# Docker Network

Una volta installato docker si hanno a disposizione 3 reti:



1. Bridge (rete privata): **<code>sudo docker run debian</code></strong>
2. None (nessuna rete): <strong><code>sudo docker run debian --network=none</code></strong>
3. Host (rete pubblica): <strong><code>sudo docker run debian --network=host</code></strong>

Se viene usata la rete “<strong>host</strong>” non c’è bisogno del mapping delle porte ottenuto con il tag “<strong>-p</strong>” nel comando perché vengono usate le stesse porte sia dentro che fuori dal container senza alcun isolamento. Se un'app usa la porta 5000 nel container, la stessa porta sarà quella esposta pubblicamente se viene utilizzata la rete “host”.

Le porte vengono così condivise tra tutti i containers e, essendo occupate, non possono essere utilizzate da altri.


# Container Orchestration

Insieme di strumenti per hostare containers in un ambiente di produzione. Un esempio tipico è hostare i container su più docker host in modo che se un’applicazione non risulta più raggiungibile da un docker host, viene resa raggiungibile da un altro docker host.

Un container orchestration gestisce tutta quella miriade di aspetti legati all’hosting di container in un ambiente produttivo, dalla replica alla disponibilità in caso di guasto.

Esistono 3 Container Orchestration: Docker Swarm, Kubernetes e Mesos


## Docker Swarm

Docker Swarm permette di combinare più Docker Host in un solo Cluster prendendosi cura della distribuzione delle applicazioni e del loro bilanciamento di carico e disponibilità.

Per usare Docker Swarm si devono avere più Docker Host con installato “docker” al loro interno e battezzare chi sarà il master (SWARM Master) con il comando “docker swarm init”. Il comando fornirà poi cosa lanciare su tutti gli “slave” che si vogliono unire al Master.

Grazie al comando “`docker service create --replica=3 my-idp`” (eseguito sul Master) io vado a creare 3 “`my-idp`” su 3 differenti docker host e Docker Swarm provvede a distribuirne il carico e la disponibilità


## Kubernetes

Altro orchestratore oltre a Docker Swarm, ma più potente.
