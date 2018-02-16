# You Don't Know JS: Escopos e Closures
# Capítulo 5: Scope Closure

Com esperança, até este ponto nós já alcançamos uma compreensão sólida e muito saudável de como escopo funciona.

Voltemos nossa atenção para uma incrivelmente importante, mas persistentemente ilusória, *quase mitológica*, parte da linguagem: **closure**. Se você acompanhou nossa discussão sobre escopo léxico até agora, a recompensa é que closure será, em grande parte, anticlímax, quase evidente por natureza. *Há um homem escondido atrás da cortina do mágico, e estamos prestes a vê-lo*. Não, seu nome não é Crockford!

No entanto, se você tem perguntas irritantes sobre escopo léxico, agora deve ser uma boa hora para voltar e rever o Capítulo 2, antes de prosseguir.

## Esclarecimento

Para aqueles que são um pouco experientes em JavaScript, mas que porventura nunca tenham entendido completamente o conceito de closures, *entender closure* pode parecer como um nirvana especial, que para atingí-lo é necessário muita luta e sacrifício.

Eu me lembro de anos atrás, quando eu tinha uma forte compreensão sobre JavaScript, mas não tinha ideia do que era closure. A sugestão de que havia *o outro lado* da linguagem, um que prometia ainda mais capacidade do que eu já possuia, me provocava. Me lembro de ler todo o código fonte dos primeiros frameworks tentando entender como eles realmente funcionavam. Lembro da primeira vez que algo do "módulo padrão" começou a surgir em minha mente. Eu me lembro dos momentos intensos de *a-ha!*.

O que eu não sabia naquela época, o que levei anos para entender, e o que eu espero transmitir para você em breve, é esse segredo: **em JavaScript, closure é tudo que estiver ao seu redor, você só precisa reconhecer e abraçar isso.** Closure não é uma ferramenta especial para a qual você deve aprender nova sintaxe e novos padrões. Não, closure não é uma arma que você aprende a manejar com destreza, como Luke treinado na Força.

Closures acontecem como um resultado da escrita de código que depende do escopo léxico. Elas apenas acontecem. Você nem sequer precisa criar closure intencionalmente para tirar proveito delas. Elas são criadas e usadas por você em todo o seu código. O que está faltando é o contexto mental apropriado para reconhecer, aceitar e alavancar closure por sua própria vontade.

O momento de esclarecimento deveria ser: **oh, closures já estão surgindo por todo o meu código, eu finalmente consigo *vê-las* agora.** O entendimento de closure é como quando Neo vê a Matrix pela primeira vez.

## Nitty Gritty

OK, chega de exageros e referências sem-vergonhas de filmes.

Aqui está uma definição curta e grossa do que você precisa saber para entender e reconhecer closure:

> Closure é quando uma função é capaz de lembrar e acessar seu escopo léxico, mesmo quando essa função está sendo executada fora do seu escopo léxico.

Vamos entrar em um pedaço código para ilustrar esta definição.

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a ); // 2
	}

	bar();
}

foo();
```

Esse código pode parecer familiar das nossas discussões sobre Escopo Aninhado. A função `bar()` tem *acesso* à variável `a` do escopo ao redor por causa das regras de consulta ao escopo léxico (neste caso, é uma referência RHS).

Isso é "closure"?

Bem, tecnicamente... *talvez*. Mas para a nossa definição o-que-você-precisa-saber acima... *não exatamente*. Eu acredito que a forma mais correta de explicar `bar()` fazendo referência a `a` é por meio das regras de procura do escopo léxico, e essas regras são *apenas* (uma importante!) **parte** do que closure é.

De uma perspectiva puramente acadêmica, o que é dito do trecho acima é que a função `bar()` tem uma *closure* sobre o escopo de `foo()` (e, realmente, até sobre o resto dos escopos a que tem acesso, como o escopo global no nosso caso). Colocando de uma forma ligeiramente diferente, é dito que `bar()` se fecha sobre o escopo de `foo()`. Por quê? Porque `bar()` aparece aninhado dentro de `foo()`. Claro e simples.

Mas, definida dessa forma, closure não é diretamente *observável*, nós nem vimos closure *exercida* nesse trecho. Nós vemos claramente o escopo léxico, mas closure permanece uma espécie de sombra misteriosa se deslocando por trás do código.

Vamos considerar, então, um código que trás closure totalmente para a luz:

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 2 -- Whoa, observamos closure, man.
```

A função `bar()` tem acesso léxico ao escopo interno de `foo()`. Mas, em seguida, nós temos `bar()`, a função em si, e passamos ela *como* um valor. Nesse caso, nós retornamos (`return`) o próprio objeto da função que `bar` faz referência.

Depois de executarmos `foo()`, nós atribuímos o valor retornado (nossa função interna `bar()`) para a variável chamada `baz` e então, nós invocamos `baz()`, que certamente está invocando nossa função interna `bar()`, apenas com um identificador diferente.

`bar()` é executada, com certeza. Mas, nesse caso, é executada *fora* do seu escopo léxico declarado.

Após a execução de `foo()`, normalmente nós esperamos que todo o escopo interno de `foo()` vá embora, porque nós sabemos que a *Engine* usa um *Coletor de Lixo* que vem paralelamente e libera a memória uma vez que não está mais em uso. Já que o conteúdo de `foo()` parece não está mais em uso, é natural que ele deve ser considerado *passado*.

Mas a "mágica" das closures não permite que isso aconteça. Esse escopo interno, de fato, *ainda* está "em uso" e, portanto, ele não desaparece. Quem está usando? **A função `bar()`**.

Em virtude de onde foi declarada, `bar()` tem uma closure sobre o escopo interno de `foo()`, que mantem esse escopo vivo para `bar()` fazer referência a qualquer momento posterior.

**`bar()` ainda possui uma referência para esse escopo, e essa referência é chamada de closure.**

Então, uns poucos microssegundos depois, quando a variável `baz` é chamada (chamando a função interna que inicialmente atribuímos o nome de `bar`), ela tem o devido acesso ao escopo léxico escrito no tempo da autoria do código, para que ela possa acessar à variável `a` exatamente como esperávamos.

A função está sendo invocada de forma adequada fora do seu escopo léxico de origem. **Closure** permite que a função continue a acessar o escopo léxico no qual foi definida no momento da sua concepção.

Naturalmente, qualquer uma das várias maneiras pelas quais as funções podem ser *passadas* como valores e, de fato, invocadas em outros lugares, são exemplos de observação/uso de *closure*.

```js
function foo() {
	var a = 2;

	function baz() {
		console.log( a ); // 2
	}

	bar( baz );
}

function bar(fn) {
	fn(); // olhe, mamãe, eu vejo closure!
}
```

Nós passamos a função interna `baz` para `bar`, chamamos essa função interna (agora rotulada de `fn`) e, quando fazemos isso, sua closure sobre o escopo interno de `foo()` é observada, acessando `a`.

Estas passagens de funções também podem ocorrer de forma indireta.

```js
var fn;

function foo() {
	var a = 2;

	function baz() {
		console.log( a );
	}

	fn = baz; // atribuindo `baz` à variável global
}

function bar() {
	fn(); // olhe, mamãe, eu vejo closure!
}

foo();

bar(); // 2
```

Seja qual for a forma que usarmos para *transportar* uma função interna para fora do seu escopo léxico, ela irá manter uma referência de escopo de onde ela for declarada originalmente, e onde for que a executarmos, essa closure irá ocorrer.

## Agora eu posso ver

Os fragmentos de código anteriores são um tanto acadêmicos artificialmente contruídos para ilustrar o *uso de closures*. Mas eu prometi para você uma coisa mais que apenas um novo brinquedinho. Eu prometi que closure seria uma coisa ao seu redor, em sou código existente. Vamos agora *ver* essa verdade.

```js
function wait(message) {

	setTimeout( function timer(){
		console.log( message );
	}, 1000 );

}

wait( "Hello, closure!" );
```

Tomamos uma função interna (chamada `timer`) e passamos ela para o `setTimeout(..)`. Mas `timer` tem um escopo fechado sobre o escopo de `wait(..)`, mantendo e usando uma referência para a variável `message`.

Mil milésimos de segundo depois de executarmos `wait(..)`, e seu escopo interno deveria ter sido extinto há muito tempo, a tal função interna `timer` ainda tem uma closure sobre esse escopo.

Lá no fundo, nas entranhas da *Engine*, o utilitário embutido `setTimeout(..)` faz uma referência por algum parâmetro, provavelmente chamado `fn`, ou `func`, ou alguma coisa do tipo. A *Engine* vai invocar essa função, que está invocando nossa função interna `timer`, e a referência do escopo léxico ainda está intacta.

**Closure.**

Ou, se você é da religião do jQuery (ou algum framework JS, para este caso):

```js
function setupBot(name,selector) {
	$( selector ).click( function activator(){
		console.log( "Activating: " + name );
	} );
}

setupBot( "Closure Bot 1", "#bot_1" );
setupBot( "Closure Bot 2", "#bot_2" );
```

Não tenho certeza do tipo de código você escreve, mas eu normalmente escrevo código que é responsável por controlar todo um exército mundial de drones de closure bots, então é totalmente realista!

(Algumas) brincadeiras à parte, essencialmente *sempre que* and *onder quer que* você tratar funções (que acessam seu respectivo escopo léxico) como valores de primeira classe e as passe por aí, provavelmente você vê aquelas funções que exercem closure. Sejam elas timers, manipuladores de eventos, requisições Ajax, mensagens de janelas cruzadas, web workers, ou alguma outra tarefa assíncrona (ou síncrona!), quando você passa em uma *função de callback*, prepare-se para lançar algumas closures por aí!

**Nota:** O capítulo 3 introduz o padrão IIFE. Embora seja frequentemente dito que IIFE (sozinho) é um exemplo de closure, Eu devo discordar um pouco, pela nossa definição acima.

```js
var a = 2;

(function IIFE(){
	console.log( a );
})();
```

Esse código "funciona", mas isso não é rigorosamente um exemplo de closure. Por quê? Porque a função (que aqui nós nomeamos de "IIFE") não é executada fora do seu escopo léxico. Ela ainda é chamada bem ali, no mesmo escopo que foi declarada (o escopo ao redor/global que também contém `a`). `a` é encontrado pelo look-up normal do escopo léxico, não exatamente por closure.

Enquanto a closure poderia técnicamente estar acontecendo na hora da declaração, _não_ é estritamente observável, e então, como eles dizem, _é uma árvore caindo na floresta sem ninguém por perto para ouvir._

Embora uma IIFE não seja *em si* um exemplo de closure, ela realmente cria escopo e é uma das ferramentas mais comuns que usamos para criar um escopo que pode ser fechado. Portanto, as IIFEs estão, de fato, profundamente relacionadas a closures, mesmo que não exerçam closure por sí mesmas.

Put this book down right now, dear reader. I have a task for you. Go open up some of your recent JavaScript code. Look for your functions-as-values and identify where you are already using closure and maybe didn't even know it before.

I'll wait.

Now... you see!

## Loops + Closure

The most common canonical example used to illustrate closure involves the humble for-loop.

```js
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

**Note:** Linters often complain when you put functions inside of loops, because the mistakes of not understanding closure are **so common among developers**. We explain how to do so properly here, leveraging the full power of closure. But that subtlety is often lost on linters and they will complain regardless, assuming you don't *actually* know what you're doing.

The spirit of this code snippet is that we would normally *expect* for the behavior to be that the numbers "1", "2", .. "5" would be printed out, one at a time, one per second, respectively.

In fact, if you run this code, you get "6" printed out 5 times, at the one-second intervals.

**Huh?**

Firstly, let's explain where `6` comes from. The terminating condition of the loop is when `i` is *not* `<=5`. The first time that's the case is when `i` is 6. So, the output is reflecting the final value of the `i` after the loop terminates.

This actually seems obvious on second glance. The timeout function callbacks are all running well after the completion of the loop. In fact, as timers go, even if it was `setTimeout(.., 0)` on each iteration, all those function callbacks would still run strictly after the completion of the loop, and thus print `6` each time.

But there's a deeper question at play here. What's *missing* from our code to actually have it behave as we semantically have implied?

What's missing is that we are trying to *imply* that each iteration of the loop "captures" its own copy of `i`, at the time of the iteration. But, the way scope works, all 5 of those functions, though they are defined separately in each loop iteration, all **are closed over the same shared global scope**, which has, in fact, only one `i` in it.

Put that way, *of course* all functions share a reference to the same `i`. Something about the loop structure tends to confuse us into thinking there's something else more sophisticated at work. There is not. There's no difference than if each of the 5 timeout callbacks were just declared one right after the other, with no loop at all.

OK, so, back to our burning question. What's missing? We need more ~~cowbell~~ closured scope. Specifically, we need a new closured scope for each iteration of the loop.

We learned in Chapter 3 that the IIFE creates scope by declaring a function and immediately executing it.

Let's try:

```js
for (var i=1; i<=5; i++) {
	(function(){
		setTimeout( function timer(){
			console.log( i );
		}, i*1000 );
	})();
}
```

Does that work? Try it. Again, I'll wait.

I'll end the suspense for you. **Nope.** But why? We now obviously have more lexical scope. Each timeout function callback is indeed closing over its own per-iteration scope created respectively by each IIFE.

It's not enough to have a scope to close over **if that scope is empty**. Look closely. Our IIFE is just an empty do-nothing scope. It needs *something* in it to be useful to us.

It needs its own variable, with a copy of the `i` value at each iteration.

```js
for (var i=1; i<=5; i++) {
	(function(){
		var j = i;
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})();
}
```

**Eureka! It works!**

A slight variation some prefer is:

```js
for (var i=1; i<=5; i++) {
	(function(j){
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})( i );
}
```

Of course, since these IIFEs are just functions, we can pass in `i`, and we can call it `j` if we prefer, or we can even call it `i` again. Either way, the code works now.

The use of an IIFE inside each iteration created a new scope for each iteration, which gave our timeout function callbacks the opportunity to close over a new scope for each iteration, one which had a variable with the right per-iteration value in it for us to access.

Problem solved!

### Block Scoping Revisited

Look carefully at our analysis of the previous solution. We used an IIFE to create new scope per-iteration. In other words, we actually *needed* a per-iteration **block scope**. Chapter 3 showed us the `let` declaration, which hijacks a block and declares a variable right there in the block.

**It essentially turns a block into a scope that we can close over.** So, the following awesome code "just works":

```js
for (var i=1; i<=5; i++) {
	let j = i; // yay, block-scope for closure!
	setTimeout( function timer(){
		console.log( j );
	}, j*1000 );
}
```

*But, that's not all!* (in my best Bob Barker voice). There's a special behavior defined for `let` declarations used in the head of a for-loop. This behavior says that the variable will be declared not just once for the loop, **but each iteration**. And, it will, helpfully, be initialized at each subsequent iteration with the value from the end of the previous iteration.

```js
for (let i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

How cool is that? Block scoping and closure working hand-in-hand, solving all the world's problems. I don't know about you, but that makes me a happy JavaScripter.

## Modules

There are other code patterns which leverage the power of closure but which do not on the surface appear to be about callbacks. Let's examine the most powerful of them: *the module*.

```js
function foo() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}
}
```

As this code stands right now, there's no observable closure going on. We simply have some private data variables `something` and `another`, and a couple of inner functions `doSomething()` and `doAnother()`, which both have lexical scope (and thus closure!) over the inner scope of `foo()`.

But now consider:

```js
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

This is the pattern in JavaScript we call *module*. The most common way of implementing the module pattern is often called "Revealing Module", and it's the variation we present here.

Let's examine some things about this code.

Firstly, `CoolModule()` is just a function, but it *has to be invoked* for there to be a module instance created. Without the execution of the outer function, the creation of the inner scope and the closures would not occur.

Secondly, the `CoolModule()` function returns an object, denoted by the object-literal syntax `{ key: value, ... }`. The object we return has references on it to our inner functions, but *not* to our inner data variables. We keep those hidden and private. It's appropriate to think of this object return value as essentially a **public API for our module**.

This object return value is ultimately assigned to the outer variable `foo`, and then we can access those property methods on the API, like `foo.doSomething()`.

**Note:** It is not required that we return an actual object (literal) from our module. We could just return back an inner function directly. jQuery is actually a good example of this. The `jQuery` and `$` identifiers are the public API for the jQuery "module", but they are, themselves, just a function (which can itself have properties, since all functions are objects).

The `doSomething()` and `doAnother()` functions have closure over the inner scope of the module "instance" (arrived at by actually invoking `CoolModule()`). When we transport those functions outside of the lexical scope, by way of property references on the object we return, we have now set up a condition by which closure can be observed and exercised.

To state it more simply, there are two "requirements" for the module pattern to be exercised:

1. There must be an outer enclosing function, and it must be invoked at least once (each time creates a new module instance).

2. The enclosing function must return back at least one inner function, so that this inner function has closure over the private scope, and can access and/or modify that private state.

An object with a function property on it alone is not *really* a module. An object which is returned from a function invocation which only has data properties on it and no closured functions is not *really* a module, in the observable sense.

The code snippet above shows a standalone module creator called `CoolModule()` which can be invoked any number of times, each time creating a new module instance. A slight variation on this pattern is when you only care to have one instance, a "singleton" of sorts:

```js
var foo = (function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

Here, we turned our module function into an IIFE (see Chapter 3), and we *immediately* invoked it and assigned its return value directly to our single module instance identifier `foo`.

Modules are just functions, so they can receive parameters:

```js
function CoolModule(id) {
	function identify() {
		console.log( id );
	}

	return {
		identify: identify
	};
}

var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );

foo1.identify(); // "foo 1"
foo2.identify(); // "foo 2"
```

Another slight but powerful variation on the module pattern is to name the object you are returning as your public API:

```js
var foo = (function CoolModule(id) {
	function change() {
		// modifying the public API
		publicAPI.identify = identify2;
	}

	function identify1() {
		console.log( id );
	}

	function identify2() {
		console.log( id.toUpperCase() );
	}

	var publicAPI = {
		change: change,
		identify: identify1
	};

	return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

By retaining an inner reference to the public API object inside your module instance, you can modify that module instance **from the inside**, including adding and removing methods, properties, *and* changing their values.

### Modern Modules

Various module dependency loaders/managers essentially wrap up this pattern of module definition into a friendly API. Rather than examine any one particular library, let me present a *very simple* proof of concept **for illustration purposes (only)**:

```js
var MyModules = (function Manager() {
	var modules = {};

	function define(name, deps, impl) {
		for (var i=0; i<deps.length; i++) {
			deps[i] = modules[deps[i]];
		}
		modules[name] = impl.apply( impl, deps );
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();
```

The key part of this code is `modules[name] = impl.apply(impl, deps)`. This is invoking the definition wrapper function for a module (passing in any dependencies), and storing the return value, the module's API, into an internal list of modules tracked by name.

And here's how I might use it to define some modules:

```js
MyModules.define( "bar", [], function(){
	function hello(who) {
		return "Let me introduce: " + who;
	}

	return {
		hello: hello
	};
} );

MyModules.define( "foo", ["bar"], function(bar){
	var hungry = "hippo";

	function awesome() {
		console.log( bar.hello( hungry ).toUpperCase() );
	}

	return {
		awesome: awesome
	};
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

Both the "foo" and "bar" modules are defined with a function that returns a public API. "foo" even receives the instance of "bar" as a dependency parameter, and can use it accordingly.

Spend some time examining these code snippets to fully understand the power of closures put to use for our own good purposes. The key take-away is that there's not really any particular "magic" to module managers. They fulfill both characteristics of the module pattern I listed above: invoking a function definition wrapper, and keeping its return value as the API for that module.

In other words, modules are just modules, even if you put a friendly wrapper tool on top of them.

### Future Modules

ES6 adds first-class syntax support for the concept of modules. When loaded via the module system, ES6 treats a file as a separate module. Each module can both import other modules or specific API members, as well export their own public API members.

**Note:** Function-based modules aren't a statically recognized pattern (something the compiler knows about), so their API semantics aren't considered until run-time. That is, you can actually modify a module's API during the run-time (see earlier `publicAPI` discussion).

By contrast, ES6 Module APIs are static (the APIs don't change at run-time). Since the compiler knows *that*, it can (and does!) check during (file loading and) compilation that a reference to a member of an imported module's API *actually exists*. If the API reference doesn't exist, the compiler throws an "early" error at compile-time, rather than waiting for traditional dynamic run-time resolution (and errors, if any).

ES6 modules **do not** have an "inline" format, they must be defined in separate files (one per module). The browsers/engines have a default "module loader" (which is overridable, but that's well-beyond our discussion here) which synchronously loads a module file when it's imported.

Consider:

**bar.js**
```js
function hello(who) {
	return "Let me introduce: " + who;
}

export hello;
```

**foo.js**
```js
// import only `hello()` from the "bar" module
import hello from "bar";

var hungry = "hippo";

function awesome() {
	console.log(
		hello( hungry ).toUpperCase()
	);
}

export awesome;
```

```js
// import the entire "foo" and "bar" modules
module foo from "foo";
module bar from "bar";

console.log(
	bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

**Note:** Separate files **"foo.js"** and **"bar.js"** would need to be created, with the contents as shown in the first two snippets, respectively. Then, your program would load/import those modules to use them, as shown in the third snippet.

`import` imports one or more members from a module's API into the current scope, each to a bound variable (`hello` in our case). `module` imports an entire module API to a bound variable (`foo`, `bar` in our case). `export` exports an identifier (variable, function) to the public API for the current module. These operators can be used as many times in a module's definition as is necessary.

The contents inside the *module file* are treated as if enclosed in a scope closure, just like with the function-closure modules seen earlier.

## Review (TL;DR)

Closure seems to the un-enlightened like a mystical world set apart inside of JavaScript which only the few bravest souls can reach. But it's actually just a standard and almost obvious fact of how we write code in a lexically scoped environment, where functions are values and can be passed around at will.

**Closure is when a function can remember and access its lexical scope even when it's invoked outside its lexical scope.**

Closures can trip us up, for instance with loops, if we're not careful to recognize them and how they work. But they are also an immensely powerful tool, enabling patterns like *modules* in their various forms.

Modules require two key characteristics: 1) an outer wrapping function being invoked, to create the enclosing scope 2) the return value of the wrapping function must include reference to at least one inner function that then has closure over the private inner scope of the wrapper.

Now we can see closures all around our existing code, and we have the ability to recognize and leverage them to our own benefit!
