# Cap 6 - Diving Into Create Operations

# 002 - Creating Documents - An Overview
--> Os comandos de criação de dados no MongoDB são basicamente:
```javascript
db.collection.insertOne({ ... })
db.collection.insertMany([ { ... }, { ... } ])
db.collection.insert( ... )
```
* `insert()` consegue criar um ou mais documentos. Todavia como sua síntaxe não é tão clara como os demais 
comandos, seu uso é depreciado.

--> Ademais, é ainda possível introduzir dados no servidor atravéz de importação com o utilitário `mongoimport`:
```
mongoimport -d cars -c carsList --drop --jsonArray
```

# 003 Understanding insert() Methods
--> O comando `insert()` tem a flexibilidade de poder receber um objeto ou um array de objetos. Mas possui a 
desvantagem em relação aos demais comandos de não retornar os IDs gerados.
```javascript
db.collection.insert({ name: "Human", angle: 54 })
db.collection.insert([ { name: "Dog", angle: 257 }, { name: "Horse", angle: 73 } ])
```

# 004 Working with Ordered Inserts
--> Por padrão, comandos que realizam operações em lote apresentam um comportamento sequencial, uma operação 
unitária é realizada após a anterior de maneira ordenada. Dessa maneira, caso uma operação falhe as anteriores 
seriam sucedidas e as posteriores abortadas. O comando `insertMany()` opera em lote e apresenta este comportamento, 
o que muitas vezes não é desejado. 

--> Para alterar o comportamento de `insertMany()` de maneira que, mesmo após um dos documentos resultar em erro, os 
demais sejam ainda tratados, basta configurar a opção `ordered` para false:
```javascript
db.collection.insertMany([ { ... }, { ... } ], {
    ordered: false
})
```

# 005 Understanding the writeConcern
--> O servidor do MongoDB internamente delega as operações para uma engine, um programa que controla os dados 
fazendo uso da memória e do armazenamento de persistência. De maneira a tornar as operações de escrita mais 
performáticas, a engine padrão se beneficia da memória do sistema para temporariamente armazenar os dados solicitados, 
para uma posterior persistência no sistema de arquivos. Esse comportamento potencialmente pode levar a perda de 
dados caso o sistema sofra uma interrupção.

--> É possível especificar parâmetros com a opção `writeConcern` nos comandos de escrita para controlar este 
comportamente:
```javascript
{
    w: 1,
    j: false,
    wtimeout: 200
}
```
* `w` define, em um ambiente de múltiplas instâncias, quantas das mesmas devem ser notificadas da operação. Por 
padrão é definido como `1`.
* `j` define se a operação deve ser anotada no arquivo de `jornal`. Este arquivo presente no sistema de arquivos 
funciona como um backup das operações que devem ser consolidadas pela engine caso o sistema sofra uma interrupção 
e seja novamente inicializado. Por padrão é definido como `false`.
* `wtimeout` define um intervalo de tempo aceitável para que a operação de escrita seja performada antes que um 
erro seja retornado.

# 006 The writeConcern in Practice
--> Por padrão `writeConcern` seria definido como:
```javascript
db.collection.insertOne({ ... }, {
    writeConcern: { w: 1 }
})
```

--> Caso seja necessário uma latência extremamente baixa e não tenhamos interesse no retorno da operação:
```javascript
db.collection.insertOne({ ... }, {
    writeConcern: { w: 0 }
})
```
* O servidor retorna `{ acknowledged: false }` não informando os IDs produzidos para os documentos inseridos.

--> Para ter um certo nível de redudância, podemos solicitar a anotação da operação no arquivo `jornal` com:
```javascript
db.collection.insertOne({ ... }, {
    writeConcern: { w: 1, j: true }
})
```

--> Em um ambiente de rede instável com perdas de conexão, podemos definir um timeout para a operação:
```javascript
db.collection.insertOne({ ... }, {
    writeConcern: { w: 1, wtimeout: 3000 }
})
```

# 007 What is Atomicity
--> Por padrão MongoDB garante a atomicidade dos comandos que operam sobre um único documento. Assim `insertOne()` 
ou `updateOne()` só insere/altera um novo documento se toda a operação de escrita foi efeturada com sucesso, e 
caso não seja, é garantido que nenhuma parte documento seja persistida ou afetada na base de dados.

--> Por outro lado, apesar de que em comandos que operam em lotes, como `insertMany()` e `updateMany()`, há 
atomicidade na operação de cada documento, como um todo o comando não é atômico. Para se conseguir um comportamento 
semelhante, é necessário fazer uso de transações.

# 009 Importing Data
--> O utilitário `mongoimport` permite fazer a importação de um arquivo `json`, contendo um documento ou um array 
de documentos, em uma coleção:
```
mongoimport /opt/series.json -d seriesdb -c series --jsonArray --drop
```
* `/opt/series.json` se refere ao arquivo com os documentos a serem importados;
* `-d` define a base de dados;
* `-c` define a coleção;
* `--jsonArray` informa que o arquivo contem um array de documentos, e não um único documento;
* `--drop` informa que a coleção deve ser deletada por completo antes da importação.