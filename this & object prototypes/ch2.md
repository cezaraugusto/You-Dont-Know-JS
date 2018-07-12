# You Don't Know JS: *this* & Prototipagem de Objetos
# Capítulo 2: `this` Agora tudo faz sentido!

No Capítulo 1, nós eliminamos diversos equívocos relacionados à `this` e aprendemos que `this` é um *binding* feito para cada invocação de função, baseado inteiramente no seu **call-site** (como a função é chamada).

## Call-site

Para entender o binding the `this`, precisamos entender o call-site: o lugar no código onde a função é chamada (**não onde ela é declarada**). Nós devemos inspecionar o call-site para responder a seguinte questão: a quem este `this` está fazendo referência?

Encontrar o call-site é geralmente: "ir até o local de onde a função é chamada", mas não é sempre tão fácil assim, já que alguns padrões de código podem obscurecer o *verdadeiro* call-site.

O que é importante é pensar sobre o **call-stack** (a pilha de funções que foram chamadas para nos deixar no momento atual na execução). O call-site que devemos nos importar está *dentro* da invocação *anterior* à função em execução atual.

Demonstremos o call-stack e o call-site:

```js
function baz() {
    // call-stack é: `baz`
    // sendo assim, nosso call-site está no escopo global

    console.log( "baz" );
    bar(); // <-- call-site para `bar`
}

function bar() {
    // call-stack é: `baz` -> `bar`
    // sendo assim, nosso call-site está em `baz`

    console.log( "bar" );
    foo(); // <-- call-site para `foo`
}

function foo() {
    // call-stack é: `baz` -> `bar` -> `foo`
    // sendo assim, nosso call-site está em `bar`

    console.log( "foo" );
}

baz(); // <-- call-site para `baz`
```

Seja cuidadoso ao analizar o código ao procurar pelo call-site atual (através do call-stack), visto que ele é a única coisa que importa para o binding the `this`.

**Nota:** Você pode visualizar o call-stack mentalmente ao ver a cadeia de funções em ordem, como fizemos nos comentários no trecho de código anterior. Entretanto, esta é uma forma dolorosa e passível a erros. Uma outra forma de de ver o call-stack é usar a ferramenta de debug do seu navegador. A maioria dos navegadores modernos tem ferramentas do desenvolvedor nativas, as quais ingluem um debugger de JS. No trecho de código anterior, você poderia ter definido um breakpoint na ferramenta para a primeira linha da função `foo()`, ou simplesmente inserido a instrução `debugger;` nessa primeira linha. Quando você rodar a página, o debugger irá pausar neste ponto, e irá mostrar à você a lista de funções que foram acionadas para poder chegar à esta linha, que virá a ser o seu call stack. Sendo assim, se você está tentando diagnosticar o binding de `this`, use as ferramentas do desenvolvedor para acessar o call-stack, e então busque o segundo item começando do topo, e ela irá mostrar à você o verdadeiro call-site.

## Nada além de Regras

Iremos direcionar nossa atenção agora para *como* o call-site determina onde o `this` irá apontar durante a execução de uma função.

Você precisa inspecionar o call-site e determinar onde as 4 regras se aplicam. Iremos primeiro explicar cada uma dessas 4 regras de maneira independente, e depois iremos ilustrar sua ordem de precedência, caso *pudermos* aplicar multiplas regraspara o call-site.

### Binding padrão

A primeira regra que iremos examinar vem do caso mais comum ao se chamar uma função: invocar uma função separada. Pense *nessa* regra de `this` como a regra padrão para todos os casos quando nenhuma outra regra puder ser aplicada. 

Considere esse código:

```js
function foo() {
  console.log( this.a );
}

var a = 2;

foo(); // 2
```

A primeira coisa a se notar, se você ainda não estiver notado, é que as variáveis declaradas no escopo global, como `var a = 2`, são sinônimos de propriedades de objetos globais com o mesmo nome. Elas não são cópias umas das outras, eles *são* as outras. Pense nisso como os dois lados da mesma moeda.

A segunda coisa a se notar, nós vemos que quando `foo()` é chamado, `this.a` se refere à nossa variável global `a`. Porque? Por que nesse caso, o *binding padrão* para `this`se aplica ao chamado da função, sendo assim ela aponta `this`para o objeto global.

Como podemos saber se a regra do *binding padrão* se aplica aqui? Nós examinaremos o call-site para ver como `foo()` é chamado. No nosso trecho de código, `foo()` é chamado como uma referência plana, sem nenhuma decoração. Nenhuma das outras regras que iremos demonstrar seriam aplicadas aqui, sendo assim o *binding padrão* é o que seria aplicado.

Se o `strict mode` estiver ativo, o objeto global não é elegível para o *binding padrão*, sendo `this` nesse caso sendo apresentado como `undefined`.

```js
function foo() {
  "use strict";

  console.log( this.a );
}

var a = 2;

foo(); // TypeError: `this` is `undefined`
```

Um detalhe sutil, mas importante, é que: mesmo que a regra geral do binding de `this` seja inteiramente baseada no call-site, o objeto global é eligível **apenas** para o *binding padrão* se o **conteúdo** de `foo()` não estiver rodando em `strict mode`; o estado `strict mode` do call-site de `foo()` é irrelevante.

```js
function foo() {
  console.log( this.a );
}

var a = 2;

(function(){
  "use strict";

  foo(); // 2
})();
```

**Nota:** Misturar código `strict mode` e código não-`strict mode` juntos não é uma boa ideia. Seu programa deveria ser inteiramente ou **Strict** ou **não-Strict**. Entretanto, às vezes você inclui uma biblioteca externa que tem um modo diferente do seu código, então esteja atento com esse sutil detalhe sobre compatibilidade.

### Binding Implícito

Outra regra à se considerar é: o call-site tem um objeto como contexto, também chamado de objeto proprietário ou que contêm, apesar *destes* termos alternativos serem levemente enganosos.

Considere:

```js
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2,
  foo: foo
};

obj.foo(); // 2
```

Primeiramente, note a maneira como `foo()` é declarado e depois adicionado como uma propriedade de referência em `obj`. Independentemente se `foo()` é inicialmente declarado *no* `obj`, ou se é adicionado como uma referência depois (como o snippet mostra), em nenhum dos casos essa **função** realmente é "possuida" ou está "contida" pelo objeto `obj`.

Entretanto, a call-site *usa* o contexto do `obj` para **referenciar** a função, então você *poderia* dizer que o objeto `obj` é "dono" ou "contém" a **função de referência** no momento que a função é chamada.

Qualquer que seja a forma que você chama esse padrão, no momento em que `foo()` é chamado, é precedido por uma referência de objeto à `obj`. Quando existe um objeto de contexto para uma função referenciada, a regra do *binding implítico* diz que deve ser *aquele* objeto que deve ser usado para o binding da função `this` chamada.

Pelo fato de que `obj` é o `this` para a chamada de `foo()`, `this.a` é sinônimo de `obj.a`.

Apenas o topo/último nível de uma cadeia de propriedade de um objeto referênciado é o que importa para o call-site. Por exemplo;

```js
function foo() {
  console.log( this.a );
}

var obj2 = {
  a: 42,
  foo: foo
};

var obj1 = {
  a: 2,
  obj2: obj2
};

obj1.obj2.foo(); // 42
```

#### Perda implícita

Uma das frustrações mais comuns que o binding do `this` cria é quando uma função *bindada implícitamente* perde seu binding, o que normalmente quer dizer que ela retorna ao seu *binding padrão*, que ou é do objeto global ou `undefined`, dependendo do `strict mode`.

Considere:

```js
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2,
  foo: foo
};

var bar = obj.foo; // referência para a função!

var a = "oops, global"; // `a` também é propriedade do objeto global

bar(); // "oops, global"
```

Apesar de `bar` aparentemente ser uma referência para `obj.foo`, na realidade, é apenas outra referência para o próprio `foo`. Além do mais, o call-site é o que importa, e o call-site é `bar()`, o que é uma chamada simples e não-decorada, e por isso o *binding padrão* se aplica.

A maneira mais sútil, mais comum, e mais inesperada em que isso acontece é quando passamos uma função de callback:

```js
function foo() {
  console.log( this.a );
}

function doFoo(fn) {
  // `fn` é apenas mais uma referência para `foo`

  fn(); // <-- call-site!
}

var obj = {
  a: 2,
  foo: foo
};

var a = "oops, global"; // `a` também é propriedade do objeto global

doFoo( obj.foo ); // "oops, global"
```

Passagem de parâmetros é apenas uma atribuição implicita, e já que estamos passando uma função, é uma atribuição por referência implícita, então o resultado final é o mesmo que no snippet anterior.

Mas e se a função que você estiver passando como callback não for sua, mas nativa da linguagem? Sem diferenças, o resultado é o mesmo.

```js
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2,
  foo: foo
};

var a = "oops, global"; // `a` também é propriedade do objeto global

setTimeout( obj.foo, 100 ); // "oops, global"
```

Pense sobre essa pseudo-implementação teórica e cru do `setTimeout()`, fornecido nativamente pelo ambiente JavaScript:

```js
function setTimeout(fn,delay) {
  // espera (de alguma forma) por `delay` milissegundos
  fn(); // <-- call-site!
}
```

É bem comum que a nossa função de callback *perca* seu binding ao `this`, como nós acabamos de ver. Mas outra maneira em que o `this` pode nós surpreender é quando a função que passamos nosso callback intencionalmente muda o `this` para a chamada. Event handlers em bibliotecas JavaScript populares são bem afeiçoados de forçar seu callback a ter um `this` que aponta para, por exemplo, o elemento da DOM que disparou o evento. Enquanto isso pode ser útil algumas vezes, outras vezes podem ser completamente enfurecedor. Infelizmente, essas ferramentas raramente deixam você escolher.

De qualquer forma o `this` é modificado inesperadamente, você não está realmente com o controle de como sua função de callback referenciada vai ser executada, então você não tem nenhuma forma (ainda) de controlar o call-site para dar a ele seu binding desejado. Veremos em breve, uma maneira de "consertar" essa problema *consertando* o `this`.

### Binding Explícito

Com o *binding implícito* como acabamos de ver, nós tivemos que alterar o objeto em questão para incluir a referência dele mesmo para a função, e usar essa referência da propriedade da função para indiretamente (implicitamente) fazer o bind do `this` para o objeto.

Mas, e se você quiser forçar uma chamada de função para usar um objeto específico para o binding `this`, sem colocar uma referência de propriedade de função no objeto?

"Todas" as funções da linguagem têm algumas utilidades disponíveis nelas (via `[[Prototype]]` -- mais disso adiante) que podem ser úteis para essa tarefa. Especificamente, funções têm os métodos `call(..)` e `apply(..)`. Tecnicamente, ambientes de host Javascript, por vezes, fornecem funções que são especiais o suficiente (uma maneira gentil de coloca-lo!) que elas não tem tal funcionalidade. Mas essas são poucas. A vasta maioria das funções fornecidas, e certamente todas as funções que você irá criar, têm acesso ao `call(..)` e `apply(..)`.

Como essas funcionalidade funcionam? Ambas tomam, como primeiro parâmetro, um objeto para usar o `this`, e então invocam a função com o `this` específicado. Já que você está clarando diretamente o que você quer que seja `this`, chamamos de *binding explícito*.

Considere:

```js
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2
};

foo.call( obj ); // 2
```

Invocando `foo` com *binding explícito* no `foo.call(..)` nos permite forçar o `this` para ser o `obj`.

Se você simplesmente passar uma valor primitivo (do tipo `string`, `boolean` ou `number`) como o binding `this`, o valor primitivo é encapsulado na sua forma de objeto (`new String(..)`, `new Boolean(..)`, ou `new Number(..)`, respectivamente). Isso é frequentemente referido como "boxing".

**Nota:** Com relação ao binding `this`, `call(..)` e `apply(..)` são idênticos. Eles *comportam-se* de maneira diferente com seus parâmetros adicionais, mas isso não é algo com que nos importamos atualmente.

Infelizmente, *binding explicito* sozinho continua não oferecendo nenhuma solução para o problema mencionado anteriormente, de uma função "perdendo" seu binding `this` pretendido, ou apenas ter ele passado por um framework, etc.

#### Hard Binding

But a variation pattern around *explicit binding* actually does the trick. Consider:

```js
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2
};

var bar = function() {
  foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// `bar` hard binds `foo`'s `this` to `obj`
// so that it cannot be overriden
bar.call( window ); // 2
```

Let's examine how this variation works. We create a function `bar()` which, internally, manually calls `foo.call(obj)`, thereby forcibly invoking `foo` with `obj` binding for `this`. No matter how you later invoke the function `bar`, it will always manually invoke `foo` with `obj`. This binding is both explicit and strong, so we call it *hard binding*.

The most typical way to wrap a function with a *hard binding* creates a pass-thru of any arguments passed and any return value received:

```js
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}

var obj = {
  a: 2
};

var bar = function() {
  return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

Another way to express this pattern is to create a re-usable helper:

```js
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}

// simple `bind` helper
function bind(fn, obj) {
  return function() {
    return fn.apply( obj, arguments );
  };
}

var obj = {
  a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

Since *hard binding* is such a common pattern, it's provided with a built-in utility as of ES5: `Function.prototype.bind`, and it's used like this:

```js
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}

var obj = {
  a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

`bind(..)` returns a new function that is hard-coded to call the original function with the `this` context set as you specified.

**Note:** As of ES6, the hard-bound function produced by `bind(..)` has a `.name` property that derives from the original *target function*. For example: `bar = foo.bind(..)` should have a `bar.name` value of `"bound foo"`, which is the function call name that should show up in a stack trace.

#### API Call "Contexts"

Many libraries' functions, and indeed many new built-in functions in the JavaScript language and host environment, provide an optional parameter, usually called "context", which is designed as a work-around for you not having to use `bind(..)` to ensure your callback function uses a particular `this`.

For instance:

```js
function foo(el) {
  console.log( el, this.id );
}

var obj = {
  id: "awesome"
};

// use `obj` as `this` for `foo(..)` calls
[1, 2, 3].forEach( foo, obj ); // 1 awesome  2 awesome  3 awesome
```

Internally, these various functions almost certainly use *explicit binding* via `call(..)` or `apply(..)`, saving you the trouble.

### `new` Binding

The fourth and final rule for `this` binding requires us to re-think a very common misconception about functions and objects in JavaScript.

In traditional class-oriented languages, "constructors" are special methods attached to classes, that when the class is instantiated with a `new` operator, the constructor of that class is called. This usually looks something like:

```js
something = new MyClass(..);
```

JavaScript has a `new` operator, and the code pattern to use it looks basically identical to what we see in those class-oriented languages; most developers assume that JavaScript's mechanism is doing something similar. However, there really is *no connection* to class-oriented functionality implied by `new` usage in JS.

First, let's re-define what a "constructor" in JavaScript is. In JS, constructors are **just functions** that happen to be called with the `new` operator in front of them. They are not attached to classes, nor are they instantiating a class. They are not even special types of functions. They're just regular functions that are, in essence, hijacked by the use of `new` in their invocation.

For example, the `Number(..)` function acting as a constructor, quoting from the ES5.1 spec:

> 15.7.2 The Number Constructor
>
> When Number is called as part of a new expression it is a constructor: it initialises the newly created object.

So, pretty much any ol' function, including the built-in object functions like `Number(..)` (see Chapter 3) can be called with `new` in front of it, and that makes that function call a *constructor call*. This is an important but subtle distinction: there's really no such thing as "constructor functions", but rather construction calls *of* functions.

When a function is invoked with `new` in front of it, otherwise known as a constructor call, the following things are done automatically:

1. a brand new object is created (aka, constructed) out of thin air
2. *the newly constructed object is `[[Prototype]]`-linked*
3. the newly constructed object is set as the `this` binding for that function call
4. unless the function returns its own alternate **object**, the `new`-invoked function call will *automatically* return the newly constructed object.

Steps 1, 3, and 4 apply to our current discussion. We'll skip over step 2 for now and come back to it in Chapter 5.

Consider this code:

```js
function foo(a) {
  this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

By calling `foo(..)` with `new` in front of it, we've constructed a new object and set that new object as the `this` for the call of `foo(..)`. **So `new` is the final way that a function call's `this` can be bound.** We'll call this *new binding*.

## Everything In Order

So, now we've uncovered the 4 rules for binding `this` in function calls. *All* you need to do is find the call-site and inspect it to see which rule applies. But, what if the call-site has multiple eligible rules? There must be an order of precedence to these rules, and so we will next demonstrate what order to apply the rules.

It should be clear that the *default binding* is the lowest priority rule of the 4. So we'll just set that one aside.

Which is more precedent, *implicit binding* or *explicit binding*? Let's test it:

```js
function foo() {
  console.log( this.a );
}

var obj1 = {
  a: 2,
  foo: foo
};

var obj2 = {
  a: 3,
  foo: foo
};

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```

So, *explicit binding* takes precedence over *implicit binding*, which means you should ask **first** if *explicit binding* applies before checking for *implicit binding*.

Now, we just need to figure out where *new binding* fits in the precedence.

```js
function foo(something) {
  this.a = something;
}

var obj1 = {
  foo: foo
};

var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```

OK, *new binding* is more precedent than *implicit binding*. But do you think *new binding* is more or less precedent than *explicit binding*?

**Note:** `new` and `call`/`apply` cannot be used together, so `new foo.call(obj1)` is not allowed, to test *new binding* directly against *explicit binding*. But we can still use a *hard binding* to test the precedence of the two rules.

Before we explore that in a code listing, think back to how *hard binding* physically works, which is that `Function.prototype.bind(..)` creates a new wrapper function that is hard-coded to ignore its own `this` binding (whatever it may be), and use a manual one we provide.

By that reasoning, it would seem obvious to assume that *hard binding* (which is a form of *explicit binding*) is more precedent than *new binding*, and thus cannot be overridden with `new`.

Let's check:

```js
function foo(something) {
  this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

Whoa! `bar` is hard-bound against `obj1`, but `new bar(3)` did **not** change `obj1.a` to be `3` as we would have expected. Instead, the *hard bound* (to `obj1`) call to `bar(..)` ***is*** able to be overridden with `new`. Since `new` was applied, we got the newly created object back, which we named `baz`, and we see in fact that  `baz.a` has the value `3`.

This should be surprising if you go back to our "fake" bind helper:

```js
function bind(fn, obj) {
  return function() {
    fn.apply( obj, arguments );
  };
}
```

If you reason about how the helper's code works, it does not have a way for a `new` operator call to override the hard-binding to `obj` as we just observed.

But the built-in `Function.prototype.bind(..)` as of ES5 is more sophisticated, quite a bit so in fact. Here is the (slightly reformatted) polyfill provided by the MDN page for `bind(..)`:

```js
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== "function") {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError( "Function.prototype.bind - what " +
        "is trying to be bound is not callable"
      );
    }

    var aArgs = Array.prototype.slice.call( arguments, 1 ),
      fToBind = this,
      fNOP = function(){},
      fBound = function(){
        return fToBind.apply(
          (
            this instanceof fNOP &&
            oThis ? this : oThis
          ),
          aArgs.concat( Array.prototype.slice.call( arguments ) )
        );
      }
    ;

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

**Note:** The `bind(..)` polyfill shown above differs from the built-in `bind(..)` in ES5 with respect to hard-bound functions that will be used with `new` (see below for why that's useful). Because the polyfill cannot create a function without a `.prototype` as the built-in utility does, there's some nuanced indirection to approximate the same behavior. Tread carefully if you plan to use `new` with a hard-bound function and you rely on this polyfill.

The part that's allowing `new` overriding is:

```js
this instanceof fNOP &&
oThis ? this : oThis

// ... and:

fNOP.prototype = this.prototype;
fBound.prototype = new fNOP();
```

We won't actually dive into explaining how this trickery works (it's complicated and beyond our scope here), but essentially the utility determines whether or not the hard-bound function has been called with `new` (resulting in a newly constructed object being its `this`), and if so, it uses *that* newly created `this` rather than the previously specified *hard binding* for `this`.

Why is `new` being able to override *hard binding* useful?

The primary reason for this behavior is to create a function (that can be used with `new` for constructing objects) that essentially ignores the `this` *hard binding* but which presets some or all of the function's arguments. One of the capabilities of `bind(..)` is that any arguments passed after the first `this` binding argument are defaulted as standard arguments to the underlying function (technically called "partial application", which is a subset of "currying").

For example:

```js
function foo(p1,p2) {
  this.val = p1 + p2;
}

// using `null` here because we don't care about
// the `this` hard-binding in this scenario, and
// it will be overridden by the `new` call anyway!
var bar = foo.bind( null, "p1" );

var baz = new bar( "p2" );

baz.val; // p1p2
```

### Determining `this`

Now, we can summarize the rules for determining `this` from a function call's call-site, in their order of precedence. Ask these questions in this order, and stop when the first rule applies.

1. Is the function called with `new` (**new binding**)? If so, `this` is the newly constructed object.

    `var bar = new foo()`

2. Is the function called with `call` or `apply` (**explicit binding**), even hidden inside a `bind` *hard binding*? If so, `this` is the explicitly specified object.

    `var bar = foo.call( obj2 )`

3. Is the function called with a context (**implicit binding**), otherwise known as an owning or containing object? If so, `this` is *that* context object.

    `var bar = obj1.foo()`

4. Otherwise, default the `this` (**default binding**). If in `strict mode`, pick `undefined`, otherwise pick the `global` object.

    `var bar = foo()`

That's it. That's *all it takes* to understand the rules of `this` binding for normal function calls. Well... almost.

## Binding Exceptions

As usual, there are some *exceptions* to the "rules".

The `this`-binding behavior can in some scenarios be surprising, where you intended a different binding but you end up with binding behavior from the *default binding* rule (see previous).

### Ignored `this`

If you pass `null` or `undefined` as a `this` binding parameter to `call`, `apply`, or `bind`, those values are effectively ignored, and instead the *default binding* rule applies to the invocation.

```js
function foo() {
  console.log( this.a );
}

var a = 2;

foo.call( null ); // 2
```

Why would you intentionally pass something like `null` for a `this` binding?

It's quite common to use `apply(..)` for spreading out arrays of values as parameters to a function call. Similarly, `bind(..)` can curry parameters (pre-set values), which can be very helpful.

```js
function foo(a,b) {
  console.log( "a:" + a + ", b:" + b );
}

// spreading out array as parameters
foo.apply( null, [2, 3] ); // a:2, b:3

// currying with `bind(..)`
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

Both these utilities require a `this` binding for the first parameter. If the functions in question don't care about `this`, you need a placeholder value, and `null` might seem like a reasonable choice as shown in this snippet.

**Note:** We don't cover it in this book, but ES6 has the `...` spread operator which will let you syntactically "spread out" an array as parameters without needing `apply(..)`, such as `foo(...[1,2])`, which amounts to `foo(1,2)` -- syntactically avoiding a `this` binding if it's unnecessary. Unfortunately, there's no ES6 syntactic substitute for currying, so the `this` parameter of the `bind(..)` call still needs attention.

However, there's a slight hidden "danger" in always using `null` when you don't care about the `this` binding. If you ever use that against a function call (for instance, a third-party library function that you don't control), and that function *does* make a `this` reference, the *default binding* rule means it might inadvertently reference (or worse, mutate!) the `global` object (`window` in the browser).

Obviously, such a pitfall can lead to a variety of *very difficult* to diagnose/track-down bugs.

#### Safer `this`

Perhaps a somewhat "safer" practice is to pass a specifically set up object for `this` which is guaranteed not to be an object that can create problematic side effects in your program. Borrowing terminology from networking (and the military), we can create a "DMZ" (de-militarized zone) object -- nothing more special than a completely empty, non-delegated (see Chapters 5 and 6) object.

If we always pass a DMZ object for ignored `this` bindings we don't think we need to care about, we're sure any hidden/unexpected usage of `this` will be restricted to the empty object, which insulates our program's `global` object from side-effects.

Since this object is totally empty, I personally like to give it the variable name `ø` (the lowercase mathematical symbol for the empty set). On many keyboards (like US-layout on Mac), this symbol is easily typed with `⌥`+`o` (option+`o`). Some systems also let you set up hotkeys for specific symbols. If you don't like the `ø` symbol, or your keyboard doesn't make that as easy to type, you can of course call it whatever you want.

Whatever you call it, the easiest way to set it up as **totally empty** is `Object.create(null)` (see Chapter 5). `Object.create(null)` is similar to `{ }`, but without the delegation to `Object.prototype`, so it's "more empty" than just `{ }`.

```js
function foo(a,b) {
  console.log( "a:" + a + ", b:" + b );
}

// our DMZ empty object
var ø = Object.create( null );

// spreading out array as parameters
foo.apply( ø, [2, 3] ); // a:2, b:3

// currying with `bind(..)`
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

Not only functionally "safer", there's a sort of stylistic benefit to `ø`, in that it semantically conveys "I want the `this` to be empty" a little more clearly than `null` might. But again, name your DMZ object whatever you prefer.

### Indirection

Another thing to be aware of is you can (intentionally or not!) create "indirect references" to functions, and in those cases,  when that function reference is invoked, the *default binding* rule also applies.

One of the most common ways that *indirect references* occur is from an assignment:

```js
function foo() {
  console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
(p.foo = o.foo)(); // 2
```

The *result value* of the assignment expression `p.foo = o.foo` is a reference to just the underlying function object. As such, the effective call-site is just `foo()`, not `p.foo()` or `o.foo()` as you might expect. Per the rules above, the *default binding* rule applies.

Reminder: regardless of how you get to a function invocation using the *default binding* rule, the `strict mode` status of the **contents** of the invoked function making the `this` reference -- not the function call-site -- determines the *default binding* value: either the `global` object if in non-`strict mode` or `undefined` if in `strict mode`.

### Softening Binding

We saw earlier that *hard binding* was one strategy for preventing a function call falling back to the *default binding* rule inadvertently, by forcing it to be bound to a specific `this` (unless you use `new` to override it!). The problem is, *hard-binding* greatly reduces the flexibility of a function, preventing manual `this` override with either the *implicit binding* or even subsequent *explicit binding* attempts.

It would be nice if there was a way to provide a different default for *default binding* (not `global` or `undefined`), while still leaving the function able to be manually `this` bound via *implicit binding* or *explicit binding* techniques.

We can construct a so-called *soft binding* utility which emulates our desired behavior.

```js
if (!Function.prototype.softBind) {
  Function.prototype.softBind = function(obj) {
    var fn = this,
      curried = [].slice.call( arguments, 1 ),
      bound = function bound() {
        return fn.apply(
          (!this ||
            (typeof window !== "undefined" &&
              this === window) ||
            (typeof global !== "undefined" &&
              this === global)
          ) ? obj : this,
          curried.concat.apply( curried, arguments )
        );
      };
    bound.prototype = Object.create( fn.prototype );
    return bound;
  };
}
```

The `softBind(..)` utility provided here works similarly to the built-in ES5 `bind(..)` utility, except with our *soft binding* behavior. It wraps the specified function in logic that checks the `this` at call-time and if it's `global` or `undefined`, uses a pre-specified alternate *default* (`obj`). Otherwise the `this` is left untouched. It also provides optional currying (see the `bind(..)` discussion earlier).

Let's demonstrate its usage:

```js
function foo() {
   console.log("name: " + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );

fooOBJ(); // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2   <---- look!!!

fooOBJ.call( obj3 ); // name: obj3   <---- look!

setTimeout( obj2.foo, 10 ); // name: obj   <---- falls back to soft-binding
```

The soft-bound version of the `foo()` function can be manually `this`-bound to `obj2` or `obj3` as shown, but it falls back to `obj` if the *default binding* would otherwise apply.

## Lexical `this`

Normal functions abide by the 4 rules we just covered. But ES6 introduces a special kind of function that does not use these rules: arrow-function.

Arrow-functions are signified not by the `function` keyword, but by the `=>` so called "fat arrow" operator. Instead of using the four standard `this` rules, arrow-functions adopt the `this` binding from the enclosing (function or global) scope.

Let's illustrate arrow-function lexical scope:

```js
function foo() {
  // return an arrow function
  return (a) => {
    // `this` here is lexically adopted from `foo()`
    console.log( this.a );
  };
}

var obj1 = {
  a: 2
};

var obj2 = {
  a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, not 3!
```

The arrow-function created in `foo()` lexically captures whatever `foo()`s `this` is at its call-time. Since `foo()` was `this`-bound to `obj1`, `bar` (a reference to the returned arrow-function) will also be `this`-bound to `obj1`. The lexical binding of an arrow-function cannot be overridden (even with `new`!).

The most common use-case will likely be in the use of callbacks, such as event handlers or timers:

```js
function foo() {
  setTimeout(() => {
    // `this` here is lexically adopted from `foo()`
    console.log( this.a );
  },100);
}

var obj = {
  a: 2
};

foo.call( obj ); // 2
```

While arrow-functions provide an alternative to using `bind(..)` on a function to ensure its `this`, which can seem attractive, it's important to note that they essentially are disabling the traditional `this` mechanism in favor of more widely-understood lexical scoping. Pre-ES6, we already have a fairly common pattern for doing so, which is basically almost indistinguishable from the spirit of ES6 arrow-functions:

```js
function foo() {
  var self = this; // lexical capture of `this`
  setTimeout( function(){
    console.log( self.a );
  }, 100 );
}

var obj = {
  a: 2
};

foo.call( obj ); // 2
```

While `self = this` and arrow-functions both seem like good "solutions" to not wanting to use `bind(..)`, they are essentially fleeing from `this` instead of understanding and embracing it.

If you find yourself writing `this`-style code, but most or all the time, you defeat the `this` mechanism with lexical `self = this` or arrow-function "tricks", perhaps you should either:

1. Use only lexical scope and forget the false pretense of `this`-style code.

2. Embrace `this`-style mechanisms completely, including using `bind(..)` where necessary, and try to avoid `self = this` and arrow-function "lexical this" tricks.

A program can effectively use both styles of code (lexical and `this`), but inside of the same function, and indeed for the same sorts of look-ups, mixing the two mechanisms is usually asking for harder-to-maintain code, and probably working too hard to be clever.

## Revisão (TL;DR)

Determinar o vínculo (bind) the `this` para uma função em execução requer a 
Determining the `this` binding for an executing function requires finding the direct call-site of that function. Once examined, four rules can be applied to the call-site, in *this* order of precedence:

1. Called with `new`? Use the newly constructed object.

2. Called with `call` or `apply` (or `bind`)? Use the specified object.

3. Called with a context object owning the call? Use that context object.

4. Default: `undefined` in `strict mode`, global object otherwise.

Be careful of accidental/unintentional invoking of the *default binding* rule. In cases where you want to "safely" ignore a `this` binding, a "DMZ" object like `ø = Object.create(null)` is a good placeholder value that protects the `global` object from unintended side-effects.

Instead of the four standard binding rules, ES6 arrow-functions use lexical scoping for `this` binding, which means they adopt the `this` binding (whatever it is) from its enclosing function call. They are essentially a syntactic replacement of `self = this` in pre-ES6 coding.
