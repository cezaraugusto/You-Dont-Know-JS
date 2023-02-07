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

### Testando números inteiros

Para testar se um valor é um número inteiro, você pode usar a especificação ES6 `Number.isInteger(..)`:

```js
Number.isInteger( 42 );		// true
Number.isInteger( 42.000 );	// true
Number.isInteger( 42.3 );	// false
```

Para um polyfill de `Number.isInteger(..)` pre-ES6:

```js
if (!Number.isInteger) {
	Number.isInteger = function(num) {
		return typeof num == "number" && num % 1 == 0;
	};
}
```

Para testar se um valor é um *número inteiro seguro*, use a especificação ES6 `Number.isSafeInteger(..)`:

```js
Number.isSafeInteger( Number.MAX_SAFE_INTEGER );	// true
Number.isSafeInteger( Math.pow( 2, 53 ) );			// false
Number.isSafeInteger( Math.pow( 2, 53 ) - 1 );		// true
```

Para um polyfill de `Number.isSafeInteger(..)` em navegadores pre-ES6:

```js
if (!Number.isSafeInteger) {
	Number.isSafeInteger = function(num) {
		return Number.isInteger( num ) &&
			Math.abs( num ) <= Number.MAX_SAFE_INTEGER;
	};
}
```

### Números Inteiros de 32-bit (Sinalizado)

Enquanto números inteiros podem variar até 9 quatrilhões de forma segura (53 bits), exitem algumas operações numéricas (como os operadores bit a bit) que são definidas apenas para `number`s de 32 bits, portanto, o "intervalo seguro" para `number`s usados dessa maneira deve ser muito menor.

O intervalo é então `Math.pow(-2,31)` (`-2147483648`, cerca de -2.1 bilhões) até `Math.pow(2,31)-1` (`2147483647`, cerca de +2.1 bilhões).

Para forçar um valor de `number` em `a` para um valor inteiro sinalizado de 32-bit, use `a | 0`. Isto funciona porque o operador `|` bit a bit só funciona para números inteiros de 32-bit (o que significa que ele só pode prestar atenção a 32 bits e todos os outros bits serão perdidos). Então, "or (|) em seguida" com zero é essenciamente uma não-op bit a bit falando.

**Nota:** Certos valores especiais (que serão abordados na próxima seção) como `NaN` e `Infinity` não são "seguros em 32-bit," pois esses valores, quando passados para um operador bit a bit, passarão pela operação abstrata `ToInt32` (veja o Capítulo 4) e se tornarão simplesmente o valor `+0` para a finalizadade dessa operação bit a bit.

## Valores Especiais

Existem vários valores especiais espalhados pelos vários tipos que o desenvolvedor JS *atento* precisa conhecer e usar corretamente.

### Os Valores Sem-valor (Non-value)

Para o tipo `undefined`, existe um e somente um valor: `undefined`. Para o tipo `null`, exite um e somente um valor: `null`. Então, para ambos, a label (rótulo) é tanto seu tipo quanto seu valor.

Ambos `undefined` e `null` são frequentemente considerados como intercambiáveis como valores "vazios" ou "sem valores". Outros desenvolvedores preferem distinguir entre eles com nuances. Por exemplo:

* `null` é um valor vazio
* `undefined` é um valor inexistente

Or:

* `undefined` ainda não teve um valor
* `null` tinha um valor e não tem mais

Independentemente de como você escolhe "definir" e usar esses dois valores, `null` é uma palavra-chave especial, não um identificador, e assim você não pode tratá-la como uma variável a ser atribuída (Por que você faria isso!?). No entando, `undefined` *é* (infelizmente) um identificador. Uh oh.

### Undefined

No modo não-`strict`, é realmente possível (embora incrivelmente imprudente!) atribuir um valor ao identificador `undefined` fornecido globalmente:

```js
function foo() {
	undefined = 2; // péssima idéia!
}

foo();
```

```js
function foo() {
	"use strict";
	undefined = 2; // Erro de tipo (TypeError)!
}

foo();
```

No modo não-`strict` e no modo `strict`, no entanto, você pode criar uma variável local com o nome `undefined`. Mas, novamente, esta é uma idéia terrível!

```js
function foo() {
	"use strict";
	var undefined = 2;
	console.log( undefined ); // 2
}

foo();
```

**Amigos não deixam amigos sobrescrever `undefined`.** Nunca.

#### Operador `void`

Enquanto `undefined` é um identificador nativo que guarda (a menos que modificado -- veja acima) o valor nativo `undefined`, outra forma de ter esse valor é pelo operador `void`.

A expressão `void ___` "esvazia" qualquer valor, então o resultado da expressão será sempre um valor `undefined`. Ele não modifica o valor existente; ele apenas garante que nenhum valor retorne do operador da expressão.

```js
var a = 42;

console.log( void a, a ); // undefined 42
```

Por convenção (principalmente a partir da linguagem de programação C), para representar o valor `undefined` independente usando `void`, você usaria o `void 0` (embora claramente mesmo `void true` ou qualquer outra expressão `void` faz a mesma coisa). Não há nenhuma diferença prática entre `void 0`, `void 1` e `undefined`.

Mas o operador `void` pode ser útil em algumas outras circunstância, se você precisar garantir que essa expressão não tenha um valor resultante (mesmo que isso tenha efeitos colaterais).

Por exemplo:

```js
function doSomething() {
	// nota: `APP.ready` é fornecido por sua aplicação
	if (!APP.ready) {
		// tente novamente mais tarde
		return void setTimeout( doSomething, 100 );
	}

	var result;

	// faça outra coisa
	return result;
}

// estamos prontos para fazer isso agora?
if (doSomething()) {
	// lida com a próxima tarefa agora
}
```

Aqui, a função `setTimeout(..)` retorna um valor numérico (o único indentificador do intervalo de tempo, se você quiser cancelá-lo), mas não queremos o `void` esvazie-o para que então o valor retornado da nossa função não dê um falso positivo com a declaração `if`.

Muitos devs preferem apenas fazer essas ações separadamente, o que funciona da mesma forma mas não usa o operador `void`:

```js
if (!APP.ready) {
	// tente novamente mais tarde
	setTimeout( doSomething, 100 );
	return;
}
```

Em geral, se houver algum lugar onde exista um valor (de alguma expressão) e você ache útil que o valor seja `undefined`, use o operador` void`. Isso provavelmente não será muito comum em seus programas, mas nos casos raros que você precisar, pode ser bastante útil.

### Números Especiais

O tipo `number` inclui vários valores especiais. Vamos dar uma olhada em cada um, em detalhes.

#### O não número, Number

Qualquer operação matemática que você execute sem que os dois operandos sejam `number`s (ou valores que possam ser interpretados como `number`s regulares na base 10 ou base 16) resultará na falha da operação em produzir um `number` válido; nesse caso, você receberá o valor de `NaN`.

`NaN` significa literalmente "não um `number`", embora este rótulo/descrição seja muito ruim e enganoso, como veremos em breve. Seria muito mais preciso pensar em `NaN` como "número inválido", "número com falha" ou até "número ruim" do que em "não um número".

Por exemplo:

```js
var a = 2 / "foo";		// NaN

typeof a === "number";	// true
```

Em outras palavras: "o tipo de não-número é 'número'!" Um viva para confundir nomes e semânticas.

`NaN` é um tipo de "valor sentinela" (um valor teoricamente normal, mas que recebe um significado especial) que representa um tipo especial de condição de erro dentro do conjunto de `number`. A condição de erro é, em essência: "Tentei executar uma operação matemática, mas falhei, então aqui está o resultado do 'número' com falha".

Portanto, se você tem um valor em alguma variável e deseja testar para ver se é esse número especial com falha `NaN`, você pode pensar em comparar diretamente com o próprio `NaN`, como pode com qualquer outro valor, como `null` ou `undefined`. Só que não.

```js
var a = 2 / "foo";

a == NaN;	// false
a === NaN;	// false
```

`NaN` é um valor muito especial, pois nunca é igual a outro valor `NaN` (ou seja, nunca é igual a si mesmo). Na verdade, é o único valor que não é reflexivo (sem a característica de identidade `x === x`). Então, `NaN !== NaN`. Um pouco estranho, né?

Então, *como* testamos isso, se não podemos comparar com `NaN` (já que essa comparação sempre falha)?

```js
var a = 2 / "foo";

isNaN( a ); // true
```

Fácil o suficiente, certo? Usamos o utilitário global interno chamado `isNaN (..)` e ele diz se o valor é `NaN` ou não. Problema resolvido!

Não tão rápido.

O utilitário `isNaN (..)` tem uma falha fatal. Parece que ele tentou levar o significado de `NaN` ("Não é um número") tão literalmente -- que seu trabalho é basicamente: "testar se a coisa passada não é um `number` ou é um` number`." Mas isso não é tão preciso.

```js
var a = 2 / "foo";
var b = "foo";

a; // NaN
b; // "foo"

window.isNaN( a ); // true
window.isNaN( b ); // true -- eita!
```

Claramente, `"foo"` é literalmente *não um `number`*, mas definitivamente também não é `NaN`! Esse bug está no JS desde o início (mais de 19 anos de *eita*).

A partir do ES6, finalmente, um utilitário de substituição foi fornecido: `Number.isNaN (..)`. Um polyfill simples para que você possa verificar com segurança os valores `NaN` *agora*, mesmo nos navegadores anteriores ao ES6:

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
Number.isNaN( b ); // false -- ufa!
```

Na verdade, podemos implementar um polyfill `Number.isNaN (..)` ainda mais fácil, aproveitando esse fato peculiar de que `NaN` não é igual a si mesmo. `NaN` é o *único* valor em todo o idioma em que isso é verdade; todo outro valor é sempre **igual a si mesmo**.

Assim:

```js
if (!Number.isNaN) {
	Number.isNaN = function(n) {
		return n !== n;
	};
}
```

Estranho, né? Mas funciona!

`NaN`s provavelmente são uma realidade em muitos programas JS do mundo real, de propósito ou por acidente. É realmente uma boa idéia usar um teste confiável, como `Number.isNaN (..)` como fornecido (ou preenchido com polyfill), para reconhecê-los adequadamente.

Se atualmente você está usando apenas `isNaN (..)` em um programa, a triste realidade é que seu programa *possui um bug*, mesmo que você não tenha sido mordido por ele ainda!

#### Infinitos

Os desenvolvedores de linguagens compiladas tradicionais como C, provavelmente estão acostumados a ver um erro do compilador ou uma exceção de tempo de execução, como "Dividir por zero", para uma operação como:

```js
var a = 1 / 0;
```
No entanto, em JS, esta operação é bem definida e resulta no valor `Infinity` (também conhecido como `Number.POSITIVE_INFINITY`). Como esperado:

```js
var a = 1 / 0;	// Infinity
var b = -1 / 0;	// -Infinity
```
Como você pode ver, `-Infinity` (também conhecido como `Number.NEGATIVE_INFINITY`) resulta de uma divisão por zero, onde qualquer um dos dois (mas não ambos!) dos operandos de divisão é negativo.

JS usa representações numéricas finitas (ponto flutuante IEEE 754, que abordamos anteriormente), portanto, ao contrário da matemática pura, parece que *é* possível transbordar (overflow) mesmo com uma operação como adição ou subtração; nesse caso, você obterá `Infinity` ou `-Infinity`.

Por exemplo:

For example:

```js
var a = Number.MAX_VALUE;	// 1.7976931348623157e+308
a + a;						// Infinity
a + Math.pow( 2, 970 );		// Infinity
a + Math.pow( 2, 969 );		// 1.7976931348623157e+308
```
De acordo com a especificação, se uma operação como adição resultar em um valor grande demais para ser representado, o modo IEEE 754 "arredondar para o mais próximo" especificará qual deve ser o resultado. Portanto, em um sentido grosseiro, `Number.MAX_VALUE + Math.pow (2, 969)` está mais próximo de `Number.MAX_VALUE` do que com `Infinity`, então "arredonda para baixo", enquanto `Number.MAX_VALUE + Math. pow (2, 970)` está mais próximo de `Infinity`, de modo que "arredonda para cima".

Se você pensar muito sobre isso, vai doer sua cabeça. Então não. Sério, pare!

Depois que você transborda (overflow) para qualquer um dos *infinitos*, no entanto, não há como voltar atrás. Em outras palavras, em um sentido quase poético, você pode ir do finito ao infinito, mas não do infinito de volta ao finito.

É quase filosófico perguntar: "O que é o infinito dividido pelo infinito". Nossos cérebros ingênuos provavelmente diriam "1" ou talvez "infinito". Acontece que nenhuma das respostas estão certas. Nem Matematicamente e nem em JavaScript, `Infinity / Infinity` não é uma operação definida. Em JS, isso resulta em `NaN`.

Mas e quanto a qualquer `number` finito positivo dividido por `Infinity`? Isso é fácil! `0`. E o que dizer de um `number` finito negativo dividido por `Infinity`? Continue lendo!

#### Zeros

Embora possa confundir um leitor mais matemático, o JavaScript tem tanto um zero normal `0` (também conhecido como zero positivo `+ 0`) *e* um zero negativo `-0`. Antes de explicar por que o `-0` existe, devemos examinar como o JS lida com isso, porque pode ser bastante confuso.

Além de ser especificado literalmente como `-0`, o zero negativo também resulta de certas operações matemáticas. Por exemplo:

```js
var a = 0 / -3; // -0
var b = 0 * -3; // -0
```
Adição e subtração não podem resultar em zero negativo.

Um zero negativo quando examinado no console do desenvolvedor geralmente revela `-0`, embora esse não fosse o caso comum até bem recentemente, portanto, alguns navegadores mais antigos que você encontrar, ainda podem reportá-lo como `0`.

No entanto, se você tentar transformar em string um valor zero negativo, ele sempre será relatado como `"0"`, de acordo com a especificação.

```js
var a = 0 / -3;

// os consoles (de alguns navegadores) pelos menos acertam
a;							// -0

// mas a especificação insiste em mentir para você!
a.toString();				// "0"
a + "";						// "0"
String( a );				// "0"

// estranhamente, até JSON tenta te enganar
JSON.stringify( a );		// "0"
```
Curiosamente, as operações reversas (passando de `string` para` number`) não mentem:

```js
+"-0";				// -0
Number( "-0" );		// -0
JSON.parse( "-0" );	// -0
```

**Atenção:** O comportamento do `JSON.stringify( -0 )` de `"0"` é particularmente estranho quando você observa que é inconsistente com o inverso: `JSON.parse (" -0 ")` que reporta `- 0` como você esperaria corretamente.

Além de a stringificação do zero negativo ser enganosa por ocultar seu valor verdadeiro, os operadores de comparação também são (intencionalmente) configurados para *mentir*.

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
Claramente, se você deseja distinguir um `-0` de um` 0` no seu código, você não pode confiar apenas no que o console do desenvolvedor produz, então você terá que ser um pouco mais inteligente:

```js
function isNegZero(n) {
	n = Number( n );
	return (n === 0) && (1 / n === -Infinity);
}

isNegZero( -0 );		// true
isNegZero( 0 / -3 );	// true
isNegZero( 0 );			// false
```

Agora, por que precisamos de um zero negativo, além de pegadinhas acadêmicas?

Existem certas aplicações em que os desenvolvedores usam a magnitude de um valor para representar uma informação (como velocidade do movimento por quadro de animação) e o sinal desse `number` para representar outra informação (como a direção desse movimento).

Nessas aplicações, como um exemplo, se uma variável chega a zero e perde seu sinal, você perderia a informação de qual direção estava seguindo antes de chegar a zero. Preservar o sinal do zero evita a perda potencial de informações indesejadas.

### Igualdade Especial

Como vimos acima, os valores `NaN` e `-0` possuem um comportamento especial quando se trata de uma comparação de igualdade. `NaN` nunca é igual a ele mesmo, então você precisa usar `Number.isNaN(..)` do ES6 (ou um polyfill). Similarmente, `-0` mente e finge ser igual (até `===` igual estrito -- veja o Capítulo 4) ao seu regular positivo `0`, então você precisa de alguma forma usar o utilitário `isNegZero(..)` que sugerimos acima. 

A partir do ES6, há um novo utilitário que pode ser usado para testar dois valores usando igualdade absoluta, sem nenhuma dessas exceções. Ele se chama `Object.is(..)`:

```js
var a = 2 / "foo";
var b = -3 * 0;

Object.is( a, NaN );	// true
Object.is( b, -0 );		// true

Object.is( b, 0 );		// false
```

Existe um polyfill simples para `Object.is(..)` para ambientes pré-ES6: 
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

`Object.is(..)` provavelmente não deveria ser usado em casos onde `==` ou `===` são conhecidos por serem *seguros* (vaja o Capítulo 4 "Coerção"), já que esses operadores são mais eficientes e certamente mais comuns. `Object.is(..)` serve mais para esses casos especiais de igualdade. 

## Valor vs. Referência

Em muitas outras linguagens, valores podem ser atribuídos/passados como valores-copiados ou como referências-copiadas dependendo da sintaxe usada.

Por exemplo, em C++, se você quer passar uma variável `number` para uma função e ter o valor dessa variável atualizado, você pode declarar um parâmetro como `int& myNum`, e quando você passar uma variável como `x`, `myNum` irá ser uma **referência para `x`**; referências são como uma forma especial de ponteiros, onde você obtém um ponteiro para outra variável (como um *alias*). Se você não declara um parâmetro como referência, o valor passado *sempre* vai ser copiado, mesmo que seja um objeto complexo.

Em Javascript não existem ponteiros e referências funcionam um pouco diferente. Você não pode ter uma referência de uma variável JS para outra variável. Isso não é possível.

Uma referência em JS aponta para um **valor** (compartilhado), então você pode ter 10 diferentes referências, elas são sempre distintas para um mesmo valor compartilhado;

Além disso, em JavaScript não há dicas sintáticas que controlam atribuição/passagem de valores vs. referênicas. Ao invés disso, *exclusivamente o tipo* de um valor controla se o ele vai ser atribuído por valor-copiado ou por referência-copiada.

Vamos ilustrar:

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

Valore simples (chamados primitivos escalares) são *sempre* atribuídos/passados por valor-copiado: `null`, `undefined`, `string`, `number`, `boolean`, e o `symbol` do  ES6.

Valores compostos -- `objects`s (incluindo `array`s, e todo objeto que engloba um valor primitivo -- veja o Capítulo 3) e `function`s -- *sempre* criam uma cópia da referência ao serem atribuídos ou passados.

No snippet acima, como `2` é um primitivo escalar, `a` espera uma cópia inicial desse valor, e a `b` é atribuído outra *cópia* do valor. Quando mudamos `b`, não alteramos o valor de `a`.

Mas **ambos `c` e `d`** são referências separadas para os mesmo valor compartilhado `[1,2,3]`, que é um valor composto. É importante perceber que nem `c` nem `d` *possuem* mais que o outro o valor `[1,2,3]` -- ambos são igualmente referências para o valor. Então, quando se usa qualquer referência para modificar (`.push(4)`) o atual `array` compartilhado, isso afeta o próprio valor, e ambas as referências vão referenciar o novo valor modificado `[1,2,3,4]`.

Desde que referências apontem para os próprios valores e não para as variáveis, você não pode usar uma referência para mudar para onde outra referência está apontando:

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

Quando fazemos essa atribuição `b = [4,5,6]`, nós estamos fazendo absolutamente nada para afetar para *onde* `a` ainda está se referindo (`[1,2,3]`). Para fazer isso, `b` deveria ser um ponteiro para `a` ao invés de uma referência para o `array` -- mas não existe essa possibilidade em JS!

O jeito mais comum de haver essa confusão ocorre com parâmetros de funções:
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

Quando nós passamos `a` para o argumento, é atribuída uma cópia da referência de `a` para `x`. `x` e `a` são referências separadas apontando para o mesmo valor `[1,2,3]`. Agora, dentro da função, nós podemos usar essa referência para mudar o próprio valor (`push(4)`). Mas quando fazemos a atribuição `x = [4,5,6]`, isso não está afetando para onde a referência inicial `a` está apontando -- ainda apontand para o valor (agora modificado) `[1,2,3,4]`.

Não há como usar a referência de `x` para mudar para onde `a` está apontando. Nós poderíamos apenas modificar os conteúdos do valor compartilhado para o qual `a` e `x` apontam.

Para conseguir a mudança de `a` para conter o valor `[4,5,6,7]`, você não pode criar um novo `array` e atribuí-lo -- você deve modificar o valor do `array` existente:

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

Como você pode ver, `x.length = 0` e `x.push(4,5,6,7)` não estavam criando um novo `array`, mas modificando o `array` compartilhado. Então, é claro que `a` vai referenciar o novo conteúdo `[4,5,6,7]`.

Lembre-se: você não pode controlar/sobrepor diretamente o comportamento valores-copiados vs. referências -- essas semânticas são controladas inteiramente pelo tipo do valor subjacente.

Para efetivamente passar um valor composto (como um `array`) como valor-copiado, você precisa manualmente fazer uma cópia dele, então essa referência passada não apontará mais para o valor original. Por exemplo:  

```js
foo( a.slice() );
```

`slice(..)` sem nenhum parâmetro por padrão faz uma nova (rasa) cópia do `array`. Então, nós passamos em uma referência apenas a cópia do `array`, e por isso `foo(..)` não pode afetar o conteúdo de `a`.

Para fazer o inverso -- passar um valor primitivo escalar de um jeito que seu valor atualizdo possa ser visto, como uma referência -- você precisa englobar o valor em outro valor composto (`object`, `array`, etc) que *pode* ser passado como referência-copiada:

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

Aqui, `obj` age como um invólucro para a propriedade primitiva escalar `a`. Quando passada para `foo(..)`, uma cópia da referência para `obj` é passada para o parâmetro `wrapper`. Agora podemos usar a referência `wrapper` para acessar o objeto compartilhado e atualizar suas propriedades. Depois que a função termina, `obj.a` terá o valor atualizado para `42`. 

Você deve pensar que se você quisesse passar em uma referência um valor primitivo escalar como `2`, você poderia apenas encaixotar o valor em um objeto `Number` (veja o Capítulo 3).

*É* verdade que uma cópia de uma referência para esse objeto `Number` *vai* ser passada para uma função, mas infelizmente, tendo uma referência para o objeto compartilhado não irá te dar a habilidade de modificar o tipo primitivo compartilhado, como você poderia esperar:

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

O problema é que o valor escalar primitivo subjacente é *não mutável* (o mesmo ocorre com `String` e `Boolean`). Se um objeto `Number` mantém o valor escalar primitivo `2`, esse exato objeto `Number` nunca pode ser mudado para manter outro valor; você só pode por inteiro um novo objeto `Number` com um valor diferente. 

Quando `x` é usado na expressão `x + 1`, o valos primitivo escalar subjacente `2` é desempacotado (extraído) do objeto `Number` automaticamente, então a linha `x = x + 1` muito sutilmente muda `x` de uma referência compartilhada para o objeto `Number`, para apenas conter o valor primitivo escalar `3` como resultado da operação de adição `2 + 1`. Portanto, `b` do lado de fora ainda referencia o `Number` imodificável/imutável com o valor original `2`.

Você *pode* adicionar propriedades no topo do objeto `Number` (só não mudar seu valor primitivo interno), então você poderia trocar infromações indiretamente por meio dessas propriedades adicionais.

Isso não é tão comum, no entanto; isso provavelmente não deveria ser considerado uma boa prática pela maioria dos desenvolvedores.

Ao invés de usar o objeto `Number` desse jeito, é provavelmente melhor usar um objeto manual (`obj`) abordado no snippet mostrado anteriormente. Isso não quer dizer que não é bom usar objeto que englobam tipos primitivos como `Number` -- apenas que você provavelmente deveria preferir o valor primitivo escalar para a maioria dos casos. 

Referências são bastante poderosas, mas às vezes elas ficam no seu caminho, e às vezes você precisa que elas não existam. O único controle que você possui sobre o comportamento de referências vs. valores-copiados é o tipo de valor, então você deve indiretamente influenciar o comportamento de atribuição/passagem pelo tipo de valor que você escolhe usar

## Review

Em Javascript, `array`s são simplesmente coleções numericamente indexadas de qualquer tipo de valor. `string`s de uma certa forma "são como `array`s", mas elas possuem comportamento distinto e deve-se tomar cuidado quando se quer tratá-las como `array`s. Números em Javascript incluem os "inteiros" e os valores de ponto flutuante.

Diversos valores especiais são definidos dentro dos tipos primitivos.

O tipo `null` só possui um valor: `null`, da mesma forma que o tipo `undefined` só possui o valor `undefined`. `undefined` é basicamente o valor padrão em qualquer variável ou propriedade se outro valor não está presente. O operador `void` permite você criar o valor `undefined` a partir de qualquer outro valor.

`number`s incluem diversos valores especiais, como `NaN` (supostamente "Not a Number", mas na verdade é  mais apropriado dizer "invalid number"); `+Infinity` e `-Infinity`; e `-0`

Primitivos escalares simples (`string`s, `number`s, etc.) são atribuídos/passados como valores-copiados, mas valores compostos (`object`s, etc.) são atribuídos/passados como referências-copiadas. Referências não são como referências/ponteiros em outras linguagens -- elas nunca apontam para outras variáveis/referências, apenas para valores subjacentes.
