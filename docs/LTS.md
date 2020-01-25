<h1 align="center">Fastify</h1>

<a name="lts"></a>

## Long Term Support

O Suporte a Longo Prazo da Fastify (LTS) é fornecido de acordo com o cronograma estabelecido
neste documento:

1. Principais lançamentos, versão "X" de [versioning semântica] [semver] versão X.Y.Z
   versões, são suportadas por um período mínimo de seis meses após o lançamento.
   A data de lançamento de qualquer versão específica pode ser encontrada em
   [https://github.com/fastify/fastify/releasesModa(https://github.com/fastify/fastify/releases).

1. Os principais lançamentos receberão atualizações de segurança por mais seis meses
   a partir do lançamento do próximo grande lançamento. Após esse período expirar,
   ainda revisaremos e liberaremos as correções de segurança, desde que sejam
   fornecidos pela comunidade e eles não violam outras restrições,
   por exemplo. versão mínima suportada do Node.js.

1. Os principais lançamentos serão testados e verificados em todos os Node.js
   releases que são suportados pela
   [Política do Node.js LTS] (https://github.com/nodejs/Release) dentro do
   Período LTS daquela linha de liberação Fastify.

Um "mês" deve ser um período de 30 dias consecutivos.

[semver]: https://semver.org/

<a name="lts-schedule"></a>

### Schedule

| Versão  | Data de Lançamento | Final do suporte LTS | Node.js         |
| :------ | :-----------       | :--------------      | :-------------- |
| 1.0.0   | 2018-03-06         | 2019-09-01           | 6, 8, 9, 10, 11 |
| 2.0.0   | 2019-02-25         | TBD                  | 6, 8, 10, 11    |

<a name="supported-os"></a>

### Sistemas operacionais testador por CI

| CI             | OS      | Versão                 | Gerenciador de Pacotes | Node.js   |
|----------------|---------|------------------------|------------------------|-----------|
| Github Actions | Linux   | Ubuntu 16.04           | npm                    | 6,8,10,12 |
| Github Actions | Linux   | Ubuntu 16.04           | yarn,pnpm              | 8,10      |
| Github Actions | Windows | Windows Server 2016 R2 | npm                    | 6,8,10,12 |
| Github Actions | MacOS   | macOS X Mojave 10.14   | npm                    | 6,8,10,12 |
