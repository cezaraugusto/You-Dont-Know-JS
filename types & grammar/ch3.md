# You Don't Know JS: Tipos e Gramática
# Capítulo 3: Nativos

Várias vezes nos Capítulos 1 e 2, nós aludimos à vários nativos, como `String` e `Number`. Vamos examiná-los em detalhes agora.

Aqui está uma lista dos nativos mais comumente usados:

* `String()`
* `Number()`
* `Boolean()`
* `Array()`
* `Object()`
* `Function()`
* `RegExp()`
* `Date()`
* `Error()`
* `Symbol()` -- adicionado no ES6!

Como você pode ver, esse nativos são na realidade funções nativas.

Se você está vindo para o JS de linguagens como Java, a função do JavaScript `String()` vai parecer com o construtor `String(..)` que está acostumado a usar para criar valores string. Assim, observará rapidamente que você pode fazer coisas como:

```js
var s = new String( "Hello World!" );

console.log( s.toString() ); // "Hello World!"
```

*É* verdade que cada um desses nativos pode ser usado como um construtor. Mas o que está sendo construído pode ser diferente do que você possa pensar.

```js
var a = new String( "abc" );

typeof a; // "object" ... e não "String"

a instanceof String; // true

Object.prototype.toString.call( a ); // "[object String]"
```

O resultado da criação de valores usando o construtor (`new String("abc")`) é um objeto que envolve o valor primitivo (`"abc"`).

É importante destacar que, `typeof` mostra que esses objetos não são seus próprios *tipos* especiais, mas mais adequadamente eles são subtipos do tipo `object`.

Esse objeto envoltório pode ainda ser observado com:

```js
console.log( a );
```

A saída dessa declaração varia dependendo do seu navegador, como os consoles são livres para escolher como eles acharem apropiado para serializar o objeto para a inspeção do desenvolvedor.

**Nota:** No momento da escrita, a última versão do Chrome imprime algo assim: `String {0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"}`. Mas versões antigas do Chrome costumavam imprimir apenas isso: `String {0: "a", 1: "b", 2: "c"}`. A última versão do Firefox imprime atualmente `String ["a","b","c"]`, mas costumavam imprimir `"abc"` em itálico, que era clicável para abrir o inspetor de objetos. Claro que esse resultados estão sujeitos à ráida mudança e sua experiência pode variar.

O ponto é, `new String("abc")` create a um objeto string ao redor de `"abc"`, não apenas o próprio valor primitivo `"abc"`.

## `[[Class]]` interno

Valores que são `typeof` `"object"` (como um array) são adicionalmente marcados com uma propriedade interna `[[Class]]` (pense nisto como uma *class*ificação interna ao invés de classes da tradicional programação orientada a classes). Essa propriedade não pode ser acessada diretamente, mas geralmente pode ser revelada indiretamente por chamar o método `Object.prototype.toString(..)` contra o valor. Por exemplo:

```js
Object.prototype.toString.call( [1,2,3] );			// "[object Array]"

Object.prototype.toString.call( /regex-literal/i );	// "[object RegExp]"
```

Assim para o array no exemplo, o valor `[[Class]]` interno é `"Array"`, e para a expressão regular, é `"RegExp"`. Na maioria dos casos, este valor `[[Class]]` interno corresponde ao construtor nativo (veja abaixo) que é relacionado ao valor, mas nem sempre é o caso.

E quanto aos tipos primitivos? Primeiro, `null` e `undefined`:

```js
Object.prototype.toString.call( null );			// "[object Null]"
Object.prototype.toString.call( undefined );	// "[object Undefined]"
```

Você vai notar de que não há nenhum construtor `Null()` ou `Undefined()`, mas mesmo assim `"Null"` e `"Undefined"` são o valores `[[Class]]` internos expostos.

Mas para os outros tipos primitivos como `string`, `number`, e `boolean`, outra coisa acontece, que geralmente é chamado de "boxing" (veja a próxima seção "Boxing Wrappers"):

```js
Object.prototype.toString.call( "abc" );	// "[object String]"
Object.prototype.toString.call( 42 );		// "[object Number]"
Object.prototype.toString.call( true );		// "[object Boolean]"
```

Nesse trecho de código, cada um dos tipos primitivos simples são automaticamente encaixotados em seus respectivos objetos que os envolvem, é por isso que `"String"`, `"Number"`, e `"Boolean"` são revelados como seus respectivos valores `[[Class]]` internos.

**Nota:** O comportamento de `toString()` e `[[Class]]` ilustrado aqui mudou um pouco do ES5 para o ES6, mas nós cobrimos esses detalhos no título *ES6 & Beyond* dessa série.

## Boxing Wrappers

Estes objetos que envolvem os tipos primitos tem um propósito muito importante. Tipos primitivos não tem propriedades ou métodos, então para acessar `.length` ou `.toString()` você precisa que um objeto o envolva. Felizmente, JS automaticamente vai *encaixotar* (também conhecido como envolver) os tipos para realizar esse acessos aos métodos.

```js
var a = "abc";

a.length; // 3
a.toUpperCase(); // "ABC"
```

Então, se você vai acessar essas propriedade/métodos de suas strings regularmente, como em uma condição `i < a.length` em loop `for` por exemplo, pode parecer fazer sentido apenas usar a forma de objeto desde o início, para o motor JS não precisar criar implicitamente para você.

Mas acaba que isso é uma péssima ideia. Os navegadores há muito tempo otimizaram o desempenho dos casos mais comuns como `.length`, que significa que o seu programa vai *na verdade ficar mais lento* se você tentar "pré-otimizar" por usar diretamente o objeto (que não é otimizado).

Em geral, não há razão para usar os objetos diretamente. É melhor apenas deixar o boxing acontecer implicitamente quando necessário. Em outras palavras, nunca faça coisas como `new String("abc")`, `new Number(42)`, etc -- sempre prefira usar os valores primitivos literais `"abc"` e `42`.

### Pegadinhas do Object Wrapper

Há algumas pegadinhas quando se usa object wrappers diretamente que você precisa estar ciente caso você *escolha* sempre usá-los.

Por exemplo, considerando valores `Boolean` embrulhados:

```js
var a = new Boolean( false );

if (!a) {
	console.log( "Oops" ); // nunca executa
}
```

O problema é que você criou um objeto *wrapper* ao redor do valor `false`, mas objetos são "truthy" por natureza (ver Capítulo 4), portanto o uso deste objeto provê um comportamento oposto ao obtido com a utilização direta do valor `false`, o que de fato representa o oposto do que normalmente se esperaria.

Se você quiser embrulhar um valor primitivo, você pode usar a função `Object(..)` (sem a palavra-chave `new`):

```js
var a = "abc";
var b = new String( a );
var c = Object( a );

typeof a; // "string"
typeof b; // "object"
typeof c; // "object"

b instanceof String; // true
c instanceof String; // true

Object.prototype.toString.call( b ); // "[object String]"
Object.prototype.toString.call( c ); // "[object String]"
```
De novo, ao usar o object wrapper embrulhado diretamente (como `b` e `c` acima) é geralmente desencorajado, mas pode haver raras ocasiões em que eles podem ser úteis.

## Unboxing

Se você tem um objeto *wrapper* e quer obter o seu valor primitivo, pode-se usar o método `valueOf()`:

```js
var a = new String( "abc" );
var b = new Number( 42 );
var c = new Boolean( true );

a.valueOf(); // "abc"
b.valueOf(); // 42
c.valueOf(); // true
```

O *Unboxing* também pode ocorrer implicitamente, quando o objeto *wrapper* é usado de uma forma que requer o valor primitivo. Este processo (coerção) vai ser abrangido em mais detalhes no Capítulo 4, mas de maneira rápida:

```js
var a = new String( "abc" );
var b = a + ""; // `b` tem o valor primitivo unboxed "abc"

typeof a; // "object"
typeof b; // "string"
```

## Nativos como Construtores

Para valores de  `array`, `object`, `function`, e expressão regular, é quase uma preferência universal usar a forma literal para criar os valores, mas a forma literal cria o mesmo tipo de objeto que o construtor cria (ou seja, não existe valor que não seja *wrapped*).

Da mesma forma como os outros nativos, esses construtores devem ser evitados, a não ser que realmente precise dele, principalmente porque eles introduzem exceções e pegadinhas que você provavelmente não vai *querer* lidar.

### `Array(..)`

```js
var a = new Array( 1, 2, 3 );
a; // [1, 2, 3]

var b = [1, 2, 3];
b; // [1, 2, 3]
```

**Nota:** O construtor `Array(..)` não exige a palavra-chave `new` na frente dele. Se você a omitir, o construtor vai se comportar da mesma maneira se você tivesse usado a palavra-chave. Portanto, `Array(1,2,3)` tem o mesmo resultado que `new Array(1,2,3)`.

O construtor `Array` tem um comportamento especial quando um único `number` é passado como argumento, em vez de fornecer esse valor como *conteúdo* do array, ele é tomado como um comprimento para "pré-dimensionar o array" (bem, mais ou menos).

Isso é uma ideia terrível. Primeiramente, você acabar usando essa forma de maneira acidental, porque é fácil se esquecer.

Mas mais importante ainda, não existe como pré-dimensionar o array. Em vez disso, você está criando de outra forma um array vazio, mas definindo a propriedade `length` para o valor numérico especificado.

Um array que não tem valores explicitos em seus slots, mas tem uma propiedade `length` que *implica* que os slots existam, é um tipo exótico de estrutura de dado no JS com alguns comportamentos muito confusos e estranhos. A capacidade de criar esses valores vem puramente de antigas, descontinuadas, funcionalidades históricas ("objetos que parecem arrays" como o objeto `arguments`).

**Nota:** Um array com pelo menos um "slot vazio" é comumente chamado de "sparse array".

Não ajuda nada o fato de que este é mais um exemplo onde cada console de navegador representa tal objeto de uma forma diferente, o que gera mais confusão.

Por exemplo:

```js
var a = new Array( 3 );

a.length; // 3
a;
```

A serialização de `a` no Chrome é (no tempo da escrita): `[ undefined x 3 ]`. **Isso é realmente triste**. O que implica que há três valores `undefined` nos slots deste array, quando na verdade os slots não existem (também chamado de "slots vazios" -- também um nome ruim!).

Para visualizar as diferenças, tente isto:

```js
var a = new Array( 3 );
var b = [ undefined, undefined, undefined ];
var c = [];
c.length = 3;

a;
b;
c;
```

**Nota:** Como você pode ver com o `c` deste exemplo, slots vazios no array podem acontecer após a criação do array. Mudando o `length` de um array para um valor acima do número de slots definidos, você implicitamente introduz slots vazios. Na verdade, você poderia até chamar `delete b[1]` no trecho de código acima, o que introduziria um slot vazio no meio de `b`.

Para `b` (no Chrome, atualmente), você vai encontrar `[ undefined, undefined, undefined ]` como serializalção, ao contrário de `[ undefined x 3 ]` para `a` e `c`. Confuso? Sim, assim como todo mundo.

Pior do que isso, no momento da escrita, Firefox imprime `[ , , , ]` para `a` e `c`. Você percebeu por que isso é tão confuso? Olhe mais de perto. Três vírgulas implica quatro slots, e não três slots como esperávamos.

**O quê!?** Firefox coloca uma `,` extra no final da serialização aqui porque no ES5, vírgulas no final de listas (valores de array, lista de propriedades, etc.) são permitidos (e então removidos e ignorados). Então se você colocar um valor `[ , , , ]` no seu programa ou no console, você na verdade obtêm o valor implícito `[ , , ]` (que é, um array com três slots vazios). Essa escolha, apesar de confusa de se ler, é defendida por copiar e colar com precisão o comportamento.

Se você está sacudindo a sua cabeça ou rolando os seus olho agora, você não está sozinho!

Infelizmente, fica pior. Mais do que apenas uam saída confusa no console, `a` e `b` do trecho de código acima na verdade se comportam da mesma maneira em alguns casos **mas diferente em outros**:

```js
a.join( "-" ); // "--"
b.join( "-" ); // "--"

a.map(function(v,i){ return i; }); // [ undefined x 3 ]
b.map(function(v,i){ return i; }); // [ 0, 1, 2 ]
```

**Ugh.**

A chamada de `a.map(..)` *falha* por que os slots na verdade não existem, então `map(..)` não tem nada sobre o que iterar. `join(..)` funciona diferente. Basicamente, nós podemos pensar que este método é implementado de uma forma parecida com essa:

```js
function fakeJoin(arr,connector) {
	var str = "";
	for (var i = 0; i < arr.length; i++) {
		if (i > 0) {
			str += connector;
		}
		if (arr[i] !== undefined) {
			str += arr[i];
		}
	}
	return str;
}

var a = new Array( 3 );
fakeJoin( a, "-" ); // "--"
```

Como você pode ver, `join(..)` funciona apenas *supondo* que os slots existem e looping o valor de `length`. Seja o que for que `map(..)` faz internamente, ele (aparentemente) não faz essa suposição, por isso os resultados dos "slots vazios" é inesperado e propensos à falhas.

Portanto, se você quer *na realidade* criar um array de valores `undefined` reais (não apenas "slots vazios"), como você pode fazer isso (além de manualmente)?

```js
var a = Array.apply( null, { length: 3 } );
a; // [ undefined, undefined, undefined ]
```

Confuso? Sim. Assim é mais ou menos como funciona.

`apply(..)` é um utilitário disponível a todas as funções, que chama a função usada mas de uma maneira especial.

O primeiro argumento é o objeto `this` (abordado no título *this & Prototipagem de Objetos* desta série), que não nos interessa aqui, então nós definimos ele para `null`. O segundo argumento deveria ser um array (ou algo *parecido com* um array -- também conhecido como um "array-like object"). O conteúdo deste "array" é "espalhado" como argumento da função em questão.

Então, `Array.apply(..)` está chamando a  função `Array(..)` e espalhando os valores (do objeto `{ length: 3 }`) como seu argumento.

Dentro do `apply(..)`, nós podemos imaginar outro loop `for` (da mesma forma que o `join(..)` acima) que vai de `0` até, mas não inclui, `length` (`3` em nosso caso).

Para cada índice, ele retira aquela chave do objeto. Então se o parâmetro objeto-array for chamado de `arr` internamente dentro da função `apply(..)`, o acesso à propriedade seria efetivamente `arr[0]`, `arr[1]`, e `arr[2]`. Claro que nenhuma dessa propriedades existem no objeto `{ length: 3 }`, por isso todos os três acesso retornariam o valor `undefined`.

Em outras palavras, acaba-se chamando `Array(..)` basicamente dessa forma: `Array(undefined,undefined,undefined)`, que é como nós acabamos com um array preenchido com valores `undefined`, e não apenas com aqueles (loucos) slots vazios.

Enquanto `Array.apply( null, { length: 3 } )` é uma estranha e verbosa de criar um array preenchido com valores `undefined`, é muito melhor e mais confiável que o que se consegue com os autodestrutivos slots vazios de `Array(3)`.

Concluindo: **nunca, sobre nenhuma circunstância**, você deve intencionalmente criar e usar esse arrays com exóticos slots vazios. Apenas não faça isso. Eles são esquisitos.

### `Object(..)`, `Function(..)`, and `RegExp(..)`

The `Object(..)`/`Function(..)`/`RegExp(..)` constructors are also generally optional (and thus should usually be avoided unless specifically called for):

```js
var c = new Object();
c.foo = "bar";
c; // { foo: "bar" }

var d = { foo: "bar" };
d; // { foo: "bar" }

var e = new Function( "a", "return a * 2;" );
var f = function(a) { return a * 2; };
function g(a) { return a * 2; }

var h = new RegExp( "^a*b+", "g" );
var i = /^a*b+/g;
```

There's practically no reason to ever use the `new Object()` constructor form, especially since it forces you to add properties one-by-one instead of many at once in the object literal form.

The `Function` constructor is helpful only in the rarest of cases, where you need to dynamically define a function's parameters and/or its function body. **Do not just treat `Function(..)` as an alternate form of `eval(..)`.** You will almost never need to dynamically define a function in this way.

Regular expressions defined in the literal form (`/^a*b+/g`) are strongly preferred, not just for ease of syntax but for performance reasons -- the JS engine precompiles and caches them before code execution. Unlike the other constructor forms we've seen so far, `RegExp(..)` has some reasonable utility: to dynamically define the pattern for a regular expression.

```js
var name = "Kyle";
var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );

var matches = someText.match( namePattern );
```

This kind of scenario legitimately occurs in JS programs from time to time, so you'd need to use the `new RegExp("pattern","flags")` form.

### `Date(..)` and `Error(..)`

The `Date(..)` and `Error(..)` native constructors are much more useful than the other natives, because there is no literal form for either.

To create a date object value, you must use `new Date()`. The `Date(..)` constructor accepts optional arguments to specify the date/time to use, but if omitted, the current date/time is assumed.

By far the most common reason you construct a date object is to get the current timestamp value (a signed integer number of milliseconds since Jan 1, 1970). You can do this by calling `getTime()` on a date object instance.

But an even easier way is to just call the static helper function defined as of ES5: `Date.now()`. And to polyfill that for pre-ES5 is pretty easy:

```js
if (!Date.now) {
	Date.now = function(){
		return (new Date()).getTime();
	};
}
```

**Note:** If you call `Date()` without `new`, you'll get back a string representation of the date/time at that moment. The exact form of this representation is not specified in the language spec, though browsers tend to agree on something close to: `"Fri Jul 18 2014 00:31:02 GMT-0500 (CDT)"`.

The `Error(..)` constructor (much like `Array()` above) behaves the same with the `new` keyword present or omitted.

The main reason you'd want to create an error object is that it captures the current execution stack context into the object (in most JS engines, revealed as a read-only `.stack` property once constructed). This stack context includes the function call-stack and the line-number where the error object was created, which makes debugging that error much easier.

You would typically use such an error object with the `throw` operator:

```js
function foo(x) {
	if (!x) {
		throw new Error( "x wasn't provided" );
	}
	// ..
}
```

Error object instances generally have at least a `message` property, and sometimes other properties (which you should treat as read-only), like `type`. However, other than inspecting the above-mentioned `stack` property, it's usually best to just call `toString()` on the error object (either explicitly, or implicitly through coercion -- see Chapter 4) to get a friendly-formatted error message.

**Tip:** Technically, in addition to the general `Error(..)` native, there are several other specific-error-type natives: `EvalError(..)`, `RangeError(..)`, `ReferenceError(..)`, `SyntaxError(..)`, `TypeError(..)`, and `URIError(..)`. But it's very rare to manually use these specific error natives. They are automatically used if your program actually suffers from a real exception (such as referencing an undeclared variable and getting a `ReferenceError` error).

### `Symbol(..)`

New as of ES6, an additional primitive value type has been added, called "Symbol". Symbols are special "unique" (not strictly guaranteed!) values that can be used as properties on objects with little fear of any collision. They're primarily designed for special built-in behaviors of ES6 constructs, but you can also define your own symbols.

Symbols can be used as property names, but you cannot see or access the actual value of a symbol from your program, nor from the developer console. If you evaluate a symbol in the developer console, what's shown looks like `Symbol(Symbol.create)`, for example.

There are several predefined symbols in ES6, accessed as static properties of the `Symbol` function object, like `Symbol.create`, `Symbol.iterator`, etc. To use them, do something like:

```js
obj[Symbol.iterator] = function(){ /*..*/ };
```

To define your own custom symbols, use the `Symbol(..)` native. The `Symbol(..)` native "constructor" is unique in that you're not allowed to use `new` with it, as doing so will throw an error.

```js
var mysym = Symbol( "my own symbol" );
mysym;				// Symbol(my own symbol)
mysym.toString();	// "Symbol(my own symbol)"
typeof mysym; 		// "symbol"

var a = { };
a[mysym] = "foobar";

Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]
```

While symbols are not actually private (`Object.getOwnPropertySymbols(..)` reflects on the object and reveals the symbols quite publicly), using them for private or special properties is likely their primary use-case. For most developers, they may take the place of property names with `_` underscore prefixes, which are almost always by convention signals to say, "hey, this is a private/special/internal property, so leave it alone!"

**Note:** `Symbol`s are *not* `object`s, they are simple scalar primitives.

### Native Prototypes

Each of the built-in native constructors has its own `.prototype` object -- `Array.prototype`, `String.prototype`, etc.

These objects contain behavior unique to their particular object subtype.

For example, all string objects, and by extension (via boxing) `string` primitives, have access to default behavior as methods defined on the `String.prototype` object.

**Note:** By documentation convention, `String.prototype.XYZ` is shortened to `String#XYZ`, and likewise for all the other `.prototype`s.

* `String#indexOf(..)`: find the position in the string of another substring
* `String#charAt(..)`: access the character at a position in the string
* `String#substr(..)`, `String#substring(..)`, and `String#slice(..)`: extract a portion of the string as a new string
* `String#toUpperCase()` and `String#toLowerCase()`: create a new string that's converted to either uppercase or lowercase
* `String#trim()`: create a new string that's stripped of any trailing or leading whitespace

None of the methods modify the string *in place*. Modifications (like case conversion or trimming) create a new value from the existing value.

By virtue of prototype delegation (see the *this & Object Prototypes* title in this series), any string value can access these methods:

```js
var a = " abc ";

a.indexOf( "c" ); // 3
a.toUpperCase(); // " ABC "
a.trim(); // "abc"
```

The other constructor prototypes contain behaviors appropriate to their types, such as `Number#toFixed(..)` (stringifying a number with a fixed number of decimal digits) and `Array#concat(..)` (merging arrays). All functions have access to `apply(..)`, `call(..)`, and `bind(..)` because `Function.prototype` defines them.

But, some of the native prototypes aren't *just* plain objects:

```js
typeof Function.prototype;			// "function"
Function.prototype();				// it's an empty function!

RegExp.prototype.toString();		// "/(?:)/" -- empty regex
"abc".match( RegExp.prototype );	// [""]
```

A particularly bad idea, you can even modify these native prototypes (not just adding properties as you're probably familiar with):

```js
Array.isArray( Array.prototype );	// true
Array.prototype.push( 1, 2, 3 );	// 3
Array.prototype;					// [1,2,3]

// don't leave it that way, though, or expect weirdness!
// reset the `Array.prototype` to empty
Array.prototype.length = 0;
```

As you can see, `Function.prototype` is a function, `RegExp.prototype` is a regular expression, and `Array.prototype` is an array. Interesting and cool, huh?

#### Prototypes As Defaults

`Function.prototype` being an empty function, `RegExp.prototype` being an "empty" (e.g., non-matching) regex, and `Array.prototype` being an empty array, make them all nice "default" values to assign to variables if those variables wouldn't already have had a value of the proper type.

For example:

```js
function isThisCool(vals,fn,rx) {
	vals = vals || Array.prototype;
	fn = fn || Function.prototype;
	rx = rx || RegExp.prototype;

	return rx.test(
		vals.map( fn ).join( "" )
	);
}

isThisCool();		// true

isThisCool(
	["a","b","c"],
	function(v){ return v.toUpperCase(); },
	/D/
);					// false
```

**Note:** As of ES6, we don't need to use the `vals = vals || ..` default value syntax trick (see Chapter 4) anymore, because default values can be set for parameters via native syntax in the function declaration (see Chapter 5).

One minor side-benefit of this approach is that the `.prototype`s are already created and built-in, thus created *only once*. By contrast, using `[]`, `function(){}`, and `/(?:)/` values themselves for those defaults would (likely, depending on engine implementations) be recreating those values (and probably garbage-collecting them later) for *each call* of `isThisCool(..)`. That could be memory/CPU wasteful.

Also, be very careful not to use `Array.prototype` as a default value **that will subsequently be modified**. In this example, `vals` is used read-only, but if you were to instead make in-place changes to `vals`, you would actually be modifying `Array.prototype` itself, which would lead to the gotchas mentioned earlier!

**Note:** While we're pointing out these native prototypes and some usefulness, be cautious of relying on them and even more wary of modifying them in anyway. See Appendix A "Native Prototypes" for more discussion.

## Review

JavaScript provides object wrappers around primitive values, known as natives (`String`, `Number`, `Boolean`, etc). These object wrappers give the values access to behaviors appropriate for each object subtype (`String#trim()` and `Array#concat(..)`).

If you have a simple scalar primitive value like `"abc"` and you access its `length` property or some `String.prototype` method, JS automatically "boxes" the value (wraps it in its respective object wrapper) so that the property/method accesses can be fulfilled.
