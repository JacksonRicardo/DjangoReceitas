# django-receitas

0. [Rodando a aplicação](#0-rodando-a-aplicação)  
1. [Preparando o ambiente virtual](#1-preparando-o-ambiente-virtual)  
2. [Criando o projeto](#2-criando-o-projeto)  
3. [Apps](#3-apps)  
  3.1. Criando um app  
  3.2. Registrando um app  
  3.3. Primeira rota  
4. [Arquivos estáticos](#4-arquivos-estáticos)  
  4.1. Definindo arquivos estáticos no HTML  
  4.2. Criando uma nova _view_  
  4.3. Extendendo código HTML e _partials_  
  4.4. _Partials_  
  4.5. Apresentando informações de forma dinâmica  
5. [Banco de Dados](#5-banco-de-dados)  
6. [_Models_ e _migrations_](#6-_models_-e-_migrations_)  
  6.1 _Models_  
  6.2 _Migrations_  
7. [Django Admin](#7-django-admin)  
  7.1 Customizando apresentação de dados  
8. [Recuperando dados do Banco e apresentando nas _views_](#8-recuperando-dados-do-banco)  
  8.1 Listando as receitas  
  8.2 Detalhes de uma receita  
9. [Integrando apps](#9-integrando-apps)  
  9.1 Criando um novo app  
  9.2 Relacionamento _Many-to-one_  
10. [_Upload_ de arquivos](#10-_upload_-de-arquivos)  
  10.1 Imagens  
11. [Formulários](#11-formulários)  
  11.1 App de usuários  
  11.2 Rotas de cadastro e _login_  
  11.3 Páginas de cadastro e _login_  
  11.4 Requisições de formulários  
  11.4.1 Enviando o formulário  
  11.4.2 Acessando os campos de um formulário  
  11.5 Cadastrando novo usuário  
  11.6 Realizando _login_  
  11.7 Formulário para receitas  
12. [Mensagens de _feedback_](#12-mensagens-de-_feedback_)  
13. [Finalizando o CRUD de receitas](#13-finalizando-o-crud-de-receitas)  
  13.1 Deletando uma receita  
  13.2 Editando uma receita  
14. [Refatorando o projeto](#14-refatorando-o-projeto)  
  14.1 Removendo um app  
15. [Paginação](#15-paginação)  

## 0 Rodando a aplicação

Execute os comandos abaixo na raiz do diretório clonado:

1. Criar ambiente virtual: `python3 -m venv ./venv`
2. Carregar ambiente virtual: `source $(pwd)/venv/bin/activate`
3. Instalar dependências (certifique-se que o ambiente virtual está ativado): `pip install -r requirements.txt`
4. Carregar arquivos estáticos: `python manage.py collectstatic`
5. Criar uma base de dados no PostgreSQL e adicionar as credênciais de acesso na váriavel `DATABASES` em `djangoreceitas/settings.py`
6. Rodar as _migrations_: `python manage.py migrate`
7. Criar um _super user_ para o Django Admin: `python manage.py createsuperuser`
8. Subir o servidor local: `python manage.py runserver`

## 1 Preparando o ambiente virtual

- Instalação do pacote no sistema:

```terminal
apt-get install python3-venv
```

- Criação do ambiente dentro da pasta que vai ficar a aplicação:

```terminal
mkdir django-receitas
cd django-receitas
python3 -m venv ./venv
```

- Carregando o ambiente: `source <full path>/django-receitas/venv/bin/activate`

```terminal
source <full path>/django-receitas/venv/bin/activate
```

ou, na raiz do diretório da aplicação:

```terminal
source $(pwd)/venv/bin/activate
```

- Instalação do Django **no ambiente virtual**:
  
```terminal
pip install django
```

- Verificando módulos instalados no ambiente virtual:
  
```terminal
pip freeze
```

[↑ voltar ao topo](#django-receitas)

## 2 Criando o projeto

- Lista de comandos disponíveis na _cli_ do Django:

```terminal
django-admin help
```

- Criando o projeto da aplicação 
```terminal
django-admin startproject djangoreicetas .
```

- Subindo o servidor:

```terminal
python manage.py runserver
```

[↑ voltar ao topo](#django-receitas)

## 3 Apps

Os Apps do Django podem ser entendidos como "sub-aplicações" com a finalidade de representar domínios da aplicação. No sistema desenvolvido aqui, por exemplo, vamos ter um app resposável por todo o gerenciamento das receitas.

Um projeto é uma coleção de configurações e apps que formam um _site_; e um app é uma "sub-aplicação" com uma finalidade/responsabilidade bem definida, podendo fazer parte de vários projetos.

### 3.1 Criando um app

Criando um app na raiz da aplicação:

```terminal
python manage.py startapp receitas
```

Para criar o app dentro de uma pasta dedicada, primeiro devemos criar essa pasta e então podemos passar como segundo parâmetro o caminho onde o app deverá ser criado:

```terminal
mkdir apps
mkdir apps/receitas
python manage.py startapp receitas ./apps/receitas
```

### 3.2 Registrando o app

Verificamos o nome do app no arquivo `apps/receitas/apps.py` e adicionamos na lista `INSTALLED_APPS` do arquivo `djangoreceitas/settings.py`. Se o app não estiver na raiz do projeto é necessário colocar seu _dot path_ . Implementando dessa forma é necessário sempre adicionar o `apps.` junto ao nome do app ao fazer sua importação.

Uma outra abordagem é definir a raiz dos apps nas configurações da aplicação, em `djangoreceitas/settings.py`. Assim poderíamos referenciar os apps apenas por seus nomes, sem a necessidade de passar o `apps.` antes:

```python
import os.path, sys

PROJECT_ROOT = os.path.dirname(__file__)
sys.path.insert(0, os.path.join(PROJECT_ROOT, '../apps'))

INSTALLED_APPS = [
    ...
    'receitas',
    'usuarios',
]
```

### 3.3 Primeira rota

Devemos criar um arquivo de rotas para o app de receitas:

```terminal
touch apps/receitas/urls.py
```

Após isso podemos fazer os "importes" necessários e mapear a rota `/` desse grupo para o método `index()`, que será definido no arquivo `apps/receitas/views.py`, retornando uma resposta HTTP com um HTML.

E por último devemos registrar o arquivo de rotas do app receitas em `djangoreceitas/urls.py` 

[↑ voltar ao topo](#django-receitas)

## 4 Arquivos estáticos

Podemos modificar o método `index()` no arquivo `apps/receitas/views.py` para ao invés de retornar uma resposta HTTP, retornar um arquivo HTML rendereizado. Então primeiro criamos o arquivo `apps/receitas/templates/index.html` que vai conter todo o HTML da página e modificamos o método `index()` .

Agora podemos começar a adicionar o estilo das páginas HTML nos chamados **arquivos estáticos**. A primeira coisa a ser feita é especificar onde a apliacação deve procurar pelas páginas HTML. Como temos apenas um app, basta colocar o caminho da pasta `templates` desse app na lista mapeada para a chave `DIRS` do dicionário na lista `TEMPLATES` do arquivo de configurações do projeto (`djangoreceitas/settings.py`).

Ainda no arquivo de configurações, devemos especificar o local em que os arquivos estáticos vão ficar. No final do arquivo, perto de `STATIC_URL` adicionamos duas variáveis:

- `STATIC_ROOT`: local em que os arquivos estáticos vão ficar
- `STATICFILES_DIRS`: diretórios que contém os arquivos estáticos

não esqueça de adicionar o `import os.path` caso ele não esteja lá.

Após isso podemos de fato adicionar os arquivos estáticos em `djangoreceitas/static`. E por último, "carregamos" os arquivos estáticos para a pasta `static` na raiz da aplicação. Nesse procedimento o Django faz uma cópia de todos arquivos estáticos da aplicação para essa pasta na raiz, para poder manipulá-los melhor:

```terminal
python manage.py collectstatic
```

É uma boa prática manter esses arquivos estáticos fora do repositório pois conforme nossa aplicação for crescendo, mais arquivos serão "copiados" para essa pasta. E também porque esse processo, em geral, é um dos passos realizados para o _deploy_ da aplicação .

### 4.1 Definindo arquivos estáticos no HTML

Vamos adicionar arquivos HTML mais completos para o app de receitas, agora fazendo uso dos arquivos estáticos que foram adicionados .

Para que o estilo seja aplicado nas páginas HTML é necessário informar ao Django que existem arquivos estáticos e isso é feito usando código Python :

- na primeira linha do arquivo HTML adicionamos `{% load static %}`, indicando que seram carregados arquivos estáticos
- em todas as referências de caminho devemos adicioná-las utilizando uma sintaxe específica para que o código Python possa ser interpretado: `{% static '<caminho relativo com extensão>' %}`
- também precisamos usar código Python para indicar os _links/URLs_: `{% url '<nome da rota>' %}`

[↑ voltar ao topo](#django-receitas)

### 4.2 Criando uma nova _view_

Vamos adicionar uma nova _view_ ao nosso _app_ e uma rota para acessá-la. No arquivo `apps/receitas/urls.py` adicionamos mais um `path()` na lista `urlpatterns` definindo na ordem: o recurso da rota como `receita`, o método em `app/receitas/views.py` que vai retornar a página HTML renderizada e o nome da rota. Além disso, obviamente temos que escrever o método `receita()` em `app/receitas/views.py` retornando essa _view_.

Para que seja possível acessar essa nova _view_ precisamos ainda "embedar" os _links_ em código Python .

### 4.3 Extendendo código HTML

Pelos dois arquivos HTML que temos, podemos supor que a estrutura básica das páginas da nossa aplicação vai ser a igual para todas, compartilhando elementos como _header_ e _footer_. Então vamos separar todo esse código comum em arquivos próprios e fazer as _views_ "extenderem" esses arquivos.

Primeiro vamos criar nosso _layout_ base. No arquivo `apps/receitas/templates/base.html` copiamos todo conteúdo do arquivo `apps/receitas/templates/index.html` exceto o conteúdo dentro da _tag_ `<body>`, com exceção da declaração dos _scripts_ (deu pra entender...). Logo após a abertura da _tag_ `<body>` devemos indicar que ali vai "entrar" um bloco de código HTML, e os delimitadores desse bloco são escritos em código Python: `{% block content %} {% endblock %}`.

No arquivo `apps/receitas/templates/index.html` mantemos apenas as linhas que não foram copiadas para o _layout_ básico e a indicação que serão carregados arquivos estáticos (`{% load static %}`). Para tirar proveito do _layout_ base devemos extender este (`{% extends 'base.html' %}`) e "envelopar" seu conteúdo como um bloco usando os delimitadores mencionados no parágrafo anterior .

### 4.4 _Partials_

Podemos "componentizar" ainda mais os elementos do nosso layout usando o resurso de _partials_, pequenos fragmentos de código HTML que podem ser compartilhados com várias _views_.

Começamos criando a pasta `apps/receitas/templates/partials` e dentro dela teremos arquivos para cada um dos _partials_ que vamos implementar, _header_ e _footer_. A partir disso basta copiar os respectivos códigos para cada arquivo, indicando que serão carregados arquivos estáticos. Para incluir os _partials_ usamos código Python `{% include 'partials/<nome do partial>.html' %}`. Note que o _partial_ do _footer_ foi adicionado apenas no _layout_ base, enquanto que o do _header_ precisou ser adicionado nas duas _views_ .

Uma conveção bastante adotada é nomear as _partials_ começando com um _underline_, tornando claro do que se trata o arquivo. Neste projeto o "_layout_ base" também foi nomeado seguindo dessa forma .

### 4.5 Apresentando informações de forma dinâmica

Agora vamos passar a enviar informações para a _view_ a partir do método que renderiza o HTML e acessa-lás. A primeira coisa a ser feita é modificar o método Python que renderiza a _view_, onde podemos passar uma coleção como terceiro parâmetro do método `render()`. E então na _view_ podemos usar código Python para iterar sobre essa coleção e acessar seus valores usando a notação `{{ variavel }}` .

[↑ voltar ao topo](#django-receitas)

## 5 Banco de Dados

Nesse ponto vamos configurar o Banco de dados da aplicação, que vai usar o PostgreSQL. Caso você não tenha familiaridade com esse SGDB ou com todos, após realizar o [download](https://www.postgresql.org/download/) e instalação, pode consultar este [apêndice](#criando-uma-base-de-dados-no-postgresql) para criar um usuário e a base de dados.

Para que a nossa aplicação consiga se conectar a um banco de dados PostgreSQL, devemos instalar um módulo Python que será responsável por intermediar essa conexão. Dentro do ambiente virtual rode o comando:

```terminal
pip install psycopg2-binary
```

E para finalizar a configuração do Banco de Dados, devemos colocar no arquivo `djangoreceitas/settings.py` as informações necessárias para que a aplicação saiba como se conectar ao Banco. Por padrão o SQLite já vem configurado, então basta trocar/adicionar as informações necessárias na váriavel `DATABASES` .

[↑ voltar ao topo](#django-receitas)

## 6 _Models_ e _migrations_

### 6.1 _Models_

Agora que já temos um banco de dados configurado, Vamos começar a utilizar o recurso das _models_, que é uma abstração para [modelgem](https://docs.djangoproject.com/en/3.1/topics/db/models/) e [consulta](https://docs.djangoproject.com/en/3.1/topics/db/queries/) dos dados em banco.

Um _model_ é uma classe Python que representa uma entidade do sistema, contendo seus atributos e comportamentos essenciais. Cada _model_ representa uma tabela no Banco de Dados, com seus atributos representando os campos da tabela e todas as _models_ no Django devem extender a classe `django.db.models.Model`.

Dentro do arquivo `apps/receitas/models.py` vamos definir a _model_ `Receita` e dentro dela definimos seus atributos. A partir da biblioteca `models` podemos definir o tipo de dados para cada campo e também as _constraints_ para esses campos, como o limite de caracteres, valor padrão ou se o campo aceita valor nulo .

Podemos definir a representação textual de um objeto utilizando o método especial `__str__` (_dunder methods_) e alterar a forma como são acessados nas _views_ .

### 6.2 _Migrations_

Para fazer o mapeamento des classes do tipo _model_ para tabelas no banco de dados usamos o recurso de _migrations_. Com o comando abaixo criamos uma _migration_ a partir de alterações nas _models_ que ainda não estão mapeadas no Banco :

```terminal
python manage.py makemigrations
```

Após rodar esse comando, por exemplo, a _migration criada agora será responsável apenas por criar a tabela de nome `receitas_receita` (_model_ receita no app receitas) com os campos correspondentes aos atributos que a model possui nesse momento.

E para rodar as _migrations_ disponíveis:

```terminal
python manage.py migrate
```

[↑ voltar ao topo](#django-receitas)

## 7 Django Admin

O Django Admin é um dos recursos mais poderosos do _framework_ segundo a própia [documentação](https://docs.djangoproject.com/en/3.1/ref/contrib/admin/). É uma interface que permite a usuários com privilégios (_super users_) gerenciar os registros de entidades registradas para tal.

Em `djangoreceitas/urls.py` já existe uma rota configurada para o painel do **admin**, basta acessar `localhost:8000/admin`. Mas antes de logar é necessário criar um **_super user_**, para isso temos o comando:

```terminal
python manage.py createsuperuser
```

basta digitar as credências e dependendo da senha que você colocar o Django vai fornecer algumas orientações para criar uma senha mais segura.

Após logar no painel do **admin** percebemos que não há nada relacionado ao app `receitas`, para que seja disponibilizado o CRUD dessa entidade é necessário registrar o _model_ em `app/receitas/admin.py` . Ao recarregar a página vemos que existe uma seção dedicada aos registros do app `receitas`.

### 7.1 Customizando apresentação de dados

Podemos customizar a forma como as receitas são apresentadas no Django Admin. Por exemplo, apresentando mais atributos e tornando alguns links para suas receitas. Também é podemos habilitar algumas funcionalidades como filtros, buscas e paginação. Para isso criamos uma classe em `apps/receitas/admin.py` extendendo `admin.ModelAdmin` e definimos as alterações que quisermos , não esquecendo de também registrar essa classe.

É possível ainda editar alguns campos dos registros diretamente na página de listagem, para isso definimos a variável `list_editable` em `apps/nome_do_app/admin.py` atribuindo uma tupla ou lista com os campos que queremos permitir a edição .

[↑ voltar ao topo](#django-receitas)

## 8 Recuperando dados do Banco

### 8.1 Listando as receitas

Agora que temos dados de receitas armazenados no Banco de Dados podemos apresentá-los nas _views_. Para isso devemos alterar o dicionário passado por contexto para a _view_.  No método `index()` em `apps/receitas/views.py`, ao invés de passar um dicionário com valores _hard coded_, vamos passar todos os objetos do tipo receita que foram cadastrados. Para isso vamos importar o _model_ `Receita` e usar o método `objects.all()` para recuperar todos os registros da tabela.

Devemos alterar a forma como é feito o acesso ao dicionário que a _view_ recebe, para que as receitas cadastradas através do Django Admin sejam apresentadas na _home_. Agora estamos enviando objetos, então podemos acessar seus atributos, e além disso, é uma boa prática verificar se a coleção de objetos não está vazia antes de iterar sobre ela .

Podemos utilizar filtros para selecionar os registros que serão recuperados do banco e enviados para a _view_, assim como podemos ordenar os registros recuperados em função de um campo de forma ascendente ou descendente .

### 8.2 Detalhes de uma receita

Neste ponto, se clicamos em uma das receitas apresentadas na _home_ somos redirecionados para a _view_ "detalhes de uma receita", mas é a página genérica sem as informações da receita que queremos acessar. Então precisamos indicar qual é a receita correta.

Para indicar que queremos acessar a página com as informações de uma receita específica devemos realizar algumas alterações:

- em `apps/receitas/templates/index.html` é necessário passar o identificador único da receita (atributo `id`) como parâmetro na URL: `<a href="{% url 'receita' valor_do_identificador %}">`
- na definição das rotas em `apps/receitas/urls.py`, devemos especificar que o recurso acessado será esse parâmetro que foi enviado pela URL: `'<int:receita_id>'`

Agora que temos acesso ao identificador único, que nesse caso é a chave primária da tabela de receitas, podemos receber esse parâmetro no método `receita()` em `apps/receitas/views.py`, recuperar o registro a partir da chave primária e passar o objeto para a _view_ .

Por fim basta apresentar os atributos do objeto na _view_ receita .

[↑ voltar ao topo](#django-receitas)

## 9 Integrando apps

As receitas são cadastradas por pessoas, então vamos criar um novo app para gerenciar as **pessoas** da aplicação e depois integrá-lo com o app de receitas.

### 9.1 Criando um novo app

Todo o procedimento é o mesmo que foi feito para o app de receitas e a entidade Receita. Primeiro criamos o novo app e o registramos na aplicação adicionando-o na váriavel `INSTALLED_APPS` de `djangoreceitas/settings.py`.

Criamos a classe para representar a entidade Pessoa em `apps/pessoas/models.py` e a registramos para ser gerenciada pelo Admin em `apps/pessoas/admin.py`, aproveitando para customizar a página de listagem.

E por fim, geramos as _migrations_ e a executamos .

### 9.2 Relacionamento _Many-to-one_

Vamos definir o relacionamento entre receitas e pessoas, no caso cada pessoa pode cadastrar várias receitas e cada receita pertence a uma única pessoa. Esse tipo de relacionamento é chamado de "um para muitos" e é definido através de uma chave estrangeira, nesse caso será uma chave estrangeira na tabela de receitas apontando para a tabela de pessoas.

Adicionamos um novo campo na _model_ Receita, do tipo _ForeigKey_ e nomeando-o de acordo com a convenção sugerida na [documentação](https://docs.djangoproject.com/en/3.1/topics/db/models/#many-to-one-relationships). Geramos a _migration_ para essa alteração e a executamos .

[↑ voltar ao topo](#django-receitas)

## 10 _Upload_ de arquivos

### 10.1 Imagens

Vamos começar a implementar a funcionalidade de _upload_ de arquivos definindo as configurações necessárias para a aplicação. Em `djangoreceitas/settings.py` vamos criar duas variáveis: `MEDIA_ROOT` vai armazenar o local/diretório onde os arquivos serão armazenados e `MEDIA_URL` é o recurso a partir do qual será possível acessar as imagens pela URL.

Ao criar o novo campo na _model_ Receita definimos o tipo do campo como `ImageField` e passamos o local relativo a `MEDIA_ROOT` em que as imagens devem ser armazenadas.

Antes de gerar e rodar a _migration_ precisamos instalar um pacote necessário para trabalhar com os arquivos de imagens:

```terminal
pip install pillow
```

então podemos atualizar o banco .

Para que seja possível apresentar essas imagens nas _views_, devemos permitir que suas URLs sejam utilizadas pela aplicação e isso é feito indicando o uso das configurações de mídia no arquivo de rotas da aplicação em `djangoreceitas/urls.py`. Após isso podemos modificar as _views_ para apresentar as imagens de cada receita .

[↑ voltar ao topo](#django-receitas)

## 11 Formulários

Se verificarmos o banco de dados da aplicação vamos encontrar uma tabela chamada `auth_user`. Nessa tabela são armazenados todos os usuários do sistema, desde os comuns até os _superusers_, com acesso ao Django Admin. Os super-usuários já podem ser criados através da linha de comando, então vamos criar um app de usuários para permitir que novos usuários se cadastrem no sistema sem precisar do privélio de ser um super-usuário. E para isso vamos precisar de páginas com formulários específicos para essas ações

### 11.1 App de usuários

O procedimento para criar um novo app dentro da pasta `apps` é: primeiro criar a pasta do novo app e então rodar o comando com passando este _path_:

```terminal
mkdir apps/usuarios
python manage.py startapp usuarios ./apps/usuarios
```

após isso ainda é necessário registrar o app nas configurações da aplicação adicionando-o na lista `INSTALLED_APPS` .

### 11.2 Rotas de cadastro e _login_

Vamos começar essa parte tratando das rotas. Criamos o arquivo de rotas (`apps/usuarios/urls.py`) para o app, definimos os recursos acessados para as páginas e registramos essas rotas nas urls da aplicação (`djangoreceitas/urls.py`). Note que podemos definir um prefixo para as rotas do app .

### 11.3 Páginas de cadastro e _login_

Antes de iniciar a implementação da primeira página que é a de cadastro, precisamos lidar com os templates base. O template básico e os _partials_ foram definidos dentro do app de receitas, mas agora que a aplicação está crescendo e temos mais apps que faram uso desses recursos devemos mover essa pasta. O ideal é que os templates fiquem no mesmo nível dos apps e não dentro de um deles, então movemos a pasta `templates` de `apps/receitas` para `apps`. Além disso devemos alterar o local da pasta `templates` nas configurações da aplicação .

Para organizar melhor as _views_ podemos criar pastas para cada um dos apps dentro de `templates`, lembrando de indicar a pasta ao retornar a _view_, começando com as páginas do app de receitas e depois com as paǵinas de usuários .

### 11.4 Requisições de formulários

#### 11.4.1 Enviando o formulário

Agora que já temos uma página para os formulários podemos começar a pensar em como enviar as informações nele preenchidas e como acessá-las na nossa aplicação.

A primeira coisa que faremos é especificar o tipo de requisição em que o formulário será submetido, como se tratam de informações sensíveis (senha) precisamos utilizar o método `POST` para a requisição, definindo isso no atributo `method` da _tag_ `<form>`. Após isso definimos para onde a requisição do formulário será enviada, passando o nome da rota através de código Python. Além disso devemos incluir o _token_ CSRF dentro do formulário para garantir que a requisição está sendo enviada pela nossa aplicação.

```python
<form action="{% url 'nome_da_rota' %}" method="POST">
    {% csrf_token %}
    ...
</form>
```

Ao contrário de outros _frameworks_ em que você define o método de acesso da requisição para cada rota, no Django mapeamos apenas a rota para um método (código que será  executado) e dentro do código devemos verificar o tipo de método (requisição) para decidir o que fazer.

#### 11.4.2 Acessando os campos de um formulário

Agora que estamos conseguindo enviar o formulário com os campos preenchidos, podemos acessar esses campos no método (código) mapeado para a rota definida como `action` do formulário. Dentro desse método temos acesso à requisição através do objeto `request` obtido por injeção de dependência. Como a requisição foi feita usando o método `POST`, dentro do atributo de mesmo nome vamos acessar um dicionário com todos os campos enviados :

```python
request.POST
```

### 11.5 Cadastrando novo usuário

Como estamos usando uma tabela criada pelo Django, podemos importar o model que modela a tabela de usuários e utilizar _built in functions_ para adicionar um novo usuário no sistema (não precisamos nem "encriptar" a senha). Antes disso podemos fazer validações básicas sobre os dados que estamso recebendo do formulário .

### 11.6 Realizando _login_

O processo de autenticação de um usuário começa com o formulário de _login_ e devemos enviar seus campos para o código que irá processar essa ação, esse processo é muito similar ao cadastro de novos usuários. Como a forma padrão de autenticação do Django é feita em função dos valores de `username` e senha, após nos certificarmos que os valores obtidos do formulário são válidos, precisamos obter o `username` do usuário. Então podemos realizar o _login_ caso o usuário exista e redirecioná-lo para sua _dashboard_ .

Agora podemos fazer alguns ajustes no _layout_ de modo que os botões do _header_ façam mais sentido para usuários logados, implementar a funcionalidade de _logout_ e impedir que usuários não autenticados não acessem algumas páginas .

### 11.7 Formulário para receitas

Vamos montar um formulário para os usuários autenticados sejam capazes de cadastrar suas receitas, então precisamos criar a rota e a _view_ com o formulário .

Anteriormente quando a única forma de cadastro de receitas era através do Django Admin, relacionamos as receitas com a classe `Pessoa`. Agora que usuários autenticados podem cadastrar receitas na aplicação, vamos relacionar as receitas com a classe `User`. Para isso basta importar a _model_ de usuário e substituir na declaração do atributo `pessoa`. Para que essa alteração tenha efeito no Banco precisamos gerar uma _migration_ com as alterações e executá-la . Os comandos necessários são:

```terminal
python manage.py makemigrations
python manage.py migrate
```

Agora podemos receber os campos do formulário e criar um novo registro do tipo `Receita`. Com exceção do campo `foto`, todos os campos ficam disponíveis no dicionário do atributo `POST` do objeto `request`. A foto foi enviada através de um _input_ do tipo arquivo, então a acessamos no atributo `FILES`. Para criar um registro do tipo Receita usamos o método `create` passando cada um dos campos recebidos na requisição e então salvamos o objeto criado .

Após realizar o cadastro da nova receita, redirecionamos o usuário para sua _dashboard_ onde serão apresentadas apenas as suas receitas .

## 12 Mensagens de _feedback_

O Django já nos fornece um sistema de [mensagens](https://docs.djangoproject.com/en/3.1/ref/contrib/messages/) de alerta (_feedback messages_) com vários níveis/tipos que podems ser definidos através da variável `MESSAGE_TAGS` em `djangoreceitas/settings.py`. Podemos aproveitar o estilo das caixas de alertas do [Bootstrap](https://getbootstrap.com/docs/4.0/components/alerts/) e definir as _tags_ de mensagens em função dessas classes.

Também precisamos criar um componente (_partial_) que irá conter o HTML da mensagem de alerta e podemos adicioná-lo no _layout_ base . Após isso podemos definir as mensagens com o devido tipo e conteúdo de acordo com a situação .

## 13 Finalizando o CRUD de receitas

### 13.1 Deletando uma receita

Vamos acessar essa funcionalidade através da _dashboard_ do usuário. Então colocamos
Começamos escrevendo um método chamado `delete()` em `apps/receitas/views.py`, que fará a exclusão do registro. Definimos uma rota para esse método em `apps/receitas/urls.py` e adicionamos um botão na _view_ `apps/templates/usuarios/dashboard.html` que permitirá acessar essa funcionalidade .

### 13.2 Editando uma receita

Para essa funcionalidade devemos criar uma _view_ parecida com a `create`, mas devemos trazer os campos preenchidos. O processo não possui nada de novo em relação ao que já foi implementado .

## 14 Refatorando o projeto

### 14.1 Removendo um app

Agora que temos implementado o registro e autenticação de usuários e atribuímos as receitas criadas a um usuário cadastrado, não precisamos mais do app de pessoas.

Podemos começar removendo o app `Pessoas` da lista `INSTALLED_APPS` em `djangoreceitas/settings.py`, em nosso caso vamos ter um problema devido a uma _migration_ que faz uso desse app, lembrando que inicialmente um _model_ desse app estava relacionado com o _model_ `Receita` dos app `Receitas`. Portanto precisamos remover todos os vínculos que o app `Pessoas` tenha com nossa aplicação: em _views_, _migrations_ e no _controller_.

Aqui no nosso projeto podemos resolver isso com apenas algumas alterações em uma das _migrations_ do app `Receitas`. Devemos copiar de `apps/receitas/migrations/0005_auto_20210121_1324.py` o código que relaciona a tabela de usuários com receita e substituir em `apps/receitas/migrations/0002_receita_pessoa.py` onde era feito o relacionamento com pessoas, não esquecendo de trazer os devidos _imports_ caso necessário. Nessa última _migration_ também apagamos a dependência do app `Pessoa` .

Para remover completamente os traços restates do app de pessoas devemos excluir seu diretório, `apps/pessoas` . atualizar o banco e remover a tabela de pessoas temos duas opções: deletar apenas a tabela de pessoas na mão; ou deletar toda a Base de dados, recriá-la e rodar as migrations novamente.

### 14.2 Modularizando o arquivo _views_

De modo a melhorar a organização dos nossos arquivos começamos alterando alguns nomes de rotas para que fiquem mais semânticos .

Outra coisa que podemos fazer é modularizar o código executado quando uma rota é acessada. Até agora os métodos que são mapeados por rotas e retornam as páginas renderizadas ou executam alguma outra ação ficam todos no arquivo `views.py` na raiz de cada app. Vamos criar uma pasta `src` na raiz de cada app e a partir disso melhorar a forma como esse código é organizado.

Começamos movendo o arquivo `apps/receitas/views.py` para a pasta `apps/receitas/src` e o renomeamos para `controller.py`. A idéia é que dentro desse arquivo fiquem apenas as funções acessadas por meio de rotas. Dessa forma vamos manter isolado todo código que precisa processar alguma informação fornecida pelo usuário e retornar uma resposta a partir disso (melhorar essa frase).

Outra coisa que poderíamos fazer é definir um inicializador dentro dessa pasta para que ela seja reconhecida como um pacote e possa ser importada em outras partes do programa. Para isso criamos um arquivo chamado `__init__.py` dentro de `apps/receitas/src` e nele importaríamos todos os arquivos dentro do pacote. Isso permitiria que no arquico de rotas a importação fosse escrita como `from .src import *` ao invés de `from .src.controller import *`. Como minha idéia é que seja possível acessar apenas as funções dentro de `apps/receitas/src/controller.py`, a forma como ficou implementada é mais adequada .

Ajustes similares e alterações na conveção de como nomear as rotas foram feitas no app de usuários .

## 15 Paginação

Conforme a aplicação for crescendo e mais receitas forem sendo cadastradas, o recurso de paginação se tornará indispensável para que o usuário tenha uma boa navegação nas páginas de listagem. Então vamos implementar essa funcionalidade.

Começamos pelo método `index` do _controller_ de receitas, importando alguns componentes do pacote Paginator do Django, definimos a quantidade de itens por página, recuperamos da URL o parâmetro indicando a página e retornamos a coleção da página selecionada. Em seguida implementamos o componente de paginação para a _view_ .

## Apêndices

### Criando uma base de dados no PostgreSQL

Considerando que você já tem o PostgreSQL instalado em sua máquina, abaixo é apresentado um "passo a passo" para criar um usuário e uma base de dados para a aplicação. Os passos listados aqui foram executados na linha de comando e em ambiente Linux, então é possível que seja um pouco diferente para outras plataformas, mas a ideia geral permanece.

Primeiro vamos nos conectar ao servidor `postgres` com seu superusuário padrão, que é `postgres`:

```terminal
usuario@pc:~$ sudo -i -u postgres
postgres@pc:~$
```

Feito isso podemos nos conectar ao banco de dados:

```terminal
postgres@pc:~$ psql
psql (12.4 (Ubuntu 12.4-0ubuntu0.20.04.1))
Type "help" for help.

postgres=#
```

Agora vamos criar um usuário chamado `dev` com privilégios de _superuser_ e uma senha forte:

```terminal
postgres=# CREATE USER dev
postgres-# WITH SUPERUSER CREATEDB CREATEROLE
postgres-# PASSWORD '123456';
CREATE ROLE
postgres=#
```

Para se desconectar do _prompt_ do banco de dados digite `\q` e depois `exit` para retornar ao terminal do Linux:

```terminal
postgres=# \q
postgres@pc:~$ exit
sair
usuario@pc:~$
```

Agora que temos um usuário com privilégios para acessar a base de dados da nossa aplicação, vamos criá-la usando esse novo usuário. Começamos logando no servidor `postgres` no _localhost_ com o usuário chamado `dev` criado a pouco:

```terminal
usuario@pc:~$ psql -U dev -h 127.0.0.1 postgres
Password for user dev:
psql (12.4 (Ubuntu 12.4-0ubuntu0.20.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=#
```

Criamos uma base de dados chamada `django_receitas` definindo o usuário `dev` como _owner_:

```terminal
postgres=# CREATE DATABASE django_receitas WITH OWNER dev;
CREATE DATABASE
postgres=#
```

E para se conectar a essa base usamos o comando `\connect` passando seu nome:

```terminal
postgres=# \connect django_receitas;
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "django_receitas" as user "dev".
django_receitas=#
```

Para sair basta executar os mesmo comandos mostrados acima. [Voltar à configuração do banco.](#5-banco-de-dados).

[↑ voltar ao topo](#django-receitas)
