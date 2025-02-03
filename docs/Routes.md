<h1 align="center">Fastify</h1>

## Rotas

Os métodos de rotas configurão os endpoints da sua aplicação.  
Tem duas formas de declarar uma rota com o Fastify, a abreviação ou declaração completa.

- [Declaração completa](#full-declaration)
- [Objeto de opções da rota](#options)
- [Declarações abreviadas](#shorthand-declaration)
- [Parâmetros de URL](#url-building)
- [Usando `async`/`await`](#async-await)
- [Resolução de `Promise`](#promise-resolution)
- [Prefixando rotas](#route-prefixing)
- Logs
  - [Nível personalizado de Log](#custom-log-level)
  - [Serializador personalizado de Log](#custom-log-serializer)
- [Configuração de `handler` da rota](#routes-config)
- [Versionamento de rotas](#version)

<a name="full-declaration"></a>
### Declaração completa

```js
fastify.route(options)
```

<a name="options"></a>
### Objeto de opções da rota

* `method`: atualmente suporta `'DELETE'`, `'GET'`, `'HEAD'`, `'PATCH'`, `'POST'`, `'PUT'` and `'OPTIONS'`. Além de possibilitar ser um array de métodos.
* `url`: o caminho da url para corresponder com a rota (alias: `path`).
* `schema`: um objeto contendo o schema para a requisição e a resposta.
Necessita estar no formato
  [JSON Schema](http://json-schema.org/), confira [aqui](https://github.com/fastify/docs-portuguese/blob/main/docs/Validation-and-Serialization.md) para mais informações.

  * `body`: valida o body `req.body` da requisição se for POST ou PUT..
  * `querystring` ou `query`: valida a querystring. Pode conter um objeto schema completo, com as propriedades `type` do `object` e `properties` do object dos parâmetros, ou simplesmente os valores que cada um deve conter nas propriedades do objeto, como mostrado abaixo.
  * `params`: valida os parâmetros `req.params`.
  * `response`: filtra e gera um schema para a resposta, definindo esta propriedade possibilita um acréscimo entre 10-20% de taxa de transferência. 
* `attachValidation`: anexa `validationError` a requisição, se existir um schema de validação de erros, ao invés de enviar o erro ao handler.
* `onRequest(request, reply, done)`: uma função [hook](https://github.com/fastify/docs-portuguese/blob/main/docs/Hooks.md#onrequest) que é acionada tão logo que uma requisição é recebida, também pode ser um array de funções.
* `preParsing(request, reply, done)`: uma função [hook](https://github.com/fastify/docs-portuguese/blob/main/docs/Hooks.md#preparsing) acionada antes de realizar o `parse` da requisição, também pode ser um array de funções.
* `preValidation(request, reply, done)`: uma função [hook](https://github.com/fastify/docs-portuguese/blob/main/docs/Hooks.md#prevalidation) acionada após os hooks `preValidation` compartilhados, muito útil se você precisa de realizar autenticação a nível de rota, por exemplo, também pode ser um array de funções.
* `preHandler(request, reply, done)`: uma função [hook](https://github.com/fastify/docs-portuguese/blob/main/docs/Hooks.md#prehandler) acionada logo antes do `handler` da requisição, também pode ser um array de funções.
* `preSerialization(request, reply, payload, done)`: uma função [hook](https://github.com/fastify/docs-portuguese/blob/main/docs/Hooks.md#preserialization) acionada logo antes da serialização, também pode ser um array de funções.
* `onSend(request, reply, payload, done)`: uma função [hook](https://github.com/fastify/docs-portuguese/blob/main/docs/Hooks.md#route-hooks) acionada logo antes da resposta ser enviada, também pode ser um array de funções.
* `onResponse(request, reply, payload, done)`: uma função [hook](https://github.com/fastify/docs-portuguese/blob/main/docs/Hooks.md#onresponse) acionada no momento que uma resposta foi enviada, so you will not be able to send more data to the client. It could also be an array of functions.
* `handler(request, reply)`: a função que vai lidar com o tratamento/processamento da requisição.
* `schemaCompiler(schema)`: a função que irá realizar a construção das funções de validação para cada schema. Confira [aqui](https://github.com/fastify/docs-portuguese/blob/main/docs/Validation-and-Serialization.md#schema-compiler)
* `bodyLimit`: previne que o `parser` padrão de JSON de interpretar conteúdos maiores que o valor definido __em bytes__. *Tem que ser um valor inteiro*. Você pode também definir essa opção globalmente quando estiver criando a instância do Fastify no objeto de opções de parâmetro __options__ em `fastify(options)`. O valor padrão é `1048576` (1 MiB).
* `logLevel`: define o nível do log para a rota. Confira o exemplo abaixo.
* `logSerializers`: define o serializador do log para a rota.
* `config`: objeto usado para armazenar configurações personalizadas.
* `version`: uma string [semver](http://semver.org/) compatível que define a versão do endpoint. [Exemplo](https://github.com/fastify/docs-portuguese/blob/main/docs/Routes.md#version).
* `prefixTrailingSlash`: string usada para determinar como lidar uma rota `/` e um prefixo.
  * `both` (__padrão__): Registrá-la ambas `/prefix` e `/prefix/`.
  * `slash`: Registrá-la somente `/prefix/`.
  * `no-slash`: Registrá-la somente `/prefix`.

  `request` é definida em [Request](https://github.com/fastify/docs-portuguese/blob/main/docs/Request.md).

  `reply` é definida em [Reply](https://github.com/fastify/docs-portuguese/blob/main/docs/Reply.md).


Examplo:
```js
fastify.route({
  method: 'GET',
  url: '/',
  schema: {
    querystring: {
      name: { type: 'string' },
      excitement: { type: 'integer' }
    },
    response: {
      200: {
        type: 'object',
        properties: {
          hello: { type: 'string' }
        }
      }
    }
  },
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
})
```

<a name="shorthand-declaration"></a>
### Declarações abreviadas
As declarações anteriores são mais parecidas com o estilo-*Hapi*, mas se prefere uma abordagem *Express/Restify*, não apenas disponibilizamos como apoiamos:<br>
`fastify.get(path, [options], handler)`<br>
`fastify.head(path, [options], handler)`<br>
`fastify.post(path, [options], handler)`<br>
`fastify.put(path, [options], handler)`<br>
`fastify.delete(path, [options], handler)`<br>
`fastify.options(path, [options], handler)`<br>
`fastify.patch(path, [options], handler)`

Examplo:
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
fastify.get('/', opts, (request, reply) => {
  reply.send({ hello: 'world' })
})
```

`fastify.all(path, [options], handler)` irá definir o mesmo `handler` para todos os métodos suportados.

O `handler` também pode ser fornecido via o objeto `options`:
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
  },
  handler (request, reply) {
    reply.send({ hello: 'world' })
  }
}
fastify.get('/', opts)
```

> Note que: se o `handler` é especificado em ambos objeto `options` e terceiro parâmetro `handler` do método abreviado dispará-la um erro de duplicação de handler.

<a name="url-building"></a>
### Parâmetros de URL
Fastify suporta ambos tipos de urls estáticas e dinâmicas.<br>

Para registrar um caminho **parametrizado**, use o caracter `:` antes do nome do parâmetro. Para **curinga** use o caracter `*`
*Lembre-se que rotas estáticas são sempre conferidas antes das parametrizadas e das curingas.*

```js
// parametrizadas
fastify.get('/example/:userId', (request, reply) => {}))
fastify.get('/example/:userId/:secretToken', (request, reply) => {}))

// curingas
fastify.get('/example/*', (request, reply) => {}))
```

Rotas com Expressões Regulares (RegExp) são suportadas também, mas **ATENÇÃO**, pois são muito custosas em termos de desempenho!
```js
// parametric with regexp
fastify.get('/example/:file(^\\d+).png', (request, reply) => {}))
```

É possível também definir mais de um parametro dentro de um par de barras `/`, exemplo:
```js
fastify.get('/example/near/:lat-:lng/radius/:r', (request, reply) => {}))
```
*Lembre-se que nesse caso de usar o traço `-` como separador.*

Finalmente é possível ter vários parâmetros com RegExp.
```js
fastify.get('/example/at/:hour(^\\d{2})h:minute(^\\d{2})m', (request, reply) => {}))
```
Neste caso é possível usar como separador qualquer caracter que não combine com a expressão regular anterior ou posterior utilizada.

Ter rotas com múltiplos parâmetros pode afetar negativamente no desempenho, por isso sempre que possível escolha a abordagem de um único parâmetro, especialmente em rotas que são/serão muito utilizadas.
Se você se interessar em saber como lidar com roteamento, confira [find-my-way](https://github.com/delvedor/find-my-way).

<a name="async-await"></a>
### Usando `async`/`await`
Você é um usuário de `async/await`? 'Então está em boas mãos'!
```js
fastify.get('/', options, async function (request, reply) {
  var data = await getData()
  var processed = await processData(data)
  return processed
})
```

Como pode ver não acionamos o método `reply.send` para retornar dados para o usuário. Basta utilizar o `return` com o conteúdo e 'está feito'!

Mas se precisar o método `reply.send` ainda assim está disponível.
```js
fastify.get('/', options, async function (request, reply) {
  var data = await getData()
  var processed = await processData(data)
  reply.send(processed)
})
```

Se a rota estiver dentro de uma API baseada em callback que irá acionar o `reply.send()` fora da 'corrente' de promessas, é possível também utilizar o `await reply`:

```js
fastify.get('/', options, async function (request, reply) {
  setImmediate(() => {
    reply.send({ hello: 'world' })
  })
  await reply
})
```

Retornar o objeto `reply` também funciona:

```js
fastify.get('/', options, async function (request, reply) {
  setImmediate(() => {
    reply.send({ hello: 'world' })
  })
  return reply
})
```

**AVISO:**
* Quando utilizar ambos `return value` e `reply.send(value)` ao mesmo tempo, o primeiro que ocorrer tem prioridade, o segundo valor será descartado e um *warn* log será emitido referente a tentativa de enviar uma resposta 2x+.
* Você não pode retornar `undefined`. Para mais detalhes leia [promise-resolution](#promise-resolution).

<a name="promise-resolution"></a>
### Resolução de `Promise`

Se o seu handler é uma função `async` ou retorna promise, você deve estar ciente de um comportamento especial onde é necessário no intuito de tornar possíveis os controles de fluxos de callbacks e promises. Se a promessa do handler é resolvida com `undefined`, será então ignorado causando a suspensão da requisição e a emissão e registro de um log do tipo *error*.

1. Se você quer usar `async/await` ou promises e irá responder com um `reply.send`:
    - **Não** use `return`
    - **Não** se esqueça de utilizar o `reply.send`.
2. If you want to use `async/await` or promises:
    - **Não** use `reply.send`.
    - **Não** returne `undefined`.

Desta forma, podemos disponibilizar ambos `callback-style` and `async-await`, com o um custo de intercâmbio mínimo. Apesar de tanta liberdade, é altamente recomendável seguir apenas um estilo, pois o tratamento de erros deve tratado de maneira consistente na sua aplicação.

**Notificação**: Cada função async já retorna por si só uma promise.

<a name="route-prefixing"></a>
### Prefixando rotas
Algumas vezes precisamos manter duas ou mais diferentes versões de uma mesma api, uma abordagem clássica é prefixar todas as rotas com  o número da versão, por exemplo `/v1/user`.
Fastify oferece a você um rápido e inteligente de criar diferentes versões da mesma api sem ter que alterar todas as rotas manualmente uma por uma, com o sistema de *prefixação de rotas*. Como funciona? Vejamos:

```js
// server.js
const fastify = require('fastify')()

fastify.register(require('./routes/v1/users'), { prefix: '/v1' })
fastify.register(require('./routes/v2/users'), { prefix: '/v2' })

fastify.listen(3000)
```

```js
// routes/v1/users.js
module.exports = function (fastify, opts, done) {
  fastify.get('/user', handler_v1)
  done()
}
```

```js
// routes/v2/users.js
module.exports = function (fastify, opts, done) {
  fastify.get('/user', handler_v2)
  done()
}
```
O Fastify não irá reclamar pelo fato de você estar utilizando o mesmo nome para duas rotas diferentes, pois será no momento de compilação que lidaremos com o prefixo automaticamente *(isso também significa que o desempenho não será afetado de forma alguma!)*.

Agora os clientes da sua API tem a disposição deles as seguintes rotas:
- `/v1/user`
- `/v2/user`

Você pode fazer isso quantas vezes quiser, isso também funciona para aninhamento de `register`s e parametrização de rotas.
Esteja ciente de que se você utilizar o [`fastify-plugin`](https://github.com/fastify/fastify-plugin) essa opção **não** funciona.

#### Lidando com a `/` ao utilizar plugins prefixados

A rota `/` tem diferentes comportamentos dependendo se o prefixo termina ou não com `/`. 
Como exemplo, se considerarmos um prefixo `/something/`, adicionando uma rota `/` o handler somente será acionado por uma requisição ao caminho `/something/`. 
Se considerarmos o prefixo `/something` e uma rota `/` esta será acionada por ambos `/something` e `/something/` endereços.

Confira a opção e rota `prefixTrailingSlash` acima para modificar este comportamento.

<a name="custom-log-level"></a>
### Nível personalizado de Log
Pode acontecer de você precisar de diferentes níveis de log em suas rotas e o Fastify realiza isso de uma maneira bastante simples.<br/>
Você precisa apenas passar a opção `logLevel` como o objeto opção para do plugin ou da rota com o [valor](https://github.com/pinojs/pino/blob/master/docs/API.md#discussion-3) que você precisa.

Esteja ciente que, se você definir o `logLevel` a nível de plugin, também serão afetados os parâmetros do servidor [`setNotFoundHandler`](https://github.com/fastify/docs-portuguese/blob/main/docs/Server.md#setnotfoundhandler) and [`setErrorHandler`](https://github.com/fastify/docs-portuguese/blob/main/docs/Server.md#seterrorhandler).

```js
// server.js
const fastify = require('fastify')({ logger: true })

fastify.register(require('./routes/user'), { logLevel: 'warn' })
fastify.register(require('./routes/events'), { logLevel: 'debug' })

fastify.listen(3000)
```

Ou você pode incluir diretamente na rota:
```js
fastify.get('/', { logLevel: 'warn' }, (request, reply) => {
  reply.send({ hello: 'world' })
})
```
*Lembre-se que um nível personalizado de Log é aplicado apenas para as rotas e não para o Fastify globalmente, acessível via `fastify.log`*

<a name="custom-log-serializer"></a>
### Serializador personalizado de Log

Em algum contexto, você pode precisar de registrar o log de um objeto muito grande mais que pode ser um total desperdício de recursos para algumas rotas. Neste caso, você pode definir algum [`serializador`](https://github.com/pinojs/pino/blob/master/docs/api.md#bindingsserializers-object) e anexar ele no contexto específico!

```js
const fastify = require('fastify')({ logger: true })

fastify.register(require('./routes/user'), { 
  logSerializers: {
    user: (value) => `My serializer one - ${value.name}`
  } 
})
fastify.register(require('./routes/events'), {
  logSerializers: {
    user: (value) => `My serializer two - ${value.name} ${value.surname}`
  }
})

fastify.listen(3000)
```

Serializadores por contexto podem ser herdados:

```js
const fastify = Fastify({ 
  logger: {
    level: 'info',
    serializers: {
      user (req) {
        return {
          method: req.method,
          url: req.url,
          headers: req.headers,
          hostname: req.hostname,
          remoteAddress: req.ip,
          remotePort: req.connection.remotePort
        }
      }
    }
  } 
})

fastify.register(context1, { 
  logSerializers: {
    user: value => `My serializer father - ${value}`
  } 
})

async function context1 (fastify, opts) {
  fastify.get('/', (req, reply) => {
    req.log.info({ user: 'call father serializer', key: 'another key' }) // exibe: { user: 'My serializer father - call father  serializer', key: 'another key' }
    reply.send({})
  })
}

fastify.listen(3000)
```

<a name="routes-config"></a>
### Configuração de `handler` da rota
Registrando um novo handler, você pode definir um objeto de configuração e receber diretamente no handler.

```js
// server.js
const fastify = require('fastify')()

function handler (req, reply) {
  reply.send(reply.context.config.output)
}

fastify.get('/en', { config: { output: 'hello world!' } }, handler)
fastify.get('/it', { config: { output: 'ciao mondo!' } }, handler)

fastify.listen(3000)
```

<a name="version"></a>
### Versionamento de rotas

#### Default
Se necessário você poderá disponibilizar uma opção de versionamento, que permitirá você declarar múltiplas versões da mesma rota. O versionamento deve seguir as especificações do [semver](http://semver.org/).<br/>
O Fastify irá automaticamente detectar o cabeçalho `Accept-Version` e roteará a requisição de acordo (intervalos avançados e pre-releases não são suportados).<br/>
*Esteja ciente que, utilizar esta funcionalidade pode causar degradação do desempenho do sistema de roteamento como um todo.*

```js
fastify.route({
  method: 'GET',
  url: '/',
  version: '1.2.0',
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
})

fastify.inject({
  method: 'GET',
  url: '/',
  headers: {
    'Accept-Version': '1.x' // também pode ser '1.2.0' ou '1.2.x'
  }
}, (err, res) => {
  // { hello: 'world' }
})
```

Se você declarar múltiplas versões com o mesmo Major or Minor, Fastify irá sempre escolher a versão compatível mais 'alta' do valor do cabeçalho `Accept-Version`.<br/>
Se a requisição não tem o cabeçalho `Accept-Version` definido, retornará um erro `404`.

#### Custom
É possível definir uma lógica personalizada de versionamento. Que pode ser realizado através da configuração [`versioning`](https://github.com/fastify/docs-portuguese/blob/main/docs/Server.md#versioning), enquanto cria a instãncia do servidor Fastify.