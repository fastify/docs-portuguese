<h1 align="center">Fastify</h1>

## Recomendações

Esse documento contém um conjunto de recomendações ou melhores práticas para usar o Fastify.

* [Usando um Proxy Reverso](#reverseproxy)

## Usando um Proxy Reverso

<a id="reverseproxy"></a>

Node.js is an early adopter of frameworks shipping with an easy to use web
server within the standard library. Previously, with languages like PHP or
Python, one would need either a web server with specific support for the
language or the ability to setup some sort of [CGI gateway][cgi] that works
with the language. With Node.js, one can simply write an application that
_directly_ handles HTTP requests. As a result, the temptation is to write
applications that handle requests for multiple domains, listen on multiple
ports (i.e. HTTP _and_ HTTPS), and various other scenarios and combinations
thereof. Further, the temptation is to then expose these applications directly
to the Internet to handle requests.

O Node.js é um dos primeiros a adotar estruturas para um servidor web fácil de usar na biblioteca padrão. Anteriormente, em linguagens como PHP ou Python, seria necessário um servidor web com suporte específico para a linguagem ou a capacidade de configurar algum tipo de [gateway CGI][cgi] que funcione com a linguagem. Com o Node.js, pode-se simplesmente escrever um aplicativo que _diretamente_ lida com solicitações HTTP.

Como resultado, a tentação é escrever aplicativos que lidam com solicitações de vários domínios, ouçam em várias
portas (ou seja, HTTP _e_ HTTPS) e vários outros cenários e combinações. Além disso, a tentação é expor esses aplicativos diretamente à Internet para lidar com solicitações.

A equipe Fastify ** fortemente ** considera que isso é um antipadrão e extremamente
má prática:

1. Ele adiciona complexidade desnecessária ao aplicativo diluindo seu foco.
2. Impede [escalabilidade horizontal][scale-horiz].

Consulte [Por que devo usar um Proxy Reverso se o Node.js estiver pronto para produção?][why-use]
para uma discussão mais aprofundada sobre por que alguém deve optar por usar um proxy reverso.

Para um exemplo concreto, considere a situação onde:

1. O aplicativo precisa de várias instâncias para lidar com a carga.
1. O aplicativo precisa da terminação TLS.
1. O aplicativo precisa redirecionar solicitações HTTP para HTTPS.
1. O aplicativo precisa servir vários domínios.
1. O aplicativo precisa veicular recursos estáticos, por exemplo arquivos JPEG.

There are many reverse proxy solutions available, and your environment may
dictate the solution to use, e.g. AWS or GCP. But given the above, we could use
[HAProxy][haproxy] to solve these requirements:
Existem muitas soluções de proxy reverso disponíveis e seu ambiente pode
ditar a solução a usar, exemplo: AWS ou GCP. Mas, considerando o exposto, poderíamos usar
[HAProxy][haproxy] para resolver estes requisitos:

```conf
# A seção global define a configuração básica da instância HAProxy(engine).
global
  log /dev/log syslog
  maxconn 4096
  chroot /var/lib/haproxy
  user haproxy
  group haproxy

  # Define algumas opções de TLS base.
  tune.ssl.default-dh-param 2048
  ssl-default-bind-options no-sslv3 no-tlsv10 no-tlsv11
  ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
  ssl-default-server-options no-sslv3 no-tlsv10 no-tlsv11
  ssl-default-server-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS

# Cada seção padrão define opções que serão aplicadas a cada subseqüente
# subseção até que outra seção de padrões seja encontrada.
defaults
  log   global
  mode  http
  option        httplog
  option        dontlognull
  retries       3
  option redispatch
  maxconn       2000
  timeout connect 5000
  timeout client 50000
  timeout server 50000

  # Ative a compactação de conteúdo para tipos de conteúdo específicos.
  compression algo gzip
  compression type text/html text/plain text/css application/javascript

# Uma seção "front-end" define um ouvinte público, ou seja, um "servidor http"
# no que diz respeito aos clientes.
frontend proxy
  # O endereço IP aqui seria o endereço IP _público_ do servidor.
  # Aqui, usamos um endereço privado como exemplo.
  bind 10.0.0.10:80
  # Esta regra de redirecionamento redirecionará todo o tráfego que não é TLS
  # para o mesmo URL de solicitação de entrada na porta HTTPS.
  redirect scheme https code 308 if !{ ssl_fc }
  # Tecnicamente, essa diretiva use_backend é inútil, pois estamos simplesmente
  # redirecionando todo o tráfego deste frontend para o front end HTTPS. Isto é
  # apenas incluído aqui por uma questão de integridade.
  use_backend default-server

# Este front-end define nosso ouvinte primário, apenas TLS. É aqui onde
# definiremos os certificados TLS a serem expostos e como direcionar a entrada
# de solicitações.
frontend proxy-ssl
  # O diretório `/etc/haproxy/certs` neste exemplo contém um conjunto de
  # arquivos PEM de certificado nomeados para os domínios em que os certificados são
  # emitido. Quando o HAProxy for iniciado, ele lerá este diretório, carregará todos os
  # os certificados encontrados aqui e usar a correspondência SNI para aplicar o correto
  # certificado para a conexão.
  bind 10.0.0.10:443 ssl crt /etc/haproxy/certs

  # Aqui definimos pares de regras para manipular recursos estáticos. Qualquer solicitação recebida
  # que possui um caminho que começa com `/static`, por exemplo
  # `https://one.example.com/static/foo.jpeg`, será redirecionado para o diretório
  # servidor de recursos estáticos.
  acl is_static path -i -m beg /static
  use_backend static-backend if is_static

  # Aqui definimos pares de regras para direcionar solicitações para o Node.js apropriado
  # com base no domínio solicitado. A linha `acl` é usada para corresponder
  # o nome do host recebido e definir um booleano indicando se é uma correspondência.
  # A linha `use_backend` é usada para direcionar o tráfego se o booleano for true
  acl example1 hdr_sub(Host) one.example.com
  use_backend example1-backend if example1

  acl example2 hdr_sub(Host) two.example.com
  use_backend example2-backend if example2

  # Finalmente, temos um redirecionamento de fallback se nenhum dos hosts solicitados
  # corresponde às regras acima.
  default_backend default-server

# Um "back-end" é usado para informar ao HAProxy onde solicitar informações para a
# solicitação em proxy. Essas seções são onde definiremos onde nosso Node.js
# apps vive e quaisquer outros servidores para itens como recursos estáticos.
backend default-server
  # Neste exemplo, estamos padronizando solicitações de domínio sem correspondência para um único
  # servidor back-end em todas as solicitações. Observe que o servidor back-end não
  # precisa estar atendendo solicitações de TLS. Isso é chamado de "terminação TLS": o TLS
  # conexão é "terminada" no proxy reverso.
  # É possível também fazer proxy para servidores de back-end que estão servindo eles mesmos em
  # solicitações por TLS, mas isso está fora do escopo deste exemplo.
  server server1 10.10.10.2:80

# Esta configuração de back-end servirá solicitações para `https://one.example.com`
# por proxy de solicitações para três servidores back-end de maneira round-robin.
backend example1-backend
  server example1-1 10.10.11.2:80
  server example1-2 10.10.11.2:80
  server example2-2 10.10.11.3:80

# Este serve requisições para `https://two.example.com`
backend example2-backend
  server example2-1 10.10.12.2:80
  server example2-2 10.10.12.2:80
  server example2-3 10.10.12.3:80

# Este recebe solicitações de recursos estáticos.
backend static-backend
  server static-server1 10.10.9.2:80
```

[cgi]: https://en.wikipedia.org/wiki/Common_Gateway_Interface
[scale-horiz]: https://en.wikipedia.org/wiki/Scalability#Horizontal
[why-use]: https://web.archive.org/web/20190821102906/https://medium.com/intrinsic/why-should-i-use-a-reverse-proxy-if-node-js-is-production-ready-5a079408b2ca
[haproxy]: https://www.haproxy.org/
