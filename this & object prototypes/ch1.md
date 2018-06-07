# You Don't Know JS: *this* & Prototipagem de Objetos
# Capítulo 1: `this` Ou Aquilo?

Um dos mecanismos mais confusos no JavaScript é a palavra-chave `this`. É uma palavra-chave de identificação especial que é definida automaticamente no escopo de cada função, mas o que exatamente ela se refere atormenta até mesmo os desenvolvedores JavaScript mais experientes.

> Qualquer tecnologia bastante *avançada* é indistinguível da magia. -- Arthur C. Clarke

O mecanismo `this` do JavaScript na verdade não é *tão* avançado, mas os desenvolvedores muitas vezes utilizam essa citação em suas mentes inserindo "complexa" ou "confusa", e não há dúvidas que, sem uma clara compreensão, `this` pode parecer totalmente magia em *sua* confunsão.

**Nota:** A palavra "this" (isto) é um pronome terrivelmente comum em diálogos de forma geral (em inglês). Então, pode ser muito difícil, especialmente oralmente, determinar se estamos usando "isto" como um pronome, ou usando para realmente referir à palavra-chave de identificação. Para deixar claro, sempre vou utilizar `this` para referi à palavra-chave, e "isto" ou *isto* ou isto caso contrário.

## Por que `this`?

Se o mecanismo `this` é tão confuso, até mesmo para desenvolvedores JavaScript experientes, pode-se perguntar: por que ainda é útil? É um problema ou vale a pena? Antes de entrarmos no *como*, devemos examinar o *porquê*.

Vamos tentar ilustrar a motivação e utilidade do `this`:

```js
function identify() {
	return this.name.toUpperCase();
}

function speak() {
	var greeting = "Hello, I'm " + identify.call( this );
	console.log( greeting );
}

var me = {
	name: "Kyle"
};

var you = {
	name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER

speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER
```

Se o *como* desse snippet confunde você, não se preocupe! Já vamos chegar lá. Apenas coloque isso de lado por um momento para que possamos olhar mais claramente o *porquê*.

Esse pedaço de código permite que as funções `identify()` e `speak()` sejam reutilizadas em diversos objetos (`me` and `you`) de *contexto*, em vez de precisar de uma versão separada da função para cada objeto.

Ao invés de confiar no `this`, você poderia passar explicitamente um objeto de contexto, tanto para `identify()`, quanto para `speak()`.

```js
function identify(context) {
	return context.name.toUpperCase();
}

function speak(context) {
	var greeting = "Hello, I'm " + identify( context );
	console.log( greeting );
}

identify( you ); // READER
speak( me ); // Hello, I'm KYLE
```

No entanto, o mecanismo `this` provê um modo mais elegante de "passar" um objeto como referência implicitamente, levando a um design de API mais limpo e fácil de ser reutilizado.

Quanto mais complexo for o padrão de utilização, você verá mais claramente que passar o contexto como um parâmetro explícito é frequentemente mais confuso do que passar o contexto com o `this`. Quando exploramos objetos e protótipos, você verá a utilidade de ter uma coleção de funções capazes de referir automaticamente a um objeto no contexto apropriado.

## Confusões

Vamos começar a explicar como `this` *realmente* funciona em breve, mas primeiro nós devemos dissipar alguns equívocos sobre como ele *não* funciona.

O nome "this" gera confusão quando os desenvolvedores tentam pensar nele de forma literal. Existem dois significados muitas vezes adotados, mas ambos estão incorretos.

### Ele mesmo

A tentação mais comum é assumir que `this` se refere a própria função. Essa é uma conclusão gramatical aceitável, pelo menos.

Por que você gostaria de referir a uma função de dentro dela mesma? As razões mais comuns poderiam ser coisas como recursão (chamar uma função de dentro dela mesma) ou tendo um manipulador de evento que pode se desvincular quando for chamado pela primeira vez.

Desenvolvedores novatos nos mecanismos do JS muitas vezes pensam que referenciar uma função como um objeto (todas as funções em JavaScript são objetos!) permite armazenar *estado* (valores em propriedades) entre as chamadas da função. Embora isso certamente seja possível e com algumas limitações de uso, o restante do livro irá expor muitos outros padrões com *melhores* lugares para se armazenar estado fora do objeto da função.

Mas por um momento, nós vamos explorar esse padrão, para ilustrar como `this` não deixa uma função pegar uma referência a ela mesma como podemos ter assumido.

Considere o seguinte código, onde nós tentaremos contar quantas vezes a função (`foo`) foi chamada:

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 0 -- WTF?
```

`foo.count` *ainda* é `0`, mesmo que as quatro declarações `console.log` claramente indicam que `foo(..)` foi, de fato, chamada quatro vezes. A frustração se origina de uma interpretação *muito literal* do que o `this` (em `this.count++`) significa.

Quando o código executa `foo.count = 0`, realmente está adicionando a propriedade `count` ao objeto da função `foo`. Mas para a referência `this.count` de dentro da função, `this` não está de fato apontando *de todo* para aquele objeto da função, e então, mesmo que os nomes das propriedades sejam os mesmos, os objetos-raiz são diferentes, e resultam em confusão.

**Nota:** Uma desenvolvedora responsável *deve* perguntar nesse momento, "Se eu estava incrementando uma propriedade `count` mas não era a que eu esperava, que `count` eu *estava* incrementando?" De fato, ela está se aprofundando, ela descobrirá que acidentalmente criou uma variável global `count` (veja o Capítulo 2 para *como* isso aconteceu!), e atualmente ela tem o valor `NaN`. Naturalmente, uma vez que ela descobre esse resultado peculiar, ela então tem um novo conjunto de perguntas: "Como essa variável é global e por que ela acabou como `NaN` ao invés de algum valor adequado?" (veja o Capítulo 2).

Ao invés de parar e analisar com profundidade o porquê da referência `this` parecer não se comportar da maneira *esperada*, e responder essas questões difíceis mas importantes, muitos desenvolvedores simplesmentes evitam esse problema completamente, e escrevem alguma outra solução, como criar um outro objeto para armazenar a propriedade `count`:

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	data.count++;
}

var data = {
	count: 0
};

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( data.count ); // 4
```

Embora seja verdade que essa abordagem "resolve" o problema, infelizmente ela simplesmente ignora o problema real -- a falta de entendimento do que o `this` significa e como ele funciona -- e, ao invés disso, retrocede para a zona de conforto de um mecanismo mais familiar: o escopo léxico.

**Nota:** Escopo léxico é um mecanismo muito bom e útil; Eu não estou menosprezando o uso dele, de forma alguma (veja o título *"Escopo & Closures"* desta série de livros). Mas constantemente *adivinhar* como usar o `this`, e geralmente estar *errado*, não é uma boa razão para se voltar para o escopo léxico e nunca aprender *porquê* o `this` ilude você.

Para referenciar o objeto da função de dentro dela mesma, `this` por si só será em geral insuficiente. Normalmente você precisa de uma referência para o objeto da função por um identificador léxico (variável) que aponta para ela.

Considere essas duas funções:

```js
function foo() {
	foo.count = 4; // `foo` refers to itself
}

setTimeout( function(){
	// anonymous function (no name), cannot
	// refer to itself
}, 10 );
```

Na primeira função, chamada de "função nomeada", `foo` é uma referência que pode ser usada para se referir a função de dentro dela mesma.

Mas no segundo exemplo, a função de callback passada para o `setTimeout(..)` não tem identificador (então chamada de "função anônima"), então não há uma maneira adequada de se referenciar o próprio objeto da função.

**Nota:** A referência old-school, mas agora deprecada e de torcer o nariz, `arguments.callee` dentro uma função *também* aponta para o objeto da função que está sendo executado. Essa referência é normalmente a única forma de acessa um objeto de função anônima de dentro dela mesma. A melhor abordagem no entanto, é evitar o uso de funções anônimas completamente, pelo menos para as que necessitam de auto-referência, e em vez disso utilizar funções nomeadas (expressões). `arguments.callee` é obsoleta e não deve ser usada.

Outra solução para nosso atual exemplo seria usar o identificador `foo` como referência ao objeto da função em cada lugar, e não utilizar o `this` absolutamente, o que *funciona*:

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	foo.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

No entanto, essa abordagem igualmente o *real* entendimento do `this` e depende inteiramente do escopo léxico da variável `foo`.

Ainda outro modo de abordar esse problema é forçar o `this` para realmente apontar para o objeto da função `foo`:

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	// Note: `this` IS actually `foo` now, based on
	// how `foo` is called (see below)
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		// using `call(..)`, we ensure the `this`
		// points at the function object (`foo`) itself
		foo.call( foo, i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

**Ao invés de evitar o `this`, nós abraçamos ele.** Vamos explicar daqui a pouco o *como* essas técnicas funcionam muito mais completamente, então não se preocupe se você ainda estiver um pouco confuso!

### Its Scope

The next most common misconception about the meaning of `this` is that it somehow refers to the function's scope. It's a tricky question, because in one sense there is some truth, but in the other sense, it's quite misguided.

To be clear, `this` does not, in any way, refer to a function's **lexical scope**. It is true that internally, scope is kind of like an object with properties for each of the available identifiers. But the scope "object" is not accessible to JavaScript code. It's an inner part of the *Engine*'s implementation.

Consider code which attempts (and fails!) to cross over the boundary and use `this` to implicitly refer to a function's lexical scope:

```js
function foo() {
	var a = 2;
	this.bar();
}

function bar() {
	console.log( this.a );
}

foo(); //undefined
```

There's more than one mistake in this snippet. While it may seem contrived, the code you see is a distillation of actual real-world code that has been exchanged in public community help forums. It's a wonderful (if not sad) illustration of just how misguided `this` assumptions can be.

Firstly, an attempt is made to reference the `bar()` function via `this.bar()`. It is almost certainly an *accident* that it works, but we'll explain the *how* of that shortly. The most natural way to have invoked `bar()` would have been to omit the leading `this.` and just make a lexical reference to the identifier.

However, the developer who writes such code is attempting to use `this` to create a bridge between the lexical scopes of `foo()` and `bar()`, so that `bar()` has access to the variable `a` in the inner scope of `foo()`. **No such bridge is possible.** You cannot use a `this` reference to look something up in a lexical scope. It is not possible.

Every time you feel yourself trying to mix lexical scope look-ups with `this`, remind yourself: *there is no bridge*.

## What's `this`?

Having set aside various incorrect assumptions, let us now turn our attention to how the `this` mechanism really works.

We said earlier that `this` is not an author-time binding but a runtime binding. It is contextual based on the conditions of the function's invocation. `this` binding has nothing to do with where a function is declared, but has instead everything to do with the manner in which the function is called.

When a function is invoked, an activation record, otherwise known as an execution context, is created. This record contains information about where the function was called from (the call-stack), *how* the function was invoked, what parameters were passed, etc. One of the properties of this record is the `this` reference which will be used for the duration of that function's execution.

In the next chapter, we will learn to find a function's **call-site** to determine how its execution will bind `this`.

## Review (TL;DR)

`this` binding is a constant source of confusion for the JavaScript developer who does not take the time to learn how the mechanism actually works. Guesses, trial-and-error, and blind copy-n-paste from Stack Overflow answers is not an effective or proper way to leverage *this* important `this` mechanism.

To learn `this`, you first have to learn what `this` is *not*, despite any assumptions or misconceptions that may lead you down those paths. `this` is neither a reference to the function itself, nor is it a reference to the function's *lexical* scope.

`this` is actually a binding that is made when a function is invoked, and *what* it references is determined entirely by the call-site where the function is called.
