# Cap 9 - Understanding Delete Operations

# 002 Understanding deleteOne() & deleteMany()
--> `deleteOne()` permite remover o primeiro documento compatível com um critério de busca informado.
```javascript
db.collection.deleteOne({ name: "human creature" })
```

--> `deleteMany()` permite remover todos os documentos compatíveis com um critério de busca informado.
```javascript
db.collection.deleteMany({ type: "animal" })
```

# 003 Deleting All Entries in a Collection
--> A deleção de uma coleção pode ser feita com `drop()`:
```javascript
db.collection.drop()
```

--> A deleção de um base de dados pode ser feito com `dropDatabase()`:
```javascript
db.dropDatabase()
```