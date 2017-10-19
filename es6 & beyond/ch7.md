# You Don't Know JS: ES6 & Além
# Capítulo 7: Metaprogramação

Metaprogramação é programação onde a operação visa o comportamento do próprio programa. Em outras palavras, é programar a programação do seu programa. Sim, um bocado, não é?

Por exemplo, se você investigar a relação entre um objeto `a` e outro `b` -- eles são `[[Prototype]]` conectados? -- usando `a.isPrototype(b)`, isso é comumente referido como instrospecão, uma forma de metaprogramação. Macros (que não existem em JS, ainda) -- onde o código se modifica no momento da compilação -- são outros exemplos óbvios de metaprogramação.
Enumerar as chaves de um objeto com um loop `for..in`, ou verificar se um objeto é uma *instância de* um "construtor de uma classe", são outros tipos comuns de metaprogramação.

Metaprogramação concentra-se em um ou mais dos seguintes itens: inspeção de código, modificação do código, ou código que modifica o comportamento padrão da linguagem, de modo que outro código seja afetado.

O objetivo da metaprogramação é aproveitar as capacidades intrísecas para tornar o resto do seu código mais descritivo, expressivo, e/ou flexível· Por conta da natureza do termo *meta* de metaprogramação, é um tanto difícil de colocar uma definição mais precisa do que essa. A melhor forma de entender metaprogramação é ve-la através de exemplos.

A ES6 adiciona várias novos métodos/funcionalidades para metaprogramação em cima do que o JS já possuía.

## Nome de Funções

Existes casos em que seu código pode querer se voltar pra si mesmo e perguntar qual o nome de alguma função. Se você perguntar qual o nome da função, a reposta será algo surpreendentemente abíguo. Considere:

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

Há várias maneiras nas quais funções podem ser expressadas em programas, e nem sempre está claro e sem abiguidade qual o "nome" que a função deveria ter.

Mais importante, nós precisamos distinguir se o nome de uma função se refere ao `name` da sua propriedade -- sim, função tem uma propriedade chamada `name` -- ou o que quer que se refira ao *nome da ligação léxica*(lexical binding name), assim como `bar` em `function bar() { .. }`.

O nome da ligação léxica é o que você usa para coisas como recursão: 

```js
function foo(i) {
	if (i < 10) return foo( i * 2 );
	return i;
}
```

A propriedade `name` é o que você usaria para os propósitos da metaprogramação, então é o que vamos focar nessa discussão.

A confusão aparece porque por padrão, o nome léxico que a função tinha (se algum) também foi definido como a propriedade do seu `name`. Atualmente não há requerimentos oficiais para esse comportamento pela especificação ES5 (e anterior). A definição da propriedade `name` não era padrão, mas ainda era bastante confiável. A partir da ES6, isso foi padronizado.

**Dica** Se a função tem um valor de `name` atribuído, esse é tipicamente o nome usado em traços de pilha nas ferramentas dos desenvolvedores.

### Inferferências

Mas o que acontece com o nome da propriedade `name` se uma função não tiver um nome léxico?

A partir da ES6, há agora regras de interferência que podem determinar um nome sensível da propriedade `name` para atribuir à uma função, mesmo se essa função não tiver um nome léxico para usar.

Considere:

```js
var abc = function() {
	// ..
};

abc.name;	// "abc"
```

Nós demos à função um nome léxico como `abc = function def() { .. }`, a propriedade `name` será com certeza `"def"`. Mas, na ausência de um nome léxico, intuitivamente o nome `"abc"` parece ser apropriado.

Aqui estão outras formas que inferião um nome (ou não) no ES6:

```js
(function(){ .. });					// nome:
(function*(){ .. });				// nome:
window.foo = function(){ .. };		// nome:

class Awesome {
	constructor() { .. }			// nome: Awesome
	funny() { .. }					// nome: funny
}

var c = class Awesome { .. };		// name: Awesome

var o = {
	foo() { .. },					// nome: foo
	*bar() { .. },					// nome: bar
	baz: () => { .. },				// nome: baz
	bam: function(){ .. },			// nome: bam
	get qux() { .. },				// nome: get qux
	set fuz() { .. },				// nome: set fuz
	["b" + "iz"]:
		function(){ .. },			// nome: biz
	[Symbol( "buz" )]:
		function(){ .. }			// nome: [buz]
};

var x = o.foo.bind( o );			// nome: bound foo
(function(){ .. }).bind( o );		// nome: bound

export default function() { .. }	// nome: default

var y = new Function();				// nome: anônima
var GeneratorFunction =
	function*(){}.__proto__.constructor;
var z = new GeneratorFunction();	// nome: anônima
```

A propriedade `name` não é editável por padrão, mas é configurável, significando que você pode usar `Object.defineProperty(..)` para mudar manualmente se desejar.

## Meta Propriedades

Na seção "`new.target`" do Capítulo 3, nós introduzimos um conceito novo para o JS no ES6: a meta propriedade. Como o nome sugeste, meta propriedades têm a intenção de fornencer meta informações especiais na forma de acesso de uma propriedade que de outra forma não teria sido possível.

No caso de `new.target`, a palavra-chave `new` serve como contexto para o acesso de uma propriedade. Claramente `new` não é um objeto propriamente, o que torna essa capacidade especial. Entretanto, quando `new.target` é usado dentro de uma chamada de um construtor (uma função/método invocado com `new`), `new` se torna um contexto virtual, tanto que `new.target` pode se referir ao construtor de destino no qual `new` foi invocado.

Esse é um exemplo claro de uma operação de meta programação, como a intenção é determinar a chamada a partir de dentro de um construtor no qual a origem `new` estava, geralmente para propósitos de instrospecção (examinar tipagem/estrutura) ou acesso de propriedades estáticas.

Por exemplo, você pode querer ter diferentes comportamentos em um construtor, dependendo se ele foi invocado diretamente ou invocado através do filho de uma classe:

```js
class Parent {
	constructor() {
		if (new.target === Parent) {
			console.log( "Pai instanciado" );
		}
		else {
			console.log( "Um filho instanciado" );
		}
	}
}

class Child extends Parent {}

var a = new Parent();
// Pai instanciado

var b = new Child();
// Um filho instanciado
```

Há uma pequena nuance aqui, qual é o `constructor()` dentro da definição da classe `Parent`, é atualmente dado o nome léxico da classe (`Parent`), mesmo que a sintaxe signifique que a classe é uma entidade separada do construtor.

**Atenção** Assim como todas técnicas de meta programação, tenha cuidado na criação de códigos que sejam muito espertos para a manutenabilidade e entendimento no seu próprio futuro ou de outros. Use esses macetes com cautela.

## Símbolos bem conhecidos

Na seção "Símbolos" do Capítulo 2, nós abordamos o novo tipo primitivo `symbol` do ES6. Além dos símbolos que você pode definir em seu próprio programa, o JS predefine uma série de símbolos internos, conhecidos como *Símbolos bem conhecidos* (WKS - *do original*).

Esses valores de símbolos são definidos, primeiramente, para expor uma meta propriedade especial que está sendo exposta em seus programas JS, para te dar mais controle sobre o comportamento do JS.

Nós vamos introduzir brevemente a cada um e discutir suas propostas.

### `Symbol.iterator`

No Capítulo 2 e 3, nós apresentamos e usamos o símbolo `@@iterator`, automaticamente usado pelos spreads `...` e loops `for..of`. Nós também vimos `@@iterator` definidos nas coleções no novo ES6, como mostrado no Capítulo 5.

`Symbol.iterator` representa a localização especial (propriedade) de qualquer objeto em que o mecanismo da lignuagem olha e automaticamente acha um método que vai construir uma instânica de iteração para consumir os valores daquele objeto. Muitos objetos vêm com um valor definido como padrão.

Entretanto, nós podemos definir a nossa própria lógica de iterador para qualquer valor de objeto definindo a propriedade `Symbol.iterator`, mesmo se isso esiver sibstituindo o iterador padrão. O apecta da meta programação é que nós estamos definindo comportamentos que outras partes do JS (nomeclatura, operadores e loopings construtores) usam quando peocessam o valor de um objeto que nós definimos.

Considere:

```js
var arr = [4,5,6,7,8,9];

for (var v of arr) {
	console.log( v );
}
// 4 5 6 7 8 9

// define o iterador que produz apenas valores
// de índices estranhos
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

Umas das tarefas de metaprogramação mais comuns é inspecionar um valor para descobrir qual *tipo* ele é, geralmente para decidir quais operadores são apropriados para atuar neles. Com objetos, as duas técnicas de inspeção mais comuns são `toString()` e `instanceof`.

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

O símbolo `@@toString` no prototype (ou na própria instância) especifica o valor da string para uso no `[object ___]` como stringification.

O símbolo `@@hasInstance` é um método da função construtora que recebe o valor da instância de um objeto e deixa vocçê decidir se retorna `true` ou `false` se o valor poderá ser considerado uma instância ou não.

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

## Feature Testing

What is a feature test? It's a test that you run to determine if a feature is available or not. Sometimes, the test is not just for existence, but for conformance to specified behavior -- features can exist but be buggy.

This is a meta programming technique, to test the environment your program runs in to then determine how your program should behave.

The most common use of feature tests in JS is checking for the existence of an API and if it's not present, defining a polyfill (see Chapter 1). For example:

```js
if (!Number.isNaN) {
	Number.isNaN = function(x) {
		return x !== x;
	};
}
```

The `if` statement in this snippet is meta programming: we're probing our program and its runtime environment to determine if and how we should proceed.

But what about testing for features that involve new syntax?

You might try something like:

```js
try {
	a = () => {};
	ARROW_FUNCS_ENABLED = true;
}
catch (err) {
	ARROW_FUNCS_ENABLED = false;
}
```

Unfortunately, this doesn't work, because our JS programs are compiled. Thus, the engine will choke on the `() => {}` syntax if it is not already supporting ES6 arrow functions. Having a syntax error in your program prevents it from running, which prevents your program from subsequently responding differently if the feature is supported or not.

To meta program with feature tests around syntax-related features, we need a way to insulate the test from the initial compile step our program runs through. For instance, if we could store the code for the test in a string, then the JS engine wouldn't by default try to compile the contents of that string, until we asked it to.

Did your mind just jump to using `eval(..)`?

Not so fast. See the *Scope & Closures* title of this series for why `eval(..)` is a bad idea. But there's another option with less downsides: the `Function(..)` constructor.

Consider:

```js
try {
	new Function( "( () => {} )" );
	ARROW_FUNCS_ENABLED = true;
}
catch (err) {
	ARROW_FUNCS_ENABLED = false;
}
```

OK, so now we're meta programming by determining if a feature like arrow functions *can* compile in the current engine or not. You might then wonder, what would we do with this information?

With existence checks for APIs, and defining fallback API polyfills, there's a clear path for what to do with either test success or failure. But what can we do with the information that we get from `ARROW_FUNCS_ENABLED` being `true` or `false`?

Because the syntax can't appear in a file if the engine doesn't support that feature, you can't just have different functions defined in the file with and without the syntax in question.

What you can do is use the test to determine which of a set of JS files you should load. For example, if you had a set of these feature tests in a bootstrapper for your JS application, it could then test the environment to determine if your ES6 code can be loaded and run directly, or if you need to load a transpiled version of your code (see Chapter 1).

This technique is called *split delivery*.

It recognizes the reality that your ES6 authored JS programs will sometimes be able to entirely run "natively" in ES6+ browsers, but other times need transpilation to run in pre-ES6 browsers. If you always load and use the transpiled code, even in the new ES6-compliant environments, you're running suboptimal code at least some of the time. This is not ideal.

Split delivery is more complicated and sophisticated, but it represents a more mature and robust approach to bridging the gap between the code you write and the feature support in browsers your programs must run in.

### FeatureTests.io

Defining feature tests for all of the ES6+ syntax, as well as the semantic behaviors, is a daunting task you probably don't want to tackle yourself. Because these tests require dynamic compilation (`new Function(..)`), there's some unfortunate performance cost.

Moreover, running these tests every single time your app runs is probably wasteful, as on average a user's browser only updates once in a several week period at most, and even then, new features aren't necessarily showing up with every update.

Finally, managing the list of feature tests that apply to your specific code base -- rarely will your programs use the entirety of ES6 -- is unruly and error-prone.

The "https://featuretests.io" feature-tests-as-a-service offers solutions to these frustrations.

You can load the service's library into your page, and it loads the latest test definitions and runs all the feature tests. It does so using background processing with Web Workers, if possible, to reduce the performance overhead. It also uses LocalStorage persistence to cache the results in a way that can be shared across all sites you visit which use the service, which drastically reduces how often the tests need to run on each browser instance.

You get runtime feature tests in each of your users' browsers, and you can use those tests results dynamically to serve users the most appropriate code (no more, no less) for their environments.

Moreover, the service provides tools and APIs to scan your files to determine what features you need, so you can fully automate your split delivery build processes.

FeatureTests.io makes it practical to use feature tests for all parts of ES6 and beyond to make sure that only the best code is ever loaded and run for any given environment.

## Tail Call Optimization (TCO)

Normally, when a function call is made from inside another function, a second *stack frame* is allocated to separately manage the variables/state of that other function invocation. Not only does this allocation cost some processing time, but it also takes up some extra memory.

A call stack chain typically has at most 10-15 jumps from one function to another and another. In those scenarios, the memory usage is not likely any kind of practical problem.

However, when you consider recursive programming (a function calling itself repeatedly) -- or mutual recursion with two or more functions calling each other -- the call stack could easily be hundreds, thousands, or more levels deep. You can probably see the problems that could cause, if memory usage grows unbounded.

JavaScript engines have to set an arbitrary limit to prevent such programming techniques from crashing by running the browser and device out of memory. That's why we get the frustrating "RangeError: Maximum call stack size exceeded" thrown if the limit is hit.

**Warning:** The limit of call stack depth is not controlled by the specification. It's implementation dependent, and will vary between browsers and devices. You should never code with strong assumptions of exact observable limits, as they may very well change from release to release.

Certain patterns of function calls, called *tail calls*, can be optimized in a way to avoid the extra allocation of stack frames. If the extra allocation can be avoided, there's no reason to arbitrarily limit the call stack depth, so the engines can let them run unbounded.

A tail call is a `return` statement with a function call, where nothing has to happen after the call except returning its value.

This optimization can only be applied in `strict` mode. Yet another reason to always be writing all your code as `strict`!

Here's a function call that is *not* in tail position:

```js
"use strict";

function foo(x) {
	return x * 2;
}

function bar(x) {
	// not a tail call
	return 1 + foo( x );
}

bar( 10 );				// 21
```

`1 + ..` has to be performed after the `foo(x)` call completes, so the state of that `bar(..)` invocation needs to be preserved.

But the following snippet demonstrates calls to `foo(..)` and `bar(..)` where both *are* in tail position, as they're the last thing to happen in their code path (other than the `return`):

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

In this program, `bar(..)` is clearly recursive, but `foo(..)` is just a regular function call. In both cases, the function calls are in *proper tail position*. The `x + 1` is evaluated before the `bar(..)` call, and whenever that call finishes, all that happens is the `return`.

Proper Tail Calls (PTC) of these forms can be optimized -- called tail call optimization (TCO) -- so that the extra stack frame allocation is unnecessary. Instead of creating a new stack frame for the next function call, the engine just reuses the existing stack frame. That works because a function doesn't need to preserve any of the current state, as nothing happens with that state after the PTC.

TCO means there's practically no limit to how deep the call stack can be. That trick slightly improves regular function calls in normal programs, but more importantly opens the door to using recursion for program expression even if the call stack could be tens of thousands of calls deep.

We're no longer relegated to simply theorizing about recursion for problem solving, but can actually use it in real JavaScript programs!

As of ES6, all PTC should be optimizable in this way, recursion or not.

### Tail Call Rewrite

The hitch, however, is that only PTC can be optimized; non-PTC will still work of course, but will cause stack frame allocation as they always did. You'll have to be careful about structuring your functions with PTC if you expect the optimizations to kick in.

If you have a function that's not written with PTC, you may find the need to manually rearrange your code to be eligible for TCO.

Consider:

```js
"use strict";

function foo(x) {
	if (x <= 1) return 1;
	return (x / 2) + foo( x - 1 );
}

foo( 123456 );			// RangeError
```

The call to `foo(x-1)` isn't a PTC because its result has to be added to `(x / 2)` before `return`ing.

However, to make this code eligible for TCO in an ES6 engine, we can rewrite it as follows:

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

If you run the previous snippet in an ES6 engine that implements TCO, you'll get the `3810376848.5` answer as shown. However, it'll still fail with a `RangeError` in non-TCO engines.

### Non-TCO Optimizations

There are other techniques to rewrite the code so that the call stack isn't growing with each call.

One such technique is called *trampolining*, which amounts to having each partial result represented as a function that either returns another partial result function or the final result. Then you can simply loop until you stop getting a function, and you'll have the result. Consider:

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

This reworking required minimal changes to factor out the recursion into the loop in `trampoline(..)`:

1. First, we wrapped the `return _foo ..` line in the `return partial() { ..` function expression.
2. Then we wrapped the `_foo(1,x)` call in the `trampoline(..)` call.

The reason this technique doesn't suffer the call stack limitation is that each of those inner `partial(..)` functions is just returned back to the `while` loop in `trampoline(..)`, which runs it and then loop iterates again. In other words, `partial(..)` doesn't recursively call itself, it just returns another function. The stack depth remains constant, so it can run as long as it needs to.

Trampolining expressed in this way uses the closure that the inner `partial()` function has over the `x` and `acc` variables to keep the state from iteration to iteration. The advantage is that the looping logic is pulled out into a reusable `trampoline(..)` utility function, which many libraries provide versions of. You can reuse `trampoline(..)` multiple times in your program with different trampolined algorithms.

Of course, if you really wanted to deeply optimize (and the reusability wasn't a concern), you could discard the closure state and inline the state tracking of `acc` into just one function's scope along with a loop. This technique is generally called *recursion unrolling*:

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

This expression of the algorithm is simpler to read, and will likely perform the best (strictly speaking) of the various forms we've explored. That may seem like a clear winner, and you may wonder why you would ever try the other approaches.

There are some reasons why you might not want to always manually unroll your recursions:

* Instead of factoring out the trampolining (loop) logic for reusability, we've inlined it. This works great when there's only one example to consider, but as soon as you have a half dozen or more of these in your program, there's a good chance you'll want some reusability to keep things shorter and more manageable.
* The example here is deliberately simple enough to illustrate the different forms. In practice, there are many more complications in recursion algorithms, such as mutual recursion (more than just one function calling itself).

   The farther you go down this rabbit hole, the more manual and intricate the *unrolling* optimizations are. You'll quickly lose all the perceived value of readability. The primary advantage of recursion, even in the PTC form, is that it preserves the algorithm readability, and offloads the performance optimization to the engine.

If you write your algorithms with PTC, the ES6 engine will apply TCO to let your code run in constant stack depth (by reusing stack frames). You get the readability of recursion with most of the performance benefits and no limitations of run length.

### Meta?

What does TCO have to do with meta programming?

As we covered in the "Feature Testing" section earlier, you can determine at runtime what features an engine supports. This includes TCO, though determining it is quite brute force. Consider:

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

In a non-TCO engine, the recursive loop will fail out eventually, throwing an exception caught by the `try..catch`. Otherwise, the loop completes easily thanks to TCO.

Yuck, right?

But how could meta programming around the TCO feature (or rather, the lack thereof) benefit our code? The simple answer is that you could use such a feature test to decide to load a version of your application's code that uses recursion, or an alternative one that's been converted/transpiled to not need recursion.

#### Self-Adjusting Code

But here's another way of looking at the problem:

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

This algorithm works by attempting to do as much of the work with recursion as possible, but keeping track of the progress via scoped variables `x` and `acc`. If the entire problem can be solved with recursion without an error, great. If the engine kills the recursion at some point, we simply catch that with the `try..catch` and then try again, picking up where we left off.

I consider this a form of meta programming in that you are probing during runtime the ability of the engine to fully (recursively) finish the task, and working around any (non-TCO) engine limitations that may restrict you.

At first (or even second!) glance, my bet is this code seems much uglier to you compared to some of the earlier versions. It also runs a fair bit slower (on larger runs in a non-TCO environment).

The primary advantage, other than it being able to complete any size task even in non-TCO engines, is that this "solution" to the recursion stack limitation is much more flexible than the trampolining or manual unrolling techniques shown previously.

Essentially, `_foo()` in this case is a sort of stand-in for practically any recursive task, even mutual recursion. The rest is the boilerplate that should work for just about any algorithm.

The only "catch" is that to be able to resume in the event of a recursion limit being hit, the state of the recursion must be in scoped variables that exist outside the recursive function(s). We did that by leaving `x` and `acc` outside of the `_foo()` function, instead of passing them as arguments to `_foo()` as earlier.

Almost any recursive algorithm can be adapted to work this way. That means it's the most widely applicable way of leveraging TCO with recursion in your programs, with minimal rewriting.

This approach still uses a PTC, meaning that this code will *progressively enhance* from running using the loop many times (recursion batches) in an older browser to fully leveraging TCO'd recursion in an ES6+ environment. I think that's pretty cool!

## Review

Meta programming is when you turn the logic of your program to focus on itself (or its runtime environment), either to inspect its own structure or to modify it. The primary value of meta programming is to extend the normal mechanisms of the language to provide additional capabilities.

Prior to ES6, JavaScript already had quite a bit of meta programming capability, but ES6 significantly ramps that up with several new features.

From function name inferences for anonymous functions to meta properties that give you information about things like how a constructor was invoked, you can inspect the program structure while it runs more than ever before. Well Known Symbols let you override intrinsic behaviors, such as coercion of an object to a primitive value. Proxies can intercept and customize various low-level operations on objects, and `Reflect` provides utilities to emulate them.

Feature testing, even for subtle semantic behaviors like Tail Call Optimization, shifts the meta programming focus from your program to the JS engine capabilities itself. By knowing more about what the environment can do, your programs can adjust themselves to the best fit as they run.

Should you meta program? My advice is: first focus on learning how the core mechanics of the language really work. But once you fully know what JS itself can do, it's time to start leveraging these powerful meta programming capabilities to push the language further!
