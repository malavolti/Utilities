============
Docker Notes
============

++++++
Indice
++++++

#. `Le Basi <https://github.com/malavolti/Utilities/new/master#le-basi>`_
#. `Comandi <https://github.com/malavolti/Utilities/new/master#comandi>`_

-------
Le Basi
-------

* Un container Docker è l’istanziazione di un’immagine Docker.
* Un’immagine Docker è generabile attraverso un Dockerfile: file di testo che permette di eseguire operazioni.
* Docker è installabile attraverso il Convenience script, ma è preferibile installarlo via repository APT/PKG.
* https://hub.docker.com contiene le immagini Docker già create da altri e pronte ad essere usate.
* Un container vive per il tempo di vita dei processi che in esso girano, ossia vive per il tempo necessario a svolgere il compito richiesto.
* | Per uscire da un container bloccato bisogna trovare il suo ``<CONTAINER-ID>`` attraverso il comando ``docker ps`` 
  | ed eseguire ``docker stop <CONTAINER-ID>``
* | Ogni nuova immagine Docker si deve basare su una già esistente, 
  | quindi ogni Dockerfile **DEVE INIZIARE** con l’istruzione ``FROM <IMAGE-NAME>``, 
  | dove ``<IMAGE-NAME>`` è l’immagine base recuperabile dal Docker Hub.
* Quando creiamo delle immagini personali/custom, dobbiamo nominarle come ``<username-docker-hub>/<my-image-name>``
* Docker Host o Docker Engine: è la macchina fisica su cui viene installato ed eseguito docker
* | Ogni container, per default, ottiene un IP locale raggiungibile solo dal Docker Host. Per permettere agli esterni di utilizzare l’applicazione web che gira all’interno del container è necessario mappare la porta utilizzata dall’applicazione con una libera del Docker Host:
  |
  | ``sudo docker run -p 443:8443 <IMAGE-NAME>``
  |
  | questo comando crea un container da ``<IMAGE-NAME>`` e rende disponibile al Docker Host sulla porta 443 l’applicazione in esecuzione sul container occupante la porta 8443. 
  | *Le porte devono essere libere per essere usate.*
* Ogni container mantiene i dati in esso memorizzati fino alla sua rimozione con ``sudo docker rm <CONTAINER-ID o CONTAINER-NAME>``.
* Il processo di generazione delle proprie immagini è rapido grazie alla cache che mantiene le operazioni già eseguite.
* Non è possibile eseguire 2 container con lo stesso nome.
* Docker ottimizza l’uso dei layer ri-utilizzando il riutilizzabile. In questo modo il build di un’immagine è sempre performante.
* Il comando ``docker build`` serve per creare un’immagine con cui si può istanziare un container. Qualsiasi nuovo container che usa la medesima immagine, condividerà la cache per la sua istanziazione rendendo più veloce il tutto. Tutto quello che viene scritto in un container, viene cancellato alla sua rimozione.
* Per connettersi via rete ad un container a partire da un altro è necessario usare il suo **nome** in quanto l’indirizzo IP della rete privata *bridge* non è garantito che rimanga lo stesso al riavvio.


-------
Comandi
-------

