<h1 align="center">Fastify</h1>

## Hooks
(`Hooks`, 'gancho' em tradução livre, é um conceito onde uma função é acionada quando um evento é disparado, sendo então um `listener`, 'ouvinte' em tradução livre.)

Hooks são registrado com o método `fastify.addHook` method e possibilita ouvir a eventos específicos na aplicação ou no cliclo de vida de uma requisição ou resposta. Você deve registrar um hook antes do evento ser disparado, caso contrário o evento é perdido.

Ao utilizar hooks você pode interagir diratamente com o ciclo de vida do Fastify. Existem hooks a nível de Request/Reply e de Aplicação:

- [Hooks de Request/Reply](#requestreply-hooks)
  - [onRequest](#onrequest)
  - [preParsing](#preparsing)
  - [preValidation](#prevalidation)
  - [preHandler](#prehandler)
  - [preSerialization](#preserialization)
  - [onError](#onerror)
  - [onSend](#onsend)
  - [onResponse](#onresponse)
- [Hooks de Aplicação](#application-hooks)
  - [onClose](#onclose)
  - [onRoute](#onroute)
  - [onRegister](#onregister)

**Note que:** o callback `done` não está disponível ao utilizar `async`/`await` ou ao retornar uma `Promise`. Se você invocar o callback `done` nessas situações, comportamentos inexperados/indejesados podem ocorrer (ex. invocação duplicada de handlers).

## Hooks de Request/Reply

[Request](https://github.com/fastify/docs-portuguese/blob/master/docs/Request.md) e [Reply](https://github.com/fastify/docs-portuguese/blob/master/docs/Reply.md) são objetos internos do Fastify.<br/>
`done` é uma função para continuar o [Ciclo de vida](https://github.com/fastify/docs-portuguese/blob/master/docs/Lifecycle.md).

É super fácil entender quando cada hook é executado olhando a [página Ciclo de vida](https://github.com/fastify/docs-portuguese/blob/master/docs/Lifecycle.md).<br>
Hooks são influênciados pelo encapsulamento do Fastify e assim podem ser aplicados em rotas específicas. Confira a seção [Scopes](#scope) para mais informações.

Existem 8 diferentes Request/Reply hooks que podem ser utilizados *(em ordem de execução)*:

### onRequest
```js
fastify.addHook('onRequest', (request, reply, done) => {
  // Algum código
  done()
})
```
Ou `async/await`:
```js
fastify.addHook('onRequest', async (request, reply) => {
  // Algum código
  await asyncMethod()
  // Ocorreu um erro
  if (err) {
    throw new Error('Some errors occurred.')
  }
  return
})
```

**Note que:** no hook [onRequest](#onRequest), `request.body` sempre terá o valor `null`, porque o parse do body acontece exatamente antes do hook [preValidation](#preValidation).

### preParsing
```js
fastify.addHook('preParsing', (request, reply, done) => {
  // Algum código
  done()
})
```
Ou `async/await`:
```js
fastify.addHook('preParsing', async (request, reply) => {
  // Algum código
  await asyncMethod()
  // Ocorreu um erro
  if (err) {
    throw new Error('Some errors occurred.')
  }
  return
})
```

**Notice:** in the [preParsing](#preParsing) hook, `request.body` sempre terá o valor `null`, porque o parse do body acontece exatamente antes do hook [preValidation](#preValidation).

### preValidation
```js
fastify.addHook('preValidation', (request, reply, done) => {
  // Algum código
  done()
})
```
Ou `async/await`:
```js
fastify.addHook('preValidation', async (request, reply) => {
  // Algum código
  await asyncMethod()
  // Ocorreu um erro
  if (err) {
    throw new Error('Some errors occurred.')
  }
  return
})
```

### preHandler
```js
fastify.addHook('preHandler', (request, reply, done) => {
  // Algum código
  done()
})
```
Ou `async/await`:
```js
fastify.addHook('preHandler', async (request, reply) => {
  // Algum código
  await asyncMethod()
  // Ocorreu um erro
  if (err) {
    throw new Error('Some errors occurred.')
  }
  return
})
```
### preSerialization

Se você está utilizando o hook `preSerialization`, você pode alterar (ou substituir) o payload antes dele ser serializado. Por exemplo:

```js
fastify.addHook('preSerialization', (request, reply, payload, done) => {
  const err = null;
  const newPayload = { wrapped: payload }
  done(err, newPayload)
})
```
Ou `async/await`
```js
fastify.addHook('preSerialization', async (request, reply, payload) => {
  return { wrapped: payload }
})
```

**Note que:** o hook `preSerialization` não será executado se o payload for uma `string`, um `Buffer`, um `stream` ou `null`.

### onError
```js
fastify.addHook('onError', (request, reply, error, done) => {
  // Algum código
  done()
})
```
Ou `async/await`:
```js
fastify.addHook('onError', async (request, reply, error) => {
  // Útil para personalizar o registro de log de erro
  // Você NÃO deve utilizar esse hook para atualizar o contéudo objeto de erro 'error'
})
```
Esse hook é útil se você necessita de customizar o log de erro ou adicionar algum header em caso de erro.<br/>
Ele não foi projetado para alterar o `error`, e acionar o `reply.send` irá disparar uma exceção.<br/>
Este hook somente será executado após o `customErrorHandler` tiver sido executado, e somente se `customErrorHandler` enviar um erro usuário *(Note que o `customErrorHandler` padrão sempre envia um erro ao usuário)*.<br/>
**Note que:** diferentemente de outros hooks, não é suportado passar um erro para a função `done`.

### onSend
Se você utilizar o hook `onSend`, você pode alterar o payload. Por exemplo:

```js
fastify.addHook('onSend', (request, reply, payload, done) => {
  const err = null;
  const newPayload = payload.replace('some-text', 'some-new-text')
  done(err, newPayload)
})
```
Ou `async/await`:
```js
fastify.addHook('onSend', async (request, reply, payload) => {
  const newPayload = payload.replace('some-text', 'some-new-text')
  return newPayload
})
```

Você também pode limpar o payload a ser enviado como resposta com um body vazio ou substituindo o payload com `null`:

```js
fastify.addHook('onSend', (request, reply, payload, done) => {
  reply.code(304)
  const newPayload = null
  done(null, newPayload)
})
```

> Você também pode enviar um body vazio ao substituir o payload por uma string vazia `''`, mas esteja ciente de que isso irá alterar o valor do header `Content-Length` para `0`, enquanto o valor do header `Content-Length` não será definido caso o payload for `null`.

**Note que:** Se você alterar o payload, você só pode defini-lo com uma `string`, um `Buffer`, um `stream` ou `null`.


### onResponse
```js

fastify.addHook('onResponse', (request, reply, done) => {
  // Algum código
  done()
})
```
Ou `async/await`:
```js
fastify.addHook('onResponse', async (request, reply) => {
  // Algum código
  await asyncMethod()
  // Ocorreu um erro
  if (err) {
    throw new Error('Some errors occurred.')
  }
  return
})
```

O hook `onResponse` é executado quando a resposta já foi enviada, então você não será capaz de enviar mais dados para o cliente. Entretanto, ele pode ser útil para enviar dados para serviços externos, por exemplo para coleta de estatísticas.

### Gerenciar erros de um hook
Se você obter um erro durante a execução de um hook, simplesmente adicione-o como parâmetro do método `done` e o Fastify irá automáticamente concluir a requisição enviando o código de erro apropriado para o usuário.

```js
fastify.addHook('onRequest', (request, reply, done) => {
  done(new Error('Some error'))
})
```

Se você deseja definir um código de erro específico, basta utilizar o método `reply.code`:
```js
fastify.addHook('preHandler', (request, reply, done) => {
  reply.code(400)
  done(new Error('Some error'))
})
```

*O erro será tratado pelo [`Reply`](https://github.com/fastify/docs-portuguese/blob/master/docs/Reply.md#errors).*

### Respondendo a uma requisição a partir de um hook
Se necessário, você poderá responder a uma requisição antes desta acionar o handler da rota, por exemplo quando implementar um hook de autenticação. Se foce estiver utilizando os hooks `onRequest` ou `preHandler` use `reply.send`; se estiver utilizando um middleware, use `res.end`.

```js
fastify.addHook('onRequest', (request, reply, done) => {
  reply.send('Early response')
})

// Também funciona com funções async
fastify.addHook('preHandler', async (request, reply) => {
  reply.send({ hello: 'world' })
})
```

Se você responder com um stream, você deve evitar usar uma função `async` para o hook. Se você precisar usar uma função `async`, nesse caso, utilize o seguinte padrão [test/hooks-async.js](https://github.com/fastify/docs-portuguese/blob/94ea67ef2d8dce8a955d510cd9081aabd036fa85/test/hooks-async.js#L269-L275).

```js
fastify.addHook('onRequest', (request, reply, done) => {
  const stream = fs.createReadStream('some-file', 'utf8')
  reply.send(stream)
})
```

## Hooks de Aplicação

Você pode utlizar hooks do cliclo de vida da aplicação também. É importante notar que esses hooks não são totalmente encapsulados. O `this` dentro desses hooks é encapsulado, todavia os handlers podem responder a um evento externo aos limites do encapsulamento.

- [onClose](#onclose)
- [onRoute](#onroute)
- [onRegister](#onregister)

<a name="on-close"></a>

### onClose
Disparado quando `fastify.close()` é invocado para parar o servidor. É útil quando [plugins](https://github.com/fastify/docs-portuguese/blob/master/docs/Plugins.md) precisam de um evento para 'desligar/finalizar', por exemplo para fechar uma conexão aberta com o banco de dados.<br>
O primeiro argumento é a instância do Fastify, o segundo é o callback `done`.
```js
fastify.addHook('onClose', (instance, done) => {
  // Algum código
  done()
})
```
<a name="on-route"></a>

### onRoute
Disparado quando uma nova rota é registrada. Listeners são passados ao objeto `routeOptions` como o único parâmetro. A interface é síncrona, e, assim sendo, os listeners não recebem um callback.
```js
fastify.addHook('onRoute', (routeOptions) => {
  // Algum código
  routeOptions.method
  routeOptions.schema
  routeOptions.url
  routeOptions.bodyLimit
  routeOptions.logLevel
  routeOptions.logSerializers
  routeOptions.prefix
})
```

Se você está escrevendo um plugin e necessita personalizar as rotas da aplicação, como modificar as opções or adicionar uma novo hook de rootas, esse é o lugar certo.

```js
fastify.addHook('onRoute', (routeOptions) => {
  function onPreSerialization(request, reply, payload, done) {
    // Seu código
    done(null, payload)
  }
  // preSerialization pode ser um array ou undefined
  routeOptions.preSerialization = [...(routeOptions.preSerialization || []), onPreSerialization]
})
```

<a name="on-register"></a>

### onRegister
Disparado quando um novo plugin é registrado e um novo encapsulamento de contexto é criado. Esse hook será executado **antes** do registro do código.<br/>
Esse hook pode ser útil se você está desenvolvendo um plugin que precisa ter conhecimento de quando um contexto de plugin é formado e você deseja trabalhar naquele contexto específico.<br/>
**Note que:** Esse hook não será acionado se um plugin estiver dentro de um [`fastify-plugin`](https://github.com/fastify/fastify-plugin).
```js
fastify.decorate('data', [])

fastify.register(async (instance, opts) => {
  instance.data.push('hello')
  console.log(instance.data) // ['hello']

  instance.register(async (instance, opts) => {
    instance.data.push('world')
    console.log(instance.data) // ['hello', 'world']
  }, { prefix: '/hola' })
}, { prefix: '/ciao' })

fastify.register(async (instance, opts) => {
  console.log(instance.data) // []
}, { prefix: '/hello' })

fastify.addHook('onRegister', (instance, opts) => {
  // Cria um novo array a partir de um antigo
  // mas guarda a referência 
  // possibilitando o usuário ter instâncias
  // encapsuladas da proriedade `data`
  instance.data = instance.data.slice()

  // As opções da nova instância registrada.
  console.log(opts.prefix)
})
```

<a name="scope"></a>

### Escopo
Exceto pelos [Hooks de Aplicação](#application-hooks), todos os hooks são encapsulados. Isso significa que você pode decidir quando seus hooks devem executar ao utilizar `register`  como explicado no [Guia de plugins](https://github.com/fastify/docs-portuguese/blob/master/docs/Plugins-Guide.md). Se você definir uma função, ela será vinculada ao contexto correto do Fastify e, a partir daí, você terá total acesso à API Fastify.

```js
fastify.addHook('onRequest', function (request, reply, done) {
  const self = this // Contexto Fastify
  done()
})
```
**Note que:** usando 'arrow function' irá quebrar o vínculo do `this` com a instância do Fastify.

<a name="route-hooks"></a>

## Hooks a nível de rotas
Você pode declarar um ou mais hooks  [onRequest](#onRequest), [onReponse](#onResponse), [preParsing](#preParsing), [preValidation](#preValidation), [preHandler](#preHandler) e [preSerialization](#preSerialization) personalizados que atuarão **unicamente** para aquela rota.
Assim sendo, aqueles hooks sempre serão executados como o último hook na categoria específica do hook.<br/>
Isso pode ser útil se você precisar implementar autenticação, onde os hooks [preParsing](#preParsing) ou [preValidation](#preValidation) são exatamente o que você precisa.
Múltiplos hooks a nível de rotas também podem ser definidos como um array.

```js
fastify.addHook('onRequest', (request, reply, done) => {
  // Seu código
  done()
})

fastify.addHook('onResponse', (request, reply, done) => {
  // Seu código
  done()
})

fastify.addHook('preParsing', (request, reply, done) => {
  // Seu código
  done()
})

fastify.addHook('preValidation', (request, reply, done) => {
  // Seu código
  done()
})

fastify.addHook('preHandler', (request, reply, done) => {
  // Seu código
  done()
})

fastify.addHook('preSerialization', (request, reply, payload, done) => {
  // Seu código
  done(null, payload)
})

fastify.route({
  method: 'GET',
  url: '/',
  schema: { ... },
  onRequest: function (request, reply, done) {
    // Este hook sempre será executado após o hook `onRequest` compartilhado
    done()
  },
  onResponse: function (request, reply, done) {
    // Este hook sempre será executado após o hook `onResponse` compartilhado
    done()
  },
  preParsing: function (request, reply, done) {
    // Este hook sempre será executado após o hook `preParsing` compartilhado
    done()
  },
  preValidation: function (request, reply, done) {
    // Este hook sempre será executado após o hook `preValidation` compartilhado
    done()
  },
  preHandler: function (request, reply, done) {
    // Este hook sempre será executado após o hook `preHandler` compartilhado
    done()
  },
  // // Exemplo com um array. Todos os hooks suportam essa sintaxe.
  //
  // preHandler: [function (request, reply, done) {
  //   // Este hook sempre será executado após o hook `preHandler` compartilhado
  //   done()
  // }],
  preSerialization: (request, reply, payload, done) => {
    // Este hook sempre será executado após o hook `preSerialization` compartilhado
    done(null, payload)
  },
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
})
```

**Note que**: ambas opções também aceitam um array de funções.