# Cap 1 - Introduction

# 002 What is MongoDB
--> MongoDB é um banco de dados não relacional, mantido pela empresa de mesmo nome, cuja principal proposta 
é ser capaz de operar grandes grantidade de dados de maneira performática.

--> É possível definir diversas `bases de dados` dentro de um servidor MongoDB, que possua vez podem conter 
diversas `coleções`, que por sua vez podem ter um grande volume de `documentos`.
* Dentro de uma coleção os documento, ou registros, podem ter diferentes propriedades ou mesmo podem conter 
outros documentos.
* Não há definição rígida de schema ou de relacionamentos.

# 004 The Key MongoDB Characteristics (and how they differ from SQL Databases)
--> Sem a definição estrita de schema e relacionamentos como em bancos de dados SQL, os dados são armazenados 
da maneira mais conveniente à necessidade de acesso. Assim a recuperação pode ser feita de maneira direta e 
eficiênte sem o trabalho de calcular cruzamentos entre diferentes dados.

# 005 Understanding the MongoDB Ecosystem
--> Existem difersas ferramentas em volta do banco de dados MongoDB:
* Servidores Self-Managed e Enterprise;
* Plataforma na núvem Atlas;
* Ferramentas de administração CloudManager e OpsManager;
* Cliente mobile;
* Interface gráfica Compass;
* Ferramentas para análise BI Connectors e MongoDB Charts.

--> A parte, Stitch é um servidor oferecido pela MongoDB que oferece ainda mais funcionalidades à plataforma:
* Serverless Query API;
* Serverless Functions;
* Database Triggers;
* Real-Time Sync.

# 006 Installing MongoDB
--> Uma vez instalado, junto ao banco de dados MongoDB é oferecido um cliente de linha chamado `mongo`:
* Para acessar a instância local da máquina: `mongo`
* Para acessar uma instância de outro host: `mongo --host <hostAddr> -u <db_user> -p <db_pass>`

--> MongoDB na plataforma Docker:
* Docker compose file:
```
version: '3.1'

services:
  mongo:
    image: mongo
    restart: always
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: root
    volumes:
      - mongodb:/data/db

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: root

volumes:
  mongodb:
    external:
      name: vol_mongodb
```

* Criar volume:
```shell
mkdir /opt/mongodb
mkdir data
docker volume create --name vol_mongodb --opt type=none --opt device=/opt/mongodb/data --opt o=bind
```

* Iniciar instância:
```
docker-compose -f docker-compose.yml -d up
```

* Criar database e usuário:
```
docker exec -it mongodb_mongo_1 mongo --host localhost -u root -p root
```
```javascript
use node-complete
db.deleteme.insertOne( { x: 1 } )
db.createUser(
  {
    user: "node-curse-app",
    pwd: "node-curse-pass",
    roles: [
       { role: "readWrite", db: "node-complete" }
    ]
  }
)
```

# 008 Time To Get Started!
--> `show dbs` lista as bases de dados no servidor e seus respectivos tamanhos.
* Por padrão o servidor possui as bases de dados admin, config e local, para propósitos internos.

--> `use shop` altera o contexto para uma base de dados nova, ou já existente, de nome shop.
* Bases de dados e estruturas no MongoDB são criadas na medidade que são utilizadas.

--> `db.products.insertOne({ name: "Human eye", price: 42.35 })` insere um documento em uma coleção chamada 
products.
* A função insertOne recebe um objeto JSON, todavia MongoDB permite que o nome das propriedades não sejam 
envolvidas com `"`: `{ "name": "Human eye" }`
* Por padrão MongoDB atribui um ID randômico aos documentos, qual é retornado quando inserido. 

--> `db.products.find()` recupera os documentos da coleção.
* `db.products.find().pretty()` formata a saída visual dos documentos.

# 009 Shell vs Drivers
--> A síntaxe de comandos do MongoDB encontrado no cliente de linha de comando é bastante similar a síntaxe 
de utilização dos drivers de conexão encontrados em diversas linguagens.

# 010 MongoDB + Clients The Big Picture
--> Os comandos introduzidos por clientes e drivers são recebidos pelo servidor do MongoDB. Internamente o 
servidor encaminha os comandos de acesso aos dados a uma `storege engine`. A Engine é quem de fato administra 
os dados no sistema de arquivos assim como faz uso da memória para otimizar a sua performance.