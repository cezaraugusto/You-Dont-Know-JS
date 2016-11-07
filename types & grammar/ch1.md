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

## Valores como tipos

Em JavaScript, variáveis não possuem tipos -- **valores possuem tipos**. Variáveis podem conter qualquer valor à qualquer momento.

Outra maneira de pensar sobre tipos do JS é que JS não possui "tipagem forçada (type enforcement)," pois a engine não insiste que uma *variável* sempre terá valores do *mesmo tipo inicial* com que ela começou. Uma variável pode, em uma declaração de atribuição, possuir um valor do tipo `string`, e no futuro ter um `number`, e assim por diante.

O *valor* `42` possui o tipo `number` intrínseco, e seu *tipo* não pode ser modificado. Um outro valor, como `"42"` com o tipo `string`, pode ser criado *à partir* do valor `42` de tipo `number` através de um processo chamado **coerção (coercion)** (veja o Capítulo 4).

Se você usa o `typeof` em uma variável, ele não está perguntando "qual o tipo da variável?" como pode parecer, pois variáveis JS não possuem tipos. Pelo contrário, ele está perguntando "qual é o tipo do valor *na* variável?"

```js
var a = 42;
typeof a; // "number" ("número")

a = true;
typeof a; // "boolean" ("booleano")
```

O operador `typeof` sempre retorna uma string. Assim:

```js
typeof typeof 42; // "string"
```

O primeiro `typeof 42` retorna `"number"`, e `typeof "number"` é `"string"`.

### `undefined` vs "undeclared"

Variáveis que não possuem valores *neste momento*, atualmente possuem o valor `undefined (indefinido)`. Chamar `typeof` nestas variáveis retornará `"undefined"`:

```js
var a;

typeof a; // "undefined" ("indefinido")

var b = 42;
var c;

// depois
b = c;

typeof b; // "undefined" ("indefinido")
typeof c; // "undefined" ("indefinido")
```

É tentador para muitos desenvolvedores olharem para a palavra "undefined (indefinido)" e pensar nela como sinônimo para "undeclared (não declarada)." Contudo, em JS, esses dois conceitos são bastante diferentes.

Uma variável "undefined" é aquela que foi declarada no escopo acessível, mas *neste momento* não possui nenhum outro valor. Por contraste, uma variável "undeclared" é aquela que não foi formalmente declarada no escopo acessível.

Considere:

```js
var a;

a; // undefined (indefinido)
b; // ReferenceError: b is not defined (Erro de Referência: b não está definido)
```

Uma irritante confusão é a mensagem de erro que os navegadores atribuem para essa condição. Como você pode ver, a mensagem é "b is not defined (b não está definido)," a qual é com certeza muito fácil de confundir com "b is undefined (b é indefinido)." Novamente, "undefined (indefinido)" e "is not defined (não está definido)" são coisas bem diferentes. Seria ótimo se os navegadores retornassem algo como "b is not found (b não foi encontrado)" ou "b is not declared (b não está declarado) ," para reduzir essa confusão!

Há também um comportamento especial associado com `typeof` no que se refere às variáveis não declaradas que reforça ainda mais a confusão. Considere:

```js
var a;

typeof a; // "undefined" ("indefinido")

typeof b; // "undefined" ("indefinido"?)
```

O operador `typeof` retorna `"undefined"` até para variáveis "undeclared (não declaradas)" (ou "não definidas"). Note que aqui não há erros disparados quando nós executamos `typeof b`, apesar de `b` ser uma variável "não declarada". Isso é uma questão especial de segurança no comportamento de `typeof`.

Similarmente, teria sido ótimo se `typeof` usado com uma variável não declarada retornasse "undeclared" ao invés de confundir o resultado com os diferentes tipos de "undefined".

### `typeof` Undeclared

Mesmo assim, essa medida de segurança é uma funcionalidade útil quando estamos lidando com JavaScript no navegador, onde múltiplos scripts podem carregar variáveis no escopo global compartilhado.

**Nota:** Muitos desenvolvedores acreditam que nunca deveria existir nenhuma variável no namespace global, e que tudo deve estar em módulos e namespaces privados/separados. Isso é ótimo em teoria mas praticamente impossível na prática; Ainda é um ótimo objetivo à ser alcançado! Felizmente, o ES6 adotou suporte à classes para os módulos, o que, eventualmente, deixará isso muito mais prático.

Como um simples exemplo, imagine ter um "modo de depuração" em seu programa que é controlado por uma variável global (flag) chamada `DEBUG`. Você gostaria de verificar se essa variável foi declarada antes de realizar uma tarefa de depuração, como enviar uma mensagem para o console. A declaração de uma variável global de nível superior `var DEBUG = true` seria incluída em um arquivo "debug.js", o qual você carregaria no seu browser apenas quando estivesse em modo de desenvolvimento/testes, mas não em modo de produção.

Contudo, você deve ter cuidado no modo como verifica pela variável global `DEBUG` no resto do código de sua aplicação, de modo que não dispare um `ReferenceError`. A medida de segurança no `typeof` é sua amiga nesse caso.

```js
// oops, this would throw an error! (oops, isso irá disparar um erro!)
if (DEBUG) {
    console.log( "Debugging is starting" );
}

// this is a safe existence check (Essa é uma verificação de existência segura)
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
