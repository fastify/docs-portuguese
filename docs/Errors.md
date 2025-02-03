<h1 align="center">Fastify</h1>

<a id="errors"></a>
## Errors

<a name="error-handling"></a>
### Manipulalação de erros

É provável que erros não capturados causem vazamentos de memória, vazamentos no descritor de arquivos e outros problemas importantes em produção. [Domínios](https://nodejs.org/en/docs/guides/domain-postmortem/) foram introduzidos para tentar corrigir esse problema, mas não o fizeram. Dado que não é possível processar todos os erros não detectados de maneira sensata, a melhor maneira de lidar com eles no momento é [travar](https://nodejs.org/api/process.html#process_warning_using_uncaughtexception_correctly). Em caso de promessas, certifique-se de [manipular](https://nodejs.org/dist/latest-v8.x/docs/api/deprecations.html#deprecations_dep0018_unhandled_promise_rejections) erros [corretamente] (https://github.com/mcollina/make-promises-safe).

O Fastify segue uma abordagem do tipo tudo ou nada e visa ser o mais enxuto e otimizado possível. Assim, o desenvolvedor é responsável por garantir que os erros sejam tratados corretamente. Como a maioria dos erros geralmente resulta de dados de entrada inesperados, recomendamos especificar uma [validação JSON.schema](https://github.com/fastify/fastify/blob/main/docs/Validation-and-Serialization.md) para seus dados de entrada.

Observe que o Fastify não captura erros não capturados em rotas baseadas em retorno de chamada para você, portanto, qualquer erro não capturado resultará em uma falha.
Se as rotas forem declaradas como `async` - o erro será capturado com segurança pela promessa e roteado para o manipulador de erros padrão do Fastify para uma resposta genérica do `Internal Server Error`. Para personalizar esse comportamento, você deve usar [setErrorHandler](https://github.com/fastify/fastify/blob/main/docs/Server.md#seterrorhandler).

<a name="fastify-error-codes"></a>
### Fastify códigos de erro

<a name="FST_ERR_BAD_URL"></a>
#### FST_ERR_BAD_URL

O rota recebeu um URL inválido.

<a name="FST_ERR_CTP_ALREADY_PRESENT"></a>
#### FST_ERR_CTP_ALREADY_PRESENT

O analisador para este tipo de conteúdo já estava registrado.

<a name="FST_ERR_CTP_INVALID_TYPE"></a>
#### FST_ERR_CTP_INVALID_TYPE

O `Content-Type` deve ser uma string.

<a name="FST_ERR_CTP_EMPTY_TYPE"></a>
#### FST_ERR_CTP_EMPTY_TYPE

O tipo de conteúdo não pode ser uma sequência vazia.

<a name="FST_ERR_CTP_INVALID_HANDLER"></a>
#### FST_ERR_CTP_INVALID_HANDLER

Um manipulador inválido foi passado para o tipo de conteúdo.

<a name="FST_ERR_CTP_INVALID_PARSE_TYPE"></a>
#### FST_ERR_CTP_INVALID_PARSE_TYPE

O tipo de análise fornecido não é suportado. Os valores aceitos são `string` ou `buffer`.

<a name="FST_ERR_CTP_BODY_TOO_LARGE"></a>
#### FST_ERR_CTP_BODY_TOO_LARGE

O corpo da solicitação é maior que o limite fornecido.

<a name="FST_ERR_CTP_INVALID_MEDIA_TYPE"></a>
#### FST_ERR_CTP_INVALID_MEDIA_TYPE

O tipo de mídia recebido não é suportado (ou seja, não há um analisador `Content-Type` adequado para ele).

<a name="FST_ERR_CTP_INVALID_CONTENT_LENGTH"></a>
#### FST_ERR_CTP_INVALID_CONTENT_LENGTH

O tamanho do corpo da solicitação não corresponde ao comprimento do conteúdo.

<a name="FST_ERR_DEC_ALREADY_PRESENT"></a>
#### FST_ERR_DEC_ALREADY_PRESENT

Um decorador com o mesmo nome já está registrado.

<a name="FST_ERR_DEC_MISSING_DEPENDENCY"></a>
#### FST_ERR_DEC_MISSING_DEPENDENCY

O decorador não pode ser registrado devido a uma dependência ausente.

<a name="FST_ERR_HOOK_INVALID_TYPE"></a>
#### FST_ERR_HOOK_INVALID_TYPE

O nome do gancho deve ser uma sequência.

<a name="FST_ERR_HOOK_INVALID_HANDLER"></a>
#### FST_ERR_HOOK_INVALID_HANDLER

O retorno de chamada do gancho deve ser uma função.

<a name="FST_ERR_LOG_INVALID_DESTINATION"></a>
#### FST_ERR_LOG_INVALID_DESTINATION

O log aceita um `'stream'` ou um `'arquivo'` como destino.

<a id="FST_ERR_REP_ALREADY_SENT"></a>
### FST_ERR_REP_ALREADY_SENT

Uma resposta já foi enviada.

<a id="FST_ERR_SEND_INSIDE_ONERR"></a>
#### FST_ERR_SEND_INSIDE_ONERR

Você não pode usar o `send` dentro do gancho `onError`.

<a name="FST_ERR_REP_INVALID_PAYLOAD_TYPE"></a>
#### FST_ERR_REP_INVALID_PAYLOAD_TYPE

A carga útil da resposta pode ser uma `string` ou um `Buffer`.

<a name="FST_ERR_SCH_MISSING_ID"></a>
#### FST_ERR_SCH_MISSING_ID

O esquema fornecido não possui a propriedade `$id`.

<a name="FST_ERR_SCH_ALREADY_PRESENT"></a>
#### FST_ERR_SCH_ALREADY_PRESENT

Um esquema com o mesmo `$id` já existe.

<a name="FST_ERR_SCH_NOT_PRESENT"></a>
#### FST_ERR_SCH_NOT_PRESENT

Não existe nenhum esquema com o `$id` fornecido.

<a name="FST_ERR_SCH_BUILD"></a>
#### FST_ERR_SCH_BUILD

O esquema JSON fornecido para uma rota não é válido.

<a name="FST_ERR_PROMISE_NOT_FULLFILLED"></a>
#### FST_ERR_PROMISE_NOT_FULLFILLED

Uma promessa pode não ser cumprida com 'undefined' quando statusCode não for 204.

<a name="FST_ERR_SEND_UNDEFINED_ERR"></a>
#### FST_ERR_SEND_UNDEFINED_ERR

Ocorreu um erro indefinido.
