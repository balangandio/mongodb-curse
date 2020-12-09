# Cap 17 - From Shell to Driver

# 002 Splitting Work Between the Driver & the Shell
--> Apesar de conseguimos executar os mesmos comandos a partir do cliente de linha de comando ou apartir 
de drivers de conexão, usualmente cada um é utilizado para proprósitos distintos:
 * Shell - configuração de databases, criação de indexes e coleções;
 * Drivers - utilização de databases em operações CRUD e consultas por agregações.

 # 005 Installing the Node.js Driver
 --> O driver de conexão em um projeto Node.js consiste na instalação do pacote NPM  `mongodb`:
 ```
 npm install --save mongodb
 ```

 # 006 Connecting Node.js & the MongoDB Cluster
 --> Exemplo de estabelecimento de conexão com o servidor:
 ```javascript
 const mongodb = require('mongodb');

const MongoClient = mongodb.MongoClient;

const client = await MongoClient.connect('mongodb://username:userpass@localhost/mydb?retryWrites=true');

const db = client.db();
// do something

client.close();
 ```
 * Conexão com o usuário `username` senha `userpass` no servidor `localhost` database `mydb`;
 * A variável `db` se recebe um objeto que se comporta como a notação `db.` dos comandos do MongoDB;
 * `client.close()` encerra a conexão do cliente.

 # 007 Storing Products in the Database
 --> Quase todos os comandos do cliente de linha de comando possuem uma síntaxe igual ou muito parecida com 
 os comandos utilizados com o driver Node.js. A inserção de um objeto em uma coleção seria:
```javascript
const db = getClient().db();

const newProduct = {
    name: 'human fear',
    price: 42
};

const result = await db.collection('products').insertOne(newProduct);
```

# 008 Storing the Price as 128bit Decimal
--> Decimais de alta precisão são representados pelo construtor `Decimal128`:
```javascript
const Decimal128 = require('mongodb').Decimal128;

const db = getClient().db();

const newProduct = {
    name: 'human fear',
    price: Decimal128.fromString('42.1')
};

const result = await db.collection('products').insertOne(newProduct);
```

# 009 Fetching Data From the Database
--> A obterção dos documentos em uma coleção:
```javascript
const db = getClient().db();

const products = await db.collection('products').find().toArray();
```

# 010 Creating a More Realistic Setup
--> O driver de conexão do Node.js já trabalha com connection pooling. Assim, a conexão inicial com o servidor 
e instância do cliente só precisa ser feita uma vez, na inicialização da aplicação.

# 011 Getting a Single Product
--> A instância de objetos id do MongoDB é feita com o construtor `ObjectId`:
```javascript
const ObjectId = require('mongodb').ObjectId;
const db = getClient().db();

const product = await db.collection('products').findOne({ _id: new ObjectId('...')});
```

# 012 Editing & Deleting Products
--> A atualização de um documento pode ser feita com:
```javascript
const ObjectId = require('mongodb').ObjectId;
const db = getClient().db();

const newProduct = { name: 'changed name' };

const result = await db.collection('products').updateOne({ _id: new ObjectId('...')}, {
    $set: { ...newProduct }
});
```

--> A deleção de um documento pode ser feita com:
```javascript
const ObjectId = require('mongodb').ObjectId;
const db = getClient().db();

const newProduct = { name: 'changed name' };

const result = await db.collection('products').deleteOne({ _id: new ObjectId('...')});
```

# 013 Implementing Pagination
--> Uma estratégia de paginação pode ser feita com uso de `skipt()` e `limit()`:
```javascript
const db = getClient().db();

const page = 1;
const pageSize = 10;

const products = await db.collection('products').find()
    .skip((page - 1) * pageSize)
    .limit(pageSize)
    .toArray();
```