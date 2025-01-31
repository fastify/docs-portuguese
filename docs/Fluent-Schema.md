<h1 align="center">Fastify</h1>

## Fluent Schema

A documentação [Validação e Serialização](https://github.com/fastify/fastify/blob/main/docs/Validation-and-Serialization.md) descreve todos os parâmetros aceitos pelo Fastify para configurar Validações de shemas de JSON no intuito de validar o recebimento de dados e Serialização de schemas de JSON no intuto de otimizar o envido de dados.

[`fluent-schema`](https://github.com/fastify/fluent-schema) pode ser utilizado para simplificar essa tarefa enquanto possibilita o reuso de constantes.

### Configurações básicas

```js
const S = require('fluent-schema')

// Você pode ter um objeto como este, ou uma query a um DB para receber os valores
const MY_KEY = {
  KEY1: 'ONE',
  KEY2: 'TWO'
}

const bodyJsonSchema = S.object()
  .prop('someKey', S.string())
  .prop('someOtherKey', S.number())
  .prop('requiredKey', S.array().maxItems(3).items(S.integer()).required())
  .prop('nullableKey', S.mixed([S.TYPES.NUMBER, S.TYPES.NULL]))
  .prop('multipleTypesKey', S.mixed([S.TYPES.BOOLEAN, S.TYPES.NUMBER]))
  .prop('multipleRestrictedTypesKey', S.oneOf([S.string().maxLength(5), S.number().minimum(10)]))
  .prop('enumKey', S.enum(Object.values(MY_KEYS)))
  .prop('notTypeKey', S.not(S.array()))

const queryStringJsonSchema = S.object()
  .prop('name', S.string())
  .prop('excitement', S.integer())

const paramsJsonSchema = S.object()
  .prop('par1', S.string())
  .prop('par2', S.integer())

const headersJsonSchema = S.object()
  .prop('x-foo', S.string().required())

// Note que não há necessidade de chamar `.valueOf()`!
const schema = {
  body: bodyJsonSchema,
  querystring: queryStringJsonSchema, // (ou) query: queryStringJsonSchema
  params: paramsJsonSchema,
  headers: headersJsonSchema
}

fastify.post('/the/url', { schema }, handler)
```

### Reutilização

Com `fluent-schema` você pode manipular seus schemas de forma fácil e programática e então reutilizá-las graças ao método `addSchema()`. Você pode fazer referência ao schema de duas maneiras diferentes, que são detalhadas na documentção [Validação e Serialização.md](./Validation-and-Serialization.md#adding-a-shared-schema).

Eis alguns exemplos de utilização:

**`$ref-way`**: utilizando um schema externo

```js
const addressSchema = S.object()
  .id('#address')
  .prop('line1').required()
  .prop('line2')
  .prop('country').required()
  .prop('city').required()
  .prop('zipcode').required()

const commonSchemas = S.object()
  .id('https://fastify/demo')
  .definition('addressSchema', addressSchema)
  .definition('otherSchema', otherSchema) // Você pode adicionar quantos schemas que precisar

fastify.addSchema(commonSchemas)

const bodyJsonSchema = S.object()
  .prop('residence', S.ref('https://fastify/demo#address')).required()
  .prop('office', S.ref('https://fastify/demo#/definitions/addressSchema')).required()

const schema = { body: bodyJsonSchema }

fastify.post('/the/url', { schema }, handler)
```


**`replace-way`**: utilizando um schema compartilhado para substituição antes do processo de validação.

```js
const sharedAddressSchema = {
  $id: 'sharedAddress',
  type: 'object',
  required: ['line1', 'country', 'city', 'zipcode'],
  properties: {
    line1: { type: 'string' },
    line2: { type: 'string' },
    country: { type: 'string' },
    city: { type: 'string' },
    zipcode: { type: 'string' }
  }
}
fastify.addSchema(sharedAddressSchema)

const bodyJsonSchema = {
  type: 'object',
  properties: {
    vacation: 'sharedAddress#'
  }
}

const schema = { body: bodyJsonSchema }

fastify.post('/the/url', { schema }, handler)
```

**Note que**: você pode misturar ambos `$ref-way` e `replace-way` quanto estiver utilizando `fastify.addSchema`.
