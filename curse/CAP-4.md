# Cap 4 - Exploring The Shell & The Server

# 003 Setting dbpath & logpath
--> O servidor MongoDB inicializado com o executável `mongod`, recebe os parâmetros `--dbpath` e `--logpath` 
que respectivamente definem o diretório onde os arquivos contendo os dados são armazenados, e o diretório 
onde os arquivos de log são escritos.
```
mongod --dbpath /opt/mongo/files --logpath /opt/mongo/logs
```
* Por padrão o diretório dos arquivos de dados é `/db`.

# 005 MongoDB as a Background Service
--> O servidor pode ser inicializado em background fazendo uso do parâmetro `--fork` e obrigatoriamente 
definindo o diretório para a escrita de logs:
```
mongod --fork --logpath /opt/mongo/logs
```
* Em sistemas Windows o parâmetro não é disponível, é feito o uso de serviços do sistema;
* É possível interromper o servidor do MongoDB executando o comando:
```javascript
db.shutdownServer()
```

# 006 Using a Config File
--> É possível definir um arquivo contendo as configurações de inicialização do servidor e fornece-lo ao 
inicializador com o parâmetro `-f`:
```
mongod -f /opt/mongo/config.conf
```
* Exemplo de um arquivo definindo path do log e do dados:
```
storage:
  dbPath: "/your/path/to/the/db/folder"
systemLog:
  destination:  file
  path: "/your/path/to/the/logs.log"
```