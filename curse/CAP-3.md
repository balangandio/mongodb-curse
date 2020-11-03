# Cap 3 - Schemas & Relations How to Structure Documents

# 003 Why Do We Use Schemas
--> Apesar do MongoDB não prever qualquer restrição sobre a heterogeneidade dos documentos em uma coleção, 
implicitamente as aplicações e negócios levam em consideração a existência de schemas sobre os dados. 
Nesse sentido, o uso do MongoDB não se torna um impeditivo uma vez que a homogeniedade dos dados não 
necessariamente precisa ser estabelecida pelo banco de dados.

# 004 Structuring Documents
--> É possível definir o valor `null` a uma propriedade de um documento de maneira a evidenciar a existência 
do campo e não necessariamente definir um valor ao mesmo. Essa possibilidade se torna uma alternativa no uso 
de propriedades extras em documentos, mantendo ainda um schema homogênio na coleção:
```json
{
  "name": "gerger",
  "price": 435.23,
  "details": null
}, {
  "name": "yrvere",
  "price": 5.53,
  "details": {
    "usage": "vbtrthr"
  }
}
```

# 005 Data Types - An Overview
--> Dentre os principais tipos de dados no MongoDB tempos:
* `Text`: valores string delimitados com aspas duplas;
* `Boolean`: valores literais true/false;
* `Integer`: int32 para valores inteiros;
* `NumberLong`: int64 para valores inteiros longos;
* `NumberDecimal`: para valores decimais comuns;
* `ObjectId`: usado para a identificação única de um documento; ordenável;
* `ISODate`: usado para datas com horário;
* `Timestamp`: precisão horária; timestamps são sempre únicos no MongoDB;
* `Embedded documents`: qualquer outro documento;
* `Array`: listas de qualquer valor, ou de listas.

# 006 Data Types in Action
--> É possível deletar a base de dados atualmente em seleção com o comando:
```javascript
db.dropDatabase()
```

--> Cada driver de conexão com o MongoDB pode adotar uma maneira diferente de construir tipos de dados. 
No cliente shell objetos `Date` e `Timestamp` podem ser instânciados com:
```javascript
db.products.insertOne({ name: "frweger", build: new Date(), insertedAt: new Timestamp() });
```

--> Como o cliente shell do MongoDB possui um interpretador javascript, valores inteiros e decimais são 
tratados como um mesmo tipo, que é lido pelo driver como `NumberDecimal`. Para forçar o armazenamento de 
um valor inteiro, é necessário instanciar o valor com `NumberInt` (int32):
```javascript
db.products.insertOne({ value: NumberInt(42) });
```

--> No cliente shell, para obter o tipo de dado de uma propriedade presente nos documentos de uma coleção, 
a função `typeof` pode ser utilizada:
```javascript
typeof db.products.findOne({ v: 'f4r2'}).propName
```

--> É possível consultar algumas estatísticas sobre a base de dados em seleção com o comando:
```javascript
db.stats()
```

# 008 How to Derive your Data Structure - Requirements
--> A modelagem de uma base de dados leva em consideração várias questões:
* Quais dados são necessários ou produzidos;
* Onde os dados são necessário na solução;
* Quais informações se deseja exibir;
* Qual a frequência de recuperação dos dados;
* Qual a frequência de escrita/alterações sofrem os dados.

# 009 Understanding Relations
--> Relacionamentos são estabelecidos com documentos embutidos ou com referências. A conveniência de cada 
estratégia esta dependente das necessidades da solução.
* Em uma coleção de custumers, os dados de residência podem ser armazenados em um documento embutido, uma 
vez que os dados não são compartilhados entre custumers;
* Em uma coleção de users de uma comunidade de leitores, talvez uma lista de livros favoritos armazene apenas 
referências à coleção de books. Visto que diferentes usuários possuem livros em comum, e que a atualização de 
algum dado em um livro não levará algum objeto à obsolescência.

# 010 One To One Relations - Embedded
--> Em um relacionado `OneToOne`, o uso de documentos embutidos se torna mais conveniente e a divisão em coleções 
e uso de referências agrega mais complexidade para a aplicação.
* Em uma coleção de pacientes, os dados do seu prontuário não precisão ser alocados em uma outra coleção.

# 011 One To One - Using References
--> Em um relacionado `OneToOne`, por vezes as entidades envolvidas não são periféricas e possuem seus respectivos 
usos dentro da solução. Em um momento apenas os dados de uma entidade são relevantes, e em outro, o contrário.
* Em uma em hospital veterinário, podem existir coleções separadas de proprietários e pets. Onde a coleção de 
pets apresenta dados genéticos e estatísticos utilizados em uma aplicação de análise de dados. Caso os documentos 
sejam embutidos, a aplicação sempre consumiria dados de proprietário desnecessariamente;

# 012 One To Many - Embedded
--> Em um relacionamento `OneToMany`, documentos embutidos podem ser utilizados numa situação em que entidades 
apresentam uma forte dependência em uma entidade em comum, e se resumem a uma pequena quantidade de maneira que 
o limite de armazenamento de documento não seja antigido.
* Em um blog, uma publicação pode conter a própria lista de comentários.

# 013 One To Many - Using References
--> Em um relacionamento `OneToMany`, a quantidade de entidades periféricas podem ser muito grande a ponto de 
ser limitada pelo tamanho máximo de documentos, forçando o uso de referências.
* Em um aplicação de monitoramento de veículos, o histório de localização emitido por um veículo pode ser bem 
extenso, sendo dedicado à uma coleção em separada à de veículos.

# 014 Many To Many - Embedded
--> Em um relacionamento `ManyToMany`, documentos embutidos podem ser convenientes quando a duplicação de dados 
é gerenciável ou a obsolescência faz parte da própria solução.
* Em um site de vendas, o relacionamento entre clientes e produtos pode ser implementado com um objeto pedido 
contendo os dados do produto por completo, uma vez que pode ser interessante para a solução manter registro o 
estado do produto no momento do pedido.

# 015 Many To Many - Using References
--> Em um relacionamento `ManyToMany`, referências podem ser interessantes quando a duplicação e obsolescência 
de dados é um problema para a solução.
* Em uma biblioteca, dados de autores e livros podem ser mantidos em coleções com o documento livro armazenando 
a lista de referência aos autores. Alteração em um dado de autor não implicariam em uma obsolescência.

# 016 Summarizing Relations
--> `Embedded documents`: 
* Grupos de dados logicamente atrelados;
* Dados isolados, não associados com outras entidades;
* Requer cuidados quantidade de níveis de aninhamento e tamanho de documentos;

--> `References`:
* Adequando para dados compartilhados e armazenamento de dados sobre o próprio relacionamento;
* Incrementa uma certa complexidade à aplicação.

# 017 Using lookUp() for Merging Reference Relations
--> É possível utilizar o operador `$lookup` com a função `aggregate` para recuperar dados de diferentes coleções 
que estejam relacionadas por referência:
```javascript
db.books.aggregate([{
  $lookup: {
    from: "authors",
    localField: "authors",
    foreignField: "_id",
    as: "creators"
  }
}])
```
* Levando em consideração que em uma coleção de `books` os documentos possuem um campo `localField: "authors"` 
com a lista de referências para a coleção `from: "authors"` campo `foreignField: "_id"`;
* Como resultado, é recuperado todos os dados de `books` com a inclusão do campo `as: "creators"` contendo a 
lista de objetos `authors`.

# 020 Understanding Schema Validation
--> MongoDB permite definir validações de schema de maneira a alertar ou rejeitar quando documentos são intruduzido 
apresentamd uma estrutura diferente da definida.
* É possível definir que a validação ocorra em qualquer operação (insert/update) na coleção, ou caso já exista dados 
armazenados, apenas para operações sobre novos documentos ou documentos que já tenham sofrido validação;
* É possível definir que a validação lançe um erro ou apenas um aviso.

# 021 Adding Collection Document Validation
--> A função `db.createCollection()` cria coleções de maneira explícita e permite definir validação de schema:
```javascript
db.createCollection('posts', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['title', 'text', 'creator', 'comments'],
      properties: {
        title: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        text: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        creator: {
          bsonType: 'objectId',
          description: 'must be an objectid and is required'
        },
        comments: {
          bsonType: 'array',
          description: 'must be an array and is required',
          items: {
            bsonType: 'object',
            required: ['text', 'author'],
            properties: {
              text: {
                bsonType: 'string',
                description: 'must be a string and is required'
              },
              author: {
                bsonType: 'objectId',
                description: 'must be an objectid and is required'
              }
            }
          }
        }
      }
    }
  }
})
```
* `bsonType` indica o tipo de dado do documento da coleção e de propriedades;
* `required` lista os nomes das propriedades obrigratoriamente requeridas;
* `properties` define o conjunto de proprieades;
* `items` permite definir a estrutura dos items de uma propriedade array.

# 022 Changing the Validation Action
--> A validação de schema de uma coleção já existente pode ser alterado com o comando `db.runCommand()`:
```javascript
db.runCommand({
  collMod: 'posts',
  validator: {
    $jsonSchema: { ... }
  },
  validationAction: 'warn'
})
```
* `collMod` identifica a coleção;
* `validator` recebe a definição do schema;
* `validationAction` define a ação tomada quando a validação falha. Tem por valor padrão `error`, e quando 
`warn` é definido, um aviso do ocorrido é escrito no log do sistema de bando de dados.
