# Começando com o Django

* OBS.: $ Está sendo usado para simbolizar o uso de códigos, não add ele no terminal

### 1 - Criando um ambiente virtual
   * Criamos uma pasta para armazenar os projetos

``` shell
   $ mkdir nomePasta
   $ cd nomePasta
```
   * Dentro da pasta, vamos criar um ambiente virtual chamado myenv
``` sell
   $ python3 -m venv myenv
```


### 2 - Iniciando o ambiente virtual(Fazer isso sempre)
   * Dentro da pasta criada anteriormente 
``` shell
   $ source myenv/bin/activate #(Sendo myenv o nome do ambiente virtual)
```


* Se tudo funcionar, no terminal vai aparecer (myenv) no inicio da linha

### 3 - Criando um projeto
* Com o ambiente virtual ativado

``` shell
   $ django-admin startproject mysite #(Sendo mysite o nome do projeto que se deseja criar)
```
* Se tudo funcionar, no diretório criado mysite, vai existir um arquivo manage.py e uma pasta mysite

### 4 - Criando um banco de dados (Com o Sqlite3)

``` shell
   $ python manage.py migrate
```
* Feito isso, podemos rodar o projeto para ver se está tudo funcionando
``` shell
   $ python manage.py runserver #(Comando deve ser executado na pasta que tem o arquivo manage.py)
``` 
* Para saber se funcionou, no navegador digite http://127.0.0.1:8000/

### 5 - Modelos
* Antes de criar modelos, devemos criar um aplicativo para nosso projeto:

``` shell
$ python manage.py startapp blog #(Sendo blog o nome do app)
```
* Depois de criar um aplicativo, devemos dizer ao Django para usá-lo. Fazemos isso no arquivo mysite/settings.py, na linha onde tem 
	INSTALLED_APPS, e adicionar uma linha conforme descrito abaixo:

``` python
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog', # essa linha
)
```

* Para nosso blog, vamos querer criar um modelo que armazene nome do autor, titulo da postagem, texto da postagem, data de criação e data da 
publicação, além de um método que publique nossas postagens, para isso, no arquivo blog/models.py, fazemos: 

``` python	
 from django.db import models
 from django.utils import timezone
 class Post(models.Model):
	author = models.ForeignKey('auth.User') 
	title = models.CharField(max_length=200)
	text = models.TextField()
	created_date = models.DateTimeField(default=timezone.now)
	published_date = models.DateTimeField(blank=True, null=True)

	def publish(self):
		self.published_date = timezone.now()
		self.save()

	def __str__(self):
		return self.title


```
* Para criar no BD as tabelas referentes aos modelos execute:

``` shell
 $ python manage.py makemigrations blog (Sendo blog o nome do app)
```


* Com isso o Django prepara um arquivo de migração, que deve ser aplicado com o comando abaixo

``` shell
 $ python manage.py migrate blog
```
* Com isso o banco foi criado, porém para add e remover dados, precisamos passar o nome dos modelos para o arquivo blog/admin.py
 
``` python
 from django.contrib import admin
 from .models import Post

 admin.site.register(Post)
```

* Feito isso, acessando http://127.0.0.1:8000/admin/ no navegador, uma pagina pedindo autenticação vai aparecer, no terminal   
execute o comando abaixo, para criar um usuário. 

``` shell
 $ python manage.py createsuperuser
```
* Depois é só acessar com seu login e senha, e uma tela contendo o nosso modelo Post será apresentada. 

### 6 - Versionando nosso código com GIT
* Dentro da pasta do nosso projeto (projetoDjango no nosso caso)

``` shell
 $ git init
 $ git config user.name "Seu nome"
 $ git config user.email "Seu_Email@email.com"
```

* Inicializar o repositório git é algo que só precisamos fazer uma vez por projeto 
* Git irá controlar as alterações para todos os arquivos e pastas neste diretório, mas existem alguns arquivos que queremos ignorar. Fazemos isso através da criação de um arquivo chamado .gitignore no diretório base. crie um novo arquivo com o seguinte conteúdo:

```
 *.pyc
 __pycache__
 myvenv
 db.sqlite3
 .DS_Store
```

* E salve como .gitignore na pasta de nível superior "djangogirls".
* Para verificar o que será salvo, use git status no terminal.

* Para salvar as edições 

``` shell
 $ git add --all
 $ git commit -m "My Django APP, first commit"
```

### 7 - Publicando código no Github
* Primeiro crie um repositório no Github
* Depois execute o comando abaixo, onde o link após git remote add origin, é o link do seu repositório no github

``` shell
 $ git remote add origin Link_repositorio_Github
 $ git push -u origin master
```
* Será solicitado usuário e senha do Github. 

### 8 - URLs
Obs.: Linhas que possuem comentários, foram removidas para melhorar a visualização. 
* Dentro do arquivo mysite/urls.py, temos o seguinte conteúdo:

``` python
 from django.conf.urls import include, url
 from django.contrib import admin

 urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
 ]

```
* Podemos notar que a URL do admin, visitada anteriormente, já está aqui, e é assim que quando digitamos no navegador, o Django sabe como acessar aquela página que nos foi apresentada. 
* Um problema que vamos encontrar se decidirmos adicionar todas as URLs do nosso projeto nesse arquivo, é que ele ficará grande demais, e consequentemente muito desorganizado, e por isso é uma boa prática, criar um arquivo chamado urls.py dentro de nosso app, onde nesse arquivo nós apenas apontaremos o caminho para o nosso arquivo presente no APP. Fazemos isso adicionando a linha, que se segue, logo abaixo da linha da url admin.

``` python
 url(r'', include('blog.urls')), #(Com a vírgula no final)
```
* O Django agora irá redirecionar tudo o que entra em 'http://127.0.0.1:8000 /'para blog.urls e procurar por novas instruções lá.

* Em nosso arquivo blog/urls.py, que está vazio no momento, devemos adicionar em um primeiro momento, as linhas que se seguem:

``` python
 from django.conf.urls import include, url
 from . import views
```
* Aqui nós estamos apenas importando métodos do Django e todos os nossos views do aplicativo blog (ainda não temos nenhuma, mas teremos em um minuto!)
* Depois disso nós podemos adicionar nosso primeira URL padrão:

``` python
urlpatterns = [
    url(r'^$', views.post_list),
]
```
* Como pode se notar, estamos atribuindo uma view chamada post_list para nossa URL, porém essa view ainda não existe, então se for feito o acesso a http://127.0.0.1:8000, vamos ver uma tela de erro, mas isso será resolvido após a criação da view. 

### 9 - Views
* As views são armazenados no arquivo blog/views.py, elas recebem a lógica da nossa aplicação, e fazem o intermédio entre os modelos e os templates (Que ainda vamos criar), solicitando e filtrando os dados. 
* Dentro do arquivo, vamos adicioanr a função post_list: 

``` python
 from django.shortcuts import render
 def post_list(request):
 	return render(request, 'blog/post_list.html', {})
```

* Como você pode ver, nós criamos um método (def) chamado post_list que aceita o pedido e retornar um método render será processado (para montar) nosso modelo blog/post_list.html.
* Da forma como está, se acessarmos no navegador, agora teremos um novo erro, o "TemplateDoesNotExist", isso ocorre pois aida não criamos nenhum template, porém é passado o nome de um na função post_list, no caso é "post_list.html".

### 10 - Templates
* Dentro do diretório blog, vamos criar uma pasta chamada templates, e dentro do nosso diretório template, vamos criar um arquivo chamado post_list.html, vamos deixar em branco por agora. 
* Se executarmos agora no navegador, veremos uma tela em branco, isso significa que tudo funcionou, agora só precisamos preencher nosso template, de forma que ele exiba todos as nossas postagens cadastradas. 

* Para que o template possa exibir algo, a consulta que é feita na view, deve ser armazena em algum objeto e esse objeto deve ser passado numa sessão como paramêtro de "render", anteriormente nos fizemos a função post_list na nossa view, contendo o seguinte código:
```python
 return render(request, 'blog/post_list.html', {})
```

* Antes de tudo, vamos ter que conectar nossa view com nossos modelos e para isso, vamos importar nossos modelos no arquivo views.py:
``` python
 from django.shortcuts import render
 from .models import Post
```
* O ponto depois de from significa o diretório atual ou o aplicativo atual. Como views.py e models.py estão no mesmo diretório podemos simplesmente usar . e o nome do arquivo (sem .py). Então nós importamos o nome do modelo (Post).
* E o que vem agora? Para pegar os posts reais do model Post nós precisamos de uma coisa chamada QuerySet, que nada mais é do que uma lista de objetos de um dado modelo, como exemplo, podemos ter um QuerySet dos posts publicados ordenados pela data de publicação:

``` python
 Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
```

* Agora se colocarmos o pedaço de código acima dentro da função post_list, ficaremos com o seguinte código. 
 
``` python
 from django.shortcuts import render
 from django.utils import timezone
 from .models import Post

 def post_list(request):
     posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
     return render(request, 'blog/post_list.html', {})
```

* A última parte que falta é passar o QuerySet posts, e podemos fazer isso passando ele para o último parâmetro de render "{}", devendo ficar assim: 
	['posts': posts] 
* Onde o 'posts' entre aspas é um nome para podermos nos referenciar a lista posts, mão é obrigatório ter o mesmo nome da lista, mas é uma boa prática. 

11 - Usando tags de templates para exibir dados nos templates
* No capitulo anterior, nós fornecemos ao nosso template uma lista de postagens e a variável posts. Agora vamos exibir em nosso HTML.
* Para exibir uma variável no Django template, nós usamos colchetes duplos com o nome da variável dentro, exemplo:
``` python
 {{ posts }}
```
-- Dentro do nosso template adiciona a linha acima, o resultado deve se parecer com: 
	[<Post: My second post>, <Post: My first post>]
* Isto significa que o Django a entende como uma lista de objetos. E em Python podemos exibir listas utilizando loops! Em um template Django, fazemos isso da seguinte maneira:

``` python
 {% for post in posts %}
     {{ post }}
 {% endfor %}
```

* O resultado deve ser algo parecido com:
	My second post My first post
* Funcionou! Mas ainda não exibe de uma boa forma, para isso podemos adicionar algumas tags HTML:
``` html
	<div>
	    <h1><a href="/">Django Girls Blog</a></h1>
	</div>

	{% for post in posts %}
	    <div>
		<p>published: {{ post.published_date }}</p>
		<h1><a href="">{{ post.title }}</a></h1>
		<p>{{ post.text|linebreaksbr }}</p>
	    </div>
	{% endfor %}	

```
* Tudo que você põe enrte ```{% for %}``` e ```{% endfor %}``` será repetido para cada objeto na lista. Atualize sua página:

