<h1 align="center">Fastify</h1>

## Testando
Testing is one of the most important parts of developing an application. Fastify is very flexible when it comes to testing and is compatible with most testing frameworks (such as [Tap](https://www.npmjs.com/package/tap), which is used in the examples below).
O teste é uma das partes mais importantes do desenvolvimento de um aplicativo. O Fastify é muito flexível quando se trata de teste e é compatível com a maioria das estruturas de teste (como [Tap](https://www.npmjs.com/package/tap), que é usado nos exemplos abaixo).

<a name="inject"></a>
### Teste com injeção http
O Fastify vem com suporte embutido para injeção de http falsa, graças a [`light-my-request`](https://github.com/fastify/light-my-request).

Para injetar uma solicitação http falsa, use o método `inject`:
```js
fastify.inject({
  method: String,
  url: String,
  query: Object,
  payload: Object,
  headers: Object,
  cookies: Object
}, (error, response) => {
  // your tests
})
```

Os métodos `.inject` também podem ser encadeados, omitindo a função de retorno de chamada:

```js
fastify
  .inject()
  .get('/')
  .headers({ foo: 'bar' })
  .query({ foo: 'bar' })
  .end((err, res) => { // a chamada .end acionará a solicitação
    console.log(res.payload)
  })
```

ou na versão com promise

```js
fastify
  .inject({
    method: String,
    url: String,
    query: Object,
    payload: Object,
    headers: Object,
    cookies: Object
  })
  .then(response => {
    // seus testes
  })
  .catch(err => {
    // handle error
  })
```

Async/Await também é suportado!
```js
try {
  const res = await fastify.inject({ method: String, url: String, payload: Object, headers: Object })
  // your tests
} catch (err) {
  // handle error
}
```

#### Exemplo:

**app.js**
```js
const Fastify = require('fastify')

function buildFastify () {
  const fastify = Fastify()

  fastify.get('/', function (request, reply) {
    reply.send({ hello: 'world' })
  })
  
  return fastify
}

module.exports = buildFastify
```

**test.js**
```js
const tap = require('tap')
const buildFastify = require('./app')

tap.test('GET `/` route', t => {
  t.plan(4)
  
  const fastify = buildFastify()
  
  // No final de seus testes, é altamente recomendável chamar `.close()`
  // para garantir que todas as conexões com serviços externos sejam fechadas.
  t.tearDown(() => fastify.close())

  fastify.inject({
    method: 'GET',
    url: '/'
  }, (err, response) => {
    t.error(err)
    t.strictEqual(response.statusCode, 200)
    t.strictEqual(response.headers['content-type'], 'application/json; charset=utf-8')
    t.deepEqual(response.json(), { hello: 'world' })
  })
})
```
### Testando com um servidor em execução
O Fastify também pode ser testado após iniciar o servidor com `fastify.listen()` ou após inicializar rotas e plugins com `fastify.ready()`.

#### Exemplo:

Usa **app.js** do exemplo anterior.

**test-listen.js** (testando com [`Request`](https://www.npmjs.com/package/request))
```js
const tap = require('tap')
const request = require('request')
const buildFastify = require('./app')

tap.test('GET `/` route', t => {
  t.plan(5)
  
  const fastify = buildFastify()
  
  t.tearDown(() => fastify.close())
  
  fastify.listen(0, (err) => {
    t.error(err)
    
    request({
      method: 'GET',
      url: 'http://localhost:' + fastify.server.address().port
    }, (err, response, body) => {
      t.error(err)
      t.strictEqual(response.statusCode, 200)
      t.strictEqual(response.headers['content-type'], 'application/json; charset=utf-8')
      t.deepEqual(JSON.parse(body), { hello: 'world' })
    })
  })
})
```

**test-ready.js** (testando com [`SuperTest`](https://www.npmjs.com/package/supertest))
```js
const tap = require('tap')
const supertest = require('supertest')
const buildFastify = require('./app')

tap.test('GET `/` route', async (t) => {
  const fastify = buildFastify()

  t.tearDown(() => fastify.close())
  
  await fastify.ready()
  
  const response = await supertest(fastify.server)
    .get('/')
    .expect(200)
    .expect('Content-Type', 'application/json; charset=utf-8')
  t.deepEqual(response.body, { hello: 'world' })
})
```

### Como inspecionar testes de toque
1. Isole seu teste passando na opção `{only: true}`
```javascript
test('should ...', {only: true}, t => ...)
```
2. Run `tap` using `npx`
```bash
> npx tap -O -T --node-arg=--inspect-brk test/<test-file.test.js>
```
- `-O` specifies to run tests with the `only` option enabled
- `-T` specifies not to timeout (while you're debugging)
- `--node-arg=--inspect-brk` will launch the node debugger
3. In VS Code, create and launch a `Node.js: Attach` debug configuration. No modification should be necessary.

`-O` especifica para executar testes com a opção` only` ativada
- `-T` especifica para não atingir o tempo limite (enquanto você estiver depurando)
- `--node-arg=--inspect-brk` iniciará o depurador do node
3. No VS Code, crie e inicie uma configuração de depuração `Node.js: Attach`. Nenhuma modificação deve ser necessária.

Agora você deve poder percorrer seu arquivo de teste (e o restante do `fastify`) no seu editor de código
