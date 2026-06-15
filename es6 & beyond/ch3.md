# You Don't Know JS: ES6 & Além
# Capítulo 3: Organização

Uma coisa é escrever código JS, outra coisa é organizá-lo propriamente. Utilizar padrões comuns para organização e reuso é muito bom para melhorar a legibilidade e entendimento do seu código. Lembre-se: código é tanto comunicação com outros desenvolvedores quanto fornecer instruções de computador.

ES6 tem várias funcionalidades importantes que ajudam significantemente a melhorar esses padrões, incluindo: iteradores, geradores, módulos e classes.

## Iteradores

Um *iterador* é um padrão estruturado para obter informações de uma fonte, de uma a uma. Esse padrão tem estado pela programação há muito tempo. E para ser exato, desenvolvedores JS têm pensado e implementado iteradores em programas JS desde antes que alguém possa se lembrar, então isso não é uma novidade.

O que ES6 tem feito é introduzir uma interface padronizada implícita para iteradores. Muitas das estruturas de dados embutidas em JavaScript vão agora expor um iterador implementando esse padrão. E você também pode construir seus próprios iteradores aderindo ao mesmo padrão, para máxima interoperabilidade.

Iteradores são uma maneira de organizar o consumo ordenado, sequencial e baseado em obtenção de dados.

Por exemplo, você pode implementar um utilitário que produz um novo identificador único a cada vez que é requisitado. Ou você pode produzir uma série infinita de valores que alternam através de uma lista fixa, pelo método de round-robin. Ou você pode anexar um iterador a uma consulta de banco de dados para obter novas linhas, uma por vez.

Embora eles não sejam geralmente usados em JS de tal forma, iteradores também podem ser considerados como comportamentos de controle de etapas, uma de cada vez. Isso pode ser ilustrado de forma mais clara quando pensamos em geradores (veja "Geradores" mais à frente nesse capítulo), certamente você possa fazer a mesma coisa sem geradores.

### Interfaces

Na época em que isto está sendo escrito, a seção ES6 25.1.1.2 (https://people.mozilla.org/~jorendorff/es6-draft.html#sec-iterator-interface) detalha a interface `Iterator` como tendo os seguintes requerimentos:

```
Iterator [necessário]
	next() {método}: recupera o próximo Resultado do Iterator
```

Tem dois membros opcionais com alguns iterators que são estendidos juntos:

```
Iterator [opcional]
	return() {método}: para o iterator e retorna o Resultado do Iterator
	throw() {método}: sinaliza o erro e retorna o Resultado do Iterator
```

A interface do `Resultado do Iterator` é especificada como:

```
IteratorResult
	value {propriedade}: valor atual da iteração ou retorno final
		(opcional se estiver `undefined`)
	done {propriedade}: booleano, indica que o status está completo
```

**Nota:** Chamo estas interfaces implícitas não porque não estejam explicitamente chamadas na especificação -- estão! -- mas porque não estão expostas como objetos de acesso direto no código. JavaScript na versão ES6 não suporta qualquer noção de “interfaces”, então a aderência ao seu próprio código é puramente convencional. Porém, onde quer que o JS espere um iterador -- um loop `for..of`, por exemplo -- o que você prover deve aderir a estas interfaces ou falhará.

Existe também uma interface `Iterable`, que descreve objetos que são capazes de produzir iteradores:

```
Iterable
	@@iterador() {método}: produz um iterador
```

Se você se lembra do Capítulo 2 "Built-In Symbols", `@@iterator` é o símbolo nativo especial do método que pode produzir iterador(es) para o objeto.

#### IteratorResult

A interface `IteratorResult` especifica que o valor de retorno de qualquer operação de iterador será um objeto da forma:

```js
{ value: .. , done: true / false }
```

Iteradores embutidos sempre retornarão valores nessa forma, mas mais propriedades são, é claro, permitidas de estarem presentes no valor de retorno, conforme necessário.

Por exemplo, um iterador customizado pode adicionar metadados adicionais ao objeto de resultado (por exemplo, de onde os dados vieram, quanto tempo levou para recuperá-los, tempo de expiração do cache, frequência para a próxima requisição apropriada, etc.).

**Nota:** Tecnicamente, `value` é opcional se ele de outra forma fosse considerado ausente ou não definido, como no caso do valor `undefined`. Como acessar `res.value` produzirá `undefined` quer ele esteja presente com esse valor quer esteja inteiramente ausente, a presença/ausência da propriedade é mais um detalhe de implementação ou uma otimização (ou ambos), do que uma questão funcional.

### Iteração com `next()`

Vamos olhar para um array, que é um iterável, e o iterador que ele pode produzir para consumir seus valores:

```js
var arr = [1,2,3];

var it = arr[Symbol.iterator]();

it.next();		// { value: 1, done: false }
it.next();		// { value: 2, done: false }
it.next();		// { value: 3, done: false }

it.next();		// { value: undefined, done: true }
```

Cada vez que o método localizado em `Symbol.iterator` (veja os Capítulos 2 e 7) é invocado nesse valor `arr`, ele produzirá um novo iterador fresco. A maioria das estruturas fará o mesmo, incluindo todas as estruturas de dados embutidas em JS.

Entretanto, uma estrutura como um consumidor de fila de eventos pode produzir apenas um único iterador (padrão singleton). Ou uma estrutura pode permitir apenas um iterador único por vez, exigindo que o atual seja concluído antes que um novo possa ser criado.

O iterador `it` no fragmento anterior não reporta `done: true` quando você recebe o valor `3`. Você precisa chamar `next()` novamente, em essência indo além do fim dos valores do array, para obter o sinal de conclusão `done: true`. Pode não ficar claro por que até mais adiante nesta seção, mas essa decisão de design tipicamente será considerada uma boa prática.

Valores primitivos de string também são iteráveis por padrão:

```js
var greeting = "hello world";

var it = greeting[Symbol.iterator]();

it.next();		// { value: "h", done: false }
it.next();		// { value: "e", done: false }
..
```

**Nota:** Tecnicamente, o próprio valor primitivo não é iterável, mas graças ao "boxing", `"hello world"` é coagido/convertido para sua forma de objeto wrapper `String`, que *é* um iterável. Veja o título *Types & Grammar* desta série para mais informações.

ES6 também inclui várias novas estruturas de dados, chamadas coleções (veja o Capítulo 5). Essas coleções não são apenas iteráveis em si, mas também fornecem método(s) de API para gerar um iterador, como:

```js
var m = new Map();
m.set( "foo", 42 );
m.set( { cool: true }, "hello world" );

var it1 = m[Symbol.iterator]();
var it2 = m.entries();

it1.next();		// { value: [ "foo", 42 ], done: false }
it2.next();		// { value: [ "foo", 42 ], done: false }
..
```

O método `next(..)` de um iterador pode opcionalmente receber um ou mais argumentos. Os iteradores embutidos em sua maioria não exercem essa capacidade, embora o iterador de um gerador definitivamente o faça (veja "Geradores" mais adiante neste capítulo).

Por convenção geral, incluindo todos os iteradores embutidos, chamar `next(..)` em um iterador que já foi exaurido não é um erro, mas simplesmente continuará a retornar o resultado `{ value: undefined, done: true }`.

### Opcional: `return(..)` e `throw(..)`

Os métodos opcionais na interface do iterador -- `return(..)` e `throw(..)` -- não são implementados na maioria dos iteradores embutidos. Entretanto, eles definitivamente significam algo no contexto de geradores, então veja "Geradores" para informações mais específicas.

`return(..)` é definido como enviar um sinal a um iterador de que o código consumidor está completo e não puxará mais valores dele. Esse sinal pode ser usado para notificar o produtor (o iterador respondendo às chamadas de `next(..)`) a realizar qualquer limpeza que precise fazer, como liberar/fechar recursos de rede, banco de dados ou manipuladores de arquivo.

Se um iterador tem um `return(..)` presente e qualquer condição ocorre que pode ser automaticamente interpretada como término anormal ou antecipado do consumo do iterador, `return(..)` será chamado automaticamente. Você também pode chamar `return(..)` manualmente.

`return(..)` retornará um objeto `IteratorResult` assim como `next(..)` faz. Em geral, o valor opcional que você envia para `return(..)` seria enviado de volta como `value` nesse `IteratorResult`, embora existam casos sutis em que isso possa não ser verdade.

`throw(..)` é usado para sinalizar uma exceção/erro a um iterador, que possivelmente pode ser usado de forma diferente pelo iterador do que o sinal de conclusão implícito por `return(..)`. Ele não necessariamente implica uma parada completa do iterador como `return(..)` geralmente faz.

Por exemplo, com iteradores de geradores, `throw(..)` na verdade injeta uma exceção lançada no contexto de execução pausado do gerador, que pode ser capturada com um `try..catch`. Uma exceção `throw(..)` não capturada acabaria abortando anormalmente o iterador do gerador.

**Nota:** Por convenção geral, um iterador não deve produzir mais resultados após ter chamado `return(..)` ou `throw(..)`.

### Loop de Iterador

Como cobrimos na seção "`for..of`" no Capítulo 2, o loop `for..of` do ES6 consome diretamente um iterável compatível.

Se um iterador também é um iterável, ele pode ser usado diretamente com o loop `for..of`. Você torna um iterador um iterável dando a ele um método `Symbol.iterator` que simplesmente retorna o próprio iterador:

```js
var it = {
	// torna o iterador `it` um iterável
	[Symbol.iterator]() { return this; },

	next() { .. },
	..
};

it[Symbol.iterator]() === it;		// true
```

Agora podemos consumir o iterador `it` com um loop `for..of`:

```js
for (var v of it) {
	console.log( v );
}
```

Para entender totalmente como tal loop funciona, lembre-se do equivalente `for` de um loop `for..of` do Capítulo 2:

```js
for (var v, res; (res = it.next()) && !res.done; ) {
	v = res.value;
	console.log( v );
}
```

Se você olhar de perto, verá que `it.next()` é chamado antes de cada iteração, e então `res.done` é consultado. Se `res.done` for `true`, a expressão avalia para `false` e a iteração não ocorre.

Lembre-se de que anteriormente sugerimos que iteradores em geral não deveriam retornar `done: true` junto com o valor final pretendido do iterador. Agora você pode ver por quê.

Se um iterador retornasse `{ done: true, value: 42 }`, o loop `for..of` descartaria completamente o valor `42` e ele seria perdido. Por essa razão, assumindo que seu iterador possa ser consumido por padrões como o loop `for..of` ou seu equivalente manual `for`, você provavelmente deveria esperar para retornar `done: true` para sinalizar conclusão até depois de já ter retornado todos os valores de iteração relevantes.

**Aviso:** Você pode, é claro, intencionalmente projetar seu iterador para retornar algum `value` relevante ao mesmo tempo em que retorna `done: true`. Mas não faça isso a menos que tenha documentado que esse é o caso, e assim implicitamente forçado os consumidores do seu iterador a usar um padrão diferente para iteração do que o implícito por `for..of` ou seu equivalente manual que descrevemos.

### Iteradores Customizados

Além dos iteradores embutidos padrão, você pode fazer os seus próprios! Tudo que é necessário para fazê-los interoperar com as facilidades de consumo do ES6 (por exemplo, o loop `for..of` e o operador `...`) é aderir à(s) interface(s) apropriada(s).

Vamos tentar construir um iterador que produz a série infinita de números na sequência de Fibonacci:

```js
var Fib = {
	[Symbol.iterator]() {
		var n1 = 1, n2 = 1;

		return {
			// torna o iterador um iterável
			[Symbol.iterator]() { return this; },

			next() {
				var current = n2;
				n2 = n1;
				n1 = n1 + current;
				return { value: current, done: false };
			},

			return(v) {
				console.log(
					"Fibonacci sequence abandoned."
				);
				return { value: v, done: true };
			}
		};
	}
};

for (var v of Fib) {
	console.log( v );

	if (v > 50) break;
}
// 1 1 2 3 5 8 13 21 34 55
// Fibonacci sequence abandoned.
```

**Aviso:** Se não tivéssemos inserido a condição `break`, esse loop `for..of` teria rodado para sempre, o que provavelmente não é o resultado desejado em termos de quebrar seu programa!

O método `Fib[Symbol.iterator]()`, quando chamado, retorna o objeto iterador com os métodos `next()` e `return(..)` nele. O estado é mantido através das variáveis `n1` e `n2`, que são mantidas pelo closure.

Vamos *a seguir* considerar um iterador que é projetado para percorrer uma série (também conhecida como fila) de ações, um item por vez:

```js
var tasks = {
	[Symbol.iterator]() {
		var steps = this.actions.slice();

		return {
			// torna o iterador um iterável
			[Symbol.iterator]() { return this; },

			next(...args) {
				if (steps.length > 0) {
					let res = steps.shift()( ...args );
					return { value: res, done: false };
				}
				else {
					return { done: true }
				}
			},

			return(v) {
				steps.length = 0;
				return { value: v, done: true };
			}
		};
	},
	actions: []
};
```

O iterador em `tasks` percorre as funções encontradas na propriedade do array `actions`, se houver, e as executa uma por vez, passando quaisquer argumentos que você passar para `next(..)`, e retornando qualquer valor de retorno para você no objeto `IteratorResult` padrão.

Veja como poderíamos usar esta fila `tasks`:

```js
tasks.actions.push(
	function step1(x){
		console.log( "step 1:", x );
		return x * 2;
	},
	function step2(x,y){
		console.log( "step 2:", x, y );
		return x + (y * 2);
	},
	function step3(x,y,z){
		console.log( "step 3:", x, y, z );
		return (x * y) + z;
	}
);

var it = tasks[Symbol.iterator]();

it.next( 10 );			// step 1: 10
						// { value:   20, done: false }

it.next( 20, 50 );		// step 2: 20 50
						// { value:  120, done: false }

it.next( 20, 50, 120 );	// step 3: 20 50 120
						// { value: 1120, done: false }

it.next();				// { done: true }
```

Esse uso específico reforça que iteradores podem ser um padrão para organizar funcionalidade, não apenas dados. Também é uma reminiscência do que veremos com geradores na próxima seção.

Você poderia até ser criativo e definir um iterador que representa meta operações sobre um único pedaço de dado. Por exemplo, poderíamos definir um iterador para números que por padrão varia de `0` até (ou para baixo até, para números negativos) o número em questão.

Considere:

```js
if (!Number.prototype[Symbol.iterator]) {
	Object.defineProperty(
		Number.prototype,
		Symbol.iterator,
		{
			writable: true,
			configurable: true,
			enumerable: false,
			value: function iterator(){
				var i, inc, done = false, top = +this;

				// iterar positivamente ou negativamente?
				inc = 1 * (top < 0 ? -1 : 1);

				return {
					// torna o próprio iterador um iterável!
					[Symbol.iterator](){ return this; },

					next() {
						if (!done) {
							// iteração inicial sempre 0
							if (i == null) {
								i = 0;
							}
							// iterando positivamente
							else if (top >= 0) {
								i = Math.min(top,i + inc);
							}
							// iterando negativamente
							else {
								i = Math.max(top,i + inc);
							}

							// concluído após esta iteração?
							if (i == top) done = true;

							return { value: i, done: false };
						}
						else {
							return { done: true };
						}
					}
				};
			}
		}
	);
}
```

Agora, que truques essa criatividade nos proporciona?

```js
for (var i of 3) {
	console.log( i );
}
// 0 1 2 3

[...-3];				// [0,-1,-2,-3]
```

Esses são alguns truques divertidos, embora a utilidade prática seja um tanto discutível. Mas, novamente, alguém poderia se perguntar por que o ES6 simplesmente não veio com um easter egg de funcionalidade tão pequeno mas encantador!?

Eu seria negligente se não pelo menos lembrasse você de que estender protótipos nativos como estou fazendo no fragmento anterior é algo que você só deveria fazer com cautela e consciência dos riscos potenciais.

Nesse caso, as chances de você ter uma colisão com outro código ou até mesmo uma funcionalidade futura do JS são provavelmente extremamente baixas. Mas apenas tome cuidado com a leve possibilidade. E documente o que você está fazendo verbosamente para o bem da posteridade.

**Nota:** Eu expus essa técnica específica neste post de blog (http://blog.getify.com/iterating-es6-numbers/) se você quiser mais detalhes. E este comentário (http://blog.getify.com/iterating-es6-numbers/comment-page-1/#comment-535294) até sugere um truque similar mas para fazer intervalos de caracteres de string.

### Consumo de Iterador

Já mostramos consumir um iterador item por item com o loop `for..of`. Mas existem outras estruturas do ES6 que podem consumir iteradores.

Vamos considerar o iterador anexado a este array (embora qualquer iterador que escolhêssemos teria os seguintes comportamentos):

```js
var a = [1,2,3,4,5];
```

O operador spread `...` exaure completamente um iterador. Considere:

```js
function foo(x,y,z,w,p) {
	console.log( x + y + z + w + p );
}

foo( ...a );			// 15
```

`...` também pode espalhar um iterador dentro de um array:

```js
var b = [ 0, ...a, 6 ];
b;						// [0,1,2,3,4,5,6]
```

A desestruturação de array (veja "Desestruturação" no Capítulo 2) pode consumir parcial ou completamente (se combinada com um operador rest/gather `...`) um iterador:

```js
var it = a[Symbol.iterator]();

var [x,y] = it;			// pega apenas os dois primeiros elementos de `it`
var [z, ...w] = it;		// pega o terceiro, depois o resto de uma só vez

// `it` está totalmente exaurido? Sim.
it.next();				// { value: undefined, done: true }

x;						// 1
y;						// 2
z;						// 3
w;						// [4,5]
```

## Geradores

Todas as funções rodam até a conclusão, certo? Em outras palavras, uma vez que uma função começa a rodar, ela termina antes que qualquer outra coisa possa interrompê-la.

Pelo menos foi assim por toda a história do JavaScript até este ponto. A partir do ES6, uma nova forma um tanto exótica de função está sendo introduzida, chamada de gerador. Um gerador pode pausar a si mesmo no meio da execução, e pode ser retomado tanto imediatamente quanto em um momento posterior. Então ele claramente não mantém a garantia de rodar-até-a-conclusão que as funções normais têm.

Além disso, cada ciclo de pausa/retomada no meio da execução é uma oportunidade para passagem de mensagens nos dois sentidos, onde o gerador pode retornar um valor, e o código de controle que o retoma pode enviar um valor de volta.

Assim como com iteradores na seção anterior, há múltiplas maneiras de pensar sobre o que um gerador é, ou melhor, para o que ele é mais útil. Não há uma resposta única correta, mas tentaremos considerar vários ângulos.

**Nota:** Veja o título *Async & Performance* desta série para mais informações sobre geradores, e veja também o Capítulo 4 deste título atual.

### Sintaxe

A função geradora é declarada com esta nova sintaxe:

```js
function *foo() {
	// ..
}
```

A posição do `*` não é funcionalmente relevante. A mesma declaração poderia ser escrita como qualquer uma das seguintes:

```js
function *foo()  { .. }
function* foo()  { .. }
function * foo() { .. }
function*foo()   { .. }
..
```

A *única* diferença aqui é preferência estilística. A maioria das outras literaturas parece preferir `function* foo(..) { .. }`. Eu prefiro `function *foo(..) { .. }`, então é assim que vou apresentá-los pelo resto deste título.

Minha razão é puramente de natureza didática. Neste texto, ao me referir a uma função geradora, usarei `*foo(..)`, em oposição a `foo(..)` para uma função normal. Observo que `*foo(..)` se aproxima mais do posicionamento do `*` em `function *foo(..) { .. }`.

Além disso, como vimos no Capítulo 2 com os concise methods, existe uma forma de gerador concisa em literais de objeto:

```js
var a = {
	*foo() { .. }
};
```

Eu diria que, com geradores concisos, `*foo() { .. }` é bem mais natural do que `* foo() { .. }`. Então isso reforça ainda mais o argumento a favor de manter a consistência com `*foo()`.

A consistência facilita o entendimento e o aprendizado.

#### Executando um Gerador

Embora um gerador seja declarado com `*`, você ainda o executa como uma função normal:

```js
foo();
```

Você ainda pode passar argumentos para ele, como em:

```js
function *foo(x,y) {
	// ..
}

foo( 5, 10 );
```

A principal diferença é que executar um gerador, como `foo(5,10)`, na verdade não roda o código no gerador. Em vez disso, ele produz um iterador que controlará o gerador para executar seu código.

Voltaremos a isso mais adiante em "Controle de Iterador", mas resumidamente:

```js
function *foo() {
	// ..
}

var it = foo();

// para iniciar/avançar `*foo()`, chame
// `it.next(..)`
```

#### `yield`

Geradores também têm uma nova palavra-chave que você pode usar dentro deles, para sinalizar o ponto de pausa: `yield`. Considere:

```js
function *foo() {
	var x = 10;
	var y = 20;

	yield;

	var z = x + y;
}
```

Neste gerador `*foo()`, as operações nas duas primeiras linhas rodariam no início, então `yield` pausaria o gerador. Se e quando retomado, a última linha de `*foo()` rodaria. `yield` pode aparecer qualquer número de vezes (ou nenhuma vez, tecnicamente!) em um gerador.

Você pode até colocar `yield` dentro de um loop, e ele pode representar um ponto de pausa repetido. De fato, um loop que nunca se completa significa apenas um gerador que nunca se completa, o que é completamente válido, e às vezes exatamente o que você precisa.

`yield` não é apenas um ponto de pausa. É uma expressão que envia um valor ao pausar o gerador. Aqui está um loop `while..true` em um gerador que, para cada iteração, faz `yield` de um novo número aleatório:

```js
function *foo() {
	while (true) {
		yield Math.random();
	}
}
```

A expressão `yield ..` não apenas envia um valor -- `yield` sem um valor é o mesmo que `yield undefined` -- mas também recebe (por exemplo, é substituída por) o eventual valor de retomada. Considere:

```js
function *foo() {
	var x = yield 10;
	console.log( x );
}
```

Esse gerador primeiro fará `yield` do valor `10` ao pausar a si mesmo. Quando você retoma o gerador -- usando o `it.next(..)` ao qual nos referimos anteriormente -- qualquer valor (se houver) com o qual você retomar substituirá/completará toda a expressão `yield 10`, significando que esse valor será atribuído à variável `x`.

Uma expressão `yield ..` pode aparecer em qualquer lugar onde uma expressão normal pode. Por exemplo:

```js
function *foo() {
	var arr = [ yield 1, yield 2, yield 3 ];
	console.log( arr, yield 4 );
}
```

`*foo()` aqui tem quatro expressões `yield ..`. Cada `yield` resulta no gerador pausando para esperar um valor de retomada que é então usado nos vários contextos de expressão.

`yield` não é tecnicamente um operador, embora quando usado como `yield 1` ele certamente pareça um. Como `yield` pode ser usado sozinho como em `var x = yield;`, pensar nele como um operador pode às vezes ser confuso.

Tecnicamente, `yield ..` é da mesma "precedência de expressão" -- conceitualmente similar à precedência de operadores -- que uma expressão de atribuição como `a = 3`. Isso significa que `yield ..` pode basicamente aparecer em qualquer lugar onde `a = 3` possa validamente aparecer.

Vamos ilustrar a simetria:

```js
var a, b;

a = 3;					// válido
b = 2 + a = 3;			// inválido
b = 2 + (a = 3);		// válido

yield 3;				// válido
a = 2 + yield 3;		// inválido
a = 2 + (yield 3);		// válido
```

**Nota:** Se você pensar a respeito, faz uma espécie de sentido conceitual que uma expressão `yield ..` se comportaria de forma similar a uma expressão de atribuição. Quando uma expressão `yield` pausada é retomada, ela é completada/substituída pelo valor de retomada de uma forma não muito diferente de ter esse valor "atribuído" a ela.

A conclusão: se você precisa que `yield ..` apareça em uma posição onde uma atribuição como `a = 3` não seria por si só permitida, ela precisa ser envolvida em um `( )`.

Por causa da baixa precedência da palavra-chave `yield`, quase qualquer expressão após um `yield ..` será computada primeiro antes de ser enviada com `yield`. Apenas o operador spread `...` e o operador vírgula `,` têm precedência menor, significando que eles se ligariam após o `yield` ter sido avaliado.

Então, assim como com múltiplos operadores em instruções normais, outro caso onde `( )` pode ser necessário é para sobrescrever (elevar) a baixa precedência de `yield`, como a diferença entre estas expressões:

```js
yield 2 + 3;			// o mesmo que `yield (2 + 3)`

(yield 2) + 3;			// `yield 2` primeiro, depois `+ 3`
```

Assim como a atribuição `=`, `yield` também é "associativo à direita", o que significa que múltiplas expressões `yield` em sucessão são tratadas como tendo sido agrupadas com `( .. )` da direita para a esquerda. Então, `yield yield yield 3` é tratado como `yield (yield (yield 3))`. Uma interpretação "associativa à esquerda" como `((yield) yield) yield 3` não faria sentido nenhum.

Assim como com operadores, é uma boa ideia usar agrupamento `( .. )`, mesmo que não seja estritamente exigido, para desambiguar sua intenção se `yield` for combinado com outros operadores ou outros `yield`s.

**Nota:** Veja o título *Types & Grammar* desta série para mais informações sobre precedência e associatividade de operadores.

#### `yield *`

Da mesma forma que o `*` transforma uma declaração `function` em uma declaração de gerador `function *`, um `*` transforma `yield` em `yield *`, que é um mecanismo muito diferente, chamado de *delegação de yield*. Gramaticalmente, `yield *..` se comportará da mesma forma que um `yield ..`, como discutido na seção anterior.

`yield * ..` requer um iterável; ele então invoca o iterador desse iterável, e delega o controle de seu próprio gerador hospedeiro para esse iterador até que ele esteja exaurido. Considere:

```js
function *foo() {
	yield *[1,2,3];
}
```

**Nota:** Assim como com a posição do `*` na declaração de um gerador (discutido anteriormente), o posicionamento do `*` em expressões `yield *` é estilisticamente decisão sua. A maioria das outras literaturas prefere `yield* ..`, mas eu prefiro `yield *..`, por razões muito simétricas como já discutido.

O valor `[1,2,3]` produz um iterador que percorrerá seus valores, então o gerador `*foo()` fará yield desses valores conforme é consumido. Outra forma de ilustrar o comportamento é na delegação de yield para outro gerador:

```js
function *foo() {
	yield 1;
	yield 2;
	yield 3;
}

function *bar() {
	yield *foo();
}
```

O iterador produzido quando `*bar()` chama `*foo()` é delegado via `yield *`, significando que qualquer valor(es) que `*foo()` produzir será produzido por `*bar()`.

Enquanto com `yield ..` o valor de conclusão da expressão vem da retomada do gerador com `it.next(..)`, o valor de conclusão da expressão `yield *..` vem do valor de retorno (se houver) do iterador para o qual foi delegado.

Iteradores embutidos geralmente não têm valores de retorno, como cobrimos ao final da seção "Loop de Iterador" anteriormente neste capítulo. Mas se você definir seu próprio iterador (ou gerador) customizado, você pode projetá-lo para fazer `return` de um valor, que `yield *..` capturaria:

```js
function *foo() {
	yield 1;
	yield 2;
	yield 3;
	return 4;
}

function *bar() {
	var x = yield *foo();
	console.log( "x:", x );
}

for (var v of bar()) {
	console.log( v );
}
// 1 2 3
// x: 4
```

Enquanto os valores `1`, `2` e `3` recebem `yield` para fora de `*foo()` e então para fora de `*bar()`, o valor `4` retornado de `*foo()` é o valor de conclusão da expressão `yield *foo()`, que então é atribuído a `x`.

Como `yield *` pode chamar outro gerador (por meio da delegação ao seu iterador), ele também pode realizar uma espécie de recursão de gerador chamando a si mesmo:

```js
function *foo(x) {
	if (x < 3) {
		x = yield *foo( x + 1 );
	}
	return x * 2;
}

foo( 1 );
```

O resultado de `foo(1)` e então chamar o `next()` do iterador para rodá-lo através de seus passos recursivos será `24`. A primeira execução de `*foo(..)` tem `x` com valor `1`, que é `x < 3`. `x + 1` é passado recursivamente para `*foo(..)`, então `x` é então `2`. Mais uma chamada recursiva resulta em `x` igual a `3`.

Agora, como `x < 3` falha, a recursão para, e `return 3 * 2` devolve `6` à expressão `yield *..` da chamada anterior, que é então atribuído a `x`. Outro `return 6 * 2` devolve `12` ao `x` da chamada anterior. Finalmente `12 * 2`, ou `24`, é retornado da execução completa do gerador `*foo(..)`.

### Controle de Iterador

Anteriormente, introduzimos brevemente o conceito de que geradores são controlados por iteradores. Vamos nos aprofundar nisso totalmente agora.

Lembre-se do `*foo(..)` recursivo da seção anterior. Veja como rodaríamos isso:

```js
function *foo(x) {
	if (x < 3) {
		x = yield *foo( x + 1 );
	}
	return x * 2;
}

var it = foo( 1 );
it.next();				// { value: 24, done: true }
```

Nesse caso, o gerador na verdade nunca pausa, pois não há expressão `yield ..`. Em vez disso, `yield *` apenas mantém o passo de iteração atual em andamento via a chamada recursiva. Então, apenas uma chamada à função `next()` do iterador roda completamente o gerador.

Agora vamos considerar um gerador que terá múltiplos passos e, portanto, múltiplos valores produzidos:

```js
function *foo() {
	yield 1;
	yield 2;
	yield 3;
}
```

Já sabemos que podemos consumir um iterador, mesmo um anexado a um gerador como `*foo()`, com um loop `for..of`:

```js
for (var v of foo()) {
	console.log( v );
}
// 1 2 3
```

**Nota:** O loop `for..of` requer um iterável. Uma referência a uma função geradora (como `foo`) por si só não é um iterável; você deve executá-la com `foo()` para obter o iterador (que também é um iterável, como explicamos anteriormente neste capítulo). Você poderia teoricamente estender o `GeneratorPrototype` (o protótipo de todas as funções geradoras) com uma função `Symbol.iterator` que essencialmente apenas faz `return this()`. Isso tornaria a própria referência `foo` um iterável, o que significa que `for (var v of foo) { .. }` (note a ausência de `()` em `foo`) funcionará.

Vamos em vez disso iterar o gerador manualmente:

```js
function *foo() {
	yield 1;
	yield 2;
	yield 3;
}

var it = foo();

it.next();				// { value: 1, done: false }
it.next();				// { value: 2, done: false }
it.next();				// { value: 3, done: false }

it.next();				// { value: undefined, done: true }
```

Se você olhar de perto, há três instruções `yield` e quatro chamadas de `next()`. Isso pode parecer uma incompatibilidade estranha. De fato, sempre haverá uma chamada de `next()` a mais do que expressões `yield`, assumindo que todas sejam avaliadas e o gerador seja rodado completamente até a conclusão.

Mas se você olhar da perspectiva oposta (de dentro para fora em vez de de fora para dentro), a correspondência entre `yield` e `next()` faz mais sentido.

Lembre-se de que a expressão `yield ..` será completada pelo valor com o qual você retomar o gerador. Isso significa que o argumento que você passa para `next(..)` completa qualquer que seja a expressão `yield ..` que está atualmente pausada esperando por uma conclusão.

Vamos ilustrar essa perspectiva desta forma:

```js
function *foo() {
	var x = yield 1;
	var y = yield 2;
	var z = yield 3;
	console.log( x, y, z );
}
```

Neste fragmento, cada `yield ..` está enviando um valor para fora (`1`, `2`, `3`), mas mais diretamente, está pausando o gerador para esperar por um valor. Em outras palavras, é quase como fazer a pergunta: "Que valor devo usar aqui? Vou esperar para ouvir a resposta."

Agora, veja como controlamos `*foo()` para iniciá-lo:

```js
var it = foo();

it.next();				// { value: 1, done: false }
```

Essa primeira chamada de `next()` está iniciando o gerador a partir de seu estado pausado inicial, e rodando-o até o primeiro `yield`. No momento em que você chama esse primeiro `next()`, não há expressão `yield ..` esperando por uma conclusão. Se você passasse um valor para essa primeira chamada de `next()`, ele seria atualmente apenas descartado, porque nenhum `yield` está esperando para receber tal valor.

**Nota:** Uma proposta inicial para o período "além do ES6" *permitiria* que você acessasse um valor passado a uma chamada inicial de `next(..)` via uma meta property separada (veja o Capítulo 7) dentro do gerador.

Agora, vamos responder à pergunta atualmente pendente: "Que valor devo atribuir a `x`?" Vamos respondê-la enviando um valor à *próxima* chamada de `next(..)`:

```js
it.next( "foo" );		// { value: 2, done: false }
```

Agora, `x` terá o valor `"foo"`, mas também fizemos uma nova pergunta: "Que valor devo atribuir a `y`?" E respondemos:

```js
it.next( "bar" );		// { value: 3, done: false }
```

Resposta dada, outra pergunta feita. Resposta final:

```js
it.next( "baz" );		// "foo" "bar" "baz"
						// { value: undefined, done: true }
```

Agora deve estar mais claro como cada "pergunta" `yield ..` é respondida pela *próxima* chamada de `next(..)`, e assim a chamada de `next()` "extra" que observamos é sempre apenas a inicial que coloca tudo em movimento.

Vamos juntar todos esses passos:

```js
var it = foo();

// inicia o gerador
it.next();				// { value: 1, done: false }

// responde à primeira pergunta
it.next( "foo" );		// { value: 2, done: false }

// responde à segunda pergunta
it.next( "bar" );		// { value: 3, done: false }

// responde à terceira pergunta
it.next( "baz" );		// "foo" "bar" "baz"
						// { value: undefined, done: true }
```

Você pode pensar em um gerador como um produtor de valores, caso em que cada iteração está simplesmente produzindo um valor a ser consumido.

Mas em um sentido mais geral, talvez seja apropriado pensar em geradores como execução de código controlada e progressiva, muito parecido com o exemplo da fila `tasks` da seção anterior "Iteradores Customizados".

**Nota:** Essa perspectiva é exatamente a motivação para como revisitaremos geradores no Capítulo 4. Especificamente, não há razão para que `next(..)` tenha que ser chamado imediatamente após o `next(..)` anterior terminar. Enquanto o contexto de execução interno do gerador está pausado, o resto do programa continua desbloqueado, incluindo a capacidade de ações assíncronas controlarem quando o gerador é retomado.

### Conclusão Antecipada

Como cobrimos anteriormente neste capítulo, o iterador anexado a um gerador suporta os métodos opcionais `return(..)` e `throw(..)`. Ambos têm o efeito de abortar imediatamente um gerador pausado.

Considere:

```js
function *foo() {
	yield 1;
	yield 2;
	yield 3;
}

var it = foo();

it.next();				// { value: 1, done: false }

it.return( 42 );		// { value: 42, done: true }

it.next();				// { value: undefined, done: true }
```

`return(x)` é como forçar um `return x` a ser processado exatamente naquele momento, de modo que você recebe o valor especificado de volta. Uma vez que um gerador é completado, seja normalmente ou antecipadamente como mostrado, ele não processa mais nenhum código nem retorna mais nenhum valor.

Além de `return(..)` ser chamável manualmente, ele também é chamado automaticamente ao final da iteração por qualquer uma das construções do ES6 que consomem iteradores, como o loop `for..of` e o operador spread `...`.

O propósito dessa capacidade é que o gerador possa ser notificado se o código de controle não vai mais iterar sobre ele, para que ele possa talvez fazer quaisquer tarefas de limpeza (liberar recursos, redefinir status, etc.). Idêntico a um padrão de limpeza de função normal, a principal maneira de realizar isso é usar uma cláusula `finally`:

```js
function *foo() {
	try {
		yield 1;
		yield 2;
		yield 3;
	}
	finally {
		console.log( "cleanup!" );
	}
}

for (var v of foo()) {
	console.log( v );
}
// 1 2 3
// cleanup!

var it = foo();

it.next();				// { value: 1, done: false }
it.return( 42 );		// cleanup!
						// { value: 42, done: true }
```

**Aviso:** Não coloque uma instrução `yield` dentro da cláusula `finally`! É válido e legal, mas é uma ideia realmente terrível. Ele age, em certo sentido, como adiar a conclusão da chamada de `return(..)` que você fez, já que quaisquer expressões `yield ..` na cláusula `finally` são respeitadas para pausar e enviar mensagens; você não obtém imediatamente um gerador completado como esperado. Não há basicamente nenhuma boa razão para optar por essa loucura *má parte*, então evite fazê-lo!

Além do fragmento anterior mostrar como `return(..)` aborta o gerador enquanto ainda dispara a cláusula `finally`, ele também demonstra que um gerador produz um iterador completamente novo cada vez que é chamado. De fato, você pode usar múltiplos iteradores anexados ao mesmo gerador concorrentemente:

```js
function *foo() {
	yield 1;
	yield 2;
	yield 3;
}

var it1 = foo();
it1.next();				// { value: 1, done: false }
it1.next();				// { value: 2, done: false }

var it2 = foo();
it2.next();				// { value: 1, done: false }

it1.next();				// { value: 3, done: false }

it2.next();				// { value: 2, done: false }
it2.next();				// { value: 3, done: false }

it2.next();				// { value: undefined, done: true }
it1.next();				// { value: undefined, done: true }
```

#### Aborto Antecipado

Em vez de chamar `return(..)`, você pode chamar `throw(..)`. Assim como `return(x)` é essencialmente injetar um `return x` no gerador em seu ponto de pausa atual, chamar `throw(x)` é essencialmente como injetar um `throw x` no ponto de pausa.

Além do comportamento de exceção (cobrimos o que isso significa para cláusulas `try` na próxima seção), `throw(..)` produz o mesmo tipo de conclusão antecipada que aborta a execução do gerador em seu ponto de pausa atual. Por exemplo:

```js
function *foo() {
	yield 1;
	yield 2;
	yield 3;
}

var it = foo();

it.next();				// { value: 1, done: false }

try {
	it.throw( "Oops!" );
}
catch (err) {
	console.log( err );	// Exception: Oops!
}

it.next();				// { value: undefined, done: true }
```

Como `throw(..)` basicamente injeta um `throw ..` em substituição da linha `yield 1` do gerador, e nada trata essa exceção, ela se propaga imediatamente de volta para o código chamador, que a trata com um `try..catch`.

Diferentemente de `return(..)`, o método `throw(..)` do iterador nunca é chamado automaticamente.

É claro que, embora não mostrado no fragmento anterior, se uma cláusula `try..finally` estivesse esperando dentro do gerador quando você chama `throw(..)`, a cláusula `finally` teria a chance de ser concluída antes que a exceção seja propagada de volta para o código chamador.

### Tratamento de Erros

Como já insinuamos, o tratamento de erros com geradores pode ser expresso com `try..catch`, que funciona em ambas as direções, de entrada e de saída:

```js
function *foo() {
	try {
		yield 1;
	}
	catch (err) {
		console.log( err );
	}

	yield 2;

	throw "Hello!";
}

var it = foo();

it.next();				// { value: 1, done: false }

try {
	it.throw( "Hi!" );	// Hi!
						// { value: 2, done: false }
	it.next();

	console.log( "never gets here" );
}
catch (err) {
	console.log( err );	// Hello!
}
```

Erros também podem se propagar em ambas as direções através da delegação `yield *`:

```js
function *foo() {
	try {
		yield 1;
	}
	catch (err) {
		console.log( err );
	}

	yield 2;

	throw "foo: e2";
}

function *bar() {
	try {
		yield *foo();

		console.log( "never gets here" );
	}
	catch (err) {
		console.log( err );
	}
}

var it = bar();

try {
	it.next();			// { value: 1, done: false }

	it.throw( "e1" );	// e1
						// { value: 2, done: false }

	it.next();			// foo: e2
						// { value: undefined, done: true }
}
catch (err) {
	console.log( "never gets here" );
}

it.next();				// { value: undefined, done: true }
```

Quando `*foo()` chama `yield 1`, o valor `1` passa através de `*bar()` intocado, como já vimos.

Mas o que é mais interessante sobre este fragmento é que quando `*foo()` chama `throw "foo: e2"`, esse erro se propaga para `*bar()` e é imediatamente capturado pelo bloco `try..catch` de `*bar()`. O erro não passa através de `*bar()` como o valor `1` passou.

O `catch` de `*bar()` então faz uma saída normal de `err` (`"foo: e2"`) e então `*bar()` termina normalmente, e é por isso que o resultado de iterador `{ value: undefined, done: true }` retorna de `it.next()`.

Se `*bar()` não tivesse um `try..catch` em volta da expressão `yield *..`, o erro é claro se propagaria todo o caminho para fora, e no caminho ainda completaria (abortaria) `*bar()`.

### Transpilando um Gerador

É possível representar as capacidades de um gerador antes do ES6? Acontece que é, e existem várias ótimas ferramentas que fazem isso, incluindo notavelmente a ferramenta Regenerator do Facebook (https://facebook.github.io/regenerator/).

Mas apenas para entender melhor os geradores, vamos tentar fazer uma conversão manual. Basicamente, vamos criar uma máquina de estados simples baseada em closure.

Vamos manter nosso gerador de origem realmente simples:

```js
function *foo() {
	var x = yield 42;
	console.log( x );
}
```

Para começar, precisaremos de uma função chamada `foo()` que possamos executar, que precisa retornar um iterador:

```js
function foo() {
	// ..

	return {
		next: function(v) {
			// ..
		}

		// vamos pular `return(..)` e `throw(..)`
	};
}
```

Agora, precisamos de alguma variável interna para acompanhar onde estamos nos passos da lógica do nosso "gerador". Vamos chamá-la de `state`. Haverá três estados: `0` inicialmente, `1` enquanto espera para cumprir a expressão `yield`, e `2` uma vez que o gerador esteja completo.

Cada vez que `next(..)` é chamado, precisamos processar o próximo passo, e então incrementar `state`. Por conveniência, colocaremos cada passo em uma cláusula `case` de uma instrução `switch`, e a manteremos em uma função interna chamada `nextState(..)` que `next(..)` pode chamar. Além disso, como `x` é uma variável ao longo do escopo geral do "gerador", ela precisa viver fora da função `nextState(..)`.

Aqui está tudo junto (obviamente um tanto simplificado, para manter a ilustração conceitual mais clara):

```js
function foo() {
	function nextState(v) {
		switch (state) {
			case 0:
				state++;

				// a expressão `yield`
				return 42;
			case 1:
				state++;

				// expressão `yield` cumprida
				x = v;
				console.log( x );

				// o `return` implícito
				return undefined;

			// não há necessidade de tratar o estado `2`
		}
	}

	var state = 0, x;

	return {
		next: function(v) {
			var ret = nextState( v );

			return { value: ret, done: (state == 2) };
		}

		// vamos pular `return(..)` e `throw(..)`
	};
}
```

E finalmente, vamos testar nosso "gerador" pré-ES6:

```js
var it = foo();

it.next();				// { value: 42, done: false }

it.next( 10 );			// 10
						// { value: undefined, done: true }
```

Nada mau, hein? Espero que este exercício solidifique em sua mente que geradores são na verdade apenas uma sintaxe simples para lógica de máquina de estados. Isso os torna amplamente aplicáveis.

### Usos de Geradores

Então, agora que entendemos muito mais profundamente como geradores funcionam, para que eles são úteis?

Vimos dois padrões principais:

* *Produzir uma série de valores:* Esse uso pode ser simples (por exemplo, strings aleatórias ou números incrementados), ou pode representar acesso a dados mais estruturado (por exemplo, iterar sobre linhas retornadas de uma consulta de banco de dados).

   De qualquer forma, usamos o iterador para controlar um gerador de modo que alguma lógica possa ser invocada para cada chamada de `next(..)`. Iteradores normais em estruturas de dados meramente puxam valores sem qualquer lógica de controle.
* *Fila de tarefas a serem realizadas serialmente:* Esse uso frequentemente representa controle de fluxo para os passos de um algoritmo, onde cada passo requer a recuperação de dados de alguma fonte externa. O cumprimento de cada pedaço de dado pode ser imediato, ou pode ser assincronamente adiado.

   Da perspectiva do código dentro do gerador, os detalhes de síncrono ou assíncrono em um ponto `yield` são inteiramente opacos. Além disso, esses detalhes são intencionalmente abstraídos, de modo a não obscurecer a expressão sequencial natural dos passos com tais complicações de implementação. A abstração também significa que as implementações podem ser trocadas/refatoradas frequentemente sem tocar no código do gerador de forma alguma.

Quando geradores são vistos à luz desses usos, eles se tornam muito mais do que apenas uma sintaxe diferente ou mais agradável para uma máquina de estados manual. Eles são uma poderosa ferramenta de abstração para organizar e controlar a produção e o consumo ordenados de dados.

## Módulos

Não acho que seja exagero sugerir que o padrão de organização de código mais importante de todo o JavaScript é, e sempre foi, o módulo. Para mim, e acho que para uma grande parcela da comunidade, o padrão de módulo conduz a vasta maioria do código.

### A Maneira Antiga

O padrão de módulo tradicional é baseado em uma função externa com variáveis e funções internas, e uma "API pública" retornada com métodos que têm closure sobre os dados e capacidades internas. Frequentemente é expresso assim:

```js
function Hello(name) {
	function greeting() {
		console.log( "Hello " + name + "!" );
	}

	// API pública
	return {
		greeting: greeting
	};
}

var me = Hello( "Kyle" );
me.greeting();			// Hello Kyle!
```

Esse módulo `Hello(..)` pode produzir múltiplas instâncias sendo chamado vezes subsequentes. Às vezes, um módulo é chamado apenas como um singleton (ou seja, ele precisa apenas de uma instância), caso em que uma leve variação do fragmento anterior, usando uma IIFE, é comum:

```js
var me = (function Hello(name){
	function greeting() {
		console.log( "Hello " + name + "!" );
	}

	// API pública
	return {
		greeting: greeting
	};
})( "Kyle" );

me.greeting();			// Hello Kyle!
```

Esse padrão é experimentado e testado. Também é flexível o suficiente para ter uma ampla variedade de variações para um número de cenários diferentes.

Um dos mais comuns é o Asynchronous Module Definition (AMD), e outro é o Universal Module Definition (UMD). Não cobriremos os detalhes desses padrões e técnicas aqui, mas eles são explicados extensivamente em muitos lugares online.

### Avançando

A partir do ES6, não precisamos mais depender da função envolvente e do closure para nos fornecer suporte a módulos. Os módulos do ES6 têm suporte sintático e funcional de primeira classe.

Antes de entrarmos na sintaxe específica, é importante entender algumas diferenças conceituais bastante significativas dos módulos ES6 comparados a como você pode ter lidado com módulos no passado:

* O ES6 usa módulos baseados em arquivo, significando um módulo por arquivo. No momento, não há uma maneira padronizada de combinar múltiplos módulos em um único arquivo.

   Isso significa que se você for carregar módulos ES6 diretamente em uma aplicação web de navegador, você os carregará individualmente, não como um grande bundle em um único arquivo, como tem sido comum em esforços de otimização de performance.

   Espera-se que o advento contemporâneo do HTTP/2 mitigue significativamente quaisquer preocupações de performance desse tipo, já que ele opera em uma conexão de socket persistente e, portanto, pode carregar muito eficientemente muitos arquivos menores em paralelo e intercalados uns com os outros.
* A API de um módulo ES6 é estática. Ou seja, você define estaticamente quais são todos os exports de nível superior na API pública do seu módulo, e esses não podem ser alterados posteriormente.

   Alguns usos estão acostumados a poder fornecer definições de API dinâmicas, onde métodos podem ser adicionados/removidos/substituídos em resposta a condições de tempo de execução. Ou esses usos terão que mudar para se adequar às APIs estáticas do ES6, ou terão que restringir as mudanças dinâmicas a propriedades/métodos de um objeto de segundo nível.
* Módulos ES6 são singletons. Ou seja, há apenas uma instância do módulo, que mantém seu estado. Cada vez que você importa esse módulo em outro módulo, você obtém uma referência à única instância centralizada. Se você quiser ser capaz de produzir múltiplas instâncias de módulo, seu módulo precisará fornecer algum tipo de factory para fazê-lo.
* As propriedades e métodos que você expõe na API pública de um módulo não são apenas atribuições normais de valores ou referências. Eles são bindings reais (quase como ponteiros) para os identificadores na sua definição interna de módulo.

   Em módulos pré-ES6, se você colocasse uma propriedade na sua API pública que contém um valor primitivo como um número ou string, essa atribuição de propriedade era por cópia de valor, e qualquer atualização interna de uma variável correspondente seria separada e não afetaria a cópia pública no objeto da API.

   Com o ES6, exportar uma variável privada local, mesmo que ela atualmente contenha uma string/número/etc. primitivo, exporta um binding para a variável. Se o módulo mudar o valor da variável, o binding de import externo agora resolve para esse novo valor.
* Importar um módulo é a mesma coisa que solicitar estaticamente que ele seja carregado (se ainda não foi). Se você está em um navegador, isso implica um carregamento bloqueante pela rede. Se você está em um servidor (ou seja, Node.js), é um carregamento bloqueante do sistema de arquivos.

   Entretanto, não entre em pânico sobre as implicações de performance. Como os módulos ES6 têm definições estáticas, os requisitos de import podem ser escaneados estaticamente, e os carregamentos acontecerão preemptivamente, mesmo antes de você ter usado o módulo.

   O ES6 na verdade não especifica nem trata a mecânica de como essas requisições de carregamento funcionam. Há uma noção separada de um Module Loader, onde cada ambiente hospedeiro (navegador, Node.js, etc.) fornece um Loader padrão apropriado ao ambiente. A importação de um módulo usa um valor de string para representar onde obter o módulo (URL, caminho de arquivo, etc.), mas esse valor é opaco no seu programa e só tem significado para o próprio Loader.

   Você pode definir seu próprio Loader customizado se quiser controle mais granular do que o Loader padrão proporciona -- que é basicamente nenhum, já que ele está totalmente oculto do código do seu programa.

Como você pode ver, módulos ES6 servirão ao caso de uso geral de organizar código com encapsulamento, controlar APIs públicas e referenciar imports de dependências. Mas eles têm uma maneira muito particular de fazê-lo, e isso pode ou não se adequar muito de perto a como você já vem fazendo módulos há anos.

#### CommonJS

Há uma sintaxe de módulo similar, mas não totalmente compatível, chamada CommonJS, que é familiar para aqueles no ecossistema Node.js.

Por falta de uma maneira mais delicada de dizer isso, no longo prazo, os módulos ES6 essencialmente estão fadados a substituir todos os formatos e padrões anteriores de módulos, até mesmo o CommonJS, já que eles são construídos sobre suporte sintático na linguagem. Isso, com o tempo, inevitavelmente vencerá como a abordagem superior, nem que seja por nenhuma outra razão além da ubiquidade.

Enfrentamos uma estrada bastante longa para chegar a esse ponto, porém. Existem literalmente centenas de milhares de módulos no estilo CommonJS no mundo do JavaScript do lado do servidor, e 10 vezes mais módulos de vários padrões de formato (UMD, AMD, ad hoc) no mundo dos navegadores. Levará muitos anos para que as transições façam qualquer progresso significativo.

No ínterim, transpiladores/conversores de módulos serão uma absoluta necessidade. É melhor você simplesmente se acostumar com essa nova realidade. Quer você escreva em módulos regulares, AMD, UMD, CommonJS ou ES6, essas ferramentas terão que parsear e converter para um formato que seja adequado a qualquer ambiente onde seu código vá rodar.

Para Node.js, isso provavelmente significa (por enquanto) que o alvo é CommonJS. Para o navegador, é provavelmente UMD ou AMD. Espere bastante flutuação nisso ao longo dos próximos anos conforme essas ferramentas amadurecem e as boas práticas emergem.

Daqui em diante, meu melhor conselho sobre módulos é este: qualquer que seja o formato ao qual você esteja religiosamente apegado com forte afinidade, desenvolva também uma apreciação e entendimento dos módulos ES6, tais como são, e deixe suas outras tendências de módulo desvanecerem. Eles *são* o futuro dos módulos em JS, mesmo que essa realidade esteja um pouco distante.

### A Maneira Nova

As duas principais novas palavras-chave que habilitam módulos ES6 são `import` e `export`. Há muita nuance na sintaxe, então vamos dar uma olhada mais profunda.

**Aviso:** Um detalhe importante que é fácil de ignorar: tanto `import` quanto `export` devem sempre aparecer no escopo de nível superior de seu respectivo uso. Por exemplo, você não pode colocar um `import` ou `export` dentro de um condicional `if`; eles devem aparecer fora de todos os blocos e funções.

#### `export`ando Membros da API

A palavra-chave `export` é colocada ou na frente de uma declaração, ou usada como um operador (de certa forma) com uma lista especial de bindings a exportar. Considere:

```js
export function foo() {
	// ..
}

export var awesome = 42;

var bar = [1,2,3];
export { bar };
```

Outra maneira de expressar os mesmos exports:

```js
function foo() {
	// ..
}

var awesome = 42;
var bar = [1,2,3];

export { foo, awesome, bar };
```

Esses são todos chamados de *named exports*, já que você está, com efeito, exportando os bindings de nome das variáveis/funções/etc.

Qualquer coisa que você não *rotule* com `export` permanece privada dentro do escopo do módulo. Ou seja, embora algo como `var bar = ..` pareça estar declarando no escopo global de nível superior, o escopo de nível superior é na verdade o próprio módulo; não há escopo global em módulos.

**Nota:** Módulos *ainda* têm acesso a `window` e a todas as "globais" que dependem dele, apenas não como escopo léxico de nível superior. Entretanto, você realmente deveria ficar longe das globais nos seus módulos se possível.

Você também pode "renomear" (também conhecido como criar alias) um membro de módulo durante o named export:

```js
function foo() { .. }

export { foo as bar };
```

Quando este módulo é importado, apenas o nome de membro `bar` está disponível para import; `foo` permanece oculto dentro do módulo.

Exports de módulo não são apenas atribuições normais de valores ou referências, como você está acostumado com o operador de atribuição `=`. Na verdade, quando você exporta algo, você está exportando um binding (meio que como um ponteiro) para aquela coisa (variável, etc.).

Dentro do seu módulo, se você mudar o valor de uma variável para a qual você já exportou um binding, mesmo que ele já tenha sido importado (veja a próxima seção), o binding importado resolverá para o valor atual (atualizado).

Considere:

```js
var awesome = 42;
export { awesome };

// mais tarde
awesome = 100;
```

Quando este módulo é importado, independentemente de isso ser antes ou depois do ajuste `awesome = 100`, uma vez que essa atribuição tenha acontecido, o binding importado resolve para o valor `100`, não `42`.

Isso porque o binding é, em essência, uma referência a, ou um ponteiro para, a própria variável `awesome`, em vez de uma cópia de seu valor. Esse é um conceito majoritariamente sem precedentes para JS introduzido com os bindings de módulo do ES6.

Embora você possa claramente usar `export` múltiplas vezes dentro da definição de um módulo, o ES6 definitivamente prefere a abordagem de que um módulo tenha um único export, o que é conhecido como *default export*. Nas palavras de alguns membros do comitê TC39, você é "recompensado com uma sintaxe `import` mais simples" se seguir esse padrão, e inversamente "penalizado" com sintaxe mais verbosa se não fizer.

Um default export define um binding exportado específico para ser o padrão ao importar o módulo. O nome do binding é literalmente `default`. Como você verá mais adiante, ao importar bindings de módulo você também pode renomeá-los, como comumente fará com um default export.

Só pode haver um `default` por definição de módulo. Cobriremos `import` na próxima seção, e você verá como a sintaxe de `import` é mais concisa se o módulo tem um default export.

Há uma nuance sutil na sintaxe de default export à qual você deveria prestar muita atenção. Compare estes dois fragmentos:

```js
function foo(..) {
	// ..
}

export default foo;
```

E este aqui:

```js
function foo(..) {
	// ..
}

export { foo as default };
```

No primeiro fragmento, você está exportando um binding para o valor da expressão de função naquele momento, *não* para o identificador `foo`. Em outras palavras, `export default ..` recebe uma expressão. Se você posteriormente atribuir `foo` a um valor diferente dentro do seu módulo, o import do módulo ainda revela a função originalmente exportada, não o novo valor.

A propósito, o primeiro fragmento também poderia ter sido escrito como:

```js
export default function foo(..) {
	// ..
}
```

**Aviso:** Embora a parte `function foo..` aqui seja tecnicamente uma expressão de função, para os propósitos do escopo interno do módulo, ela é tratada como uma declaração de função, no sentido de que o nome `foo` é ligado no escopo de nível superior do módulo (frequentemente chamado de "hoisting"). O mesmo é verdade para `export default class Foo..`. Entretanto, embora você *possa* fazer `export var foo = ..`, atualmente você não pode fazer `export default var foo = ..` (ou `let` ou `const`), em um caso frustrante de inconsistência. No momento em que isto é escrito, já há discussão sobre adicionar essa capacidade em breve, pós-ES6, por uma questão de consistência.

Lembre-se do segundo fragmento novamente:

```js
function foo(..) {
	// ..
}

export { foo as default };
```

Nesta versão do export de módulo, o binding de default export é na verdade para o identificador `foo` em vez de seu valor, então você obtém o comportamento de binding descrito anteriormente (ou seja, se você posteriormente mudar o valor de `foo`, o valor visto do lado do import também será atualizado).

Tenha muito cuidado com essa pegadinha sutil na sintaxe de default export, especialmente se a sua lógica exige que valores de export sejam atualizados. Se você nunca planeja atualizar o valor de um default export, `export default ..` está ótimo. Se você planeja atualizar o valor, você deve usar `export { .. as default }`. De qualquer forma, certifique-se de comentar seu código para explicar sua intenção!

Como só pode haver um `default` por módulo, você pode ser tentado a projetar seu módulo com um único default export de um objeto com todos os seus métodos de API nele, como:

```js
export default {
	foo() { .. },
	bar() { .. },
	..
};
```

Esse padrão parece mapear de perto a como muitos desenvolvedores já estruturaram seus módulos pré-ES6, então parece uma abordagem natural. Infelizmente, ele tem algumas desvantagens e é oficialmente desencorajado.

Em particular, o motor JS não pode analisar estaticamente o conteúdo de um objeto comum, o que significa que ele não pode fazer algumas otimizações para performance de `import` estático. A vantagem de ter cada membro individualmente e explicitamente exportado é que o motor *pode* fazer a análise estática e a otimização.

Se a sua API tem mais de um membro, parece que esses princípios -- um default export por módulo, e todos os membros da API como named exports -- estão em conflito, não é? Mas você *pode* ter um único default export bem como outros named exports; eles não são mutuamente exclusivos.

Então, em vez deste padrão (desencorajado):

```js
export default function foo() { .. }

foo.bar = function() { .. };
foo.baz = function() { .. };
```

Você pode fazer:

```js
export default function foo() { .. }

export function bar() { .. }
export function baz() { .. }
```

**Nota:** Neste fragmento anterior, usei o nome `foo` para a função que `default` rotula. Esse nome `foo`, no entanto, é ignorado para os propósitos do export -- `default` é na verdade o nome exportado. Quando você importa esse binding default, você pode dar a ele qualquer nome que quiser, como verá na próxima seção.

Alternativamente, alguns preferirão:

```js
function foo() { .. }
function bar() { .. }
function baz() { .. }

export { foo as default, bar, baz, .. };
```

Os efeitos de misturar default e named exports ficarão mais claros quando cobrirmos `import` em breve. Mas essencialmente significa que a forma de import default mais concisa recuperaria apenas a função `foo()`. O usuário poderia adicionalmente listar manualmente `bar` e `baz` como named imports, se os quiser.

Você provavelmente pode imaginar quão tedioso isso vai ser para os consumidores do seu módulo se você tiver muitos bindings de named export. Há uma forma de import wildcard onde você importa todos os exports de um módulo dentro de um único objeto de namespace, mas não há como fazer um import wildcard para bindings de nível superior.

Novamente, o mecanismo de módulo do ES6 é intencionalmente projetado para desencorajar módulos com muitos exports; relativamente falando, é desejado que tais abordagens sejam um pouco mais difíceis, como uma espécie de engenharia social para encorajar design de módulo simples em favor de design de módulo grande/complexo.

Eu provavelmente recomendaria que você não misture default export com named exports, especialmente se você tem uma API grande e refatorar para módulos separados não é prático ou desejado. Nesse caso, apenas use todos os named exports, e documente que os consumidores do seu módulo provavelmente deveriam usar a abordagem `import * as ..` (namespace import, discutida na próxima seção) para trazer toda a API de uma só vez em um único namespace.

Mencionamos isso anteriormente, mas vamos voltar a isso em mais detalhe. Além da forma `export default ...` que exporta um binding de valor de expressão, todas as outras formas de export estão exportando bindings para identificadores locais. Para esses bindings, se você mudar o valor de uma variável dentro de um módulo após exportar, o binding importado externo acessará o valor atualizado:

```js
var foo = 42;
export { foo as default };

export var bar = "hello world";

foo = 10;
bar = "cool";
```

Quando você importar este módulo, os exports `default` e `bar` estarão ligados às variáveis locais `foo` e `bar`, significando que eles revelarão os valores atualizados `10` e `"cool"`. Os valores no momento do export são irrelevantes. Os valores no momento do import são irrelevantes. Os bindings são links vivos, então tudo o que importa é qual é o valor atual quando você acessa o binding.

**Aviso:** Bindings nos dois sentidos não são permitidos. Se você importar um `foo` de um módulo, e tentar mudar o valor da sua variável `foo` importada, um erro será lançado! Revisitaremos isso na próxima seção.

Você também pode re-exportar os exports de outro módulo, como:

```js
export { foo, bar } from "baz";
export { foo as FOO, bar as BAR } from "baz";
export * from "baz";
```

Essas formas são similares a primeiro importar do módulo `"baz"` e então listar seus membros explicitamente para export do seu módulo. Entretanto, nessas formas, os membros do módulo `"baz"` nunca são importados para o escopo local do seu módulo; eles meio que passam intocados.

#### `import`ando Membros da API

Para importar um módulo, sem surpresa você usa a instrução `import`. Assim como `export` tem várias variações com nuances, `import` também tem, então gaste bastante tempo considerando as seguintes questões e experimentando suas opções.

Se você quiser importar certos membros nomeados específicos da API de um módulo para o seu escopo de nível superior, você usa esta sintaxe:

```js
import { foo, bar, baz } from "foo";
```

**Aviso:** A sintaxe `{ .. }` aqui pode parecer um literal de objeto, ou até mesmo uma sintaxe de desestruturação de objeto. Entretanto, sua forma é especial apenas para módulos, então tome cuidado para não confundi-la com outros padrões `{ .. }` em outros lugares.

A string `"foo"` é chamada de *module specifier*. Como todo o objetivo é uma sintaxe estaticamente analisável, o module specifier deve ser um literal de string; ele não pode ser uma variável contendo o valor da string.

Da perspectiva do seu código ES6 e do próprio motor JS, o conteúdo deste literal de string é completamente opaco e sem significado. O module loader interpretará essa string como uma instrução de onde encontrar o módulo desejado, seja como um caminho de URL ou um caminho de sistema de arquivos local.

Os identificadores `foo`, `bar` e `baz` listados devem corresponder a named exports na API do módulo (análise estática e asserção de erro se aplicam). Eles são ligados como identificadores de nível superior no seu escopo atual:

```js
import { foo } from "foo";

foo();
```

Você pode renomear os identificadores ligados importados, como:

```js
import { foo as theFooFunc } from "foo";

theFooFunc();
```

Se o módulo tem apenas um default export que você quer importar e ligar a um identificador, você pode optar por pular a sintaxe `{ .. }` envolvente para esse binding. O `import` neste caso preferido obtém a mais agradável e concisa das formas de sintaxe de `import`:

```js
import foo from "foo";

// ou:
import { default as foo } from "foo";
```

**Nota:** Como explicado na seção anterior, a palavra-chave `default` no `export` de um módulo especifica um named export onde o nome é na verdade `default`, como ilustrado pela segunda opção de sintaxe mais verbosa. A renomeação de `default` para, neste caso, `foo`, é explícita na sintaxe posterior e idêntica, ainda que implícita, na sintaxe anterior.

Você também pode importar um default export junto com outros named exports, se o módulo tiver tal definição. Lembre-se desta definição de módulo de antes:

```js
export default function foo() { .. }

export function bar() { .. }
export function baz() { .. }
```

Para importar o default export desse módulo e seus dois named exports:

```js
import FOOFN, { bar, baz as BAZ } from "foo";

FOOFN();
bar();
BAZ();
```

A abordagem fortemente sugerida pela filosofia de módulos do ES6 é que você importe apenas os bindings específicos de um módulo que você precisa. Se um módulo fornece 10 métodos de API, mas você precisa apenas de dois deles, alguns acreditam que é um desperdício trazer todo o conjunto de bindings da API.

Um benefício, além de o código ser mais explícito, é que imports estreitos tornam a análise estática e a detecção de erros (usar acidentalmente o nome de binding errado, por exemplo) mais robustas.

É claro que essa é apenas a posição padrão influenciada pela filosofia de design do ES6; não há nada que exija aderência a essa abordagem.

Muitos desenvolvedores seriam rápidos em apontar que tais abordagens podem ser mais tediosas, exigindo que você regularmente revisite e atualize sua(s) instrução(ões) de `import` cada vez que perceber que precisa de algo mais de um módulo. A compensação é em troca de conveniência.

À luz disso, a preferência pode ser importar tudo do módulo para um único namespace, em vez de importar membros individuais, cada um diretamente no escopo. Felizmente, a instrução `import` tem uma variação de sintaxe que pode suportar esse estilo de consumo de módulo, chamado de *namespace import*.

Considere um módulo `"foo"` exportado como:

```js
export function bar() { .. }
export var x = 42;
export function baz() { .. }
```

Você pode importar toda essa API para um único binding de namespace de módulo:

```js
import * as foo from "foo";

foo.bar();
foo.x;			// 42
foo.baz();
```

**Nota:** A cláusula `* as ..` requer o wildcard `*`. Em outras palavras, você não pode fazer algo como `import { bar, x } as foo from "foo"` para trazer apenas parte da API mas ainda ligar ao namespace `foo`. Eu teria gostado de algo assim, mas para o ES6 é tudo ou nada com o namespace import.

Se o módulo que você está importando com `* as ..` tem um default export, ele é nomeado `default` no namespace especificado. Você pode adicionalmente nomear o import default fora do binding de namespace, como um identificador de nível superior. Considere um módulo `"world"` exportado como:

```js
export default function foo() { .. }
export function bar() { .. }
export function baz() { .. }
```

E este `import`:

```js
import foofn, * as hello from "world";

foofn();
hello.default();
hello.bar();
hello.baz();
```

Embora essa sintaxe seja válida, pode ser bastante confuso que um método do módulo (o default export) seja ligado no nível superior do seu escopo, enquanto o resto dos named exports (e um chamado `default`) são ligados como propriedades em um namespace de identificador nomeado de forma diferente (`hello`).

Como mencionei anteriormente, minha sugestão seria evitar projetar os exports do seu módulo dessa forma, para reduzir as chances de que os usuários do seu módulo sofram com essas peculiaridades estranhas.

Todos os bindings importados são imutáveis e/ou somente leitura. Considere o import anterior; todas estas tentativas de atribuição subsequentes lançarão `TypeError`s:

```js
import foofn, * as hello from "world";

foofn = 42;			// (runtime) TypeError!
hello.default = 42;	// (runtime) TypeError!
hello.bar = 42;		// (runtime) TypeError!
hello.baz = 42;		// (runtime) TypeError!
```

Lembre-se de que anteriormente na seção "`export`ando Membros da API" falamos sobre como os bindings `bar` e `baz` estão ligados aos identificadores reais dentro do módulo `"world"`. Isso significa que se o módulo mudar esses valores, `hello.bar` e `hello.baz` agora referenciam os valores atualizados.

Mas a natureza imutável/somente leitura dos seus bindings importados locais impõe que você não pode mudá-los a partir dos bindings importados, daí os `TypeError`s. Isso é bem importante, porque sem essas proteções, suas mudanças acabariam afetando todos os outros consumidores do módulo (lembre-se: singleton), o que poderia criar alguns efeitos colaterais muito surpreendentes!

Além disso, embora um módulo *possa* mudar seus membros de API por dentro, você deveria ser muito cauteloso ao projetar intencionalmente seus módulos dessa maneira. Módulos ES6 são *pretendidos* a serem estáticos, então desvios desse princípio deveriam ser raros e deveriam ser cuidadosa e verbosamente documentados.

**Aviso:** Existem filosofias de design de módulo onde você na verdade pretende deixar um consumidor mudar o valor de uma propriedade na sua API, ou APIs de módulo são projetadas para serem "estendidas" tendo outros "plug-ins" adicionando ao namespace da API. Como acabamos de afirmar, APIs de módulos ES6 deveriam ser pensadas e projetadas como estáticas e imutáveis, o que restringe e desencoraja fortemente esses padrões alternativos de design de módulo. Você pode contornar essas limitações exportando um objeto comum, que é claro pode então ser mudado à vontade. Mas tenha cuidado e pense duas vezes antes de seguir por esse caminho.

Declarações que ocorrem como resultado de um `import` são "hoisted" (veja o título *Scope & Closures* desta série). Considere:

```js
foo();

import { foo } from "foo";
```

`foo()` pode rodar porque não apenas a resolução estática da instrução `import ..` descobriu o que `foo` é durante a compilação, mas também "hoisted" a declaração para o topo do escopo do módulo, tornando-a assim disponível por todo o módulo.

Finalmente, a forma mais básica do `import` se parece com isto:

```js
import "foo";
```

Essa forma na verdade não importa nenhum dos bindings do módulo para o seu escopo. Ela carrega (se ainda não carregado), compila (se ainda não compilado) e avalia (se ainda não rodado) o módulo `"foo"`.

Em geral, esse tipo de import provavelmente não vai ser terrivelmente útil. Pode haver casos de nicho onde a definição de um módulo tem efeitos colaterais (como atribuir coisas ao objeto `window`/global). Você também poderia imaginar usar `import "foo"` como uma espécie de pré-carregamento para um módulo que pode ser necessário mais tarde.

### Dependência Circular de Módulos

A importa B. B importa A. Como isso realmente funciona?

Já declaro de cara que projetar sistemas com dependência circular intencional é geralmente algo que eu tento evitar. Dito isso, reconheço que há razões pelas quais pessoas fazem isso e que pode resolver algumas situações de design complicadas.

Vamos considerar como o ES6 trata isso. Primeiro, o módulo `"A"`:

```js
import bar from "B";

export default function foo(x) {
	if (x > 10) return bar( x - 1 );
	return x * 2;
}
```

Agora, o módulo `"B"`:

```js
import foo from "A";

export default function bar(y) {
	if (y > 5) return foo( y / 2 );
	return y * 3;
}
```

Essas duas funções, `foo(..)` e `bar(..)`, funcionariam como declarações de função padrão se estivessem no mesmo escopo, porque as declarações são "hoisted" para todo o escopo e, portanto, disponíveis umas para as outras independentemente da ordem de escrita.

Com módulos, você tem declarações em escopos inteiramente diferentes, então o ES6 tem que fazer trabalho extra para ajudar a fazer essas referências circulares funcionarem.

Em um sentido conceitual aproximado, é assim que dependências de `import` circulares são validadas e resolvidas:

* Se o módulo `"A"` for carregado primeiro, o primeiro passo é escanear o arquivo e analisar todos os exports, para que ele possa registrar todos esses bindings disponíveis para import. Então ele processa o `import .. from "B"`, que sinaliza que ele precisa ir buscar `"B"`.
* Uma vez que o motor carrega `"B"`, ele faz a mesma análise de seus bindings de export. Quando ele vê o `import .. from "A"`, ele já conhece a API de `"A"`, então pode verificar que o `import` é válido. Agora que ele conhece a API de `"B"`, ele também pode validar o `import .. from "B"` no módulo `"A"` que está aguardando.

Em essência, os imports mútuos, junto com a verificação estática que é feita para validar ambas as instruções `import`, virtualmente compõem os dois escopos de módulo separados (via os bindings), de modo que `foo(..)` pode chamar `bar(..)` e vice-versa. Isso é simétrico a se eles tivessem sido originalmente declarados no mesmo escopo.

Agora vamos tentar usar os dois módulos juntos. Primeiro, tentaremos `foo(..)`:

```js
import foo from "foo";
foo( 25 );				// 11
```

Ou podemos tentar `bar(..)`:

```js
import bar from "bar";
bar( 25 );				// 11.5
```

No momento em que as chamadas `foo(25)` ou `bar(25)` são executadas, toda a análise/compilação de todos os módulos foi concluída. Isso significa que `foo(..)` internamente sabe diretamente sobre `bar(..)` e `bar(..)` internamente sabe diretamente sobre `foo(..)`.

Se tudo o que precisamos é interagir com `foo(..)`, então precisamos apenas importar o módulo `"foo"`. Da mesma forma com `bar(..)` e o módulo `"bar"`.

É claro que *podemos* importar e usar ambos se quisermos:

```js
import foo from "foo";
import bar from "bar";

foo( 25 );				// 11
bar( 25 );				// 11.5
```

A semântica de carregamento estático da instrução `import` significa que um `"foo"` e um `"bar"` que dependem mutuamente um do outro via `import` garantirão que ambos sejam carregados, parseados e compilados antes de qualquer um deles rodar. Então a dependência circular deles é resolvida estaticamente e isso funciona como você esperaria.

### Carregamento de Módulos

Afirmamos no início desta seção "Módulos" que a instrução `import` usa um mecanismo separado, fornecido pelo ambiente hospedeiro (navegador, Node.js, etc.), para na verdade resolver a string do module specifier em alguma instrução útil para encontrar e carregar o módulo desejado. Esse mecanismo é o *Module Loader* do sistema.

O module loader padrão fornecido pelo ambiente interpretará um module specifier como uma URL se estiver no navegador, e (geralmente) como um caminho de sistema de arquivos local se estiver em um servidor como o Node.js. O comportamento padrão é assumir que o arquivo carregado foi escrito no formato de módulo padrão do ES6.

Além disso, você poderá carregar um módulo no navegador via uma tag HTML, similar a como programas de script atuais são carregados. No momento em que isto é escrito, não está totalmente claro se essa tag será `<script type="module">` ou `<module>`. O ES6 não controla essa decisão, mas discussões nos órgãos de padrões apropriados já estão bem avançadas em paralelo ao ES6.

Seja qual for a aparência da tag, você pode ter certeza de que por baixo dos panos ela usará o loader padrão (ou um customizado que você tenha pré-especificado, como discutiremos na próxima seção).

Assim como a tag que você usará na marcação, o próprio module loader não é especificado pelo ES6. É um padrão separado e paralelo (http://whatwg.github.io/loader/) controlado atualmente pelo grupo de padrões de navegadores WHATWG.

No momento em que isto é escrito, as discussões a seguir refletem uma passagem inicial do design da API, e as coisas provavelmente vão mudar.

#### Carregando Módulos Fora de Módulos

Um uso para interagir diretamente com o module loader é se um não-módulo precisa carregar um módulo. Considere:

```js
// script normal carregado no navegador via `<script>`,
// `import` é ilegal aqui

Reflect.Loader.import( "foo" ) // retorna uma promise para `"foo"`
.then( function(foo){
	foo.bar();
} );
```

O utilitário `Reflect.Loader.import(..)` importa o módulo inteiro para o parâmetro nomeado (como um namespace), assim como o namespace import `import * as foo ..` que discutimos anteriormente.

**Nota:** O utilitário `Reflect.Loader.import(..)` retorna uma promise que é cumprida uma vez que o módulo esteja pronto. Para importar múltiplos módulos, você pode compor promises de múltiplas chamadas de `Reflect.Loader.import(..)` usando `Promise.all([ .. ])`. Para mais informações sobre Promises, veja "Promises" no Capítulo 4.

Você também pode usar `Reflect.Loader.import(..)` em um módulo real para dinamicamente/condicionalmente carregar um módulo, onde o próprio `import` não funcionaria. Você poderia, por exemplo, escolher carregar um módulo contendo um polyfill para alguma funcionalidade ES7+ se um teste de funcionalidade revelar que ela não está definida pelo motor atual.

Por razões de performance, você vai querer evitar carregamento dinâmico sempre que possível, já que ele prejudica a capacidade do motor JS de disparar buscas antecipadas a partir de sua análise estática.

#### Carregamento Customizado

Outro uso para interagir diretamente com o module loader é se você quiser customizar seu comportamento através de configuração ou até mesmo redefinição.

No momento em que isto é escrito, há um polyfill para a API do module loader sendo desenvolvido (https://github.com/ModuleLoader/es6-module-loader). Embora os detalhes sejam escassos e altamente sujeitos a mudança, podemos explorar quais possibilidades podem eventualmente surgir.

A chamada `Reflect.Loader.import(..)` pode suportar um segundo argumento para especificar várias opções para customizar a tarefa de import/load. Por exemplo:

```js
Reflect.Loader.import( "foo", { address: "/path/to/foo.js" } )
.then( function(foo){
	// ..
} )
```

Também se espera que uma customização seja fornecida (por algum meio) para se conectar ao processo de carregamento de um módulo, onde uma tradução/transpilação poderia ocorrer após o carregamento, mas antes de o motor compilar o módulo.

Por exemplo, você poderia carregar algo que já não é um formato de módulo compatível com ES6 (por exemplo, CoffeeScript, TypeScript, CommonJS, AMD). Seu passo de tradução poderia então convertê-lo para um módulo compatível com ES6 para o motor então processar.

## Classes

Quase desde o início do JavaScript, sintaxe e padrões de desenvolvimento têm todos se esforçado (leia-se: lutado) para colocar uma fachada de suporte a desenvolvimento orientado a classes. Com coisas como `new` e `instanceof` e uma propriedade `.constructor`, quem não poderia deixar de ser provocado de que o JS tinha classes escondidas em algum lugar dentro de seu sistema de protótipos?

É claro que as "classes" do JS não são nem de longe as mesmas que as classes clássicas. As diferenças são bem documentadas, então não vou me alongar mais nesse ponto aqui.

**Nota:** Para aprender mais sobre os padrões usados em JS para fingir "classes", e uma visão alternativa de protótipos chamada "delegação", veja a segunda metade do título *this & Object Prototypes* desta série.

### `class`

Embora o mecanismo de protótipos do JS não funcione como classes tradicionais, isso não impede a forte onda de demanda na linguagem para estender o açúcar sintático de modo que expressar "classes" se pareça mais com classes reais. Entra a palavra-chave `class` do ES6 e seu mecanismo associado.

Essa funcionalidade é o resultado de um debate altamente controverso e prolongado, e representa um subconjunto de compromisso menor de várias visões fortemente opostas sobre como abordar classes em JS. A maioria dos desenvolvedores que querem classes completas em JS achará partes da nova sintaxe bastante convidativas, mas achará que partes importantes ainda estão faltando. Não se preocupe, porém. O TC39 já está trabalhando em funcionalidades adicionais para aumentar classes no período pós-ES6.

No cerne do novo mecanismo de classe do ES6 está a palavra-chave `class`, que identifica um *bloco* cujo conteúdo define os membros do protótipo de uma função. Considere:

```js
class Foo {
	constructor(a,b) {
		this.x = a;
		this.y = b;
	}

	gimmeXY() {
		return this.x * this.y;
	}
}
```

Algumas coisas a observar:

* `class Foo` implica criar uma função (especial) com o nome `Foo`, muito parecido com o que você fazia pré-ES6.
* `constructor(..)` identifica a assinatura dessa função `Foo(..)`, bem como o conteúdo de seu corpo.
* Os métodos de classe usam a mesma sintaxe de "concise method" disponível para literais de objeto, como discutido no Capítulo 2. Isso também inclui a forma de gerador conciso como discutido anteriormente neste capítulo, bem como a sintaxe de getter/setter do ES5. Entretanto, métodos de classe são não-enumeráveis, enquanto métodos de objeto são por padrão enumeráveis.
* Diferentemente de literais de objeto, não há vírgulas separando membros em um corpo de `class`! De fato, elas nem mesmo são permitidas.

A definição de sintaxe `class` no fragmento anterior pode ser pensada grosseiramente como este equivalente pré-ES6, que provavelmente parecerá bastante familiar para aqueles que já fizeram codificação em estilo de protótipo antes:

```js
function Foo(a,b) {
	this.x = a;
	this.y = b;
}

Foo.prototype.gimmeXY = function() {
	return this.x * this.y;
}
```

Tanto na forma pré-ES6 quanto na nova forma `class` do ES6, essa "classe" pode agora ser instanciada e usada exatamente como você esperaria:

```js
var f = new Foo( 5, 15 );

f.x;						// 5
f.y;						// 15
f.gimmeXY();				// 75
```

Cuidado! Embora `class Foo` se pareça muito com `function Foo()`, há diferenças importantes:

* Uma chamada `Foo(..)` de `class Foo` *deve* ser feita com `new`, já que a opção pré-ES6 de `Foo.call( obj )` *não* funcionará.
* Enquanto `function Foo` é "hoisted" (veja o título *Scope & Closures* desta série), `class Foo` não é; a cláusula `extends ..` especifica uma expressão que não pode ser "hoisted". Então, você deve declarar uma `class` antes de poder instanciá-la.
* `class Foo` no escopo global de nível superior cria um identificador léxico `Foo` nesse escopo, mas, diferentemente de `function Foo`, não cria uma propriedade do objeto global com esse nome.

O operador `instanceof` estabelecido ainda funciona com classes ES6, porque `class` apenas cria uma função construtora de mesmo nome. Entretanto, o ES6 introduz uma maneira de customizar como `instanceof` funciona, usando `Symbol.hasInstance` (veja "Well-Known Symbols" no Capítulo 7).

Outra maneira de pensar sobre `class`, que eu acho mais conveniente, é como uma *macro* que é usada para popular automaticamente um objeto `prototype`. Opcionalmente, ela também conecta o relacionamento `[[Prototype]]` se usar `extends` (veja a próxima seção).

Uma `class` do ES6 não é realmente uma entidade em si, mas um meta conceito que envolve outras entidades concretas, como funções e propriedades, e as amarra juntas.

**Dica:** Além da forma de declaração, uma `class` também pode ser uma expressão, como em: `var x = class Y { .. }`. Isso é primariamente útil para passar uma definição de classe (tecnicamente, o próprio construtor) como um argumento de função ou atribuí-la a uma propriedade de objeto.

### `extends` e `super`

Classes ES6 também têm açúcar sintático para estabelecer o link de delegação `[[Prototype]]` entre dois protótipos de função -- comumente rotulado erroneamente de "herança" ou confusamente rotulado de "herança de protótipo" -- usando a terminologia familiar orientada a classes `extends`:

```js
class Bar extends Foo {
	constructor(a,b,c) {
		super( a, b );
		this.z = c;
	}

	gimmeXYZ() {
		return super.gimmeXY() * this.z;
	}
}

var b = new Bar( 5, 15, 25 );

b.x;						// 5
b.y;						// 15
b.z;						// 25
b.gimmeXYZ();				// 1875
```

Uma adição nova significativa é `super`, que é na verdade algo não diretamente possível pré-ES6 (sem algumas trocas de hack infelizes). No construtor, `super` automaticamente refere-se ao "construtor pai", que no exemplo anterior é `Foo(..)`. Em um método, ele refere-se ao "objeto pai", de modo que você pode então fazer um acesso a propriedade/método a partir dele, como `super.gimmeXY()`.

`Bar extends Foo` é claro significa ligar o `[[Prototype]]` de `Bar.prototype` ao `Foo.prototype`. Então, `super` em um método como `gimmeXYZ()` significa especificamente `Foo.prototype`, enquanto `super` significa `Foo` quando usado no construtor de `Bar`.

**Nota:** `super` não é limitado a declarações de `class`. Ele também funciona em literais de objeto, da mesma maneira que estamos discutindo aqui. Veja "Object `super`" no Capítulo 2 para mais informações.

#### Aqui Há Dragões de `super`

Não é insignificante observar que `super` se comporta diferentemente dependendo de onde ele aparece. Para ser justo, na maioria das vezes, isso não será um problema. Mas surpresas aguardam se você se desviar de uma norma estreita.

Pode haver casos onde no construtor você queira referenciar o `Foo.prototype`, como para acessar diretamente uma de suas propriedades/métodos. Entretanto, `super` no construtor não pode ser usado dessa forma; `super.prototype` não funcionará. `super(..)` significa aproximadamente chamar `new Foo(..)`, mas não é na verdade uma referência usável ao próprio `Foo`.

Simetricamente, você pode querer referenciar a função `Foo(..)` de dentro de um método não-construtor. `super.constructor` apontará para a função `Foo(..)`, mas cuidado que essa função *só* pode ser invocada com `new`. `new super.constructor(..)` seria válido, mas não seria terrivelmente útil na maioria dos casos, porque você não pode fazer essa chamada usar ou referenciar o contexto do objeto `this` atual, que é provavelmente o que você gostaria.

Além disso, `super` parece que poderia ser conduzido pelo contexto de uma função assim como `this` -- ou seja, que ambos seriam ligados dinamicamente. Entretanto, `super` não é dinâmico como `this` é. Quando um construtor ou método faz uma referência a `super` dentro dele em tempo de declaração (no corpo da `class`), esse `super` é estaticamente ligado àquela hierarquia de classe específica, e não pode ser sobrescrito (ao menos no ES6).

O que isso significa? Significa que se você tem o hábito de pegar um método de uma "classe" e "emprestá-lo" para outra classe sobrescrevendo seu `this`, digamos com `call(..)` ou `apply(..)`, isso pode muito bem criar surpresas se o método que você está emprestando tem um `super` nele. Considere esta hierarquia de classe:

```js
class ParentA {
	constructor() { this.id = "a"; }
	foo() { console.log( "ParentA:", this.id ); }
}

class ParentB {
	constructor() { this.id = "b"; }
	foo() { console.log( "ParentB:", this.id ); }
}

class ChildA extends ParentA {
	foo() {
		super.foo();
		console.log( "ChildA:", this.id );
	}
}

class ChildB extends ParentB {
	foo() {
		super.foo();
		console.log( "ChildB:", this.id );
	}
}

var a = new ChildA();
a.foo();					// ParentA: a
							// ChildA: a
var b = new ChildB();		// ParentB: b
b.foo();					// ChildB: b
```

Tudo parece bastante natural e esperado neste fragmento anterior. Entretanto, se você tentar emprestar `b.foo()` e usá-lo no contexto de `a` -- em virtude da ligação dinâmica de `this`, tal empréstimo é bastante comum e usado de muitas maneiras diferentes, incluindo mixins notavelmente -- você pode achar esse resultado uma surpresa feia:

```js
// empresta `b.foo()` para usar no contexto de `a`
b.foo.call( a );			// ParentB: a
							// ChildB: a
```

Como você pode ver, a referência `this.id` foi religada dinamicamente de modo que `: a` é reportado em ambos os casos em vez de `: b`. Mas a referência `super.foo()` de `b.foo()` não foi religada dinamicamente, então ela ainda reportou `ParentB` em vez do esperado `ParentA`.

Como `b.foo()` referencia `super`, ele é estaticamente ligado à hierarquia `ChildB`/`ParentB` e não pode ser usado contra a hierarquia `ChildA`/`ParentA`. Não há solução ES6 para essa limitação.

`super` parece funcionar intuitivamente se você tem uma hierarquia de classe estática sem polinização cruzada. Mas, com toda justiça, um dos principais benefícios de fazer codificação ciente de `this` é exatamente esse tipo de flexibilidade. Simplesmente, `class` + `super` exige que você evite tais técnicas.

A escolha se resume a estreitar o design do seu objeto a essas hierarquias estáticas -- `class`, `extends` e `super` serão bastante agradáveis -- ou abandonar todas as tentativas de "fingir" classes e em vez disso abraçar objetos dinâmicos, flexíveis e sem classes e a delegação `[[Prototype]]` (veja o título *this & Object Prototypes* desta série).

#### Construtor de Subclasse

Construtores não são obrigatórios para classes ou subclasses; um construtor padrão é substituído em ambos os casos se omitido. Entretanto, o construtor padrão substituído é diferente para uma classe direta versus uma classe estendida.

Especificamente, o construtor de subclasse padrão automaticamente chama o construtor pai, e repassa quaisquer argumentos. Em outras palavras, você poderia pensar no construtor de subclasse padrão mais ou menos assim:

```js
constructor(...args) {
	super(...args);
}
```

Esse é um detalhe importante a observar. Nem todas as linguagens de classe têm o construtor de subclasse chamando automaticamente o construtor pai. C++ tem, mas Java não. Mas, mais importante, em classes pré-ES6, tal chamada automática de "construtor pai" não acontece. Tenha cuidado ao converter para `class` do ES6 se você vinha confiando em tais chamadas *não* acontecerem.

Outro desvio/limitação talvez surpreendente dos construtores de subclasse do ES6: em um construtor de uma subclasse, você não pode acessar `this` até que `super(..)` tenha sido chamado. A razão é sutil e complicada, mas se resume ao fato de que o construtor pai é na verdade o único que cria/inicializa o `this` da sua instância. Pré-ES6, funciona de forma oposta; o objeto `this` é criado pelo "construtor de subclasse", e então você chama um "construtor pai" com o contexto do `this` da "subclasse".

Vamos ilustrar. Isso funciona pré-ES6:

```js
function Foo() {
	this.a = 1;
}

function Bar() {
	this.b = 2;
	Foo.call( this );
}

// `Bar` "estende" `Foo`
Bar.prototype = Object.create( Foo.prototype );
```

Mas este equivalente ES6 não é permitido:

```js
class Foo {
	constructor() { this.a = 1; }
}

class Bar extends Foo {
	constructor() {
		this.b = 2;			// não permitido antes de `super()`
		super();			// para consertar troque estas duas declarações
	}
}
```

Nesse caso, a correção é simples. Apenas troque as duas instruções no construtor da subclasse `Bar`. Entretanto, se você vinha confiando pré-ES6 em poder pular a chamada do "construtor pai", cuidado porque isso não será mais permitido.

#### `extend`endo Nativos

Um dos benefícios mais aclamados do novo design de `class` e `extend` é a capacidade de (finalmente!) fazer subclasse dos nativos embutidos, como `Array`. Considere:

```js
class MyCoolArray extends Array {
	first() { return this[0]; }
	last() { return this[this.length - 1]; }
}

var a = new MyCoolArray( 1, 2, 3 );

a.length;					// 3
a;							// [1,2,3]

a.first();					// 1
a.last();					// 3
```

Antes do ES6, uma falsa "subclasse" de `Array` usando criação manual de objeto e ligação ao `Array.prototype` só funcionava parcialmente. Ela perdia os comportamentos especiais de um array real, como a propriedade `length` que atualiza automaticamente. Subclasses do ES6 devem funcionar totalmente com comportamentos "herdados" e aumentados como esperado!

Outra limitação comum de "subclasse" pré-ES6 é com o objeto `Error`, ao criar "subclasses" de erro customizadas. Quando objetos `Error` genuínos são criados, eles automaticamente capturam informação especial de `stack`, incluindo o número da linha e o arquivo onde o erro é criado. "Subclasses" de erro customizadas pré-ES6 não têm tal comportamento especial, o que limita severamente sua utilidade.

ES6 ao resgate:

```js
class Oops extends Error {
	constructor(reason) {
		super(reason);
		this.oops = reason;
	}
}

// mais tarde:
var ouch = new Oops( "I messed up!" );
throw ouch;
```

O objeto de erro customizado `ouch` neste fragmento anterior se comportará como qualquer outro objeto de erro genuíno, incluindo capturar `stack`. Isso é uma grande melhoria!

### `new.target`

ES6 introduz um novo conceito chamado *meta property* (veja o Capítulo 7), na forma de `new.target`.

Se isso parece estranho, é mesmo; emparelhar uma palavra-chave com um `.` e um nome de propriedade é definitivamente um padrão fora do comum para JS.

`new.target` é um novo valor "mágico" disponível em todas as funções, embora em funções normais ele sempre seja `undefined`. Em qualquer construtor, `new.target` sempre aponta para o construtor que o `new` na verdade diretamente invocou, mesmo que o construtor esteja em uma classe pai e tenha sido delegado por uma chamada `super(..)` de um construtor filho. Considere:

```js
class Foo {
	constructor() {
		console.log( "Foo: ", new.target.name );
	}
}

class Bar extends Foo {
	constructor() {
		super();
		console.log( "Bar: ", new.target.name );
	}
	baz() {
		console.log( "baz: ", new.target );
	}
}

var a = new Foo();
// Foo: Foo

var b = new Bar();
// Foo: Bar   <-- respeita o call-site do `new`
// Bar: Bar

b.baz();
// baz: undefined
```

A meta property `new.target` não tem muito propósito em construtores de classe, exceto acessar uma propriedade/método estático (veja a próxima seção).

Se `new.target` é `undefined`, você sabe que a função não foi chamada com `new`. Você pode então forçar uma invocação com `new` se isso for necessário.

### `static`

Quando uma subclasse `Bar` estende uma classe pai `Foo`, já observamos que `Bar.prototype` é ligado por `[[Prototype]]` a `Foo.prototype`. Mas adicionalmente, `Bar()` é ligado por `[[Prototype]]` a `Foo()`. Essa parte pode não ter um raciocínio tão óbvio.

Entretanto, é bem útil no caso em que você declara métodos `static` (não apenas propriedades) para uma classe, já que esses são adicionados diretamente ao objeto função dessa classe, não ao objeto `prototype` do objeto função. Considere:

```js
class Foo {
	static cool() { console.log( "cool" ); }
	wow() { console.log( "wow" ); }
}

class Bar extends Foo {
	static awesome() {
		super.cool();
		console.log( "awesome" );
	}
	neat() {
		super.wow();
		console.log( "neat" );
	}
}

Foo.cool();					// "cool"
Bar.cool();					// "cool"
Bar.awesome();				// "cool"
							// "awesome"

var b = new Bar();
b.neat();					// "wow"
							// "neat"

b.awesome;					// undefined
b.cool;						// undefined
```

Tenha cuidado para não se confundir achando que membros `static` estão na cadeia de protótipos da classe. Eles estão na verdade na cadeia dupla/paralela entre os construtores de função.

#### Getter de Construtor `Symbol.species`

Um lugar onde `static` pode ser útil é em definir o getter `Symbol.species` (conhecido internamente na especificação como `@@species`) para uma classe derivada (filha). Essa capacidade permite que uma classe filha sinalize a uma classe pai qual construtor deveria ser usado -- quando não se pretende usar o próprio construtor da classe filha -- se algum método da classe pai precisar fornecer uma nova instância.

Por exemplo, muitos métodos em `Array` criam e retornam uma nova instância de `Array`. Se você define uma classe derivada de `Array`, mas quer que esses métodos continuem a fornecer instâncias reais de `Array` em vez de da sua classe derivada, isso funciona:

```js
class MyCoolArray extends Array {
	// força `species` a ser o construtor pai
	static get [Symbol.species]() { return Array; }
}

var a = new MyCoolArray( 1, 2, 3 ),
	b = a.map( function(v){ return v * 2; } );

b instanceof MyCoolArray;	// false
b instanceof Array;			// true
```

Para ilustrar como um método de classe pai pode usar a declaração de species de um filho mais ou menos como o `Array#map(..)` está fazendo, considere:

```js
class Foo {
	// adia `species` para o construtor derivado
	static get [Symbol.species]() { return this; }
	spawn() {
		return new this.constructor[Symbol.species]();
	}
}

class Bar extends Foo {
	// força `species` a ser o construtor pai
	static get [Symbol.species]() { return Foo; }
}

var a = new Foo();
var b = a.spawn();
b instanceof Foo;					// true

var x = new Bar();
var y = x.spawn();
y instanceof Bar;					// false
y instanceof Foo;					// true
```

O `Symbol.species` da classe pai faz `return this` para adiar a qualquer classe derivada, como você normalmente esperaria. `Bar` então sobrescreve para manualmente declarar que `Foo` deve ser usado para tal criação de instância. É claro que uma classe derivada ainda pode fornecer instâncias de si mesma usando `new this.constructor(..)`.

## Revisão

ES6 introduz várias novas funcionalidades que ajudam na organização de código:

* Iteradores fornecem acesso sequencial a dados ou operações. Eles podem ser consumidos por novas funcionalidades da linguagem como `for..of` e `...`.
* Geradores são funções localmente capazes de pausar/retomar controladas por um iterador. Eles podem ser usados para programaticamente (e interativamente, através da passagem de mensagens `yield`/`next(..)`) *gerar* valores a serem consumidos via iteração.
* Módulos permitem encapsulamento privado de detalhes de implementação com uma API exportada publicamente. Definições de módulo são baseadas em arquivo, instâncias singleton, e resolvidas estaticamente em tempo de compilação.
* Classes fornecem uma sintaxe mais limpa em torno da codificação baseada em protótipos. A adição de `super` também resolve questões complicadas com referências relativas na cadeia `[[Prototype]]`.

Essas novas ferramentas deveriam ser sua primeira parada ao tentar melhorar a arquitetura dos seus projetos JS abraçando o ES6.
