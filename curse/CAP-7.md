# Cap 7 - Read Operations - A Closer Look

# 002 Methods, Filters & Operators
--> As operações de consulta apresentam um estrutura básica:
```javascript
db.collection.find({ age: { $gt: 42 } })
```
* `find` é o método de consulta;
* `{ age: { .. } }` é o objeto de filtro;
* `$gt` é um operador de comparação.

# 003 Operators - An Overview
--> MongoDB oferece um conjunto de operadores utilizados nos comandos para especificar o comportamento da 
operação. Os operadores se dividem em:
* Operadores usados em comandos de leitura - `read`;
* Operadores usados em comandos de atualização - `update`;
* Operadores usados em comandos de agregação - `aggregation`.

--> Dentre os operadores leitura, temos:
* Operadores de seleção - `query selectors`;
* Operadores de projeção - `projection`.

# 004 Query Selectors & Projection Operators
--> Query selectors, operadores de seleção, se dividem nas categorias:
* `Comparison`
* `Logical`
* `Element`
* `Evaluation`
* `Array`
* `Comments`
* `Geospatial`

--> Projetion operators, operadores de projeção, são:
* `$`
* `$elemMath`
* `$meta`
* `$slice`

# 005 Understanding findOne() & find()
--> `find()` e `findOne()` se diferenciam devido a find retornar um cursor para consumo da consulta, enquanto 
que findOne sempre retorna o primeiro elemento da consulta.
* Não informando nenhum filtro para `findOne()`, o primeiro documento da coleção é retornado.

# 006 Working with Comparison Operators
--> Quando nenhum operador é informado em um filtro com valores, o operador de igualdade `$eq` é usado. E para 
obter a operação contrária, `$ne`, não igual, pode ser usado:
```javascript
db.collection.find({
    age: 42,
    age: { $eq: 42 },
    height: { $ne: 42 }
})
```

--> É oferecido os operadores de intervalo: `$gt` (maior que), `$gte` (maior que ou igual), `$lt` (menor que), 
`$lte` (menor que ou igual).
```javascript
db.collection.find({
    age: { $gt: 42 },
    height: { $gte: 654 },
    size: { $lt: 23 },
    weight: { $lte: 23 }
})
```

# 007 Querying Embedded Fields & Arrays
--> Propriedades em objetos embutidos podem ser consultas envolvendo um path com aspas duplas `"`:
```javascript
db.collection.find({ "rating.average": { $gt: 5 } })
```

--> Igualdades com arrays se comportam de maneira diferente:
* Quando o valor comparado não é um array, é verificado se o mesmo se encontra dentro da estrutura nos 
documentos.
```javascript
db.collection.find({ genres: "drama" })
```
* Quando o valor comparado é um array, é verificado se todos os elementos são iguais em valor, tipo e 
posição na estrutura nos documentos.
```javascript
db.collection.find({ genres: ["drama"] })
```

# 008 Understanding $in and $nin
--> O operador `$in` e o seu oposto `$nin` permite efetuar uma comparação de igualdade com conjunto de valores 
informados em um array:
```javascript
db.collection.find({
    age: { $in: [30, 40, 50] },
    height: { $nin: [143, 267] }
})
```
* O valor `age` deve ser igual a pelo menos um dos valores [30, 40, 50];
* O valor `height` não deve estar incluso no conjunto de valores [143, 267].

# 009 $or and $nor
--> Para os operadores lógicos `$or`, e seu contrário `$nor`, é especificado um conjunto de comparações cujo os 
documentos devem atender a ao menos uma, quando o primeiro é usado, ou nenhuma, quando o segundo é utilizado:
```javascript
db.collection.find({
    $or: [
        { stars: { $lt: 3 } },
        { stars: { $gt: 8 } }
    ],
    $nor: [
        { age: 42 },
        { height: 653 }
    ]
})
```
* `stars` deve possuir um valor inferior a 3 ou superior a 8;
* `age` não pode ser igual a 42 e `height` não pode ser igual a 653.

# 010 Understanding the $and Operator
--> O operador lógico `$and` verifica se o documento atende a um conjunto de comparações especificadas. 
O que corresponde ao mesmo comportamento apresentado quando separamos comparações com `,`:
```javascript
db.collection.find({
    $and: [
        { age: 42 },
        { height: 535 }
    ]
})
db.collection.find({ age: 42, height: 535 })
```

--> Todavia `$and` se diferiencia da concatenação de comparações por permitir que repetimos a mesma 
propriedade, o que não é permitido dentro de um objeto json:
```javascript
db.collection.find({
    $and: [
        { genres: "drama" },
        { genres: { $nq: ["drama"] } }
    ]
})
```
* Consulta os documentos cujo array `genres` possui o valor `drama`, todavia não somente ele;
* No cliente shell ao especificar um objeto json com propriedades repetidas, somente a última especificação 
é usada:
```javascript
db.collection.find({ age: 42, age: 535 })
db.collection.find({ age: 535 })
```

# 011 Using $not
--> O operador lógico `$not` inverte o resultado lógico de uma comparação:
```javascript
db.collection.find({ age: { $not: {$eq: 42} }  })
```
* Consulta os documentos cuja propriedade `age` não é igual a 42;
* Seria o mesmo que:
```javascript
db.collection.find({ age: { $ne: 42 }  })
```

# 012 Diving Into Element Operators
--> O operador `$exists` permite verificar se uma propriedade existe dentro de documentos:
```javascript
db.collection.find({ age: { $exists: true, $ne: null }  })
```
* Consulta os documentos que possuem a propriedade `age` e o seu valor não é `null`.

# 013 Working with $type
--> O operador `$type` permite veirficar se o valor presente em uma propriedade é de um determinado tipo:
```javascript
db.collection.find({ age: { $exists: "double" }  })
```
* Consulta os documentos que possuem a propriedade `age` com valor do tipo `double`;
* É possível informar um array de tipos nos quais o valor pode ser aceito:
```javascript
db.collection.find({ age: { $exists: ["double", "string"] }  })
```

# 014 Understanding Evaluation Operators - $regex
--> O operador `$regex` permite verificar se valores textuais são compatíveis com uma expressão regular:
```javascript
db.collection.find({ summary: { $regex: /musical/ }  })
```
* Deve se ter em mente que a performance da operação pode se tornar um problema ao processar textos longos e 
coleções volumosas;
* O uso de índices textuais é mais performático para buscas de texto.

# 015 Understanding Evaluation Operators - $expr
--> O operador `$expr` permite refenreciar campos nos documentos de maneira a produzir expressões lógicas:
```javascript
db.players.find({
    $expr: {
        $gt: ["$wins", "$defeats"]
    }
})
```
* Verifica os documentos onde o valor do campo `wins` é maior que do `defeats`.
* Os campos são citados entre `"` com o nome iniciando com o caracter `$`;

--> É possível escrever expressões mais complexas com condicionais fazendo uso do operador `$cond`:
```javascript
db.players.find({
    $expr: {
        $gt: [
            {
                $cond: {
                    if: {
                        $gt: ["$level", 1]
                    },
                    then: "$wins",
                    else: {
                        $sum: ["$wins", "$draws"]
                    }
                }
            },
            "$defeats"
        ]
    }
})
```
* Players acima do level 1: `wins > defeats`;
* Players no level 1, `(wins + draws) > defeats`;
* `$sum` é um dos muitos operadores de agregação que permitem realizar cálculos dentro de expressões.

# 017 Diving Deeper Into Querying Arrays
--> Arrays podem não só conter valores literais como também objetos embutidos. Nesta ocasião MongoDB 
possibilita acessar propriedades destes objetos de maneira a verificar se ao menos um elemento do array 
atende a determinada comparação:
```javascript
db.collection.find({ "sports.type": { $in: ["endurance", "motorsport"] } })
```
* Consulta documentos em que o array `sports` possui algum objeto embutido cuja propriedade `type` é igual 
a um dos valores `["endurance", "motorsport"]`.

# 018 Using Array Query Selectors - $size
--> É comparar tamanhos de arrays com o operador `$size`:
```javascript
db.collection.find({ sports: { $size: 2 } })
```
* Consulta documentos em que o array `sposts` possui apenas 2 elementos;
* O operador `$size` possui a limitação de não suportar comparações que não igualdade, como `$gt` ou `$lt`.

# 019 Using Array Query Selectors - $all
--> O operador `$all` permite verificar se um array possui um conjunto de valores independente da ordem na 
qual eles estão dispostos:
```javascript
db.collection.find({ sports: { $all: ["cycling", "marathon", "swimming"] } })
```

# 020 Using Array Query Selectors - $elemMatch
--> O operador `$elemMath` permite verificar se um array possui um objeto embutido que, diferentemente da 
navegação em propriedade que só permite uma comparação por objeto, atenda a um conjunto de critérios:
```javascript
db.collection.find({
    sports: {
        $elemMath: {
            type: "endurance",
            athletes: { $gt: 42 }
        }
    }
})
```
* Consulta documentos cujo array `sports` possui um objeto cuja propriedade `type` é igual a `"endurance"` e 
`athletes` é maior que `42`.

# 022 Understanding Cursors
--> Cursores possibilitam que resultados volumosos de consultas sejam consumidos de maneira controlada, 
economizando recursos de hardware e rede.

--> O cliente shell exibe por padrão os 20 primeiros resultados em consultas que retornam cursores.

# 023 Applying Cursors
--> Cursores oferecem algumas funções básicas:
```javascript
const cursor = db.collection.find({})
```
* `cursor.next()`: retorna o próximo elemento do cursor, ou lança um erro caso tenha sido totalmente consumido;
* `cursor.hasNext()`: retorna true/false sinalizando a existência de um próximo elemento;
* `cursor.forEach(elem => {})`: consume inteiramente o cursor executando um callback de tratamento;
* `cursor.count()`: retorna a quantidade de elementos disponíveis no cursor. Retorna um valor já previamente 
mensurado pelo banco de dados.

# 024 Sorting Cursor Results
--> É possível fazer ordenação dos documentos de um cursor com a função `cursor.sort()`:
```javascript
db.collection.find({}).sort({ rating: 1, runtime: -1 })
```
* Ordena os documentos pela propriedade `rating` em ordem crescente e, para valores coincidentes, ordena os 
documentos por `runtime` em ordem decrescente.

# 025 Skipping & Limiting Cursor Results
--> Cursores oferecem as funções `cursor.skip()` e `cursor.limit()` que respectivamente permitem saltar uma 
quantidade de elementos, e consumir uma quantidade. Juntas são úteis para implementar paginações.
```javascript
db.collection.find({}).sort({ rating: 1, runtime: -1 }).skip(10).limit(10)
```
* Retorna os elementos a partir da posição 11ª à 21ª;
* Internamente o MongoDB entende que a operação de `sort` precede `skip` que possua vez precede `limit`, 
independente da ordem as mesmas são executadas: `db.collection.find().skip().sort().limit()`.

# 026 Using Projection to Shape our Results
--> É possível realizar uma projeção sobre o conjunto de propriedades dos documentos em uma consulta fazendo 
uso do segundo parâmetro dos comandos, de maneira a recuperar somente os dados necessários:
```javascript
db.collection.find({}, { title: 1, "content.description": 1, _id: 0 })
```
* Propriedades assinaladas com valor `1` são incluídas, e valor `0` excluídas da projeção;
* Por padrão todas as propriedades são excluídas com exceção de `_id`;
* A seleção de propriedades de objetos embutidos é feito com a notação separada por `.`.

# 027 Using Projection in Arrays
--> É possível realizar projeção sobre arrays de maneira a recuperar somente os valores relevantes usando 
`$elemMatch`:
```javascript
db.collection.find({}, {
    luckyNumbers: {
        $elemMatch: {
            $gte: 42
        }
    },
    _id: 0
})
```
* Retorna uma projeção do array contendo somente valores acima de 42, ex.: `{ "luckyNumbers": [63, 654, 42, 98] }`

--> Existe uma síntaxe específica para arrays que permite retornar a primeira ocorrência de um array que é 
citado no objeto de filtro:
```javascript
db.collection.find({genres: "drama"}, { "genres.$": 1, _id: 0 })
```
* `"genres.$"`: nome da propriedade array termiando com `.$`;
* É retornando apenas documentos contendo: `{ "genres": ["drama"] }`
* Neste caso, a primeira ocorrência do array é `"drama"`. Caso o filtro seja feito com o operador `$all`: 
`{genres: {$all: ["drama", "action"]}}`. Seria projetado apenas: `{ "genres": ["action"] }`, e somente para os 
documentos com o array contendo `"action"`.

# 028 Understanding $slice
--> É possível realizar projeção sobre arrays de maneira a recuperar apenas um intervalo de valores, com uso de 
`$slice`:
```javascript
db.collection.find({}, { genres: { $slice: 2 } })
db.collection.find({}, { genres: { $slice: [1, 3] } })
```
* `$slice: 2` seleciona apenas os 2 primeiros elementos do array;
* `$slice: [1, 3]` define a quantidade de elementos que é saltada, e em sequência a quantidade de elementos que 
é selecionada. Neste caso o 1º é ignorado, e os 3 seguintes selecionados: [2º, 3º, 4º].