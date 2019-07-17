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

Ou seja, a cadeia `[[Prototype]]` é consultada, uma ligação por vez, quando você realiza buscas de propriedades de diferentes maneiras. As buscas param assim que a propriedade é encontrada ou assim que a cadeia termina. 

### `Object.prototype`

Mas *onde* exatamente a cadeia `[[Prototype]]` "termina"?

No topo de toda cadeia `[[Prototype]]` *normal* está objeto nativo `Object.prototype`. Este objeto inclui uma varidade de utilitários normalmente utilizados por todo JS, porque todos objetos normais (nativos, não extensões *self-hosted*) em JavaScript "descendem" (ou constam no topo de sua cadeia `[[Prototype]]`) do objeto `Object.prototype`. 

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
3. Se `foo` é encontrada no alto da cadeia `[[Prototype]]` e for um setter (veja Capítulo 3), então o setter sempre será chamado. Nenhuma `foo` será adicionada (ou sombreada) em `myObject`, e nem o setter de `foo` será redefinido.

Muitos desenvolvedores assumem que a atribuição de uma propriedade (`[[Put]]`) sempre resultará em sombreamento se a propriedade já existir no alto da cadeia `[[Prototype]]`, mas como se nota, isso só é verdadeiro em uma (a primeira) das três situações descritas acima.

Se você quer sombrear `foo` nos casos 2 e 3, não poderá usar `=` para fazer a atribuição, ao invés disso deve utilizar `Object.defineProperty(..)` (veja Capítulo 3) para adicionar `foo` em `myObject`.

**Nota:** O caso 2 pode ser o mais surpreendente dos três. A presença de uma propriedade *somente leitura* evita que uma propriedade de mesmo nome seja implicitamente criada (sombreada) em um nível mais baixo da cadeia `[[Prototype]]`. O motivo para esta restrição é primariamente para reforçar a ilusão de se ter propriedades herdadas de classes. Se você pensar em `foo` em um nível alto da cadeia como tendo sido herdado (copiado para baixo) para `myObject`, então faz sentido impor a natureza não-gravável desta propriedade `foo` em `myObject`. Se você no entanto separar a ilusão e o fato, e reconhecer que nenhuma cópia de herança *realmente* ocorreu (veja Capítulos 4 e 5), é um pouco antinatural que `myObject` seja impedido de ter uma propriedade `foo` apenas porque algum outro objeto tinha um `foo` não-gravável nele. É ainda mais estranho que esta restrição só se aplique à designação `=`, mas não seja aplicada quando `Object.defineProperty (..)` é utilizado.    

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

## "Classes"

Neste momento, você deve estar imaginando: "*Por que* um objeto precisa ser ligado à um outro objeto?" Qual é o real benefício? Esta é uma pergunta muito pertinente de se fazer, mas primeiro precisamos entender o que `[[Prototype]]` **não é** antes de entendê-lo completamente e apreciar o que **é** e como pode ser útil.

Como explicamos no Capítulo 4, em JavaScript, não há padrões abstratos para objetos chamados de "classes" como no caso de linguagens orientadas à classes. JavaScript tem **apenas** objetos.

Na verdade, JavaScript é **quase único** entre as linguagens pelo fato de talvez ser a única linguagem com o direito de ser rotulada de "orientada à objetos", porque é uma entre uma lista bem pequena de linguagens em que objetos podem ser criados diretamente, sem a existência de uma classe.

Em JavaScript, classes não podem (já que não existem!) descrever o que um objeto pode fazer. O objeto define diretamente seu próprio comportamento. **Existe *apenas* o objeto.**

### Funções de "Classes"

Existe um tipo de comportamento peculiar em JavaScript que vem sendo descaradamente abusado por anos para *hackear* algo que somente *parece* com "classes". Nós examinaremos essa abordagem em detalhes. 

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

Quando `a` é criado ao chamar `new Foo()`, uma das coisas (ceja Capítulo 2 para todos os *quatro* passos) que acontecem é que `a` obtêm uma ligação `[[Prototype]]` interna com o objeto que `Foo.prototype` está apontando.

Pare um momento e pondere as implicações desta declaração.

Em linguagens orientadas à classes, multiplas **cópias** (também conhecidas como "instâncias") de uma classe podem ser criadas, como carimbar algo à partir de um molde. Como vimos no Capítulo 4, isso acontece porque o processo de instanciação (ou de herdar de) uma classe significa "copiar o plano de comportamento desta classe para um objeto físico", e isso é feito novamente para cada nova instância.  

Mas em JavaScript, não há ações de cópias deste tipo. Você não cria múltiplas instâncias de uma classe. Você cria múltiplos objetos que são *ligados* pelo `[[Prototype]]` com um objeto comum. Mas por padrão, nenhuma cópia ocorre, e portanto esses objetos acabam não sendo totalmente separados e nem desconectados um do outro, pelo contrário, estão bem ***ligados***. 

`new Foo()` resulta em um novo objeto (que chamamos de `a`), e **este** novo objeto `a` é internamente ligado por `[[Prototype]]` com o objeto `Foo.prototype`.  

**Nós acabamos com dois objetos, um ligado ao outro.** E *é isso*. Nós não instanciamos uma classe. Nós certamente não fizemos nenhuma cópia de comportamento de uma "classe" para um objeto concreto. Nós só fizemos com que dois objetos fossem ligados um ao outro.

Na verdade, o segredo, que ilude a maioria dos desenvolvedores JS, é que a chamada da função `new Foo()` não tem praticamente nenhuma relação *direta* com o processo de criar a ligação. **Foi uma espécie de efeito colateral acidental.** `new Foo()` é uma forma indireta de se conseguir o que queremos: **um novo objeto ligado à um outro objeto**. 

Nós conseguimos ter o que queremos de uma forma mais *direta*? **Sim!** o herói é `Object.create(..)`. Mas nós chegaremos lá daqui à pouco. 

#### O que está em um nome?

Em JavaScript, nós não fazemos *cópias* de um objeto ("classe") para outro ("instância"). Nós criamos *ligações* entre objetos. Para o mecanismo `[[Prototype]]`, visualmente, as setas movem da direita pra esquerda, e de baixo pra cima.
<img src="fig3.png">

Este mecanismo é frequentemente chamado de "herança prototípica" (nós examinaremos o código em detalhes à seguir), e é comumente considerado como a versão em linguagem dinâmica para a "herança clássica". É uma tentativa de se apoiar no entendimento comum do que "herança" significa no mundo orientado à classes, mas *ajustar* (**leia: pavimentar**) a semântica compreendida para se adequar ao script dinâmico.

A palavra "herança" tem um significado poderoso (veja Capítulo 4), com muito precedente mental. Meramente adicionar o termo "prototípica" para distinguir um comportamento *que é na verdade quase oposto* em JavaScript deixou um rastro de quase duas décadas de confusão.

Eu gosto de dizer que o rótulo de "protótipo" para uma "herança" inverte drásticamente o seu real significado, é como segurar uma laranja em uma mão, uma maçã na outra, e insistir para que chamem a maçã de "laranja vermelha". Não importa o quão confuso seja o rótulo que eu utilize, isso não muda o *fato* que uma fruta é maçã e a outra é laranja.

A melhor abordagem é simplesmente chamar uma maçã de maçã -- para usar a terminologia mais precisa e direta. Isso facilita o entendimento tanto em suas similaridades quanto suas **muitas diferenças**, porque tudo que temos é um entendimento simples e compartilhado do que "maçã" significa. 

Graças à confusão e conflação de termos, acredito que o próprio rótulo de "herança prototípica" (e a tentativa de se aplicar incorretamente toda a sua terminologia de orientação à classes associada, como "classe", "construtor", "instância", "polimorfismo", etc.) tem causado efeitos **mais negativos que positivos** em explicar como o mecanismo de JavaScript *realmente* funciona. 

"Herança" implica em uma operação de *cópia*, e JavaScript não copia propriedades de objetos (nativamente, por padrão). Ao invés disso, JS cria uma ligação entre dois objetos, onde um objeto pode essencialmente *delegar* acesso às propriedades/funções para outro objeto. "Delegação" (veja Capítulo 6) é um termo muito mais preciso para o mecanismo de ligação entre objetos do JavaScript.

Outro termo que às vezes é jogado em JavaScript é "herança diferencial". A ideia aqui é que descrevemos o comportamento de um objeto em termos do que é *diferente* de um descritor mais generalizado. Por exemplo, você explica que um carro é um tipo de veículo, mas que tem exatamente 4 rodas, ao invés de descrever todos os detalhes do que compõe um veículo em geral (motor, etc). 

Se você tentar pensar em qualquer objeto em JS como a soma total de todo o comportamento que está *disponível* via delegação, e **em sua mente você nivela** todo esse comportamento em apenas uma *coisa* tangível, então você pode (de certa forma) ver como "herança diferencial" poderá se encaixar.

Mas assim como "herança prototípica", "herança diferencial" finge que seu modelo mental é mais importante que o que está fisicamente acontecendo na linguagem. Ela negligencia o fato que o objeto `B` não é realmente diferencialmente construído, mas é construído com características específicas definidas, ao lado de "buracos" onde nada é definido. É nesses "buracos" (lacunas, ou falta de definição) que a delegação *pode* assumir e, na mosca, "preenchê-los" com o comportamento delegado.

O objeto não é, por padrão nativo, nivelado em um único objeto diferencial, ***através de cópia***, que o modelo mental de "herança diferencial" implica. Como tal, "herança diferencial" não é apenas um ajuste natural para descrever como o mecanismo `[[Prototype]]` do JavaScript realmente funciona.

Você *pode escolher* em preferir a terminologia de "herança diferencial" e o modelo mental, por questão de gosto, mas não há como negar o fato de que isso se encaixa *apenas* nas acrobacias mentais dentro de sua mente, e não do comportamento físico da engine.

### "Construtores"

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

O objeto `Foo.prototype` por padrão (no momento da declaração na linha 1 do código) recebe uma propriedade pública, não enumerável (veja Capítulo 3) chamada `.constructor`, e essa propriedade é uma referência de volta à função (`Foo` neste caso) em que o objeto está associado. Além disso, vemos que o objeto `a` criado através da chamada do "construtor" `new Foo ()` *parece* ter também uma propriedade nele chamada `.constructor` que similarmente aponta para "a função que o criou".  

**Nota:** Isso não é realmente verdade. `a` não possui uma propriedade `.constructor`, e ainda que `a.constructor` de fato funcione para a função `Foo`, "construtor" **não significa realmente** "foi construído por", como parece. Nós explicaremos essa estranha situação em breve.   

Ah sim, outra coisa... por convenção no mundo JavaScript, "classes" são nomeadas com letra maiúscula, então o fato de ser `Foo` ao invés de `foo` é uma forte pista de que a intenção é de que seja uma "classe". Isso é totalmente óbvio pra você, certo!?

**Nota:** Essa convenção é tão forte que muitos linters de JS de fato *reclamam* se você chama `new` em um método com letra minúscula, ou se não chamamos `new` em uma função que por acaso comece com uma letra maiúscula. Isso meio que confunde a ideia de que lutamos tanto para obter uma (falsa) "orientação à classe" *do jeito certo* em JavaScript em que criamos regras de linter para assegurar que usamos letras maiúsculas, mesmo que letra maiúscula não signifique ***absolutamente* nada** para o motor (engine) JS. 

#### Construtor ou Chamada?

No código acima, é tentador pensar que `Foo` é um "construtor", porque nós o chamamos com `new` e observamos que isso "constrói" um objeto.

Na realidade, `Foo` não é mais "construtor" que qualquer outra função em seu programa. Funções por si só **não** são construtores. Entretanto, quando se coloca a palavra-chave `new` em frente à chamada de uma função normal, isso faz com que a função chame uma "chamada de construtor". Na verdade, `new` meio que se apropria de qualquer função normal e a chama de uma forma que constrói um objeto, **junto com qualquer outra coisa que irá fazer**.

Por exemplo:

```js
function NothingSpecial() {
	console.log( "Don't mind me!" );
}

var a = new NothingSpecial();
// "Don't mind me!"

a; // {}
```

`NothingSpecial` é apenas uma função normal, mas quando chamada com `new`, ela *constrói* um objeto, quase como um efeito colateral, que nós por acaso atribuímos à `a`. A **chamada** foi uma *chamada de construtor*, mas `NothingSpecial` não é, por si só, um *construtor*.  

Em outras palavras, em JavaScript, é mais apropriado dizer que um "construtor" é **qualquer função chamada através de uma palavra-chave `new`** na frente.

Funções não são construtores, mas chamadas de funções são "chamadas de construtores" se e apenas se `new` é utilizado.

### Mecânicas

São *esses* os únicos gatilhos comuns para as malfadadas discussões sobre "classes" em JavaScript?

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

Este código mostra dois truques adicionais "orientados à classe" em jogo:

1. `this.name = name`: adiciona a propriedade `.name` em cada objeto (`a` e `b`, respectivamente; veja Capítulo 2 sobre ligação `this`), similar em como instâncias de classes encapsulam dados.

2. `Foo.prototype.myName = ...`: talvez seja a técnica mais interessante, ela adiciona uma propriedade (função) para o objeto `Foo.prototype`. Agora, talvez surpreendentemente, `a.myName()` funciona. Como?   

No código acima, é fortemente tentador pensar que quando `a` e `b` são criados, as propriedades/funções no objeto `Foo.prototype` são *copiadas* sobre cada objeto `a` e `b`. **Entretanto, não é isso o que acontece.**

No início deste capítulo, nós explicamos a ligação `[[Prototype]]`, e cada passo que ela segue para realizar as buscas caso uma referência da propriedade não seja diretamente encontrada no objeto, como parte do padrão do algoritmo `[[Get]]`.

Então, por virtude do que criamos, cada `a` e `b` acabam com uma ligação `[[Prototype]]` interna com `Foo.prototype`. Quando `myName` não é encontrada em `a` ou `b`, respectivamente, é encontrada em seu lugar (através de delegação, veja Capítulo 6) em `Foo.prototype`. 

#### "Construtor" Redux

Você se lembra da discussão mais cedo sobre a propriedade `.constructor`, e como ela *faz parecer* que `a.constructor === Foo` sendo verdadeiro significa que `a` tem de fato uma propriedade `.constructor` em si, apontando para `Foo`? **Isso não está correto.**   

Essa é apenas uma infeliz confusão. Na realidade, a refêrencia de `.constructor` também é *delegada* para `Foo.prototype`, que **acontece de**, por padrão, ter um `.constructor` que aponta para `Foo`.

*Parece* muito conveniente que um objeto `a` "construído por" `Foo` deveria ter acesso à propriedade `.constructor` que aponta para `Foo`. Mas isso nada mais é que uma sensação falsa de segurança. É um feliz acidente, quase tangencial, que `a.constructor` *por acaso* aponta para `Foo` através desta delegação `[[Prototype]]` padrão. Existem diversas maneiras da malfadada suposição de que `.constructor` significa "foi constrúido por" voltar para te assombrar.

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

O que está acontecendo? `a1` não possui nenhuma propriedade `.constructor`, então ele delega até a cadeia `[[Prototype]]` para `Foo.prototype`. Mas este objeto também não tem um `.constructor` (como o objeto padrão `Foo.prototype` teria!), então ele continua delegando, dessa vez até `Object.prototype`, o topo da cadeia de delegação. *Este* objeto de fato possui um `.constructor` em si, que aponta para a função nativa `Object(..)`.

**Equívoco encontrado.**

Claro, você pode adicionar `.constructor` de volta ao objeto `Foo.prototype`, mas isso requer trabalho manual, especialmente se você quer que tenha um comportamento nativo e que seja não enumerável (veja Capítulo 3).

Por exemplo:

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // cria um novo objeto prototype

// É preciso "consertar" de forma apropriada a propriedade 
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

`.constructor` não é uma propriedade imutável mágica. Ela *é* não enumerável (veja código abaixo), mas seu valor é gravável (pode ser alterado), e além disso, você pode adicionar ou sobreescrever (de forma intencional ou acidental) uma propriedade com nome de `constructor` em qualquer objeto que esteja em qualquer cadeia `[[Prototype]]`, com o valor que julgar necessário.

Em virtude de como o algoritmo `[[Get]]` percorre a cadeia `[[Prototype]]`, uma referência da propriedade `.constructor` encontrada em qualquer lugar pode se resolver de forma bem diferente do que se espera.

Vê como é arbitrário seu real significado?

O resultado? Alguma referência arbitrária de objeto-propriedade como `a1.constructor` não pode ser *confiada* como sendo a referência de função que se assume como padrão. Além disso, como veremos em breve, apenas por simples omissão, `a1.constructor` pode até apontar pra algum lugar bem surpreendente e insensível.

`a1.constructor` não é nada confiável, e é uma referência insegura para se colocar em seu código. **Geralmente, essas referências devem ser evitadas quando possível.**

## "Herança (Prototípica)"

Nós vimos algumas aproximações de mecânicas de "classe" como normais na escrita de programas JavaScript. Mas as "classes" JavaScript seriam vazias se não tivéssemos também uma aproximação de "herança".

Na verdade, nós já vimos o mecânismo que geralmente é chamado de "herança prototípica" funcionando quando `a` foi capaz de "ser herdada de" `Foo.prototype`,e por isso recebe acesso à função `myName()`. Mas nós tradicionalmente pensamos em "herança" como sendo uma relação entre duas "classes", e não entre "classe" e "instância".

<img src="fig3.png">

Reveja essa figura exibida anteriormente, que mostra não só a delegação de um objeto (também conhecido como "instância") `a1` para o objeto `Foo.prototype`, como também de `Bar.prototype` para `Foo.prototype`, o que assemelha-se ao conceito de herança de classe Parent-Child. *Assemelha-se*, exceto claro pela direção das setas, que mostram que são ligações via delegações ao invés de operações de cópia.

Abaixo temos o típico código "estilo protótipo" que cria ligações como essas:

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

// aqui, nós criamos um novo `Bar.prototype`
// ligado à `Foo.prototype`
Bar.prototype = Object.create( Foo.prototype );

// Cuidado! Agora `Bar.prototype.constructor` se foi, 
// e pode precisar ser manualmente "consertado" se 
// você tiver o hábito de confiar em propriedades deste tipo!

Bar.prototype.myLabel = function() {
	return this.label;
};

var a = new Bar( "a", "obj a" );

a.myName(); // "a"
a.myLabel(); // "obj a"
```

**Nota:** Para entender por que `this` aponta para `a` no código acima, veja o Capítulo 2.

A parte importante é `Bar.prototype = Object.create( Foo.prototype )`. `Object.create(..)` *cria* um "novo" objeto do nada, e liga o `[[Prototype]]` interno deste objeto com o objeto que você especificar (`Foo.prototype` neste caso).

Em outras palavras, essa linha diz: "faça um *novo* objeto 'Bar ponto prototype' que seja ligado à 'Foo ponto prototype'."   

Quando `function Bar() { .. }` é declarado, `Bar`, como qualquer outra função, tem uma ligação `.prototype` com seu objeto padrão. Mas *esse* objeto não está ligado à `Foo.prototype` como queremos que seja. Então, nós criamos um *novo* objeto que *é* ligado como queremos, efetivamente jogando fora seu objeto original que foi incorretamente ligado.    

**Nota:** Um equívoco ou confusão comum aqui é que qualquer uma das abordagens a seguir *também* podem funcionar, mas não da forma que se espera:

```js
// não funciona como você quer!
Bar.prototype = Foo.prototype;

// funciona mais ou menos como quer, mas com
// efeitos colaterais que você provavelmente não quer :(
Bar.prototype = new Foo();
```

`Bar.prototype = Foo.prototype` não cria um novo objeto para ser ligado à `Bar.prototype`. Apenas faz com que `Bar.prototype` seja outra referência para `Foo.prototype`, o que efetivamente liga `Bar` diretamente com **o mesmo objeto que** `Foo` se conecta: `Foo.prototype`. Isso significa que quando você começa a atribuir, como em `Bar.prototype.myLabel = ...`, você está modificando **não um objeto separado** mas *o próprio* objeto `Foo.prototype` compartilhado, o que poderia afetar qualquer objeto ligado à `Foo.prototype`. Isso é quase certo que não seja o que quer. Se *é* o que você quer, então você provavelmente nem sequer precisa de `Bar`, e deve apenas usar `Foo` e simplificar seu código.

`Bar.prototype = new Foo()` **cria de fato** um novo objeto que é devidamente ligado à `Foo.prototype` como gostaríamos. Mas, ele usa a "chamada de construtor" `Foo(..)` para fazer isso. Se esta função tiver qualquer efeito colateral (como logging, mudança de estado, registro contra outros objetos, **adição de propriedades para `this`**, etc.), esses efeitos colaterais acontecerão no momento dessa ligação (e provavelmente no objeto errado!), ao invés de apenas quando eventuais "descendentes" de `Bar()` são criados, como provavelmente é esperado.

Então, acabamos usando `Object.create(..)` para criar um novo objeto que esteja ligado da forma apropriada, mas sem os efeitos colaterais de se chamar `Foo(..)`. O lado ligeiramente negativo disso é que temos que criar um novo objeto, jogando fora o antigo, ao invés de modificar o objeto padrão existente que havíamos criado.

Seria *bom* se existisse uma maneira padrão e confiável de se modificar a ligação de um objeto existente. Antes do ES6, existia uma maneira fora do padrão e não inteiramente cross-browser, através da propriedade `.__proto__`, que é configurável. ES6 adiciona o utilitário auxiliar `Object.setPrototypeOf(..)`, que consegue fazer esse truque de forma mais padrão e previsível.

Compare as técnicas pré-ES6 e no padrão ES6 de ligar `Bar.prototype` à `Foo.prototype`, lado à lado:

```js
// pré-ES6
// joga fora o `Bar.prototype` padrão existente
Bar.prototype = Object.create( Foo.prototype );

// ES6+
// modifica o `Bar.prototype` existente
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

Ignorando a pequena desvantagem na performance (jogando fora um objeto que mais tarde será lixo coletado) da abordagem `Object.create(..)`, ela é um pouco menor e talvez seja um pouco mais fácil de se ler que a abordagem em ES6+. Mas é uma provável limpeza na síntaxe de qualquer forma.

### Inspecionando o Relacionamento de "Classes"

E se você tiver um objeto como `a` e quer descobrir para qual objeto (se houver algum) ele delega? Inspecionar uma instância (que é só um objeto em JS) para buscar sua herança ancestral (que é uma ligação por delegação em JS) é frequentemente chamado de *introspecção* (ou *reflexão*) em ambientes tradicionais orientados à classes.

Considere:

```js
function Foo() {
	// ...
}

Foo.prototype.blah = ...;

var a = new Foo();
```

Como introspectamos `a` para descobrir sua "ancestralidade" (ligação por delegação)? A primeira abordagem abraça a confusão de "classe":

```js
a instanceof Foo; // true
```

O operador `instanceof` pega um objeto simples como seu operando à esquerda e uma **função** como seu operando à direita. A questão que `instanceof` responde é: **em toda a cadeia `[[Prototype]]` de `a`, o objeto arbitrariamente apontado por `Foo.prototype` aparece alguma vez?**

Infelizmente, isso significa que você pode apenas questionar sobre a "ancestralidade" de algum objeto (`a`) se você tiver alguma **função** (`Foo`, que está anexada à referência `.prototype`) para poder testar. Se você tiver dois objetos arbitrários, digamos `a` e `b`, e quiser descobrir se *os objetos* estão relacionados entre si através da cadeia `[[Prototype]]`, o `instanceof` sozinho não poderá te ajudar. 

**Nota:** Se você usar o utilitário nativo `.bind(..)` para criar uma função hard-bound (veja Capítulo 2), a função criada não terá uma propriedade `.prototype`. Usando `instanceof` com este tipo de função substitui de forma transparente o `.prototype` da *função de destino* da qual a função hard-bound foi criada. 

É muito comum de se utilizar funções hard-bound como "chamadas de construtor", mas ao invés disso se você o fizer, irá se comportar como a *função de destino* original que foi invocada, o que significa que usar `instanceof` com uma função hard-bound também faz com que se comporte de acordo com a função original. 

Este código ilustra o quão rídicula é a tentativa de racionalizar sobre as relações entre **dois objetos** usando semânticas de "classe" e `instanceof`:  

```js
// utilitário de ajuda para ver se `o1`
// está relacionado (delegado) à `o2`
function isRelatedTo(o1, o2) {
	function F(){}
	F.prototype = o2;
	return o1 instanceof F;
}

var a = {};
var b = Object.create( a );

isRelatedTo( b, a ); // true
```

Dentro de `isRelatedTo(..)`, nós pegamos uma função descartável `F`, reatribuímos sua propriedade `.prototype` para apontar arbitrariamente para algum objeto `o2`, e em seguida perguntamos se `o1` é uma "instância de" `F`. Obviamente `o1` não é *realmente* herdada ou descende de ou até mesmo construída a partir de `F`, então deveria ser claro o porquê desse tipo de exercício ser tolo e confuso. **O problema aparece devido à embaraçosa semântica de classe forçada em cima do JavaScript**, neste caso revelada através da semântica de `instanceof`.

A segunda, e muito mais clara, abordagem para refletir `[[Prototype]]` é:

```js
Foo.prototype.isPrototypeOf( a ); // true
```

Perceba que neste caso, nós não nos importamos realmente (ou sequer *precisamos*) de `Foo`, nós só precisamos de um **objeto** (em nosso caso, arbitrariamente rotulado de `Foo.prototype`) para testar contra outro **objeto**. A questão que `isPrototypeOf(..)` responde é: **em toda a cadeia `[[Prototype]]` de `a`, alguma vez `Foo.prototype` é encontrado?**

Mesma questão, e exatamente a mesma resposta. Mas nesta segunda abordagem, nós não precisamos de verdade da indireção de referenciar uma **função** (`Foo`) cuja propriedade `.prototype` será automaticamente consultada.

Nós *só precisamos* de dois **objetos** para inspecionar uma relação entre eles. Por exemplo:

```js
// Simplesmente: `b` aparece em algum lugar
// da cadeia [[Prototype]] de `c`?
b.isPrototypeOf( c );
```

Perceba, essa abordagem não requer nenhuma função ("classe"). Ela só usa as referências de objeto diretamente para `b` e `c`, e pergunta sobre sua relação. Em outras palavras, nosso utilitário `isRelatedTo(..)` acima é nativo da linguagem, e é chamado de `isPrototypeOf(..)`.

Nós também podemos recuperar diretamente o `[[Prototype]]` de um objeto. Em ES5, a maneira padrão de fazer isso é:

```js
Object.getPrototypeOf( a );
```

E você irá notar que a referência do objeto é o que esperamos:

```js
Object.getPrototypeOf( a ) === Foo.prototype; // true
```

Muitos browsers (não todos!) também suportam há muito tempo uma alternativa fora do padrão para acesso ao `[[Prototype]]` interno:

```js
a.__proto__ === Foo.prototype; // true
```

O estranho como `.__proto__` (que não fazia parte do padrão até ES6!) "magicamente" recupera o `[[Prototype]]` interno de um objeto como uma referência, o que é bem útil se você quer inspecionar (ou até cruzar: `.__proto__.__proto__...`) diretamente a cadeia.

Assim como vimos anteriormente com `.constructor`, `.__proto__` não existe de fato no objeto que você está inspecionando (`a` no nosso exemplo atual). Na verdade, existe (não enumerável; veja Capítulo 2) no `Object.prototype` nativo, junto com outros utilitários comuns (`.toString()`, `.isPrototypeOf(..)`, etc).    

Além disso, `.__proto__` parece com uma propriedade, mas é mais apropriado considerá-la como um getter/setter (veja Capítulo 3).

À grosso modo, nós podemos visualizar `.__proto__` implementada (veja Capítulo 3 para definições de propriedade de objetos) desta forma:

```js
Object.defineProperty( Object.prototype, "__proto__", {
	get: function() {
		return Object.getPrototypeOf( this );
	},
	set: function(o) {
		// setPrototypeOf(..) a partir do ES6
		Object.setPrototypeOf( this, o );
		return o;
	}
} );
```

Então, quando nós acessamos (recuperamos o valor de) `a.__proto__`, é como se chamassemos `a.__proto__()` (chamando a função getter). *Essa* chamada de função possui `a` como seu `this` mesmo que a função getter exista no objeto `Object.prototype` (veja Capítulo 2 para as regras de vínculo de `this`), então é como dizer `Object.getPrototypeOf( a )`.

`.__proto__` também é uma propriedade configurável, assim como o uso de `Object.setPrototypeOf(..)` exibido anteriormente. Entretanto, geralmente você **não deve mudar o `[[Prototype]]` de um objeto existente**.

Existem algumas técnicas avançadas e muito complexas utilizadas no fundo de alguns frameworks que permitem truques como criar "subclasses" em um `Array`, mas em geral isso não é algo visto como uma boa prática de programação, já que leva há um entendimento e manutenção muito mais difíceis do código.

**Nota:** A partir do ES6, a palavra-chave `class` permitirá algo que que se aproxima da criação de "subclasses" para utilitários nativos como o `Array`. Veja o Apêndice A para a discussão da síntaxe `class` adicionada no ES6.  

A única pequena exceção (como mencionado antes) seria configurar o `[[Prototype]]` de um objeto `.prototype` como uma função padrão para referenciar algum outro objeto (além de `Object.prototype`). Isso evitaria substituir inteiramente o objeto padrão com um novo objeto ligado. Caso contrário, **é melhor tratar a ligação `[[Prototype]]` do objeto como uma característica de somente leitura** para facilitar a leitura de seu código mais tarde.

**Nota:** A comunidade JavaScript cunhou um termo não oficial para o underscore duplo, especificamente para propriedades como o favorito `__proto__`: "dunder". Então, os "descolados" em JavaScript geralmente pronunciam `__proto__` como "dunder proto".

## Ligações de Objeto

Como já vimos, o mecânismo `[[Prototype]]` é uma ligação interna que existe em um objeto que referencia algum outro objeto.

Essa ligação é (primariamente) exercida quando uma propriedade/método é feita contra o primeiro objeto, e tal propriedade/método não existe. Nesse caso, a ligação `[[Prototype]]` diz ao motor para procurar pela propriedade/método no objeto ligado. Por sua vez, se esse objeto não puder completar a busca, seu `[[Prototype]]` é seguido, e assim por diante. Essa série de ligações entre formas de objetos é o que chamados de "cadeia de protótipos".

### `create()` Criando ligações

Nós revelamos em detalhes porque o mecanismo `[[Prototype]]` de JavaScript **não** é o mesmo que *classes*, e vimos como ele ao invés disso cria **ligações** entre os objetos.

Qual o sentido do mecânismo `[[Prototype]]`? Por que é tão comum para desenvolvedores JS fazer tanto esforço (emulando classes) em seu código para criar essas ligações?

Lembra-se que dizemos bem cedo neste capítulo que `Object.create(..)` seria o herói? Agora, estamos preparados para ver como.

```js
var foo = {
	something: function() {
		console.log( "Tell me something good..." );
	}
};

var bar = Object.create( foo );

bar.something(); // Tell me something good...
```

`Object.create(..)` cria um novo objeto (`bar`) ligado ao objeto que especificamos (`foo`), o que nos dá todo o poder (delegação) do mecânismo `[[Prototype]]`, mas sem qualquer complicação desnecessária de funções `new` atuando como classes e chamadas de construtores, confundindo referências `.prototype` e `.constructor`, ou qualquer outra coisa a mais.

**Nota:** `Object.create(null)` cria um objeto que possui uma ligação `[[Prototype]]` vazia (também conhecida como `null`), e portanto esse objeto não pode delegar à lugar nenhum. Como um objeto desse tipo não tem uma cadeia de protótipo, o operador `instanceof` (explicado mais cedo) não tem nada pra checar, então sempre irá retornar `false`. Esses objetos especiais de `[[Prototype]]` vazios são frequentemente chamados de "dicionários" já que eles são tipicamente usados apenas para guardar dados em propriedades, na maioria das vezes porque eles não tem nenhum efeito surpresa vindo de qualquer propriedade/função delegada na cadeia `[[Prototype]]`, e por isso servem somente para armazenagem de dados.

Nós não *precisamos* de classes para criar relacionamentos relevantes entre dois objetos. A única coisa que **realmente devemos nos importar** são objetos ligados entre si por delegação, e `Object.create(..)` nos dá essa ligação sem toda a desnecessária complexidade de uma classe.    

#### `Object.create()` com Polyfill

`Object.create(..)` foi adicionado no ES5. Você pode precisar de suporte a ambientes pré-ES5 (como os antigos Internet Explorers), então vamos dar uma olhada em um simples polyfill **parcial** para `Object.create(..)` que nos dá a capacidade que precisamos mesmo nesses ambientes antigos de JS:

```js
if (!Object.create) {
	Object.create = function(o) {
		function F(){}
		F.prototype = o;
		return new F();
	};
}
```

Esse polyfill funciona atrávés do uso da função descartável `F` e sobrescrevendo sua propriedade `.prototype` para apontar para o objeto que desejamos criar uma ligação. Então nós usamos a construção `new F()` para criar um novo objeto que será ligado conforme especificamos.

Esse uso do `Object.create(..)` é de longe o mais comum, porque é a parte dele em que *pode* ser aplicado o polyfill. Existe uma série de funcionalidades adicionais que o `Object.create(..)` nativo de ES5 fornece, e que *não podem* ter o polyfill aplicado em ambientes pré-ES5. Devido à isso, sua capacidade é muito menos explorada. Para tornar isso mais claro, vamos dar uma olha nessa funcionalidade adicional:

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

O segundo argumento para o `Object.create(..)` especifica nomes de propriedades para serem adicionadas ao novo objeto criado, através da declaração do *descritor de propriedade* (veja Capítulo 3) de cada nova propriedade. Devido ao fato de não ser possível aplicar o pollyfill em descritores de propriedades pré-ES5, também não é possível aplicá-lo nessa funcionalidade adicional em `Object.create(..)`.

Em grande parte do uso de `Object.create(..)` também se faz uso de uma série de funcionalidades aptas para realizar o polyfill, então a maioria dos desenvolvedores não se importam em ter um **polyfill parcial** em ambientes pré-ES5.

Alguns desenvolvedores possuem uma visão mais rígida, de que nenhuma função pode ter o polyfill aplicado se não for por *completo*. Como `Object.create(..)` é um desses utilitários **parcialmente aptos para o polyfill**, essa perspectiva mais limitada diz que se você precisa de qualquer funcionalidade de um `Object.create(..)` em um ambiente pré-ES5, ao invés de polyfill, deve-se utilizar um utilitário customizado, e evitar completamente o uso do nome `Object.create`. Você deve ao invés disso definir seu próprio utilitário, como em:

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

Eu não partilho dessa visão mais rígida. Eu endosso completamente o uso de `Object.create(..)` mesmo sendo parcialmente apto ao polyfill como mostrado acima, e utilizá-lo em seu código mesmo em situações pré-ES5. Fica à seu critério para decidir o que é melhor para você.

### Ligações como Fallbacks?

Pode parecer tentador pensar que essas ligações entre objetos fornecem *primariamente* um tipo de fallback para propriedades ou métodos que "sumiram". Apesar de ser possível chegar à esse resultado, eu não penso que represente a forma correta de se pensar sobre `[[Prototype]]`.

Considere:

```js
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};

var myObject = Object.create( anotherObject );

myObject.cool(); // "cool!"
```

Esse código irá funcionar em virtude do `[[Prototype]]`, mas se você escreveu dessa forma para que `anotherObject` agisse como um fallback **só por garantia** `myObject` não poderá manipular alguma propriedade/método que algum desenvolvedor pode tentar chamar, as chances são de seu software se tornar um pouco mais "mágico" e mais difícil de se entender e manter.

Isso não quer dizer que não existam casos onde fallbacks sejam um design pattern apropriado, mas não é muito comum ou idiomático em JS, então se perceber que está fazendo isso, você deve parar pra pensar se isso está realmente apropriado e sensível ao design.

**Nota:** Em ES6, uma funcionalidade avançada chamada de `Proxy` foi introduzida e fornece um tipo de comportamento de "método não encontrado". `Proxy` está além do escopo deste livro, mas será abordado em detalhes em um futuro livro da série *"You Don't Know JS"*.

**Não perca um importante ponto mas que pode ter diferentes nuances por aqui.**

Projetando um software onde você tenha intenção de que um desenvolvedor, por exemplo, chame `myObject.cool()` e tenha isso funcionando mesmo que não exista o método `cool()` em `myObject`, introduz um tipo de "mágica" no desenho de sua API que pode causar surpresa em futuros desenvolvedores que irão realizar uma manutenção em seu software.   

Você pode no entanto desenhar sua API com menos "mágica", e ainda assim tirar vantagem do poder da ligação `[[Prototype]]`.

```js
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};

var myObject = Object.create( anotherObject );

myObject.doCool = function() {
	this.cool(); // delegação interna!
};

myObject.doCool(); // "cool!"
```

Aqui, nós chamamos `myObject.doCool()`, que é o método que *de fato existe* em `myObject`, fazendo com que o desenho de sua API seja mais explícito (menos "mágico"). *Internamente*, nossa implementação segue o **design pattern de delegação** (veja Capítulo 6), tirando vantagem da delegação de `[[Prototype]]` para `anotherObject.cool()`.

Em outras palavras, delegação tende à ser menos surpreendente/confusa se for um detalhe de implementação interno ao invés de abertamente exposto no desenho da interface de sua API. Nós iremos explorar **delegação** em muitos detalhes no próximo capítulo.

## Revisão (TL;DR)

Ao tentar acessar uma propriedade em um objeto que não possui essa propriedade, a ligação `[[Prototype]]` interna do objeto irá definir onde a operação `[[Get]]` (veja Capítulo 3) irá procurar na sequência. Essa ligação em cascata de um objeto para outro essencialmente define uma "cadeia de protótipos" (de certa forma similar à cadeia de escopo aninhada) de objetos à serem percorridos para resolução da propriedade.

Todos os objetos normais possuem o `Object.prototype` nativo como o topo da cadeia de protótipos (como o escopo global na buscar de escopo), onde a resolução da propriedade será interrompida caso a mesma não seja encontrada em nenhum lugar da cadeia. `toString()`, `valueOf()`, e muitos outros utilitários comuns existem nesse objeto `Object.prototype`, o que explica como todos os objetos na linguagem são capazes de acessá-los. 

A maneira mais comum de se ligar dois objetos entre si é usando a palavra-chave `new` com a chamada de uma função, que entre seus quatro passos (veja Capítulo 2) cria um novo objeto ligado à outro objeto.

O "outro objeto" à que esse novo objeto está ligado acontece de ser o objeto referenciado pela propriedade arbitrariamente nomeada de `.prototype` da função que foi chamada com `new`. Funções chamadas com `new` são frequentemente chamadas de "construtores", apesar do fato de que não estão realmente instanciando uma classe como *construtores* fazem em linguagens orientadas à classe tradicionais.

Enquanto esses mecânismos JavaScript podem lembrar "instanciamento de classe" e "herança de classe" de linguagens orientadas à classe tradicionais, a chave para distinguir isso é que em JavaScript, nenhuma cópia é realizada. Ao invés disso, objetos acabam ligados um ao outro através da cadeia `[[Prototype]]` interna.

Por uma variedade de motivos, não menos que uma terminologia precedente, "herança" (e "herança prototípica") e todos os termos de orientação à objetos não fazem nenhum sentido quando considera-se como JavaScript funciona *de fato* (e não apenas aplicados aos nossos modelos mentais forçados).

Ao invés disso, "delegação" é um termo muito mais apropriado, porque esses relacionamentos não são *cópias* mas sim **ligações** delegadas.
