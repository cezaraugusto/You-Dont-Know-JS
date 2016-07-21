# You Don't Know JS: ES6 e além
# Chapter 4: Controle de fluxo assincrono

Se você já escreveu uma quantidade significativa de JavaScript, não é segredo que programação assincrona é uma habilidade necessária. Até o momento, o mecanismo principal para gerenciar assicronicidade tem sido o uso de funções *callback*.

No entanto, o ES6 adiciona um novo recurso que ajuda a resolver as deficiências significativas de assicronicidade com *callbacks*: *Promises*. Ainda, podemos revisar o capítulo anterior sobre `generators` e ver um padrão que combina os dois que é melhoria ao programar controles de fluxo assincrono no JavaScript.

## Promises

Vamos esclarecer alguns equivocos: *Promises* não substituem *callbacks*. *Promises* provêm uma intermediação confiável -- isso é, entre a execução do seu código e o código assincrono que processará a tarefa -- para gerenciar *callbacks*.

Outra forma de pensar a respeito de uma *promise* é como sendo um *event listener*, onde você registra um evento que te informa quanto uma tarefa foi concluída. É um evento que será disparado apenas uma vez, mas mesmo assim podemos imaginal-los como se fosse um evento.

*Promises* podem ser encadeadas em conjunto, as quais podem sequenciar uma série de passos que completam-se assincronamente. Juntas com uma abstração de alto-nivel como o método `all(..)` (em termo classico, uma porta) e o método `race(..)` (em um termo classico, um cadeado), o encadeamento de uma *promise* proporciona um mecanismo para controle de fluxo assincrono.

Ainda, mais uma forma de se entender uma *promise*, é que ela tem um valor futuro, um recipiente em torno de um valor que é independente do tempo envolvido. Esse recipiente pode ser subentendido de forma idêntica, independentemente se o seu valor é ou não o final, observando-se que a resolução de uma *promise* extrai este valor uma vez que ele estiver disponível. Em outras palavras, uma *promise* é dita como a versão assincrona de valores retornados por funções síncronas.

Uma *promise* pode possuir apenas uma, das duas possíves resoluções: concluída ou rejeitada, com um valor único opcional. Se uma *promise* for cumprida, o valor final é chamado de cumprimento. Se for rejeitada, o valor final é denominado de motivo (como sendo, o "motivo da rejeição"). *Promises* podem apenas ser resolvidas (cumpridas ou rejeitadas) uma única vez. Qualquer tentativa futura de cumpri-las ou rejeita-las são simplesmente ignoradas. Assim, uma vez que uma *promise* for resolvida, seu valor se torna imutável, e não pode ser alterado.

Claramente, há muitas maneiras difrentes de imaginar o que uma *promise* é. Nenhuma das perspectiva é completamente suficiente, mas cada uma provê um aspecto diferente do todo. Podemos concluir que *Promises* oferecem melhorias significantes a respeito da utilização de *callbacks* para código assincrono, visto que elas promovem ordem, previsibilidade e confiabilidade.

### Fazendo e Usando Promises

Para instanciar uma *promise*, use seu construtor `Promise(..)`:

```js
var p = new Promise( function pr(resolve,reject){
	// ..
} );
```

O construtor `Promise(..)` utiliza uma função (`pr(..)`), que é chamada imediatamente e recebe duas funções de controle como argumentos, normalmente chamada de `resolve(..)` e `reject(..)`. Elas são usadas assim:

* Se você invocar `reject(..)`, a *promise* é rejeitada, e se algum valor for passado para `reject(..)`, ele será considerado como o motivo da rejeição.
* Se você invocar `resolve(..)` sem passar um valor, ou algum valor que não seja uma *promise*, a *promise* será cumprida.
* Se você invocar `resolve(..)` e passar uma outra *promise*, a primeira *promise* simplesmente adota o estado -- sendo imediato ou eventual -- da segunda *promise* informada (sendo ela cumprida ou rejeitada).

Aqui está como você normalmente utiliza uma *promise* para refatorar uma chamada de função *callback*. Vamos iniciar com uma função utilitária `ajax(..)` que espera ser capaz de primeiramente tratar algum erro no *callback*:

```js
function ajax(url,cb) {
	// faz uma requisição, ao final invoca `cb(..)`
}

// ..

ajax( "http://some.url.1", function handler(err,contents){
	if (err) {
		// manipula o erro no retorno do ajax
	}
	else {
		// manipula o `conteúdo` no sucesso
	}
} );
```

Você pode converter para:

```js
function ajax(url) {
	return new Promise( function pr(resolve,reject){
		// faz a requisição, no final invoca
		// `resolve(..)` ou `reject(..)`
	} );
}

// ..

ajax( "http://some.url.1" )
.then(
	function fulfilled(contents){
		// manipula o `conteúdo` no sucesso
	},
	function rejected(reason){
		// manipula a razão do erro da requisição
	}
);
```

*Promises* possuiem um metodo chamado `then(..)` que aceita uma ou duas funções como *callback*. A primeira função (se presente) é tratata como o manipulador de uma *promise* cumprida com sucesso. A segunda função (se presente) é tratada como o manipulador se a *promise* for explicitamente rejeitada, ou se algum erro ou exeção ocorrer durante a resolução.

Se um dos argumentos for omitido ou se não for uma função válida -- normalmente você usará `null` em vez isso -- um valor padrão equivalente é usado. O *callback* padrão de sucesso passa seu valor de cumprimento adiante e a função *callback* padrão de erro propaga seu valor de rejeição adiante.

O atalho para chamar o método `then(null,handleRejection)` é `catch(handleRejection)`.

Ambos `then(..)` e `catch(..)` automaticamente constroem e retornam uma instancia de outra *promise*, que está preparada para receber a resolução de qualquer que seja o valor de retorno da *promise* original, cumprimento ou rejeição (seja como for realmente chamado). Considere:

```js
ajax( "http://some.url.1" )
.then(
	function fulfilled(contents){
		return contents.toUpperCase();
	},
	function rejected(reason){
		return "DEFAULT VALUE";
	}
)
.then( function fulfilled(data){
	// manipula data da promise original
} );
```

Nesse trexo de código, estamos retornando um valor apartir de qualquer um dos métodos `fulfilled(..)` ou `rejected(..)`, que então é recebido no próximo `fulfilled(..)` do segundo `then(..)`. Se ao invés disso retornarmos uma nova *promise*, essa nova *promise* é adotada como a resolução:

```js
ajax( "http://some.url.1" )
.then(
	function fulfilled(contents){
		return ajax(
			"http://some.url.2?v=" + contents
		);
	},
	function rejected(reason){
		return ajax(
			"http://backup.url.3?err=" + reason
		);
	}
)
.then( function fulfilled(contents){
	// `contents` vem de uma das chamadas subsequentes de `ajax(..)`
} );
```

É importante notar que uma exceção (ou uma *promise* rejeitada) no primeiro `fulfilled(..)` *não* resultará  na primeira execução do método `rejected(..)`, já que esse manipulador responde apenas a resolução da *promise* original. Ao invés disso, a segunda *promise*, cujo segundo `then(..)` é chamado de encontro, recebe a rejeição.

 No trecho de código anterior, não estamos ouvindo a rejeição, o que significa que ela será silenciosamente guardada para futuras observações. Se você não lidar com a rejeição utilizando `then(..)` ou `catch(..)`, ela não será processada. Os consoles de alguns navegadores podem detectar essas rejeições não processadas e reporta-las, mas isso não é garantido; você deveria sempre observar a rejeição de uma *promise*.

**Nota:** Isso foi apenas uma visão geral e breve da teoria e comportamento em respeito de uma *promise*. Para uma exploração detalhada, veje o Capitulo 3 do título dessa série *Async & Performance*.

### Thenables

*Promises* são instâncias do construtor `Promise(..)`. Contudo, há objetos *semelhantes à promise* chamados *thenables* que geralmente podem trabalhar em conjunto com mecanismos *Promise*

Qualquer objeto (ou função) com um método `then(..)` é tido como um *thenable*. Qualquer lugar onde mecanismos *Promise* podem aceitar e adotar o estado de uma *promise* genuína, ele pode também lidar com um *thenable*.

*Thenables* são basicamente rótulos genericos para qualquer valor *semelhante à promise* que pode ter sido criado por outro mecanismo diferente do atual construtor `Promise()`. Nessa perspectiva, um *thenable* é geralmente menos confiável que uma *Promise* genuína. Considere esse *thenable* inapropriado, por exemplo:

```js
var th = {
	then: function thener( fulfilled ) {
		// chama `fulfilled(..)` uma vez a cada 100ms eternamete.
		setInterval( fulfilled, 100 );
	}
};
```

If you received that thenable and chained it with `th.then(..)`, você provavelmente se surpreendeu pelo manipulador de cumprimento ter sido chamado repetidamente, enquanto em *Promises* originais espera-se que sejam resolvidas apenas uma única vez.

Geralmente, você não deve confirar cegamente se você está esperando receber o que se propõe a ser uma *promise* ou *thenable* de algum outro sistema. Na próxima seção, nós veremos uma vantagem incluída nas Promises do ES6 que ajuda a endereçar essas preocupações com a confiabilidade.

Mas para entender melhor os perigos desse assunto, considere que *qualquer* objeto em *qualquer* pedaço de código que foi definido para ter um método chamado `then(..)` pode ser pontencialmente confundido com um *thenable* -- se usado como *Promises*, claro -- mesmo se esse código nem remotamente tiver sido destinado a ser um código assincrono *estilo Promise*.

Antes do ES6, nunca houve qualquer método reservado chamado `then(..)`, e como você pode imaginar há ao menos alguns casos onde esse nome foi escolhido para um método antes das *Promises* terem entrado em cena. O equívoco mais provável de *thenable* seriam bibliotecas assíncronas que usam `then(..)` mas que não são estritamente compatíveis com Promises -- existem muitas por aí.

A responsabilidade será sua de se proteger de usar diretamente valores com mecanismo *Promise* que poderíam ser erradamente assumindos como *thenable*.

### *promise* API

The *promise* API also provides some static methods for working with Promises.

`Promise.resolve(..)` creates a promise resolved to the value passed in. Let's compare how it works to the more manual approach:

```js
var p1 = Promise.resolve( 42 );

var p2 = new Promise( function pr(resolve){
	resolve( 42 );
} );
```

`p1` and `p2` will have essentially identical behavior. The same goes for resolving with a promise:

```js
var theP = ajax( .. );

var p1 = Promise.resolve( theP );

var p2 = new Promise( function pr(resolve){
	resolve( theP );
} );
```

**Tip:** `Promise.resolve(..)` is the solution to the thenable trust issue raised in the previous section. Any value that you are not already certain is a trustable promise -- even if it could be an immediate value -- can be normalized by passing it to `Promise.resolve(..)`. If the value is already a recognizable promise or thenable, its state/resolution will simply be adopted, insulating you from misbehavior. If it's instead an immediate value, it will be "wrapped" in a genuine promise, thereby normalizing its behavior to be async.

`Promise.reject(..)` creates an immediately rejected promise, the same as its `Promise(..)` constructor counterpart:

```js
var p1 = Promise.reject( "Oops" );

var p2 = new Promise( function pr(resolve,reject){
	reject( "Oops" );
} );
```

While `resolve(..)` and `Promise.resolve(..)` can accept a promise and adopt its state/resolution, `reject(..)` and `Promise.reject(..)` do not differentiate what value they receive. So, if you reject with a promise or thenable, the promise/thenable itself will be set as the rejection reason, not its underlying value.

`Promise.all([ .. ])` accepts an array of one or more values (e.g., immediate values, promises, thenables). It returns a promise back that will be fulfilled if all the values fulfill, or reject immediately once the first of any of them rejects.

Starting with these values/promises:

```js
var p1 = Promise.resolve( 42 );
var p2 = new Promise( function pr(resolve){
	setTimeout( function(){
		resolve( 43 );
	}, 100 );
} );
var v3 = 44;
var p4 = new Promise( function pr(resolve,reject){
	setTimeout( function(){
		reject( "Oops" );
	}, 10 );
} );
```

Let's consider how `Promise.all([ .. ])` works with combinations of those values:

```js
Promise.all( [p1,p2,v3] )
.then( function fulfilled(vals){
	console.log( vals );			// [42,43,44]
} );

Promise.all( [p1,p2,v3,p4] )
.then(
	function fulfilled(vals){
		// never gets here
	},
	function rejected(reason){
		console.log( reason );		// Oops
	}
);
```

While `Promise.all([ .. ])` waits for all fulfillments (or the first rejection), `Promise.race([ .. ])` waits only for either the first fulfillment or rejection. Consider:

```js
// NOTE: re-setup all test values to
// avoid timing issues misleading you!

Promise.race( [p2,p1,v3] )
.then( function fulfilled(val){
	console.log( val );				// 42
} );

Promise.race( [p2,p4] )
.then(
	function fulfilled(val){
		// never gets here
	},
	function rejected(reason){
		console.log( reason );		// Oops
	}
);
```

**Warning:** While `Promise.all([])` will fulfill right away (with no values), `Promise.race([])` will hang forever. This is a strange inconsistency, and speaks to the suggestion that you should never use these methods with empty arrays.

## Generators + Promises

It *is* possible to express a series of promises in a chain to represent the async flow control of your program. Consider:

```js
step1()
.then(
	step2,
	step1Failed
)
.then(
	function step3(msg) {
		return Promise.all( [
			step3a( msg ),
			step3b( msg ),
			step3c( msg )
		] )
	}
)
.then(step4);
```

However, there's a much better option for expressing async flow control, and it will probably be much more preferable in terms of coding style than long promise chains. We can use what we learned in Chapter 3 about generators to express our async flow control.

The important pattern to recognize: a generator can yield a promise, and that promise can then be wired to resume the generator with its fulfillment value.

Consider the previous snippet's async flow control expressed with a generator:

```js
function *main() {

	try {
		var ret = yield step1();
	}
	catch (err) {
		ret = yield step1Failed( err );
	}

	ret = yield step2( ret );

	// step 3
	ret = yield Promise.all( [
		step3a( ret ),
		step3b( ret ),
		step3c( ret )
	] );

	yield step4( ret );
}
```

On the surface, this snippet may seem more verbose than the promise chain equivalent in the earlier snippet. However, it offers a much more attractive -- and more importantly, a more understandable and reason-able -- synchronous-looking coding style (with `=` assignment of "return" values, etc.) That's especially true in that `try..catch` error handling can be used across those hidden async boundaries.

Why are we using Promises with the generator? It's certainly possible to do async generator coding without Promises.

Promises are a trustable system that uninverts the inversion of control of normal callbacks or thunks (see the *Async & Performance* title of this series). So, combining the trustability of Promises and the synchronicity of code in generators effectively addresses all the major deficiencies of callbacks. Also, utilities like `Promise.all([ .. ])` are a nice, clean way to express concurrency at a generator's single `yield` step.

So how does this magic work? We're going to need a *runner* that can run our generator, receive a `yield`ed promise, and wire it up to resume the generator with either the fulfillment success value, or throw an error into the generator with the rejection reason.

Many async-capable utilities/libraries have such a "runner"; for example, `Q.spawn(..)` and my asynquence's `runner(..)` plug-in. But here's a stand-alone runner to illustrate how the process works:

```js
function run(gen) {
	var args = [].slice.call( arguments, 1), it;

	it = gen.apply( this, args );

	return Promise.resolve()
		.then( function handleNext(value){
			var next = it.next( value );

			return (function handleResult(next){
				if (next.done) {
					return next.value;
				}
				else {
					return Promise.resolve( next.value )
						.then(
							handleNext,
							function handleErr(err) {
								return Promise.resolve(
									it.throw( err )
								)
								.then( handleResult );
							}
						);
				}
			})( next );
		} );
}
```

**Note:** For a more prolifically commented version of this utility, see the *Async & Performance* title of this series. Also, the run utilities provided with various async libraries are often more powerful/capable than what we've shown here. For example, asynquence's `runner(..)` can handle `yield`ed promises, sequences, thunks, and immediate (non-promise) values, giving you ultimate flexibility.

So now running `*main()` as listed in the earlier snippet is as easy as:

```js
run( main )
.then(
	function fulfilled(){
		// `*main()` completed successfully
	},
	function rejected(reason){
		// Oops, something went wrong
	}
);
```

Essentially, anywhere that you have more than two asynchronous steps of flow control logic in your program, you can *and should* use a promise-yielding generator driven by a run utility to express the flow control in a synchronous fashion. This will make for much easier to understand and maintain code.

This yield-a-promise-resume-the-generator pattern is going to be so common and so powerful, the next version of JavaScript after ES6 is almost certainly going to introduce a new function type that will do it automatically without needing the run utility. We'll cover `async function`s (as they're expected to be called) in Chapter 8.

## Review

As JavaScript continues to mature and grow in its widespread adoption, asynchronous programming is more and more of a central concern. Callbacks are not fully sufficient for these tasks, and totally fall down the more sophisticated the need.

Thankfully, ES6 adds Promises to address one of the major shortcomings of callbacks: lack of trust in predictable behavior. Promises represent the future completion value from a potentially async task, normalizing behavior across sync and async boundaries.

But it's the combination of Promises with generators that fully realizes the benefits of rearranging our async flow control code to de-emphasize and abstract away that ugly callback soup (aka "hell").

Right now, we can manage these interactions with the aide of various async libraries' runners, but JavaScript is eventually going to support this interaction pattern with dedicated syntax alone!
