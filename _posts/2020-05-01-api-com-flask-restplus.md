---
layout: post
title: API com Flask RESTPlus
date: 2020-05-06 20:49:05 +0300
description: API com Flask RESTPlus
img: api-com-restplus.jpg
img-credits: 
fig-caption: 
tags: [Flask, Python, Rest, Web]
---

Neste post vou mostrar uma forma de construir uma API em Python utilizando a biblioteca [Flask RESTPlus](https://flask-restplus.readthedocs.io/en/stable/){:target="_blank"}, uma extenção do framework [Flask](https://flask.palletsprojects.com/en/1.1.x/){:target="_blank"} para construção de APIs [REST](https://pt.wikipedia.org/wiki/REST){:target="_blank"}. 

Essa extensão já facilitou muito minha vida, me possibilitando entregar APIs bem documentadas em um curto espaço de tempo. A extensão Flask RESTPlus fornece algumas abstrações que facilitam o processo de documentação dos endpoints além de funcionar muito bem com o conceito de "blueprints" do Flask. Isso traz facilidade ao manter a documentação atualizada após várias mudanças no código da API.


### Objetivo

O objetivo final é destacar as seguintes característica no caso de uso que vou implementar:

- Criação automática da documentação seguindo a estrutura do [Swagger](https://swagger.io/){:target="_blank"}.
- Separação de *namespaces* na mesma API utilizando o conceito de *Blueprint*.


### Importante
Neste post vou me concentrar apenas no funcionamento da extensão Flask RESTPlus. Para melhor aproveitamento do conteúdo é ideal você tenha um conhecimento básico do framework Flask, pelo menos ter lido a documentação introdutória ja é suficiente. No final do *post*, sugiro que pesquise um pouco de cada um dos itens abaixo que serão utilizados na contrução da API em questão.

- Flask BluePrints
- Swagger e Especificação OpenAPI
- SQLAlchemy

Para o banco de dados vamos usar o SQLite, devido à sua simplicidade, em vez de usar um servidor de banco de dados.

### Caso de uso

Uma API que gerencia produtores e episódios de *podcasts* com endpoints básicos. Os dados seráo gravados em um arquivo SQLite que representrá o banco de dados.


### Itens não abordados (melhorias)
- Autenticação dos endpoints
- Deploy em algum serviço de computação em nuvem.


### Código
O código ficará disponível neste [repositório](https://github.com/evertoncastro/flask-restplus-post/tree/post1){:target="_blank"}.

### Dependências
- Python 3.7
- Flask
- Flask-restplus
- SQLAlchemy
- FlaskMigrate

### Estrutura da API

-  **API**
    - Podcasts

-  **Rotas**
    - Cadastrar produtor
    - Pegar produtor
    - Listar produtores
    - Remover produtor
    - Cadastrar episódio
    - Pegar episódio
    - Listar episódios
    - Remover episódio


### Estrutura do código
{% highlight txt%}

.
├── README.md                        # Instruções sobre o projeto
├── app                              # Diretório com o aplicativo Flask
│   ├── __init__.py                  # Arquivo onde é realizada a criaçao do Flask App
│   ├── api                          # Diretório base para os arquivos da API
│   │   ├── __init__.py
│   │   └── podcast                  # Diretório da API de podcasts
│   │       ├── __init__.py          # Arquivo com a criação do objeto API Flask RESTPlus
│   │       └── namespaces           # Diretório base para os arquivos de namespaces
│   │           ├── __init__.py
│   │           ├── episode.py       # Namespace para episódios
│   │           └── producer.py      # Namespace para produtores
│   ├── config.py                    # Arquivo de configuração do Flask
│   ├── main.py                      # Arquivo que inicia a aplicação Flask
│   ├── models.py                    # Classes que mapeiam as tebelas de banco de dados - Modelos
│   └── requirements.txt             # Declaração de dependências/bibliotecas
└── tests                            # Diretório de testes unitários e integração
    ├── __init__.py
    ├── runner.py                    # Ponto inicial da suite de testes
    ├── test_models.py               # Testes dos Modelos
    ├── test_podcast_episode_ns.py   # Testes dos endpoints do namespace de episódios
    └── test_podcast_producer_ns.py  # Testes dos endpoints do namespace de produtores

{% endhighlight %}

### Arquivos e módulos

O arquivo *./app/requirements.txt* abaixo não demanda muita explicação, apenas o fato de conter as bibliotecas que a aplicação vai utilizar.
{% highlight txt %}
Flask==1.1.2
flask-restplus==0.13.0
Werkzeug==0.16.1
SQLAlchemy==1.3.12
Flask-Migrate==2.5.2
{% endhighlight %}

No arquivo  *./app/\__init__\.py* abaixo foi realizada a criação do Aplicativo Flask utilizando as configurações do arquivo config.py como vemos na linha 14. Neste mesmo arquivo foram feitas as inicializações do banco de dados e migrações. Um ponto importante é evitar que esse arquivo faça importação de outros módulos cridos no projeto. Fiz isso para evitar dependência circular.

{% highlight python linenos %}
from os import getenv
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate


def setup_app() -> object:
    """
    Creates a new Flask application
    :rtype: Flask object
    """
    _app = Flask(__name__)
    _app.config.from_object(
        f"config.{getenv('FLASK_ENV', 'development')}"
    )
    return _app


def setup_database(_app: object) -> SQLAlchemy:
    """
    Adiciona um novo banco de dados ao aplicativo Flask
    :param _app: Aplicativo Flask
    :return: Novo banco de dados criado
    """
    _db = SQLAlchemy()
    _db.init_app(_app)
    return _db


def setup_database_migration(_app: object, _db: SQLAlchemy) -> Migrate:
    """
    Cria a migração de estrutura do banco de dados
    :param _app: Aplicativo Flask
    :param _db: Banco de dados
    :return: O objeto de migração
    """
    return Migrate(_app, _db)


app = setup_app()
db = setup_database(app)
migrate = setup_database_migration(app, db)

{% endhighlight %}

O arquivo *./app/main.py* abaixo é a entrada da aplicação. Aqui importamos o **app** para ser inicializado na linha 8. Apenas com isso o aplicativo rodaria na porta 5000 se executássemos o comando `flask run`, mas não haveria nenhuma rota disponível. Aproveitando a inicialização do aplicativo carreguei a API de podcasts na linha 5.

{% highlight python linenos %}
from app import app
from api.podcast import load_api as load_podcast_api

# Aplicativo Flask fazendo o carregamento da API
load_podcast_api(app)

if __name__ == "__main__":
    app.run(debug=app.config['DEBUG'])
{% endhighlight %}


O arquivo *./app/config.py* contém as configurações do aplicativo Flask. Nele criei uma configuração para desenvolvimento e testes. Fiz isso apenas para ter um banco de dados para cada ambiente. 

{% highlight python linenos %}
import os
basedir = os.path.abspath(os.path.dirname(__file__))

class Config(object):
    Debug = True
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    RESTPLUS_MASK_HEADER = False


class DevelopmentConfig(Config):
    SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, 'app.db')


class TestingConfig(Config):
    SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, 'test.db')


development = DevelopmentConfig()
testing = TestingConfig()
{% endhighlight %}


No arquivo *.app/api/podcast/\__init__\.py* criei a API de podcasts utilizando a biblioteca Flask RESTPlus. É importante entender cada etapa deste arquivo conforme explicado abaixo: 
- Linha 6, criação do objeto *api*.
- Na linha 7 vale destacar que passamos uma BluePrint do Flask para a API Flask RESTPlus.
- Linha 14 e 15, registro dos *namespaces* de *produtor* e *episódios* no objeto api.
- Na linha 18 criei o método que pode ser utilizado externamente para registrar a API como uma BluePrint. Ver .app/main.py. 

Enquanto não registrar os namespaces, como fiz nas linhas 14 e 15, a API terá somente o seu caminho base "/podcast_api/v1.0" mas não haverá nenhum endpoint disponível. Essa estrutura nos permite remover e adicionar *namespaces* de forma fácil e aproveitarmos a modularidade da extenção Flask RESTPlus. Na documentação você verá exemplos mais básicos. Decidi não ficar no básico para mostrar um cenário que esta mais próximo de um caso real.

{% highlight python linenos %}
from flask import Blueprint
from flask_restplus import Api as ApiRestPlus
from api.podcast.namespaces import producer
from api.podcast.namespaces import episode

api = ApiRestPlus(
    Blueprint('API de PodCasts', __name__),
    title='API para gestão de podcasts',
    version='1.0',
    description='Endpoints para gestão de produtores e episódios de podcasts'
)

# Atrela o namespace à API de podcast
producer.bind_with_api(api)
episode.bind_with_api(api)


def load_api(app) -> object:
    """
    Este método serve para o aplicativo Flask carregar a API em si
    :param app: Aplicativo Flask
    :return: Vazio
    """
    app.register_blueprint(api.blueprint, url_prefix='/podcast_api/v1.0')
    return None
{% endhighlight %}

No arquivo *./app/api/podcast/namespaces/producer.py* criei 4 endpoints relacionados às operações de criar, pegar, listar e remover *produtor(es)*. Cada endpoint é uma classe filha da classe "Resource" fo Flask RESTPlus. O método da classe representa o método HTTP do endpoint. Aqui fiz um método por classe mas é possível ter um get e um post na mesma classe, por exemplo. 

Os compartamentos e configurações dos endpoints são aplicados por meio de decoradores na classe e métodos, é aqui que esta a beleza dessa extensão. Automaticamente os comportamentos e configurações são aplicados na documentação que pode ser acessada na rota base da API. Para mais detalhes sobre cada decorador veja os comentários na classe "CreateProducer".

{% highlight python linenos %}
from app import db
from flask_restplus import Api
from flask_restplus import Namespace, Resource, fields
from models import Producer
from werkzeug.exceptions import HTTPException
from werkzeug.exceptions import NotFound
from werkzeug.exceptions import InternalServerError

namespace = Namespace('produtor', description='Produtor')

# É possível criar modelos que podem ser usados nas configurações dos endpoints
# para serem recebidos como parâmetros
create_producer_request = namespace.model('Dados para criação de produtor', {
    'name': fields.String(required=True, description='Nome do produtor')
})

# É possível criar modelos que podem ser usados nas respostas dos endpoints
create_producer_response = namespace.model('Resposta da criaçao de produtor', {
    'id': fields.Integer(required=True, description='Identificador único do produtor')
})

get_producer_response = namespace.model('Resposta pegar produtor', {
    'id': fields.Integer(required=True, description='Identificador único do produtor'),
    'name': fields.String(required=True, description='Nome do produtor')
})

list_producers = namespace.model('Lista de produtores', {
    'id': fields.Integer(required=True, description='Identificador único do produtor'),
    'name': fields.String(required=True, description='Nome do produtor')
})

list_producers_response = namespace.model('Resposta da lista de produtores', {
    'list': fields.Nested(list_producers, required=True, description='Lista de produtores')
})

delete_producer_response = namespace.model('Resposta da remocao de produtores', {
    'removed': fields.Boolean(required=True, description='Indicador de remocao com sucesso')
})

headers = namespace.parser()
# Aqui podemos adicionar mais parametros ao headers


# O decorador .route define o caminho do endpoint dentro da API
@namespace.route('/cria', doc={"description": 'Cria um novo produtor'}) 
# o decorador .expect declara as configurações, obrigatórias ou não, que devem ser enviadas
@namespace.expect(headers)
class CreateProducer(Resource):
    # O .response deixa explícito na documentação as possíveis respostas
    @namespace.response(200, 'Success')
    @namespace.response(400, 'Request Error')
    @namespace.response(500, 'Server Error')
    # O .expect declara os parâmetros, obrigatórios ou não, que o endpoit espera
    @namespace.expect(create_producer_request, validate=True)
    # o .marshal_with declara a estrutura da resposta com base no model recebido como parâmetro
    @namespace.marshal_with(create_producer_response)
    def post(self):
        """Cria novo produtor"""
        session = db.session
        try:
            producer = Producer().create(session, name=namespace.payload['name'])
            session.commit()
            return {'id': producer.id}
        except Exception as e:
            raise InternalServerError(e.args[0])
        finally:
            session.close()


@namespace.route('/<int:id>', doc={"description": 'Pega produtor'})
@namespace.param('id', 'Identificador único do produtor')
@namespace.expect(headers)
class GetProducer(Resource):
    @namespace.response(200, 'Success')
    @namespace.response(404, 'Not Found Error')
    @namespace.response(500, 'Server Error')
    @namespace.marshal_with(get_producer_response)
    def get(self, id):
        """Pega produtor"""
        session = db.session
        try:
            producer = Producer().fetch(session, id)
            if not producer:
                raise NotFound('Not found producer')
            return producer
        except HTTPException as e:
            raise e
        except Exception as e:
            raise InternalServerError(e.args[0])
        finally:
            session.close()


@namespace.route('/todos', doc={"description": 'Lista todos os produtor'})
@namespace.expect(headers)
class ListProducers(Resource):
    @namespace.response(200, 'Success')
    @namespace.response(404, 'Not Found Error')
    @namespace.response(500, 'Server Error')
    @namespace.marshal_with(list_producers_response)
    def get(self):
        """Lista todos os produtores"""
        session = db.session
        try:
            producers = Producer().fetch_all(session)
            return {'list': producers}
        except HTTPException as e:
            raise e
        except Exception as e:
            raise InternalServerError(e.args[0])
        finally:
            session.close()


@namespace.route('/remove/<int:id>', doc={"description": 'Apaga produtor'})
@namespace.param('id', 'Identificador único do produtor')
@namespace.expect(headers)
class DeleteProducers(Resource):
    @namespace.response(200, 'Success')
    @namespace.response(500, 'Server Error')
    @namespace.marshal_with(delete_producer_response)
    def delete(self, id):
        """Remove produtor"""
        session = db.session
        try:
            removed = Producer().delete(session, id)
            session.commit()
            return {'removed': removed}
        except Exception as e:
            raise InternalServerError(e.args[0])
        finally:
            session.close()


def bind_with_api(api: Api):
    """
    Adiciona o namespace à API recebida
    :param api: Flask Restplus API
    :return: Vazio
    """
    api.add_namespace(namespace)
    return None

{% endhighlight %}
No arquivo *./app/api/podcast/namespaces/episodes.py* criei 4 endpoints relacionados às operações de criar, pegar, listar e remover *episódio(s)*.

{% highlight python linenos %}
from app import db
from flask_restplus import Api
from flask_restplus import Namespace, Resource, fields
from models import Episode
from werkzeug.exceptions import HTTPException
from werkzeug.exceptions import NotFound
from werkzeug.exceptions import InternalServerError

namespace = Namespace('episodio', description='Episódio')

create_episode_request = namespace.model('Dados para criação de episódio', {
    'producer_id': fields.Integer(required=True, description='Identificador do produtor'),
    'name': fields.String(required=True, description='Nome do episódio'),
    'url': fields.String(required=True, description='Url do episódio')
})

create_episode_response = namespace.model('Resposta da criaçao de episódio', {
    'id': fields.Integer(required=True, description='Identificador único do episódio')
})

get_episode_response = namespace.model('Resposta pegar episódio', {
    'id': fields.Integer(required=True, description='Identificador único do episódio'),
    'name': fields.String(required=True, description='Nome do episódio'),
    'url': fields.String(required=True, description='Url do episódio')
})

list_episodes = namespace.model('Lista de episódios', {
    'id': fields.Integer(required=True, description='Identificador único do episódio'),
    'name': fields.String(required=True, description='Nome do episódio'),
    'url': fields.String(required=True, description='Url do episódio')
})

list_episodes_response = namespace.model('Resposta da lista de episódios', {
    'list': fields.Nested(list_episodes, required=True, description='Lista de episódios')
})

delete_episode_response = namespace.model('Resposta da remocao de episódio', {
    'removed': fields.Boolean(required=True, description='Indicador de remocao com sucesso')
})

headers = namespace.parser()
# Aqui podemos adicionar mais parametros ao headers


@namespace.route('/cria', doc={"description": 'Cria um novo episódio'})
@namespace.expect(headers)
class CreateEpisode(Resource):
    @namespace.response(200, 'Success')
    @namespace.response(400, 'Request Error')
    @namespace.response(500, 'Server Error')
    @namespace.expect(create_episode_request, validate=True)
    @namespace.marshal_with(create_episode_response)
    def post(self):
        """Cria novo episódio"""
        session = db.session
        try:
            episode = Episode().create(
                session,
                producer_id=namespace.payload['producer_id'],
                name=namespace.payload['name'],
                url=namespace.payload['url']
            )
            session.commit()
            return {'id': episode.id}
        except Exception as e:
            raise InternalServerError(e.args[0])
        finally:
            session.close()


@namespace.route('/<int:producer_id>/<int:id>', doc={"description": 'Pega episódio'})
@namespace.param('producer_id', 'Identificador único do produtor')
@namespace.param('id', 'Identificador único do episódio')
@namespace.expect(headers)
class GetEpisode(Resource):
    @namespace.response(200, 'Success')
    @namespace.response(404, 'Not Found Error')
    @namespace.response(500, 'Server Error')
    @namespace.marshal_with(get_episode_response)
    def get(self, producer_id, id):
        """Pega episódio"""
        session = db.session
        try:
            producer = Episode().fetch(session, producer_id, id)
            if not producer:
                raise NotFound('Not found producer')
            return producer
        except HTTPException as e:
            raise e
        except Exception as e:
            raise InternalServerError(e.args[0])
        finally:
            session.close()


@namespace.route('/todos', doc={"description": 'Lista todos os episódios'})
@namespace.expect(headers)
class ListEpisodes(Resource):
    @namespace.response(200, 'Success')
    @namespace.response(404, 'Not Found Error')
    @namespace.response(500, 'Server Error')
    @namespace.marshal_with(list_episodes_response)
    def get(self):
        """Lista todos os episódios"""
        session = db.session
        try:
            episodes = Episode().fetch_all(session)
            return {'list': episodes}
        except HTTPException as e:
            raise e
        except Exception as e:
            raise InternalServerError(e.args[0])
        finally:
            session.close()


@namespace.route('/remove/<int:producer_id>/<int:id>',
                 doc={"description": 'Apaga episódio'})
@namespace.param('producer_id', 'Identificador único do produtor')
@namespace.param('id', 'Identificador único do episódio')
@namespace.expect(headers)
class DeleteProducers(Resource):
    @namespace.response(200, 'Success')
    @namespace.response(500, 'Server Error')
    @namespace.marshal_with(delete_episode_response)
    def delete(self, producer_id, id):
        """Remove episódio"""
        session = db.session
        try:
            removed = Episode().delete(session, producer_id, id)
            session.commit()
            return {'removed': removed}
        except Exception as e:
            raise InternalServerError(e.args[0])
        finally:
            session.close()


def bind_with_api(api: Api):
    """
    Adiciona o namespace à API recebida
    :param api: Flask Restplus API
    :return: Vazio
    """
    api.add_namespace(namespace)
    return None
{% endhighlight %}


No arquivo *./tests/runner.py* criei um centralizador dos testes automatizados.

{% highlight python linenos %}
import os
import sys
from unittest.loader import TestLoader
from unittest import TextTestRunner
from app import app
from app import db


def test_suite():
    # A abertura de contexto faz o banco de dados estar pronto para todos os casos de teste
    # Isso nos poupará de ter que fazer a configuração em cada classe de testes
    with app.app_context():
        try:
            os.remove('app/test.db')
        except IOError:
            pass
        db.create_all()
        # O Test Loader vai procurar e executar testes em todos os arquivos que tenham o padrão test_*
        suite = TestLoader().discover(
            'tests',
            pattern='test_*.py',
            top_level_dir=os.environ['PYTHONPATH'].split(os.pathsep)[0]
        )
        return TextTestRunner(verbosity=1).run(suite)


def clear_database(_db):
    # Remove todas as tabelas do banco para o contexto de teste
    db.session.rollback()
    for table in reversed(_db.metadata.sorted_tables):
        _db.session.execute(table.delete())
    _db.session.commit()


if __name__ == '__main__':
    result = test_suite()
    if not result.wasSuccessful():
        sys.exit(1)
{% endhighlight %}


Arquivo *./tests/test_podcast_producer_ns.py* com testes dos endpoints de *produtor*.

{% highlight python linenos %}
from unittest import TestCase
from models import Producer
from models import Episode
from app import db
from tests.runner import clear_database


class TestProducerModel(TestCase):

    def setUp(self):
        self.db = db
        self.session = self.db.session
        self.db.create_all()

    def tearDown(self):
        self.session.close()
        clear_database(self.db)

    def testa_se_cria_produtor_com_sucesso(self):
        novo_produtor = Producer().create(self.session, name='Produtor 1')
        self.session.commit()
        teste_produtor = self.session.query(Producer).filter_by(id=novo_produtor.id).first()
        self.assertIsNotNone(teste_produtor)

    def testa_se_busca_produtor_que_existe_no_banco(self):
        novo_produtor = Producer().create(self.session, name='Produtor 1')
        self.session.commit()
        teste_produtor = Producer().fetch(self.session, novo_produtor.id)
        self.assertIsNotNone(teste_produtor)
        self.assertEqual(teste_produtor.name, 'Produtor 1')

    def testa_se_busca_todos_produtores_no_banco(self):
        Producer().create(self.session, name='Produtor 1')
        Producer().create(self.session, name='Produtor 2')
        Producer().create(self.session, name='Produtor 3')
        self.session.commit()
        produtores = Producer().fetch_all(self.session)
        self.assertEqual(len(produtores), 3)

    def testa_se_remove_um_produtor_do_banco(self):
        novo_produtor = Producer().create(self.session, name='Produtor 1')
        self.session.commit()
        teste_produtor = Producer().fetch(self.session, novo_produtor.id)
        self.assertIsNotNone(teste_produtor)
        apagado = Producer().delete(self.session, teste_produtor.id)
        self.session.commit()
        self.assertTrue(apagado)
        teste_produtor = Producer().fetch(self.session, novo_produtor.id)
        self.assertIsNone(teste_produtor)


class TestEpisodeModel(TestCase):

    def setUp(self):
        self.db = db
        self.session = self.db.session
        self.db.create_all()

    def tearDown(self):
        self.session.close()
        clear_database(self.db)

    def testa_se_cria_episodio_com_sucesso(self):
        novo_episodio = Episode().create(
            self.session,
            producer_id=1,
            name='Episódio 1',
            url='http://produtor/episodio/1'
        )
        self.session.commit()
        teste_episodio = self.session.query(Episode).filter_by(id=novo_episodio.id).first()
        self.assertIsNotNone(teste_episodio)

    def testa_se_busca_episodio_que_existe_no_banco(self):
        novo_episodio = Episode().create(
            self.session,
            producer_id=1,
            name='Episódio 1',
            url='http://produtor/episodio/1'
        )
        self.session.commit()
        teste_episodio = Episode().fetch(self.session, 1, novo_episodio.id)
        self.assertIsNotNone(teste_episodio)

    def testa_se_busca_episodios_de_um_produtor(self):
        Episode().create(
            self.session,
            producer_id=1,
            name='Episódio 1',
            url='http://produtor/episodio/1'
        )
        Episode().create(
            self.session,
            producer_id=1,
            name='Episódio 2',
            url='http://produtor/episodio/2'
        )
        Episode().create(
            self.session,
            producer_id=1,
            name='Episódio 3',
            url='http://produtor/episodio/3'
        )
        self.session.commit()
        episodios = Episode().fetch_all(self.session)
        self.assertEqual(len(episodios), 3)

    def testa_se_remove_episodio_de_um_produtor(self):
        novo_episodio = Episode().create(
            self.session,
            producer_id=1,
            name='Episódio 1',
            url='http://produtor/episodio/1'
        )
        self.session.commit()
        teste_episodio = Episode().fetch(self.session, 1, novo_episodio.id)
        self.assertIsNotNone(teste_episodio)
        apagado = Episode().delete(self.session, 1, teste_episodio.id)
        self.session.commit()
        self.assertTrue(apagado)
        teste_produtor = Episode().fetch(self.session, 1, teste_episodio.id)
        self.assertIsNone(teste_produtor)
{% endhighlight %}


Arquivo *./tests/test_models.py* com testes dos endpoints de *produtor*.

{% highlight python linenos %}
from unittest import TestCase
from main import app
from app import db
from tests.runner import clear_database
from models import Producer
from json import loads


class TestProducerNamespace(TestCase):

    def setUp(self):
        self.app_context = app.test_request_context()
        self.app_context.push()
        self.client = app.test_client()
        self.db = db
        self.session = self.db.session

        clear_database(self.db)

    def tearDown(self):
        clear_database(self.db)

    def testa_se_retorna_400_para_parametros_vazios(self):
        response = self.client.post(
            '/podcast_api/v1.0/produtor/cria',
            json={}
        )
        self.assertEqual(response.status_code, 400)

    def testa_se_retorna_400_para_nome_vazio(self):
        response = self.client.post(
            '/podcast_api/v1.0/produtor/cria',
            json={'name': None}
        )
        self.assertEqual(response.status_code, 400)

    def testa_se_cria_produtor_no_banco_e_retorna_200(self):
        response = self.client.post(
            '/podcast_api/v1.0/produtor/cria',
            json={'name': 'Produtor 1'}
        )
        data = loads(response.get_data())
        producer = Producer().fetch(self.session, data['id'])
        self.assertIsNotNone(producer)
        self.assertEqual(response.status_code, 200)

    def testa_se_retorna_um_produtor_que_existe_no_banco(self):
        Producer().create(self.session, name='Produtor 1')
        self.session.commit()
        response = self.client.get(
            '/podcast_api/v1.0/produtor/1'
        )
        self.assertEqual(response.status_code, 200)
        data = loads(response.get_data())
        self.assertEquals(data, {
            'id': 1,
            'name': 'Produtor 1'
        })

    def testa_se_retorna_404_quando_id_nao_existe(self):
        response = self.client.get(
            '/podcast_api/v1.0/produtor/1'
        )
        self.assertEqual(response.status_code, 404)

    def testa_se_a_lista_de_produtores_que_esta_no_banco(self):
        Producer().create(self.session, name='Produtor 1')
        Producer().create(self.session, name='Produtor 2')
        Producer().create(self.session, name='Produtor 3')
        self.session.commit()
        response = self.client.get(
            '/podcast_api/v1.0/produtor/todos'
        )
        self.assertEqual(response.status_code, 200)
        data = loads(response.get_data())
        self.assertEquals(len(data['list']), 3)

    def testa_se_remove_um_produtor_do_banco_de_dados(self):
        Producer().create(self.session, name='Produtor 1')
        self.session.commit()
        response = self.client.delete(
            '/podcast_api/v1.0/produtor/remove/1'
        )
        self.assertEqual(response.status_code, 200)
{% endhighlight %}


Arquivo *./tests/test_podcast_episode_ns.py* com testes dos endpoints de *episódio*.

{% highlight python linenos %}
from unittest import TestCase
from main import app
from app import db
from tests.runner import clear_database
from models import Episode
from json import loads


class TestEpisodeNamespace(TestCase):

    def setUp(self):
        self.app_context = app.test_request_context()
        self.app_context.push()
        self.client = app.test_client()
        self.db = db
        self.session = self.db.session

        clear_database(self.db)

    def tearDown(self):
        clear_database(self.db)

    def testa_se_retorna_400_para_parametros_vazios(self):
        response = self.client.post(
            '/podcast_api/v1.0/episodio/cria',
            json={}
        )
        self.assertEqual(response.status_code, 400)

    def testa_se_retorna_400_para_nome_vazio(self):
        response = self.client.post(
            '/podcast_api/v1.0/episodio/cria',
            json={'name': None}
        )
        self.assertEqual(response.status_code, 400)

    def testa_se_cria_episodio_no_banco_e_retorna_200(self):
        response = self.client.post(
            '/podcast_api/v1.0/episodio/cria',
            json={
                'producer_id': 1,
                'name': 'Produtor 1',
                'url': 'http://produtor1/episodio1'
            }
        )
        data = loads(response.get_data())
        self.assertEqual(response.status_code, 200)
        producer = Episode().fetch(self.session, 1, data['id'])
        self.assertIsNotNone(producer)

    def testa_se_retorna_um_episodio_que_existe_no_banco(self):
        Episode().create(
            self.session,
            producer_id=1, name='Episódio 1', url='/')
        self.session.commit()
        response = self.client.get(
            '/podcast_api/v1.0/episodio/1/1'
        )
        self.assertEqual(response.status_code, 200)
        data = loads(response.get_data())
        self.assertEquals(data, {
            'id': 1,
            'name': 'Episódio 1',
            'url': '/'
        })

    def testa_se_retorna_404_quando_id_do_episodio_nao_existe(self):
        response = self.client.get(
            '/podcast_api/v1.0/episodio/1/1'
        )
        self.assertEqual(response.status_code, 404)

    def testa_se_retorna_a_lista_de_episodio_que_esta_no_banco(self):
        Episode().create(
            self.session,
            producer_id=1, name='Episódio 1', url='/')
        Episode().create(
            self.session,
            producer_id=2, name='Episódio 1', url='/')
        self.session.commit()
        response = self.client.get(
            '/podcast_api/v1.0/episodio/todos'
        )
        self.assertEqual(response.status_code, 200)
        data = loads(response.get_data())
        self.assertEquals(len(data['list']), 2)

    def testa_se_remove_um_episodio_do_banco_de_dados(self):
        Episode().create(
            self.session,
            producer_id=1, name='Episódio 1', url='/')
        self.session.commit()
        response = self.client.delete(
            '/podcast_api/v1.0/episodio/remove/1/1'
        )
        self.assertEqual(response.status_code, 200)
{% endhighlight %}


### Testando a API

Todos os comando abaixo devem ser executados no diretório raiz do projeto.

#### Preparando ambiente virtual
```
pip install virtualenv
virtualenv venv
source venv/bin/activate
```

#### Instalando bibliotecas
```
pip install -r app/requirements.txt
```

#### Definindo variáveis de ambiente
```
export PYTHONPATH=$PYTHONPATH:$(pwd)/app
export FLASK_ENV=development
export FLASK_APP=app/main.py
```

#### Criação de banco e tabelas (arquivo sqlite, ver .app/config.py)
```
flask db init --directory=development_migrations (esse comando cria o banco de dados)
flask db migrate --directory=development_migrations
flask db upgrade --directory=development_migrations
```

#### Executando os tests
```
python tests/runner.py
```

#### Iniciando a aplicação
```
flask run
```

Se tudo der certo acesse: [http://127.0.0.1:5000/podcast_api/v1.0/](http://127.0.0.1:5000/podcast_api/v1.0/){:target="_blank"}

Nevegue pelos *namespaces* e veja que cada *endpoint* possui detalhes sobre os parâmetros, respostas e protocolos de seu funcionamento.

Exemplo de funcionamento da API.
 ![yay](/assets/gifs/post3.gif)

Bom, a ideia era essa, mostrar a API funcionando, gravando e buscando dados do arquivo SQLite. Espero ter ajudado mostrando este exemplo que pode ser utilizado em uma API para casos reais.

Se tiver dúvidas, comentários ou encontrar alguma falha, sinta-se a vontade e poste nos comentários abaixo.

Obrigado!
