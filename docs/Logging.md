<h1 align="center">Fastify</h1>

## Logging
(Sistema de Log [Registro de atividades/ocorrências])

Logging é desabilitado por parão e você pode habilitá-lo informando `{ logger: true }` ou `{ logger: { level: 'info' } }` quando estiver criando a instância do Fastify. Note que se o logger estiver desabilitado é impossível habilitado em tempo de execução. Para essa funcionalidade usamos o pacote [abstract-logging](https://www.npmjs.com/package/abstract-logging).

Desempenho é o foco principal do Fastify e por isso usa o pacote [pino](https://github.com/pinojs/pino) como logger. Quando habilitado, utiliza `'info'` como nível padrão de log, se não for definido explicitamente para utilizar outro nível.

É extremamente simples habilitá-lo, veja:

```js
const fastify = require('fastify')({
  logger: true
})

fastify.get('/', options, function (request, reply) {
  request.log.info('Some info about the current request')
  reply.send({ hello: 'world' })
})
```

Se deseja personalizar as opções do logger basta informá-las ao Fastify.
Encontre toas as opções disponíveis na [documentação do Pino (English)](https://github.com/pinojs/pino/blob/master/docs/api.md#pinooptions-stream). Se você desejar utilizar um arquivo para armazenar os registro use: 

```js
const fastify = require('fastify')({
  logger: {
    level: 'info',
    file: '/path/to/file' // Será usado pino.destination()
  }
})

fastify.get('/', options, function (request, reply) {
  request.log.info('Some info about the current request')
  reply.send({ hello: 'world' })
})
```

Caso queira utilizar um stream personalizada para a instância do Pino simplesmente adicione na propriedade stream do objeto logger.

```js
const split = require('split2')
const stream = split(JSON.parse)

const fastify = require('fastify')({
  logger: {
    level: 'info',
    stream: stream
  }
})
```

<a name="logging-request-id"></a>
Por padrão o Fastify adiciona um id para cada requisição para facilitar a identificação. Se estiver presente o header "resquest-id", este valor será usado, caso contrário um novo valor incremental será gerado. Para personalizar esse comportamento veja as propriedade [`requestIdHeader`](https://github.com/fastify/docs-portuguese/blob/master/docs/Server.md#factory-request-id-header) e [`genReqId`](https://github.com/fastify/docs-portuguese/blob/master/docs/Server.md#gen-request-id) da Factory do Fastify.

O logger padrão é configurado com um conjunto padrão de serializadores que serializam os objetos com propriedades `req`, `res` e `err`. Esse comportamento pode ser personalizado informando um serializador customizado.
```js
const fastify = require('fastify')({
  logger: {
    serializers: {
      req: function (req) {
        return { url: req.url }
      }
    }
  }
})
```
Por exemplo, o conteúdo e o cabeçalho da resposta podem ser logados utilizando a seguinte abordagem (mesmo *não sendo recomendado*):

```js
const fastify = require('fastify')({
  logger: {
    prettyPrint: true,
    serializers: {
      res(res) {
        // O mesmo do padrão
        return {
          statusCode: res.statusCode
        }
      },
      req(req) {
        return {
          method: req.method,
          url: req.url,
          path: req.path,
          parameters: req.parameters,
          // Incluindo o cabeçado no log é uma violação de leis de privacidade, ex. GDPR.
          // Você deve utilizar a opção "redact" para remover campos sensíveis.
          // Pois a opção abaixo também acarreta em vazamento de dados de autenticação.
          headers: req.headers
        };
      }
    }
  }
});
```
**Note que**: O `body` da requisição não pode ser serializada dentro do método `req`, porque a requisição é serializada quando criamos o logger 'filho'. E nesse momento o `body` ainda não foi 'parseado'.

Veja uma abordagem para realizar o log do `req.body`

```js
app.addHook('preHandler', function (req, reply, done) {
  if (req.body) {
    req.log.info({ body: req.body }, 'parsed body')
  }
  done()
})
```


*Essa opção será ignorada por qualquer outro logger diferente do Pino.*

Você pode informar uma instância de logger que já esteja utilizando. Ao invés de informar opções de configuração simplesmente defina a instância a ser utilizada.
O logger que você informar deve obedecer a interface do Pino; isto é, deve conter os seguintes métodos:
`info`, `error`, `debug`, `fatal`, `warn`, `trace`, `child`.

Example:

```js
const log = require('pino')({ level: 'info' })
const fastify = require('fastify')({ logger: log })

log.info('does not have request information')

fastify.get('/', function (request, reply) {
  request.log.info('includes request information, but is the same logger instance as `log`')
  reply.send({ hello: 'world' })
})
```

*A instância do logger para a requisição atual está disponível em todas as fases do [ciclo de vida](https://github.com/fastify/docs-portuguese/blob/master/docs/Lifecycle.md) do Fastify.*

## Log Redaction

[Pino](https://getpino.io) oferece log rectaction de baixo consumo para obscurecer valores de propriedades específicas nos registros de log.
Por exemplo, podemos querer registrar log de todo o header HTTP exceto, por questões de segurança, o conteúdo de `Authorization`:

```js
const fastify = Fastify({
  logger: {
    stream: stream,
    redact: ['req.headers.authorization'],
    level: 'info',
    serializers: {
      req (req) {
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
```

Para mais informações, acesse https://getpino.io/#/docs/redaction.
