Kubernetes Notes
================

#. `Contesto`_
#. `Comandi in Kubernetes`_
#. `ETCD (v3)`_

   * `Inserimento di valori`_
   * `Recupero di valori`_
#. `Kube API Server`_
#. `Kube Controller Manager`_
#. `Author`_     

  
Contesto
--------

Kubernetes è governato dalle sue API ed il Kube API Server è il componente responsabile dell'orchestrazione di tutti i componenti interni al Cluster Kubernetes.

Un Cluster Kubernetes consiste in un insieme di nodi che ospitano le applicazioni sottoforma di container.

Un Worker Node è il nodo capace di ospitare le applicazioni sottoforma di container, 
mentre il Master Node si occupa di gestire il caricamento dei container sul/sui Worker Node e del loro monitoraggio.

Un Worker Node viene guidato da un "kubelet" agent che carica o distrugge i container che ospita.

Il Kube API Server tiene monitorato lo stato dei singoli Worker Node attraverso le informazioni che periodicamente riceve dai singoli "kubelet" agent.

La comunicazione tra i container presenti sui vari Worker Node è demandata al Kube Proxy Service di ogni Worker Node.

Il Master Node si compone di diverse parti (che può essere fatto di soli container):

* ETCD:

  * è un database sempre attivo e disponibile che memorizza informazioni sottoforma di oggetti <chiave>:<valore> quali:

    * Su quale/Da quale Worker Node il container è stato caricato/scaricato
    * Giorno e Ora del caricamento/scaricamentole informazioni sui container caricati/scaricati
    * ...

  * kube-scheduler:

    * sceglie il Worker Node giusto in cui inserire un container sulla base di:

      * risorse disponibili al Worker Node
      * risorse richieste dal container
      * ...

  * node-controller:

    * si occupa della gestione dei Worker Node:

      * Inserimento di nuovi Worker Node nel Cluster
      * Gestione dei Worker Node non più disponibili o distrutti

  * replication-controller: 

    * si assicura che i container che si vogliono attivi lo siano sempre e in replica.

Dato che tutto gira sottoforma di container, installiamo Docker su tutti i nodi del cluster, compreso il Master Node (se lo vogliamo gestire con i container).

`[TOP] <#kubernetes-notes>`_

Comandi in Kubernetes
---------------------

Il supporto a Docker è stato rimosso a favore di Containerd, quindi non è possibile usare ``docker run``, ``docker build``, ...

Si deve ricorrere a ``nerdctl`` che è molto simile a ``docker``:

* ``docker run --name ubuntu ubuntu:22.04`` diventa

  ``nerdctl run --name ubuntu ubuntu:22.04``

* ``docker run --name webserver -p 80:80 -d nginx`` diventa

  ``nerdctl run --name webserver -p 80:80 -d nginx``

Il ``cri control utility`` è uno strumento da usare nel caso in cui si voglia fare debugging.
Se creo container con ``crictl`` l'agente ``kubelet`` non riconosce il container e lo rimuove.
Non è da usare nel quotidiano, ma solo all'occorrenza come ultima spiaggia.

`[TOP] <#kubernetes-notes>`_


ETCD (v3)
---------

`ETCD`_ è un database le cui tabelle sono composte da solo 2 colonne: 

+----------+----------+
| **KEY**  |**VALUE** |
+----------+----------+
|  Nome    | Marco    |
+----------+----------+

che memorizza tutte le informazioni riguardanti il Cluster Kubernetes.
Solamente quando l'informazione è stata memorizzata su ETCD può dirsi "completata".

Il parametro ``--advertise-client-urls`` dell'eseguibile di ETCD indica l'indirizzo e la porta che usa
per ricevere le informazioni dal Cluster.
Questo indirizzo va dato al Kube API Server per mettersi in contatto con l'ETCD server.

Kubernetes memorizza i dati con una struttura avente come radice la cartella ``registry/``.

In un sistema di High Availability (HA) si hanno diversi ETCD server sparsi su diversi Master Node
ed è necessario configurare opportunamente l'eseguibile dell'ETCD server con il paramentro ``--initial-cluster``.

ETCDCTL è la CLI (Command Line Interface) usata per interagire con il database ETCD.

Inserimento di valori
"""""""""""""""""""""

.. code-block:: bash

   $ etcdctl put greeting "Hello, etcd"
   OK

Recupero di valori
""""""""""""""""""

.. code-block:: bash

   $ etcdctl get greeting
   greeting
   Hello, etcd

`[TOP] <#kubernetes-notes>`_


Kube API Server
---------------

``kubectl`` è la CLI (Command Line Interface) usata per interagire con il Kube API Server.

Ogni richiesta fatta al Kube API Server è autenticata e validata prima di essere eseguita.

Non è necessario usare ``kubectl`` quando è possibile ottenere lo stesso risultato con una POST come questa:

.. code-block:: HTML

   curl -X POST /api/v1/namespaces/default/pods ...[other]

Cosa succede quando chiedo di creare un POD attraverso le API di Kubernetes?

#. La mia richiesta via API viene autenticata e validata
#. Kube API Server crea un oggetto "pod", ma non lo assegna ad alcun Worker node
#. Kube API Server aggiorna l'ETCD con l'informazione "oggetto pod creato" e l'utente dicendo che il POD è stato creato
#. Kube Scheduler, che monitora di continuo il Kube API server, scopre che c'è un nuovo POD senza Worker node
#. Kube Scheduler, trova il giusto Worker node su cui mettere il POD e lo comunica al Kube API server
#. Kube API Server aggiorna l'ETCD con l'informazione ricevuta dal Kube Scheduler
#. Kube API Server contatta il Kubelet Agent del Worker node indicato dal Kube Scheduler
#. Il Kubelet Agent del Worker node contattato crea il POD e dice al Container Runtime Engine di deployare l'immagine dell'applicazione
#. Una volta deployata l'applicazione, il Kubelet Agent informa il Kube API Server
#. Kube API Server aggiorna l'ETCD con le informazioni passate dal Kubelet Agent del Worker node su cui l'app è stata deployata.

I passi eseguiti sopra vengono ripetuti per ogni modifica applicata al Cluster Kubernetes.

Il parametro ``--etcd-servers`` dell'eseguibile del Kube API Server gli permette di connettersi ai database ETCD da utilizzare.

Se il Kube API server è deployato con ``kubeadmin``, i suoi parametri sono recuperabili dal file:

* ``/etc/kubernetes/manifests/kube-apiserver.yaml``

mentre senza ``kubeadmin`` è necessario guardare i parametri con cui è stato avviato il servizio ``kube-apiserver`` da:

* ``/etc/systemd/system/kube-apiserver.service``

o attraverso il comando:

* ``ps aux | grep kube-apiserver``

`[TOP] <#kubernetes-notes>`_


Kube Controller Manager
-----------------------

Si tratta di un processo che verifica continuamente lo stato dei componenti del Cluster Kubernetes e
lavora per mantenere l'intero sistema allo stato desiderato.

Il Kube Controller Manager contiene tutti i "controller" utilizzati da Kubernetes.

Anche il Kube Controller Manager è un eseguibile di Systemd che ha parametri configurabili come:

* ``--node-monitor-period=5s``
* ``--node-monitor-grace-period=40s``
* ``--pov-eviction-timeout=5m0s``

Node Controller
"""""""""""""""

il Node Controller monitora lo stato dei Worker Node ogni 5 secondi
ed esegue le azioni necessarie per mantenere le applicazioni in esecuzione con l'aiuto del Kube API Server.

Se non riceve più risposta dal Worker Node, il Node Controller si segna che è in uno stato "non raggiungibile/unreachable",
ma è solo dopo ulteriori 40 secondi che il Worker Node viene marcato come "non raggiungibile/unreachable".

Una volta entrato nello stato di "non raggiungibile/unreachable", il Worker Node ha 5 minuti per tornare operativo o
il Node Controller rimuove tutti i suoi POD e li trasferisce su un Worker Node funzionante (se i POD sono parte di un "replica set").

Replica Controller
""""""""""""""""""

Il Replica Controller monitora lo stato dei "replica set"
ed esegue le azioni necessarie per mantenere il numero di POD inalterato nei "replica set".
Se un POD muore, lui ne crea uno nuovo

Se il Kube Controller Manager è deployato con ``kubeadmin``, i suoi parametri sono recuperabili dal file:

* ``/etc/kubernetes/manifests/kube-controller-manager.yaml``

mentre senza ``kubeadmin`` è necessario guardare i parametri con cui è stato avviato il servizio ``kube-controller-manager`` da:

* ``/etc/systemd/system/kube-controller-manager.service``

o attraverso il comando:

* ``ps aux | grep kube-controller-manager``

`[TOP] <#kubernetes-notes>`_


Author
------

* `Marco Malavolti <mailto:marco.malavolti@gmail.com>`_

`[TOP] <#kubernetes-notes>`_


.. _ETCD: https://etcd.io/
