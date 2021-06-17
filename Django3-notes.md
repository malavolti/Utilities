# Django3 Udemy Course

1. [Django installation](#django-installation)
2. [Django running](#django-running)
3. [Creare una nuova App per il progetto](#creare-una-nuova-app-per-il-progetto)
4. [Creare la Homepage del sito](#creare-la-homepage-del-sito)
5. [Creare e usare i Templates](#creare-e-usare-i-templates)
6. [Creare e usare i Form](#creare-e-usare-i-form)
7. [Django3 Admin interface](#django3-admin-interface---creare-il-superutente-admin)
8. [Configurare la cartella 'media' e 'static' del sito](#configurare-la-cartella-media-e-static-del-sito)
9. [Models e Database](#models-e-database)
10. [Includere URL legate alle App (blog)](#includere-url-legate-alle-app-blog)
11. [Utilities di objects()](#utilities-di-objects)
12. [Contenuto Statico](#contenuto-statico)
13. [Ad ogni articolo il giusto path](#ad-ogni-articolo-il-giusto-path)
14. [Come ottenere il numero di oggetti creati su una pagina HTML](#come-ottenere-il-numero-di-oggetti-creati-su-una-pagina-html)
15. [Come gestire le pluralità nel testo (inglese di default)](#come-gestire-le-pluralit%C3%A0-nel-testo-inglese-di-default)
16. [Funzioni Utili per la manipolazione dei valori visualizzati](#funzioni-utili-per-la-manipolazione-dei-valori-visualizzati)
17. [Estendere un template base](#estendere-un-template-base)
18. [Web Interface Utilities](#web-interface-utilities)
19. [Source Code Utilities](#source-code-utilities)
20. [Usare settings diversi in produzione e in development](#usare-settings-diversi-in-produzione-e-in-development)
21. [Autenticazione degli utenti](#autenticazione-degli-utenti)
22. [Models e relazioni con gli utenti](#models-e-relazioni-con-gli-utenti)
23. [Creare una custom Form](#creare-una-custom-form)

## Django installation

* `sudo pip3 install django-admin`

## Django running

Creare il proprio progetto:

* Mi sposto dove voglio ospitare il progetto:

  `cd $HOME`
  
* Creo il progetto con la sua prima App `password_generator`:

  * `django-admin startproject password_generator`

* Rinomino la cartella genitore per distinguerla dalla cartella dell'App:

  * `mv password_generator password_generator-project`

* Mi sposto nella cartella di progetto:

  * `cd password_generator-project/`

* Sistemo le migrazioni con:

  * `python3 manage.py migrate`

* Eseguo il server di sviluppo:

  * `python3 manage.py runserver`

Aprire la URL che viene data alla fine per vedere Django in azione.

Utilità:

* `python3 manage.py help` (visualizza tutto quello che può fare)
* `settings.py` e `urls.py` (sono file molto importanti per il sito Django)

## Creare una nuova App per il progetto

La Apps esse sono i moduli in cui si può dividere il progetto.

* `cd $HOME/password_generator-project/`
* `python3 manage.py startapp generator` (nome dell’app = generator)
* `vim password_generator/settings.py` e aggiungere "**generator**" alle "**INSTALLED_APPS**":

  ```python
  # Application definition

  INSTALLED_APPS = [
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
      'generator',
  ]
  ```

## Creare la Homepage del sito

* `cd $HOME/password_generator-project/`
* `vim password_generator/urls.py`

  ```bash
  from generator import views as generator_views   #questo importa il file views.py dell'app 'generator' rinominandolo in 'generator_views'
 
  urlpatterns = [
     path('admin/', admin.site.urls),
     path('', generator_views.homepage, name='homepage'),   #questo aggiunge la url per la homepage
  ]
  ```

* `vim generator/views.py`

  ```python
  from django.shortcuts import render
  from django.http import HttpResponse

  def homepage(request):
      return HttpResponse("<h1>Homepage di Marco</h1>")
  ```

## Creare e usare i Templates

* Mi sposto all'interno del nuovo progetto:
  * `cd $HOME/password_generator-project/password_generator`

* Avvio il server con:
  * `python3 manage.py runserver`

* Creo la cartella `generator` contenente i `templates` dell'App `generator`:

  * `mkdir -p generator/templates/generator`

* Creo il template `tmpl-example.html`:
  * `vim generator/templates/generator/tmpl-example.html`

    ```html
    <h1>Homepage di Marco</h1>
    <h2>{{ password }}</h2>
    <br/>
    <a href="{% url 'homepage' %}"'>Home</a>
    ```

* Modifico `views.py` dell'App per usare il template `tmpl-example.html`:
  * `vim generator/views.py`

    ```python
    from django.shortcuts import render

    def tmpl_example(request):
        renderDict = {
            'password':'dcndocndosnd',
        }
        return render(request, 'generator/tmpl-example.html', renderDict)
    ```

* Modifico le `urls.py` per abilitare il path della Homepage
  * `vim password_generator/urls.py`

    ```bash
    from generator import views as generator_views   #questo importa il file views.py dell'app 'generator' rinominandolo in 'generator_views'
 
    urlpatterns = [
       path('admin/', admin.site.urls),
       path('', generator_views.homepage, name='homepage'),   #questo aggiunge la url per la homepage e la nomina 'homepage'
    ]
    ```

* Salvando vedrò immediatamente il risultato sulla mia homepage su `http://127.0.0.1:8000/`

## Creare e usare i Form

* Mi sposto all'interno del nuovo progetto:
  * `cd $HOME/password_generator-project/password_generator`

* Avvio il server con:
  * `python3 manage.py runserver`

* Creo la cartella `generator` contenente i `templates` dell'App `generator`:

  * `mkdir -p generator/templates/generator`

* Creo il template `form.html`:
  * `vim generator/templates/generator/form.html`

    ```python
    <h1>Password Generator</h1>

    <form action="{% url 'newPassword' %}">
       <select name="length">
          <option value="6">6 char long</option>
          <option value="7">7 char long</option>
          <option value="8">8 char long</option>
          <option value="9">9 char long</option>
       </select>

       <input type="submit" value="Generate Password"/>
    </form>
    ```

* Modifico `views.py` dell'App per usare il template `form.html`:
  (in questo esempio viene catturata la variabile `length` dalla URL
  modificata dalla form e se richiamata con `{{ length }}` nel template potrà essere usata. Tutto quello che proviene dalla URL è di tipo "stringa" e per convertirlo in numero va eseguito il CAST: `int(request.GET.get('length'))`)

  * `vim generator/views.py`

    ```python
    from django.shortcuts import render

    # Render a Page that obtain values from parameters (request.GET.get('<param_name'))
    def new_password(request):
       renderDict = {
          'length' : int(request.GET.get('length')),
          'special' : 'no',
          'numbers' : 'no'
       }

       return render(request, 'generator/new-password.html', renderDict)    
    
    def form(requests):
        return render(request, 'generator/form.html')
    ```

* Modifico le `urls.py` per abilitare il path della form e del tmpl-example
  * `vim password_generator/urls.py`

    ```python
    from generator import views as generator_views   #questo importa il file views.py dell'app 'generator' rinominandolo in 'generator_views'
 
    urlpatterns = [
       ...
       path('form/', views.form, name='formPath'),
       path('new-password/', views.new_password, name='new_password'),
    ]
    ```

* Salvando vedrò immediatamente il risultato sulla mia homepage su `http://127.0.0.1:8000/form`

## Django3 Admin interface - Creare il Superutente 'admin'

* Creo il SuperUser in grado di accedere all'interfaccia "**/admin**":
  * `cd personal_portfolio-project`
  * `python3 manage.py createsuperuser`

## Configurare la cartella 'media' e 'static' del sito

La cartella `media/` è utile per memorizzare i media file: immagini, video, suoni, documenti,... 
La cartella `static/` è utile per memorizzare i file statici: CSS, JS

Ma per poter utilizzare i file memorizzati nel database legato al sito, c'è bisogno di quanto segue per dare loro una URL valida.

* Aggiungere `MEDIA_ROOT` e `STATIC_ROOT` in coda al file `settings.py` sotto la costante `STATIC_URL`:

  ```python
  STATIC_URL = '/static/'
  STATIC_ROOT = BASE_DIR / 'static'

  MEDIA_URL = '/media/'
  MEDIA_ROOT = BASE_DIR / 'media'
  ```
  
  `STATIC_URL` è il path usato dal browser per raggiungere i file statici
  `STATIC_ROOT` è il percorso della cartella in cui salvare i file statici
  
  `MEDIA_URL` è il path usato dal browser per raggiungere i media
  `MEDIA_ROOT` è il percorso della cartella in cui salvare i media
  
  (il valore potrebbe cambiare. Guardare quanto riportato per i DATABASES per la prima parte `BASE_DIR /`)
  
* Aggiungere il path agli `url_pattern` di `urls.py` quanto segue:

  ```python
  from django.contrib import admin
  from django.urls import path
  from django.conf.urls.static import static
  from django.conf import settings


  urlpatterns = [
     path('admin/', admin.site.urls),
  ]

  urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
  ```
  
* `python3 manage.py collectstatic`
* `python3 manage.py runserver`

## Models e Database

* Creare un nuovo progetto con 2 apps: `blog` e `portfolio`
  * `cd $HOME`
  * `django-admin startproject personal_portfolio`
  * `mv personal_portfolio personal_portfolio-project`
    (rinominare con "-project" per distinguere il il progetto dalla cartella omonima interna)
  * [Creare una nuova "app"](#Creare-una-nuova-App-per-il-progetto) per ogni parte del sito che si vuole gestire:
    * blog
    * portfolio

    e aggiungerle alle "**INSTALLED_APPS**" di `settings.py`
  * Lanciare il server di sviluppo:
    * `python3 manage.py runserver`

  Se il comando solleva un messaggio del tipo:
  
  ```bash
  You have 18 unapplied migration(s). Your project may not work properly  
  until you apply the migrations for app(s): 
  admin, auth, contenttypes, sessions.
  Run 'python manage.py migrate' to apply them.
  ```
  
  Eseguire `python3 manage.py migrate` per risolvere.

* Creare una nuova classe `Project` dentro `models.py` dell'app. La classe definirà un nuovo modello:
  * `vim personal_portfolio-project/portfolio/models.py`

    ```python
    from django.db import models
    
    class Project(models.Model):
        title = models.CharField(max_length=100)
        description = models.CharField(max_length=250)
        image = models.ImageField(upload_to='portfolio/images/')
        url = models.URLField(blank=True)
         
        # Questa funzione permette di rinominare ogni nuovo oggetto
        # con il suo titolo
        def __str__(self):
           return self.title
    ```

* Elaborare il nuovo modello o sue eventuali modifiche:
  * `cd personal_portfolio-project`
  * `python3 manage.py makemigrations`

    Risultato:

    ```bash
    Migrations for 'portfolio':
      portfolio/migrations/0001_initial.py
        - Create model Project
    ```

  * `python3 manage.py migrate`

    ```bash
    Operations to perform:
      Apply all migrations: admin, auth, contenttypes, portfolio, sessions
    Running migrations:
      Applying portfolio.0001_initial... OK    
    ```

    Notare la presenza della propria app `portfolio` tra quelle considerate per le migrations.

  * `python3 manage.py runserver`

* Registrare il modello in `admin.py` del progetto:
  * `vim personal_portfolio-project/admin.py`

    ```python
    from django.contrib import admin
    from .models import Project

    # Voglio vedere questo modello nell'interfaccia "admin/"
    admin.site.register(Project)
    ```
    
    Questo permette di ottenere il controllo della creazione delle classi `Project` dentro `admin/`.

* Abilitare il modello creato in `views.py` con:
  * `vim personal_portfolio-project/views.py`

    ```python
    from .models import Project
    
    def homepage(request):
       projects = Project.objects.all()
       return render(request, 'portfolio/homepage.html', {'projects':projects})
    ```
    
    Questo permette di usare le informazioni nel DB dentro i template.

* Catturare le informazioni nel template della pagina Web:
  * `vim personal_portfolio-project/portfolio/templates/portfolio/homepage.html`

    ```html
    <h1>Homepage</h1>

    <a href="{% url 'homepage' %}">Home</a>
    
    <br/><br/>

    {% for project in projects %}

    <h2>{{ project.title }}</h2>
    <p>{{ project.description }}</p>
    <img src="{{ project.image.url }}" height="100" width="100"/>

    {% if project.url %}
    <a href="{{ project.url }}">LINK</a>
    {% endif %}

    {% endfor %}
    ```

## Includere URL legate alle App (blog)

* Aggiungere il path `blog/` tra gli `urls.py` del sito principale `portfolio`:
  * `vim personal_portfolio-project/portfolio/urls.py`

    ```Python
    urlpatterns = [
      # ...
      path('blog/', include('blog.urls')
    ]
    ```

* Creare `blog/urls.py` nella cartella dell'app per la "homepage" (path = '') dell'app "blog":

  * `vim personal_portfolio-project/blog/urls.py`

    ```Python
    from django.urls import path
    from . import views

    urlpatterns = [
       path ('', views.home_blogs, name='home_blogs')
    ```

* Aggiungere la views per la Homepage di tutti i blog:

  * `vim personal_portfolio-project/blog/views.py`
  
    ```Python
    from django.shortcuts import render

    # Create your views here.

    def all_blogs(request):
        return render(request, 'blog/home_blogs.html')
    ```
    
* Creare il template per la Homepage dei Blogs:

  * `mkdir -p personal_portfolio-project/blog/templates/blog`
  * `vim personal_portfolio-project/blog/templates/blog/home_blogs.html`

    ```html
    <h1>Hi!</h1>

    <h1>Io sono la Home di tutti i Blog!</h1>
    ```

* Vedere la Homepage dell'app `blog` su `http://127.0.0.1:8000/blog/`

 
## Utilities di objects()

* Se voglio ordinare gli oggetti per la data di creazione (`-date`):
  * `vim personal_portfolio-project/portfolio/views.py`

    ```Python
    def homepage(request):
        projects = Project.objects.order_by('-date')
        return render(request, 'portfolio/homepage.html', {'projects': projects})
    ```
    
* Se voglio visualizzare solo 4 oggetti alla volta:
  * `vim personal_portfolio-project/portfolio/views.py`

    ```Python
    def homepage(request):
        projects = Project.objects.all()[:4]
       return render(request, 'portfolio/homepage.html', {'projects': projects})
    ```

* Se voglio visualizzare solo 4 oggetti alla volta ordinati per data:
  * `vim personal_portfolio-project/portfolio/views.py`

    ```Python
    def homepage(request):
        projects = Project.objects.order_by('-date')[:5]
        return render(request, 'portfolio/homepage.html', {'projects': projects})
    ```

## Contenuto Statico

Il contenuto statico viene inserito nel path `/static/` definito da Django nel `settings.py`.

Per utilizzare contenuto statico devo:

1. Creare la cartella `static/<nomeApp>` all'interno della cartella dell'App per cui voglio creare del contenuto statico.

   Esempio:
   * `mkdir -p personal_portfolio-project/blog/static/blog`

2. Caricare il mio contenuto statico (file, immagini, video, audio) nella cartella creata.

3. Modificare la pagina HTML del sito inserendo:
   * `{% load static%}` in testa al file.html per indicare che voglio caricare contenuto statico.
   * `{% static 'pathDelFileDentroStaticDir' %}` dove ho bisogno di richiamare il file statico.

     Esempio:
     * `<img src="{% static 'blog/esempio.png' %}">`
     * `<a href="{% static 'blog/esempio.pdf' %}">Download PDF</a>`
   

## Ad ogni articolo il giusto path

Se voglio raggiungere ogni articolo del mio blog usando le URL devo:

1. Modificare `urls.py` per recuperare tutti gli articolo tramite il loro `article_id` (il nome può essere qualsiasi):

   ```python
   urlpatterns = [
      path('', views.all_blogs, name='all_blogs'),
      path('<int:article_id>/', views.detail, name='detail')
   ]
   ```

2. Modificare `views.py` per rispondere alla view aggiunta richiamando `blog_id`:

   ```python
   # Importo la funzione 'get_object_or_404' per recuperare
   # l'oggetto o restiture l'errore 404 (File Not Found)
   # se non viene trovato.
   from django.shortcuts import render, get_object_or_404
   from .models import Article

   # ...altro...

   # Definisco la funzione 'detail' che risponde alla URL creata in 'urls.py'
   # in cui ricevo un 404 se l'articolo con ID=article_id non è stato trovato
   # o la pagina dell'articolo composta da un template.
   def detail(request, article_id):
       article = get_object_or_404(Article, pk=article_id)
       return render(request, 'blog/detail.html', {'article':article})
   ```

3. Creare il template `detail.html` per mostrare le informazioni sugli articoli:
   * `vim blog/template/blog/detail.html`

     ```html
     {{ article.title }}
     {{ article.data }}
     {{ article.description }}
     ```

4. Da questo momento in poi la pagina `/2/` mostrerà l'articolo `2`.

5. Per richiamare tutte le pagine degli articoli procedere come segue:

   1. Modificare il template ove elencare gli articoli `all_blogs.html`:

      ```html
      {% for article in articles %}
      
      <a href="{% url 'blog:detail' article.id %}"><h2>{{ article.title }}</h2></a>

      {% endfor %}
      ```

      Ogni `article` ha il suo `id` numerico: 1,2,3,4,... Quel numero è l'`article.id`. `blog:detail` indica il nome dato alla URL associata al template che si vuole usare con l'`article.id` (E.s.: blog/1, blog/2, ...)

   2. Modificare `urls.py` aggiungendo `app_name = <nomeApp>`:

      ```python
      from django.urls import path
      from . import views

      app_name = 'blog'
      
      urlpatterns = [
         path('', views.all_blogs, name='all_blogs'),
         path('<int:article_id>/', views.detail, name='detail')
      ]
      ```
      
   3. Da questo momento in avanti tutte le direttive che vorranno usare i **name** definiti in `urls.py` dovranno usare il prefisso `app_name:`:
      Esempio:
      
      * `app_name = paperino` (in `urls.py`)
      * `{% url 'paperino:<nameUrlPattern>'%}` (in `template.html`)

## Come ottenere il numero di oggetti creati su una pagina HTML

Inserire nel posto voluto la direttiva `{{ <dict>.count }}` dove `<dict>` è il dizionario dato in `views.py` alla funzione legata al template.

Esempio:

* `views.py`:

  ```python
  def detail(request, article_id):
      articles = get_object_or_404(Article, pk=article_id)
      return render(request, 'blog/detail.html', {'article': articles})
  ```
  
* `vim blog/all_blogs.html`

  ```html
  <h2>Marco ha scritto {{ articles.count }} articoli</h2>
  ```
  
  **Nota:** Prestare attenzione a quanto fatto in [Utilities di objects()](#Utilities-di-objects) perchè potrebbe limitare il numero di oggetti visibili.
  
## Come gestire le pluralità nel testo (inglese di default)

Nel testo del template che cambia da singolare (e.s.: "article") a plurale (e.s.: "articles") è utile usare:

* `<h2>Marco has written {{ articles.count }} article{{ articles.count|pluralize }}</h2>`

## Funzioni Utili per la manipolazione dei valori visualizzati

Se si è interessati a modificare la visualizzazione di un valore, agire come indicato nella [documentazione](https://docs.djangoproject.com/en/3.2/ref/templates/builtins/#date):
* Con `{{ article.date|date:"SHORT_DATE_FORMAT" }}` si ottiene una data nel formato "Mese/Giogno/Anno" (con 01,02,03,...)
* Con `{{ article.date|date:"j-n-Y" }}` si ottiene una data nel formato "Giorno-Mese-Anno" (con 1, 2, 3, ...)
* Con `{{ article.date|upper }}` si ottiene una data nel formato "Anno-Mese-Giorno" (con 01, 02, 03, ...)
* Con `{{ article.date }}` si ottiene una data nel formato "June 2, 2021"
* Con `{{ article.title|upper }}` si ottiene una testo tutto MAIUSCOLO
* Con `{{ article.description|truncatechars:100 }}` si ottiene un testo troncato al centesimo carattere sostituendo il resto con dei "..."

## Estendere un template base

1. Creare il template base `base.html` nella cartella `template` che sarà esteso a seconda delle esigenze:
   * `vim portfolio/template/portfolio/base.html`

     ```html
     <!-- Parte componibile da getboostrap.com:
          https://getbootstrap.com/docs/5.0/getting-started/introduction/#starter-template 
     -->
     <!doctype html>
     <html lang="en">
       <head>
         <!-- Required meta tags -->
         <meta charset="utf-8">
         <meta name="viewport" content="width=device-width, initial-scale=1">

         <!-- Bootstrap CSS -->
         <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-+0n0xVW2eSR5OomGNYDnhzAbDsOXxcvSN1TPprVMTNDbiYZCxYbOOl7+AMvyTG2x" crossorigin="anonymous">
         
         <!-- Custom CSS -->
         <link rel="stylesheet" href="{% static 'portfolio/custom.css' %}">
         
         <!-- Custom Favicon -->
         <link rel="icon" type="image/png" href="{% static 'portfolio/favicon.png' %}">

         {% block title %}{% endblock %}
       </head>
       <body>
         {% block content %}{% endblock %}

         <!-- Option 1: Bootstrap Bundle with Popper -->
         <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.1/dist/js/bootstrap.bundle.min.js" integrity="sha384-gtEjrD/SeCtmISkJkNUaaKMoLD0//ElJ19smozuHV6z3Iehds+3Ulb9Bn9Plx0x4" crossorigin="anonymous"></script>
       </body>
     </html>
     ```
     
     * `{% block content %}{% endblock %}` è il luogo in cui estenderò questo template base con altro contenuto. `content` è il nome del blocco da estendere.
     * `{% static 'portfolio/favicon.png' %}` e `{% static 'portfolio/custom.css' %}` sono il riferimento a 2 file statici

2. Creo il contenuto della pagina in un secondo template `homepage.html` che estenderà il template base `base.html`:
   * `vim portfolio/template/portfolio/homepage.html`

     ```html
     {% extends 'porfolio/base.html' %}
     
     {% block content %}
     ...contenuto da inserire...
     {% endblock %}
     ```
## Web Interface Utilities

* [Lorem Ipsum Generator](https://www.webfx.com/tools/lorem-ipsum-generator/)
* [Google Fonts](https://fonts.google.com/)

## Source Code Utilities

* [gitignore.io](https://www.toptal.com/developers/gitignore)

## Usare settings diversi in produzione e in development

1. Aggiungere in fondo al `settings.py` del progetto quanto segue:

   ```python
   try:
      from .local_settings import *
   except ImportError:
      print("Nessun local_settings.py. Si lavorando in produzione!")
   ```

2. Creare il file `local_settings.py` dentro la la stessa cartella di `settings.py` con:

   ```python
   DEBUG = True
   ALLOW_HOSTS = []
   ```

3. Avviare il server con:
   * `python3 migrate.py runserver`

## Autenticazione degli utenti

Come sempre dovremo procedere per gradi:

1. Avviare il server di sviluppo:
   * `python3 manage.py runserver`

2. `*-project/settings.py`:

   * Aggiungere la nuova app `todo` (nome dell'app) alle `INSTALLED_APPS`:

     ```python
     INSTALLED_APPS = [
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
      'todo',
     ]
     ```

3. `*-project/urls.py`:

   1. Importare le `views` dell'app:

      ```python
      from django.contrib import admin
      from django.urls import path
      from todo import views
      ```

   2. Decidere il path e assegnargli un nome univoco:

      ```python
      urlpatterns = [
          path('admin/', admin.site.urls),

          # Homepage
          path('', views.homepage, name='homepage'),

          # Sign Up
          path('signup/', views.signupuser, name='signupuser'),

          # Login
          path('login/', views.loginuser, name='loginuser'),

          # Logout
          path('logout/', views.logoutuser, name='logoutuser'),
         
          # Another page
          path('current/', views.currenttodos, name='currenttodos')
      ]
      ```

4. `todo/views.py`:

   1. Importare le librerire necessarie:

      ```python
      from django.shortcuts import render, redirect
      from django.contrib.auth.forms import UserCreationForm, AuthenticationForm
      from django.contrib.auth.models import User
      from django.db import IntegrityError
      from django.contrib.auth import login, logout, authenticate
      ```

   2. Definire le funzioni corrispondenti ai path di `urls.py`:

      ```python
      def signupuser(request):
          renderDict = {'form': UserCreationForm()}
          if request.method == 'GET':
              return render(request, 'todo/signupuser.html', renderDict)
          else:
              # Controllo che le password inserite nella form coincidano.
              # 'password1' e 'password2' sono i name degli <input> che formano la form
              if request.POST['password1'] == request.POST['password2']:
                  try:
                      # Creo l'utente con username = 'username' e password = 'password1'
                      # solo se le password inserite coincidono.
                      #
                      # https://docs.djangoproject.com/it/3.2/topics/auth/default/#creating-users
                      user = User.objects.create_user(request.POST['username'], password=request.POST['password1'])
                      user.save()
                      # Accedo con l'utente
                      #
                      # https://docs.djangoproject.com/it/3.2/topics/auth/default/#how-to-log-a-user-in
                      login(request, user)
                      # Redirigo alla pagina che l'utente può vedere da autenticato.
                      #
                      # https://docs.djangoproject.com/en/3.2/topics/http/shortcuts/#redirect
                      return redirect('currenttodos')
                  except IntegrityError:
                      renderDict['error'] = 'Username già esistente. Scegliere altro valore.'
                      return render(request, 'todo/signupuser.html', renderDict)
              else:
                  # Entro in questo else se le password inserite non coincidono
                  renderDict['error'] = 'Le password immesse non coincidono.'
                  return render(request, 'todo/signupuser.html', renderDict)

      def loginuser(request):
          renderDict = {'form': AuthenticationForm()}
          if request.method == 'GET':
              return render(request, 'todo/loginuser.html', renderDict)
          else:
              # 'username' è il name della <input> destinata alla username che forma la form
              # 'password' è il name della <input> destinata alla password che forma la form
              user = authenticate(request, username=request.POST['username'], password=request.POST['password'])
              if user is None:
                  renderDict['error'] = 'Utente e Password non trovati'
                  return render(request, 'todo/loginuser.html', renderDict)
              else:
                  # Accedo con l'utente
                  #
                  # https://docs.djangoproject.com/it/3.2/topics/auth/default/#how-to-log-a-user-in
                  login(request, user)
                  # Redirigo alla pagina che l'utente può vedere da autenticato.
                  #
                  # https://docs.djangoproject.com/en/3.2/topics/http/shortcuts/#redirect
                  return redirect('currenttodos')

      def logoutuser(request):
          # E' importante che il Logout sia una POST altrimenti
          # i browser web tenteranno di aprire la sua URL e
          # automaticamente disconnetteranno l'utente loggato.
          if request.method == 'POST':
              logout(request)
              return redirect('homepage')

      def currenttodos(request):
          return render(request, 'todo/currenttodos.html')

      def homepage(request):
          return render(request, 'todo/homepage.html')
      ```

5. `todo/templates/todo/<nomeTemplate>.html`

   1. `base.html`: template di base da estendere. Servirà a visualizzare chi è loggato e permettergli il Logout.

      ```html
      {% if user.is_authenticated %}

      Logged in as {{ user.username }}

      <form method="POST" action="{% url 'logoutuser' %}">
          {% csrf_token %}
          <button type="submit">Logout</button>
      </form>

      {% else %}

      <a href="{% url 'signupuser' %}">Sign Up</a>
      <a href="{% url 'loginuser' %}">Login</a>

      {% endif %}


      {% block content %}{% endblock %}
      ```

   2. `signupuser.html`: template usato per la registrazione dell'utente e conseguente creazione del suo account su Django Admin.

      ```html
      {% extends 'todo/base.html' %}

      {% block content %}
      
      <h1>Sign Up</h1>
      
      <h2>{{ error }}</h2>

      <form method="POST">
          {% csrf_token %}
          {{ form.as_p }}
          <button type="submit">Sign Up</button>
      </form>
      
      {% endblock %}
      ```

      `.as_p` inserirà ogni elemento in un `<p>` TAG cambiandone la visualizzazione.

   3. `currenttodos.html`: template usato per redirigere l'utente autenticato in uno spazio visibile solo a lui.

      ```html
      {% extends 'todo/base.html' %}

      {% block content %}

      <h1>CURRENT TODOS</h1>

      {% endblock %}
      ```

   4. `homepage.html`: template usato per la Homepage del sito.

      ```html
      {% extends 'todo/base.html' %}

      {% block content %}

      <h1>This is the Homepage</h1>

      {% endblock %}
      ```

6. Fermare il server di sviluppo e creare un utente superuser per l'accesso all'interfaccia amministrativa di Django:

   * `python3 manage.py createsuperuser`

7. Avviare nuovamente il server di sviluppo:

   * `python3 manage.py runserver`

## Models e relazioni con gli utenti

Questo esempio si basa sul modello `todowoos` che memorizza note per ciascun utente autenticato.

* `todo/models.py`

  ```python
  from django.db import models
  from django.contrib.auth.models import User


  class Todo(models.Model):
      # Questa funzione permette di rinominare ogni nuovo oggetto
      # con il suo titolo
      def __str__(self):
          return self.title

      title = models.CharField(max_length=100)
      memo = models.TextField(blank=True)
      created = models.DateTimeField(auto_now_add=True)
      # devo usare "blank=True" se voglio poter usare i campi vuoti
      datecompleted = models.DateTimeField(null=True, blank=True)
      important = models.BooleanField(default=False)
      user = models.ForeignKey(User, on_delete=models.CASCADE)
  ```

* `todo/admin.py`:

  ```python
  from django.contrib import admin
  from .models import Todo

  
  # Questa classe mostra i campi che non sono modificaili
  class TodoAdmin(admin.ModelAdmin):
      readonly_fields = ('created',)

  # Voglio vedere questo modello nell'interfaccia "admin/"
  admin.site.register(Todo)
  ```

## Creare una custom Form

Obiettivo: Memorizzare quanto inserito dall'utente limitatamente ai campi che desidero.

1. Creo il path in `urls.py`:

   ```python
    # Create
    path('create/', views.createtodo, name='createtodo')
    ```

2. Creo la nuova `TodoForm` in `todo/forms.py`:

   ```python
   # Ho importato i modello Todo per avere accesso ai suoi campi
   from django.forms import ModelForm
   from .models import Todo 


   class TodoForm(ModelForm):
       class Meta:
           model = Todo
           fields = ['title', 'memo', 'important']
   ```

   I fields/campi indicati saranno gli unici ad essere visualizzati.

3. Creo la funzione `createtodo` in `views.py` che richiama la mia nuova `TodoForm`:

   ```python
   from .forms import TodoForm
   # ...
   def createtodo(request):
       renderDict = {'form': TodoForm()}
       if request.method == 'GET':
           # Questo è il ramo seguito se non viene inviato alcun dato a Django, GET
           return render(request, 'todo/createtodo.html', renderDict)
       else:
           try:
               # Questo è il ramo seguito se vengono inviati dei dati a Django
               form = TodoForm(request.POST)
               # creo l'oggetto 'newtodo' prelevando i dati dalla form
               # e con il 'commit=False' non salvo subito su DB quanto inviato alla form
               newtodo = form.save(commit=False)
               # aggiungo l'utente autore della creazione e salvo
               newtodo.user = request.user
               newtodo.save()
               return redirect('currenttodos')
           except ValueError:
               renderDict['error'] = "Inserimento dati non corretto. Prova ancora!"
               return render(request, 'todo/createtodo.html', renderDict)
   ```

4. Creo il template usato da `createtodo` in `templates/todo/createtodo.html`:

   ```html
   {% extends 'todo/base.html' %}

   {% block content %}

   <h1>Create</h1>

   <h2>{{ error }}</h2>

    <form method="POST">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Save</button>
    </form>

    {% endblock %}
   ```

5. Apro la pagina `/create` e vedo la mia nuova Form. Dentro vi potrò scrivere quello che serve e salvarlo nel DB gestito da Django.
