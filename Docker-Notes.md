# Docker Notes

## Indice

1. [Le Basi](./Docker-Notes.md#le-basi)
2. [Comandi](./Docker-Notes.md#comandi)

### Le Basi

* Un container Docker è l’istanziazione di un’immagine Docker.
* Un’immagine Docker è generabile attraverso un `Dockerfile`: file di testo che permette di eseguire operazioni.
* Docker è installabile attraverso il Convenience script, ma è preferibile installarlo via repository `apt`/`pkg`.
* <https://hub.docker.com> contiene le immagini Docker già create da altri e pronte per essere usate.
* Un container vive per il tempo di vita dei processi che in esso girano, ossia vive per il tempo necessario a svolgere il compito richiesto.
* Per uscire da un container bloccato bisogna:
  1. trovare il suo **<CONTAINER-ID>** attraverso il comando `docker ps`
  2. ed eseguire `docker stop <CONTAINER-ID>`.
* Ogni nuova immagine Docker si deve basare su una già esistente,  
  quindi ogni Dockerfile **DEVE INIZIARE** con l’istruzione `FROM <IMAGE-NAME>`,  
  dove `<IMAGE-NAME>` è l’immagine base recuperabile dal Docker Hub.
* Quando creiamo delle immagini personali/custom, dobbiamo nominarle come `<username-docker-hub>/<my-image-name>`.
* Docker Host o Docker Engine: è la macchina fisica su cui viene, di fatto, installato ed eseguito docker.
* Ogni container, in modo predefinito, ottiene un IP locale raggiungibile solo dal Docker Host.
  Per permettere agli esterni di utilizzare l’applicazione web che gira all’interno del container è necessario mappare la porta utilizzata dall’applicazione con una porta libera del Docker Host:
  * `sudo docker run -p 443:8443 <IMAGE-NAME>`  
    questo comando crea un container da ``<IMAGE-NAME>`` e rende disponibile al Docker Host sulla porta 443  
    l’applicazione in esecuzione sul container occupante la porta 8443.  
    *Le porte sul Docker Host devono essere libere per essere occupate.*
* Ogni container mantiene i dati in esso memorizzati fino alla sua rimozione con `sudo docker rm <CONTAINER-ID o CONTAINER-NAME>`.
* Il processo di generazione delle proprie immagini è rapido grazie alla cache che mantiene le operazioni già eseguite.
* Non è possibile eseguire 2 container con lo stesso nome.
* Docker ottimizza l’uso dei layer ri-utilizzando il riutilizzabile. In questo modo il build di un’immagine è sempre performante.
* Il comando `docker build` serve per creare un’immagine con cui si può istanziare un container.  
  Qualsiasi nuovo container che usa la medesima immagine, condividerà la cache per la sua istanziazione rendendo più veloce il tutto.  
  Tutto quello che viene scritto in un container, viene cancellato alla sua rimozione.
* Per connettersi via rete ad un container a partire da un altro è necessario usare il suo **nome** in quanto l’indirizzo IP della rete privata *bridge* non è garantito che rimanga lo stesso al riavvio.


### Comandi (CTRL+C per terminare)

* `docker run <IMAGE-NAME or IMAGE-ID>`:  
  crea un container dall’immagine `<IMAGE-NAME or IMAGE-ID>` (che viene scaricata se non presente). Necessita di `sudo` se non si usa la [Rootless Mode](https://docs.docker.com/engine/security/rootless/).
  * `sudo docker run -it debian bash`:  
    crea il container docker dall’ultima versione dell’immagine **debian** e apre una shell Bash sul container pronta da usare.
  * `sudo docker run debian:8.0`  
    crea il container docker dal TAG **8.0** dell’immagine **debian**
  * `sudo docker run -V <PATH-OUTSIDE-DIR>:<PATH-INSIDE-DIR> <IMAGE-NAME or IMAGE-ID>`  
    crea un container in cui la cartella al suo interno `<PATH-INSIDE-DIR>` rimane sincronizzata, nel contenuto, con quella presente sul Docker Host `<PATH-OUTSIDE-DIR>`.
  * `sudo docker run --name <CONTAINER-NAME> <IMAGE-NAME or IMAGE-ID>`  
    crea il container docker da un’immagine e lo nomina `<CONTAINER-NAME>`
  * `sudo docker run -e CONSTANT-NAME=value <IMAGE-NAME or IMAGE-ID>`  
    crea il container docker dall’immagine passando anche la variabile di ambiente CONSTANT-NAME
  * `sudo docker run -p 443:8443 <IMAGE-NAME>`  
 	  crea un container da <IMAGE-NAME> e rende disponibile al Docker Host sulla porta 443 l’applicazione in esecuzione sul container che usa la porta 8443.
  * `docker run -d <IMAGE-NAME or IMAGE-ID>`  
    esegue il container in background lasciando libero il terminale per eseguire altre operazioni. Per vedere tutti i container in background usare “docker ps” e per riportare i container in foreground usare
