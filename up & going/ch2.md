# You Don't Know JS: Iniciando
# Chapter 2: Por dentro do JavaScript

No capítulo anterior, fiz uma introdução básica sobre os blocos construtores da programação, como variáveis, loops, condicionais e funções. Claro, todo o código demonstrado foi em JavaScript. Mas neste capítulo, iremos focar especificamente no que deve ser aprendido em JavaScript para começarmos como um desenvolvedor JS.

Iremos introduzir alguns conceitos neste capítulo que não serão totalmente explorados até a sequência dos próximos livros desta série. Você pode pensar nesse capítulo como uma visão geral dos tópicos que serão abordados ao longo dos outros livros.

Especialmente se você for novo ao JavaScript, você pode esperar utilizar boa parte do seu tempo revisando por diversas vezes os conceitos e exemplos de código abordados. Toda boa fundação é feita de tijolo em tijolo, então não espere que você entenderá de imediato conforme for progredindo na leitura.

Sua jornada para entender à fundo JavaScript começa aqui.

**Nota:** Como havia dito no Capítulo 1, você deve definitivamente testar por conta própria todos os códigos apresentados enquanto você estiver lendo através do capítulo. Tome nota que alguns dos códigos escritos aqui utilizam capacidades introduzidas na nova versão do  JavaScript no momento em que estou escrevendo  (comumente chamado de "ES6" por ser a 6ª edição do ECMAScript -- o nome oficial da especificação JS). Caso aconteça de você estar utilizando um navegador antigo, pre-ES6, o código pode não funcionar. Uma versão atualizada de um navegador moderno (como Chrome, Firefox, or IE) deverá ser usada.

## Valores e Tipos

Como definimos no Capítulo 1, JavaScript tem valores tipados, não variáveis tipadas. Os seguintes tipos nativos estão disponíveis:

* `string`
* `number`
* `boolean`
* `null` e `undefined`
* `object`
* `symbol` (novidade do ES6)

O JavaScript dispõe de um operador `typeof`que pode examinar um valor e dizer a você qual é o tipo informado:

```js
var a;
typeof a;				// "undefined"

a = "hello world";
typeof a;				// "string"

a = 42;
typeof a;				// "number"

a = true;
typeof a;				// "boolean"

a = null;
typeof a;				// "object" -- weird, bug

a = undefined;
typeof a;				// "undefined"

a = { b: "c" };
typeof a;				// "object"
```

O valor que é retornado pelo operador `typeof`é sempre um dos seis (sete com o ES6!) valores em string. Isso é, `typeof "abc"` retorna `"string"`, não `string`.

Note como nesse snippet a variável `a` contém cada tipo diferente tipo de valor, e apesar de parecer, `typeof a` não está perguntando pelo "tipo de `a`", mas sim pelo "tipo de valor atualmente armazenado em `a`." Apenas valores possuem tipos em JavaScript; variáveis são apenas _containers_ para esses valores.

`typeof null` é um caso interessante, porque erradamente retorna um `"objeto"`, enquanto você espera que ele reetorne `"null"`.

**Atenção:** Esse é um bug antigo em JS, mas um do tipo que é provável de nunca ser consertado. Muitos códigos na Web dependem desse bug e portanto consertá-lo iria trazer ainda mais bugs!

Além disso, note que `a = undefined`. Nós explicitamente indicamos `a` para o valor `undefined`, mas a forma com que se comporta não é diferente de uma variável que não tem valor definido, como a linha `var a;`no topo do snippet. Uma variável pode chegar a esse valor "undefined" de diversas maneiras, incluíndo funções que não retornam valores e o uso do operador`void`.

### Objetos

O tipo `object` se refere a um valor composto onde você pode definir propriedades (lugares nomeados prontos para armazenar informação) que podem armazenar seus próprios valores de qualquer tipo. Esse é talvez um dos tipos de valor mais úteis em todo JavaScript:


```js
var obj = {
	a: "hello world",
	b: 42,
	c: true
};

obj.a;		// "hello world"
obj.b;		// 42
obj.c;		// true

obj["a"];	// "hello world"
obj["b"];	// 42
obj["c"];	// true
```

Talvez seja útil pensar nesse valor `obj` visualmente:

<img src="fig4.png">

Propriedades podem ser acessadas tanto com *notação com ponto* (_dot notation_, ex: `obj.a`) quanto por notação em colchetes (ex: `obj["a"]`). A notação por ponto é menor e geralmente mais fácil de ser lida, e por isso é a notação preferida, sempre que possível.

A notação em colchetes é útil caso você tenha um nome de propriedade que conténha caracteres especiais nele, como `obj["hello world!"]` -- esses tipos de propriedades são geralmente referenciadas com chaves (*keys*) quando acessadas por notação em colchetes. A notação `[ ]`  requer ou uma variável (explicarei a seguir) ou uma `string` *literal* (que precisa ser englobada em `" .. "` ou `' .. '`).

É claro, a notação em colchetes também é útil se você quiser acessar uma propriedade/chave onde o nome é armazenado dentro de outra variável, como por exemplo:

```js
var obj = {
	a: "hello world",
	b: 42
};

var b = "a";

obj[b];			// "hello world"
obj["b"];		// 42
```

**Nota:** Para mais informação sobre objetos (`objects`) em JavaScrip, veja o título desta série *this & Prototipagem de Objetos*, especificamente o Capítulo 3.

Existem outros tipos de valores que você pode facilmente interagir com programas em JavaScript: *array* e *function*. Mas ao invés de serem tipos nativos (built-in), eles devem ser vistos mais como sub-tipos -- versões especializadas do tipo `object`.

#### Arrays

Uma array é um `object` que armazena valores (qualquer tipo de valor), não necesariamente em propriedades ou chaves nomeadas, mas em posições enumeradas. Por exemplo:

```js
var arr = [
	"hello world",
	42,
	true
];

arr[0];			// "hello world"
arr[1];			// 42
arr[2];			// true
arr.length;		// 3

typeof arr;		// "object"
```

**Nota:** Linguagens que começam contando do zero, como JS faz, usam `0` como índice do primeiro elemento da array.

Talvez seja útil pensar sobre `arr` visualmente:

<img src="fig5.png">

Visto que arrays são objetos especiais (assim como `typeof` sugere), elas também podem conter propriedades, incluindo a propriedade `length` que é automaticamente atualizada.

Em teoria, você pode usar uma array como um objeto normal com suas próprias propriedades nomeadas, ou pode usar um `object` e apenas determinar propriedades numéricas (`0`, `1`, etc.) de maneira similar a uma array. Entretanto, isso geralmente é considerado uma maneira indevida de utilizar seus respectivos tipos.

A melhor maneira (e a mais natural) é utilizar arrays para valores posicionados numericamente e usar `object`s para propriedades nomeadas.

#### Funções

O outro subtipo de `object` que você usará ao longo de seus programas é uma função:

```js
function foo() {
	return 42;
}

foo.bar = "hello world";

typeof foo;			// "function"
typeof foo();		// "number"
typeof foo.bar;		// "string"
```

Novamente, funções são um subtipo de `objects` -- o `typeof` retorna `"function"`, que indica que `function` é um tipo padrão -- e por isso pode ter propriedades. Entretanto, é provável que você use as propriedades do objeto de `function` (como `foo.bar`) apenas em alguns casos.

**Nota:** Para mais informações sobre valores em JS e seus tipos, veja os primeiros dois capítulos do título *Tipos & Gramática*, desta série.

### Métodos de Tipos Nativos

Os tipos e subtipos nativos que acabamos de ver tem alguns comportamentos bem úteis, expostos como propriedades e métodos.

Por exemplo:

```js
var a = "hello world";
var b = 3.14159;

a.length;				// 11
a.toUpperCase();		// "HELLO WORLD"
b.toFixed(4);			// "3.1416"
```

é mais complicado do que apenas o método existente em um valor.

Resumidamente, existe uma forma agregadora do objeto `String` (`S` maiúsculo), que pareia com o tipo primitivo `string`. É esse objeto agregador que define o método `toUpperCase()` em seu protótipo.

Quando você usa um valor primitivo, como "hello world", como um `object` referenciando a propriedade ou método (exemplo `a.toUpperCase()` no snippet anterior), o JS automaticamente "encaixota" o valor na contraparte do agregador de seu objeto (escondido).

Um valor `string` pode ser englobado por um objeto `String`, um `number` pode ser englobado por um objeto `Number`, e um `boolean` pode ser englobado por um objeto `Boolean`. Para a maioria dos casos, você não precisa se preocupar sobre isso ou usar diretamente essas formas de agregar os valores do objeto -- preferindo a forma de valores primitivos em todos os casos que puder e o JavaScript vai cuidar do resto pra você.

**Nota:** Para mais iformações em nativos em JS e formas de "encaixotar", veja o Capítulo 3 do título deste livro *Tipos e Gramática*. Para melhor entendimento dos protótipos de um objeto, veja o Capítulo 5 do título *this & Object Prototypes*.

### Comparando Valores

Existem dois tipos principais de comparação de valores que você irá preccisar para fazer seus programas em JS: *igualdade* and *desigualdade*. O resultado de qualquer comparação é estritmente um valor `boolean` (`true` ou `false`), independente do tipo de valor comparado.

#### Coerção

Falamos brevemente sobre coerção no Capítulo 1, mas vamos revisitá-lo aqui.

A coerção vem em duas formas em JavaScript: *explicita* e *implicita*. A coerção explícita é a forma que você pode, obviamente, através do código, que uma conversão de um tipo para o outro vai acontecer, e a coerção implícita é quando o tipo de conversão ocorre como um efeito paralelo, não tão óbvio, de alguma outra operação.

Você provavelmente ouviu coisas como "coerção é do mau", por conta da surpresa nos resultados que algumas situações específicas podem causar. Talvez nenhuma outra situação frustre mais um desenvolvedor do que quando a linguagem o surpreende.

Coerções não são do mau, nem mesmo devem ser surpreendentes. De fato, a maioria dos casos que você pode construir com a corção de tipos são bem sensíveis e entendíveis, e podem até mesmo serem usados como maneira de *melhorar* a legibilidade do código. Mas não iremos entrar muito nesse debate -- O Capítulo 4 do título *Tipos e Gramática* desta série cobre bem essa parte.

Aqui temos um exemplo de coerção *explícita*:

```js
var a = "42";

var b = Number( a );

a;				// "42"
b;				// 42 -- o número!
```

E aqui um exemplo de coerção *implícita*:

```js
var a = "42";

var b = a * 1;	// "42" implicitamente coergido para 42 aqui

a;				// "42"
b;				// 42 -- o número!
```

#### Truthy & Falsy

No Capítulo 1, nós mencionamos a natureza "truthy" e "falsy" dos valores: quando um valor não-`boolean` é coergido para um valor `boolean`, eke se tornam `true` ou `false`, respectivamente?

A lista de valores "falsy" em JavaScript é a seguinte:

* `""` (string vazia)
* `0`, `-0`, `NaN` (`number` inválido)
* `null`, `undefined`
* `false`

Qualquer valor que não esteja nessa lista de "falsy", é considerado "truthy". Aqui estão alguns exemplos deles:

* `"hello"`
* `42`
* `true`
* `[ ]`, `[ 1, "2", 3 ]` (arrays)
* `{ }`, `{ a: 42 }` (objects)
* `function foo() { .. }` (functions)

É importante lembrar que um valor não-`boolean` segue a coerção como "truthy"/"falsy" apenas se ele for coergido para `boolean`. Não é difícil se confundir com uma situação onde parece que estamos coergindo um valor para um `boolean` quando na verdade não está.

#### Igualdade

Existem quatro operadores de igualdade: `==`, `===`, `!=`, e `!==`. A forma `!` é a versão simétrica de "não igual" de suas contrapartes; *não-igualdade* não deve ser confundido com  *desigualdade*.

A diferença entre `==` e `===` é geralmente caracterizada por `==` verificar a igualdade de valores e `===` verificar a igualdade do valor e do tipo. Entretanto, essa forma não é a mais apurada. A maneira correta de caracterizá-los é que `==` verifica  por igualdade com coerção autorizada, e `===` verifica a igualdade do valor sem autorizar a coerção; `===` é comumente chamada de "igualdade estrita" por essa razão.

Considere a coerção implícita que é autorizada pelo comparador de igualdade `==` e não permitido com a igualdade estrita `===`:

```js
var a = "42";
var b = 42;

a == b;			// true
a === b;		// false
```

Na comparação `a == b`, o JS percebe que os tipos não combinam, então ele segue uma sequência de etapas para coergir um ou ambos os valores para um tipo diferente até que os tipos combinem, de forma que um valor de igualdade simples possa ser considerado.

Se você pensar sobre isso, não existem dois modos possíveis onde `a == b` possa dar `true` por coerção. Ou a comparação por se dar por `42 == 42` ou ela pode ser `"42" == "42"`. Sendo assim, qual das duas é a correta?

Resposta: `"42"` se torna `42`, para fazer a comparação `42 == 42`. Nesse exemplo simples, não parece importante saber qual processo será, no final o resultado é o mesmo. Existem casos mais complexos onde não apenas importa qual é o resultado final como *como* foi possível chegar lá.

A igualdade `a === b` produz um resultado `false`, porque a coerção não é permitida, assim obviamente a comparação falha. Muitos desenvolvedores pregam que `===` é mais previsível, permanecendo usando sempre esta forma e ficando longe de `==`. Acho esse ponto de vista limitado. Acredito que `==` é uma ferramenta poderosa que pode ajudar você em seus programas,  *se você se dedicar a aprender como ele funciona.*

Não vamos nos aprofundar nos detalhes de como a coerção em comparações com `==` funciona. Muito sobre ele é bem intuitivo, mas existem casos específicos importantes de se tomar nota. Você pode ler a seção 11.9.3 da especificação do ES5 (http://www.ecma-international.org/ecma-262/5.1/) para ver suas regras exatas, e você ficará surpreso em como seu mecanismo é bem desenvolvido, comparado a toda hype negativa à sua volta.

Para resumir um monte de detalhes em passos bem simples e ajudar você a decidir sobre usar `==` ou `===` em várias situações, aqui vão minhas regras simples:

* Se em um dos lados da comparação você puder ter um valor `true` ou `false`, evite `==` e use `===`.
* Se em um dos lados da comparação você puder ter esses valores específicos (`0`, `""`, ou `[]` -- array vazia), evite `==` e use `===`.
* Em *todos* os outros casos, você estará seguro usando `==`. Não apenas é mais seguro, mas em muitos casos simplifica seu código de forma a melhorar sua leitura.

O que essas regras fazem é obrigar você a pensar criticamente sobre seu código e quais tipos de valor podem aparecer através de variáveis que são comparadas pela igualdade. Se você estiver certo desses valores, e `==` é seguro, use-o! Se você não pode estar certo dos resultados, use `===`. É simples assim.

A não igualdade `!=`forma um par com `==`, e sua forma `!==` forma um par com `===`. Todas as regras e observações que discutimos até aqui funcionam de maneira simétrica para essas comparações de não-igualdade.

Você deve ter uma atenção especial sobre as regras de comparação de `==` e `===` se você estiver comparando dois valores não-primitivos, como `object`s (incluíndo `function` e `array`). Por estes valores serem regidos por suas referências, ambas as comparações `==` e `===` irão apenas verificar se suas referências são compatíveis, e não irá comparar nada sobre seus valores subjacentes.

Por exemplo, `array`s são por padrão convertidas para`string`s por simplesmente se juntarem todos os valores com vírgulas (`,`) entree elas. Você pode pensar que duas `array`s com o mesmo conteúdo são iguais `==`, quando na verdade não são:

```js
var a = [1,2,3];
var b = [1,2,3];
var c = "1,2,3";

a == c;		// true
b == c;		// true
a == b;		// false
```

**Nota:** Para mais informações sobre as regras de comparação de igualdade `==`, veja a especificação do ES5 (seção 11.9.3) e também consulte o Capítulo  4 do título desta série *Tipos & Gramática*; veja o Capítulo 2 sobre mais informações sobre valores versus referências.

#### Desigualdade

Os operadores `<`, `>`, `<=`, e `>=` são usados para representar uma desigualdade, sendo referenciados na especificação como "compraradores relacionais". Tipicamente eles são usados para comparar valores ordinários como `number`s. É fácil entender que  `3 < 4`.

Mas em JavaScript, valores `string` também podem ser comparados para desigualdade, usando regras alfabéticas (`"bar" < "foo"`).

E como fica a coerção: Regras similares à comparação `==` (apesar de não serem idênticas!) aplicam-se aos operadores de desigualdade. Uma nota importante, é que não existe um operador de "desigualdade estrita" que possa desabilitar a coerção da mesma forma que a "igualdade estrita" `===` faz.

Considere:

```js
var a = 41;
var b = "42";
var c = "43";

a < b;		// true
b < c;		// true
```

O que acontece aqui: Na seção 11.8.5, da especificaão do ES5, ela diz que ambos os valores na comparação `<` são `string`s, assim como em `b < c`, a comparação é feita lexicograficamente (em outras palasvras: alfabeticamente, como um dicionário). Mas se um ou ambos os valores não forem uma `string`, como acontece em `a < b`, então ambos os valores são coergidos para serem `number`s, e uma comparação típica de números acontece.

A maior pegadinha que você pode encontrar aqui é em comparações entre diferentes tipos de valores -- lembrando, não existem formas de usar uma "desigualdade estrita" -- é quando um dos valores não pode ser transformado em um número válido, como por exemplo:

```js
var a = 42;
var b = "foo";

a < b;		// false
a > b;		// false
a == b;		// false
```

Espera, como podem as três comparações serem `false`? Porque o valor de `b` é coergido para um "valor numérico inválido" (`NaN`), nas comparações `<` e `>`, e a especificação diz que `NaN` não é nem maior nem menor do que qualquer valor.

A comparação `==` falha por uma razão diferente. `a == b` pode falhar se for interpretada tanto como `42 == NaN` ou como `"42" == "foo"` -- como explicamos anteriormente.

**Nota:** For more information about the inequality comparison rules, see section 11.8.5 of the ES5 specification and also consult Chapter 4 of the *Types & Grammar* title of this series.

## Variáveis

Em JavaScript, nome de variáveis (incluindo nomes de funções) precisam ter *identificadores* válidos. As restritas e completas regras para caracteres válidos como identificadores são um pouco complexas considerando os caracteres não tradicionais como Unicode. Se você considerar apenas caracteres alfanuméricos típicos do ASCII, as regras ficam mais simples.

Um identificador precisa começar com `a`-`z`, `A`-`Z`, `$`, ou `_`. Ele pode conter qualquer um desses caracteres e incluem também numerais `0`-`9`.

Em geral, as mesmas regras que se aplicam tanto para identificar variáveis como para nomear uma propriedade. Entretanto, algumas palavras não podem ser usadas para nomear variáveis, mas não tem impedimento para nomear propriedades. Essas palavras são denominadas "palavras reservadasˆ(reserved words), e incluem as palavras-chave (`for`, `in`, `if`, etc.) assim como `null`, `true`, e `false`.

**Nota:** Para mais informações sobre palavas reservadas, veja o Apêndice A do título desta série *Tipos & Gramática*.

### Escopos de Função

Você usa a palavra-chave `var` para declarar a variável que irá referenciar o escopo da função corrente, ou o escopo global se ela estiver no topo de tudo, fora de qualquer outra função.

#### Hoisting

Onde quer que `var` apareça dentro de um escopo, sua declaração é tomada como parte de todo o escopo e acessada em qualquer área dentro dele.

Metaforicamente, esse comportamento é chamado de *hoisting*, quando uma declaração `var` é conceitualmente "movida" para o topo do escopo. Tecnicamente, este processo é explicado de forma mais apurada entendendo como o código é compilado, mas vamos pular estes detalhes por ora.

Considere:

```js
var a = 2;

foo();					// funciona porque a declaração `foo()`
						// é "hoisted"

function foo() {
	a = 3;

	console.log( a );	// 3

	var a;				// a declaração é "hoisted"
						// para o topo de `foo()`
}

console.log( a );	// 2
```

**Atenção:** Não é comum nem uma boa ideia se basear no *hoisting* de variáveis para usar uma variável antes de seu escopo do que quando ao invés de quando a declaração `var` aparece; pode ficar confuso. It's much more common and accepted to use *hoisted* function declarations, as we do with the `foo()` call appearing before its formal declaration.

#### Escopos Aninhados

Quando você declara uma variável, ela é disponibilizada em todos os lugares dentro do escopo, assim como dentro de qualquer escopo interno. Por exemplo:

```js
function foo() {
	var a = 1;

	function bar() {
		var b = 2;

		function baz() {
			var c = 3;

			console.log( a, b, c );	// 1 2 3
		}

		baz();
		console.log( a, b );		// 1 2
	}

	bar();
	console.log( a );				// 1
}

foo();
```

Note que `c` não está disponível dentro de `bar()`, porque está declarado dentro do escopo de `baz()`, e o `b` não está disponível para `foo()` pelo mesmo motivo.

Se você tentar acessar o valor da variável dentro de um escopo onde ela não está disponível, você irá receber um erro de `ReferenceError`. Se você tentar setar uma variável que ainda não foi declarada, ou você terminará criando uma variável no escopo global (ruim!) ou gerar um erro, dependendo de você ter declarado "strict mode" (veja "Strict Mode"). Vamos dar uma olhada:

```js
function foo() {
	a = 1;	// `a` not formally declared
}

foo();
a;			// 1 -- oops, auto global variable :(
```

Esta é uma prática muito ruim. Não faa isso! Sempre declare suas variáveis formalmente.

Além de criarmos declarações de variáveis no mesmo nível da função, o ES6 *deixa* (let) você criar variáveis que irão pertencer os blocos individuais (pares de `{ .. }`), usando a palavra-chave `let`. Apesar de suas nuances e detalhes, as regras do escopo terão o comportamento bem parecido com o que vimos em funções.

```js
function foo() {
	var a = 1;

	if (a >= 1) {
		let b = 2;

		while (b < 5) {
			let c = b * 2;
			b++;

			console.log( a + c );
		}
	}
}

foo();
// 5 7 9
```

Por usarmos `let` ao invés de `var`, `b` irá pertencer apenas à instrução `if` e não para todo o escopo da função `foo()`. De maneira similar, `c` pertence somente ao loop `while`. Escopamentos de bloco são muito úteis para controlar seus escopos de variáveis, usando uma maneira requintada, o que faz seu código muito mais fácil de manter ao longo do tempo.

**Nota:** Para mais informações sobre escopos, veja o título desta série *Escopos & Encerramentos*. Veja o título *ES6 & Além*para mais informações sobre o bloco de escopo `let`.

## Condicionais

Em adição à instrução condicional `if` que introduzimos brevemente no Capítulo 1, o JavaScript nos fornece alguns outros mecanismos de condicionais que devemos saber.

Por vezes você se encontrará escrevendo uma série de instruções `if..else..if` como estas:

```js
if (a == 2) {
	// faça alguma coisa
}
else if (a == 10) {
	// faça outra coisa
}
else if (a == 42) {
	// faça outra coisa diferente
}
else {
	// resultado se nenhuma instrução for atendida (fallback) 
}
```

Essa estrutura funciona, mais é um pouco verbosa porque você precisa especificar um teste para `a` em cada um dos casos. Abaixo veremos uma outra opção, a instrução `switch`:

```js
switch (a) {
	case 2:
		// faça alguma coisa
		break;
	case 10:
		// faça outra coisa
		break;
	case 42:
		// faça outra coisa diferente
		break;
	default:
		// resultado se nenhuma instrução for atendida (fallback) 
}
```

O `break` é importante se você quiser que apenas uma instrução seja executada em cada`case`. Se você omitir o `break` de um `case`, e esse `case` for aceito ou rodar, a execução irá continuar pelos próximos `case`'s independente do `case` que foi aceito. Esse então chamado "fall through" é por vezes útil/proposital:

```js
switch (a) {
	case 2:
	case 10:
		// alguma coisa legal
		break;
	case 42:
		// outra coisa
		break;
	default:
		// resultado se nenhuma instrução for atendida (fallback) 
}
```

Aqui, se `a` for ou `2` ou `10`, iremos  executar a instrução de código "some cool stuff".

Uma outra forma de condicional em JavaScript é o "operador condicional," chamado também de "operador ternário." Ele é como se fosse uma forma concisa/simplificada de uma instrução `if..else`, como em:

```js
var a = 42;

var b = (a > 41) ? "hello" : "world";

// similar a:

// if (a > 41) {
//    b = "hello";
// }
// else {
//    b = "world";
// }
```

Se a expressão teste (`a > 41` aqui) for avaliada como `true`, a primeira cláusula (`"hello"`) é retornada, se não for, a segunda cláusula é retornada (`"world"`), e qualquer que seja o resultado ele é designado para `b`.

O operador condicional não precisa necessariamente ser usado em uma atribuição, mas é definitivamente a forma mais comum de usá-lo.

**Nota:** Para mais informações sobre testes de condições e outros padrões para `switch` e `? :`, veja o título desta série *Tipos & Gramática*.

## Strict Mode

ES5 added a "strict mode" to the language, which tightens the rules for certain behaviors. Generally, these restrictions are seen as keeping the code to a safer and more appropriate set of guidelines. Also, adhering to strict mode makes your code generally more optimizable by the engine. Strict mode is a big win for code, and you should use it for all your programs.

You can opt in to strict mode for an individual function, or an entire file, depending on where you put the strict mode pragma:

```js
function foo() {
	"use strict";

	// this code is strict mode

	function bar() {
		// this code is strict mode
	}
}

// this code is not strict mode
```

Compare that to:

```js
"use strict";

function foo() {
	// this code is strict mode

	function bar() {
		// this code is strict mode
	}
}

// this code is strict mode
```

One key difference (improvement!) with strict mode is disallowing the implicit auto-global variable declaration from omitting the `var`:

```js
function foo() {
	"use strict";	// turn on strict mode
	a = 1;			// `var` missing, ReferenceError
}

foo();
```

If you turn on strict mode in your code, and you get errors, or code starts behaving buggy, your temptation might be to avoid strict mode. But that instinct would be a bad idea to indulge. If strict mode causes issues in your program, almost certainly it's a sign that you have things in your program you should fix.

Not only will strict mode keep your code to a safer path, and not only will it make your code more optimizable, but it also represents the future direction of the language. It'd be easier on you to get used to strict mode now than to keep putting it off -- it'll only get harder to convert later!

**Note:** For more information about strict mode, see the Chapter 5 of the *Types & Grammar* title of this series.

## Functions As Values

So far, we've discussed functions as the primary mechanism of *scope* in JavaScript. You recall typical `function` declaration syntax as follows:

```js
function foo() {
	// ..
}
```

Though it may not seem obvious from that syntax, `foo` is basically just a variable in the outer enclosing scope that's given a reference to the `function` being declared. That is, the `function` itself is a value, just like `42` or `[1,2,3]` would be.

This may sound like a strange concept at first, so take a moment to ponder it. Not only can you pass a value (argument) *to* a function, but *a function itself can be a value* that's assigned to variables, or passed to or returned from other functions.

As such, a function value should be thought of as an expression, much like any other value or expression.

Consider:

```js
var foo = function() {
	// ..
};

var x = function bar(){
	// ..
};
```

The first function expression assigned to the `foo` variable is called *anonymous* because it has no `name`.

The second function expression is *named* (`bar`), even as a reference to it is also assigned to the `x` variable. *Named function expressions* are generally more preferable, though *anonymous function expressions* are still extremely common.

For more information, see the *Scope & Closures* title of this series.

### Immediately Invoked Function Expressions (IIFEs)

In the previous snippet, neither of the function expressions are executed -- we could if we had included `foo()` or `x()`, for instance.

There's another way to execute a function expression, which is typically referred to as an *immediately invoked function expression* (IIFE):

```js
(function IIFE(){
	console.log( "Hello!" );
})();
// "Hello!"
```

The outer `( .. )` that surrounds the `(function IIFE(){ .. })` function expression is just a nuance of JS grammar needed to prevent it from being treated as a normal function declaration.

The final `()` on the end of the expression -- the `})();` line -- is what actually executes the function expression referenced immediately before it.

That may seem strange, but it's not as foreign as first glance. Consider the similarities between `foo` and `IIFE` here:

```js
function foo() { .. }

// `foo` function reference expression,
// then `()` executes it
foo();

// `IIFE` function expression,
// then `()` executes it
(function IIFE(){ .. })();
```

As you can see, listing the `(function IIFE(){ .. })` before its executing `()` is essentially the same as including `foo` before its executing `()`; in both cases, the function reference is executed with `()` immediately after it.

Because an IIFE is just a function, and functions create variable *scope*, using an IIFE in this fashion is often used to declare variables that won't affect the surrounding code outside the IIFE:

```js
var a = 42;

(function IIFE(){
	var a = 10;
	console.log( a );	// 10
})();

console.log( a );		// 42
```

IIFEs can also have return values:

```js
var x = (function IIFE(){
	return 42;
})();

x;	// 42
```

The `42` value gets `return`ed from the `IIFE`-named function being executed, and is then assigned to `x`.

### Closure

*Closure* is one of the most important, and often least understood, concepts in JavaScript. I won't cover it in deep detail here, and instead refer you to the *Scope & Closures* title of this series. But I want to say a few things about it so you understand the general concept. It will be one of the most important techniques in your JS skillset.

You can think of closure as a way to "remember" and continue to access a function's scope (its variables) even once the function has finished running.

Consider:

```js
function makeAdder(x) {
	// parameter `x` is an inner variable

	// inner function `add()` uses `x`, so
	// it has a "closure" over it
	function add(y) {
		return y + x;
	};

	return add;
}
```

The reference to the inner `add(..)` function that gets returned with each call to the outer `makeAdder(..)` is able to remember whatever `x` value was passed in to `makeAdder(..)`. Now, let's use `makeAdder(..)`:

```js
// `plusOne` gets a reference to the inner `add(..)`
// function with closure over the `x` parameter of
// the outer `makeAdder(..)`
var plusOne = makeAdder( 1 );

// `plusTen` gets a reference to the inner `add(..)`
// function with closure over the `x` parameter of
// the outer `makeAdder(..)`
var plusTen = makeAdder( 10 );

plusOne( 3 );		// 4  <-- 1 + 3
plusOne( 41 );		// 42 <-- 1 + 41

plusTen( 13 );		// 23 <-- 10 + 13
```

More on how this code works:

1. When we call `makeAdder(1)`, we get back a reference to its inner `add(..)` that remembers `x` as `1`. We call this function reference `plusOne(..)`.
2. When we call `makeAdder(10)`, we get back another reference to its inner `add(..)` that remembers `x` as `10`. We call this function reference `plusTen(..)`.
3. When we call `plusOne(3)`, it adds `3` (its inner `y`) to the `1` (remembered by `x`), and we get `4` as the result.
4. When we call `plusTen(13)`, it adds `13` (its inner `y`) to the `10` (remembered by `x`), and we get `23` as the result.

Don't worry if this seems strange and confusing at first -- it can be! It'll take lots of practice to understand it fully.

But trust me, once you do, it's one of the most powerful and useful techniques in all of programming. It's definitely worth the effort to let your brain simmer on closures for a bit. In the next section, we'll get a little more practice with closure.

#### Modules

The most common usage of closure in JavaScript is the module pattern. Modules let you define private implementation details (variables, functions) that are hidden from the outside world, as well as a public API that *is* accessible from the outside.

Consider:

```js
function User(){
	var username, password;

	function doLogin(user,pw) {
		username = user;
		password = pw;

		// do the rest of the login work
	}

	var publicAPI = {
		login: doLogin
	};

	return publicAPI;
}

// create a `User` module instance
var fred = User();

fred.login( "fred", "12Battery34!" );
```

The `User()` function serves as an outer scope that holds the variables `username` and `password`, as well as the inner `doLogin()` function; these are all private inner details of this `User` module that cannot be accessed from the outside world.

**Warning:** We are not calling `new User()` here, on purpose, despite the fact that probably seems more common to most readers. `User()` is just a function, not a class to be instantiated, so it's just called normally. Using `new` would be inappropriate and actually waste resources.

Executing `User()` creates an *instance* of the `User` module -- a whole new scope is created, and thus a whole new copy of each of these inner variables/functions. We assign this instance to `fred`. If we run `User()` again, we'd get a new instance entirely separate from `fred`.

The inner `doLogin()` function has a closure over `username` and `password`, meaning it will retain its access to them even after the `User()` function finishes running.

`publicAPI` is an object with one property/method on it, `login`, which is a reference to the inner `doLogin()` function. When we return `publicAPI` from `User()`, it becomes the instance we call `fred`.

At this point, the outer `User()` function has finished executing. Normally, you'd think the inner variables like `username` and `password` have gone away. But here they have not, because there's a closure in the `login()` function keeping them alive.

That's why we can call `fred.login(..)` -- the same as calling the inner `doLogin(..)` -- and it can still access `username` and `password` inner variables.

There's a good chance that with just this brief glimpse at closure and the module pattern, some of it is still a bit confusing. That's OK! It takes some work to wrap your brain around it.

From here, go read the *Scope & Closures* title of this series for a much more in-depth exploration.

## `this` Identifier

Another very commonly misunderstood concept in JavaScript is the `this` identifier. Again, there's a couple of chapters on it in the *this & Object Prototypes* title of this series, so here we'll just briefly introduce the concept.

While it may often seem that `this` is related to "object-oriented patterns," in JS `this` is a different mechanism.

If a function has a `this` reference inside it, that `this` reference usually points to an `object`. But which `object` it points to depends on how the function was called.

It's important to realize that `this` *does not* refer to the function itself, as is the most common misconception.

Here's a quick illustration:

```js
function foo() {
	console.log( this.bar );
}

var bar = "global";

var obj1 = {
	bar: "obj1",
	foo: foo
};

var obj2 = {
	bar: "obj2"
};

// --------

foo();				// "global"
obj1.foo();			// "obj1"
foo.call( obj2 );	// "obj2"
new foo();			// undefined
```

There are four rules for how `this` gets set, and they're shown in those last four lines of that snippet.

1. `foo()` ends up setting `this` to the global object in non-strict mode -- in strict mode, `this` would be `undefined` and you'd get an error in accessing the `bar` property -- so `"global"` is the value found for `this.bar`.
2. `obj1.foo()` sets `this` to the `obj1` object.
3. `foo.call(obj2)` sets `this` to the `obj2` object.
4. `new foo()` sets `this` to a brand new empty object.

Bottom line: to understand what `this` points to, you have to examine how the function in question was called. It will be one of those four ways just shown, and that will then answer what `this` is.

**Note:** For more information about `this`, see Chapters 1 and 2 of the *this & Object Prototypes* title of this series.

## Prototypes

The prototype mechanism in JavaScript is quite complicated. We will only glance at it here. You will want to spend plenty of time reviewing Chapters 4-6 of the *this & Object Prototypes* title of this series for all the details.

When you reference a property on an object, if that property doesn't exist, JavaScript will automatically use that object's internal prototype reference to find another object to look for the property on. You could think of this almost as a fallback if the property is missing.

The internal prototype reference linkage from one object to its fallback happens at the time the object is created. The simplest way to illustrate it is with a built-in utility called `Object.create(..)`.

Consider:

```js
var foo = {
	a: 42
};

// create `bar` and link it to `foo`
var bar = Object.create( foo );

bar.b = "hello world";

bar.b;		// "hello world"
bar.a;		// 42 <-- delegated to `foo`
```

It may help to visualize the `foo` and `bar` objects and their relationship:

<img src="fig6.png">

The `a` property doesn't actually exist on the `bar` object, but because `bar` is prototype-linked to `foo`, JavaScript automatically falls back to looking for `a` on the `foo` object, where it's found.

This linkage may seem like a strange feature of the language. The most common way this feature is used -- and I would argue, abused -- is to try to emulate/fake a "class" mechanism with "inheritance."

But a more natural way of applying prototypes is a pattern called "behavior delegation," where you intentionally design your linked objects to be able to *delegate* from one to the other for parts of the needed behavior.

**Note:** For more information about prototypes and behavior delegation, see Chapters 4-6 of the *this & Object Prototypes* title of this series.

## Old & New

Some of the JS features we've already covered, and certainly many of the features covered in the rest of this series, are newer additions and will not necessarily be available in older browsers. In fact, some of the newest features in the specification aren't even implemented in any stable browsers yet.

So, what do you do with the new stuff? Do you just have to wait around for years or decades for all the old browsers to fade into obscurity?

That's how many people think about the situation, but it's really not a healthy approach to JS.

There are two main techniques you can use to "bring" the newer JavaScript stuff to the older browsers: polyfilling and transpiling.

### Polyfilling

The word "polyfill" is an invented term (by Remy Sharp) (https://remysharp.com/2010/10/08/what-is-a-polyfill) used to refer to taking the definition of a newer feature and producing a piece of code that's equivalent to the behavior, but is able to run in older JS environments.

For example, ES6 defines a utility called `Number.isNaN(..)` to provide an accurate non-buggy check for `NaN` values, deprecating the original `isNaN(..)` utility. But it's easy to polyfill that utility so that you can start using it in your code regardless of whether the end user is in an ES6 browser or not.

Consider:

```js
if (!Number.isNaN) {
	Number.isNaN = function isNaN(x) {
		return x !== x;
	};
}
```

The `if` statement guards against applying the polyfill definition in ES6 browsers where it will already exist. If it's not already present, we define `Number.isNaN(..)`.

**Note:** The check we do here takes advantage of a quirk with `NaN` values, which is that they're the only value in the whole language that is not equal to itself. So the `NaN` value is the only one that would make `x !== x` be `true`.

Not all new features are fully polyfillable. Sometimes most of the behavior can be polyfilled, but there are still small deviations. You should be really, really careful in implementing a polyfill yourself, to make sure you are adhering to the specification as strictly as possible.

Or better yet, use an already vetted set of polyfills that you can trust, such as those provided by ES5-Shim (https://github.com/es-shims/es5-shim) and ES6-Shim (https://github.com/es-shims/es6-shim).

### Transpiling

There's no way to polyfill new syntax that has been added to the language. The new syntax would throw an error in the old JS engine as unrecognized/invalid.

So the better option is to use a tool that converts your newer code into older code equivalents. This process is commonly called "transpiling," a term for transforming + compiling.

Essentially, your source code is authored in the new syntax form, but what you deploy to the browser is the transpiled code in old syntax form. You typically insert the transpiler into your build process, similar to your code linter or your minifier.

You might wonder why you'd go to the trouble to write new syntax only to have it transpiled away to older code -- why not just write the older code directly?

There are several important reasons you should care about transpiling:

* The new syntax added to the language is designed to make your code more readable and maintainable. The older equivalents are often much more convoluted. You should prefer writing newer and cleaner syntax, not only for yourself but for all other members of the development team.
* If you transpile only for older browsers, but serve the new syntax to the newest browsers, you get to take advantage of browser performance optimizations with the new syntax. This also lets browser makers have more real-world code to test their implementations and optimizations on.
* Using the new syntax earlier allows it to be tested more robustly in the real world, which provides earlier feedback to the JavaScript committee (TC39). If issues are found early enough, they can be changed/fixed before those language design mistakes become permanent.

Here's a quick example of transpiling. ES6 adds a feature called "default parameter values." It looks like this:

```js
function foo(a = 2) {
	console.log( a );
}

foo();		// 2
foo( 42 );	// 42
```

Simple, right? Helpful, too! But it's new syntax that's invalid in pre-ES6 engines. So what will a transpiler do with that code to make it run in older environments?

```js
function foo() {
	var a = arguments[0] !== (void 0) ? arguments[0] : 2;
	console.log( a );
}
```

As you can see, it checks to see if the `arguments[0]` value is `void 0` (aka `undefined`), and if so provides the `2` default value; otherwise, it assigns whatever was passed.

In addition to being able to now use the nicer syntax even in older browsers, looking at the transpiled code actually explains the intended behavior more clearly.

You may not have realized just from looking at the ES6 version that `undefined` is the only value that can't get explicitly passed in for a default-value parameter, but the transpiled code makes that much more clear.

The last important detail to emphasize about transpilers is that they should now be thought of as a standard part of the JS development ecosystem and process. JS is going to continue to evolve, much more quickly than before, so every few months new syntax and new features will be added.

If you use a transpiler by default, you'll always be able to make that switch to newer syntax whenever you find it useful, rather than always waiting for years for today's browsers to phase out.

There are quite a few great transpilers for you to choose from. Here are some good options at the time of this writing:

* Babel (https://babeljs.io) (formerly 6to5): Transpiles ES6+ into ES5
* Traceur (https://github.com/google/traceur-compiler): Transpiles ES6, ES7, and beyond into ES5

## Non-JavaScript

So far, the only things we've covered are in the JS language itself. The reality is that most JS is written to run in and interact with environments like browsers. A good chunk of the stuff that you write in your code is, strictly speaking, not directly controlled by JavaScript. That probably sounds a little strange.

The most common non-JavaScript JavaScript you'll encounter is the DOM API. For example:

```js
var el = document.getElementById( "foo" );
```

The `document` variable exists as a global variable when your code is running in a browser. It's not provided by the JS engine, nor is it particularly controlled by the JavaScript specification. It takes the form of something that looks an awful lot like a normal JS `object`, but it's not really exactly that. It's a special `object,` often called a "host object."

Moreover, the `getElementById(..)` method on `document` looks like a normal JS function, but it's just a thinly exposed interface to a built-in method provided by the DOM from your browser. In some (newer-generation) browsers, this layer may also be in JS, but traditionally the DOM and its behavior is implemented in something more like C/C++.

Another example is with input/output (I/O).

Everyone's favorite `alert(..)` pops up a message box in the user's browser window. `alert(..)` is provided to your JS program by the browser, not by the JS engine itself. The call you make sends the message to the browser internals and it handles drawing and displaying the message box.

The same goes with `console.log(..)`; your browser provides such mechanisms and hooks them up to the developer tools.

This book, and this whole series, focuses on JavaScript the language. That's why you don't see any substantial coverage of these non-JavaScript JavaScript mechanisms. Nevertheless, you need to be aware of them, as they'll be in every JS program you write!

## Revisão

O primeiro passo para aprender o sabor JavaScript de programação é ter um entendimento básico dos mecanismos básicos como valores, tipos, encerramentos de funções, `this` e prototipagem.

Claro, cada um desses tópicos merecem uma cobertura muito maior do que você viu aqui, mas é por isso que eles tem capítulos e livros dedicados para eles ao longo desta série de livros. Depois de se sentir confortável com os conceitos e exemplos de código deste capítulo, o resto da série te aguarda para que você se aprofunde na linguagem.

O capítulo final deste livro irá fazer um breve sumário de todos os outros títulos da série e outros conceitos que serão cobertos além do que já exploramos até aqui.
