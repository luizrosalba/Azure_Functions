# Projeto Construindo sexy APIs usando arquitetura serverless
## Projeto constituinte do Bootcamp Desenvolvimento Node.js da Digital Innovation One 
Nesse desafio desenvolvemos e entregamos um projeto de “APIs para Gestão de Produtos utilizando Node.js” no qual  qual praticamos e aplicamos os conceitos de desenvolvimento de APIs e Arquitetura Serverless com Node.js.


## Serverless 

Exemplo : Um server rodando em node.js onde você se preocupa prioritariamente 
com a rota a ser construída 

Vantagens : 
- Roda uma função 
- Paga-se o quante se usa 
- Escalabilidade 
- Facilidade no bootspraping 
- Facilidade de por outros triggers 

## Azures Functions x  AWs 
- AWS mais recursos (Aws lambda , Googleecloud Functions , OpenWhisk... ) 
- Azure mais facil de se trabalhar serverless 

- Serverless é muito usado para disparar triggers quando algo acontece 
- Também para criar APIs 
- listNotes ();
- createNote();
- updateNote();
- deleteNote ();
-  azure functions core tools : 
- https://docs.microsoft.com/pt-br/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#v2
- Extensão do vccode : Azure Functions 
## Criando uma API 
- CRUD de produtos 
- cria uma pasta No H:\DIO
- func init 
- opção 2 node 
- opção 1 javascript 
- criou o dir e o json 
- func new (cria uma nova funcao) 
- template (triggers) - 8 HTTP trigger
- function name : GetProducts
- code .
- sim 
- toda config pronta 
 - alteramos o index.js para : 
```
module.exports = async function (context, req) {
    context.res =  { 
        status:200, 
        body: 'Hello World'
    }
}
```
- no terminal : func host start 
- funciona ! 
- tiramos o metodo post da function.json
- definindo o nome da rota 
```
{
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get"],
      "route": "products"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```
- funciona : http://localhost:7071/api/products
- https://github.com/IgorHalfeld/digital-innovation-one-demo
- copiamos o github para mongoclient.js em shared 
- npm i mongodb
- em index.js 
```
const createMongoClient = require('../shared/mongoClient');

module.exports = async context => {
  const { client: MongoClient, closeConnectionFn } = await createMongoClient();
  const Products = MongoClient.collection('products');
  const res = await Products.find({});
  const body = await res.toArray();
  
  closeConnectionFn();
  context.res = { status: 200, body };
};
```
- http://localhost:7071/api/products retorna os produtos cadastrados no banco que o prof. criou em mongodb+srv://god:dog@cluster0-dfsvs.mongodb.net/dgo?retryWrites=true&w=majority
- trazendo só p id 
- func new 
- 8 HTTP TRIGGER
- GetProductByID
- no functions 
```
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get"],
      "route": "products/{id}"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```
- no index 
```
const { ObjectID } = require('mongodb');
const createMongoClient = require('../shared/mongoClient');

module.exports = async function (context, req) {
  const { id } = req.params;

  if (!id) {
    context.res = {
      status: 400,
      body: 'Provide a product id on params',
    };
    return;
  }

  const { client: MongoClient, closeConnectionFn } = await createMongoClient();
  const Products = MongoClient.collection('products');
  const body = await Products.findOne({ _id: ObjectID(id) });

  closeConnectionFn();
  context.res = { status: 200, body };
};
```
- retorna o produto 
- http://localhost:7071/api/products/5ddb00d8d90791a2afee4055
- Rota para criar produtos 
- func new 
- 8 
 - CreateProduct
- index.js
```
const createMongoClient = require('../shared/mongoClient');

module.exports = async function (context, req) {
  const product = req.body || {};

  if (product) {
    context.res = {
      status: 400,
      body: 'Product is required',
    };
  }

  const { client: MongoClient, closeConnectionFn } = await createMongoClient();
  const Products = MongoClient.collection('products');

  try {
    const products = await Products.insert(product);
    closeConnectionFn();
    context.res = { status: 201, body: products.ops[0] };
  } catch (error) {
    context.res = {
      status: 500,
      body: 'Error on insert product',
    }; 
  }
};
```
- no function.json 
```
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["post"],
      "route": "products"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```
- no postman 
- criar o body raw 
```
{
    "nome" : "curso" , 
    "price": 3 
}
```
-POST  http://localhost:7071/api/products/
- 201 created 
- ao dar get percebemos o produto criado 
```
  {
    "_id": "5f45667fc4d15216d86e6efe",
    "nome": "curso",
    "price": 3
  }
```
- func new 
- 8 
- UpdateProduct
- 
- no index 
```
const { ObjectID } = require('mongodb');
const createMongoClient = require('../shared/mongoClient');

module.exports = async function (context, req) {
  const { id } = req.params;
  const product = req.body || {};

  if (!id || !product) {
    context.res = {
      status: 400,
      body: 'Provide a product and product id on params',
    };
    return;
  }

  const { client: MongoClient, closeConnectionFn } = await createMongoClient();
  const Products = MongoClient.collection('products');

  try {
    const products = await Products.findOneAndUpdate(
      { _id: ObjectID(id) },
      { $set: product },
    );
    closeConnectionFn();
    context.res = { status: 200, body: products };
  } catch (error) {
    context.res = {
      status: 500,
      body: 'Error on insert product',
    }; 
  }
};
```
na function 
```
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["put"],
      "route": "products/{id}"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```
- put http://localhost:7071/api/products/5f45667fc4d15216d86e6efe
com o body raw para atualizar o preco para 4 reais 
```
{
    "nome" : "curso" , 
    "price": 4 
}
```
- funciona ! 
Criando o delete 
- func new 
- 8 
- Delete Product 
no index 
```
const { ObjectID } = require('mongodb');
const createMongoClient = require('../shared/mongoClient');

module.exports = async function (context, req) {
  const { id } = req.params;

  if (!id) {
    context.res = {
      status: 400,
      body: 'Provide a product id on params',
    };
    return;
  }

  const { client: MongoClient, closeConnectionFn } = await createMongoClient();
  const Products = MongoClient.collection('products');

  try {
    await Products.findOneAndDelete({ _id: ObjectID(id) });
    closeConnectionFn();
    context.res = {
      status: 200,
      body: 'Product deleted successfully!',
    };
  } catch (error) {
    context.res = {
      status: 500,
      body: 'Error on delete product ' + id,
    };
  }
};
```
No Functions 
```
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["delete"],
      "route": "products/{id}"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```
- DELETE http://localhost:7071/api/products/5f45667fc4d15216d86e6efe
- produto deletado com sucesso 
- deploy no azure 
- em todas as functions "authLevel": "anonymous", pois só uma function consegue acessar uma function 
- clica com o botao direito sobre o painel do VS onde ficam os arquivos 
- deploy to function app 
- pay as you go  ( funcao ligada quando recebe ) 
- visual studio enterprise ( funcao sempre ligada )
- qeure reescreveR ? 
- coloque o nome da nova funcao 
- digital-innovation-one-api
- cria os resources 
- no painel function apps 
- na funcao que vc criou , no submenu cors 
- allowed Origins * ( libera o acesso pra todos os dominios ) 
- sua api : 
https://digital-innovation-one-api.azurewebsites.net/api/products
- commit 
- publish  azurefunc 
- https://github.com/luizrosalba/azure_func

```
```
