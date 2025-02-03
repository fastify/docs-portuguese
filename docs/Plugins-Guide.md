<h1 align="center">Fastify</h1>
# O guia do mochileiro para plugins

Antes de mais nada, `NÃO ENTRE EM PÂNICO!`

O Fastify foi construído desde o início para ser um sistema extremamente modular. Criamos uma API poderosa que permite adicionar métodos e utilitários ao Fastify, criando um espaço para nome. Criamos um sistema que cria um modelo de encapsulamento que permite dividir seu aplicativo em vários microsserviços a qualquer momento, sem a necessidade de refatorar o aplicativo inteiro.

**Índice**
- [Register](#register)
- [Decoradores](#decoradores)
- [Ganchos](#ganchos)
- [Middlewares](#middlewares)
- [Como lidar com encapsulamento e distribuição](#distribuição)
- [suporte ESM](#suporte-esm)
- [Manipular erros](#manipular-erros)
- [Vamos começar!](#vamos-comecar)

<a name="register"></a>

## Register
Como no JavaScript, onde tudo é um objeto, no Fastify tudo é um plugin. <br>
Suas rotas, seus utilitários e assim por diante são todos plugins. Para adicionar um novo plug-in, qualquer que seja sua funcionalidade, no Fastify você possui uma API agradável e exclusiva: [`register`](https://github.com/fastify/fastify/blob/main/docs/Plugins.md) .

```js
fastify.register (
  require ('./ my-plugin'),
  {opções}
)
```
O `register` cria um novo contexto Fastify, o que significa que, se você executar alguma alteração na instância Fastify, essas alterações não serão refletidas nos ancestrais do contexto. Em outras palavras, encapsulamento!

**Por que o encapsulamento é importante?** <br>
Bem, digamos que você esteja criando uma nova inicialização disruptiva, o que você faz? Você cria um servidor de API com todas as suas coisas, tudo no mesmo lugar, um monólito! <br>
Ok, você está crescendo muito rápido e deseja alterar sua arquitetura e experimentar microsserviços. Geralmente, isso implica em uma enorme quantidade de trabalho, devido às dependências cruzadas e à falta de separação na base de código. <br>
Fastify ajuda você a esse respeito. Graças ao modelo de encapsulamento, ele evita completamente as dependências cruzadas e ajuda a estruturar seu código em blocos coesos.

**Vamos voltar a como usar corretamente o `register`.** <br>
Como você provavelmente sabe, os plugins necessários devem expor uma única função com a seguinte assinatura

```js
module.exports = function (fastify, options, done) {}
```
Onde `fastify` é a instância Fastify encapsulada, `options` é o objeto de opções e `done` é a função que você **deve** chamar quando seu plug-in estiver pronto.

O modelo de plug-in do Fastify é totalmente reentrante e baseado em gráficos, ele lida com código assíncrono sem problemas e aplica a carga e a ordem de fechamento dos plug-ins. *Como?* Que bom que você perguntou, confira [`avvio`](https://github.com/mcollina/avvio)! O Fastify começa a carregar o plugin __depois__ do `.listen()`, `.inject()` ou `.ready()` são chamados.

Dentro de um plugin, você pode fazer o que quiser, registrar rotas, utilitários (veremos isso em um momento) e fazer registros aninhados, lembre-se de chamar `concluído` quando tudo estiver configurado!
```js
module.exports = function (fastify, options, done) {
  fastify.get('/plugin', (request, reply) => {
    reply.send({ hello: 'world' })
  })

  done()
}
```
Bem, agora você sabe como usar a API `register` e como ela funciona, mas como adicionamos novas funcionalidades ao Fastify e, melhor ainda, as compartilhamos com outros desenvolvedores?

<a name="decorators"></a>
## Decoradores

Ok, digamos que você escreveu um utilitário tão bom que decidiu disponibilizá-lo juntamente com todo o seu código. Como você faria? Provavelmente algo como o seguinte:

```js
// your-awesome-utility.js
module.exports = function (a, b) {
  return a + b
}
```
```js
const util = require('./your-awesome-utility')
console.log(util('that is ', 'awesome'))
```
E agora você importará seu utilitário em todos os arquivos em que você precisar. (E não esqueça que você provavelmente também precisará dele em seus testes).

O Fastify oferece uma maneira mais elegante e confortável de fazer isso **decoradores**.
Criar um decorador é extremamente fácil, basta usar a API [`decorate`](https://github.com/fastify/fastify/blob/main/docs/Decorators.md):
```js
fastify.decorate('util', (a, b) => a + b)
```
Agora você pode acessar seu utilitário apenas chamando `fastify.util` sempre que precisar - mesmo dentro do seu teste. <br>
E aqui começa a mágica; você se lembra como agora estávamos falando sobre encapsulamento? Bem, o uso de `register` e` decorate` em conjunto permite exatamente isso, deixe-me mostrar um exemplo para esclarecer isso:
```js
fastify.register((instance, opts, done) => {
  instance.decorate('util', (a, b) => a + b)
  console.log(instance.util('that is ', 'awesome'))

  done()
})

fastify.register((instance, opts, done) => {
  console.log(instance.util('that is ', 'awesome')) // This will throw an error

  done()
})
```
Dentro do segundo registro, chamada `instance.util` gerará um erro, porque o `util` existe apenas dentro do primeiro contexto do registro. <br>
Vamos voltar por um momento e aprofundar isso: toda vez que você usa a API `register`, é criado um novo contexto que evita as situações negativas mencionadas acima.

Observe que o encapsulamento se aplica aos antepassados e irmãos, mas não aos filhos.
```js
fastify.register((instance, opts, done) => {
  instance.decorate('util', (a, b) => a + b)
  console.log(instance.util('that is ', 'awesome'))

  fastify.register((instance, opts, done) => {
    console.log(instance.util('that is ', 'awesome')) // This will not throw an error
    done()
  })

  done()
})

fastify.register((instance, opts, done) => {
  console.log(instance.util('that is ', 'awesome')) // This will throw an error

  done()
})
```
*Leve a mensagem para casa: se você precisar de um utilitário disponível em todas as partes do seu aplicativo, verifique se ele está declarado no escopo raiz do seu aplicativo. Se isso não for uma opção, você pode usar o utilitário `fastify-plugin` como descrito [aqui](#distribuição).*

`decorate` não é a única API que você pode usar para estender a funcionalidade do servidor, você também pode usar `decorateRequest` e `decorateReply`.

*`decorateRequest` e `decorateReply`? Por que precisamos deles se já temos 'decorate'?* <br>
Boa pergunta, nós os adicionamos para tornar o Fastify mais amigável ao desenvolvedor. Vamos ver um exemplo:
```js
fastify.decorate('html', payload => {
  return generateHtml(payload)
})

fastify.get('/html', (request, reply) => {
  reply
    .type('text/html')
    .send(fastify.html({ hello: 'world' }))
})
```
Isso funciona, mas poderia ser melhor!
```js
fastify.decorateReply('html', function (payload) {
  this.type('text/html') // This is the 'Reply' object
  this.send(generateHtml(payload))
})

fastify.get('/html', (request, reply) => {
  reply.html({ hello: 'world' })
})
```
E da mesma forma podemos fazer isso com o objeto `request`:
```js
fastify.decorate('getHeader', (req, header) => {
  return req.headers[header]
})

fastify.addHook('preHandler', (request, reply, done) => {
  request.isHappy = fastify.getHeader(request.raw, 'happy')
  done()
})

fastify.get('/happiness', (request, reply) => {
  reply.send({ happy: request.isHappy })
})
```
Novamente, isso funciona, mas poderia ser melhor!
```js
fastify.decorateRequest('setHeader', function (header) {
  this.isHappy = this.headers[header]
})

fastify.decorateRequest('isHappy', false) // This will be added to the Request object prototype, yay speed!

fastify.addHook('preHandler', (request, reply, done) => {
  request.setHeader('happy')
  done()
})

fastify.get('/happiness', (request, reply) => {
  reply.send({ happy: request.isHappy })
})
```

Vimos como estender a funcionalidade do servidor e como lidar com o sistema de encapsulamento, mas e se você precisar adicionar uma função que deve ser executada sempre que o servidor "[emitir] (https://github.com/fastify/fastify/blob/main/docs/Lifecycle.md)" um evento?

<a name="hooks"></a>
## Ganchos
Você acabou de criar um utilitário incrível, mas agora é necessário executá-lo para cada solicitação, é o que você provavelmente fará:
```js
fastify.decorate('util', (request, key, value) => { request[key] = value })

fastify.get('/plugin1', (request, reply) => {
  fastify.util(request, 'timestamp', new Date())
  reply.send(request)
})

fastify.get('/plugin2', (request, reply) => {
  fastify.util(request, 'timestamp', new Date())
  reply.send(request)
})
```
Acho que todos concordamos que isso é terrível. Código repetido, legibilidade terrível e não pode ser dimensionado.

Então, o que você pode fazer para evitar esse problema irritante? Sim, você está certo, use um [gancho](https://github.com/fastify/fastify/blob/main/docs/Hooks.md)! <br>

```js
fastify.decorate('util', (request, key, value) => { request[key] = value })

fastify.addHook('preHandler', (request, reply, done) => {
  fastify.util(request, 'timestamp', new Date())
  done()
})

fastify.get('/plugin1', (request, reply) => {
  reply.send(request)
})

fastify.get('/plugin2', (request, reply) => {
  reply.send(request)
})
```
Agora, para cada solicitação, você executará seu utilitário. Obviamente, você pode registrar quantos ganchos precisar. <br>
Às vezes você deseja um gancho que deve ser executado apenas para um subconjunto de rotas, como você pode fazer isso? Sim, encapsulamento!

```js
fastify.register((instance, opts, done) => {
  instance.decorate('util', (request, key, value) => { request[key] = value })

  instance.addHook('preHandler', (request, reply, done) => {
    instance.util(request, 'timestamp', new Date())
    done()
  })

  instance.get('/plugin1', (request, reply) => {
    reply.send(request)
  })

  done()
})

fastify.get('/plugin2', (request, reply) => {
  reply.send(request)
})
```
Agora seu gancho funcionará apenas para a primeira rota!

Como você provavelmente já deve ter percebido, `request` e` reply` não são os objetos padrão Nodejs *request* e *response*, mas os objetos do Fastify. <br>

<a name="middleware"></a>
## Middleware
Fastify [suporta] (https://github.com/fastify/fastify/blob/main/docs/Middleware.md) Express/Restify/Connect middleware pronto para uso, o que significa que você pode simplesmente entrar seu código antigo e funcionará! *(mais rápido, a propósito)* <br>
Digamos que você esteja chegando do Express e já tenha um Middleware que faça exatamente o que você precisa e não queira refazer todo o trabalho.
Como podemos fazer isso? Confira nosso mecanismo de middleware, [middie](https://github.com/fastify/middie).
```js
const yourMiddleware = require('your-middleware')
fastify.use(yourMiddleware)
```

<a name="distribuição"></a>
## Como lidar com encapsulamento e distribuição
Perfeito, agora você conhece (quase) todas as ferramentas que você pode usar para estender o Fastify. Mas é provável que você tenha encontrado um grande problema: como a distribuição é tratada?

A maneira preferida de distribuir um utilitário é agrupar todo o seu código dentro de um `register`, dessa forma, seu plug-in pode suportar bootstrapping assíncrono *(já que `decorate` é uma API síncrona)*, no caso de uma conexão com o banco de dados, por exemplo.

*Espere o que? Você não me disse que o `register` cria um encapsulamento e que as coisas que eu crio dentro não estarão disponíveis fora?* <br>
Eu disse isso. Mas o que eu não disse foi que você pode dizer ao Fastify para evitar esse comportamento, com o módulo [`fastify-plugin`](https://github.com/fastify/fastify-plugin).
```js
const fp = require('fastify-plugin')
const dbClient = require('db-client')

function dbPlugin (fastify, opts, done) {
  dbClient.connect(opts.url, (err, conn) => {
    fastify.decorate('db', conn)
    done()
  })
}

module.exports = fp(dbPlugin)
```
Você também pode dizer ao `fastify-plugin` para verificar a versão instalada do Fastify, caso precise de uma API específica.

Como mencionamos anteriormente, o Fastify começa a carregar seus plugins __after__ `.listen()`, `.inject()` ou `.ready()` são chamados e, como tal, __after__ foram declarados. Isso significa que, embora o plug-in possa injetar variáveis na instância de fastify externa via [`decorate`](https://github.com/fastify/fastify/blob/main/docs/Decorators.md), as variáveis decoradas não vão estar acessível antes de chamar `.listen()`, `.inject()` ou `.ready()`.

Caso você confie em uma variável injetada por um plug-in anterior e queira passar isso no argumento `options` do` register`, você pode fazer isso usando uma função em vez de um objeto:
```js
const fastify = require('fastify')()
const fp = require('fastify-plugin')
const dbClient = require('db-client')

function dbPlugin (fastify, opts, done) {
  dbClient.connect(opts.url, (err, conn) => {
    fastify.decorate('db', conn)
    done()
  })
}

fastify.register(fp(dbPlugin), { url: 'https://example.com' })
fastify.register(require('your-plugin'), parent => {
  return { connection: parent.db, otherOption: 'foo-bar' }
})
```
No exemplo acima, a variável `parent` da função transmitida como o segundo argumento do `register` é uma cópia da **instância de fastify externa** na qual o plug-in foi registrado. Isso significa que somos capazes de acessar todas as variáveis que foram injetadas pelos plugins anteriores na ordem da declaração.

<a name="suporte-esm"></a>
## Suporte ESM

O ESM também é suportado a partir do [Node.js `v13.3.0`](https://nodejs.org/api/esm.html) e acima! Basta exportar seu plugin como módulo ESM e você estará pronto!

```js
// plugin.mjs
async function plugin (fastify, opts) {
  fastify.get('/', async (req, reply) => {
    return { hello: 'world' }
  })
}

export default plugin
```

<a name="manipular-erros"></a>
## Manipular erros
Pode acontecer que um dos seus plugins falhe durante a inicialização. Talvez você espere e tenha uma lógica personalizada que será acionada nesse caso. Como você pode implementar isso?
A API `after` é o que você precisa. O `after` simplesmente registra um retorno de chamada que será executado logo após um registro e pode levar até três parâmetros. <br>
O retorno de chamada muda com base nos parâmetros que você está fornecendo:

1. Se nenhum parâmetro for fornecido ao retorno de chamada e houver um erro, esse erro será passado para o próximo manipulador de erros.
1. Se um parâmetro for fornecido ao retorno de chamada, esse parâmetro será o objeto de erro.
1. Se dois parâmetros forem fornecidos ao retorno de chamada, o primeiro será o objeto de erro, o segundo será o retorno de chamada concluído.
1. Se três parâmetros forem fornecidos ao retorno de chamada, o primeiro será o objeto de erro, o segundo será o contexto de nível superior, a menos que você tenha especificado servidor e substituição, nesse caso, o contexto será o que a substituição retornará, e o terceiro, o retorno de chamada concluído.

Vamos ver como usá-lo:
```js
fastify
  .register(require('./database-connector'))
  .after(err => {
    if (err) throw err
  })
```

<a name="vamos-comecar"></a>
## Vamos começar!
Impressionante, agora você sabe tudo o que precisa saber sobre o Fastify e seu sistema de plugins para começar a criar seu primeiro plug-in. Por favor, conte-nos! Nós o adicionaremos à seção [*ecossistema*](https://github.com/fastify/fastify#ecosystem) da nossa documentação!

Se você quiser ver algum exemplo do mundo real, confira:
- [`ponto de vista`](https://github.com/fastify/point-of-view)
Suporte de plug-in de renderização de modelos (*ejs, pug, guidão, marko*) para o Fastify.
- [`fastify-mongodb`](https://github.com/fastify/fastify-mongodb)
Aperte o plug-in de conexão MongoDB, com isso você poderá compartilhar o mesmo pool de conexões MongoDB em todas as partes do servidor.
- [`fastify-multipart`](https://github.com/fastify/fastify-multipart)
Suporte multipartes para Fastify
- [`fastify-helmet`](https://github.com/fastify/fastify-helmet) Cabeçalhos de segurança importantes para o Fastify


*Você sente que algo está faltando aqui? Nos informe! :)*
