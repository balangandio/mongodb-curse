# Cap 15 - Performance, Fault Tolerancy & Deployment

# 002 What Influences Performance
--> A performance do servidor MongoDB é principalmente afetada por:
* Eficiência de consultas e operações;
* Uso de indexes;
* Organização do schema;
* Hadware e rede;
* Sharding e Replica Sets.

# 003 Understanding Capped Collections
--> MongoDB oferece uma opção `capped` na criação de coleções `db.createCollection()` que permite especificar um 
limite para o total de dados presente na coleção, de maneira que quando o limite é atingido, o dado mais antigo 
inserido é removido:
```javascript
db.createCollection("capped", { capped: true, size: 10000, max: 3 })
```
* Quando o 4º documento é inserido, o 1º é removido;
* `size` especifica o limite de bytes da coleção;
* `max` especifica o limite de documentos da coleção.

--> Coleções `capped` possuem a particularidade de que a ordem em que os documentos são recuperados corresponde 
extamente a ordem que os mesmos foram inseridos. É possível obter a ordem versa usando o operador `$natural`:
```javascript
db.capped.find({}).sort({ $natural: -1 })
```

--> O uso de coleções `capped` possui a vantagem performática sobre a implementação do mesmo comportamento em 
coleções comuns, de que a operação de remoção é administrada pelo servidor e não possui o custo do envio de 
solicitações de exclusão adicionais.

# 004 What are Replica Sets
--> Replica Sets constitue um mecanismo de balanceamento de carga no MongoDB. Por padrão a MongoDB considera 
a existência de um `node` principal, que seria a base de dados da própria máquina no qual a instalação foi feita. 
Todavia, é possível adicionar indefinidos nodes distribuídos pela rede de maneira que sempre que é solicitado 
a inserção de novos dados, é efetuado assincronamente a replica dos mesmos pelo Replica Set.
* Como os dados são replicados em máquias distintas, o mecanismo naturalmente uma implementação de backups;
* É também um mecanismo de redundância, pois caso o node principal venha a ficar indisponível, o trabalho pode 
ser repassado para os demais;
* Caso seja configurado, a carga de consultas pode ser balanceada no Set de maneira a performance não seja afetada 
em um ambiente de alta demanda.

# 005 Understanding Sharding
--> Sharding constitue um mecanismo de balanceamento de carga no MongoDB que, diferentemente de Replica Sets, os dados 
não são replicados mas sim divididos entre outros servidores na rede, chamados de `shard`. O cliente passa então a 
se comunicar com um roteador `mongos`, que se encarrega de encaminhar a solicitação de consulta ou inserção para o 
devido `shard` na rede.

--> Para que o roteador consiga desempenhar seu trabalho de maneira eficiente, é necessário configurar uma estratégia 
de distribuição do dado segundo algum critério, como por exemplo, cada `shard` se encarrega de parte do alfabeto da 
inicial de um campo `name`. 
* Cada `shard` teria então uma `shard key` que seria calculada pelo roteador quando a inserção de um novo dado é 
solicitada, ou quando é feito a consulta pelo determinado campo, de maneira a contactar apenas o `shard` responsável;
* Quando o roteador não consegue precisar para qual `shard` a solicitação deve ser encaminhada, ele realiza o broadcast 
para todos na rede e faz o merge das respostas. O que no final impacta na performance para o cliente.

# 006 Deploying a MongoDB Server
--> MongoDB `Atlas` é um serviço na núvem que oferece uma infraestrura MongoDB gerenciada. Toda a implementação de 
requisitos de performance e segurança são abstraídos pelo serviço, mas que ainda sim nos permite fazer uso de recursos 
mais avançados como shards e replica sets.

--> É bastante recomendado o uso de um serviço gerenciado como `Atlas` caso a administração de um servidor MongoDB não 
seja exatamente o proprósito do seu uso.

--> Para se conectar a um servidor remoto como o disponibilizado pelo `Atlas` com o cliente `mongo`, especificamos um 
uma URL de conexão:
```
mongo "mongodb+src://server.addr.com/test" --username human
```
* `server.addr.com` é o endereço do servido, `test` o database de autenticação e `human` o nome do usuário;
* Será pedido a senha assim que o comando é executado.