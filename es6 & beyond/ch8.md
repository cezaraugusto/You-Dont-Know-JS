# You Don't Know JS: ES6 & Além
# Capítulo 8: ES6 & Além

No momento atual em que escrevo, o rascunho final do ES6 (*ECMAScript 2015*) está perto de ser encaminhado ao último voto oficial de aprovação pelo ECMA. Entretanto, ao mesmo tempo em que o ES6 está sendo finalizado, o comitê TC39 já está trabalhando duro nas funcionalidades para o ES7/2016 e além. 

Como discutimos no Capítulo 1, espera-se que o ritmo de progresso do JS será mais acelerado, e, ao invés de ser atualizado uma vez a cada vários anos, terá uma versão oficial uma vez por ano (por isso a nomenclatura baseada no ano). Isso, por si só, já mudará radicalmente como os desenvolvedores JS aprendem e se mantêm atualizado com a linguagem.

Ainda mais importante que isso, o comitê passará em realidade a trabalhar em uma funcionalidade de cada vez. Assim que o spec de uma funcionalidade esteja completo e todos suas excentricidades tiverem sido testadas em experimentos de implementação em alguns navegadores, ela será considerada estável suficiente para uso. Somos encorajados a adotar funcionalidades uma vez estejam prontas ao invés de esperar por uma votação de padrões oficiais. Se você ainda não aprendeu ES6, já está mais que na hora de fazer isso! 

No momento atual em que escrevo, uma lista de propostas futuras e seus status podem ser encontrados aqui (https://github.com/tc39/ecma262#current-proposals).

Transpilers e polyfills serão utilizados por nós como ponte para essas novas funcionalidade, antes mesmo que todos os navegadores que apoiamos as tenham implementado. Babel, Traceur, e vários outros grandes transpilers já tem suporte para algumas dessas funcionalidades pós-ES6 que são mais prováveis de serem estabilizadas. 

Com isso em mente, é hora de dar uma olhada em algumas dessas funcionalidades. Vamos lá! 

**Atenção:** Essas funcionalidades estão em diferentes estágios de desenvolvimento. Apesar de ser provável que elas venham a existir, e provavelmente parecerão similares, leia o conteúdo desse capítulo com bastante cautela. Esse capítulo evoluirá em edições futuras desse livro a medida que essas (e outras!) funcionalidades estejam finalizadas.

## `funções async`

Na seção “Generators + Promises” do Capítulo 4, mencionamos que existe uma proposta para suporte sintático direto para o padrão de *generators* que `YIELD` (entregam) promessas (*promises*) à uma utilidade do tipo RUNNER que irá retomá-lo uma vez a promessa seja completada. Vamos dar uma olhada rápida nessa funcionalidade proposta, chamada de `função async`. 

Lembre-se desse exemplo de *generator* do Capítulo 4: 

```js
run( function *main() {
	var ret = yield step1();

	try {
		ret = yield step2( ret );
	}
	catch (err) {
		ret = yield step2Failed( err );
	}

	ret = yield Promise.all([
		step3a( ret ),
		step3b( ret ),
		step3c( ret )
	]);

	yield step4( ret );
} )
.then(
	function fulfilled(){
		// `*main()` foi completada com sucesso
	},
	function rejected(reason){
		// Oops, algo deu errado
	}
);
```
A sintaxe proposta para `função async` pode expressar essa mesma lógica de controle de fluxo sem precisar da utilidade `run(..)`, porque o JS saberá automaticamente como buscar promessas para esperar e retomar. Considere:

```js
async function main() {
	var ret = await step1();

	try {
		ret = await step2( ret );
	}
	catch (err) {
		ret = await step2Failed( err );
	}

	ret = await Promise.all( [
		step3a( ret ),
		step3b( ret ),
		step3c( ret )
	] );

	await step4( ret );
}

main()
.then(
	function fulfilled(){
		// `main()` completada com sucesso
	},
	function rejected(reason){
		// Oops, algo deu errado 
	}
);
```
Ao invés da declaração `function *main() { ..`, declaramos com o formato `async function main() { ..`. E ao invés de `yield` (entregar) uma promessa, nós a esperamos (`await`). A chamada para executar a função `main()` em realidade retorna a promessa que podemos observar diretamente. É o equivalente à promessa que recebemos de volta da execução `run(main)`.

Você consegue ver a simeteria? A `função async` é basicamente açúcar sintático para padrões como os de generators + promises + `run(..)`; por detrás dos panos, funciona da mesma maneira!

Se você é um desenvolvedor C# e `async`/`await` parece familiar, é porque essa funcionalidade foi diretamente inspirada por uma de C#. É bom ver precedência de linguagem formando convergência. 

Babel, Traceur e outros transpilers já tem um suporte antecipado para o status atual das `funções async`, então você já poderia começar a usá-las. Entretanto, na próxima seção "Ressalvas" veremos porque talvez você não deveria pular nesse barco por agora.

**Observação:** Há também uma proposta para `função* async`, que seria chamada de "async generator." Você poderia usar `yield` e `await` no mesmo código e até mesmo combinar essas operações em uma mesma instrução: `x = await yield y`. Essa proposta de "async generator" parece estar ainda em curso – ou seja, o valor de retorno não está completamente definido ainda. Algumas pessoas pensam que deveria ser *observável*, o que seria como a combinação de um iterador e uma promise. Por agora, não iremos entrar em mais detalhes sobre esse tópico, mas fique atento à medida que evolua. 

### Ressalvas

Um ponto de discórdia não resolvido com a `função async` se deve ao fato de que ela só retorna uma promessa, e não é possível cancelar uma `função async` desde fora dela. Isso pode ser um problema se a operação async é RESOURCE INTENSIVE e você quisesse liberar os RESOURCES assim que você tivesse certeza que o resultado não fosse mais necessário. 

Por exemplo: 

```js
async function request(url) {
	var resp = await (
		new Promise( function(resolve,reject){
			var xhr = new XMLHttpRequest();
			xhr.open( "GET", url );
			xhr.onreadystatechange = function(){
				if (xhr.readyState == 4) {
					if (xhr.status == 200) {
						resolve( xhr );
					}
					else {
						reject( xhr.statusText );
					}
				}
			};
			xhr.send();
		} )
	);

	return resp.responseText;
}

var pr = request( "http://some.url.1" );

pr.then(
	function fulfilled(responseText){
		// sucesso do ajax
	},
	function rejected(reason){
		// Oops, algo deu errado
	}
);
```

Essa função `request(..)` que é de certa forma parecida com a utilidade `fetch(..)` que recentemente se propôs incluir na plataforma web. A preocupação então é: o que acontece se você quer usar o valor `pr` para de alguma maneira indicar que você quer cancelar um pedido de Ajax de longa-duração, por exemplo?

Promessas não são canceláveis (pelo menos não no momento em que escrevo). Na minha opinião, assim como de muitas outras pessoas, elas nunca deveriam ser (veja o título *Async e Performance*). E mesmo se uma promessa tivesse um método de cancelar como `cancel()`, isso deveria mesmo significar que chamar  `pr.cancel()` propagaria o sinal de cancelamento de volta por todo o camino da sequência até a função `async`?

Várias possíveis resoluções a esse debate surgiram: 

* `funções async` não seriam canceláveis de maneira alguma (status quo)
* Um "token de cancelamento" poderia ser passado como argumento a uma função async na hora da função ser chamada
* O valor retornado se torna um tipo de promessa-cancelável que é adicionado 
* O valor retornado se torna algo que não uma promessa (por exemplo: observável, ou um token de controle com capacidades de promessa e cancelamento) 

No momento em que escrevo, `funções async` devolvem promessas normais, então é menos provável que o valor retornado irá mudar completamente. Entretanto, é muito cedo para saber onde as coisas vão terminar. Fique de olho nessa discussão. 

## `Object.observe(..)`

One of the holy grails of front-end web development is data binding -- listening for updates to a data object and syncing the DOM representation of that data. Most JS frameworks provide some mechanism for these sorts of operations.

It appears likely that post ES6, we'll see support added directly to the language, via a utility called `Object.observe(..)`. Essentially, the idea is that you can set up a listener to observe an object's changes, and have a callback called any time a change occurs. You can then update the DOM accordingly, for instance.

There are six types of changes that you can observe:

* add
* update
* delete
* reconfigure
* setPrototype
* preventExtensions

By default, you'll be notified of all these change types, but you can filter down to only the ones you care about.

Consider:

```js
var obj = { a: 1, b: 2 };

Object.observe(
	obj,
	function(changes){
		for (var change of changes) {
			console.log( change );
		}
	},
	[ "add", "update", "delete" ]
);

obj.c = 3;
// { name: "c", object: obj, type: "add" }

obj.a = 42;
// { name: "a", object: obj, type: "update", oldValue: 1 }

delete obj.b;
// { name: "b", object: obj, type: "delete", oldValue: 2 }
```

In addition to the main `"add"`, `"update"`, and `"delete"` change types:

* The `"reconfigure"` change event is fired if one of the object's properties is reconfigured with `Object.defineProperty(..)`, such as changing its `writable` attribute. See the *this & Object Prototypes* title of this series for more information.
* The `"preventExtensions"` change event is fired if the object is made non-extensible via `Object.preventExtensions(..)`.

   Because both `Object.seal(..)` and `Object.freeze(..)` also imply `Object.preventExtensions(..)`, they'll also fire its corresponding change event. In addition, `"reconfigure"` change events will also be fired for each property on the object.
* The `"setPrototype"` change event is fired if the `[[Prototype]]` of an object is changed, either by setting it with the `__proto__` setter, or using `Object.setPrototypeOf(..)`.

Notice that these change events are notified immediately after said change. Don't confuse this with proxies (see Chapter 7) where you can intercept the actions before they occur. Object observation lets you respond after a change (or set of changes) occurs.

### Custom Change Events

In addition to the six built-in change event types, you can also listen for and fire custom change events.

Consider:

```js
function observer(changes){
	for (var change of changes) {
		if (change.type == "recalc") {
			change.object.c =
				change.object.oldValue +
				change.object.a +
				change.object.b;
		}
	}
}

function changeObj(a,b) {
	var notifier = Object.getNotifier( obj );

	obj.a = a * 2;
	obj.b = b * 3;

	// queue up change events into a set
	notifier.notify( {
		type: "recalc",
		name: "c",
		oldValue: obj.c
	} );
}

var obj = { a: 1, b: 2, c: 3 };

Object.observe(
	obj,
	observer,
	["recalc"]
);

changeObj( 3, 11 );

obj.a;			// 12
obj.b;			// 30
obj.c;			// 3
```

The change set (`"recalc"` custom event) has been queued for delivery to the observer, but not delivered yet, which is why `obj.c` is still `3`.

The changes are by default delivered at the end of the current event loop (see the *Async & Performance* title of this series). If you want to deliver them immediately, use `Object.deliverChangeRecords(observer)`. Once the change events are delivered, you can observe `obj.c` updated as expected:

```js
obj.c;			// 42
```

In the previous example, we called `notifier.notify(..)` with the complete change event record. An alternative form for queuing change records is to use `performChange(..)`, which separates specifying the type of the event from the rest of event record's properties (via a function callback). Consider:

```js
notifier.performChange( "recalc", function(){
	return {
		name: "c",
		// `this` is the object under observation
		oldValue: this.c
	};
} );
```

In certain circumstances, this separation of concerns may map more cleanly to your usage pattern.

### Ending Observation

Just like with normal event listeners, you may wish to stop observing an object's change events. For that, you use `Object.unobserve(..)`.

For example:

```js
var obj = { a: 1, b: 2 };

Object.observe( obj, function observer(changes) {
	for (var change of changes) {
		if (change.type == "setPrototype") {
			Object.unobserve(
				change.object, observer
			);
			break;
		}
	}
} );
```

In this trivial example, we listen for change events until we see the `"setPrototype"` event come through, at which time we stop observing any more change events.

## Exponentiation Operator

An operator has been proposed for JavaScript to perform exponentiation in the same way that `Math.pow(..)` does. Consider:

```js
var a = 2;

a ** 4;			// Math.pow( a, 4 ) == 16

a **= 3;		// a = Math.pow( a, 3 )
a;				// 8
```

**Note:** `**` is essentially the same as it appears in Python, Ruby, Perl, and others.

## Objects Properties and `...`

As we saw in the "Too Many, Too Few, Just Enough" section of Chapter 2, the `...` operator is pretty obvious in how it relates to spreading or gathering arrays. But what about objects?

Such a feature was considered for ES6, but was deferred to be considered after ES6 (aka "ES7" or "ES2016" or ...). Here's how it might work in that "beyond ES6" timeframe:

```js
var o1 = { a: 1, b: 2 },
	o2 = { c: 3 },
	o3 = { ...o1, ...o2, d: 4 };

console.log( o3.a, o3.b, o3.c, o3.d );
// 1 2 3 4
```

The `...` operator might also be used to gather an object's destructured properties back into an object:

```js
var o1 = { b: 2, c: 3, d: 4 };
var { b, ...o2 } = o1;

console.log( b, o2.c, o2.d );		// 2 3 4
```

Here, the `...o2` re-gathers the destructured `c` and `d` properties back into an `o2` object (`o2` does not have a `b` property like `o1` does).

Again, these are just proposals under consideration beyond ES6. But it'll be cool if they do land.

## `Array#includes(..)`

One extremely common task JS developers need to perform is searching for a value inside an array of values. The way this has always been done is:

```js
var vals = [ "foo", "bar", 42, "baz" ];

if (vals.indexOf( 42 ) >= 0) {
	// found it!
}
```

The reason for the `>= 0` check is because `indexOf(..)` returns a numeric value of `0` or greater if found, or `-1` if not found. In other words, we're using an index-returning function in a boolean context. But because `-1` is truthy instead of falsy, we have to be more manual with our checks.

In the *Types & Grammar* title of this series, I explored another pattern that I slightly prefer:

```js
var vals = [ "foo", "bar", 42, "baz" ];

if (~vals.indexOf( 42 )) {
	// found it!
}
```

The `~` operator here conforms the return value of `indexOf(..)` to a value range that is suitably boolean coercible. That is, `-1` produces `0` (falsy), and anything else produces a non-zero (truthy) value, which is what we for deciding if we found the value or not.

While I think that's an improvement, others strongly disagree. However, no one can argue that `indexOf(..)`'s searching logic is perfect. It fails to find `NaN` values in the array, for example.

So a proposal has surfaced and gained a lot of support for adding a real boolean-returning array search method, called `includes(..)`:

```js
var vals = [ "foo", "bar", 42, "baz" ];

if (vals.includes( 42 )) {
	// found it!
}
```

**Note:** `Array#includes(..)` uses matching logic that will find `NaN` values, but will not distinguish between `-0` and `0` (see the *Types & Grammar* title of this series). If you don't care about `-0` values in your programs, this will likely be exactly what you're hoping for. If you *do* care about `-0`, you'll need to do your own searching logic, likely using the `Object.is(..)` utility (see Chapter 6).

## SIMD

We cover Single Instruction, Multiple Data (SIMD) in more detail in the *Async & Performance* title of this series, but it bears a brief mention here, as it's one of the next likely features to land in a future JS.

The SIMD API exposes various low-level (CPU) instructions that can operate on more than a single number value at a time. For example, you'll be able to specify two *vectors* of 4 or 8 numbers each, and multiply the respective elements all at once (data parallelism!).

Consider:

```js
var v1 = SIMD.float32x4( 3.14159, 21.0, 32.3, 55.55 );
var v2 = SIMD.float32x4( 2.1, 3.2, 4.3, 5.4 );

SIMD.float32x4.mul( v1, v2 );
// [ 6.597339, 67.2, 138.89, 299.97 ]
```

SIMD will include several other operations besides `mul(..)` (multiplication), such as `sub()`, `div()`, `abs()`, `neg()`, `sqrt()`, and many more.

Parallel math operations are critical for the next generations of high performance JS applications.

## WebAssembly (WASM)

Brendan Eich made a late breaking announcement near the completion of the first edition of this title that has the potential to significantly impact the future path of JavaScript: WebAssembly (WASM). We will not be able to cover WASM in detail here, as it's extremely early at the time of this writing. But this title would be incomplete without at least a brief mention of it.

One of the strongest pressures on the recent (and near future) design changes of the JS language has been the desire that it become a more suitable target for transpilation/cross-compilation from other languages (like C/C++, ClojureScript, etc.). Obviously, performance of code running as JavaScript has been a primary concern.

As discussed in the *Async & Performance* title of this series, a few years ago a group of developers at Mozilla introduced an idea to JavaScript called ASM.js. ASM.js is a subset of valid JS that most significantly restricts certain actions that make code hard for the JS engine to optimize. The result is that ASM.js compatible code running in an ASM-aware engine can run remarkably faster, nearly on par with native optimized C equivalents. Many viewed ASM.js as the most likely backbone on which performance-hungry applications would ride in JavaScript.

In other words, all roads to running code in the browser *lead through JavaScript*.

That is, until the WASM announcement. WASM provides an alternate path for other languages to target the browser's runtime environment without having to first pass through JavaScript. Essentially, if WASM takes off, JS engines will grow an extra capability to execute a binary format of code that can be seen as somewhat similar to a bytecode (like that which runs on the JVM).

WASM proposes a format for a binary representation of a highly compressed AST (syntax tree) of code, which can then give instructions directly to the JS engine and its underpinnings, without having to be parsed by JS, or even behave by the rules of JS. Languages like C or C++ can be compiled directly to the WASM format instead of ASM.js, and gain an extra speed advantage by skipping the JS parsing.

The near term for WASM is to have parity with ASM.js and indeed JS. But eventually, it's expected that WASM would grow new capabilities that surpass anything JS could do. For example, the pressure for JS to evolve radical features like threads -- a change that would certainly send major shockwaves through the JS ecosystem -- has a more hopeful future as a future WASM extension, relieving the pressure to change JS.

In fact, this new roadmap opens up many new roads for many languages to target the web runtime. That's an exciting new future path for the web platform!

What does it mean for JS? Will JS become irrelevant or "die"? Absolutely not. ASM.js will likely not see much of a future beyond the next couple of years, but the majority of JS is quite safely anchored in the web platform story.

Proponents of WASM suggest its success will mean that the design of JS will be protected from pressures that would have eventually stretched it beyond assumed breaking points of reasonability. It is projected that WASM will become the preferred target for high-performance parts of applications, as authored in any of a myriad of different languages.

Interestingly, JavaScript is one of the lesser likely languages to target WASM in the future. There may be future changes that carve out subsets of JS that might be tenable for such targeting, but that path doesn't seem high on the priority list.

While JS likely won't be much of a WASM funnel, JS code and WASM code will be able to interoperate in the most significant ways, just as naturally as current module interactions. You can imagine calling a JS function like `foo()` and having that actually invoke a WASM function of that name with the power to run well outside the constraints of the rest of your JS.

Things which are currently written in JS will probably continue to always be written in JS, at least for the foreseeable future. Things which are transpiled to JS will probably eventually at least consider targeting WASM instead. For things which need the utmost in performance with minimal tolerance for layers of abstraction, the likely choice will be to find a suitable non-JS language to author in, then targeting WASM.

There's a good chance this shift will be slow, and will be years in the making. WASM landing in all the major browser platforms is probably a few years out at best. In the meantime, the WASM project (https://github.com/WebAssembly) has an early polyfill to demonstrate proof-of-concept for its basic tenets.

But as time goes on, and as WASM learns new non-JS tricks, it's not too much a stretch of imagination to see some currently-JS things being refactored to a WASM-targetable language. For example, the performance sensitive parts of frameworks, game engines, and other heavily used tools might very well benefit from such a shift. Developers using these tools in their web applications likely won't notice much difference in usage or integration, but will just automatically take advantage of the performance and capabilities.

What's certain is that the more real WASM becomes over time, the more it means to the trajectory and design of JavaScript. It's perhaps one of the most important "beyond ES6" topics developers should keep an eye on.

## Review

If all the other books in this series essentially propose this challenge, "you (may) not know JS (as much as you thought)," this book has instead suggested, "you don't know JS anymore." The book has covered a ton of new stuff added to the language in ES6. It's an exciting collection of new language features and paradigms that will forever improve our JS programs.

But JS is not done with ES6! Not even close. There's already quite a few features in various stages of development for the "beyond ES6" timeframe. In this chapter, we briefly looked at some of the most likely candidates to land in JS very soon.

`async function`s are powerful syntactic sugar on top of the generators + promises pattern (see Chapter 4). `Object.observe(..)` adds direct native support for observing object change events, which is critical for implementing data binding. The `**` exponentiation operator, `...` for object properties, and `Array#includes(..)` are all simple but helpful improvements to existing mechanisms. Finally, SIMD ushers in a new era in the evolution of high performance JS.

Cliché as it sounds, the future of JS is really bright! The challenge of this series, and indeed of this book, is incumbent on every reader now. What are you waiting for? It's time to get learning and exploring!
