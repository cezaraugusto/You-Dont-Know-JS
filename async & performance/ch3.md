# You Don't Know JS: Async & Performance
# Capítulo 3: Promises

No Capítulo 2, identificamos duas grandes categorias de deficiências ao usar callbacks para expressar a assincronia de um programa e gerenciar concorrência: falta de sequencialidade e falta de confiabilidade. Agora que entendemos os problemas de forma mais íntima, está na hora de voltarmos nossa atenção para padrões que possam resolvê-los.

A questão que queremos resolver primeiro é a *inversão de controle*, a confiança que é mantida de forma tão frágil e perdida tão facilmente.

Lembre-se de que encapsulamos a *continuação* do nosso programa em uma função de callback, entregamos esse callback para outra parte (potencialmente até código externo) e simplesmente cruzamos os dedos para que ela faça a coisa certa com a invocação do callback.

Fazemos isso porque queremos dizer: "aqui está o que acontece *depois*, após o passo atual terminar."

Mas e se pudéssemos desinverter essa *inversão de controle*? E se, em vez de entregar a continuação do nosso programa para outra parte, pudéssemos esperar que ela nos retornasse uma capacidade de saber quando sua tarefa termina, e então nosso código pudesse decidir o que fazer em seguida?

Esse paradigma é chamado de **Promises**.

As Promises estão começando a tomar o mundo JS de assalto, à medida que desenvolvedores e autores de especificações buscam desesperadamente desemaranhar a insanidade do inferno dos callbacks em seu código/design. De fato, a maioria das novas APIs assíncronas sendo adicionadas à plataforma JS/DOM está sendo construída sobre Promises. Então provavelmente é uma boa ideia se aprofundar e aprendê-las, não acha!?

**Nota:** A palavra "imediatamente" será usada com frequência neste capítulo, geralmente para se referir a alguma ação de resolução de Promise. No entanto, em essencialmente todos os casos, "imediatamente" significa em termos do comportamento da fila de Jobs (veja o Capítulo 1), e não no sentido estritamente síncrono de *agora*.

## O Que É uma Promise?

Quando desenvolvedores decidem aprender uma nova tecnologia ou padrão, geralmente o primeiro passo é "Me mostra o código!". É bem natural para nós simplesmente mergulharmos de cabeça e aprender enquanto avançamos.

Mas acontece que algumas abstrações se perdem quando olhamos apenas para as APIs. Promises são uma dessas ferramentas em que pode ficar dolorosamente óbvio, pela forma como alguém a usa, se a pessoa entende para que ela serve e do que ela trata, em vez de apenas aprender e usar a API.

Então, antes de mostrar o código de Promise, quero explicar plenamente o que uma Promise realmente é conceitualmente. Espero que isso te oriente melhor à medida que você explora como integrar a teoria de Promises ao seu próprio fluxo assíncrono.

Com isso em mente, vamos olhar para duas analogias diferentes sobre o que uma Promise *é*.

### Valor Futuro

Imagine este cenário: chego ao balcão de um restaurante de fast-food e faço um pedido de um cheeseburger. Entrego ao caixa $1,47. Ao fazer meu pedido e pagar por ele, fiz uma requisição por um *valor* de volta (o cheeseburger). Iniciei uma transação.

Mas, muitas vezes, o cheeseburger não está imediatamente disponível para mim. O caixa me entrega algo no lugar do meu cheeseburger: um recibo com um número de pedido nele. Esse número de pedido é uma *promessa* de "te devo" (IOU, do inglês "I owe you") que garante que, eventualmente, eu deverei receber meu cheeseburger.

Então fico segurando meu recibo e o número do pedido. Sei que ele representa meu *cheeseburger futuro*, então não preciso mais me preocupar com isso -- a não ser por estar com fome!

Enquanto espero, posso fazer outras coisas, como enviar uma mensagem de texto para um amigo dizendo: "Ei, você pode vir almoçar comigo? Vou comer um cheeseburger."

Já estou raciocinando sobre meu *cheeseburger futuro*, mesmo sem tê-lo em mãos ainda. Meu cérebro consegue fazer isso porque está tratando o número do pedido como um marcador de posição (placeholder) para o cheeseburger. O marcador de posição essencialmente torna o valor *independente do tempo*. Ele é um **valor futuro**.

Eventualmente, ouço "Pedido 113!" e caminho alegremente de volta ao balcão com o recibo na mão. Entrego meu recibo ao caixa e recebo meu cheeseburger em troca.

Em outras palavras, uma vez que meu *valor futuro* estava pronto, troquei minha promessa-de-valor pelo valor em si.

Mas há outro desfecho possível. Eles chamam meu número de pedido, mas quando vou buscar meu cheeseburger, o caixa me informa, com pesar: "Sinto muito, mas parece que estamos sem cheeseburgers." Deixando de lado por um momento a frustração do cliente nesse cenário, podemos ver uma característica importante dos *valores futuros*: eles podem indicar tanto um sucesso quanto uma falha.

Toda vez que peço um cheeseburger, sei que ou vou conseguir um cheeseburger eventualmente, ou vou receber a triste notícia da falta de cheeseburgers, e terei que descobrir outra coisa para almoçar.

**Nota:** No código, as coisas não são tão simples, porque, metaforicamente, o número do pedido pode nunca ser chamado, caso em que ficamos indefinidamente em um estado não resolvido. Voltaremos a lidar com esse caso mais adiante.

#### Valores Agora e Depois

Tudo isso pode parecer abstrato demais para se aplicar ao seu código. Então vamos ser mais concretos.

No entanto, antes de podermos introduzir como as Promises funcionam dessa maneira, vamos derivar, com código que já entendemos -- callbacks! --, como lidar com esses *valores futuros*.

Quando você escreve código para raciocinar sobre um valor, como fazer contas com um `number`, quer perceba ou não, você esteve assumindo algo muito fundamental sobre aquele valor, que é o fato de ele já ser um valor concreto de *agora*:

```js
var x, y = 2;

console.log( x + y ); // NaN  <-- porque `x` ainda não foi definido
```

A operação `x + y` assume que tanto `x` quanto `y` já estão definidos. Em termos que detalharemos em breve, assumimos que os valores `x` e `y` já estão *resolvidos*.

Seria um absurdo esperar que o operador `+` por si só fosse de alguma forma magicamente capaz de detectar e esperar até que tanto `x` quanto `y` estivessem resolvidos (ou seja, prontos), só então realizando a operação. Isso causaria caos no programa se diferentes instruções terminassem *agora* e outras terminassem *depois*, certo?

Como você poderia raciocinar sobre as relações entre duas instruções se uma delas (ou ambas) talvez não tivesse terminado ainda? Se a instrução 2 depende de a instrução 1 ter terminado, há apenas dois desfechos: ou a instrução 1 terminou *agora* mesmo e tudo prossegue bem, ou a instrução 1 ainda não terminou, e portanto a instrução 2 vai falhar.

Se esse tipo de coisa soa familiar do Capítulo 1, ótimo!

Vamos voltar à nossa operação matemática `x + y`. Imagine se houvesse uma forma de dizer: "Some `x` e `y`, mas se algum deles ainda não estiver pronto, apenas espere até que estejam. Some-os assim que puder."

Seu cérebro pode ter pulado direto para callbacks. OK, então...

```js
function add(getX,getY,cb) {
	var x, y;
	getX( function(xVal){
		x = xVal;
		// ambos estão prontos?
		if (y != undefined) {
			cb( x + y );	// envia a soma adiante
		}
	} );
	getY( function(yVal){
		y = yVal;
		// ambos estão prontos?
		if (x != undefined) {
			cb( x + y );	// envia a soma adiante
		}
	} );
}

// `fetchX()` e `fetchY()` são funções
// síncronas ou assíncronas
add( fetchX, fetchY, function(sum){
	console.log( sum ); // foi fácil, né?
} );
```

Reserve apenas um momento para deixar a beleza (ou a falta dela) desse trecho de código assentar (assoviando pacientemente).

Embora a feiura seja inegável, há algo muito importante a se notar sobre esse padrão assíncrono.

Naquele trecho, tratamos `x` e `y` como valores futuros, e expressamos uma operação `add(..)` que (de fora) não se importa se `x`, ou `y`, ou ambos estão disponíveis imediatamente ou não. Em outras palavras, ela normaliza o *agora* e o *depois*, de modo que podemos contar com um resultado previsível da operação `add(..)`.

Ao usar um `add(..)` que é temporalmente consistente -- ele se comporta da mesma forma ao longo dos tempos *agora* e *depois* --, o código assíncrono fica muito mais fácil de raciocinar.

Colocando de forma mais clara: para lidar consistentemente tanto com *agora* quanto com *depois*, tornamos ambos *depois*: todas as operações se tornam assíncronas.

É claro que essa abordagem grosseira baseada em callbacks deixa muito a desejar. É apenas um primeiro pequeno passo rumo à percepção dos benefícios de raciocinar sobre *valores futuros* sem se preocupar com o aspecto temporal de quando ele está disponível ou não.

#### Valor de Promise

Com certeza entraremos em muito mais detalhes sobre Promises mais adiante no capítulo -- então não se preocupe se parte disso for confusa --, mas vamos apenas dar uma breve olhada em como podemos expressar o exemplo `x + y` via `Promise`s:

```js
function add(xPromise,yPromise) {
	// `Promise.all([ .. ])` recebe um array de promises,
	// e retorna uma nova promise que espera por todas
	// elas terminarem
	return Promise.all( [xPromise, yPromise] )

	// quando essa promise é resolvida, vamos pegar os
	// valores `X` e `Y` recebidos e somá-los.
	.then( function(values){
		// `values` é um array das mensagens das
		// promises resolvidas anteriormente
		return values[0] + values[1];
	} );
}

// `fetchX()` e `fetchY()` retornam promises para
// seus respectivos valores, que podem estar prontos
// *agora* ou *depois*.
add( fetchX(), fetchY() )

// recebemos de volta uma promise para a soma desses
// dois números.
// agora encadeamos a chamada de `then(..)` para esperar
// pela resolução daquela promise retornada.
.then( function(sum){
	console.log( sum ); // foi mais fácil!
} );
```

Há duas camadas de Promises nesse trecho.

`fetchX()` e `fetchY()` são chamadas diretamente, e os valores que elas retornam (promises!) são passados para `add(..)`. Os valores subjacentes que essas promises representam podem estar prontos *agora* ou *depois*, mas cada promise normaliza o comportamento para ser o mesmo independentemente disso. Raciocinamos sobre os valores `X` e `Y` de uma forma independente do tempo. Eles são *valores futuros*.

A segunda camada é a promise que `add(..)` cria (via `Promise.all([ .. ])`) e retorna, pela qual esperamos chamando `then(..)`. Quando a operação `add(..)` é concluída, nosso *valor futuro* `sum` está pronto e podemos imprimi-lo. Escondemos dentro de `add(..)` a lógica para esperar pelos *valores futuros* `X` e `Y`.

**Nota:** Dentro de `add(..)`, a chamada `Promise.all([ .. ])` cria uma promise (que está esperando `promiseX` e `promiseY` resolverem). A chamada encadeada `.then(..)` cria outra promise, que a linha `return values[0] + values[1]` resolve imediatamente (com o resultado da soma). Assim, a chamada `then(..)` que encadeamos ao final da chamada `add(..)` -- no fim do trecho -- está na verdade operando sobre aquela segunda promise retornada, e não sobre a primeira criada por `Promise.all([ .. ])`. Além disso, embora não estejamos encadeando nada ao final daquele segundo `then(..)`, ele também criou outra promise, caso tivéssemos escolhido observá-la/usá-la. Esse assunto de encadeamento de Promises será explicado em muito mais detalhes mais adiante neste capítulo.

Assim como nos pedidos de cheeseburger, é possível que a resolução de uma Promise seja uma rejeição (rejection) em vez de uma realização (fulfillment). Diferentemente de uma Promise realizada, em que o valor é sempre programático, um valor de rejeição -- comumente chamado de "motivo da rejeição" (rejection reason) -- pode tanto ser definido diretamente pela lógica do programa quanto resultar implicitamente de uma exceção em tempo de execução.

Com Promises, a chamada `then(..)` pode na verdade receber duas funções, a primeira para a realização (como mostrado anteriormente) e a segunda para a rejeição:

```js
add( fetchX(), fetchY() )
.then(
	// handler de realização
	function(sum) {
		console.log( sum );
	},
	// handler de rejeição
	function(err) {
		console.error( err ); // que pena!
	}
);
```

Se algo deu errado ao obter `X` ou `Y`, ou algo de alguma forma falhou durante a soma, a promise que `add(..)` retorna é rejeitada, e o segundo callback handler de erro passado para `then(..)` receberá o valor de rejeição da promise.

Como as Promises encapsulam o estado dependente do tempo -- esperando pela realização ou rejeição do valor subjacente --, de fora, a própria Promise é independente do tempo, e assim as Promises podem ser compostas (combinadas) de formas previsíveis, independentemente do tempo ou do resultado por baixo.

Além disso, uma vez que uma Promise é resolvida, ela permanece assim para sempre -- ela se torna um *valor imutável* nesse ponto -- e pode então ser *observada* tantas vezes quantas forem necessárias.

**Nota:** Como uma Promise é externamente imutável uma vez resolvida, agora é seguro passar esse valor adiante para qualquer parte e saber que ele não pode ser modificado acidentalmente ou maliciosamente. Isso é especialmente verdadeiro em relação a múltiplas partes observando a resolução de uma Promise. Não é possível que uma parte afete a capacidade de outra parte de observar a resolução da Promise. Imutabilidade pode soar como um tópico acadêmico, mas é na verdade um dos aspectos mais fundamentais e importantes do design de Promises, e não deveria ser passado por cima de forma despreocupada.

Esse é um dos conceitos mais poderosos e importantes a se entender sobre Promises. Com uma boa quantidade de trabalho, você poderia criar de forma improvisada (ad hoc) os mesmos efeitos com nada além de feia composição de callbacks, mas essa não é realmente uma estratégia eficaz, especialmente porque você teria que fazer isso repetidamente.

Promises são um mecanismo facilmente repetível para encapsular e compor *valores futuros*.

### Evento de Conclusão

Como acabamos de ver, uma Promise individual se comporta como um *valor futuro*. Mas há outra forma de pensar na resolução de uma Promise: como um mecanismo de controle de fluxo -- um isto-então-aquilo temporal -- para dois ou mais passos em uma tarefa assíncrona.

Vamos imaginar chamar uma função `foo(..)` para realizar alguma tarefa. Não sabemos nenhum de seus detalhes, nem nos importamos. Ela pode concluir a tarefa de imediato, ou pode levar um tempo.

Só precisamos saber quando `foo(..)` termina para que possamos seguir para nossa próxima tarefa. Em outras palavras, gostaríamos de uma forma de ser notificados da conclusão de `foo(..)` para que possamos *continuar*.

Ao bom estilo JavaScript, se você precisa escutar por uma notificação, provavelmente pensaria nisso em termos de eventos. Então poderíamos reformular nossa necessidade de notificação como uma necessidade de escutar por um evento de *conclusão* (ou *continuação*) emitido por `foo(..)`.

**Nota:** Se você o chama de "evento de conclusão" ou de "evento de continuação" depende da sua perspectiva. O foco está mais no que acontece com `foo(..)`, ou no que acontece *após* `foo(..)` terminar? Ambas as perspectivas são precisas e úteis. A notificação do evento nos diz que `foo(..)` *concluiu*, mas também que está OK *continuar* com o próximo passo. De fato, o callback que você passa para ser chamado na notificação do evento é, ele próprio, o que anteriormente chamamos de *continuação*. Como *evento de conclusão* tem um foco um pouco maior em `foo(..)`, que está mais em nossa atenção no momento, damos uma leve preferência a *evento de conclusão* no restante deste texto.

Com callbacks, a "notificação" seria nosso callback invocado pela tarefa (`foo(..)`). Mas com Promises, invertemos a relação, e esperamos poder escutar por um evento de `foo(..)` e, quando notificados, prosseguir de acordo.

Primeiro, considere algum pseudocódigo:

```js
foo(x) {
	// começa a fazer algo que pode levar um tempo
}

foo( 42 )

on (foo "completion") {
	// agora podemos fazer o próximo passo!
}

on (foo "error") {
	// ops, algo deu errado em `foo(..)`
}
```

Chamamos `foo(..)` e então configuramos dois ouvintes de evento (event listeners), um para `"completion"` e um para `"error"` -- os dois possíveis desfechos *finais* da chamada `foo(..)`. Em essência, `foo(..)` sequer parece estar ciente de que o código chamador se inscreveu nesses eventos, o que proporciona uma bela *separação de responsabilidades*.

Infelizmente, tal código exigiria alguma "mágica" do ambiente JS que não existe (e provavelmente seria um pouco impraticável). Eis a forma mais natural com que poderíamos expressar isso em JS:

```js
function foo(x) {
	// começa a fazer algo que pode levar um tempo

	// cria uma capacidade de notificação de evento
	// `listener` para retornar

	return listener;
}

var evt = foo( 42 );

evt.on( "completion", function(){
	// agora podemos fazer o próximo passo!
} );

evt.on( "failure", function(err){
	// ops, algo deu errado em `foo(..)`
} );
```

`foo(..)` cria expressamente uma capacidade de inscrição em eventos para retornar de volta, e o código chamador recebe e registra os dois handlers de evento contra ela.

A inversão em relação ao código normal orientado a callbacks deveria ser óbvia, e ela é intencional. Em vez de passar os callbacks para `foo(..)`, ela retorna uma capacidade de evento que chamamos de `evt`, a qual recebe os callbacks.

Mas se você se lembra do Capítulo 2, os callbacks em si representam uma *inversão de controle*. Então inverter o padrão de callback é na verdade uma *inversão da inversão*, ou uma *desinversão de controle* -- restaurando o controle de volta para o código chamador, onde queríamos que ele estivesse desde o início.

Um benefício importante é que múltiplas partes separadas do código podem receber a capacidade de escuta de eventos, e todas podem ser independentemente notificadas de quando `foo(..)` é concluída, para realizar passos subsequentes após sua conclusão:

```js
var evt = foo( 42 );

// deixa `bar(..)` escutar a conclusão de `foo(..)`
bar( evt );

// também deixa `baz(..)` escutar a conclusão de `foo(..)`
baz( evt );
```

A *desinversão de controle* possibilita uma melhor *separação de responsabilidades*, em que `bar(..)` e `baz(..)` não precisam estar envolvidas em como `foo(..)` é chamada. De forma similar, `foo(..)` não precisa saber ou se importar que `bar(..)` e `baz(..)` existem ou estão esperando para ser notificadas quando `foo(..)` for concluída.

Em essência, esse objeto `evt` é uma negociação neutra de terceiros entre as responsabilidades separadas.

#### "Eventos" de Promise

Como você já deve ter adivinhado, a capacidade de escuta de eventos `evt` é uma analogia para uma Promise.

Em uma abordagem baseada em Promise, o trecho anterior teria `foo(..)` criando e retornando uma instância de `Promise`, e essa promise seria então passada para `bar(..)` e `baz(..)`.

**Nota:** Os "eventos" de resolução de Promise que escutamos não são estritamente eventos (embora certamente se comportem como eventos para esses propósitos), e eles tipicamente não são chamados de `"completion"` ou `"error"`. Em vez disso, usamos `then(..)` para registrar um evento `"then"`. Ou, mais precisamente, `then(..)` registra evento(s) de `"fulfillment"` (realização) e/ou `"rejection"` (rejeição), embora não vejamos esses termos usados explicitamente no código.

Considere:

```js
function foo(x) {
	// começa a fazer algo que pode levar um tempo

	// constrói e retorna uma promise
	return new Promise( function(resolve,reject){
		// eventualmente, chama `resolve(..)` ou `reject(..)`,
		// que são os callbacks de resolução para
		// a promise.
	} );
}

var p = foo( 42 );

bar( p );

baz( p );
```

**Nota:** O padrão mostrado com `new Promise( function(..){ .. } )` é geralmente chamado de ["construtor revelador" (revealing constructor)](http://domenic.me/2014/02/13/the-revealing-constructor-pattern/). A função passada é executada imediatamente (não deferida de forma assíncrona, como são os callbacks de `then(..)`), e ela recebe dois parâmetros, que neste caso nomeamos `resolve` e `reject`. Essas são as funções de resolução para a promise. `resolve(..)` geralmente sinaliza realização, e `reject(..)` sinaliza rejeição.

Você provavelmente consegue adivinhar como podem ser os detalhes internos de `bar(..)` e `baz(..)`:

```js
function bar(fooPromise) {
	// escuta `foo(..)` ser concluída
	fooPromise.then(
		function(){
			// `foo(..)` agora terminou, então
			// faz a tarefa de `bar(..)`
		},
		function(){
			// ops, algo deu errado em `foo(..)`
		}
	);
}

// idem para `baz(..)`
```

A resolução de Promise não necessariamente precisa envolver o envio de uma mensagem, como acontecia quando examinávamos Promises como *valores futuros*. Ela pode ser apenas um sinal de controle de fluxo, como usado no trecho anterior.

Outra forma de abordar isso é:

```js
function bar() {
	// `foo(..)` definitivamente terminou, então
	// faz a tarefa de `bar(..)`
}

function oopsBar() {
	// ops, algo deu errado em `foo(..)`,
	// então `bar(..)` não rodou
}

// idem para `baz()` e `oopsBaz()`

var p = foo( 42 );

p.then( bar, oopsBar );

p.then( baz, oopsBaz );
```

**Nota:** Se você já viu código baseado em Promise antes, pode se sentir tentado a acreditar que as duas últimas linhas desse código poderiam ser escritas como `p.then( .. ).then( .. )`, usando encadeamento, em vez de `p.then(..); p.then(..)`. Isso teria um comportamento inteiramente diferente, então tenha cuidado! A diferença pode não estar clara agora, mas é na verdade um padrão assíncrono diferente do que vimos até aqui: divisão/bifurcação (splitting/forking). Não se preocupe! Voltaremos a este ponto mais adiante neste capítulo.

Em vez de passar a promise `p` para `bar(..)` e `baz(..)`, usamos a promise para controlar quando `bar(..)` e `baz(..)` serão executadas, se é que serão. A diferença principal está no tratamento de erros.

Na abordagem do primeiro trecho, `bar(..)` é chamada independentemente de `foo(..)` ter sucesso ou falhar, e ela trata sua própria lógica de contingência se for notificada de que `foo(..)` falhou. O mesmo vale para `baz(..)`, obviamente.

No segundo trecho, `bar(..)` só é chamada se `foo(..)` tiver sucesso, e caso contrário `oopsBar(..)` é chamada. Idem para `baz(..)`.

Nenhuma das abordagens é *correta* em si. Haverá casos em que uma é preferível à outra.

Em qualquer caso, a promise `p` que retorna de `foo(..)` é usada para controlar o que acontece em seguida.

Além disso, o fato de ambos os trechos acabarem chamando `then(..)` duas vezes contra a mesma promise `p` ilustra o ponto feito anteriormente, que é o de que as Promises (uma vez resolvidas) mantêm a mesma resolução (realização ou rejeição) para sempre, e podem subsequentemente ser observadas tantas vezes quantas forem necessárias.

Sempre que `p` for resolvida, o próximo passo será sempre o mesmo, tanto *agora* quanto *depois*.

## Duck Typing de Thenable

Na terra das Promises, um detalhe importante é como saber com certeza se algum valor é uma Promise genuína ou não. Ou, mais diretamente, ele é um valor que se comportará como uma Promise?

Dado que as Promises são construídas pela sintaxe `new Promise(..)`, você pode pensar que `p instanceof Promise` seria uma verificação aceitável. Mas, infelizmente, há vários motivos pelos quais isso não é totalmente suficiente.

Principalmente, você pode receber um valor de Promise de outra janela do navegador (iframe, etc.), que teria sua própria Promise, diferente da que existe na janela/frame atual, e essa verificação falharia em identificar a instância de Promise.

Além disso, uma biblioteca ou framework pode escolher fornecer suas próprias Promises e não usar a implementação nativa de `Promise` do ES6 para isso. De fato, você pode muito bem estar usando Promises com bibliotecas em navegadores mais antigos que não têm Promise nenhuma.

Quando discutirmos os processos de resolução de Promise mais adiante neste capítulo, ficará mais óbvio por que um valor que-não-é-genuíno-mas-é-parecido-com-Promise ainda assim seria muito importante de ser reconhecido e assimilado. Mas, por ora, apenas acredite em mim quando digo que é uma peça crítica do quebra-cabeça.

Sendo assim, decidiu-se que a forma de reconhecer uma Promise (ou algo que se comporta como uma Promise) seria definir algo chamado de "thenable" como qualquer objeto ou função que tenha um método `then(..)` nele. Assume-se que qualquer valor desse tipo seja um thenable conforme a Promise.

O termo geral para "verificações de tipo" que fazem suposições sobre o "tipo" de um valor com base em seu formato (quais propriedades estão presentes) é chamado de "duck typing" -- "Se parece um pato, e grasna como um pato, deve ser um pato" (veja o título *Types & Grammar* desta série de livros). Então a verificação de duck typing para um thenable seria mais ou menos:

```js
if (
	p !== null &&
	(
		typeof p === "object" ||
		typeof p === "function"
	) &&
	typeof p.then === "function"
) {
	// assume que é um thenable!
}
else {
	// não é um thenable
}
```

Eca! Deixando de lado o fato de que essa lógica é um pouco feia de implementar em vários lugares, há algo mais profundo e mais preocupante acontecendo.

Se você tentar realizar uma Promise com qualquer valor de objeto/função que por acaso tenha uma função `then(..)` nele, mas você não pretendia que ele fosse tratado como uma Promise/thenable, você está sem sorte, porque ele será automaticamente reconhecido como thenable e tratado com regras especiais (veja mais adiante no capítulo).

Isso é verdade até mesmo se você não percebeu que o valor tem um `then(..)` nele. Por exemplo:

```js
var o = { then: function(){} };

// faz `v` ser `[[Prototype]]`-vinculado a `o`
var v = Object.create( o );

v.someStuff = "cool";
v.otherStuff = "not so cool";

v.hasOwnProperty( "then" );		// false
```

`v` não se parece em nada com uma Promise ou thenable. É apenas um objeto simples com algumas propriedades nele. Você provavelmente apenas pretende enviar esse valor adiante como qualquer outro objeto.

Mas, sem você saber, `v` também está `[[Prototype]]`-vinculado (veja o título *this & Object Prototypes* desta série de livros) a outro objeto `o`, que por acaso tem um `then(..)` nele. Então as verificações de duck typing de thenable vão pensar e assumir que `v` é um thenable. Ah, não.

Não precisa nem ser algo tão diretamente intencional quanto isso:

```js
Object.prototype.then = function(){};
Array.prototype.then = function(){};

var v1 = { hello: "world" };
var v2 = [ "Hello", "World" ];
```

Tanto `v1` quanto `v2` serão assumidos como thenables. Você não pode controlar ou prever se algum outro código acidentalmente ou maliciosamente adiciona `then(..)` a `Object.prototype`, `Array.prototype`, ou qualquer um dos outros protótipos nativos. E se o que for especificado é uma função que não chama nenhum de seus parâmetros como callbacks, então qualquer Promise resolvida com tal valor simplesmente ficará pendurada silenciosamente para sempre! Loucura.

Soa implausível ou improvável? Talvez.

Mas tenha em mente que havia várias bibliotecas conhecidas que não eram Promise, preexistentes na comunidade antes do ES6, que por acaso já tinham um método nelas chamado `then(..)`. Algumas dessas bibliotecas escolheram renomear seus próprios métodos para evitar colisão (que droga!). Outras simplesmente foram relegadas ao infeliz status de "incompatível com código baseado em Promise" como recompensa por sua incapacidade de mudar para sair do caminho.

A decisão dos padrões de sequestrar o nome de propriedade `then`, anteriormente não reservado -- e de som completamente genérico --, significa que nenhum valor (ou qualquer um de seus delegados), seja passado, presente ou futuro, pode ter uma função `then(..)` presente, seja de propósito ou por acidente, ou esse valor será confundido com um thenable em sistemas de Promises, o que provavelmente criará bugs que são realmente difíceis de rastrear.

**Aviso:** Eu não gosto de como acabamos com o duck typing de thenables para o reconhecimento de Promise. Havia outras opções, como "branding" (marcação) ou até "anti-branding"; o que conseguimos parece um compromisso de pior caso. Mas nem tudo é desgraça e melancolia. O duck typing de thenable pode ser útil, como veremos mais adiante. Apenas tome cuidado, porque o duck typing de thenable pode ser perigoso se identificar incorretamente algo como uma Promise quando não é.

## Confiança de Promise

Já vimos duas analogias fortes que explicam diferentes aspectos do que as Promises podem fazer pelo nosso código assíncrono. Mas se pararmos por aí, perdemos talvez a única característica mais importante que o padrão de Promise estabelece: a confiança.

Enquanto as analogias de *valores futuros* e *eventos de conclusão* se desenrolam explicitamente nos padrões de código que exploramos, não será inteiramente óbvio por que ou como as Promises são projetadas para resolver todas as questões de confiança de *inversão de controle* que apresentamos na seção "Questões de Confiança" do Capítulo 2. Mas com uma pequena escavação, podemos descobrir algumas garantias importantes que restauram a confiança no código assíncrono que o Capítulo 2 derrubou!

Vamos começar revisando as questões de confiança com a programação baseada apenas em callbacks. Quando você passa um callback para um utilitário `foo(..)`, ele pode:

* Chamar o callback cedo demais
* Chamar o callback tarde demais (ou nunca)
* Chamar o callback poucas vezes demais ou vezes demais
* Falhar em passar adiante qualquer ambiente/parâmetros necessários
* Engolir quaisquer erros/exceções que possam acontecer

As características das Promises são intencionalmente projetadas para fornecer respostas úteis e repetíveis a todas essas preocupações.

### Chamando Cedo Demais

Primariamente, esta é uma preocupação sobre se o código pode introduzir efeitos do tipo Zalgo (veja o Capítulo 2), em que às vezes uma tarefa termina de forma síncrona e às vezes de forma assíncrona, o que pode levar a condições de corrida (race conditions).

As Promises, por definição, não podem ser suscetíveis a essa preocupação, porque mesmo uma Promise imediatamente realizada (como `new Promise(function(resolve){ resolve(42); })`) não pode ser *observada* de forma síncrona.

Ou seja, quando você chama `then(..)` em uma Promise, mesmo que aquela Promise já estivesse resolvida, o callback que você fornece a `then(..)` será **sempre** chamado de forma assíncrona (para mais sobre isso, retorne a "Jobs" no Capítulo 1).

Não há mais necessidade de inserir seus próprios truques (hacks) de `setTimeout(..,0)`. As Promises previnem o Zalgo automaticamente.

### Chamando Tarde Demais

De forma similar ao ponto anterior, os callbacks de observação registrados pelo `then(..)` de uma Promise são automaticamente agendados quando `resolve(..)` ou `reject(..)` são chamados pela capacidade de criação da Promise. Esses callbacks agendados serão previsivelmente disparados no próximo momento assíncrono (veja "Jobs" no Capítulo 1).

Não é possível observação síncrona, então não é possível que uma cadeia síncrona de tarefas execute de tal forma a, na prática, "atrasar" outro callback de acontecer como esperado. Ou seja, quando uma Promise é resolvida, todos os callbacks registrados via `then(..)` nela serão chamados, em ordem, imediatamente na próxima oportunidade assíncrona (novamente, veja "Jobs" no Capítulo 1), e nada que aconteça dentro de um desses callbacks pode afetar/atrasar a chamada dos outros callbacks.

Por exemplo:

```js
p.then( function(){
	p.then( function(){
		console.log( "C" );
	} );
	console.log( "A" );
} );
p.then( function(){
	console.log( "B" );
} );
// A B C
```

Aqui, `"C"` não pode interromper e preceder `"B"`, em virtude de como as Promises são definidas para operar.

#### Peculiaridades de Agendamento de Promise

É importante notar, porém, que há muitas nuances de agendamento em que a ordenação relativa entre callbacks encadeados a partir de duas Promises separadas não é previsível de forma confiável.

Se duas promises `p1` e `p2` já estão ambas resolvidas, deveria ser verdade que `p1.then(..); p2.then(..)` acabaria chamando o(s) callback(s) de `p1` antes do(s) de `p2`. Mas há casos sutis em que isso pode não ser verdade, como o seguinte:

```js
var p3 = new Promise( function(resolve,reject){
	resolve( "B" );
} );

var p1 = new Promise( function(resolve,reject){
	resolve( p3 );
} );

var p2 = new Promise( function(resolve,reject){
	resolve( "A" );
} );

p1.then( function(v){
	console.log( v );
} );

p2.then( function(v){
	console.log( v );
} );

// A B  <-- não  B A  como você poderia esperar
```

Cobriremos isso mais adiante, mas, como você pode ver, `p1` é resolvida não com um valor imediato, mas com outra promise `p3`, que por sua vez é resolvida com o valor `"B"`. O comportamento especificado é *desembrulhar* (unwrap) `p3` em `p1`, mas de forma assíncrona, de modo que o(s) callback(s) de `p1` ficam *atrás* do(s) callback(s) de `p2` na fila de Jobs assíncrona (veja o Capítulo 1).

Para evitar tais pesadelos cheios de nuances, você nunca deveria depender de nada relativo à ordenação/agendamento de callbacks entre Promises. De fato, uma boa prática é não programar de uma forma em que a ordenação de múltiplos callbacks importe de jeito nenhum. Evite isso se puder.

### Nunca Chamando o Callback

Esta é uma preocupação muito comum. Ela é tratável de várias formas com Promises.

Primeiro, nada (nem mesmo um erro de JS) pode impedir uma Promise de notificá-lo de sua resolução (se ela for resolvida). Se você registrar tanto callbacks de realização quanto de rejeição para uma Promise, e a Promise for resolvida, um dos dois callbacks sempre será chamado.

É claro que, se seus próprios callbacks tiverem erros de JS, você pode não ver o resultado que espera, mas o callback de fato terá sido chamado. Cobriremos mais adiante como ser notificado de um erro no seu callback, porque mesmo esses não são engolidos.

Mas e se a própria Promise nunca for resolvida de jeito nenhum? Até para isso as Promises fornecem uma resposta, usando uma abstração de nível mais alto chamada "race" (corrida):

```js
// um utilitário para dar timeout em uma Promise
function timeoutPromise(delay) {
	return new Promise( function(resolve,reject){
		setTimeout( function(){
			reject( "Timeout!" );
		}, delay );
	} );
}

// configura um timeout para `foo()`
Promise.race( [
	foo(),					// tenta `foo()`
	timeoutPromise( 3000 )	// dá 3 segundos a ela
] )
.then(
	function(){
		// `foo(..)` foi realizada a tempo!
	},
	function(err){
		// ou `foo()` foi rejeitada, ou apenas
		// não terminou a tempo, então inspecione
		// `err` para saber qual foi o caso
	}
);
```

Há mais detalhes a considerar com esse padrão de timeout de Promise, mas voltaremos a ele mais adiante.

O importante é que podemos garantir um sinal sobre o resultado de `foo()`, para impedir que ela deixe nosso programa pendurado indefinidamente.

### Chamando Poucas Vezes ou Vezes Demais

Por definição, *uma* é a quantidade apropriada de vezes para o callback ser chamado. O caso de "poucas vezes" seria zero chamadas, que é o mesmo que o caso de "nunca" que acabamos de examinar.

O caso de "vezes demais" é fácil de explicar. As Promises são definidas de modo que só podem ser resolvidas uma vez. Se por algum motivo o código de criação da Promise tentar chamar `resolve(..)` ou `reject(..)` múltiplas vezes, ou tentar chamar ambos, a Promise aceitará apenas a primeira resolução, e ignorará silenciosamente quaisquer tentativas subsequentes.

Como uma Promise só pode ser resolvida uma vez, quaisquer callbacks registrados via `then(..)` só serão chamados uma vez (cada).

É claro que, se você registrar o mesmo callback mais de uma vez (por exemplo, `p.then(f); p.then(f);`), ele será chamado tantas vezes quantas foi registrado. A garantia de que uma função de resposta é chamada apenas uma vez não impede você de dar um tiro no próprio pé.

### Falhando em Passar Adiante Quaisquer Parâmetros/Ambiente

As Promises podem ter, no máximo, um valor de resolução (realização ou rejeição).

Se você não resolver explicitamente com um valor de uma forma ou de outra, o valor é `undefined`, como é típico em JS. Mas qualquer que seja o valor, ele sempre será passado a todos os callbacks registrados (e apropriados: realização ou rejeição), seja *agora* ou no futuro.

Algo a se estar ciente: se você chamar `resolve(..)` ou `reject(..)` com múltiplos parâmetros, todos os parâmetros subsequentes além do primeiro serão silenciosamente ignorados. Embora isso possa parecer uma violação da garantia que acabamos de descrever, não é exatamente, porque constitui um uso inválido do mecanismo de Promise. Outros usos inválidos da API (como chamar `resolve(..)` múltiplas vezes) são igualmente *protegidos*, então o comportamento da Promise aqui é consistente (mesmo que um pouquinho frustrante).

Se você quer passar adiante múltiplos valores, você deve envolvê-los em outro valor único que você passa, como um `array` ou um `object`.

Quanto ao ambiente, funções em JS sempre retêm seu closure do escopo no qual são definidas (veja o título *Scope & Closures* desta série), então elas obviamente continuariam tendo acesso a qualquer estado circundante que você forneça. É claro que o mesmo é verdade no design baseado apenas em callbacks, então isso não é uma melhoria de benefício específica das Promises -- mas é uma garantia com a qual podemos contar mesmo assim.

### Engolindo Quaisquer Erros/Exceções

No sentido básico, isto é uma reafirmação do ponto anterior. Se você rejeita uma Promise com um *motivo* (ou seja, mensagem de erro), esse valor é passado ao(s) callback(s) de rejeição.

Mas há algo muito maior em jogo aqui. Se em qualquer ponto na criação de uma Promise, ou na observação de sua resolução, um erro de exceção de JS ocorrer, como um `TypeError` ou `ReferenceError`, essa exceção será capturada, e forçará a Promise em questão a se tornar rejeitada.

Por exemplo:

```js
var p = new Promise( function(resolve,reject){
	foo.bar();	// `foo` não está definido, então erro!
	resolve( 42 );	// nunca chega aqui :(
} );

p.then(
	function fulfilled(){
		// nunca chega aqui :(
	},
	function rejected(err){
		// `err` será um objeto de exceção `TypeError`
		// da linha `foo.bar()`.
	}
);
```

A exceção de JS que ocorre a partir de `foo.bar()` se torna uma rejeição de Promise que você pode capturar e responder.

Isto é um detalhe importante, porque ele efetivamente resolve outro potencial momento Zalgo, que é o de que erros poderiam criar uma reação síncrona enquanto não-erros seriam assíncronos. As Promises transformam até exceções de JS em comportamento assíncrono, reduzindo assim grandemente as chances de condição de corrida.

Mas o que acontece se uma Promise é realizada, mas há um erro de exceção de JS durante a observação (em um callback registrado via `then(..)`)? Mesmo esses não são perdidos, mas você pode achar a forma como eles são tratados um pouco surpreendente, até se aprofundar um pouco mais:

```js
var p = new Promise( function(resolve,reject){
	resolve( 42 );
} );

p.then(
	function fulfilled(msg){
		foo.bar();
		console.log( msg );	// nunca chega aqui :(
	},
	function rejected(err){
		// nunca chega aqui tampouco :(
	}
);
```

Espera, isso faz parecer que a exceção de `foo.bar()` realmente foi engolida. Não tema, ela não foi. Mas algo mais profundo está errado, que é o de que falhamos em escutá-la. A própria chamada `p.then(..)` retorna outra promise, e é *essa* promise que será rejeitada com a exceção `TypeError`.

Por que ela simplesmente não pôde chamar o handler de erro que temos definido ali? Parece um comportamento lógico à primeira vista. Mas isso violaria o princípio fundamental de que as Promises são **imutáveis** uma vez resolvidas. `p` já tinha sido realizada com o valor `42`, então ela não pode mais tarde ser mudada para uma rejeição só porque há um erro na observação da resolução de `p`.

Além da violação do princípio, tal comportamento poderia causar estragos, se, digamos, houvesse múltiplos callbacks registrados via `then(..)` na promise `p`, porque alguns seriam chamados e outros não, e seria muito opaco entender o porquê.

### Promise Confiável?

Há um último detalhe a examinar para estabelecer a confiança baseada no padrão de Promise.

Você sem dúvida notou que as Promises não se livram dos callbacks de jeito nenhum. Elas apenas mudam para onde o callback é passado. Em vez de passar um callback para `foo(..)`, recebemos *algo* (ostensivamente uma Promise genuína) de volta de `foo(..)`, e passamos o callback para esse *algo*.

Mas por que isso seria mais confiável do que apenas callbacks sozinhos? Como podemos ter certeza de que o *algo* que recebemos de volta é de fato uma Promise confiável? Isso não é basicamente um castelo de cartas em que só podemos confiar porque já confiávamos?

Um dos detalhes mais importantes, mas frequentemente negligenciados, das Promises é que elas têm uma solução para essa questão também. Incluído na implementação nativa de `Promise` do ES6 está `Promise.resolve(..)`.

Se você passar um valor imediato, não-Promise, não-thenable, para `Promise.resolve(..)`, você recebe uma promise que é realizada com esse valor. Em outras palavras, estas duas promises `p1` e `p2` vão se comportar basicamente de forma idêntica:

```js
var p1 = new Promise( function(resolve,reject){
	resolve( 42 );
} );

var p2 = Promise.resolve( 42 );
```

Mas se você passar uma Promise genuína para `Promise.resolve(..)`, você simplesmente recebe a mesma promise de volta:

```js
var p1 = Promise.resolve( 42 );

var p2 = Promise.resolve( p1 );

p1 === p2; // true
```

Ainda mais importante, se você passar um valor thenable que não é Promise para `Promise.resolve(..)`, ela tentará desembrulhar esse valor, e o desembrulhamento continuará até que um valor final concreto não-parecido-com-Promise seja extraído.

Lembra da nossa discussão anterior sobre thenables?

Considere:

```js
var p = {
	then: function(cb) {
		cb( 42 );
	}
};

// isso funciona OK, mas só por sorte
p
.then(
	function fulfilled(val){
		console.log( val ); // 42
	},
	function rejected(err){
		// nunca chega aqui
	}
);
```

Este `p` é um thenable, mas não é uma Promise genuína. Por sorte, ele é razoável, como a maioria será. Mas e se você recebesse de volta, em vez disso, algo que se parecesse com:

```js
var p = {
	then: function(cb,errcb) {
		cb( 42 );
		errcb( "evil laugh" );
	}
};

p
.then(
	function fulfilled(val){
		console.log( val ); // 42
	},
	function rejected(err){
		// ops, não deveria ter rodado
		console.log( err ); // evil laugh
	}
);
```

Este `p` é um thenable, mas não é uma promise tão bem comportada. Ele é malicioso? Ou apenas ignorante de como as Promises deveriam funcionar? Não importa muito, para ser honesto. Em qualquer caso, ele não é confiável como está.

Mesmo assim, podemos passar qualquer uma dessas versões de `p` para `Promise.resolve(..)`, e receberemos o resultado normalizado e seguro que esperaríamos:

```js
Promise.resolve( p )
.then(
	function fulfilled(val){
		console.log( val ); // 42
	},
	function rejected(err){
		// nunca chega aqui
	}
);
```

`Promise.resolve(..)` aceitará qualquer thenable, e o desembrulhará até seu valor não-thenable. Mas você recebe de volta de `Promise.resolve(..)` uma Promise real e genuína em seu lugar, **uma na qual você pode confiar**. Se o que você passou já é uma Promise genuína, você simplesmente a recebe de volta, então não há desvantagem nenhuma em filtrar através de `Promise.resolve(..)` para ganhar confiança.

Então digamos que estamos chamando um utilitário `foo(..)` e não temos certeza se podemos confiar que seu valor de retorno seja uma Promise bem comportada, mas sabemos que ele é pelo menos um thenable. `Promise.resolve(..)` nos dará um invólucro de Promise confiável para encadear:

```js
// não faça apenas isto:
foo( 42 )
.then( function(v){
	console.log( v );
} );

// em vez disso, faça isto:
Promise.resolve( foo( 42 ) )
.then( function(v){
	console.log( v );
} );
```

**Nota:** Outro efeito colateral benéfico de envolver `Promise.resolve(..)` em torno do valor de retorno de qualquer função (thenable ou não) é que é uma forma fácil de normalizar essa chamada de função em uma tarefa assíncrona bem comportada. Se `foo(42)` retorna um valor imediato às vezes, ou uma Promise outras vezes, `Promise.resolve( foo(42) )` garante que será sempre um resultado de Promise. E evitar o Zalgo gera um código muito melhor.

### Confiança Construída

Espero que a discussão anterior agora "resolva" (trocadilho intencional) plenamente em sua mente por que a Promise é confiável e, mais importante, por que essa confiança é tão crítica para construir software robusto e mantenível.

Você consegue escrever código assíncrono em JS sem confiança? É claro que consegue. Nós, desenvolvedores JS, temos programado de forma assíncrona com nada além de callbacks por quase duas décadas.

Mas, uma vez que você começa a questionar o quanto pode confiar nos mecanismos sobre os quais você constrói para serem de fato previsíveis e confiáveis, você começa a perceber que os callbacks têm uma fundação de confiança bem instável.

As Promises são um padrão que amplia os callbacks com semântica confiável, de modo que o comportamento seja mais razoável e mais confiável. Ao desinverter a *inversão de controle* dos callbacks, colocamos o controle em um sistema confiável (Promises) que foi projetado especificamente para trazer sanidade à nossa assincronia.

## Fluxo Encadeado

Já demos a entender isso algumas vezes, mas as Promises não são apenas um mecanismo para uma operação do tipo *isto-então-aquilo* de um único passo. Esse é o bloco de construção, é claro, mas acontece que podemos amarrar múltiplas Promises juntas para representar uma sequência de passos assíncronos.

A chave para fazer isso funcionar é construída sobre dois comportamentos intrínsecos das Promises:

* Toda vez que você chama `then(..)` em uma Promise, ela cria e retorna uma nova Promise, com a qual podemos *encadear* (chain).
* Qualquer valor que você retorna do callback de realização da chamada `then(..)` (o primeiro parâmetro) é automaticamente definido como a realização da Promise *encadeada* (do primeiro ponto).

Vamos primeiro ilustrar o que isso significa, e *então* derivaremos como isso nos ajuda a criar sequências assíncronas de controle de fluxo. Considere o seguinte:

```js
var p = Promise.resolve( 21 );

var p2 = p.then( function(v){
	console.log( v );	// 21

	// realiza `p2` com o valor `42`
	return v * 2;
} );

// encadeia a partir de `p2`
p2.then( function(v){
	console.log( v );	// 42
} );
```

Ao retornar `v * 2` (ou seja, `42`), realizamos a promise `p2` que a primeira chamada `then(..)` criou e retornou. Quando a chamada `then(..)` de `p2` roda, ela está recebendo a realização da instrução `return v * 2`. É claro que `p2.then(..)` cria ainda outra promise, que poderíamos ter armazenado em uma variável `p3`.

Mas é um pouco irritante ter que criar uma variável intermediária `p2` (ou `p3`, etc.). Felizmente, podemos facilmente encadeá-las todas juntas:

```js
var p = Promise.resolve( 21 );

p
.then( function(v){
	console.log( v );	// 21

	// realiza a promise encadeada com o valor `42`
	return v * 2;
} )
// aqui está a promise encadeada
.then( function(v){
	console.log( v );	// 42
} );
```

Então agora o primeiro `then(..)` é o primeiro passo em uma sequência assíncrona, e o segundo `then(..)` é o segundo passo. Isso poderia continuar por quanto tempo você precisasse estender. Apenas continue encadeando a partir de um `then(..)` anterior com cada Promise criada automaticamente.

Mas há algo faltando aqui. E se quisermos que o passo 2 espere o passo 1 fazer algo assíncrono? Estamos usando uma instrução `return` imediata, que imediatamente realiza a promise encadeada.

A chave para tornar uma sequência de Promise verdadeiramente capaz de ser assíncrona em cada passo é lembrar como `Promise.resolve(..)` opera quando o que você passa a ela é uma Promise ou thenable em vez de um valor final. `Promise.resolve(..)` retorna diretamente uma Promise genuína recebida, ou desembrulha o valor de um thenable recebido -- e continua recursivamente enquanto continua desembrulhando thenables.

O mesmo tipo de desembrulhamento acontece se você fizer `return` de um thenable ou Promise a partir do handler de realização (ou rejeição). Considere:

```js
var p = Promise.resolve( 21 );

p.then( function(v){
	console.log( v );	// 21

	// cria uma promise e a retorna
	return new Promise( function(resolve,reject){
		// realiza com o valor `42`
		resolve( v * 2 );
	} );
} )
.then( function(v){
	console.log( v );	// 42
} );
```

Mesmo tendo envolvido `42` em uma promise que retornamos, ele ainda foi desembrulhado e acabou como a resolução da promise encadeada, de modo que o segundo `then(..)` ainda recebeu `42`. Se introduzirmos assincronia àquela promise de envolvimento, tudo ainda funciona da mesma forma de maneira agradável:

```js
var p = Promise.resolve( 21 );

p.then( function(v){
	console.log( v );	// 21

	// cria uma promise para retornar
	return new Promise( function(resolve,reject){
		// introduz assincronia!
		setTimeout( function(){
			// realiza com o valor `42`
			resolve( v * 2 );
		}, 100 );
	} );
} )
.then( function(v){
	// roda após o atraso de 100ms no passo anterior
	console.log( v );	// 42
} );
```

Isso é incrivelmente poderoso! Agora podemos construir uma sequência de quantos passos assíncronos quisermos, e cada passo pode atrasar o próximo passo (ou não!), conforme necessário.

É claro que a passagem de valores de passo para passo nesses exemplos é opcional. Se você não retorna um valor explícito, um `undefined` implícito é assumido, e as promises ainda se encadeiam da mesma forma. Cada resolução de Promise é, assim, apenas um sinal para prosseguir ao próximo passo.

Para aprofundar a ilustração de encadeamento, vamos generalizar a criação de uma Promise-de-atraso (sem mensagens de resolução) em um utilitário que possamos reutilizar para múltiplos passos:

```js
function delay(time) {
	return new Promise( function(resolve,reject){
		setTimeout( resolve, time );
	} );
}

delay( 100 ) // passo 1
.then( function STEP2(){
	console.log( "passo 2 (após 100ms)" );
	return delay( 200 );
} )
.then( function STEP3(){
	console.log( "passo 3 (após mais 200ms)" );
} )
.then( function STEP4(){
	console.log( "passo 4 (próximo Job)" );
	return delay( 50 );
} )
.then( function STEP5(){
	console.log( "passo 5 (após mais 50ms)" );
} )
...
```

Chamar `delay(200)` cria uma promise que será realizada em 200ms, e então retornamos isso do primeiro callback de realização do `then(..)`, o que faz com que a promise do segundo `then(..)` espere por aquela promise de 200ms.

**Nota:** Como descrito, tecnicamente há duas promises nessa troca: a promise de atraso de 200ms e a promise encadeada da qual o segundo `then(..)` encadeia. Mas você pode achar mais fácil combinar mentalmente essas duas promises juntas, porque o mecanismo de Promise automaticamente mescla seus estados para você. Nesse aspecto, você poderia pensar em `return delay(200)` como a criação de uma promise que substitui a promise encadeada retornada anteriormente.

Para ser honesto, porém, sequências de atrasos sem passagem de mensagens não é um exemplo terrivelmente útil de controle de fluxo com Promise. Vamos olhar para um cenário que é um pouco mais prático.

Em vez de timers, vamos considerar fazer requisições Ajax:

```js
// assuma um utilitário `ajax( {url}, {callback} )`

// ajax ciente de Promise
function request(url) {
	return new Promise( function(resolve,reject){
		// o callback de `ajax(..)` deveria ser a função
		// `resolve(..)` da nossa promise
		ajax( url, resolve );
	} );
}
```

Primeiro definimos um utilitário `request(..)` que constrói uma promise para representar a conclusão da chamada `ajax(..)`:

```js
request( "http://some.url.1/" )
.then( function(response1){
	return request( "http://some.url.2/?v=" + response1 );
} )
.then( function(response2){
	console.log( response2 );
} );
```

**Nota:** Desenvolvedores comumente encontram situações em que querem fazer controle de fluxo assíncrono ciente de Promise com utilitários que não são, eles próprios, habilitados para Promise (como `ajax(..)` aqui, que espera um callback). Embora o mecanismo nativo de `Promise` do ES6 não resolva automaticamente esse padrão para nós, praticamente todas as bibliotecas de Promise *resolvem*. Elas geralmente chamam esse processo de "lifting" (elevação) ou "promisifying" (promisificação), ou alguma variação disso. Voltaremos a essa técnica mais adiante.

Usando o `request(..)` que retorna Promise, criamos o primeiro passo em nossa cadeia implicitamente chamando-o com a primeira URL, e encadeamos a partir daquela promise retornada com o primeiro `then(..)`.

Uma vez que `response1` volta, usamos esse valor para construir uma segunda URL, e fazemos uma segunda chamada `request(..)`. A promise daquele segundo `request(..)` é retornada (`return`) para que o terceiro passo em nosso controle de fluxo assíncrono espere aquela chamada Ajax ser concluída. Por fim, imprimimos `response2` assim que ele volta.

A cadeia de Promise que construímos não é apenas um controle de fluxo que expressa uma sequência assíncrona de múltiplos passos, mas também atua como um canal de mensagens para propagar mensagens de passo para passo.

E se algo desse errado em um dos passos da cadeia de Promise? Um erro/exceção é por Promise, o que significa que é possível capturar tal erro em qualquer ponto na cadeia, e essa captura age de certa forma para "reiniciar" a cadeia de volta à operação normal naquele ponto:

```js
// passo 1:
request( "http://some.url.1/" )

// passo 2:
.then( function(response1){
	foo.bar(); // undefined, erro!

	// nunca chega aqui
	return request( "http://some.url.2/?v=" + response1 );
} )

// passo 3:
.then(
	function fulfilled(response2){
		// nunca chega aqui
	},
	// handler de rejeição para capturar o erro
	function rejected(err){
		console.log( err );	// `TypeError` do erro de `foo.bar()`
		return 42;
	}
)

// passo 4:
.then( function(msg){
	console.log( msg );		// 42
} );
```

Quando o erro ocorre no passo 2, o handler de rejeição no passo 3 o captura. O valor de retorno (`42` neste trecho), se houver, daquele handler de rejeição realiza a promise para o próximo passo (4), de modo que a cadeia está agora de volta a um estado de realização.

**Nota:** Como discutimos anteriormente, ao retornar uma promise de um handler de realização, ela é desembrulhada e pode atrasar o próximo passo. Isso também é verdade ao retornar promises de handlers de rejeição, de modo que, se o `return 42` no passo 3 em vez disso retornasse uma promise, essa promise poderia atrasar o passo 4. Uma exceção lançada dentro do handler de realização ou de rejeição de uma chamada `then(..)` faz com que a próxima promise (encadeada) seja imediatamente rejeitada com aquela exceção.

Se você chama `then(..)` em uma promise, e passa apenas um handler de realização a ele, um handler de rejeição assumido é substituído:

```js
var p = new Promise( function(resolve,reject){
	reject( "Oops" );
} );

var p2 = p.then(
	function fulfilled(){
		// nunca chega aqui
	}
	// handler de rejeição assumido, se omitido ou
	// qualquer outro valor que não seja função for passado
	// function(err) {
	//     throw err;
	// }
);
```

Como você pode ver, o handler de rejeição assumido simplesmente relança o erro, o que acaba forçando `p2` (a promise encadeada) a rejeitar com o mesmo motivo de erro. Em essência, isso permite que o erro continue se propagando ao longo de uma cadeia de Promise até que um handler de rejeição explicitamente definido seja encontrado.

**Nota:** Cobriremos mais detalhes do tratamento de erros com Promises um pouco mais adiante, porque há outros detalhes cheios de nuances com os quais devemos nos preocupar.

Se uma função válida apropriada não é passada como o parâmetro de handler de realização para `then(..)`, há também um handler padrão substituído:

```js
var p = Promise.resolve( 42 );

p.then(
	// handler de realização assumido, se omitido ou
	// qualquer outro valor que não seja função for passado
	// function(v) {
	//     return v;
	// }
	null,
	function rejected(err){
		// nunca chega aqui
	}
);
```

Como você pode ver, o handler de realização padrão simplesmente passa adiante qualquer valor que recebe para o próximo passo (Promise).

**Nota:** O padrão `then(null,function(err){ .. })` -- tratando apenas rejeições (se houver), mas deixando as realizações passarem -- tem um atalho na API: `catch(function(err){ .. })`. Cobriremos `catch(..)` mais completamente na próxima seção.

Vamos revisar brevemente os comportamentos intrínsecos das Promises que possibilitam o controle de fluxo encadeado:

* Uma chamada `then(..)` contra uma Promise automaticamente produz uma nova Promise para retornar da chamada.
* Dentro dos handlers de realização/rejeição, se você retorna um valor ou uma exceção é lançada, a nova Promise retornada (encadeável) é resolvida de acordo.
* Se o handler de realização ou rejeição retorna uma Promise, ela é desembrulhada, de modo que qualquer que seja sua resolução, ela se tornará a resolução da Promise encadeada retornada do `then(..)` atual.

Embora o controle de fluxo encadeado seja útil, é provavelmente mais preciso pensar nele como um benefício colateral de como as Promises compõem (combinam) juntas, em vez da intenção principal. Como já discutimos em detalhe várias vezes, as Promises normalizam a assincronia e encapsulam o estado de valor dependente do tempo, e *é isso* que nos permite encadeá-las juntas dessa forma útil.

Certamente, a expressividade sequencial da cadeia (isto-então-isto-então-isto...) é uma grande melhoria em relação à bagunça emaranhada de callbacks que identificamos no Capítulo 2. Mas ainda há uma boa quantidade de código repetitivo (boilerplate) (`then(..)` e `function(){ .. }`) para atravessar. No próximo capítulo, veremos um padrão significativamente mais agradável para a expressividade de controle de fluxo sequencial, com geradores.

### Terminologia: Resolve, Fulfill e Reject

Há uma leve confusão em torno dos termos "resolve", "fulfill" e "reject" que precisamos esclarecer antes de você se aprofundar demais no aprendizado sobre Promises. Vamos primeiro considerar o construtor `Promise(..)`:

```js
var p = new Promise( function(X,Y){
	// X() para realização
	// Y() para rejeição
} );
```

Como você pode ver, dois callbacks (aqui rotulados `X` e `Y`) são fornecidos. O primeiro é *geralmente* usado para marcar a Promise como realizada, e o segundo *sempre* marca a Promise como rejeitada. Mas o que significa esse "geralmente", e o que isso implica sobre nomear esses parâmetros com precisão?

Em última análise, é apenas seu código de usuário e os nomes dos identificadores não são interpretados pelo motor como significando algo, então não *importa* tecnicamente; `foo(..)` e `bar(..)` são igualmente funcionais. Mas as palavras que você usa podem afetar não só como você está pensando sobre o código, mas como outros desenvolvedores da sua equipe vão pensar sobre ele. Pensar erroneamente sobre código assíncrono cuidadosamente orquestrado é quase certamente pior do que as alternativas espaguete-de-callback.

Então, na verdade, importa, sim, como você os chama.

O segundo parâmetro é fácil de decidir. Quase toda a literatura usa `reject(..)` como seu nome, e como é exatamente (e apenas!) o que ele faz, essa é uma escolha muito boa para o nome. Eu recomendaria fortemente que você sempre usasse `reject(..)`.

Mas há um pouco mais de ambiguidade em torno do primeiro parâmetro, que na literatura de Promise é frequentemente rotulado `resolve(..)`. Essa palavra está obviamente relacionada a "resolution" (resolução), que é o que é usado em toda a literatura (incluindo este livro) para descrever a definição de um valor/estado final para uma Promise. Já usamos "resolver a Promise" várias vezes para significar tanto realizar quanto rejeitar a Promise.

Mas se esse parâmetro parece ser usado para especificamente realizar a Promise, por que não deveríamos chamá-lo de `fulfill(..)` em vez de `resolve(..)` para ser mais preciso? Para responder a essa pergunta, vamos também dar uma olhada em dois dos métodos da API `Promise`:

```js
var fulfilledPr = Promise.resolve( 42 );

var rejectedPr = Promise.reject( "Oops" );
```

`Promise.resolve(..)` cria uma Promise que é resolvida para o valor dado a ela. Neste exemplo, `42` é um valor normal, não-Promise, não-thenable, então a promise realizada `fulfilledPr` é criada para o valor `42`. `Promise.reject("Oops")` cria a promise rejeitada `rejectedPr` para o motivo `"Oops"`.

Vamos agora ilustrar por que a palavra "resolve" (como em `Promise.resolve(..)`) é não ambígua e de fato mais precisa, se usada explicitamente em um contexto que poderia resultar tanto em realização quanto em rejeição:

```js
var rejectedTh = {
	then: function(resolved,rejected) {
		rejected( "Oops" );
	}
};

var rejectedPr = Promise.resolve( rejectedTh );
```

Como discutimos anteriormente neste capítulo, `Promise.resolve(..)` retornará diretamente uma Promise genuína recebida, ou desembrulhará um thenable recebido. Se aquele desembrulhamento de thenable revelar um estado rejeitado, a Promise retornada de `Promise.resolve(..)` está de fato naquele mesmo estado rejeitado.

Então `Promise.resolve(..)` é um nome bom e preciso para o método da API, porque ele pode na verdade resultar tanto em realização quanto em rejeição.

O primeiro parâmetro de callback do construtor `Promise(..)` desembrulhará tanto um thenable (identicamente a `Promise.resolve(..)`) quanto uma Promise genuína:

```js
var rejectedPr = new Promise( function(resolve,reject){
	// resolve esta promise com uma promise rejeitada
	resolve( Promise.reject( "Oops" ) );
} );

rejectedPr.then(
	function fulfilled(){
		// nunca chega aqui
	},
	function rejected(err){
		console.log( err );	// "Oops"
	}
);
```

Deveria estar claro agora que `resolve(..)` é o nome apropriado para o primeiro parâmetro de callback do construtor `Promise(..)`.

**Aviso:** O `reject(..)` mencionado anteriormente **não** faz o desembrulhamento que `resolve(..)` faz. Se você passar um valor de Promise/thenable para `reject(..)`, esse valor intocado será definido como o motivo da rejeição. Um handler de rejeição subsequente receberia a própria Promise/thenable que você passou para `reject(..)`, e não seu valor imediato subjacente.

Mas agora vamos voltar nossa atenção para os callbacks fornecidos a `then(..)`. Como eles deveriam ser chamados (tanto na literatura quanto no código)? Eu sugeriria `fulfilled(..)` e `rejected(..)`:

```js
function fulfilled(msg) {
	console.log( msg );
}

function rejected(err) {
	console.error( err );
}

p.then(
	fulfilled,
	rejected
);
```

No caso do primeiro parâmetro de `then(..)`, é não ambiguamente sempre o caso de realização, então não há necessidade da dualidade da terminologia "resolve". Como nota lateral, a especificação do ES6 usa `onFulfilled(..)` e `onRejected(..)` para rotular esses dois callbacks, então eles são termos precisos.

## Tratamento de Erros

Já vimos vários exemplos de como a rejeição de Promise -- seja intencional, por meio da chamada `reject(..)`, ou acidental, por meio de exceções de JS -- permite um tratamento de erros mais sensato na programação assíncrona. Vamos voltar, porém, e ser explícitos sobre alguns dos detalhes que passamos por cima.

A forma mais natural de tratamento de erros para a maioria dos desenvolvedores é a construção síncrona `try..catch`. Infelizmente, ela é apenas síncrona, então ela falha em ajudar em padrões de código assíncrono:

```js
function foo() {
	setTimeout( function(){
		baz.bar();
	}, 100 );
}

try {
	foo();
	// mais tarde lança erro global de `baz.bar()`
}
catch (err) {
	// nunca chega aqui
}
```

`try..catch` certamente seria bom de se ter, mas ele não funciona através de operações assíncronas. Ou seja, a não ser que haja algum suporte adicional do ambiente, ao qual voltaremos com geradores no Capítulo 4.

Em callbacks, alguns padrões emergiram para o tratamento de erros padronizado, mais notavelmente o estilo "callback com erro primeiro" (error-first callback):

```js
function foo(cb) {
	setTimeout( function(){
		try {
			var x = baz.bar();
			cb( null, x ); // sucesso!
		}
		catch (err) {
			cb( err );
		}
	}, 100 );
}

foo( function(err,val){
	if (err) {
		console.error( err ); // que pena :(
	}
	else {
		console.log( val );
	}
} );
```

**Nota:** O `try..catch` aqui funciona apenas sob a perspectiva de que a chamada `baz.bar()` ou terá sucesso ou falhará imediatamente, de forma síncrona. Se `baz.bar()` fosse, ela mesma, sua própria função de conclusão assíncrona, quaisquer erros assíncronos dentro dela não seriam capturáveis.

O callback que passamos para `foo(..)` espera receber um sinal de um erro pelo primeiro parâmetro reservado `err`. Se presente, assume-se erro. Se não, assume-se sucesso.

Esse tipo de tratamento de erros é tecnicamente *capaz de ser assíncrono*, mas não compõe bem de jeito nenhum. Múltiplos níveis de callbacks com erro primeiro tecidos juntos com essas onipresentes verificações de instrução `if` inevitavelmente vão te levar aos perigos do inferno dos callbacks (veja o Capítulo 2).

Então voltamos ao tratamento de erros em Promises, com o handler de rejeição passado para `then(..)`. As Promises não usam o popular estilo de design "callback com erro primeiro", mas em vez disso usam o estilo "callbacks divididos" (split callbacks); há um callback para realização e um para rejeição:

```js
var p = Promise.reject( "Oops" );

p.then(
	function fulfilled(){
		// nunca chega aqui
	},
	function rejected(err){
		console.log( err ); // "Oops"
	}
);
```

Embora esse padrão de tratamento de erros faça todo o sentido à primeira vista, as nuances do tratamento de erros de Promise são frequentemente bem mais difíceis de captar plenamente.

Considere:

```js
var p = Promise.resolve( 42 );

p.then(
	function fulfilled(msg){
		// números não têm funções de string,
		// então isso vai lançar um erro
		console.log( msg.toLowerCase() );
	},
	function rejected(err){
		// nunca chega aqui
	}
);
```

Se `msg.toLowerCase()` legitimamente lança um erro (e lança!), por que nosso handler de erro não é notificado? Como explicamos anteriormente, é porque *aquele* handler de erro é para a promise `p`, que já foi realizada com o valor `42`. A promise `p` é imutável, então a única promise que pode ser notificada do erro é a retornada de `p.then(..)`, que neste caso não capturamos.

Isso deveria pintar um quadro claro de por que o tratamento de erros com Promises é propenso a erros (trocadilho intencional). É fácil demais ter erros engolidos, já que isso é muito raramente o que você pretenderia.

**Aviso:** Se você usa a API de Promise de uma forma inválida e um erro ocorre que impede a construção apropriada da Promise, o resultado será uma exceção imediatamente lançada, **não uma Promise rejeitada**. Alguns exemplos de uso incorreto que falham na construção de Promise: `new Promise(null)`, `Promise.all()`, `Promise.race(42)`, e assim por diante. Você não pode obter uma Promise rejeitada se você não usar a API de Promise de forma válida o suficiente para de fato construir uma Promise em primeiro lugar!

### Poço do Desespero

Jeff Atwood observou anos atrás: linguagens de programação são frequentemente configuradas de tal forma que, por padrão, os desenvolvedores caem no "poço do desespero" (http://blog.codinghorror.com/falling-into-the-pit-of-success/) -- onde acidentes são punidos -- e que você tem que se esforçar mais para fazer certo. Ele nos implorou que, em vez disso, criássemos um "poço do sucesso", onde, por padrão, você cai na ação esperada (bem-sucedida), e assim teria que se esforçar muito para falhar.

O tratamento de erros de Promise é inquestionavelmente um design de "poço do desespero". Por padrão, ele assume que você quer que qualquer erro seja engolido pelo estado da Promise, e se você esquecer de observar esse estado, o erro definha/morre silenciosamente na obscuridade -- geralmente no desespero.

Para evitar perder um erro no silêncio de uma Promise esquecida/descartada, alguns desenvolvedores afirmaram que uma "boa prática" para cadeias de Promise é sempre terminar sua cadeia com um `catch(..)` final, como:

```js
var p = Promise.resolve( 42 );

p.then(
	function fulfilled(msg){
		// números não têm funções de string,
		// então isso vai lançar um erro
		console.log( msg.toLowerCase() );
	}
)
.catch( handleErrors );
```

Como não passamos um handler de rejeição ao `then(..)`, o handler padrão foi substituído, o qual simplesmente propaga o erro para a próxima promise na cadeia. Sendo assim, tanto os erros que chegam a `p` quanto os erros que vêm *depois* de `p` em sua resolução (como o de `msg.toLowerCase()`) vão filtrar até o `handleErrors(..)` final.

Problema resolvido, certo? Não tão rápido!

O que acontece se o próprio `handleErrors(..)` também tiver um erro nele? Quem captura isso? Ainda há ainda outra promise desacompanhada: a que `catch(..)` retorna, a qual não capturamos e para a qual não registramos um handler de rejeição.

Você não pode simplesmente grudar outro `catch(..)` no final daquela cadeia, porque ele também poderia falhar. O último passo em qualquer cadeia de Promise, seja qual for, sempre tem a possibilidade, mesmo que cada vez menor, de ficar pendurado com um erro não capturado preso dentro de uma Promise não observada.

Já soa como um dilema impossível?

### Tratamento de Não Capturados

Não é exatamente um problema fácil de resolver completamente. Há outras formas de abordá-lo que muitos diriam ser *melhores*.

Algumas bibliotecas de Promise adicionaram métodos para registrar algo como um handler de "rejeição global não tratada", que seria chamado em vez de um erro lançado globalmente. Mas a solução delas para como identificar um erro como "não capturado" é ter um timer de duração arbitrária, digamos 3 segundos, rodando a partir do momento da rejeição. Se uma Promise é rejeitada, mas nenhum handler de erro é registrado antes de o timer disparar, então assume-se que você nunca vai registrar um handler, então ele é "não capturado".

Na prática, isso funcionou bem para muitas bibliotecas, já que a maioria dos padrões de uso tipicamente não exige um atraso significativo entre a rejeição da Promise e a observação dessa rejeição. Mas esse padrão é problemático porque 3 segundos é tão arbitrário (mesmo que empírico), e também porque há de fato alguns casos em que você quer que uma Promise se agarre à sua condição de rejeitada por algum período de tempo indefinido, e você não quer realmente ter seu handler de "não capturado" chamado para todos esses falsos positivos ("erros não capturados" ainda-não-tratados).

Outra sugestão mais comum é que as Promises deveriam ter um `done(..)` adicionado a elas, que essencialmente marca a cadeia de Promise como "concluída" (done). `done(..)` não cria e retorna uma Promise, então os callbacks passados para `done(..)` obviamente não estão conectados para reportar problemas a uma Promise encadeada que não existe.

Então o que acontece em vez disso? É tratado como você normalmente esperaria em condições de erro não capturado: qualquer exceção dentro de um handler de rejeição de `done(..)` seria lançada como um erro global não capturado (no console do desenvolvedor, basicamente):

```js
var p = Promise.resolve( 42 );

p.then(
	function fulfilled(msg){
		// números não têm funções de string,
		// então isso vai lançar um erro
		console.log( msg.toLowerCase() );
	}
)
.done( null, handleErrors );

// se `handleErrors(..)` causasse sua própria exceção, ela
// seria lançada globalmente aqui
```

Isso pode soar mais atraente do que a cadeia sem fim ou os timeouts arbitrários. Mas o maior problema é que ele não faz parte do padrão ES6, então não importa o quão bom soe, na melhor das hipóteses está bem longe de ser uma solução confiável e onipresente.

Então estamos apenas presos? Não inteiramente.

Os navegadores têm uma capacidade única que nosso código não tem: eles podem rastrear e saber com certeza quando qualquer objeto é descartado e coletado pelo coletor de lixo (garbage collected). Então, os navegadores podem rastrear objetos de Promise e, sempre que eles são coletados pelo coletor de lixo, se houver uma rejeição neles, o navegador sabe com certeza que isso foi um legítimo "erro não capturado", e pode assim saber com confiança que deveria reportá-lo ao console do desenvolvedor.

**Nota:** No momento em que isto foi escrito, tanto o Chrome quanto o Firefox têm tentativas iniciais desse tipo de capacidade de "rejeição não capturada", embora o suporte seja, na melhor das hipóteses, incompleto.

No entanto, se uma Promise não é coletada pelo coletor de lixo -- é extremamente fácil isso acontecer acidentalmente por meio de muitos padrões de código diferentes --, a farejação da coleta de lixo do navegador não vai te ajudar a saber e diagnosticar que você tem uma Promise silenciosamente rejeitada por aí.

Há alguma outra alternativa? Sim.

### Poço do Sucesso

O que segue é apenas teórico, como as Promises *poderiam* algum dia ser mudadas para se comportar. Eu acredito que seria muito superior ao que temos atualmente. E eu acho que essa mudança seria possível mesmo pós-ES6, porque eu não acho que ela quebraria a compatibilidade web com as Promises do ES6. Além disso, ela pode ser polyfillada/prollyfillada, se você tiver cuidado. Vamos dar uma olhada:

* As Promises poderiam, por padrão, reportar (ao console do desenvolvedor) qualquer rejeição, no próximo Job ou tick do loop de eventos, se naquele exato momento nenhum handler de erro tiver sido registrado para a Promise.
* Para os casos em que você quer que uma Promise rejeitada se agarre ao seu estado rejeitado por uma quantidade indefinida de tempo antes de observar, você poderia chamar `defer()`, que suprime o reporte automático de erro naquela Promise.

Se uma Promise é rejeitada, ela, por padrão, reporta ruidosamente esse fato ao console do desenvolvedor (em vez de, por padrão, ficar em silêncio). Você pode optar por sair (opt out) desse reporte tanto implicitamente (registrando um handler de erro antes da rejeição) quanto explicitamente (com `defer()`). Em qualquer caso, *você* controla os falsos positivos.

Considere:

```js
var p = Promise.reject( "Oops" ).defer();

// `foo(..)` é ciente de Promise
foo( 42 )
.then(
	function fulfilled(){
		return p;
	},
	function rejected(err){
		// trata o erro de `foo(..)`
	}
);
...
```

Quando criamos `p`, sabemos que vamos esperar um tempo para usar/observar sua rejeição, então chamamos `defer()` -- assim, sem reporte global. `defer()` simplesmente retorna a mesma promise, para fins de encadeamento.

A promise retornada de `foo(..)` recebe um handler de erro anexado *imediatamente*, então ela implicitamente optou por sair e nenhum reporte global para ela ocorre tampouco.

Mas a promise retornada da chamada `then(..)` não tem `defer()` nem handler de erro anexado, então se ela rejeitar (de dentro de qualquer um dos handlers de resolução), então *ela* será reportada ao console do desenvolvedor como um erro não capturado.

**Esse design é um poço do sucesso.** Por padrão, todos os erros ou são tratados ou são reportados -- o que quase todos os desenvolvedores em quase todos os casos esperariam. Você ou tem que registrar um handler ou tem que intencionalmente optar por sair, e indicar que você pretende deferir o tratamento de erros para *depois*; você está optando pela responsabilidade extra apenas naquele caso específico.

O único perigo real nessa abordagem é se você faz `defer()` de uma Promise mas então falha em de fato algum dia observar/tratar sua rejeição.

Mas você teve que intencionalmente chamar `defer()` para optar por aquele poço do desespero -- o padrão era o poço do sucesso --, então não há muito mais que pudéssemos fazer para te salvar dos seus próprios erros.

Eu acho que ainda há esperança para o tratamento de erros de Promise (pós-ES6). Eu espero que os poderes constituídos repensem a situação e considerem essa alternativa. Nesse meio-tempo, você pode implementar isso você mesmo (um exercício desafiador para o leitor!), ou usar uma biblioteca de Promise *mais inteligente* que faça isso por você!

**Nota:** Esse exato modelo para tratamento/reporte de erros está implementado na minha biblioteca de abstração de Promise *asynquence*, que será discutida no Apêndice A deste livro.

## Padrões de Promise

Já vimos implicitamente o padrão de sequência com cadeias de Promise (controle de fluxo isto-então-isto-então-aquilo), mas há muitas variações de padrões assíncronos que podemos construir como abstrações em cima das Promises. Esses padrões servem para simplificar a expressão do controle de fluxo assíncrono -- o que ajuda a tornar nosso código mais razoável e mais mantenível -- mesmo nas partes mais complexas dos nossos programas.

Dois desses padrões são codificados diretamente na implementação nativa de `Promise` do ES6, então nós os recebemos de graça, para usar como blocos de construção para outros padrões.

### Promise.all([ .. ])

Em uma sequência assíncrona (cadeia de Promise), apenas uma tarefa assíncrona está sendo coordenada em qualquer momento dado -- o passo 2 segue estritamente o passo 1, e o passo 3 segue estritamente o passo 2. Mas e quanto a fazer dois ou mais passos concorrentemente (ou seja, "em paralelo")?

Na terminologia clássica de programação, um "gate" (portão) é um mecanismo que espera duas ou mais tarefas paralelas/concorrentes serem concluídas antes de continuar. Não importa em que ordem elas terminam, apenas que todas elas têm que ser concluídas para que o gate abra e deixe o controle de fluxo passar.

Na API de Promise, chamamos esse padrão de `all([ .. ])`.

Digamos que você queira fazer duas requisições Ajax ao mesmo tempo, e esperar ambas terminarem, independentemente da ordem delas, antes de fazer uma terceira requisição Ajax. Considere:

```js
// `request(..)` é um utilitário Ajax ciente de Promise,
// como o que definimos anteriormente no capítulo

var p1 = request( "http://some.url.1/" );
var p2 = request( "http://some.url.2/" );

Promise.all( [p1,p2] )
.then( function(msgs){
	// tanto `p1` quanto `p2` se realizam e passam
	// suas mensagens aqui
	return request(
		"http://some.url.3/?v=" + msgs.join(",")
	);
} )
.then( function(msg){
	console.log( msg );
} );
```

`Promise.all([ .. ])` espera um único argumento, um `array`, consistindo geralmente em instâncias de Promise. A promise retornada da chamada `Promise.all([ .. ])` receberá uma mensagem de realização (`msgs` neste trecho) que é um `array` de todas as mensagens de realização das promises passadas, na mesma ordem em que foram especificadas (independentemente da ordem de realização).

**Nota:** Tecnicamente, o `array` de valores passado para `Promise.all([ .. ])` pode incluir Promises, thenables, ou até valores imediatos. Cada valor na lista é essencialmente passado por `Promise.resolve(..)` para garantir que seja uma Promise genuína a ser esperada, então um valor imediato será apenas normalizado em uma Promise para aquele valor. Se o `array` está vazio, a Promise principal é imediatamente realizada.

A promise principal retornada de `Promise.all([ .. ])` só será realizada se e quando todas as suas promises constituintes forem realizadas. Se qualquer uma dessas promises em vez disso for rejeitada, a promise principal de `Promise.all([ .. ])` é imediatamente rejeitada, descartando todos os resultados de quaisquer outras promises.

Lembre-se de sempre anexar um handler de rejeição/erro a cada promise, inclusive e especialmente à que retorna de `Promise.all([ .. ])`.

### Promise.race([ .. ])

Embora `Promise.all([ .. ])` coordene múltiplas Promises concorrentemente e assuma que todas são necessárias para a realização, às vezes você só quer responder à "primeira Promise a cruzar a linha de chegada", deixando as outras Promises de lado.

Esse padrão é classicamente chamado de "latch" (trinco), mas em Promises ele é chamado de "race" (corrida).

**Aviso:** Embora a metáfora de "apenas o primeiro a cruzar a linha de chegada vence" se encaixe bem no comportamento, infelizmente "race" é um termo meio carregado, porque "condições de corrida" (race conditions) são geralmente tidas como bugs em programas (veja o Capítulo 1). Não confunda `Promise.race([ .. ])` com "condição de corrida".

`Promise.race([ .. ])` também espera um único argumento `array`, contendo uma ou mais Promises, thenables, ou valores imediatos. Não faz muito sentido prático ter uma corrida com valores imediatos, porque o primeiro listado obviamente vencerá -- como uma corrida a pé em que um corredor começa na linha de chegada!

De forma similar a `Promise.all([ .. ])`, `Promise.race([ .. ])` se realizará se e quando qualquer resolução de Promise for uma realização, e rejeitará se e quando qualquer resolução de Promise for uma rejeição.

**Aviso:** Uma "corrida" exige pelo menos um "corredor", então se você passar um `array` vazio, em vez de resolver imediatamente, a Promise principal de `race([..])` nunca resolverá. Isso é uma armadilha (footgun)! O ES6 deveria ter especificado que ela ou se realiza, rejeita, ou apenas lança algum tipo de erro síncrono. Infelizmente, por causa de precedência em bibliotecas de Promise anteriores à `Promise` do ES6, eles tiveram que deixar essa pegadinha aí, então tome cuidado para nunca enviar um `array` vazio.

Vamos revisitar nosso exemplo anterior de Ajax concorrente, mas no contexto de uma corrida entre `p1` e `p2`:

```js
// `request(..)` é um utilitário Ajax ciente de Promise,
// como o que definimos anteriormente no capítulo

var p1 = request( "http://some.url.1/" );
var p2 = request( "http://some.url.2/" );

Promise.race( [p1,p2] )
.then( function(msg){
	// ou `p1` ou `p2` vencerá a corrida
	return request(
		"http://some.url.3/?v=" + msg
	);
} )
.then( function(msg){
	console.log( msg );
} );
```

Como apenas uma promise vence, o valor de realização é uma única mensagem, não um `array` como era para `Promise.all([ .. ])`.

#### Corrida de Timeout

Vimos este exemplo anteriormente, ilustrando como `Promise.race([ .. ])` pode ser usado para expressar o padrão de "timeout de promise":

```js
// `foo()` é uma função ciente de Promise

// `timeoutPromise(..)`, definida anteriormente, retorna
// uma Promise que rejeita após um atraso especificado

// configura um timeout para `foo()`
Promise.race( [
	foo(),					// tenta `foo()`
	timeoutPromise( 3000 )	// dá 3 segundos a ela
] )
.then(
	function(){
		// `foo(..)` foi realizada a tempo!
	},
	function(err){
		// ou `foo()` foi rejeitada, ou apenas
		// não terminou a tempo, então inspecione
		// `err` para saber qual foi o caso
	}
);
```

Esse padrão de timeout funciona bem na maioria dos casos. Mas há algumas nuances a considerar e, francamente, elas se aplicam igualmente tanto a `Promise.race([ .. ])` quanto a `Promise.all([ .. ])`.

#### "Finally"

A pergunta-chave a fazer é: "O que acontece com as promises que são descartadas/ignoradas?" Não estamos fazendo essa pergunta da perspectiva de desempenho -- elas tipicamente acabariam elegíveis para coleta de lixo --, mas da perspectiva comportamental (efeitos colaterais, etc.). As Promises não podem ser canceladas -- e não deveriam ser, pois isso destruiria a confiança de imutabilidade externa discutida na seção "Promise Incancelável" mais adiante neste capítulo --, então elas só podem ser silenciosamente ignoradas.

Mas e se `foo()` no exemplo anterior está reservando algum tipo de recurso para uso, mas o timeout dispara primeiro e faz com que aquela promise seja ignorada? Há algo nesse padrão que proativamente libere o recurso reservado após o timeout, ou de outra forma cancele quaisquer efeitos colaterais que ela possa ter tido? E se tudo que você queria fosse registrar o fato de que `foo()` deu timeout?

Alguns desenvolvedores propuseram que as Promises precisam de um registro de callback `finally(..)`, que é sempre chamado quando uma Promise resolve, e permite que você especifique qualquer limpeza que possa ser necessária. Isso não existe na especificação no momento, mas pode vir no ES7+. Teremos que esperar para ver.

Poderia se parecer com:

```js
var p = Promise.resolve( 42 );

p.then( something )
.finally( cleanup )
.then( another )
.finally( cleanup );
```

**Nota:** Em várias bibliotecas de Promise, `finally(..)` ainda cria e retorna uma nova Promise (para manter a cadeia andando). Se a função `cleanup(..)` retornasse uma Promise, ela seria ligada à cadeia, o que significa que você ainda poderia ter as questões de rejeição não tratada que discutimos anteriormente.

Nesse meio-tempo, poderíamos fazer um utilitário auxiliar estático que nos permita observar (sem interferir) a resolução de uma Promise:

```js
// verificação de guarda segura para polyfill
if (!Promise.observe) {
	Promise.observe = function(pr,cb) {
		// observa-paralelamente a resolução de `pr`
		pr.then(
			function fulfilled(msg){
				// agenda o callback de forma assíncrona (como Job)
				Promise.resolve( msg ).then( cb );
			},
			function rejected(err){
				// agenda o callback de forma assíncrona (como Job)
				Promise.resolve( err ).then( cb );
			}
		);

		// retorna a promise original
		return pr;
	};
}
```

Eis como usaríamos isso no exemplo de timeout de antes:

```js
Promise.race( [
	Promise.observe(
		foo(),					// tenta `foo()`
		function cleanup(msg){
			// faz a limpeza após `foo()`, mesmo que ela
			// não tenha terminado antes do timeout
		}
	),
	timeoutPromise( 3000 )	// dá 3 segundos a ela
] )
```

Esse auxiliar `Promise.observe(..)` é apenas uma ilustração de como você poderia observar as conclusões de Promises sem interferir nelas. Outras bibliotecas de Promise têm suas próprias soluções. Independentemente de como você faça isso, você provavelmente terá lugares onde quer ter certeza de que suas Promises não são *apenas* silenciosamente ignoradas por acidente.

### Variações em all([ .. ]) e race([ .. ])

Embora as Promises nativas do ES6 venham com `Promise.all([ .. ])` e `Promise.race([ .. ])` embutidos, há vários outros padrões comumente usados com variações dessas semânticas:

* `none([ .. ])` é como `all([ .. ])`, mas as realizações e rejeições são transpostas. Todas as Promises precisam ser rejeitadas -- as rejeições se tornam os valores de realização e vice-versa.
* `any([ .. ])` é como `all([ .. ])`, mas ignora quaisquer rejeições, então apenas uma precisa se realizar em vez de *todas* elas.
* `first([ .. ])` é como uma corrida com `any([ .. ])`, no sentido de que ignora quaisquer rejeições e se realiza assim que a primeira Promise se realiza.
* `last([ .. ])` é como `first([ .. ])`, mas apenas a realização mais recente vence.

Algumas bibliotecas de abstração de Promise fornecem essas, mas você também poderia defini-las você mesmo usando os mecanismos das Promises, `race([ .. ])` e `all([ .. ])`.

Por exemplo, eis como poderíamos definir `first([ .. ])`:

```js
// verificação de guarda segura para polyfill
if (!Promise.first) {
	Promise.first = function(prs) {
		return new Promise( function(resolve,reject){
			// percorre todas as promises
			prs.forEach( function(pr){
				// normaliza o valor
				Promise.resolve( pr )
				// qualquer uma que se realizar primeiro vence, e
				// consegue resolver a promise principal
				.then( resolve );
			} );
		} );
	};
}
```

**Nota:** Esta implementação de `first(..)` não rejeita se todas as suas promises rejeitam; ela simplesmente fica pendurada, muito como um `Promise.race([])` faz. Se desejado, você poderia adicionar lógica adicional para rastrear cada rejeição de promise e, se todas rejeitarem, chamar `reject()` na promise principal. Deixaremos isso como um exercício para o leitor.

### Iterações Concorrentes

Às vezes você quer iterar sobre uma lista de Promises e realizar alguma tarefa contra todas elas, muito como você pode fazer com `array`s síncronos (por exemplo, `forEach(..)`, `map(..)`, `some(..)` e `every(..)`). Se a tarefa a ser realizada contra cada Promise é fundamentalmente síncrona, essas funcionam bem, assim como usamos `forEach(..)` no trecho anterior.

Mas se as tarefas são fundamentalmente assíncronas, ou podem/deveriam de outra forma ser realizadas concorrentemente, você pode usar versões assíncronas desses utilitários, como fornecidas por muitas bibliotecas.

Por exemplo, vamos considerar um utilitário `map(..)` assíncrono que recebe um `array` de valores (poderiam ser Promises ou qualquer outra coisa), mais uma função (tarefa) para realizar contra cada um. O próprio `map(..)` retorna uma promise cujo valor de realização é um `array` que contém (na mesma ordem de mapeamento) o valor de realização assíncrono de cada tarefa:

```js
if (!Promise.map) {
	Promise.map = function(vals,cb) {
		// nova promise que espera por todas as promises mapeadas
		return Promise.all(
			// nota: o `map(..)` normal de array transforma
			// o array de valores em um array de
			// promises
			vals.map( function(val){
				// substitui `val` por uma nova promise que
				// resolve após `val` ser mapeado de forma assíncrona
				return new Promise( function(resolve){
					cb( val, resolve );
				} );
			} )
		);
	};
}
```

**Nota:** Nesta implementação de `map(..)`, você não pode sinalizar rejeição assíncrona, mas se uma exceção/erro síncrono ocorrer dentro do callback de mapeamento (`cb(..)`), a promise principal retornada de `Promise.map(..)` rejeitaria.

Vamos ilustrar o uso de `map(..)` com uma lista de Promises (em vez de valores simples):

```js
var p1 = Promise.resolve( 21 );
var p2 = Promise.resolve( 42 );
var p3 = Promise.reject( "Oops" );

// dobra os valores na lista mesmo que eles estejam
// em Promises
Promise.map( [p1,p2,p3], function(pr,done){
	// garante que o próprio item seja uma Promise
	Promise.resolve( pr )
	.then(
		// extrai o valor como `v`
		function(v){
			// mapeia a realização `v` para um novo valor
			done( v * 2 );
		},
		// ou, mapeia para a mensagem de rejeição da promise
		done
	);
} )
.then( function(vals){
	console.log( vals );	// [42,84,"Oops"]
} );
```

## Recapitulação da API de Promise

Vamos revisar a API de `Promise` do ES6 que já vimos se desenrolar em pedaços ao longo deste capítulo.

**Nota:** A API a seguir é nativa apenas a partir do ES6, mas há polyfills compatíveis com a especificação (não apenas bibliotecas de Promise estendidas) que podem definir `Promise` e todo o seu comportamento associado para que você possa usar Promises nativas mesmo em navegadores pré-ES6. Um desses polyfills é o "Native Promise Only" (http://github.com/getify/native-promise-only), que eu escrevi!

### Construtor new Promise(..)

O *construtor revelador* `Promise(..)` deve ser usado com `new`, e deve receber um callback de função que é chamado de forma síncrona/imediata. Essa função recebe dois callbacks de função que atuam como capacidades de resolução para a promise. Comumente rotulamos esses `resolve(..)` e `reject(..)`:

```js
var p = new Promise( function(resolve,reject){
	// `resolve(..)` para resolver/realizar a promise
	// `reject(..)` para rejeitar a promise
} );
```

`reject(..)` simplesmente rejeita a promise, mas `resolve(..)` pode tanto realizar a promise quanto rejeitá-la, dependendo do que lhe é passado. Se `resolve(..)` recebe um valor imediato, não-Promise, não-thenable, então a promise é realizada com aquele valor.

Mas se `resolve(..)` recebe um valor de Promise genuína ou thenable, esse valor é desembrulhado recursivamente, e qualquer que seja sua resolução/estado final, ele será adotado pela promise.

### Promise.resolve(..) e Promise.reject(..)

Um atalho para criar uma Promise já rejeitada é `Promise.reject(..)`, então estas duas promises são equivalentes:

```js
var p1 = new Promise( function(resolve,reject){
	reject( "Oops" );
} );

var p2 = Promise.reject( "Oops" );
```

`Promise.resolve(..)` é geralmente usado para criar uma Promise já realizada de forma similar a `Promise.reject(..)`. No entanto, `Promise.resolve(..)` também desembrulha valores thenable (como discutido várias vezes já). Nesse caso, a Promise retornada adota a resolução final do thenable que você passou, que poderia ser tanto realização quanto rejeição:

```js
var fulfilledTh = {
	then: function(cb) { cb( 42 ); }
};
var rejectedTh = {
	then: function(cb,errCb) {
		errCb( "Oops" );
	}
};

var p1 = Promise.resolve( fulfilledTh );
var p2 = Promise.resolve( rejectedTh );

// `p1` será uma promise realizada
// `p2` será uma promise rejeitada
```

E lembre-se, `Promise.resolve(..)` não faz nada se o que você passa já é uma Promise genuína; ela apenas retorna o valor diretamente. Então não há sobrecarga em chamar `Promise.resolve(..)` em valores cuja natureza você não conhece, caso algum por acaso já seja uma Promise genuína.

### then(..) e catch(..)

Cada instância de Promise (**não** o namespace da API `Promise`) tem métodos `then(..)` e `catch(..)`, que permitem o registro de handlers de realização e rejeição para a Promise. Uma vez que a Promise é resolvida, um ou outro desses handlers será chamado, mas não ambos, e sempre será chamado de forma assíncrona (veja "Jobs" no Capítulo 1).

`then(..)` recebe um ou dois parâmetros, o primeiro para o callback de realização e o segundo para o callback de rejeição. Se qualquer um for omitido ou de outra forma passado como um valor que não seja função, um callback padrão é substituído respectivamente. O callback de realização padrão simplesmente passa a mensagem adiante, enquanto o callback de rejeição padrão simplesmente relança (propaga) o motivo do erro que recebe.

`catch(..)` recebe apenas o callback de rejeição como parâmetro, e automaticamente substitui o callback de realização padrão, como acabamos de discutir. Em outras palavras, é equivalente a `then(null,..)`:

```js
p.then( fulfilled );

p.then( fulfilled, rejected );

p.catch( rejected ); // ou `p.then( null, rejected )`
```

`then(..)` e `catch(..)` também criam e retornam uma nova promise, que pode ser usada para expressar o controle de fluxo encadeado de Promise. Se os callbacks de realização ou rejeição tiverem uma exceção lançada, a promise retornada é rejeitada. Se qualquer callback retorna um valor imediato, não-Promise, não-thenable, esse valor é definido como a realização para a promise retornada. Se o handler de realização especificamente retorna uma promise ou valor thenable, esse valor é desembrulhado e se torna a resolução da promise retornada.

### Promise.all([ .. ]) e Promise.race([ .. ])

Os auxiliares estáticos `Promise.all([ .. ])` e `Promise.race([ .. ])` na API de `Promise` do ES6 ambos criam uma Promise como seu valor de retorno. A resolução dessa promise é controlada inteiramente pelo array de promises que você passa.

Para `Promise.all([ .. ])`, todas as promises que você passa devem se realizar para que a promise retornada se realize. Se qualquer promise é rejeitada, a promise principal retornada é imediatamente rejeitada também (descartando os resultados de quaisquer outras promises). Para a realização, você recebe um `array` de todos os valores de realização das promises passadas. Para a rejeição, você recebe apenas o primeiro valor de motivo de rejeição de promise. Esse padrão é classicamente chamado de "gate" (portão): todas devem chegar antes de o portão abrir.

Para `Promise.race([ .. ])`, apenas a primeira promise a resolver (realização ou rejeição) "vence", e qualquer que seja essa resolução, ela se torna a resolução da promise retornada. Esse padrão é classicamente chamado de "latch" (trinco): o primeiro a abrir o trinco passa. Considere:

```js
var p1 = Promise.resolve( 42 );
var p2 = Promise.resolve( "Hello World" );
var p3 = Promise.reject( "Oops" );

Promise.race( [p1,p2,p3] )
.then( function(msg){
	console.log( msg );		// 42
} );

Promise.all( [p1,p2,p3] )
.catch( function(err){
	console.error( err );	// "Oops"
} );

Promise.all( [p1,p2] )
.then( function(msgs){
	console.log( msgs );	// [42,"Hello World"]
} );
```

**Aviso:** Tome cuidado! Se um `array` vazio é passado para `Promise.all([ .. ])`, ele se realizará imediatamente, mas `Promise.race([ .. ])` ficará pendurado para sempre e nunca resolverá.

A API de `Promise` do ES6 é bem simples e direta. Ela é pelo menos boa o suficiente para servir aos casos assíncronos mais básicos, e é um bom lugar para começar ao reorganizar seu código do inferno dos callbacks para algo melhor.

Mas há uma porção inteira de sofisticação assíncrona que aplicativos frequentemente exigem, e que as próprias Promises serão limitadas em endereçar. Na próxima seção, vamos mergulhar nessas limitações como motivações para o benefício das bibliotecas de Promise.

## Limitações de Promise

Muitos dos detalhes que discutiremos nesta seção já foram aludidos neste capítulo, mas vamos apenas garantir revisar essas limitações especificamente.

### Tratamento de Erros em Sequência

Cobrimos o tratamento de erros à moda das Promises em detalhe anteriormente neste capítulo. As limitações de como as Promises são projetadas -- como elas encadeiam, especificamente -- criam uma armadilha muito fácil em que um erro em uma cadeia de Promise pode ser silenciosamente ignorado acidentalmente.

Mas há algo mais a considerar com erros de Promise. Como uma cadeia de Promise não é nada mais do que suas Promises constituintes conectadas juntas, não há entidade para se referir à cadeia inteira como uma única *coisa*, o que significa que não há forma externa de observar quaisquer erros que possam ocorrer.

Se você constrói uma cadeia de Promise que não tem tratamento de erros nela, qualquer erro em qualquer lugar na cadeia se propagará indefinidamente cadeia abaixo, até ser observado (registrando um handler de rejeição em algum passo). Então, nesse caso específico, ter uma referência à *última* promise na cadeia é suficiente (`p` no trecho a seguir), porque você pode registrar um handler de rejeição ali, e ele será notificado de quaisquer erros propagados:

```js
// `foo(..)`, `STEP2(..)` e `STEP3(..)` são
// todos utilitários cientes de promise

var p = foo( 42 )
.then( STEP2 )
.then( STEP3 );
```

Embora possa parecer sorrateiramente confuso, `p` aqui não aponta para a primeira promise na cadeia (a da chamada `foo(42)`), mas sim para a última promise, a que vem da chamada `then(STEP3)`.

Além disso, nenhum passo na cadeia de promise está observavelmente fazendo seu próprio tratamento de erros. Isso significa que você poderia então registrar um handler de erro de rejeição em `p`, e ele seria notificado se quaisquer erros ocorrerem em qualquer lugar na cadeia:

```
p.catch( handleErrors );
```

Mas se qualquer passo da cadeia de fato faz seu próprio tratamento de erros (talvez escondido/abstraído do que você pode ver), seu `handleErrors(..)` não será notificado. Isso pode ser o que você quer -- afinal, foi uma "rejeição tratada" --, mas também pode *não* ser o que você quer. A completa falta de capacidade de ser notificado (de erros de rejeição "já tratados") é uma limitação que restringe as capacidades em alguns casos de uso.

É basicamente a mesma limitação que existe com um `try..catch` que pode capturar uma exceção e simplesmente engoli-la. Então isso não é uma limitação **exclusiva das Promises**, mas *é* algo para o qual poderíamos desejar ter uma solução alternativa.

Infelizmente, muitas vezes não há referência mantida para os passos intermediários em uma sequência de cadeia de Promise, então sem tais referências, você não pode anexar handlers de erro para observar os erros de forma confiável.

### Valor Único

As Promises, por definição, têm apenas um único valor de realização ou um único motivo de rejeição. Em exemplos simples, isso não é grande coisa, mas em cenários mais sofisticados, você pode achar isso limitante.

O conselho típico é construir um invólucro de valores (como um `object` ou `array`) para conter essas múltiplas mensagens. Essa solução funciona, mas pode ser bem desajeitada e tediosa envolver e desembrulhar suas mensagens a cada passo da sua cadeia de Promise.

#### Dividindo Valores

Às vezes você pode tomar isso como um sinal de que você poderia/deveria decompor o problema em duas ou mais Promises.

Imagine que você tem um utilitário `foo(..)` que produz dois valores (`x` e `y`) de forma assíncrona:

```js
function getY(x) {
	return new Promise( function(resolve,reject){
		setTimeout( function(){
			resolve( (3 * x) - 1 );
		}, 100 );
	} );
}

function foo(bar,baz) {
	var x = bar * baz;

	return getY( x )
	.then( function(y){
		// envolve ambos os valores em um contêiner
		return [x,y];
	} );
}

foo( 10, 20 )
.then( function(msgs){
	var x = msgs[0];
	var y = msgs[1];

	console.log( x, y );	// 200 599
} );
```

Primeiro, vamos reorganizar o que `foo(..)` retorna para que não tenhamos que envolver `x` e `y` em um único valor de `array` para transportar através de uma Promise. Em vez disso, podemos envolver cada valor em sua própria promise:

```js
function foo(bar,baz) {
	var x = bar * baz;

	// retorna ambas as promises
	return [
		Promise.resolve( x ),
		getY( x )
	];
}

Promise.all(
	foo( 10, 20 )
)
.then( function(msgs){
	var x = msgs[0];
	var y = msgs[1];

	console.log( x, y );
} );
```

Um `array` de promises é realmente melhor do que um `array` de valores passado através de uma única promise? Sintaticamente, não é grande melhoria.

Mas essa abordagem abraça mais de perto a teoria de design de Promise. Agora é mais fácil no futuro refatorar para dividir o cálculo de `x` e `y` em funções separadas. É mais limpo e mais flexível deixar o código chamador decidir como orquestrar as duas promises -- usando `Promise.all([ .. ])` aqui, mas certamente não a única opção -- em vez de abstrair tais detalhes para dentro de `foo(..)`.

#### Desembrulhar/Espalhar Argumentos

As atribuições `var x = ..` e `var y = ..` ainda são uma sobrecarga desajeitada. Podemos empregar alguma trapaça funcional (créditos a Reginald Braithwaite, @raganwald no Twitter) em um utilitário auxiliar:

```js
function spread(fn) {
	return Function.apply.bind( fn, null );
}

Promise.all(
	foo( 10, 20 )
)
.then(
	spread( function(x,y){
		console.log( x, y );	// 200 599
	} )
)
```

Isso é um pouco mais agradável! É claro que você poderia colocar a mágica funcional em linha para evitar o auxiliar extra:

```js
Promise.all(
	foo( 10, 20 )
)
.then( Function.apply.bind(
	function(x,y){
		console.log( x, y );	// 200 599
	},
	null
) );
```

Esses truques podem ser bacanas, mas o ES6 tem uma resposta ainda melhor para nós: desestruturação (destructuring). A forma de atribuição por desestruturação de array se parece com isto:

```js
Promise.all(
	foo( 10, 20 )
)
.then( function(msgs){
	var [x,y] = msgs;

	console.log( x, y );	// 200 599
} );
```

Mas o melhor de tudo, o ES6 oferece a forma de desestruturação de parâmetro de array:

```js
Promise.all(
	foo( 10, 20 )
)
.then( function([x,y]){
	console.log( x, y );	// 200 599
} );
```

Agora abraçamos o mantra de um-valor-por-Promise, mas mantivemos nosso código repetitivo de apoio ao mínimo!

**Nota:** Para mais informações sobre as formas de desestruturação do ES6, veja o título *ES6 & Beyond* desta série.

### Resolução Única

Um dos comportamentos mais intrínsecos das Promises é que uma Promise só pode ser resolvida uma vez (realização ou rejeição). Para muitos casos de uso assíncrono, você só está recuperando um valor uma vez, então isso funciona bem.

Mas há também muitos casos assíncronos que se encaixam em um modelo diferente -- um que é mais parecido com eventos e/ou fluxos (streams) de dados. Não está claro à primeira vista o quão bem as Promises podem se encaixar em tais casos de uso, se é que se encaixam. Sem uma abstração significativa em cima das Promises, elas vão ficar completamente aquém para lidar com a resolução de múltiplos valores.

Imagine um cenário em que você pode querer disparar uma sequência de passos assíncronos em resposta a um estímulo (como um evento) que pode de fato acontecer múltiplas vezes, como um clique de botão.

Isso provavelmente não vai funcionar do jeito que você quer:

```js
// `click(..)` vincula o evento `"click"` a um elemento do DOM
// `request(..)` é o Ajax ciente de Promise definido anteriormente

var p = new Promise( function(resolve,reject){
	click( "#mybtn", resolve );
} );

p.then( function(evt){
	var btnID = evt.currentTarget.id;
	return request( "http://some.url.1/?id=" + btnID );
} )
.then( function(text){
	console.log( text );
} );
```

O comportamento aqui só funciona se sua aplicação demanda que o botão seja clicado apenas uma vez. Se o botão é clicado uma segunda vez, a promise `p` já foi resolvida, então a segunda chamada `resolve(..)` seria ignorada.

Em vez disso, você provavelmente precisaria inverter o paradigma, criando uma cadeia de Promise inteiramente nova para cada disparo de evento:

```js
click( "#mybtn", function(evt){
	var btnID = evt.currentTarget.id;

	request( "http://some.url.1/?id=" + btnID )
	.then( function(text){
		console.log( text );
	} );
} );
```

Essa abordagem *funcionará*, no sentido de que uma sequência de Promise inteiramente nova será disparada para cada evento `"click"` no botão.

Mas, além da simples feiura de ter que definir a cadeia de Promise inteira dentro do handler de evento, esse design em certos aspectos viola a ideia de separação de responsabilidades/capacidades (SoC). Você pode muito bem querer definir seu handler de evento em um lugar diferente no seu código de onde você define a *resposta* ao evento (a cadeia de Promise). Isso é bem desajeitado de fazer nesse padrão, sem mecanismos auxiliares.

**Nota:** Outra forma de articular essa limitação é que seria bom se pudéssemos construir algum tipo de "observável" (observable) ao qual pudéssemos inscrever uma cadeia de Promise. Há bibliotecas que criaram essas abstrações (como o RxJS -- http://rxjs.codeplex.com/), mas as abstrações podem parecer tão pesadas que você nem consegue mais ver a natureza das Promises. Tal abstração pesada traz à mente questões importantes, como se (sem Promises) esses mecanismos são tão *confiáveis* quanto as próprias Promises foram projetadas para ser. Revisitaremos o padrão "Observable" no Apêndice B.

### Inércia

Uma barreira concreta para começar a usar Promises no seu próprio código é todo o código que atualmente existe e que não é já ciente de Promise. Se você tem muito código baseado em callbacks, é muito mais fácil simplesmente continuar programando naquele mesmo estilo.

"Uma base de código em movimento (com callbacks) permanecerá em movimento (com callbacks) a menos que sobre ela aja um desenvolvedor inteligente e ciente de Promises."

As Promises oferecem um paradigma diferente e, como tal, a abordagem ao código pode variar de apenas um pouco diferente a, em alguns casos, radicalmente diferente. Você tem que ser intencional a respeito, porque as Promises não vão simplesmente brotar naturalmente das mesmas velhas formas de fazer código que te serviram bem até aqui.

Considere um cenário baseado em callbacks como o seguinte:

```js
function foo(x,y,cb) {
	ajax(
		"http://some.url.1/?x=" + x + "&y=" + y,
		cb
	);
}

foo( 11, 31, function(err,text) {
	if (err) {
		console.error( err );
	}
	else {
		console.log( text );
	}
} );
```

É imediatamente óbvio quais são os primeiros passos para converter esse código baseado em callbacks para código ciente de Promise? Depende da sua experiência. Quanto mais prática você tiver com isso, mais natural vai parecer. Mas certamente, as Promises não anunciam no rótulo exatamente como fazer isso -- não há uma resposta de tamanho único --, então a responsabilidade é sua.

Como cobrimos antes, definitivamente precisamos de um utilitário Ajax que seja ciente de Promise em vez de baseado em callbacks, que poderíamos chamar de `request(..)`. Você pode fazer o seu próprio, como já fizemos. Mas a sobrecarga de ter que definir manualmente invólucros cientes de Promise para cada utilitário baseado em callbacks torna menos provável que você escolha refatorar para código ciente de Promise de jeito nenhum.

As Promises não oferecem resposta direta para essa limitação. A maioria das bibliotecas de Promise, no entanto, oferece um auxiliar. Mas mesmo sem uma biblioteca, imagine um auxiliar como este:

```js
// verificação de guarda segura para polyfill
if (!Promise.wrap) {
	Promise.wrap = function(fn) {
		return function() {
			var args = [].slice.call( arguments );

			return new Promise( function(resolve,reject){
				fn.apply(
					null,
					args.concat( function(err,v){
						if (err) {
							reject( err );
						}
						else {
							resolve( v );
						}
					} )
				);
			} );
		};
	};
}
```

OK, isso é mais do que apenas um pequeno utilitário trivial. No entanto, embora possa parecer um pouco intimidante, não é tão ruim quanto você pensaria. Ele recebe uma função que espera um callback no estilo erro-primeiro como seu último parâmetro, e retorna uma nova que automaticamente cria uma Promise para retornar, e substitui o callback para você, conectado à realização/rejeição da Promise.

Em vez de desperdiçar muito tempo falando sobre *como* esse auxiliar `Promise.wrap(..)` funciona, vamos apenas olhar como nós o usamos:

```js
var request = Promise.wrap( ajax );

request( "http://some.url.1/" )
.then( .. )
..
```

Uau, isso foi bem fácil!

`Promise.wrap(..)` **não** produz uma Promise. Ele produz uma função que produzirá Promises. Em certo sentido, uma função que produz Promises poderia ser vista como uma "fábrica de Promises". Eu proponho "promisória" como o nome para tal coisa ("Promise" + "fábrica", no inglês "promisory" = "Promise" + "factory").

O ato de envolver uma função que espera callback para ser uma função ciente de Promise é às vezes referido como "lifting" (elevação) ou "promisifying" (promisificação). Mas não parece haver um termo padrão para como chamar a função resultante além de uma "função elevada" (lifted function), então eu gosto mais de "promisória", pois acho que é mais descritiva.

**Nota:** Promisória não é um termo inventado. É uma palavra real (em inglês, "promisory"), e sua definição significa conter ou transmitir uma promessa. É exatamente isso que essas funções estão fazendo, então acaba sendo uma combinação de terminologia bem perfeita!

Então, `Promise.wrap(ajax)` produz uma promisória de `ajax(..)` que chamamos de `request(..)`, e essa promisória produz Promises para respostas Ajax.

Se todas as funções já fossem promisórias, não precisaríamos fazê-las nós mesmos, então o passo extra é uma pequena pena. Mas pelo menos o padrão de envolvimento é (geralmente) repetível, então podemos colocá-lo em um auxiliar `Promise.wrap(..)`, como mostrado, para auxiliar nosso código de promise.

Então, voltando ao nosso exemplo anterior, precisamos de uma promisória tanto para `ajax(..)` quanto para `foo(..)`:

```js
// faz uma promisória para `ajax(..)`
var request = Promise.wrap( ajax );

// refatora `foo(..)`, mas a mantém externamente
// baseada em callbacks por compatibilidade com outras
// partes do código por enquanto -- usa apenas
// a promise de `request(..)` internamente.
function foo(x,y,cb) {
	request(
		"http://some.url.1/?x=" + x + "&y=" + y
	)
	.then(
		function fulfilled(text){
			cb( null, text );
		},
		cb
	);
}

// agora, para os propósitos deste código, faz uma
// promisória para `foo(..)`
var betterFoo = Promise.wrap( foo );

// e usa a promisória
betterFoo( 11, 31 )
.then(
	function fulfilled(text){
		console.log( text );
	},
	function rejected(err){
		console.error( err );
	}
);
```

É claro que, enquanto estamos refatorando `foo(..)` para usar nossa nova promisória `request(..)`, poderíamos simplesmente fazer de `foo(..)` ela própria uma promisória, em vez de permanecer baseada em callbacks e precisar fazer e usar a promisória subsequente `betterFoo(..)`. Essa decisão depende apenas de se `foo(..)` precisa permanecer compatível com callbacks com outras partes da base de código ou não.

Considere:

```js
// `foo(..)` agora também é uma promisória porque ela
// delega para a promisória `request(..)`
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

foo( 11, 31 )
.then( .. )
..
```

Embora as Promises do ES6 não venham nativamente com auxiliares para tal envolvimento promisório, a maioria das bibliotecas os fornece, ou você pode fazer o seu próprio. De qualquer forma, essa limitação particular das Promises é tratável sem muita dor (certamente comparada à dor do inferno dos callbacks!).

### Promise Incancelável

Uma vez que você cria uma Promise e registra um handler de realização e/ou rejeição para ela, não há nada externo que você possa fazer para parar essa progressão se algo mais acontecer que torne essa tarefa irrelevante.

**Nota:** Muitas bibliotecas de abstração de Promise fornecem facilidades para cancelar Promises, mas isso é uma ideia terrível! Muitos desenvolvedores desejam que as Promises tivessem nativamente sido projetadas com capacidade de cancelamento externo, mas o problema é que isso permitiria que um consumidor/observador de uma Promise afetasse a capacidade de algum outro consumidor de observar essa mesma Promise. Isso viola a confiabilidade do valor futuro (imutabilidade externa), mas, além disso, é a personificação do antipadrão "ação à distância" (action at a distance) (http://en.wikipedia.org/wiki/Action_at_a_distance_%28computer_programming%29). Independentemente de quão útil pareça, isso na verdade vai te levar direto de volta aos mesmos pesadelos que os callbacks.

Considere nosso cenário de timeout de Promise de antes:

```js
var p = foo( 42 );

Promise.race( [
	p,
	timeoutPromise( 3000 )
] )
.then(
	doSomething,
	handleError
);

p.then( function(){
	// ainda acontece mesmo no caso de timeout :(
} );
```

O "timeout" era externo à promise `p`, então `p` em si continua andando, o que provavelmente não queremos.

Uma opção é definir de forma invasiva seus callbacks de resolução:

```js
var OK = true;

var p = foo( 42 );

Promise.race( [
	p,
	timeoutPromise( 3000 )
	.catch( function(err){
		OK = false;
		throw err;
	} )
] )
.then(
	doSomething,
	handleError
);

p.then( function(){
	if (OK) {
		// só acontece se não houver timeout! :)
	}
} );
```

Isso é feio. Funciona, mas está longe de ser ideal. Geralmente, você deveria tentar evitar tais cenários.

Mas se você não pode, a feiura desta solução deveria ser uma pista de que *cancelamento* é uma funcionalidade que pertence a um nível mais alto de abstração em cima das Promises. Eu recomendaria que você procurasse assistência em bibliotecas de abstração de Promise em vez de gambiarrar você mesmo.

**Nota:** Minha biblioteca de abstração de Promise *asynquence* fornece justamente tal abstração e uma capacidade `abort()` para a sequência, tudo o que será discutido no Apêndice A.

Uma Promise única não é realmente um mecanismo de controle de fluxo (pelo menos não em um sentido muito significativo), que é exatamente a que *cancelamento* se refere; é por isso que o cancelamento de Promise pareceria desajeitado.

Em contraste, uma cadeia de Promises tomada coletivamente em conjunto -- o que eu gosto de chamar de "sequência" -- *é* uma expressão de controle de fluxo, e assim é apropriado que o cancelamento seja definido naquele nível de abstração.

Nenhuma Promise individual deveria ser cancelável, mas faz sentido que uma *sequência* seja cancelável, porque você não passa adiante uma sequência como um único valor imutável como você faz com uma Promise.

### Desempenho de Promise

Essa limitação particular é tanto simples quanto complexa.

Comparando quantas peças estão se movendo com uma cadeia básica de tarefas assíncronas baseada em callbacks versus uma cadeia de Promise, fica claro que as Promises têm uma boa quantidade a mais acontecendo, o que significa que elas são naturalmente pelo menos um pouquinho mais lentas. Pense de volta apenas na simples lista de garantias de confiança que as Promises oferecem, comparada ao código de solução improvisado (ad hoc) que você teria que colocar em camadas em cima dos callbacks para alcançar as mesmas proteções.

Mais trabalho a fazer, mais guardas para proteger, significa que as Promises *são* mais lentas comparadas a callbacks nus e não confiáveis. Isso é óbvio, e provavelmente simples de digerir.

Mas quão mais lentas? Bem... isso na verdade está se provando uma pergunta incrivelmente difícil de responder de forma absoluta, de forma generalizada.

Francamente, é meio que uma comparação de maçãs com laranjas, então é provavelmente a pergunta errada a fazer. Você deveria na verdade comparar se um sistema de callback improvisado com todas as mesmas proteções colocadas manualmente em camadas é mais rápido do que uma implementação de Promise.

Se as Promises têm uma limitação de desempenho legítima, é mais que elas não oferecem realmente uma escolha item por item de quais proteções de confiabilidade você quer/precisa ou não -- você recebe todas elas, sempre.

No entanto, se concedermos que uma Promise é geralmente *um pouquinho mais lenta* do que seu equivalente não-Promise, não confiável, baseado em callbacks -- assumindo que há lugares onde você sente que pode justificar a falta de confiabilidade --, isso significa que as Promises deveriam ser evitadas de forma generalizada, como se sua aplicação inteira fosse movida por nada além de código que tem-que-ser-absolutamente-o-mais-rápido-possível?

Verificação de sanidade: se seu código é legitimamente assim, **será que JavaScript é sequer a linguagem certa para tais tarefas?** O JavaScript pode ser otimizado para rodar aplicações de forma muito performática (veja o Capítulo 5 e o Capítulo 6). Mas será que obcecar sobre minúsculos trade-offs de desempenho com Promises, à luz de todos os benefícios que elas oferecem, é *realmente* apropriado?

Outra questão sutil é que as Promises tornam *tudo* assíncrono, o que significa que alguns passos completados imediatamente (de forma síncrona) ainda deferem o avanço do próximo passo para um Job (veja o Capítulo 1). Isso significa que é possível que uma sequência de tarefas de Promise pudesse ser concluída ligeiramente-mais-devagar do que a mesma sequência conectada com callbacks.

É claro que a pergunta aqui é esta: esses potenciais deslizes em minúsculas frações de desempenho *valem* todos os outros benefícios articulados das Promises que apresentamos ao longo deste capítulo?

Minha opinião é que em virtualmente todos os casos em que você pode pensar que o desempenho de Promise é lento o suficiente para ser uma preocupação, é na verdade um antipadrão otimizar para longe os benefícios da confiabilidade e da componibilidade das Promises evitando-as por completo.

Em vez disso, você deveria, por padrão, usá-las ao longo da base de código, e então perfilar e analisar os caminhos quentes (críticos) da sua aplicação. As Promises são *realmente* um gargalo, ou são apenas uma lentidão teórica? Só *então*, armado com benchmarks válidos reais (veja o Capítulo 6), é responsável e prudente fatorar para fora as Promises apenas naquelas áreas críticas identificadas.

As Promises são um pouco mais lentas, mas em troca você está recebendo muita confiabilidade, previsibilidade não-Zalgo, e componibilidade embutidas. Talvez a limitação não seja na verdade o desempenho delas, mas sua falta de percepção dos benefícios delas?

## Revisão

As Promises são incríveis. Use-as. Elas resolvem as questões de *inversão de controle* que nos atormentam com código baseado apenas em callbacks.

Elas não se livram dos callbacks, elas apenas redirecionam a orquestração desses callbacks para um mecanismo intermediário confiável que fica entre nós e outro utilitário.

As cadeias de Promise também começam a endereçar (embora certamente não de forma perfeita) uma forma melhor de expressar o fluxo assíncrono de maneira sequencial, o que ajuda nossos cérebros a planejar e manter código JS assíncrono melhor. Veremos uma solução ainda melhor para *esse* problema no próximo capítulo!
