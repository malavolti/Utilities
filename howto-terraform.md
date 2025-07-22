# Terraform Notes

## IAC - Infrastructure as Code

Gestire i componenti di un'infrastruttura via codice.

Terraform è più adatto per creare le VM, Ansible per configurarle.

Docker: infrastruttura immutevole perchè dobbiamo distribuire una nuova immagine per ogni modifica.

HCL è il linguaggio di Terraform ed è dichiarativo.

I file di terraform finiscono con ".tf".

Terraform ha 3 fasi: Init, Plan e Apply

1. Init: inizializzazione del progetto
2. Plan: decide un piano per raggiungere lo stato desiderato
3. Apply: esegue le operazioni per raggiungere lo stato desiderato

Se cambia qualcosa, alla successiva esecuzione torna tutto come stabilito da Terraform.

esempio.tf:

```bash
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content = "We love pets!"
}
```

`resource` = Block name
`local` = tipo di provider da usare
`file` = risorsa da creare
`pet` = nome della risorsa creata

Quattro fasi per l'esecuzione di Terraform:

1. scrivere il file.tf
2. eseguire il comando `terraform init`
3. rivedere il piano di esecuzione con `terraform plan`
4. applicare il piano con `terraform apply`

Eseguendo il comando `terraform show` nella cartella del `file.tf` è possibile vedere i dettagli della risorsa creata.

I tipi di risorse che si possono creare si trovano in:
https://registry.terraform.io/prividers/hashicorp/local/latest/docs

Con il comando `terraform destroy` si distrugge tutto.

`main.tf`: contiene tutte le definizioni delle risorse
`variables.tf`: contiene tutte le variabili
`outputs.tf`: contiene gli output delle risorse
`provider.tf`: contiene le definizioni dei provider

## Variabili

Le variabili servono per poter riutilizzare gli stessi valori in N situazioni diverse.

```bash
variable "filename" {
  description = "facoltativa, ma utile per capire a cosa server la variabile"
  default = "root/pets.txt"
  type = string | number | bool | any | list | map | object | tuple | set  #facoltativo 
}
```

Il nome della variabile usata dai file.tf è buona prassi che coincida con il parametro
che vuole valorizzare nella risorsa. Usare "default" è una buona prassi per assegnare un
valore predefinito alla variabile.

```bash
resource "local_file" "pet" {
  filename = var.filename
  content = var.content
}
```

### Variabile "list":

Esempio di lista con indicati gli indici a partire da 0:

```bash
variable "prefix" {
              0    1    2
  default = ["A", "B", "C"]
  type = list
}
```

0, 1, 2 sono usati solo per indicare gli indici dell'array, non devono essere presenti nel codice.
Utilizzo variabile di tipo "list": var.prefix[0]  sta per "A"

### Variabile "map" (dizionario):

```bash
variable file-content {
  default = {
    "key1" = "value1"
    "key2" = "value2"
    "key3" = "value3"
  }
  type = map
}
```

Utilizzo Var tipo "map": var.file-content["key1"]  sta per "value1"

Se voglio definire una varibile come lista di stringhe:

```bash
variable "prefix" {
  default = ["A", "B", "C"]
  type = list(string)
}
```

Se voglio definire una varibile come lista di numeri:

```bash
variable "prefix" {
  default = [1, 2, 3]
  type = list(number)
}
```

## Tipo "set"

il tipo "set" non ammette valori duplicati (come in Python)

## Tipo "object"

Tipo complesso che permette di combinare insieme tutti i tipi semplici.

```bash
variable "prefix" {
  type = object ({
    name = string
    color = string
    food = list(string)
    favorite_pet = bool
  })
  
  default = {
    name = "Marco"
    color = "red"
    foot = ["pizza", "pasta", "kebab"]
    favorite_pet = true
  }
}
```

## Tipo "tuple"

```bash
variable gatto {
  type = tuple ([string, number, bool])
  default = ["cat", 4, true]
}
```

Se non definisco un "default" value per le variabili,
Terraform me le chiederà a terminale.

Le variabili possono essere definite nel file "terraform.tfvars" o "qualsiasi-nome.auto.tfvars"
per essere automaticamente ricevute da Terraform.

La precedenza con cui le variabili vengono valorizzate è:

1. ENV vars
2. terraform.tfvars
3. *.auto.tfvars
4. command line (-var o --var-file)

## Resource Attributes - Come condividere valori di altre risorse

Nella documentazione sono identificati con la sezione "Read-only".

`${ <tipo_risorsa>.<nome-risorsa>.<returned_value> }`: esempio: `${random_pet.mypet.id}`

```bash
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content = "We love ${random_pet.my-pet.id}!"
}
```

## Dipendenza esplicita delle risorse

Se voglio creare una risorsa dopo l'esecuzione di un'altra,
devo inserire:

```bash
depends_on = [
  tipo_risorsa.nome_risorsa
]
```

nella definizione della risorsa.

## Output Variables (utile per Debug?)

Se voglio salvare un "Read-Only"/Resource Attribute in una variabile,
uso le Output Variables.

```bash
output "pet-name" {
  value = random_pet.my-pet.id
  description = "Salvo l'id del pet in pet-name"
}
```

Le Output Varibles sono mostrate a schermo durante l'apply di Terraform,
o con il comando `terraform output`. Se voglio vedere una sola variabile uso:

`terraform output <nome-variabile-di-output>`

Terraform usa `file.tfstate` per tenere traccia delle modifiche eseguite
e decidere cosa fare ad ogni `terraform apply`.

Ogni risorsa in Terraform ha un ID univoco.

Per velocizzare l'esecuzione di Terraform, si può eseguire `terraform plan --refresh=false`
per considerare solo lo stato della risorsa memorizzato da Terraform.

Il `file.tfstate` diventa prezioso se condiviso con tutti i membri del Team
per rimanere allineati sullo stato delle azioni applicate da Terraform sull'infrastruttura.

Non è possibile rinunciare allo stato di terraform.

In uno scenario di Cloud, lo stato definisce le VM erogate e configurate (CPU, RAM, Disco, ...)

I file.tf è bene che siano condivisi via GIT,
ma per quanto riguarda il file di Stato (visto le informazioni che contiene)
è meglio memorizzarli in Remote State Backends.

MAI MODIFICARE I FILE DI STATO DI TERRAFORM MANUALMENTE

## terraform validate

Informa se il file di configurazione è scritto bene con suggerimenti per risolvere gli errori

## terraform fmt

formatta il codice del file di configurazione in un modo standard

## terraform show

mostra le risorse create da Terraform, anche quelle non visibili

## terraform providers

mostra tutti i providers disponibili e scaricati

## terraform output

stampa a video

## Configuration drift

Quando diversi sistemi che dovrebbero essere uguali per software installato e configurazioni,
divergono per problemi di allineamento risultando così avere stati differenti.
La risoluzione dei problemi con un configuration drift diventa più complicato.

## Immutable Infrastructure

E' un'infrastruttura in cui non si aggiorna il software "in-place", 
ma si crea una nuova istanza con il software aggiornato.
Se avviene un imprevisto che rende il sistema instabile o non funzionante,
quel sistema viene sostituito con uno nuovo della versione stabilita.
Con un Immutable Infrastructure è possibile mantenere ben chiaro la versione che si sta usando
ed eseguire un roll-back delle configurazione.

## Lifecycle rules

quando voglio controllare come Terraform applica i cambiamenti al sistema,
uso le lifecycle rules.

In questo esempio indico a Terraform di creare la risorse prima di distruggere
quella precedente:

```bash
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content = "We love pets!"
  file_permission = "0700"

  lifecycle {
    create_before_destroy = true
  }
}
```

ATTENZIONE! Se creo un file nuovo prima e poi lo distruggo... NON AVRO' PIU' QUEL FILE!

In questo altro esempio si indica a Terraform di non distruggere la risorsa
e se dovesse avvenire, Terraform avviserà l'utente via terminale.

```bash
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content = "We love pets!"
  file_permission = "0700"

  lifecycle {
    prevent_destroy = true
  }
}
```

In questo esempio invece si indica a Terraform di ignorare le modifiche
al contenuto del file:

```bash
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content = "We love pets!"
  file_permission = "0700"

  lifecycle {
    ignore_changes = [
      content
    ]
  }
}
```

Se viene utilizzato "ignore_changes = all", 
Terraform ignorerà ogni modifica al file indicata attraverso gli argomenti della resource.

## Data Sources (o Data Resources)

utili quando voglio usare file o sorgenti di input esterni al controllo di Terraform,
ad esempio: Un file creato da uno script che viene eseguito a CRON.

https://registry.terraform.io/providers/hashicorp/local/latest/docs/resources/file

```bash
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content = "We love pets!"
  file_permission = "0700"
}

data "local_file" "dog" {
  filename = "/root/dogs.txt"
}
```
per chiamare il contenuto della risorsa "dog" si usa:

`data.local_file.dog.content`

```bash
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content = data.local_file.dog.content
  file_permission = "0700"
}
```

## Meta Arguments

Vengono inseriti all'interno del blocco della risorsa e servono a determinarne il comportamento:
- creare una risorsa in seguito alla creazione di un'altra (depends_on)
- controllare il ciclo di vita di una risorsa (lifecycle)
- creare molte volte una risorsa

### count (lavora con le liste)

con `count` è possibile generare più risorse con un solo blocco risorsa.

- vim main.tf

  ```bash
  resource "local_file" "pet" {
    filename = var.filename[count.index]
    count = length(var.filename)
  }
  ```

- vim variables.tf

  ```bash
  variabile "filename" {
    default = [
      "tmp/file1.txt",
      "tmp/file2.txt",
      "tmp/file3.txt"
    ]
  }
  ```

  Dato che lavora con le liste, se rimuovo un qualsiasi elemento della lista,
  Terraform ricrea tutti gli elementi per mantenere invariati i giusti indici: 0,1,2,...

### for_each (lavora con i dizionari)

- `vim main.tf`

  ```bash
  resource "local_file" "pet" {
    filename = each.value
    for_each = toset(var.filename)
  }
  ```

  `toset` trasforma una lista in un set.

- `vim variables.tf`

  ```bash
  variabile "filename" {
    default = [
      "tmp/file1.txt",
      "tmp/file2.txt",
      "tmp/file3.txt"
    ]
  }
  ```

  Dato che lavora con i dizionari, non ci sono indici numerici da mantenere in ordine
  e la rimozione di un qualsiasi elemento non determina la ricombinazione di quelli già esistenti.


## Version Constraints

Come selezionare una specifica versione di un provider di Terraform che, 
se non specificato, Terraform "init" scarica sempre l'ultima versione disponibile.

1. Scegliere quanto si vuole da: https://registry.terraform.io/
2. Scegliere la versione desiderata
3. Aggiungere il blocco "terraform" fornito premendo il bottone "USE PROVIDER" al proprio main.tf.

E' possibile usare i simboli: !=, <, >, <=, >=
per determinare quali versioni devono essere scaricate.

Con l'operatore '~>', Terraform scaricherà tutte le versioni minori più aggiornate. Esempio:

~> 1.2 (scarica: 1.2, 1.3, 1.4 ... 1.9)
~> 1.2.3 (scarica: 1.2.3, 1.2.4, 1.2.5 ... 1.2.9)


## Remote State

Mai memorizzare i file di stato ".tfstate" in un sistema di controllo delle versioni (GIT),
perchè contiene IP, password e dati che non dovrebbero mai essere condivisi esternamente
al proprio sistema di sviluppo.
Se poi 2 diverse persone usano lo stesso file di stato, le conseguenze sono imprevedibili
e lo stato ottenuto potrebbe non essere quello desiderato.
Terraform usa lo "state locking" per impedire l'esecuzione di altre azioni sulle stesse risorse,
mentre queste sono già occupate in altro, ma può farlo solo localmente alla macchina in cui è installato
o su un Remote State Backend tipo (AWS S3, Google Cloud Storage o Terraform State storage).

## Terraform State Commands

terraform state <subcommand> [options] [args]

- `terraform state list`: elenca tutte le risorse gestite da Terraform
- `terraform state show <nome-risorsa>`: mostra tutte le caratteristiche della risorsa richiesta
- `terraform state mv [options] SOURCE DESTINATION`: essensialmente usato per rinominare risorse
- `terraform state rm ADDRESS: server per rimuovere una risorsa dallo stato perchè non la si vuole più gestire con Terraform.

## Terraform Provisioner

permettono di eseguire comandi sulla risorsa configurata attraverso Terraform, ma non è consigliabile farlo.
Utilizzare invece quanto già previsto dalla risorsa o costruire la risorsa in modo personalizzato
e contenente tutto quello di cui si ha bisogno.

Ad esempio, per eseguire uno script in un webserver remoto si usa "remote-exec":

```bash
resource "aws_instance" "webserver" {
  ami = "ami-12345678"
  instance_type = "t2.micro"
  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install nginx -y",
      "sudo systemctl enable nginx",
      "sudo systemctl start nginx",
    ]
  }
  connection {
    type = "ssh"
    host = self.public_ip
    user = "ubuntu"
    private_key = file ("~/.ssh/id_rsa")
  }

  key_name = aws_key_pair.web.id
  vpc_security_group_ids = [aws_security_group.ssh-access.id]
}
```

`self.public_ip` verrà tradotta con l'IP Pubblico della macchina remota che si sta configurando.

Per eseguire un comando localmente a dove viene eseguito Terraform, si usa `local-exe`:

```bash
resource "aws_instance" "webserver" {
  ami = "ami-12345678"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo ${aws_instance.webserver.public_ip} >> /tmp/ips.txt"
  }
}
```

che avrà come risultato quello di creare un file in `/tmp/ips.txt` con il risultato del compando.
  
In modo predefinito, i provisioner sono eseguiti dopo l'avvenuta creazione delle risorse,
ma aggiungendo `when = destroy` prima del `command`, possiamo eseguire il comando prima della creazione della risorsa, 
ovvero dopo la sua distruzione.

```bash
resource "aws_instance" "webserver" {
  ami = "ami-12345678"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo ${aws_instance.webserver.public_ip} Created! >> /tmp/instance_state.txt"
  }

  provisioner "local-exec" {
    when = destroy
    command = "echo ${aws_instance.webserver.public_ip} Destroyed! >> /tmp/instance_state.txt"
  }
}
```

Per continuare l'esecuzione di Terraform anche in caso di errori si usa `on_failure = continue`:

```bash
resource "aws_instance" "webserver" {
  ami = "ami-12345678"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    on_failure = continue
    command = "echo ${aws_instance.webserver.public_ip} Created! >> /temp/instance_state.txt"
  }

  provisioner "local-exec" {
    when = destroy
    command = "echo ${aws_instance.webserver.public_ip} Destroyed! >> /tmp/instance_state.txt"
  }
}
```

La cartella `/temp` non esiste, ma questo non ferma l'esecuzione.

Da Best Practice, l'uso dei provisioner è l'ultima via da usare.

## Debugging

per memorizzare i log in un file eseguire:

`export TF_LOG_PATH=/tmp/terraform.log`

per indicare la profondità dei log eseguire:

`export TF_LOG=TRACE`

(INFO | WARNING | ERROR | DEBUG | TRACE sono accettati)

per disattivare:

- `unset TF_LOG`
- `unset TF_LOG_PATH`

## Terraform import

il comando `terraform import` è possibile importare nuove risorse in Terraform, ma create da altri.

https://developer.hashicorp.com/terraform/cli/import

## Terraform Workspace

utile per riutilizzare lo stesso codice per più progetti, ognuno con il proprio state file.

https://developer.hashicorp.com/terraform/cli/workspaces
