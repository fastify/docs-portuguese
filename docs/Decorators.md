<h1 align="center">Fastify</h1>

## Decorators
(Decorators, 'decorador' em tradução livre, é um conceito onde uma instância é incrementada com novas propriedades)

Se você precisa adicionar funcionalidades a instância do Fastify, a API `decorate` é o que você precisa.

A API te possibilita adicionar novas propriedades a instância do Fastify. Os valores possíveis não tem restrições quanto ao tipo e podem ser por exemplo `funcões`, `objetos` ou `string`. 

<a name="usage"></a>

### Utilização
<a name="decorate"></a>

**decorate**
Basta chamar a API `decorate` e passar o nome da propiedade e seu respectivo valor.
```js
fastify.decorate('utility', () => {
  // Alguma coisa útil
})
```

Como mencionado acima, você tam´bem pode 'decorar' a instância com valores que não são funções:
```js
fastify.decorate('conf', {
  db: 'algum.db',
  port: 3000
})
```

Uma vez 'decorada' a instância você poderá acessar o novo valor utilizando o nome que você informou como parâmetro:
```js
fastify.utility()

console.log(fastify.conf.db)
```

<a name="decorate-reply"></a>

**decorateReply**
Como o nome sugere, essa API pode ser utilizada para adicionar métodos ao objeto interno `Reply`. Basta chamar a API `decorateReply` e passar o nome da propriedade e seu respectivo valor:
```js
fastify.decorateReply('utility', function () {
  // Alguma coisa útil
})
```

**Note que**: utilizar a notação `=>` (arrow function) quebrará o vínculo com o escopo da propriedade `this` com a instância do `Reply` do Fastify.

<a name="decorate-request"></a>

**decorateRequest**
No caso de querer decorar o objeto interno `Request` é essa API que deverá utilizar. Basta chamar a API `decorateRequest` e passar o nome da propriedade e seu respectivo valor:
```js
fastify.decorateRequest('utility', function () {
  // Alguma coisa útil
})
```

**Note que**: utilizar a notação `=>` (arrow function) quebrará o vínculo com o escopo da propriedade `this` com a instância do `Request` do Fastify.

<a name="decorators-encapsulation"></a>

#### Decoradores e Encpsulamento

Se você define um decorador (utilizando `decorate`, `decorateRequest` ou `decorateReply`) com o mesmo nome mais de uma vez no mesmo plugin **encapsulado**, o Fastify irá disparar uma `exception`.

Como o exemplo abaixo irá:

```js
const server = require('fastify')()

server.decorateReply('view', function (template, args) {
  // Maravilhoso mecanismo de renderização de 'view'
})

server.get('/', (req, reply) => {
  reply.view('/index.html', { hello: 'world' })
})

// Em algum outro lugar do seu código, ao definir
// outro decorador de 'view'. Que irá despar uma `exception`.
server.decorateReply('view', function (template, args) {
  // Outro mecanismo de renderização
})

server.listen(3000)
```


Todavia, o exemplo abaixo ***não*** irá disparar `exception`:

```js
const server = require('fastify')()

server.decorateReply('view', function (template, args) {
  // Maravilhoso mecanismo de renderização de 'view'.
})

server.register(async function (server, opts) {
  // Nós adicionamos um decorador view ao plugin encapsulado 
  // atual. Isso não irá disparar uma `exception` uma vez que 
  // esse novo decorador refere-se e está encapsulado a este plugin 
  // específico.
  server.decorateReply('view', function (template, args) {
    // Outro mecanismo de renderização 
  })

  server.get('/', (req, reply) => {
    reply.view('/index.page', { hello: 'world' })
  })
}, { prefix: '/bar' })

server.listen(3000)
```

<a name="getters-setters"></a>

#### Getters and Setters

Decorators aceitam os objetos especiais "getter/setter". Esses objetos tem funções com os nomes `getter` e `setter` (apesar da função `setter` ser opcional). Isso permite definir propriedades via decorators. Por exemplo: 

```js
fastify.decorate('foo', {
  getter () {
    return 'a getter'
  }
})
```

Isso irá definir a propriedade `foo` na instância do *Fastify*:

```js
console.log(fastify.foo) // 'a getter'
```

<a name="usage_notes"></a>

#### Usage Notes
`decorateReply` e `decorateRequest` são utilizados para modificar os construtores `Reply` e `Request`, respectivamente, adicionando métodos ou propriedades. Para atualizar essas propriedades você deve acessar diretamente a propriedade dos objetos `Reply` ou `Request`.

Como por exemplo, vamos adicionar a propriedade `user` ao objeto `Request`:

```js
// Decorando o request com a propriedade 'user'
fastify.decorateRequest('user', '')

// Atualizando nossa propriedade
fastify.addHook('preHandler', (req, reply, done) => {
  req.user = 'Bob Dylan'
  done()
})
// E, finalmente, acessando seu valor: 
fastify.get('/', (req, reply) => {
  reply.send(`Olá, ${req.user}!`)
})
```
**Note que**: a utilização de `decorateReply` e `decorateRequest` e opcional neste caso, mas irá posibilitar ao Fastify otimizar para desempenho.

<a name="sync-async"></a>

#### Sync and Async
`decorate` é uma API *síncrona*. Se você necessita adicionar um decorator que tenha algum processamento *asíncrono*, o Fastify pode ser carregado antes que seu decorador esteja pronto. Para evitar esse problema você deve utilizar a API `register` combinada com o `fastify-plugin`. Para aprender mais sobre, confira também a documentação [Plugins](https://github.com/fastify/docs-portuguese/blob/master/docs/Plugins.md).

<a name="dependencies"></a>

#### Dependências
Se seu decorador depende de outro decorador você pode, facilmente, declarar o outro decorador como uma dependência. Você só precisa adicionar um array de string (representando os nomes dos decoradores que seu decorador depende) como terceiro parâmetro: :
```js
fastify.decorate('utility', fn, ['greet', 'log'])
```

Se uma dependência não é satisfeita, `decorate` irá disparar uma `exception`, mais não se preocupe: a checagem de dependências é executada antes do servidor iniciar, então isso não irá acontecer em tempo de execução.

<a name="has-decorator"></a>

#### hasDecorator
Você pode verificar a presença de um decorador com o método `hasDecorator`:
```js
fastify.hasDecorator('utility')
```

<a name="has-request-decorator"></a>

#### hasRequestDecorator
Você pode verificar a presença de um decorador de Request com o método `hasRequestDecorator`:
```js
fastify.hasRequestDecorator('utility')
```

<a name="has-reply-decorator"></a>

#### hasReplyDecorator
Você pode verificar a presença de um decorador de Reply com o método `hasReplyDecorator` API:
```js
fastify.hasReplyDecorator('utility')
```
