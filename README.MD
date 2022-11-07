# Projeto com passo a passo para Criar um AWS SAM
>**OBS :** Todos os processos serão feito no próprio console da AWS, lembrando que o console da AWS está em constante atualizações, podendo mudar o layout porém os conceitos permanecem.

## Finalidade do projeto

Esse projeto tem um intuito de orientar na criação de um Serverless Application Model (SAM) ou seja, um aplicativo sem uso de um servidor. Onde explicará de forma mais prática e detalhada possível a usar os serviços da AWS Lambda, DynamoDB para criação do banco e dados e o serviço API Gateway para o gerenciamento da api realizando CRUD, de uma forma mais prática e sequêncial possível.

### Para esse projeto será necessário o uso dos seguintes serviços da AWS :
---
* DynamoDB
* Lambda
* API Gateway
---
### Conceitos dos Serviços AWS:

### DynamoDB      
![Icon-DynamoDB](img\icon-DynamoDb.png) 
O Amazon DynamoDB é um banco de dados NoSQL que suporta modelos de dados de valores-chave e de documentos. Os desenvolvedores podem usar o Amazon DynamoDB para criar aplicações modernas sem servidor, que podem começar pequenas e serem dimensionadas globalmente para oferecer suporte a petabytes e dezenas de milhões de solicitações de leitura e gravação por segundo.

### Lambda

![Lambda](img\Lambda_resized.png) 
O AWS Lambda é um serviço computacional sem servidor que executa código em resposta a eventos e gerencia automaticamente os recursos computacionais subjacentes para você. Esses eventos podem incluir mudanças no estado ou uma atualização, como um usuário colocando um item em um carrinho de compras de um site de comércio eletrônico por exemplo. 

### API Gateway

![Lambda](img\API-Gateway_resized.png) 
O Amazon API Gateway é um serviço da AWS para criação, publicação, manutenção, monitoramento e proteção de APIs REST e WebSocket em qualquer escala. Os desenvolvedores de API podem criar APIs que acessem a AWS ou outros web services, bem como dados armazenados na Nuvem AWS.


## Inicializando o banco de dados não relacional com o serviço DynamoDB


**I. Pesquisar o serviço no console da AWS**

![Console-aws](img\Console-AWS-DynamoDB.png) 
1. Buscar o serviço no console.  

2. Escolher o serviço

3. N.Virginia é a região que será hospedado este serviço. 

> Todos os serviços serão feito na região da N.Virginia

![Get Started-DynamoDB](img\GetStart-Dynamodb.png) 

**II. Criação da tabela**

1. Nomear um nome para a tabela.

2. Partition Key e o campo da chave primária, onde se pode nomear este campo com nome **id** do tipo **String**.
> 🔑  Não existe chave com auto-incremento na tabela não relacional
3. Clicar no botão Create table

![Get Started-DynamoDB](img\Create-Table-DynamoDB.png)

**III. Acessar o menu lateral Tables**  

Clicar na tabela tbl-person

![Get Started-DynamoDB](img\acess-table-DynamoDB.png)

**IV Clique no botão Explore table items**

Detalhamento sobre a tabela. ![Get Started-DynamoDB](img\Explore-Table-items.png)

Caso queira pode nomear uma tag para tabela, seja criando um próprio nome ou com tag's já definidas como Aplicação, Homologação.
Em Items returned está vazio pois não foi criado nenhum campo na tabela.

![Get Started-DynamoDB](img\items-returned-DynamoDb.png)

## Inicializando o serviço Lambda

**I. Pesquisar o serviço no console da AWS**

1. Buscar o serviço no console.  
2. Escolher o serviço
3. N.Virginia é a região que será hospedado este serviço. 

![Get Started-DynamoDB](img\console-aws-Lambda.png)

**II. Clicar no botão Create Function**

![Get Started-DynamoDB](img\Lambda-functions.png)

**III. Criando uma Função Lambda**

1. Nomeando uma função Lambda
2. Escolhendo uma linguagem de programação e sua versão, para criar o código da função com os endpoints na linguagem escolhida.

![Get Started-DynamoDB](img\Create-Lambda.png)

**IV. Criando e configurando uma Role**

Para criar uma Role com policy da própria AWS:

**V. Selecionar a opção**

  Create a new role from AWS policy template

- [ ] Create a new role with Lambda permissions 
- [ ] Use an existing role

 ✅ Create a new role from AWS policy template

2. Criando um nome para Role
3. Colocando uma Policy padrão **Simple microservice permissions**
4. Clicar em Create Function

![Get Started-DynamoDB](img\lambda-roles.png)

**VI. Criar um código para o CRUD em JS**

Esse código é na linguagem que foi escolhida na hora da criação da função Lambda.

<details>
<summary>
Código fonte da função Lambda
</summary>

```
const AWS = require("aws-sdk");

const dynamo = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event, context) => {
  let body;
  let statusCode = 200;
  const headers = {
    "Content-Type": "application/json"
  };

  try {
    switch (event.routeKey) {
      case "DELETE /items/{id}":
        await dynamo
          .delete({
            TableName: "tbl-person",
            Key: {
              id: event.pathParameters.id
            }
          })
          .promise();
        body = `Deleted item ${event.pathParameters.id}`;
        break;
      case "GET /items/{id}":
        body = await dynamo
          .get({
            TableName: "tbl-person",
            Key: {
              id: event.pathParameters.id
            }
          })
          .promise();
        break;
      case "GET /items":
        body = await dynamo.scan({ TableName: "tbl-person" }).promise();
        break;
      case "PUT /items":
        let requestJSON = JSON.parse(event.body);
        await dynamo
          .put({
            TableName: "tbl-person",
            Item: {
              id: requestJSON.id,
              name: requestJSON.name,
              cpf: requestJSON.cpf,
              sexo: requestJSON.sexo,
              altura: requestJSON.altura,
              nacionalidade: requestJSON.nacionalidade
            }
          })
          .promise();
        body = `Put item ${requestJSON.id}`;
        break;
      default:
        throw new Error(`Unsupported route: "${event.routeKey}"`);
    }
  } catch (err) {
    statusCode = 400;
    body = err.message;
  } finally {
    body = JSON.stringify(body);
  }

  return {
    statusCode,
    body,
    headers
  };
};
```
</details>

Após o código clicar no Deploy.

---- 

Criando uma variável AWS que recebe os módulos de serviços da AWS, com o "aws-sdk".

```const AWS = require("aws-sdk");```

Acessando o banco de dados do DynamoDB e colocando na variável dynamo.

```const dynamo = new AWS.DynamoDB.DocumentClient();```

Exportando a função com um método assincrono 

```exports.handler = async (event, context) => {...}```

## Inicializando o serviço API Gateway

**I. Pesquisar o serviço no console da AWS**

![Get Started-API-Gateway](img\API-Gateway-search.png)

**II. Escolher o tipo de API**

Nesse exemplo será uma HTTP API.

A API sera responsável para disponibilzar um endpoint de HTTP para a função do lambda que foi criada.
![Get Started-API-Gateway](img\HTTP_API-Gateway.png)

**III. Criando a API**

Criando um nome para API

```http-api-person```

Após isso clicar no Next.      

![Get Started-API-Gateway](img\name-api-gateway.png)

**IV. Criando rotas**

>As rota serão inseridas depois

Clicar em Next.

![Get Started-API-Gateway](img\config-routes.png)

**V. Define stages**

Clicar em Next.

![Get Started-API-Gateway](img\define-status-api_Gateway.png)

**VI. Review da API**

![Get Started-API-Gateway](img\Review-Api-Gateway.png)

**VII. Definindo as rotas**

Para definir as rotas tem que ir no menu lateral clicar em Develop e depois em  Routes.

Na Api que você criou clicar em Create.

Para criar cada rota, no caso todas as rotas do CRUD.

![Get Started-API-Gateway](img\Routes-api-gateway.png)

### IX. Criação de rotas Http

**Criando a rota de Delete**

Escolhendo o verbo Http específico e mapeando o caminho do mesmo jeito que está no função Lambda.

![Get Started-API-Gateway](img\route-Delete.png)

**Criando uma rota GET obtendo um ID**

Mapeando o método GET para obter um objeto específico pelo ID.

![Get Started-API-Gateway](img\route-GetById.png)

**Criando uma rota GET obtendo uma Lista de Objetos**

Mapeando o método GET objtendo todos os objetos da tabela tbl-Person.

![Get Started-API-Gateway](img\Get-All.png)

**Criando uma rota PUT**

![Get Started-API-Gateway](img\PUT-items.png)

**Mostrando todas as rotas da API**

![Get Started-API-Gateway](img\Routes-all-api-person.png)

## Fazendo toda a comunicação entre API, função Lambda e o banco DynamoDB

>Tem que realizar a integração da comunicação entre as rotas do Api-Gateway para a rotas da função Lambda, para poder realizar o CRUD com o banco de dados.

![Lambda](img\fluxo-aws-SAM.png) 

Clicando no Integrations no menu lateral, e criando um gerenciamento para a integração.

![Get Started-API-Gateway](img\Integrations-routes.png)

Criando um tipo de integração, escolhendo uma integração Lambda Function.

![Get Started-API-Gateway](img\create-integration.png)

**Detalhes de Integração**

Associando uma função lambda com a integração da API que foi criada.

escolhendo a região.
Depois clicar em Next.

![Get Started-API-Gateway](img\integrations-details.png)

### Integração dos métodos Http 

>Associando todos os caminhos dos verbos Http

**O método Put Integrado**

Clicando no Attach integrations to routes.

Clicando no método PUT, e escolhendo a função lambda para integrar o mapeamento.

![Get Started-API-Gateway](img\integration-route-PUT.png)

Método integrado e mapeado com a função lambda

![Get Started-API-Gateway](img\PUT-aws_lambda.png)

Fazer o mesmo processo com todos os outros métodos.

Todos os métodos mapeados e integrados, ficam assim :

 ![Get Started-API-Gateway](img\review-integrations.png)

Para acessar a URL da API, tem que ir no menu lateral e clicar em API: nome-sua-API

![Get Started-API-Gateway](img\URL-Api_Gateway.png)

## ⚡ Os testes foram feitos com o Software Postman ⚡

* PUT

Adicionando um objeto na tabela tbl-person.
>Colocando no Postman a URL da API/caminho dos métodos, que está na API Gateway de cada verbo Http.
Todos os dados estão no arquivo **Dados_tbl-person-JSON.**

![Get Started-API-Gateway](img\Postman-PUT.png)

* GET

Obtendo um registro da tabela tbl-Person pelo ID.

![Get Started-API-Gateway](img\Postman-GET-ID.png)

* GET-ALL

Listando todos os dados na tabela tbl-person, após adicionar os objetos na tabela.

![Get Started-API-Gateway](img\Postman-GET-ALL.png)

* DELETE

Deletando um objeto da tabela tbl-person, pelo seu ID.

![Get Started-API-Gateway](img\Postman-DELETE.png)

## Demonstrando os dados na tabela DynamoDB ![Get Started-API-Gateway](img\database.png)

Demonstrando a tabela tbl-person sendo povoada, pelo Postman.

![Get Started-API-Gateway](img\intems-returned-Dynamo.png)

Com isso todo o sistema está finalizado.