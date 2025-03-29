> Intro
> 
Projeto criado para servir como passo a passa para a criação de um projeto basico de livraria em django, utilizando djangorest-framework, allauth para autenticação, swagger e postgress como db.

> Back end

Primeiramente sera criado um ambiente virtual para que as dependencias do projeto possam ser intaladas dentro desse ambiente virtual as dependências do Django e outras bibliotecas relacionadas ao backend. Nesse projeto o ambiente virtual tera o nome de ProjetoDev

`python -m venv ProjetoDev`

`ProjetoDev\Scripts\activate`

Após a criação e ativação do ambiente virtual, é preciso que o django seja instalado nesse ambiente para que possamos iniciar o projeto. 

`pip install django`

vamos instalar a biblioteca Django rest framwork, que vai facilitar bastante a criação das views (lembrar de adicionalo ao settings.py. 

`pip install djangorestframework`

executar o comando para criar um novo projeto Django :

`django-admin startproject ProjetoPatoEvents`

Vamos instalar esta biblioteca que vai facilitar para ler as variáveis no settings.py.

`pip install python-decouple`

Como nesse projeto iremos utilizar o PostgreSQL, devemos instalar o  o pacote psycopg2:

`pip install psycopg2`

Para gerar a documentação (após instalar inserir nos installed apps)

`pip install drf-yasg`

Também vamos precisar adicionar esta linha logo abaixo de STATIC_URL:

`STATIC_URL = 'static/'
STATIC_ROOT = 'staticfiles'` 

Após rodar o comando `python manage.py collectstatic. `

vamos tambem adicionar as urls do swagger no arquivo principal de urls

```
from django.contrib import admin
from django.urls import path , include
from drf_yasg import openapi
from drf_yasg.views import get_schema_view
from rest_framework import permissions

schema_view = get_schema_view(
    openapi.Info(
        title='ProjetoPatoEvents API',
        default_version='v1',
    ),
    public=True,
    permission_classes=[permissions.AllowAny],
)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/swagger.<slug:format>)', schema_view.without_ui(cache_timeout=0), name='schema-json'),
    path('api/swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
]
```
Precisamos tambem instalar o Pillow para  lidar com campos de imagem,

`python -m pip install Pillow`

para recebermos os itens de imagem corretamente devemos adicionar em urls.py

```

# urls.py

from django.conf import settings
from django.conf.urls.static import static
from django.contrib import admin
from django.urls import path

urlpatterns = [
    # ... suas outras URLs ...]

# Adicione a configuração para servir as mídias durante o desenvolvimento
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

Após abrimos o arquivo settings.py e adicionamos nele 
```
DATABASES = {
    'default': {
      'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```
> Criando os APPs

Com o banco de dado ja definido agora sera necessario criar os APPs para nosso projeto. Mas antes iremos criar uma pasta para armazenar as imagens do nosso projeto.    No diretorio raiz do projeto criamos uma pasta chamada 'media'. Após adicionamos em settings.py

```
from pathlib import Path
import os

BASE_DIR = Path(__file__).resolve().parent.parent
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

Para iniciar esse projeto precisamos criar os app que iram compor ele. Normalmente um app é usado para reperesentar um domínio da aplicação. No caso do projeto Pato Events podemos pensar em alguns domínios: 

- **Usuarios**: incluindo as tabelas de usuário que armazena informações do usuário, como nome, email, senha, etc. Pode ser estendido do modelo de usuário fornecido pelo Django e a tabela de perfil de Usuário que  armazena informações adicionais ao usuário, como imagem de perfil, biografia, etc. Relacionado a um usuário por meio de um relacionamento um para um.

- **Eventos**: Inclui uma tabela de evento que armazena informações sobre o evento, como título, descrição, data, localização, etc. Relacionado a um usuário por meio de um relacionamento muitos para um (um usuário pode criar vários eventos). E uma tabela de participante que mapeia a participação de usuários em eventos específicos. Pode conter informações adicionais sobre a participação, como status de confirmação, papel no evento, etc. Relacionado a um evento e a um usuário por meio de relacionamentos muitos para um.

Esses seriam os APPs mais importantes para nosso projeto, com o tempo podemos adicionar apps que iram tratar de outras funcionalidades como por exemplo as notificações enviadas para os usuarios.

### API 

Primeiramente sera criado o APP para os usuarios e para os eventos

`python manage.py startapp Usuarios` 

`python manage.py startapp Eventos` 


Após a criação do APPs deve-se cadastra-los no arquivo settings.py 

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions', 
    'django.contrib.messages',
    'django.contrib.staticfiles',
     'drf_yasg',
    'rest_framework',
    'Usuarios',
    'Eventos',
]
```

Agora no dentro do APP de eventos no arquivo models.py sera criado as classes que dos eventos junto com a classe endereço e categoria.

```
from django.db import models

class Categoria(models.Model):
    nome = models.CharField(max_length=255)
    
    def __str__(self) -> str:
        return self.nome

class Evento(models.Model):
    
    titulo = models.CharField(max_length=255)
    data = models.DateField()
    data_postagem = models.DateTimeField(default=timezone.now)
    descricao = models.TextField()
    valor_entrada = models.DecimalField(max_digits=10, decimal_places=2)
    foto_principal = models.ImageField(upload_to='media/eventos/', blank=True, null=True)
    cep = models.CharField(max_length=8, default='00000000')
    rua = models.CharField(max_length=255, null=True)
    numero = models.IntegerField(null=True)
    cidade = models.CharField(max_length=255, default='Pato Branco')  
    categoria = models.ForeignKey(Categoria, related_name='eventos_categoria', on_delete=models.CASCADE,null=True)

    def __str__(self) -> str:
        return self.titulo
```
Depois de criar o modelo de Evento em models.py, devemos registralo em admin.py

```
from Eventos.models import Evento, Categoria

@admin.register(Evento)
class EventoAdmin(admin.ModelAdmin):
    # campos que vão aparecer na listagem
    list_display = ('id', 'titulo', 'data','data_postagem',
                    'descricao','valor_entrada','foto_principal','cep','rua','numero','cidade','categoria',)


@admin.register(Categoria)
class CategoriaAdmin(admin.ModelAdmin):
    # campos que vão aparecer na listagem
    list_display = ('id', 'nome')
```

Depois de adicionar o app no settings.py e registrar a tabela no admin.py é necessário rodar o comando: 

`python manage.py makemigrations` e `python manage.py migrate`

Após pode-se criar um superuser com o comando `python manage.py createsuperuser`, definir usuario e senha para que se possa acessar o painel admin do djungo 

Para isso deve-se digitar o comando  `python manage.py runserver` e acessar a porta 8000. 

Caso esteja tudo funcionando é possivel então prosseguir para os serializers, no arquivo de views.py criar: 

```
from django.shortcuts import render
from rest_framework import serializers
from rest_framework import viewsets
from Eventos.models import Evento, Categoria


class EventoSerializer(serializers.ModelSerializer):

    foto_principal = serializers.ImageField(max_length=None, use_url=True, required=False)

    class Meta:
        model = Evento
        # campos que queremos serializar
        fields = ('id', 'titulo', 'data','data_postagem',
                    'descricao','valor_entrada','foto_principal','cep','rua','numero','cidade','categoria',)

class CategoriaSerializer(serializers.ModelSerializer):
    class Meta:
        model = Categoria
        # campos que queremos serializar
        fields = ('id', 'nome')

class CategoriaView(viewsets.ModelViewSet):
    queryset = Categoria.objects.all() 
    serializer_class = CategoriaSerializer  
    http_method_names = ['get', 'post', 'put', 'delete']

class EventoView(viewsets.ModelViewSet):
    queryset = Evento.objects.all() 
    serializer_class = EventoSerializer  
    http_method_names = ['get', 'post', 'put', 'delete']   
```

No app Eventos agora criar um arquivo chamado urls.py com o conteúdo:

```
from django.urls import path, include
from rest_framework import routers

from Eventos.views import EventoView


router = routers.DefaultRouter()
router.register('eventos', EventoView)  # nome do objeto da view

urlpatterns = [
    path('eventos/', include(router.urls)),  # nome do app
]
```
Desta maneira teremos definido o endpoint http://127.0.0.1:8000/eventos/eventos/ (get, post, put e delete), onde o primeiro "eventos" é referente ao app e o segundo é referente ao objeto eventos.

No urls.py que já existe no package "ProjetoPatoEvents" será necessário incluir as urls do app eventos:

```
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('Eventos.urls')),
    path('', include('Usuarios.urls')),
    path('api/swagger.<slug:format>)', schema_view.without_ui(cache_timeout=0), name='schema-json'),
    path('api/swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
]
```
Após a criação da tabela eventos vamos trabalhar nos usuarios, para isso iremos começar instalando a lib para autenticação por JWT.

`pip install djangorestframework_simplejwt`    

Depois precisamos adcionar no settings.py 
```
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}
```
adicionar no INSTALLED_APPS:

`'rest_framework_simplejwt'`

Para trabalhar com a autenticação de usuarios devemos instalar as bibliotecas dj-rest-auth e django-allauth.

`pip install 'dj-rest-auth[with_social]'`

Nos ISTALLED_APPS, adicionar:
```
   ...
    'rest_framework.authtoken'
    'dj_rest_auth',
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'dj_rest_auth.registration',
    ...
```
e novas variáveis no settings:

```
...
SITE_ID = 1

REST_AUTH = {
    'USE_JWT': True,
    'JWT_AUTH_HTTPONLY': False,
}
```
Nas urls vamos adicionar a url:

```  ...
    path('accounts/', include('dj_rest_auth.urls')),
    ...
```

para que possa ser feita a recuperação de senha e cadastro de novos usuarios

adicionar no settings
`EMAIL_BACKEND = "django.core.mail.backends.console.EmailBackend"`

adicionar na url.py
```
...
from dj_rest_auth.views import PasswordResetConfirmView
...
    path(
        'api/accounts/password/reset/confirm/<str:uidb64>/<str:token>',
        PasswordResetConfirmView.as_view(),
        name='password_reset_confirm',
    ),
    path('accounts/registration/', include('dj_rest_auth.registration.urls')),
```

Com isso dois endpoints novos vão aparecer no swagger:

POST accounts/registration/
POST accounts/registration/verify-email/
O primeiro permite criar um novo usuário. Caso esteja configurado (no momento não está), ao criar o usuário é enviado um e-mail para que a pessoa possa fazer a confirmação de e-mail. Esta é feita pelo segundo endpoint.

para começarmos conectando com o front precisamos instalar 

`pip install django-cors-headers`

adicionar ele no installed_APPS

```
  INSTALLED_APPS = [
    # ...
    'corsheaders',
    # ...
]
```

Adicione 'corsheaders.middleware.CorsMiddleware' em Middleware: 
```
MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    # ...
]
```
e adicionar em settings.py para definir quais cabeçalhos permitir na configuração do CORS

`CORS_ALLOW_ALL_HEADERS = True`
