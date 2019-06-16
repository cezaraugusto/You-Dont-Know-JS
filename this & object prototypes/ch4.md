# You Don't Know JS: *this* & Prototipagem de Objetos
# Chapter 4: Confundindo Objetos com "Classes"

Continuando com nossa exploração de objetos do capítulo anterior, é natural que agora nós demos atenção a "programação orientada a objetos (OO)", com "classes". Nós primeiro iremos olhar para "orientação à classes" como um padrão de projeto, antes de examinar as mecânicas de "classes": "instanciação", "herança" e "polimorfismo (relativo)".

Nós veremos que esses conceitos não mapeiam muito naturalmente para o mecanismo de objetos no JS e também até onde muitos desenvolvedores JavaScript vão para superar estes desafios (mixins etc.).

**Nota:** Este capítulo gasta um tempo considerável (a primeira metade!) na exploração massiva da teoria por trás da "programação orientada a objetos". Nós eventualmente relacionaremos a um código JavaScript concreto na segunda metade, quando nós falamos sobre "Mixins". Mas há uma grande quantidade de conceito e pseudocódigo para percorrer primeiro, então não se perca -- apenas fique com isso!

## Teoria das Classes

Classe/Herança descreve uma certa forma de organização de código e arquitetura -- uma maneira de modelar problemas de domínio do mundo real em nosso software.

OO ou programação orientada à classes enfatiza que os dados têm um comportamento associado intrinsecamente (claro, diferente dependendo do tipo e natureza do dado!) que opera sobre ele, então o design adequado é empacotar (também conhecido como encapsular) o dado e o comportamento juntos. Isso às vezes é chamado "estruturas de dados" na ciência da computação.

Por exemplo, uma série de caracteres que representa uma palavra ou frase geralmente é chamada de "string". Os caracteres são os dados. Mas você quase nunca se importa apenas com os dados, você geralmente quer *fazer coisas* com os dados, assim os comportamentos que podem se aplicar *para* aquele dado (calcular seu tamanho, anexar dados, busca etc.) são todos concebidos como métodos de uma classe `String`.

Qualquer string é apenas uma instância dessa classe, o que significa que é um pacote cuidadosamente organizado contendo tanto os caracteres quanto as funcionalidades que podemos executar neles.

Classe também insinuam uma maneira de *classificar* uma certa estrutura de dados. A maneira que fazemos isso é assumindo que qualquer estrutura é uma variação específica de uma definição `base` mais ampla.

Vamos explorar esse processo de classificação olhando para um exemplo comumente citado. Um *carro* pode ser descrito como uma implementação específica de uma "classe" de coisas, chamada de *veículo*.

Nós modelamos esse relacionamento em software com classes definindo uma classe `Vehicle` e uma classe `Car`.

A definição de `Vehicle` pode incluir coisas como propulsão (motores etc.), a capacidade de transportar pessoas etc., que seriam todos os comportamentos. O que nós definimos em `Vehicle` é todo o material que é comum a todos (ou a maioria) dos diferentes tipos de veículos (como os "aviões, trens e automóveis").

Pode não fazer sentido no nosso software redefinir a essência básica da "capacidade de transportar pessoas"
repetidas vezes para cada tipo diferente de veículo. Ao invés disso, nós definimos essa capacidade uma vez em `Vehicle`, e então quando definimos `Car`, nós simplesmente indicamos que ele "herda" (ou "estende") sua definição base de `Vehicle`. Diz-se que a definição de `Car` especializa a definição geral de `Vehicle`.

Enquanto `Vehicle` e `Car` definem coletivamente o comportamento por meio de métodos, os dados em uma instância seriam coisas como o "número do chassi" de um carro específico etc.

**E assim, classes, herança e instanciação surgem.**

Outro conceito chave com classes é o "polimorfismo", o qual descreve a ideia que um comportamento genérico de uma classe pai pode ser sobrescrito em uma classe filha para dar mais especificidade. De fato, o polimorfismo relativo nos permite referenciar o comportamento base a partir do comportamento sobrescrito.

A teoria de Classe sugere fortemente que uma classe pai e uma classe filha compartilhem o mesmo nome de método para um determinado comportamento, de modo que esse filho substitui o pai (diferentemente). Como nós veremos mais tarde, fazer isso no seu código JavaScript é optar pela frustração e pela fragilidade do código.

### Padrão de projeto "Classe"

Você pode nunca ter pensando sobre classes como um "padrão de projeto", já que é mais comum ver discussões sobre os populares "Padrões de projeto OO", como "Iterator", "Observer", "Factory", "Singleton" etc. Apresentado dessa maneira, é quase uma suposição de que as classes OO são a mecânica baixo nível pela qual implementamos todos os padrões de projeto (alto nível), como se OO fosse uma base para *todo* o código (apropriado).

Dependendo do seu nível de educação formal em programação, você pode ter ouvido falar sobre "programação procedural" como uma maneira de descrever o código que consiste apenas de procedimentos (também conhecido como funções) chamando outras funções, sem nenhuma maior abstração. Você pode ter sido ensinado que classes eram a maneira *adequada* de transformar o estilo procedural "código espaguete" em um código bem moldado e organizado.

Claro que, se você tem experiência com "programação funcional" (Monads etc.), você sabe muito bem que as classes são apenas um dos vários padrões de projetos comuns. Mas para outras pessoas, esta pode ser a primeira vez que você se pergunta se as classes realmente são uma base fundamental para o código, ou se elas são uma abstração opcional sobre o código.

Algumas linguagens (como Java) não te dão a opção de escolha, então não é muito *opcional* -- tudo é uma classe. Outras linguagens como C/C++ ou PHP fornecem tanto a sintaxe procedural quanto a orientada a classes deixando mais para o desenvolvedor escolher qual estilo ou mistura de estilos é apropriado.

### "Classes" JavaScript

Onde o JavaScript se enquadra nesse aspecto? JS já possuía *alguns* elementos sintáticos baseados em classes (como `new` e `instaceof`) por um tempo, e mais recentemente no ES6, foram feitas algumas adições, como a palavra chave `class` (veja o Apêndice A).

Mas isso significa que atualmente o Javascript *possui* classes? Claro e simples: **Não.**

Como as classes são um design pattern, você *pode*, com um pouco de esforço (como veremos ao longo desse capítulo), implementar aproximações de muitas funcionalidades clássicas das classes. O JS tenta satisfazer o *desejo* extremamente comum de projetar com classes, fornecendo uma sintaxe aparentemente semelhante com a de uma classe.

Embora possamos ter uma sintaxe parecida com as das classes, é como se o mecanismo do Javascript estivesse lutando contra o seu desejo de utilizar o *design pattern de classes*, porque por trás da cortina, os mecanismos que você constrói estão operando de forma bastante diferente. Açúcar sintático e (as amplamente utilizadas) bibliotecas "Class" JS percorrem um longo caminho para esconder essa realidade de você, porém mais cedo ou mais tarde você vai enfrentar o fato de que classes que você tem em outras linguagens não são como as "classes" do Javascript.

Isso resume que as classes são um pattern opcional no design de um software, e você tem a escolha de usá-los no Javascript ou não. Como muitos desenvolvedores possuem uma grande afinidade com design de software orientado a classes, passaremos o restante deste capítulo explorando o que é necessário para manter a ilusão de que o JS provem classes, e os pontos problemáticos que teremos.

## Mecânica das Classes

Na maioria das linguagens orientadas a classes, a "biblioteca padrão" provê uma "stack" de estruturas de dados (push, pop, etc) como uma classe `Stack`.
Essa classe teria um conjunto interno de variáveis que armazena dados e um conjunto de comportamentos publicamente acessíveis ("métodos") fornecidos pela classe, o que dá ao seu código a capacidade de interagir (adicionando e removendo, etc.) com os dados (ocultos).

Porém em tais linguagens você não precisa trabalhar diretamente com essa classe `Stack` (a menos que esteja fazendo uma referência a um membro de uma classe do tipo **Static**, o que está fora do escopo da nossa discussão). A classe `Stack` é apenas uma explicação abstrata do que qualquer objeto do tipo "stack" pode fazer, mas não é em si uma "stack". Você deve **instanciar** a classe `Stack` antes de ter uma estrutura de dados concreta onde possa operar.

### Construção

O pensamento metafórico tradicional baseado em "classe" e "instância" vêm da construção civil.

Uma arquiteta planeja todos as características de um edifício: quão largo, quão alto, janelas e em quais locais, até mesmo o tipo de material que será usado no teto e nas paredes. Até esse ponto, ela não necessáriamente se importa, onde o prédio será construido, nem se importa em quantas cópias do edifício serão construídas.

Ela também não se importa muito com o conteúdo do edifício -- o mobiliário, o papel de parede, ventiladores de teto, etc. - somente o tipo de estrutura que serão contidos.

Os projetos arquitetônicos que ela produz são apenas *planos* para um edifício. Eles não constituem realmente um prédio onde podemos caminhar e sentar. Nós precisamos de um construtor para essa tarefa. Um construtor irá pegar esses planos e os seguir, minuciosamente, enquanto *constrói* o prédio. Em um sentido mais real, ele está *copiando* as características dos planos para o prédio físico.

Uma vez concluído, o prédio é uma instanciação física do projeto arquitetônico, com sorte um *cópia* perfeita. E então o construtor pode se mover para o lote aberto ao lado e fazer tudo novamente, criando outra *cópia*.

O relacionamento entre o prédio e o plano arquitetônico é indireto. Você pode examinar um projeto arquitetônico para entender como o prédio foi estruturado, para todas as partes em que a inspeção direta do edifício em si fosse insuficiente. Mas se você quer abrir uma porta, você tem que ir para o prédio em si -- o projeto tem apenas linhas desenhadas em uma página *representando* onde a porta deveria estar.

Uma classe é um projeto arquitetônico. Para realmente *conseguir* um objeto com o qual podemos interagir, nós devemos construir (também conhecido como "instanciar") algo da classe. O resultado final dessa "construção" é um objeto, tipicamente chamado de "instância", no qual podemos chamar métodos diretamente e acessar quaisquer propriedades de dados públicos, conforme o necessário.

**O objeto é uma *cópia*** de todas as características descritas pela classe.

Você provavelmente não esperaria entrar em um prédio e encontrar, emoldurado na parede, uma cópia do projeto arquitetônico usado para planejar o prédio, embora projetos arquitetônicos provavelmente estejam arquivadas em um escritório de registros públicos. Similarmente, você geralmente não usa uma instância de objeto para diretamente acessar e manipular a classe, mas é geralmente possível ao menos determinar *de qual classe* um determinado objeto vem.

É mais útil considerar a relação direta entre a classe e uma instância de objeto, ao invés do relacionamento indireto entre uma instância de objeto e a classe do qual ela vem. **Uma classe é instanciada em um objeto por uma operação de cópia.**

<img src="fig1.png">

Como você pode ver, as setas se movem da esquerda para direita, e de cima para baixo, o que indica as operações de cópia que ocorrem tanto conceitualmente quanto fisicamente.

### Construtores

Instâncias das classes são construídas por um método especial da classe, que geralmente possui o mesmo nome da classe, chamado de *construtor*. O trabalho desse método é inicializar qualquer informação (estado) que a instância irá precisar.

Por exemplo, considere esse pseudo-código solto (sintaxe inventada) para as classes:

```js
class CoolGuy {
	specialTrick = nothing

	CoolGuy( trick ) {
		specialTrick = trick
	}

	showOff() {
		output( "Here's my trick: ", specialTrick )
	}
}
```

Para *criar* uma instância de `CoolGuy`, nós vamos chamar o construtor da classe:

```js
Joe = new CoolGuy( "jumping rope" )

Joe.showOff() // Here's my trick: jumping rope
```

Perceba que a classe `CoolGuy` tem um construtor `CoolGuy()`, que é o que chamamos quando dizemos `new CoolGuy(..)`. Nós temos como retorno um objeto (uma instância da nossa classe) do construtor, e nós podemos chamar o método `showOff()`, que vai mostrar o truque especial daquele determinado `CoolGuy`

*Obviamente, pular corda torna Joe um cara muito legal*

O construtor de uma classe *pertence* a classe, quase universalmente com o mesmo nome da classe. Além disso, construtores quase sempre precisam ser chamados com o operador `new` para que o motor da linguagem saiba que você quer construir uma nova instância da classe.

## Heranças de classe

Em linguagens orientadas a classes, você pode definir não somente qual classe pode ser instânciada, como também pode definir que outra classe que herda da primeira classe.

A segunda classe é também chamada de classe-filha enquanto que a primeira é chamada de classe-pai. Esses termos se originam, obviamente, de uma metáfora entre pais e filhos, embora as metáforas aqui estejam empregadas em um conceito mais amplo, como você verá em breve.

Quando um pai tem um filho/filha biológico, as características genéticas do pai são copiadas no filho. Obviamente, na maioria dos sistemas de reprodução biológicos, há dois pais que contribuem igualmente para a variabilidade genética. Mas por efeitos da metáfora, vamos considerar somente um pai.

A partir do momento que o filho existe, ele ou ela é separado do pai. O filho foi fortemente influenciado pela herança genética do seu pai, mas é único e distinto. Se uma criança tem cabelo ruivo isso não necessariamente significa que algum de seus pais tenha cabelo vermelho.

De maneira similar, uma vez que uma classe filha é definida, ela é separada e distinta da classe pai. A classe filha possui uma cópia inicial do comportamento do pai, mas elas podem sobrescrever qualquer comportamento herdado e até mesmo definir um novo comportamento.

É importante relembrar que estamos falando sobre **classes** pais e filhos, que não são coisas físicas. E é isso o que torna a metáfora de pais e filhos um tanto quanto confusas, por que o correto seria dizer que uma classe pai é na verdade o DNA de um pai, e uma classe filho seria como o DNA de um filho. Nós podemos criar (também conhecido como "instanciar") uma pessoa de cada conjunto de DNA para realmente ter uma pessoa com a qual podemos conversar.

Vamos colocar de o conceito biológico de pais e filhos, e olhar a herança através de uma lente ligeramente diferente: diferentes tipos de veículos. Essa é uma das metáforas mais canônicas (e muitas vezes dignas) para entender herança.

Vamos revisitar a discussão sobre `Vehicle` (veículo) e `Car` (carro) que tivemos anteriormente nesse capítulo. Considere esse pseudo-código solto (sintaxe inventada) para classes herdadas:

```js
class Vehicle {
	engines = 1

	ignition() {
		output( "Turning on my engine." )
	}

	drive() {
		ignition()
		output( "Steering and moving forward!" )
	}
}

class Car inherits Vehicle {
	wheels = 4

	drive() {
		inherited:drive()
		output( "Rolling on all ", wheels, " wheels!" )
	}
}

class SpeedBoat inherits Vehicle {
	engines = 2

	ignition() {
		output( "Turning on my ", engines, " engines." )
	}

	pilot() {
		inherited:drive()
		output( "Speeding through the water with ease!" )
	}
}
```

**Nota:** Para clareza e abreviação, os construtores dessas classes foram omitidos

Nós definimos a classe `Veiculo` para que ela assumisse um motor, a forma de ligar a ignição e uma maneira de dirigir. Mas você não fabricaria um veículo genérico, então até esse ponto ela é um conceito abstrato.

Então, definimos dosi tipos específicos de veículos: `Carro` e `SpeedBoat`. Ambos herdam as características gerais de `Veiculo`, mas especializam características apropriadamente para cada tipo. Um carro precisa de 4 rodas, e um SpeedBoat de dois motores, o que significa que é preciso atenção extra para ligar a ignição de ambos motores.

### Polimorfismo

`Car` define seu próprio método `drive()`, que sobrescreve o método de mesmo nome herdado da classe `Veiculo`. Mas então o método `drive()` pertencente a `Car` chama `inherited:drive()`, o que indica que o carro pode referenciar o método original `drive()` pré-sobrescrito herdado. O método `pilot()` do SpeedBoat também faz referência à sua cópia herdada do `drive()`.

Essa técnica é chamada de "polimorfismo", ou "polimorfismo virtual". Mais especificamente para nosso ponto atual, vamos chamar isso de "polimorfismo relativo".

Polimorfismo é um tópico muito mais abrangente do que vamos abordar aqui, mas nossa atual semântica "relativa" refere-se a um aspecto em particular: a ideia de que qualquer método pode referenciar outro método (com o mesmo nome, ou um nome diferente) em um nível mais alto da hierarquia de herança. Nós chamamos de "relativo" porque não estabelecemos absolutamente qual nível de herança (também conhecido como classe) queremos acessar, mas referencia-lo relativamente basicamente dizendo "procure um nível acima".

Em muitas linguagens, a palavra chave `super` é usada, no lugar de `inherited` desse exemplo, apoiando-se na ideia de que uma "superclasse" é o pai/ancestral da classe atual.

Outro aspecto do polimorfismo é que um nome específico de método pode ter mútiplas definições em diferentes níveis da cadeia de herança, e essas definiçõessão selecionadas de forma automática de acordo com os métodos que estão sendo chamados.

Nós vemos duas ocorrências desse comportamento em nosso exemplo acima: `drive()` é definido tanto em `Veiculo` como em `Car`, e `ignition()` é definido tanto em `Veiculo` como em `SpeedBoat`.

**Nota:** Outra coisa que linguagens orientadas a classes fornecem a você através do `super()` é uma forma do construtor da classe filho referenciar diretamente o construtor de sua classe pai. Isso é verdade principalmente porque, com classes reais, o construtor pertence a classe. No entanto em JS é o contrário - é mais apropriado pensar que a "classe" pertence ao construtor (as referências `Foo.prototype`). Já que em JS o relacionamento entre pai e filho existe apenas entre os dois objetos `.prototype` dos respectivos construtores, os próprios construtores não estão diretamente relacionados, e portanto, não há uma forma de referenciar um do outro (consulte o apêndice A para ver como `class` resolve isso com `super`).

Uma implicação interessante do polimorfismo pode ser visto especificamente com `ignition()`. Dentro de `pilot()`, uma referência polimórfica é feita para (a herdada) versão `drive()` de `Veiculo`. Mas esse método `drive()` faz referência ao método `ignition()` apenas pelo nome (sem termos referência relativa).

Qual versão de `ignition()` o motor da linguagem irá usar?, a de `Veiculo` ou a de `SpeedBoat`? **Ela usa a versão de `SpeedBoat` de `ignition()`.** Se você *fosse* instanciar a clase `Veiculo`, e chamar o seu método `drive()`, o motro da linguagem usaria apenas a definição do método `ignition()` pertencete a `Veiculo`.

Dito de outra forma, a definição para o método `ignition()` polimorfa (muda) de acordo com a classe (nível de herança) que a sua instância está referenciando.

Isso pode parecer um detalhe acadêmico muito profundo. Mas entender esses detalhes é necessário para diferenciar adequadamente comportamentos semelhantes (porém diferentes) no mecanismo `[[Prototype]]` do Javascript.

Quando classes são herdadas, existe uma maneira de as próprias classes (não as instâncias de objetos criadas a partir delas!) referirem-se relativamente à classe herdada, e essa referência relativa é geralmente chamada de `super()`.

Lembre se dessa figura anterior:

<img src="fig1.png">

Note como todas as instanciações (`a1`, `a2`, `b1` e `b2`) e a herança (`Bar`) indicam operações de cópia.

Conceitualmente, parece que uma classe filho `Bar` pode acessar o comportamento em sua classe pai `Foo` usando uma referência polimórfica relativa (também conhecida como `super`). No entanto, na realidade, a classe filho recebe meramente uma cópia do comportamento herdado da classe pai. Se o filho "sobrescrever" um método que ele herdou, as versões originais e sobrescritas do método são mantidas, para que ambas sejam acessíveis.

Não deixe o polimorfismo confundir você em pensar que uma classe filho é ligada a uma classe pai. Uma classe filho recebe uma cópia do que precisa da classe pai. **Herança de classes significa cópias.**

### Herança múltipla

Se lembra da nossa conversa sobre pais, filhos e DNA? Nós dizemos que a metáfora era um pouco estranha porque biologicamente a maioria dos descendentes vêm de dois pais. Se uma classe pudesse herdar de duas outras classes distintas, haveria uma maior aproximação com a metáfora de pais e filhos.

Algumas linguagens orientadas a classes permitem que você especifique mais de uma classe "pai" para "herdar". Herança múltipla significa que cada definição das classes pai foram copiadas para a classe filha.

Superficialmente, isso parece um adicional poderoso para a orientação a classes, nos dando a possibilidade de compor mais funcionalidades juntos. No entanto, há certamente algumas questões complicadas que se levantam. Se ambas as classes pai possuem um método chamado `drive()`, qual versão de `drive()` será referenciada pela classe filha? Você sempre teria que manualmente especificar qual o método pai `drive()` você quis dizer, perdendo um pouco da graça da herança polimórfica?

Há uma outra variação, o chamado "Problema do Diamante", que se refere a um cenário onde uma classe filha "D" herda de outras duas classes pai ("B" e "C"), e essas classes, por sua vez, herdam de um pai "A" comum. Se "A" provem um método `drive()`, e "B" e "C" sobrescrevem (polimorficamente) esse método, quando `D` referencia `drive()`, qual versão ele irá usar(`B:drive()` ou `C:drive()`)?

<img src="fig2.png">

Essas complicações são ainda mais profundas do que essa rápida análise. Nós as expomos aqui apenas para que possamos contrastar com o funcionamento dos mecanismos Javascript.

Javascript é mais simples: ele não fornece um mecanismo nativo para "herança múltipla". Muitos vêem isso como uma coisa boa, porque a economia de complexidade é mais compensatória do que a funcionalidade "reduzida". Mas isso não impede os programadores de fingirem fazer isso de várias formas diferentes, como vamos ver a seguir.

## Mixins

JavaScript's object mechanism does not *automatically* perform copy behavior when you "inherit" or "instantiate". Plainly, there are no "classes" in JavaScript to instantiate, only objects. And objects don't get copied to other objects, they get *linked together* (more on that in Chapter 5).

Since observed class behaviors in other languages imply copies, let's examine how JS developers **fake** the *missing* copy behavior of classes in JavaScript: mixins. We'll look at two types of "mixin": **explicit** and **implicit**.

### Explicit Mixins

Let's again revisit our `Vehicle` and `Car` example from before. Since JavaScript will not automatically copy behavior from `Vehicle` to `Car`, we can instead create a utility that manually copies. Such a utility is often called `extend(..)` by many libraries/frameworks, but we will call it `mixin(..)` here for illustrative purposes.

```js
// vastly simplified `mixin(..)` example:
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		// only copy if not already present
		if (!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}

	return targetObj;
}

var Vehicle = {
	engines: 1,

	ignition: function() {
		console.log( "Turning on my engine." );
	},

	drive: function() {
		this.ignition();
		console.log( "Steering and moving forward!" );
	}
};

var Car = mixin( Vehicle, {
	wheels: 4,

	drive: function() {
		Vehicle.drive.call( this );
		console.log( "Rolling on all " + this.wheels + " wheels!" );
	}
} );
```

**Note:** Subtly but importantly, we're not dealing with classes anymore, because there are no classes in JavaScript. `Vehicle` and `Car` are just objects that we make copies from and to, respectively.

`Car` now has a copy of the properties and functions from `Vehicle`. Technically, functions are not actually duplicated, but rather *references* to the functions are copied. So, `Car` now has a property called `ignition`, which is a copied reference to the `ignition()` function, as well as a property called `engines` with the copied value of `1` from `Vehicle`.

`Car` *already* had a `drive` property (function), so that property reference was not overridden (see the `if` statement in `mixin(..)` above).

#### "Polymorphism" Revisited

Let's examine this statement: `Vehicle.drive.call( this )`. This is what I call "explicit pseudo-polymorphism". Recall in our previous pseudo-code this line was `inherited:drive()`, which we called "relative polymorphism".

JavaScript does not have (prior to ES6; see Appendix A) a facility for relative polymorphism. So, **because both `Car` and `Vehicle` had a function of the same name: `drive()`**, to distinguish a call to one or the other, we must make an absolute (not relative) reference. We explicitly specify the `Vehicle` object by name, and call the `drive()` function on it.

But if we said `Vehicle.drive()`, the `this` binding for that function call would be the `Vehicle` object instead of the `Car` object (see Chapter 2), which is not what we want. So, instead we use `.call( this )` (Chapter 2) to ensure that `drive()` is executed in the context of the `Car` object.

**Note:** If the function name identifier for `Car.drive()` hadn't overlapped with (aka, "shadowed"; see Chapter 5) `Vehicle.drive()`, we wouldn't have been exercising "method polymorphism". So, a reference to `Vehicle.drive()` would have been copied over by the `mixin(..)` call, and we could have accessed directly with `this.drive()`. The chosen identifier overlap **shadowing** is *why* we have to use the more complex *explicit pseudo-polymorphism* approach.

In class-oriented languages, which have relative polymorphism, the linkage between `Car` and `Vehicle` is established once, at the top of the class definition, which makes for only one place to maintain such relationships.

But because of JavaScript's peculiarities, explicit pseudo-polymorphism (because of shadowing!) creates brittle manual/explicit linkage **in every single function where you need such a (pseudo-)polymorphic reference**. This can significantly increase the maintenance cost. Moreover, while explicit pseudo-polymorphism can emulate the behavior of "multiple inheritance", it only increases the complexity and brittleness.

The result of such approaches is usually more complex, harder-to-read, *and* harder-to-maintain code. **Explicit pseudo-polymorphism should be avoided wherever possible**, because the cost outweighs the benefit in most respects.

#### Mixing Copies

Recall the `mixin(..)` utility from above:

```js
// vastly simplified `mixin()` example:
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		// only copy if not already present
		if (!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}

	return targetObj;
}
```

Now, let's examine how `mixin(..)` works. It iterates over the properties of `sourceObj` (`Vehicle` in our example) and if there's no matching property of that name in `targetObj` (`Car` in our example), it makes a copy. Since we're making the copy after the initial object exists, we are careful to not copy over a target property.

If we made the copies first, before specifying the `Car` specific contents, we could omit this check against `targetObj`, but that's a little more clunky and less efficient, so it's generally less preferred:

```js
// alternate mixin, less "safe" to overwrites
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		targetObj[key] = sourceObj[key];
	}

	return targetObj;
}

var Vehicle = {
	// ...
};

// first, create an empty object with
// Vehicle's stuff copied in
var Car = mixin( Vehicle, { } );

// now copy the intended contents into Car
mixin( {
	wheels: 4,

	drive: function() {
		// ...
	}
}, Car );
```

Either approach, we have explicitly copied the non-overlapping contents of `Vehicle` into `Car`. The name "mixin" comes from an alternate way of explaining the task: `Car` has `Vehicle`s contents **mixed-in**, just like you mix in chocolate chips into your favorite cookie dough.

As a result of the copy operation, `Car` will operate somewhat separately from `Vehicle`. If you add a property onto `Car`, it will not affect `Vehicle`, and vice versa.

**Note:** A few minor details have been skimmed over here. There are still some subtle ways the two objects can "affect" each other even after copying, such as if they both share a reference to a common object (such as an array).

Since the two objects also share references to their common functions, that means that **even manual copying of functions (aka, mixins) from one object to another doesn't *actually emulate* the real duplication from class to instance that occurs in class-oriented languages**.

JavaScript functions can't really be duplicated (in a standard, reliable way), so what you end up with instead is a **duplicated reference** to the same shared function object (functions are objects; see Chapter 3). If you modified one of the shared **function objects** (like `ignition()`) by adding properties on top of it, for instance, both `Vehicle` and `Car` would be "affected" via the shared reference.

Explicit mixins are a fine mechanism in JavaScript. But they appear more powerful than they really are. Not much benefit is *actually* derived from copying a property from one object to another, **as opposed to just defining the properties twice**, once on each object. And that's especially true given the function-object reference nuance we just mentioned.

If you explicitly mix-in two or more objects into your target object, you can **partially emulate** the behavior of "multiple inheritance", but there's no direct way to handle collisions if the same method or property is being copied from more than one source. Some developers/libraries have come up with "late binding" techniques and other exotic work-arounds, but fundamentally these "tricks" are *usually* more effort (and lesser performance!) than the pay-off.

Take care only to use explicit mixins where it actually helps make more readable code, and avoid the pattern if you find it making code that's harder to trace, or if you find it creates unnecessary or unwieldy dependencies between objects.

**If it starts to get *harder* to properly use mixins than before you used them**, you should probably stop using mixins. In fact, if you have to use a complex library/utility to work out all these details, it might be a sign that you're going about it the harder way, perhaps unnecessarily. In Chapter 6, we'll try to distill a simpler way that accomplishes the desired outcomes without all the fuss.

#### Parasitic Inheritance

A variation on this explicit mixin pattern, which is both in some ways explicit and in other ways implicit, is called "parasitic inheritance", popularized mainly by Douglas Crockford.

Here's how it can work:

```js
// "Traditional JS Class" `Vehicle`
function Vehicle() {
	this.engines = 1;
}
Vehicle.prototype.ignition = function() {
	console.log( "Turning on my engine." );
};
Vehicle.prototype.drive = function() {
	this.ignition();
	console.log( "Steering and moving forward!" );
};

// "Parasitic Class" `Car`
function Car() {
	// first, `car` is a `Vehicle`
	var car = new Vehicle();

	// now, let's modify our `car` to specialize it
	car.wheels = 4;

	// save a privileged reference to `Vehicle::drive()`
	var vehDrive = car.drive;

	// override `Vehicle::drive()`
	car.drive = function() {
		vehDrive.call( this );
		console.log( "Rolling on all " + this.wheels + " wheels!" );
	};

	return car;
}

var myCar = new Car();

myCar.drive();
// Turning on my engine.
// Steering and moving forward!
// Rolling on all 4 wheels!
```

As you can see, we initially make a copy of the definition from the `Vehicle` "parent class" (object), then mixin our "child class" (object) definition (preserving privileged parent-class references as needed), and pass off this composed object `car` as our child instance.

**Note:** when we call `new Car()`, a new object is created and referenced by `Car`s `this` reference (see Chapter 2). But since we don't use that object, and instead return our own `car` object, the initially created object is just discarded. So, `Car()` could be called without the `new` keyword, and the functionality above would be identical, but without the wasted object creation/garbage-collection.

### Implicit Mixins

Implicit mixins are closely related to *explicit pseudo-polymorphism* as explained previously. As such, they come with the same caveats and warnings.

Consider this code:

```js
var Something = {
	cool: function() {
		this.greeting = "Hello World";
		this.count = this.count ? this.count + 1 : 1;
	}
};

Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1

var Another = {
	cool: function() {
		// implicit mixin of `Something` to `Another`
		Something.cool.call( this );
	}
};

Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1 (not shared state with `Something`)
```

With `Something.cool.call( this )`, which can happen either in a "constructor" call (most common) or in a method call (shown here), we essentially "borrow" the function `Something.cool()` and call it in the context of `Another` (via its `this` binding; see Chapter 2) instead of `Something`. The end result is that the assignments that `Something.cool()` makes are applied against the `Another` object rather than the `Something` object.

So, it is said that we "mixed in" `Something`s behavior with (or into) `Another`.

While this sort of technique seems to take useful advantage of `this` rebinding functionality, it is the brittle `Something.cool.call( this )` call, which cannot be made into a relative (and thus more flexible) reference, that you should **heed with caution**. Generally, **avoid such constructs where possible** to keep cleaner and more maintainable code.

## Review (TL;DR)

Classes are a design pattern. Many languages provide syntax which enables natural class-oriented software design. JS also has a similar syntax, but it behaves **very differently** from what you're used to with classes in those other languages.

**Classes mean copies.**

When traditional classes are instantiated, a copy of behavior from class to instance occurs. When classes are inherited, a copy of behavior from parent to child also occurs.

Polymorphism (having different functions at multiple levels of an inheritance chain with the same name) may seem like it implies a referential relative link from child back to parent, but it's still just a result of copy behavior.

JavaScript **does not automatically** create copies (as classes imply) between objects.

The mixin pattern (both explicit and implicit) is often used to *sort of* emulate class copy behavior, but this usually leads to ugly and brittle syntax like explicit pseudo-polymorphism (`OtherObj.methodName.call(this, ...)`), which often results in harder to understand and maintain code.

Explicit mixins are also not exactly the same as class *copy*, since objects (and functions!) only have shared references duplicated, not the objects/functions duplicated themselves. Not paying attention to such nuance is the source of a variety of gotchas.

In general, faking classes in JS often sets more landmines for future coding than solving present *real* problems.
