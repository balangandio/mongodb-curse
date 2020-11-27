# Cap 12 - Understanding the Aggregation Framework

# 002 What is the Aggregation Framework
--> MongoDB agrupa um conjunto de operadores em uma funcionalidade chamada agregação. As operações de agregação 
fornecem uma maneira de pre-processar os dados no servidor para que sejam recuperados já no formato desejado.
```javascript
db.collection.aggregate(...)
```

# 004 Using the Aggregation Framework
--> A agregação é utilizada pela função `aggregate()` que recebe um array com as operações a serem processadas 
em uma pipeline. Assim como uma `find()`, a agregação faz uso de indexes da coleção, mas apenas na primeira 
operação da pipeline. Como retorno, a função também devolve um iterador.

--> O operador `$match` permite fazer um filtro no mesmo formato que uma consulta com find:
```javascript
db.collection.aggregate([
    {
        $match: { type: "animal" }
    }
])
```

# 005 Understanding the Group Stage
--> O operador `$group` permite agrupar os documentos por campos que identifiquem subconjuntos dentro da 
coleção, e processar alguma operação para tais conjuntos. A utilização em conjunto com o operador `$sum`:
```javascript
db.creatures.aggregate([
    { $match: { environment: "terrestrial" } },
    {
        $group: {
            _id: { kingdom: "$type.kingdom" },
            totalCreatures: { $sum: 1 }
        }
    }
])
```
* Filtra a coleção pelo campo `environment`, e em seguida agrupa os documentos resultantes pelo campo `kingdom`, 
calculando por fim o total de documentos de cada grupo;
* `_id` identifica como os grupos formados da agregação serão identificados;
* `totalCreatures` especifica uma coluna que será formada e cujo valor é calculado pelo operador `$sum`;
* `$sum: 1` permite para cada documento de um subconjunto seja equivalente a um numeral, no caso `1`, para o 
cálculo de soma;
* Obtermos como resultado:
```json
[
    { "_id": { "kingdom": "value1" }, "totalCreatures": 42 },
    { "_id": { "kingdom": "value2" }, "totalCreatures": 23 },
    { "_id": { "kingdom": "value3" }, "totalCreatures": 67 }
    ...
]
```

# 006 Diving Deeper Into the Group Stage
--> O operador `$sort` permite ordenar os dados oriundos de determinado estágio do pipeline:
```javascript
db.creatures.aggregate([
    { $match: { environment: "terrestrial" } },
    { $group: { _id: { kingdom: "$type.kingdom" }, totalCreatures: { $sum: 1 } } },
    { $sort: { totalCreatures: -1 } }
])
```
* Order os dados resultantes do estágio `$group` pelo campo `totalCreatures` decrescentemente.

# 008 Working with $project
--> O operador `$project` permite realizar a operação de projeção sobre os dados em um estágio do pipeline, 
identificando campos que serão incluíndos ou excluídos da projeção. E adicionalmente permite gerar novos campos, 
quando trabalhado em conjunto com outros operadores auxiliares.
```javascript
db.peoples.aggregate([
    { $match: { continent: "asian" } },
    {
        $project: {
            _id: 0,
            gender: 1,
            fullname: {
                $concat: [
                    { $toUpper: { $substrCP: ['$firstName', 0, 1] } },
                    {
                        $substrCP: [
                            '$firstName',
                            1,
                            { $subtract: [{ $strLenCP: '$firstName' }, 1] }
                        ]
                    },
                    ' ',
                    '$lastName'
                ]
            }
        }
    }
])
```
* Realiza a projeção dos campos excluindo `_id`, selecionando `gender`, e gerando um novo campo `fullname`;
* `fullname` é gerado concatenando os campos `firstName` e `lastName`, tendo `firstName` sua inicial em maiúsculo;
* `$substrCP` realiza a operação de substring sobre um campo;
* `$strLenCP` calcula o length de um campo string;
* `$subtract` calcula uma subtração.

# 009 Turning the Location Into a geoJSON Object
--> A conversão de dados entre etapas em uma pipeline é uma situação comum. Para isso operador `$convert` pode 
ser utilizado em projeções:
```javascript
db.peoples.aggregate([
    {
        $project: {
            gender: 1,
            location: [
                $convert: {
                    input: '$home.location.longitude', to: 'double', onError: 0.0, onNull: 0.0
                },
                $convert: {
                    input: '$home.location.latitude', to: 'double', onError: 0.0, onNull: 0.0
                }
            ]
        }
    }
])
```
* Projeto os campos `gender` e `location`;
* `$convert` recebe os parâmetros respectivamente, `input` (campo a ser convertido), `to` (tipo de dado final, 
`onError` (valor resultando quando a conversão falha) e `onNull` (valor resultante quando o campo é nulo).

# 011 Using Shortcuts for Transformations
--> Algumas conversões possuem operadores para escrita mais resumida:
```javascript
db.peoples.aggregate([{
    $project: {
        date: { $toDate: '$birthDate' }
    }
}])
```
* Seria o mesmo que:
```javascript
db.peoples.aggregate([{
    $project: {
        date: { $convert: { input: '$birthDate', to: 'date' } }
    }
}])
```

# 012 Understanding the $isoWeekYear Operator
--> O operador utilitário `$isoWeekYear` permite extratir o valor numérico do ano de um objeto `ISODate`:
```javascript
db.peoples.aggregate([{
    $project: { date: { $toDate: '$birthDate' } },
    $project: { year: { $isoWeekYear: '$date' } }
}])
```

# 014 Pushing Elements Into Newly Created Arrays
--> O operador utilitário `$push` em conjunto com `$group` permite agrupar os valores de um campo dos 
documentos de um grupo, em um array.
```javascript
db.peoples.aggregate([{
    $group: { _id: { age: "$age" }, allHobbies: { $push: "$favoriteHobbie" } }
}])
```
* Agrupa os documentos por `age`, formando o array `allHobbies` com o valor de `favoriteHobbie` de cada 
documento.

# 015 Understanding the $unwind Stage
--> O operador `$unwind`, ao contrário de `$group`, expandi os documentos segundo todos os seus valores 
presentes em um array:
```javascript
db.peoples.aggregate([{
    $unwind: { "$skills" }
}])
```
* Considerando o array `skills`, cada documento será repetido para cada elemento do array;
* Como resultando da operação, o campo `skills` de cada documento será igual a um valor do array. Considerando:
```javascript
[
    { "field": "doc1", "array": [1, 2] },
    { "field": "doc2", "array": [1] }
]
```
Resulta em:
```javascript
[
    { "field": "doc1", "array": 1 },
    { "field": "doc1", "array": 2 },
    { "field": "doc2", "array": 1 }
]
```

# 016 Eliminating Duplicate Values
--> O operador utilitário `$addToSet` funciona da mesma maneira de `$push`, todavia ele mantem o array de valores 
agrupados sem repetições:
```javascript
db.peoples.aggregate([{
    $group: { _id: { age: "$age" }, allHobbies: { $addToSet: "$favoriteHobbie" } }
}])
```
* Agrupa os documentos por `age`, formando o array `allHobbies` sem repetições com o valor de `favoriteHobbie` de 
cada documento.

# 017 Using Projection with Arrays
--> O operador utilitário `$slice` permite obter um intervalo de valores de um array:
```javascript
db.peoples.aggregate([{
    $project: { selectedHobbies: { $slice: ["$allHobbies", 2, 3] } }
}])
```
* Obtem `3` elementos de `allHobbies` situados a partir do índice `2`;
* Caso seja desejado apenas os 3 primeiros: `["$allHobbies", 3]`;
* Caso seja desejado apenas os 3 últimos: `["$allHobbies", -3]`.

# 018 Getting the Length of an Array
--> O operador utilitário `$size` permite obter o quantitativo de elementos de um array:
```javascript
db.peoples.aggregate([{
    $project: { numberOfHobbies: { $size: "$allHobbies" }
}])
```

# 019 Using the $filter Operator
--> O operador utilitário `$filter` permite filtrar em um array somente os elementos que atendam a determinada 
condição:
```javascript
db.peoples.aggregate([{
    $project: {
        selectedHobbies: {
            $filter: {
                input: '$allHobbies',
                as: 'hob',
                cond: { $gt: ["$$hob.frequency", 5] }
            }
        }
    }
}])
```
* Seleciona do array `allHobbies` somente objetos embutidos cujo campo `frequency` é maior que `5`;
* `as` declara a variável de iteração, onde sua referência é feita com `$$` para não conflitar com campos do 
documento.

# 020 Applying Multiple Operations to our Array
--> Obtendo o melhor `placing` de cada atleta conbinando `$unwind` e `$group`:
```json
[
    { "name": "Anna", "races": [{ "day": "clouded", "placing": 5 }, { "day": "sunny", "placing": 16 }, { "day": "clear", "placing": 42 }] },
    { "name": "Cate", "races": [{ "day": "clear", "placing": 12 }, { "day": "raining", "placing": 3 }] }
]
```
* Pipeline:
```javascript
db.athlets.aggregate([
    { $unwind: "$races" },
    { $project: { name: 1, placing: "$races.placing" } },
    { $group: { _id: {id: "$_id"}, name: { $first: "$name" }, bestPlacing: { $min: "$placing" } } },
    { $project: { _id: 0, name: 1, bestPlacing: 1 } },
    { $sort: { bestPlacing: 1 } }
])
```
* O operador `$first` permite trazer o primeiro valor encontrado no subgrupo. Resultado:
```json
[
    { "name" : "Cate", "bestPlacing" : 3 },
    { "name" : "Anna", "bestPlacing" : 5 }
]
```

# 021 Understanding $bucket
--> O operador `$bucket` possui um comportamento semelhante ao `$group`, todavia o seu agrupamento opera 
sobre um campo numérico formando intervalos (buckets) de valores:
```javascript
db.peoples.aggregate([{
    $bucket: {
        groupBy: "$age",
        boundaries: [18, 30, 50],
        output: {
            total: { $sum: 1},
            avg: { $avg: "$age" }
        }
    }
}])
```
* Agrupa os documentos pelo campo `$age` em faixas etárias ([18 a 29, 30 a 49, > 50]) retornando o total e média;
* É retornado no campo `_id` o valor do `boudarie` representado.

--> O operador `$bucketAuto` possui a mesma função, todavia recebe o total de `buckets` desejado como parâmetro e 
produz o intervalo de `boundaries` fazendo uma distribuição sobre os valores do campo agrupado:
```javascript
db.peoples.aggregate([{
    $bucketAuto: {
        groupBy: "$age",
        buckets: 3,
        output: {
            total: { $sum: 1},
            avg: { $avg: "$age" }
        }
    }
}])
```
* É retorno no campo `_id` o valor mínimo e máximo do bucket calculado.

# 022 Diving Into Additional Stages
--> Os operadores `$skip` e `$limit` também podem ser utilizados como estágios na agregação. Todavia como a os 
estágios são executados sequencialmente, deve-se ter atenção na posição em que se econtram na pipeline:
```javascript
db.peoples.aggregate([
    { $sort: { age: -1 } },
    { $skip: 10 },
    { $limit: 10 }
])
```
* Fornece os documentos do 11º ao 20º na lista ordenada decrescentemente pelo campo `age`;
* Caso `$limite` anteceda `$skip`, ou `$sort` é feito depois, não obtemos o resultado desejado.

# 024 Writing Pipeline Results Into a New Collection
--> O operador `$out` permite armazenar o resultado de uma agregação em uma nova coleção:
```javascript
db.peoples.aggregate([
    { $sort: { age: -1 } },
    { $skip: 0 },
    { $limit: 10 },
    { $out: "oldestpeoples" }
])
```
* Armazena o resultado em uma coleção chamada `oldestpeoples`.

# 025 Working with the $geoNear Stage
--> O operador `$geoNear` permite calcular a proximidade entre objetos geográficos. Todavia, como operações 
geográficas dependem da indexação `2dsphere`, o operador só pode ser utilizado no primeiro estágio da pipeline:
```javascript
db.cars.aggregate([{
    $geoNear: {
        near: { type: "Point", coordinates: [-23.5434, 43.3244] },
        maxDistance: 1000,
        num: 10,
        query: { speed: { $gt: 20 } },
        distanceField: "distance"
    }
}])
```
* Consulta `10` documentos cujo campo `speed` é maior que `20`, e que estão localizados a no máximo `1000` metros 
de um ponto;
* O parâmetro `num` permite limitar os resultados retornados pelo estágio, o que é um pouco mais performático que 
fazer a limitação em um outro estágio;
* Como `$geoNear` precisa ser o primeiro estágio da pipeline, não é possível filtrar a coleção previamente com `$match`. 
Para contornar essa limitação, é oferecido o parâmetro `query`, que permite filtrar documentos antes da operação;
* O parâmetro `distanceField` adiciona um campo aos documentos retornados, contendo a distância veificada na operação.