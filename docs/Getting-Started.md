<h1 align="center">Fastify</h1>

## 'Aquecendo'
Olá! Obrigado por conferir o Fastify!<br>
Este documento visa em apresentar amigavelmente o framework e suas funcionalidades. Sendo um prefácio elementar com exemplos e links para outras partes da documentação.<br>
Iniciemos!

<a name="install"></a>
### Instalação
Instalação via npm:
```
npm i fastify --save
```
Instalação via yarn:
```
yarn add fastify
```

<a name="first-server"></a>
### Seu primeiro servidor
Vamos escrever nosso primeiro servidor:
```js
// Declara, 'importa' e 'instacializa'
const fastify = require('fastify')({
  logger: true
})

// Declara uma rota
fastify.get('/', function (request, reply) {
  reply.send({ hello: 'world' })
})

// Inicia o servidor, na porta 3000!
fastify.listen(3000, function (err, address) {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
  fastify.log.info(`Servidor respondendo no endereço ${address}`)
})
```

Você prefere usar `async/await`? Fastify suporta de nativamente.<br>
*(Também sugerimos o uso do [make-promises-safe](https://github.com/mcollina/make-promises-safe) a fim de evitar exposição de arquivo e vazamentos de memória.)*
```js
const fastify = require('fastify')()

fastify.get('/', async (request, reply) => {
  return { hello: 'world' }
})

const start = async () => {
  try {
    await fastify.listen(3000)
  } catch (err) {
    fastify.log.error(err)
    process.exit(1)
  }
}
start()
```

Maravilha, simples assim.<br>
Infelizmente, escrever uma aplicação complexa exige significantemente mais código do que este simples exemplo. Um clássico problema quando você está construindo uma nova aplicação é como lidar com múltiplos arquivos, fluxos assíncronos e a arquitetura do seu código.<br>
Fastify oferece uma plataforma simples que ajuda a resolver todos os problemas mencionados, e mais!

> ## Nota
> O exemplo acima, assim como os exemplos subsequentes, por padrão respondem *apenas* na interface de rede `localhost - 127.0.0.1`. Para utilizar todas as interfaces IPv4 se faz necessária a inclusão do parâmetro `0.0.0.0`, como segue: 
>
> ```js
> fastify.listen(3000, '0.0.0.0', function (err, address) {
>   if (err) {
>     fastify.log.error(err)
>     process.exit(1)
>   }
>   fastify.log.info(`Servidor respondendo no endereço ${address}`)
> })
> ```
>
> De forma similar, informe `::1` para aceitar apenas conexões locais via IPv6. Ou informe `::` para aceitar conexões em todos os endereços IPv6 e também em todos endereços IPv4 se o sistema operacional suportar.
>
> Quando estiver implantando em um container Docker, ou qualquer outro, usar `0.0.0.0` ou `::` é o método mais fácil de expôr sua aplicação.

<a name="first-plugin"></a>
### Primeiro plugin
Siminar ao fato de que em JavaScript tudo é um objeto, As with JavaScript, no Fastify tudo é um plugin.<br>
Antes de aprofundarmos, vamos ver como funciona!<br>
Vamos declarar nosso servidor básico, contudo, ao invés de declarar a rota dentro arquivo principal, iremos declará-lo em um arquivo separado (confira a documentação [declaração de rotas](https://github.com/fastify/docs-portuguese/blob/master/docs/Routes.md)).
```js
const fastify = require('fastify')({
  logger: true
})

fastify.register(require('./nossa-primeira-rota'))

fastify.listen(3000, function (err, address) {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
  fastify.log.info(`Servidor respondendo no endereço ${address}`)
})
```

```js
// nossa-primeira-rota.js

async function routes (fastify, options) {
  fastify.get('/', async (request, reply) => {
    return { hello: 'world' }
  })
}

module.exports = routes
```
Nesse exemplo, nós usamos a API `register`, que faz parte do core do framework Fastify. Esse é o único jeito de adicionar rotas, plugins, etc.

No início deste guia, podemos observar que o Fastify provê as bases para fluxo assíncrono. Por que isso é importante?  
Considere o seguinte cenário em que uma conexão com o banco de dados se faça necessária para realizar armazenamento de dados. Obviamente, a conexão com o banco de dados precisa estar disponível antes do servidor aceitar conexões. Como resolvemos esse problema?<br>
Uma solução típica é utilizar de funções 'callback' complexas ou 'promises' - um sistema que irá mesclar as API do framework com outras bibliotecas e o código da aplicação.<br>
O Fastify cuida disso internamente, sem o menor esforço!

'Uai sô! Simbôra' escrever um exemplo com conexão a um banco de dados.<br>
*(Nós iremos usar um exemplo simples. Para uma solução mais robusta considere utilizar [`fastify-mongo`](https://github.com/fastify/fastify-mongodb) ou outro do [ecossistema](https://github.com/fastify/docs-portuguese/blob/master/docs/Ecosystem.md) Fastify)*
*(Para esse exemplo você precisa de ter instalada uma versão do MongoDB localmente)*

Primeiro, instale o `fastify-plugin` e o `mongodb`:

```
npm install --save fastify-plugin mongodb
```

**server.js**
```js
const fastify = require('fastify')({
  logger: true
})

fastify.register(require('./our-db-connector'), {
  url: 'mongodb://localhost:27017/'
})
fastify.register(require('./our-first-route'))

fastify.listen(3000, function (err, address) {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
  fastify.log.info(`Servidor respondendo no endereço ${address}`)
})
```

**our-db-connector.js**
```js
const fastifyPlugin = require('fastify-plugin')
const MongoClient = require('mongodb').MongoClient

async function dbConnector (fastify, options) {
  const url = options.url
  delete options.url

  const db = await MongoClient.connect(url, options)
  fastify.decorate('mongo', db)
}

// Envolver uma função plugin com fastify-plugin expõem os decorators,
// hooks, and middlewares declarados dentro do plugin para o escopo pai.
module.exports = fastifyPlugin(dbConnector)
```

**our-first-route.js**
```js
async function routes (fastify, options) {
  const database = fastify.mongo.db('db')
  const collection = database.collection('test')

  fastify.get('/', async (request, reply) => {
    return { hello: 'world' }
  })

  fastify.get('/search/:id', async (request, reply) => {
    const result = await collection.findOne({ id: request.params.id })
    if (result.value === null) {
      throw new Error('Valor inválido')
    }
    return result.value
  })
}

module.exports = routes
```

Uau, isso foi rápido!<br>
'Está na hora da revisão' do que fizemos até aqui desde uma vez que apresentamos alguns conceitos novos.<br>
Como pode ser visto, nós usamos `register` para ambas funcionalidades, conector de banco de dados e registro de rotas.
Essa é uma das melhores características do Fastify, ele irá carregar seus plugins na mesma ordem que você declarar eles e irá carregar o próximo plugin *apenas* quando o atual acabar de ser carregado. Desta forma, nós podemos/devemos registrar nosso conector de banco de dados e podemos utilizar ele no segundo. *(leia [lidando com o escopo](https://github.com/fastify/docs-portuguese/blob/master/docs/Plugins.md#handle-the-scope) para entender como lidar com o escopo de um plugin)*.
O carregamento de plugin inicia no momento em que os métodos `fastify.listen()`, `fastify.inject()` ou `fastify.ready()` são invocados.

Nós também usamos a API `decorate` para adicionar um objeto personalizado no namespace do Fastify, tornando-o disponível para o uso em todos dos lugares, subsequentes. O uso dessa API é encorajado para facilitar a fácil reutilização de código, evitando assim a quantidade de código ou a duplicação da lógica.

Para investigar a fundo como os plugins funcionam no Fastify, como implementar novos plugins, detalhes como utilizar todo arsenal da API do Fastify para lidar com complexidades de fluxo assíncrono de uma aplicação, leia [O 'Guia do mochileiro' para plugins](https://github.com/fastify/docs-portuguese/blob/master/docs/Plugins-Guide.md).

<a name="plugin-loading-order"></a>
### Ordem de carregamento dos plugins
Para garantir a consistência e previsibilidade de comportamentos de sua aplicação sugerimos efusivamente para sempre carregar seu código como mostrado abaixo:
```
└── plugins (do ecossistema Fastify)
└── seus plugins (seus plugins personalizados)
└── decorators
└── hooks e middlewares
└── seus serviços 
```
Desta forma você sempre terá acesso a todas as propriedades declaradas no escopo atual.<br/>
Como mencionado previamente o Fastify oferece um sólido modelo de encapsulamento no intuito de te ajudar na construção de suas aplicações como simples e independentes serviços. Se você deseja registrar um plugin apenas para um subconjunto de rotas basta replicar a estrutura acima.
```
└── plugins (do ecossitema Fastify)
└── seus plugins (seus plugins personalizados)
└── decorators
└── hooks and middlewares
└── seus serviços
    │
    └──  serviço A
    │     └── plugins (do ecossitema Fastify)
    │     └── seus plugins (seus plugins personalizados)
    │     └── decorators
    │     └── hooks and middlewares
    │     └── seus serviços
    │
    └──  serviço B
          └── plugins (do ecossitema Fastify)
          └── seus plugins (seus plugins personalizados)
          └── decorators
          └── hooks and middlewares
          └── seus serviços
```

<a name="validate-data"></a>
### Validação de dados
Validação de dados é extremamente importante e um conceito fundamental do framework.<br>
Para validar requisições o Fastify utiliza [JSON Schema](http://json-schema.org/).
Vamos ver um exemplo demonstrando validações para rotas:
```js
const opts = {
  schema: {
    body: {
      type: 'object',
      properties: {
        someKey: { type: 'string' },
        someOtherKey: { type: 'number' }
      }
    }
  }
}

fastify.post('/', opts, async (request, reply) => {
  return { hello: 'world' }
})
```
Este exemplo mostra como passar um objeto de opções como parametro da rota, que aceita uma chave `schema`, que por sua vez contém todos os schemas para a rota: `body`, `querystring`, `params` and `headers`.<br>
Leia [Validações e Serialização](https://github.com/fastify/docs-portuguese/blob/master/docs/Validation-and-Serialization.md) para aprender mais.

<a name="serialize-data"></a>
### Serialização de dados
Fastify tem suporte de primeira classe para JSON. É extremamente otimizado para converter conteúdo JSON e para serializar retornos JSON.<br>
Para acelerar a serialização JSON (sim, é lento!) use a chave `response` do schema como exibido no código a seguir:
```js
const opts = {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          hello: { type: 'string' }
        }
      }
    }
  }
}

fastify.get('/', opts, async (request, reply) => {
  return { hello: 'world' }
})
```
Pelo simples fato de especificar o schema como mostrado é possível acelerar a serialização 2-3x. Isso também ajuda a proteger contra vazamento de dados potenciamente sensíveis, uma vez que o Fastify irá serializar apenas os dados declarados na chave `response` do schema.
Leia [Validação e Serialização](https://github.com/fastify/docs-portuguese/blob/master/docs/Validation-and-Serialization.md) para aprender mais.

<a name="extend-server"></a>
### Expandindo seu servidor
Fastify é desenvolvido para ser extremamente expandível e mínimo, acreditamos que a 'espinha dorsal' de um framework é tudo o que é necessário para possibilitar a criação de aplicações maravilhosas.<br>
Em outras palavras, o Fastify não é um framework com 'baterias inclusas', e conta com um extraordinário [ecossistema](https://github.com/fastify/docs-portuguese/blob/master/docs/Ecosystem.md)!

<a name="test-server"></a>
### Testando seu servidor
Fastify não oferece uma estrutura de teste, mas recomendamos uma forma de escrever seus testes que usa as funcionalidades e a arquitetura do Fastify.<br>
Leia a documentação [testando](https://github.com/fastify/docs-portuguese/blob/master/docs/Testing.md) para aprender mais!

<a name="cli"></a>
### 'Rodar' seu servidor utilizando a CLI
Fastify também tem integração a CLI graças a [fastify-cli](https://github.com/fastify/fastify-cli).

Primeiro, instale `fastify-cli`:

```
npm i fastify-cli
```

Você também pode instalar globalmente utilizando o parâmetro `-g`.

Então, adicione as seguintes linhas ao arquivo `package.json`:
```json
{
  "scripts": {
    "start": "fastify start server.js"
  }
}
```

Crie também seu arquivo do servidor:
```js
// server.js
'use strict'

module.exports = async function (fastify, opts) {
  fastify.get('/', async (request, reply) => {
    return { hello: 'world' }
  })
}
```

Para em seguida 'rodar' o servidor com:
```bash
npm start
```

<a name="slides"></a>
### Slides and Videos
- Slides
  - [Alcance velocidades 'absurdas' com seu servidor HTTP (English)](https://mcollina.github.io/take-your-http-server-to-ludicrous-speed) por [@mcollina](https://github.com/mcollina)
  - [E se eu te disser que HTTP pode ser rápido (English)](https://delvedor.github.io/What-if-I-told-you-that-HTTP-can-be-fast) por [@delvedor](https://github.com/delvedor)

- Videos
  - [Alcance velocidades 'absurdas' com seu servidor HTTP (English)](https://www.youtube.com/watch?v=5z46jJZNe8k) por [@mcollina](https://github.com/mcollina)
  - [E se eu te disser que HTTP pode ser rápido (English)](https://www.webexpo.net/prague2017/talk/what-if-i-told-you-that-http-can-be-fast/) por [@delvedor](https://github.com/delvedor)
