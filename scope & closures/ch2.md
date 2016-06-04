# You Don't Know JS: Escopos e Clausuras
# Chapter 2: Escopo Léxico

No Capítulo 1, definimos "escopo" como o conjunto de regras que dita a forma com que o *Motor* poderá buscar e eventualmente localizar variáveis através de seus identificadores, tanto no *Escopo* atual quando nos *Escopos aninhados* que este possa estar inserido. 

Existem dois modelos principais para a definição de funcionamento do escopo. O primeiro e mais comum, utilizado pela grande maioria das linguagens de programação, é chamado de **Escopo Léxico**, e vamos examiná-lo em profundidade. O outro modelo, que ainda é utilizado em algumas linguagens como Bash scripting e alguns modos de Perl, é chamado de **Escopo Dinâmico**.

Escopo Dinâmico é tratado no Apêndice A. Menciono esta informação aqui apenas para definir um contraste em relação ao Escopo Léxico, que é o modelo utilizado pelo JavaScript.

## Hora do léxico

Conforme discutimos no Capítulo 1, a primeira etapa da compilação de linguagens tradicionais é chamada de Análise Léxica (ou Tokenização). Caso não se lembre, o processo de Análise Léxica examina uma sequência de caractéres do código fonte e atribui um significado semântico para estes símbolos (tokens) como resultado de uma análise stateful.

Este é o conceito que provê as bases para compreensão do que é o Escopo Léxico e a origem do seu nome.

Para uma definição de certa forma redundante, o Escopo Léxico é o escopo definido durante a etapa de Análise Léxica. Em outras palavras, o Escopo Léxico baseia-se no local onde variáveis e blocos de escopo são criados por você durante a escrita do código, portanto (normalmente) já definidos no momento que o analisador léxico processa seu código. 

**Nota:** Veremos em alguns instantes que existem formas de enganar o Escopo Léxico, e assim sendo modificá-lo após sua passagem pelo analisador léxico, mas isso é, de certa forma, mal visto. É considerado boa pratica tratar o escopo léxico como, de fato, léxico, e portanto inteiramente associado ao momento em que foi definido pelo autor do código. 

Consideremos o seguinte bloco de código:

```js
function foo(a) {

	var b = a * 2;

	function bar(c) {
		console.log( a, b, c );
	}

	bar(b * 3);
}

foo( 2 ); // 2 4 12
```

Existem três escopos distintos neste exemplo de código. Talvez facilite pensar neles como bolhas dentro umas das outras.

<img src="fig2.png" width="500">

**Bolha 1** representa o escopo global, e possui apenas um identificador: `foo`.

**Bolha 2** representa o escopo de `foo`, que por sua vez possui três identificadores: `a`, `bar` e `b`.

**Bolha 3** representa o escopo de `bar`, que possui apenas um identificador: `c`.

Estas bolhas são definidas pelo local onde o escopo foi definido, cada um deles aninhado com outro e assim por diante. No próximo capítulo, vamos discutir diferentes unidades de escopo, mas, por ora, vamos assumir que cada função cria uma nova bolha de escopo.

A bolha de `bar` está contida na bolha de `foo`, porque (e somente por isso) foi o local que optamos por declarar a função `bar`.

Observe que estas bolhas são estritamente aninhadas. Não estamos falando de um Diagrama de Venn, onde as fronteiras (dos conjuntos matemáticos) podem ser atravessadas (para definição de interseções). Em outras palavras, e diferente dos conjuntos, não é possível que a bolha de escopo de uma função esteja presente simultaneamente em duas outras bolhas de escopo, assim como não é possível que uma mesma função seja declarada parte em uma função e parte em outra.

### Consultas

A estrutura e a relação de posicionamento destas bolhas de escopo descreve para o *Motor* todos os locais nos quais deve consultar para localizar um identificador.

No trecho de código acima, o *Motor* executa a instrução `console.log(..)` e vai em busca das três variáveis referenciadas `a`, `b` e `c`. Ele inicia com o escopo mais interno, o escopo da função `bar`. Não encontrará `a` por lá, então sobe um nível para a bolha de escopo mais próxima, o escopo de `foo(..)`. Lá ele localiza `a`, e utiliza este `a`. A mesma coisa ocorre com `b`. Porém, no caso de `c`, ele localiza dentro de `bar(..)`.

Caso houvesse um `c` definido em `bar(..)` e outro em `foo(..)`, a instrução `console.log(..)` teria localizado e utilizado o `c` definido em `bar(..)` e nunca chegaria até o valor definido em `foo(..)`.

**A consulta de escopo se encerra no momento que uma ocorrência é localizada**. Um mesmo identificador pode ser definido em diferentes camadas de escopo aninhadas, o que é chamado de "sombreamento" (*shadowing* -- o identificador interno "põe sombra" sobre o identificador externo). Independente do sombreamento, a consulta de escopo sempre se inicia no escopo mais próximo do ponto de execução atual, e segue seu caminho para fora/cima até a localização de uma ocorrência, quando se encerra.

**Nota:** Variáveis globais tornam-se automaticamente propriedades do objeto global (`window` nos navegadores, etc.), portanto *é possível* referenciar uma variável global de forma direta através de seu nome léxico, mas também de forma indireta ao referenciar uma propriedade do objeto global.

```js
window.a
```

Esta técnica garante o acesso a uma variável global que não poderia ser acessada por conta de um eventual sombreamento. Entretanto, variáveis não-globais e que foram sombreadas não podem ser acessadas. 

Não importa o *local* onde uma função é invocada, ou até mesmo *como* é invocada, seu escopo léxico será definido **apenas** pelo local onde a função foi declarada.

O processo de consulta ao escopo léxico ocorre *apenas" em identificadores de primeira classe, como `a`, `b` e `c`. Se você tivesse uma referência para `foo.bar.baz` em um trecho de código, ocorreria uma consulta ao escopo léxico para localizar o identificador `foo`, mas no momento que esta variável é localizada, regras de acesso à propriedades de objetos assumem o comando para resolução das propriedades `bar` e `baz`, respectivamente.

## Trapaceando o Léxico

Se o escopo léxico é de fato definido apenas pelo local onde uma função é declarada, cuja decisão vem do autor do código durante a programação, como pode haver uma maneira de "modificar" (ou trapacear) o escopo léxico em tempo de execução?

JavaScript possui dois mecanismos para isso. Ambos são vistos como má prática e igualmente (e amplamente!) desencorajados pela comunidade de modo geral, embora os argumentos que sustentam esta opinião normalmente não trazem consigo o ponto mais relevante: **trapacear o escopo léxico leva a um pior desempenho.**

Antes de explicar a questão da performance, porém, vamos olhar a forma com que estes dois mecanismos funcionam.

### `eval`

A função `eval(..)` em JavaScript recebe uma string como argumento e trata o conteúdo desta string como se tivesse de fato sido programado pelo autor do código naquele ponto do programa. Em outras palavras, você pode gerar código dinamicamente dentro do seu programa e executar este código como se estivesse lá desde o momento da programação.

Colocando desta forma, deve estar claro a forma com que `eval(..)` permite a você modificar o ambiente do escopo léxico ao trapacear e portanto fingir que aquele código estava lá desde o princípio.

Durante a execução das linhas que sucedem a chamada para `eval(..)`, o *Motor* não vai "saber" ou "se importar" se o código em questão foi interpretado dinamicamente e portanto modificou o ambiente do escopo léxico. O *Motor* vai seguir efetuando suas consultas ao escopo léxico da mesma forma de sempre.

Considere o código a seguir:

```js
function foo(str, a) {
	eval( str ); // trapaça!
	console.log( a, b );
}

var b = 2;

foo( "var b = 3;", 1 ); // 1, 3
```

A string `"var b = 3;"` é tratada, naquele ponto onde `eval(..)` é chamado, como um código que esteve lá desde o princípio. Pelo fato deste código declarar uma nova variável `b`, ele modifica o atual escopo léxico de `foo(..)`. O que ocorre, como mencionado acima, é que este código literalmente cria a variável `b` dentro de `foo(..)`, o que acaba por sombrear a variável `b` que foi declarada no escopo externo (neste caso, o global).

Quando a chamada para `console.log(..)` ocorre, são encontradas tanto `a` quanto `b` no escopo de `foo(..)`, e portanto nunca a variável `b` externa. Sendo assim, imprimimos "1, 3" em vez de "1, 2" como normalmente ocorreria.

**Note:** In this example, for simplicity's sake, the string of "code" we pass in was a fixed literal. But it could easily have been programmatically created by adding characters together based on your program's logic. `eval(..)` is usually used to execute dynamically created code, as dynamically evaluating essentially static code from a string literal would provide no real benefit to just authoring the code directly.

By default, if a string of code that `eval(..)` executes contains one or more declarations (either variables or functions), this action modifies the existing lexical scope in which the `eval(..)` resides. Technically, `eval(..)` can be invoked "indirectly", through various tricks (beyond our discussion here), which causes it to instead execute in the context of the global scope, thus modifying it. But in either case, `eval(..)` can at runtime modify an author-time lexical scope.

**Note:** `eval(..)` when used in a strict-mode program operates in its own lexical scope, which means declarations made inside of the `eval()` do not actually modify the enclosing scope.

```js
function foo(str) {
   "use strict";
   eval( str );
   console.log( a ); // ReferenceError: a is not defined
}

foo( "var a = 2" );
```

There are other facilities in JavaScript which amount to a very similar effect to `eval(..)`. `setTimeout(..)` and `setInterval(..)` *can* take a string for their respective first argument, the contents of which are `eval`uated as the code of a dynamically-generated function. This is old, legacy behavior and long-since deprecated. Don't do it!

The `new Function(..)` function constructor similarly takes a string of code in its **last** argument to turn into a dynamically-generated function (the first argument(s), if any, are the named parameters for the new function). This function-constructor syntax is slightly safer than `eval(..)`, but it should still be avoided in your code.

The use-cases for dynamically generating code inside your program are incredibly rare, as the performance degradations are almost never worth the capability.

### `with`

The other frowned-upon (and now deprecated!) feature in JavaScript which cheats lexical scope is the `with` keyword. There are multiple valid ways that `with` can be explained, but I will choose here to explain it from the perspective of how it interacts with and affects lexical scope.

`with` is typically explained as a short-hand for making multiple property references against an object *without* repeating the object reference itself each time.

For example:

```js
var obj = {
	a: 1,
	b: 2,
	c: 3
};

// more "tedious" to repeat "obj"
obj.a = 2;
obj.b = 3;
obj.c = 4;

// "easier" short-hand
with (obj) {
	a = 3;
	b = 4;
	c = 5;
}
```

However, there's much more going on here than just a convenient short-hand for object property access. Consider:

```js
function foo(obj) {
	with (obj) {
		a = 2;
	}
}

var o1 = {
	a: 3
};

var o2 = {
	b: 3
};

foo( o1 );
console.log( o1.a ); // 2

foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2 -- Oops, leaked global!
```

In this code example, two objects `o1` and `o2` are created. One has an `a` property, and the other does not. The `foo(..)` function takes an object reference `obj` as an argument, and calls `with (obj) { .. }` on the reference. Inside the `with` block, we make what appears to be a normal lexical reference to a variable `a`, an LHS reference in fact (see Chapter 1), to assign to it the value of `2`.

When we pass in `o1`, the `a = 2` assignment finds the property `o1.a` and assigns it the value `2`, as reflected in the subsequent `console.log(o1.a)` statement. However, when we pass in `o2`, since it does not have an `a` property, no such property is created, and `o2.a` remains `undefined`.

But then we note a peculiar side-effect, the fact that a global variable `a` was created by the `a = 2` assignment. How can this be?

The `with` statement takes an object, one which has zero or more properties, and **treats that object as if *it* is a wholly separate lexical scope**, and thus the object's properties are treated as lexically defined identifiers in that "scope".

**Note:** Even though a `with` block treats an object like a lexical scope, a normal `var` declaration inside that `with` block will not be scoped to that `with` block, but instead the containing function scope.

While the `eval(..)` function can modify existing lexical scope if it takes a string of code with one or more declarations in it, the `with` statement actually creates a **whole new lexical scope** out of thin air, from the object you pass to it.

Understood in this way, the "scope" declared by the `with` statement when we passed in `o1` was `o1`, and that "scope" had an "identifier" in it which corresponds to the `o1.a` property. But when we used `o2` as the "scope", it had no such `a` "identifier" in it, and so the normal rules of LHS identifier look-up (see Chapter 1) occurred.

Neither the "scope" of `o2`, nor the scope of `foo(..)`, nor the global scope even, has an `a` identifier to be found, so when `a = 2` is executed, it results in the automatic-global being created (since we're in non-strict mode).

It is a strange sort of mind-bending thought to see `with` turning, at runtime, an object and its properties into a "scope" *with* "identifiers". But that is the clearest explanation I can give for the results we see.

**Note:** In addition to being a bad idea to use, both `eval(..)` and `with` are affected (restricted) by Strict Mode. `with` is outright disallowed, whereas various forms of indirect or unsafe `eval(..)` are disallowed while retaining the core functionality.

### Performance

Both `eval(..)` and `with` cheat the otherwise author-time defined lexical scope by modifying or creating new lexical scope at runtime.

So, what's the big deal, you ask? If they offer more sophisticated functionality and coding flexibility, aren't these *good* features? **No.**

The JavaScript *Engine* has a number of performance optimizations that it performs during the compilation phase. Some of these boil down to being able to essentially statically analyze the code as it lexes, and pre-determine where all the variable and function declarations are, so that it takes less effort to resolve identifiers during execution.

But if the *Engine* finds an `eval(..)` or `with` in the code, it essentially has to *assume* that all its awareness of identifier location may be invalid, because it cannot know at lexing time exactly what code you may pass to `eval(..)` to modify the lexical scope, or the contents of the object you may pass to `with` to create a new lexical scope to be consulted.

In other words, in the pessimistic sense, most of those optimizations it *would* make are pointless if `eval(..)` or `with` are present, so it simply doesn't perform the optimizations *at all*.

Your code will almost certainly tend to run slower simply by the fact that you include an `eval(..)` or `with` anywhere in the code. No matter how smart the *Engine* may be about trying to limit the side-effects of these pessimistic assumptions, **there's no getting around the fact that without the optimizations, code runs slower.**

## Review (TL;DR)

Lexical scope means that scope is defined by author-time decisions of where functions are declared. The lexing phase of compilation is essentially able to know where and how all identifiers are declared, and thus predict how they will be looked-up during execution.

Two mechanisms in JavaScript can "cheat" lexical scope: `eval(..)` and `with`. The former can modify existing lexical scope (at runtime) by evaluating a string of "code" which has one or more declarations in it. The latter essentially creates a whole new lexical scope (again, at runtime) by treating an object reference *as* a "scope" and that object's properties as scoped identifiers.

The downside to these mechanisms is that it defeats the *Engine*'s ability to perform compile-time optimizations regarding scope look-up, because the *Engine* has to assume pessimistically that such optimizations will be invalid. Code *will* run slower as a result of using either feature. **Don't use them.**
