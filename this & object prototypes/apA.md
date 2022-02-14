# You Don't Know JS: *this* & Prototipagem de Objetos
# Apêndice A: ES6 `class`

Se há uma mensagem a ser guardada da segunda metade deste livro (Capítulos 4-6), é que classes são um padrão de design opcional para códigos (não uma regra necessária), e que elas são normalmente um pouco estranhas de se implementar em uma linguagem `[[Prototype]]` como JavaScript.

Essa estranheza *não* ocorre apenas na sintaxe, embora ela tenha grande parte nisso. Os Capítulos 4 e 5 examinaram uma boa quantidade de horrores sintáticos, desde a verbosidade de referências `.prototype` poluindo o código, até o *pseudo-polimorfismo explicito* (veja o Capítulo 4) onde você dá o mesmo nome a métodos em diferentes níveis da cadeia e tenta implementar uma referência polimórfica de um método de baixo nível para outro de nível maior. `.constructor` ser erroneamente interpretado como "foi construído por" e ainda ser incerto para essa definição é mais um horror sintático.

Mas os problemas com design de classes são muito mais profundos. O Capítulo 4 mostra que, em linguagens tradicionais orientadas a classe, as classes na verdade produzem uma *cópia* da ação de pai para filho para instância, enquanto em `[[Prototype]]` essa ação **não** é uma cópia, mas sim o contrário -- um link de delegação.

Quando comparadas à simplicidade de código no estilo OLOO e delegação de comportamentos (veja Capítulo 6), que abraçam o `[[Prototype]]` ao invés de se esconderem dele, as classes parecem um peixe fora da água em JS.

## `class`

Mas nós *não* precisamos re-argumentar esse caso novamente. Remeto esses problemas brevemente apenas para que você os mantenha frescos em sua mente agora que voltamos nossa atenção para o mecanismo de 'classe' do ES6. Vamos demonstrar aqui como funciona, e ver se a `class` faz ou não algo substancial para resolver qualquer uma dessas preocupações de "class".

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

Além dessa sintaxe *parecendo* melhor, quais problemas o ES6 resolve?

1. Não há mais (bem, mais ou menos, veja abaixo!) referências a `.prototype` bagunçando o código.
2. `Button` é declarado diretamente para "herdar de" (também conhecido como `extends`) `Widget`, em vez de precisar usar `Object.create(..)` para substituir um objeto `.prototype` que está vinculado, ou ter para definir com `.__proto__` ou `Object.setPrototypeOf(..)`.
3. `super(..)` agora nos dá uma capacidade muito útil de **polimorfismo relativo**, de modo que qualquer método em um nível da cadeia pode referir-se a um método de mesmo nome relativamente um nível acima na cadeia. Isso inclui uma solução para a nota do Capítulo 4 sobre a estranheza dos construtores que não pertencem à sua classe e, portanto, não estão relacionados -- `super()` funciona dentro dos construtores exatamente como você esperaria.
4. A sintaxe literal `class` não possui recursos para especificar propriedades (somente métodos). Isso pode parecer limitante para alguns, mas espera-se que a grande maioria dos casos em que uma propriedade (estado) existe em outro lugar, exceto as "instâncias" da cadeia final, isso geralmente é um erro e surpreendente (já que é um estado implicitamente "compartilhado" entre todas as "instâncias"). Então, pode-se dizer que a sintaxe `class` está protegendo você de erros.
5. `extends` permite estender até mesmo (sub)tipos de objetos embutidos, como `Array` ou `RegExp`, de uma maneira muito natural. Fazê-lo sem `class .. extends` tem sido uma tarefa extremamente complexa e frustrante, que apenas os autores mais adeptos de frameworks foram capazes de lidar com precisão. Agora, será bastante trivial!

Com toda a justiça, essas são algumas soluções substanciais para muitos dos problemas (sintáticos) mais óbvios e surpresas que as pessoas têm com o código de estilo protótipo clássico.

## `class` Pegadinhas

Não é tudo chiclete e rosas, no entanto. Ainda existem alguns problemas profundos e profundamente preocupantes com o uso de "classes" como um padrão de design em JS.

Em primeiro lugar, a sintaxe `class` pode convencê-lo de que existe um novo mecanismo de "classe" no JS a partir do ES6. **Não é assim.** `class` é, principalmente, apenas açúcar sintático em cima do mecanismo `[[Prototype]]` (delegação!) existente.

Isso significa que `class` não está realmente copiando definições estaticamente no momento da declaração, como faz em linguagens tradicionais orientadas a classes. Se você alterar/substituir um método (de propósito ou por acidente) na "classe" pai, a "classe" e/ou instâncias filhas ainda serão "afetadas", pois não obtiveram cópias no momento da declaração, elas todos ainda estão usando o modelo de delegação ao vivo baseado em `[[Prototype]]`:

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

c1.rand(); // "Random: 432" -- oops!!!
```

Isso só parece um comportamento razoável *se você já sabe* sobre a natureza de delegação das coisas, ao invés de esperar *cópias* de "classes reais". Portanto, a pergunta a se fazer é: por que você está escolhendo a sintaxe `class` para algo fundamentalmente diferente das classes?

A sintaxe ES6 `class` não **simplesmente torna mais difícil** ver e entender a diferença entre classes tradicionais e objetos delegados?

A sintaxe `class` *não* fornece uma maneira de declarar propriedades de membros de classes (somente métodos). Então, se você precisa fazer isso para rastrear o estado compartilhado entre instâncias, você acaba voltando para a feia sintaxe `.prototype`, assim:

```js
class C {
	constructor() {
		// make sure to modify the shared state,
		// not set a shadowed property on the
		// instances!
		C.prototype.count++;

		// here, `this.count` works as expected
		// via delegation
		console.log( "Hello: " + this.count );
	}
}

// add a property for shared state directly to
// prototype object
C.prototype.count = 0;

var c1 = new C();
// Hello: 1

var c2 = new C();
// Hello: 2

c1.count === 2; // true
c1.count === c2.count; // true
```

O maior problema aqui é que ele trai a sintaxe `class` expondo (vazamento!) `.prototype` como um detalhe de implementação.

Mas, também temos a surpresa de que `this.count++` criaria implicitamente uma propriedade separada `.count` sombreada em objetos `c1` e `c2`, em vez de atualizar o estado compartilhado. `class` não nos oferece nenhum consolo para esse problema, exceto (presumivelmente) para sugerir por falta de suporte sintático que você não deveria estar fazendo isso *de forma alguma*.

Além disso, o sombreamento acidental ainda é um perigo:

```js
class C {
	constructor(id) {
		// oops, gotcha, we're shadowing `id()` method
		// with a property value on the instance
		this.id = id;
	}
	id() {
		console.log( "Id: " + id );
	}
}

var c1 = new C( "c1" );
c1.id(); // TypeError -- `c1.id` is now the string "c1"
```

Há também alguns problemas sutis com nuances sobre como o `super` funciona. Você pode supor que `super` seria vinculado de forma análoga a como `this` é vinculado (veja o Capítulo 2), que é que `super` sempre seria vinculado a um nível mais alto do que qualquer posição do método atual no ` [[Prototype]]` cadeia é.

No entanto, por motivos de desempenho (`esta` ligação já é cara), `super` não é vinculado dinamicamente. Está vinculado "estaticamente", como tempo de declaração. Não é grande coisa, certo?

Eh... talvez, talvez não. Se você, como a maioria dos desenvolvedores de JS, começar a atribuir funções a diferentes objetos (que vieram de definições `class`), de várias maneiras diferentes, você provavelmente não estará muito ciente de que, em todos esses casos, o mecanismo `super` sob as capas estão tendo que ser reencadernadas a cada vez.

E dependendo do tipo de abordagem sintática que você adota para essas atribuições, pode muito bem haver casos em que o `super` não possa ser vinculado adequadamente (pelo menos, não onde você suspeita), então você pode (no momento da redação, A discussão do TC39 está em andamento sobre o tópico) tem que ligar manualmente `super` com `toMethod(..)` (mais ou menos como você tem que fazer `bind(..)` para `this` -- veja o Capítulo 2).

Você está acostumado a ser capaz de atribuir métodos a diferentes objetos para *automaticamente* tirar vantagem do dinamismo de `this` através da regra de *ligação implícita* (veja o Capítulo 2). Mas o mesmo provavelmente não será verdade com métodos que usam `super`.

Considere o que `super` deve fazer aqui (contra `D` e `E`):

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

// Link E to D for delegation
Object.setPrototypeOf( E, D );

E.foo(); // "P.foo"
```

Se você estivesse pensando (bastante razoavelmente!) que `super` seria vinculado dinamicamente na hora da chamada, você poderia esperar que `super()` reconheceria automaticamente que `E` delega para `D`, então `E.foo( )` usando `super()` deve chamar `D.foo()`.

**Não é assim.** Por motivos de pragmatismo de desempenho, `super` não é *limitado tardio* (também conhecido como vinculado dinamicamente) como `this` é. Em vez disso, é derivado no momento da chamada de `[[HomeObject]].[[Prototype]]`, onde `[[HomeObject]]` é estaticamente vinculado no momento da criação.

Neste caso em particular, `super()` ainda está resolvendo para `P.foo()`, já que `[[HomeObject]]` do método ainda é `C` e `C.[[Prototype]]` é `P `.

Haverá *provavelmente* maneiras de resolver manualmente essas pegadinhas. Usar `toMethod(..)` para ligar/religar o `[[HomeObject]]` de um método (junto com a configuração do `[[Prototype]]` desse objeto!) parece funcionar neste cenário:

```js
var D = {
	foo: function() { console.log( "D.foo" ); }
};

// Link E to D for delegation
var E = Object.create( D );

// manually bind `foo`s `[[HomeObject]]` as
// `E`, and `E.[[Prototype]]` is `D`, so thus
// `super()` is `D.foo()`
E.foo = C.prototype.foo.toMethod( E, "foo" );

E.foo(); // "D.foo"
```

**Observação:** `toMethod(..)` clona o método e recebe `homeObject` como seu primeiro parâmetro (é por isso que passamos `E`), e o segundo parâmetro (opcionalmente) define um `name` para o novo método (que mantém em "foo").

Resta saber se existem outras pegadinhas que os desenvolvedores encontrarão além desse cenário. Independentemente disso, você terá que ser diligente e ficar ciente de quais locais o mecanismo descobre automaticamente 'super' para você e quais locais você precisa cuidar dele manualmente. **ECA!**

# Estático > Dinâmico?

Mas o maior problema de tudo sobre ES6 `class` é que todas essas várias pegadinhas significam `class` meio que opta por uma sintaxe que parece implicar (como classes tradicionais) que uma vez que você declara uma `class`, é uma definição estática de uma coisa (instanciada no futuro). Você perde completamente de vista o fato de que 'C' é um objeto, uma coisa concreta, com a qual você pode interagir diretamente.

Em linguagens tradicionais orientadas a classes, você nunca ajusta a definição de uma classe posteriormente, portanto, o padrão de design de classe não sugere tais recursos. Mas **uma das partes mais poderosas** do JS é que ele *é* dinâmico, e a definição de qualquer objeto é (a menos que você o torne imutável) uma *coisa* fluida e mutável.

`class` parece implicar que você não deve fazer tais coisas, forçando você a usar a sintaxe mais feia do `.prototype` para fazer isso, ou forçando você a pensar em `super` pegadinhas, etc. Ele também oferece *muito pouco* suporte para qualquer uma das armadilhas que esse dinamismo pode trazer.

Em outras palavras, é como se `class` estivesse lhe dizendo: "dinâmica é muito difícil, então provavelmente não é uma boa idéia. Aqui está uma sintaxe de aparência estática, então codifique suas coisas estaticamente."

Que comentário triste sobre JavaScript: **dinâmico é muito difícil, vamos fingir ser (mas não ser de fato!) estático**.

Estas são as razões pelas quais ES6 `class` está disfarçado como uma boa solução para dores de cabeça sintáticas, mas na verdade está turvando ainda mais as águas e tornando as coisas piores para JS e para uma compreensão clara e concisa.

**Nota:** Se você usar o utilitário `.bind(..)` para fazer uma função hard-bound (veja o Capítulo 2), a função criada não é subclassificável com `extend` do ES6 como as funções normais são.

## Revisão (TL;DR)

`class` faz um bom trabalho fingindo corrigir os problemas com o padrão de design de classe/herança em JS. Mas na verdade faz o oposto: **esconde muitos dos problemas e apresenta outros sutis, mas perigosos**.

`class` contribui para a confusão contínua de "classe" em JavaScript que tem atormentado a linguagem por quase duas décadas. Em alguns aspectos, ele faz mais perguntas do que responde, e na totalidade parece um ajuste muito antinatural em cima da elegante simplicidade do mecanismo `[[Prototype]]`.

Conclusão: se ES6 `class` torna mais difícil alavancar robustamente `[[Prototype]]` e oculta a natureza mais importante do mecanismo de objeto JS -- **os links de delegação ao vivo entre objetos** -- não deveria vemos `class` como criando mais problemas do que resolve, e apenas relegá-la a um anti-padrão?

Eu realmente não posso responder a essa pergunta para você. Mas espero que este livro tenha explorado completamente a questão em um nível mais profundo do que você jamais esteve antes, e tenha lhe dado as informações de que você precisa *para respondê-la você mesmo*.
