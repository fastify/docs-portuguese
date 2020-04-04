<h1 align = "center">Serverless</h1>

Execute aplicativos serverless e APIs REST usando sua aplicação Fastify existente.

### Conteúdo

- [AWS Lambda](#aws-lambda)
- [Google Cloud Run](#google-cloud-run)
- [Zeit Now](#zeit-now)

### Atenção Leitores:
> O Fastify não foi projetado para ser executado em ambientes sem servidor.
A estrutura Fastify foi projetada para facilitar a implementação de um servidor HTTP/S tradicional.
Ambientes sem servidor solicitam de maneira diferente de um servidor HTTP/S padrão;
portanto, não podemos garantir que funcione conforme o esperado com o Fastify.
Independentemente disso, com base nos exemplos dados neste documento,
é possível usar o Fastify em um ambiente sem servidor.
Novamente, lembre-se de que este não é o caso de uso pretendido do Fastify e
não testamos esses cenários de integração.

## AWS Lambda
A amostra fornecida permite criar facilmente aplicativos/serviços web serverless
e APIs RESTful usando o Fastify sobre o AWS Lambda e o Amazon API Gateway.

*Observação: o uso de [aws-lambda-fastify](https://github.com/fastify/aws-lambda-fastify) é apenas uma maneira possível.*

### app.js

```js
const fastify = require('fastify');

function init() {
  const app = fastify();
  app.get('/', (request, reply) => reply.send({ hello: 'world' }));
  return app;
}

if (require.main === module) {
  // chamando diretamente node app
  init().listen(3000, (err) => {
    if (err) console.error(err);
    console.log('server listening on 3000');
  });
} else {
  // requisitado como um módulo => executado sobre o aws lambda
  module.exports = init;
}
```
Quando executado em sua função lambda, não precisamos ouvir uma porta específica,
então apenas exportamos a função wrapper `init` neste caso.
O arquivo [`lambda.js`](https://www.fastify.io/docs/latest/Serverless/#lambda-js) utilizará essa exportação.

Quando você executa seu aplicativo Fastify como sempre,
ou seja, `node app.js` *(a detecção para isso pode ser `require.main === module`)*,
normalmente você pode ouvir sua porta, para continuar executando a função Fastify localmente.

### lambda.js

```js
const awsLambdaFastify = require('aws-lambda-fastify')
const init = require('./app');

const proxy = awsLambdaFastify(init())
// ou
// const proxy = awsLambdaFastify(init(), { binaryMimeTypes: ['application/octet-stream'] })

exports.handler = proxy;
// ou
// exports.handler = (event, context, callback) => proxy(event, context, callback);
// ou
// exports.handler = (event, context) => proxy(event, context);
// ou
// exports.handler = async (event, context) => proxy(event, context);
```

We just require [aws-lambda-fastify](https://github.com/fastify/aws-lambda-fastify)
(make sure you install the dependency `npm i --save aws-lambda-fastify`) and our
[`app.js`](https://www.fastify.io/docs/latest/Serverless/#app-js) file and call the
exported `awsLambdaFastify` function with the `app` as the only parameter.
The resulting `proxy` function has the correct signature to be used as lambda `handler` function. 
This way all the incoming events (API Gateway requests) are passed to the `proxy` function of [aws-lambda-fastify](https://github.com/fastify/aws-lambda-fastify).

Nós apenas exigimos [aws-lambda-fastify](https://github.com/fastify/aws-lambda-fastify)
(certifique-se de instalar a dependência `npm i --save aws-lambda-fastify`) e nosso
[`app.js`](https://www.fastify.io/docs/latest/Serverless/#app-js) e chame o arquivo que
exportou a função `awsLambdaFastify` com o `app` como o único parâmetro.
A função `proxy` resultante tem a assinatura correta para ser usada como função `handler` lambda.
Dessa forma, todos os eventos recebidos (solicitações do API Gateway) são passados para a função `proxy` do [aws-lambda-fastify](https://github.com/fastify/aws-lambda-fastify).

### Exemplo

Um exemplo entregável com [claudia.js](https://claudiajs.com/tutorials/serverless-express.html) pode ser encontrado [aqui](https://github.com/claudiajs/example-projects/tree/master/fastify-app-lambda).


### Considerações

- O API Gateway ainda não suporta streams, portanto, você não pode lidar com [streams](https://www.fastify.io/docs/latest/Reply/#streams).
- O API Gateway tem um tempo limite de 29 segundos, por isso é importante fornecer uma resposta durante esse período.

## Google Cloud Run

Diferentemente do AWS Lambda ou do Google Cloud Functions, o Google Cloud Run é um ambiente de **contêiner** serverless. Seu principal objetivo é fornecer um ambiente abstrato de infraestrutura para executar contêineres arbitrários. Como resultado, o Fastify pode ser implantado no Google Cloud Run com poucas ou nenhuma alterações de código da maneira como você escreveria seu aplicativo Fastify normalmente.

*Siga as etapas abaixo para implantar no Google Cloud Run se você já está familiarizado com o gcloud ou apenas siga o [início rápido](https://cloud.google.com/run/docs/quickstarts/build-and-deploy)*.

### Adjust Fastify server

Para que o Fastify ouça corretamente solicitações no contêiner, defina a porta e o endereço corretos:

```js
function build() {
  const fastify = Fastify({ trustProxy: true })
  return fastify
}

async function start() {
  // O Google Cloud Run definirá essa variável de ambiente para você, então
  // você também pode usá-lo para detectar se você está executando no Cloud Run
  const IS_GOOGLE_CLOUD_RUN = process.env.K_SERVICE !== undefined

  // Você deve ouvir sobre a porta que o Cloud Run provê
  const port = process.env.PORT || 3000

  // Você deve ouvir sobre todo endereço IPV4 no Cloud Run
  const address = IS_GOOGLE_CLOUD_RUN ? "0.0.0.0" : undefined

  try {
    const server = build()
    const address = await server.listen(port, address)
    console.log(`Listening on ${address}`)
  } catch (err) {
    console.error(err)
    process.exit(1)
  }
}

module.exports = build

if (require.main === module) {
  start()
}
```

### Adiciona um Dockerfile

Você pode adicionar qualquer `Dockerfile` válido que empacote e execute um aplicativo Node. Um `Dockerfile` básico pode ser encontrado nos documentos oficiais [gcloud docs](https://github.com/knative/docs/blob/2d654d1fd6311750cc57187a86253c52f273d924/docs/serving/samples/hello-world/helloworld-nodejs/).

```Dockerfile
# Usando a versão oficial do Node.js 10
# https://hub.docker.com/_/node
FROM node:10

# Cria e muda o diretório da aplicação.
WORKDIR /usr/src/app

# A dependência do aplicativo de cópia se manifesta na imagem do contêiner.
# Um wildcard é usado para garantir que o package.json e o package-lock.json sejam copiados.
# Copiar isso separadamente impede a execução da instalação do npm em todas as alterações de código.
COPY package*.json ./

# Instala dependências de produção.
RUN npm install --only=production

# Copia o código local para a imagem do container.
COPY . .

# Rode o web service sobre o startup do container.
CMD [ "npm", "start" ]
```

### Adicionando um arquivo .dockerignore

Para manter os artefatos de construção fora do contêiner (o que o mantém pequeno e melhora o tempo de construção), adicione um arquivo `.dockerignore` como o abaixo:

```.dockerignore
Dockerfile
README.md
node_modules
npm-debug.log
```

### Construindo imagem

Em seguida, envie seu aplicativo para ser incorporado em uma imagem do Docker executando o seguinte comando (substituindo `PROJECT-ID` e `APP-NAME` pelo seu ID de projeto do GCP e um nome de aplicativo):

```bash
gcloud builds submit --tag gcr.io/PROJECT-ID/APP-NAME
```

### Deploy Imagem

Depois que sua imagem for construída, você pode realizar o deploy com o seguinte comando:

```bash
gcloud beta run deploy --image gcr.io/PROJECT-ID/APP-NAME --platform managed
```

Seu app vai ser acessível via a URL que o GCP provê.

## Zeit Now

[now](https://zeit.co/home) provê zero confiração de deploy para aplicações Node.js.
Para usar agora, é tão simples quanto
configurando seu arquivo `now.json` da seguinte maneira:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "api/serverless.js",
      "use": "@now/node",
      "config": {
        "helpers": false
      }
    }
  ],
  "routes": [
    { "src": "/.*", "dest": "/api/serverless.js"}
  ]
}
```


Então, escreva um `api/serverless.js` como:

```js
'use strict'

const build = require('./index')

const app = build()

module.exports = async function (req, res) {
  await app.ready()
  app.server.emit('request', req, res)
}
```

E um arquivo `api/index.js`:

```js
'use strict'

const fastify = require('fastify')

function build () {
  const app = fastify({
    logger: true
  })

  app.get('/', async (req, res) => {
    const { name = 'World' } = req.query
    req.log.info({ name }, 'hello world!')
    return `Hello ${name}!`
  })

  return app
}

module.exports = build
```

Note que você vai precisar usar Node 10 configurando no `package.json`:

```js
  "engines": {
    "node": "10.x"
  },
```
