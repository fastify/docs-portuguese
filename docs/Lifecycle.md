<h1 align="center">Fastify</h1>

## Ciclo de vida 
O esquema a seguir apresenta o ciclo de vida interno do Fastify.<br>
A direita de cada seção é apresentado o nome da fase do ciclo de vida, a esquerda (quando houver) é apresentado o código de erro correspondente que será gerado se ocorrer algum erro na fase 'pai' *(note que todos os erros são tratados tratados automaticamente pelo Fastify)*.
```
Incoming Request
  │
  └─▶ Routing
        │
        └─▶ Instance Logger
             │
       404 ◀─┴─▶ onRequest Hook
                  │
        4**/5** ◀─┴─▶ run Middlewares
                        │
              4**/5** ◀─┴─▶ preParsing Hook
                              │
                    4**/5** ◀─┴─▶ Parsing
                                   │
                         4**/5** ◀─┴─▶ preValidation Hook
                                        │
                                  415 ◀─┴─▶ Validation
                                              │
                                        400 ◀─┴─▶ preHandler Hook
                                                    │
                                          4**/5** ◀─┴─▶ User Handler
                                                          │
                                                          └─▶ Reply
                                                                │
                                                      4**/5** ◀─┴─▶ preSerialization Hook
                                                                      │
                                                                      └─▶ onSend Hook
                                                                            │
                                                                  4**/5** ◀─┴─▶ Outgoing Response
                                                                                  │
                                                                                  └─▶ onResponse Hook
```
