<h1 align="center">Fastify</h1>

## HTTP2

_Fastify_ oferece **suporte experimental** para HTTP2 a partir do Node
8.8.0, que inclui HTTP2 sem uma flag. _Fastify_ suporta HTTP2
em HTTPS ou em plaintext(texto sem formatação). Observe que o HTTP2 está disponível apenas para versões de node >= `8.8.1`.

Atualmente, nenhuma das APIs específicas do HTTP2 está disponível através
_Fastify_, mas o `req` e` res` do Node podem ser acessados através da nossa
Interface `Request` e `Reply`. PRs são bem-vindos.

### Secure (HTTPS)

O HTTP2 é suportado em todos os navegadores modernos __apenas em um servidor com.
conexão segura__:

```js
'use strict'

const fs = require('fs')
const path = require('path')
const fastify = require('fastify')({
  http2: true,
  https: {
    key: fs.readFileSync(path.join(__dirname, '..', 'https', 'fastify.key')),
    cert: fs.readFileSync(path.join(__dirname, '..', 'https', 'fastify.cert'))
  }
})

fastify.get('/', function (request, reply) {
  reply.code(200).send({ hello: 'world' })
})

fastify.listen(3000)
```

ALPN negotiation allows support for both HTTPS and HTTP/2 over the same socket.
Node core `req` and `res` objects can be either [HTTP/1](https://nodejs.org/api/http.html)
or [HTTP/2](https://nodejs.org/api/http2.html).
_Fastify_ supports this out of the box:

A negociação ALPN permite suporte para HTTPS e HTTP/2 no mesmo socket.
Os objetos `req` e `res` do Node podem ser [HTTP/1](https://nodejs.org/api/http.html)
ou [HTTP/2](https://nodejs.org/api/http2.html).
_Fastify_ ja suporta isso:

```js
'use strict'

const fs = require('fs')
const path = require('path')
const fastify = require('fastify')({
  http2: true,
  https: {
    allowHTTP1: true, // fallback support for HTTP1
    key: fs.readFileSync(path.join(__dirname, '..', 'https', 'fastify.key')),
    cert: fs.readFileSync(path.join(__dirname, '..', 'https', 'fastify.cert'))
  }
})

// essa rota pode ser acessada em ambos protocolos
fastify.get('/', function (request, reply) {
  reply.code(200).send({ hello: 'world' })
})

fastify.listen(3000)
```

Você pode testar seu novo servidor com:

```
$ npx h2url https://localhost:3000
```

### Simples ou inseguro

Se você estiver criando microsserviços, poderá conectar-se ao HTTP2 em plaintext (texto sem formatação), no entanto, isso não é suportado pelos navegadores.

```js
'use strict'

const fastify = require('fastify')({
  http2: true
})

fastify.get('/', function (request, reply) {
  reply.code(200).send({ hello: 'world' })
})

fastify.listen(3000)
```

Você pode testar seu novo servidor com:

```
$ npx h2url http://localhost:3000
```
