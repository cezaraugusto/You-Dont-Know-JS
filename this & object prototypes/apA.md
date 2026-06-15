# You Don't Know JS: *this* & Prototipagem de Objetos
# Apêndice A: ES6 `class`

Se há uma mensagem a ser guardada da segunda metade deste livro (Capítulos 4-6), é que classes são um padrão de design opcional para códigos (não uma regra necessária), e que elas são normalmente um pouco estranhas de se implementar em uma linguagem `[[Prototype]]` como JavaScript.

Essa estranheza *não* ocorre apenas na sintaxe, embora ela tenha grande parte nisso. Os Capítulos 4 e 5 examinaram uma boa quantidade de horrores sintáticos, desde a verbosidade de referências `.prototype` poluindo o código, até o *pseudo-polimorfismo explicito* (veja o Capítulo 4) onde você dá o mesmo nome a métodos em diferentes níveis da cadeia e tenta implementar uma referência polimórfica de um método de baixo nível para outro de nível maior. `.constructor` ser erroneamente interpretado como "foi construído por" e ainda ser incerto para essa definição é mais um horror sintático.

Mas os problemas com design de classes são muito mais profundos. O Capítulo 4 mostra que, em linguagens tradicionais orientadas a classe, as classes na verdade produzem uma *cópia* da ação de pai para filho para instância, enquanto em `[[Prototype]]` essa ação **não** é uma cópia, mas sim o contrário -- um link de delegação.

Quando comparadas à simplicidade de código no estilo OLOO e delegação de comportamentos (veja Capítulo 6), que abraçam o `[[Prototype]]` ao invés de se esconderem dele, as classes parecem um peixe fora da água em JS.

## `class`

Mas nós *não* precisamos rediscutir esse caso novamente. Eu menciono brevemente esses problemas apenas para que você os mantenha frescos em sua mente agora que voltamos nossa atenção para o mecanismo `class` do ES6. Vamos demonstrar aqui como ele funciona, e ver se `class` faz ou não algo substancial para resolver alguma dessas preocupações com "classes".

Vamos revisitar o exemplo `Widget` / `Button` do Capítulo 6:

```js
class Widget {
	constructor(width,height) {
		this.width = width || 50;
		this.height = height || 50;
		this.$elem = null;
	}
	render($where){
		if (this.$elem) {
			this.$elem.css( {
				width: this.width + "px",
				height: this.height + "px"
			} ).appendTo( $where );
		}
	}
}

class Button extends Widget {
	constructor(width,height,label) {
		super( width, height );
		this.label = label || "Default";
		this.$elem = $( "<button>" ).text( this.label );
	}
	render($where) {
		super.render( $where );
		this.$elem.click( this.onClick.bind( this ) );
	}
	onClick(evt) {
		console.log( "Button '" + this.label + "' clicked!" );
	}
}
```

Além dessa sintaxe *parecer* mais agradável, quais problemas o ES6 resolve?

1. Não há mais (bem, mais ou menos, veja abaixo!) referências a `.prototype` poluindo o código.
2. `Button` é declarado diretamente para "herdar de" (também conhecido como `extends`) `Widget`, ao invés de precisar usar `Object.create(..)` para substituir um objeto `.prototype` que está ligado, ou ter que configurar com `.__proto__` ou `Object.setPrototypeOf(..)`.
3. `super(..)` agora nos dá uma capacidade de **polimorfismo relativo** muito útil, de modo que qualquer método em um nível da cadeia pode se referir relativamente a um nível acima na cadeia a um método de mesmo nome. Isso inclui uma solução para a nota do Capítulo 4 sobre a estranheza de construtores não pertencerem a sua classe, e portanto não terem relação -- `super()` funciona dentro de construtores exatamente como você esperaria.
4. A sintaxe literal de `class` não tem recurso para especificar propriedades (apenas métodos). Isso pode parecer limitante para alguns, mas é esperado que na vasta maioria dos casos em que uma propriedade (estado) exista em outro lugar que não as "instâncias" no fim da cadeia, isso seja normalmente um erro e surpreendente (já que é estado que está implicitamente "compartilhado" entre todas as "instâncias"). Então, pode-se dizer que a sintaxe `class` está protegendo você de erros.
5. `extends` permite que você estenda até mesmo (sub)tipos de objetos embutidos, como `Array` ou `RegExp`, de uma forma muito natural. Fazer isso sem `class .. extends` há muito tempo tem sido uma tarefa extremamente complexa e frustrante, uma que apenas os mais habilidosos autores de frameworks já conseguiram abordar com precisão. Agora, será bem trivial!

Para ser justo, essas são algumas soluções substanciais para muitos dos problemas e surpresas (sintáticos) mais óbvios que as pessoas têm com código clássico no estilo de protótipos.

## Pegadinhas de `class`

Mas nem tudo são flores, porém. Ainda existem alguns problemas profundos e profundamente preocupantes em usar "classes" como um padrão de design em JS.

Primeiramente, a sintaxe `class` pode te convencer de que um novo mecanismo de "classe" existe em JS a partir do ES6. **Não é o caso.** `class` é, na maior parte, apenas açúcar sintático em cima do mecanismo `[[Prototype]]` (delegação!) existente.

Isso significa que `class` na verdade não está copiando definições estaticamente no momento da declaração da forma como faz em linguagens tradicionais orientadas a classe. Se você alterar/substituir um método (de propósito ou por acidente) na "classe" pai, a "classe" filha e/ou as instâncias ainda serão "afetadas", no sentido de que elas não receberam cópias no momento da declaração, elas estão todas ainda usando o modelo de delegação em tempo real baseado em `[[Prototype]]`:

```js
class C {
	constructor() {
		this.num = Math.random();
	}
	rand() {
		console.log( "Random: " + this.num );
	}
}

var c1 = new C();
c1.rand(); // "Random: 0.4324299..."

C.prototype.rand = function() {
	console.log( "Random: " + Math.round( this.num * 1000 ));
};

var c2 = new C();
c2.rand(); // "Random: 867"

c1.rand(); // "Random: 432" -- ops!!!
```

Isso só parece um comportamento razoável *se você já souber* sobre a natureza de delegação das coisas, ao invés de esperar *cópias* de "classes reais". Então a pergunta a se fazer é: por que você está escolhendo a sintaxe `class` para algo fundamentalmente diferente de classes?

A sintaxe `class` do ES6 **não torna mais difícil** ver e entender a diferença entre classes tradicionais e objetos delegados?

A sintaxe `class` *não* fornece uma maneira de declarar propriedades de membro de classe (apenas métodos). Então, se você precisa fazer isso para rastrear estado compartilhado entre instâncias, então você acaba voltando para a feia sintaxe `.prototype`, assim:

```js
class C {
	constructor() {
		// certifique-se de modificar o estado compartilhado,
		// e não configurar uma propriedade sombreada nas
		// instâncias!
		C.prototype.count++;

		// aqui, `this.count` funciona como esperado
		// via delegação
		console.log( "Hello: " + this.count );
	}
}

// adiciona uma propriedade para estado compartilhado diretamente ao
// objeto prototype
C.prototype.count = 0;

var c1 = new C();
// Hello: 1

var c2 = new C();
// Hello: 2

c1.count === 2; // true
c1.count === c2.count; // true
```

O maior problema aqui é que isso trai a sintaxe `class` ao expor (vazamento!) `.prototype` como um detalhe de implementação.

Mas, nós também ainda temos a pegadinha surpresa de que `this.count++` criaria implicitamente uma propriedade `.count` sombreada separada em ambos os objetos `c1` e `c2`, ao invés de atualizar o estado compartilhado. `class` não nos oferece nenhum consolo desse problema, exceto (presumivelmente) implicar, pela falta de suporte sintático, que você não deveria estar fazendo isso *de jeito nenhum*.

Além disso, o sombreamento acidental ainda é um perigo:

```js
class C {
	constructor(id) {
		// ops, pegadinha, estamos sombreando o método `id()`
		// com um valor de propriedade na instância
		this.id = id;
	}
	id() {
		console.log( "Id: " + this.id );
	}
}

var c1 = new C( "c1" );
c1.id(); // TypeError -- `c1.id` agora é a string "c1"
```

Há também alguns problemas muito sutis e nuançados sobre como `super` funciona. Você pode supor que `super` seria ligado de uma forma análoga a como `this` é ligado (veja o Capítulo 2), que é que `super` seria sempre ligado a um nível acima de qualquer que seja a posição atual do método na cadeia `[[Prototype]]`.

No entanto, por razões de desempenho (a ligação de `this` já é cara), `super` não é ligado dinamicamente. Ele é ligado de uma forma "estática", em tempo de declaração. Não é grande coisa, certo?

Ehh... talvez, talvez não. Se você, como a maioria dos devs JS, começar a atribuir funções por aí para diferentes objetos (que vieram de definições de `class`), de várias maneiras diferentes, você provavelmente não estará muito ciente de que, em todos esses casos, o mecanismo `super` por baixo dos panos está tendo que ser religado a cada vez.

E dependendo de quais tipos de abordagens sintáticas você adota para essas atribuições, pode muito bem haver casos em que `super` não pode ser ligado adequadamente (ao menos, não onde você suspeita), então você pode (no momento em que isso é escrito, a discussão no TC39 sobre o tópico está em andamento) ter que ligar `super` manualmente com `toMethod(..)` (mais ou menos como você tem que fazer `bind(..)` para `this` -- veja o Capítulo 2).

Você está acostumado a poder atribuir métodos por aí para diferentes objetos para *automaticamente* tirar vantagem do dinamismo de `this` via a regra de *ligação implícita* (veja o Capítulo 2). Mas o mesmo provavelmente não será verdade com métodos que usam `super`.

Considere o que `super` deveria fazer aqui (contra `D` e `E`):

```js
class P {
	foo() { console.log( "P.foo" ); }
}

class C extends P {
	foo() {
		super();
	}
}

var c1 = new C();
c1.foo(); // "P.foo"

var D = {
	foo: function() { console.log( "D.foo" ); }
};

var E = {
	foo: C.prototype.foo
};

// Liga E a D para delegação
Object.setPrototypeOf( E, D );

E.foo(); // "P.foo"
```

Se você estava pensando (de forma bem razoável!) que `super` seria ligado dinamicamente em tempo de chamada, você poderia esperar que `super()` reconhecesse automaticamente que `E` delega para `D`, então `E.foo()` usando `super()` deveria chamar `D.foo()`.

**Não é o caso.** Por razões pragmáticas de desempenho, `super` não é *ligado tardiamente* (também conhecido como ligado dinamicamente) como `this` é. Ao invés disso, ele é derivado em tempo de chamada de `[[HomeObject]].[[Prototype]]`, onde `[[HomeObject]]` é ligado estaticamente em tempo de criação.

Neste caso particular, `super()` ainda está resolvendo para `P.foo()`, já que o `[[HomeObject]]` do método ainda é `C` e `C.[[Prototype]]` é `P`.

*Provavelmente* haverá formas de abordar manualmente tais pegadinhas. Usar `toMethod(..)` para ligar/religar o `[[HomeObject]]` de um método (junto com configurar o `[[Prototype]]` daquele objeto!) parece funcionar neste cenário:

```js
var D = {
	foo: function() { console.log( "D.foo" ); }
};

// Liga E a D para delegação
var E = Object.create( D );

// liga manualmente o `[[HomeObject]]` de `foo` como
// `E`, e `E.[[Prototype]]` é `D`, então portanto
// `super()` é `D.foo()`
E.foo = C.prototype.foo.toMethod( E, "foo" );

E.foo(); // "D.foo"
```

**Nota:** `toMethod(..)` clona o método, e recebe `homeObject` como seu primeiro parâmetro (que é por isso que passamos `E`), e o segundo parâmetro (opcionalmente) configura um `name` para o novo método (que mantemos como "foo").

Resta ver se há outras pegadinhas de casos extremos com que os devs vão se deparar além deste cenário. Independentemente disso, você terá que ser diligente e ficar ciente de quais lugares o motor descobre `super` automaticamente para você, e quais lugares você tem que cuidar disso manualmente. **Eca!**

# Estático > Dinâmico?

Mas o maior problema de todos sobre o `class` do ES6 é que todas essas várias pegadinhas significam que `class` meio que te coloca em uma sintaxe que parece implicar (como classes tradicionais) que uma vez que você declara uma `class`, ela é uma definição estática de uma coisa (a ser instanciada futuramente). Você perde completamente de vista o fato de que `C` é um objeto, uma coisa concreta, com a qual você pode interagir diretamente.

Em linguagens tradicionais orientadas a classe, você nunca ajusta a definição de uma classe depois, então o padrão de design de classes não sugere tais capacidades. Mas **uma das partes mais poderosas** de JS é que ela *é* dinâmica, e a definição de qualquer objeto é (a menos que você o torne imutável) uma *coisa* fluida e mutável.

`class` parece implicar que você não deveria fazer tais coisas, ao te forçar a usar a sintaxe mais feia `.prototype` para isso, ou te forçar a pensar nas pegadinhas de `super`, etc. Ela também oferece *muito pouco* suporte para qualquer uma das armadilhas que esse dinamismo pode trazer.

Em outras palavras, é como se `class` estivesse te dizendo: "dinâmico é difícil demais, então provavelmente não é uma boa ideia. Aqui está uma sintaxe com aparência estática, então escreva suas coisas estaticamente."

Que comentário triste sobre o JavaScript: **dinâmico é difícil demais, vamos fingir ser (mas não realmente ser!) estático**.

Essas são as razões pelas quais o `class` do ES6 está se disfarçando como uma boa solução para dores de cabeça sintáticas, mas na verdade está tornando as águas ainda mais turvas e piorando as coisas para o JS e para um entendimento claro e conciso.

**Nota:** Se você usar o utilitário `.bind(..)` para fazer uma função fortemente ligada (veja o Capítulo 2), a função criada não pode ser usada como subclasse com o `extend` do ES6 como funções normais podem.

## Revisão (TL;DR)

`class` faz um trabalho muito bom de fingir corrigir os problemas com o padrão de design de classe/herança em JS. Mas na verdade ela faz o oposto: **ela esconde muitos dos problemas, e introduz outros sutis mas perigosos**.

`class` contribui para a contínua confusão de "classe" em JavaScript que tem assolado a linguagem por quase duas décadas. Em alguns aspectos, ela faz mais perguntas do que responde, e parece, em sua totalidade, um encaixe muito não natural em cima da elegante simplicidade do mecanismo `[[Prototype]]`.

Resumindo: se o `class` do ES6 torna mais difícil aproveitar de forma robusta o `[[Prototype]]`, e esconde a natureza mais importante do mecanismo de objetos do JS -- **os links de delegação em tempo real entre objetos** -- não deveríamos ver `class` como criando mais problemas do que resolve, e simplesmente relegá-la a um anti-padrão?

Eu não posso realmente responder essa pergunta por você. Mas eu espero que este livro tenha explorado completamente a questão em um nível mais profundo do que você já foi antes, e tenha te dado a informação que você precisa *para respondê-la você mesmo*.
