# You Don't Know JS: Tipos e Gramática
# Capítulo 3: Nativos

Várias vezes nos Capítulos 1 e 2, nós aludimos à vários nativos, como `String` e `Number`. Vamos examiná-los em detalhes agora.

Aqui está uma lista dos nativos mais comumente usados:

* `String()`
* `Number()`
* `Boolean()`
* `Array()`
* `Object()`
* `Function()`
* `RegExp()`
* `Date()`
* `Error()`
* `Symbol()` -- adicionado no ES6!

Como você pode ver, esses nativos são na realidade funções nativas.

Se você está vindo para o JS de linguagens como Java, a função do JavaScript `String()` vai parecer com o construtor `String(..)` que está acostumado a usar para criar valores string. Assim, observará rapidamente que você pode fazer coisas como:

```js
var s = new String( "Hello World!" );

console.log( s.toString() ); // "Hello World!"
```

*É* verdade que cada um desses nativos pode ser usado como um construtor. Mas o que está sendo construído pode ser diferente do que você possa pensar.

```js
var a = new String( "abc" );

typeof a; // "object" ... e não "String"

a instanceof String; // true

Object.prototype.toString.call( a ); // "[object String]"
```

O resultado da criação de valores usando o construtor (`new String("abc")`) é um objeto que envolve o valor primitivo (`"abc"`).

É importante destacar que, `typeof` mostra que esses objetos não são seus próprios *tipos* especiais, mas mais adequadamente eles são subtipos do tipo `object`.

Esse objeto envoltório pode ainda ser observado com:

```js
console.log( a );
```

A saída dessa declaração varia dependendo do seu navegador, como os consoles são livres para escolher como eles acharem apropriado para serializar o objeto para a inspeção do desenvolvedor.

**Nota:** No momento da escrita, a última versão do Chrome imprime algo assim: `String {0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"}`. Mas versões antigas do Chrome costumavam imprimir apenas isso: `String {0: "a", 1: "b", 2: "c"}`. A última versão do Firefox imprime atualmente `String ["a","b","c"]`, mas costumava imprimir `"abc"` em itálico, que era clicável para abrir o inspetor de objetos. Claro que esse resultados estão sujeitos à rápida mudança e sua experiência pode variar.

O ponto é, `new String("abc")` cria a um objeto string ao redor de `"abc"`, não apenas o próprio valor primitivo `"abc"`.

## `[[Class]]` interno

Valores que são `typeof` `"object"` (como um array) são adicionalmente marcados com uma propriedade interna `[[Class]]` (pense nisto como uma *class*ificação interna ao invés de classes da tradicional programação orientada a classes). Essa propriedade não pode ser acessada diretamente, mas geralmente pode ser revelada indiretamente por chamar o método `Object.prototype.toString(..)` contra o valor. Por exemplo:

```js
Object.prototype.toString.call( [1,2,3] );			// "[object Array]"

Object.prototype.toString.call( /regex-literal/i );	// "[object RegExp]"
```

Assim para o array no exemplo, o valor `[[Class]]` interno é `"Array"`, e para a expressão regular, é `"RegExp"`. Na maioria dos casos, este valor `[[Class]]` interno corresponde ao construtor nativo (veja abaixo) que é relacionado ao valor, mas nem sempre é o caso.

E quanto aos tipos primitivos? Primeiro, `null` e `undefined`:

```js
Object.prototype.toString.call( null );			// "[object Null]"
Object.prototype.toString.call( undefined );	// "[object Undefined]"
```

Você vai notar de que não há nenhum construtor `Null()` ou `Undefined()`, mas mesmo assim `"Null"` e `"Undefined"` são os valores internos para `[[Class]]` expostos.

Mas para os outros tipos primitivos como `string`, `number`, e `boolean`, outra coisa acontece, que geralmente é chamado de "boxing" (veja a próxima seção "Boxing Wrappers"):

```js
Object.prototype.toString.call( "abc" );	// "[object String]"
Object.prototype.toString.call( 42 );		// "[object Number]"
Object.prototype.toString.call( true );		// "[object Boolean]"
```

Nesse trecho de código, cada um dos tipos primitivos simples são automaticamente encaixotados em seus respectivos objetos que os envolvem, é por isso que `"String"`, `"Number"`, e `"Boolean"` são revelados como seus respectivos valores `[[Class]]` internos.

**Nota:** O comportamento de `toString()` e `[[Class]]` ilustrado aqui mudou um pouco do ES5 para o ES6, mas nós cobrimos esses detalhos no título *ES6 & Além* dessa série.

## Boxing Wrappers

Estes objetos que envolvem os tipos primitos tem um propósito muito importante. Tipos primitivos não tem propriedades ou métodos, então para acessar `.length` ou `.toString()` você precisa que um objeto o envolva. Felizmente, JS automaticamente vai *encaixotar* (também conhecido como envolver) os valores, permitindo esta forma de acesso.

```js
var a = "abc";

a.length; // 3
a.toUpperCase(); // "ABC"
```

Então, se você vai acessar essas propriedade/métodos de suas strings regularmente, como em uma condição `i < a.length` em um loop `for` por exemplo, pode parecer fazer sentido apenas usar a forma de objeto desde o início, para o motor JS não precisar criar implicitamente para você.

Mas acaba que isso é uma péssima ideia. Os navegadores há muito tempo otimizaram o desempenho dos casos mais comuns como `.length`, que significa que o seu programa vai *na verdade ficar mais lento* se você tentar "pré-otimizar" por usar diretamente o objeto (que não é otimizado).

Em geral, não há razão para usar os objetos diretamente. É melhor apenas deixar o boxing acontecer implicitamente quando necessário. Em outras palavras, nunca faça coisas como `new String("abc")`, `new Number(42)`, etc -- sempre prefira usar os valores primitivos literais `"abc"` e `42`.

### Pegadinhas do Object Wrapper

Há algumas pegadinhas quando se usa object wrappers diretamente que você precisa estar ciente caso algum dia *escolha* utilizá-los.

Por exemplo, considerando valores `Boolean` embrulhados:

```js
var a = new Boolean( false );

if (!a) {
	console.log( "Oops" ); // nunca executa
}
```

O problema é que você criou um objeto *wrapper* ao redor do valor `false`, mas objetos são "truthy" por natureza (ver Capítulo 4), portanto o uso deste objeto provê um comportamento oposto ao obtido com a utilização direta do valor `false`, o que de fato representa o oposto do que normalmente se esperaria.

Se você quiser embrulhar um valor primitivo, você pode usar a função `Object(..)` (sem a palavra-chave `new`):

```js
var a = "abc";
var b = new String( a );
var c = Object( a );

typeof a; // "string"
typeof b; // "object"
typeof c; // "object"

b instanceof String; // true
c instanceof String; // true

Object.prototype.toString.call( b ); // "[object String]"
Object.prototype.toString.call( c ); // "[object String]"
```

De novo, ao usar o object wrapper embrulhado diretamente (como `b` e `c` acima) é geralmente desencorajado, mas pode haver raras ocasiões em que eles podem ser úteis.

## Unboxing

Se você tem um objeto *wrapper* e quer obter o seu valor primitivo, pode-se usar o método `valueOf()`:

```js
var a = new String( "abc" );
var b = new Number( 42 );
var c = new Boolean( true );

a.valueOf(); // "abc"
b.valueOf(); // 42
c.valueOf(); // true
```

O *Unboxing* também pode ocorrer implicitamente, quando o objeto *wrapper* é usado de uma forma que requer o valor primitivo. Este processo (coerção) vai ser abrangido em mais detalhes no Capítulo 4, mas de maneira rápida:

```js
var a = new String( "abc" );
var b = a + ""; // `b` tem o valor primitivo unboxed "abc"

typeof a; // "object"
typeof b; // "string"
```

## Nativos como Construtores

Para valores de  `array`, `object`, `function`, e expressão regular, é quase uma preferência universal usar a forma literal para criar os valores, mas a forma literal cria o mesmo tipo de objeto que o construtor cria (ou seja, não existe valor que não seja *wrapped*).

Da mesma forma como os outros nativos, esses construtores devem ser evitados, a não ser que realmente precise dele, principalmente porque eles introduzem exceções e pegadinhas que você provavelmente não vai *querer* lidar.

### `Array(..)`

```js
var a = new Array( 1, 2, 3 );
a; // [1, 2, 3]

var b = [1, 2, 3];
b; // [1, 2, 3]
```

**Nota:** O construtor `Array(..)` não exige a palavra-chave `new` na frente dele. Se você a omitir, o construtor vai se comportar da mesma maneira se você tivesse usado a palavra-chave. Portanto, `Array(1,2,3)` tem o mesmo resultado que `new Array(1,2,3)`.

O construtor `Array` tem um comportamento especial quando um único `number` é passado como argumento, em vez de fornecer esse valor como *conteúdo* do array, ele é tomado como um comprimento para "pré-dimensionar o array" (bem, mais ou menos).

Isso é uma ideia terrível. Primeiramente, você acabar usando essa forma de maneira acidental, porque é fácil se esquecer.

Mas mais importante ainda, não existe como pré-dimensionar o array. Em vez disso, você está criando de outra forma um array vazio, mas definindo a propriedade `length` para o valor numérico especificado.

Um array que não tem valores explicitos em suas posições, mas tem uma propriedade `length` que *implica* que as posições existam, é um tipo exótico de estrutura de dado no JS com alguns comportamentos muito confusos e estranhos. A capacidade de criar esses valores vem puramente de antigas, descontinuadas, funcionalidades históricas ("objetos que parecem arrays" como o objeto `arguments`).

**Nota:** Um array com pelo menos uma "posição vazia" é comumente chamado de "array escasso".

Não ajuda nada o fato de que este é mais um exemplo onde cada console de navegador representa tal objeto de uma forma diferente, o que gera mais confusão.

Por exemplo:

```js
var a = new Array( 3 );

a.length; // 3
a;
```

A serialização de `a` no Chrome é (no tempo da escrita): `[ undefined x 3 ]`. **Isso é realmente triste**. O que implica que há três valores `undefined` nas posições deste array, quando na verdade as posições não existem (também chamado de "posições vazias" -- também um nome ruim!).

Para visualizar as diferenças, tente isto:

```js
var a = new Array( 3 );
var b = [ undefined, undefined, undefined ];
var c = [];
c.length = 3;

a;
b;
c;
```

**Nota:** Como você pode ver com o `c` deste exemplo, posições vazias no array podem acontecer após a criação do array. Mudando o `length` de um array para um valor acima do número de slots definidos, você implicitamente introduz posições vazias. Na verdade, você poderia até chamar `delete b[1]` no trecho de código acima, o que introduziria uma posição vazia no meio de `b`.

Para `b` (no Chrome, atualmente), você vai encontrar `[ undefined, undefined, undefined ]` como serializalção, ao contrário de `[ undefined x 3 ]` para `a` e `c`. Confuso? Sim, assim como todo mundo.

Pior do que isso, no momento da escrita, Firefox imprime `[ , , , ]` para `a` e `c`. Você percebeu por que isso é tão confuso? Olhe mais de perto. Três vírgulas implica quatro slots, e não três slots como esperávamos.

**O quê!?** Firefox coloca uma `,` extra no final da serialização aqui porque no ES5, vírgulas no final de listas (valores de array, lista de propriedades, etc.) são permitidos (e então removidos e ignorados). Então se você colocar um valor `[ , , , ]` no seu programa ou no console, você na verdade obtêm o valor implícito `[ , , ]` (que é, um array com três slots vazios). Essa escolha, apesar de confusa de se ler, é defendida por copiar e colar com precisão o comportamento.

Se você está sacudindo a sua cabeça ou rolando os seus olho agora, você não está sozinho!

Infelizmente, fica pior. Mais do que apenas uam saída confusa no console, `a` e `b` do trecho de código acima na verdade se comportam da mesma maneira em alguns casos **mas diferente em outros**:

```js
a.join( "-" ); // "--"
b.join( "-" ); // "--"

a.map(function(v,i){ return i; }); // [ undefined x 3 ]
b.map(function(v,i){ return i; }); // [ 0, 1, 2 ]
```

**Ugh.**

A chamada de `a.map(..)` *falha* por que os slots na verdade não existem, então `map(..)` não tem nada sobre o que iterar. `join(..)` funciona diferente. Basicamente, nós podemos pensar que este método é implementado de uma forma parecida com essa:

```js
function fakeJoin(arr,connector) {
	var str = "";
	for (var i = 0; i < arr.length; i++) {
		if (i > 0) {
			str += connector;
		}
		if (arr[i] !== undefined) {
			str += arr[i];
		}
	}
	return str;
}

var a = new Array( 3 );
fakeJoin( a, "-" ); // "--"
```

Como você pode ver, `join(..)` funciona apenas *supondo* que as posições existem e iterando sobre o valor de `length`. Seja o que for que `map(..)` faz internamente, ele (aparentemente) não faz essa suposição, por isso os resultados das "posições vazias" é inesperado e propensos à falhas.

Portanto, se você quer *na realidade* criar um array de valores `undefined` reais (não apenas "posições vazias"), como você pode fazer isso (além de manualmente)?

```js
var a = Array.apply( null, { length: 3 } );
a; // [ undefined, undefined, undefined ]
```

Confuso? Sim. Assim é mais ou menos como funciona.

`apply(..)` é um utilitário disponível a todas as funções, que chama a função usada mas de uma maneira especial.

O primeiro argumento é o objeto `this` (abordado no título *this & Prototipagem de Objetos* desta série), que não nos interessa aqui, então nós definimos ele para `null`. O segundo argumento deveria ser um array (ou algo *parecido com* um array -- também conhecido como um "array-like object"). O conteúdo deste "array" é "espalhado" como argumento da função em questão.

Então, `Array.apply(..)` está chamando a função `Array(..)` e espalhando os valores (do objeto `{ length: 3 }`) como seu argumento.

Dentro do `apply(..)`, nós podemos imaginar outro loop `for` (da mesma forma que o `join(..)` acima) que vai de `0` até, mas não inclui, `length` (`3` em nosso caso).

Para cada índice, ele busca aquela chave do objeto. Então se o parâmetro objeto-array for chamado de `arr` internamente dentro da função `apply(..)`, o acesso à propriedade seria efetivamente `arr[0]`, `arr[1]`, e `arr[2]`. Claro que nenhuma dessa propriedades existem no objeto `{ length: 3 }`, por isso todos os três acesso retornariam o valor `undefined`.

Em outras palavras, acaba-se chamando `Array(..)` basicamente dessa forma: `Array(undefined,undefined,undefined)`, que é como nós acabamos com um array preenchido com valores `undefined`, e não apenas com aquelas (loucas) posições vazias.

Enquanto `Array.apply( null, { length: 3 } )` é uma forma estranha e verbosa de criar um array preenchido com valores `undefined`, é muito melhor e mais confiável do que se consegue com as autodestrutivas posições vazias de `Array(3)`.

Concluindo: **nunca, sob nenhuma circunstância**, você deve intencionalmente criar e usar esse arrays com exóticos posições vazias. Apenas não faça isso. Eles são esquisitos.

### `Object(..)`, `Function(..)`, e `RegExp(..)`

Os construtores `Object(..)`/`Function(..)`/`RegExp(..)` também são geralmente opcionais (e portanto devem ser evitados a menos que sejam especificamente chamados):

```js
var c = new Object();
c.foo = "bar";
c; // { foo: "bar" }

var d = { foo: "bar" };
d; // { foo: "bar" }

var e = new Function( "a", "return a * 2;" );
var f = function(a) { return a * 2; };
function g(a) { return a * 2; }

var h = new RegExp( "^a*b+", "g" );
var i = /^a*b+/g;
```

Não há praticamente nenhuma razão para usar o construtor `new Object()`, especialmente porque ele obriga que você adicione as propriedades uma por uma em vez de todas de uma vez usando objetos literais.

O construtor `Function` é útil apenas em raras ocasiões, onde você precisa definir dinamicamente os parâmetros ou o corpo de uma função. **Apenas não trate `Function(..)` como uma alternativa ao `eval(..)`.** Você quase nunca vai precisar definir uma função dinamicamente dessa maneira.

Expressões regulares definidas na forma literal (`/^a*b+/g`) são fortemente preferidas, não apenas pela sintaxe mais simples mas também por razões de performance -- o motor JS pré-compila e faz o cache antes da execução do código. Diferente dos outros construtores que nós vimos até agora, `RegExp(..)` tem alguma utilidade: definir dinamicamente o padrão da expressão regular.

```js
var name = "Kyle";
var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );

var matches = someText.match( namePattern );
```

Esse tipo de cenário realmente ocorre em programas JS de tempos em tempos, então você precisa usar a forma `new RegExp("pattern","flags")`.

### `Date(..)` e `Error(..)`

Os construtorres `Date(..)` e `Error(..)` são muito mais úteis que os outros nativos, porque não existe uma forma literal para nenhum dos dois.

Para criar um objeto de data, você precisa usar `new Date()`. O construtor `Date(..)` aceita argumentos opcionais para especificar a data/tempo a ser usado, mas se omitidos, a data/tempo atual é assumido.

A principal razão para contruir um objeto de data é obter o timestamp atual (o número de milisegundos decorridos desde 1 de janeiro de 1970). Você pode fazer isso por chamar `getTime()` em uma instância do objeto de data.

Mas uma forma mais fácil de fazer isso é apenas chamar a função estática definida no ES5: `Date.now()`. E para fazer um polyfill para pre-ES5 é bem fácil:

```js
if (!Date.now) {
	Date.now = function(){
		return (new Date()).getTime();
	};
}
```

**Nota:** Se você chamar `Date()` sem `new`, irá receber uma string representando a data/tempo no momento. A forma exata dessa representação não é definida na especificação da linguagem, mas os navegadores tendem a fazer algo próximo a: `"Fri Jul 18 2014 00:31:02 GMT-0500 (CDT)"`.

O construtor `Error(..)` (muito parecido com `Array()` acima) se comporta da mesma maneira com a palavra-chave `new` estando presente ou não.

A principal razão para criar um objeto de erro é que ele captura o contexto da pilha de execução para dentro do objeto (na maioria dos motores JS é revelado, após construído, em uma propriedade somente leitura chamada `.stack`). O contexto da pilha de execução inclui o *call-stack* de funções e o número da linha onde o objeto de erro foi criado, o que facilita o *debugging* do erro.

Normalmente você usuaria um objeto de erro com o operador `throw`:

```js
function foo(x) {
	if (!x) {
		throw new Error( "x wasn't provided" );
	}
	// ..
}
```

Os objetos de error geralmente tem pelo menos uma propriedade `message`, e as vezes outras propriedades (que você deve tratar como somente leitura), como `type`. No entanto, além de inspecionar a propriedade `stack` mencionada acima, normalmente é melhor chamar `toString()` no objeto de erro (explicitamente ou implicitamente através da coerção -- veja o Capítulo 4) para uma mensagem de erro formatada de maneira amigável.

**Dica:** Técnicamente, em adição ao nativo `Error(..)` geral, há vários outros nativos para erros específicos: `EvalError(..)`, `RangeError(..)`, `ReferenceError(..)`, `SyntaxError(..)`, `TypeError(..)`, e `URIError(..)`. Mas é bem raro usar manualmente esse nativos de erros específicos. Eles são automaticamente usados se o seu programa sofre com um erro real (como referenciar uma variável que não foi declarado e receber um `ReferenceError`).

### `Symbol(..)`

No ES6 um novo tipo primitivo foi adicionado, chamado "Symbol". Symbols são valores únicos (não é estritamente garantido!) que podem ser usados como propriedade em objetos com pouco medo de qualquer colisão. Eles são primeiramente projetados para comportamentos nativos de ES6 construtores, mas você pode definir os seus próprios symbols.

Symbols podem ser usados como nomes de propriedades, mas você não pode ver ou acessar o valor de um symbol de seu programa, nem do console de desenvolvedor. Se você avaliar um symbol no console de desenvolvedor, o que é mostrado parece com `Symbol(Symbol.create)`, por exemplo.

Há vários symbols pré-definidos no ES6, acessados como propriedades estáticas do objeto função `Symbol`, como `Symbol.create`, `Symbol.iterator`, etc. Para usar eles, faça algo como:

```js
obj[Symbol.iterator] = function(){ /*..*/ };
```

Para definir os seus próprios symbols customizados, use o nativo `Symbol(..)`. O "construtor" nativo ``Symbol(..)` é único porque você não pode usar `new` com ele, se você fizer isso um erro será gerado.

```js
var mysym = Symbol( "my own symbol" );
mysym;				// Symbol(my own symbol)
mysym.toString();	// "Symbol(my own symbol)"
typeof mysym; 		// "symbol"

var a = { };
a[mysym] = "foobar";

Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]
```

Apesar de symbols não serem realmente privados (`Object.getOwnPropertySymbols(..)` reflete no objeto e revela os symbols de forma bastante pública), usar eles para criar propriedades privadas é possivelmente seu principal caso de uso. Para a maioria dos desenvolvedores, eles podem tomar o lugar de nomes de propriedade com `_` prefixo underscore, que é quase sempre uma convenção para dizer, "hey, esta é uma propriedade privada/especial/interna, então não mexa nela!"

**Nota:** `Symbol`s *não* são `objects`s, eles são simples primitivos escalares.

### Prototypes Nativos

Cada um dos construtores nativos têm seu próprio objeto `.prototype` -- `Array.prototype`, `String.prototype`, etc.

Estes objetos contém comportamento únicos para seu subtipo de objeto específico.

Por exemplo, todos os objetos strings, e por extensão (via boxing) `string` primitivos, têm acesso ao comportamento padrão como métodos definidos no objeto `String.prototype`.

**Nota:** Por convenção, `String.prototype.XYZ` é encurtado para `String#XYZ`, e da mesma todos os outros `.prototype`s.

* `String#indexOf(..)`: encontra em uma string a posição de outra substring
* `String#charAt(..)`: acessa o caractere em uma posição na string
* `String#substr(..)`, `String#substring(..)`, e `String#slice(..)`: extrai uma parte da string como uma nova string
* `String#toUpperCase()` e `String#toLowerCase()`: cria uma nova string a convertendo para maiúscula ou minúscula
* `String#trim()`: cria uma nova string sem espaços em branco no início e no fim

Nenhum desses métodos modifica a string *existente*. Modificações (como conversão de caixa ou remoção de espaços) criam um novo valor a partir do valor existente.

Em virtudade da delegação de prototype (veja o título *this & Prototipagem de Objetos* nesta série), qualquer string pode acessar esses métodos.

```js
var a = " abc ";

a.indexOf( "c" ); // 3
a.toUpperCase(); // " ABC "
a.trim(); // "abc"
```

Os prototypes dos outros construtores contêm comportamentos apropriados a seus tipos, como `Number#toFixed(..)` (converte para string um número com uma quantidade fixa de dígitos decimais) e `Array#concat(..)` (funde arrays). Todas as funções tem acessa a `apply(..)`, `call(..)`, e `bind(..)` porque `Function.prototype` define esses métodos.

Mas, alguns desse prototypes nativos não são *apenas* simples objetos:

```js
typeof Function.prototype;			// "function"
Function.prototype();				// é uma função vazia!

RegExp.prototype.toString();		// "/(?:)/" -- regex vazia
"abc".match( RegExp.prototype );	// [""]
```

Uma ideia particularmente ruim, você pode ainda modificar esses prototypes nativos (não apenas adicionar propriedades como você já sabe):

```js
Array.isArray( Array.prototype );	// true
Array.prototype.push( 1, 2, 3 );	// 3
Array.prototype;					// [1,2,3]

// não deixe desta forma ou coisas estranhas podem acontecer!
// redefina `Array.prototype` para vazio
Array.prototype.length = 0;
```

Como você pode ver, `Function.prototype` é uma função, `RegExp.prototype` é uma expressão regular, e `Array.prototype` é um array. Interessante e legal, huh?

#### Prototypes como valor padrão (default)

Sendo `Function.prototype` uma função vazia, `RegExp.prototype` uma regex "vazia" (não coincidente), e `Array.prototype` um array vazio, fazem deles um bom valor "padrão" para atribuir a variáveis se essas variáveis ainda não têm um valor com o tipo apropriado.

Por exemplo:

```js
function isThisCool(vals,fn,rx) {
	vals = vals || Array.prototype;
	fn = fn || Function.prototype;
	rx = rx || RegExp.prototype;

	return rx.test(
		vals.map( fn ).join( "" )
	);
}

isThisCool();		// true

isThisCool(
	["a","b","c"],
	function(v){ return v.toUpperCase(); },
	/D/
);					// false
```

**Nota:** A partir do ES6, não precisamos mais usar o truque `vals = vals || ..` para definir valores padrões (veja o Capítulo 4), porque valores padrões podem ser definidos através de uma sintaxe nativa na declaração de funções (veja o Capítulo 5).

Um pequeno benefício dessa abordagem é que os `.prototype`s já estão criados, assim são criados apenas uma vez. Em contraste com isso, usar `[]`, `function(){}`, e `/(?:)/` para definir esses padrões (provavelmente, dependendo da implementação do motor) os valores serão recriados (e possivelmente adicionados à garbage-collection depois) para *cada chamada* de `isThisCool(..)`. O que pode resultar em um diperdício de memória/CPU.

Também, tenha bastante cuidado em não usar `Array.prototype` como um valor padrão **que irá ser modificado depois**. Neste exemplo, `vals` é somente leitura, mas se fosse modificar `vals`, você estaria na verdade modificando o próprio `Array.prototype`, o que levaria às pegadinha mencionadas a pouco!

**Nota:** Enquanto nós estamos mostrando esses prototypes nativos e algumas de suas utilidades, seja cauteloso ao confiar neles e ainda mais prudente se for modificá-los. Veja o Apêncice A "Prototypes Nativos" para maior discussão.

## Revisão

O Javascript disponibiliza *object wrappers* ao redor de valores primitivos, conhecidos como nativos (`String`, `Number`, `Boolean`, etc). Estes *object wrappers* dão aos valores acesso aos comportamentos apropriados para o subtipo de cada objeto (`String#trim()` e `Array#concat(..)`).

Se tem um simples valor primitivo como `"abc"` e você acessa a sua propriedade `length` ou outro método de `String.prototype`, o JS automaticamente "boxes" o valor (envolve ele no se respectivo *object wrapper*) para que o acesso da propriedade/método para ser cumprido.
