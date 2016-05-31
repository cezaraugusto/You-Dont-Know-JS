# You Don't Know JS: Tipos e Gramática
# Capítulo 1: Tipos

Muitos desenvolvedores diriam que uma linguagem dinâmica (como JS) não possui *tipos*. Vamos ver o que a especificação ES5.1 (http://www.ecma-international.org/ecma-262/5.1/) tem a dizer sobre este tópico:

> Algorítimos dentro desta especificação manipulam valores cada um dos quais possui um tipo associado. Os tipos de valores possíveis são exatamente os definidos nessa cláusula. Tipos são ainda sub-classificados na ECMAScript como tipos de linguagem e tipos de especificação.
>
> A tipagem da linguagem ECMAScript corresponde à valores que são diretamente manipulados por um programador ECMAScript usando a linguagem ECMAScript. Os tipos existentes da linguagem ECMAScript são Undefined, Null, Boolean, String, Number, e Object.

Agora, se você é um fã de linguagens com tipagem forte (tipagem estática), você pode opôr-se a esse uso da palavra "tipo." Nessas linguagens, "tipo" significa muito *mais* do que ele faz aqui no JS.

Algumas pessoas dizem que JS não deveria possuir "tipos," antes estes deveriam ser chamados de "tags" ou talvez "subtipos".

Bah! Nós iremos usar essa grosseira definição (a mesma que parece dirigir a escrita da especificação): um *tipo* é intrínseco, construído com um conjunto de características que unicamente identifica o comportamento de um valor em particular e distingue-o de outros valores, tanto para o motor (engine) **quanto para o desenvolvedor**.

Em outras palavras, se tanto o motor (engine) quanto o desenvolvedor tratam o valor `42` (o número) diferentemente do que eles tratam o valor `"42"` (a string), então esses dois valores possuem *tipos* diferentes -- `number` e `string`, respectivamente. Quando você usa `42`, você *pretende* fazer algo numérico, como matemática. Mas quando você usa `"42"`, você está *pretendendo* fazer alguma coisa com texto/strings, como saída (output) para a página, etc. **Esses dois valores possuem tipagem diferente.**

Isso não é uma definição perfeita. Mas é bom o suficiente para essa discussão. E é consistente com a maneira que JS descreve-se.

# O que importa em um tipo é o que ele é, não pelo quê é chamado...

Além dos desacordos da definição acadêmica, por que é importante se JavaScript possui *tipos* ou não?

Ter uma compreensão adequada de cada *tipo* e seu comportamento intrínsico é absolutamente essencial para entender como converter valores corretamente e precisamente para diferentes tipos (veja Coerção (Coercion), Capítulo 4). Quase todos programas escritos em JS precisarão lidar com coerção de valores em algum tipo de formulário, assim é importante você fazê-los com responsabilidade e confiança.

Se você possui o `número (number)` com valor `42`, mas você deseja tratá-lo como uma `string`, tal que a saída dê-se como um caractere `"2"` na posição `1`, você obviamente deve primeiro converter (coerce) o valor de `número (number)` para `string`.

Isso parece bastante simples.

Mas exitem muitas maneiras diferentes em que a coerção pode acontecer. Algumas dessas maneiras são explícitas, fáceis de assimilar e confiáveis. Mas se você não for cuidadoso, a coerção (coercion) pode acontecer de formas muito estranhas e surpreendentes.

Confusão na coerção é possivelmente uma das mais profundas frustrações para desenvolvedores JavaScript. Tem sido frequentemente criticada como sendo tão *perigosa* que pode ser considerada uma falha no design da linguagem, à ser banida e evitada.

Armados com total conhecimento dos tipos do JavaScript, nosso objetivo é demonstrar por que a *má reputação* da coerção é largamente famosa e um pouco imerecida -- mudando a sua perspectiva, para ver a coerção como uma poderosa utilidade. Mas primeiro, devemos ter uma compreensão melhor sobre valores e tipos.

## Tipos nativos (built-in)

JavaScript define sete tipos nativos:

* `null`
* `undefined`
* `boolean`
* `number`
* `string`
* `object`
* `symbol` -- adicionado no ES6!

**Nota:** Todos esses tipos, exceto `object` são chamados "primitivos".

O operador `typeof` inspeciona o tipo do valor obtido, e sempre retorna um dos sete valores de string -- surpreendentemente, não há uma correspondência exata de 1-para-1 com os sete tipos nativos que listamos.

```js
typeof undefined     === "undefined"; // true (verdadeiro)
typeof true          === "boolean";   // true (verdadeiro)
typeof 42            === "number";    // true (verdadeiro)
typeof "42"          === "string";    // true (verdadeiro)
typeof { life: 42 }  === "object";    // true (verdadeiro)

// adicionado no ES6!
typeof Symbol()      === "symbol";    // true (verdadeiro)
```

Estes seis tipos listados possuem valores do tipo correspondente e retornam uma string com o mesmo nome, como mostrado. `Symbol` é um novo tipo de dados introduzido à partir do ES6, e será explicado no Capítulo 3.

Como você pode perceber, eu excluí `null` da lista acima. Ele é *especial* -- especial no sentido que é problemático quando combinado com o operador `typeof`:

```js
typeof null === "object"; // true (verdadeiro)
```

Teria sido legal (e correto!) se retornasse `"null"`, mas esse "bug" original no JS tem persistido por aproximadamente duas décadas, e provavelmente nunca será corrigido porque existe muito conteúdo na web que se baseia nesse comportamente "bugado", e corrigí-lo *criaria* mais "bugs", e causaria rupturas em uma grade quantidade de softwares web.

Se você deseja testar um valor `null` usando seu tipo, precisará de uma condição composta:

```js
var a = null;

(!a && typeof a === "object"); // true (verdadeiro)
```

`null` é o único valor primitivo que pode ser "falso" (também conhecido como "false-like"; veja o Capítulo 4) mas que também retorna `"object"` na checagem de `typeof`.

Então qual é o sétimo valor de string que `typeof` pode retornar?

```js
typeof function a(){ /* .. */ } === "function"; // true (verdadeiro)
```

É fácil pensar que `function (função)` é um tipo nativo de alto nível no JS, especialmente obtendo esse comportamento do operador `typeof`. Contudo, se você ler a especificação, verá que atualmente é um "subtipo" de objeto. Especificamente, uma função é referenciada como um "objeto chamável (callable object)" -- um objeto que possui uma propriedade `[[Call]]` interna que possibilita ao mesmo ser invocado.

O fato de funções atualmente serem objetos é muito útil. Mais importante, elas podem ter propriedades. Por exemplo:

```js
function a(b,c) {
	/* .. */
}
```

O objeto de função tem uma propriedade `length` definido com o número de parâmetros formais com o qual a função foi declarada.

```js
a.length; // 2
```

Uma vez que você tenha declarado a função com dois parâmetros formalmente nomeados (`b` e `c`), o "length (tamanho) da função" será `2`.

E quanto aos arrays? Eles são nativos para o JS, então eles são um tipo especial?

```js
typeof [1,2,3] === "object"; // true (verdadeiro)
```

Não, apenas objetos. É muito apropriado pensar sobre eles também como um "subtipo" de objeto (veja o Capítulo 3), neste caso com as características adicionais de serem numéricamente indexáveis (em oposição aos objetos simples com chaves do tipo string) e manterem uma propriedade `.length` automaticamente atualizada.

## Valores possuem Tipos, e não as variáveis

Em JavaScript, variáveis não possuem tipos -- **valores possuem tipos**. Variáveis podem conter qualquer valor à qualquer momento.

Outra maneira de pensar sobre tipos do JS é que JS não possui "tipagem forçada (type enforcement)," pois a engine não insiste que uma *variável* sempre terá valores do *mesmo tipo inicial* com que ela começou. Uma variável pode, em uma declaração de atribuição, possuir um valor do tipo `string`, e no futuro ter um `number`, e assim por diante.

O *valor* `42` possui o tipo `number` intrínseco, e seu *tipo* não pode ser modificado. Um outro valor, como `"42"` com o tipo `string`, pode ser criado *à partir* do valor `42` de tipo `number` através de um processo chamado **coerção (coercion)** (veja o Capítulo 4).

If you use `typeof` against a variable, it's not asking "what's the type of the variable?" as it may seem, since JS variables have no types. Instead, it's asking "what's the type of the value *in* the variable?"

```js
var a = 42;
typeof a; // "number"

a = true;
typeof a; // "boolean"
```

The `typeof` operator always returns a string. So:

```js
typeof typeof 42; // "string"
```

The first `typeof 42` returns `"number"`, and `typeof "number"` is `"string"`.

### `undefined` vs "undeclared"

Variables that have no value *currently*, actually have the `undefined` value. Calling `typeof` against such variables will return `"undefined"`:

```js
var a;

typeof a; // "undefined"

var b = 42;
var c;

// later
b = c;

typeof b; // "undefined"
typeof c; // "undefined"
```

It's tempting for most developers to think of the word "undefined" and think of it as a synonym for "undeclared." However, in JS, these two concepts are quite different.

An "undefined" variable is one that has been declared in the accessible scope, but *at the moment* has no other value in it. By contrast, an "undeclared" variable is one that has not been formally declared in the accessible scope.

Consider:

```js
var a;

a; // undefined
b; // ReferenceError: b is not defined
```

An annoying confusion is the error message that browsers assign to this condition. As you can see, the message is "b is not defined," which is of course very easy and reasonable to confuse with "b is undefined." Yet again, "undefined" and "is not defined" are very different things. It'd be nice if the browsers said something like "b is not found" or "b is not declared," to reduce the confusion!

There's also a special behavior associated with `typeof` as it relates to undeclared variables that even further reinforces the confusion. Consider:

```js
var a;

typeof a; // "undefined"

typeof b; // "undefined"
```

The `typeof` operator returns `"undefined"` even for "undeclared" (or "not defined") variables. Notice that there was no error thrown when we executed `typeof b`, even though `b` is an undeclared variable. This is a special safety guard in the behavior of `typeof`.

Similar to above, it would have been nice if `typeof` used with an undeclared variable returned "undeclared" instead of conflating the result value with the different "undefined" case.

### `typeof` Undeclared

Nevertheless, this safety guard is a useful feature when dealing with JavaScript in the browser, where multiple script files can load variables into the shared global namespace.

**Note:** Many developers believe there should never be any variables in the global namespace, and that everything should be contained in modules and private/separate namespaces. This is great in theory but nearly impossible in practicality; still it's a good goal to strive toward! Fortunately, ES6 added first-class support for modules, which will eventually make that much more practical.

As a simple example, imagine having a "debug mode" in your program that is controlled by a global variable (flag) called `DEBUG`. You'd want to check if that variable was declared before performing a debug task like logging a message to the console. A top-level global `var DEBUG = true` declaration would only be included in a "debug.js" file, which you only load into the browser when you're in development/testing, but not in production.

However, you have to take care in how you check for the global `DEBUG` variable in the rest of your application code, so that you don't throw a `ReferenceError`. The safety guard on `typeof` is our friend in this case.

```js
// oops, this would throw an error!
if (DEBUG) {
	console.log( "Debugging is starting" );
}

// this is a safe existence check
if (typeof DEBUG !== "undefined") {
	console.log( "Debugging is starting" );
}
```

This sort of check is useful even if you're not dealing with user-defined variables (like `DEBUG`). If you are doing a feature check for a built-in API, you may also find it helpful to check without throwing an error:

```js
if (typeof atob === "undefined") {
	atob = function() { /*..*/ };
}
```

**Note:** If you're defining a "polyfill" for a feature if it doesn't already exist, you probably want to avoid using `var` to make the `atob` declaration. If you declare `var atob` inside the `if` statement, this declaration is hoisted (see the *Scope & Closures* title of this series) to the top of the scope, even if the `if` condition doesn't pass (because the global `atob` already exists!). In some browsers and for some special types of global built-in variables (often called "host objects"), this duplicate declaration may throw an error. Omitting the `var` prevents this hoisted declaration.

Another way of doing these checks against global variables but without the safety guard feature of `typeof` is to observe that all global variables are also properties of the global object, which in the browser is basically the `window` object. So, the above checks could have been done (quite safely) as:

```js
if (window.DEBUG) {
	// ..
}

if (!window.atob) {
	// ..
}
```

Unlike referencing undeclared variables, there is no `ReferenceError` thrown if you try to access an object property (even on the global `window` object) that doesn't exist.

On the other hand, manually referencing the global variable with a `window` reference is something some developers prefer to avoid, especially if your code needs to run in multiple JS environments (not just browsers, but server-side node.js, for instance), where the global variable may not always be called `window`.

Technically, this safety guard on `typeof` is useful even if you're not using global variables, though these circumstances are less common, and some developers may find this design approach less desirable. Imagine a utility function that you want others to copy-and-paste into their programs or modules, in which you want to check to see if the including program has defined a certain variable (so that you can use it) or not:

```js
function doSomethingCool() {
	var helper =
		(typeof FeatureXYZ !== "undefined") ?
		FeatureXYZ :
		function() { /*.. default feature ..*/ };

	var val = helper();
	// ..
}
```

`doSomethingCool()` tests for a variable called `FeatureXYZ`, and if found, uses it, but if not, uses its own. Now, if someone includes this utility in their module/program, it safely checks if they've defined `FeatureXYZ` or not:

```js
// an IIFE (see "Immediately Invoked Function Expressions"
// discussion in the *Scope & Closures* title of this series)
(function(){
	function FeatureXYZ() { /*.. my XYZ feature ..*/ }

	// include `doSomethingCool(..)`
	function doSomethingCool() {
		var helper =
			(typeof FeatureXYZ !== "undefined") ?
			FeatureXYZ :
			function() { /*.. default feature ..*/ };

		var val = helper();
		// ..
	}

	doSomethingCool();
})();
```

Here, `FeatureXYZ` is not at all a global variable, but we're still using the safety guard of `typeof` to make it safe to check for. And importantly, here there is *no* object we can use (like we did for global variables with `window.___`) to make the check, so `typeof` is quite helpful.

Other developers would prefer a design pattern called "dependency injection," where instead of `doSomethingCool()` inspecting implicitly for `FeatureXYZ` to be defined outside/around it, it would need to have the dependency explicitly passed in, like:

```js
function doSomethingCool(FeatureXYZ) {
	var helper = FeatureXYZ ||
		function() { /*.. default feature ..*/ };

	var val = helper();
	// ..
}
```

There are lots of options when designing such functionality. No one pattern here is "correct" or "wrong" -- there are various tradeoffs to each approach. But overall, it's nice that the `typeof` undeclared safety guard gives us more options.

## Review

JavaScript has seven built-in *types*: `null`, `undefined`,  `boolean`, `number`, `string`, `object`, `symbol`. They can be identified by the `typeof` operator.

Variables don't have types, but the values in them do. These types define intrinsic behavior of the values.

Many developers will assume "undefined" and "undeclared" are roughly the same thing, but in JavaScript, they're quite different. `undefined` is a value that a declared variable can hold. "Undeclared" means a variable has never been declared.

JavaScript unfortunately kind of conflates these two terms, not only in its error messages ("ReferenceError: a is not defined") but also in the return values of `typeof`, which is `"undefined"` for both cases.

However, the safety guard (preventing an error) on `typeof` when used against an undeclared variable can be helpful in certain cases.
