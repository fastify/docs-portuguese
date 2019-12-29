<h1 align="center">Fastify</h1>

<a id="typescript"></a>
## TypeScript
O Fastify é enviado com um arquivo de tipos, mas pode ser necessário instalar o `@types/node`, dependendo da versão do Node.js que você está usando.

## Suporte a tipos
Preocupamo-nos com a comunidade TypeScript, e um de nossos membros principais da equipe está atualmente reformulando todos os tipos.
Fazemos o possível para atualizar os tipos com a versão mais recente da API, mas *pode acontecer* que não estejam totalmente sincronizadas. <br/>
Felizmente, esse é o código-fonte aberto e você pode contribuir para corrigi-los. Teremos o maior prazer em aceitar a correção e lançá-la o mais rápido possível no lançamento do patch. Confira as regras de [contribuição](#contribuindo)!

Os plug-ins podem ou não incluir tipos. Veja [Tipos de plugins](#plugin-types) para mais informações.
## Exemplo
Este exemplo de aplicativo TypeScript está alinhado com os exemplos em JavaScript:

```ts
import * as fastify from 'fastify'
import { Server, IncomingMessage, ServerResponse } from 'http'

// Crie um servidor http. Passamos os tipos relevantes para a nossa versão http usada.
// Ao passar tipos, obtemos acesso corretamente aos objetos http subjacentes nas rotas.
// Se usar o http2, passaríamos <http2.Http2Server, http2.Http2ServerRequest, http2.Http2ServerResponse>
// Para https, passe http2.Http2SecureServer ou http.SecureServer em vez de Server.
const server: fastify.FastifyInstance<Server, IncomingMessage, ServerResponse> = fastify({})

const opts: fastify.RouteShorthandOptions = {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          pong: {
            type: 'string'
          }
        }
      }
    }
  }
}

server.get('/ping', opts, (request, reply) => {
  console.log(reply.res) // este é o http.ServerResponse com os tipos corretos!
  reply.code(200).send({ pong: 'it worked!' })
})
```

<a id="generic-parameters"></a>
## Parâmetros Genéricos
Como você pode validar a string de consulta, os parâmetros, o corpo e os cabeçalhos, também é possível substituir os tipos padrão desses valores na interface de solicitação:

```ts
import * as fastify from 'fastify'

const server = fastify({})

interface Query {
  foo?: number
}

interface Params {
  bar?: string
}

interface Body {
  baz?: string
}

interface Headers {
  a?: string
}

const opts: fastify.RouteShorthandOptions = {
  schema: {
    querystring: {
      type: 'object',
      properties: {
        foo: {
          type: 'number'
        }
      }
    },
    params: {
      type: 'object',
      properties: {
        bar: {
          type: 'string'
        }
      }
    },
    body: {
      type: 'object',
      properties: {
        baz: {
          type: 'string'
        }
      }
    },
    headers: {
      type: 'object',
      properties: {
        a: {
          type: 'string'
        }
      }
    }
  }
}

server.get<Query, Params, Headers, Body>('/ping/:bar', opts, (request, reply) => {
  console.log(request.query) // isso é do tipo Query!
  console.log(request.params) // isso é do tipo Params!
  console.log(request.body) // isso é do tipo Body!
  console.log(request.headers) // isso é do tipo Headers!
  reply.code(200).send({ pong: 'it worked!' })
})
```

Todos os tipos genéricos são opcionais, portanto, você também pode transmitir tipos para as partes que você valida com esquemas:
```ts
import * as fastify from 'fastify'

const server = fastify({})

interface Params {
  bar?: string
}

const opts: fastify.RouteShorthandOptions = {
  schema: {
    params: {
      type: 'object',
      properties: {
        bar: {
          type: 'string'
        }
      }
    },
  }
}

server.get<fastify.DefaultQuery, Params, unknown>('/ping/:bar', opts, (request, reply) => {
  console.log(request.query) // isso é do tipo fastify.DefaultQuery!
  console.log(request.params) // isso é do tipo Params!
  console.log(request.body) // isso é do tipo unknown!
  console.log(request.headers) // isso é do tipo fastify.DefaultHeader porque o typescript vai usar um valor default!
  reply.code(200).send({ pong: 'it worked!' })
})

// Como você não validou a string de consulta, o corpo ou os cabeçalhos, seria melhor
// para digitar esses parâmetros como 'unknow'. No entanto, cabe a você. O exemplo abaixo é a
// melhor maneira de impedir que você atire no próprio pé. Em outras palavras, não
// use valores que você não validou.
server.get<unknown, Params, unknown, unknown>('/ping/:bar', opts, (request, reply) => {
  console.log(request.query) // é do tipo unknown!
  console.log(request.params) // é do tipo Params!
  console.log(request.body) // é do tipo unknown!
  console.log(request.headers) // é do tipo unknown!
  reply.code(200).send({ pong: 'it worked!' })
})
```

<a id="http-prototypes"></a>
## HTTP Prototypes
Por padrão, o fastify determinará qual versão do http está sendo usada com base nas opções que você passar para ela. Se por alguma
razão você precisar substituir isso, você pode fazê-lo como mostrado abaixo:
```ts
interface CustomIncomingMessage extends http.IncomingMessage {
  getClientDeviceType: () => string
}

// Passagem de substituições para os http prototypes
const server: fastify.FastifyInstance<http.Server, CustomIncomingMessage, http.ServerResponse> = fastify()

server.get('/ping', (request, reply) => {
  // Acesse nosso método personalizado no prototype http
  const clientDeviceType = request.raw.getClientDeviceType()

  reply.send({ clientDeviceType: `você chamou esse endpoint de um ${clientDeviceType}` })
})
```

Neste exemplo, passamos uma interface `http.IncomingMessage` modificada, uma vez que ela foi estendida em outros lugares da nossa
inscrição.

<a id="contributing"></a>
## Contribuindo
As alterações relacionadas ao TypeScript podem ser consideradas uma de duas categorias:

* [`Core`](#tipagem-principal) - Os tipos incluídos no fastify
* [`Plugins`](#tipagem-de-plugins) - Fastify plugins do ecossistema

Leia nosso arquivo [`CONTRIBUTING.md`](https://github.com/fastify/fastify/blob/master/CONTRIBUTING.md) antes de começar para garantir que tudo corra bem!

<a id="core-types"></a>
### Tipagem principal(Core)
Ao atualizar os tipos principais, você deve fazer um PR neste repositório. Garanta que você:

1. Atualize `examples/typescript-server.ts` para refletir as alterações (se necessário)
2. Atualize `test/types/index.ts` para validar as alterações como esperado
<a id="plugin-types"></a>
### Tipagem de Plugins

Os plugins mantidos e organizados sob a organização fastify no GitHub devem ser fornecidos com as tipologias, assim como o próprio fastify.
Alguns plugins já incluem tipos, mas muitos não. Estamos felizes em aceitar contribuições para esses plugins sem nenhuma digitação, consulte [fastify-cors](https://github.com/fastify/fastify-cors) para obter um exemplo de plugin que vem com suas próprias tipagens.

As digitações para plugins de terceiros podem ser incluídas no plugin ou hospedadas no DefinitelyTyped. Lembre-se, se você criar um plugin para incluir digitações ou publicá-las no DefinitelyTyped! Informações sobre como instalar tipificações do DefinitelyTyped podem ser encontradas aqui (https://github.com/DefinitelyTyped/DefinitelyTyped#npm).

Alguns tipos podem ainda não estar disponíveis, portanto, não tenha vergonha de contribuir.
<a id="authoring-plugin-types"></a>
### Authoring Plugin Types
Typings for many plugins that extend the `FastifyRequest`, `FastifyReply` or `FastifyInstance` objects can be achieved as shown below.

This code shows the typings for the `fastify-static` plugin.
Tipagens para muitos plugins que estendem os objetos `FastifyRequest`,` FastifyReply` ou `FastifyInstance` podem ser obtidas como mostrado abaixo.

Este código mostra tipos para o plugin [`fastify-static`](https://github.com/fastify/fastify-static).
```ts
/// <reference types="node" />

// require fastify typings
import * as fastify from 'fastify';

// importe se necessário http, http2, https typings
import { Server, IncomingMessage, ServerResponse } from "http";
import { Http2SecureServer, Http2Server, Http2ServerRequest, Http2ServerResponse } from "http2";
import * as https from "https";

type HttpServer = Server | Http2Server | Http2SecureServer | https.Server;
type HttpRequest = IncomingMessage | Http2ServerRequest;
type HttpResponse = ServerResponse | Http2ServerResponse;

// extendendo fastify typings
declare module "fastify" {
  interface FastifyReply<HttpResponse> {
    sendFile(filename: string): FastifyReply<HttpResponse>;
  }
}

// declarando tipo de plugin usando fastify.Plugin
declare function fastifyStatic(): fastify.Plugin<
  Server,
  IncomingMessage,
  ServerResponse,
  {
    root: string;
    prefix?: string;
    serve?: boolean;
    decorateReply?: boolean;
    schemaHide?: boolean;
    setHeaders?: (...args: any[]) => void;
    redirect?: boolean;
    wildcard?: boolean | string;

    // Passando sobre o `send`
    acceptRanges?: boolean;
    cacheControl?: boolean;
    dotfiles?: boolean;
    etag?: boolean;
    extensions?: string[];
    immutable?: boolean;
    index?: string[];
    lastModified?: boolean;
    maxAge?: string | number;
  }
>;

declare namespace fastifyStatic {
  interface FastifyStaticOptions {}
}

// exportando tipo de plugin
export = fastifyStatic;
```

Agora você está pronto e pode usar o plugin da seguinte forma:

```ts
import * as Fastify from 'fastify'
import * as fastifyStatic from 'fastify-static'

const app = Fastify()

// as opções aqui são verificadas
app.register(fastifyStatic, {
  acceptRanges: true,
  cacheControl: true,
  decorateReply: true,
  dotfiles: true,
  etag: true,
  extensions: ['.js'],
  immutable: true,
  index: ['1'],
  lastModified: true,
  maxAge: '',
  prefix: '',
  root: '',
  schemaHide: true,
  serve: true,
  setHeaders: (res, pathName) => {
    res.setHeader('some-header', pathName)
  }
})

app.get('/file', (request, reply) => {
  // usando a função recém-definida no FastifyReply
  reply.sendFile('some-file-name')
})
```

Adicionar digitações a todos os nossos plugins é um esforço da comunidade, portanto, fique à vontade para contribuir!
