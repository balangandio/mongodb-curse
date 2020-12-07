# Cap 13 - Working with Numeric Data

# 002 Number Types - An Overview
--> Existem basicamente quatro tipos de dados numéricos no MongoDB:
* Integers (int32) - valores inteiros comuns;
* Longs (int64) - valores interiros longos;
* Doubles (64bit) - valores decimais com grande numeral mas com baixa precisão decimal;
* High Precision Doubles (128bit) - valores decimais com pequeno numeral mas com alto precisão decimal.

# 004 Understanding Programming Language Defaults
--> Valores numéricos são interpretados segundo o ambiente em que são escritos. Como o cliente de linha de comando do MongoDB é 
baseado em javascript, valores numéricos são vistos como Double pelo driver. Neste caso, a criação de valores inteiros precisar 
ser explicitamente feita.

# 005 Working with int32
--> A instância de valores int32 pode ser feito com `NumberInt`:
```javascript
db.collection.insertOne({ name: "human", age: NumberInt("42") })
```
* Caso o número extrapole o range de um int32, não é lançado nenhum erro, todavia um valor totalmente diferente é armazenado.

# 006 Working with int64
--> A instância de valores int64 pode ser feito com `NumberLong`:
```javascript
db.collection.insertOne({ name: "human", age: NumberLong("42") })
```

--> Os objetos podem ser criados sem o uso de string `NumberLong(42)`, todavia essa maneira é potencialmente limitada pelo 
ambiente utilizado. No javascript, como literais numéricos são double, não é possível representar o maior valor de um int64, 
ocorre um erro.

# 007 Doing Maths with Floats int32s & int64s
--> Operações matemáticas podem causar a alteração do tipo de dado dos documentos quando os valores envolvidos não possuem o 
mesmo tipo:
```javascript
db.collection.insertOne({ name: "human", age: NumberInt("42") })
db.collection.updateOne({}, { age: {$inc: 1} })
```
* O campo `age` passa a armazenar o valor 43, todavai com o tipo double, pois a operação `$inc` envolvendo o double `1`;
* Caso a operação seja feita em um valor grande do tipo NumberLong, o resultado seria um valor incorreto devido ao resultado 
da operação extrapolar o range de double.

# 008 What's Wrong with Normal Doubles
--> A limitação de valores Double não chega a ser um problema enquanto que eles são apenas armazenados e recuperados. Todavia 
quando operapções matemáticas são feitas envolvendo decimais, a imprecisão se torna evidente:
```javascript
db.products.insertOne({ desc: "human brain", current: 0.3, previous: 0.1 })
db.products.aggregation({ project: {newPrice: { $subtract: ["$current", "$previous"]}} })
```
* `newPrice` não é igual a `0.2`, mas sim uma aproximação como `0.199999998`.

# 009 Working with Decimal 128bit
--> A instância de valores decimais de alta precisão pode ser feito com `NumberDecimal`:
```javascript
db.products.insertOne({ price: NumberDecimal("0.2") })
```

--> Deve-se ter atenção ao operar valores NumberDecimal pois valores double introduzem imprecisão no resultado das operações.
```javascript
db.products.insertOne({ price: NumberDecimal("0.2") })
db.products.updateOne({}, { price: {$inc: 0.1} })
```
* Mesmo que o tipo de dados do campo `price` seja mantido NumberDecimal, o valor resultando da operação ganhará casas adicionais 
da impressão do double `0.1`.