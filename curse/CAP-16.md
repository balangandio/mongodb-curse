# Cap 16 - Transactions

# 002 What are Transactions
--> Transações agrupam um conjunto de operações que potencialmente alteram o estado do banco de dados, e cujo 
sucesso não pode ser parcial. Ou todo o conjunto de operações são executadas sem problemas, ou caso alguma não 
consiga por algum motivo, o estado do banco de dados no momento anterior à transação deve ser preservado.

# 004 How Does a Transaction Work
--> Um exemplo de transação:
```javascript
const sessin = db.getMongo().startSession()

const postsColl = session.getDatabase("blog").posts
const usersColl = session.getDatabase("blog").users

session.startTransaction()
userColl.deleteOne({ name: 'human' })
postsColl.deleteMany({ username: 'human' })
session.commitTransaction()
```
* Transações operam sobre uma sessão, que por sua vez é criada com `db.getMongo().startSession()`;
* A interação com coleções deve ser feita obtendo a referência das mesmas a partir da sessão `session.getDatabase("blog")`;
* `session.startTransaction()` sinaliza que as operações registradas na sessão fazem parte de uma transação;
* `session.commitTransaction()` sinaliza que as operações registradas podem ser agora efetuadas;