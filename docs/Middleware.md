<h1 align="center">Fastify</h1>

## Middleware
(Middleware é um conceito que pode ser entendido como um intermediador, no nosso caso, entre o Fastify e nossa aplicação)

O Fastify fornece um mecanismo de middleware nativamente via a biblioteca [middie](https://github.com/fastify/middie), que é compatível com middleware do [Express](https://expressjs.com/) e do [Restify](http://restify.com/) middleware.

*Para entender melhor quando um middleware é executado, confira a página [Ciclo de vida](https://github.com/fastify/docs-portuguese/blob/main/docs/Lifecycle.md) do Fastify.*

O mecanismo de middleware do Fastify não suporta a sintaxe completa `middleware(err, req, res, next)`, pelo fato de que o tratamento de erros é feito dentro do Fastify.
Além disso, métodos para incrementar versões de `req` e `res` adicionados pelo Express e pelo Restify não são suportados no mecanismo de middlewares do Fastify.

Além disso se você está utilizando middlewares que empacotam vários e diferentes middlewares menores, como é o caso do [*helmet*](https://helmetjs.github.io/), recomendamos usar os módulos individualmente para melhor desempenho.

```js
fastify.use(require('cors')())
fastify.use(require('dns-prefetch-control')())
fastify.use(require('frameguard')())
fastify.use(require('hide-powered-by')())
fastify.use(require('hsts')())
fastify.use(require('ienoopen')())
fastify.use(require('x-xss-protection')())
```

Ou no caso específico do *helmet*, pode ainda utlizar a versão de integração otimizada para o Fastify [*fastify-helmet*](https://github.com/fastify/fastify-helmet):

```js
const fastify = require('fastify')()
const helmet = require('fastify-helmet')

fastify.register(helmet)
```

E lembre-se que middleware podem ser encapsulados, isso significa que você pode decidir quando o middleware deve ser executado utilizando `register` como explicado no [Guia de plugins](https://github.com/fastify/docs-portuguese/blob/main/docs/Plugins-Guide.md).

O mecanismo de middlewares do Fastify middleware também não expõe o método `send` ou outros métodos específicos da instância de resposta [Reply]('./Reply.md' "Reply") do Fastify. Isso porque internamente o Fastify encapsula as instâncias `req` e `res` recebidas do Node utilizando respectivamente as instâncias [Request](./Request.md "Request") e [Reply](./Reply.md "Reply"), mas isso é feito depois da fase de middleware. Se você precisa de criar um middleware, você precisa utilizar as instâncias do Node `req` e `res`. Caso contrário utilize o hook `preHandler` que já conterá as instâncias [Request](./Request.md "Request") and [Reply](./Reply.md "Reply") do Fastify. Para mais informações, confira [Hooks](./Hooks.md "Hooks").

<a name="restrict-usage"></a>

#### Restringindo a execução de middleware para endpoints específicos
Se você deseja executar um middleware somente em alguma endpoint específico, simplesmente informe como primeiro parâmetro do método `use` e 'é isso aí'!

*Note que essa restrição não suporta endpoints com parâmetros (ex: `/user/:id/comments`) e curingas não são suportados ao utilizar múltiplos endpoints.*

```js
const path = require('path')
const serveStatic = require('serve-static')

// Endpoint simple e individual
fastify.use('/css', serveStatic(path.join(__dirname, '/assets')))

// Enpoint individual com curinga
fastify.use('/css/*', serveStatic(path.join(__dirname, '/assets')))

// Múltiplos endpoints
fastify.use(['/css', '/js'], serveStatic(path.join(__dirname, '/assets')))
```

<a name="express-middleware"></a>

#### Compatibilidade com Express middleware
O Express modifica profundamente o prototype dos objetos nativos Request and Response do Node,
então o Fastify não tem como garantir compatibilidade integral. 
Também não irão funcionar, certas funcionalidades específicas do Express como `res.sendFile()`, `res.send()` or `express.Router()`. Por exemplo, [cors](https://github.com/expressjs/cors) é compatível enquanto [passport](https://github.com/jaredhanson/passport) não é.
