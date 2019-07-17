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

Se o *como* desse trecho de código confunde você, não se preocupe! Já vamos chegar lá. Apenas coloque isso de lado por um momento para que possamos olhar mais claramente o *porquê*.

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

Quanto mais complexo for o padrão de utilização, você verá mais claramente que passar o contexto como um parâmetro explícito é frequentemente mais confuso do que passar o contexto com o `this`. Quando exploramos objetos e protótipos, você verá a utilidade de uma coleção de funções sendo capaz de referenciar automaticamente o objeto no contexto apropriado.

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

**Nota:** A referência old-school, mas agora depreciada e de torcer o nariz, `arguments.callee` dentro uma função *também* aponta para o objeto da função que está sendo executado. Essa referência é normalmente a única forma de acessar um objeto de função anônima de dentro dela mesma. A melhor abordagem no entanto, é evitar o uso de funções anônimas completamente, pelo menos para as que necessitam de auto-referência, e em vez disso utilizar funções nomeadas (expressões). `arguments.callee` é obsoleta e não deve ser usada.

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

### Seu Escopo

O próximo equívoco mais comum sobre o significado do `this` é que ele, de alguma forma, se refere ao escopo léxico da função. É uma questão complicada, porque por um lado isso é verdade, mas pelo outro é bastante errado.

Para ser claro, `this` não é, de forma alguma, uma referência para o **escopo léxico** da função. É verdade que internamente, o escopo é como um objeto com propriedades para cada um dos identificadores disponíveis. Mas o "objeto" do escopo não é acessível para o código JavaScript. É uma parte interna da implementação da *Engine*.

Considere o código que tenta (e falha!) cruzar o limite e usar o `this` para, implicitamente, se referir ao escopo léxico da função:

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

Existe mais de um erro neste trecho de código. Embora ele pode parecer artificial, o código que você vê é uma essência do verdadeiro código do mundo real, que foi postado em fóruns públicos de ajuda da comunidade. É uma maravilhosa (se não triste) ilustração de quão errada pode ser a suposição do `this`.

Primeiramente, é feita uma tentativa de referenciar a função `bar()` pelo `this.bar()`. É certamente quase um *acidente* que isso funcione, mas vamos explicar *como* em breve. A forma mais natural de chamar `bar()` seria omitindo o sedutor `this.` e apenas fazer uma referência léxica ao identificador.

No entanto, o desenvolvedor que escreve esse tipo de código está tentando utilizar `this` para criar uma ponte entre o escopo léxico de `foo()` e `bar()`, então `bar()` teria acesso a variável `a` de dentro do escopo de `foo()`. **Tais pontes não são possíveis.** Você não pode usar uma referência `this` para procurar algo em um escopo léxico. Isso não é possível.

Toda vez que você sentir que está tentando mixar escopo léxico procurando algo com `this`, lembre-se: *não existem pontes*.

## O que é `this`?

Deixando de lado as várias premissas incorretas, vamos focar nossa atenção para como o mecanismo `this` realmente funciona.

Mais cedo falamos que o `this` não é um vínculo do momento de escrita mas um vínculo do momento de execução. Ele é baseado nas condições do contexto da função que o chama. O vínculo `this` não tem nada a ver com onde a função foi declarada, mas em vez disso, tem tudo a ver com com a forma que a função é chamada.

Quando uma função é chamada, um registro de ativação, também conhecido como contexto de execução, é criado. Esse registro contém informações sobre de onde a função foi chamada (a call-stack), *como* a função foi chamada, quais parâmetros foram passados, etc. Uma das propriedades desse registro é a referência `this`, que será usada pela duração da execução dessa função.

No próximo capítulo, nós vamos aprender a encontrar o **call-site** da função para determinar como sua execução será vinculada ao `this`.

## Review (TL;DR)

O vínculo `this` é uma constante fonte de confusão para os desenvolvedores JavaScript que não param para aprender como o mecanismo realmente funciona. Suposições, tentativa e falha, e copiar-e-colar cegamente das respostas do Stack Overflow não são maneiras eficazes ou adequadas de alavancar esse importante mecanismo `this`.

Para entender o `this`, você primeiro precisa aprender o que o `this` *não* é, apesar de quaisquer suposições ou equívocos que podem levá-lo por esses caminhos. `this` não é uma referência para a própria função, nem é uma referência para o escopo *léxico* da função.

`this` na verdade é um vínculo que é feito quando a função é chamada, e *o que* é referenciado é determinado inteiramente pelo call-site de onde a função é chamada.
