# Kubernetes Notes

1.  [Contesto](#contesto)
2.  [Comandi in Kubernetes](#comandi-in-kubernetes)
3.  [ETCD (v3)](#etcd-v3)
    -   [Inserimento di valori](#inserimento-di-valori)
    -   [Recupero di valori](#recupero-di-valori)
4.  [Kube API Server](#kube-api-server)
5.  [Kube Controller Manager](#kube-controller-manager)
    -   [Node Controller](#node-controller)
    -   [Replication Controller & Replica Set](#replication-controller--replica-set)
        -   [Replication Controller](#replication-controller)
        -   [Replica Set](#replica-set)
6.  [Kube Scheduler](#kube-scheduler)
7.  [Kubelet](#kubelet)
8.  [Kube Proxy](#kube-proxy)
9.  [Kubernetes PODs](#kubernetes-pods)
    -   [Creare POD in YAML](#creare-pod-in-yaml)
10. [Kubernetes Deployment](#kubernetes-deployment)
11. [Kubernetes Services](#kubernetes-services)
    -   [NodePort Kubernetes service](#nodeport-kubernetes-service)
    -   [ClusterIP Kubernetes service](#clusterip-kubernetes-service)
    -   [LoadBalancer Kubernetes service](#loadbalancer-kubernetes-service)
12. [Namespaces](#namespaces)
13. [Imperativo VS Dichiarativo](#imperativo-vs-dichiarativo)
14. [Kubectl Apply](#kubectl-apply)
15. [Scheduling](#scheduling)
    - [Manual Scheduling](#manual-scheduling)
16. [Labels & Selectors](#labels--selectors)
17. [Author](#author)

## Contesto

Un Cluster Kubernetes è governato da API ed il Kube API Server è il
componente responsabile al coordinamento di tutti gli elementi che lo
compongono.

Un Cluster Kubernetes si compone di un'insieme di server (nodi) che
ospitano le applicazioni sottoforma di container.

Un Worker Node è un server capace di ospitare applicazioni sottoforma di
container, mentre il Master Node gestisce il caricamento dei container
sui Worker Node e li monitorara.

Ogni Worker Node possiede un "kubelet" agent e lo usa per:

-   unirsi al Cluster Kubernetes
-   ottenere la lista dei container da ospitare
-   eseguire i container necessari
-   distruggere i container non necessari
-   riportare lo stato dei container al Master Node
-   riportare lo stato del Worker Node stesso al Kube API Server

Il Kube API Server tiene monitorato lo stato dei singoli Worker Node
attraverso le informazioni che periodicamente riceve dai singoli
"kubelet" agent.

La comunicazione tra i container presenti sui vari Worker Node è gestita
dal Kube Proxy Service di ogni Worker Node.

Il Master Node si compone di diverse parti (che possono essere fatte di
soli container):

-   ETCD:
    -   è un database sempre attivo e disponibile che memorizza le
        informazioni sottoforma di oggetti \<chiave\>:\<valore\>. Ad
        esempio:
        -   Su quale/Da quale Worker Node il container è stato
            caricato/scaricato
        -   Giorno e Ora del caricamento/scaricamento delle informazioni
            sui container caricati/scaricati
        -   ...
-   kube-scheduler:
    -   sceglie il Worker Node giusto in cui inserire un container sulla
        base di:
        -   risorse disponibili nel Worker Node
        -   risorse richieste dal container
        -   ...
-   node-controller:
    -   gestisce i Worker Node:
        -   Inserendo i nuovi Worker Node nel Cluster
        -   Rimuovendo i Worker Node non più disponibili o distrutti
-   replication-controller:
    -   si assicura che i container che si vogliono attivi lo siano
        sempre e in replica.

Dato che tutto gira sottoforma di container, installiamo Docker su tutti
i nodi del cluster, compreso il Master Node (se lo vogliamo gestire con
i container).

Kubernetes usa YAML files per creare qualsiasi cosa: POD, repliche,
deployments, services, ....

[\[TOP\]](#kubernetes-notes)

## Comandi in Kubernetes

Il supporto a Docker è stato rimosso a favore di Containerd, quindi non
è possibile usare `docker run`, `docker build`, ...

Si deve ricorrere a `nerdctl` che è molto simile a `docker`:

-   ``` bash
    docker run --name ubuntu ubuntu:22.04 
    ```

    diventa

    ``` bash
    nerdctl run --name ubuntu ubuntu:22.04
    ```

-   ``` bash
    docker run --name webserver -p 80:80 -d nginx
    ```

    diventa

    ``` bash
    nerdctl run --name webserver -p 80:80 -d nginx
    ```

Il Container Runtime Interface control utility (`cri control utility`) è
uno strumento da usare nel caso in cui si voglia fare debugging. Se creo
container con `crictl` l'agente `kubelet` non riconosce il container e
lo rimuove. Non è da usare nel quotidiano, ma solo all'occorrenza come
ultima spiaggia.

[\[TOP\]](#kubernetes-notes)

## ETCD (v3)

[ETCD](https://etcd.io/) è un database le cui tabelle sono composte da solo 2 colonne:

| KEY  | VALUE |
|------|-------|
| Nome | Marco |

che memorizza tutte le informazioni riguardanti il Cluster Kubernetes.
Solamente quando l'informazione è stata memorizzata su ETCD può dirsi
"completata".

Il parametro `--advertise-client-urls` dell'eseguibile di ETCD indica
l'indirizzo e la porta utilizzati per ricevere le informazioni dal
Cluster. Questo indirizzo va dato al Kube API Server per mettersi in
contatto con l'ETCD server.

Kubernetes memorizza i dati con una struttura avente come radice la
cartella `registry/`.

In un sistema di High Availability (HA) si hanno diversi ETCD server
sparsi su diversi Master Node ed è necessario configurare opportunamente
l'eseguibile dell'ETCD server con il paramentro `--initial-cluster`.

ETCDCTL è la CLI (Command Line Interface) usata per interagire con il
database ETCD.

### Inserimento di valori

``` bash
$ etcdctl put greeting "Hello, etcd"
OK
```

### Recupero di valori

``` bash
$ etcdctl get greeting
greeting
Hello, etcd
```

[\[TOP\]](#kubernetes-notes)

## Kube API Server

`kubectl` è la CLI (Command Line Interface) usata per interagire con il
Kube API Server.

Ogni richiesta fatta al Kube API Server è autenticata e validata prima
di essere eseguita.

Non è necessario usare `kubectl` quando è possibile ottenere lo stesso
risultato con una POST come questa:

``` HTML
curl -X POST /api/v1/namespaces/default/pods ...[other]
```

Cosa succede quando chiedo di creare un POD attraverso le API di
Kubernetes?

1.  La mia richiesta via API viene autenticata e validata
2.  Kube API Server crea un oggetto "pod", ma non lo assegna ad alcun
    Worker node
3.  Kube API Server aggiorna l'ETCD con l'informazione "oggetto pod
    creato" e l'utente dicendo che il POD è stato creato
4.  Kube Scheduler, che monitora di continuo il Kube API server, scopre
    che c'è un nuovo POD senza Worker node
5.  Kube Scheduler, trova il giusto Worker node su cui mettere il POD e
    lo comunica al Kube API server
6.  Kube API Server aggiorna l'ETCD con l'informazione ricevuta dal Kube
    Scheduler
7.  Kube API Server contatta il Kubelet Agent del Worker node indicato
    dal Kube Scheduler
8.  Il Kubelet Agent del Worker node contattato crea il POD e dice al
    Container Runtime Engine di deployare l'immagine dell'applicazione
9.  Una volta deployata l'applicazione, il Kubelet Agent informa il Kube
    API Server
10. Kube API Server aggiorna l'ETCD con le informazioni passate dal
    Kubelet Agent del Worker node su cui l'app è stata deployata.

I passi eseguiti sopra vengono ripetuti per ogni modifica applicata al
Cluster Kubernetes.

Il parametro `--etcd-servers` dell'eseguibile del Kube API Server gli
permette di connettersi ai database ETCD da utilizzare.

Se il Kube API server è deployato con `kubeadmin`, i suoi parametri sono
recuperabili dal file:

-   `/etc/kubernetes/manifests/kube-apiserver.yaml`

mentre senza `kubeadmin` è possibile guardare i parametri con cui è
stato avviato il servizio `kube-apiserver` da:

-   `/etc/systemd/system/kube-apiserver.service`

o attraverso il comando:

-   `ps aux | grep kube-apiserver`

[\[TOP\]](#kubernetes-notes)

## Kube Controller Manager

Si tratta di un processo che verifica continuamente lo stato dei
componenti del Cluster Kubernetes e lavora per mantenere l'intero
sistema allo stato desiderato.

Il Kube Controller Manager contiene tutti i "controller" utilizzati da
Kubernetes.

Anche il Kube Controller Manager è un eseguibile di Systemd che ha
parametri configurabili come:

-   `--node-monitor-period=5s`
-   `--node-monitor-grace-period=40s`
-   `--pov-eviction-timeout=5m0s`

Se il Kube Controller Manager è deployato con `kubeadmin`, i suoi
parametri sono recuperabili dal file:

-   `/etc/kubernetes/manifests/kube-controller-manager.yaml`

mentre senza `kubeadmin` è possibile guardare i parametri con cui è
stato avviato il servizio `kube-controller-manager` da:

-   `/etc/systemd/system/kube-controller-manager.service`

o attraverso il comando:

-   `ps aux | grep kube-controller-manager`

### Node Controller

il Node Controller monitora lo stato dei Worker Node ogni 5 secondi ed
esegue le azioni necessarie per mantenere le applicazioni in esecuzione
con l'aiuto del Kube API Server.

Se non riceve più risposta dal Worker Node, il Node Controller si segna
che è in uno stato "non raggiungibile/unreachable", ma è solo dopo
ulteriori 40 secondi che il Worker Node viene marcato come "non
raggiungibile/unreachable".

Una volta entrato nello stato di "non raggiungibile/unreachable", il
Worker Node ha 5 minuti per tornare operativo o il Node Controller
rimuove tutti i suoi POD e li trasferisce su un Worker Node funzionante
(se i POD sono parte di un "replica set").

### Replication Controller & Replica Set

Il Replication Controller(old way) o il Replica Set(new way) monitora il
numero di POD attivi ed mantiene il numero di repliche stabilito
inalterato. Se un POD muore, lui ne crea subito uno nuovo. Questo
permette di non perdere mai l'accesso alle applicazioni web e di
sviluppare l'HA(High Availability) per il Cluster Kuberbernetes.

Il Replication Controller o il Replica Set si occupa anche del
Bilanciamento del Carico (Load Balancing) e della Scalabilità (Scaling).
Se il numero di richieste ad un POD aumentano perchè il numero di utenti
che lo usano aumenta, il Replication Controller o il Replica Set crea
repliche del POD sul Worker Node per bilanciare il carico di lavoro e
mantiene prestante la risposta dell'applicazione. Se le risorse di un
Worker Node non bastano più a soddisfare le richieste inviate
all'applicazione, il Replication Controller o il Replica Set sceglie un
altro Worker Node con abbastanza risorse e crea in esso le repliche
necessarie a garantisce la scalabilità della gestione su altri Worker
Node.

#### Replication Controller

Sostituito dai [Replica Set](#replica-set).

1.  Creare un File YAML che definisce il Replication Controller (ad
    esempio: `my-rc-1.yml`) con:
    1.  `apiVersion`: versione delle API di Kubernetes

    2.  `kind`: tipo di oggetto da creare

    3.  `metadata`: dizionario che contiene, in modo annidato, le
        informazioni proprie del Replication Controller (name, label,
        ...).

        Il numero di spazi usati per indentare/annidare i valori nel
        dizionario deve essere sempre uguale.

        Aggiungendo `type: front-end` al dizionario `labels` sarà
        possibile distinguere i Replication Controller specifici per il
        frontend.

    4.  `spec`: cosa metto nell'oggetto che sto per creare.

        Nel caso del Replication Controller, `spec` è un template del
        POD da replicare composto da `metadata` e `spec`.

        ``` yaml
        apiVersion: v1
        kind: ReplicationController
        metadata:
          name: my-rc-1
          labels:
            app: my-rc-app-1
            type: front-end
        spec:
          template:
            metadata:
              name: my-pod-1
              labels:
                app: my-app-1
                type: front-end
            spec:
              containers:
                - name: nginx-container
                  image: nginx
          replicas: 3
        ```

        dentro a `image`, se non si usa Docker Hub, deve essere inserito
        tutto il path dell'immagine, mentre `template` e `replicas` sono
        fratelli e hanno la stessa indentazione.

        Il campo facoltativo `selector`, fratello di `template` e
        `replicas`, serve per indicare al Replication Controller quali
        POD considerare, dato che può gestire POD al di fuori della sua
        definizione e creati precedentemente.
2.  Eseguire il comando:
    -   `kubectl create -f my-rc-1.yml` oppure
        `kubectl apply -f my-rc-1.yml`

Per vedere tutti i Replication Controller creati usare il comando:

-   `kubectl get replicationcontrollers`

Per vedere tutti i POD creati dal Replication Controller creati usare il
comando:

-   `kubectl get pods`

#### Replica Set

Processo che Monitora e Gestisce le repliche dei POD sui Worker Node del
Cluster Kubernetes.

1.  Creare un File YAML che definisce il Replica Set (ad esempio:
    `my-rs-1.yml`) con:
    1.  `apiVersion`: versione delle API di Kubernetes

    2.  `kind`: tipo di oggetto da creare

    3.  `metadata`: dizionario che contiene, in modo annidato, le
        informazioni proprie del Replica Set (name, label, ...).

        Il numero di spazi usati per indentare/annidare i valori nel
        dizionario deve essere sempre uguale.

        Aggiungendo `type: front-end` al dizionario `labels` sarà
        possibile distinguere i Replica Set specifici per il frontend.

    4.  `spec`: cosa metto nell'oggetto che sto per creare.

        Nel caso del Replica Set, `spec` è un template del POD da
        replicare composto da `metadata` e `spec`.

        ``` yaml
        apiVersion: apps/v1
        kind: ReplicaSet
        metadata:
          name: my-rs-1
          labels:
            app: my-rs-app-1
            type: front-end
        spec:
          template:
            metadata:
              name: my-pod-1
              labels:
                app: my-app-1
                type: front-end
            spec:
              containers:
                - name: nginx-container
                  image: nginx
          replicas: 3
          selector:
            matchLabels:
              type: front-end
        ```

        dentro a `image`, se non si usa Docker Hub, deve essere inserito
        tutto il path dell'immagine, mentre `template`, `replicas` e
        `selector` sono fratelli e hanno la stessa indentazione.

        Il campo `selector` aggiuntivo serve per indicare al Replica Set
        quali POD considerare, dato che può gestire POD al di fuori
        della sua definizione e creati precedentemente.
2.  Eseguire il comando:
    -   `kubectl create -f my-rs-1.yml` oppure
        `kubectl apply -f my-rs-1.yml`

Per modificare un Replica Set usare uno dei comandi che seguono:

-   `kubectl edit replicaset <replicaset-name>`
-   `kubectl edit rs <replicaset-name>`

Per vedere tutti i Replica Set creati usare uno dei comandi che seguono:

-   `kubectl get replicasets`
-   `kubectl get rs`

Per vedere i dettagli di un `replicaset` avviato usare uno dei comandi
che seguono:

-   `kubectl describe replicaset <replicaset name>`
-   `kubectl describe rs <replicaset name>`

Per vedere tutti i POD creati dal Replication Controller creati usare il
comando:

-   `kubectl get pods`

Perchè è utile assegnare le `labels` ai POD o agli oggetti in
Kubernetes?

Perchè le label fungono da guida ai Replica Set che attraverso
`matchLabels` trovano i POD da monitorare.

Come posso scalare il numero di repliche di un Replica Set?

-   Modo 1 - Modificando il numero di `replicas` sul file YAML
    `my-rs-1.yml` prima di lanciare `kubectl replace -f my-rs-1.yml`
-   Modo 2 - Settando il numero di `replicas` del comando
    `kubectl scale --replicas=6 -f my-rs-1.yml`

Come posso eliminare un Replica Set?

-   `kubectl delete -f my-rs-1.yml` (modo 1 - modifico prima il file
    `my-rs-1.yml`)
-   `kubectl delete replicaset my-rs-1` o
    `kubectl delete rs my-rs-1`(modo 2 - non modifico alcun file)

Come visualizzo il manuale delle replicaset?

-   `kubectl explain replicaset`

[\[TOP\]](#kubernetes-notes)

## Kube Scheduler

Il Kube Scheduler è responsabile della schedulazione dei POD sui Worker
Node, ovvero, decide quale POD va su quale Worker Node in base ai
requisiti del POD.

Il Worker node selezionato sarà quello che potrà ospitare meglio il POD
sulla base dei criteri usati dallo Scheduler nella scelta.

I criteri per la scelta del Worker Node a cui destinare i POD sono
personalizzabili.

**Non carica alcun POD sul Worker Node, cosa che invece farà il Kubelet
Agent del Worker Node scelto.**

Se il Kube Scheduler è deployato con `kubeadmin`, i suoi parametri sono
recuperabili dal file:

-   `/etc/kubernetes/manifests/kube-scheduler.yaml`

mentre senza `kubeadmin` è possibile guardare i parametri con cui è
stato avviato il servizio `kube-scheduler` da:

-   `/etc/systemd/system/kube-scheduler.service`

o attraverso il comando:

-   `ps aux | grep kube-scheduler`

[\[TOP\]](#kubernetes-notes)

## Kubelet

Kubelet si occupa di:

-   registrare il Worker Node sul Kubernetes Cluster
-   contattare il Container Runtime Engine per deployare un container, o
    un POD, e renderlo attivo
-   monitorare continuamente lo stato dei container e dei POD
-   riportare tutto al Kube API Server

Il Kubelet Agent va sempre installato manualmente su ogni Worker Node,
anche se si utilizza `kubeadmin`.

I parametri del Kubelet Agent sono recuperabili dal file attraverso il
comando:

-   `ps aux | grep kubelet`

[\[TOP\]](#kubernetes-notes)

## Kube Proxy

In un Cluster Kubernetes, ogni POD può raggiungere un altro POD ovunque
esso sia grazie ad una rete virtuale interna.

Un POD può dunque raggiungere un altro POD attraverso il suo indirizzo
IP, ma gli indirizzi IP non sono persistenti e non si può avere la
certezza che rimangano sempre gli stessi.

Kube Proxy è un processo eseguito su ogni Worker Node che controlla la
comparsa di nuovi servizi e per ogni nuovo servizio creato, genera le
regole di instradamento del traffico su ogni Worker Node che servono per
raggiungerlo. Questo obiettivo si può raggiungere con `iptables`.

Se il Kube Proxy è deployato con `kubeadmin`, verrà inserito su ogni
Worker Node sottoforma di POD:

-   `kubectl get pods -n kube-system`

mentre senza `kubeadmin` è possibile recuperare i parametri con cui è
stato avviato il servizio `kube-proxy` da:

-   `/etc/systemd/system/kube-proxy.service`

o attraverso il comando:

-   `ps aux | grep kube-proxy`

[\[TOP\]](#kubernetes-notes)

## Kubernetes PODs

Il POD è l'oggetto più piccolo presente in Kubernetes e contiene il
container che permette l'esecuzione della nostra applicazione. Il POD
deve essere deployato su di un Worker Node per poter attivare
l'applicazione desiderata. Di solito un POD contiene un solo container
da deployare, ma è possibile che ne contenga anche più di uno. Ad
esempio: Se un container ha la necessità di un altro container per
funzionare adeguatamente, entrambi possono restare sullo stesso POD. In
questo modo vengono deployati entrambi i container alla replica e
vengono distrutti entrambi se serve. I container nello stesso POD
comunicano tra loro attraverso `localhost` e condividono lo stesso
spazio disco.

Quando le richieste per l'applicazione deployata con un POD diventano
eccessive, si deve creare un nuovo POD e deployare una nuova istanza
dell'applicazione dividendo il carico. Se le istanze sono troppe per un
Worker Node, si crea un altro Worker Node in cui caricare il nuovo POD e
deployare l'istanza dell'applicazione.

-   `kubectl run nginx --image nginx`:

    Creo un POD e lancio un'istanza di `nginx` su di un Worker Node
    capace di ospitarlo prelevando l'immagine di `nginx` direttamente
    dal Docker Hub, il default docker repository per Kubernetes. (Posso
    configuare la sorgente delle immagini tra le impostazioni di
    Kubernetes)

-   `kubectl get pods`:

    Guardo i POD presenti sul mio Kubernetes Cluster.

-   `kubectl describe pod <pod-metadata-name>`:

    Restituisce informazioni utili sul POD.

### Creare POD in YAML

**NOTE**: YAML is Case-Sensitive.

1.  Creare un File YAML che definisce il POD (ad esempio:
    `my-pod-1.yml`) con almeno:
    1.  `apiVersion`: versione delle API di Kubernetes

    2.  `kind`: tipo di oggetto da creare

    3.  `metadata`: dizionario che contiene, in modo annidato, le
        informazioni proprie del POD (name, label, ...).

        Il numero di spazi usati per indentare/annidare i valori nel
        dizionario deve essere sempre uguale. Aggiungendo
        `type: front-end` a dizionario `label` sarà possibile
        distinguere i POD specifici per il frontend da altri.

    4.  `spec`: cosa metto nell'oggetto che sto per creare.

        Nel caso dei POD, `spec` è un dizionario di liste che indica i
        container da deployare sul Worker Node.

        ``` yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: my-pod-1
          labels:
            app: my-app-1
            type: front-end
        spec:
          containers:
            - name: nginx-container
              image: nginx
        ```

        dentro a `image`, se non si usa Docker Hub, deve essere inserito
        tutto il path dell'immagine.

        Un modo rapido per creare un file YAML per un POD è il seguente:

        -   `kubectl run nginx --image=nginx --dry-run=client -o yaml > my-pod-1.yml`

            `--dry-run=client`: impedisce la creazione di qualsiasi
            oggetto Kubernetes e indica solo se è possibile crearlo o se
            il comando è errato. `-o yaml`: genera la definizione YAML
            dell'oggetto in output.
2.  Eseguire il comando:
    -   `kubectl create -f my-pod-1.yml` oppure
        `kubectl apply -f my-pod-1.yml`

[\[TOP\]](#kubernetes-notes)

## Kubernetes Deployment

Quando, in un ambiente di produzione, andiamo ad aggiornare una
componente/applicazione dopo l'altra invece di aggiornarle tutte insieme
nello stesso momento, stiamo eseguendo un "rolling update". Se
l'aggiornamento di una componente/applicazione fallisce per un errore,
in un ambiente di produzione si dovrebbe poter "tornare indietro" e
ristabilire la piena funzionalità dell'applicazione.

Questo e molto altro è svolto dal **Kubernetes Deployment**.

Come guardare il manuale del Kubernetes Deployment?

-   `kubectl create deployment --help`

Come si crea il Kubernetes Deployment?

1.  Definisci il Kuberneted Deployment con un file YAML
    `my-kd-1-def.yml`

    ``` yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-kd-1
      labels:
        app: my-kd-app-1
        type: front-end
    spec:
      template:
        metadata:
          name: my-pod-1
          labels:
            app: my-app-1
            type: front-end
        spec:
          containers:
            - name: nginx-container
              image: nginx
      replicas: 3
      selector:
        matchLabels:
          type: front-end
    ```

2.  Esegui `kubectl create -f my-kd-1-def.yml`

3.  Controlla che il Kubernetes Deployment sia stato creato con
    `kubectl get deployments` o `kubectl get deploy`.

4.  Controlla che il Kubernetes Deplyment abbia creato il Replicat Set
    contenuto nella sua definizione: `kubectl get replicasets`.

5.  Controlla che il Replica Set abbia creato i POD contenuti nella
    definizione del Kubernetes Deployment: `kubectl get pods`.

Per controllare tutto insieme: `kubectl get all`

Un modo rapido per creare un file YAML per un Kubernetes Deployment con
replica 4 è il seguente:

-   `kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml`

[\[TOP\]](#kubernetes-notes)

## Kubernetes Services

I Kubernetes Services sono oggetti che connettono tra loro i componenti
interni ed esterni delle applicazioni deployate attraverso i POD.

Se per esempio un'applicazione web è formata da una parte Front-End, una
parte Back-End e un Database esterno, allora i Kubernetes Services
consentiranno:

1.  alla parte Front-End di essere raggiunta dagli utenti esterni che la
    devono utilizzare,
2.  alla parte Back-End di essere raggiunta dalla parte Front-End,
3.  al Database di essere raggiunto dalla parte Back-End.

Ogni Worker Node ha il proprio indirizzo IP (192.168.1.5), mentre ogni
POD ha il suo (10.244.0.3), ma stanno su due reti differenti. Non
riusciranno mai a parlare tra loro essendo su reti differenti. Il
risultato desiderato è quello di poter raggiungere l'applicazione di un
POD utilizzando l'IP del Worker Node, ma per farlo Serve qualcosa che
mappi le richieste e le instradi nel modo corretto al POD e viceversa.

Il Kubernetes Service è un oggetto come i ReplicaSet, i Deployment, ...
che ascolta il traffico di una porta del Worker Node e lo instrada alla
porta del POD che esegue l'applicazione.

Ecco alcuni dei Kubernetes Services disponibili:

-   NodePort Service: ascolta il traffico di una porta del Worker Node e
    lo instrada alla porta del POD che esegue l'applicazione.
-   ClusterIP: consente di creare una singola interfaccia di accesso a
    gruppi di POD all'interno del Kubernetes Cluster.
-   LoadBalancer: distribuisce il carico delle richieste sui
    POD/Container dello stesso `type`.

[\[TOP\]](#kubernetes-notes)

### NodePort Kubernetes service

Questo Kubernetes Service consente alle applicazioni deployate dai POD
nei container di essere raggiunte dall'esterno su di una specifica
porta.

Le porte utilizzabili del Worker Node vanno da 30000 a 32767 (valid
range).

1.  Definisci il Kuberneted Service con un file YAML `my-ks-1-def.yml`

    ``` yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-ks-1
    spec:
      type: NodePort
      ports:
        - targetPort: 80
          port: 80
          nodePort: 30008
      selector:
        name: my-pod-1
        labels:
          app: my-app-1
          type: front-end
    ```

    `spec['type']` può assumere il valore di `NodePort`, `ClusterIP` o
    `LoadBalancer`.

    `spec['ports']` è una lista contenente la mappatura delle porte.

    `targetPort` è la porta su cui risponde l'applicazione istanziata
    dal POD. Se non valorizzata, assume il valore di `port`.

    `port` è la porta del Kubernetes Service. (OBBLIGATORIO).

    `nodePort` è la porta del Worker Node. Se non valorizzata, assume un
    valore casuale valido.

    `selector` è il modo attraverso cui il NodePort service comprende
    per quale POD agire. Più sono le `labels` da controllare, più
    saranno specifici i POD da connettere, anche su Worker Node
    differenti.

    Se, ad esempio, lasciassi solo la label `app: my-app-1`, il
    Kubernetes NodePort service agirebbe per tutti i POD con quella
    label e non solo per quelli del front-end. Al bilanciamento del
    carico (Load Balancing) delle richieste ai POD coinvolti ci pensa
    già il Kubernetes Service.

2.  Esegui `kubectl create -f my-ks-1-def.yml`

3.  Controlla che il Kubernetes Service sia stato creato con
    `kubectl get services` o `kubectl get svc`.

4.  Da questo momento in poi è possibile raggiungere l'applicazione del
    POD sulla porta 30008 dalla rete locale.

[\[TOP\]](#kubernetes-notes)

### ClusterIP Kubernetes service

Mediamente in un'applicazione web entrano in gioco: Front-End, Back-End
e Database. Queste 3 componenti possono essere realizzate con diversi
POD che devono poter comunicare tra loro. Non possono farlo in modo
sicuro attraverso il proprio indirizzo IP perchè, non essendo statico,
può cambiare se i POD vengono distrutti, quindi devono usare il
ClusterIP Kubernetes service per avere un'interfaccia di accesso e
comunicazione.

In poche parole: I diversi POD che formano il Front-End verranno messi
in comunicazione con i diversi POD del Back-End attraverso un ClusterIP
Kubernetes service, così come i diversi POD che formano il Back-End
verranno messi in comunicazione con i diversi POD del Database
attraverso un altro ClusterIP Kubernetes service. I ClusterIP Kubernetes
service creano l'interfaccia di accesso per tutti i POD del medesimo
livello. Le richieste per ciascun livello vengono instradate in modo
casuale.

1.  Definisci il Kuberneted Service con un file YAML
    `my-cluip-1-def.yml`

    ``` yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-ks-1
    spec:
      type: ClusterIP
      ports:
        - targetPort: 80
          port: 80
      selector: 
          app: my-app-1
          type: front-end
    ```

    `targetPort` è la porta su cui risponde l'applicazione istanziata
    dal POD. Se non valorizzata, assume il valore di `port`.

    `port` è la porta del Kubernetes Service. (OBBLIGATORIO).

2.  Esegui `kubectl create -f my-cluip-1-def.yml`

3.  Controlla che il Kubernetes Service sia stato creato con
    `kubectl get services` o `kubectl get svc`.

[\[TOP\]](#kubernetes-notes)

### LoadBalancer Kubernetes service

Questo Kubernetes Service consente di distribuire il carico delle
richieste verso un'applicazione spalmata su più POD/Worker node fornendo
un unico punto di accesso all'applicazione. Questo lavoro di
intercettazione delle richieste e indirizzamento delle stesse per conto
dei POD/Worker node relativi all'applicazione si chiama "Load
Balancing".

Per creare un LoadBalancer Kubernetes service serve:

1.  Definire il Kuberneted Service con un file YAML
    `my-loadbalancer-1-def.yml`

    ``` yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-lb-1
    spec:
      type: loadBalancer
      ports:
        - targetPort: 80
          port: 80
          nodePort: 30008
    ```

    `targetPort` è la porta su cui risponde l'applicazione istanziata
    dal POD. Se non valorizzata, assume il valore di `port`.

    `port` è la porta del Kubernetes Service. (OBBLIGATORIO).

    `nodePort` è la porta del Worker Node. Se non valorizzata, assume un
    valore casuale valido.

2.  Esegui `kubectl create -f my-loadbalancer-1-def.yml` **su una Cloud
    che supporta questo Kubernetes service: Google Cloud Platform, AWS,
    Azure**. Farlo su un ambiente che non lo supporta, risulterebbe
    uguale a istanziare un NodePort Kubernetes service.

3.  Controlla che il Kubernetes Service sia stato creato con
    `kubectl get services` o `kubectl get svc`.

[\[TOP\]](#kubernetes-notes)

## Namespaces

I Namespace in un Kubernetes Cluster servono per distribuire le risorse
presenti nel Kubernetes Cluster e, attraverso le Policy, decidere a chi
destinarle.

Kubernetes parte con i seguenti namespace:

1.  `default`: usato per i POD, Deplyments, Serivices creati dall'utente
    che utilizza il Kubernetes Cluster.
2.  `kube-system`: usato per i POD, Deployments, Service interni al
    Kubernetes Cluster. In questo modo non è possibile che l'utente
    possa intaccare il sistema di Kubernetes.
3.  `kube-public`: usato per i POD, Deployments, Service che tutti gli
    utenti creati.

Quando si utilizza il namespace `default` non è necessario indicare il
nome del namespace per richiamare il POD/Deployment/Service, ma se si
vuole usare un POD/Deployment/Service di un altro namespace lo si deve
specificare nel nome dell'oggetto che si vuole.

`db-service.dev.svc.cluster.local`

-   `db-service`: indica l'oggetto Kubernetes che si vuole.
-   `dev`: indica il namespace di riferimento.
-   `svc`: indica che si tratta di un Kubernetes Service.
-   `cluster.local` è il dominio di default di un Kubernetes Cluster.

Per trovare i POD di un altro namespace è necessario specificarlo con
`--namespace=<NAMESPACE-NAME>` o `-n=<NAMESPACE-NAME>`:

-   `kubectl get pods --namespace=kube-system` o
    `kubectl get pods -n=kube-system`

Per creare i POD in un altro namespace è necessario specificarlo con
`--namespace=<NAMESPACE-NAME>`:

-   `kubectl create -f my-pod-1.yml --namespace=dev` o
    `kubectl create -f my-pod-1.yml -n=dev`

oppure inserire il campo `namespace: dev` nel `my-pod-1.yml` in
`metadata` ed eseguire:

-   `kubectl create -f my-pod-1.yml`

in questo modo il POD verrà sempre creato nel namespace `dev`.

Come si crea un nuovo Namespace?

1.  Definire il Kuberneted Namespace con un file YAML
    `my-dev-namespace-def.yml`

    ``` yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: dev
    ```

2.  Esegui `kubectl create -f my-dev-namespace-def.yml` oppure
    `kubectl create namespace dev`.

3.  Controlla che il Namespace sia stato creato con
    `kubectl get namespaces` o `kubectl get ns`.

Come posso passare da un namespace all'altro senza più specificarlo nel
comando o nello YAML file?

`kubectl config set-context $(kubectl config current-context) --namespace=dev`

Come vedo i POD di tutti i namespace?

`kubectl get pods --all-namespaces` o `kubectl get pods -A`

Come limito le risorse disponibili per un Namespace?

1.  Definire il ResourceQuota con un file YAML
    `my-resource-quota-def.yml`

    ``` yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: dev-quota
      namespace: dev
    spec:
      hard:
        pods: "10"
        requests.cpu: "4"
        requests.memory: "5Gi"
        limits.cpu: "10"
        limits.memory: "10Gi"
    ```

2.  Esegui `kubectl create -f my-resource-quota-def.yml`.

[\[TOP\]](#kubernetes-notes)

## Imperativo VS Dichiarativo

In Kubernetes sto usando l'imperativo quando istruisco dettagliatamente
cosa deve fare Kubernetes sull'infrastruttura:

-   Creare un Deployment:
    `kubectl create deployment --image=nginx nginx`
-   Creare un oggetto attraverso un file YAML:
    `kubectl create -f object-def.yml`
-   Replicare un oggetto: `kubectl scale deployment nginx --replica=5`

Sono comandi one-shot che non vengono tracciati se non fosse per la
history del proprio terminale. Rende difficile la vita a chi vuole
capire come sono stati creati, modificati o eliminati degli oggetti sul
Kubernetes Cluster. Non è indicato per la produzione.

In Kubernetes uso il dichiarativo quando, con un solo file di
configurazione, lascio decidere a Kubernetes quello che è meglio fare
per ottenere quello che mi serve. Il dichiarativo usa
`kubernetes apply -f config-def.yml` o
`kubernetes apply -f /path/of/config/dir/files/` (per applicare quanto
richiesto da tutti i file di configurazione interni alla cartella).

Quando si vuole fare una modifica ad un oggetto Kubernetes istanziato
con uno YAML file si deve:

1.  Modificare il file YAML
2.  Lanciare `kubectl replace -f my-conf-file.yml`

E se voglio cancellare tutto e ricrearlo?

-   `kubectl replace --force -f my-config-file.yml`

[\[TOP\]](#kubernetes-notes)

## Kubectl Apply

Quando viene usato il comando `kubectl apply -f my-def-file.yml`,
Kubernetes mette a confronto:

-   Il file locale contenente la definizione dell'oggetto:
    `my-def-file.yml`
-   La configurazione attiva sul Kubernetes Cluster (Live Object)
-   L'ultima configurazione applicata dell'oggetto (Last applied
    configuration)

Ad ogni aggiornamento/modifica al file locale, l'esecuzione di
`kubectl apply` provoca l'aggiornamento della Live Object e dell'ultima
configurazione applicata.

Ad ogni rimozione sul file locale, l'esecuzione di `kubectl apply`
provoca la rimozione sulla Live Object, ma non sulla
`Last applied configuration`. In questo modo è possibile capire cosa è
stato rimosso se dovesse servire.

[\[TOP\]](#kubernetes-notes)

## Scheduling

Kubernetes decide dove schedulare un POD sulla base del parametro "nodeName" posto all'interno dello YAML file del POD.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-1
  labels:
    app: my-app-1
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
  nodeName: node-02
```

Lo scheduler predefinito di Kubernetes, scansiona tutti i POD alla ricerca di quelli in cui non è presente il parametro `nodeName`, decide quali di quelli trovati è adatto per essere spostato
e stabilisce su quale Worker Node metterlo settando il valore mancante di `nodeName`.

### Manual Scheduling

Nel caso in cui lo scheduler predefinito di Kubernetes non sia presente, l'avvio di un POD lo porterà in uno stato **pending** in quanto non può essere selezionato alcun Worker Node in cui eseguirlo.
Si deve quindi intervenire con il "Manual Scheduling" andando ad aggiungere il parametro `nodeName` allo YAML del POD prima della sua creazione.

E per i POD pending? Si deve creare un oggetto "Binding":

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: my-binding-obj
target:
  apiVersion: v1
  kind: Node
  name: node-02
```

ed inviarlo alle API di binding del POD via POST request convertendo lo YAML in JSON.

Ma molto meglio modificare lo YAML e crearlo bene.

[\[TOP\]](#kubernetes-notes)

## Labels & Selectors

Le `labels` sono utilizzate per creare gruppi che contengono entità dalle medesime caratteristiche.
Più `label` vengono aggiunte all'entità, più finemente potrà essere eseguita la ricerca di gruppi di risorse specifiche.
Le `labels` possono essere applicate a diversi tipi di oggetti: PODs, ReplicaSet, Services, Deployments...
Una label ha il formato <chiave>:<valore>:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-1
  labels:
    app: my-app-1
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
  nodeName: node-02
```

I `selectors` selezionano il gruppo che possiede tutte le labels a lui date e per vedere quali POD fanno parte di un gruppo:

`kubectl get pods --selector <chiave>=<valore>`

Se volessimo creare un ReplicaSet composto di 3 PODs aventi la label `type: front-end` nel loro YAML, dovremmo inserire sotto `spec/template/metadata/labels` la label `type: front-end` del POD che poi viene selezionata dal `matchLabels` in `selector`. Possiamo inserire tutte le label che ci servono per selezionare correttamente quanto ci serve. Quanto verrà selezionato dal `selectors` sarà l'entità che contiene tutte le `labels` indicate.
Le `labels` all'interno di `metadata/labels` sono quelle che caratterizzano l'oggetto ReplicaSet e non sono collegate al POD.

``` yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-rs-1
  labels:
    app: my-rs-app-1
    type: front-end
  annotations:
    buildVersion: 1.34
spec:
  template:
    metadata:
      name: my-pod-1
      labels:
        app: my-app-1
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

Le `annotations` permettono, invece, di aggiungere ulteriori dettagli utili, ma esse non vengono utilizzate altrove.


[\[TOP\]](#kubernetes-notes)

## Author

-   [Marco Malavolti](mailto:marco.malavolti@gmail.com)

[\[TOP\]](#kubernetes-notes)
