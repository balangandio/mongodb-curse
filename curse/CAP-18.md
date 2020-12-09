# Cap 18 - Introducing Stitch

# 002 What is Stitch
--> Stitch é uma plataforma serverless oferecida pela MongoDB, que permite construir serviços sem se preocupar 
com a implementação de infraestrutura envolvendo aplicações web. Dentre recursos disponíveis pela plataforma, 
se destaca:
* Banco de dados na núvem Atlas;
* Autenticação de usuários;
* Responsividade a eventos com Stitch Triggers;
* Execução de código sem servidor com Stitch Functions e Stitch Services.

# 004 Start Using Stitch
--> Para fazer uso de Stitch, a aplicação frontend precisa de um SDK. No caso de uma aplicação javascript como 
React, a instalação pode ser feita com :
```
npm install --save mongodb-stitch-browser-sdk
```

# 005 Adding Stitch to our App & Initializing It
--> Stitch oferece um cliente MongoDB remoto que pode ser utilizado da seguinte maneira:
```javascript
import { Stitch, RemoteMongoClient, AnonymousCredential } from 'mongodb-stitch-browser-sdk';

const client = Stitch.initializeDefaultAppClient('myshop-nzhmm');
client.auth.loginWithCredential(new AnonymousCredential());

const mongodb = Stitch.defaultAppClient.getServiceClient(RemoteMongoClient.factory, 'mongodb-atlas');
const db = mongodb.db('shop');

const products = await db.collection('products').find().asArray();
```
* `loginWithCredential()` efetua a autenticação do usuário na plataforma Stitch. `AnonymousCredential` 
permite efetuar a autenticação sem credenciais, o que precisa ser habilitado na plataforma;
* Stitch estabele que por princípio os usuários não tem nenhum nível de acesso. Para definir que a coleção 
`products` seja acessível, é necessário que assim seja configurar nas `Rules` da pltaforma.

# 007 Sending Data Access Rules
--> As `Rules` do Stitch define permições de acesso para coleções e campos, assim como validações de schema 
para os dados.

# 009 Deleting Products
--> Os IDs do MongoDB também são representados de maneira especial por objetos. Na aplicação frontend, o pacote 
`bson` pode ser utilizado para instância dos mesmos. 
```javascript
import BSON from 'bson';

const mongodb = getRemoteMongoClient();
const db = mongodb.db('shop');

const result = await db.collection('products').deleteOne({ _id: new BSON.ObjectID('...') });
```
* Instalação: `npm install --save bson`

# 010 Finding a Single Product
--> Decimais de alta precisão podem ser instanciados com `bson` também:
```javascript
import BSON from 'bson';

const mongodb = getRemoteMongoClient();
const db = mongodb.db('shop');

const newProduct = {
    name: 'human fear',
    price: BSON.Decimal128.fromString('42.1')
};

const result = await db.collection('products').insertOne(newProduct);
```

# 014. Adding User Sign Up & Confirmation
--> Após habilitar a autenticação por e-mail e senha, o registro de um usuário pode ser feito com:
```javascript
import { Stitch, UserPasswordAuthProviderClient } from 'mongodb-stitch-browser-sdk';

const client = Stitch.initializeDefaultAppClient('myshop-nzhmm');
const emailPassClient = client.auth.getProviderClient(UserPasswordAuthProviderClient.factory);

const userCredential = { email: '<useremail>', pass: '<userpass>' };

const result = await emailPassClient.registerWithEmail(userCredential.email, userCredential.pass);
```
* Será enviado um e-mail para o usuário com um link de confirmação. Este link é configurado na 
plataforma e conterá parâmetros na query da URL referentes ao token de confirmação.

--> Quando o usuário acionar o link de confirmação (ex.: `http://myapp/confirm?token=<token>&tokenId=<tokenId>`), 
a aplicação deve coletar o token da URL nos parâmetros `token` e `tokenId`, e então solicitar a confirmação 
do registro ao Stitch:
```javascript
import { Stitch, UserPasswordAuthProviderClient } from 'mongodb-stitch-browser-sdk';

const token = getQueryParam('token');
const tokenId = getQueryParam('tokenId');

const emailPassClient = Stitch.defaultAppClient.auth.getProviderClient(UserPasswordAuthProviderClient.factory);

const result = await emailPassClient.confirmUser(token, tokenId);
```

# 015 Adding User Login
--> Para efetuar o login de um usuário:
```javascript
import { Stitch, UserPasswordCredential } from 'mongodb-stitch-browser-sdk';

const userCredential = new UserPasswordCredential('<useremail>', '<userpass>');

const client = Stitch.initializeDefaultAppClient('myshop-nzhmm');

const result = await client.auth.loginWithCredential(userCredential);
```
* O token que confirma que o usuário está logado é armazenado no localstore do browser e utilizado nas solicitações 
subsequentes.

# 018 Functions & Triggers
--> Uma vez criada na plataforma, uma function pode ser acionada pela aplicação da seguinte maneira:
```javascript
import { Stitch, UserPasswordCredential } from 'mongodb-stitch-browser-sdk';

const client = Stitch.initializeDefaultAppClient('myshop-nzhmm');

client.callFunction('MyStartupFuncion', ['paramString']);
```
* `callFunction()` identifica a função a ser chamada e adicionalmente seus parâmetros;
* A mesma função pode ser executada pela plataforma por um trigger como a inserção de um documento.