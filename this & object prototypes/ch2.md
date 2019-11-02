# You Don't Know JS: *this* & Prototipagem de Objetos
# Capítulo 2: `this` Agora tudo faz sentido!

No Capítulo 1, nós eliminamos diversos equívocos relacionados à `this` e aprendemos que `this` é um *binding* feito para cada invocação de função, baseado inteiramente no seu **call-site** (como a função é chamada).

## Call-site

Para entender o binding do `this`, precisamos entender o **call-site**: o lugar no código onde a função é chamada (**não onde ela é declarada**). Nós devemos inspecionar o call-site para responder a seguinte questão: a quem este `this` está fazendo referência?

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

Seja cuidadoso ao analizar o código ao procurar pelo call-site atual (através do call-stack), visto que ele é a única coisa que importa para o binding do `this`.

**Nota:** Você pode visualizar o call-stack mentalmente ao ver a cadeia de funções em ordem, como fizemos nos comentários no trecho de código anterior. Entretanto, esta é uma forma dolorosa e passível a erros. Uma outra forma de ver o call-stack é usar a ferramenta de debug do seu navegador. A maioria dos navegadores modernos tem ferramentas do desenvolvedor nativas, as quais incluem um debugger de JS. No trecho de código anterior, você poderia ter definido um breakpoint na ferramenta para a primeira linha da função `foo()`, ou simplesmente inserido a instrução `debugger;` nessa primeira linha. Quando você rodar a página, o debugger irá pausar neste ponto, e irá mostrar à você a lista de funções que foram acionadas para poder chegar à esta linha, que virá a ser o seu call stack. Sendo assim, se você está tentando diagnosticar o binding de `this`, use as ferramentas do desenvolvedor para acessar o call-stack, e então busque o segundo item começando do topo, e ela irá mostrar à você o verdadeiro call-site.

## Nada além de Regras

Iremos direcionar nossa atenção agora para *como* o call-site determina onde o `this` irá apontar durante a execução de uma função.

Você precisa inspecionar o call-site e determinar onde as 4 regras se aplicam. Iremos primeiro explicar cada uma dessas 4 regras de maneira independente, e depois iremos ilustrar sua ordem de precedência, caso *possamos* aplicar multiplas regras para o call-site.

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

A segunda coisa a se notar, nós vemos que quando `foo()` é chamado, `this.a` se refere à nossa variável global `a`. Por quê? Porque nesse caso, o *binding padrão* para `this`se aplica ao chamado da função, sendo assim ela aponta `this`para o objeto global.

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

Pense sobre essa pseudo-implementação teórica e crua do `setTimeout()`, fornecido nativamente pelo ambiente JavaScript:

```js
function setTimeout(fn,delay) {
  // espera (de alguma forma) por `delay` milissegundos
  fn(); // <-- call-site!
}
```

É bem comum que a nossa função de callback *perca* seu binding ao `this`, como nós acabamos de ver. Mas outra maneira em que o `this` pode nos surpreender é quando a função que passamos nosso callback intencionalmente muda o `this` para a chamada. Event handlers em bibliotecas JavaScript populares são bem afeiçoados de forçar seu callback a ter um `this` que aponta para, por exemplo, o elemento da DOM que disparou o evento. Enquanto isso pode ser útil algumas vezes, outras vezes podem ser completamente enfurecedor. Infelizmente, essas ferramentas raramente deixam você escolher.

De qualquer forma o `this` é modificado inesperadamente, você não está realmente com o controle de como sua função de callback referenciada vai ser executada, então você não tem nenhuma forma (ainda) de controlar o call-site para dar a ele seu binding desejado. Veremos em breve, uma maneira de "consertar" essa problema *consertando* o `this`.

### Binding Explícito

Com o *binding implícito* como acabamos de ver, nós tivemos que alterar o objeto em questão para incluir a referência dele mesmo para a função, e usar essa referência da propriedade da função para indiretamente (implicitamente) fazer o bind do `this` para o objeto.

Mas, e se você quiser forçar uma chamada de função para usar um objeto específico para o binding `this`, sem colocar uma referência de propriedade de função no objeto?

"Todas" as funções da linguagem têm algumas utilidades disponíveis nelas (via `[[Prototype]]` -- mais disso adiante) que podem ser úteis para essa tarefa. Especificamente, funções têm os métodos `call(..)` e `apply(..)`. Tecnicamente, ambientes de host Javascript, por vezes, fornecem funções que são especiais o suficiente (uma maneira gentil de colocá-lo!) que elas não tem tal funcionalidade. Mas essas são poucas. A vasta maioria das funções fornecidas, e certamente todas as funções que você irá criar, têm acesso ao `call(..)` e `apply(..)`.

Como essas ferramentas funcionam? Ambas tomam, como primeiro parâmetro, um objeto para usar o `this`, e então invocam a função com o `this` específicado. Já que você está claramente indicando o que você quer que seja `this`, chamamos de *binding explícito*.

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

Mas um padrão diferente sobre *binding explícito* realmente faz o truque. Considere:

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

// `bar` aplica hard bind no `this` de `foo` para `obj`
// então isso não pode ser substituído
bar.call( window ); // 2
```

Vamos agora examinar como essa variação funciona. Nós criamos uma função `bar()` que, internamente, chama manualmente `foo.call(obj)`, invocando de forma forçada `foo` com o binding `obj` para `this`. Não importa quão tarde você invoque a função `bar`, ela vai manualmente invocar `foo` com `obj`. Esse binding é explícito e forte, então o chamamos de *hard binding*.

O modo mais comum de encapsular uma função com um *hard binding* é criar uma via de quaisquer argumentos passados ​​e qualquer valor de retorno recebido:

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

Outra forma de representar esse padrão é criar um helper reutilizável:

```js
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}

// `bind` helper simples
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

Já que *hard binding* é um padrão bem comum, é fornecido como uma utilidade nativa do ES5: `Function.prototype.bind`, e é usada assim:

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

`bind(..)` retorna uma nova função que é escrita para chamar a função original com o contexto do `this` definido como você especificou.

**Nota:** No ES6, a função que faz o hard binding produzida por `bind(..)` tem uma propriedade `.name` que deriva da *função alvo* original. Por exemplo: `bar = foo.bind(..)` deveria ter um valor `bar.name` de `"bound foo"` , que é o nome da chamada da função que deve aparecer em um rastreamento de pilha.

#### "Contextos" de chamadas de API

Muitas funções de bibliotecas, e de fato muitas funções nativas na linguagem JavaScript e em seus ambientes, fornecem um parâmetro opcional, geralmente chamado "contexto", que é projetado como uma solução para você não ter que usar o `bind(..)` para garantir que seu callback use um `this` em particular.

Por exemplo:

```js
function foo(el) {
  console.log( el, this.id );
}

var obj = {
  id: "awesome"
};

// use `obj` como `this` para chamadas `foo(..)`
[1, 2, 3].forEach( foo, obj ); // 1 awesome  2 awesome  3 awesome
```

Internamente, essas funções certamente usam *ligação explícita* via `call (..)` ou `apply (..)`, poupando você do problema.

### `new` Binding

A quarta e última regra para binding do `this` requer que nós repensemos um equívoco muito comum sobre funções e objetos no JavaScript.

Em linguagens tradicionais orientadas por classes, "construtores" (constructors) são métodos especiais anexados nas classes, que quando a classe é instanciada com um operador `new`, o construtor dessa classe é chamado. Isso geralmente se parece com algo assim:

```js
something = new MyClass(..);
```

JavaScript tem um operador `new` e o padrão de código para utilizá-lo é basicamente o mesmo que vemos naquelas linguagens orientadas à classes; a maioria dos desenvolvedores assumem que o mecanismo JavaScript está fazendo algo similar. Porém, realmente não há *nenhuma relação* com funcionalidades orientadas à classes no uso do `new` em JS.

Primeiro, vamos redefinir o que é um "construtor" em JavaScript. Em JS, construtores são **apenas funções** que são chamadas quando o operador `new` está na frente delas. Elas não são vinculadas à classes, nem estão instanciando-as. Elas não são nem tipos especiais de funções. Elas são apenas funções normais que são, em essência, sequestradas pelo uso de `new` quando invocadas.

Por exemplo, a função `Number(..)` atua como um construtor, citação da especificação ES5.1:

> 15.7.2 O Construtor Number
>
> Quando Number é chamado como parte de uma expressão ele é um construtor: ele inicializa o novos objetos criados.

Então, praticamente qualquer função, incluindo funções de objetos nativos como `Number(..)` (Veja o capítulo 3) podem ser chamadas com `new` à frente, e isso faz dessa chamada uma *chamada de construtor*. Essa é uma importante mas sútil diferença: não existem "funções construtoras" mas sim *chamadas construtoras* de funções.

Quando uma função é invocada com `new` à sua frente, também conhecida como chamada de construtor, as seguintes coisas são feitas automaticamente:

1. um objeto novo em folha é criado (construído)
2. *o objeto recém construído é linkado ao `[[Prototype]]`*
3. o objeto recém construído é definido como bind do `this` para aquela chamada de função
4. a menos que a função retorne seu próprio **objeto** alternado, a chamada da função invocada `new` vai retornar o objeto recém construído *automaticamente*.

Os passos 1, 3 e 4 aplicam-se à nossa discussão atual. Nós vamos pular o passo 2 por agora e voltar nele no Capítulo 5.

Considere esse código:

```js
function foo(a) {
  this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

Ao chamar `foo(..)` com `new` a sua frente, nós construímos um novo objeto e definimos esse novo objeto como `this` para a chamada de `foo(..)`. **Então `new` é a última maneira que uma chamada de função `this` pode sofrer o bind.** Vamos chamar isso de *new binding* .

## Tudo em Ordem

Então, agora descobrimos as 4 regras para ligação do `this` em chamadas de função. *Tudo* que você precisa fazer é encontrar o local de chamada e inspecioná-lo para ver qual regra se aplica. Mas, e se o local de chamada tiver várias regras elegíveis? Deve haver uma ordem de precedência para essas regras e, assim, demonstraremos em seguida que ordem aplicar à elas.

Deve ficar claro que o *default binding* (binding padrão) é a regra de prioridade mais baixa das 4. Então, vamos deixar isso de lado.

Qual tem maior precedência, *binding implícito* ou *binding explícito*? Vamos testar:

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

Portanto, *binding explícito* tem precedência sobre *binding implícito*, o que significa que você deve perguntar **primeiro** se *o binding explícito* é aplicado antes de verificar pelo *binding implícito*.

Agora, só precisamos descobrir em qual precedência o *new binding* se encaixa.

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

OK, o *new binding* tem maior precedência que o *binding implícito*. Mas você acha que o *new binding* tem maior ou menor precedêcia sobre o *binding explícito*?

**Nota** `new` e `call`/`apply` não podem ser usados juntos, então `new foo.call(obj1)` não é permitido, para testar o *new binding* diretamente contra o *binding explícito*. Mas nós ainda podemos usar o *hard binding* para testar a precedência entre as duas regras.

Antes de explorarmos isso em uma listagem de código, pense em como o *hard binding* funciona fisicamente, `Function.prototype.bind(..)` cria uma nova função de encapsulamento que é codificada para ignorar seu próprio binding `this` (qualquer que seja), e usar um que fornecemos manualmente.

Por esse raciocínio, parece óbvio assumir que *hard binding* (que é uma forma de *binding explícito*) tem maior precedência que *new binding*, e assim não pode ser substituído por `new`.

Vamos checar:

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

Epa! `bar` tem hard binding contra `obj1`, mas `new bar(3)` **não** mudou o `obj1.a` para ser `3` como havíamos esperado. Em vez disso, a chamada *hard binding* (para `obj1`) **é** passível de ser substituída com `new`. Desde que `new` foi aplicada, nós temos de volta o objeto recém criado, que nomeamos como `baz`, e nós vemos de fato que `baz.a` tem o valor `3`.

Isso pode ser surpreendente se você voltar para nosso bind helper "fake":

```js
function bind(fn, obj) {
  return function() {
    fn.apply( obj, arguments );
  };
}
```

Se você pensar sobre como o código do helper funciona, não há uma maneira da chamada do operador `new` substituir o hard binding para `obj` como nós observamos.

Mas a função nativa `Function.prototype.bind(..)` a partir do ES5 é mais sofisticada, na verdade é mais ou menos. Aqui está o polyfill (ligeriamente reformulado) fornecido pela página do MDN para `bind(..)`.

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

**Nota** o polyfill de `bind(..)` mostrado acima difere do `bind(..)` nativo no ES5 com respeito à funções com hard binding que serão usadas com `new` (veja abaixo porque isso é útil). Porque o polyfill não pode criar uma função sem um `prototype` como as funcionalidades nativas fazem, há uma aproximação suavemente indireta para o mesmo comportamento. Tenha cuidado se você planeja usar `new` com uma função hard binding e você confia nesse polyfill.

A parte que está permitindo a substituição do `new` é:

```js
this instanceof fNOP &&
oThis ? this : oThis

// ... e:

fNOP.prototype = this.prototype;
fBound.prototype = new fNOP();
```

Nós não vamos realmente explicar como esse truque funciona (é complicado e além do nosso escopo aqui), mas essencialmente a utilidade determina se a função hard binding foi chamada com `new` (resultando em um objeto recém construído sendo `this`), e se assim for, ele usa *esse* recém criado `this` ao invés do *hard binding* para o `this`.

Porque a capacidade do `new` de substituir o hard binding é útil?

A principal razão para este comportamento é criar uma função (que pode ser usada com `new` para construir objetos) que essencialmente ignora o `this` com *hard binding*, mas que pré configura alguns ou todos os argumentos da função. Uma das capacidades do `bind (..)` é que quaisquer argumentos passados ​​após o primeiro argumento de binding do `this` são padronizados como argumentos padrão para a função subjacente (tecnicamente chamado de "aplicação parcial", que é um subconjunto de "currying" -- Vide [WORDREFERENCE](https://github.com/cezaraugusto/You-Dont-Know-JS/blob/portuguese-translation/WORDREFERENCE.md#c)).

Por exemplo:

```js
function foo(p1,p2) {
  this.val = p1 + p2;
}

// usando o `null` aqui porque não nos importamos
// com o `this` hard-binding nesse cenário, e isso
// será substituído pela chamada `new` de qualquer forma!
var bar = foo.bind( null, "p1" );

var baz = new bar( "p2" );

baz.val; // p1p2
```

### Determinando o `this`

Agora, podemos resumir as regras para determinar o `this` a partir do local de chamada de uma função, na sua ordem de precedência. Faça estas perguntas nesta ordem e pare quando a primeira regra se aplicar.

1. A função está sendo chamada com `new` (**new binding**)? Se sim, `this` é o objeto recém construído.

    `var bar = new foo()`

2. A função é chamada com `call` ou `apply` (**explicit binding**), mesmo oculta dentro de um `bind` *hard binding*? Se sim, `this` é o objeto especificado explicitamente.

    `var bar = foo.call( obj2 )`

3. A função é chamada com um contexto (**implicit binding**), também conhecido como objeto proprietário ou contido? Se for assim, `this` é *aquele* objeto de contexto.

    `var bar = obj1.foo()`

4. Caso contrário, o padrão é `this` (**default binding**). Se em `strict mode`, escolha `undefined`, caso contrário escolha o objeto `global`.

    `var bar = foo()`

É isso aí. Isso é *tudo o que é preciso* para entender as regras de binding do `this` para chamadas normais de funções. Bem... quase.

## Exceções do Binding

Como de costume, há algumas *exceções* às "regras".

O comportamento do binding de `this` pode, em alguns cenários, ser surpreendente, onde você pretendia um binding diferente, mas acaba tendo um comportamento de binding da regra de *default binding* (veja anteriormente).

### Ignorando o `this`

Se você passar `null` ou `undefined` como um parâmetro de binding do `this` para `call`, `apply` ou `bind`, esses valores serão de fato ignorados e, em vez disso, a regra *default binding* será aplicada à invocação.

```js
function foo() {
  console.log( this.a );
}

var a = 2;

foo.call( null ); // 2
```

Por que você iria intencionalmente passar algo como `null` para um binding `this`?

É muito comum usar `apply(..)` para espalhar arrays de valores como parâmetros para uma chamada de função. Da mesma forma, `bind(..)` pode arranjar parâmetros (valores pré-definidos), o que pode ser muito útil.

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

Ambas das utilidades requerem um binding `this` para o primeiro parâmetro. Se as funções em questão não se preocuparem com `this`, você precisará de um valor substituto, e `null` pode parecer uma escolha razoável, conforme mostrado neste trecho.

**Nota:** Não abordamos neste livro, mas o ES6 tem o operador `...` spread que permite sintaticamente "espalhar" um array como parâmetros sem precisar de `apply(..)`, tal como `foo(... [1,2])`, que equivale a `foo(1,2)` - sintaticamente evitando um binding 'this' se for desnecessário. Infelizmente, não há nenhum substituto sintático do ES6 para curry, então o parâmetro `this` da chamada `bind(..)` ainda precisa de atenção.

Entretanto, existe um pequeno "perigo" oculto em sempre usar `null` quando você não se importa com o binding de `this`. Se você sempre usa isso em uma chamada de função (por exemplo, uma função de biblioteca de terceiros que você não controla), e essa função *faz* uma referência a this, a regra *default binding* significa que pode acidentalmente fazer referência (ou pior, mudar!) o objeto `global` (`window` no navegador).

Obviamente, essa armadilha pode levar a uma variedade de bugs *muito difícil* de diagnosticar/rastrear .

#### `this` Seguro

Talvez uma prática mais "segura" seja passar um objeto para `this` que garanta que este não seja um objeto que possa criar efeitos colaterais em seu programa. Pegando emprestado a terminologia da área de redes (e da militar), nós podemos criar um objeto "DMZ" (de-militarized zone / zona desmilitarizada) -- nada mais especial que um objeto completamente vazio, não delegado (veja no Capítulo 5).

Se nós sempre passarmos um objeto DMZ para bindings `this` ignorados, nós achamos que não precisamos nos preocupar, temos certeza que qualquer uso oculto/inesperado de `this` estará restrito ao objeto vazio, o que isola o objeto `global` do nosso programa de efeitos colaterais.

Já que esse objeto é totalmente vazio, eu pessoalmente gosto de dar um nome de variável `ø` à ele (o símbolo matemático para um conjunto vazio). Em muitos teclados (como o US-layout no Mac), esse símbolo é facilmente digitado com `⌥`+`o` (option+`o`). Alguns sistemas também deixam você definir teclas de atalho para símbolos específicos. Se você não gostar do símbolo `ø`, ou se seu teclado não o torna fácil de digitá-lo, claro que você pode chamá-lo como quiser.

Seja do que for que você o chame, a melhor forma de configurá-lo como **totalmente vazio** é com `Object.create(null)` (veja o Capítulo 5). `Object.create(null)` é similar à `{ }`, mas sem a delegação do `Object.prototype`, então ele é ainda "mais vazio" do que apenas `{ }`.

```js
function foo(a,b) {
  console.log( "a:" + a + ", b:" + b );
}

// nosso objeto vazio DMZ
var ø = Object.create( null );

// espalhando arrays como parâmetros
foo.apply( ø, [2, 3] ); // a:2, b:3

// currying com `bind(..)`
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

Essa funcionalidade não é apenas "mais segura", há vários benefícios estéticos para `ø` na medida que se transmite semanticamente "eu quero que o `this` seja vazio" mais claramente do que o `null` poderia. Mas, de novo, nomeie seu objeto DMZ da forma que você preferir.

### Referências Indiretas

Outra coisa para ter cuidado é que você pode (intencionalmente ou não!) criar "referências indiretas" para funções, e nesses casos, quando a referência dessa função é invocada, a regra de *default binding* também se aplica.

Uma das formas mais comuns pelas quais *referências indiretas* ocorrem é de uma atribuição:

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

O *valor do resultado* da expressão de atribuição `p.foo = o.foo` é uma referência apenas ao objeto de função adjacente. Como tal, o local efetivo das chamadas é `foo()`, não `p.foo()` ou `o.foo()` como você pode ter esperado. De acordo com as regras acima, a regra de *default binding* se aplica.

Lembrete: independentemente de como você chega à uma invocação de função usando a regra de *default binding*, o status `strict mode` do **conteúdo** da função invocada fazendo a referência `this` -- não ao local de chamada da função -- determinar o valor do *default binding*: o objeto `global` se estiver em modo não `strict` ou `undefined` se estiver em `strict mode`.

### Soft Binding

Nós vimos anteriormente que *hard binding* era uma das estratégias para prevenir que uma chamada de função caia na regra de *default binding* acidentalmente, forçando-a a ter o binding em um `this` específico (a menos que você use o `new` para substituí-lo!). O problema é, o *hard binding* diminui muito a flexibilidade de uma função, prevenindo a substituição manual do `this` tanto com tentativas de *binding implícito* ou até subsequentemente com o *binding explícito*.

Seria legal se houvesse uma forma de fornecer um padrão diferente para *default binding* (não `global` ou `undefined`), enquanto continuamos deixando a função capaz de fazer o binding do `this` manualmente pelas técnicas de *binding implícito* ou *binding explícito*.

Nós podemos construir a utilidade denominada *soft binding* que emula nosso comportamento desejado.

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

A utilidade `softBind(..)` fornecida aqui funciona de forma parecida com a utilidade `bind(..)` nativa do ES5, exceto com nosso comportamento de *soft binding*. Ela encapsula a função especificada na lógica que verifica o `this` no momento da chamada e se ele for `global` ou `undefined`, usa uma alternativa *default* de (`obj`)  pré-especificada. Caso contrário o `this` fica intocado. Ele também fornece currying opcional (veja a discussão `bind (..)` antes).

Vamos demonstrar seu uso:

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
obj2.foo(); // name: obj2   <---- olha!!!

fooOBJ.call( obj3 ); // name: obj3   <---- olha!

setTimeout( obj2.foo, 10 ); // name: obj   <---- volta para o soft binding
```

A versão da função `foo()` que sofreu o soft binding pode fazer o bind em `this` manualmente para `obj2` ou `obj3`, como demonstrado, mas cai de volta para `obj` se o *default binding* se aplicar.

## `this` Léxico

Funções normais suportam as 4 regras que acabamos de abordar. Mas o ES6 introduz um tipo especial de função que não usa nenhuma dessas regras: arrow-function.

Arrow-functions são identificadas não pela palavra-chave `function`, mas pelo operador `=>` então chamado de "fat arrow". Em vez de usar as quatro regras padrão para o `this`, as arrow-functions adotam o binding do `this` para o escopo ao redor (da função ou global).

Vamos ilustrar o escopo léxico da arrow function:

```js
function foo() {
	// retorna uma arrow function
	return (a) => {
		// `this` aqui é adotado léxicamente de `foo()`
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
bar.call( obj2 ); // 2, não 3!
```

A arrow function criada em `foo()` lexicamente captura tudo o que `this` de `foo()` tem em seu tempo de chamada. Já que `foo()` teve binding de `this` para` obj1`, `bar` (uma referência à arrow function retornada) também terá o binding `this` para `obj1`. O binding léxico de uma arrow function não pode ser sobrescrito (mesmo com `new`!).

O caso de uso mais comum provavelmente é o uso de callbacks, como os handlers de evento ou timers:

```js
function foo() {
	setTimeout(() => {
		// `this` aqui é adotado léxicamente de `foo()`
		console.log( this.a );
	},100);
}

var obj = {
  a: 2
};

foo.call( obj ); // 2
```

Enquanto arrow-functions fornecem uma alternativa ao uso de `bind(..)` em uma função para assegurar seu `this`, que pode parecer atraente, é importante notar que elas estão essencialmente desativando o tradicional mecanismo `this` em favor de mais escopo léxico amplamente compreendido. Antes do ES6, já temos um padrão bastante comum para isso, que é basicamente quase indistinguível da essência das arrow-functions do ES6:

```js
function foo() {
	var self = this; // captura léxica de `this`
	setTimeout( function(){
		console.log( self.a );
	}, 100 );
}

var obj = {
  a: 2
};

foo.call( obj ); // 2
```

Embora `self = this` e arrow-functions pareçam ser boas "soluções" para não querer usar `bind(...)`, elas estão essencialmente fugindo de `this` ao invés de entendê-lo e abraçá-lo.

Se você se encontrar escrevendo o código com `this`, mas na maior parte ou em todo o tempo, você derrota o mecanismo `this` com o léxico `self = this` ou truques com arrow-functions, talvez você deveria:

1. Usar somente o escopo léxico e esqueçer a falsa pretensão do código com `this`.

2. Adotar os mecanismos de `this` completamente, incluindo o uso de `bind(..)` onde for necessário, e tentar evitar `self = this` e truques de arrow-functions com "this léxico".

Um programa pode efetivamente usar ambos os estilos de código (léxico e `this`), mas dentro da mesma função, e na verdade para os mesmos tipos de abordagens, misturar os dois mecanismos geralmente é ter um código mais difícil de manter, e provavelmente trabalhando muito para ser inteligente.

## Revisão (TL;DR)

Determinar a ligação `this` para uma função em execução requer que se encontre o local de chamada direto dessa função. Uma vez examinada, quatro regras podem ser aplicadas ao local de chamada, *nesta* ordem de precedência:

1. Chamada com `new`? Use o objeto recém construído.

2. Chamada com `call` ou` apply` (ou `bind`)? Use o objeto especificado.

3. Chamada com um objeto do contexto que possui a chamada? Use esse objeto de contexto.

4. Padrão: `undefined` em `strict mode`, do contrário, objeto global.

Cuidado com a invocação acidental/involuntária da regra *default binding*. Nos casos em que você deseja "com segurança" ignorar o binding de `this`, um objeto "DMZ" como `ø = Object.create(null)` é um bom placeholder que protege o objeto `global` de efeitos colaterais indesejados.

Em vez das quatro regras de binding padrão, as arrow functions do ES6 usam o escopo léxico para o binding de `this`, o que significa que adotam binding `this` (o que quer que seja) de sua chamada de função delimitadora. Eles são essencialmente uma substituição sintática de `self = this` na codificação pré-ES6.
