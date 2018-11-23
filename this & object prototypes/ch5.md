# You Don't Know JS: *this* & Prototipagem de Objetos
# Capítulo 5: Protótipos

Nos capítulos 3 e 4, nós mencionamos a cadeia `[[Prototype]]` por diversas vezes, mas não havíamos dito ainda do que exatamente se trata. Agora, nós vamos examinar protótipos em detalhes.

**Nota:** Todas as tentativas de emular o comportamento de cópia de classes, como descrito anteriormente no Capítulo 4, rotuladas como variações de "mixins", contornam completamente o mecanismo de cadeia de prototipagem `[[Prototype]]` que iremos examinar neste capítulo. 

## `[[Prototype]]`

Objetos em JavaScript possuem propriedades internas, denominadas em sua especificação como `[[Prototype]]`, que trata-se simplesmente de uma referência à um outro objeto. Quase todos os objetos recebem um valor não nulo para essa propriedade, no momento de sua criação. 

**Nota:** Nós veremos em breve que *é* possível para um objeto ter uma ligação vazia de `[[Prototype]]`, embora isso seja pouco comum.

Considere:

```js
var myObject = {
	a: 2
};

myObject.a; // 2
```

Qual é a referência de `[[Prototype]]` utilizada? No capítulo 3, nós examinamos a operação `[[Get]]` que é invocada quando você referencia uma propriedade à um objeto, como o `myObject.a`. Para uma operação `[[Get]]` padrão, o primeiro passo é checar se o próprio objeto possui uma propriedade `a`, e caso possua, se a mesma é utilizada.

**Nota:** ES6 Proxies estão fora do escopo de discussão deste livro (isso será visto em um futuro livro da série!), mas tudo que nós discutimos aqui sobre comportamentos normais de  `[[Get]]` e `[[Put]]` não se aplicam caso um `Proxy` seja envolvido.

Mas é o que acontece se `a` **não está** presente em `myObject` que chama nossa atenção agora para a ligação `[[Prototype]]` do objeto. 

O procedimento de uma operação `[[Get]]` padrão segue a ligação `[[Prototype]]` do objeto caso não consiga encontrar a propriedade que for solicitada diretamente no objeto.

```js
var anotherObject = {
	a: 2
};

// cria um objeto ligado com `anotherObject`
var myObject = Object.create( anotherObject );

myObject.a; // 2
```

**Nota:** Nós explicaremos o que `Object.create(..)` faz, e como ele opera, em breve. Por enquanto, apenas assuma que ele cria um objeto com uma ligação `[[Prototype]]` no objeto especificado que estamos examinando.

Então, temos `myObject` que agora está ligado através do `[[Prototype]]` com `anotherObject`. Embora `myObject.a` não exista realmente, o acesso à propriedade é bem sucedido (encontrado em `anotherObject` ao invés disso) e de fato encontra o valor `2`.

Mas, caso `a` também não seja encontrado em `anotherObject`, sua cadeia `[[Prototype]]`, caso não esteja vazia, é novamente consultada e seguida.

Esse processo continua até que seja encontrada uma propriedade com o mesmo nome, ou até que a cadeia `[[Prototype]]` termine. Se *nenhuma* propriedade for encontrada até o final da cadeia `[[Prototype]]`, o resultado que a operação `[[Get]]` retorna é `undefined`.

Similar à esse processo de busca na cadeia `[[Prototype]]`, se você usar o laço `for..in` para iterar um objeto, qualquer propriedade que seja alcançada através da cadeia (e que também seja `enumerable` -- veja Capítulo 3) será enumerada. Se você utilizar o operador `in` para testar a existência de uma propriedade dentro de um objeto, `in` irá verificar por toda a cadeia do objeto (independentemente da *enumerabilidade* do mesmo).

```js
var anotherObject = {
	a: 2
};

// cria um objeto ligado ao `anotherObject`
var myObject = Object.create( anotherObject );

for (var k in myObject) {
	console.log("found: " + k);
}
// found: a

("a" in myObject); // retorna true
```

Ou seja, a cadeia `[[Prototype]]` é consultada, uma ligação por vez, quando você realiza buscas de propriedades de diferentes maneiras. As buscas param assim que a propriedade é encontrada ou assim que cadeia termina. 

### `Object.prototype`

Mas *onde* exatamente a cadeia `[[Prototype]]` "termina"?

No topo de toda cadeia `[[Prototype]]` *normal* está embutido o `Object.prototype`. Este objeto inclui uma varidade de utilitários normalmente utilizados por todo JS, porque todos objetos normais (que estão embutidos na linguagem, não extensões self-host) em Javascript "descendem" (ou constam no topo de sua cadeia `[[Prototype]]`) do objeto `Object.prototype`. 

Alguns utilitários encontrados aqui com os quais você pode estar familiarizado incluem `.toString()` e `.valueOf()`. No Capítulo 3, nós introduzímos um outro: `.hasOwnProperty(..)`. E uma outra função dentro do `Object.prototype` da qual você não deve estar familiriazado, mas que iremos tratar adiante neste capítulo, é `.isPrototypeOf(..)`. 

### Configuração e Sombreamento de Propriedades

No Capítulo 3, nós havíamos mencionado que a configuração das propriedades de um objeto é mais do que simplesmente adicionar uma nova propriedade ao objeto ou alterar o valor de uma propriedade existente. Nós vamos agora revisitar essa situação de uma forma mais completa.

```js
myObject.foo = "bar";
```

Se o objeto `myObject` já possui uma propriedade de acesso à dados normal chamada de `foo` diretamente presente, a atribuição é tão simples quanto alterar o valor de uma propriedade existe.

Se `foo` não estiver diretamente presente em `myObject`, a cadeia `[[Prototype]]` é percorrida, assim como em uma operação `[[Get]]`. Se `foo` não for encontrada na cadeia, a propriedade `foo` é adicionada diretamente para `myObject` com seu valor especificado, como esperado. 

Entretanto, se `foo` já está presente em algum lugar mais alto da cadeia, comportamentos sutilmente (e talvez surpreendentemente) diferentes podem ocorrer com a atribuição `myObject.foo = "bar"`. Nós examinaremos isso em instantes.

Se a propriedade com nome `foo` acabar tanto no próprio `myObject` quanto em um nível mais alto da cadeia `[[Prototype]]` que começa em `myObject`, trata-se de algo chamado de *sombreamento*. A propriedade `foo` que se encontra diretamente em `myObject` *faz sombra* à qualquer propriedade `foo` que apareça mais alto na cadeia, porque a busca de `myObject.foo` irá sempre encontrar a propriedade `foo` que estiver em um lugar mais baixo na cadeia.      

Como acabamos de indicar, sombrear `foo` em `myObject` não é tão simples quanto pode parecer. Nós vamos agora examinar três cenários para a atribuição `myObject.foo = "bar"` quando `foo` **não está** diretamente presente em `myObject`, mas **está** presente em um nível mais alto da cadeia `[[Prototype]]` de `myObject`.

1. Se uma propriedade de acesso à dados normal (veja Capítulo 3) de nome `foo` é encontrada em qualquer lugar da cadeia `[[Prototype]]`, **e não está marcada como somente leitura (`writable:false`)** então uma nova propriedade de nome `foo` é diretamente adicionada à `myObject`, resultando em uma **propriedade sombreada**.
2. Se `foo` é encontrada no alto da cadeia `[[Prototype]]`, mas está marcada como **somente leitura (`writable:false`)**, então tanto a configuração de uma propriedade existente quanto a criação de uma propriedade sombreada em `myObject` **não são permitidas**. Se o código estiver rodando em `strict mode`, um erro será retornado. Caso não esteja, uma atribuição de valor à propriedade será silenciosamente ignorada. Em todo caso, **não haverá sombreamento**. 
3. Se `foo` é encontrada no alto da cadeia `[[Prototype]]` é for um setter (veja Capítulo 3), então o setter sempre será chamado. Nenhuma `foo` será adicionada (ou sombreada) em `myObject`, e nem o setter de `foo` será redefinido.

Muitos desenvolvedores assumem que a atribuição de uma propriedade (`[[Put]]`) sempre resultará em sombreamento se a propriedade já existir no alto da cadeia `[[Prototype]]`, mas como se nota, isso só é verdadeiro em uma (a primeira) das três situações descritas acima.

Se você quer sombrear `foo` nos casos 2 e 3, não poderá usar `=` para fazer a atribuição, ao invés disso deve utilizar `Object.defineProperty(..)` (veja Capítulo 3) para adicionar `foo` em `myObject`.

**Nota:** O caso 2 pode ser o mais surpreendente dos três. A presença de uma propriedade *somente leitura* previne que uma propriedade de mesmo nome seja implicitamente criada (sombreada) em um nível mais baixo da cadeia `[[Prototype]]`. O motivo para esta restrição é primariamente para reforçar a ilusão de propriedades herdadas de classes. Se você pensar em `foo` em um nível alto da cadeia como tendo sido herdado (copiado para baixo) para `myObject`, então faz sentido impor a natureza não-gravável desta propriedade `foo` em `myObject`. Se você no entanto separar a ilusão e o fato, e reconhecer que nenhuma cópia de herança *realmente* ocorreu (veja Capítulos 4 e 5), é um pouco antinatural que `myObject` seja impedido de ter uma propriedade `foo` apenas porque algum outro objeto tinha um `foo` não-gravável nele. É ainda mais estranho que esta restrição só se aplique à designação `=`, mas não seja aplicada quando `Object.defineProperty (..)` é utilizado.    

Sombreamento com **métodos** leva ao terrível *pseudo-polimorfismo explícito* (veja Capítulo 4), se você precisa delegar entre eles. Normalmente, sombreamento é mais complicado e específico do que sua importância, **então você deve tentar evitar de usar, se possível**. Veja o Capítulo 6 para um design pattern alternativo, que entre outras coisas desencoraja o uso do sombreamento em favor de alternativas mais limpas.

Sombreamento pode até ocorrer implicitamente de maneiras sutis, então um cuidado deve ser tomado na tentativa de evitá-lo. Considere:

```js
var anotherObject = {
	a: 2
};

var myObject = Object.create( anotherObject );

anotherObject.a; // 2
myObject.a; // 2

anotherObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "a" ); // false

myObject.a++; // ops, sombreamento implicito!

anotherObject.a; // 2
myObject.a; // 3

myObject.hasOwnProperty( "a" ); // true
```

Embora possa parecer que `myObject.a++` deveria (via delegação) procurar e apenas incrementar a propriedade `anotherObject.a` propriamente dita *em seu lugar*, em vez disso a operação `++` corresponde à `myObject.a = myObject.a + 1`. O resultado é um `[[Get]]` procurando a propriedade `a` através de `[[Prototype]]` para obter o valor atual `2` de `anotherObject.a`, incrementando o valor em um, e então um `[[Put]]` atribuindo o valor `3` à uma nova propriedade sombreada `a` em `myObject`. Ops!

Tenha muito cuidado ao lidar com as propriedades delegadas que modifica. Se você quer incrementar `anotherObject.a`, a única maneira apropriada é `anotherObject.a++`.

## "Class"

Neste momento, você deve estar imaginando: "*Por que* um objeto precisa ser ligado à um outro objeto?" Qual é o real benefício? Esta é uma pergunta muito apropriada de se fazer, mas primeiro precisamos entender o que `[[Prototype]]` **não é** antes de entendê-lo completamente e apreciar o que **é** e como pode ser útil.

Como explicamos no Capítulo 4, em Javascript, não há padrões abstratos para objetos chamados de "classes" como no caso de linguagens orientadas à classes. Javascript tem **apenas** objetos.

Na verdade, Javascript é **quase única** entre as linguagens pelo fato de talvez ser a única linguagem com o direito de ser rotulada de "orientada à objetos", porque é uma entre uma lista bem pequena de linguagens em que objetos podem ser criados diretamente, sem a existência de uma classe.

Em Javascript, classes não podem (já que não existem!) descrever o que um objeto pode fazer. O objeto define diretamente seu próprio comportamento. **Existe *apenas* o objeto.**

### "Class" Functions

Existe um tipo de comportamento peculiar em Javascript que vem sendo descaradamente abusado por anos para *hackear* algo que somente *parece* com "classes". Nós examinaremos essa abordagem em detalhes. 

O peculiar comportamento de "espécie de classe" depende de uma estranha característica das funções: por padrão, todas as funções obtêm uma propriedade pública, não enumerável (veja o Capítulo 3) nelas chamada  `prototype`, que aponta para um objeto arbitrário.

```js
function Foo() {
	// ...
}

Foo.prototype; // { }
```

Esse objeto é frequentemente chamado de "prototype de Foo", porque o acessamos atráves de uma referência de propriedade com o infeliz nome de `Foo.prototype`. Entretanto, essa terminologia está fatalmente destinada à nos confundir, como veremos em breve. Ao invés disso, irei chamar de "o objeto que antes era conhecido como prototype de Foo". Brincadeira. Que tal: "o objeto arbitrariamente rotulado de 'Foo ponto prototype'"?

Independente de como chamamos, o que exatamente é este objeto? 

A forma mais direta de se explicar é que cada objeto criado ao chamar `new Foo()` (veja Capítulo 2) acabará (de certa forma, arbitrariamente) ligado pelo `[[Prototype]]` com esse objeto 'Foo ponto prototype'.

Vamos ilustrar:

```js
function Foo() {
	// ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
```

Quando `a` é criado ao chamar `new Foo()`, uma das coisas (ceja Capítulo 2 para todos os *quatro* passos) que acontece é que `a` obtêm uma ligação `[[Prototype]]` interna com o objeto que `Foo.prototype` está apontando.

Pare um momento e pondere as implicações desta declaração.

Em linguagens orientadas à classes, multiplas **cópias** (também conhecidas como "instâncias") de uma classe podem ser criadas, como carimbar algo à partir de um molde. Como vimos no Capítulo 4, isso acontece porque o processo de instânciação (ou de herdar de) uma classe significa "copiar o plano de comportamento desta classe para um objeto físico", e isso é feito novamente para cada nova instância.  

Mas em Javascript, não há ações de cópias deste tipo. Você não cria múltiplas instâncias de uma classe. Você cria múltiplos objetos que são *ligados* pelo `[[Prototype]]` com um objeto comum. Mas por padrão, nenhuma cópia ocorre, e portanto esses objetos acabam não sendo totalmente separados e nem desconectados um do outro, pelo contrário, estão bem ***ligados***. 

`new Foo()` resulta em um novo objeto (que chamamos de `a`), e **este** novo objeto `a` é internamente ligado por `[[Prototype]]` com o objeto `Foo.prototype`.  

**Nós acabamos com dois objetos, um ligado ao outro.** E *é isso*. Nós não instanciamos uma classe. Nós certamente não fizemos nenhuma cópia de comportamento de uma "classe" para um objeto concreto. Nós só fizemos com que dois objetos fossem ligados um ao outro.

Na verdade, o segredo, que ilude a maioria dos desenvolvedores de JS, é que a chamada da função `new Foo()` não tem praticamente nada à ver *diretamente* com o processo de criar a ligação. **Foi uma espécie de efeito colateral acidental.** `new Foo()` é uma forma indireta de se conseguir o que queremos: **um novo objeto ligado à um outro objeto**. 

Nós conseguimos ter o que queremos de uma forma mais *direta*? **Sim!** o herói é `Object.create(..)`. Mas nós chegaremos lá daqui à pouco. 

#### O que está em um nome?

Em Javascript, nós não fazemos *cópias* de um objeto ("class") para outro ("instância"). Nós criamos *ligações* entre objetos. Para o mecanismo `[[Prototype]]`, visualmente, as setas movem da direita pra esquerda, e de baixo pra cima.
<img src="fig3.png">

Este mecanismo é frequentemente chamado de "herança prototípica" (nós examinaremos o código em detalhes à seguir), e é comumente considerado como a versão em linguagem dinâmica para a "herança clássica". É uma tentativa de se apoiar no entendimento comum do que "herança" significa no mundo orientado a classes, mas *ajustar* (**leia: pavimentar**) a semântica compreendida para se adequar ao script dinâmico.

A palavra "herança" tem um significado poderoso (veja Capítulo 4), com muito precedente mental. Meramente adicionar o termo "prototípica" para distinguir um comportamento *que é na verdade quase oposto* em Javascript deixou um rastro de quase duas décadas de confusão.

Eu gosto de dizer que o rótulo de "protótipo" para uma "herança" inverte drásticamente o seu real significado, é como segurar uma laranja em uma mão, uma maçã na outra, e insistir para que chamem a maçã de "laranja vermelha". Não importa o quão confuso seja o rótulo que eu utilize, isso não muda o *fato* que uma fruta é maçã e a outra é laranja.

A melhor abordagem é simplesmente chamar uma maçã de maçã -- para usar a terminologia mais precisa e direta. Isso facilita o entendimento tanto em suas similaridades quanto suas **muitas diferenças**, porque tudo que temos é um entendimento simples e compartilhado do que "maçã" significa. 

Graças à confusão e conflação de termos, acredito que o próprio rótulo de "herança prototípica" (e a tentativa de se aplicar incorretamente toda a sua terminologia de orientação de classe associada, como "classe", "construtor", "instância", "polimorfismo", etc.) tem causado efeitos **mais negativos que positivos** em explicar como o mecanismo de Javascript *realmente* funciona. 

"Herança" implica em uma operação de *cópia*, e JavaScript não copia propriedades de objetos (nativamente, por padrão). Ao invés disso, JS cria uma ligação entre dois objetos, onde um objeto pode essencialmente *delegar* acesso às propriedades/funções para outro objeto. "Delegação" (veja Capítulo 6) é um termo muito mais preciso para o mecanismo de ligação entre objetos do JavaScript.

Outro termo que às vezes é jogado em JavaScript é "herança diferencial". A ideia aqui é que descrevemos o comportamento de um objeto em termos do que é *diferente* de um descritor mais generalizado. Por exemplo, você explica que um carro é um tipo de veículo, mas que tem exatamente 4 rodas, ao invés de descrever todos os detalhes do que compõe um veículo em geral (motor, etc). 

Se você tentar pensar em qualquer objeto em JS como a soma total de todo o comportamento que está *disponível* via delegação, e **em sua mente você nivela** todo esse comportamento em apenas uma *coisa* tangível, então você pode (de certa forma) ver como "herança diferencial" poderá se encaixar.

Mas assim como "herança prototípica", "herança diferencial" finge que seu modelo mental é mais importante que o que está fisicamente acontecendo na linguagem. Ela negligencia o fato que o objeto `B` não é realmente diferencialmente construído, mas é construído com características específicas definidas, ao lado de "buracos" onde nada é definido. É nesses "buracos" (lacunas ou falta de definição) que a delegação *pode* assumir e, na mosca, "preenchê-los" com o comportamento delegado.

O objeto não é, por padrão nativo, nivelado em um único objeto diferencial, ***através de cópia***, que o modelo mental de "herança diferencial" implica. Como tal, "herança diferencial" não é apenas um ajuste natural para descrever como o mecanismo `[[Prototype]]` do JavaScript realmente funciona.

Você *pode escolher* em preferir a terminologia de "herança diferencial" e o modelo mental, por questão de gosto, mas não há como negar o fato de que *apenas* se encaixa nas acrobacias mentais dentro de sua mente, e não do comportamento físico da engine.

### "Constructors"

Vamos voltar à um código visto mais cedo:

```js
function Foo() {
	// ...
}

var a = new Foo();
```

O que exatamente nos leva a pensar que `Foo` é uma "classe"?

Primeiramente, nós vimos o uso da palavra-chave `new`, assim como em linguagens orientadas à classes quando se instânciam classes. Além disso, parece que estamos na verdade executando um método *construtor* de uma classe, porque `Foo()` é o método de que de fato é chamado, assim como construtores reais de classes são chamados quando se instância está classe. 

Para aumentar a confusão sobre a semântica de um "construtor", o objeto arbitrariamente rotulado de `Foo.prototype` tem outro truque na manga. Considere este código: 

```js
function Foo() {
	// ...
}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```

O objeto `Foo.prototype` por padrão (no momento da declaração na linha 1 do snippet) recebe uma propriedade pública, não enumerável (veja Capítulo 3) chamada `.constructor`, e essa propriedade é uma referência de volta à função (`Foo` neste caso) em que o objeto está associado. Além disso, vemos que o objeto `a` criado através da chamada do "construtor" `new Foo ()` *parece* ter também uma propriedade nele chamada `.constructor` que similarmente aponta para "a função que o criou".  

**Nota:** Isso não é realmente verdade. `a` não possui uma propriedade `.constructor`, e ainda que `a.constructor` de fato funcione para a função `Foo`, "construtor" **não significa realmente** "foi construído por", como parece. Nós explicaremos essa estranha situação em breve.   

Ah sim, outra coisa... por convenção no mundo JavaScript, "classes" são nomeadas com letra maiúscula, então o fato de ser `Foo` ao invés de `foo` é uma forte pista de que a intenção é de que seja uma "classe". Isso é totalmente óbvio pra você, certo!?

**Nota:** Essa convenção é tão forte que muitos linters de JS de fato *reclamam* se você chama `new` em um método com letra minúscula, ou se não chamamos `new` em uma função que por acaso comece com uma letra maiúscula. Isso meio que confunde a ideia de que lutamos tanto para obter uma (falsa) "orientação à classe" *do jeito certo* em Javascript em que criamos regras de linter para assegurar que usamos letras maiúsculas, mesmo que letra maiúscula não signifique ***absolutamente* nada** para o motor (engine) JS. 

#### Construtor ou Chamada?

No snippet acima, é tentador pensar que `Foo` é um "construtor", porque nós o chamamos com `new` e observamos que isso "constrói" um objeto.

Na realidade, `Foo` não é mais "construtor" que qualquer outra função em seu programa. Funções por si só **não** são construtores. Entretanto, quando se coloca a palavra-chave `new` em frente à chamada de uma função normal, isso faz com que a função chame uma "chamada de construtor". Na verdade, `new` meio que sequestra qualquer função normal e chama de uma forma que constrói um objeto, **junto com qualquer outra coisa que iria fazer**.

Por exemplo:

```js
function NothingSpecial() {
	console.log( "Não ligue pra mim!" );
}

var a = new NothingSpecial();
// "Não ligue pra mim!"

a; // {}
```

`NothingSpecial` é apenas uma função normal, mas quando chamada com `new`, ela *constrói* um objeto, quase como um efeito colateral, que nós por acaso atribuímos à `a`. A **chamada** foi uma *chamada de construtor*, mas `NothingSpecial` não é, por si só, um *construtor*.  

Em outras palavras, em Javascript, é mais apropriado dizer que um "construtor" é **qualquer função chamada através de uma palavra-chave `new`** na frente.

Funções não são construtores, mas chamadas de funções são "chamadas de construtores" se e apenas se `new` é utilizado.

### Mecânicas

São *esses* os únicos gatilhos comuns para as malfadadas discussões sobre "classe" em Javascript?

**Não exatamente.** Desenvolvedores JS se esforçaram o máximo possível para simular orientação à classes:

```js
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

var a = new Foo( "a" );
var b = new Foo( "b" );

a.myName(); // "a"
b.myName(); // "b"
```

Este snippet mostra dois truques adicionais "orientados à classe" em jogo:

1. `this.name = name`: adiciona a propriedade `.name` em cada objeto (`a` e `b`, respectivamente; veja Capítulo 2 sobre ligação `this`), similar em como instâncias de classes encapsulam dados.

2. `Foo.prototype.myName = ...`: talvez a técnica mais interessante, ela adiciona uma propriedade(função) para o objeto `Foo.prototype`. Agora, surpreendentemente talvez, `a.myName()` funciona. Como?   

No snippet acima, é fortemente tentador pensar que quando `a` e `b` são criados, as propriedades/funções em objeto `Foo.prototype` são *copiadas* sobre cada objeto `a` e `b`. **Entretanto, não é isso o que acontece.**

No início deste capítulo, nós explicamos a ligação `[[Prototype]]`, e cada passo que ela segue para realizar as buscas caso uma referência da propriedade não seja diretamente encontrada no objeto, como parte do padrão do algoritmo `[[Get]]`.

Então, por virtude do que criamos, cada `a` e `b` acabam com uma ligação `[[Prototype]]` interna com `Foo.prototype`. Quando `myName` não é encontrada em `a` ou `b`, respectivamente, é encontrada em seu lugar (através de delegação, veja Capítulo 6) em `Foo.prototype`. 

#### "Construtor" Redux

Você se lembra da discussão mais cedo sobre a propriedade `.constructor`, e como ela *faz parecer* que `a.constructor === Foo` sendo verdadeiro significa que `a` tem de fato uma propriedade `.constructor` em si, apontando para `Foo`? **Isso não está correto.**   

Essa é apenas uma infeliz confusão. Na realidade, a refêrencia de `.constructor` também é *delegada* para `Foo.prototype`, que **acontece de**, por padrão, ter um `.constructor` que aponta para `Foo`.

*Parece* muito conveniente que o objeto `a` "construído por" `Foo` deveria ter acesso à propriedade `.constructor` que aponta para `Foo`. Mas isso nada mais é que uma sensação falsa de segurança. É um acidente feliz, quase tangencial, que `a.constructor` *por acaso* aponta para `Foo` através desta delegação `[[Prototype]]` padrão. Existem diversas maneiras da malfadada suposição de que `.constructor` significa "foi constrúido por" voltar para te assombrar.

Uma delas é que, a propriedade `.constructor` em `Foo.prototype` só está lá por padrão no objeto criado quando a função `Foo` é declarada. Se você cria um novo objeto, e substitui a referência padrão do objeto `.prototype` de uma função, o novo objeto não irá magicamente obter um `.constructor` em si.

Considere:

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // cria um novo objeto prototype

var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!
```
`Object(..)` não "construiu" `a1`, construiu? Certamente parece que `Foo()` "construiu". Muitos desenvolvedores pensam que `Foo()` estava fazendo a construção, mas esse pensamento cai por terra quando se pensa que "construtor" significa "foi construído por", porque seguindo essa lógica, `a1.constructor` deveria ser `Foo`, mas não é!

O que está acontecendo? `a1` não possui nenhuma propriedade `.constructor`, então ele delega até a cadeia `[[Prototype]]` para `Foo.prototype`. Mas este objeto também não tem um `.constructor` (como o objeto padrão `Foo.prototype` teria!), então ele continua delegando, dessa vez até `Object.prototype`, o topo da cadeia de delegação. *Este* objeto de fato possui um `.constructor` em si, que aponta para a função embutida `Object(..)`.


**Equívoco encontrado.**

Claro, você pode adicionar `.constructor` de volta ao objeto `Foo.prototype`, mas isso requer trabalho manual, especialmente se você quer que tenha um comportamento nativo e que seja não enumerável (veja Capítulo 3).

Por exemplo:

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // cria um novo objeto prototype

// Precisa "consertar" de forma apropriada a propriedade 
// `.constructor` que está faltando no novo objeto 
// servindo como `Foo.prototype`.  
// Veja o Capítulo 3 sobre `defineProperty(..)`.
Object.defineProperty( Foo.prototype, "constructor" , {
	enumerable: false,
	writable: true,
	configurable: true,
	value: Foo    // aponta `.constructor` para `Foo`
} );
```

Trata-se de muito trabalho manual só para poder consertar o `.constructor`. Além disso, nós estamos realmente perpetuando o equívoco de que "construtor" significa "foi construído por". Essa é uma ilusão *bem cara*.

O fato é, `.constructor` em um objeto aponta arbitrariamente, por padrão, para uma função que, recíprocamente, possui uma referência de volta ao objeto -- uma referência que se chama `.prototype`. As palavras "constructor" e "prototype" neste caso possuem um significado padrão que podem ou não se manter o mesmo mais tarde. A melhor coisa a se fazer é lembrar a si mesmo, "construtor não significa construído por". 

`.constructor` não é uma propriedade imutável mágica. Ela *é* não enumerável (veja snippet abaixo), mas seu valor é gravável (pode ser alterado), e além disso, você pode adicionar ou sobreescrever (de forma intencional ou acidental) uma propriedade com nome de `constructor` em qualquer objeto que esteja em qualquer cadeia `[[Prototype]]`, com o valor que julgar necessário.

Em virtude de como o algoritmo `[[Get]]` atravessa a cadeia `[[Prototype]]`, uma referência da propriedade `.constructor` encontrada em qualquer lugar pode se resolver de forma bem diferente do que se espera.

Vê como é arbitrário seu real significado?

O resultado? Alguma referência arbitrária de objeto-propriedade como `a1.constructor` não pode ser *confiada* como sendo a referência de função que se assume como padrão. Além disso, como veremos em breve, apenas por simples omissão, `a1.constructor` pode até apontar pra algum lugar bem surpreendente e insensível.

`a1.constructor` não é nada confiável, e uma referência insegura para se colocar em seu código. **Geralmente, essas referências devem ser evitadas quando possível.**

## "(Prototypal) Inheritance"

We've seen some approximations of "class" mechanics as typically hacked into JavaScript programs. But JavaScript "class"es would be rather hollow if we didn't have an approximation of "inheritance".

Actually, we've already seen the mechanism which is commonly called "prototypal inheritance" at work when `a` was able to "inherit from" `Foo.prototype`, and thus get access to the `myName()` function. But we traditionally think of "inheritance" as being a relationship between two "classes", rather than between "class" and "instance".

<img src="fig3.png">

Recall this figure from earlier, which shows not only delegation from an object (aka, "instance") `a1` to object `Foo.prototype`, but from `Bar.prototype` to `Foo.prototype`, which somewhat resembles the concept of Parent-Child class inheritance. *Resembles*, except of course for the direction of the arrows, which show these are delegation links rather than copy operations.

And, here's the typical "prototype style" code that creates such links:

```js
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

function Bar(name,label) {
	Foo.call( this, name );
	this.label = label;
}

// here, we make a new `Bar.prototype`
// linked to `Foo.prototype`
Bar.prototype = Object.create( Foo.prototype );

// Beware! Now `Bar.prototype.constructor` is gone,
// and might need to be manually "fixed" if you're
// in the habit of relying on such properties!

Bar.prototype.myLabel = function() {
	return this.label;
};

var a = new Bar( "a", "obj a" );

a.myName(); // "a"
a.myLabel(); // "obj a"
```

**Note:** To understand why `this` points to `a` in the above code snippet, see Chapter 2.

The important part is `Bar.prototype = Object.create( Foo.prototype )`. `Object.create(..)` *creates* a "new" object out of thin air, and links that new object's internal `[[Prototype]]` to the object you specify (`Foo.prototype` in this case).

In other words, that line says: "make a *new* 'Bar dot prototype' object that's linked to 'Foo dot prototype'."

When `function Bar() { .. }` is declared, `Bar`, like any other function, has a `.prototype` link to its default object. But *that* object is not linked to `Foo.prototype` like we want. So, we create a *new* object that *is* linked as we want, effectively throwing away the original incorrectly-linked object.

**Note:** A common mis-conception/confusion here is that either of the following approaches would *also* work, but they do not work as you'd expect:

```js
// doesn't work like you want!
Bar.prototype = Foo.prototype;

// works kinda like you want, but with
// side-effects you probably don't want :(
Bar.prototype = new Foo();
```

`Bar.prototype = Foo.prototype` doesn't create a new object for `Bar.prototype` to be linked to. It just makes `Bar.prototype` be another reference to `Foo.prototype`, which effectively links `Bar` directly to **the same object as** `Foo` links to: `Foo.prototype`. This means when you start assigning, like `Bar.prototype.myLabel = ...`, you're modifying **not a separate object** but *the* shared `Foo.prototype` object itself, which would affect any objects linked to `Foo.prototype`. This is almost certainly not what you want. If it *is* what you want, then you likely don't need `Bar` at all, and should just use only `Foo` and make your code simpler.

`Bar.prototype = new Foo()` **does in fact** create a new object which is duly linked to `Foo.prototype` as we'd want. But, it uses the `Foo(..)` "constructor call" to do it. If that function has any side-effects (such as logging, changing state, registering against other objects, **adding data properties to `this`**, etc.), those side-effects happen at the time of this linking (and likely against the wrong object!), rather than only when the eventual `Bar()` "descendents" are created, as would likely be expected.

So, we're left with using `Object.create(..)` to make a new object that's properly linked, but without having the side-effects of calling `Foo(..)`. The slight downside is that we have to create a new object, throwing the old one away, instead of modifying the existing default object we're provided.

It would be *nice* if there was a standard and reliable way to modify the linkage of an existing object. Prior to ES6, there's a non-standard and not fully-cross-browser way, via the `.__proto__` property, which is settable. ES6 adds a `Object.setPrototypeOf(..)` helper utility, which does the trick in a standard and predictable way.

Compare the pre-ES6 and ES6-standardized techniques for linking `Bar.prototype` to `Foo.prototype`, side-by-side:

```js
// pre-ES6
// throws away default existing `Bar.prototype`
Bar.prototype = Object.create( Foo.prototype );

// ES6+
// modifies existing `Bar.prototype`
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

Ignoring the slight performance disadvantage (throwing away an object that's later garbage collected) of the `Object.create(..)` approach, it's a little bit shorter and may be perhaps a little easier to read than the ES6+ approach. But it's probably a syntactic wash either way.

### Inspecting "Class" Relationships

What if you have an object like `a` and want to find out what object (if any) it delegates to? Inspecting an instance (just an object in JS) for its inheritance ancestry (delegation linkage in JS) is often called *introspection* (or *reflection*) in traditional class-oriented environments.

Consider:

```js
function Foo() {
	// ...
}

Foo.prototype.blah = ...;

var a = new Foo();
```

How do we then introspect `a` to find out its "ancestry" (delegation linkage)? The first approach embraces the "class" confusion:

```js
a instanceof Foo; // true
```

The `instanceof` operator takes a plain object as its left-hand operand and a **function** as its right-hand operand. The question `instanceof` answers is: **in the entire `[[Prototype]]` chain of `a`, does the object arbitrarily pointed to by `Foo.prototype` ever appear?**

Unfortunately, this means that you can only inquire about the "ancestry" of some object (`a`) if you have some **function** (`Foo`, with its attached `.prototype` reference) to test with. If you have two arbitrary objects, say `a` and `b`, and want to find out if *the objects* are related to each other through a `[[Prototype]]` chain, `instanceof` alone can't help.

**Note:** If you use the built-in `.bind(..)` utility to make a hard-bound function (see Chapter 2), the function created will not have a `.prototype` property. Using `instanceof` with such a function transparently substitutes the `.prototype` of the *target function* that the hard-bound function was created from.

It's fairly uncommon to use hard-bound functions as "constructor calls", but if you do, it will behave as if the original *target function* was invoked instead, which means that using `instanceof` with a hard-bound function also behaves according to the original function.

This snippet illustrates the ridiculousness of trying to reason about relationships between **two objects** using "class" semantics and `instanceof`:

```js
// helper utility to see if `o1` is
// related to (delegates to) `o2`
function isRelatedTo(o1, o2) {
	function F(){}
	F.prototype = o2;
	return o1 instanceof F;
}

var a = {};
var b = Object.create( a );

isRelatedTo( b, a ); // true
```

Inside `isRelatedTo(..)`, we borrow a throw-away function `F`, reassign its `.prototype` to arbitrarily point to some object `o2`, then ask if `o1` is an "instance of" `F`. Obviously `o1` isn't *actually* inherited or descended or even constructed from `F`, so it should be clear why this kind of exercise is silly and confusing. **The problem comes down to the awkwardness of class semantics forced upon JavaScript**, in this case as revealed by the indirect semantics of `instanceof`.

The second, and much cleaner, approach to `[[Prototype]]` reflection is:

```js
Foo.prototype.isPrototypeOf( a ); // true
```

Notice that in this case, we don't really care about (or even *need*) `Foo`, we just need an **object** (in our case, arbitrarily labeled `Foo.prototype`) to test against another **object**. The question `isPrototypeOf(..)` answers is: **in the entire `[[Prototype]]` chain of `a`, does `Foo.prototype` ever appear?**

Same question, and exact same answer. But in this second approach, we don't actually need the indirection of referencing a **function** (`Foo`) whose `.prototype` property will automatically be consulted.

We *just need* two **objects** to inspect a relationship between them. For example:

```js
// Simply: does `b` appear anywhere in
// `c`s [[Prototype]] chain?
b.isPrototypeOf( c );
```

Notice, this approach doesn't require a function ("class") at all. It just uses object references directly to `b` and `c`, and inquires about their relationship. In other words, our `isRelatedTo(..)` utility above is built-in to the language, and it's called `isPrototypeOf(..)`.

We can also directly retrieve the `[[Prototype]]` of an object. As of ES5, the standard way to do this is:

```js
Object.getPrototypeOf( a );
```

And you'll notice that object reference is what we'd expect:

```js
Object.getPrototypeOf( a ) === Foo.prototype; // true
```

Most browsers (not all!) have also long supported a non-standard alternate way of accessing the internal `[[Prototype]]`:

```js
a.__proto__ === Foo.prototype; // true
```

The strange `.__proto__` (not standardized until ES6!) property "magically" retrieves the internal `[[Prototype]]` of an object as a reference, which is quite helpful if you want to directly inspect (or even traverse: `.__proto__.__proto__...`) the chain.

Just as we saw earlier with `.constructor`, `.__proto__` doesn't actually exist on the object you're inspecting (`a` in our running example). In fact, it exists (non-enumerable; see Chapter 2) on the built-in `Object.prototype`, along with the other common utilities (`.toString()`, `.isPrototypeOf(..)`, etc).

Moreover, `.__proto__` looks like a property, but it's actually more appropriate to think of it as a getter/setter (see Chapter 3).

Roughly, we could envision `.__proto__` implemented (see Chapter 3 for object property definitions) like this:

```js
Object.defineProperty( Object.prototype, "__proto__", {
	get: function() {
		return Object.getPrototypeOf( this );
	},
	set: function(o) {
		// setPrototypeOf(..) as of ES6
		Object.setPrototypeOf( this, o );
		return o;
	}
} );
```

So, when we access (retrieve the value of) `a.__proto__`, it's like calling `a.__proto__()` (calling the getter function). *That* function call has `a` as its `this` even though the getter function exists on the `Object.prototype` object (see Chapter 2 for `this` binding rules), so it's just like saying `Object.getPrototypeOf( a )`.

`.__proto__` is also a settable property, just like using ES6's `Object.setPrototypeOf(..)` shown earlier. However, generally you **should not change the `[[Prototype]]` of an existing object**.

There are some very complex, advanced techniques used deep in some frameworks that allow tricks like "subclassing" an `Array`, but this is commonly frowned on in general programming practice, as it usually leads to *much* harder to understand/maintain code.

**Note:** As of ES6, the `class` keyword will allow something that approximates "subclassing" of built-in's like `Array`. See Appendix A for discussion of the `class` syntax added in ES6.

The only other narrow exception (as mentioned earlier) would be setting the `[[Prototype]]` of a default function's `.prototype` object to reference some other object (besides `Object.prototype`). That would avoid replacing that default object entirely with a new linked object. Otherwise, **it's best to treat object `[[Prototype]]` linkage as a read-only characteristic** for ease of reading your code later.

**Note:** The JavaScript community unofficially coined a term for the double-underscore, specifically the leading one in properties like `__proto__`: "dunder". So, the "cool kids" in JavaScript would generally pronounce `__proto__` as "dunder proto".

## Object Links

As we've now seen, the `[[Prototype]]` mechanism is an internal link that exists on one object which references some other object.

This linkage is (primarily) exercised when a property/method reference is made against the first object, and no such property/method exists. In that case, the `[[Prototype]]` linkage tells the engine to look for the property/method on the linked-to object. In turn, if that object cannot fulfill the look-up, its `[[Prototype]]` is followed, and so on. This series of links between objects forms what is called the "prototype chain".

### `Create()`ing Links

We've thoroughly debunked why JavaScript's `[[Prototype]]` mechanism is **not** like *classes*, and we've seen how it instead creates **links** between proper objects.

What's the point of the `[[Prototype]]` mechanism? Why is it so common for JS developers to go to so much effort (emulating classes) in their code to wire up these linkages?

Remember we said much earlier in this chapter that `Object.create(..)` would be a hero? Now, we're ready to see how.

```js
var foo = {
	something: function() {
		console.log( "Tell me something good..." );
	}
};

var bar = Object.create( foo );

bar.something(); // Tell me something good...
```

`Object.create(..)` creates a new object (`bar`) linked to the object we specified (`foo`), which gives us all the power (delegation) of the `[[Prototype]]` mechanism, but without any of the unnecessary complication of `new` functions acting as classes and constructor calls, confusing `.prototype` and `.constructor` references, or any of that extra stuff.

**Note:** `Object.create(null)` creates an object that has an empty (aka, `null`) `[[Prototype]]` linkage, and thus the object can't delegate anywhere. Since such an object has no prototype chain, the `instanceof` operator (explained earlier) has nothing to check, so it will always return `false`. These special empty-`[[Prototype]]` objects are often called "dictionaries" as they are typically used purely for storing data in properties, mostly because they have no possible surprise effects from any delegated properties/functions on the `[[Prototype]]` chain, and are thus purely flat data storage.

We don't *need* classes to create meaningful relationships between two objects. The only thing we should **really care about** is objects linked together for delegation, and `Object.create(..)` gives us that linkage without all the class cruft.

#### `Object.create()` Polyfilled

`Object.create(..)` was added in ES5. You may need to support pre-ES5 environments (like older IE's), so let's take a look at a simple **partial** polyfill for `Object.create(..)` that gives us the capability that we need even in those older JS environments:

```js
if (!Object.create) {
	Object.create = function(o) {
		function F(){}
		F.prototype = o;
		return new F();
	};
}
```

This polyfill works by using a throw-away `F` function and overriding its `.prototype` property to point to the object we want to link to. Then we use `new F()` construction to make a new object that will be linked as we specified.

This usage of `Object.create(..)` is by far the most common usage, because it's the part that *can be* polyfilled. There's an additional set of functionality that the standard ES5 built-in `Object.create(..)` provides, which is **not polyfillable** for pre-ES5. As such, this capability is far-less commonly used. For completeness sake, let's look at that additional functionality:

```js
var anotherObject = {
	a: 2
};

var myObject = Object.create( anotherObject, {
	b: {
		enumerable: false,
		writable: true,
		configurable: false,
		value: 3
	},
	c: {
		enumerable: true,
		writable: false,
		configurable: false,
		value: 4
	}
} );

myObject.hasOwnProperty( "a" ); // false
myObject.hasOwnProperty( "b" ); // true
myObject.hasOwnProperty( "c" ); // true

myObject.a; // 2
myObject.b; // 3
myObject.c; // 4
```

The second argument to `Object.create(..)` specifies property names to add to the newly created object, via declaring each new property's *property descriptor* (see Chapter 3). Because polyfilling property descriptors into pre-ES5 is not possible, this additional functionality on `Object.create(..)` also cannot be polyfilled.

The vast majority of usage of `Object.create(..)` uses the polyfill-safe subset of functionality, so most developers are fine with using the **partial polyfill** in pre-ES5 environments.

Some developers take a much stricter view, which is that no function should be polyfilled unless it can be *fully* polyfilled. Since `Object.create(..)` is one of those partial-polyfill'able utilities, this narrower perspective says that if you need to use any of the functionality of `Object.create(..)` in a pre-ES5 environment, instead of polyfilling, you should use a custom utility, and stay away from using the name `Object.create` entirely. You could instead define your own utility, like:

```js
function createAndLinkObject(o) {
	function F(){}
	F.prototype = o;
	return new F();
}

var anotherObject = {
	a: 2
};

var myObject = createAndLinkObject( anotherObject );

myObject.a; // 2
```

I do not share this strict opinion. I fully endorse the common partial-polyfill of `Object.create(..)` as shown above, and using it in your code even in pre-ES5. I'll leave it to you to make your own decision.

### Links As Fallbacks?

It may be tempting to think that these links between objects *primarily* provide a sort of fallback for "missing" properties or methods. While that may be an observed outcome, I don't think it represents the right way of thinking about `[[Prototype]]`.

Consider:

```js
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};

var myObject = Object.create( anotherObject );

myObject.cool(); // "cool!"
```

That code will work by virtue of `[[Prototype]]`, but if you wrote it that way so that `anotherObject` was acting as a fallback **just in case** `myObject` couldn't handle some property/method that some developer may try to call, odds are that your software is going to be a bit more "magical" and harder to understand and maintain.

That's not to say there aren't cases where fallbacks are an appropriate design pattern, but it's not very common or idiomatic in JS, so if you find yourself doing so, you might want to take a step back and reconsider if that's really appropriate and sensible design.

**Note:** In ES6, an advanced functionality called `Proxy` is introduced which can provide something of a "method not found" type of behavior. `Proxy` is beyond the scope of this book, but will be covered in detail in a later book in the *"You Don't Know JS"* series.

**Don't miss an important but nuanced point here.**

Designing software where you intend for a developer to, for instance, call `myObject.cool()` and have that work even though there is no `cool()` method on `myObject` introduces some "magic" into your API design that can be surprising for future developers who maintain your software.

You can however design your API with less "magic" to it, but still take advantage of the power of `[[Prototype]]` linkage.

```js
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};

var myObject = Object.create( anotherObject );

myObject.doCool = function() {
	this.cool(); // internal delegation!
};

myObject.doCool(); // "cool!"
```

Here, we call `myObject.doCool()`, which is a method that *actually exists* on `myObject`, making our API design more explicit (less "magical"). *Internally*, our implementation follows the **delegation design pattern** (see Chapter 6), taking advantage of `[[Prototype]]` delegation to `anotherObject.cool()`.

In other words, delegation will tend to be less surprising/confusing if it's an internal implementation detail rather than plainly exposed in your API interface design. We will expound on **delegation** in great detail in the next chapter.

## Review (TL;DR)

When attempting a property access on an object that doesn't have that property, the object's internal `[[Prototype]]` linkage defines where the `[[Get]]` operation (see Chapter 3) should look next. This cascading linkage from object to object essentially defines a "prototype chain" (somewhat similar to a nested scope chain) of objects to traverse for property resolution.

All normal objects have the built-in `Object.prototype` as the top of the prototype chain (like the global scope in scope look-up), where property resolution will stop if not found anywhere prior in the chain. `toString()`, `valueOf()`, and several other common utilities exist on this `Object.prototype` object, explaining how all objects in the language are able to access them.

The most common way to get two objects linked to each other is using the `new` keyword with a function call, which among its four steps (see Chapter 2), it creates a new object linked to another object.

The "another object" that the new object is linked to happens to be the object referenced by the arbitrarily named `.prototype` property of the function called with `new`. Functions called with `new` are often called "constructors", despite the fact that they are not actually instantiating a class as *constructors* do in traditional class-oriented languages.

While these JavaScript mechanisms can seem to resemble "class instantiation" and "class inheritance" from traditional class-oriented languages, the key distinction is that in JavaScript, no copies are made. Rather, objects end up linked to each other via an internal `[[Prototype]]` chain.

For a variety of reasons, not the least of which is terminology precedent, "inheritance" (and "prototypal inheritance") and all the other OO terms just do not make sense when considering how JavaScript *actually* works (not just applied to our forced mental models).

Instead, "delegation" is a more appropriate term, because these relationships are not *copies* but delegation **links**.
