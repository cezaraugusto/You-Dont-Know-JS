# You Don't Know JS: Async & Performance
# Capítulo 4: Geradores

No capítulo 2, nós identificamos duas desvantagens importantes ao expressar controle de fluxo assíncrono com callbacks(retornos):

* Fluxo assíncrono baseado em callback(retorno) não se alinha com o jeito de como o nosso cérebro planeja os passos de uma tarefa.
* Callbacks(retornos) não são confiáveis ou combináveis por causa da *inversão de controle*.

No capítulo 3, nós detalhamos como Promises desinvertem a *inversão de controle* dos callbacks(retornos), restaurando a confiabilidade/composibilidade.

Agora nós mudamos o nosso foco para expressar controle de fluxo assíncrono em uma sequência, de um jeito parecido com síncrono. A "mágica" que faz isto possível são **geradores** (generators) do ES6.

## Desmembrando o Rodar-até-acabar

No capítulo 1, nós explicamos uma expectativa que quase todas as pessoas desenvolvedoras JS têm com seu código: quando uma função começa a executar, ela roda até acabar, e nenhum outro código pode interromper este processo e rodar entre ela.

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

Nesse exemplo, nós temos certeza de que `bar()` roda entre `x++` e `console.log(x)`. Mas e se `bar()` não estivesse ali? É óbvio que o resultado seria `2` e não `3`.

Agora vamos brincar com o nosso cérebro. E se `bar()` não estivesse ali, mas ainda pudesse rodar entre `x++` e `console.log(x)`? Como isso seria possível?

Em linguagens de programação **preemptivas** multi-tarefas, seria essencialmente possível que `bar()` "interrompesse" e rodasse exatamente entre esses dois pedaços de código. Mas JS não é preemptivo, nem é (atualmente) multithread. E, ainda assim, uma forma **cooperativa** dessa "interrupção" (concorrência) é possível, se `foo()` em si pudesse de alguma forma indicar uma "pausa" naquela parte do código.


**Nota:** Eu uso a palavra "cooperativa" não apenas por causa da conexão com a terminologia clássica de concorrência (veja o capítulo 1), mas porque, como você verá no próximo trecho, a sintaxe do ES6 para indicar um ponto de pausa no código é `yield` -- sugerindo uma cessão (yielding) de controle de forma educadamente *cooperativa*.

Aqui está o código ES6 para realizar essa concorrência cooperativa:

```js
var x = 1;

function *foo() {
	x++;
	yield; // pausa!
	console.log( "x:", x );
}

function bar() {
	x++;
}
```

**Nota:** Você provavelmente verá a maioria das outras documentações/códigos JS formatarem uma declaração de gerador como `function* foo() { .. }` em vez de como eu fiz aqui com `function *foo() { .. }` -- a única diferença sendo o posicionamento estilístico do `*`. As duas formas são funcional/sintaticamente idênticas, assim como uma terceira forma `function*foo() { .. }` (sem espaço). Há argumentos para os dois estilos, mas eu basicamente prefiro `function *foo..` porque assim ele combina quando eu referencio um gerador na escrita com `*foo()`. Se eu dissesse apenas `foo()`, você não saberia tão claramente se eu estava falando de um gerador ou de uma função comum. É puramente uma preferência estilística.

Agora, como podemos rodar o código no trecho anterior de forma que `bar()` execute no ponto do `yield` dentro de `*foo()`?

```js
// constrói um iterator `it` para controlar o gerador
var it = foo();

// inicie `foo()` aqui!
it.next();
x;						// 2
bar();
x;						// 3
it.next();				// x: 3
```

OK, há bastante coisa nova e potencialmente confusa nesses dois trechos de código, então temos muito o que examinar. Mas antes de explicarmos a mecânica/sintaxe diferente com geradores do ES6, vamos percorrer o fluxo de comportamento:

1. A operação `it = foo()` *não* executa o gerador `*foo()` ainda, ela apenas constrói um *iterator* que controlará sua execução. Mais sobre *iterators* daqui a pouco.
2. O primeiro `it.next()` inicia o gerador `*foo()`, e roda o `x++` na primeira linha de `*foo()`.
3. `*foo()` pausa na instrução `yield`, ponto no qual a primeira chamada `it.next()` termina. Neste momento, `*foo()` ainda está rodando e ativo, mas está em um estado pausado.
4. Nós inspecionamos o valor de `x`, e agora ele é `2`.
5. Nós chamamos `bar()`, que incrementa `x` novamente com `x++`.
6. Nós inspecionamos o valor de `x` novamente, e agora ele é `3`.
7. A chamada final `it.next()` retoma o gerador `*foo()` de onde ele havia pausado, e roda a instrução `console.log(..)`, que usa o valor atual de `x` que é `3`.

Claramente, `*foo()` iniciou, mas *não* rodou até acabar -- ele pausou no `yield`. Nós retomamos `*foo()` depois, e o deixamos terminar, mas isso nem sequer era necessário.

Então, um gerador é um tipo especial de função que pode iniciar e parar uma ou mais vezes, e não necessariamente precisa terminar algum dia. Embora ainda não vá ficar terrivelmente óbvio por que isso é tão poderoso, à medida que avançarmos pelo restante deste capítulo, esse será um dos blocos fundamentais que usamos para construir geradores-como-controle-de-fluxo-assíncrono como um padrão para o nosso código.

### Entrada e Saída

Uma função geradora é uma função especial com o novo modelo de processamento que acabamos de mencionar. Mas ela ainda é uma função, o que significa que ela ainda tem alguns princípios básicos que não mudaram -- a saber, que ela ainda aceita argumentos (também conhecidos como "entrada"), e que ela ainda pode retornar um valor (também conhecido como "saída"):

```js
function *foo(x,y) {
	return x * y;
}

var it = foo( 6, 7 );

var res = it.next();

res.value;		// 42
```

Nós passamos os argumentos `6` e `7` para `*foo(..)` como os parâmetros `x` e `y`, respectivamente. E `*foo(..)` retorna o valor `42` de volta para o código chamador.

Agora vemos uma diferença em como o gerador é invocado comparado a uma função normal. `foo(6,7)` obviamente parece familiar. Mas, sutilmente, o gerador `*foo(..)` na verdade ainda não rodou como teria acontecido com uma função.

Em vez disso, estamos apenas criando um objeto *iterator*, que atribuímos à variável `it`, para controlar o gerador `*foo(..)`. Então chamamos `it.next()`, que instrui o gerador `*foo(..)` a avançar de sua localização atual, parando ou no próximo `yield` ou no fim do gerador.

O resultado dessa chamada `next(..)` é um objeto com uma propriedade `value` que contém qualquer que seja o valor (se houver algum) que foi retornado de `*foo(..)`. Em outras palavras, `yield` fez com que um valor fosse enviado para fora do gerador no meio de sua execução, mais ou menos como um `return` intermediário.

Novamente, ainda não vai ficar óbvio por que precisamos de todo esse objeto *iterator* indireto para controlar o gerador. Nós chegaremos lá, eu *prometo*.

#### Mensageria de Iteração

Além de geradores aceitarem argumentos e terem valores de retorno, há uma capacidade de mensageria de entrada/saída ainda mais poderosa e atraente embutida neles, via `yield` e `next(..)`.

Observe:

```js
function *foo(x) {
	var y = x * (yield);
	return y;
}

var it = foo( 6 );

// inicia `foo(..)`
it.next();

var res = it.next( 7 );

res.value;		// 42
```

Primeiro, nós passamos `6` como o parâmetro `x`. Então chamamos `it.next()`, e ele inicia `*foo(..)`.

Dentro de `*foo(..)`, a instrução `var y = x ..` começa a ser processada, mas então ela esbarra em uma expressão `yield`. Nesse ponto, ela pausa `*foo(..)` (no meio da instrução de atribuição!), e essencialmente requisita ao código chamador que forneça um valor de resultado para a expressão `yield`. Em seguida, chamamos `it.next( 7 )`, que está passando o valor `7` de volta para *ser* esse resultado da expressão `yield` pausada.

Então, neste ponto, a instrução de atribuição é essencialmente `var y = 6 * 7`. Agora, `return y` retorna esse valor `42` de volta como o resultado da chamada `it.next( 7 )`.

Note algo muito importante, mas também facilmente confuso, mesmo para desenvolvedores JS experientes: dependendo da sua perspectiva, há um descompasso entre o `yield` e a chamada `next(..)`. Em geral, você vai ter uma chamada `next(..)` a mais do que instruções `yield` -- o trecho anterior tem um `yield` e duas chamadas `next(..)`.

Por que o descompasso?

Porque o primeiro `next(..)` sempre inicia um gerador, e roda até o primeiro `yield`. Mas é a segunda chamada `next(..)` que satisfaz a primeira expressão `yield` pausada, e o terceiro `next(..)` satisfaria o segundo `yield`, e assim por diante.

##### A História de Duas Perguntas

Na verdade, em qual código você está pensando primariamente vai afetar se há ou não um descompasso percebido.

Considere apenas o código do gerador:

```js
var y = x * (yield);
return y;
```

Esse **primeiro** `yield` está basicamente *fazendo uma pergunta*: "Qual valor devo inserir aqui?"

Quem vai responder a essa pergunta? Bem, o **primeiro** `next()` já rodou para levar o gerador até esse ponto, então obviamente *ele* não pode responder a pergunta. Portanto, a **segunda** chamada `next(..)` deve responder a pergunta *colocada* pelo **primeiro** `yield`.

Vê o descompasso -- segunda-para-primeira?

Mas vamos inverter a nossa perspectiva. Vamos olhar para isso não do ponto de vista do gerador, mas do ponto de vista do iterator.

Para ilustrar adequadamente essa perspectiva, também precisamos explicar que as mensagens podem ir em ambas as direções -- `yield ..` como uma expressão pode enviar mensagens para fora em resposta a chamadas `next(..)`, e `next(..)` pode enviar valores para uma expressão `yield` pausada. Considere este código ligeiramente ajustado:

```js
function *foo(x) {
	var y = x * (yield "Hello");	// <-- yield de um valor!
	return y;
}

var it = foo( 6 );

var res = it.next();	// primeiro `next()`, não passa nada
res.value;				// "Hello"

res = it.next( 7 );		// passa `7` para o `yield` que está esperando
res.value;				// 42
```

`yield ..` e `next(..)` se emparelham como um sistema de passagem de mensagens em duas vias **durante a execução do gerador**.

Então, olhando apenas para o código do *iterator*:

```js
var res = it.next();	// primeiro `next()`, não passa nada
res.value;				// "Hello"

res = it.next( 7 );		// passa `7` para o `yield` que está esperando
res.value;				// 42
```

**Nota:** Nós não passamos um valor para a primeira chamada `next()`, e isso é de propósito. Apenas um `yield` pausado poderia aceitar tal valor passado por um `next(..)`, e no começo do gerador, quando chamamos o primeiro `next()`, **não há um `yield` pausado** para aceitar tal valor. A especificação e todos os navegadores compatíveis simplesmente **descartam** silenciosamente qualquer coisa passada para o primeiro `next()`. Ainda é uma má ideia passar um valor, pois você está apenas criando código que "falha" silenciosamente e que é confuso. Então, sempre inicie um gerador com um `next()` sem argumentos.

A primeira chamada `next()` (sem nada passado para ela) está basicamente *fazendo uma pergunta*: "Qual *próximo* valor o gerador `*foo(..)` tem para me dar?" E quem responde a essa pergunta? A primeira expressão `yield "hello"`.

Viu? Sem descompasso aí.

Dependendo de *quem* você acha que está fazendo a pergunta, há ou não um descompasso entre as chamadas `yield` e `next(..)`.

Mas espere! Ainda há um `next()` extra comparado ao número de instruções `yield`. Então, aquela chamada final `it.next(7)` está novamente fazendo a pergunta sobre qual *próximo* valor o gerador vai produzir. Mas não há mais instruções `yield` restantes para responder, há? Então quem responde?

A instrução `return` responde a pergunta!

E se **não houver `return`** no seu gerador -- `return` certamente não é mais obrigatório em geradores do que em funções comuns -- há sempre um `return;` (também conhecido como `return undefined;`) assumido/implícito, que serve ao propósito de responder por padrão a pergunta *colocada* pela chamada final `it.next(7)`.

Essas perguntas e respostas -- a passagem de mensagens em duas vias com `yield` e `next(..)` -- são bastante poderosas, mas não é nada óbvio como esses mecanismos estão conectados ao controle de fluxo assíncrono. Nós estamos chegando lá!

### Múltiplos Iterators

Pode parecer, pelo uso sintático, que quando você usa um *iterator* para controlar um gerador, você está controlando a própria função geradora declarada. Mas há uma sutileza fácil de não perceber: cada vez que você constrói um *iterator*, você está implicitamente construindo uma instância do gerador que esse *iterator* vai controlar.

Você pode ter múltiplas instâncias do mesmo gerador rodando ao mesmo tempo, e elas podem até interagir:

```js
function *foo() {
	var x = yield 2;
	z++;
	var y = yield (x * z);
	console.log( x, y, z );
}

var z = 1;

var it1 = foo();
var it2 = foo();

var val1 = it1.next().value;			// 2 <-- yield 2
var val2 = it2.next().value;			// 2 <-- yield 2

val1 = it1.next( val2 * 10 ).value;		// 40  <-- x:20,  z:2
val2 = it2.next( val1 * 5 ).value;		// 600 <-- x:200, z:3

it1.next( val2 / 2 );					// y:300
										// 20 300 3
it2.next( val1 / 4 );					// y:10
										// 200 10 3
```

**Atenção:** O uso mais comum de múltiplas instâncias do mesmo gerador rodando concorrentemente não são essas interações, mas sim quando o gerador está produzindo seus próprios valores sem entrada, talvez a partir de algum recurso conectado independentemente. Nós falaremos mais sobre produção de valores na próxima seção.

Vamos percorrer brevemente o processamento:

1. Ambas as instâncias de `*foo()` são iniciadas ao mesmo tempo, e ambas as chamadas `next()` revelam um `value` de `2` das instruções `yield 2`, respectivamente.
2. `val2 * 10` é `2 * 10`, que é enviado para a primeira instância do gerador `it1`, de forma que `x` recebe o valor `20`. `z` é incrementado de `1` para `2`, e então `20 * 2` é cedido (`yield`) para fora, definindo `val1` para `40`.
3. `val1 * 5` é `40 * 5`, que é enviado para a segunda instância do gerador `it2`, de forma que `x` recebe o valor `200`. `z` é incrementado novamente, de `2` para `3`, e então `200 * 3` é cedido (`yield`) para fora, definindo `val2` para `600`.
4. `val2 / 2` é `600 / 2`, que é enviado para a primeira instância do gerador `it1`, de forma que `y` recebe o valor `300`, então imprimindo `20 300 3` para seus valores `x y z`, respectivamente.
5. `val1 / 4` é `40 / 4`, que é enviado para a segunda instância do gerador `it2`, de forma que `y` recebe o valor `10`, então imprimindo `200 10 3` para seus valores `x y z`, respectivamente.

Esse é um exemplo "divertido" de se percorrer mentalmente. Você conseguiu mantê-lo organizado?

#### Intercalando

Relembre este cenário da seção "Rodar-até-acabar" do capítulo 1:

```js
var a = 1;
var b = 2;

function foo() {
	a++;
	b = b * a;
	a = b + 3;
}

function bar() {
	b--;
	a = 8 + b;
	b = a * 2;
}
```

Com funções JS normais, é claro que ou `foo()` pode rodar completamente primeiro, ou `bar()` pode rodar completamente primeiro, mas `foo()` não pode intercalar suas instruções individuais com `bar()`. Então, há apenas dois resultados possíveis para o programa anterior.

No entanto, com geradores, claramente intercalar (até mesmo no meio das instruções!) é possível:

```js
var a = 1;
var b = 2;

function *foo() {
	a++;
	yield;
	b = b * a;
	a = (yield b) + 3;
}

function *bar() {
	b--;
	yield;
	a = (yield 8) + b;
	b = a * (yield 2);
}
```

Dependendo de qual ordem respectiva os *iterators* que controlam `*foo()` e `*bar()` são chamados, o programa anterior poderia produzir vários resultados diferentes. Em outras palavras, podemos de fato ilustrar (de uma forma meio fingida) as circunstâncias teóricas de "condições de corrida com threads" discutidas no capítulo 1, intercalando as iterações dos dois geradores sobre as mesmas variáveis compartilhadas.

Primeiro, vamos fazer uma função auxiliar chamada `step(..)` que controla um *iterator*:

```js
function step(gen) {
	var it = gen();
	var last;

	return function() {
		// o que quer que seja cedido (`yield`) para fora,
		// apenas envie de volta na próxima vez!
		last = it.next( last ).value;
	};
}
```

`step(..)` inicializa um gerador para criar seu *iterator* `it`, então retorna uma função que, quando chamada, avança o *iterator* em um passo. Adicionalmente, o valor previamente cedido (`yield`) para fora é enviado de volta no *próximo* passo. Então, `yield 8` simplesmente se tornará `8` e `yield b` simplesmente será `b` (qualquer que fosse seu valor no momento do `yield`).

Agora, só por diversão, vamos experimentar para ver os efeitos de intercalar esses diferentes pedaços de `*foo()` e `*bar()`. Vamos começar com o caso base entediante, garantindo que `*foo()` termine totalmente antes de `*bar()` (assim como fizemos no capítulo 1):

```js
// certifique-se de resetar `a` e `b`
a = 1;
b = 2;

var s1 = step( foo );
var s2 = step( bar );

// roda `*foo()` completamente primeiro
s1();
s1();
s1();

// agora roda `*bar()`
s2();
s2();
s2();
s2();

console.log( a, b );	// 11 22
```

O resultado final é `11` e `22`, exatamente como era na versão do capítulo 1. Agora vamos misturar a ordem de intercalação e ver como isso muda os valores finais de `a` e `b`:

```js
// certifique-se de resetar `a` e `b`
a = 1;
b = 2;

var s1 = step( foo );
var s2 = step( bar );

s2();		// b--;
s2();		// yield 8
s1();		// a++;
s2();		// a = 8 + b;
			// yield 2
s1();		// b = b * a;
			// yield b
s1();		// a = b + 3;
s2();		// b = a * 2;
```

Antes de eu te dizer os resultados, você consegue descobrir quais são `a` e `b` depois do programa anterior? Sem trapacear!

```js
console.log( a, b );	// 12 18
```

**Nota:** Como um exercício para o leitor, tente ver quantas outras combinações de resultados você consegue obter rearranjando a ordem das chamadas `s1()` e `s2()`. Não esqueça que você sempre precisará de três chamadas `s1()` e quatro chamadas `s2()`. Relembre a discussão anterior sobre combinar `next()` com `yield` para entender os motivos.

Você quase certamente não vai querer criar intencionalmente *este* nível de confusão de intercalação, pois ele cria um código incrivelmente difícil de entender. Mas o exercício é interessante e instrutivo para entender mais sobre como múltiplos geradores podem rodar concorrentemente no mesmo escopo compartilhado, porque haverá lugares onde essa capacidade é bem útil.

Nós discutiremos a concorrência de geradores em mais detalhes no fim deste capítulo.

## Gerando Valores

Na seção anterior, mencionamos um uso interessante para geradores, como uma forma de produzir valores. Este **não** é o foco principal neste capítulo, mas seríamos negligentes se não cobríssemos o básico, especialmente porque esse caso de uso é essencialmente a origem do nome: geradores (generators).

Nós vamos fazer um leve desvio para o tópico de *iterators* por um tempo, mas voltaremos a como eles se relacionam com geradores e a usar um gerador para *gerar* valores.

### Produtores e Iterators

Imagine que você está produzindo uma série de valores onde cada valor tem uma relação definível com o valor anterior. Para fazer isso, você vai precisar de um produtor com estado que lembra o último valor que deu.

Você pode implementar algo assim de forma direta usando um closure de função (veja o título *Escopos & Closures* desta série):

```js
var gimmeSomething = (function(){
	var nextVal;

	return function(){
		if (nextVal === undefined) {
			nextVal = 1;
		}
		else {
			nextVal = (3 * nextVal) + 6;
		}

		return nextVal;
	};
})();

gimmeSomething();		// 1
gimmeSomething();		// 9
gimmeSomething();		// 33
gimmeSomething();		// 105
```

**Nota:** A lógica de cálculo do `nextVal` aqui poderia ter sido simplificada, mas conceitualmente, não queremos calcular o *próximo valor* (também conhecido como `nextVal`) até que a *próxima* chamada `gimmeSomething()` aconteça, porque, em geral, isso poderia ser um design propenso a vazamento de recursos para produtores de valores mais persistentes ou limitados em recursos do que simples `number`s.

Gerar uma série numérica arbitrária não é um exemplo terrivelmente realista. Mas e se você estivesse gerando registros de uma fonte de dados? Você poderia imaginar praticamente o mesmo código.

De fato, essa tarefa é um padrão de design muito comum, geralmente resolvido por iterators. Um *iterator* é uma interface bem definida para percorrer uma série de valores de um produtor. A interface JS para iterators, como é na maioria das linguagens, é chamar `next()` toda vez que você quer o próximo valor do produtor.

Nós poderíamos implementar a interface padrão de *iterator* para nosso produtor de série numérica:

```js
var something = (function(){
	var nextVal;

	return {
		// necessário para loops `for..of`
		[Symbol.iterator]: function(){ return this; },

		// método padrão da interface de iterator
		next: function(){
			if (nextVal === undefined) {
				nextVal = 1;
			}
			else {
				nextVal = (3 * nextVal) + 6;
			}

			return { done:false, value:nextVal };
		}
	};
})();

something.next().value;		// 1
something.next().value;		// 9
something.next().value;		// 33
something.next().value;		// 105
```

**Nota:** Nós explicaremos por que precisamos da parte `[Symbol.iterator]: ..` deste trecho de código na seção "Iterables". Sintaticamente, porém, dois recursos do ES6 estão em jogo. Primeiro, a sintaxe `[ .. ]` é chamada de *nome de propriedade computado* (veja o título *this & Object Prototypes* desta série). É uma forma, em uma definição de objeto literal, de especificar uma expressão e usar o resultado dessa expressão como o nome da propriedade. Em seguida, `Symbol.iterator` é um dos valores `Symbol` especiais predefinidos do ES6 (veja o título *ES6 & Além* desta série de livros).

A chamada `next()` retorna um objeto com duas propriedades: `done` é um valor `boolean` sinalizando o status de conclusão do *iterator*; `value` contém o valor da iteração.

ES6 também adiciona o loop `for..of`, o que significa que um *iterator* padrão pode ser automaticamente consumido com a sintaxe nativa de loop:

```js
for (var v of something) {
	console.log( v );

	// não deixe o loop rodar para sempre!
	if (v > 500) {
		break;
	}
}
// 1 9 33 105 321 969
```

**Nota:** Como nosso *iterator* `something` sempre retorna `done:false`, este loop `for..of` rodaria para sempre, e é por isso que colocamos o condicional `break`. É totalmente aceitável para iterators serem infinitos, mas há também casos onde o *iterator* vai percorrer um conjunto finito de valores e eventualmente retornar um `done:true`.

O loop `for..of` automaticamente chama `next()` para cada iteração -- ele não passa nenhum valor para o `next()` -- e ele vai automaticamente terminar ao receber um `done:true`. É bem prático para fazer loop sobre um conjunto de dados.

Claro, você poderia fazer loop manualmente sobre iterators, chamando `next()` e verificando a condição `done:true` para saber quando parar:

```js
for (
	var ret;
	(ret = something.next()) && !ret.done;
) {
	console.log( ret.value );

	// não deixe o loop rodar para sempre!
	if (ret.value > 500) {
		break;
	}
}
// 1 9 33 105 321 969
```

**Nota:** Essa abordagem manual com `for` é certamente mais feia do que a sintaxe do loop `for..of` do ES6, mas sua vantagem é que ela te dá a oportunidade de passar valores para as chamadas `next(..)` se necessário.

Além de criar seus próprios *iterators*, muitas estruturas de dados embutidas em JS (a partir do ES6), como `array`s, também têm *iterators* padrão:

```js
var a = [1,3,5,7,9];

for (var v of a) {
	console.log( v );
}
// 1 3 5 7 9
```

O loop `for..of` pede a `a` por seu *iterator*, e automaticamente o usa para iterar sobre os valores de `a`.

**Nota:** Pode parecer uma omissão estranha do ES6, mas `object`s comuns intencionalmente não vêm com um *iterator* padrão do jeito que `array`s vêm. As razões são mais profundas do que cobriremos aqui. Se tudo o que você quer é iterar sobre as propriedades de um objeto (sem nenhuma garantia particular de ordenação), `Object.keys(..)` retorna um `array`, que pode então ser usado como `for (var k of Object.keys(obj)) { ..`. Um loop `for..of` desse tipo sobre as chaves de um objeto seria similar a um loop `for..in`, exceto que `Object.keys(..)` não inclui propriedades da cadeia `[[Prototype]]` enquanto `for..in` inclui (veja o título *this & Object Prototypes* desta série).

### Iterables

O objeto `something` em nosso exemplo corrente é chamado de *iterator*, pois ele tem o método `next()` em sua interface. Mas um termo intimamente relacionado é *iterable*, que é um `object` que **contém** um *iterator* capaz de iterar sobre seus valores.

A partir do ES6, a forma de recuperar um *iterator* de um *iterable* é que o *iterable* deve ter uma função nele, com o nome sendo o valor especial de símbolo do ES6 `Symbol.iterator`. Quando essa função é chamada, ela retorna um *iterator*. Embora não seja obrigatório, geralmente cada chamada deve retornar um *iterator* novinho em folha.

`a` no trecho anterior é um *iterable*. O loop `for..of` automaticamente chama sua função `Symbol.iterator` para construir um *iterator*. Mas nós poderíamos, é claro, chamar a função manualmente, e usar o *iterator* que ela retorna:

```js
var a = [1,3,5,7,9];

var it = a[Symbol.iterator]();

it.next().value;	// 1
it.next().value;	// 3
it.next().value;	// 5
..
```

Na listagem de código anterior que definiu `something`, você pode ter notado esta linha:

```js
[Symbol.iterator]: function(){ return this; }
```

Esse pedacinho de código confuso está fazendo o valor `something` -- a interface do *iterator* `something` -- ser também um *iterable*; ele agora é tanto um *iterable* quanto um *iterator*. Então, passamos `something` para o loop `for..of`:

```js
for (var v of something) {
	..
}
```

O loop `for..of` espera que `something` seja um *iterable*, então ele procura e chama sua função `Symbol.iterator`. Nós definimos essa função para simplesmente fazer `return this`, então ela apenas devolve a si mesma, e o loop `for..of` nem percebe a diferença.

### Iterator de Gerador

Vamos voltar nossa atenção para os geradores, no contexto de *iterators*. Um gerador pode ser tratado como um produtor de valores que extraímos um de cada vez através das chamadas `next()` de uma interface de *iterator*.

Então, um gerador em si não é tecnicamente um *iterable*, embora seja bem similar -- quando você executa o gerador, você recebe um *iterator* de volta:

```js
function *foo(){ .. }

var it = foo();
```

Nós podemos implementar o produtor de série numérica infinita `something` de antes com um gerador, assim:

```js
function *something() {
	var nextVal;

	while (true) {
		if (nextVal === undefined) {
			nextVal = 1;
		}
		else {
			nextVal = (3 * nextVal) + 6;
		}

		yield nextVal;
	}
}
```

**Nota:** Um loop `while..true` normalmente seria uma coisa muito ruim de se incluir em um programa JS real, ao menos se ele não tiver um `break` ou `return` nele, pois ele provavelmente rodaria para sempre, de forma síncrona, e travaria/bloquearia a UI do navegador. No entanto, em um gerador, tal loop é geralmente totalmente aceitável se tiver um `yield` nele, já que o gerador vai pausar a cada iteração, cedendo (`yield`) de volta ao programa principal e/ou à fila do loop de eventos. Para colocar de forma jocosa, "geradores trouxeram o `while..true` de volta para a programação JS!"

Isso é bem mais limpo e simples, certo? Porque o gerador pausa a cada `yield`, o estado (escopo) da função `*something()` é mantido por perto, o que significa que não há necessidade do boilerplate de closure para preservar o estado das variáveis entre as chamadas.

Não só é um código mais simples -- não precisamos fazer nossa própria interface de *iterator* -- ele na verdade é um código mais sensato (reason-able), porque ele expressa mais claramente a intenção. Por exemplo, o loop `while..true` nos diz que o gerador foi feito para rodar para sempre -- para continuar *gerando* valores enquanto continuarmos pedindo por eles.

E agora podemos usar nosso novinho gerador `*something()` com um loop `for..of`, e você verá que ele funciona basicamente de forma idêntica:

```js
for (var v of something()) {
	console.log( v );

	// não deixe o loop rodar para sempre!
	if (v > 500) {
		break;
	}
}
// 1 9 33 105 321 969
```

Mas não pule por cima do `for (var v of something()) ..`! Nós não apenas referenciamos `something` como um valor, como nos exemplos anteriores, mas em vez disso chamamos o gerador `*something()` para obter seu *iterator* para o loop `for..of` usar.

Se você estiver prestando bastante atenção, duas perguntas podem surgir dessa interação entre o gerador e o loop:

* Por que não pudemos dizer `for (var v of something) ..`? Porque `something` aqui é um gerador, que não é um *iterable*. Nós temos que chamar `something()` para construir um produtor para o loop `for..of` iterar.
* A chamada `something()` produz um *iterator*, mas o loop `for..of` quer um *iterable*, certo? Sim. O *iterator* do gerador também tem uma função `Symbol.iterator` nele, que basicamente faz um `return this`, exatamente como o *iterable* `something` que definimos antes. Em outras palavras, o *iterator* de um gerador também é um *iterable*!

#### Parando o Gerador

No exemplo anterior, pareceria que a instância do *iterator* para o gerador `*something()` ficou basicamente deixada em um estado suspenso para sempre depois que o `break` no loop foi chamado.

Mas há um comportamento oculto que cuida disso para você. A "conclusão anormal" (ou seja, "terminação precoce") do loop `for..of` -- geralmente causada por um `break`, `return`, ou uma exceção não capturada -- envia um sinal para o *iterator* do gerador para que ele termine.

**Nota:** Tecnicamente, o loop `for..of` também envia esse sinal para o *iterator* na conclusão normal do loop. Para um gerador, isso é essencialmente uma operação irrelevante, pois o *iterator* do gerador teve que terminar primeiro para que o loop `for..of` terminasse. No entanto, *iterators* customizados podem desejar receber esse sinal adicional dos consumidores do loop `for..of`.

Embora um loop `for..of` vá automaticamente enviar esse sinal, você pode desejar enviar o sinal manualmente para um *iterator*; você faz isso chamando `return(..)`.

Se você especificar uma cláusula `try..finally` dentro do gerador, ela sempre será rodada mesmo quando o gerador for concluído externamente. Isso é útil se você precisar limpar recursos (conexões de banco de dados, etc.):

```js
function *something() {
	try {
		var nextVal;

		while (true) {
			if (nextVal === undefined) {
				nextVal = 1;
			}
			else {
				nextVal = (3 * nextVal) + 6;
			}

			yield nextVal;
		}
	}
	// cláusula de limpeza
	finally {
		console.log( "cleaning up!" );
	}
}
```

O exemplo anterior com `break` no loop `for..of` vai disparar a cláusula `finally`. Mas você poderia, em vez disso, terminar manualmente a instância do *iterator* do gerador de fora com `return(..)`:

```js
var it = something();
for (var v of it) {
	console.log( v );

	// não deixe o loop rodar para sempre!
	if (v > 500) {
		console.log(
			// conclui o iterator do gerador
			it.return( "Hello World" ).value
		);
		// nenhum `break` necessário aqui
	}
}
// 1 9 33 105 321 969
// cleaning up!
// Hello World
```

Quando chamamos `it.return(..)`, ele imediatamente termina o gerador, o que, claro, roda a cláusula `finally`. Além disso, ele define o `value` retornado para o que quer que você tenha passado para `return(..)`, que é como `"Hello World"` volta logo em seguida. Nós também não precisamos incluir um `break` agora, porque o *iterator* do gerador é definido para `done:true`, então o loop `for..of` vai terminar em sua próxima iteração.

Geradores devem seu nome principalmente a este uso de *consumir valores produzidos*. Mas, novamente, esse é apenas um dos usos para geradores, e francamente nem mesmo o principal com o qual nos preocupamos no contexto deste livro.

Mas agora que entendemos mais plenamente algumas das mecânicas de como eles funcionam, podemos *a seguir* voltar nossa atenção para como geradores se aplicam à concorrência assíncrona.

## Iterando Geradores Assincronamente

O que geradores têm a ver com padrões de codificação assíncrona, com corrigir problemas com callbacks, e coisas assim? Vamos chegar à resposta dessa pergunta importante.

Devemos revisitar um dos nossos cenários do capítulo 3. Vamos relembrar a abordagem com callback:

```js
function foo(x,y,cb) {
	ajax(
		"http://some.url.1/?x=" + x + "&y=" + y,
		cb
	);
}

foo( 11, 31, function(err,text) {
	if (err) {
		console.error( err );
	}
	else {
		console.log( text );
	}
} );
```

Se quiséssemos expressar esse mesmo controle de fluxo de tarefa com um gerador, poderíamos fazer:

```js
function foo(x,y) {
	ajax(
		"http://some.url.1/?x=" + x + "&y=" + y,
		function(err,data){
			if (err) {
				// lança um erro para dentro de `*main()`
				it.throw( err );
			}
			else {
				// retoma `*main()` com os `data` recebidos
				it.next( data );
			}
		}
	);
}

function *main() {
	try {
		var text = yield foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}

var it = main();

// inicie tudo!
it.next();
```

À primeira vista, este trecho é mais longo, e talvez com uma aparência um pouco mais complexa, do que o trecho com callback anterior a ele. Mas não deixe essa impressão te desviar. O trecho com gerador é na verdade **muito** melhor! Mas há muita coisa acontecendo para nós explicarmos.

Primeiro, vamos olhar para esta parte do código, que é a mais importante:

```js
var text = yield foo( 11, 31 );
console.log( text );
```

Pense por um momento em como esse código funciona. Estamos chamando uma função normal `foo(..)` e aparentemente conseguimos obter de volta o `text` da chamada Ajax, mesmo sendo assíncrono.

Como isso é possível? Se você relembrar o começo do capítulo 1, nós tínhamos um código quase idêntico:

```js
var data = ajax( "..url 1.." );
console.log( data );
```

E aquele código não funcionava! Você consegue identificar a diferença? É o `yield` usado em um gerador.

Essa é a mágica! É isso que nos permite ter o que parece ser código bloqueante, síncrono, mas que na verdade não bloqueia o programa inteiro; ele apenas pausa/bloqueia o código dentro do próprio gerador.

Em `yield foo(11,31)`, primeiro a chamada `foo(11,31)` é feita, que não retorna nada (ou seja, `undefined`), então estamos fazendo uma chamada para requisitar dados, mas na verdade estamos então fazendo `yield undefined`. Isso é OK, porque o código não está atualmente dependendo de um valor cedido (`yield`) para fazer algo interessante. Nós revisitaremos esse ponto mais adiante no capítulo.

Não estamos usando `yield` num sentido de passagem de mensagens aqui, apenas num sentido de controle de fluxo para pausar/bloquear. Na verdade, ele vai ter passagem de mensagens, mas apenas em uma direção, depois que o gerador for retomado.

Então, o gerador pausa no `yield`, essencialmente fazendo a pergunta: "qual valor eu devo retornar para atribuir à variável `text`?" Quem vai responder a essa pergunta?

Olhe para `foo(..)`. Se a requisição Ajax for bem-sucedida, nós chamamos:

```js
it.next( data );
```

Isso está retomando o gerador com os dados de resposta, o que significa que nossa expressão `yield` pausada recebe esse valor diretamente, e então, ao reiniciar o código do gerador, esse valor é atribuído à variável local `text`.

Bem legal, né?

Dê um passo atrás e considere as implicações. Nós temos um código que parece totalmente síncrono dentro do gerador (além da própria palavra-chave `yield`), mas escondido nos bastidores, dentro de `foo(..)`, as operações podem ser concluídas de forma assíncrona.

**Isso é enorme!** Essa é uma solução quase perfeita para o nosso problema declarado anteriormente de que callbacks não conseguem expressar assincronia de uma forma sequencial, síncrona, com a qual nossos cérebros conseguem se relacionar.

Em essência, estamos abstraindo a assincronia para longe, como um detalhe de implementação, de forma que possamos raciocinar de maneira síncrona/sequencial sobre nosso controle de fluxo: "Faça uma requisição Ajax, e quando ela terminar, imprima a resposta." E, claro, nós apenas expressamos dois passos no controle de fluxo, mas essa mesma capacidade se estende sem limites, para nos deixar expressar quantos passos precisarmos.

**Dica:** Essa é uma percepção tão importante, simplesmente volte e leia os três últimos parágrafos novamente para que ela seja absorvida!

### Tratamento Síncrono de Erros

Mas o código do gerador anterior tem ainda mais bondade para nos *ceder* (yield). Vamos voltar nossa atenção para o `try..catch` dentro do gerador:

```js
try {
	var text = yield foo( 11, 31 );
	console.log( text );
}
catch (err) {
	console.error( err );
}
```

Como isso funciona? A chamada `foo(..)` está sendo concluída de forma assíncrona, e o `try..catch` não falha em capturar erros assíncronos, como vimos no capítulo 3?

Nós já vimos como o `yield` deixa a instrução de atribuição pausar para esperar `foo(..)` terminar, de forma que a resposta concluída possa ser atribuída a `text`. A parte incrível é que essa pausa do `yield` *também* permite que o gerador `catch` (capture) um erro. Nós lançamos esse erro para dentro do gerador com esta parte da listagem de código anterior:

```js
if (err) {
	// lança um erro para dentro de `*main()`
	it.throw( err );
}
```

A natureza de pausa-no-`yield` dos geradores significa que não só obtemos valores de `return` com aparência síncrona de chamadas de função assíncronas, mas também podemos `catch` (capturar) erros dessas chamadas de função assíncronas de forma síncrona!

Então vimos que podemos lançar erros *para dentro* de um gerador, mas e quanto a lançar erros *para fora* de um gerador? Exatamente como você esperaria:

```js
function *main() {
	var x = yield "Hello World";

	yield x.toLowerCase();	// causa uma exceção!
}

var it = main();

it.next().value;			// Hello World

try {
	it.next( 42 );
}
catch (err) {
	console.error( err );	// TypeError
}
```

Claro, poderíamos ter lançado um erro manualmente com `throw ..` em vez de causar uma exceção.

Podemos até `catch` (capturar) o mesmo erro que nós `throw(..)` (lançamos) para dentro do gerador, essencialmente dando ao gerador uma chance de tratá-lo, mas se ele não tratar, o código do *iterator* deve tratá-lo:

```js
function *main() {
	var x = yield "Hello World";

	// nunca chega aqui
	console.log( x );
}

var it = main();

it.next();

try {
	// será que `*main()` vai tratar esse erro? vamos ver!
	it.throw( "Oops" );
}
catch (err) {
	// não, não tratou!
	console.error( err );			// Oops
}
```

Tratamento de erros com aparência síncrona (via `try..catch`) com código assíncrono é uma grande vitória para a legibilidade e a sensatez (reason-ability).

## Generators + Promises

Em nossa discussão anterior, mostramos como geradores podem ser iterados assincronamente, o que é um enorme passo adiante em sensatez sequencial em relação à bagunça de espaguete dos callbacks. Mas nós perdemos algo muito importante: a confiabilidade e a composibilidade das Promises (veja o capítulo 3)!

Não se preocupe -- nós podemos recuperar isso. O melhor de todos os mundos no ES6 é combinar geradores (código assíncrono com aparência síncrona) com Promises (confiáveis e combináveis).

Mas como?

Relembre do capítulo 3 a abordagem baseada em Promises para o nosso exemplo corrente de Ajax:

```js
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

foo( 11, 31 )
.then(
	function(text){
		console.log( text );
	},
	function(err){
		console.error( err );
	}
);
```

Em nosso código de gerador anterior para o exemplo corrente de Ajax, `foo(..)` não retornava nada (`undefined`), e nosso código de controle do *iterator* não se importava com esse valor cedido (`yield`).

Mas aqui o `foo(..)` ciente de Promises retorna uma promise depois de fazer a chamada Ajax. Isso sugere que poderíamos construir uma promise com `foo(..)` e então cedê-la (`yield`) do gerador, e então o código de controle do *iterator* receberia essa promise.

Mas o que o *iterator* deveria fazer com a promise?

Ele deveria escutar a promise se resolver (fulfillment ou rejection), e então ou retomar o gerador com a mensagem de fulfillment, ou lançar um erro para dentro do gerador com a razão da rejection.

Deixe-me repetir isso, pois é muito importante. A forma natural de tirar o máximo proveito de Promises e geradores é **ceder (`yield`) uma Promise**, e conectar essa Promise para controlar o *iterator* do gerador.

Vamos tentar! Primeiro, juntaremos o `foo(..)` ciente de Promises com o gerador `*main()`:

```js
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

function *main() {
	try {
		var text = yield foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}
```

A revelação mais poderosa nesse refatoramento é que o código dentro de `*main()` **não precisou mudar nada!** Dentro do gerador, quaisquer que sejam os valores cedidos (`yield`) para fora, é apenas um detalhe de implementação opaco, então nem sequer estamos cientes de que isso está acontecendo, nem precisamos nos preocupar com isso.

Mas como vamos rodar `*main()` agora? Ainda temos algum trabalho de encanamento de implementação a fazer, para receber e conectar a promise cedida (`yield`) de forma que ela retome o gerador na resolução. Vamos começar tentando isso manualmente:

```js
var it = main();

var p = it.next().value;

// espera a promise `p` se resolver
p.then(
	function(text){
		it.next( text );
	},
	function(err){
		it.throw( err );
	}
);
```

Na verdade, isso não foi tão doloroso, foi?

Esse trecho deve parecer muito similar ao que fizemos antes com o gerador conectado manualmente, controlado pelo callback error-first. Em vez de um `if (err) { it.throw..`, a promise já separa fulfillment (sucesso) e rejection (falha) para nós, mas, fora isso, o controle do *iterator* é idêntico.

Agora, nós passamos por cima de alguns detalhes importantes.

Mais importante, tiramos proveito do fato de que sabíamos que `*main()` tinha apenas um passo ciente de Promises nele. E se quiséssemos ser capazes de dirigir por Promise um gerador não importa quantos passos ele tenha? Nós certamente não queremos escrever manualmente a cadeia de Promises de forma diferente para cada gerador! O que seria muito melhor seria se houvesse uma forma de repetir (ou seja, fazer "loop") sobre o controle de iteração, e cada vez que uma Promise saísse, esperar pela sua resolução antes de continuar.

Além disso, e se o gerador lançar um erro (intencionalmente ou acidentalmente) durante a chamada `it.next(..)`? Devemos desistir, ou devemos `catch` (capturar) e enviá-lo de volta para dentro? Da mesma forma, e se nós `it.throw(..)` (lançarmos) uma rejection de Promise para dentro do gerador, mas ela não for tratada, e voltar logo em seguida?

### Executor de Geradores Ciente de Promises

Quanto mais você começa a explorar esse caminho, mais você percebe: "uau, seria ótimo se existisse algum utilitário para fazer isso por mim." E você está absolutamente correto. Esse é um padrão tão importante, e você não quer errá-lo (ou se exaurir repetindo-o vez após vez), então sua melhor aposta é usar um utilitário que seja especificamente projetado para *executar* geradores que cedem (`yield`) Promises da maneira que ilustramos.

Várias bibliotecas de abstração de Promises fornecem justamente tal utilitário, incluindo minha biblioteca *asynquence* e seu `runner(..)`, que será discutido no Apêndice A deste livro.

Mas, por uma questão de aprendizado e ilustração, vamos apenas definir nosso próprio utilitário independente que chamaremos de `run(..)`:

```js
// obrigado a Benjamin Gruenbaum (@benjamingr no GitHub) por
// grandes melhorias aqui!
function run(gen) {
	var args = [].slice.call( arguments, 1), it;

	// inicializa o gerador no contexto atual
	it = gen.apply( this, args );

	// retorna uma promise para a conclusão do gerador
	return Promise.resolve()
		.then( function handleNext(value){
			// roda até o próximo valor cedido (yield)
			var next = it.next( value );

			return (function handleResult(next){
				// o gerador terminou de rodar?
				if (next.done) {
					return next.value;
				}
				// caso contrário, continue
				else {
					return Promise.resolve( next.value )
						.then(
							// retoma o loop assíncrono no
							// sucesso, enviando o valor resolvido
							// de volta para dentro do gerador
							handleNext,

							// se `value` for uma promise
							// rejeitada, propaga o erro de volta
							// para dentro do gerador para seu próprio
							// tratamento de erro
							function handleErr(err) {
								return Promise.resolve(
									it.throw( err )
								)
								.then( handleResult );
							}
						);
				}
			})(next);
		} );
}
```

Como você pode ver, ele é um bom tanto mais complexo do que você provavelmente gostaria de escrever você mesmo, e você especialmente não gostaria de repetir esse código para cada gerador que usar. Então, um utilitário/biblioteca auxiliar é definitivamente o caminho a seguir. Mesmo assim, eu te encorajo a passar alguns minutos estudando essa listagem de código para ter uma melhor noção de como gerenciar a negociação gerador+Promise.

Como você usaria `run(..)` com `*main()` em nosso exemplo *corrente* de Ajax?

```js
function *main() {
	// ..
}

run( main );
```

É isso! Da forma que conectamos `run(..)`, ele vai automaticamente avançar o gerador que você passa para ele, assincronamente até a conclusão.

**Nota:** O `run(..)` que definimos retorna uma promise que está conectada para se resolver assim que o gerador estiver completo, ou para receber uma exceção não capturada se o gerador não a tratar. Nós não mostramos essa capacidade aqui, mas voltaremos a ela mais adiante no capítulo.

#### ES7: `async` e `await`?

O padrão anterior -- geradores cedendo (`yield`) Promises que então controlam o *iterator* do gerador para avançá-lo até a conclusão -- é uma abordagem tão poderosa e útil, que seria melhor se pudéssemos fazê-la sem a desordem do auxiliar utilitário da biblioteca (ou seja, `run(..)`).

Provavelmente há boas notícias nessa frente. No momento em que isto é escrito, há um suporte inicial mas forte para uma proposta de mais adição sintática nesse domínio para o período pós-ES6, mais ou menos do ES7. Obviamente, é cedo demais para garantir os detalhes, mas há uma chance bem decente de que ela se concretize de forma similar ao seguinte:

```js
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

async function main() {
	try {
		var text = await foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}

main();
```

Como você pode ver, não há chamada `run(..)` (o que significa que não há necessidade de um utilitário de biblioteca!) para invocar e dirigir `main()` -- ele é apenas chamado como uma função normal. Além disso, `main()` não é mais declarado como uma função geradora; é um novo tipo de função: `async function`. E, finalmente, em vez de ceder (`yield`) uma Promise, nós `await` (aguardamos) que ela se resolva.

A `async function` automaticamente sabe o que fazer se você `await` uma Promise -- ela vai pausar a função (assim como com geradores) até a Promise se resolver. Nós não ilustramos isso neste trecho, mas chamar uma função async como `main()` automaticamente retorna uma promise que é resolvida sempre que a função terminar completamente.

**Dica:** A sintaxe `async` / `await` deve parecer muito familiar para leitores com experiência em C#, pois ela é basicamente idêntica.

A proposta essencialmente codifica suporte para o padrão que já derivamos, em um mecanismo sintático: combinar Promises com código de controle de fluxo com aparência síncrona. Esse é o melhor de ambos os mundos combinado, para efetivamente lidar com praticamente todas as principais preocupações que delineamos com callbacks.

O mero fato de que tal proposta mais ou menos do ES7 já exista e tenha suporte e entusiasmo iniciais é um grande voto de confiança na importância futura desse padrão assíncrono.

### Concorrência de Promises em Geradores

Até aqui, tudo o que demonstramos foi um fluxo assíncrono de passo único com Promises+geradores. Mas o código do mundo real frequentemente terá muitos passos assíncronos.

Se você não tomar cuidado, o estilo de aparência síncrona dos geradores pode te embalar numa complacência sobre como você estrutura sua concorrência assíncrona, levando a padrões de desempenho subótimos. Então queremos passar um pouco de tempo explorando as opções.

Imagine um cenário onde você precisa buscar dados de duas fontes diferentes, depois combinar essas respostas para fazer uma terceira requisição, e finalmente imprimir a última resposta. Nós exploramos um cenário similar com Promises no capítulo 3, mas vamos reconsiderá-lo no contexto de geradores.

Seu primeiro instinto pode ser algo como:

```js
function *foo() {
	var r1 = yield request( "http://some.url.1" );
	var r2 = yield request( "http://some.url.2" );

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

// usa o utilitário `run(..)` definido anteriormente
run( foo );
```

Esse código vai funcionar, mas, nas especificidades do nosso cenário, ele não é ótimo. Você consegue identificar por quê?

Porque as requisições `r1` e `r2` podem -- e, por razões de desempenho, *deveriam* -- rodar concorrentemente, mas neste código elas vão rodar sequencialmente; a URL `"http://some.url.2"` não é buscada via Ajax até depois que a requisição `"http://some.url.1"` terminar. Essas duas requisições são independentes, então a abordagem de melhor desempenho seria provavelmente tê-las rodando ao mesmo tempo.

Mas como exatamente você faria isso com um gerador e `yield`? Sabemos que `yield` é apenas um único ponto de pausa no código, então você não pode realmente fazer duas pausas ao mesmo tempo.

A resposta mais natural e efetiva é basear o fluxo assíncrono em Promises, especificamente em sua capacidade de gerenciar estado de uma forma independente do tempo (veja "Valor Futuro" no capítulo 3).

A abordagem mais simples:

```js
function *foo() {
	// faz ambas as requisições "em paralelo"
	var p1 = request( "http://some.url.1" );
	var p2 = request( "http://some.url.2" );

	// espera até que ambas as promises se resolvam
	var r1 = yield p1;
	var r2 = yield p2;

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

// usa o utilitário `run(..)` definido anteriormente
run( foo );
```

Por que isso é diferente do trecho anterior? Olhe onde o `yield` está e onde não está. `p1` e `p2` são promises para requisições Ajax feitas concorrentemente (ou seja, "em paralelo"). Não importa qual delas termina primeiro, porque promises vão segurar seu estado resolvido por quanto tempo for necessário.

Então usamos duas instruções `yield` subsequentes para esperar por e recuperar as resoluções das promises (para `r1` e `r2`, respectivamente). Se `p1` se resolver primeiro, o `yield p1` retoma primeiro e então espera no `yield p2` para retomar. Se `p2` se resolver primeiro, ela vai apenas segurar pacientemente esse valor de resolução até ser solicitada, mas o `yield p1` vai segurar primeiro, até `p1` se resolver.

De qualquer forma, tanto `p1` quanto `p2` vão rodar concorrentemente, e ambas têm que terminar, em qualquer ordem, antes que a requisição Ajax `r3 = yield request..` seja feita.

Se esse modelo de processamento de controle de fluxo soa familiar, é basicamente o mesmo que identificamos no capítulo 3 como o padrão "portão" (gate), habilitado pelo utilitário `Promise.all([ .. ])`. Então, poderíamos também expressar o controle de fluxo assim:

```js
function *foo() {
	// faz ambas as requisições "em paralelo," e
	// espera até que ambas as promises se resolvam
	var results = yield Promise.all( [
		request( "http://some.url.1" ),
		request( "http://some.url.2" )
	] );

	var r1 = results[0];
	var r2 = results[1];

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

// usa o utilitário `run(..)` definido anteriormente
run( foo );
```

**Nota:** Como discutimos no capítulo 3, podemos até usar a atribuição via desestruturação (destructuring) do ES6 para simplificar as atribuições `var r1 = .. var r2 = ..`, com `var [r1,r2] = results`.

Em outras palavras, todas as capacidades de concorrência das Promises estão disponíveis para nós na abordagem gerador+Promise. Então, em qualquer lugar onde você precise de mais do que passos sequenciais de controle de fluxo assíncrono este-então-aquele, Promises são provavelmente sua melhor aposta.

#### Promises, Escondidas

Como uma palavra de cautela estilística, tenha cuidado com quanta lógica de Promise você inclui **dentro dos seus geradores**. O objetivo todo de usar geradores para assincronia da forma que descrevemos é criar código simples, sequencial, com aparência síncrona, e esconder o máximo possível dos detalhes de assincronia para longe desse código.

Por exemplo, esta poderia ser uma abordagem mais limpa:

```js
// nota: função normal, não gerador
function bar(url1,url2) {
	return Promise.all( [
		request( url1 ),
		request( url2 )
	] );
}

function *foo() {
	// esconde os detalhes de concorrência baseada em Promise
	// dentro de `bar(..)`
	var results = yield bar(
		"http://some.url.1",
		"http://some.url.2"
	);

	var r1 = results[0];
	var r2 = results[1];

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

// usa o utilitário `run(..)` definido anteriormente
run( foo );
```

Dentro de `*foo()`, está mais limpo e claro que tudo o que estamos fazendo é apenas pedir a `bar(..)` para nos obter alguns `results`, e nós vamos esperar via `yield` que isso aconteça. Nós não temos que nos importar que, por baixo dos panos, uma composição de Promise `Promise.all([ .. ])` será usada para fazer isso acontecer.

**Nós tratamos a assincronia, e de fato as Promises, como um detalhe de implementação.**

Esconder sua lógica de Promise dentro de uma função que você apenas chama do seu gerador é especialmente útil se você vai fazer um controle de fluxo em série sofisticado. Por exemplo:

```js
function bar() {
	Promise.all( [
		baz( .. )
		.then( .. ),
		Promise.race( [ .. ] )
	] )
	.then( .. )
}
```

Esse tipo de lógica às vezes é necessário, e se você o despejar diretamente dentro do(s) seu(s) gerador(es), você derrotou a maior parte da razão pela qual você gostaria de usar geradores em primeiro lugar. Nós *deveríamos* intencionalmente abstrair tais detalhes para longe do nosso código de gerador, de forma que eles não atravanquem a expressão de tarefa de mais alto nível.

Além de criar código que seja tanto funcional quanto performático, você também deveria se esforçar para fazer código que seja o mais sensato (reason-able) e manutenível possível.

**Nota:** Abstração nem *sempre* é uma coisa saudável para programação -- muitas vezes ela pode aumentar a complexidade em troca de concisão. Mas, neste caso, eu acredito que ela é muito mais saudável para o seu código assíncrono gerador+Promise do que as alternativas. Como com todo conselho desse tipo, porém, preste atenção às suas situações específicas e tome as decisões adequadas para você e seu time.

## Delegação de Geradores

Na seção anterior, mostramos como chamar funções comuns de dentro de um gerador, e como isso continua sendo uma técnica útil para abstrair detalhes de implementação (como o fluxo assíncrono de Promise). Mas a principal desvantagem de usar uma função normal para essa tarefa é que ela tem que se comportar pelas regras de função normal, o que significa que ela não pode pausar a si mesma com `yield` como um gerador pode.

Pode então te ocorrer que você poderia tentar chamar um gerador de dentro de outro gerador, usando nosso auxiliar `run(..)`, como:

```js
function *foo() {
	var r2 = yield request( "http://some.url.2" );
	var r3 = yield request( "http://some.url.3/?v=" + r2 );

	return r3;
}

function *bar() {
	var r1 = yield request( "http://some.url.1" );

	// "delegando" para `*foo()` via `run(..)`
	var r3 = yield run( foo );

	console.log( r3 );
}

run( bar );
```

Nós rodamos `*foo()` dentro de `*bar()` usando nosso utilitário `run(..)` novamente. Tiramos proveito aqui do fato de que o `run(..)` que definimos anteriormente retorna uma promise que é resolvida quando seu gerador roda até a conclusão (ou dá erro), então se nós cedermos (`yield`) para fora, para uma instância de `run(..)`, a promise de outra chamada `run(..)`, ela automaticamente pausa `*bar()` até `*foo()` terminar.

Mas há uma forma ainda melhor de integrar a chamada de `*foo()` em `*bar()`, e ela é chamada de delegação de `yield`. A sintaxe especial para delegação de `yield` é: `yield * __` (note o `*` extra). Antes de vermos isso funcionar em nosso exemplo anterior, vamos olhar para um cenário mais simples:

```js
function *foo() {
	console.log( "`*foo()` starting" );
	yield 3;
	yield 4;
	console.log( "`*foo()` finished" );
}

function *bar() {
	yield 1;
	yield 2;
	yield *foo();	// delegação de `yield`!
	yield 5;
}

var it = bar();

it.next().value;	// 1
it.next().value;	// 2
it.next().value;	// `*foo()` starting
					// 3
it.next().value;	// 4
it.next().value;	// `*foo()` finished
					// 5
```

**Nota:** De forma similar a uma nota anterior no capítulo onde expliquei por que prefiro `function *foo() ..` em vez de `function* foo() ..`, eu também prefiro -- diferindo da maioria das outras documentações sobre o tópico -- dizer `yield *foo()` em vez de `yield* foo()`. O posicionamento do `*` é puramente estilístico e fica a seu melhor critério. Mas eu acho a consistência do estilo atraente.

Como funciona a delegação `yield *foo()`?

Primeiro, chamar `foo()` cria um *iterator* exatamente como já vimos. Então, `yield *` delega/transfere o controle da instância do *iterator* (do presente gerador `*bar()`) para esse outro *iterator* de `*foo()`.

Então, as duas primeiras chamadas `it.next()` estão controlando `*bar()`, mas quando fazemos a terceira chamada `it.next()`, agora `*foo()` inicia, e agora estamos controlando `*foo()` em vez de `*bar()`. É por isso que se chama delegação -- `*bar()` delegou seu controle de iteração para `*foo()`.

Assim que o controle do *iterator* `it` esgota inteiramente o *iterator* de `*foo()`, ele automaticamente volta a controlar `*bar()`.

Então, agora de volta ao exemplo anterior com as três requisições Ajax sequenciais:

```js
function *foo() {
	var r2 = yield request( "http://some.url.2" );
	var r3 = yield request( "http://some.url.3/?v=" + r2 );

	return r3;
}

function *bar() {
	var r1 = yield request( "http://some.url.1" );

	// "delegando" para `*foo()` via `yield*`
	var r3 = yield *foo();

	console.log( r3 );
}

run( bar );
```

A única diferença entre este trecho e a versão usada anteriormente é o uso de `yield *foo()` em vez do `yield run(foo)` anterior.

**Nota:** `yield *` cede controle de iteração, não controle de gerador; quando você invoca o gerador `*foo()`, você está agora delegando via `yield` para o *iterator* dele. Mas você pode na verdade delegar via `yield` para qualquer *iterable*; `yield *[1,2,3]` consumiria o *iterator* padrão para o valor de array `[1,2,3]`.

### Por que Delegação?

O propósito da delegação de `yield` é principalmente a organização de código, e nesse sentido ela é simétrica com a chamada normal de função.

Imagine dois módulos que respectivamente fornecem os métodos `foo()` e `bar()`, onde `bar()` chama `foo()`. A razão de os dois serem separados é geralmente porque a organização adequada do código para o programa pede que eles estejam em funções separadas. Por exemplo, pode haver casos onde `foo()` é chamado de forma autônoma, e outros lugares onde `bar()` chama `foo()`.

Por todas essas exatas mesmas razões, manter geradores separados ajuda na legibilidade, manutenção e depurabilidade do programa. Nesse aspecto, `yield *` é um atalho sintático para iterar manualmente sobre os passos de `*foo()` enquanto se está dentro de `*bar()`.

Tal abordagem manual seria especialmente complexa se os passos em `*foo()` fossem assíncronos, e é por isso que você provavelmente precisaria usar aquele utilitário `run(..)` para fazê-lo. E, como mostramos, `yield *foo()` elimina a necessidade de uma subinstância do utilitário `run(..)` (como `run(foo)`).

### Delegando Mensagens

Você pode se perguntar como essa delegação de `yield` funciona não apenas com o controle do *iterator*, mas com a passagem de mensagens em duas vias. Acompanhe cuidadosamente o fluxo de mensagens para dentro e para fora, através da delegação de `yield`:

```js
function *foo() {
	console.log( "inside `*foo()`:", yield "B" );

	console.log( "inside `*foo()`:", yield "C" );

	return "D";
}

function *bar() {
	console.log( "inside `*bar()`:", yield "A" );

	// delegação de `yield`!
	console.log( "inside `*bar()`:", yield *foo() );

	console.log( "inside `*bar()`:", yield "E" );

	return "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A

console.log( "outside:", it.next( 1 ).value );
// inside `*bar()`: 1
// outside: B

console.log( "outside:", it.next( 2 ).value );
// inside `*foo()`: 2
// outside: C

console.log( "outside:", it.next( 3 ).value );
// inside `*foo()`: 3
// inside `*bar()`: D
// outside: E

console.log( "outside:", it.next( 4 ).value );
// inside `*bar()`: 4
// outside: F
```

Preste atenção particular aos passos de processamento depois da chamada `it.next(3)`:

1. O valor `3` é passado (através da delegação de `yield` em `*bar()`) para a expressão `yield "C"` que está esperando dentro de `*foo()`.
2. `*foo()` então chama `return "D"`, mas esse valor não é retornado todo o caminho de volta até a chamada externa `it.next(3)`.
3. Em vez disso, o valor `"D"` é enviado como o resultado da expressão `yield *foo()` que está esperando dentro de `*bar()` -- essa expressão de delegação de `yield` esteve essencialmente pausada enquanto todo o `*foo()` era esgotado. Então `"D"` acaba dentro de `*bar()` para que ele o imprima.
4. `yield "E"` é chamado dentro de `*bar()`, e o valor `"E"` é cedido (`yield`) para fora como o resultado da chamada `it.next(3)`.

Da perspectiva do *iterator* externo (`it`), não parece haver nenhuma diferença entre controlar o gerador inicial ou um delegado.

De fato, a delegação de `yield` nem precisa ser direcionada para outro gerador; ela pode ser direcionada apenas para um *iterable* geral, não gerador. Por exemplo:

```js
function *bar() {
	console.log( "inside `*bar()`:", yield "A" );

	// delegação de `yield` para um não-gerador!
	console.log( "inside `*bar()`:", yield *[ "B", "C", "D" ] );

	console.log( "inside `*bar()`:", yield "E" );

	return "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A

console.log( "outside:", it.next( 1 ).value );
// inside `*bar()`: 1
// outside: B

console.log( "outside:", it.next( 2 ).value );
// outside: C

console.log( "outside:", it.next( 3 ).value );
// outside: D

console.log( "outside:", it.next( 4 ).value );
// inside `*bar()`: undefined
// outside: E

console.log( "outside:", it.next( 5 ).value );
// inside `*bar()`: 5
// outside: F
```

Note as diferenças em onde as mensagens foram recebidas/reportadas entre este exemplo e o anterior.

De forma mais marcante, o *iterator* padrão de `array` não se importa com nenhuma mensagem enviada via chamadas `next(..)`, então os valores `2`, `3` e `4` são essencialmente ignorados. Além disso, como esse *iterator* não tem nenhum valor de `return` explícito (diferentemente do `*foo()` usado anteriormente), a expressão `yield *` recebe um `undefined` quando termina.

#### Exceções Delegadas, Também!

Da mesma forma que a delegação de `yield` passa mensagens de forma transparente em ambas as direções, erros/exceções também passam em ambas as direções:

```js
function *foo() {
	try {
		yield "B";
	}
	catch (err) {
		console.log( "error caught inside `*foo()`:", err );
	}

	yield "C";

	throw "D";
}

function *bar() {
	yield "A";

	try {
		yield *foo();
	}
	catch (err) {
		console.log( "error caught inside `*bar()`:", err );
	}

	yield "E";

	yield *baz();

	// nota: não dá pra chegar aqui!
	yield "G";
}

function *baz() {
	throw "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A

console.log( "outside:", it.next( 1 ).value );
// outside: B

console.log( "outside:", it.throw( 2 ).value );
// error caught inside `*foo()`: 2
// outside: C

console.log( "outside:", it.next( 3 ).value );
// error caught inside `*bar()`: D
// outside: E

try {
	console.log( "outside:", it.next( 4 ).value );
}
catch (err) {
	console.log( "error caught outside:", err );
}
// error caught outside: F
```

Algumas coisas a notar deste trecho:

1. Quando chamamos `it.throw(2)`, ele envia a mensagem de erro `2` para dentro de `*bar()`, que delega isso para `*foo()`, que então `catch` (captura) e trata graciosamente. Então, o `yield "C"` envia `"C"` de volta para fora como o `value` de retorno da chamada `it.throw(2)`.
2. O valor `"D"` que é lançado (`throw`) em seguida de dentro de `*foo()` se propaga para fora, até `*bar()`, que o `catch` (captura) e trata graciosamente. Então o `yield "E"` envia `"E"` de volta para fora como o `value` de retorno da chamada `it.next(3)`.
3. A seguir, a exceção lançada (`throw`) de `*baz()` não é capturada em `*bar()` -- embora tenhamos feito o `catch` dela do lado de fora -- então tanto `*baz()` quanto `*bar()` são definidos para um estado concluído. Depois deste trecho, você não conseguiria obter o valor `"G"` com qualquer chamada `next(..)` subsequente -- elas vão apenas retornar `undefined` para `value`.

### Delegando Assincronia

Vamos finalmente voltar ao nosso exemplo anterior de delegação de `yield` com as múltiplas requisições Ajax sequenciais:

```js
function *foo() {
	var r2 = yield request( "http://some.url.2" );
	var r3 = yield request( "http://some.url.3/?v=" + r2 );

	return r3;
}

function *bar() {
	var r1 = yield request( "http://some.url.1" );

	var r3 = yield *foo();

	console.log( r3 );
}

run( bar );
```

Em vez de chamar `yield run(foo)` dentro de `*bar()`, nós apenas chamamos `yield *foo()`.

Na versão anterior deste exemplo, o mecanismo de Promise (controlado por `run(..)`) foi usado para transportar o valor de `return r3` em `*foo()` para a variável local `r3` dentro de `*bar()`. Agora, esse valor é apenas retornado diretamente via a mecânica do `yield *`.

Fora isso, o comportamento é praticamente idêntico.

### Delegando "Recursão"

Claro, a delegação de `yield` pode continuar seguindo quantos passos de delegação você conectar. Você poderia até usar a delegação de `yield` para "recursão" de geradores com capacidade assíncrona -- um gerador delegando via `yield` para si mesmo:

```js
function *foo(val) {
	if (val > 1) {
		// recursão de gerador
		val = yield *foo( val - 1 );
	}

	return yield request( "http://some.url/?v=" + val );
}

function *bar() {
	var r1 = yield *foo( 3 );
	console.log( r1 );
}

run( bar );
```

**Nota:** Nosso utilitário `run(..)` poderia ter sido chamado com `run( foo, 3 )`, porque ele suporta parâmetros adicionais sendo passados adiante para a inicialização do gerador. No entanto, usamos um `*bar()` sem parâmetros aqui para destacar a flexibilidade do `yield *`.

Quais passos de processamento decorrem desse código? Segura aí, isso vai ser bem intrincado de descrever em detalhe:

1. `run(bar)` inicia o gerador `*bar()`.
2. `foo(3)` cria um *iterator* para `*foo(..)` e passa `3` como seu parâmetro `val`.
3. Porque `3 > 1`, `foo(2)` cria outro *iterator* e passa `2` como seu parâmetro `val`.
4. Porque `2 > 1`, `foo(1)` cria ainda outro *iterator* e passa `1` como seu parâmetro `val`.
5. `1 > 1` é `false`, então a seguir chamamos `request(..)` com o valor `1`, e recebemos de volta uma promise para essa primeira chamada Ajax.
6. Essa promise é cedida (`yield`) para fora, o que volta para a instância do gerador `*foo(2)`.
7. O `yield *` passa essa promise de volta para fora, para a instância do gerador `*foo(3)`. Outro `yield *` passa a promise para fora, para a instância do gerador `*bar()`. E mais uma vez outro `yield *` passa a promise para fora, para o utilitário `run(..)`, que vai esperar nessa promise (da primeira requisição Ajax) para prosseguir.
8. Quando a promise se resolve, sua mensagem de fulfillment é enviada para retomar `*bar()`, que passa através do `yield *` para a instância `*foo(3)`, que então passa através do `yield *` para a instância do gerador `*foo(2)`, que então passa através do `yield *` para o `yield` normal que está esperando na instância do gerador `*foo(3)`.
9. A resposta Ajax daquela primeira chamada é agora imediatamente retornada (`return`) da instância do gerador `*foo(3)`, que envia esse valor de volta como o resultado da expressão `yield *` na instância `*foo(2)`, e atribuído à sua variável local `val`.
10. Dentro de `*foo(2)`, uma segunda requisição Ajax é feita com `request(..)`, cuja promise é cedida (`yield`) de volta para a instância `*foo(1)`, e então `yield *` propaga todo o caminho para fora, até `run(..)` (passo 7 novamente). Quando a promise se resolve, a segunda resposta Ajax propaga todo o caminho de volta para dentro da instância do gerador `*foo(2)`, e é atribuída à sua variável local `val`.
11. Finalmente, a terceira requisição Ajax é feita com `request(..)`, sua promise vai para `run(..)`, e então seu valor de resolução vem todo o caminho de volta, que é então retornado (`return`) de forma que ele volta para a expressão `yield *` que está esperando em `*bar()`.

Ufa! Um monte de malabarismo mental maluco, hein? Você talvez queira ler isso mais algumas vezes, e depois ir pegar um lanche para clarear a cabeça!

## Concorrência de Geradores

Como discutimos tanto no capítulo 1 quanto anteriormente neste capítulo, dois "processos" rodando simultaneamente podem intercalar suas operações cooperativamente, e muitas vezes isso pode *ceder* (yield) (trocadilho intencional) expressões de assincronia bem poderosas.

Francamente, nossos exemplos anteriores de intercalação de concorrência de múltiplos geradores mostraram como deixá-la realmente confusa. Mas insinuamos que há lugares onde essa capacidade é bem útil.

Relembre um cenário que vimos no capítulo 1, onde dois manipuladores de resposta Ajax simultâneos diferentes precisavam coordenar entre si para garantir que a comunicação de dados não fosse uma condição de corrida. Nós encaixamos as respostas no array `res` assim:

```js
function response(data) {
	if (data.url == "http://some.url.1") {
		res[0] = data;
	}
	else if (data.url == "http://some.url.2") {
		res[1] = data;
	}
}
```

Mas como podemos usar múltiplos geradores concorrentemente para este cenário?

```js
// `request(..)` é um utilitário Ajax ciente de Promises

var res = [];

function *reqData(url) {
	res.push(
		yield request( url )
	);
}
```

**Nota:** Nós vamos usar duas instâncias do gerador `*reqData(..)` aqui, mas não há diferença em rodar uma única instância de dois geradores diferentes; ambas as abordagens são raciocinadas de forma idêntica. Veremos dois geradores diferentes coordenando daqui a pouco.

Em vez de ter que organizar manualmente as atribuições de `res[0]` e `res[1]`, usaremos ordenação coordenada para que `res.push(..)` encaixe adequadamente os valores na ordem esperada e previsível. A lógica expressa, portanto, deve parecer um pouco mais limpa.

Mas como vamos de fato orquestrar essa interação? Primeiro, vamos apenas fazê-la manualmente, com Promises:

```js
var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );

var p1 = it1.next().value;
var p2 = it2.next().value;

p1
.then( function(data){
	it1.next( data );
	return p2;
} )
.then( function(data){
	it2.next( data );
} );
```

As duas instâncias de `*reqData(..)` são ambas iniciadas para fazer suas requisições Ajax, e então pausadas com `yield`. Então escolhemos retomar a primeira instância quando `p1` se resolver, e então a resolução de `p2` vai reiniciar a segunda instância. Dessa forma, usamos a orquestração de Promise para garantir que `res[0]` terá a primeira resposta e `res[1]` terá a segunda resposta.

Mas, francamente, isso é terrivelmente manual, e na verdade não deixa os geradores se orquestrarem por si mesmos, que é onde o verdadeiro poder pode residir. Vamos tentar de uma forma diferente:

```js
// `request(..)` é um utilitário Ajax ciente de Promises

var res = [];

function *reqData(url) {
	var data = yield request( url );

	// transfere o controle
	yield;

	res.push( data );
}

var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );

var p1 = it1.next().value;
var p2 = it2.next().value;

p1.then( function(data){
	it1.next( data );
} );

p2.then( function(data){
	it2.next( data );
} );

Promise.all( [p1,p2] )
.then( function(){
	it1.next();
	it2.next();
} );
```

OK, isto está um pouco melhor (embora ainda manual!), porque agora as duas instâncias de `*reqData(..)` rodam verdadeiramente concorrentes, e (ao menos na primeira parte) independentemente.

No trecho anterior, a segunda instância não recebia seus dados até depois que a primeira instância estivesse totalmente terminada. Mas aqui, ambas as instâncias recebem seus dados assim que suas respectivas respostas voltam, e então cada instância faz outro `yield` para fins de transferência de controle. Nós então escolhemos em que ordem retomá-las no manipulador `Promise.all([ .. ])`.

O que pode não ser tão óbvio é que essa abordagem insinua uma forma mais fácil para um utilitário reutilizável, por causa da simetria. Podemos fazer ainda melhor. Vamos imaginar usar um utilitário chamado `runAll(..)`:

```js
// `request(..)` é um utilitário Ajax ciente de Promises

var res = [];

runAll(
	function*(){
		var p1 = request( "http://some.url.1" );

		// transfere o controle
		yield;

		res.push( yield p1 );
	},
	function*(){
		var p2 = request( "http://some.url.2" );

		// transfere o controle
		yield;

		res.push( yield p2 );
	}
);
```

**Nota:** Nós não estamos incluindo uma listagem de código para `runAll(..)`, pois ela não só é longa o bastante para atravancar o texto, mas é uma extensão da lógica que já implementamos em `run(..)` anteriormente. Então, como um bom exercício suplementar para o leitor, tente a sua mão em evoluir o código de `run(..)` para funcionar como o imaginado `runAll(..)`. Além disso, minha biblioteca *asynquence* fornece um utilitário `runner(..)` mencionado anteriormente com esse tipo de capacidade já embutido, e será discutido no Apêndice A deste livro.

Aqui está como o processamento dentro de `runAll(..)` operaria:

1. O primeiro gerador obtém uma promise para a primeira resposta Ajax de `"http://some.url.1"`, então cede (`yield`) o controle de volta para o utilitário `runAll(..)`.
2. O segundo gerador roda e faz o mesmo para `"http://some.url.2"`, cedendo (`yield`) o controle de volta para o utilitário `runAll(..)`.
3. O primeiro gerador retoma, e então cede (`yield`) para fora sua promise `p1`. O utilitário `runAll(..)` faz o mesmo, neste caso, que nosso `run(..)` anterior, em que ele espera nessa promise se resolver, e então retoma o mesmo gerador (sem transferência de controle!). Quando `p1` se resolve, `runAll(..)` retoma o primeiro gerador novamente com esse valor de resolução, e então `res[0]` recebe seu valor. Quando o primeiro gerador então termina, isso é uma transferência implícita de controle.
4. O segundo gerador retoma, cede (`yield`) para fora sua promise `p2`, e espera por ela se resolver. Assim que ela se resolve, `runAll(..)` retoma o segundo gerador com esse valor, e `res[1]` é definido.

Neste exemplo corrente, usamos uma variável externa chamada `res` para armazenar os resultados das duas respostas Ajax diferentes -- essa é a nossa coordenação de concorrência tornando isso possível.

Mas pode ser bastante útil estender ainda mais `runAll(..)` para fornecer um espaço de variável interno para as múltiplas instâncias de gerador *compartilharem*, como um objeto vazio que chamaremos de `data` abaixo. Além disso, ele poderia receber valores não-Promise que são cedidos (`yield`) e entregá-los ao próximo gerador.

Observe:

```js
// `request(..)` é um utilitário Ajax ciente de Promises

runAll(
	function*(data){
		data.res = [];

		// transfere o controle (e passa mensagem)
		var url1 = yield "http://some.url.2";

		var p1 = request( url1 ); // "http://some.url.1"

		// transfere o controle
		yield;

		data.res.push( yield p1 );
	},
	function*(data){
		// transfere o controle (e passa mensagem)
		var url2 = yield "http://some.url.1";

		var p2 = request( url2 ); // "http://some.url.2"

		// transfere o controle
		yield;

		data.res.push( yield p2 );
	}
);
```

Nesta formulação, os dois geradores não estão apenas coordenando a transferência de controle, mas na verdade comunicando-se um com o outro, tanto através de `data.res` quanto das mensagens cedidas (`yield`) que trocam os valores `url1` e `url2`. Isso é incrivelmente poderoso!

Tal percepção também serve como uma base conceitual para uma técnica de assincronia mais sofisticada chamada CSP (Communicating Sequential Processes), que cobriremos no Apêndice B deste livro.

## Thunks

Até aqui, fizemos a suposição de que ceder (`yield`) uma Promise de um gerador -- e fazer essa Promise retomar o gerador via um utilitário auxiliar como `run(..)` -- era a melhor forma possível de gerenciar assincronia com geradores. Para deixar claro, é.

Mas pulamos por cima de outro padrão que tem alguma adoção moderadamente difundida, então, no interesse da completude, daremos uma breve olhada nele.

Na ciência da computação geral, há um conceito antigo, anterior ao JS, chamado de "thunk". Sem se atolar na natureza histórica, uma expressão estrita de um thunk em JS é uma função que -- sem nenhum parâmetro -- está conectada para chamar outra função.

Em outras palavras, você embrulha uma definição de função em torno de uma chamada de função -- com quaisquer parâmetros que ela precise -- para *adiar* a execução dessa chamada, e essa função que embrulha é um thunk. Quando você mais tarde executa o thunk, você acaba chamando a função original.

Por exemplo:

```js
function foo(x,y) {
	return x + y;
}

function fooThunk() {
	return foo( 3, 4 );
}

// mais tarde

console.log( fooThunk() );	// 7
```

Então, um thunk síncrono é bem direto. Mas e quanto a um thunk assíncrono? Podemos essencialmente estender a definição estrita de thunk para incluir o recebimento de um callback.

Observe:

```js
function foo(x,y,cb) {
	setTimeout( function(){
		cb( x + y );
	}, 1000 );
}

function fooThunk(cb) {
	foo( 3, 4, cb );
}

// mais tarde

fooThunk( function(sum){
	console.log( sum );		// 7
} );
```

Como você pode ver, `fooThunk(..)` espera apenas um parâmetro `cb(..)`, pois ele já tem os valores `3` e `4` (para `x` e `y`, respectivamente) pré-especificados e prontos para passar a `foo(..)`. Um thunk está apenas esperando pacientemente pela última peça de que precisa para fazer seu trabalho: o callback.

Você não quer fazer thunks manualmente, porém. Então, vamos inventar um utilitário que faz esse embrulho para nós.

Observe:

```js
function thunkify(fn) {
	var args = [].slice.call( arguments, 1 );
	return function(cb) {
		args.push( cb );
		return fn.apply( null, args );
	};
}

var fooThunk = thunkify( foo, 3, 4 );

// mais tarde

fooThunk( function(sum) {
	console.log( sum );		// 7
} );
```

**Dica:** Aqui assumimos que a assinatura da função original (`foo(..)`) espera seu callback na última posição, com quaisquer outros parâmetros vindo antes dele. Esse é um "padrão" bastante ubíquo para padrões de função assíncrona em JS. Você poderia chamá-lo de "estilo callback-por-último." Se por alguma razão você tivesse a necessidade de lidar com assinaturas de "estilo callback-primeiro," você apenas faria um utilitário que usasse `args.unshift(..)` em vez de `args.push(..)`.

A formulação anterior de `thunkify(..)` recebe tanto a referência da função `foo(..)` quanto quaisquer parâmetros de que ela precise, e retorna o próprio thunk (`fooThunk(..)`). No entanto, essa não é a abordagem típica que você encontrará para thunks em JS.

Em vez de `thunkify(..)` fazer o thunk em si, tipicamente -- se não de forma perplexa -- o utilitário `thunkify(..)` produziria uma função que produz thunks.

Uhhhh... pois é.

Observe:

```js
function thunkify(fn) {
	return function() {
		var args = [].slice.call( arguments );
		return function(cb) {
			args.push( cb );
			return fn.apply( null, args );
		};
	};
}
```

A principal diferença aqui é a camada extra `return function() { .. }`. Aqui está como seu uso difere:

```js
var whatIsThis = thunkify( foo );

var fooThunk = whatIsThis( 3, 4 );

// mais tarde

fooThunk( function(sum) {
	console.log( sum );		// 7
} );
```

Obviamente, a grande questão que este trecho implica é como `whatIsThis` é propriamente chamado? Ele não é o thunk, ele é a coisa que vai produzir thunks a partir de chamadas `foo(..)`. É meio como uma "fábrica" de "thunks." Não parece haver nenhum tipo de acordo padrão para nomear tal coisa.

Então, minha proposta é "thunkory" ("thunk" + "factory"). Então, `thunkify(..)` produz um thunkory, e um thunkory produz thunks. Esse raciocínio é simétrico à minha proposta de "promisory" no capítulo 3:

```js
var fooThunkory = thunkify( foo );

var fooThunk1 = fooThunkory( 3, 4 );
var fooThunk2 = fooThunkory( 5, 6 );

// mais tarde

fooThunk1( function(sum) {
	console.log( sum );		// 7
} );

fooThunk2( function(sum) {
	console.log( sum );		// 11
} );
```

**Nota:** O exemplo corrente `foo(..)` espera um estilo de callback que não é "estilo error-first." Claro, "estilo error-first" é muito mais comum. Se `foo(..)` tivesse algum tipo de expectativa legítima de produzir erro, poderíamos mudá-lo para esperar e usar um callback error-first. Nenhuma da maquinaria subsequente de `thunkify(..)` se importa com qual estilo de callback é assumido. A única diferença no uso seria `fooThunk1(function(err,sum){..`.

Expor o método thunkory -- em vez de como o `thunkify(..)` anterior esconde esse passo intermediário -- pode parecer uma complicação desnecessária. Mas, em geral, é bastante útil fazer thunkories no começo do seu programa para embrulhar métodos de API existentes, e então ser capaz de passar adiante e chamar esses thunkories quando você precisar de thunks. Os dois passos distintos preservam uma separação de capacidade mais limpa.

Para ilustrar:

```js
// mais limpo:
var fooThunkory = thunkify( foo );

var fooThunk1 = fooThunkory( 3, 4 );
var fooThunk2 = fooThunkory( 5, 6 );

// em vez de:
var fooThunk1 = thunkify( foo, 3, 4 );
var fooThunk2 = thunkify( foo, 5, 6 );
```

Independentemente de você gostar de lidar com os thunkories explicitamente ou não, o uso dos thunks `fooThunk1(..)` e `fooThunk2(..)` permanece o mesmo.

### s/promise/thunk/

Então, o que toda essa coisa de thunk tem a ver com geradores?

Comparando thunks com promises de forma geral: eles não são diretamente intercambiáveis, pois não são equivalentes em comportamento. Promises são vastamente mais capazes e confiáveis do que thunks puros.

Mas, em outro sentido, ambos podem ser vistos como uma requisição por um valor, que pode ser assíncrona em sua resposta.

Relembre que, do capítulo 3, definimos um utilitário para "promisificar" uma função, que chamamos de `Promise.wrap(..)` -- poderíamos tê-lo chamado de `promisify(..)`, também! Esse utilitário de embrulho de Promise não produz Promises; ele produz promisories que por sua vez produzem Promises. Isso é completamente simétrico aos thunkories e thunks que estão sendo discutidos no momento.

Para ilustrar a simetria, vamos primeiro alterar o exemplo corrente `foo(..)` de antes para assumir um callback "estilo error-first":

```js
function foo(x,y,cb) {
	setTimeout( function(){
		// assume `cb(..)` como "estilo error-first"
		cb( null, x + y );
	}, 1000 );
}
```

Agora, vamos comparar o uso de `thunkify(..)` e `promisify(..)` (ou seja, `Promise.wrap(..)` do capítulo 3):

```js
// simétrico: construindo o questionador
var fooThunkory = thunkify( foo );
var fooPromisory = promisify( foo );

// simétrico: fazendo a pergunta
var fooThunk = fooThunkory( 3, 4 );
var fooPromise = fooPromisory( 3, 4 );

// obtém a resposta do thunk
fooThunk( function(err,sum){
	if (err) {
		console.error( err );
	}
	else {
		console.log( sum );		// 7
	}
} );

// obtém a resposta da promise
fooPromise
.then(
	function(sum){
		console.log( sum );		// 7
	},
	function(err){
		console.error( err );
	}
);
```

Tanto o thunkory quanto o promisory estão essencialmente fazendo uma pergunta (por um valor), e respectivamente o thunk `fooThunk` e a promise `fooPromise` representam as respostas futuras a essa pergunta. Apresentado sob essa luz, a simetria é clara.

Com essa perspectiva em mente, podemos ver que geradores que cedem (`yield`) Promises para assincronia poderiam em vez disso ceder (`yield`) thunks para assincronia. Tudo de que precisaríamos é de um utilitário `run(..)` mais inteligente (como o de antes) que possa não só procurar e conectar-se a uma Promise cedida (`yield`), mas também fornecer um callback a um thunk cedido (`yield`).

Observe:

```js
function *foo() {
	var val = yield request( "http://some.url.1" );
	console.log( val );
}

run( foo );
```

Neste exemplo, `request(..)` poderia ser tanto um promisory que retorna uma promise, quanto um thunkory que retorna um thunk. Da perspectiva do que está acontecendo dentro da lógica do código do gerador, não nos importamos com esse detalhe de implementação, o que é bem poderoso!

Então, `request(..)` poderia ser tanto:

```js
// promisory `request(..)` (veja o capítulo 3)
var request = Promise.wrap( ajax );

// vs.

// thunkory `request(..)`
var request = thunkify( ajax );
```

Finalmente, como um patch ciente de thunk para o nosso utilitário `run(..)` anterior, precisaríamos de uma lógica assim:

```js
// ..
// recebemos um thunk de volta?
else if (typeof next.value == "function") {
	return new Promise( function(resolve,reject){
		// chama o thunk com um callback error-first
		next.value( function(err,msg) {
			if (err) {
				reject( err );
			}
			else {
				resolve( msg );
			}
		} );
	} )
	.then(
		handleNext,
		function handleErr(err) {
			return Promise.resolve(
				it.throw( err )
			)
			.then( handleResult );
		}
	);
}
```

Agora, nossos geradores podem ou chamar promisories para ceder (`yield`) Promises, ou chamar thunkories para ceder (`yield`) thunks, e, em qualquer caso, `run(..)` lidaria com esse valor e o usaria para esperar pela conclusão para retomar o gerador.

Em termos de simetria, essas duas abordagens parecem idênticas. No entanto, devemos apontar que isso é verdade apenas da perspectiva de Promises ou thunks representando a continuação de valor futuro de um gerador.

Da perspectiva mais ampla, thunks não têm em si e por si mesmos quase nenhuma das garantias de confiabilidade ou composibilidade com as quais Promises são projetadas. Usar um thunk como substituto para uma Promise neste padrão particular de assincronia de gerador é viável, mas deveria ser visto como menos do que ideal quando comparado a todos os benefícios que Promises oferecem (veja o capítulo 3).

Se você tiver a opção, prefira `yield pr` em vez de `yield th`. Mas não há nada de errado em ter um utilitário `run(..)` que possa lidar com ambos os tipos de valor.

**Nota:** O utilitário `runner(..)` em minha biblioteca *asynquence*, que será discutido no Apêndice A, lida com `yield`s de Promises, thunks e sequências *asynquence*.

## Geradores Pré-ES6

Você está esperançosamente convencido agora de que geradores são uma adição muito importante à caixa de ferramentas de programação assíncrona. Mas é uma nova sintaxe no ES6, o que significa que você não pode simplesmente fazer polyfill de geradores como você pode com Promises (que são apenas uma nova API). Então, o que podemos fazer para trazer geradores para o nosso JS de navegador se não temos o luxo de ignorar navegadores pré-ES6?

Para todas as novas extensões de sintaxe no ES6, há ferramentas -- o termo mais comum para elas é transpiladores, de trans-compiladores -- que podem pegar sua sintaxe ES6 e transformá-la em código pré-ES6 equivalente (mas obviamente mais feio!). Então, geradores podem ser transpilados em código que terá o mesmo comportamento, mas funcionará no ES5 e abaixo.

Mas como? A "mágica" do `yield` não parece obviamente um código fácil de transpilar. Nós na verdade insinuamos uma solução em nossa discussão anterior sobre *iterators* baseados em closure.

### Transformação Manual

Antes de discutirmos os transpiladores, vamos derivar como a transpilação manual funcionaria no caso de geradores. Isso não é apenas um exercício acadêmico, porque fazer isso na verdade vai ajudar a reforçar ainda mais como eles funcionam.

Observe:

```js
// `request(..)` é um utilitário Ajax ciente de Promises

function *foo(url) {
	try {
		console.log( "requesting:", url );
		var val = yield request( url );
		console.log( val );
	}
	catch (err) {
		console.log( "Oops:", err );
		return false;
	}
}

var it = foo( "http://some.url.1" );
```

A primeira coisa a observar é que ainda precisaremos de uma função `foo()` normal que possa ser chamada, e ela ainda precisará retornar um *iterator*. Então, vamos esboçar a transformação não-geradora:

```js
function foo(url) {

	// ..

	// faz e retorna um iterator
	return {
		next: function(v) {
			// ..
		},
		throw: function(e) {
			// ..
		}
	};
}

var it = foo( "http://some.url.1" );
```

A próxima coisa a observar é que um gerador faz sua "mágica" suspendendo seu escopo/estado, mas podemos emular isso com closure de função (veja o título *Escopos & Closures* desta série). Para entender como escrever tal código, vamos primeiro anotar diferentes partes do nosso gerador com valores de estado:

```js
// `request(..)` é um utilitário Ajax ciente de Promises

function *foo(url) {
	// ESTADO *1*

	try {
		console.log( "requesting:", url );
		var TMP1 = request( url );

		// ESTADO *2*
		var val = yield TMP1;
		console.log( val );
	}
	catch (err) {
		// ESTADO *3*
		console.log( "Oops:", err );
		return false;
	}
}
```

**Nota:** Para uma ilustração mais precisa, dividimos a instrução `val = yield request..` em duas partes, usando a variável temporária `TMP1`. `request(..)` acontece no estado `*1*`, e a atribuição de seu valor de conclusão a `val` acontece no estado `*2*`. Nós nos livraremos desse `TMP1` intermediário quando convertermos o código para seu equivalente não-gerador.

Em outras palavras, `*1*` é o estado inicial, `*2*` é o estado se o `request(..)` for bem-sucedido, e `*3*` é o estado se o `request(..)` falhar. Você provavelmente consegue imaginar como quaisquer passos `yield` extras seriam apenas codificados como estados extras.

De volta ao nosso gerador transpilado, vamos definir uma variável `state` no closure que podemos usar para acompanhar o estado:

```js
function foo(url) {
	// gerencia o estado do gerador
	var state;

	// ..
}
```

Agora, vamos definir uma função interna chamada `process(..)` dentro do closure, que lida com cada estado, usando uma instrução `switch`:

```js
// `request(..)` é um utilitário Ajax ciente de Promises

function foo(url) {
	// gerencia o estado do gerador
	var state;

	// declarações de variáveis de todo o gerador
	var val;

	function process(v) {
		switch (state) {
			case 1:
				console.log( "requesting:", url );
				return request( url );
			case 2:
				val = v;
				console.log( val );
				return;
			case 3:
				var err = v;
				console.log( "Oops:", err );
				return false;
		}
	}

	// ..
}
```

Cada estado em nosso gerador é representado por seu próprio `case` na instrução `switch`. `process(..)` será chamado cada vez que precisarmos processar um novo estado. Voltaremos a como isso funciona em apenas um momento.

Para quaisquer declarações de variáveis de todo o gerador (`val`), nós movemos essas para uma declaração `var` fora de `process(..)` para que elas possam sobreviver a múltiplas chamadas a `process(..)`. Mas a variável `err` com "escopo de bloco" só é necessária para o estado `*3*`, então a deixamos no lugar.

No estado `*1*`, em vez de `yield request(..)`, fizemos `return request(..)`. No estado terminal `*2*`, não havia `return` explícito, então apenas fazemos um `return;` que é o mesmo que `return undefined`. No estado terminal `*3*`, havia um `return false`, então preservamos isso.

Agora precisamos definir o código nas funções do *iterator* para que elas chamem `process(..)` apropriadamente:

```js
function foo(url) {
	// gerencia o estado do gerador
	var state;

	// declarações de variáveis de todo o gerador
	var val;

	function process(v) {
		switch (state) {
			case 1:
				console.log( "requesting:", url );
				return request( url );
			case 2:
				val = v;
				console.log( val );
				return;
			case 3:
				var err = v;
				console.log( "Oops:", err );
				return false;
		}
	}

	// faz e retorna um iterator
	return {
		next: function(v) {
			// estado inicial
			if (!state) {
				state = 1;
				return {
					done: false,
					value: process()
				};
			}
			// yield retomado com sucesso
			else if (state == 1) {
				state = 2;
				return {
					done: true,
					value: process( v )
				};
			}
			// gerador já concluído
			else {
				return {
					done: true,
					value: undefined
				};
			}
		},
		"throw": function(e) {
			// o único tratamento de erro explícito está no
			// estado *1*
			if (state == 1) {
				state = 3;
				return {
					done: true,
					value: process( e )
				};
			}
			// caso contrário, um erro não será tratado,
			// então apenas o lança de volta para fora
			else {
				throw e;
			}
		}
	};
}
```

Como esse código funciona?

1. A primeira chamada ao `next()` do *iterator* moveria o gerador do estado não inicializado para o estado `1`, e então chamaria `process()` para lidar com esse estado. O valor de retorno de `request(..)`, que é a promise para a resposta Ajax, é retornado de volta como a propriedade `value` da chamada `next()`.
2. Se a requisição Ajax for bem-sucedida, a segunda chamada a `next(..)` deve enviar o valor de resposta Ajax, o que move nosso estado para `2`. `process(..)` é novamente chamado (desta vez com o valor de resposta Ajax passado), e a propriedade `value` retornada de `next(..)` será `undefined`.
3. No entanto, se a requisição Ajax falhar, `throw(..)` deve ser chamado com o erro, o que moveria o estado de `1` para `3` (em vez de `2`). Novamente `process(..)` é chamado, desta vez com o valor de erro. Esse `case` retorna `false`, que é definido como a propriedade `value` retornada da chamada `throw(..)`.

De fora -- isto é, interagindo apenas com o *iterator* -- essa função normal `foo(..)` funciona praticamente da mesma forma que o gerador `*foo(..)` teria funcionado. Então, efetivamente "transpilamos" nosso gerador ES6 para compatibilidade pré-ES6!

Poderíamos então instanciar manualmente nosso gerador e controlar seu iterator -- chamando `var it = foo("..")` e `it.next(..)` e coisas assim -- ou, melhor, poderíamos passá-lo para o nosso utilitário `run(..)` definido anteriormente como `run(foo,"..")`.

### Transpilação Automática

O exercício anterior de derivar manualmente uma transformação do nosso gerador ES6 para o equivalente pré-ES6 nos ensina como geradores funcionam conceitualmente. Mas essa transformação foi realmente intrincada e muito pouco portável para outros geradores no nosso código. Seria bem impraticável fazer esse trabalho à mão, e obviaria completamente todo o benefício dos geradores.

Mas, felizmente, várias ferramentas já existem que podem converter automaticamente geradores ES6 em coisas como o que derivamos na seção anterior. Não só elas fazem o trabalho pesado por nós, mas elas também lidam com várias complicações que passamos por cima.

Uma dessas ferramentas é o regenerator (https://facebook.github.io/regenerator/), do pessoal inteligente do Facebook.

Se usarmos o regenerator para transpilar nosso gerador anterior, aqui está o código produzido (no momento em que isto é escrito):

```js
// `request(..)` é um utilitário Ajax ciente de Promises

var foo = regeneratorRuntime.mark(function foo(url) {
    var val;

    return regeneratorRuntime.wrap(function foo$(context$1$0) {
        while (1) switch (context$1$0.prev = context$1$0.next) {
        case 0:
            context$1$0.prev = 0;
            console.log( "requesting:", url );
            context$1$0.next = 4;
            return request( url );
        case 4:
            val = context$1$0.sent;
            console.log( val );
            context$1$0.next = 12;
            break;
        case 8:
            context$1$0.prev = 8;
            context$1$0.t0 = context$1$0.catch(0);
            console.log("Oops:", context$1$0.t0);
            return context$1$0.abrupt("return", false);
        case 12:
        case "end":
            return context$1$0.stop();
        }
    }, foo, this, [[0, 8]]);
});
```

Há algumas semelhanças óbvias aqui com nossa derivação manual, como as instruções `switch` / `case`, e até vemos `val` puxado para fora do closure exatamente como fizemos.

Claro, uma desvantagem é que a transpilação do regenerator requer uma biblioteca auxiliar `regeneratorRuntime` que contém toda a lógica reutilizável para gerenciar um gerador / *iterator* geral. Bastante desse boilerplate parece diferente da nossa versão, mas, mesmo assim, os conceitos podem ser vistos, como com `context$1$0.next = 4` acompanhando o próximo estado para o gerador.

A principal lição é que geradores não estão restritos a serem úteis apenas em ambientes ES6+. Uma vez que você entende os conceitos, você pode empregá-los por todo o seu código, e usar ferramentas para transformar o código para ser compatível com ambientes mais antigos.

Isso é mais trabalho do que apenas usar um polyfill de API `Promise` para Promises pré-ES6, mas o esforço vale totalmente a pena, porque geradores são muito melhores em expressar controle de fluxo assíncrono de uma forma sensata, sensível, com aparência síncrona e sequencial.

Uma vez que você fica viciado em geradores, você nunca vai querer voltar para o inferno do espaguete de callbacks assíncronos!

## Revisão

Geradores são um novo tipo de função do ES6 que não roda até acabar como funções normais. Em vez disso, o gerador pode ser pausado no meio da conclusão (preservando inteiramente seu estado), e ele pode mais tarde ser retomado de onde parou.

Essa troca de pausa/retomada é cooperativa em vez de preemptiva, o que significa que o gerador tem a única capacidade de pausar a si mesmo, usando a palavra-chave `yield`, e ainda assim o *iterator* que controla o gerador tem a única capacidade (via `next(..)`) de retomar o gerador.

A dualidade `yield` / `next(..)` não é apenas um mecanismo de controle, ela é na verdade um mecanismo de passagem de mensagens em duas vias. Uma expressão `yield ..` essencialmente pausa esperando por um valor, e a próxima chamada `next(..)` passa um valor (ou `undefined` implícito) de volta para essa expressão `yield` pausada.

O principal benefício dos geradores relacionado ao controle de fluxo assíncrono é que o código dentro de um gerador expressa uma sequência de passos para a tarefa de uma forma naturalmente síncrona/sequencial. O truque é que essencialmente escondemos a potencial assincronia atrás da palavra-chave `yield` -- movendo a assincronia para o código onde o *iterator* do gerador é controlado.

Em outras palavras, geradores preservam um padrão de código sequencial, síncrono e bloqueante para código assíncrono, o que deixa nossos cérebros raciocinarem sobre o código muito mais naturalmente, abordando uma das duas principais desvantagens da assincronia baseada em callbacks.
