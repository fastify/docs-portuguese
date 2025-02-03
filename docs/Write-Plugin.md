
<h1 align="center">Fastify</h1>

# Como escrever um bom plugin

Primeiramente, obrigado por decidir escrever um plugin para Fastify. Fastify é um micro framework a seus plugins são peças fundamentais, portanto obrigado! <br>
Os principais princípios Fastify, é seu foco em performance, baixa sobrecarga provendo uma boa experiência para seus usuários. Quando escrevemos um plugin é imprescindível ter esses princípios em mente. Portanto, neste documento vamos analizar oque caracteriza um bom plugin.

*Precisa de inspiração? Você pode usar a label ["plugin suggestion"](https://github.com/fastify/fastify/issues?q=is%3Aissue+is%3Aopen+label%3A%22plugin+suggestion%22) em nosso rastreador de issues!*

## Code

Fastify utiliza diferentes técnicas para otimzar seu código, grande parte delas estão documentadas em nossos Guias. Nós recomendamos fortemente que você leia [o guia do plugin](https://github.com/fastify/docs-portguese/blob/main/docs/Plugins-Guide.md) para conhecer toda API que você pode usar para construir seu plugin e aprender como usar-las.
Tem alguma dúvida ou sugestão? Ficaremos felizes em ajudar! Basta abrir uma issue em nosso [repositório de ajuda](https://github.com/fastify/help).

Você pode submeter um plugin em nosssa [lista de plugins](https://github.com/fastify/docs-portuguese/blob/main/docs/Ecosystem.md) que iremos revisar seu código e lhe ajudar a melhora-lo se necessário.

## Documentation
Documentação é extremamente importante. Se seu plugin não for bem documentado, nós não iremos aceita-lo em nossa lista de plugins. Falta de documentação de qualidade dificulta o uso do plugin por outras pessoas.
Caso queira ver alguns bons exemplos de como documentar um plugin, dê uma olhada nesses plugins:

- [`fastify-caching`](https://github.com/fastify/fastify-caching)
- [`fastify-compress`](https://github.com/fastify/fastify-compress)
- [`fastify-cookie`](https://github.com/fastify/fastify-cookie)
- [`point-of-view`](https://github.com/fastify/point-of-view)
- [`under-pressure`](https://github.com/fastify/under-pressure)

## License
Você pode licenciar seu plugin como preferir, nós não temos uma licença obrigatória para os plugins.
Contudo, nós preferimos a [Licença MIT](https://choosealicense.com/licenses/mit/) porque achamos que permite que mais pessoas usem o código livremente. Para uma lista de licenças alternativas verifique a lista [OSI](https://opensource.org/licenses) ou a do GitHub [choosealicense.com](https://choosealicense.com/).

## Exemplos
Recomendamos que sempre coloque um arquivo de exemplo em seu repositório. Exemplos são muito úteis para os usuários e oferecem uma maneira muito rápida de testar seu plugin. Seus usuários serão gratos por isso.

## Testes
É de extrema importância que um plugin seja completamente testado para verificar se está tudo funcionando corretamente.<br>
Um plugin sem testes não será aceitado em nossa lista de plugins. A falta de testes não inspira confiança nem garante que o código continuará funcionando entre diferentes versões de suas dependências.

Nós não temos uma biblioteca de testes obrigatória. Utilizamos [`tap`](http://www.node-tap.org/) que oferece testes paralelos e cobertura de código prontos para uso, mas cabe a você escolher sua biblioteca de preferências.

## Linter de código
Não é obrigatório, mas é altamente recomendável que você use um linter de código em seu plugin. Isso garantirá um estilo de código consistente e ajudará a evitar muitos erros.

Nós usamos [`standard`](https://standardjs.com/) pois funciona sem a necessidade de configurá-lo e é muito fácil integrar em um conjunto de testes.

## Integração Contínua (CI)
Não é obrigatório, mas se você liberar seu código open source, Integração Contínua ajudará a garantir que as contribuições não quebrem seu plugin e para mostrar que o plugin funciona conforme o esperado. [Travis](https://travis-ci.org/) é gratuito para projetos de código aberto e fácil de configurar. <br>
Além disso, você pode ativar serviços como o [Greenkeeper](https://greenkeeper.io/), que o ajudará a manter suas dependências atualizadas e descobrir se uma nova versão do Fastify tem alguns problemas com seu plugin.

## Vamos começar!
Ótimo, agora você sabe tudo o que precisa saber sobre como escrever um bom plugin para o Fastify!
Depois de criar um (ou mais!), Avise-nos! Nós o adicionaremos à nossa [lista de plugins](https://github.com/fastify/fastify#ecosystem) em nossa documentação!

Se você quiser ver alguns exemplos reais, verfique:
- [`point-of-view`](https://github.com/fastify/point-of-view)
Plugin para renderização de template (*ejs, pug, handlebars, marko*) para Fastify.
- [`fastify-mongodb`](https://github.com/fastify/fastify-mongodb)
Plugin para conexão com MongoDB, com isso você poderá compartilhar o mesmo pool de conexões em qualquer parte da sua aplicação.
- [`fastify-multipart`](https://github.com/fastify/fastify-multipart)
Plugin para suporte de multipart em sua aplicação.
- [`fastify-helmet`](https://github.com/fastify/fastify-helmet) Plugin que adiciona imporantes headers de segurança na sua aplicação.
