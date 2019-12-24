<h1 align="center">Fastify</h1>

## Validação e Serialização
Fastify usa uma abordagem baseada em esquema e, mesmo que não seja obrigatório, recomendamos o uso de [JSON Schema](http://json-schema.org/) para validar suas rotas e serializar suas saídas.
Internamente, o Fastify compila o esquema em uma função de alto desempenho.


> ## ⚠  Aviso de segurança
> Trate a definição do esquema como código do aplicativo.
> Como os recursos de validação e serialização avaliam dinamicamente
> código com `new Function()`, não é seguro usar
> esquemas fornecidos pelo usuário. Veja [Ajv](http://npm.im/ajv) e
> [fast-json-stringify](http://npm.im/fast-json-stringify) para obter mais detalhes.

<a name="validation"></a>
### Validação
A validação de rota depende internamente do [Ajv](https://www.npmjs.com/package/ajv), que é um validador de esquema JSON de alto desempenho. A validação da entrada é muito fácil: basta adicionar os campos que você precisa dentro do esquema de rota e pronto! As validações suportadas são:
- `body`: valida o corpo da solicitação se for um POST ou PUT.
- `querystring` ou` query`: valida a string de consulta. Pode ser um objeto JSON Schema completo (com uma propriedade `type` de `'object'` e um objeto `'properties'` contendo parâmetros) ou uma variação mais simples na qual os atributos` type` e `properties` são perdidos e os parâmetros da consulta estão listados no nível superior (veja o exemplo abaixo).
- `params`: valida os parâmetros da rota.
- `headers`: valida os cabeçalhos da solicitação.

Exemplo:
```js
const bodyJsonSchema = {
  type: 'object',
  required: ['requiredKey'],
  properties: {
    someKey: { type: 'string' },
    someOtherKey: { type: 'number' },
    requiredKey: {
      type: 'array',
      maxItems: 3,
      items: { type: 'integer' }
    },
    nullableKey: { type: ['number', 'null'] }, // ou { type: 'number', nullable: true }
    multipleTypesKey: { type: ['boolean', 'number'] },
    multipleRestrictedTypesKey: {
      oneOf: [
        { type: 'string', maxLength: 5 },
        { type: 'number', minimum: 10 }
      ]
    },
    enumKey: {
      type: 'string',
      enum: ['John', 'Foo']
    },
    notTypeKey: {
      not: { type: 'array' }
    }
  }
}

const queryStringJsonSchema = {
  name: { type: 'string' },
  excitement: { type: 'integer' }
}

const paramsJsonSchema = {
  type: 'object',
  properties: {
    par1: { type: 'string' },
    par2: { type: 'number' }
  }
}

const headersJsonSchema = {
  type: 'object',
  properties: {
    'x-foo': { type: 'string' }
  },
  required: ['x-foo']
}

const schema = {
  body: bodyJsonSchema,

  querystring: queryStringJsonSchema,

  params: paramsJsonSchema,

  headers: headersJsonSchema
}

fastify.post('/the/url', { schema }, handler)
```
*Observe que o Ajv tentará [coagir](https://github.com/epoberezkin/ajv#coercing-data-types) os valores para os tipos especificados nas palavras-chave `type` do seu esquema, para passar a validação use o tipo primitivo correto.*
<a name="shared-schema"></a>
#### Adicionando um esquema compartilhado.
Graças à API `addSchema`, você pode adicionar vários esquemas à instância Fastify e reutilizá-los em várias partes do seu aplicativo. Como sempre, essa API é encapsulada.

Há duas maneiras de reutilizar seus esquemas compartilhados:
+ **`$ref-way`**: conforme descrito em [standard](https://tools.ietf.org/html/draft-handrews-json-schema-01#section-8),
você pode consultar um esquema externo. Para usá-lo, você precisa `addSchema` com um URI absoluto válido para `$id`.
+ **`replace-way`**: este é um utilitário Fastify que permite substituir alguns campos por um esquema compartilhado.
Para usá-lo, você precisa `addSchema` com um`$id` com um fragmento de URI relativo, que é uma string simples que
aplica-se apenas a caracteres alfanuméricos `[A-Za-z0-9]`.

Aqui está uma visão geral sobre _como_ definir um `$id` e _como_ referenciá-lo:

+ `replace-way`
  + `myField: 'foobar#'` Irá procurar por um esquema compartilhado adicionado com `$id: 'foobar'`
+ `$ref-way`
  + `myField: { $ref: '#foo'}` Irá procurar por um campo com `$id: '#foo'` dentro do esquema atual
  + `myField: { $ref: '#/definitions/foo'}` Irá procurar por um campo `definitions.foo` dentro do esquema atual
  + `myField: { $ref: 'http://url.com/sh.json#'}` Irá procurar por um esquema compartilhado adicionado com `$id: 'http://url.com/sh.json'`
  + `myField: { $ref: 'http://url.com/sh.json#/definitions/foo'}` Irá procurar por um esquema compartilhado adicionado com `$id: 'http://url.com/sh.json'` e usar o campo `definitions.foo`
  + `myField: { $ref: 'http://url.com/sh.json#foo'}` Irá procurar por um esquema compartilhado adicionado com `$id: 'http://url.com/sh.json'` e vai olhar dentro para um objeto com `$id: '#foo'`


Mais exemplos:

**`$ref-way`** Exemplo de uso:

```js
fastify.addSchema({
  $id: 'http://example.com/common.json',
  type: 'object',
  properties: {
    hello: { type: 'string' }
  }
})

fastify.route({
  method: 'POST',
  url: '/',
  schema: {
    body: {
      type: 'array',
      items: { $ref: 'http://example.com/common.json#/properties/hello' }
    }
  },
  handler: () => {}
})
```

**`replace-way`** Exemplo de uso:

```js
const fastify = require('fastify')()

fastify.addSchema({
  $id: 'greetings',
  type: 'object',
  properties: {
    hello: { type: 'string' }
  }
})

fastify.route({
  method: 'POST',
  url: '/',
  schema: {
    body: 'greetings#'
  },
  handler: () => {}
})

fastify.register((instance, opts, done) => {

  /**
   * No escopo filho, pode-se usar esquemas definidos no escopo pai, como 'greetings'.
   * O escopo pai não pode usar os esquemas filhos.
   */
  instance.addSchema({
    $id: 'framework',
    type: 'object',
    properties: {
      fastest: { type: 'string' },
      hi: 'greetings#'
    }
  })

  instance.route({
    method: 'POST',
    url: '/sub',
    schema: {
      body: 'framework#'
    },
    handler: () => {}
  })

  done()
})
```

Você pode usar o esquema compartilhado em qualquer lugar, como esquema de nível superior ou aninhado dentro de outros esquemas:
```js
const fastify = require('fastify')()

fastify.addSchema({
  $id: 'greetings',
  type: 'object',
  properties: {
    hello: { type: 'string' }
  }
})

fastify.route({
  method: 'POST',
  url: '/',
  schema: {
    body: {
      type: 'object',
      properties: {
        greeting: 'greetings#',
        timestamp: { type: 'number' }
      }
    }
  },
  handler: () => {}
})
```

<a name="get-shared-schema"></a>
#### Recuperando uma cópia de esquemas compartilhados

A função `getSchemas` retorna os esquemas compartilhados disponíveis no escopo selecionado:
```js
fastify.addSchema({ $id: 'one', my: 'hello' })
fastify.get('/', (request, reply) => { reply.send(fastify.getSchemas()) })

fastify.register((instance, opts, done) => {
  instance.addSchema({ $id: 'two', my: 'ciao' })
  instance.get('/sub', (request, reply) => { reply.send(instance.getSchemas()) })

  instance.register((subinstance, opts, done) => {
    subinstance.addSchema({ $id: 'three', my: 'hola' })
    subinstance.get('/deep', (request, reply) => { reply.send(subinstance.getSchemas()) })
    done()
  })
  done()
})
```
Esse exemplo vai retornar:
| URL   | Esquemas        |
|-------|-----------------|
| /     | one             |
| /sub  | one, two        |
| /deep | one, two, three |

<a name="ajv-plugins"></a>
#### Ajv Plugins

Você pode fornecer uma lista de plugins que você deseja utilizar com Ajv:

> Referência a [`ajv options`](https://github.com/fastify/fastify/blob/master/docs/Server.md#factory-ajv) para checar o formato dos plugins.

```js
const fastify = require('fastify')({
  ajv: {
    plugins: [
      require('ajv-merge-patch')
    ]
  }
})

fastify.route({
  method: 'POST',
  url: '/',
  schema: {
    body: {
      $patch: {
        source: {
          type: 'object',
          properties: {
            q: {
              type: 'string'
            }
          }
        },
        with: [
          {
            op: 'add',
            path: '/properties/q',
            value: { type: 'number' }
          }
        ]
      }
    }
  },
  handler (req, reply) {
    reply.send({ ok: 1 })
  }
})

fastify.route({
  method: 'POST',
  url: '/',
  schema: {
    body: {
      $merge: {
        source: {
          type: 'object',
          properties: {
            q: {
              type: 'string'
            }
          }
        },
        with: {
          required: ['q']
        }
      }
    }
  },
  handler (req, reply) {
    reply.send({ ok: 1 })
  }
})
```

<a name="schema-compiler"></a>
#### Schema Compiler

O `schemaCompiler` é uma função que retorna uma função que valida o corpo, os parâmetros de URL, os cabeçalhos e a string de consulta. O `schemaCompiler` padrão retorna uma função que implementa a interface de validação [ajv](https://ajv.js.org/). O Fastify o utiliza internamente para acelerar a validação.
Fastify's [configuração base](https://github.com/epoberezkin/ajv#options-to-modify-validated-data) é:

```js
{
  removeAdditional: true, // remove propriedades adicionais
  useDefaults: true, // substitui propriedades e itens ausentes pelos valores da palavra-chave padrão correspondente
  coerceTypes: true, // alterar o tipo de dados para corresponder ao tipo de palavra-chave
  allErrors: true,   // checagem para todos os erros
  nullable: true     // suporte a palavra-chave "anulável" da especificação Open API 3.
}
```
Essa configuração base pode ser modificada fornecendo [`ajv.customOptions`] (https://github.com/fastify/fastify/blob/master/docs/Server.md#factory-ajv) à sua fábrica do Fastify.

Se você deseja alterar ou definir opções de configuração adicionais, será necessário criar sua própria instância e substituir a existente, como:
```js
const fastify = require('fastify')()
const Ajv = require('ajv')
const ajv = new Ajv({
  // o padrão do fastify (se necessário)
  removeAdditional: true,
  useDefaults: true,
  coerceTypes: true,
  allErrors: true,
  nullable: true,
  // qualquer outra opção
  // ...
})
fastify.setSchemaCompiler(function (schema) {
  return ajv.compile(schema)
})

// -------
//Você definir um schema compiler usando a propriedade setter.
fastify.schemaCompiler = function (schema) { return ajv.compile(schema) })
```
_**Nota:** Se você usar uma instância personalizada de qualquer validador (até Ajv), precisará adicionar esquemas ao validador em vez de fastify, pois o validador padrão do fastify não será mais usado, e o método `addSchema` do fastify não terá idéia de qual validador você está usando._
<a name="using-other-validation-libraries"></a>
#### Usando outra biblioteca de validação

A função `schemaCompiler` facilita a substituição do `ajv` por quase qualquer biblioteca de validação Javascript ([joi](https://github.com/hapijs/joi/), [yup](https://github.com/jquense/yup/), ...).

No entanto, para fazer com que o mecanismo de validação escolhido funcione bem com o pipeline de solicitação/resposta do Fastify, a função retornada pela função `schemaCompiler` deve retornar um objeto com:

* em caso de falha na validação: uma propriedade `error`, preenchida com uma instância de` Error` ou uma string que descreve o erro de validação
* em caso de êxito da validação: uma propriedade `value`, preenchida com o valor coagido que passou na validação

Os exemplos abaixo são, portanto, equivalentes:

```js
const joi = require('joi')

// Opções de validação para corresponder às opções da base do ajv usadas no Fastify
const joiOptions = {
  abortEarly: false, // retorna todos erros
  convert: true, // alterar o tipo de dados para corresponder ao tipo de palavra-chave
  allowUnknown : false, // remove propriedades adicionais
  noDefaults: false
}

const joiBodySchema = joi.object().keys({
  age: joi.number().integer().required(),
  sub: joi.object().keys({
    name: joi.string().required()
  }).required()
})

const joiSchemaCompiler = schema => data => {
  // A função joi `validate` retorna um objeto com uma propriedade de erro (se a validação falhar) e uma propriedade de valor (sempre presente, valor coagido se a validação for bem-sucedida)
  const { error, value } = joiSchema.validate(data, joiOptions)
  if (error) {
    return { error }
  } else {
    return { value }
  }
}

// ou mais simples...
const joiSchemaCompiler = schema => data => joiSchema.validate(data, joiOptions)

fastify.post('/the/url', {
  schema: {
    body: joiBodySchema
  },
  schemaCompiler: joiSchemaCompiler
}, handler)
```

```js
const yup = require('yup')

// Opções de validação para corresponder às opções base do ajv usadas no Fastify
const yupOptions = {
  strict: false,
  abortEarly: false, // retorna todos erros
  stripUnknown: true, // remove propriedades adicionais
  recursive: true
}

const yupBodySchema = yup.object({
  age: yup.number().integer().required(),
  sub: yup.object().shape({
    name: yup.string().required()
  }).required()
})

const yupSchemaCompiler = schema => data => {
  // com a opção strict = false, a função yup `validateSync` retorna o valor coagido se a validação foi bem-sucedida ou lança se a validação falhou
  try {
    const result = schema.validateSync(data, yupOptions)
    return { value: result }
  } catch (e) {
    return { error: e }
  }
}

fastify.post('/the/url', {
  schema: {
    body: yupBodySchema
  },
  schemaCompiler: yupSchemaCompiler
}, handler)
```
##### Mensagens de validação com outras bibliotecas de validação

As mensagens de erro de validação do Fastify estão fortemente acopladas ao mecanismo de validação padrão: os erros retornados do `ajv` acabam sendo executados através da função `schemaErrorsText`, responsável pela criação de mensagens de erro amigáveis ​​ao ser humano. No entanto, a função `schemaErrorsText` é escrita com` ajv` em mente: como resultado, você pode receber mensagens de erro estranhas ou incompletas ao usar outras bibliotecas de validação.

Para contornar esse problema, você tem 2 opções principais:

1. certifique-se de que sua função de validação (retornada pelo seu `schemaCompiler` personalizado) retorne erros exatamente na mesma estrutura e formato que o `ajv` (embora isso possa ser difícil e complicado devido a diferenças entre os mecanismos de validação)
2. ou use um `errorHandler` personalizado para interceptar e formatar seus erros de validação 'personalizados'

Para ajudá-lo a escrever um `errorHandler` personalizado, o Fastify adiciona 2 propriedades a todos os erros de validação:

* validation: o conteúdo da propriedade `error` do objeto retornado pela função de validação (retornada pelo seu `schemaCompiler` personalizado)
* validationContext: o 'contexto' (corpo, parâmetros, consulta, cabeçalhos) em que ocorreu o erro de validação

Um exemplo muito bem elaborado de um `errorHandler` personalizado que manipula erros de validação é mostrado abaixo:

```js
const errorHandler = (error, request, reply) => {

  const statusCode = error.statusCode
  let response

  const { validation, validationContext } = error

  // verifica se houve um erro de validação
  if (validation) {
    response = {
      mensagem: `Ocorreu um erro de validação ao validar o ${validationContext}...`, // validationContext será 'body', 'params', 'headers' ou 'query'
      errors: validation // este é o resultado da sua biblioteca de validação...
    }
  } else {
    response = {
      message: 'Um erro ocorreu...'
    }
  }

  // qualquer trabalho adicional aqui, por exemplo: log de erro
  // ...

  reply.status(statusCode).send(response)

}
```

<a name="schema-resolver"></a>
#### Resolvedor de esquema

O `schemaResolver` é uma função que trabalha junto com o `schemaCompiler`: você não pode usá-la
com o compilador de esquema padrão. Esse recurso é útil quando você usa esquemas complexos com a palavra-chave `$ref`
em suas rotas e um validador personalizado.

Isso é necessário porque todos os esquemas adicionados ao seu compilador personalizado são desconhecidos para o Fastify, mas
precisa resolver os caminhos `$ref`.

```js
const fastify = require('fastify')()
const Ajv = require('ajv')
const ajv = new Ajv()

ajv.addSchema({
  $id: 'urn:schema:foo',
  definitions: {
    foo: { type: 'string' }
  },
  type: 'object',
  properties: {
    foo: { $ref: '#/definitions/foo' }
  }
})
ajv.addSchema({
  $id: 'urn:schema:response',
  type: 'object',
  required: ['foo'],
  properties: {
    foo: { $ref: 'urn:schema:foo#/definitions/foo' }
  }
})
ajv.addSchema({
  $id: 'urn:schema:request',
  type: 'object',
  required: ['foo'],
  properties: {
    foo: { $ref: 'urn:schema:foo#/definitions/foo' }
  }
})

fastify.setSchemaCompiler(schema => ajv.compile(schema))
fastify.setSchemaResolver((ref) => {
  return ajv.getSchema(ref).schema
})

fastify.route({
  method: 'POST',
  url: '/',
  schema: {
    body: ajv.getSchema('urn:schema:request').schema,
    response: {
      '2xx': ajv.getSchema('urn:schema:response').schema
    }
  },
  handler (req, reply) {
    reply.send({ foo: 'bar' })
  }
})
```

<a name="serialization"></a>
### Serialização
Normalmente, você envia seus dados para os clientes via JSON, e o Fastify possui uma ferramenta poderosa para ajudá-lo, [fast-json-stringify](https://www.npmjs.com/package/fast-json-stringify) que será usado se você tiver fornecido um esquema de saída nas opções de rota. Recomendamos que você use um esquema de saída, pois aumentará sua taxa de transferência de 100 a 400%, dependendo da carga útil, e evitará a divulgação acidental de informações confidenciais.

Example:
```js
const schema = {
  response: {
    200: {
      type: 'object',
      properties: {
        value: { type: 'string' },
        otherValue: { type: 'boolean' }
      }
    }
  }
}

fastify.post('/the/url', { schema }, handler)
```
Como você pode ver, o esquema de resposta é baseado no código de status. Se você deseja usar o mesmo esquema para vários códigos de status, pode usar `'2xx'`, por exemplo:
```js
const schema = {
  response: {
    '2xx': {
      type: 'object',
      properties: {
        value: { type: 'string' },
        otherValue: { type: 'boolean' }
      }
    },
    201: {
      type: 'object',
      properties: {
        value: { type: 'string' }
      }
    }
  }
}

fastify.post('/the/url', { schema }, handler)
```
*Se você precisar de um serializador personalizado em uma parte muito específica do seu código, poderá configurá-lo com `reply.serializer (...)`.*

### Manipulação de erros
Quando a validação do esquema falha em uma solicitação, o Fastify retorna automaticamente uma resposta de status 400, incluindo o resultado do validador na carga útil. Como exemplo, se você tiver o seguinte esquema para sua rota

```js
const schema = {
  body: {
    type: 'object',
    properties: {
      name: { type: 'string' }
    },
    required: ['name']
  }
}
```

e não satisfazê-lo, a rota retornará imediatamente uma resposta com a seguinte carga útil

```js
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "body should have required property 'name'"
}
```

Se você deseja lidar com erros dentro da rota, pode especificar a opção `attachValidation` para sua rota. Se houver um erro de validação, a propriedade `validationError` da solicitação conterá o objeto `Error` com o resultado bruto de `validation`, como mostrado abaixo

```js
const fastify = Fastify()

fastify.post('/', { schema, attachValidation: true }, function (req, reply) {
  if (req.validationError) {
    // `req.validationError.validation` contém o erro bruto de validação
    reply.code(400).send(req.validationError)
  }
})
```

Você também pode usar [setErrorHandler](https://www.fastify.io/docs/latest/Server/#seterrorhandler) para definir uma resposta personalizada para erros de validação, como:

```js
fastify.setErrorHandler(function (error, request, reply) {
  if (error.validation) {
     // error.validationContext pode ser um de [body, params, querystring, headers]
     reply.status(422).send(new Error(`Erro de validação: ${error.validationContext}`))
  }
})
```
Se você deseja uma resposta de erro personalizada no esquema sem dores de cabeça e rapidamente, consulte [aqui](https://github.com/epoberezkin/ajv-errors)

### Suporte a esquema JSON e esquema compartilhado

O esquema JSON possui algum tipo de utilitário para otimizar seus esquemas que,
em conjunto com o esquema compartilhado do Fastify, permite reutilizar todos os seus esquemas facilmente.

| Caso de uso                                      | Validator | Serializer |
|--------------------------------------------------|-----------|------------|
| Esquema compartilhado                            | ✔️         | ✔️          |
| `$ref` para `$id`                                | ✔️         | ✔️          |
| `$ref` para `/definitions`                       | ✔️         | ✔️          |
| `$ref` para esquema compartilhado `$id`          | ✔️         | ✔️          |
| `$ref` para esquema compartilhado `/definitions` | ✔️         | ✔️          |

#### Exemplos

```js
// Uso do recurso Esquema Compartilhado
fastify.addSchema({
  $id: 'sharedAddress',
  type: 'object',
  properties: {
    city: { 'type': 'string' }
  }
})

const sharedSchema = {
  type: 'object',
  properties: {
    home: 'sharedAddress#',
    work: 'sharedAddress#'
  }
}
```

```js
// Uso de $ref para $id no mesmo esquema JSON
const refToId = {
  type: 'object',
  definitions: {
    foo: {
      $id: '#address',
      type: 'object',
      properties: {
        city: { 'type': 'string' }
      }
    }
  },
  properties: {
    home: { $ref: '#address' },
    work: { $ref: '#address' }
  }
}
```


```js
// Uso de $ref para /definitions no mesmo esquema JSON
const refToDefinitions = {
  type: 'object',
  definitions: {
    foo: {
      $id: '#address',
      type: 'object',
      properties: {
        city: { 'type': 'string' }
      }
    }
  },
  properties: {
    home: { $ref: '#/definitions/foo' },
    work: { $ref: '#/definitions/foo' }
  }
}
```

```js
// Uso de $ref para um esquema compartilhado $id como esquema externo
fastify.addSchema({
  $id: 'http://foo/common.json',
  type: 'object',
  definitions: {
    foo: {
      $id: '#address',
      type: 'object',
      properties: {
        city: { 'type': 'string' }
      }
    }
  }
})

const refToSharedSchemaId = {
  type: 'object',
  properties: {
    home: { $ref: 'http://foo/common.json#address' },
    work: { $ref: 'http://foo/common.json#address' }
  }
}
```


```js
// Usa $ref para um esquema compartilhado /definições como um schema externo 
fastify.addSchema({
  $id: 'http://foo/common.json',
  type: 'object',
  definitions: {
    foo: {
      type: 'object',
      properties: {
        city: { 'type': 'string' }
      }
    }
  }
})

const refToSharedSchemaDefinitions = {
  type: 'object',
  properties: {
    home: { $ref: 'http://foo/common.json#/definitions/foo' },
    work: { $ref: 'http://foo/common.json#/definitions/foo' }
  }
}
```

<a name="resources"></a>
### Recursos
- [JSON Schema](http://json-schema.org/)
- [Entendendo JSON schema](https://spacetelescope.github.io/understanding-json-schema/)
- [fast-json-stringify documentação](https://github.com/fastify/fast-json-stringify)
- [Ajv documentação](https://github.com/epoberezkin/ajv/blob/master/README.md)
- [Ajv i18n](https://github.com/epoberezkin/ajv-i18n)
- [Ajv custom errors](https://github.com/epoberezkin/ajv-errors)
- Manipulação de erro personalizada com métodos principais com dumping de arquivo de erro [exemplo](https://github.com/fastify/example/tree/master/validation-messages)
