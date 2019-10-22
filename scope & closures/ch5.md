# You Don't Know JS: Escopos & Clausuras
# Capítulo 5: Clausuras de Escopo

Com esperança, até este ponto nós já alcançamos uma compreensão sólida e muito saudável de como escopo funciona.

Voltemos nossa atenção para uma incrivelmente importante, mas persistentemente ilusória, *quase mitológica*, parte da linguagem: **closure**. Se você acompanhou nossa discussão sobre escopo léxico até agora, a recompensa é que closure será, em grande parte, anticlímax, quase evidente por natureza. *Há um homem escondido atrás da cortina do mágico, e estamos prestes a vê-lo*. Não, seu nome não é Crockford!

No entanto, se você tem perguntas irritantes sobre escopo léxico, agora deve ser uma boa hora para voltar e rever o Capítulo 2, antes de prosseguir.

## Esclarecimento

Para aqueles que são um pouco experientes em JavaScript, mas que porventura nunca tenham entendido completamente o conceito de closures, *entender closure* pode parecer como um nirvana especial, que para atingí-lo é necessário muita luta e sacrifício.

Eu me lembro de anos atrás, quando eu tinha uma forte compreensão sobre JavaScript, mas não tinha ideia do que era closure. A sugestão de que havia *o outro lado* da linguagem, um que prometia ainda mais capacidade do que eu já possuia, me provocava. Me lembro de ler todo o código fonte dos primeiros frameworks tentando entender como eles realmente funcionavam. Lembro da primeira vez que algo do "módulo padrão" começou a surgir em minha mente. Eu me lembro dos momentos intensos de *a-ha!*.

O que eu não sabia naquela época, o que levei anos para entender, e o que eu espero transmitir para você em breve, é esse segredo: **em JavaScript, closure é tudo que estiver ao seu redor, você só precisa reconhecer e abraçar isso.** Closure não é uma ferramenta especial para a qual você deve aprender nova sintaxe e novos padrões. Não, closure não é uma arma que você aprende a manejar com destreza, como Luke treinado na Força.

Closures acontecem como um resultado da escrita de código que depende do escopo léxico. Elas apenas acontecem. Você nem sequer precisa criar closure intencionalmente para tirar proveito delas. Elas são criadas e usadas por você em todo o seu código. O que está faltando é o contexto mental apropriado para reconhecer, aceitar e alavancar closure por sua própria vontade.

O momento de esclarecimento deveria ser: **oh, closures já estão surgindo por todo o meu código, eu finalmente consigo *vê-las* agora.** O entendimento de closure é como quando Neo vê a Matrix pela primeira vez.

## Nitty Gritty

OK, chega de exageros e referências sem-vergonhas de filmes.

Aqui está uma definição curta e grossa do que você precisa saber para entender e reconhecer closure:

> Closure é quando uma função é capaz de lembrar e acessar seu escopo léxico, mesmo quando essa função está sendo executada fora do seu escopo léxico.

Vamos entrar em um pedaço de código para ilustrar esta definição.

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a ); // 2
	}

	bar();
}

foo();
```

Esse código pode parecer familiar das nossas discussões sobre Escopo Aninhado. A função `bar()` tem *acesso* à variável `a` do escopo ao redor por causa das regras de consulta ao escopo léxico (neste caso, é uma referência RHS).

Isso é "closure"?

Bem, tecnicamente... *talvez*. Mas para a nossa definição o-que-você-precisa-saber acima... *não exatamente*. Eu acredito que a forma mais correta de explicar `bar()` fazendo referência a `a` é por meio das regras de procura do escopo léxico, e essas regras são *apenas* (uma importante!) **parte** do que closure é.

De uma perspectiva puramente acadêmica, o que é dito do trecho acima é que a função `bar()` tem uma *closure* sobre o escopo de `foo()` (e, realmente, até sobre o resto dos escopos a que tem acesso, como o escopo global no nosso caso). Colocando de uma forma ligeiramente diferente, é dito que `bar()` se fecha sobre o escopo de `foo()`. Por quê? Porque `bar()` aparece aninhado dentro de `foo()`. Claro e simples.

Mas, definida dessa forma, closure não é diretamente *observável*, nós nem vimos closure *exercida* nesse trecho. Nós vemos claramente o escopo léxico, mas closure permanece uma espécie de sombra misteriosa se deslocando por trás do código.

Vamos considerar, então, um código que trás closure totalmente para a luz:

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 2 -- Whoa, observamos closure, man.
```

A função `bar()` tem acesso léxico ao escopo interno de `foo()`. Mas, em seguida, nós temos `bar()`, a função em si, e passamos ela *como* um valor. Nesse caso, nós retornamos (`return`) o próprio objeto da função que `bar` faz referência.

Depois de executarmos `foo()`, nós atribuímos o valor retornado (nossa função interna `bar()`) para a variável chamada `baz` e então, nós invocamos `baz()`, que certamente está invocando nossa função interna `bar()`, apenas com um identificador diferente.

`bar()` é executada, com certeza. Mas, nesse caso, é executada *fora* do seu escopo léxico declarado.

Após a execução de `foo()`, normalmente nós esperamos que todo o escopo interno de `foo()` vá embora, porque nós sabemos que o *Motor* usa um *Coletor de Lixo* que vem paralelamente e libera a memória uma vez que não está mais em uso. Já que o conteúdo de `foo()` parece não está mais em uso, é natural que ele deve ser considerado *passado*.

Mas a "mágica" das closures não permite que isso aconteça. Esse escopo interno, de fato, *ainda* está "em uso" e, portanto, ele não desaparece. Quem está usando? **A função `bar()`**.

Em virtude de onde foi declarada, `bar()` tem uma closure sobre o escopo interno de `foo()`, que mantem esse escopo vivo para `bar()` fazer referência a qualquer momento posterior.

**`bar()` ainda possui uma referência para esse escopo, e essa referência é chamada de closure.**

Então, uns poucos microssegundos depois, quando a variável `baz` é chamada (chamando a função interna que inicialmente atribuímos o nome de `bar`), ela tem o devido acesso ao escopo léxico escrito no tempo da autoria do código, para que ela possa acessar à variável `a` exatamente como esperávamos.

A função está sendo invocada de forma adequada fora do seu escopo léxico de origem. **Closure** permite que a função continue a acessar o escopo léxico no qual foi definida no momento da sua concepção.

Naturalmente, qualquer uma das várias maneiras pelas quais as funções podem ser *passadas* como valores e, de fato, invocadas em outros lugares, são exemplos de observação/uso de *closure*.

```js
function foo() {
	var a = 2;

	function baz() {
		console.log( a ); // 2
	}

	bar( baz );
}

function bar(fn) {
	fn(); // olhe, mamãe, eu vejo closure!
}
```

Nós passamos a função interna `baz` para `bar`, chamamos essa função interna (agora rotulada de `fn`) e, quando fazemos isso, sua closure sobre o escopo interno de `foo()` é observada, acessando `a`.

Estas passagens de funções também podem ocorrer de forma indireta.

```js
var fn;

function foo() {
	var a = 2;

	function baz() {
		console.log( a );
	}

	fn = baz; // atribuindo `baz` à variável global
}

function bar() {
	fn(); // olhe, mamãe, eu vejo closure!
}

foo();

bar(); // 2
```

Seja qual for a forma que usarmos para *transportar* uma função interna para fora do seu escopo léxico, ela irá manter uma referência de escopo de onde ela for declarada originalmente, e onde for que a executarmos, essa closure irá ocorrer.

## Agora eu posso ver

Os fragmentos de código anteriores são um tanto acadêmicos e artificialmente construídos para ilustrar o *uso de closures*. Mas eu prometi para você uma coisa mais que apenas um novo brinquedinho. Eu prometi que closure seria uma coisa ao seu redor, em sou código existente. Vamos agora *ver* essa verdade.

```js
function wait(message) {

	setTimeout( function timer(){
		console.log( message );
	}, 1000 );

}

wait( "Hello, closure!" );
```

Tomamos uma função interna (chamada `timer`) e passamos ela para o `setTimeout(..)`. Mas `timer` tem um escopo fechado sobre o escopo de `wait(..)`, mantendo e usando uma referência para a variável `message`.

Mil milésimos de segundo depois de executarmos `wait(..)`, e seu escopo interno deveria ter sido extinto há muito tempo, a tal função interna `timer` ainda tem uma closure sobre esse escopo.

Lá no fundo, nas entranhas do *Motor*, o utilitário embutido `setTimeout(..)` faz uma referência por algum parâmetro, provavelmente chamado `fn`, ou `func`, ou alguma coisa do tipo. O *Motor* vai invocar essa função, que está invocando nossa função interna `timer`, e a referência do escopo léxico ainda está intacta.

**Closure.**

Ou, se você é da religião do jQuery (ou algum framework JS, para este caso):

```js
function setupBot(name,selector) {
	$( selector ).click( function activator(){
		console.log( "Activating: " + name );
	} );
}

setupBot( "Closure Bot 1", "#bot_1" );
setupBot( "Closure Bot 2", "#bot_2" );
```

Não tenho certeza do tipo de código você escreve, mas eu normalmente escrevo código que é responsável por controlar todo um exército mundial de drones de closure bots, então é totalmente realista!

(Algumas) brincadeiras à parte, essencialmente *sempre que* e *onde quer que* você trate funções (que acessam seu respectivo escopo léxico) como valores de primeira classe e as passe por aí, provavelmente você vê aquelas funções que exercem closure. Sejam elas timers, manipuladores de eventos, requisições Ajax, mensagens de janelas cruzadas, web workers, ou alguma outra tarefa assíncrona (ou síncrona!), quando você passa em uma *função de callback*, prepare-se para lançar algumas closures por aí!

**Nota:** O capítulo 3 introduz o padrão IIFE. Embora seja frequentemente dito que IIFE (sozinho) é um exemplo de closure, Eu devo discordar um pouco, pela nossa definição acima.

```js
var a = 2;

(function IIFE(){
	console.log( a );
})();
```

Esse código "funciona", mas isso não é rigorosamente um exemplo de closure. Por quê? Porque a função (que aqui nós nomeamos de "IIFE") não é executada fora do seu escopo léxico. Ela ainda é chamada bem ali, no mesmo escopo que foi declarada (o escopo ao redor/global que também contém `a`). `a` é encontrado pelo look-up normal do escopo léxico, não exatamente por closure.

Enquanto a closure poderia técnicamente estar acontecendo na hora da declaração, _não_ é estritamente observável, e então, como eles dizem, _é uma árvore caindo na floresta sem ninguém por perto para ouvir._

Embora uma IIFE não seja *em si* um exemplo de closure, ela realmente cria escopo e é uma das ferramentas mais comuns que usamos para criar um escopo que pode ser fechado. Portanto, as IIFEs estão, de fato, profundamente relacionadas a closures, mesmo que não exerçam closure por sí mesmas.

Abaixe esse livro agora, querido leitor. Tenho uma tarefa para você. Abra algum código JavaScript recente. Procure por suas funções-como-valores e identifique onde você já está usando closures e talvez ainda nem sabia disso.

Eu espero.

Agora... você vê!

## Loops + Closure

O exemplo canônico mais comum usado para ilustrar closures envolve o humilde loop for.

```js
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

**Nota:** Linters frequentemente reclamam quando você coloca funções dentro de loops, porque os erros do não entendimento de closures são **muito comuns entre desenvolvedores**. Nós explicaremos como fazer apropriadamente aqui, liberando o total poder do closure. Mas essa sutileza é frequentemente perdida em linters e eles vão reclamar, assumindo que, *na verdade*, você não sabe o que está fazendo. 

A essência desse trecho de código é que nós normalmente *esperaríamos* que o comportamento fosse que os números "1", "2", .. "5" fossem impressos, um de cada vez, um por segundo, respectivamente.

De fato, se você rodar esse código, você terá o "6" impresso 5 vezes, no intervalo de 1 segundo.

**Que?**

Primeiramente, vamos explicar de onde vem o `6`. A condição de encerramento do loop é quando `i` *não* é `<=5`. A primeira vez que isso será o caso é quando `i` é 6. Então, a saída está refletindo no valor final do `i` depois que o loop termina.

Na verdade, isso parece óbvio à segunda vista. Os callbacks da função timeout estão todos funcionando bem após a conclusão do loop. Na verdade, assim que o timer avança, mesmo que esteja `setTimeout(.., 0)` em cada iteração, todos esses callbacks da função ainda seriam executados estritamente após a conclusão do loop, e, assim, imprimir `6` a cada vez.

Mas há uma questão mais profunda em jogo aqui. O que *está faltando* em nosso código para realmente fazê-lo se comportar como nós semanticamente subentendemos?

O que está faltando é que estamos tentando *implicar* que cada iteração do loop "capture" sua própria cópia de `i`, no momento da iteração. Mas, a forma como o escopo funciona, todas essas 5 funções, embora sejam definidas separadamente em cada iteração do loop, todas **são encapsuladas pelo mesmo escopo global compartilhado**, que tem, de fato, apenas um `i` nele.

Colocando assim, *é claro* todas as funções compartilham uma referência ao mesmo `i`. Algo na estrutura do loop tende a nos confundir e a pensar que há algo mais sofisticado em funcionamento. Não há. Não há diferença se cada um dos 5 callbacks de timeout fossem declarados um logo após o outro, sem nenhum loop.

Ok, então, de volta à nossa questão crucial. O que está faltando? Nós precisamos de mais ~~rufando os tambores~~ closures. Especificamente, nós precisamos de um novo closure para cada iteração do loop.

Nós aprendemos no Capítulo 3 que a IIFE cria escopos declarando uma função e imediatamente executando-a.

Vamos tentar:

```js
for (var i=1; i<=5; i++) {
	(function(){
		setTimeout( function timer(){
			console.log( i );
		}, i*1000 );
	})();
}
```

Isso funciona? Tente. De novo, eu vou esperar.

Eu vou acabar com o suspense para você. **Não.** Mas, por quê? Nós agora obviamente temos mais escopo léxico. Cada callback da função timeout está, de fato, fechado no seu próprio escopo de iteração criado respectivamente por cada IIFE.

Não é suficiente ter um escopo para encapsular **se o escopo está vazio**. Olhe mais perto. Nosso IIFE é apenas um escopo vazio que faz nada. Ele precisa de *algo* nele para ser útil para nós.

Ele precisa da sua própria variável, com uma cópia do valor do `i` em cada iteração.

```js
for (var i=1; i<=5; i++) {
	(function(){
		var j = i;
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})();
}
```

**Eureka! Funciona!**

Uma ligeira variação que alguns preferem é:

```js
for (var i=1; i<=5; i++) {
	(function(j){
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})( i );
}
```

É claro, desde que esse IIFEs sejam apenas funções, nós podemos passá-las no `i`, e podemos chamar de `j` se preferirmos, ou podemos mesmo chamar de `i` de novo. De qualquer forma, o código funciona agora.

O uso de um IIFE dentro de cada iteração cria um novo escopo para cada uma, o que dá aos callbacks da função timeout a oportunidade de fechar um novo escopo para cada iteração, cada uma terá uma variável com o valor certo do iterador para acessarmos.

Problema resolvido!

### Escopo do Bloco Revisitado

Olhe atentamente para nossa análise da solução anterior. Nós usamos um IIFE para criar um novo escopo por iteração. Em outras palavras, nós na verdade *precisamos* de um **bloco de escopo** por iteração. O Capítulo 3 nos mostrou a declaração `let`, que sequestra um bloco e declara uma variável bem ali.

**Ele essencialmente transforma o bloco em um escopo que podemos fechar.** Então, o impressionante código a seguir "simplesmente funciona":

```js
for (var i=1; i<=5; i++) {
	let j = i; // Isso, escopo de bloco para closure!
	setTimeout( function timer(){
		console.log( j );
	}, j*1000 );
}
```

*Mas isso não é tudo!* (na minha melhor voz de Bob Barker). Há um comportamento especial definido para declarações `let` usadas no topo de um loop for. Esse comportamento diz que a variável vai ser declarada não apenas uma vez no loop, **mas a cada iteração**. E será, proveitosamente, inicializada em cada iteração subsequente com o valor final da iteração anterior.

```js
for (let i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

Quão legal é isso? Escopo de bloco e closure funcionam lado a lado, resolvendo todos os problemas do mundo. Eu não sei quanto a você, mas isso me faz um desenvolvedor Javascript feliz.

## Módulos

Há outros padrões de código que elevam o poder do closure mas que não aparentam superficialmente ser sobre callbacks. Vamos examinar o mais poderoso deles: *o módulo*

```js
function foo() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}
}
```

Como esse código está agora, não há um closure visível acontecendo. Nós simplesmente temos alguns dados de variáveis privadas `something` e `another`, e um par de funções internas `doSomething()` e `doAnother()`, ambas têm escopo léxico (e assim, closure!) sobre a o escopo interno de `foo()`.

Mas considere:

```js
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

Esse é o padrão que chamamos em Javascript de *módulo*. A forma mais comum de implementar o padrão de módulo é frequentemente chamado "Módulo Revelado", e é uma variação do que apresentamos aqui.

Vamos examinar algumas coisas sobre esse código.

Primeiramente, `CoolModule()` é apenas uma função, mas ela *tem que ser invocada* para que seja criada uma instância do módulo. Sem a execução de uma função externa, a criação do escopo interno e closures não vai acontecer.

Em segundo, a função `CoolModule()` retorna um objeto, denotado pela sintaxe literal de objeto `{ chave: valor, ... }`. O objeto que retornamos tem como referência nossas funções internas, mas *não* com os dados das nossas variáveis internas. Nós as mantemos privadas e escondidas. É apropriado pensar nesse valor do objeto retornado essencialmente como uma **API pública para nosso módulo**.

Esse valor de objeto retornado é associado à variável externa `foo`, e então nós podemos acessar os métodos apropriados na API, como `foo.doSomething()`.

**Nota:** Não é obrigatório que nós retornemos um objeto de fato(literal) para nosso módulo. Nós podemos apenas retornar uma função interna diretamente. JQuery é na verdade um bom exemplo disso. Os identificadores `jQuery` e `$` são as API públicas para o "módulo" JQuery, mas elas são, propriamente, apenas uma função (que podem ter propriedades, já que todas as funções são objetos).

As funções `doSomething()` e `doAnother()` têm closure no escopo interno da "instância" do módulo (que chegou quando invocamos `CoolModule()`). Quando nós transportamos essas funções para fora do escopo léxico, através de propriedades referenciadas no objeto que retornamos, nós agora definimos uma condição para que cada closure possa ser observado e exercido.

Para colocar de forma mais simples, existem dois "requisitos" para que o padrão de módulo seja exercido:

1. É preciso existir uma função de inclusão externa, e ela precisa ser invocada pelo menos uma vez (cada vez cria uma nova instância do módulo).

2. A função de inclusão precisa retornar pelo menos uma função interna, então essa função interna tem um closure sobre o escopo privado, e pode acessar e/ou modificar esse estado privado.

Um objeto com uma propriedade de função sozinha não é *realmente* um módulo. Um objeto que é retornado de uma invocação de função que só possui propriedades de dados e nenhuma função fechada não é * realmente * um módulo, no sentido observável.

O trecho de código acima mostra um criador de módulo autônomo chamado `CoolModule()` que pode ser invocado inúmeras vezes, cada vez criando uma nova instância do módulo. Uma leve variação nesse padrão é quando você apenas se importa em ter uma instância, um "juntado"(singleton) dos tipos:

```js
var foo = (function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

Aqui, nós tornamos nossa função de módulo em uma IIFE (veja o Capítulo 3), e nós *imediatamente* invocamos e associamos seu valor de retorno diretamente ao nosso identificador de instância de módulo único `foo`.

Módulos são apenas funções, então eles podem receber parâmetros:

```js
function CoolModule(id) {
	function identify() {
		console.log( id );
	}

	return {
		identify: identify
	};
}

var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );

foo1.identify(); // "foo 1"
foo2.identify(); // "foo 2"
```

Outra pequena, mas poderosa, variação no padrão de módulo é nomear o objeto que você está retornando como sua API pública:

```js
var foo = (function CoolModule(id) {
	function change() {
		// modificando a API pública
		publicAPI.identify = identify2;
	}

	function identify1() {
		console.log( id );
	}

	function identify2() {
		console.log( id.toUpperCase() );
	}

	var publicAPI = {
		change: change,
		identify: identify1
	};

	return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

Mantendo uma referência interna ao objeto da API pública dentro da sua intância de módulo, você pode modificar aquela instância de módulo  **de dentro**, incluindo adição e remoção de métodos, propriedades, *e* alterar seus valores.

### Módulos Modernos

Vários carregadores/gerenciadores de dependências de módulos essencialmente encapsulam essa definição de padrão de módulo em uma API amigável. Em vez  de examinar cada uma das bibliotecas particularmente, deixe-me apresentar uma *muito simples* prova de conceito **para propósitos ilustrativos (apenas)**:

```js
var MyModules = (function Manager() {
	var modules = {};

	function define(name, deps, impl) {
		for (var i=0; i<deps.length; i++) {
			deps[i] = modules[deps[i]];
		}
		modules[name] = impl.apply( impl, deps );
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();
```

A parte chave desse código é `modules[name] = impl.apply(impl, deps)`. Isso está invocando a função de definição de encapsulamento para um módulo (passando em qualquer dependência), e guardando o valor retornado, a API do módulo, em uma lista interna de módulos rastreado pelo nome.

E aqui está como eu posso usá-lo para definir alguns módulos:

```js
MyModules.define( "bar", [], function(){
	function hello(who) {
		return "Let me introduce: " + who;
	}

	return {
		hello: hello
	};
} );

MyModules.define( "foo", ["bar"], function(bar){
	var hungry = "hippo";

	function awesome() {
		console.log( bar.hello( hungry ).toUpperCase() );
	}

	return {
		awesome: awesome
	};
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

Ambos os módulos, "foo" e "bar", são definidos com a função que retorna uma API pública. "foo" até recebe a instância de "bar" como um parâmetro de  dependência e pode usa-lo de acordo.

Gaste algum tempo examinando esses trechos de código para entender completamente o poder dos closures colocados em uso para nossos propósitos. O ponto crucial é que na verdade não existe nenhuma "mágica" em gerenciadores de módulo. Eles complementam ambas características do padrão de módulo que listei acima: invocando um encapsulador de definição de função, e mantendo seu valor de retorno como a API para aquele módulo.

Em outras palavras, módulos são apenas módulos, mesmo se você colocar uma ferramenta de encapsulamento amigável no topo dele.

### Módulos do Futuro

O ES6 adicionou suporte à sintaxe do conceito de primeira classe dos módulos. Quando carregado via sistema do módulo, o ES6 trata um arquivo como um módulo separado. Cada módulo pode tanto importar outros módulos ou especificar membros de API, assim como exportar seus pŕoprios membros de API pública.

**Nota:** Módulos baseados em funções não são, estatisticamente, padrões reconhecidos (alguma coisa o compilador reconhece), então sua semântica da API não os consideram até o tempo de execução(*runtime*). Isso é, você pode de fato modificar a API de um módulo durante o tempo de execução ((*runtime*) - Veja a discussão anterior de `publicAPI`).

Em contraste, APIs de módulos ES6 são estáticos (a API não muda em tempo de execução(*runtime*)). Desde que o compilador saiba *disso*, ele pode (e vai!) verificar durante (o carregamento de arquivo) a compilação que uma referência a um membro da API de um módulo importado *existe de fato*. Se a referência a API não exsitir, o compilador lança um erro "antecipado" na compilação, em vez de esperar pela dinâmica de resolução em tempo de execução(*runtime*) tradicional (e erros, se existirem).

Módulos ES6 **não** possuem um formato "inline", eles devem ser definidos em arquivos separados (um por módulo). O browser/motor tem um "carregador de módulo" padrão (que é substituível, mas isso está muito além da nossa discussão aqui) que de forma síncrona carrega um arquivo de módulo quando ele é importado.

Considere:

**bar.js**
```js
function hello(who) {
	return "Let me introduce: " + who;
}

export hello;
```

**foo.js**
```js
// importa apenas `hello()` so módulo "bar"
import hello from "bar";

var hungry = "hippo";

function awesome() {
	console.log(
		hello( hungry ).toUpperCase()
	);
}

export awesome;
```

```js
// importa todo o módulo "foo" e "bar"
module foo from "foo";
module bar from "bar";

console.log(
	bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

**Nota:** Arquivos separados **"foo.js"** e **"bar.js"** precisarão ser criados com os conteúdos como mostrados no primeiro trecho de código, respectivamente. Então, seu programa vai carregar/importar esses módulos para usá-los, como mostrado no terceiro trecho de código.

`import` importa um ou mais membros de uma API de módulo dentro do escopo atual, cada um ligado à uma variável (no nosso caso, `hello`). `module` importa toda uma API do módulo ligado à uma variável (em nosso caso, `foo` e `bar`). `export` exporta um identificador (variável, função) para a API pública para o módulo atual. Esses operadores podem ser usados quantas vezes forem necessários em uma definição de módulo.

O conteúdo dentro do *arquivo de módulo* é tratado como se fosse encapsulado em um escopo de closure, assim como com os módulos de funções closure vistos anteriormente.

## Revisão (TL;DR)

Closure parece para os não-iluminados como um mundo místico à parte dentro do Javascript que apenas as poucas almas corajosas podem alcançar. Mas é na verdade apenas um padrão e quase um fato óbvio de como nós escrevemos código em um ambiente de escopo léxico, onde funções são valores e podem ser passados adiante à vontade.

**Closure é quando uma função consegue lembrar e acessar seu escopo léxico mesmo quando invocada fora do seu escopo léxico.**

 Closures podem nos enganar, por exemplo, com loops, se não tivermos cuidado para vê-los e saber como eles funcionam. Mas eles também são uma ferramenta imensamente poderosa, permitindo padrões como os *módulos* em suas variadas formas.

Módulos requerem duas características chave: 1) uma função de encapsulamento externa ser invocada, para criar o escopo fechado 2) o valor de retorno da função de encapsulamento deve incluir referência para ao menos uma função interna que então tenha closure sobre o escopo privado interno do encapsulamento.

Agora nós podemos ver closures em todo nosso código existente, e temos a habilidade para reconhece-los e aproveitá-los para nosso próprio benefício!
