# Cap 14 - MongoDB & Security

# 001 Module Introduction
--> É preciso levar em consideração alguns pontos chaves quando pretendemos manter um ambiente segundo no MongoDB:
* Autenticação e autorização: controle sobre usuários e suas permissões;
* Criptografia no transporte: criptografia na conexão entre cliente e servidor MongoDB;
* Criptografia dos dados: criptografia dos dados no armazenamento;
* Auditoria: manutenção de registros das operações realizadas;
* Segurança da rede: segurança da infraestrutura de rede que conecta cliente e servidor;
* Backups e atualizações de software: manutenção de rotinas de backups e atualização do servidor.

# 002 Understanding Role Based Access Control
--> Para um usuário interagir com o servidor MongoDB, necessariamente ele precisa se autenticar fornecendo suas credenciais. 
Todavia, após autenticado não poderá interagir com nenhum recurso na base de dado até que seja a ele atribuído algum `privilege`.
* Um `privilege` consiste de uma `action` autorizada a ser executado em um `resource`;
* Um conjunto de `privileges` podem ser agrupados em uma `role`, que por sua vez pode ser atribuída a usuários.

# 003 Roles - Examples
--> Tipicamente, podemos ter as seguintes `roles`:
* Administradores: privilégios sobre ações de manutenção da base dados;
* Desenvolvedores / aplicações: privilégios sobre ações de interação e inserção de dados;
* Analístas: privilégios sobre ações de consulta de dados.

# 004 Creating a User
--> Usuários são criados por usuários existentes que possuam privilégios `createUser()`. Necessarimanete é especificado um 
banco de dados que deverá ser informado no processo de autenticação. Todavia isso não significa que ele esteja limitado a 
interagir com os dados do mesmo, pois privilégios de acesso a outros bancos podem ser atribuídos. Já a edição de usuários, 
é feita com com o privilégio `updateUser()`.

--> A autenticação com o cliente de linha de comando, pode ser feita informando as credenciais na execução:
```
mongo -u user -p pass --authenticationDatabase admin
```
* O parâmetro `--authenticationDatabase` não precisa ser informado numa instação recém feita.

--> Posteriormente, a autenticação pode pode ser feita também após o processo de conexão com o comando `db.auth()`:
```javascript
db.auth('username', 'userpass')
```

--> Quando o servidor MongoDB é pela primeira vez iniciado, ele não possui usuários. Neste caso, é excepcionalmente permitido 
executar o comando `createUser()` sem efetuar autenticação quando a conexão é feita pelo endereço `localhost`. MongoDB oferece 
uma `role` já predefinida para o primeiro usuário, a administração total `userAdminAnyDatabase`:
```javascript
db.createUser({ user :'superAdmin', pwd: 'superPassword', roles: ["userAdminAnyDatabase"]})
```
* Após o usuário administrador ser criado, a autenticação passa a ser obrigatória.

# 005 Built-In Roles - An Overview
--> MongoDB contém um conjunto de `roles` já predefinidas na instalação.
* Database User - roles usualmente atribuídas a usuários;
```
read
readWrite
```
* Database Admin - roles usualmente atribuídas a administradores;
```
dbAdmin
userAdmin
dbOwner
```
* All Database Roles - roles atribuídas usuários com acesso a qualquer database;
```
readAnyDatabase
readWriteAnyDatabase
userAdminAnyDatabase
dbAdminAnyDatabase
```
* Cluster Admin - roles atribuídas a administradores de clusters;
```
clusterManager
clusterMonitor
hostManager
clusterAdmin
```
* Backup/Restore - roles atribuídas a usuários com tarefas de backup e restauração;
```
backup
restore
```
* Superuser - roles atribuídas a administradores do servidor;
```
dbOwner (admin)
userAdmin (admin)
userAdminAnyDatabase
root
```

# 006 Assigning Roles to Users & Databases
--> Para determinar na criação de um usuário uma role em determinado database, deve ser realizada a troca do contexto com `use`:
```javascript
use shop
db.createUser({user: 'appdev', pwd: 'pass', roles: ["readWrite"]})
```
* Cria uma usuário com a role `readWrite` no database `shop`;
* O database especificado na criação será o database que o mesmo deve informar na sua autenticação.

--> Para efetuar a autenticação com o usuário recém criado no cliente de linha de comando, é necessario deslogar do usuário corrente, 
o que pode ser feito com o comando `db.logout()`:
```javascript
db.logout()
use shop
db.auth('appdev', 'pass')
```

--> Para efetuar a autenticação com o usuário pela execução do cliente, é necessário especificar o database de autenticação:
```
mongo -u appdev -p pass --authenticationDatabase shop
```
* Uma vez autenticado, o usuário ainda deve trocar o contexto para o database `use shop` no qual o mesmo possui acesso.

# 007 Updating & Extending Roles to Other Databases
--> Para atribuir roles a um usuário para outro database que não o seu de autenticação, o comando `db.updateUser()` pode ser utilizado 
redefinindo o conjunto de roles do mesmo.
```javascript
use shop
db.updateUser('appdev', { roles: ["readWrite", {role: "readWrite", db: "blog"}] })
```
* É definido a role `readWrite` implicitamente em `shop`, e é definido `readWrite` explicitamente em `blog`;
* A trocar do contexto com `use` é necessária pois o usuário existe apenas no seu database de autenticação.

--> Para consultar o conjunto de roles do usuário, o comando `db.getUser()` pode ser utilizado:
```javascript
db.getUser("appdev")
```

# 009 Adding SSL Transport Encryption
--> Para fazer uso de SSL é necessário antes de tudo gerar as chaves e certificados, o que pode ser feito com a ferramenta `openssl`:
```
openssl req -newkey rsa:2048 -new -x509 -days 365 -nodes -out mongodb-cert.crt -keyout mongodb-cert.key
```
* No processo de geração, será pedido algumas informações que constarão no certificado. Para `Common Name` deve ser informado o 
endereço host no qual o cliente fará a validação se o mesmo está presente no certificado enviado pelo servidor. Caso a conexão seja 
feita localmente, pode ser informado `localhost`

--> A criptografia das conexões com o servidor pode ser habilitada na inicialização do mesmo:
```
cat mongodb-cert.key mongodb-cert.crt > mongodb.pem
mongod --sslMode requireSSL --sslPEMKeyFile mongodb.pem
```
* A chave e certificado gerados são concatenados no arquivo `mongodb.pem`;
* `--sslMode requireSSL` determina que todas as conexão necessariamente deve utilizar SSL, o que poderia ser opcional;
* Adicionalmente o parâmetro `--sslCAFile` poderia informar o certificado de autoridade adquirido com uma entidade de registro.

--> Para efetuar a conexão com o cliente `mongo`:
```
mongo --ssl --sslCAFile mongodb.pem --host localhost
```
* O mesmo arquivo `mongodb.pem` do servidor é informado no cliente;
* O parâmetro `--host` especifica o `Common Name` informado na geração do certificado.

# 010 Encryption at REST
--> A criptografia dos dados mantidos pelo banco de dados por ser feita em dois níveis:
* Criptografia no sistema de arquivo, disponível na linçensa Enterprise;
* Criptografia em campos específicos dos documentos, implementável pelo usuário.