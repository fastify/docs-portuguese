<h1 align="center">Fastify</h1>

## Benchmarking
O benchmarking é importante se você deseja medir como uma mudança pode afetar o desempenho da sua aplicação. Nós fornecemos uma maneira simples de comparar sua aplicação do ponto de vista de um usuário e colaborador. A configuração permite automatizar benchmarks em diferentes branches e em diferentes versões do Node.js.

Os módulos que usamos são:
- [Autocannon](https://github.com/mcollina/autocannon): Uma HTTP/1.1 ferramenta de benchmarking escrita em Node.js.
- [Branch-comparer](https://github.com/StarpTech/branch-comparer): Faça checkout de várias branches git, execute scripts e registre os resultados.
- [Simultaneamente] - [Concurrently](https://github.com/kimmobrunfeldt/concurrently): Rode comandos concorrentemente.
- [Npx](https://github.com/zkat/npx) NPM package runner - Estamos usando-o para executar scripts em diferentes versões do Node.js. e para executar binários locais.

## Simples

### Rode os testes na branch local
```sh
npm run benchmark
```

### Rode os testes em diferentes versões do Node.js ✨
```sh
npx -p node@6 -- npm run benchmark
```

## Avançado

### Rode os testes em branches diferentes
```sh
branchcmp --rounds 2 --script "npm run benchmark"
```

### Rode os testes em difrentes branches e versões do Node.js ✨
```sh
branchcmp --rounds 2 --script "npm run benchmark"
```

### Compare a branch atual com a main(Gitflow)
```sh
branchcmp --rounds 2 --gitflow --script "npm run benchmark"
```
or
```sh
npm run bench
```

### Execute exemplos diferentes

```sh
branchcmp --rounds 2 -s "node ./node_modules/concurrently -k -s first \"node ./examples/asyncawait.js\" \"node ./node_modules/autocannon -c 100 -d 5 -p 10 localhost:3000/\""
```
