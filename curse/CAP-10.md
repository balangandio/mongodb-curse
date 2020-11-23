# Cap 10 - Working with Indexes

# 002 What Are Indexes & Why Do We Use Them
--> Indexes permitem incrementar a performance de consultas. Por padrão quando documentos são consultados 
por um determinado campo, é feito um COLLSCAN onde toda a coleção é percorrida e a comparação do campo é 
efetuada para cada documento. Caso o campo seja indexado, é feito um IXSCAN na estrutura ordenada do index, 
que possua vez armazena o ponteiro para a localização do documento.

--> Colunas são escolhidas para compor um index, uma estrutura de dados ordenada armazenada em adição a 
coleção. Em contrapartida ao incremento de performance, essa estrutura precisa ser mantida consistente entre 
inserções, atualizações e deleções de documentos, o que decrementa a performance destas operações.

# 003 Adding a Single Field Index
--> É possível verificar alguns dados sobre a execução de um consulta, atualização ou deleção, com `explain()`:
```javascript
db.collection.explain().find({...})
db.collection.explain("executionStats").find({...})
```
* `explain("executionStats")` permite verificar dados como tempo de execução, documentos lidos, documentos 
retornados, etc, que permitem verificar a saúde da operação em determinada coleção;
* É informado também a estratégia de consulta utilizada, como COLLSCAN e IXSCAN.

--> A criação de um index simples pode ser feita com:
```javascript
db.collection.createIndex({"detail.type": 1})
```
* É criado um index a partir do objeto embutido `detail` propriedade `type`;
* `1` corresponde ao sentido da ordenção no index, cujo valor contrário seria `0`, ordenação decrescente.

# 005 Understanding Index Restrictions
--> Indexes podem ser removidos com:
```javascript
db.collection.dropIndex({"detail.type": 1})
```

--> A eficiência de indexes em consultas que retornam uma grande parte do total de documentos da coleção tende 
muitas vezes ser pior que uma consulta COLLSCAN usual. Neste caso, o processo de recuperação do documento indicado 
por um registro do index é executado mutias vezes, o que é mais lento que uma leitura completa da coleção.

--> A eficiência de indexes também está atrelada a diversidade de valores. Por vezes algumas colunas armazenam 
apenas alguns poucos tipos de valores diferentes, o que pode abrangir uma grande quantidade de documentos.

# 006 Creating Compound Indexes
--> Indexes também podem ser compostos por mais de uma coluna:
```javascript
db.collection.createIndex({age: 1, gender: 1})
```

--> A ordem das colunas em indexes compostos é importante, pois a mesma determina qual a coluna determinante para 
o uso do index:
```javascript
db.collection.find({age: 42, gender: "female"})
db.collection.find({age: 42})
db.collection.find({gender: "female"})
```
* Somente as duas primeiras consultas fazem uso do index devido ao uso da coluna inicial do index.

# 007 Using Indexes for Sorting
--> Indexes são utilizados internamente para ordenação também, quando as mesmas são baseadas em colunas indexadas. 
Além do incremento em performance, esse comportamento é em particular bastante útil para ordenação em grandes 
coleções, pois MongoDB possui limitações de espaço em memória que é utilizada na ordenação (32 Megabytes).
```javascript
db.collection.find({age: 42}).sort({gender: 1})
```
* Caso `gender` seja um index ou faça parte de um index composto, a ordenação é beneficiada.

# 008 Understanding the Default Index
--> Os indexes associados a uma coleção pode ser listados com:
```javascript
db.collection.getIndexes()
```
* Por padrão MongoDB cria um index para a campo `_id`.

# 009 Configuring Indexes
--> Ao criar um index, é possível especificar adicionalmente a propriedade `unique`, de maneira que seja garantido 
que o index e, consequentemente o campo, não possuam valores duplicados:
```javascript
db.collection.createIndex({email: 1}, {unique: true})
```
* `createIndex()` retornará um erro caso o campo já possua valores duplicados.

# 010 Understanding Partial Filters
--> Lidando com grandes coleções, muitas vezes a criação de um index pode impactar as demais operações devido a 
manutenção da sua igualmente grande estrutura. Neste contexto, é possível pensar em qual parcela dos dados da coluna 
do index é mais requisitada em consultas, de maneira a armazenar no index somente os dados de maior utilidade. 
MondoDB permite criar indexes parciais informando adicionalmente um filtro:
```javascript
db.collection.createIndex({age: 1}, {partialFilterExpression: {gender: "female"}})
```
* `partialFilterExpression` recebe uma expressão de filtro igualmente a operação de consulta;

--> Quando uma consulta é solicitada, MongoDB verifica se a mesma consegue ser satisfeita somente com o index ou se 
extrapola o conjunto de documentos por ele referenciado, sendo neste caso feita uma COLLSCAN usual:
```javascript
db.collection.find({age: {$gt: 42}, gender: "female"})
db.collection.find({age: {$gt: 42}})
```
* A última consulta não consegue tirar proveito do index pois potencialmente abarca pode mais documentos que os do index.

# 011 Applying the Partial Index
--> Indexes com a propriedade `unique` consideram a não existência do seu campo como um dos valores do index. Assim não é
possível ter mais que um documento sem o campo do index.

--> Uma maneira de contornar isso mantendo ainda sim o index como `unique`, consiste em usar `partialFilterExpression` 
para manter na estrutura somente os valores existentes:
 ```javascript
db.collection.find({email: 1}, { 
    unique: true,
    partialFilterExpression: { email: { $exists: true } }
})
```

# 012 Understanding the Time-To-Live (TTL) Index
--> Indexes criados sobre campos do tipo `ISODate` permitem adicionalmente configurar a propriedade `expireAfterSeconds`, 
qual possibilita definir um intervalo limite para que o valor `ISODate` esteja defasado em atual momento. Quando é 
verificado que o limite foi extrapolado, o documento é removido da coleção.
```javascript
db.collection.createIndex({createdAt: 1}, {expireAfterSeconds: 20})
```
* Determina que documentos serão removidos após ser constatado que o campo se encontra defasado em `20` segundos;
* Quando o index é criado, os documentos já existentes não são verificados. A verificação somente ocorre quando algum 
documento é inserido na coleção;
* A opção `expireAfterSeconds` é ignorada em campos não temporais.

# 013 Query Diagnosis & Query Planning
--> Antes de executar uma consulta, MongoDB realiza uma avaliação sobre qual extratégia seria a mais performática no 
contexto dos dados consultados. Tais estratégias são identificadas como planos de execução, que podem ser escolhidos ou não.

--> Adicionalmente, `explain()` permite extrair mais detalhes informando certos parâmetros, que esclarecem os planos de 
execução avaliados e o plano vencedor:
* `"queryPlanner"`: exibe o resumo sobre a execução e o plano vencedor;
* `"executionStats"`: adicionalmente exibe os planos rejeitados;
* `"allPlansExecution"`: adicionalmente detalhe o processo de decisão do plano de execução.

--> É interessante avaliar alguns dados em uma consulta de maneira a identificar o contexto mais performático de execução. 
É interessante observar:
* Tempo de execução em IXSCAN e COLLSCAN;
* Total de chaves examinadas;
* Total de documentos examinados;
* Total de documentos retornados.

# 014 Understanding Covered Queries
--> Por vezes, os dados requeridos por uma consulta conseguem ser cobertos integralmente por um index, não sendo feito 
acesso aos documentos da coleção, o que é bastante performático:
 ```javascript
db.persons.createIndex({name: 1})
db.persons.find({name: "Max"}, {_id: 0, name: 1})
```
* O index criado sobre `name` armazena todos os valores do campo na coleção;
* `{_id: 0, name: 1}` requerindo somente o campo `name`, os documentos não precisam ser acessados.
* Nessa situação, a consulta pode ser chamada de `covered query`, o index cobriu as necessidades da consulta;
* Indexes compostos permitem cobrir mais campos.

--> Nem sempre é possível fazer uso de covered queries, todavia é interessante estudar a possibilidade de uso. O comando 
`explain()`, exibindo o total de documentos avalidados, permite aferir quando o caso ocorre.

# 015 How MongoDB Rejects a Plan
--> Antes de executar uma consulta, internamente MongoDB identifica as estratégias possíveis com base nos campos filtrados 
e indexes presentes. 
* As estratégias são postas a prova em um conjunto pequeno de documentos de maneira que a performance de cada uma seja 
mensurada;
* Com a melhor estratégia aferida no teste em mãos, é então registrado em um cache que, para a consulta requerida, e com 
os parâmetros informados, a mesma deve ser utilizada;
* O cache das melhores estratégias é então utilizado nas próximas consultas iguais a anterior.

--> Este cache de estratégias sofre expiração quando nos seguintes casos:
* Uma certa quantidade de documentos foi inserida na coleção, usualmente 1000 registros;
* Indexes envolvidos na consulta forma recriados;
* Outros indexes foram criados ou removidos na coleção;
* O servidor MongoDB foi reinicializado.

# 016 Using Multi-Key Indexes
--> Campos do tipo array também podem ser indexados por meio do tipo de index chamado `Multi-Key`. Neste tipo de index, os 
valores do array de todos os documentos se tornam registros no index. O que deve ser levado em consideração em coleções 
onde os arrays apresentam grande quantidade de elementos.
 ```javascript
db.persons.createIndex({ favoriteBooks: 1 })
db.persons.find({ favoriteBooks: "1984" })
```
* Levando em consideração que `favoriteBooks` é um array;
* É possível verificar com `explain()` se o index é utilizado e se o mesmo é um Multi-key.

--> Quando um campo do tipo array contém documentos embutidos, o index pode ser criado e utilizado de duas maneiras:
* O index pode ser criado para comparar todo o documento embutido como um todo:
```javascript
db.persons.createIndex({ friends: 1 })
db.persons.find({ friends: { name: "Ana", age: "42" } })
```
* O index pode ser criado para comparar campos específicos do documento embutido:
```javascript
db.persons.createIndex({ "friends.name": 1 })
db.persons.find({ "friends.name": "Ana" })
```

--> Indexes `Multi-Key` devem ser utilizados com cautela pois afetam ainda mais a performance das demais operações, 
dependendo da quantidade de valores envolvida nos arrays e coleções.

# 017 Understanding Text Indexes
--> A busca por parte de um campo textual não precisa ser feita com o operador `$regex`, qual enfreta problemas de 
performance dependente das dimensões dos valores e da coleção. MongoDB oferece um tipo de index textual que permite 
trabalhar com buscas em texto de forma mais eficiente.
```javascript
db.movies.createIndex({ sumary: "text" })
```
* Ao contrário de especificar a ordem do index, é informado `"text"`.

--> Internalmente o index é criado colocando cada palavra significante do texto, ignorando preposições por exemplo, 
como um registro no index. E por motivos de performance, só existe um único por coleção.

--> O efetivo uso do text index da coleção só é efeito com os operadores `$text` e `$search`:
```javascript
db.movies.find({ $text: { $search: "airplane airport" } })
```
* Busca os documentos que possuem as palavras `airplane` ou `airport`;
* Para busca uma sentença completa, é necessário envolver as palavras com aspas: `"\"airplane airport\""`

# 018 Text Indexes & Sorting
--> A busca textual oferece adicionalmente uma pontuação `score` atribuida a cada documento expressando o quão acertivo 
é o seu contendo em relação ao termo buscado.
```javascript
db.movies.find({ $text: { $search: "airplane airport" }}, {
    scope: { $meta: "textScore" }
}).sort({ score: { $meta: "textScore" }})
```
* Documentos que tiverem as palavras `airplane` e `airport` recebem pontuação maior em relação a outros que possuam 
apenas uma das duas, por exemplo;
* `sort()` permite ordenar os resultados pelo score calculado.

# 019 Creating Combined Text Indexes
--> Para que o text index de uma coleção abrange mais que um campo, é necessário criar o index com todos eles, pois não
é possível adicionar mais campos após a criação já ter sido feita.
```javascript
db.movies.createIndex({ sumary: "text", review: "text" })
```
* Cria o text index da coleção com o conjunto de valores dos campos `sumary` e `review`.

# 020 Using Text Indexes to Exclude Words
--> Adicionalmente pode ser especificado na busca textual a exclusão de termos:
```javascript
db.movies.find({ $text: { $search: "airplane -airport" } })
```
* Busca os documentos que possuam a palavra `airplane` e não possuam `airport`;
* O caracter `-` identifica palavras excluídas.

# 021 Setting the Default Language & Using Weights
--> A otimização na escolha das palavras que formaram o text index da coleção depende da língua. Na criação do index é 
possível espeficiar a linguagem, que por padrão é english, com parâmetro `default_language`:
```javascript
db.movies.createIndex({ sumary: "text" }, { default_language: "german" })
```
* Os valores para o parâmetro são específicos e devem ser consultados;

--> É possível definir um index que abranga múltiplas línguas. Neste caso, na busca deve ser especificado com `$language`:
```javascript
db.movies.find({ $text: { $search: "airplane -airport", $language: "german" } })
```

--> É possível determinar pesos para os campos que formaram o text index, de maneira que o `score` de cada documento seja 
balanceado segundo os mesmos:
```javascript
db.movies.createIndex({ sumary: "text", review: "text" }, {
    weights: { sumary: 2, review: 1 }
})
```
* O campo `sumary` possui um peso duas vezes maior que `review`.

# 022 Building Indexes
--> Por padrão a criação de indexes bloqueia o acesso a coleção, o que pode se mostrar um problema quando o processo demanda 
tempo devido as dimenções dos dados tratados em um ambiente de produção. Para evitar isso, é possível determinar que a 
execução do procedimento de criação seja executado internamente em segundo plano no servidor do MongoDB com `background`:
```javascript
db.movies.createIndex({ sumary: "text", review: "text" }, { background: true })
```
* Em contrapartida, o processo pode demandar mais tempo.

--> É possível executar arquivos javascript com o cliente de linha de comando `mongo` de maneira a automatizar tarefas.
* Conteúdo de um arquivo `tarefa.js`
```javascript
conn = new Mongo();
db = conn.getDB("credit");

for (let i = 0; i < 1000000; i++) {
    db.ratings.insertOne({ "person_id": i + 1, "score": Math.random() * 100, "age": Math.floor(Math.random() * 70) + 18  })
}
```
* Execução do arquivo:
`mongo tarefa.js`
