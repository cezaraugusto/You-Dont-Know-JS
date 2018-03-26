# You Don't Know JS: Types & Grammar

# Capítulo 2: Valores

`array`s, `strings`s e `number`s são os componentes básicos de qualquer programa, mas em JavaScript esses tipos tem algumas características únicas que podem frustar ou animar você.

Vamos examinar vários tipos de valores nativos em JS e explorar como podemos entendê-los de forma completa e usar seus comportamentos corretamente.

## Arrays

Em comparação com outras linguagens fortemente tipadas, os `arary`s em JavaScript são apenas caixas para qualquer tipo de valor, como `string`, `number`, `object` e até outros `array` (neste caso, tem-se `array`s multidimensionais).

```javascript
var a = [ 1, "2", [3] ];

a.length;       // 3
a[0] === 1;     // true
a[2][0] === 3;  // true
```

Você não precisa definir o tamanho dos `array`s antecipadamente (veja "Arrays", no capítulo 3), você pode simplesmente declará-los e adicionar valores quando necessário:

```javascript
var a = [ ];

a.length;    // 0

a[0] = 1;
a[1] = "2";
a[2] = [ 3 ];

a.length;    // 3
```

**Aviso:** Usar `delete` em um elemento de um `array` irá remover o espaço do `array`, mas mesmo se você remover o último elemento, a propriedade `length` **não** será atualizada, então tenha cuidado! Iremos entender o operador `delete` em mais detalhes no capítulo 5.

Tenha cuidado ao criar `array`s "esparsados" (deixando ou criando espaços vazios/indefinidos):


```javascript
var a = [ ];

a[0] = 1;
// o espaço `a[1]` não foi definido
a[2] = [ 3 ];

a[1];        // undefined

a.length;    // 3
```

Apesar de funcionar, "espaços vazios" podem levar a alguns comportamentos confusos. Apesar dos espaços parecerem ter o valor `undefined`, eles não irão se comportar da mesma forma como se o espaço tivesse sido explicitamente definido (`a[1] = undefined`). Veja "Arrays" no capítulo 3 para mais informações.

`array`s são indexidados numericamente (como esperado), mas o complicado é que eles também são objetos que podem ter chaves/propriedades `string` adicionadas a eles (que não contam para a propriedade `length` de `array`):

```javascript
var a = [ ];

a[0] = 1;
a["foobar"] = 2;

a.length;        // 1
a["foobar"];    // 2
a.foobar;        // 2
```

Entretanto, uma pegadinha para estar alerta é que se um valor `string` que pode ser convertido para um `number` de base-10 é usado como chave, então assume-se que você quis usar um index `number`, ao invés de uma chave `string`!

```javascript
var a = [ ];

a["13"] = 42;

a.length; // 14
```

Geralmente, não é uma boa ideia adicionar chaves/propriedades `string` a `array`s. Use `object`s para guardar valores em chaves/propriedades e use `array`s apenas com valores de índice numéricos.

### Estruturas Semelhantes a Arrays

Haverá situações onde você precisará convertar estruturas semelhantes a arrays (uma coleção numericamente indexada de valores) em um array propriamente dito, normalmente para que você possa chamar funções de array (como `indexOf(..)`, `concat(..)`, `forEach(..)`, etc.) nesta coleção de valores.

Por exemplo, várias operações de query no DOM retornam listas de elementos DOM que não são verdadeiros `array`s, mas são semelhantes a `array`s o suficiente para a nossa proposta de conversão. Outro exemplo comum é quando funções expõem o objeto `arguments` (estrutura equivalente a `array`s, desencorajada na ES6) para acesso aos argumentos como uma lista.

Uma forma comum de fazer tal conversão é usar a função `slice(..)` em um valor:


```javascript
function foo() {
    var arr = Array.prototype.slice.call( arguments );
    arr.push( "bam" );
    console.log( arr );
}

foo( "bar", "baz" ); // ["bar","baz","bam"]
```

Se `slice()` é chamada sem nenhum parâmetro, como no exemplo acima, os valores padrão de seus parâmetros tem o efeito de duplicar o `array` (neste caso, uma estrutura semelhante a arrays).

A partir da ES6, existe uma função padrão chamada `Array.from(..)` que pode fazer o mesmo efeito:


```javascript
...
var arr = Array.from( arguments );
...
```

**Nota:** `Array.from(..)` possui alguns recursos poderosos, os quais iremos cobrir em detalhes em no livro _ES6 e Além_ desta série.

## Strings

É muito comum acreditar que uma `string` é essencialmente apenas um `array` de caracteres. Embora implementação por baixo dos panos poder usar ou não `array`s, é importante notar que `string`s, em JavaScript, não são a mesma coisa que `array`s de caracteres. Sua similaridade é apenas superficial.

Por exemplo, considere os dois valores abaixo:

```javascript
var a = "foo";
var b = ["f","o","o"];
```

Strings tem uma semelhança superficial com `array` -- estruturas semelhantes a `array`s, como dito acima -- por exemplo, ambos tem uma propriedade `length` (*tamanho*), um método `indexOf(..)` (`array`s apenas a partir da versão ES5) e um método `concat(..)` (*concatenar*):

```javascript
a.length;                            // 3
b.length;                            // 3

a.indexOf( "o" );                    // 1
b.indexOf( "o" );                    // 1

var c = a.concat( "bar" );            // "foobar"
var d = b.concat( ["b","a","r"] );    // ["f","o","o","b","a","r"]

a === c;                            // false
b === d;                            // false

a;                                    // "foo"
b;                                    // ["f","o","o"]
```

Então, ambos são basicamente apenas "arrays de caracateres", certo? **Não exatamente**:

```javascript
a[1] = "O";
b[1] = "O";

a; // "foo"
b; // ["f","O","o"]
```

`String`s em JavaScript são imutáveis, enquanto `arrays` são bastante mutáveis. Além do mais, a forma de acesso da posição de caractere `a[1]` nem sempre foi amplamente aceita em JavaScript. Antigas versões do IE não permitiam esta sintaxe (atualmente, elas aceitam). Em vez disso, a forma _correta_ tem sido `a.chartAt(1)`.

Outra consequência da imutabilidade de `string`s é que nenhum método que altera seu conteúdo pode ser feito localmente, mas sim gera-se uma nova `string`, que é retornada. Em contraste, muitos método que mudam o conteúdo de `array`s modificam localmente.

```javascript
c = a.toUpperCase();
a === c;      // false
a;            // "foo"
c;            // "FOO"

b.push( "!" );
b;            // ["f","O","o","!"]

```

Além disso, muitos dos métodos de `array` que poderiam ser úteis para lidar com `string`s, na verdade não estão disponíveis para elas, mas nós podemos "pegar emprestados" métodos não-mutáveis de `array`s para nossas `string`:

```javascript
a.join;       // undefined
a.map;        // undefined

var c = Array.prototype.join.call( a, "-" );
var d = Array.prototype.map.call( a, function(v){
    return v.toUpperCase() + ".";
} ).join( "" );

c;                // "f-o-o"
d;                // "F.O.O."
```

Vamos olhar em outro exemplo: Invertendo uma `string` (a propósito, uma pergunta comum em entrevistas de JavaScript). `array`s tem um método modificador local `reverse()`, mas `string`s não tem:

```javascript
a.reverse;        // undefined

b.reverse();      // ["!","o","O","f"]
b;                // ["!","o","O","f"]
```

Infelizmente, esse "empréstimo" não funciona com método mutáveis, pois `string`s são imutáveis e, por tanto, não podem ser modificadas localmente:

```javascript
Array.prototype.reverse.call( a );
// ainda retorna um objeto wrapper de String (ver capítulo 3)
// for "foo" :(
```

Outra alternativa (ou seja, um hack) é converter uma `string` em um `array`, realizar a operação desejada, e converter de volta em uma `string`.

```javascript
var c = a
    // transforma `a` em um array de caracteres
    .split( "" )
    // inverte o array de caracteres
    .reverse()
    // transforma o array de caracteres em uma string
    .join( "" );

c; // "oof"
```

Se isso parece feio, é porque é mesmo. Entretanto, _isso funciona_ para `string`s simples, então, se você precisar de algo rápido e sujo, em geral, essa abordagem servirá.

**Aviso:** Tenha cuidado! Essa abordagem **não funciona** para `string`s com caracteres complexos (unicode) nelas (símbolos astrais, caracteres multibyte, etc). Você precisa de bibliotecas mais sofisticadas, que são unicode-conscientes, para lidar com caracteres unicode corretamente. Veja o trabalo de Mathias Bynens neste assunto: _Esrever_ (<https://github.com/mathiasbynens/esrever>).

Outra visão nesse ponto é: Se você comumente trata "strings" como _arrays de caracteres_, talvez seja melhor usar `array`s, ao invés de `string`s. Você provavelmente salvará muito trabalho de conversão de `string` para `array` cada vez. Você sempre pode chamar `join("")` em um `array` _de caracteres_ sempre que realmente precisar da representação de `string`.

## Números

JavaScript possui apenas um tipo numérico: `number`. Este tipo inclui ambos valores "inteiro" e números decimais. Digo "inteiro" entre aspas porque há uma crítica de longa data ao JavaScript de que não há inteiros verdadeiros, como há em outras linguagens. Isso pode mudar no futuro, mas por enquanto, temos apenas `number`s para tudo.

Então, em JS, um "inteiro" é apenas um valor que não contém a parte decimal. Sendo assim, `42.0` é tão "inteiro" quanto `42`.

Como na maioria das linguagens modernas, incluindo praticamente todas as linguagens de script, a implementação de `number`s do JavaScript é baseada no padrão "IEEE 754", frequentemente chamado de "ponto flutuante". JavaScript especificamente usa o formato de "dupla precisão" (ou "binário de 64 bits") desse padrão.

Existem vários escritos muito bons na Web que abordam os detalhes essenciais de como números binários de ponto flutuante são armazenados em memória, e as implicações dessas escolhas. Por não ser estritamente necessário o entendimento dos padrões de bit em memória para entender como usar corretamente `number`s em JS, nós o deixaremos como exercício para o leitor que tenha o interesse de se aprofundar nos detalhes do padrão IEEE 754.

### Sintaxe Numérica

No JavaScript, geralmente números são declarados como decimais literais de base 10. Por exemplo:

```js
var a = 42;
var b = 42.3;
```

A parte inicial de um valor decimal, se `0`, é opcional:

```js
var a = 0.42;
var b = .42;
```

De maneira similar, a parte fracionária de um valor decimal após o `.`, se `0`, é opcional:

```js
var a = 42.0;
var b = 42.;
```

**Aviso:** `42.` é bem incomum, e talvez não seja uma boa ideia se você estiver tentando evitar que outras pessoas fiquem confusas ao lerem seu código. Mas, no entanto, é válido.

A maioria dos `number`s, por padrão, serão exibidos como decimais na base 10, com os `0`s à direita da parte fracionária removidos. Então:

```js
var a = 42.300;
var b = 42.0;

a; // 42.3
b; // 42
```

Por padrão, `number`s muito grandes ou muito pequenos serão exibidos na forma exponencial, a mesma que é retornada pelo método `toExponential()`, como:

```js
var a = 5E10;
a;					// 50000000000
a.toExponential();	// "5e+10"

var b = a * a;
b;					// 2.5e+21

var c = 1 / a;
c;					// 2e-11
```

Como valores `number` podem ser englobados no objeto `Number`, que envolve tipos primitivos (veja o Capítulo 3), esses tem acesso aos métodos presentes no `Number.prototype` (veja o Capítulo 3). Por exemplo, o método `toFixed(..)` permite que você especifique com quantas casas decimais você gostaria que o valor fosse representado:

```js
var a = 42.59;

a.toFixed( 0 ); // "43"
a.toFixed( 1 ); // "42.6"
a.toFixed( 2 ); // "42.59"
a.toFixed( 3 ); // "42.590"
a.toFixed( 4 ); // "42.5900"
```

Note que a saída é uma representação `string` do `number`, e que ao valor são acrescidos `0`s à direita se você solicita mais casas decimais do que o valor mantém.

`toPrecision(..)` é semelhante, porém especifica quantos *dígitos significativos* deveriam ser usados para representar o valor:

```js
var a = 42.59;

a.toPrecision( 1 ); // "4e+1"
a.toPrecision( 2 ); // "43"
a.toPrecision( 3 ); // "42.6"
a.toPrecision( 4 ); // "42.59"
a.toPrecision( 5 ); // "42.590"
a.toPrecision( 6 ); // "42.5900"
```

Você não precisa usar uma variável com um valor atribuído a ela para acessar estes métodos; você pode acessá-los diretamente em literais `number`. Porém tenha cuidado com o operador `.`. Como `.` é um caractere numérico válido, ele será primeiro interpretado como parte do literal `number`, se possível, ao invés de ser interpretado como um operador de acesso.

```js
// sintaxe inválida:
42.toFixed( 3 );	// SyntaxError

// todos estes são válidos:
(42).toFixed( 3 );	// "42.000"
0.42.toFixed( 3 );	// "0.420"
42..toFixed( 3 );	// "42.000"
```

`42.toFixed(3)` está com sintaxe inválida, porque o `.` faz parte do literal `42.` (que é válido -- veja acima!), então não há operador de acesso `.` presente para acessar `.toFixed`.

`42..toFixed(3)` funciona porque o primeiro `.` é parte do `number` e o segundo `.` é o operador de acesso. Mas isso parece estranho, e na verdade é bastante raro ver algo assim em um código JavaScript atual. De fato, é bem incomum acessar métodos diretamente de qualquer um dos valores primitivos. Incomum não significa *ruim* ou *errado*.

**Nota:** Há bibliotecas que estendem o `Number.prototype` interno (veja o Capítulo 3) para fornecer operações extras em/com `number`s, e nesses casos, é perfeitamente válido usar algo assim `10..makeItRain()` para definir uma chuva de dinheiro de 10 segundos, ou alguma outra tolice como essa.

Isto também é tecnicamente válido (note o espaço):

```js
42 .toFixed(3); // "42.000"
```

No entanto, tratando-se especificamente do literal `number`, **isso é um estilo de código particularmente confuso** e não terá propósito algum exceto o de confundir outros desenvolvedores (e você mesmo no futuro). Evite isso.

`number`s também podem ser especificados na forma exponencial, algo comum ao representar `number`s grandes, tipo:

```js
var onethousand = 1E3;						// significa 1 * 10^3
var onemilliononehundredthousand = 1.1E6;	// significa 1.1 * 10^6
```

Literais `number` também podem ser representados em outras bases, como binária, octal, e hexadecimal.

Estes formatos funcionam em versões atuais do JavaScript:

```js
0xf3; // hexadecimal para: 243
0Xf3; // idem

0363; // octal para: 243
```

**Nota:** A partir do ES6 + `strict mode`, a forma `0363` das literais octais não é mais permitida (veja abaixo a nova forma). A forma `0363` ainda é aceita em modo não `strict`, mas mesmo assim você deveria parar de usá-la, para ser aceitável no futuro (e pelo fato de que você já deveria estar usando `strict mode`!).

A partir do ES6, as novas formas seguintes são também válidas:

```js
0o363;		// octal para: 243
0O363;		// idem

0b11110011;	// binário para: 243
0B11110011; // idem
```

Faça um favor aos seus companheiros no desenvolvimento: nunca use a forma `0O363`. Usar o `0` ao lado do `O` maiúsculo é pedir confusão. Sempre use predicados minúsculos `0x`, `0b`, e `0o`.

### Valores Decimais Pequenos

O efeito colateral mais (vergonhoso)famoso de usar números de ponto flutuante binários (lembrando que, é verdade para **todas** as linguagens que usam IEEE 754 -- não *apenas* JavaScript como muitos assumem/acreditam) é:

```js
0.1 + 0.2 === 0.3; // false
```

Matematicamente, sabemos que essa afirmação deveria ser `true`. Por que é `false`?

Simplificando, as representações para `0.1` e `0.2` em ponto flutuante binário não são exatas, então, quando elas são somadas, o resultado não é exatamnte `0.3`. Ele é **realmente** próximo: `0.30000000000000004`, mas se sua comparação falhar, "próximo" é irrelevante.

**Nota:** O JavaScript deveria alterar para uma implementação diferente de `number` que tenha representações exatas de todos os valores? Alguns pensam que sim. Houveram muitas alternativas apresentadas ao longo dos anos. Nenhuma delas foi aceita, e talvez nunca seja. Por mais fácil que pareça apenas acenar e dizer, "corrija esse erro já!", Não é tão fácil. Se fosse, definitivamente teria sido alterado há muito tempo.

Agora a questão é, se alguns `number`s (números) não podem ser *confiáveis* para serem exatos, isso significa que não podemos usar `number`s (números)? **Claro que não.**

Existem algumas aplicações nas quais você precisa ter mais cuidado, especialmente quando se trata de valores decimais fracionários. Há também muitas aplicações (talvez a maioria?) que lidam apenas com números ("inteiros") e, além disso, lidam apenas com números na casa dos milhões ou trilhões no máximo. Estas utilizações foram, e sempre serão, **perfeitamente seguras** para utilizar operações numéricas em JS.

E se nós *precisássemos* comparar dois `number`s (números) como, `0.1 + 0.2` a `0.3`, sabendo que o teste de igualdade simples falha?

A prática mais comumente aceita é usar um pequeno valor de "arredondamento" como uma *tolerância* para a comparação. Este valor é normalmente chamado de "machine epsilon", que é geralmente `2^-52` (`2.220446049250313e-16`) para os tipos de `number`s (números) em JavaScript.

A partir do ES6, `Number.EPSILON` está predefinido com este valor de tolerância, você gostaria de utilizá-lo, mas você pode seguramente definir um polyfill para pre-ES6:

```js
if (!Number.EPSILON) {
	Number.EPSILON = Math.pow(2,-52);
}
```

Podemos utilizar este `Number.EPSILON` para comparar a "igualdade" de dois `number`s (números) (dentro da tolerância de arredondamento):

```js
function numbersCloseEnoughToEqual(n1,n2) {
	return Math.abs( n1 - n2 ) < Number.EPSILON;
}

var a = 0.1 + 0.2;
var b = 0.3;

numbersCloseEnoughToEqual( a, b );					// true
numbersCloseEnoughToEqual( 0.0000001, 0.0000002 );	// false
```

O valor máximo de ponto flutuante que pode ser representado é, aproximadamente, `1.798e+308` (que é realmente, realmente, realmente enorme!), predefinido para você como `Number.MAX_VALUE`. Na ponta menor, `Number.MIN_VALUE` é, aproximadamente, `5e-324`, que não é negativo, mas é muito próximo a zero!

### Variação Segura de Inteiros

Por causa da forma como os `number`s (números) são representados, existem uma série de valores "seguros" para todo `number` "inteiro", e é significativamente menor que `Number.MAX_VALUE`.

O número inteiro máximo que pode ser representado com "segurança" (isto é, há garantia de que o valor solicitado é realmente representável de forma inequívoca) é `2^53 - 1`, que é `9007199254740991`. Se você inserir a pontuação, verá que é um pouco mais de 9 quadrilhões. Então, esta é uma variação bem grande para `number`s (números).

Este valor está automaticamente predefinido no ES6, como `Number.MAX_SAFE_INTEGER`. Não é surpreendente que haja um valor mínimo, `-9007199254740991`, que é definido como `Number.MIN_SAFE_INTEGER` no ES6.

A principal maneira na qual as aplicaçoes JS se deparam com números tão grandes é quando lidam com IDs de 64-bits de banco de dados, etc. Os números de 64-bit não podem ser representados com precisão com o tipo `number`, e então devem ser armazenados (e transmitidos de/para) em JavaScript usando `string`.

As operações numéricas de valores tão grandes de ID com `number` (além da comparação, que será passível com `string`s) não são tão comuns, felizmente. Mas se você *precisar* executar cálculos matemáticos nesses valores muito grandes, por enquanto você precisará utilizar um utilitário para *big number (números grandes)*. Big numbers (números grandes) pode obter suporte oficial em uma futura versão do JavaScript.

### Testing for Integers

To test if a value is an integer, you can use the ES6-specified `Number.isInteger(..)`:

```js
Number.isInteger( 42 );		// true
Number.isInteger( 42.000 );	// true
Number.isInteger( 42.3 );	// false
```

To polyfill `Number.isInteger(..)` for pre-ES6:

```js
if (!Number.isInteger) {
	Number.isInteger = function(num) {
		return typeof num == "number" && num % 1 == 0;
	};
}
```

To test if a value is a *safe integer*, use the ES6-specified `Number.isSafeInteger(..)`:

```js
Number.isSafeInteger( Number.MAX_SAFE_INTEGER );	// true
Number.isSafeInteger( Math.pow( 2, 53 ) );			// false
Number.isSafeInteger( Math.pow( 2, 53 ) - 1 );		// true
```

To polyfill `Number.isSafeInteger(..)` in pre-ES6 browsers:

```js
if (!Number.isSafeInteger) {
	Number.isSafeInteger = function(num) {
		return Number.isInteger( num ) &&
			Math.abs( num ) <= Number.MAX_SAFE_INTEGER;
	};
}
```

### 32-bit (Signed) Integers

While integers can range up to roughly 9 quadrillion safely (53 bits), there are some numeric operations (like the bitwise operators) that are only defined for 32-bit `number`s, so the "safe range" for `number`s used in that way must be much smaller.

The range then is `Math.pow(-2,31)` (`-2147483648`, about -2.1 billion) up to `Math.pow(2,31)-1` (`2147483647`, about +2.1 billion).

To force a `number` value in `a` to a 32-bit signed integer value, use `a | 0`. This works because the `|` bitwise operator only works for 32-bit integer values (meaning it can only pay attention to 32 bits and any other bits will be lost). Then, "or'ing" with zero is essentially a no-op bitwise speaking.

**Note:** Certain special values (which we will cover in the next section) such as `NaN` and `Infinity` are not "32-bit safe," in that those values when passed to a bitwise operator will pass through the abstract operation `ToInt32` (see Chapter 4) and become simply the `+0` value for the purpose of that bitwise operation.

## Special Values

There are several special values spread across the various types that the *alert* JS developer needs to be aware of, and use properly.

### The Non-value Values

For the `undefined` type, there is one and only one value: `undefined`. For the `null` type, there is one and only one value: `null`. So for both of them, the label is both its type and its value.

Both `undefined` and `null` are often taken to be interchangeable as either "empty" values or "non" values. Other developers prefer to distinguish between them with nuance. For example:

* `null` is an empty value
* `undefined` is a missing value

Or:

* `undefined` hasn't had a value yet
* `null` had a value and doesn't anymore

Regardless of how you choose to "define" and use these two values, `null` is a special keyword, not an identifier, and thus you cannot treat it as a variable to assign to (why would you!?). However, `undefined` *is* (unfortunately) an identifier. Uh oh.

### Undefined

In non-`strict` mode, it's actually possible (though incredibly ill-advised!) to assign a value to the globally provided `undefined` identifier:

```js
function foo() {
	undefined = 2; // really bad idea!
}

foo();
```

```js
function foo() {
	"use strict";
	undefined = 2; // TypeError!
}

foo();
```

In both non-`strict` mode and `strict` mode, however, you can create a local variable of the name `undefined`. But again, this is a terrible idea!

```js
function foo() {
	"use strict";
	var undefined = 2;
	console.log( undefined ); // 2
}

foo();
```

**Friends don't let friends override `undefined`.** Ever.

#### `void` Operator

While `undefined` is a built-in identifier that holds (unless modified -- see above!) the built-in `undefined` value, another way to get this value is the `void` operator.

The expression `void ___` "voids" out any value, so that the result of the expression is always the `undefined` value. It doesn't modify the existing value; it just ensures that no value comes back from the operator expression.

```js
var a = 42;

console.log( void a, a ); // undefined 42
```

By convention (mostly from C-language programming), to represent the `undefined` value stand-alone by using `void`, you'd use `void 0` (though clearly even `void true` or any other `void` expression does the same thing). There's no practical difference between `void 0`, `void 1`, and `undefined`.

But the `void` operator can be useful in a few other circumstances, if you need to ensure that an expression has no result value (even if it has side effects).

For example:

```js
function doSomething() {
	// note: `APP.ready` is provided by our application
	if (!APP.ready) {
		// try again later
		return void setTimeout( doSomething, 100 );
	}

	var result;

	// do some other stuff
	return result;
}

// were we able to do it right away?
if (doSomething()) {
	// handle next tasks right away
}
```

Here, the `setTimeout(..)` function returns a numeric value (the unique identifier of the timer interval, if you wanted to cancel it), but we want to `void` that out so that the return value of our function doesn't give a false-positive with the `if` statement.

Many devs prefer to just do these actions separately, which works the same but doesn't use the `void` operator:

```js
if (!APP.ready) {
	// try again later
	setTimeout( doSomething, 100 );
	return;
}
```

In general, if there's ever a place where a value exists (from some expression) and you'd find it useful for the value to be `undefined` instead, use the `void` operator. That probably won't be terribly common in your programs, but in the rare cases you do need it, it can be quite helpful.

### Special Numbers

The `number` type includes several special values. We'll take a look at each in detail.

#### The Not Number, Number

Any mathematic operation you perform without both operands being `number`s (or values that can be interpreted as regular `number`s in base 10 or base 16) will result in the operation failing to produce a valid `number`, in which case you will get the `NaN` value.

`NaN` literally stands for "not a `number`", though this label/description is very poor and misleading, as we'll see shortly. It would be much more accurate to think of `NaN` as being "invalid number," "failed number," or even "bad number," than to think of it as "not a number."

For example:

```js
var a = 2 / "foo";		// NaN

typeof a === "number";	// true
```

In other words: "the type of not-a-number is 'number'!" Hooray for confusing names and semantics.

`NaN` is a kind of "sentinel value" (an otherwise normal value that's assigned a special meaning) that represents a special kind of error condition within the `number` set. The error condition is, in essence: "I tried to perform a mathematic operation but failed, so here's the failed `number` result instead."

So, if you have a value in some variable and want to test to see if it's this special failed-number `NaN`, you might think you could directly compare to `NaN` itself, as you can with any other value, like `null` or `undefined`. Nope.

```js
var a = 2 / "foo";

a == NaN;	// false
a === NaN;	// false
```

`NaN` is a very special value in that it's never equal to another `NaN` value (i.e., it's never equal to itself). It's the only value, in fact, that is not reflexive (without the Identity characteristic `x === x`). So, `NaN !== NaN`. A bit strange, huh?

So how *do* we test for it, if we can't compare to `NaN` (since that comparison would always fail)?

```js
var a = 2 / "foo";

isNaN( a ); // true
```

Easy enough, right? We use the built-in global utility called `isNaN(..)` and it tells us if the value is `NaN` or not. Problem solved!

Not so fast.

The `isNaN(..)` utility has a fatal flaw. It appears it tried to take the meaning of `NaN` ("Not a Number") too literally -- that its job is basically: "test if the thing passed in is either not a `number` or is a `number`." But that's not quite accurate.

```js
var a = 2 / "foo";
var b = "foo";

a; // NaN
b; // "foo"

window.isNaN( a ); // true
window.isNaN( b ); // true -- ouch!
```

Clearly, `"foo"` is literally *not a `number`*, but it's definitely not the `NaN` value either! This bug has been in JS since the very beginning (over 19 years of *ouch*).

As of ES6, finally a replacement utility has been provided: `Number.isNaN(..)`. A simple polyfill for it so that you can safely check `NaN` values *now* even in pre-ES6 browsers is:

```js
if (!Number.isNaN) {
	Number.isNaN = function(n) {
		return (
			typeof n === "number" &&
			window.isNaN( n )
		);
	};
}

var a = 2 / "foo";
var b = "foo";

Number.isNaN( a ); // true
Number.isNaN( b ); // false -- phew!
```

Actually, we can implement a `Number.isNaN(..)` polyfill even easier, by taking advantage of that peculiar fact that `NaN` isn't equal to itself. `NaN` is the *only* value in the whole language where that's true; every other value is always **equal to itself**.

So:

```js
if (!Number.isNaN) {
	Number.isNaN = function(n) {
		return n !== n;
	};
}
```

Weird, huh? But it works!

`NaN`s are probably a reality in a lot of real-world JS programs, either on purpose or by accident. It's a really good idea to use a reliable test, like `Number.isNaN(..)` as provided (or polyfilled), to recognize them properly.

If you're currently using just `isNaN(..)` in a program, the sad reality is your program *has a bug*, even if you haven't been bitten by it yet!

#### Infinities

Developers from traditional compiled languages like C are probably used to seeing either a compiler error or runtime exception, like "Divide by zero," for an operation like:

```js
var a = 1 / 0;
```

However, in JS, this operation is well-defined and results in the value `Infinity` (aka `Number.POSITIVE_INFINITY`). Unsurprisingly:

```js
var a = 1 / 0;	// Infinity
var b = -1 / 0;	// -Infinity
```

As you can see, `-Infinity` (aka `Number.NEGATIVE_INFINITY`) results from a divide-by-zero where either (but not both!) of the divide operands is negative.

JS uses finite numeric representations (IEEE 754 floating-point, which we covered earlier), so contrary to pure mathematics, it seems it *is* possible to overflow even with an operation like addition or subtraction, in which case you'd get `Infinity` or `-Infinity`.

For example:

```js
var a = Number.MAX_VALUE;	// 1.7976931348623157e+308
a + a;						// Infinity
a + Math.pow( 2, 970 );		// Infinity
a + Math.pow( 2, 969 );		// 1.7976931348623157e+308
```

According to the specification, if an operation like addition results in a value that's too big to represent, the IEEE 754 "round-to-nearest" mode specifies what the result should be. So, in a crude sense, `Number.MAX_VALUE + Math.pow( 2, 969 )` is closer to `Number.MAX_VALUE` than to `Infinity`, so it "rounds down," whereas `Number.MAX_VALUE + Math.pow( 2, 970 )` is closer to `Infinity` so it "rounds up".

If you think too much about that, it's going to make your head hurt. So don't. Seriously, stop!

Once you overflow to either one of the *infinities*, however, there's no going back. In other words, in an almost poetic sense, you can go from finite to infinite but not from infinite back to finite.

It's almost philosophical to ask: "What is infinity divided by infinity". Our naive brains would likely say "1" or maybe "infinity." Turns out neither is true. Both mathematically and in JavaScript, `Infinity / Infinity` is not a defined operation. In JS, this results in `NaN`.

But what about any positive finite `number` divided by `Infinity`? That's easy! `0`. And what about a negative finite `number` divided by `Infinity`? Keep reading!

#### Zeros

While it may confuse the mathematics-minded reader, JavaScript has both a normal zero `0` (otherwise known as a positive zero `+0`) *and* a negative zero `-0`. Before we explain why the `-0` exists, we should examine how JS handles it, because it can be quite confusing.

Besides being specified literally as `-0`, negative zero also results from certain mathematic operations. For example:

```js
var a = 0 / -3; // -0
var b = 0 * -3; // -0
```

Addition and subtraction cannot result in a negative zero.

A negative zero when examined in the developer console will usually reveal `-0`, though that was not the common case until fairly recently, so some older browsers you encounter may still report it as `0`.

However, if you try to stringify a negative zero value, it will always be reported as `"0"`, according to the spec.

```js
var a = 0 / -3;

// (some browser) consoles at least get it right
a;							// -0

// but the spec insists on lying to you!
a.toString();				// "0"
a + "";						// "0"
String( a );				// "0"

// strangely, even JSON gets in on the deception
JSON.stringify( a );		// "0"
```

Interestingly, the reverse operations (going from `string` to `number`) don't lie:

```js
+"-0";				// -0
Number( "-0" );		// -0
JSON.parse( "-0" );	// -0
```

**Warning:** The `JSON.stringify( -0 )` behavior of `"0"` is particularly strange when you observe that it's inconsistent with the reverse: `JSON.parse( "-0" )` reports `-0` as you'd correctly expect.

In addition to stringification of negative zero being deceptive to hide its true value, the comparison operators are also (intentionally) configured to *lie*.

```js
var a = 0;
var b = 0 / -3;

a == b;		// true
-0 == 0;	// true

a === b;	// true
-0 === 0;	// true

0 > -0;		// false
a > b;		// false
```

Clearly, if you want to distinguish a `-0` from a `0` in your code, you can't just rely on what the developer console outputs, so you're going to have to be a bit more clever:

```js
function isNegZero(n) {
	n = Number( n );
	return (n === 0) && (1 / n === -Infinity);
}

isNegZero( -0 );		// true
isNegZero( 0 / -3 );	// true
isNegZero( 0 );			// false
```

Now, why do we need a negative zero, besides academic trivia?

There are certain applications where developers use the magnitude of a value to represent one piece of information (like speed of movement per animation frame) and the sign of that `number` to represent another piece of information (like the direction of that movement).

In those applications, as one example, if a variable arrives at zero and it loses its sign, then you would lose the information of what direction it was moving in before it arrived at zero. Preserving the sign of the zero prevents potentially unwanted information loss.

### Special Equality

As we saw above, the `NaN` value and the `-0` value have special behavior when it comes to equality comparison. `NaN` is never equal to itself, so you have to use ES6's `Number.isNaN(..)` (or a polyfill). Simlarly, `-0` lies and pretends that it's equal (even `===` strict equal -- see Chapter 4) to regular positive `0`, so you have to use the somewhat hackish `isNegZero(..)` utility we suggested above.

As of ES6, there's a new utility that can be used to test two values for absolute equality, without any of these exceptions. It's called `Object.is(..)`:

```js
var a = 2 / "foo";
var b = -3 * 0;

Object.is( a, NaN );	// true
Object.is( b, -0 );		// true

Object.is( b, 0 );		// false
```

There's a pretty simple polyfill for `Object.is(..)` for pre-ES6 environments:

```js
if (!Object.is) {
	Object.is = function(v1, v2) {
		// test for `-0`
		if (v1 === 0 && v2 === 0) {
			return 1 / v1 === 1 / v2;
		}
		// test for `NaN`
		if (v1 !== v1) {
			return v2 !== v2;
		}
		// everything else
		return v1 === v2;
	};
}
```

`Object.is(..)` probably shouldn't be used in cases where `==` or `===` are known to be *safe* (see Chapter 4 "Coercion"), as the operators are likely much more efficient and certainly are more idiomatic/common. `Object.is(..)` is mostly for these special cases of equality.

## Value vs. Reference

In many other languages, values can either be assigned/passed by value-copy or by reference-copy depending on the syntax you use.

For example, in C++ if you want to pass a `number` variable into a function and have that variable's value updated, you can declare the function parameter like `int& myNum`, and when you pass in a variable like `x`, `myNum` will be a **reference to `x`**; references are like a special form of pointers, where you obtain a pointer to another variable (like an *alias*). If you don't declare a reference parameter, the value passed in will *always* be copied, even if it's a complex object.

In JavaScript, there are no pointers, and references work a bit differently. You cannot have a reference from one JS variable to another variable. That's just not possible.

A reference in JS points at a (shared) **value**, so if you have 10 different references, they are all always distinct references to a single shared value; **none of them are references/pointers to each other.**

Moreover, in JavaScript, there are no syntactic hints that control value vs. reference assignment/passing. Instead, the *type* of the value *solely* controls whether that value will be assigned by value-copy or by reference-copy.

Let's illustrate:

```js
var a = 2;
var b = a; // `b` is always a copy of the value in `a`
b++;
a; // 2
b; // 3

var c = [1,2,3];
var d = c; // `d` is a reference to the shared `[1,2,3]` value
d.push( 4 );
c; // [1,2,3,4]
d; // [1,2,3,4]
```

Simple values (aka scalar primitives) are *always* assigned/passed by value-copy: `null`, `undefined`, `string`, `number`, `boolean`, and ES6's `symbol`.

Compound values -- `object`s (including `array`s, and all boxed object wrappers -- see Chapter 3) and `function`s -- *always* create a copy of the reference on assignment or passing.

In the above snippet, because `2` is a scalar primitive, `a` holds one initial copy of that value, and `b` is assigned another *copy* of the value. When changing `b`, you are in no way changing the value in `a`.

But **both `c` and `d`** are seperate references to the same shared value `[1,2,3]`, which is a compound value. It's important to note that neither `c` nor `d` more "owns" the `[1,2,3]` value -- both are just equal peer references to the value. So, when using either reference to modify (`.push(4)`) the actual shared `array` value itself, it's affecting just the one shared value, and both references will reference the newly modified value `[1,2,3,4]`.

Since references point to the values themselves and not to the variables, you cannot use one reference to change where another reference is pointed:

```js
var a = [1,2,3];
var b = a;
a; // [1,2,3]
b; // [1,2,3]

// later
b = [4,5,6];
a; // [1,2,3]
b; // [4,5,6]
```

When we make the assignment `b = [4,5,6]`, we are doing absolutely nothing to affect *where* `a` is still referencing (`[1,2,3]`). To do that, `b` would have to be a pointer to `a` rather than a reference to the `array` -- but no such capability exists in JS!

The most common way such confusion happens is with function parameters:

```js
function foo(x) {
	x.push( 4 );
	x; // [1,2,3,4]

	// later
	x = [4,5,6];
	x.push( 7 );
	x; // [4,5,6,7]
}

var a = [1,2,3];

foo( a );

a; // [1,2,3,4]  not  [4,5,6,7]
```

When we pass in the argument `a`, it assigns a copy of the `a` reference to `x`. `x` and `a` are separate references pointing at the same `[1,2,3]` value. Now, inside the function, we can use that reference to mutate the value itself (`push(4)`). But when we make the assignment `x = [4,5,6]`, this is in no way affecting where the initial reference `a` is pointing -- still points at the (now modified) `[1,2,3,4]` value.

There is no way to use the `x` reference to change where `a` is pointing. We could only modify the contents of the shared value that both `a` and `x` are pointing to.

To accomplish changing `a` to have the `[4,5,6,7]` value contents, you can't create a new `array` and assign -- you must modify the existing `array` value:

```js
function foo(x) {
	x.push( 4 );
	x; // [1,2,3,4]

	// later
	x.length = 0; // empty existing array in-place
	x.push( 4, 5, 6, 7 );
	x; // [4,5,6,7]
}

var a = [1,2,3];

foo( a );

a; // [4,5,6,7]  not  [1,2,3,4]
```

As you can see, `x.length = 0` and `x.push(4,5,6,7)` were not creating a new `array`, but modifying the existing shared `array`. So of course, `a` references the new `[4,5,6,7]` contents.

Remember: you cannot directly control/override value-copy vs. reference -- those semantics are controlled entirely by the type of the underlying value.

To effectively pass a compound value (like an `array`) by value-copy, you need to manually make a copy of it, so that the reference passed doesn't still point to the original. For example:

```js
foo( a.slice() );
```

`slice(..)` with no parameters by default makes an entirely new (shallow) copy of the `array`. So, we pass in a reference only to the copied `array`, and thus `foo(..)` cannot affect the contents of `a`.

To do the reverse -- pass a scalar primitive value in a way where its value updates can be seen, kinda like a reference -- you have to wrap the value in another compound value (`object`, `array`, etc) that *can* be passed by reference-copy:

```js
function foo(wrapper) {
	wrapper.a = 42;
}

var obj = {
	a: 2
};

foo( obj );

obj.a; // 42
```

Here, `obj` acts as a wrapper for the scalar primitive property `a`. When passed to `foo(..)`, a copy of the `obj` reference is passed in and set to the `wrapper` parameter. We now can use the `wrapper` reference to access the shared object, and update its property. After the function finishes, `obj.a` will see the updated value `42`.

It may occur to you that if you wanted to pass in a reference to a scalar primitive value like `2`, you could just box the value in its `Number` object wrapper (see Chapter 3).

It *is* true a copy of the reference to this `Number` object *will* be passed to the function, but unfortunately, having a reference to the shared object is not going to give you the ability to modify the shared primitive value, like you may expect:

```js
function foo(x) {
	x = x + 1;
	x; // 3
}

var a = 2;
var b = new Number( a ); // or equivalently `Object(a)`

foo( b );
console.log( b ); // 2, not 3
```

The problem is that the underlying scalar primitive value is *not mutable* (same goes for `String` and `Boolean`). If a `Number` object holds the scalar primitive value `2`, that exact `Number` object can never be changed to hold another value; you can only create a whole new `Number` object with a different value.

When `x` is used in the expression `x + 1`, the underlying scalar primitive value `2` is unboxed (extracted) from the `Number` object automatically, so the line `x = x + 1` very subtly changes `x` from being a shared reference to the `Number` object, to just holding the scalar primitive value `3` as a result of the addition operation `2 + 1`. Therefore, `b` on the outside still references the original unmodified/immutable `Number` object holding the value `2`.

You *can* add properties on top of the `Number` object (just not change its inner primitive value), so you could exchange information indirectly via those additional properties.

This is not all that common, however; it probably would not be considered a good practice by most developers.

Instead of using the wrapper object `Number` in this way, it's probably much better to use the manual object wrapper (`obj`) approach in the earlier snippet. That's not to say that there's no clever uses for the boxed object wrappers like `Number` -- just that you should probably prefer the scalar primitive value form in most cases.

References are quite powerful, but sometimes they get in your way, and sometimes you need them where they don't exist. The only control you have over reference vs. value-copy behavior is the type of the value itself, so you must indirectly influence the assignment/passing behavior by which value types you choose to use.

## Review

In JavaScript, `array`s are simply numerically indexed collections of any value-type. `string`s are somewhat "`array`-like", but they have distinct behaviors and care must be taken if you want to treat them as `array`s. Numbers in JavaScript include both "integers" and floating-point values.

Several special values are defined within the primitive types.

The `null` type has just one value: `null`, and likewise the `undefined` type has just the `undefined` value. `undefined` is basically the default value in any variable or property if no other value is present. The `void` operator lets you create the `undefined` value from any other value.

`number`s include several special values, like `NaN` (supposedly "Not a Number", but really more appropriately "invalid number"); `+Infinity` and `-Infinity`; and `-0`.

Simple scalar primitives (`string`s, `number`s, etc.) are assigned/passed by value-copy, but compound values (`object`s, etc.) are assigned/passed by reference-copy. References are not like references/pointers in other languages -- they're never pointed at other variables/references, only at the underlying values.
