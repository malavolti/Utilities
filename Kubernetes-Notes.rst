Kubernetes Notes
================

#. `Contesto`_
#. `Comandi in Kubernetes`_
#. `ETCD (v3)`_

   * `Inserimento di valori`_
   * `Recupero di valori`_
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

`[TOP] <##kubernetes-notes>`_

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

`[TOP] <##kubernetes-notes>`_


ETCD (v3)
---------

`ETCD`_ è un database le cui tabelle sono composte da solo 2 colonne: 

+----------+----------+
| **KEY**  |**VALUE** |
+----------+----------+
|  Nome    | Marco    |
+----------+----------+

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

`[TOP] <##kubernetes-notes>`_

Author
------

* `Marco Malavolti <mailto:marco.malavolti@gmail.com>`_

`[TOP] <##kubernetes-notes>`_


.. _ETCD: https://etcd.io/
