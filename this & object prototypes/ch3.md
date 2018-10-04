# You Don't Know JS: *this* & Protótipos de Objeto

# Capítulo 3: Objetos


Nos capítulos 1 e 2, nós explicamos como a ligação do `this` aponta para vários objetos dependendo de onde é feita a chamada da função. Mas o que exatamente são objetos e por que nós precisamos salientá-los? Nós aprenderemos sobre objetos, em detalhes, nesse capítulo.

## Sintaxe

Objetos possuem duas formas: a forma declarativa (literal) e a forma construída.

A sintaxe literal de um objeto se parece com isso:

```js
var myObj = {
	key: value
	// ...
};
```

A forma construída se parece com isso:

```js
var myObj = new Object();
myObj.key = value;
```

A forma construída e a forma literal resultam exatamente no mesmo tipo de objeto. A única diferença é que você pode adicionar um ou mais pares chave/valor na declaração literal, enquanto que com objetos de forma construída, você tem que adicionar as propriedades uma por uma.

**Lembrete:** É extremamente raro o uso da "forma construída" para a criação de objetos como foi mostrado. Você praticamente sempre irá preferir usar a sintaxe da forma literal. O mesmo acontece com a maioria dos objetos nativos (veja abaixo).

## Tipo

Objetos são o bloco de construção geral no qual muito do JS é construído. Eles são um do 6 tipos primários (chamados "tipos de linguagem" na especificação) em JS:

* `string`
* `number`
* `boolean`
* `null`
* `undefined`
* `object`


Note que os *primitivos simples* (`string`, `number`, `boolean`, `null`, e `undefined`) **não** são por si só `objetcs`. `null` é algumas vezes referido como um tipo de objeto, mas esse equívoco surge a partir de um bug na linguagem que faz com que `typeof null` retorne a string `"object"` incorretamente (e de modo confuso). De fato, `null` é o seu próprio tipo primitivo.

**Há uma frequente distorção de que "Tudo em JavaScript é um objeto. Isso claramente não é verdade".**

Por outro lado, há alguns subtipos de objeto especiais, os quais podemos referir como *primtivos complexos*

`function` é um subtipo de objeto (tecnicamente, um "objeto que pode ser chamado"). Funções em JS são consideradas como "primeira classe" que são basicamente objetos normais (com a adição de semântica de comportamento de algo que pode ser chamado), e então elas podem ser manipuladas como qualquer outro objeto simples.

Arrays também são uma forma de objetos, com comportamento extra. A organização de conteúdos nos arrays é uma pouco mais estruturada do que em objetos gerais.

### Objetos nativos

Existem diversos outros subtipos de objetos, normalmente referidos como objetos nativos. Para alguns deles, seus nomes parecem sugerir que eles estão diretamente relacionados, em contrapartida, a seus primitivos simples, mas na verdade, o relacionamento deles é mais complicado, o qual iremos explorar em breve.

* `String`
* `Number`
* `Boolean`
* `Object`
* `Function`
* `Array`
* `Date`
* `RegExp`
* `Error`

Esses nativos aparentam ser tipos reais, até mesmo classes, se você considerar a similaridade com outras linguagens como a classe `String` do Java.

Mas no JS existem, na verdade, apenas funções nativas. Cada uma dessas funções nativas pode ser usada como um construtor (que é uma chamada de função com o operador `new` -- veja o Capítulo 2), com o resultado sendo novamente um objeto *construído* do subtipo em questão.

Por exemplo:

```js
var strPrimitive = "I am a string";
typeof strPrimitive;							// "string"
strPrimitive instanceof String;					// false

var strObject = new String( "I am a string" );
typeof strObject; 								// "object"
strObject instanceof String;					// true

// inspeciona o subtipo de 'object'
Object.prototype.toString.call( strObject );	// [object String]
```

Nós veremos, em detalhes, mais adiante nesse capítulo exatamente como o trecho `Object.prototype.toString...` funciona, mas brevemente podemos inspecionar o interior o subtipo pegando emprestado o padrão base do método `toString()` e você pode ver que ele revela que `strObject` é um objeto que foi de fato criado pelo construtor de `String`.

O valor primitivo `"I am a string"` não é um objeto, mas sim um primitivo literal e valor imutável. Para realizar operações com ele, tais como checar seu comprimento, acessar o conteúdo de caracteres individuais etc, um objeto `String` é necessário.

Por sorte, a linguagem automaticamente converte um primitivo `"string"` para um objeto `String` quando necessário, o que significa que você quase nunca precisa explicitamente criar a forma de objeto. É **fortemente preferível** pela maior parte da comunidade de JS usar o forma literal para um valor, quando possível, em vez da forma de objeto construído.

Considere:

```js
var strPrimitive = "I am a string";

console.log( strPrimitive.length );			// 13

console.log( strPrimitive.charAt( 3 ) );	// "m"
```

Em ambos os casos, podemos chamar uma propriedade ou método de uma string primitiva, e o *Motor JS* automaticamente converte a mesma para um objeto `String` para que o acesso a propriedade/método funcione.

O mesmo tipo de comportamento acontece entre o número primitivo literal `42` e o invólucro do objeto `new Number(42)` para o uso de métodos como `42.359.toFixed(2)`. Da mesma maneira para objetos `Boolean` de primitivos `"boolean"`.

`null` e `undefined` não tem formato de invólucro de objeto, apenas valores primitivos. De maneira oposta, valores de `Date` só podem ser criados a partir da forma de objetos construídos, uma vez que eles não possuem uma forma literal.

`Object`s, `Array`s, `Function`s, e `RegExp`s (expressões regulares) são todos objetos, independente se a forma literal ou construída é usada. A forma construída oferece, em alguns casos, mais opções na criação do que a forma literal. Uma vez que objetos são criados das duas formas, a forma literal (mais simples) é quase que universalmente preferida. **Apenas use a forma construída se você precisar de opções extras.**

Objetos `Error` são raramente criados explicitamente em código, mas geralmente são criados automaticamente quando exceções são lançadas. Eles pode ser criados com a forma de instanciação `new Error(..)`, mas muitas vezes é desnecessário.

## Conteúdos

Como mencionamos anteriormente, o conteúdo de um objeto consiste de valores (qualquer tipo) armazenados em locais específicos, o qual chamamos de propriedades.

É importante notar que quando dizemos "conteúdos" implica que esses valores são *na verdade* armazenados dentro do objeto, que é meramente uma aparência. O motor JS armazena valores de maneiras dependentes de implementação e pode muito bem armazená-los *em* algum container de objeto. O que *é* armazenado no container são nomes de propriedade, que funcionam como ponteiros (tecnicamente, *referências*) para onde os valores são armazenados.

Considere:

```js
var myObject = {
	a: 2
};

myObject.a;		// 2

myObject["a"];	// 2
```

Para acessar o valor no *local* `a` em `myObject`, precisamos usar o operador `.` ou o operator `[ ]`. A sintaxe `.a` normalmente se refere ao acesso à "propriedade", enquando que a sintaxe `["a"]` é geralmente referente ao acesso a "chave". Na realidade, as duas formas acessam o mesmo *local*, e retornarão o mesmo valor, `2`, então os termos podem ser usados indiferetemente. Usaremos o termo mais comum, "acesso à propriedade" de agora em diante.

A principal diferença entre as duas sintaxes é que o operador `.` requer um nome de propriedade `Identifier` compatível logo após ele, enquanto a sintaxe `[".."]` pode aceitar basicamente qualquer com string compatível com UTF-8/unicode como o nome da propriedade. Para referenciar uma propriedade de nome `Super-Fun!`, por exemplo, você teria que usar o sintaxe de acesso `["Super-Fun!"]`, pois `Super-Fun!` não é um nome de propriedade `Identifier` válido.

Além disso, um vez que a sintaxe `[".."]` usa **valor** de string para especificar a localização, significa que o programa pode, programaticamente, construir um valor de string, tal como:

```js
var wantA = true;
var myObject = {
	a: 2
};

var idx;

if (wantA) {
	idx = "a";
}

// depois

console.log( myObject[idx] ); // 2
```

Em objetos, os nomes de propriedade são  **sempre** strings. Se você usa qualquer outro valor além de um `string` (primitivo) como propriedade, ele será convertido para string primeiro. Isso inclue até mesmo números, que são normalmente usados como índices de array, então tenha cuidado para não confundir o uso de números entre objetos e arrays.

```js
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"];				// "foo"
myObject["3"];					// "bar"
myObject["[object Object]"];	// "baz"
```

### Nomes de propriedade computados

A sintaxe de acesso às propriedades de `myObject[..]` que acabamos de descrever é útil se você precisar usar um valor de expressão computado *como* o nome da chave, tal como `myObject[prefix + name]`. Mas não é realmente útil quando se declara objetos usando a sintaxe de objeto-literal.

ES6 adiciona *nomes de propriedade computados*, onde você pode especificar uma expressão, cercado por um par de `[ ]`, na posição do nome-da-chave de uma declaração de objeto-literal.

```js
var prefix = "foo";

var myObject = {
	[prefix + "bar"]: "hello",
	[prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

O uso mais comum de *nomes de propriedade computados* serão provavelmente com `Símbolo` no ES6, o qual não abordaremos em detalhes nesse livro. Em resumo, eles são um novo tipo de dado primitivo que tem um valor imprevisível (tecnicamente um valor de tipo `string`). Você será fortemente desencorajado a trabalhar com o *valor real* de um `Symbol` (que pode, teoricamente, ser diferente entre diferentes motores JS), portanto o nome do `Symbol`, como `Symbol.Something` (apenas um nome inventado!), será o que você vai usar:

```js
var myObject = {
	[Symbol.Something]: "hello world"
};
```

### Propriedade vs. Método

Alguns desenvolvedores gostam de fazer uma distinção em relação ao acesso de propriedade de um objeto, se o valor acessado for uma função. Porque é tentador pensar em uma função como sendo *pertencente* a um objeto e, em outras linguagens, funções que pertencem a objetos (conhecidos como "classes") são referidos como "métodos", não é incomum ouvir "acesso de método" como o oposto ao "acesso de propriedade".

**A especificação faz a mesma distinção**, curiosamente.

Tecnicamente, funções nunca "pertencem" a objetos, então falando que uma função que apenas foi acessada em uma referência de objeto é automaticamente um "método" que parece um pouco com uma extensão de semântica.

*É* verdade que algumas funções tem referências `this`, e que *às vezes* essas referências `this` se referem a uma referência de objeto no call-site. Mas esse uso realmente não faz uma função ("método") ter algo mais que qualquer outra função, pois o `this` é vinculado dinâmicamente em tempo de execução, no call-site, e assim seu relacionamento com o objeto é indireto, na melhor das hipóteses.

Toda vez que você acessa uma propriedade de um objeto, isso é considerado um **acesso à propriedade**, independente do tipo do valor que é retornado. Se *por acaso* você obter uma função a partir do acesso à propriedade, ela não é magicamente um "método" nesse momento. Não há nada especial (fora o possível _bind_ implícito do `this` como foi explicado anteriormente) em uma função que é obtida a partir de um acesso à propriedade.

Por exemplo:

```js
function foo() {
	console.log( "foo" );
}

var someFoo = foo; // variável que referencia `foo`

var myObject = {
	someFoo: foo
};

foo;				// função foo(){..}

someFoo;			// função foo(){..}

myObject.someFoo;	// função foo(){..}
```

`someFoo` e `myObject.someFoo` são apenas duas referências separadas para a mesma função, e nenhuma das duas implica em algo sobre a função ser especial ou "pertencente" a outro objeto. Se `foo()` acima foi definido tendo uma referência `this`, o `myObject.someFoo` *_binding_ implícito* será a **única** diferença observável entre as duas referências. Nenhuma das referências realmente faz sentido em ser chamada de "método".

**Talvez alguém pudesse argumentar** que uma função *se torna um método* não no momento de sua definição, mas durante tempo de execução, dependendo de como é chamado em seu call-site (com um contexto de referência de objeto ou não -- veja Capítulo 2 para mais detalhes). Mesmo assim essa interpretação é pouco provável.

Provavelmente, a conclusão mais segura é que "função" e "método" são permutáveis em JavaScript.

**Nota:** ES6 adiciona uma referência `super`, que é tipicamente usada com `class` (veja Apêndice A). A maneira que `super` se comporta (_binding_ estático em vez de _binding_ tardio como `this`) proporciona mais peso na ideia que uma função que é vinculada ao `super` em algum lugar é mais "método" que "função". Mas mais uma vez, são apenas detalhes sutis de semântica (e mecânica).

Mesmo quando você declara uma expressão de função como parte do objeto-literal, a função não *pertence* magicamente ao objeto -- apenas várias referências ao mesmo objeto de função.

```js
var myObject = {
	foo: function foo() {
		console.log( "foo" );
	}
};

var someFoo = myObject.foo;

someFoo;		// function foo(){..}

myObject.foo;	// function foo(){..}
```

**Nota:** No capítulo 6, nós veremos uma forma abreviada no ES6 para a sintaxe de declaração de `foo: function foo(){ .. }` em nosso objeto-literal.

### Arrays

Arrays também usam a forma de acesso `[ ]`, mas como mencionado acima, eles tem uma organização um pouco mais estruturada com relação a como e onde valores são armazenados (ainda que não tenha restrição de *tipo* de valores que são armazenados). Arrays assumem *indexação numérica*, onde valores são armazenados em locais, normalmente chamados de *índices*, em inteiros não-negativos, tal como `0` e `42`.

```js
var myArray = [ "foo", 42, "bar" ];

myArray.length;		// 3

myArray[0];			// "foo"

myArray[2];			// "bar"
```

Arrays *são* objetos, então mesmo que cada índice seja um inteiro positivo, você *também* pode adicionar propriedades no array.

```js
var myArray = [ "foo", 42, "bar" ];

myArray.baz = "baz";

myArray.length;	// 3

myArray.baz;	// "baz"
```

Note que adicionando propriedades nomeadas (independente da sintaxe do operador `.` ou `[ ]`) não muda o `length` informado do array.

Você *poderia* usar um array como um objeto chave/valor simples, e nunca adicionar qualquer índice numérico, mas essa é uma má ideia porque arrays tem um comportamento e otimizações específicas para o uso que eles são destinados, e da mesma forma acontence com objetos simples. Use objetos para armazenar pares de chave/valor e arrays para armazenar em índices numéricos.

**Tenha cuidado:** Se você tentar adicionar uma propriedade em um array, mas o nome da propriedade *parecer* um número, pode acontecer da mesma ser interpretada como um índice numérico (consequentemente modificando o conteúdo do array):

```js
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";

myArray.length;	// 4

myArray[3];		// "baz"
```

### Duplicando Objetos

Um dos recursos mais requisitados frequentemente por novos desenvolvedores JavaScript é como duplicar um objeto. Parece como se devesse existir um método nativo `copy()`, certo? Acontece que é um pouco mais complicado que isso, porque não é totalmente claro, por padrão, qual deveria ser o algoritmo para a duplicação.

Por exemplo, considere esse objeto:

```js
function anotherFunction() { /*..*/ }

var anotherObject = {
	c: true
};

var anotherArray = [];

var myObject = {
	a: 2,
	b: anotherObject,	// referência, não uma cópia!
	c: anotherArray,	// outra referência!
	d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```

O que exatamente deveria ser a representação de um *copy* de `myObject`?

Primeiramente, nós devemos responder se deveria ser uma cópia *rasa* ou *profunda*? Uma cópia rasa terminaria com `a` no novo objeto como uma cópia do valor `2`, mas as propriedades `b`, `c`, e `d` são apenas referências para o objeto original. Uma cópia profunda duplicaria não apenas `myObject`, mas `anotherObject` e `anotherArray`. Mas temos problemas nos quais `anotherArray` possue referências para `anotherObject` e `myObject`, então *esses* também devem ser duplicados em vez de preservarem referências. Agora temos um problema de duplicação circular infinita por causa da referência circular.

Devemos detectar uma referência circular e apenas quebrar a transversal circular (deixando o elemento profundo não completamente duplicado)?

Além disso, não fica realmente claro o que "duplicar" uma função significaria? Existem alguns hacks como retirar a serialização do `toString()` do código fonte de uma função (que varia entre diferentes implementações e nem é confiável em todos os motores JS, dependendo do tipo de função que está sendo inspecionada).

Então como resolvemos todas essas difíceis questões? Vários frameworks JS possuem suas próprias interpretações e decisões. Mas qual delas (se existe alguma) o JS deveria adotar como *o* padrão? Durante um longo tempo, não havia uma resposta clara.

Uma parte da solução é que objetos que são JSON-safe (que é, pode ser serializado para uma string JSON e depois re-transformada em um objeto com a mesma estrutura e valores) podem facilmente ser *duplicados* com:


```js
var newObj = JSON.parse( JSON.stringify( someObj ) );
```

É claro que requer que você assegure que seu objeto é JSON-safe. Em algumas situações, é trivial. Em outras, é insuficiente.

Ao mesmo tempo, um cópia rasa é completamente compreensível e tem muito menos problemas, então ES6 agora definiu `Object.assign(..)` para essa tarefa. `Object.assign(..)` seleciona um objeto *alvo* como primeiro parâmetro e um ou mais objetos *fonte* como parâmetros subsequentes. Ele itera sobre todos os *enumerable* (veja abaixo), *chaves de propriedade* (**imediatamente presente**) nos objeto(s) *fonte* e copia eles para um *alvo*. Ele também retorna o *alvo*, como pode ver abaixo:


```js
var newObj = Object.assign( {}, myObject );

newObj.a;						// 2
newObj.b === anotherObject;		// true
newObj.c === anotherArray;		// true
newObj.d === anotherFunction;	// true
```

**Lembrete:** Na próxima seção, descrevemos "descritores de propriedade" (características de propriedade) e mostramos o uso de `Object.defineProperty(..)`. Entretanto, a duplicação que ocorre com `Object.assign(..)` é puramente atribuição de estilo `=`, então qualquer característica especial de uma propriedade (como `writable`) em um objeto de origem **não são preservadas** em um objeto de destino.


### Descritores de propriedade

Antes do ES5, a linguagem JavaScript não dava nenhuma forma direta no seu código para inspecionar ou obter qualquer distinção entre as características de propriedades, tais como se a propriedade era somente-leitura ou não.

Mas como no ES5, todas as propriedades são descritas em termos de um **descritor de propriedade**.

Considere esse código:

```js
var myObject = {
	a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```

Como pode ver, o descritor da propriedade (chamado de "descritor de dados", um vez que ele apenas guarda um valor de dado) para nossa propriedade de objeto normal `a` é muito mais que apenas o `valor` de `2`. Ele inclue outras 3 características: `writable`, `enumerable`, and `configurable`.

Enquanto nós podemos ver o que os valores padrão para as características do descritor de propriedade são quando criamos uma propriedade normal, nós podemos usar `Object.defineProperty(..)` para adicionar uma nova propriedade ou modificar uma já existente (se ela for `configurable`!), com as características desejadas.

Por exemplo:

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: true,
	enumerable: true
} );

myObject.a; // 2
```

Usando `defineProperty(..)`, adicionamos uma simples propriedade `a` ao `myObject` de maneira explícita manualmente. Entretanto, geralmente não usaríamos essa forma manual a menos que quiséssemos modificar uma das características do descritor a partir de seu comportamento normal.

#### Gravável

A habilidade de você mudar o valor de uma propriedade é controlada por `writable`.

Considere:

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // não gravável!
	configurable: true,
	enumerable: true
} );

myObject.a = 3;

myObject.a; // 2
```

Como pode ver, nossa modificação do `value` falhou silenciosamente. Se tentarmos em `strict mode`, obtemos um erro:

```js
"use strict";

var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // não gravável!
	configurable: true,
	enumerable: true
} );

myObject.a = 3; // TypeError
```
O `TypeError` diz que não podemos mudar uma propriedade não gravável.

**Lembrete:** Nós discutiremos getters/setters mais adiante, mas resumidamente, você pode observar que `writable:false` significa um valor que não pode ser alterado, equivalente um pouco no caso de quando uma operação de setter não é definida. Na verdade, a não-operação de setter precisaria lançar um `TypeError` quando fosse chamada, para se comportar de maneira similar ao `writable:false`.


#### Configurável

Contanto que uma propriedade seja configurável, podemos modificar sua definição de descritor usando o mesmo método `defineProperty(..)`.

```js
var myObject = {
	a: 2
};

myObject.a = 3;
myObject.a;					// 3

Object.defineProperty( myObject, "a", {
	value: 4,
	writable: true,
	configurable: false,	// não configurável!
	enumerable: true
} );

myObject.a;					// 4
myObject.a = 5;
myObject.a;					// 5

Object.defineProperty( myObject, "a", {
	value: 6,
	writable: true,
	configurable: true,
	enumerable: true
} ); // TypeError
```

A última chamada de `defineProperty(..)` resulta em um TypeError, independente do `strict mode`, se você tentar mudar a definição do descritor de uma propriedade não configurável. Cuidado: como você pode ver, alterando `configurable` para `false` é uma **ação de via única, e não pode ser desfeita!**

**Nota:** Há uma exceção diferenciada para estar ciente: mesmo se a propriedade já está com `configurable:false`, `writable` sempre pode ser alterada de `true` para `false` sem erro, mas não o contrário, de `false` para `true`.

Outra coisa que `configurable:false` previne é a habilidade de usar o operador `delete` para remover uma propriedade existente.

```js
var myObject = {
	a: 2
};

myObject.a;				// 2
delete myObject.a;
myObject.a;				// undefined

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: false,
	enumerable: true
} );

myObject.a;				// 2
delete myObject.a;
myObject.a;				// 2
```

Como pode ver, a última chamada de `delete` falhou (silenciosamente) porque definimos a propriedade `a` como não-configurável.

`delete` não é apenas usado para remover propriedades (que podem ser removidas) de objeto. Se uma propriedade de objeto é a última *referência* que resta para algum objeto/função, e você aplica `delete` nela, isso remove a referência e faz com que o objeto/função sem referência possa ser alocado para _garbage collection_. Mas, *não* é adequado pensar no `delete` como sendo uma ferramenta para liberar memória alocada, como acontece em outras linguagens (a exemplo de C/C++). `delete` é apenas uma operação de remoção de propriedade de objeto -- nada mais.

#### Enumerável

A última característica de descritor que vamos mencionar aqui (existem outras duas, que veremos em breve quando discutirmos getter/setters) é `enumerable`.

O nome provavelmente é óbvio, mas essa característica controla se uma propriedade aparecerá em certas enumerações objeto-propriedade, tal como o laço `for..in`. Configure para `false` para não aparecer em tais enumerações, mesmo que ainda esteja completamente acessível. Configure `true` para mantê-lo presente.

Todas as propriedades normais definidas pelo usuário são padronizadas para `enumerable`, visto que isso é que você geralmente deseja. Mas se você tem uma propriedade especial que queira ocultar da enumeração, configure-a para `enumerable:false`.

Iremos demonstrar enumerabilidade em mais detalhes mais adiante, então lembre-se desse tópico.

#### Imutabilidade

Às vezes é desejável fazer com que propriedades ou objetos não possam ser alterados (por acidente ou intencionalmente). ES5 adiciona suporte para lidar com isso através de diferentes formas.

É importante notar que **todas** essas abordagens criam imutabilidade rasa. O que significa que afeta apenas o objeto e suas características de propriedade direta. Se um objeto tem uma referência para outro objeto (array, objeto, função, etc), o *conteúdo* desse objeto não é afetado e permanece mutável.

```js
myImmutableObject.foo; // [1,2,3]
myImmutableObject.foo.push( 4 );
myImmutableObject.foo; // [1,2,3,4]
```

Nós assumimos nesse trecho de código que `myImmutableObject` já é criado e protegido como imutável. Mas, para proteger também o conteúdo de `myImmutableObject.foo` (que é seu próprio objeto -- array), será necessário fazer com que `foo` seja imutável também, usando um ou mais das seguintes funcionalidades.

**Nota:** Não é extremamente comum criar, em programas JS, objetos imutáveis profundamente enraizados. Casos especiais podem certamente existir, mas tendo como um padrão de projeto geral, se você quiser usar *seal* ou *freeze* em todos os seus objetos, você pode voltar e reconsiderar o design de seu programa visando uma maior resistência a possíveis mudanças nos valores dos objetos.

### Constante de Objeto

Combinando `writable:false` e `configurable:false`, você pode essencialmente criar uma *constante* (não pode ser alterada, redefinida ou removida) como uma propriedade de objeto, como:

```js
var myObject = {};

Object.defineProperty( myObject, "FAVORITE_NUMBER", {
	value: 42,
	writable: false,
	configurable: false
} );
```

### Prevenir Extensões

Se você quiser prevenir que um objeto tenha novas propriedades adicionadas a ele, mantendo apenas o resto das propriedades do objeto, chame `Object.preventExtensions(..)`:

```js
var myObject = {
	a: 2
};

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b; // undefined
```

No `non-strict mode` a criação de `b` falha silenciosamente. No `strict mode` é lançado um `TypeError`.

#### Seal

`Object.seal(..)` cria um objeto "selado", que pega um objeto existente e essencialmente chama `Object.preventExtensions(..)`, mas também marca todas as propriedades existentes como `configurable:false`.

Então, além de você não poder adicionar mais propriedades, também não pode reconfigurar ou deletar qualquer propriedade existente (embora ainda *possa* modificar seu valor).

#### Freeze

`Object.freeze(..)` cria um objeto "congelado", que pega um objeto existente e essencialmente chama `Object.seal(..)`, mas também marca todas as propriedades "acessores de dados" como `writable:false`, com isso os valores das propriedades não podem ser alteradas.

Essa abordagem é o nível mais alto de imutabilidade para uma objeto que você pode atingir, visto que previne qualquer mudança no objeto ou em qualquer de suas propriedades diretas (embora, como mencionado acima, o conteúdo de qualquer objeto referenciado não é afetado).

Você poderia "deep freeze" um objeto através do `Object.freeze(..)`, e então iterar recursivamente sobre todos os objetos que dado objeto referencia (que teriam sido afetados até agora), e chamando `Object.freeze(..)` para eles também. Cuidado, embora isso possa afetar outros objetos (compartilhados) você não tem intenção de afetá-los.

### `[[Get]]`

Há um pequeno, mas importante, detalhe sobre como os acessos às propriedades são realizados.

Considere:

```js
var myObject = {
	a: 2
};

myObject.a; // 2
```

O `myObject.a` é um acesso à propriedade, mas não é *apenas* procurar em `myObject` por uma propriedade de nome `a`, como pode parecer.

De acordo com a especificação, o código acima na verdade realiza uma operação `[[Get]]` (tipo uma chamada de função: `[[Get]]()`) no `myObject`. A operação nativa padrão `[[Get]]` de um objeto inspeciona *primeiro* o objeto à procura de uma propriedade com nome solicitado, se encontrar, retornará o respectivo valor.

Entretanto, o algoritmo de `[[Get]]` define outro importante comportamento caso *não* encontre a propriedade com nome solicitado. Examinaremos no Capítulo 5 o que acontece *em seguida* (passagem pela cadeia de `[[Prototype]]`, caso houver).

Mas um importante resultado dessa operação `[[Get]]` é que se a propriedade solicitada não for encontrada, o valor `undefined` é retornado.

```js
var myObject = {
	a: 2
};

myObject.b; // undefined
```

Esse comportamento é diferente de quando você referencia *variáveis* através de seus nomes de identificadores. Se você referencia uma variável que não pode ser resolvida no escopo léxico aplicável, o resultado não é `undefined` como é para propriedades de objeto, mas em vez disso um `ReferenceError` é lançado.

```js
var myObject = {
	a: undefined
};

myObject.a; // undefined

myObject.b; // undefined
```

Do ponto de vista de *valor*, não há diferença entre essas duas referências -- As duas resultam em `undefined`. Entretanto, por baixo, a operação `[[Get]]`, à primeira vista, realizou uma pouco mais de "trabalho" para a referência `myObject.b` do que para a referência `myObject.a`.

Analisando apenas os resultados do valor, você não consegue diferenciar se uma propriedade existe e mantém um valor explícito `undefined` ou se a propriedade *não* existe e `undefined` foi o valor padrão retornado após `[[Get]]` falhar em retornar algo explicitamente. Contudo, nós veremos em breve como podemos diferenciar esses dois cenários.

### `[[Put]]`

Uma vez que há uma operação `[[Get]]` definida internamente para obter o valor de uma propriedade, deveria ser óbvio que há também uma operação `[[Put]]` padrão.

Pode ser tentador pensar que a atribuição de uma propriedade de um objeto iria apenas invocar `[[Put]]` para definir ou criar a propriedade do objeto em questão. Mas esse caso possui algumas especificidades a mais.

Quando invocamos `[[Put]]`, o modo como ele se comporta muda baseado em alguns fatores, incluindo (o mais impactante) se a propriedade já está presente no objeto ou não.

Se a propriedade estiver presente, o algoritmo de `[[Put]]` irá checar grosseiramente:

1. A propriedade é um descritor de acessor? (veja a seção abaixo sobre "Getters & Setters") **Se sim, chame o setter, caso exista um**
2. A propriedade é um descritor de dado com `writable` definido como `false`? **Se sim, falha silenciosamente no `non-strict mode` ou lança `TypeError` no `strict mode`.**
3. Caso contrário, define o valor para a propriedade existente normalmente.

Se a propriedade ainda não estiver presente no objeto em questão, a operação `[[Put]]` é ainda mais diferenciada e complexa. Nós iremos rever esse caso com mais clareza no Capítulo 5 quando discutirmos `[[Prototype]]`.

### Getters & Setters

As operações padrão `[[Put]]` e `[[Get]]` de objetos controlam como valores são definidos em propriedades novas ou já existentes, ou obtidos de propriedades existentes, respectivamente.

**Nota:** Utilizando futuros/avançados recursos da linguagem, pode ser possível sobrescrever as operações padrão `[[Get]]` or `[[Put]]` para um objeto inteiro (não apenas por propriedade). Isso está além do escopo da nossa discussão nesse livro, mas será abordado depois nas séries "You Don't Know JS".

ES5 introduziu uma forma de sobrescrever parte dessas operações, não no nível de objeto mas no nível por-propriedade, através do uso de _getters_ e _setters_. Getters são propriedades que, na verdade, chamam uma função oculta para recuperar um valor. Setters são propriedades que chamam uma função oculta para estabelecer um valor.

Quando você define uma propriedade para ter um getter ou um setter ou ambos, a definição da propriedade se torna um "descritor de acessor" (o oposto de um "descritor de dados"). Para descritores de acessor, as características da propriedade de `value` e `writable` são discutíveis e ignoradas, em vez disso, o JS considera as características da propriedade de `set` e `get` (como também `configurable` e `enumerable`).

Considere:

```js
var myObject = {
	// define um getter para `a`
	get a() {
		return 2;
	}
};

Object.defineProperty(
	myObject,	// alvo
	"b",		// nome da propriedade
	{			// descritor
		// define um getter para `b`
		get: function(){ return this.a * 2 },

		// certifica que `b` aparece como uma propriedade do objeto
		enumerable: true
	}
);

myObject.a; // 2

myObject.b; // 4
```

Através da sintaxe de objeto-literal com `get a() { .. }` ou por definição explícita com `defineProperty(..)`, nos dois casos nós criamos uma propriedade do objeto que realmente não mantém um valor, mas o acesso à propriedade resulta automaticamente em uma chamada de função oculta a uma função getter, com qualquer valor que ela retorna sendo o resultado do acesso à propriedade.

```js
var myObject = {
	// define um getter para `a`
	get a() {
		return 2;
	}
};

myObject.a = 3;

myObject.a; // 2
```

Desde que nós apenas definimos um getter para `a`, se tentarmos definir o valor de `a` depois, a operação de set não lançará um erro, mas irá descartar a atribuição silenciosamente. Mesmo se houvesse um setter válido, nosso getter personalizado é _hard-coded_ para retornar apenas `2`, logo a operação de set seria discutível.

Para deixar esse cenário mais sensível, propriedades deveriam ser definidas com setters também, que sobrescrevem a operação `[[Put]]` padrão (conhecida como atribuição), por-propriedade, apenas como esperaríamos. Você provavelmente vai sempre querer declarar tanto getter como setter (ter apenas um ou outro geralmente leva a um comportamento inesperado/espantoso):

```js
var myObject = {
	// define um getter para `a`
	get a() {
		return this._a_;
	},

	// define um setter para `a`
	set a(val) {
		this._a_ = val * 2;
	}
};

myObject.a = 2;

myObject.a; // 4
```

**Nota:** Nesse exemplo, nós armazenamos um específico valor `2` da atribuição (`[[Put]]` operation) em outra variável `_a_`. O nome `_a_` é uma mera convenção para esse exemplo e não implica em nada especial em relação a comportamento -- é uma propriedade normal assim como qualquer outra.

### Existência

Nós mostramos anteriormente que o acesso à propriedade como `myObject.a` pode resultar em um valor `undefined` caso o `undefined` for armazenado explicitamente na propriedade ou a propriedade `a` não existir de forma alguma. Logo, se o valor é o mesmo em ambos os casos, como é que diferenciamos?

Podemos perguntar a um objeto se ele possui certa propriedade *sem* pedir para obter o valor da propriedade:

```js
var myObject = {
	a: 2
};

("a" in myObject);				// true
("b" in myObject);				// false

myObject.hasOwnProperty( "a" );	// true
myObject.hasOwnProperty( "b" );	// false
```

O operador `in` verificará se a propriedade *está* no objeto ou se ela existe em algum nível mais alto da cadeia de `[[Prototype]]` do objeto (veja Capítulo 5). Em contraste ao `in`, `hasOwnProperty(..)` verifica *apenas* se `myObject` tem a propriedade ou não, e não consultará a cadeia de `[[Prototype]]`. Nós voltaremos a discutir importantes diferenças entre essas duas operações no Capítulo 5, onde examinamos `[[Prototype]]`s em detalhes.

`hasOwnProperty(..)` é acessível para todos os objetos normais via delegação ao `Object.prototype` (veja o Capítulo 5). Mas é possível criar um objeto que seja ligado ao `Object.prototype` (via `Object.create(null)`) -- (veja o Capítulo 5). Nesse caso, uma chamada de método como `myObject.hasOwnProperty(..)` falharia.

Nesse cenário, um modo mais robusto de realizar tal verificação é `Object.prototype.hasOwnProperty.call(myObject,"a")`, que toma emprestado o método base `hasOwnProperty(..)` e usa um *_binding_ explícito de `this`* (veja o Capítulo 2) para aplicar ao nosso `myObject`.

**Nota:** O operador `in` aparenta que verificará a existência de um *valor* dentro de um container, mas na verdade ele verifica a existência de um nome de propriedade. Essa diferença é importante de se notar no que se diz respeito a arrays, onde há uma forte tentação de verificar algo como `4 in [2, 4, 6]`, mas isso não se comportará da maneira esperada.

#### Enumeração

Anteriormente, explicamos brevemente a ideia de "enumerabilidade" quando olhamos para a característica do descritor de propriedade de `enumerable`. Vamos rever e examinar isso mais detalhadamente.

```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// torne `a` enumarável normalmente
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// torne `b` não-enumerável
	{ enumerable: false, value: 3 }
);

myObject.b; // 3
("b" in myObject); // true
myObject.hasOwnProperty( "b" ); // true

// .......

for (var k in myObject) {
	console.log( k, myObject[k] );
}
// "a" 2
```

Você vai perceber que `myObject.b` de fato **existe** e tem um valor acessível, mas não aparece no laço `for..in` (embora, surpreendentemente, ele seja mostrado pelo operador de `in` que verifica se a propriedade existe). Isso acontece porque "enumerable" basicamente significa "será incluído se for possível iterar sobre as propriedades do objeto".

**Lembrete:** Laços `for..in` aplicados a arrays podem gerar resultados inesperados, nisso a enumeração de um array incluirá não apenas todos os índices numéricos, mas também qualquer propriedade enumerável. É uma boa ideia usar laços `for..in` *apenas* em objetos e laços tradicionais `for` com iteração com índices numéricos para os valores armazenados em arrays.

Outra maneira que propriedades enumeráveis e não-enumeráveis podem ser distinguidas:

```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// Faz com que `a` seja enumerável
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// Faz com que `b` seja não-enumerável
	{ enumerable: false, value: 3 }
);

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```

`propertyIsEnumerable(..)` testa se dado nome de propriedade existe *diretamente* em um objeto e também se é `enumerable:true`.

`Object.keys(..)` retorna um array de todas propriedades enumeráveis, enquanto que `Object.getOwnPropertyNames(..)` retorna um array de *todas* as propriedades, enumerável ou não.

Enquanto que `in` vs. `hasOwnProperty(..)` divergem com relação se eles consultam a cadeia de `[[Prototype]]` ou não, `Object.keys(..)` e `Object.getOwnPropertyNames(..)` inspecionam *apenas* o objeto direto especificado.

Atualmente, não há uma maneira nativa de obter uma lista de **todas as propriedades** equivalente ao que o operator `in` faz (percorrendo todas as propriedades de toda a cadeia de `[[Prototype]]`, como foi explicado no Capítulo 5). Você poderia ter um recurso parecido percorrendo recursivamente a cadeia de `[[Prototype]]` de um objeto e, para cada nível, capturar a lista de propriedades de `Object.keys(..)` -- apenas propriedades enumeráveis.

## Iteração

O laço `for..in` itera sobre a lista de propriedades enumeráveis de um objeto (incluindo a série de `[[Prototype]]`). Mas e se você quiser iterar sobre os valores?

Com arrays indexados numericamente, iterar sobre os valores é geralmente realizado usado o laço padrão `for`, assim:

```js
var myArray = [1, 2, 3];

for (var i = 0; i < myArray.length; i++) {
	console.log( myArray[i] );
}
// 1 2 3
```

Essa forma não está iterando sobre os valores, e sim sobre os índices, onde depois você usa tais índices para referenciar os valores, como `myArray[i]`.

ES5 também adiciona alguns _helpers_ de interação para arrays, including `forEach(..)`, `every(..)` e `some(..)`. Cada um desses _helpers_ aceita uma função de callback para ser aplicada em cada elemento do array, diferenciando apenas na forma que cada um responde um valor de retorno do callback.

`forEach(..)` irá iterar sobre todos os valores do array, e ignora qualquer valor retornado de callback. `every(..)` itera até o fim do array *ou* até o callback retornar um valor `false` (ou "algo falso"), enquanto que `some(..)` itera até o fim *ou* até o callback retornar um valor `true` (ou "algo verdadeiro").

Esses valores de retorno especiais dentro de `every(..)` e `some(..)` funcionam um pouco como a declaração `break` dentro de um laço `for` normal, no qual a iteração é interrompida antes de atingir o seu final.

Se você iterar em um objeto com um laço `for..in`, você também está obtendo o valores indiretamente porque ele está iterando apenas sobre as propriedades enumeráveis do objeto, fazendo com que você acesse as propriedades manualmente para obter os valores.

**Nota:** Em contraste a iteração sobre índices de arrays numericamente ordenados (laço `for` ou outros iteradores), a ordem de iteração sobre as propriedades de um objeto **não é garantida** e pode variar entre diferentes motores JS. **Não confie** em qualquer ordenação _observada_ para qualquer coisa que requer consistência entre ambientes, uma vez que qualquer acordo _observado_ não é confiável.

Mas se você quiser iterar sobre os valores diretamente em vez dos índices do array (ou propriedades do objeto)? Felizmente, ES6 adiciona uma sintaxe de laço `for..of` para iterar sobre arrays (e objetos, se o objeto definir seu próprio iterador customizado):

```js
var myArray = [ 1, 2, 3 ];

for (var v of myArray) {
	console.log( v );
}
// 1
// 2
// 3
```

O laço `for..of` precisa de um objeto iterador (de uma função interna padrão conhecido nas especificações como `@@iterator`) da *coisa* a ser iterada, e o laço então itera sobre sucessivos valores retornado da chamada do método `next()` do objeto iterador, uma vez para cada iteração do laço.

Arrays têm um `@@iterator` nativo, então o `for..of` funciona facilmente neles, como mostrado. Mas vamos iterar o array usando o `@@iterator` nativo para ver como funciona:

```js
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();

it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { done:true }
```

**Nota** Nós chegamos ao `@@iterator` *propriedade interna* de um objeto usando `Symbol` do ES6: `Symbol.iterator`. Nós mencionamos brevemente sobre a semântica do `Symbol` nesse capítulo (veja "Nomes de propriedades computadas"), então o mesmo raciocínio se aplica aqui. Você sempre irá querer referenciar tais propriedades especiais pela referência de nome do `Symbol` em vez de um valor especial. Além disso, apesar das implicações de nomes, `@@iterator` **não é o objeto iterador**, mas uma **função que retorna** o objeto iterador -- um detalhe simples, mas importante!

Como o trecho acima revela, o valor de retorno de uma chamada `next()` do iterador é um objeto na forma de `{ value: .. , done: .. }`, onde `value` é o atual valor da iteração e o `done` é um `boolean` que indica se há algo mais para iterar.

Note que o valor `3` foi retornado com `done:false`, que parece estranho à primeira vista. Você tem que chamar o `next()` uma quarta vez (que o laço `for..of` no trecho de código anterior faz automaticamente) para obter `done:true` e saber que você realmente finalizou a iteração. O motivo dessa peculiaridade está além do escopo do que iremos discutir aqui, mas vem de semânticas de funções do gerador no ES6.

Enquanto arrays iteram automaticamente nos laços `for..of`, objetos comuns **não possuem um `@@iterator` nativo**.
As razões para omissão intencional são mais complexas do que examinaremos aqui, mas em geral foi melhor não incluir alguma implementação que pudesse ser problemática em futuros tipos de objetos.

*É* possível definir seu próprio `@@iterator` padrão para qualquer objeto que você queira iterar. Por exemplo:

```js
var myObject = {
	a: 2,
	b: 3
};

Object.defineProperty( myObject, Symbol.iterator, {
	enumerable: false,
	writable: false,
	configurable: true,
	value: function() {
		var o = this;
		var idx = 0;
		var ks = Object.keys( o );
		return {
			next: function() {
				return {
					value: o[ks[idx++]],
					done: (idx > ks.length)
				};
			}
		};
	}
} );

// itera `myObject` manualmente
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }

// itera `myObject` com `for..of`
for (var v of myObject) {
	console.log( v );
}
// 2
// 3
```

**Nota:** Nós usamos `Object.defineProperty(..)` para definir nosso `@@iterator` personalizado (na maioria das vezes poderíamos definí-lo como não-numérico), mas usando o `Symbol` como um *nome de propriedade computado* (abordado mais cedo nesse capítulo), nós poderíamos ter declarado diretamente, como `var myObject = { a:2, b:3, [Symbol.iterator]: function(){ /* .. */ } }`.

Cada vez que o laço `for..of` chama `next()` no objeto iterador de `myObject`, o ponteiro interno avançará e retornará o próximo valor da lista de propriedades do objeto (veja uma nota anterior sobre ordenação de iteração nas propriedaes/valores de objeto).

A iteração que acabamos de demonstrar é uma simples iteração valor-por-valor, mas claro que você pode definir arbitrariamente iterações complexas para suas estruturas de dados como achar melhor. Iteradores personalizados combinados com o laço `for..of` do ES6 são um nova e poderosa ferramenta sintática para manipulação de objetos definidos pelo o usuário.

Por exemplo, uma lista de objetos `Pixel` (com valores de coordenadas `x` e `y`) poderia decidir ordenar sua iteração baseada na distância linear da origem `(0,0)` ou filtrar pontos que são "muito distantes" etc. Contanto que seu iterador retorne os valores de retorno `{ value: .. }` esperados das chamadas do `next()`, e um `{ done: true }` depois que a iteração esteja completa, o `for..of` do ES6 pode iterar sobre os valores.

Na verdade, você pode até mesmo gerar iteradores "infinitos" que nunca "terminam" e sempre retornam um novo valor (tal como um número randômico, um valor incrementado, um identificador único etc), contudo você provavelmente não usará tais iteradores com um laço `for..of` infinito, pois isso nunca terminaria e suspenderia o seu programa.

```js
var randoms = {
	[Symbol.iterator]: function() {
		return {
			next: function() {
				return { value: Math.random() };
			}
		};
	}
};

var randoms_pool = [];
for (var n of randoms) {
	randoms_pool.push( n );

	// não continue infinitamente!
	if (randoms_pool.length === 100) break;
}
```

Esse iterador gerará números randômicos "para sempre", por isso tivemos cuidado em pegar apenas 100 valores, fazendo com que nosso programa não fique suspenso.


## Revisão (TL;DR)

Objetos em JS possuem uma forma literal (tal como `var a = { .. }`) e uma forma construída (tal como `var a = new Array(..)`). A forma literal é quase sempre preferível, mas a forma construída oferece, em alguns casos, mais opções de criação.

Muitas pessoas alegam erradamente que "tudo em JavaScript é um objeto", mas essa afirmação é incorreta. Objetos são um dos 6 (or 7, dependendo de sua perspectiva) tipos primitivos. Objetos têm subtipos, incluindo `function`, e também podem ser comportamento-especializado, como `[object Array]`, rótulo interno representando o subtipo de objeto de array.

Objetos são coleções de pares chave/valor. Os valores podem ser acessados como propriedades, via sintaxe `.propName` ou `["propName"]`. A qualquer momento que uma propriedade é acessada, o motor JS, na verdade, invoca a operação interna `[[Get]]` padrão (e `[[Put]]` para definir valores), que não só procura pela propriedade diretamente no objeto, mas irá percorrer a cadeia de `[[Prototype]]` se não for encontrada.

Propriedades têm certas características que podem ser controladas por decritores de propriedade, tais como `writable` e `configurable`. Além disso, objetos podem ter suas mutabilidades (e de suas propriedades) controladas para vários níveis de imutabilidade usando `Object.preventExtensions(..)`, `Object.seal(..)` e `Object.freeze(..)`.

Propriedades não têm que conter valores -- elas podem ser "propriedades de acessor" também, com getters/setters. Elas também podem ser *enumeráveis* ou não, que controlam se eles aparecem nas iterações do laço `for..in`, por exemplo.

Você também pode iterar sobre **os valores** nas estruturas de dados (arrays, objetos etc) usando a sintaxe `for..of` do ES6, que procura por um objeto `@@iterator` nativo ou personalizado consistindo de um método `next()` para avançar pelos valores de dados um de cada vez.
