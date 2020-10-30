# Cap 2 - Understanding Databases, Collections & Documents

# 005 Understanding JSON Data
--> Documentos inseridos em coleções recebem em uma propriedade `_id` um valor de um tipo interno, `ObjectId`
, que identifica unicamente o documento na base de dados como um todo, além de ser internamente gerado com o 
timestamp de usa criação e por isso ser ordenável.

# 006 Comparing JSON & BSON
--> Apensar de representarmos documentos em formato JSON na síntaxe dos comandos, internamente MongoDB lida 
com os mesmos no formato BSON. E os drivers utilizados na interação com o servidor realizam o parse dos valores 
binários para objetos na memória. O uso deste formato tem as vantagens:
* É mais performático de se trabalhar e mais eficiênte para ser armazenado;
* Permite o armazenamentos de valores não primitivos, como o próprio tipo `ObjectId` do MongoDB.

--> A propriedade `_id` que novos documentos recebem pode ser especificada e não necessarimente precisa ter um 
valor do tipo `ObjectId`:
`db.products.insertOne({ name: "Human eye", _id: "human-eye-1" })`
* Todavia é necessário garantir que o valor seja único caso contrário é lançado um erro;

# 007 Create, Read, Update, Delete (CRUD) & MongoDB
--> Create: operações mais comuns de inserção:
* `insertOne(data, options)`
* `insertMany(data, options)`

--> Read: operações mais comuns de leitura:
* `find(filter, options)`
* `findOne(filter, options)`

--> Update: operações mais comuns de atualização:
* `updateOne(filter, data, options)`
* `updateMany(filter, data, options)`
* `replaceOne(filter, data, options)`

--> Delete: operações mais comuns de deleção:
* `deleteOne(filter, options)`
* `deleteMany(filter, options)`

# 008 Finding, Inserting, Deleting & Updating Elements
--> O objeto `filter` das operações de leitura, atualização e deleção consiste de um objeto que descreve os valores 
que as propriedades dos documentos devem apresentar:
```javascript
db.products.find({ name: "Human" })
```
* Documentos cuja `name` seja igual a `Human`;
* Quando não se deseja realizar nenhum restrição, basta informar um objeto vazio `{}`;
`db.products.deleteMany({})`
* Deleta todos os documentos da coleção.

--> MongoDB possui operadores que descrevem como uma operação deve ser feita. Seus nomes são iniciados com o caracter 
`$`. O objeto `data` nas operações de atualização deve conter o operador `$set` para descrever os valores que devem 
ser definidos pela operação de atualização:
```javascript
db.products.updateOne({ name: "Human" }, { $set: { name: "Homo Sapiens" } })
```
* Altera `name` de um documento cujo valor da propriedade é `Human`.

# 009 Understanding insertMany()
--> A inserção de vários documentos com `insertMany` é feita informando os mesmos em um array:
```javascript
db.products.insertMany([
  { name: "Human", price: 42 },
  { name: "Alien", price: 5342 }
])
```

# 010 Diving Deeper Into Finding Data
--> As restrições no objeto `filter` não só se limitam a igualdade. O operador `$gt` permite restringir valores 
`greater than` (>) por exemplo:
```javascript
db.products.find({ price: { $gt: 42 } })
```

--> O uso de operações que presupõe a existência de um único documento, como `findOne`, podem lançar um erro quando 
a restrição afeta mais de um.

# 011 update vs updateMany()
--> Valores do tipo `ObjectId` são comparáveis apenas com valores do mesmo tipo. Para busca um documento que possui 
um ID em específico, é necessário construir o objeto:
```javascript
db.products.updateOne({ _id: ObjectId("43bfh3j4hr3jb4hb3j4h") }, { $set: { price: 42 } })
```

--> Assim como `updateMany()`, o comando `update()` se proprõe a alterar um ou mais documentos em uma coleção. Todavia 
`update()` permite que o parâmetro `data` não especifique nenhum operador, como `$set`. E como consequência neste caso, 
seu comportamento passa a ser sobrescrever todas as propriedades do documento com o objeto especificado, exceto `_id`:
```javascript
db.products.update({ _id: ObjectId("43bfh3j4hr3jb4hb3j4h") }, { name: "gaerger", price: 42 } })
```
* Não possuindo um comportamento explícito, o uso de `update()` é em parte desencorajado para o uso em casos separados 
de `updateMany()` e `replaceOne()`

# 012 Understanding find() & the Cursor Object
--> Uma vez que uma coleção pode conter muitos documentos, o comando `find()` retorna um objeto que representa um cursor 
para leitura do resultado de maneira controlada. 
```javascript
db.products.find({}).toArray()
db.products.find({}).forEach(doc => printjson(doc))
```
* `toArray()` consome o cursor retornando um array com os resultados;
* `forEach()` consome o cursor executando uma função em cada elemento;
* `printjson()` é uma função built-in do `mongo` que exibe no console objetos JSON formatadamente;
* No cliente de linha comando `mongo` quando find é usado e a quantidade de documentos retornados excede a tela, o comando 
`it` pode ser acionado para avançar a exibição consumindo o cursor.

# 013 Understanding Projection
--> É possível realizar uma projeção sobre as propriedades dos documentos de maneira que seja recuperado do banco de dados 
somente os dados relevantes para a consulta. O comando `find()` pode receber como segundo parâmetro um objeto que define 
quais propriedades são incluídas/excluídas da projeção:
```javascript
db.products.find({}, { name: 1, _id: 0 })
```
* Propriedades definidas com valor `1` são incluídas, e `0` excluídas das projeção;
* Todas as propriedades não citadas são excluídas da projeção com exceção de `_id`, que só pode ser excluída explicitamente 
com `0`.

# 014 Embedded Documents & Arrays - The Theory
--> Documentos em MongoDB não se limitam a armazenar apenas a tipos primitivos, podem possuir outros objetos e arrays.
```javascript
db.products.insertOne({ name: "fger", price: { currency: "BR", value: 42 } })
db.products.insertOne({ name: "gere", price: 5334, attr: [ "clever", "small", "fast" ] })
```
* Todavia existe um limite de `100` níveis de aninhamento de objetos;
* Como um todo, o documento não pode exceder `16 Megabytes` de tamanho.

# 017 Accessing Structured Data
--> É possível consultar se uma estrutura array possui um determinado elemento:
```javascript
db.products.find({ attr: "small" })
```
* MongoDB identifica comparações com propriedades do tipo array e altera o seu comportamento para verificar se o valor 
informado está presenta na estrutura.

--> É possível consultar propriedades de objetos aninhados navegando entre os níveis separados com o caracter `.`, e 
envolvendo a especificação da consulta com `"`:
```javascript
db.products.find({ "price.currency": "BR" })
```
* Consulta os documentos cuja propriedade `price` é um objeto que possui a propriedades `currency` igual ao valor `BR`.