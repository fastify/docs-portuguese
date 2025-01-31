<h1 align="center">Fastify</h1>

<a name="factory"></a>
## Factory
(Factory é um 'design pattern' para geração de instâncias, em modelo de fábrica, daí o nome 'factory')

O módulo Fastify exporta uma função factory que é usada para criar novas instâncias do 
<a href="https://github.com/fastify/docs-portuguese/blob/main/docs/Server.md"><code><b>servidor Fastify</b></code></a>. 
Essa função factory aceita um objeto como parâmetro que é usado para customizar a instância resultante. 
Este documento descreve as propriedades disponíveis nesse objeto JSON.

<a name="factory-http2"></a>
### `http2`

Se `true` utiliza o módulo Node.js core's [HTTP/2](https://nodejs.org/dist/latest-v8.x/docs/api/http2.html) para a vinculação do socket.

+ Valor padrão: `false`

<a name="factory-https"></a>
### `https`

Um objeto usado para configurar o socket de 'escuta' para TLS.
As opções são as mesmas do método [`createServer`](https://nodejs.org/dist/latest-v8.x/docs/api/https.html#https_https_createserver_options_requestlistener) do Node.js core.
Quando o valor desta propriedade é `null`, o socket não será configurado para utilizar TLS.

Esta opção também se aplica quando a propriedade 
<a href="https://github.com/fastify/docs-portuguese/blob/main/docs/Server.md#factory-http2">
<code><b>http2</b></code>
</a> estiver configurada.

+ Valor padrão: `null`

<a name="factory-ignore-slash"></a>
### `ignoreTrailingSlash`

Fastify utiliza o [find-my-way](https://github.com/delvedor/find-my-way) para lidar com o roteamento. 
Esta opção pode ser configurada para `true` no intuito de ignorar barras à direita nas rotas.
Esta opção se aplica para *todas* as rotas registradas para a instância resultante do servidor.

+ Valor padrão: `false`

```js
const fastify = require('fastify')({
  ignoreTrailingSlash: true
})

// registrado para ambos "/foo" e "/foo/"
fastify.get('/foo/', function (req, reply) {
  reply.send('foo')
})

// registrado para ambos "/bar" e "/bar/"
fastify.get('/bar', function (req, reply) {
  reply.send('bar')
})
```

<a name="factory-max-param-length"></a>
### `maxParamLength`
Você pode configurar um tamanho personalizado para o conteúdo dos parâmetros (standard, regex e multi) de rotas utilizando essa opção, o valor padrão é 100 caracteres.<br>
Isto pode ser bastante útil especialmente se você tem alguma rota baseada em regex, protegendo assim contra [ataques DoS (English)](https://www.owasp.org/index.php/Regular_expression_Denial_of_Service_-_ReDoS).<br>
*Se o tamanho máximo for atingido a rota '404 - não encontrado' será invocada.*

<a name="factory-body-limit"></a>
### `bodyLimit`

Define o tamanho máximo do conteúdo do body da requisição, em bytes, que o servidor está autorizado a aceitar.

+ Valor padrão: `1048576` (1MiB)

<a name="factory-on-proto-poisoning"></a>
### `onProtoPoisoning`

Define qual ação o framework terá que realizar ao realizar o parse de um objeto JSON com `__proto__`.
Esta funcionalidade é provida por [secure-json-parse](https://github.com/fastify/secure-json-parse).
Veja https://hueniverse.com/a-tale-of-prototype-poisoning-2610fa170061 (English) para maiores informações sobre ataques do tipo 'protótipo envenenado'.

Valores possíveis são `'error'`, `'remove'` e `'ignore'`.

+ Valor padrão: `'error'`

<a name="factory-on-constructor-poisoning"></a>
### `onConstructorPoisoning`

Define qual ação o framework terá de realizar quando realizar o parse de um objeto JSON com `constructor`.
Esta funcionalidade é provida por [secure-json-parse](https://github.com/fastify/secure-json-parse).
Veja https://hueniverse.com/a-tale-of-prototype-poisoning-2610fa170061 (English) para maiores informações sobre ataques do tipo 'protótipo envenenado'.

Valores possíveis são `'error'`, `'remove'` e `'ignore'`.

+ Valor padrão: `'ignore'`

<a name="factory-logger"></a>
### `logger`

Fastify inclui internamente a funcionalidade de log via o logger [Pino](https://getpino.io/).
Esta propriedade é utilizada para configurar a instância interna de logger.

Os valores possíveis que esta propriedade pode ter são:

+ Valor padrão: `false`. O logger é desabilitado. Todos os métodos de log apontarão para um logger null [abstract-logging (English)](https://npm.im/abstract-logging) instance.

+ `pinoInstance`: uma instância previamente ativa do pino. O logger interno irá apontar para a instância definida.

+ `object`: um padrão Pino [objeto de opções (English)](https://github.com/pinojs/pino/blob/c77d8ec5ce/docs/API.md#constructor).
Esta opção irá passar o objeto recebido diretamente para o construtor do Pino. 
Se as seguintes propriedades não estiverem presentes no objeto elas utilizarão os valores padrões como segue:
    * `genReqId`: uma função síncrona que será utilizada para gerar os identificadores das requisições. O função padrão gera identificadores sequenciais.
    * `level`: o nível mínimo para log. Valor padrão é `'info'`.
    * `serializers`: uma função de serialização hash. Por padrão, serializadores serão adicionados aos objetos `req` (objeto da requisição), `res` (objetos de resposta) e `err` (objeto `Error` padrão). Quando um método log receber um objeto com qualquer uma dessas propriedades então o respectivo serializador será utilizado para aquela propriedade. Por exemplo:
        ```js
        fastify.get('/foo', function (req, res) {
          req.log.info({req}) // registra o log do objeto req serializado
          res.send('foo')
        })
        ```
      Qualquer serializador fornecido pelo usuário irá sobrescrever a propriedade correspondente.
+ `loggerInstance`: uma instância logger em uso. O logger deve estar em conformidade com a interface Pino, tendo os seguintes métodos: `info`, `error`, `debug`, `fatal`, `warn`, `trace`, `child`. Por exemplo:
  ```js
  const pino = require('pino')();

  const customLogger = {
    info: function (o, ...n) {},
    warn: function (o, ...n) {},
    error: function (o, ...n) {},
    fatal: function (o, ...n) {},
    trace: function (o, ...n) {},
    debug: function (o, ...n) {},
    child: function() {
      const child = Object.create(this);
      child.pino = pino.child(...arguments);
      return child;
    },
  };

  const fastify = require('fastify')({logger: customLogger});
  ```

<a name="factory-disable-request-logging"></a>
### `disableRequestLogging`
Por padrão, quando a funcionalidade de log estiver ativa, o Fastify irá disparar uma mensagem de log com nível `info` quando uma requisição for recebida e quando a resposta para esta requisição for enviada. Ao configurar esta opção para `true` estas mensagens de log serão desativadas.
Isso possibilita o início e fim mais flexível de requisições ao anexar hooks `onRequest` e `onResponse` personalizados.

+ Valor padrão: `false`

```js
// Examples of hooks to replicate the disabled functionality.
fastify.addHook('onRequest', (req, reply, done) => {
  req.log.info({ url: req.req.url, id: req.id }, 'received request')
  done()
})

fastify.addHook('onResponse', (req, reply, done) => {
  req.log.info({ url: req.req.originalUrl, statusCode: res.res.statusCode }, 'request completed')
  done()
})
```

<a name="custom-http-server"></a>
### `serverFactory`
Você pode passar um servidor http customizado para o Fastify utilizando a opção `serverFactory`.<br/>
`serverFactory` é uma função que pega um parâmetro função `handler`, que recebe os objetos `request` e `response` como parâmetros, e um objeto de opções, que é o mesmo que você passou para o Fastify.

```js
const serverFactory = (handler, opts) => {
  const server = http.createServer((req, res) => {
    handler(req, res)
  })

  return server
}

const fastify = Fastify({ serverFactory, modifyCoreObjects: false })

fastify.get('/', (req, reply) => {
  reply.send({ hello: 'world' })
})

fastify.listen(3000)
```

Internamente o Fastify usa a API server nativa do Node, então se você estiver usando um servidor customizado tenha certeza que tenha a mesma API exposta. Se não você pode incrementar a instância do servidor dentro da função `serveFactory` antes da declaração `return`.<br/>
*Note que nós também adicionamos `modifyCoreObjects: false` porque em alguns ambientes [serverless](https://github.com/fastify/docs-portuguese/blob/main/docs/Serverless.md), como Google Cloud Functions, algumas propriedades não são modificáveis.*

<a name="factory-case-sensitive"></a>
### `caseSensitive`

Por padrão o valor é `true`, rotas são registradas diferenciando maiúsculas de minúsculas. Isto é, `/foo` não é equivalente a `/Foo`. Quando configurado para `false`, rotas são registradas de maneira a interpretar que `/foo` é equivalente a `/Foo` que por sua vez é equivalente a `/FOO`.

Ao configurar `caseSensitive` com `false`, todos os caminhos irão corresponder como minúsculas, entretanto os valores dos parâmetros de rota ou curingas irãm manter seu valor original.

```js
fastify.get('/user/:username', (request, reply) => {
  // Given the URL: /USER/NodeJS
  console.log(request.params.username) // -> 'NodeJS'
})
```

Note, por favor, que ao configurar essa opção para `false` vai contra a [RFC3986 (English)](https://tools.ietf.org/html/rfc3986#section-6.2.2.1).

<a name="factory-request-id-header"></a>
### `requestIdHeader`

O cabeçalho utilizado para identificar o id da requisição. Veja a seção [id da requisição](https://github.com/fastify/docs-portuguese/blob/main/docs/Logging.md#logging-request-id).

+ Valor padrão: `'request-id'`

<a name="factory-request-id-log-label"></a>
### `requestIdLogLabel`

Define o rótulo usado para o identificador da requisição quando efetuar o log desta.

+ Valor padrão: `'reqId'`

<a name="factory-gen-request-id"></a>
### `genReqId`

Função para gerar o id da requisição. Recebe o objeto requeste como parâmetro.

+ Valor padrão: `valor da propriedade 'request-id' se fornecida ou uniformemente incrementa inteiros`

Especialmente em sistemas distribuídos, você pode querer sobrescrever o comportamento padrão de geração de id como mostrado abaixo. Para geração de `UUID`s você pode querer conferir [hyperid](https://github.com/mcollina/hyperid)

```js
let i = 0
const fastify = require('fastify')({
  genReqId: function (req) { return i++ }
})
```

**Note que: genReqId _não_ será chamado se o cabeçalho 'request-id' estiver disponível.**

<a name="factory-trust-proxy"></a>
### `trustProxy`

Ao habilitar a opção `trustProxy` o Fastify terá conhecimento de que está atrás de um proxy confiável e que os campos cabeçalho `X-Forwarded-*` também o são, que de outra forma poderiam ser facilmente falsificados.

```js
const fastify = Fastify({ trustProxy: true })
```

+ Valor padrão: `false`
+ `true/false`: Confiar em todos os proxies (`true`) or não confiar em nenhum (`false`).
+ `string`: Confiar apenas quando IP/CIDR (ex. `'127.0.0.1'`). Pode ser uma lista de valores separados por vírgula (ex. `'127.0.0.1,192.168.1.1/24'`).
+ `Array<string>`: Confiar apenas quando IP/CIDR estiver na lista (e.g. `['127.0.0.1']`).
+ `number`: Confiar no `n` salto do servidor de proxy a frente como sendo o cliente.
+ `Function`: Função que trata personalizadamente a confiança, recebe `address` e `hop` como argumentos
    ```js
    function myTrustFn(address, hop) {
      return address === '1.2.3.4' || hop === 1
    }
    ```

Para mais exemplos consulte o pacote [proxy-addr](https://www.npmjs.com/package/proxy-addr).

Você pode conferir os valores `ip`, `ips`, e `hostname` no objeto [`request`](https://github.com/fastify/docs-portuguese/blob/main/docs/Request.md).

```js
fastify.get('/', (request, reply) => {
  console.log(request.ip)
  console.log(request.ips)
  console.log(request.hostname)
})
```

<a name="plugin-timeout"></a>
### `pluginTimeout`

A quantidade máxima de tempo em *milesegundos* que um plugin tem para ser carregado.
Se não, [`ready`](https://github.com/fastify/docs-portuguese/blob/main/docs/Server.md#ready)
será concluido com um `Error` com de código `'ERR_AVVIO_PLUGIN_TIMEOUT'`.

+ Valor padrão: `10000`

<a name="factory-querystring-parser"></a>
### `querystringParser`

(query string se refere dos parâmetro enviados via url, i.e. ?param1=1&param2=2, onde o param1 de `1` e param2 de valor `2`)
O interpretador padrão de query string que o Fastify usará é o módulo nativo do Node.js `querystring`.<br/>
Você pode alterar e utlizar um personalizado ou um pacote npm de sua escolha, como o [`qs`](https://www.npmjs.com/package/qs).

```js
const qs = require('qs')
const fastify = require('fastify')({
  querystringParser: str => qs.parse(str)
})
```

<a name="versioning"></a>
### `versioning`

Por padrão você pode versionar suas rotas com [semver versioning](https://github.com/fastify/docs-portuguese/blob/main/docs/Routes.md#version), que é provido por `find-my-way`. Existe também a opção de utilizar uma estratégia personalizada. Você pode encontrar mais informações na documentação do [find-my-way](https://github.com/delvedor/find-my-way#versioned-routes).

```js
const versioning = {
  storage: function () {
    let versions = {}
    return {
      get: (version) => { return versions[version] || null },
      set: (version, store) => { versions[version] = store },
      del: (version) => { delete versions[version] },
      empty: () => { versions = {} }
    }
  },
  deriveVersion: (req, ctx) => {
    return req.headers['accept']
  }
}

const fastify = require('fastify')({
  versioning
})
```

<a name="factory-modify-core-objects"></a>
### `modifyCoreObjects`

+ Valor padrão: `true`

Por padrão, o Fastify irá adicionar as propriedades `ip`, `ips`, `hostname` e `log` no objeto request do Node (veja [`Request`](https://github.com/fastify/docs-portuguese/blob/main/docs/Request.md)) e a propriedade `log` no objeto response do Node. Altere o valor para `false` para prevenir que essas propriedades sejam adicionadas nos objetos nativos do Node.

```js
const fastify = Fastify({ modifyCoreObjects: true }) // the default

fastify.get('/', (request, reply) => {
  console.log(request.raw.ip)
  console.log(request.raw.ips)
  console.log(request.raw.hostname)
  request.raw.log('Hello')
  reply.res.log('World')
})
```

Desabilite essa opção pode ajudar em ambiente [serveless](https://github.com/fastify/docs-portuguese/blob/main/docs/Serverless.md) como Google Cloud Functions, onde `ip` e `ips` não são propriedades modificáveis.

**Note que essas propriedades estão depreciadas e serão removidas na próxima versão Major do Fastify, assim como essa opção** Sendo então recomendado o uso das mesmas propriedades dos objetos [`Request`](https://github.com/fastify/docs-portuguese/blob/main/docs/Request.md) e [`Reply`](https://github.com/fastify/docs-portuguese/blob/main/docs/Reply.md).

```js
const fastify = Fastify({ modifyCoreObjects: false })

fastify.get('/', (request, reply) => {
  console.log(request.ip)
  console.log(request.ips)
  console.log(request.hostname)
  request.log('Hello')
  reply.log('World')
})
```

<a name="factory-return-503-on-closing"></a>
### `return503OnClosing`

Retorna 503 após chamar o método `close` do server method.
Se `false`, o sistema de rotas do servidor continua recebendo requisições como de costume.

+ Valor padrão: `true`

<a name="factory-ajv"></a>
### `ajv`

Configura a instância do ajv usada pelo Fastify sem prover uma nova personalizada.

+ Valor padrão:
```js
{
  customOptions: {
    removeAdditional: true,
    useDefaults: true,
    coerceTypes: true,
    allErrors: true,
    nullable: true
  },
  plugins: []
}
```

```js
const fastify = require('fastify')({
  ajv: {
    customOptions: {
      nullable: false // Refira-se a [opções ajv](https://ajv.js.org/#options)
    },
    plugins: [
      require('ajv-merge-patch')
      [require('ajv-keywords'), 'instanceof'];
      // Uso: [plugin, pluginOptions] - Plugin com opções
      // Uso: plugin - Plugin sem opções
    ]
  }
})
```

## Instância

### Métodos do Servidor

<a name="server"></a>
#### server
`fastify.server`: Objeto nativo Node [server](https://nodejs.org/api/http.html#http_class_http_server) como retornado pela [**`Função Fastify factory`**](https://github.com/fastify/docs-portuguese/blob/main/docs/Server.md).

<a name="after"></a>
#### after
Invocada quando o plugin atual e todos os plugins que foram registrados terminaram de serem carregados.
Sempre é executada antes do método `fastify.ready`.

```js
fastify
  .register((instance, opts, done) => {
    console.log('Current plugin')
    done()
  })
  .after(err => {
    console.log('After current plugin')
  })
  .register((instance, opts, done) => {
    console.log('Next plugin')
    done()
  })
  .ready(err => {
    console.log('Everything has been loaded')
  })
```

<a name="ready"></a>
#### ready
Função chamada quando todos os plugins foram carregados.
Recebe um erro como parâmetro quanto alguma coisa 'deu errado'.
```js
fastify.ready(err => {
  if (err) throw err
})
```
Se é chamada sem nenhum argumento então retorna uma `Promise`:

```js
fastify.ready().then(() => {
  console.log('successfully booted!')
}, (err) => {
  console.log('an error happened', err)
})
```

<a name="listen"></a>
#### listen
Inicia o servidor na porta recebida após o carregamento de todos os plugins, internamente aguarda pelo evento `.ready()`. O callback é o mesmo nativo do Node. Por padrão irá responder no endereço resolvido de `localhost` quando nenhum endereço for especificado (`127.0.0.1` ou `::1` dependendo do sistema operacional). Se estiver responder em qualquer interface de rede for desejado então deverá ser especificado `0.0.0.0` respondendo assim em todos os endereços de interface de rede IPv4. Usando `::` responderá em todos endereços de interface de rede IPv6 e dependendo do SO pode responder também em todos endereços de interface de rede IPv4. Seja cauteloso na hora de decidir por escutar em todas as interfaces; for the address will listen on all IPv6 addresses, and, depending on OS, may also listen on all IPv4 addresses. Be careful when deciding to listen on all interfaces; esta configuração vem com [riscos de segurança (english)](https://web.archive.org/web/20170831174611/https://snyk.io/blog/mongodb-hack-and-secure-defaults/) inerentes.

```js
fastify.listen(3000, (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

Especificar um endereço também é suportado:

```js
fastify.listen(3000, '127.0.0.1', (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

Especificar um tamanho de lista de espera também é suportado:

```js
fastify.listen(3000, '127.0.0.1', 511, (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

Especificar opções também é suportado, o objeto é o mesmo que o [options](https://nodejs.org/api/net.html#net_server_listen_options_callback) do método listen da função nativa do Node.js:

```js
fastify.listen({ port: 3000, host: '127.0.0.1', backlog: 511 }, (err) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

Se nenhum callback é fornecido uma `Promise` é retornada:

```js
fastify.listen(3000)
  .then((address) => console.log(`server listening on ${address}`))
  .catch(err => {
    console.log('Error starting server:', err)
    process.exit(1)
  })
```

Especificar um endereço sem uma callback também é suportado:

```js
fastify.listen(3000, '127.0.0.1')
  .then((address) => console.log(`server listening on ${address}`))
  .catch(err => {
    console.log('Error starting server:', err)
    process.exit(1)
  })
```

Especificar um objeto de opções sem uma callback também é suportado:

```js
fastify.listen({ port: 3000, host: '127.0.0.1', backlog: 511 })
  .then((address) => console.log(`server listening on ${address}`))
  .catch(err => {
    console.log('Error starting server:', err)
    process.exit(1)
  })
```

Quando implantando em um container Docker, e potencialmente em outros também, é aconselhável indicar `0.0.0.0` porque geralmente não existe mapeamento padrão de exposição de portas para `localhost`:

```js
fastify.listen(3000, '0.0.0.0', (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

Se a porta é omitida (ou definida com `0`), uma porta aleatória é automaticamente escolhida (e pode ser obtida via `fastify.server.address().port`).

Os valores padrões para `listen` são:

```js
fastify.listen({
  port: 0,
  host: 'localhost',
  exclusive: false,
  readableAll: false,
  writableAll: false,
  ipv6Only: false
}, (err) => {})
```

<a name="route"></a>
#### route
Método para adicionar rotas no servidor, também existem funções abreviadas, confira [aqui](https://github.com/fastify/docs-portuguese/blob/main/docs/Routes.md).

<a name="close"></a>
#### close
`fastify.close(callback)`: chamar essa função encerra a instância do servidor e executa o hook [`'onClose'`](https://github.com/fastify/docs-portuguese/blob/main/docs/Hooks.md#on-close).<br>
Chamar `close` também faz com que o servidor responda com erro `503` para todas as requisições recebidas e destrua aquelas requisições.
Confira em [`return503OnClosing` flags](https://github.com/fastify/docs-portuguese/blob/main/docs/Server.md#factory-return-503-on-closing) como alterar este comportamento.

Se for chamada sem nenhum argumento, retorna uma `Promise`:

```js
fastify.close().then(() => {
  console.log('successfully closed!')
}, (err) => {
  console.log('an error happened', err)
})
```

<a name="decorate"></a>
#### decorate*
Função muito útil se você precisa 'decorar' a instância do Fastify, Reply ou Request, confira [aqui](https://github.com/fastify/docs-portuguese/blob/main/docs/Decorators.md).

<a name="register"></a>
#### register
O Fastify permite que os usuários extenda suas funcionalidades com plugins.
Um plugin pode ser um conjunto de rotas, 'decorações' ou o que quer que seja, confira [aqui](https://github.com/fastify/docs-portuguese/blob/main/docs/Plugins.md).

<a name="use"></a>
#### use
Função para adicionar ao Fastify, confira [aqui](https://github.com/fastify/docs-portuguese/blob/main/docs/Middleware.md).

<a name="addHook"></a>
#### addHook
Função para adicionar um hook específico no ciclo de vida do Fastify, confira [aqui](https://github.com/fastify/docs-portuguese/blob/main/docs/Hooks.md).

<a name="prefix"></a>
#### prefix
O caminho completo que irá será prefixado em uma rota.

Exemplo:

```js
fastify.register(function (instance, opts, done) {
  instance.get('/foo', function (request, reply) {
    // Irá exibir no log (caso habilitado) "prefix: /v1"
    request.log.info('prefix: %s', instance.prefix)
    reply.send({ prefix: instance.prefix })
  })

  instance.register(function (instance, opts, done) {
    instance.get('/bar', function (request, reply) {
      // Irá exibir no log (caso habilitado) "prefix: /v1/v2"
      request.log.info('prefix: %s', instance.prefix)
      reply.send({ prefix: instance.prefix })
    })

    done()
  }, { prefix: '/v2' })

  done()
}, { prefix: '/v1' })
```

<a name="pluginName"></a>
#### pluginName
Nome do plugin em carregamento no momento. Há 3 formas de definir o nome (na ordem).

1. Se o plugin usa o metadado `name` do [fastify-plugin](https://github.com/fastify/fastify-plugin).
2. Se você utiliza `module.exports` o nome do arquivo é usado.
3. Se você usa uma [declaração de função (English)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions#Defining_functions) regular.

*Fallback*: As duas linhas do seu plugin representam o nome do plugin. Novas linhas são substituidas por ` -- `. Isso ajudará na identificação da causa raiz quando lidando com muitos plugins.

Importante: Se você está lidando com plugins aninhados o nome divergirá com o uso [fastify-plugin](https://github.com/fastify/fastify-plugin) porque nenhum novo escopo é criado e portanto não temos nenhum local para anexar dados de contexto. Neste caso o nome do plugin representará a ordem inicial na qual foram invocados, utilizando o formato `plugin-A -> plugin-B`.

<a name="log"></a>
#### log
A instância do logger, confira [aqui](https://github.com/fastify/docs-portuguese/blob/main/docs/Logging.md).

<a name="inject"></a>
#### inject
Injeção falsa de http (para fins de teste), confira [aqui](https://github.com/fastify/docs-portuguese/blob/main/docs/Testing.md#inject).

<a name="add-schema"></a>
#### addSchema
`fastify.addSchema(schemaObj)`, adiciona um schema compartilhado na instância do Fastify. Possibilitará o reuso em todos os locais em sua aplicação simplesmente utilizando o id do schema que você precisa.<br/>
Para aprender mais, veja [exemplo de schema compartilhado](https://github.com/fastify/docs-portuguese/blob/main/docs/Validation-and-Serialization.md#shared-schema) na documentação sobre [Validação e Serialização](https://github.com/fastify/docs-portuguese/blob/main/docs/Validation-and-Serialization.md).

<a name="set-reply-serializer"></a>
#### setReplySerializer
Configura o serializado de resposta para todas as rotas. Será utilizado como padrão se uma função [Reply.serializer(func)](https://github.com/fastify/docs-portuguese/blob/main/docs/Reply.md#serializerfunc) não foi fornecida. A função responsável para lidar com a serialização é totalmente encapsulada, sendo assim, por exemplo, pode conter diversas funções para lidar com erros.
Note que: o parametro função `func` é chamado somente para status `2xx`. Confira [`setErrorHandler`](https://github.com/fastify/docs-portuguese/blob/main/docs/Server.md#seterrorhandler) para lidar com erros.

```js
fastify.setReplySerializer(function (payload, statusCode){
  // serializa o payload com uma função síncrona
  return `my serialized ${statusCode} content: ${payload}`
})
```

<a name="set-schema-compiler"></a>
#### setSchemaCompiler
Configura o schema do compilador para todas as rotas [aqui](https://github.com/fastify/docs-portuguese/blob/main/docs/Validation-and-Serialization.md#schema-compiler).

<a name="set-schema-resolver"></a>
#### setSchemaResolver
Define o schema de resolução `$ref` para todas as rotas [here](https://github.com/fastify/docs-portuguese/blob/main/docs/Validation-and-Serialization.md#schema-resolver).


<a name="schema-compiler"></a>
#### schemaCompiler
Esta propriedade pode ser usada para definir o compilador de schema, é uma abreviação para o método `setSchemaCompiler`, e espera o compilador de schema para todas as rotas.

<a name="set-not-found-handler"></a>
#### setNotFoundHandler

`fastify.setNotFoundHandler(handler(request, reply))`: define a função que irá lidar com status `404`. Essa função é encapsulada por prefixo, então diferentes plugins podem ser definidos para diferentes funções que lidam com situações 'não encontrado' se uma [opção `prefix`](https://github.com/fastify/docs-portuguese/blob/main/docs/Plugins.md#route-prefixing-option) for passada para `fastify.register()`. Tal função é tratada como uma função regular de rota então as requisições irão passar por todo [ciclo de vida do Fastify](https://github.com/fastify/docs-portuguese/blob/main/docs/Lifecycle.md#lifecycle).

Você pode também registar um hook [`preValidation`](https://www.fastify.io/docs/latest/Hooks/#route-hooks) and [preHandler](https://www.fastify.io/docs/latest/Hooks/#route-hooks) para cuidar dos `404`.

```js
fastify.setNotFoundHandler({
  preValidation: (req, reply, done) => {
    // Seu código
    done()
  },
  preHandler: (req, reply, done) => {
    // Seu código
    done()
  }
}, function (request, reply) {
    // Função padrão para lidar com `404` após os hooks preValidation e preHandler
})

fastify.register(function (instance, options, done) {
  instance.setNotFoundHandler(function (request, reply) {
    // Lida com requisições em situação `404` sem os hooks preValidation e preHandler
    // to URLs that begin with '/v1'
  })
  done()
}, { prefix: '/v1' })
```

<a name="set-error-handler"></a>
#### setErrorHandler

`fastify.setErrorHandler(handler(error, request, reply))`: Define uma função para ser executada sempre que um erro acontecer. O handler é totalmente encapsulado, então diferentes plugins podem definir direfentes handlers para erros diferentes. *async-await* também é suportado.<br>
*Note que: Se o `statusCode` do erro é menor que 400, o Fastify irá automaticamente mudá-lo para 500 antes de chamar o handler do erro.*

```js
fastify.setErrorHandler(function (error, request, reply) {
  // Log erro
  // Envia resposta de erro
})
```

Fastify é fornecido com uma função padrão que é chamada se nenhum handler de erro é definido e ela registra o log do erro respeitando o `statusCode`:

```js
var statusCode = error.statusCode
if (statusCode >= 500) {
  log.error(error)
} else if (statusCode >= 400) {
  log.info(error)
} else {
  log.error(error)
}
```

<a name="print-routes"></a>
#### printRoutes

`fastify.printRoutes()`: 'Prita' a representação da árvore raiz interna usada para o roteamento, muito útil para 'debugar'.<br/>
*Lembre-se de chamar dentro ou após o evento `ready` ser chamado.*

```js
fastify.get('/test', () => {})
fastify.get('/test/hello', () => {})
fastify.get('/hello/world', () => {})

fastify.ready(() => {
  console.log(fastify.printRoutes())
  // └── /
  //   ├── test (GET)
  //   │   └── /hello (GET)
  //   └── hello/world (GET)
})
```

<a name="initial-config"></a>
#### initialConfig

`fastify.initialConfig`: Expõem um objeto 'congelado' e não modificável de registro das opções iniciais passadas pelo usuário como configuração da instância do Fastify.

Atualmente as propriedades que podem ser expostas são:
- bodyLimit
- caseSensitive
- http2
- https (retorna `false`/`true` ou `{ allowHTTP1: true/false }` se explicitamente definido)
- ignoreTrailingSlash
- maxParamLength
- onProtoPoisoning
- pluginTimeout
- requestIdHeader

```js
const { readFileSync } = require('fs')
const Fastify = require('fastify')

const fastify = Fastify({
  https: {
    allowHTTP1: true,
    key: readFileSync('./fastify.key'),
    cert: readFileSync('./fastify.cert')
  },
  logger: { level: 'trace'},
  ignoreTrailingSlash: true,
  maxParamLength: 200,
  caseSensitive: true,
  trustProxy: '127.0.0.1,192.168.1.1/24',
})

console.log(fastify.initialConfig)
/*
irá registrar no log :
{
  caseSensitive: true,
  https: { allowHTTP1: true },
  ignoreTrailingSlash: true,
  maxParamLength: 200
}
*/

fastify.register(async (instance, opts) => {
  instance.get('/', async (request, reply) => {
    return instance.initialConfig
    /*
    irá retornar :
    {
      caseSensitive: true,
      https: { allowHTTP1: true },
      ignoreTrailingSlash: true,
      maxParamLength: 200
    }
    */
  })

  instance.get('/error', async (request, reply) => {
    // irá disparar um erro porque initialConfig é read-only/somente-leitura
    // e não pode ser modificado
    instance.initialConfig.https.allowHTTP1 = false

    return instance.initialConfig
  })
})

// Começa a escutar.
fastify.listen(3000, (err) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```
