# Cap 8 - Update Operations

# 002 Updating Fields with updateOne(), updateMany() and $set
--> `updateOne()` atualiza os dados do primeiro documento compatível com o critério de filtro, mesmo que 
ao todo existam mais.
```javascript
db.collection.updateOne({ _id: ObjectId(...) }, { $set: {...} })
```

--> `updateMany()` atualiza todos os documentos compatíveis com o critério de filtro.
```javascript
db.collection.updateMany({ type: "flat" }, { $set: {...} })
```

--> O operador `$set` é utilizado para definir valores para propriedades. Como resultado, se restringe a 
alterar o valor de uma propriedade, ou incluir a mesma caso já não exista. Em nenhuma instância, ocorre 
deleção de propriedades.
```javascript
db.collection.updateMany({ type: "flat" }, {
    $set: {
        type: "superflat",
        content: null
    }
})
```
* Define valores para as propriedades `type` e `content`.

# 004 Incrementing & Decrementing Values
--> O operador `$inc` permite incrementar/decrementar campos numéricos:
```javascript
db.collection.updateMany({ type: "flat" }, { $inc: { height: -42 } })
```
* O valor da propriedade `height` é decrementado em `42`. Caso o seja positivo, ocorreria um incremento.

--> É possível utilizar `$inc` e `$set` no mesmo comando. Todavia caso seja tentado alterar a mesma propriedade 
por âmbos as operações, será lançado um erro.
```javascript
db.collection.updateMany({ type: "flat" }, { $set: { height: 653 }, $inc: { height: -42 } })
```
* O camando falha pois `height` está sendo alterada por âmbos operadores.

# 005 Using $min, $max and $mul
--> Os operadores `$min` e `$max` permitem definir um valor para um campo numérico somente se o valor, respectivamente, 
for menor que o atual, ou maior que o atual.
```javascript
db.collection.updateOne({ _id: ObjectId(...) }, { $min: { weight: 54 }, $max: { height: 42 } })
```
* Caso `54` seja inferior ao valor presente em `weight`, o mesmo será definido na propriedade;
* Caso `42` seja superior ao valor presente em `height`, o mesmo será definido na propriedade;
* Quando as condições não são aceitas, o comando apenas não retorna a quantidade de alterações esperada.

--> O operador `$mul` permite multiplicar o valor de um campo numérico ou um determinado número e armazenar o 
resultado na respectiva propriedade.
```javascript
db.collection.updateOne({ _id: ObjectId(...) }, { $mul: { weight: 1.5 } })
```
* O valor da propriedade `weight` definido para `$weight * 1.5`.

# 006 Getting Rid of Fields
--> O operador `$unset` permite remover propriedades de um documento:
```javascript
db.collection.updateOne({ _id: ObjectId(...) }, { $unset: { hasHumanKind: "" } })
```
* Remove a propriedade `hasHumanKind` do documento;
* O valor nos nomes das propriedades do objeto `hasHumanKind: ""` não tem importância. Por convenção usa-se `""`.

# 007 Renaming Fields
--> O operador `$rename` permite renomeiar propriedades, quando as mesmas existem.
```javascript
db.collection.updateOne({ _id: ObjectId(...) }, { $rename: { age: "yearsOfAge" } })
```
* A propriedade `age`, caso exista no documento, é renomeada para `yearsOfAge`.

# 008 Understanding upsert()
--> `updateOne()` recebe um parâmetro adicional chamado `upsert`, que quando definido como `true`, permite 
que o comando crie um documento caso não haja nenhum que atenda ao critério de filtro.
```javascript
db.collection.updateOne({ name: "Ana" }, { $set: { age: 4242, skill: "fast" } }, { upsert: true })
```
* Caso nenhum documento com `name: "Ana"` seja encontrado, um novo documento é criado, e é retornado pelo 
comando o ID atribuido;
* O documento produzido seria: `{ "name": "Ana", "age": 4242, "skill": "fast" }`;
* Os critérios de igualdade presente no objeto de filtro são entendidos como propriedades do objeto.

# 010 Updating Matched Array Elements
--> Um único elemento específico em um array, sempre o primeiro elemento que atende aos critérios de filtro, 
pode ser alterado fazendo uso da notação `arrayName.$`.
```javascript
db.collection.updateOne({ sports: { $elemMatch: {type: "endurance"} } }, { $set: { "sports.$.hasLongDuration": true } })
db.collection.updateOne({ "sports.type": "endurance" }, { $set: { "sports.$.hasLongDuration": true } })
```
* Todos os documentos que possuirem no array `sports` algum objeto com campo `type: "endurance"`, são afetados 
de maneira que, somente no primeiro objeto que atenda ao filtro, `hasLongDuration` será definido como `true`;
* Obrigatóriamente, a notação só funciona caso o array seja citado no critério de filtro. Caso contrário, um 
erro é lançado.

--> Para redefinir o elemento do array como um todo, basta atribuir somente para `arrayName.$`:
```javascript
db.collection.updateOne({ sports: { $elemMatch: {type: "endurance"} } }, {
    $set: {
        "sports.$": { title: "endurance sport", type: "endurance", hasLongDuration: true }
    }
})
```

# 011 Updating All Array Elements
--> Todos os elementos de um array podem ser alterados utilizando a notação `arrayName.$[]`, independente se os 
mesmos atende ou não ao critério de filtro.
```javascript
db.collection.updateOne({ "sports.type": "endurance" }, { $inc: { "sports.$[].duration": 42 } })
```
* Todos os elementos do array `sports`, mesmo os não que possuem o campo `type: "endurance"`, terão a propriedade 
`duration` incrementada pelo valor `42`.

# 012 Finding & Updating Specific Fields
--> Todos os elementos de um array que necessariamente atendam a um critério de filtro podem ser alterados fazendo 
uso da notação `arrayName.$[var]` e do parâmetro adicional `arrayFilters` nos comandos `updateOne()` e `updateMany()`.
```javascript
db.collection.updateOne(
    { name: "Human" },
    { $set: { "sports.$[el].hasLongDuration": true } },
    { arrayFilters: [{ "el.duration": {$gt: 42} }] }
)
```
* Para todos os documentos com `name: "Human"`, os elementos do array `sports` que possuirem a `duration` > 42 
terão a propriedade `hasLongDuration` definida como `true`;
* `el` é um alias arbitrário para ligar o critério de filtro com o critéria de alteração;
* Como `arrayFilters` recebe um array, é possível alterar elementos em mais de um array com diferentes alias.

# 013 Adding Elements to Arrays
--> O operador `$push` permite adicionar elementos em arrays.
```javascript
db.collection.updateMany({}, { $set: { sports: { $push: {title: "cycling", type: "endurance"} } } })
```
* Adiciona um único elemento ao array `sports` de todos os documentos.

--> O operador `$each` permite adicionar múltiplos elementos, e o operador `$sort` permite estabelecer a ordem 
que todos os elementos no array, não só os novos, devem apresentar após a inserção.
```javascript
db.collection.updateMany({}, {
    $set: {
        sports: {
            $push: {
                $each: [
                    { title: "cycling", type: "endurance"},
                    { title: "run", type: "endurance"}
                ],
                $sort: { title: 1 }
            }
        }
    }
})
```
* Em `$sort` a ordenação dos campos são especificados como valor 1 (ordem crescente) e 0 (ordem decrescente).

# 014 Removing Elements from Arrays
--> O operador `$pull` permite remover todos os elementos de um array que atendam um critério de filtro.
```javascript
db.collection.updateMany({}, { $pull: { sports: { type: "endurance" } } })
```
* Para todos os documentos, remove todos os elementos do array `sports` que possuirem `type: "endurance"`.

--> O operador `$pop` permite remover apenas o primeiro ou o último elemento de um array.
```javascript
db.collection.updateMany({}, { $pop: { sports: -1 } })
db.collection.updateMany({}, { $pop: { sports: 1 } })
```
* O primeiro comando remove o primeiro elemento do array `sports`;
* O segundo comando remove o último elemento do array `sports`.

# 015 Understanding $addToSet
--> O operador `$addToSet` permite adicionar um único elemento a um array. Todavia o elemento só é adicionado 
se no array não existir nenhum outro elemento igual a ele.
```javascript
db.collection.updateMany({}, { $addToSet: { sports: { title: "cycling", type: "endurance"} } })
```
* Para todos os documentos, adiciona um elemento ao array `sports`.