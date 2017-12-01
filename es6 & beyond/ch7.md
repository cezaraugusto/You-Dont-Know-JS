# You Don't Know JS: ES6 & Além
# Capítulo 7: Metaprogramação

Metaprogramação é programação onde a operação visa o comportamento do próprio programa. Em outras palavras, é programar a programação do seu programa. Sim, um bocado, não é?

Por exemplo, se você investigar a relação entre um objeto `a` e outro `b` -- eles são `[[Prototype]]` conectados? -- usando `a.isPrototype(b)`, isso é comumente referido como instrospecão, uma forma de metaprogramação. Macros (que não existem em JS, ainda) -- onde o código se modifica no momento da compilação -- são outros exemplos óbvios de metaprogramação.
Enumerar as chaves de um objeto com um loop `for..in` ou verificar se um objeto é uma *instância de* um "construtor de uma classe", são outros tipos comuns de metaprogramação.

Metaprogramação concentra-se em um ou mais dos seguintes itens: inspeção de código, modificação do código ou código que modifica o comportamento padrão da linguagem, de modo que outro código seja afetado.

O objetivo da metaprogramação é aproveitar as capacidades intrísecas à linguagem para tornar o resto do seu código mais descritivo, expressivo, e/ou flexível· Por conta da natureza do termo *meta* de metaprogramação, é um tanto difícil de colocar uma definição mais precisa do que essa. A melhor forma de entender metaprogramação é ve-la através de exemplos.

A ES6 adiciona vários novos métodos/funcionalidades para metaprogramação em cima do que o JS já possuía.

## Nome de Funções

Existem casos em que seu código pode querer se voltar pra si mesmo e perguntar qual o nome de alguma função. Se você perguntar qual o nome da função, a reposta será algo surpreendentemente abíguo. Considere:

```js
function daz() {
	// ..
}

var obj = {
	foo: function() {
		// ..
	},
	bar: function baz() {
		// ..
	},
	bam: daz,
	zim() {
		// ..
	}
};
```

Nesse fragmento anterior, "qual é o nome de `obj.foo()`" é ligeiramente variado. É `"foo"`, `""`, ou `undefined`? E sobre `obj.bar()` -- ele é nomeado `"bar"` ou `"baz"`? O nome de  `obj.bam()` é `"bam"` ou `"daz"`? E sobre `obj.zim()`?

Além disso, e sobre as funções que são passadas como callbacks, como essas:

```js
function foo(cb) {
	// qual o nome de`cb()` aqui?
}

foo( function(){
	// Eu sou anônima!
} );
```

Há várias maneiras com as quais funções podem ser expressadas em programas, e nem sempre está claro e sem abiguidade qual o "nome" que a função deveria ter.

Mais importante, nós precisamos distinguir se o nome de uma função se refere à propriedade `name` da sua propriedade -- sim, funções têm uma propriedade chamada `name` -- ou o que quer que se refira ao *nome da ligação léxica*(lexical binding name), assim como `bar` em `function bar() { .. }`.

O nome da ligação léxica é o que você usa para coisas como recursão: 

```js
function foo(i) {
	if (i < 10) return foo( i * 2 );
	return i;
}
```

A propriedade `name` é o que você usaria para os propósitos da metaprogramação, então é o que vamos focar nessa discussão.

A confusão aparece porque por padrão, o nome léxico que a função tinha (se algum) também foi definido como a sua propriedade `name`. Atualmente não há requerimentos oficiais para esse comportamento pela especificação ES5 (e anterior). A definição da propriedade `name` não era padrão, mas ainda era bastante confiável. A partir da ES6, isso foi padronizado.

**Dica** Se a função tem um valor de `name` atribuído, esse é tipicamente o nome usado em traços de pilha(*stack traces*) nas ferramentas dos desenvolvedores.

### Inferências

Mas o que acontece com a propriedade `name` se uma função não tiver um nome léxico?

A partir da ES6, existem regras de inferência que podem determinar um valor adequado para ser atrubído à propriedade `name` de uma função, mesmo se essa função não tiver um nome léxico para usar.

Considere:

```js
var abc = function() {
	// ..
};

abc.name;	// "abc"
```

Se tivéssemos dado um nome léxico como `abc = function def() { .. }`, a propriedade `name` certamente seria `"def"`. Mas, na ausência de um nome léxico, intuitivamente o nome `"abc"` parece ser apropriado.

Aqui estão outras formas de inferir um nome (ou não) no ES6:

```js
(function(){ .. });					// name:
(function*(){ .. });				// name:
window.foo = function(){ .. };		// name:

class Awesome {
	constructor() { .. }			// name: Awesome
	funny() { .. }					// name: funny
}

var c = class Awesome { .. };		// name: Awesome

var o = {
	foo() { .. },					// name: foo
	*bar() { .. },					// name: bar
	baz: () => { .. },				// name: baz
	bam: function(){ .. },			// name: bam
	get qux() { .. },				// name: get qux
	set fuz() { .. },				// name: set fuz
	["b" + "iz"]:
		function(){ .. },			// name: biz
	[Symbol( "buz" )]:
		function(){ .. }			// name: [buz]
};

var x = o.foo.bind( o );			// name: bound foo
(function(){ .. }).bind( o );		// name: bound

export default function() { .. }	// name: default

var y = new Function();				// name: anonymous
var GeneratorFunction =
	function*(){}.__proto__.constructor;
var z = new GeneratorFunction();	// name: anonymous
```

A propriedade `name` não é editável por padrão, mas é configurável, significando que você pode usar `Object.defineProperty(..)` para mudar manualmente se desejar.

## Meta Propriedades

Na seção "`new.target`" do Capítulo 3, nós introduzimos um conceito novo para o JS no ES6: a meta propriedade. Como o nome sugere, meta propriedades têm a intenção de fornecer meta informações especiais na forma de acesso de uma propriedade que de outra forma não teria sido possível.

No caso de `new.target`, a palavra-chave `new` serve como contexto para o acesso de uma propriedade. Claramente `new` não é um objeto propriamente, o que torna essa capacidade especial. Entretanto, quando `new.target` é usado dentro de uma chamada de um construtor (uma função/método invocado com `new`), `new` se torna um contexto virtual, tanto que `new.target` pode se referir ao construtor de destino no qual `new` foi invocado.

Este é um exemplo claro de uma operação de meta programação, visto que sua intenção é determinar, dentro da chamada do construtor, qual era o alvo original de `new`, geralmente para propósitos de instrospecção (examinar tipagem/estrutura) ou acesso de propriedades estáticas.

Por exemplo, você pode querer ter diferentes comportamentos em um construtor, dependendo se ele foi invocado diretamente ou invocado através do filho de uma classe:

```js
class Parent {
	constructor() {
		if (new.target === Parent) {
			console.log( "Parent instanciada" );
		}
		else {
			console.log( "Descendente instanciada" );
		}
	}
}

class Child extends Parent {}

var a = new Parent();
// Parent instanciada

var b = new Child();
// Descendente instanciada
```

Há uma pequena nuance aqui, que é o fato do `constructor()` dentro da definição da classe `Parent` receber o nome léxico da própria classe (`Parent`),embora a sintaxe sugira que a classe é uma entidade separada do construtor.

**Atenção** Assim como todas técnicas de meta programação, tenha cuidado na criação de códigos que sejam muito "malandros"  ou que dificultem a compreensão para outros. Use esses macetes com cautela.

## Símbolos bem conhecidos

Na seção "Símbolos" do Capítulo 2, nós abordamos o novo tipo primitivo `symbol` do ES6. Além dos símbolos que você pode definir em seu próprio programa, o JS predefine uma série de símbolos internos, conhecidos como *Símbolos bem conhecidos* (WKS - *do original*).

Esses valores de símbolos são definidos, primeiramente, para expôr meta propriedades especiais, que ficam disponíveis para seus programas JS e proporcionam maior controle sobre o comportamento do JS.

Nós vamos introduzir brevemente cada um e discutir suas propostas.

### `Symbol.iterator`

Nos Capítulos 2 e 3, nós apresentamos e usamos o símbolo `@@iterator`, automaticamente usado pelos spreads `...` e loops `for..of`. Nós também vimos `@@iterator` definidos nas coleções no novo ES6, como mostrado no Capítulo 5.

`Symbol.iterator` representa uma localização especial (propriedade) em qualquer objeto onde os mecanismos da lignuagem bucam por um método que irá construir uma instânica de iteração para consumo dos valores deste objeto. Muitos objetos vêm com um valor definido como padrão.

Entretanto, nós podemos definir a nossa própria lógica de iterador para qualquer valor de objeto definindo a propriedade `Symbol.iterator`, mesmo se isso estiver substituindo o iterador padrão. O apecto de meta programação é que nós estamos definindo comportamentos que outras partes do JS (como operadores e construtores de loops) usam quando peocessam o valor de um objeto que nós definimos.

Considere:

```js
var arr = [4,5,6,7,8,9];

for (var v of arr) {
	console.log( v );
}
// 4 5 6 7 8 9

// define o iterador que produz apenas valores
// de índices ímpares
arr[Symbol.iterator] = function*() {
	var idx = 1;
	do {
		yield this[idx];
	} while ((idx += 2) < this.length);
};

for (var v of arr) {
	console.log( v );
}
// 5 7 9
```

### `Symbol.toStringTag` e `Symbol.hasInstance`

Umas das tarefas de meta programação mais comuns é inspecionar um valor para descobrir de qual *tipo* ele é, geralmente para decidir quais operações são apropriadas para atuar neles. Com objetos, as duas técnicas de inspeção mais comuns são `toString()` e `instanceof`.

Considere:

```js
function Foo() {}

var a = new Foo();

a.toString();				// [object Object]
a instanceof Foo;			// true
```

As of ES6, you can control the behavior of these operations:

```js
function Foo(greeting) {
	this.greeting = greeting;
}

Foo.prototype[Symbol.toStringTag] = "Foo";

Object.defineProperty( Foo, Symbol.hasInstance, {
	value: function(inst) {
		return inst.greeting == "hello";
	}
} );

var a = new Foo( "hello" ),
	b = new Foo( "world" );

b[Symbol.toStringTag] = "cool";

a.toString();				// [object Foo]
String( b );				// [object cool]

a instanceof Foo;			// true
b instanceof Foo;			// false
```

O símbolo `@@toStringTag` no prototype (ou na própria instância) especifica o valor da string para uso na conversão para string `[object ___]`.

O símbolo `@@hasInstance` é um método da função construtora que recebe o valor da instância de um objeto e deixa voçê decidir se retorna `true` ou `false` se o valor deverá ser considerado uma instância ou não.

**Observação** Para definir `@@hasInstance` em uma função, você precisa usar o `Object.defineProperty(..)`, assim como o padrão em `Function.prototype` é `writable: false`. Veja o título *this e Object Prototypes* dessa série para mais informações.

### `Symbol.species`

Em "Classes" no Capítulo 3, nós apresentamos o símbolo `@@species`, que controla qual construtor é usado em métodos legados de uma classe que precisa gerar novas instâncias.

O exemplo mais comum é quando se adiciona uma `Array` a subclasse e queremos definir qual métodos herdados seu construtor (`Array(..)` ou sua subclasse), como o `slice(..)` você deveria usar. Por padrão, o `slice (..)` chamado em uma instância de uma subclasse de `Array` produziria uma nova instância dessa subclasse, na qual é, francamente, o que você vai querer frequentemente.

Entretanto, você pode meta programar substituindo a definição padrão `@@species`:

```js
class Cool {
	// adiar `@@species` para um construtor derivado
	static get [Symbol.species]() { return this; }

	again() {
		return new this.constructor[Symbol.species]();
	}
}

class Fun extends Cool {}

class Awesome extends Cool {
	// forçar `@@species` para ser um construtor pai
	static get [Symbol.species]() { return Cool; }
}

var a = new Fun(),
	b = new Awesome(),
	c = a.again(),
	d = b.again();

c instanceof Fun;			// true
d instanceof Awesome;		// false
d instanceof Cool;			// true
```

A configuração `Symbol.species` padrão está nos construtores nativos incorporados ao comportamento do `return this` assim como ilustrado no snippet anterior na definição `cool`. Não há padrão em classes de usuário, mas, como mostrado, esse comportamento é fácil de emular.

Se você precisa definir métodos que irão gerar novas instâncias, use a a metaprogramação do  padrão `new this.constructor[Symbol.species](..)` em vez da engessada `new this.constructor(..)` or `new XYZ(..)`. Classes derivadas serão então capazes de customizar `Symbol.species` para controlar qual construtor vende aquelas isntâncias.

### `Symbol.toPrimitive`

No título *Typos & Gramática* dessa série, nós discutimos a operação de coerção abstrata `ToPrimitive`, que é usada quando um objeto precisa ser coagido para um valor primitivo para algumas operações (assim como a comparação `==` ou adição `+`). Antes do ES6, não havia uma maneira de controlar esse comportamento.

No ES6, o símbolo `@@toPrimitive` como uma propriedade em qualquer valor de objeto pode customizar essa coerção `toPrimitive` especificando um método.

Considere:

```js
var arr = [1,2,3,4,5];

arr + 10;				// 1,2,3,4,510

arr[Symbol.toPrimitive] = function(hint) {
	if (hint == "default" || hint == "number") {
		// soma de todos os números
		return this.reduce( function(acc,curr){
			return acc + curr;
		}, 0 );
	}
};

arr + 10;				// 25
```
O método `Symbol.toPrimitive` vai ser fornecido com uma *dica* de `"string"`, `"number"`, ou `"default"` (o que será interpretado como `"number"`), dependendo de qual tipo de operação invocada `ToPrimitive` está esperando. No snippet anterior, a operação de adição `+` não tem dica (é passado o `"default"`). Uma operação de multiplicação `*` vai ter uma dica `"number"` e uma `String(arr)` vai ter uma dica `"string"`.

**Atenção** O operador `==` vai invocar a operação `ToPrimitive` sem uma dica -- o método `@@toPrimitive`, se algum for chamado com a dica `"default"` -- em um objeto se o outro valor comparado não for um objeto. Entretanto, se os dois valores de comparação são objetos, o comportamento de `==` é idêntico ao `===`. que se referenciam a eles mesmos e são comparáveis diretamente. Nesse caso, `@@toPrimitive` não é invocado. Veja no título *Tipos & Gramática* dessa série para mais informações sobre coerção e as operaçãoes abstratas.

### Símbolos de expressões regulares

Há quatro símbolos bem conhecidos que podem ser subtituídos por objetos de expressão regular, que controla como essas expressões regulares são usadas pelas quatro funções `String.prototype` com os mesmos nomes correspondentes:

* `@@match`: O valor `Symbol.match` de uma expressão regular é o método usado para combinar todas as partes do valor de uma string com a expressão regular fornecida. É usado por `String.prototype.match(..)` se você passar ele como uma expressão regular para o padrão de combinação.

	O algotítimo padrão para combinar está na seção 21.2.5.6 da especificação ES6 (https://people.mozilla.org/~jorendorff/es6-draft.html#sec-regexp.prototype-@@match). Você poderá substituir esse algoritimo padrão e fornacer funcionalidades extras na regex, assim como assertivas looking-behind.

	`Symbol.match` também  é usado pela operação abstrata `isRegExp` (veja a nota em "Funções de inspeção da String" no Capítulo 6) para determinar se uma objeto tem a intenção de ser usado como uma expressão regular. Para forçar a falha dessa verificação no objeto oara que isso não seja tratado como expressão regular, defina o valor do símbolo `Symbol.match` para `false` (ou algo falseável).

*`@@replace`: O valor do símbolo `Symbol.replace` de uma expressão regular é o método usado pelo `String.prototype.replace(..)` para substituir dentro de uma string uma ou todas ocorrências de sequências de caracteres que combinarem com padrão fornecido das expressões regulares.

	O algoritmo padrão para substituição está explícito na seção 21.2.5.8 da especificação ES6 (https://people.mozilla.org/~jorendorff/es6-draft.html#sec-regexp.prototype-@@replace).

	Um uso legal para substituir o algoritmo padrão pe fornecer opções adicionais do argumento `replacer`, assim como o dado `"abaca".replace(/a/g,[1,2,3])` produz `"1b2c3"` consumindo a iteração por sucessivos valores substituídos.

* `@@search`: O valor `Symbol.search` de uma expressão regular é o método usado por `String.prototype.search(..)` para buscar por uma substring dentro de outra string assim que combinado com a expressão regular fornecida.

	O algoritmo padrão para buscas está descrito na seção 21.2.5.9 da especificação ES6 (https://people.mozilla.org/~jorendorff/es6-draft.html#sec-regexp.prototype-@@search).

* `@@split`: O valor do símbolo `Symbol.split` de uma expressão regular é o método usado por `String.prototype.split(..)` para repartir uma string em substrings na localização(ões) do delimitador da combinação da expressão regular fornecida.

	O algoritmo padrão para repartir (splitting) está na seção 21.2.5.11 da especificação ES6(https://people.mozilla.org/~jorendorff/es6-draft.html#sec-regexp.prototype-@@split).

Substituir os algoritmos de expressão regular nativos não é para os fracos de coração! O JS vem com um motor de expressões regulares altamente otimizado, então seu próprio código provavelmente será bem mais lento. Esse tipo de metaprogramação é limpo e poderoso, mas só deve ser usado em casos onde são realmente necessários ou benéficos.

### `Symbol.isConcatSpreadable`

O símbolo `@@isConcarSpreadable` pode ser definido como uma propriedade booleana (`Symbol.isConcatSpreadable`) de qualquer objeto (como um array ou outro iterável) para indicar se isso deve ser *espalhado* se passar para um array `concat(..)`

Considere:

```js
var a = [1,2,3],
	b = [4,5,6];

b[Symbol.isConcatSpreadable] = false;

[].concat( a, b );		// [1,2,3,[4,5,6]]
```

### `Symbol.unscopables`

O símbolo `@@unscopables` pode ser definido com uma propriedade de objeto (`Symbol.unscopables`)em qualquer objeto para indicar qual propriedade não podem ser expostas como variáveis léxicas em uma declaração `with`.

Considere:

```js
var o = { a:1, b:2, c:3 },
	a = 10, b = 20, c = 30;

o[Symbol.unscopables] = {
	a: false,
	b: true,
	c: false
};

with (o) {
	console.log( a, b, c );		// 1 20 3
}
```

Um `true` em um objeto `@@unscopables` indica que a propriedade deve ser *fora de escopo (unscopable)*, e é filtrada do escopo das variáveis léxicas. `false` significa que é OK incluir em um escopo de variáveis léxicas.

**Atenção** A declaração `with` é totalmente proibida no modo `strict`, assim como deve ser considerada obsoleta para a linguagem. Não a use. Veja o título *Escopo & Closures* dessa série para mais informações. Porque `with` deve ser evitado, o símbolo `@@ unscopables` também é discutível.

## Proxies

Uma das funcionalidades mais óbvias da metaprogramação adicionanda no ES6 é a funcionalidade `Proxy`.

Um proxy é um tipo especial de objeto que você cria aqueles "wraps" -- ou seta da frente de -- outro objeto normal. Você pode registrar marcadores especiais (aka *armadilhas*) no objeto de proxy que são chamados quando várias operações são performadas contra o proxy. Esses marcadores têm a oportunidade de realizar lógicas extras em adição a operações de *encaminhamento* no objeto alvo original.

Um exemplo do tipo de marcadores de *armadilhas* que você pode definir em um proxy é `get` que intercepta a operação `[[Get]]` -- realizada quando você tenta acessar a propriedade de um objeto.

Considere:

```js
var obj = { a: 1 },
	handlers = {
		get(target,key,context) {
			// nota: target === obj,
			// contexto === pobj
			console.log( "accessing: ", key );
			return Reflect.get(
				target, key, context
			);
		}
	},
	pobj = new Proxy( obj, handlers );

obj.a;
// 1

pobj.a;
// acessando: a
// 1
```

Nós declaramos um manipulador `get(..)` como nomeado no método no objeto *handler* (segundo argumento para `Proxy(..)`), que recebe uma referência para o objeto *target* (`obj`), o nome da propriedade *key* (`"a"`), e o `self`/recebedor/proxy (`pobj`).

Depois do `console.log(..)` traçar a declaração, nós "encaminhamos" a operação para o `obj` via `Reflect.get(..)`. Nós vamos abordar a API `Reflect` na próxima seção, mas note que cada armadilha proxy disponível tem uma funcção `Reflect` correspondente com o mesmo nome.

Esses mapeamentos têm propósitos simétricos. cada manipulador proxy intercepta quando uma respectiva tarefa de metaprogramação é realizada, e cada uma das utilidades do `Reflect` realizam a terefa de mataprogramação respectiva em um objeto. Cada manipulador proxy tem uma definição padrão que automaticamente chama uma utilidade `Reflect` correspondente. Você vai com certeza utilizar ambas, `Reflect` e `Proxy`.

Aqui está uma lista de manipuladores que você pode4rá definir em um proxy para uma função/objeto *alvo*, e como/quando eles serão disparados:

* `get(..)`: via `[[Get]]`, uma propriedade é acessado no proxy (`Reflect.get(..)`, `.` operador da propriedade, ou `[ .. ]` operador da proopriedade)
* `set(..)`: via `[[Set]]`, um valor é definido para a propriedade do proxy (`Reflect.set(..)`, o `=` operador de atribuição, ou desestruturando a atribuição se ele tiver uma propriedade de objeto como alvo)
* `deleteProperty(..)`: via `[[Delete]]`, uma propriedade é deletada do proxy (`Reflect.deleteProperty(..)` ou `delete`)
* `apply(..)` (se *target* é uma função): via `[[Call]]`, o proxy é invocado como uma função/método normal (`Reflect.apply(..)`, `call(..)`, `apply(..)`, ou o `(..)` operador de chamada)
* `construct(..)` (se o *target* é uma função construtora): via `[[Construct]]`, o proxy é invocado como como uma função construtora (`Reflect.construct(..)` ou `new`)
* `getOwnPropertyDescriptor(..)`: via `[[GetOwnProperty]]`, uma propriedade decriptada é recuperada do proxy (`Object.getOwnPropertyDescriptor(..)` ou `Reflect.getOwnPropertyDescriptor(..)`)
* `defineProperty(..)`: via `[[DefineOwnProperty]]`, uma propriedade decripitada é definida no Proxy (`Object.defineProperty(..)` ou `Reflect.defineProperty(..)`)
* `getPrototypeOf(..)`: via `[[GetPrototypeOf]]`, o `[[Prototype]]` do proxy é recuperado (`Object.getPrototypeOf(..)`, `Reflect.getPrototypeOf(..)`, `__proto__`, `Object#isPrototypeOf(..)`, ou `instanceof`)
* `setPrototypeOf(..)`: via `[[SetPrototypeOf]]`, o `[[Prototype]]` do proxy é definido(`Object.setPrototypeOf(..)`, `Reflect.setPrototypeOf(..)`, or `__proto__`)
* `preventExtensions(..)`: via `[[PreventExtensions]]`, o proxy é feito não extensível(`Object.preventExtensions(..)` ou `Reflect.preventExtensions(..)`)
* `isExtensible(..)`: via `[[IsExtensible]]`, a extensibilidade do proxy é avaliada
 (`Object.isExtensible(..)` or `Reflect.isExtensible(..)`)
* `ownKeys(..)`: via `[[OwnPropertyKeys]]`, o conjunto de propriedades próprias e/ou propriedade de símbolos do proxy é recuperado (`Object.keys(..)`, `Object.getOwnPropertyNames(..)`, `Object.getOwnSymbolProperties(..)`, `Reflect.ownKeys(..)`, ou `JSON.stringify(..)`)
* `enumerate(..)`: via `[[Enumerate]]`, um iterador é solicitado para propriedades próprias enumeradas e "herdadas" do proxy (`Reflect.enumerate(..)` ou `for..in`)
* `has(..)`: via `[[HasProperty]]`, o proxy é avalidado para ver se ele tem uma propriedade própria ou "herdada" (`Reflect.has(..)`, `Object#hasOwnProperty(..)`, ou `"prop" in obj`)

**Dica** Para mais informação sobre cada uma dessas tarefas de metaprogramação, veja a seção API `Reflect` depois nesse capítulo.

Além das notações na lista anterior sobre ações que irão disparar várias armadilhas, algumas armadilhas são disparadas indiretamente por ações padrão de outra armadilha. Poe exemplo:

```js
var handlers = {
		getOwnPropertyDescriptor(target,prop) {
			console.log(
				"getOwnPropertyDescriptor"
			);
			return Object.getOwnPropertyDescriptor(
				target, prop
			);
		},
		defineProperty(target,prop,desc){
			console.log( "defineProperty" );
			return Object.defineProperty(
				target, prop, desc
			);
		}
	},
	proxy = new Proxy( {}, handlers );

proxy.a = 2;
// getOwnPropertyDescriptor
// defineProperty
```

Os manipuladores `getOwnPropertyDescriptor(..)` e `defineProperty(..)` são disparados pelo manipulador padrão `set(..)`quando definido um valor de propriedade (tanto faz adiconar ou atualizar). Se você também definir seu próprio manipulador `set(..)`, você poderá ou não fazer as chamadas correspondentes conta o `context` (não o `target`!) que vai disparar essas armadilhas de proxy.

### Limitações de Proxy

Esses manipuladores de metaprogramação envolvem uma ampla gama de operações fundamentais que você pode executar contra um objeto. Entretanto, há algumas opreações que não são (ainda, pelo menos) disponíveis para interceptar.

Por exemplo, nenhum desses operadores são presos e encaminhados do proxy `pobj` para o target `obj`:

```js
var obj = { a:1, b:2 },
	handlers = { .. },
	pobj = new Proxy( obj, handlers );

typeof obj;
String( obj );
obj + "";
obj == pobj;
obj === pobj
```

Talvez no futuro, mais dessas operações fundamentais subjacentes na linguagem serão interceptáveis, nos dando ainda mais poder para estender o JavaScript dentro de si.

**Atenção** Há certa *invariantes* -- comportamentos que não podem ser sobrescritos -- que se aplicam no uso dos manipuladore de proxy. Por exemplo, o resultado do manipulador `isExtensible(..)` será sempre coagido em um `boolean`. Essas invariantes restringem algumas das suas possibilidadedes de customizar comportamentos so proxies, mas eles somente previnem você de criar comportamentos estranhos e incomuns (ou inconsistentes). As condições dessas invariantes são complicadas então não vamos abordá-las a fundo aqui, mas esse post (http://www.2ality.com/2014/12/es6-proxies.html#invariants) faz um excelente trabalho abordando-as.

### Proxies Revogáveis

Um proxy regular sempre se prende para o objeto alvo, e não poderá ser modificado após a criação -- Enquanto uma referência é mantida no proxy, a proxissão (*proxying*) continua sendo possível. No entanto, poderá haver casos onde vecê desejará criar um proxy que possa ser desabilitado quando você quiser parar de permitir que ele seja proxy. A solução é criar um *porxy revogável*:

```js
var obj = { a: 1 },
	handlers = {
		get(target,key,context) {
			// nota: target === obj,
			// contexto === pobj
			console.log( "accessing: ", key );
			return target[key];
		}
	},
	{ proxy: pobj, revoke: prevoke } =
		Proxy.revocable( obj, handlers );

pobj.a;
// acessando: a
// 1

// depois:
prevoke();

pobj.a;
// TypeError
```

Um proxy revogável é criado com `Proxy.revocable(..)`, que é uma função regular, não um construtor como `Proxy(..)`. Por outro lado, ele terá os mesmos dois argumentos: *target*(alvo) e *handlers*(manipuladores).

O retorno do valor de `Proxy.revocable(..)` não é o próprio proxy como com `new Proxy (..)`. Em vez disso, ele é um objeto com duas propriedades: *proxy* e *revoke* -- nós usamos a desestruturação do objeto (veja "Desestruturação" no Capítulo 2) para declarar essas propriedades para as variáveis `pobj` e `prevoke()`, respectivamente.

Uma vez que o proxy revogável é revogado, qualquer tentativa de acessá-lo (acionará qualquer uma das suas armadilhas) vai retornar um `TypeError`.

Um exemplo do uso de um proxy revogável pode ser distribuindo um proxy para outra parte em seu aplicativo que gerencia dados em seu modelo, em vez de dar a eles uma referência do próprio objeto real. Se o modelo do seu objeto mudar ou for substituído, você deseja invalidar o proxy que você entregou para que a outra parte saiba (através dos erros!) para requisitar uma atualização referente ao modelo.

### Utilizando Proxies

Os benefícios da metaprogramação desses manipuladores de Proxy devem ser óbvios. Nós podemos quase que totalmente interceptar (e substituir) o comportamento de objetos, significando que podemos extender o comportamento de objetos além do core do JS de maneiras muito poderosas. Nós vamos ver alguns poucos modelos exemplos para explorar as possibilidades.

#### Proxy Primeiro, Proxy por Último

Como mencionamos antes, você normalmente pensa em um proxy como "embrulhar" o objeto alvo. Nesse sentido, o proxy torna-se o objeto primário com o qual o código interage, o objeto alvo atual se mantém escondido/protegido.

Você pode fazer isso porque deseja passar o objeto por algum lugar que não pode ser totalmente "confiável", e então você precisa impor regras especiais em torno de seu acesso em vez de passar o próprio objeto.

Considere:

```js
var messages = [],
	handlers = {
		get(target,key) {
			// valor da string?
			if (typeof target[key] == "string") {
				// filtrando a pontuação
				return target[key]
					.replace( /[^\w]/g, "" );
			}

			// passar todo o resto através de
			return target[key];
		},
		set(target,key,val) {
			// apenas defina strings únicas, em caixa baixa
			if (typeof val == "string") {
				val = val.toLowerCase();
				if (target.indexOf( val ) == -1) {
					target.push(
						val.toLowerCase()
					);
				}
			}
			return true;
		}
	},
	messages_proxy =
		new Proxy( messages, handlers );

// em outro lugar:
messages_proxy.push(
	"heLLo...", 42, "wOrlD!!", "WoRld!!"
);

messages_proxy.forEach( function(val){
	console.log(val);
} );
// hello world

messages.forEach( function(val){
	console.log(val);
} );
// hello... world!!
```

Eu chamo isso de design de *proxy primeiro*(proxy first), como nós interagimos primeiro (primeiramente, inteiramente) com o proxy.

Nós reforçamos algumas regras especiais na interação com `messages_proxy` que não são reforçadas pelas próprias `messages`. Nós apenas adicionamos elementos se o valor é uma string e é também única; nós também deixamos o valor em caixa baixa. Quando recuperamos valores de `message_proxy`, nós filtramos qualquer pontuação nas strings.

Alternativamente, nós podemos inverter esse padrão completamente, onde o alvo interage com o proxy em vez do proxy interagir com o alvo. Portanto, o código só interage realmente com o objeto principal. A maneira mais fácil de cumprir esse *fallback* é ter o objeto proxy na cadeia `[[Prototype]]` do objeto principal.

Considere: 

```js
var handlers = {
		get(target,key,context) {
			return function() {
				context.speak(key + "!");
			};
		}
	},
	catchall = new Proxy( {}, handlers ),
	greeter = {
		speak(who = "someone") {
			console.log( "hello", who );
		}
	};

// definindo `greeter` como fall back para `catchall`
Object.setPrototypeOf( greeter, catchall );

greeter.speak();				// hello someone
greeter.speak( "world" );		// hello world

greeter.everyone();				// hello everyone!
```

Nós iteragimos diretamente com `greeter` em vez de `catchall`. Quando chamamos `speak(..)`, ele é encontrado em `greeter` e usado diretamente. Mas quando nós tentamos acessar um método como `everyone()`, essa função não existe em `greeter`.

O comportamento da propriedade padrão do objeto é verificar a cadeia `[[Prototype]]` (veja o título *this & Objects Prototypes* dessa série), então `catchall` é consultado para uma propriedade `everyone`. O manipulador proxy `get()` então bate e retorna uma função que chama `speak(..)` com o nome da propriedade sendo acessada (`"everyone"`).

Eu chamo esse padrão de *proxy por último*(proxy last), como o proxy é usado somente como último recurso.

#### "A Propriedade/Método não existe"

Uma queixa comum sobre o JS é que objetos não são por padrão muito defensivos em situações onde você tenta acessar ou definir uma propriedade que ainda não existe. Você pode desejar pré definir todas as propriedades/métodos para um objeto, e ter uma erro lançado se o nome de uma propriedade não existente é usada subsequentemente.

Nós podemos fazer isso com um proxy, tanto no design de *proxy first* ou *proxy last*. Vamos considerar ambos:

```js
var obj = {
		a: 1,
		foo() {
			console.log( "a:", this.a );
		}
	},
	handlers = {
		get(target,key,context) {
			if (Reflect.has( target, key )) {
				return Reflect.get(
					target, key, context
				);
			}
			else {
				throw "No such property/method!";
			}
		},
		set(target,key,val,context) {
			if (Reflect.has( target, key )) {
				return Reflect.set(
					target, key, val, context
				);
			}
			else {
				throw "No such property/method!";
			}
		}
	},
	pobj = new Proxy( obj, handlers );

pobj.a = 3;
pobj.foo();			// a: 3

pobj.b = 4;			// Error: No such property/method (A Propriedade/Método não existe)!
pobj.bar();			// Error: No such property/method(A Propriedade/Método não existe)!
```

Para ambos `get(..)` e `set(..)`, nós somente prosseguimos a operação se a propriedade do objeto alvo já existir; do contrário o erro é lançado. O objeto proxy (`pobj`) é o objeto principal que o código deverá interagir, assim que ele intercepta essas ações para oferecer proteções.

Agora, vamos considerar inverter com o design *proxy last*:

```js
var handlers = {
		get() {
			throw "No such property/method!";
		},
		set() {
			throw "No such property/method!";
		}
	},
	pobj = new Proxy( {}, handlers ),
	obj = {
		a: 1,
		foo() {
			console.log( "a:", this.a );
		}
	};

// definido `obj` como fall back para `pobj`
Object.setPrototypeOf( obj, pobj );

obj.a = 3;
obj.foo();			// a: 3

obj.b = 4;			// Error: No such property/method!
obj.bar();			// Error: No such property/method!
```

O design *proxy last* aqui é bem mais simples a respeito de como os manipuladores são definidos. Em vez da necessidade de interceptar as operações `[[Get]]` e `[[Set]]` e apenas prosseguir com elas se a propriedade do alvo existir, em vez disso nós confiamos no fato de que tanro `[[Get]]` ou `[[Set]]` chegue ao fallback do nosso `pobj`, as ações já percorreu toda a cadeia de `[[Prototype]]` e não achou uma propriedade compatível. Nós estamos livres nesse que ponto para lançar o erro incondicionalmente. Legal né?

#### Proxy "Hackeando" a cadeia `[[Prototype]]`

A operação `[[Get]]` é o canal primário pelo qual o mecanismo `[[Prototype]]` é invocado. Quando uma propriedade não é encontrada em um objeto imediato, `[[Get]]` automaticamente desliga a operação para o objeto `[[Prototype]]`.

Isso significa que você pode usar a armadilha `get(..)` de um proxy para emular ou extender a noção desse mechanismo `[[Prototype]]`.

O primeiro *hack* que nós vamos considerar é criar dois objetos que são ligados circularmente via `[[Prototype]]` (ou, pelo menos ele aparece dessa forma!). Na verdade você não pode criar uma cadeia `[[Prototype]]` circular reall, como o motor lançará um erro. Mas um proxy pode fingir isso!

Considere:


```js
var handlers = {
		get(target,key,context) {
			if (Reflect.has( target, key )) {
				return Reflect.get(
					target, key, context
				);
			}
			//  `[[Prototype]]` circular fingido
			else {
				return Reflect.get(
					target[
						Symbol.for( "[[Prototype]]" )
					],
					key,
					context
				);
			}
		}
	},
	obj1 = new Proxy(
		{
			name: "obj-1",
			foo() {
				console.log( "foo:", this.name );
			}
		},
		handlers
	),
	obj2 = Object.assign(
		Object.create( obj1 ),
		{
			name: "obj-2",
			bar() {
				console.log( "bar:", this.name );
				this.foo();
			}
		}
	);

// ligação fingida do circular `[[Prototype]]`
obj1[ Symbol.for( "[[Prototype]]" ) ] = obj2;

obj1.bar();
// bar: obj-1 <-- através de proxy fingindo [[Protótipo]]
// foo: obj-1 <-- `this` contexto continua preservado

obj2.foo();
// foo: obj-2 <-- através de [[Prototype]]
```

**Observação** Nós não precisamos de proxy/foward `[[Set]]` nesse exemplo, então menteremos as coisas simples. Para ser uma emulação `[[Prototype]]` completamente compatível, você vai querer implementar um manipulador `set(..)` que buscará na cadeia `[[Prototype]]` por uma propriedade correspondente e respeitará seu comportamento descritor (e.g., set, writable). Veja o título *this & Object Prototypes* dessa série.

No snipet anterior, `obj2` é `[[Prototype]]` ligado à `obj1` em virtude da declaração `Object.create(..)`. Mas para criar uma ligação reversa (circular), nós criamos a propriedade no `obj1` na localização do símbolo `Symbol.for("[[Prototype]]")`(Veja Símbolos no capítulo 2). Esse símbolo pode parecer meio especial/mágico, mas não é. Apenas me permite convenientemente comear um gancho que semanticamente aperece relacionado com a terafa que estou executando.

Então, o manipulador proxy `get(..)` procura primeiro enxergar se uma `key` requisitada está no proxy. Se não, a operação é manualmente desligada do referente objeto armazenado no local `Symbol.for("[[Prototype]]")` do `target`.

Uma vantagem importante desse padrão é que as definições de `obj1` e `obj2` na maioria das vezes não são invadidas pela configuração desta relação circular entre elas. Embora o fragmento anterior tenha todas as etapas entrelaçadas por razões de brevidade,s e você olhar mais de perto, a lógica do manipulador proxy é inteiramente genérica (não sabem sobre `ojb1` ou `obj2` especificamente) Então, essa lógica poderia ser puxada para um ajudante simples que os alinha, como um `setCircularPrototypeOf(..)` por exemplo. Nós vamos deixar isso como em exercício para o leitor.

Agora que vimos como podemos usar `get(..)` para emular uma ligação `[[Prototype]]`, vamos empurrar o hack um pouco mais. Em vez de um `[[Prorotype]]` circular, que tal múltiplas ligações `[[Prototype]]` (herança múltipla)? Isso acaba sendo bastante direto:

```js
var obj1 = {
		name: "obj-1",
		foo() {
			console.log( "obj1.foo:", this.name );
		},
	},
	obj2 = {
		name: "obj-2",
		foo() {
			console.log( "obj2.foo:", this.name );
		},
		bar() {
			console.log( "obj2.bar:", this.name );
		}
	},
	handlers = {
		get(target,key,context) {
			if (Reflect.has( target, key )) {
				return Reflect.get(
					target, key, context
				);
			}
			// fake multiple `[[Prototype]]`
			else {
				for (var P of target[
					Symbol.for( "[[Prototype]]" )
				]) {
					if (Reflect.has( P, key )) {
						return Reflect.get(
							P, key, context
						);
					}
				}
			}
		}
	},
	obj3 = new Proxy(
		{
			name: "obj-3",
			baz() {
				this.foo();
				this.bar();
			}
		},
		handlers
	);

// fake multiple `[[Prototype]]` links
obj3[ Symbol.for( "[[Prototype]]" ) ] = [
	obj1, obj2
];

obj3.baz();
// obj1.foo: obj-3
// obj2.bar: obj-3
```

**Observação** Como mencionado na nota do exemplo de `[[Prototype]]` circular anterior, nós não implementamos o manipulador `set(..)`, mas ele seria necessário para uma solução completa que emulasse cções `[[Set]]` como comportamentos do `[[Prototype]]`.

`obj3` está configurado como delegação múltipla para ambos `obj1` e `obj2`. No `obj3.baz()`, a chamada `this.foo()` termina puxando `foo()` de `obj1` (o primeiro que chegou, o primeiro a ser servido, mesmo que haja também um `foo ()` em `obj2`). Se nós reordenarmos a ligação como `obj2, obj1`, o `obj2.foo()` vai ser encontrado e utilizado. Mas, como está, a chamada `this.bar ()` não encontra uma `barra ()` em `obj1`, então cai para verificar` obj2`, onde encontra uma correspondência.

`obj1` e` obj2` representam duas cadeias paralelas `[[Protótipo]]` `obj3`. `obj1` e/ou` obj2` podiam ter uma delegação normal [[Protótipo]] "para outros objetos, ou mesmo poderia ser um proxy (como` obj3` é) que pode delegar múltiplas.

Assim como com o exemplo `[[Protótipo]]` circular anterior, as definições de `obj1`, 'obj2` e` obj3` são quase inteiramente separadas da lógica genérica de proxy que lida com a delegação múltipla. Seria trivial definir um utilitário como `setPrototypesOf (..)` (observe o "s"!) Que leva um objeto principal e uma lista de objetos para falsificar a ligação `[[Protótipo]]`. Mais uma vez, deixaremos isso como um exercício para o leitor.

Felizmente, o poder dos proxies agora está se tornando mais claro depois desses vários exemplos. Existem muitas outras tarefas poderosas de metaprogramação que os proxies habilitam.

## API `Reflect`

O objeto `Reflect` é um objeto simples (como `Math`), não uma função/construtor como outros intergrados nativos.

Ele possui funções estáticas que correspondem a várias tarefas de mataprogramação que você pode controlar. Essas funções correspondem uma a uma com métodos manipuladores (*armadilhas*) que Proxies podem definir.

Algumas das funções vão parecer familiares como funções de mesmo nome em `Object`:

* `Reflect.getOwnPropertyDescriptor(..)`
* `Reflect.defineProperty(..)`
* `Reflect.getPrototypeOf(..)`
* `Reflect.setPrototypeOf(..)`
* `Reflect.preventExtensions(..)`
* `Reflect.isExtensible(..)`

Essas utilidades em geral se comportam da mesma forma que seus `Object.*` equivalentes. Entretanto, uma diferença é que as equivalências de `Object.*` tentam coagir seu primeiro argumento (o objeto alvo) para um objeto se ele ainda não for um. Os métodos `Reflect` simplesmente lançam um erro nesse caso.

A chave de um objeto pode ser acessada/inspecionada usando essas utilidades:

* `Reflect.ownKeys(..)`: Retorna a lista de todas as chaves possuídas (não "herdado"), como retornado por ambos `Object.getOwnPropertyNames(..)` e `Object.getOwnPropertySymbols(..)`. Veja a seção "Ordem de enumeração de propriedade" para informações sobre a ordenação das chaves.
* `Reflect.enumerate(..)`: Retorna um iterador que produz a definição de todas chaves não-símbolos (próprias e "herdadas") que são *enumeráveis* (Veja o título *this & Object Prototypes* dessa série). Essencilamente, isso define se as chaves são as mesmas daquelas processadas por um loop `for..in`. Veja a seção Ordem de enumeração de propriedade" para informações sobre a ordenação das chaves.
* `Reflect.has(..)`: Essencialmente o mesmo que o operador `in` para verificar se uma propriedade está no objeto ou está na cadeia `[[Prototype]]`. Por exemplo, `Reflect.has(o,"foo")` essencialmente executa `"foo" in o`.

Chamadas de funções e invocação de construtores podem ser realizados manualmente, separados da sintaxe normal (e.g., `(..)` e `new`) usando essas utilidades:

* `Reflect.apply(..)`: Por exemplo, `Reflect.apply(foo,thisObj,[42,"bar"])` chama a função `foo(..)` com `thisObj` como se fosse `this`, e passa argumentos em `42` e `"bar'`.
* `Reflect.construct(..)`: Por exemplo, `Reflect.construct(foo,[42,"bar"])` essencialmente chama `new foo(42,"bar")`.

Acesso de propriedade do objeto, configuração, e exclusão podem ser feitas manualmente usando essas utilidades:

* `Reflect.get(..)`: Por exemplo, `Reflect.get(o,"foo")` recupera `o.foo`.
* `Reflect.set(..)`: Por exemplo, `Reflect.set(o,"foo",42)` essencilamente executa `o.foo = 42`.
* `Reflect.deleteProperty(..)`: Por exemplo, `Reflect.deleteProperty(o,"foo")` essencialmente executa `delete o.foo`.

As capacidades de `Reflect` da metaprogramação te dá equivalentes programáticos para emular várias funcionalidades sintáticas, expondo previamente operações abstratas ocultas. Por exemplo, você pode usar essas capacidades para extender funcionalidade e APIs para *domínio de linguagem específica* (domain specific labguages - DSLs).

### Ordenação de Propriedade

Antes do ES6, a ordem usada para listas chaves/propriedades de objetos eram implementações dependentes e indefinidas pela especificação. Geralmente, a maioria dos motores as enumeravam na ordem de criação, embora os desenvolvedores fossem fortemente encorajados a nunca confiar nessa ordem.

No ES6, a ordem para a listagem de propriedades próprias é agora definida (seção 9.1.12 da especificação ES6) pelo algoritmo `[[OwnPropertyKeys]]`, que produz todas as propriedades próprias (strings ou símbolos), independentemente da enumerabilidade. Essa ordem apenas é garantida pela `Reflect.ownKeys(..)` (e por extensão, `Object.getOwnPropertyNames(..)` e `Object.getOwnPropertySymbols(..)`).

A ordem é:

1. Primeiro, enumera-se quaisquer propriedades próprias que sejam índices inteiros, em ordem numérica ascendente.
2. Próximo, enumera-se o resto dos nomes das propriedades das strings próprias em ordem de criação.
3. Finalmente, enumera-se propriedades de símbolos próprios em ordem de criação.

Considere:

```js
var o = {};

o[Symbol("c")] = "yay";
o[2] = true;
o[1] = true;
o.b = "awesome";
o.a = "cool";

Reflect.ownKeys( o );				// [1,2,"b","a",Symbol(c)]
Object.getOwnPropertyNames( o );	// [1,2,"b","a"]
Object.getOwnPropertySymbols( o );	// [Symbol(c)]
```

Por outro lado, o algoritmo `[[Enumerate]]` (seção 9.1.11 da especificação ES6) produz apenas propriedades enumeradas, tanto do objeto alvo quanto da cadeia `[[Prototype]]`. Ela é usada por ambos `Reflect.enumerate(..)` e `for..in`. A ordem observável é de implementação dependente e não controlada pela especificação.

Em contraste, `Object.keys(..)` invoca o algoritmo `[[OwnPropertyKeys]]` para pegar uma lista de todas as chaves próprias. No entanto, ele filtra propriedades não enumeráveis e então reordena a lista para combinar o comportamento dependente da implementação legada, especificamente com `JSON.stringify(..)` e `for..in`. Então, por extensão a ordenação *também* corresponde ao de `Reflect.enumerate(..)`.

Em outras palavras, todos os quatro mecanismos (`Reflect.enumerate(..)`, `Object.keys(..)`, `for..in`, e `JSON.stringify(..)`) vão combinar com a mesma ondernação dependente da implementação, embora eles tecnicamente cheguem lá de maneiras diferentes.

Implementações são permitidas para combinar esses quatro para a ordenação de `[[OwnPropertyKeys]]`, mas eles também não são obrigatórios. No entanto, você provavelmente vai observar o seguinte comportamento da ordenação a partir deles:

```js
var o = { a: 1, b: 2 };
var p = Object.create( o );
p.c = 3;
p.d = 4;

for (var prop of Reflect.enumerate( p )) {
	console.log( prop );
}
// c d a b

for (var prop in p) {
	console.log( prop );
}
// c d a b

JSON.stringify( p );
// {"c":3,"d":4}

Object.keys( p );
// ["c","d"]
```

Resumindo tudo isso: A partir do ES6, `Reflect.ownKeys(..)`, `Object.getOwnPropertyNames(..)`, e `Object.getOwnPropertySymbols(..)` todos têm ordenação previsíveis e confiáveis garantidas pela especificação. Então é seguro contruir um código que confie nessa ordenação.

`Reflect.enumerate(..)`, `Object.keys(..)`, e `for..in` (assim como `JSON.stringification(..)` por extensão) continuam compartilhando uma ordem observável entre eles, como sempre fizeram. Mas essa ordem não vai necessariamente ser a mesma de `Reflect.ownKeys(..)`. Deve-se continuar a ter cuidado ao confiar nessa ordanação de implementação dependente.

## Teste de funcionalidade

O que é o teste de funcionalidade? É um teste que você pode rodar para determinar se uma funcionalidade está disponível ou não. Às vezes, o teste não é somente para existência, mas para conformidade de um comportamento especificado -- funcionalidades podem existir mas estarem bugadas.

Essa é uma técnica de metaprogramação, para testar o ambiente que seu programa roda e então determinar como seu programa deve se comportar.

O uso mais comum de teste de funcionalidade em JS é verificar pela existência de uma API e se ela não estiver presente, definir um polyfill (veja o Capítulo 1). Por exemplo:

```js
if (!Number.isNaN) {
	Number.isNaN = function(x) {
		return x !== x;
	};
}
```

A declaração `if` nesse snippet é metaprogramação: nós estamos provando nosso programa e estamos em ambiente de produção para determinar, se e como, nós devemos prosseguir.

Mas e testes para funcionalidades que envolvem sintaxe nova?

Você pode tentar algo como:

```js
try {
	a = () => {};
	ARROW_FUNCS_ENABLED = true;
}
catch (err) {
	ARROW_FUNCS_ENABLED = false;
}
```

Infelizmente, isso não funciona, porque nossos programas JS estão compilados. Portanto, o motor vai engasgar na sintaxe `() => {}` se a ES6 ainda não estiver suportando arrow functions. Ter um erro de sintaxe no seu programa impede ele de rodar, o que subsequentemente, impede seu programa de responder de forma diferente se uma funcionalidade for suportada ou não.

Para metaprogramar com testes de funcionalidades em torno de recursos relacionados à sintaxe, nós precisamos uma maneira de isolar o teste da etapa de compilação inicial em que nosso programa funciona. Por exemplo, se pudessemos armazenar o código para o teste em uma string, então o mecanismo JS não tentaria, por padrão, compilar o conteúdo dessa string, até que nós solicitássemos.

Sua mente simplesmente pensou em usar o `eval(..)`?

Não tão rápido. Veja o título *Escopo & Clausuras* dessa série para saber porque `eval(..)` é uma má ideia. Mas há outra opção com menos desvantagens: o construtor `Function(..)`.

Considere:

```js
try {
	new Function( "( () => {} )" );
	ARROW_FUNCS_ENABLED = true;
}
catch (err) {
	ARROW_FUNCS_ENABLED = false;
}
```

Ok, então agora estamos metaprogramando pela determinação se uma funcionalidade como as arrow functions *podem* compilar no motor atual ou não. Você deve estar se perguntando, o que nós iremos fazer com essa informação?

Com a verificação por APIs existentes, e definindo polyfills como API fallbacks, há um caminho claro para o que fazer com testes tanto bem sucedidos como fracassados. Mas o que nós podemos fazer com a informação que nós pegamos de `ARROW_FUNCS_ENABLED` sendo `true` ou `false`?

Por conta da sintaxe não poder aparecer em um arquivo se o motor não suportar essa funcionalidade, você não pode apenas ter funções diferentes definidas no arquivo com e sem a sintaxe em questão.

O que você pode fazer é usar o teste para determinar qual dos conjuntos de arquivos JS você deve carregar. Por exemplo, se você tem um conjunto desses testes de funcionalidade em um bootstraper para sua aplicação JS, ele poderá então testar o ambiente para determinar se seu código ES6 pode ser carregado e executado diretamente, ou se você precisa carregar uma versão transpilada do seu código (veja o Capítulo 1).

Essa técnica é chamada *entrega dividida*.

Ela reconhece a realidade que seu programa criado em JS ES6 vai, às vezes, ser capaz de rodar inteiramente *nativo* em navegadores ES6+, mas outras vezes precisará de transpilação para rodar em navegadores pré-ES6. Se você sempre carregar e usar o código transpilado, mesmo em ambientes no novo compilador ES6, você estará rodando um código sub otimizado, pelo menos por um tempo. Isso não é o ideal.

Entrega dividida é mais complicada e sofisticada, mas ela representa uma abordagem mais robusta e madura para cobrir a falha ente o código que você escreve e a funcionalidade suportada em navagadores que seu programa precisa rodar.

### FeatureTests.io

Definir testes de funcionalidade para todas as sintaxes do ES6+, bem como os comportamentos semânticos, é uma tarefa árdua que vocẽ provavelmente não vai querer enfrentar. Como esses teste requerem compilação dinâmica (`new Function(..)`), há alguns custos de performance infelizes.

Além disso, a execução desses testes a cada vez que seu app rodar é desperdício, uma vez que, em média, o navegador de um usuário comum atualiza apenas uma vez, em um período de várias semanas no máximo, e mesmo assim, novas funcionalidades não estão necessariamente aparecendo com cada atualização.

Finalmente, gerenciando a lista de testes de recursos que se aplicam à sua base de código específica -- raramente seus programas vão fazer uso inteiramente do ES6 -- é indisciplinado e propenso à erros.

O feature-tests-as-a-service "https://featuretests.io" oferece soluções para essas frustações.

Você pode carregar a biblioteca do serviço na sua página, e ele irá carregar as últimas definições de testes e rodar todos os testes de funcionalidades.  Ele faz isso usando processamento em segundo plano com Web Workers, se possível, para reduzir a sobrecarga de desempenho. Eele também usa o LocalStorage para cachear os resultados de uma maneira que podem ser compartilhados por todos os sites que vocẽ visitar que usam o serviço, o que reduz drasticamente quantas vezes os testes precisam ser executados em cada instância do navegador.

Você obtém testes de funcionalidade em tempo de execução em cada um dos navegadores de seus usuários, e você pode usar esses resultados de testes de forma dinâmica para servir aos usuários o código mais apropriado (não mais, nem menos) para seus ambientes.

Além disso, o serviço fornece ferramentas e APIs para escanear seus arquivos para determinar quais funcionalidades você precisa, então você consegue automatizar completamente seus processos de compilação de entrega dividida.

FeatureTests.io torna prático usar testes de funcionalidade para todas as partes do ES6 e além para ter certeza que apenas o melhor código será sempre carregado e executado para qualquer ambiente fornecido.

## Otimização da chamada de cauda (Tail Call Optimization - TCO)

Normalmente quando a chamada de uma função é feita de dentro de outra função, um segundo *quadro de pilha* (stack frame) é alocado para administrar separadamente as variáveis/estados daquela outra invocação da função. Essa alocação não apenas custa algum tempo de processamento, mas ela também toma uma memória extra.

Uma cadeia de quadros de pilha tipicamente têm no máximo 10-15 saltos de uma função para outra e outra. Nesses cenários, o uso da memória provavelmente não é nenhum tipo de problema prático.

Entretanto, quando você considera a programação recursiva (uma função chamando a si mesma repetidamente) -- ou recursão mútua com duas ou mais funções chamando uma a outra -- a pilha de chamadas pode facilmente ser centenas, milhares, ou mais níveis profundos. Você pode provavelmente ver os problemas que isso poderia causar, se o uso da memória cresce indiscriminadamente.

Motores JavaScript têm que definir limites arbirtrários para prevenir certas técnicas de programação de quebrar na execução e deixar o navagador e dispositivo sem memória. Essa é a causa porque temos o erro frustante *"RangeError: Maximum call stack size exceeded"* (Erro de Alcance: Tamanho máximo de pilha de chamadas excedido) lançado se esse limite for atingido.

**Atenção** O limite de chamadas da pilha não é controlado pela especificação. Isso é uma implementação dependente, e vai variar entre os navegadores e dispositivos. Você nunca deve codificar com fortes pressupostos de limites exatos observados, e eles podem muito bem mudar de versão para versão.

Certos padrões de chamadas de função, chamadas *chamadas de cauda* (tail calls), podem ser otimizadas em uma maneira para evitar a alocação extra de quadros de pilha. Se uma alocação extra pode ser evitada, não há razão para limitar arbitrariamente a profundidade de uma pilha de chamadas, então os motores podem deixá-las rodar livremente.

Uma chamada de cauda é uma declaração `return` com uma chamada de função, onde nada precisa acontecer após a chamada exceto o retorno do valor.

Essa otimização apenas pode ser aplicada em modo `strict`. Ainda outra razão para sempre escrever todo o seu código como `strict`!

Aqui está uma chamada de função que *não* é uma posição de cauda:

```js
"use strict";

function foo(x) {
	return x * 2;
}

function bar(x) {
	// não é uma chamada de cauda
	return 1 + foo( x );
}

bar( 10 );				// 21
```

`1 + ..` deve ser executado depois que a chamada de `foo(x)` completar, então o estado dessa invocação `bar(..)` precisa ser preservado.

Mas o snippet seguinte demonstra chamadas para `foo(..)` e `bar(..)` onde ambas *são* em posição de cauda, como elas são as últimas coisas que acontecem nesse caminho de código (diferente do `return`):

```js
"use strict";

function foo(x) {
	return x * 2;
}

function bar(x) {
	x = x + 1;
	if (x > 10) {
		return foo( x );
	}
	else {
		return bar( x + 1 );
	}
}

bar( 5 );				// 24
bar( 15 );				// 32
```

Nesse programa, `bar(..)` é claramente recursiva, mas `foo(..)` é aoenas uma chamada de função normal. Em ambos os casos, as chamadas das funções estão em *poisção correta da cauda (PTC)*. O `x +1` é avaliado antes da chamada de `bar(..)`, e sempre que essa chamada termina, tudo que acontece é o `return`.

Posições corretas de cauda *(PTC - proper tail calls)* dessas formas podem ser otimizadas -- chamado "otimização de chamada de cauda" -- visto isso, alocação de quadros de pilha extra são desnecessários. Em vez de criar uma novo quadro de pilha para a próxima chamada de função, o motor apenas reusa o quadro de pilha já existente. Isso funciona porque uma função não precisa preservar nenhum dos estados atuais, já que nada acontece com esse estado depois da posição correta de cauda.

Otimização da chamada de cauda significa que praticamente não há limites para quão profundo a pilha de chamadas pode ser. Esse truque melhora ligeiramente as chamadas de funções regulares em programas normais. mas mais importante, abre as portas para o uso de recursão para expressões do programa mesmo se a pilha de chamadas possa ser de milhares de chamadas de profundidade.

Nós não somos mais relegados para simplesmente teorizar sobre a recursão para a resolução de problemas, mas podemos realmente usá-los em programas reais de JavaScript!

A parir do ES6, todas as PTC devem ser otimizadas dessa forma, recursivamente ou não.

### Substituição de Chamada de Cauda

O entrave, no entanto, é que apenas o PTC pode ser otimizado; não PTC vão continuar funcionando, é claro, mas vão causar alocações de quadros de pilha como sempre fizeram. Você terá que ser cuidadoso sobre estruturar suas funções com PTC se você espera que as otimizações aconteçam.

Se você tem uma função que não é escrita em PTC, você poderá encontrar a necessidade de rearranjar seu código manualmente para que ele seja elegível para TCO.

Considere:

```js
"use strict";

function foo(x) {
	if (x <= 1) return 1;
	return (x / 2) + foo( x - 1 );
}

foo( 123456 );			// RangeError
```

A chamada para `foo(x-1)` não é um PTC porque seu resultado tem que ser adicionado em `(x / 2)` antes de `return`.

Porém, para fazer com que esse código seja elegível ao TCO em um motor ES6, nós podemos sobrescreve-lo como a seguir:

```js
"use strict";

var foo = (function(){
	function _foo(acc,x) {
		if (x <= 1) return acc;
		return _foo( (x / 2) + acc, x - 1 );
	}

	return function(x) {
		return _foo( 1, x );
	};
})();

foo( 123456 );			// 3810376848.5
```

Se você rodar o snippet anterior em uma motor ES6 que implmente o TCO, você vai ter a resposta `3810376848.5` como mostrado. Porém, ela vai continuar falhando com um `RangeError` em motores não TCO.

### Otimizações não TCO

Há outras técnicas para reescrever o código de forma que a pilha de chamadas não cresça a cada chamada.

Uma destas técnicas é chamada *tampolim*(trampolining), o que equivale a ter cada resultado parcial representado como uma função que retorna outra função de resultado parcial ou resultado final. Então você pode simplesmente ter um loop, até que você pare de ter uma função, e você vai ter o resultado.

Considere:

```js
"use strict";

function trampoline( res ) {
	while (typeof res == "function") {
		res = res();
	}
	return res;
}

var foo = (function(){
	function _foo(acc,x) {
		if (x <= 1) return acc;
		return function partial(){
			return _foo( (x / 2) + acc, x - 1 );
		};
	}

	return function(x) {
		return trampoline( _foo( 1, x ) );
	};
})();

foo( 123456 );			// 3810376848.5
```

Esse retrabalho requer mudanças mínimas para avaliar a recursão no loop em `tramponine(..)`:

1. Primeiro, nós embrulhamos a linha `return _foo ..` na espressão da função `return partial() { ..`.
2. Então embrulhamos a chamada `_foo(1,x)` na chamada `trampoline(..)`.

A razão para essa técnica não sofrer a limitação de pilha de chamada é que cada uma dessas funções `partial(..)` está apenas retornando para o loop `while` em `trampoline(..)`, que executa-os e então o loop interage novamente. Em outras palavras, `partial(..)` não chama ela mesma recursivamente, ela retorna outra função. A profundidade da pilha continua constante, então ela pode rodar o quanto ela precisar.

O trampolim expresso dessa forma usa o fechamento que a função `partial()` tem sobre as variáveis `x` e `acc` para manter o estado de iteraçãi para iteração. A vantagem é que a lógica de looping é posta para uma função `tampoline(..)` de utilidade reusável, o que muitas bibliotecas oferecem versões destas. Você pode reusar `trampoline(..)` múltiplas vezes no seu programa com diferentes algoritmos de trampolim.

É claro, se você realmente quiser otimizar profundamente (e a reusabilidade não for uma preocupação), você pode descartar o estado de fechamento e alinhar o rastreamento de estado de `acc` em apenas um escopo de função, além de um loop. Essa técnica é geralmete chamada de *desenrolamento de recursão (recursion unrolling)*:

```js
"use strict";

function foo(x) {
	var acc = 1;
	while (x > 1) {
		acc = (x / 2) + acc;
		x = x - 1;
	}
	return acc;
}

foo( 123456 );			// 3810376848.5
```

Essa expressão do algoritmo é mais simples de ler, e vai provavelmente performar melhor (estritamente falando)de várias formas que vamos explorar. Isso pode parecer claro como água, e te fazer pensar no porque você tentaria outras abordagens.

Há algumas razões do porque você não deveria querer sempre desenrolar suas recursões manualmente:

* Em vez de fabricar sua lógica de trampolin (loop) para reusabilidade, nós vamos alinhar isso. Isso funciona muito bem quando há apenas um exemplo a se considerar, mas assim que você tiver meia dúzia ou mais existem muito mais complicações em algoritmos de recursão, assim como recursões mútuas (mais do que apenas uma função chamando a si mesma).

	Quanto mais fundo você vai nessa toca de coelho, mais manuais e instricadas o *desenrolar* das otimizações são. Você perderá rapidamente todo o valor percebido da legibilidade. A primeira vantagem da recursão mesmo na forma PTC, é que ela preserva a legibilidade do algoritmo, e descarrega a otimização de desempenho para o motor.

Se vocẽ escrever algoritmos com PTC, o motor ES6 vai aplicar o TCO para deixar seu código rodar em profundidade de pilha constante (reusando quadros de pilhas). Você ganha legibilidade da recursão com a maioria dos benefícios de performance e sem limitação de largura de execução.

### Meta?

O que TCO tem a ver com metaprogramação?

Assim como abordamos na seção anterior "Teste de funcionalidade", você pode determinar em tempo de execução quais funcionalidade o motor suporta. Isso inclui TCO, apesar de determinar que isso é uma forã bem bruta.

Considere:

```js
"use strict";

try {
	(function foo(x){
		if (x < 5E5) return foo( x + 1 );
	})( 1 );

	TCO_ENABLED = true;
}
catch (err) {
	TCO_ENABLED = false;
}
```

Em um motor não TCO, o loop recursivo vai falhar eventualmente, lançando uma *exception caught* através de `try..catch`. Do contrário, o loop se completa facilmente graças a TCO.

Uhuuu, certo?

Mas como metaprogramar sobre uma funcinalidade TCO (ou melhor, a falta dela) beneficiaria nosso código? A respostam mais simples é que vocẽ poderia usar um teste de funcionalidade para decidir carregar uma versão do código da sua aplicação que usa recursão, ou uma alternativa que foi convertida/transpilada para não precisar de recursão.

#### Código auto ajustável

Mas aqui está outra maneira de olhar para o problema:

```js
"use strict";

function foo(x) {
	function _foo() {
		if (x > 1) {
			acc = acc + (x / 2);
			x = x - 1;
			return _foo();
		}
	}

	var acc = 1;

	while (x > 1) {
		try {
			_foo();
		}
		catch (err) { }
	}

	return acc;
}

foo( 123456 );			// 3810376848.5
```

Esse algoritmo funciona tentando fazer a maior parte possível do trabalho com recursão, mas mantendo o trilho de progresso via variáveis de escopo `x` e `acc`. Se todo o problema pode ser resolvido com recursão sem nenhum erro, ótimo. Se o motor matar a recursão em algum ponto, nós simplesmente o pegaremos com `try..catch` e tentaremos de novo, continuando de onde paramos.

Eu considero essa uma forma de metaprogramação em que você está provando durante a execução a habilidade do motor de finalizar (recursivamente) totalmente a tarefa, e trabalhando sobre qualquer limitação de motor (não TCO) que podem te restringir.

A primeria vista (ou até na segunda!), minha aposta é que esse código paracerá muito mais feio comparado com algumas versões mais recentes. Ele também roda um pouco mais devagar (e mais ainda em ambientes não TCO).

A primeira vantagem, além de poder completar uma tarefa de qualquer tamanho em motores não TCO, é esta uma "solução" para a limitação de pilha de recursão é muito mais flexível que o trampolim ou técnicas de desenrolamento manual mostradas anteriormente.

Essencilamente, `_foo()` nesse caso é um tipo de *stand-in* para praticamente qualquer tarefa, mesmo recursão mútua. O resto é o boilerplate que deve funcionar praticamente em qualquer algoritmo.

A única "pegadinha" é de poder retomar do caso de um limite de recursão ser batido, o estado da recursão deve estar nas variáveis do escopo que existem fora das funções recursivas. Nós fizemos isso deixando o `x` e `acc` fora da função `_foo()`, em vez de passá-las como argumentos para `_foo()` como antes.

Quase todos algoritmos recursivos podem ser adaptados para trabalhar dessa forma. Isso significa que é a aplicação mais ampla de alavancar o TCO com recursão em seus programas, com o mínimo de reescrita.

Essa abordagem continua usando PTC, o que significa que esse código vai *aumentará progressivamente* de execução usando o loop muitas vezes (lotes de recursão) em um navegador mais antigo para alavancar completamente a recursão TCO em um ambiente ES6+. Eu acho que isso é bem legal!

## Revisão

Metaprogramação é quando você vira a lógica do seu programa para focar nele mesmo (ou em seu ambiente), tanto para inspecionar sua própria estrutura quanto para modificá-la. O valor primário da metaprogramação é extender os mecanismos normais da linguagem para forcer capacidades adicionais.

Antes do ES6, o Javascript já tinha um pouco de capacidade de metaprogramação, mas o ES6 aumenta significativamente com vários novos recursos.

Desde inferências com nomes de funções anônimas para meta propriedades que te dá informação sobre coisas, sobre como um construtor é invocado, você pode inspecionar a estrutra do programa enquando ele executa muito mais do que antes. Símbolos bem conhecidos te permite subscrever comportamentos intrísecos, como coerção de uma objeto para um valor primitivo. Proxies podem interceptar e personalizar várias operações de baixo-nível em objetos, e `Reflect` oferece recusos para emula-las.

Teste de funcionalidade, mesmo para comportamentos semânticos sutis, como Otimização da chamada de cauda, alterna o foco da metaprogramação do seu programa para as próprias capacidades do motor Javascript. Conhecendo mais do que o ambiente pode fazer, seus programas podem se auto ajustar à melhor combinação enquando eles executam.

Você deveria metaprogramar? Meu conselho é: primeiro foque em como as macânicas fundamentais da linguagem realmente funcionam. Mas uma vez que você conhece completamente o que o próprio JS pode fazer, é hora de começar a alavancar essa poderosa capacidade de metaprogramação para aumentar a linguagem.