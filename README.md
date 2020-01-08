# Documentação em Português (BR)
<div align="center">
<img src="https://github.com/fastify/graphics/raw/master/full-logo.png" width="650" height="auto"/>
</div>

<div align="center">

![](https://github.com/fastify/fastify/workflows/ci/badge.svg)
![](https://github.com/fastify/fastify/workflows/package-manager-ci/badge.svg)
![](https://github.com/fastify/fastify/workflows/website/badge.svg)
[![Build Status](https://dev.azure.com/fastify/fastify/_apis/build/status/fastify.fastify)](https://dev.azure.com/fastify/fastify/_build/latest?definitionId=1)
[![Known Vulnerabilities](https://snyk.io/test/github/fastify/fastify/badge.svg)](https://snyk.io/test/github/fastify/fastify)
[![Coverage Status](https://coveralls.io/repos/github/fastify/fastify/badge.svg?branch=master)](https://coveralls.io/github/fastify/fastify?branch=master)
[![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat)](http://standardjs.com/)

</div>

<div align="center">

[![NPM version](https://img.shields.io/npm/v/fastify.svg?style=flat)](https://www.npmjs.com/package/fastify)
[![NPM downloads](https://img.shields.io/npm/dm/fastify.svg?style=flat)](https://www.npmjs.com/package/fastify) [![Gitter](https://badges.gitter.im/gitterHQ/gitter.svg)](https://gitter.im/fastify)
[![Security Responsible
Disclosure](https://img.shields.io/badge/Security-Responsible%20Disclosure-yellow.svg)](https://github.com/nodejs/security-wg/blob/master/processes/responsible_disclosure_template.md)

</div>

<br />


Um servidor eficiente implica em baixo custo de infraestrutura, uma melhor capacidade de resposta sob carga e usuários felizes.
Como lidar eficientemente com os recursos do servidor, provendo o maior número de requisições possível, sem sacrificar validações de segurança e desenvolvimento prático?

Conheça o Fastify. Fastify é um framework web focado em prover a melhor experiência de desenvolvimento com a mínima "dor de cabeça" e com uma poderosa arquitetura de plugins. Inspirado por Hapi e Express, contudo, até onde sabemos, é  um dos web frameworks mais rápido do ecossistema node.

### Instalação

Instalando com npm:
```
npm i fastify --save
```
Instalando com yarn:
```
yarn add fastify
```

### Exemplo

```js
// Importando o framework e instancializando.
const fastify = require('fastify')({
  logger: true
})

// Declarando uma rota
fastify.get('/', (request, reply) => {
  reply.send({ hello: 'world' })
})

// Inicia o servidor na porta 3000!
fastify.listen(3000, (err, address) => {
  if (err) throw err
  fastify.log.info(`Servidor respondendo no endereço ${address}`)
})
```

usando async-await:

```js
const fastify = require('fastify')({
  logger: true
})

fastify.get('/', async (request, reply) => {
  reply.type('application/json').code(200)
  return { hello: 'world' }
})

fastify.listen(3000, (err, address) => {
  if (err) throw err
  fastify.log.info(`Servidor respondendo no endereço ${address}`)
})
```

Quer conhecer melhor? Acesse: <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Getting-Started.md"><code><b>Aquecendo</b></code></a>.

### Guia rápido sobre o Fastify CLI

Boas ferramentas agilizam e facilitam a manutenabilidade do desenvolvimento de APIs para não ter que fazer tudo manualmente.

O [Fastify CLI](https://github.com/fastify/fastify-cli) é uma ferramenta, via linha de comando, que ajuda na criação de novos projetos, gerenciamento de plugins, realizam uma variedade de tarefas de desenvolvimento enquanto testam e executam a aplicação.

O objetivo desse guia é construir e executar um projeto Fastify simples, usando o [Fastify CLI](https://github.com/fastify/fastify-cli), estabelecendo Boas Práticas recomendadas que beneficiam todo e qualquer projeto Fastify.

### Examplo

Abra um terminal.

Instale o [Fastify CLI](https://github.com/fastify/fastify-cli):

```
npm install fastify-cli --global
```

Crie um novo projeto padrão, executando o comando abaixo:
Generate a new project and default app by running the following command:

```
fastify generate
```

Para mais informações, veja a [documentação Fastify CLI (English)](https://github.com/fastify/fastify-cli).

### Fastify v1.x

O código-fonte do Fastify's **v1.x** está na [Branch 1.x](https://github.com/fastify/fastify/tree/1.x), assim sendo, todas as alterações relacionadas ao Fastify 1.x devem se basear na **`branch 1.x`**.

> ## Nota
> `.listen` restringe ao host local, `localhost`, interface de rede padrão (`127.0.0.1` or `::1`, a depender das configurações do sistema operacional). Se você está utilizando o Fastify em um container (Docker, [GCP](https://cloud.google.com/), etc.), você provavelmente deverá passar como parâmetro de restrição  `0.0.0.0`. Tenha cuidado ao decidir em escutar em todas as interfaces de rede; esta configuração vem com [riscos de segurança (english)](https://web.archive.org/web/20170711105010/https://snyk.io/blog/mongodb-hack-and-secure-defaults/) inerentes.
> Veja a [documentação](https://github.com/fastify/docs-portuguese/blob/master/docs/Server.md#listen) para mais informações.

### Principais recursos

- **Alto desempenho:** até onde sabemos, Fastify é um dos mais rápidos web frameworks 'na área', dependendo da complexidade do código, com o Fastify, você pode servir mais de 76 mil requisições por segundo.
- **Extensível:** Fastify é completamente extensível via hooks, plugins e decorators.
- **Baseado em Schema:** mesmo não sendo obrigatório, recomendamos o uso de [JSON Schema](http://json-schema.org/) para validar suas rotas e serializar seus retornos, internamente o Fastify compila o schema em funções de alto desempenho.
- **Logging:** logs são extremamente importantes mas são dispendiosos; escolhemos o melhor logger para quase remover esse 'custo', [Pino](https://github.com/pinojs/pino)!
- **Amigável ao Desenvolvedor:** o framework foi construido para ser significante ao mesmo tempo que realmente ajuda o desenvolvedor no uso dia após dia, sem sacrificar desempenho ou segurança.

### Benchmarks

__Hardware:__ EX41S-SSD, Intel Core i7, 4Ghz, 64GB RAM, 4C/8T, SSD.

__Metodologia:__: `autocannon -c 100 -d 40 -p 10 localhost:3000` * 2, obtendo a estimativa por segundo

| Framework          | Versão                     | Router?      | Requisições/seg |
| :----------------- | :------------------------- | :----------: | --------------: |
| hapi               | 18.1.0                     | &#10003;     | 29,998          |
| Express            | 4.16.4                     | &#10003;     | 38,510          |
| Restify            | 8.0.0                      | &#10003;     | 39,331          |
| Koa                | 2.7.0                      | &#10007;     | 50,933          |
| **Fastify**        | **2.0.0**                  | **&#10003;** | **76,835**      |
| -                  |                            |              |                 |
| `http.Server`      | 10.15.2	                  | &#10007;     | 71,768          |

Benchmarks utilizando https://github.com/fastify/benchmarks. Este é um sintético, "hello world" benchmark que visa avalia a capacidade do framework. A capacidade que cada framework tem depende na sua aplicação, você deve __sempre__ avaliar se capacidade de resposta importa para você.

## Documentação
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Getting-Started.md"><code><b>'Aquecendo'</b></code></a> &#10003;
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Server.md"><code><b>Server</b></code></a> &#10003;
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Routes.md"><code><b>Routes</b></code></a> &#10003;
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Logging.md"><code><b>Logging</b></code></a> &#10003;
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Middleware.md"><code><b>Middleware</b></code></a> &#10003;
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Hooks.md"><code><b>Hooks</b></code></a> &#10003;
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Decorators.md"><code><b>Decorators</b></code></a> &#10003;
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Validation-and-Serialization.md"><code><b>Validação e Serialização</b></code></a> &#10003;
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Fluent-Schema.md"><code><b>Fluent Schema</b></code></a> &#10003;
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Lifecycle.md"><code><b>Ciclo de vida</b></code></a> &#10003;
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Reply.md"><code><b>Resposta</b></code></a>
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Request.md"><code><b>Requisição</b></code></a>
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Errors.md"><code><b>Erros</b></code></a>
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/ContentTypeParser.md"><code><b>Conversor de Content Type</b></code></a>
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Plugins.md"><code><b>Plugins</b></code></a>
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Testing.md"><code><b>Testando</b></code></a>
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Benchmarking.md"><code><b>Benchmarking</b></code></a>
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Write-Plugin.md"><code><b>Como escrever um bom plugin</b></code></a> &#10003;
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Plugins-Guide.md"><code><b>Guia de Plugins</b></code></a>
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/HTTP2.md"><code><b>HTTP2</b></code></a>
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/LTS.md"><code><b>LTS - Suporte de longo prazo</b></code></a>
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/TypeScript.md"><code><b>TypeScript e suporte a tipagem</b></code></a>
* <a href="https://github.com/fastify/docs-portuguese/blob/master/docs/Serverless.md"><code><b>Sem servidor</b></code></a>

Documentation [English](https://github.com/fastify/fastify/blob/master/README.md)  
中文文档[地址](https://github.com/fastify/docs-chinese/blob/master/README.md)

## Ecossistema
- [Core](https://github.com/fastify/docs-portuguese/blob/master/docs/Ecosystem.md#core) - Plugins mantidos pela [equipe](#equipe) _Fastify_ [team](#team).
- [Comunidade](https://github.com/fastify/docs-portuguese/blob/master/docs/Ecosystem.md#comunidade) - Plugins mantidos pela Comunidade.
- [Exemplos (English)](https://github.com/fastify/example) - Multirepo com um conjunto de exemplos funcionais reais.

## Suporte
- [Ajuda Fastify (English)](https://github.com/fastify/help)
- [Gitter Chat](https://gitter.im/fastify)

## Equipe

_Fastify_ é o resultado do trabalho de uma comunidade maravilhosa.
Os membros da equipe é listado em ordem alfabética.

**Líderes mantenedores:**
* [__Matteo Collina__](https://github.com/mcollina), <https://twitter.com/matteocollina>, <https://www.npmjs.com/~matteo.collina>
* [__Tomas Della Vedova__](https://github.com/delvedor), <https://twitter.com/delvedor>, <https://www.npmjs.com/~delvedor>

### Equipe Fastify Core
* [__Tommaso Allevi__](https://github.com/allevo), <https://twitter.com/allevitommaso>, <https://www.npmjs.com/~allevo>
* [__Ethan Arrowood__](https://github.com/Ethan-Arrowood/), <https://twitter.com/arrowoodtech>, <https://www.npmjs.com/~ethan_arrowood>
* [__Matteo Collina__](https://github.com/mcollina), <https://twitter.com/matteocollina>, <https://www.npmjs.com/~matteo.collina>
* [__Tomas Della Vedova__](https://github.com/delvedor), <https://twitter.com/delvedor>, <https://www.npmjs.com/~delvedor>
* [__Dustin Deus__](https://github.com/StarpTech), <https://twitter.com/dustindeus>, <https://www.npmjs.com/~starptech>
* [__Denis Fäcke__](https://github.com/SerayaEryn), <https://twitter.com/serayaeryn>, <https://www.npmjs.com/~serayaeryn>
* [__Luciano Mammino__](https://github.com/lmammino), <https://twitter.com/loige>, <https://www.npmjs.com/~lmammino>
* [__Cemre Mengu__](https://github.com/cemremengu), <https://twitter.com/cemremengu>, <https://www.npmjs.com/~cemremengu>
* [__Manuel Spigolon__](https://github.com/eomm), <https://twitter.com/manueomm>, <https://www.npmjs.com/~eomm>
* [__James Sumners__](https://github.com/jsumners), <https://twitter.com/jsumners79>, <https://www.npmjs.com/~jsumners>

### Equipe Fastify Plugins
* [__Matteo Collina__](https://github.com/mcollina), <https://twitter.com/matteocollina>, <https://www.npmjs.com/~matteo.collina>
* [__Tomas Della Vedova__](https://github.com/delvedor), <https://twitter.com/delvedor>, <https://www.npmjs.com/~delvedor>
* [__Manuel Spigolon__](https://github.com/eomm), <https://twitter.com/manueomm>, <https://www.npmjs.com/~eomm>

### Colaboradores
Grandes colaboradores em áreas específicas no ecossistema Fastify seráo convidados a se juntar a este grupo pelos Líderes Mantenedores.

* [__Luciano Mammino__](https://github.com/lmammino), <https://twitter.com/loige>, <https://www.npmjs.com/~lmammino>
* [__Evan Shortiss__](https://github.com/evanshortiss), <https://twitter.com/evanshortiss>, <https://www.npmjs.com/~evanshortiss>

**Já contribuiram**
* [__Çağatay Çalı__](https://github.com/cagataycali), <https://twitter.com/cagataycali>, <https://www.npmjs.com/~cagataycali>
* [__Trivikram Kamat__](https://github.com/trivikr), <https://twitter.com/trivikram>, <https://www.npmjs.com/~trivikr>
* [__Nathan Woltman__](https://github.com/nwoltman), <https://twitter.com/NathanWoltman>, <https://www.npmjs.com/~nwoltman>

## Hospedado por

[<img src="https://github.com/openjs-foundation/cross-project-council/blob/master/logos/openjsf-color.png?raw=true" width="250px;"/>](https://openjsf.org/projects/#incubating)

Atualmente somos um [projeto incubado (English)](https://openjsf.org/blog/2019/11/20/web-framework-fastify-joins-openjs-foundation-as-an-incubating-project/) na OpenJS Foundation.

## Acknowledgements

Este projeto é bondosamente patrocinado por:
- [nearForm](http://nearform.com)

Já patrocinaram:
- [LetzDoIt](http://www.letzdoitapp.com/)

## Licença

Licenciado sob a licença [MIT (English)](./LICENSE).

Para sua comodidade, aqui está a lista de todas as licenças das bibliotecas de produção utilizadas:
- MIT
- ISC
- BSD-3-Clause
- BSD-2-Clause
