# You Don't Know JS: Async & Performance
# Capítulo 4: Geradores

No capítulo 2, nós identificamos duas desvantagens importantes ao expressar controle de fluxo assíncrono com callbacks(retornos):

* Fluxo assíncrono baseado em callback(retorno) não fit em como o nosso cérebro planeja os passos de uma tarefa.
* Callbacks(retornos) não são confiávwis ou composable por causa da *nversão de controle*.

No capítulo 3, nós detalhamos como Promises uninvert a *inversão de controle* dos callbacks(retornos), restaurando a confiabilidade/composibilidade.

Agora nós mudamos o nosso foco para expressar controle de fluxo assíncrono em uma sequential, de um jeito parecido com síncrono. A "mágica" que faz isto possível são **geradores** ES6.

## Quebrando o Rodar-até-acabar

No capítulo 1, nós explicamos uma expectativa que quase todas as pessoas desenvolvedoras JS têm com seu código: quando uma função começa a executar, ela roda até acabar, e nenhum outro código pode interromper este processo e rodar no meio.

Por mais bizarro que isso pareça, ES6 introduziu um novo tipo de função que não se comporta com o comportamente de rodar-até-acabar. Este novo tipo de função se chama "gerador".

Para entender as implicações disso, vamos considerar o seguinte exemplo:
```js
var x = 1;

function foo() {
	x++;
	bar();				// <-- e essa linha?
	console.log( "x:", x );
}

function bar() {
	x++;
}

foo();					// x: 3
```
