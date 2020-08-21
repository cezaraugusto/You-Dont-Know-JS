# You Don't Know JS: Async & Performance
# Chapter 2: Callbacks

No capítulo 1, nós exploramos a terminologia e conceitos acerca da programação assíncrona no JavaScript. Nosso foco foi entender a fila de loop de eventos mono-thread que guia todos os "eventos" (invocação assíncrona de funções). Também exploramos diversas formas que padrões de concorrência explicam as relações (se houver alguma!) entre cadeias de eventos *executadas simultaneamente*, ou "processos" (tarefas, chamadas de funções, etc.).

Todos os nosso exemplos no Capítulo 1 usaram a função como a unidade indivisível e individual de operações, de forma que dentro da função, declarações sejam executadas de forma previsível (acima do nível do compilador!), mas no nível de ordenação de funções, eventos (aka invocações de funções assíncronas) podem acontecer em ordens variadas.

Em todos esses casos, a função está agindo como um "callback" porque ela serve como um objetivo para o loop de eventos "chamá-la de volta" para dentro do programa, sempre que aquele item na fila for processado.

Como você sem dúvida deve ter observado, callbacks são de longe a forma mais comum que a assincronia em programas em JS é expressa e gerenciada. De fato, o callback é o padrão assíncrono mais fundamental da linguagem.

Inúmeros programas JavaScript, até os muito sofisticados e complexos, têm sido escritos sobre não outra fundação assíncrona que não o callback (claro que com padrões de interação concorrentes que exploramos no Capítulo 1). A função callback é o cavalo de trabalho assíncrono para o JavaScript, e ele respeitavelmente faz seu trabalho.

Só que... callbacks não deixam de ter suas deficiências. Muitos desenvolvedores estão entusiasmados com a *promessa* (trocadilho!) de melhores padrões de assincronia. Mas é impossível usar efetivamente qualquer abstração se você não entender o que está abstraindo e por quê.

Neste capítulo, exploraremos alguns deles em profundidade, como motivação para o porquê de mais padrões assíncronos sofisticados (explorados nos capítulos subsequentes deste livro) serem necessários e desejados.

## Continuações

Vamos voltar ao exemplo de callback assíncrono que começamos no Capítulo 1, mas deixe-me modificá-lo um pouco para ilustrar um ponto:

```js
// A
ajax( "..", function(..){
	// C
} );
// B
```

`// A` e `// B` representam a primeira metade do programa (também conhecido como *agora*) e `// C` marca a segunda metade (também conhecida como *depois*). A primeira metade é executada imediatamente e, em seguida, há uma "pausa" de duração indeterminada. Em algum momento futuro, se a chamada Ajax for concluída, a execução do programa partirá de onde parou e *continuará* com a segunda metade.

Em outras palavras, a função de callback envolve ou encapsula a *continuação* do programa.

Vamos tornar o código ainda mais simples:

```js
// A
setTimeout( function(){
	// C
}, 1000 );
// B
```

Pare por um momento e pergunte a si mesmo como você descreveria (para alguém menos informado sobre como o JS funciona) o modo como este programa se comporta. Vá em frente, tente em voz alta. É um bom exercício que ajudará meus próximos pontos a fazerem mais sentido.

A maioria dos leitores provavelmente pensou ou disse algo como: "Faça A e, em seguida, defina um tempo limite para esperar 1.000 milissegundos, e depois que este tempo acabar, faça C." Quão perto foi a sua interpretação?

Você pode ter se pegado corrigindo automaticamente para: "Faça A, configure o tempo limite de espera para 1.000 milissegundos, em seguida faça B e depois que o tempo limite for disparado, faça C." Isso é mais preciso que a primeira versão. Você pode ver a diferença?

Embora a segunda versão seja mais precisa, as duas são deficientes em explicar esse código de maneira a conectar nossos cérebros ao código e o código ao mecanismo JS. A desconexão é sutil e monumental e está no cerne de compreender as deficiências das callbacks como expressão e gerenciamento assíncronos.

Assim que introduzimos uma só dessas continuações (ou várias dezenas como muitos programas fazem!) na forma de uma callback, permitimos formar uma divergência entre o funcionamento de nossos cérebros e a maneira como o código funcionará. Sempre que esses dois divergem (e longe desse ser o único lugar que isso acontece, como eu tenho certeza que você sabe!), nos deparamos com o fato inevitável de que nosso código se torna mais difícil de entender, raciocinar, depurar e manter.

## Cérebro Sequencial

Tenho certeza de que a maioria de vocês, leitores, já ouviram alguém dizer (até mesmo fez a afirmação): "Sou multitarefa". Os efeitos de tentar agir como uma multitarefa variam de humor (por exemplo, do jogo bobo das crianças batendo na cabeça e esfregando o estômago) ao mundano (mascar chiclete enquanto caminha) até completamente perigoso (mensagens de texto enquanto dirige).

Mas somos multitarefas? Podemos realmente realizar duas ações conscientes e intencionais ao mesmo tempo e pensar/raciocinar sobre as duas exatamente no mesmo momento? Nosso nível mais alto de funcionalidade cerebral tem multithreading paralelo?

A resposta pode surpreendê-lo: **provavelmente não.**

Não é exatamente assim que nosso cérebro parece estar configurado. Somos muito mais monotarefas do que muitos de nós (especialmente personalidades do Tipo A!) gostaríamos de admitir. Podemos realmente pensar em apenas uma coisa no mesmo instante.

Não estou falando de todas as nossas funções cerebrais involuntárias, subconscientes e automáticas, como batimentos cardíacos, respiração e pálpebras piscando. Todas essas são tarefas vitais para a nossa vida sustentável, mas não alocamos intencionalmente qualquer poder cerebral a elas. Felizmente, enquanto estamos obcecados em verificar feeds de redes sociais pela 15ª vez em três minutos, nosso cérebro continua em segundo plano (threads!) com todas essas tarefas importantes.

Em vez disso, estamos falando sobre qualquer tarefa que esteja na linha de frente das nossas mentes no momento. Para mim, é estar escrevendo o texto neste livro agora. Estou executando alguma outra função cerebral de nível superior exatamente neste mesmo momento? Não, na verdade não. Eu me distraio com rapidez e facilidade -- algumas dezenas de vezes nesses últimos parágrafos!

Quando *fingimos* multitarefa, como tentar digitar algo ao mesmo tempo em que conversamos com um amigo ou membro da família por telefone, o que realmente estamos fazendo é agir como rápidos alternadores de contexto. Em outras palavras, alternamos entre duas ou mais tarefas em rápida sucessão, progredindo *simultaneamente* em cada tarefa em pedaços pequenos e rápidos. Fazemos isso tão velozmente que, para o mundo exterior, parece que estamos realizando essas coisas *em paralelo*.

Isso parece como concorrência assíncrona de eventos (semelhante ao que acontece em JS) para você?! Se não, volte e leia o Capítulo 1 novamente!

Na verdade, uma maneira de simplificar (ou seja, abusar) o mundo massivamente complexo da neurologia em algo que eu possa remotamente esperar discutir aqui é que nosso cérebro funciona como a fila de loop de eventos.

Se você pensar em cada letra (ou palavra) que digito como um único evento assíncrono, apenas nesta frase existem várias dezenas de oportunidades para o meu cérebro ser interrompido por outro evento, como pelos meus sentidos, ou mesmo apenas pelos meus pensamentos aleatórios.

Eu não sou interrompido e puxado para outro "processo" em todas as oportunidades que eu poderia ter (felizmente - ou esse livro nunca seria escrito!). Mas isso acontece frequente o suficiente para que eu sinta que meu próprio cérebro está quase constantemente mudando para vários contextos diferentes (também conhecidos como "processos"). E é muito parecido com o que o mecanismo JS provavelmente sentiria.

### Fazendo Versus Planejando

OK, então nossos cérebros podem ser pensados como operando na fila do loop do eventos de thread única, assim como o mecanismo JS. Isso soa como uma boa combinação.

Mas precisamos ter mais nuances do que isso em nossa análise. Há uma grande e observável diferença entre como planejamos várias tarefas e como nosso cérebro realmente as opera.

Novamente, voltando à redação deste texto como minha metáfora. O esboço grosseiro do meu plano aqui é continuar escrevendo e escrevendo, passando sequencialmente por um conjunto de pontos que ordenei em meus pensamentos. Não pretendo ter nenhuma interrupção ou atividade não linear neste trabalho. Mas mesmo assim, meu cérebro está fazendo rodízio o tempo todo.

Embora em um nível operacional nossos cérebros sejam assíncronos, parecemos planejar tarefas de maneira sequencial e síncrona. "Eu preciso ir à loja, comprar um pouco de leite e depois deixar minha lavagem a seco."

Você notará que esse pensamento (planejamento) de alto nível não parece muito com eventos assíncronos em sua formulação. De fato, é meio raro pensarmos deliberadamente apenas em termos de eventos. Em vez disso, planejamos as coisas com cuidado, sequencialmente (A então B e depois C), e assumimos, de certa forma, uma espécie de bloqueio temporal que força B a esperar A e C a esperar B.

Quando desenvolvedores escrevem código, eles estão planejando a execução de um conjunto de ações. Se eles são bons em ser desenvolvedores, estão **planejando cuidadosamente**. "Eu preciso definir `z` para o valor de `x` e depois `x` para o valor de `y`," e assim por diante.

Quando escrevemos código síncrono, declaração por declaração, ele funciona muito como nossa lista de tarefas a fazer:

```js
// troca `x` por `y` (através da varável temporária `z`)
z = x;
x = y;
y = z;
```

Essas três instruções de atribuição são síncronas, portanto `x = y` espera que `z = x` termine e `y = z`, por sua vez, espera que `x = y` termine. Outra maneira de dizer é que essas três instruções são temporariamente obrigadas a executar em uma determinada ordem, uma logo após a outra. Felizmente, não precisamos nos preocupar com detalhes de eventos assíncronos aqui. Se o fizermos, o código fica muito mais complexo, rapidamente!

Portanto, se o planejamento do cérebro síncrono mapeia bem as instruções de código síncrono, quão bem nossos cérebros se saem planejando o código assíncrono?

Acontece que a maneira como expressamos assincronia (com callbacks) em nosso código não é muito boa para essa maneira de planejar do cérebro síncrono.

Você pode realmente imaginar ter uma linha de pensamento que planeje suas tarefas assim?

> "Eu preciso ir à loja, mas no caminho tenho certeza de que vou receber um telefonema, então 'Oi, mãe', e enquanto ela começa a falar, procurarei o endereço da loja no GPS, mas isso levará um segundo para carregar, então eu vou desligar o rádio para ouvir melhor mamãe, depois vou perceber que esqueci de vestir uma jaqueta e está frio lá fora, mas não importa, continuo dirigindo e conversando com mamãe e, em seguida, o toque do cinto de segurança me lembra de colocá-lo, então 'Sim, mãe, estou usando meu cinto de segurança, sempre uso!'. Ah, finalmente o GPS recebeu as instruções, agora..."

Por mais ridículo que pareça uma formulação de como planejamos nosso dia e pensamos sobre o que fazer e em que ordem, é exatamente dessa forma como nosso cérebro opera em um nível funcional. Lembre-se, isso não é multitarefa, é apenas uma mudança rápida de contexto.

A razão pela qual é difícil para nós, como desenvolvedores, escrever código assíncrono, especialmente quando tudo o que temos são callback, é que o fluxo de pensamento/planejamento consciente não é natural para a maioria de nós.

Nós pensamos em termos sequenciais, mas as ferramentas disponíveis no código (callbacks) não são expressas de uma maneira sequencial uma vez que passamos de síncrono para assíncrono.

E é por **isso** que é tão difícil escrever e raciocinar com precisão sobre o código JS assíncrono com callbacks: porque não é assim que o nosso planejamento cerebral funciona.

**Nota:** A única coisa pior do que não saber por que algum código quebra é, primeiramente, não saber por que ele funcionou! É a mentalidade clássica do "castelo de cartas": "funciona, mas não sei por quê, então ninguém toca!" Você pode ter ouvido: "O inferno são os outros" (Sartre), e o meme da reviravolta do programador: "O inferno é o código de outras pessoas". Eu acredito verdadeiramente que: "O inferno é não entender meu próprio código". E os callbacks são um dos principais culpados.

### Callbacks Aninhados/Encadeados

Considere:

```js
listen( "click", function handler(evt){
	setTimeout( function request(){
		ajax( "http://some.url.1", function response(text){
			if (text == "hello") {
				handler();
			}
			else if (text == "world") {
				request();
			}
		} );
	}, 500) ;
} );
```

Há uma boa chance de um código como esse ser reconhecível por você. Temos uma cadeia de três funções aninhadas, cada uma representando uma etapa em uma série assíncrona (tarefa, "processo").

Esse tipo de código costuma ser chamado de "callback hell" e às vezes também conhecido como "pirâmide da desgraça" (por sua forma triangular voltada para o lado devido ao recuo aninhado).

Mas "callback hell" na verdade não tem quase nada a ver com o aninhamento/recuo. É um problema muito mais profundo do que isso. Veremos como e porque à medida que continuarmos no restante deste capítulo.

Primeiro, aguardamos o evento de "click", depois esperamos o temporizador disparar e, em seguida, aguardamos a resposta da requisação Ajax chegar, momento em que tudo isso pode acontecer novamente.

À primeira vista, esse código parece mapear sua assincronia naturalmente para o planejamento sequencial do cérebro.

Primeiro (*agora*), nós:

```js
listen( "..", function handler(..){
	// ..
} );
```

Depois, *mais tarde*, nós:

```js
setTimeout( function request(..){
	// ..
}, 500) ;
```

Depois, ainda *mais tarde*, nós:

```js
ajax( "..", function response(..){
	// ..
} );
```

E finalmente (definitivamente *mais tarde*), nós:

```js
if ( .. ) {
	// ..
}
else ..
```

Mas há vários problemas em raciocinar sobre esse código linearmente dessa maneira.

Primeiramente, é um equívoco do exemplo que nossos passos estejam em linhas subsequentes (1, 2, 3 e 4...). Em programas JS assíncronos reais, geralmente há muito mais ruído bagunçando as coisas, ruído que temos que manobrar habilmente em nossos cérebros conforme saltamos de uma função para a próxima. Compreender o fluxo assíncrono em tal código carregado de callbacks não é impossível, mas certamente não é natural ou fácil, mesmo com muita prática.

Mas também, há algo pior que não está evidente apenas nesse exemplo de código. Deixe-me criar outro cenário (pseudocódigo) para ilustrá-lo:

```js
doA( function(){
	doB();

	doC( function(){
		doD();
	} )

	doE();
} );

doF();
```

Embora os mais experientes identificarão a verdadeira ordem das operações aqui, eu aposto que ela é mais do que uma pequena confusão à primeira vista e custa alguns ciclos mentais combinados para chegar na resposta certa. As operações acontecerão nesta ordem:

* `doA()`
* `doF()`
* `doB()`
* `doC()`
* `doE()`
* `doD()`

Você acertou na primeira vez que olhou o código?

OK, alguns de vocês estão pensando que fui injusto na nomeação das minhas funções, para fazer vocês se perderem intencionalmente. Juro que estava apenas nomeando na ordem em que aparecem, de cima para baixo. Mas deixe-me tentar novamente:

```js
doA( function(){
	doC();

	doD( function(){
		doF();
	} )

	doE();
} );

doB();
```

Agora, eu as nomeei alfabeticamente em ordem de execução real. Mas eu ainda aposto, que mesmo agora com experiência nesse cenário, ordenar as operações na ordem `A -> B -> C -> D -> E -> F` não é natural para muitos ou nenhum de vocês leitores. Certamente seus olhos saltaram várias vezes para cima e para baixo no trecho de código, certo?

Mas mesmo que tudo isso seja natural para você, há ainda mais um perigo que pode causar estragos. Você consegue identificar qual é?

E se `doA(..)` ou `doD(..)` não forem realmente assíncronas, da maneira que obviamente assumimos que fossem? Oh oh, agora a ordem é diferente. Se ambas forem síncronas (e talvez apenas algumas vezes, dependendo das condições do programa no momento), a ordem agora é `A -> C -> D -> F -> E -> B`.

Esse som que você acabou de ouvir levemente ao fundo são os suspiros de milhares de desenvolvedores JS que tiveram um momento de frustração.

O aninhamento é o problema? É isso que torna tão difícil rastrear o fluxo assíncrono? Isso faz parte, certamente.

Mas deixe-me reescrever o exemplo anterior com evento/timeout/Ajax aninhados sem usar aninhamento:

```js
listen( "click", handler );

function handler() {
	setTimeout( request, 500 );
}

function request(){
	ajax( "http://some.url.1", response );
}

function response(text){
	if (text == "hello") {
		handler();
	}
	else if (text == "world") {
		request();
	}
}
```

Essa formulação do código não é tão identificável quanto aos problemas de aninhamento/indentação de sua forma anterior e, ainda assim, é tão suscetível ao "callback hell". Por quê?

À medida que raciocinamos linearmente (sequencialmente) sobre esse código, temos que pular de uma função para a próxima e para a próxima, saltando por toda a base do código para "ver" o fluxo da sequência. E lembre-se, esse é um código simplificado na melhor das hipóteses. Todos nós sabemos que as bases de código de programas JS assíncronos reais costumam ser fantasticamente mais confusas, o que torna essas ordens de magnitude de raciocínio mais difíceis.

Outra coisa a observar: para vincular as etapas 2, 3 e 4 a fim de que ocorram em sucessão, a única possibilidade que os callbacks nos fornecem é definir fixamente a etapa 2 na etapa 1, a etapa 3 na etapa 2, a etapa 4 na etapa 3, e assim por diante. A codificação não é necessariamente uma coisa ruim, se realmente for uma condição fixa que a etapa 2 sempre deve levar à etapa 3.

Mas forçar o código dessa maneira definitivamente torna o código um pouco mais frágil, pois não leva em consideração nada de errado que possa causar um desvio na progressão das etapas. Por exemplo, se a etapa 2 falhar, a etapa 3 nunca será alcançada, nem a etapa 2 tentará novamente ou moverá para um fluxo alternativo de tratamento de erros e assim por diante.

Todos esses problemas são coisas que você *pode* codificar manualmente em cada etapa, mas esse código costuma ser muito repetitivo e não pode ser reutilizado em outras etapas ou em outros fluxos assíncronos em seu programa.

Mesmo que nossos cérebros possam planejar uma série de tarefas de uma forma sequencial (isso, depois isso, então isso), a natureza de eventos da nossa operação cerebral torna a recuperação/retentativa/bifurcação do controle de fluxo quase sem esforço. Se você está fazendo compras e percebe que deixou a lista em casa, o dia não termina porque você não planejou isso com antecedência. Seu cérebro contorna esse contratempo facilmente: você vai para casa, pega a lista e volta direto para a loja.

Mas a natureza frágil de callbacks codificados manualmente (mesmo com tratamento de erros codificados permanentemente) é com frequência muito menos elegante. Depois que você acaba especificando (também conhecido como pré-planejamento) todas as várias eventualidades/caminhos, o código se torna tão complicado que é difícil mantê-lo ou atualizá-lo.

**Isso** é o que se trata o "callback hell"! O aninhamento/indentação são basicamente um espetáculo à parte, uma pista falsa.

E como se isso não bastasse, ainda nem tocamos no que acontece quando duas ou mais cadeias dessas continuações de callback estão acontecendo *simultaneamente*, ou quando a terceira etapa se ramifica em callbacks "paralelos" com portas ou travas, ou... oh céus!, meu cérebro dói, e o seu!?

Você está entendendo a ideia aqui de que nosso cérebro, que se comporta de maneira sequencial e bloqueante, simplesmente não mapeiam bem código assíncrono orientado a callback? Esta é a primeira grande deficiência a ressaltar sobre callbacks: eles expressam assincronia em código de maneira que nosso cérebro luta apenas para mantê-lo de forma síncrona (trocadilho intencional!).

## Trust Issues

The mismatch between sequential brain planning and callback-driven async JS code is only part of the problem with callbacks. There's something much deeper to be concerned about.

Let's once again revisit the notion of a callback function as the continuation (aka the second half) of our program:

```js
// A
ajax( "..", function(..){
	// C
} );
// B
```

`// A` and `// B` happen *now*, under the direct control of the main JS program. But `// C` gets deferred to happen *later*, and under the control of another party -- in this case, the `ajax(..)` function. In a basic sense, that sort of hand-off of control doesn't regularly cause lots of problems for programs.

But don't be fooled by its infrequency that this control switch isn't a big deal. In fact, it's one of the worst (and yet most subtle) problems about callback-driven design. It revolves around the idea that sometimes `ajax(..)` (i.e., the "party" you hand your callback continuation to) is not a function that you wrote, or that you directly control. Many times it's a utility provided by some third party.

We call this "inversion of control," when you take part of your program and give over control of its execution to another third party. There's an unspoken "contract" that exists between your code and the third-party utility -- a set of things you expect to be maintained.

### Tale of Five Callbacks

It might not be terribly obvious why this is such a big deal. Let me construct an exaggerated scenario to illustrate the hazards of trust at play.

Imagine you're a developer tasked with building out an ecommerce checkout system for a site that sells expensive TVs. You already have all the various pages of the checkout system built out just fine. On the last page, when the user clicks "confirm" to buy the TV, you need to call a third-party function (provided say by some analytics tracking company) so that the sale can be tracked.

You notice that they've provided what looks like an async tracking utility, probably for the sake of performance best practices, which means you need to pass in a callback function. In this continuation that you pass in, you will have the final code that charges the customer's credit card and displays the thank you page.

This code might look like:

```js
analytics.trackPurchase( purchaseData, function(){
	chargeCreditCard();
	displayThankyouPage();
} );
```

Easy enough, right? You write the code, test it, everything works, and you deploy to production. Everyone's happy!

Six months go by and no issues. You've almost forgotten you even wrote that code. One morning, you're at a coffee shop before work, casually enjoying your latte, when you get a panicked call from your boss insisting you drop the coffee and rush into work right away.

When you arrive, you find out that a high-profile customer has had his credit card charged five times for the same TV, and he's understandably upset. Customer service has already issued an apology and processed a refund. But your boss demands to know how this could possibly have happened. "Don't we have tests for stuff like this!?"

You don't even remember the code you wrote. But you dig back in and start trying to find out what could have gone awry.

After digging through some logs, you come to the conclusion that the only explanation is that the analytics utility somehow, for some reason, called your callback five times instead of once. Nothing in their documentation mentions anything about this.

Frustrated, you contact customer support, who of course is as astonished as you are. They agree to escalate it to their developers, and promise to get back to you. The next day, you receive a lengthy email explaining what they found, which you promptly forward to your boss.

Apparently, the developers at the analytics company had been working on some experimental code that, under certain conditions, would retry the provided callback once per second, for five seconds, before failing with a timeout. They had never intended to push that into production, but somehow they did, and they're totally embarrassed and apologetic. They go into plenty of detail about how they've identified the breakdown and what they'll do to ensure it never happens again. Yadda, yadda.

What's next?

You talk it over with your boss, but he's not feeling particularly comfortable with the state of things. He insists, and you reluctantly agree, that you can't trust *them* anymore (that's what bit you), and that you'll need to figure out how to protect the checkout code from such a vulnerability again.

After some tinkering, you implement some simple ad hoc code like the following, which the team seems happy with:

```js
var tracked = false;

analytics.trackPurchase( purchaseData, function(){
	if (!tracked) {
		tracked = true;
		chargeCreditCard();
		displayThankyouPage();
	}
} );
```

**Note:** This should look familiar to you from Chapter 1, because we're essentially creating a latch to handle if there happen to be multiple concurrent invocations of our callback.

But then one of your QA engineers asks, "what happens if they never call the callback?" Oops. Neither of you had thought about that.

You begin to chase down the rabbit hole, and think of all the possible things that could go wrong with them calling your callback. Here's roughly the list you come up with of ways the analytics utility could misbehave:

* Call the callback too early (before it's been tracked)
* Call the callback too late (or never)
* Call the callback too few or too many times (like the problem you encountered!)
* Fail to pass along any necessary environment/parameters to your callback
* Swallow any errors/exceptions that may happen
* ...

That should feel like a troubling list, because it is. You're probably slowly starting to realize that you're going to have to invent an awful lot of ad hoc logic **in each and every single callback** that's passed to a utility you're not positive you can trust.

Now you realize a bit more completely just how hellish "callback hell" is.

### Not Just Others' Code

Some of you may be skeptical at this point whether this is as big a deal as I'm making it out to be. Perhaps you don't interact with truly third-party utilities much if at all. Perhaps you use versioned APIs or self-host such libraries, so that its behavior can't be changed out from underneath you.

So, contemplate this: can you even *really* trust utilities that you do theoretically control (in your own code base)?

Think of it this way: most of us agree that at least to some extent we should build our own internal functions with some defensive checks on the input parameters, to reduce/prevent unexpected issues.

Overly trusting of input:
```js
function addNumbers(x,y) {
	// + is overloaded with coercion to also be
	// string concatenation, so this operation
	// isn't strictly safe depending on what's
	// passed in.
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// "2121"
```

Defensive against untrusted input:
```js
function addNumbers(x,y) {
	// ensure numerical input
	if (typeof x != "number" || typeof y != "number") {
		throw Error( "Bad parameters" );
	}

	// if we get here, + will safely do numeric addition
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// Error: "Bad parameters"
```

Or perhaps still safe but friendlier:
```js
function addNumbers(x,y) {
	// ensure numerical input
	x = Number( x );
	y = Number( y );

	// + will safely do numeric addition
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// 42
```

However you go about it, these sorts of checks/normalizations are fairly common on function inputs, even with code we theoretically entirely trust. In a crude sort of way, it's like the programming equivalent of the geopolitical principle of "Trust But Verify."

So, doesn't it stand to reason that we should do the same thing about composition of async function callbacks, not just with truly external code but even with code we know is generally "under our own control"? **Of course we should.**

But callbacks don't really offer anything to assist us. We have to construct all that machinery ourselves, and it often ends up being a lot of boilerplate/overhead that we repeat for every single async callback.

The most troublesome problem with callbacks is *inversion of control* leading to a complete breakdown along all those trust lines.

If you have code that uses callbacks, especially but not exclusively with third-party utilities, and you're not already applying some sort of mitigation logic for all these *inversion of control* trust issues, your code *has* bugs in it right now even though they may not have bitten you yet. Latent bugs are still bugs.

Hell indeed.

## Trying to Save Callbacks

There are several variations of callback design that have attempted to address some (not all!) of the trust issues we've just looked at. It's a valiant, but doomed, effort to save the callback pattern from imploding on itself.

For example, regarding more graceful error handling, some API designs provide for split callbacks (one for the success notification, one for the error notification):

```js
function success(data) {
	console.log( data );
}

function failure(err) {
	console.error( err );
}

ajax( "http://some.url.1", success, failure );
```

In APIs of this design, often the `failure()` error handler is optional, and if not provided it will be assumed you want the errors swallowed. Ugh.

**Note:** This split-callback design is what the ES6 Promise API uses. We'll cover ES6 Promises in much more detail in the next chapter.

Another common callback pattern is called "error-first style" (sometimes called "Node style," as it's also the convention used across nearly all Node.js APIs), where the first argument of a single callback is reserved for an error object (if any). If success, this argument will be empty/falsy (and any subsequent arguments will be the success data), but if an error result is being signaled, the first argument is set/truthy (and usually nothing else is passed):

```js
function response(err,data) {
	// error?
	if (err) {
		console.error( err );
	}
	// otherwise, assume success
	else {
		console.log( data );
	}
}

ajax( "http://some.url.1", response );
```

In both of these cases, several things should be observed.

First, it has not really resolved the majority of trust issues like it may appear. There's nothing about either callback that prevents or filters unwanted repeated invocations. Moreover, things are worse now, because you may get both success and error signals, or neither, and you still have to code around either of those conditions.

Also, don't miss the fact that while it's a standard pattern you can employ, it's definitely more verbose and boilerplate-ish without much reuse, so you're going to get weary of typing all that out for every single callback in your application.

What about the trust issue of never being called? If this is a concern (and it probably should be!), you likely will need to set up a timeout that cancels the event. You could make a utility (proof-of-concept only shown) to help you with that:

```js
function timeoutify(fn,delay) {
	var intv = setTimeout( function(){
			intv = null;
			fn( new Error( "Timeout!" ) );
		}, delay )
	;

	return function() {
		// timeout hasn't happened yet?
		if (intv) {
			clearTimeout( intv );
			fn.apply( this, [ null ].concat( [].slice.call( arguments ) ) );
		}
	};
}
```

Here's how you use it:

```js
// using "error-first style" callback design
function foo(err,data) {
	if (err) {
		console.error( err );
	}
	else {
		console.log( data );
	}
}

ajax( "http://some.url.1", timeoutify( foo, 500 ) );
```

Another trust issue is being called "too early." In application-specific terms, this may actually involve being called before some critical task is complete. But more generally, the problem is evident in utilities that can either invoke the callback you provide *now* (synchronously), or *later* (asynchronously).

This nondeterminism around the sync-or-async behavior is almost always going to lead to very difficult to track down bugs. In some circles, the fictional insanity-inducing monster named Zalgo is used to describe the sync/async nightmares. "Don't release Zalgo!" is a common cry, and it leads to very sound advice: always invoke callbacks asynchronously, even if that's "right away" on the next turn of the event loop, so that all callbacks are predictably async.

**Note:** For more information on Zalgo, see Oren Golan's "Don't Release Zalgo!" (https://github.com/oren/oren.github.io/blob/master/posts/zalgo.md) and Isaac Z. Schlueter's "Designing APIs for Asynchrony" (http://blog.izs.me/post/59142742143/designing-apis-for-asynchrony).

Consider:

```js
function result(data) {
	console.log( a );
}

var a = 0;

ajax( "..pre-cached-url..", result );
a++;
```

Will this code print `0` (sync callback invocation) or `1` (async callback invocation)? Depends... on the conditions.

You can see just how quickly the unpredictability of Zalgo can threaten any JS program. So the silly-sounding "never release Zalgo" is actually incredibly common and solid advice. Always be asyncing.

What if you don't know whether the API in question will always execute async? You could invent a utility like this `asyncify(..)` proof-of-concept:

```js
function asyncify(fn) {
	var orig_fn = fn,
		intv = setTimeout( function(){
			intv = null;
			if (fn) fn();
		}, 0 )
	;

	fn = null;

	return function() {
		// firing too quickly, before `intv` timer has fired to
		// indicate async turn has passed?
		if (intv) {
			fn = orig_fn.bind.apply(
				orig_fn,
				// add the wrapper's `this` to the `bind(..)`
				// call parameters, as well as currying any
				// passed in parameters
				[this].concat( [].slice.call( arguments ) )
			);
		}
		// already async
		else {
			// invoke original function
			orig_fn.apply( this, arguments );
		}
	};
}
```

You use `asyncify(..)` like this:

```js
function result(data) {
	console.log( a );
}

var a = 0;

ajax( "..pre-cached-url..", asyncify( result ) );
a++;
```

Whether the Ajax request is in the cache and resolves to try to call the callback right away, or must be fetched over the wire and thus complete later asynchronously, this code will always output `1` instead of `0` -- `result(..)` cannot help but be invoked asynchronously, which means the `a++` has a chance to run before `result(..)` does.

Yay, another trust issued "solved"! But it's inefficient, and yet again more bloated boilerplate to weigh your project down.

That's just the story, over and over again, with callbacks. They can do pretty much anything you want, but you have to be willing to work hard to get it, and oftentimes this effort is much more than you can or should spend on such code reasoning.

You might find yourself wishing for built-in APIs or other language mechanics to address these issues. Finally ES6 has arrived on the scene with some great answers, so keep reading!

## Review

Callbacks are the fundamental unit of asynchrony in JS. But they're not enough for the evolving landscape of async programming as JS matures.

First, our brains plan things out in sequential, blocking, single-threaded semantic ways, but callbacks express asynchronous flow in a rather nonlinear, nonsequential way, which makes reasoning properly about such code much harder. Bad to reason about code is bad code that leads to bad bugs.

We need a way to express asynchrony in a more synchronous, sequential, blocking manner, just like our brains do.

Second, and more importantly, callbacks suffer from *inversion of control* in that they implicitly give control over to another party (often a third-party utility not in your control!) to invoke the *continuation* of your program. This control transfer leads us to a troubling list of trust issues, such as whether the callback is called more times than we expect.

Inventing ad hoc logic to solve these trust issues is possible, but it's more difficult than it should be, and it produces clunkier and harder to maintain code, as well as code that is likely insufficiently protected from these hazards until you get visibly bitten by the bugs.

We need a generalized solution to **all of the trust issues**, one that can be reused for as many callbacks as we create without all the extra boilerplate overhead.

We need something better than callbacks. They've served us well to this point, but the *future* of JavaScript demands more sophisticated and capable async patterns. The subsequent chapters in this book will dive into those emerging evolutions.
