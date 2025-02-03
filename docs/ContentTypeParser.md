<h1 align="center">Fastify</h1>

## `Content-Type` Parser
Nativamente, o Fastify suporta apenas os tipos de conteúdo `application/json` e `text/plain`. O conjunto de caracteres padrão é `utf-8`. Se você precisar oferecer suporte a diferentes content-types, poderá usar a API `addContentTypeParser`. *O JSON padrão e/ou o analisador de texto sem formatação podem ser alterados.*

Como nas outras APIs, `addContentTypeParser` é encapsulado no escopo em que é declarado. Isso significa que se você o declarar no escopo raiz, ele estará disponível em qualquer lugar, enquanto que se o declarar dentro de um plug-in, estará disponível apenas nesse escopo e em seus filhos.

O Fastify adiciona automaticamente a carga útil da solicitação analisada ao objeto [Fastify request](https://github.com/fastify/fastify/blob/main/docs/Request.md) que você pode acessar com `request.body`.

### Uso
```js
fastify.addContentTypeParser('application/jsoff', function (req, done) {
  jsoffParser(req, function (err, body) {
    done(err, body)
  })
})

// Adicionando multiplos content-types na mesma função
fastify.addContentTypeParser(['text/xml', 'application/xml'], function (req, done) {
  xmlParser(req, function (err, body) {
    done(err, body)
  })
})

// Async é também suportado nas versões do Node >= 8.0.0
fastify.addContentTypeParser('application/jsoff', async function (req) {
  var res = await new Promise((resolve, reject) => resolve(req))
  return res
})
```

Você também pode usar a API `hasContentTypeParser` para descobrir se já existe um analisador de content-type específico.

```js
if (!fastify.hasContentTypeParser('application/jsoff')){
  fastify.addContentTypeParser('application/jsoff', function (req, done) {
  // Código para analisar o corpo/carga útil da solicitação para o tipo de conteúdo especificado
  })
}
```

#### Body Parser
Você pode analisar o corpo de uma solicitação de duas maneiras. O primeiro é mostrado acima: você adiciona um analisador de content-type personalizado e manipula o fluxo de solicitações. No segundo, você deve passar uma opção `parseAs` para a API `addContentTypeParser`, na qual declara como deseja obter o corpo. Pode ser do tipo `string` ou `buffer`. Se você usar a opção `parseAs`, o Fastify manipulará internamente o fluxo e executará algumas verificações, como o [tamanho máximo](https://github.com/fastify/fastify/blob/main/docs/Server.md# limite do corpo da fábrica) do corpo e o comprimento do conteúdo. Se o limite for excedido, o analisador personalizado não será chamado.

```js
fastify.addContentTypeParser('application/json', { parseAs: 'string' }, function (req, body, done) {
  try {
    var json = JSON.parse(body)
    done(null, json)
  } catch (err) {
    err.statusCode = 400
    done(err, undefined)
  }
})

Como você pode ver, agora a assinatura da função é `(req, body, done)` em vez de `(req, done)`.

Veja [`exemplo/parser.js`](https://github.com/fastify/fastify/blob/main/examples/parser.js) para um exemplo.

##### Custom Parser Options

+ `parseAs` (string): `string` ou `buffer` para designar como os dados recebidos devem ser coletados. Padrão: `buffer`.
+ `bodyLimit` (número): o tamanho máximo da carga útil, em bytes, que o analisador personalizado aceitará. O padrão é o limite global do corpo passado para a [`Função Fastify factory`](https://github.com/fastify/fastify/blob/main/docs/Server.md#bodylimit).

#### Receba tudo

Existem alguns casos em que você precisa capturar todas as solicitações, independentemente do tipo de conteúdo. Com o Fastify, você pode apenas usar o content-type `*`.

```js
fastify.addContentTypeParser('*', function (req, done) {
  var data = ''
  req.on('data', chunk => { data += chunk })
  req.on('end', () => {
    done(null, data)
  })
})
```
Usando isso, todas as solicitações que não possuem um analisador de tipo de conteúdo correspondente serão tratadas pela função especificada.

Isso também é útil para canalizar o fluxo de solicitações. Você pode definir um analisador de conteúdo como:

```js
fastify.addContentTypeParser('*', function (req, done) {
  done()
})
```

e acesse a solicitação HTTP principal diretamente para canalizá-la para onde você deseja:

```js
app.post('/hello', (request, reply) => {
  reply.send(request.req)
})
```

Aqui está um exemplo completo que registra os objetos [json line](http://jsonlines.org/) recebidos:

```js
const split2 = require('split2')
const pump = require('pump')

fastify.addContentTypeParser('*', (req, done) => {
  done(null, pump(req, split2(JSON.parse)))
})

fastify.route({
  method: 'POST',
  url: '/api/log/jsons',
  handler: (req, res) => {
    req.body.on('data', d => console.log(d)) // log todo objeto recebido
  }
})
 ```

For piping file uploads you may want to checkout [this plugin](https://github.com/fastify/fastify-multipart).
