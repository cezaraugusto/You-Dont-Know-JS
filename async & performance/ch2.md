# You Don't Know JS: Async & Performance
# Capítulo 2: Callbacks

No Capítulo 1, nós exploramos a terminologia e conceitos acerca da programação assíncrona no JavaScript. Nosso foco foi entender a fila de loop de eventos mono-thread que guia todos os "eventos" (invocação assíncrona de funções). Também exploramos diversas formas que padrões de concorrência explicam as relações (se houver alguma!) entre cadeias de eventos *executadas simultaneamente*, ou "processos" (tarefas, chamadas de funções, etc.).

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

Embora a segunda versão seja mais precisa, as duas são deficientes em explicar esse código de maneira a conectar nossos cérebros ao código e o código ao motor JS. A desconexão é sutil e monumental e está no cerne de compreender as deficiências das callbacks como expressão e gerenciamento assíncronos.

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

Eu não sou interrompido e puxado para outro "processo" em todas as oportunidades que eu poderia ter (felizmente - ou esse livro nunca seria escrito!). Mas isso acontece frequente o suficiente para que eu sinta que meu próprio cérebro está quase constantemente mudando para vários contextos diferentes (também conhecidos como "processos"). E é muito parecido com o que o motor JS provavelmente sentiria.

### Fazendo Versus Planejando

OK, então nossos cérebros podem ser pensados como operando na fila do loop do eventos de thread única, assim como o motor JS. Isso soa como uma boa combinação.

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

## Problemas de Confiança

A incompatibilidade entre o planejamento sequencial do cérebro e o código JS assíncrono orientado a callbacks é apenas parte do problema com callbacks. Há algo muito mais profundo com que se preocupar.

Vamos mais uma vez revisitar a noção de uma função callback como a continuação (também conhecida como a segunda metade) do nosso programa:

```js
// A
ajax( "..", function(..){
	// C
} );
// B
```

`// A` e `// B` acontecem *agora*, sob o controle direto do programa JS principal. Mas `// C` é adiado para acontecer *depois*, e sob o controle de outra parte -- neste caso, a função `ajax(..)`. Em um sentido básico, esse tipo de transferência de controle não costuma causar muitos problemas para os programas.

Mas não se deixe enganar por sua infrequência, achando que essa troca de controle não é grande coisa. Na verdade, é um dos piores (e ainda assim mais sutis) problemas do design orientado a callbacks. Ele gira em torno da ideia de que, às vezes, `ajax(..)` (ou seja, a "parte" para quem você entrega a continuação do seu callback) não é uma função que você escreveu, ou que você controla diretamente. Muitas vezes é um utilitário fornecido por algum terceiro.

Chamamos isso de "inversão de controle", quando você pega parte do seu programa e entrega o controle de sua execução a outra parte terceira. Existe um "contrato" tácito entre o seu código e o utilitário de terceiros -- um conjunto de coisas que você espera que sejam mantidas.

### A História dos Cinco Callbacks

Pode não ser terrivelmente óbvio por que isso é tão importante. Deixe-me construir um cenário exagerado para ilustrar os riscos de confiança em jogo.

Imagine que você é um desenvolvedor encarregado de construir um sistema de checkout de comércio eletrônico para um site que vende TVs caras. Você já tem todas as várias páginas do sistema de checkout construídas tranquilamente. Na última página, quando o usuário clica em "confirmar" para comprar a TV, você precisa chamar uma função de terceiros (fornecida, digamos, por alguma empresa de rastreamento de analytics) para que a venda possa ser rastreada.

Você percebe que eles forneceram o que parece ser um utilitário de rastreamento assíncrono, provavelmente por causa das melhores práticas de performance, o que significa que você precisa passar uma função callback. Nessa continuação que você passa, você terá o código final que cobra o cartão de crédito do cliente e exibe a página de agradecimento.

Esse código poderia se parecer com:

```js
analytics.trackPurchase( purchaseData, function(){
	chargeCreditCard();
	displayThankyouPage();
} );
```

Fácil o suficiente, certo? Você escreve o código, testa, tudo funciona, e você faz o deploy em produção. Todos felizes!

Seis meses se passam e nenhum problema. Você quase esqueceu que escreveu esse código. Uma manhã, você está em uma cafeteria antes do trabalho, despreocupadamente apreciando seu latte, quando recebe uma ligação em pânico do seu chefe insistindo que você largue o café e corra para o trabalho imediatamente.

Quando você chega, descobre que um cliente de alto perfil teve seu cartão de crédito cobrado cinco vezes pela mesma TV, e ele está compreensivelmente irritado. O atendimento ao cliente já emitiu um pedido de desculpas e processou um reembolso. Mas seu chefe exige saber como isso poderia ter acontecido. "Não temos testes para coisas assim!?"

Você nem se lembra do código que escreveu. Mas você volta a investigar e começa a tentar descobrir o que pode ter dado errado.

Depois de vasculhar alguns logs, você chega à conclusão de que a única explicação é que o utilitário de analytics, de alguma forma, por algum motivo, chamou seu callback cinco vezes em vez de uma. Nada na documentação deles menciona nada sobre isso.

Frustrado, você entra em contato com o suporte ao cliente, que, é claro, está tão atônito quanto você. Eles concordam em escalar o caso para os desenvolvedores deles e prometem retornar. No dia seguinte, você recebe um longo e-mail explicando o que eles encontraram, que você prontamente encaminha ao seu chefe.

Aparentemente, os desenvolvedores da empresa de analytics estavam trabalhando em algum código experimental que, sob certas condições, tentaria novamente o callback fornecido uma vez por segundo, por cinco segundos, antes de falhar com um timeout. Eles nunca tiveram a intenção de enviar isso para produção, mas de alguma forma o fizeram, e estão totalmente envergonhados e arrependidos. Eles entram em muitos detalhes sobre como identificaram a falha e o que farão para garantir que isso nunca aconteça de novo. Blá, blá.

E agora?

Você conversa com seu chefe, mas ele não está se sentindo particularmente confortável com a situação. Ele insiste, e você relutantemente concorda, que você não pode confiar mais *neles* (foi isso que te prejudicou), e que você precisará descobrir como proteger o código de checkout de uma vulnerabilidade dessas novamente.

Depois de algumas tentativas, você implementa algum código ad hoc simples como o seguinte, com o qual a equipe parece satisfeita:

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

**Nota:** Isso deve lhe parecer familiar do Capítulo 1, porque estamos essencialmente criando uma trava para lidar com a possibilidade de haver múltiplas invocações concorrentes do nosso callback.

Mas então um dos seus engenheiros de QA pergunta: "o que acontece se eles nunca chamarem o callback?" Ops. Nenhum de vocês havia pensado nisso.

Você começa a perseguir a toca do coelho e pensa em todas as coisas possíveis que poderiam dar errado na forma como eles chamam o seu callback. Aqui está, em linhas gerais, a lista que você elabora de maneiras pelas quais o utilitário de analytics poderia se comportar mal:

* Chamar o callback cedo demais (antes de ter sido rastreado)
* Chamar o callback tarde demais (ou nunca)
* Chamar o callback poucas ou muitas vezes (como o problema que você enfrentou!)
* Não passar adiante qualquer ambiente/parâmetro necessário ao seu callback
* Engolir quaisquer erros/exceções que possam acontecer
* ...

Isso deve parecer uma lista preocupante, porque é. Você provavelmente está começando a perceber lentamente que vai ter que inventar uma enorme quantidade de lógica ad hoc **em cada um e todos os callbacks** que são passados a um utilitário no qual você não tem certeza se pode confiar.

Agora você percebe um pouco mais completamente o quão infernal é o "callback hell".

### Não Apenas o Código dos Outros

Alguns de vocês podem estar céticos neste ponto sobre se isso é tão importante quanto estou fazendo parecer. Talvez você não interaja muito, ou nada, com utilitários verdadeiramente de terceiros. Talvez você use APIs versionadas ou hospede você mesmo essas bibliotecas, para que o comportamento delas não possa ser alterado sem o seu conhecimento.

Então, contemple isto: você pode *realmente* confiar em utilitários que teoricamente controla (na sua própria base de código)?

Pense nisso desta forma: a maioria de nós concorda que, pelo menos até certo ponto, deveríamos construir nossas próprias funções internas com algumas verificações defensivas nos parâmetros de entrada, para reduzir/prevenir problemas inesperados.

Confiando demais na entrada:
```js
function addNumbers(x,y) {
	// + é sobrecarregado com coerção para também ser
	// concatenação de strings, então essa operação
	// não é estritamente segura dependendo do que é
	// passado.
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// "2121"
```

Defensivo contra entrada não confiável:
```js
function addNumbers(x,y) {
	// garante entrada numérica
	if (typeof x != "number" || typeof y != "number") {
		throw Error( "Bad parameters" );
	}

	// se chegarmos aqui, + fará com segurança a adição numérica
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// Error: "Bad parameters"
```

Ou talvez ainda seguro, mas mais amigável:
```js
function addNumbers(x,y) {
	// garante entrada numérica
	x = Number( x );
	y = Number( y );

	// + fará com segurança a adição numérica
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// 42
```

De qualquer forma que você faça, esses tipos de verificações/normalizações são bastante comuns nas entradas de funções, mesmo com código no qual teoricamente confiamos inteiramente. De um modo rudimentar, é como o equivalente em programação do princípio geopolítico de "Confie, Mas Verifique".

Então, não é razoável que devêssemos fazer a mesma coisa em relação à composição de callbacks de funções assíncronas, não apenas com código verdadeiramente externo, mas até com código que sabemos estar geralmente "sob o nosso próprio controle"? **É claro que deveríamos.**

Mas callbacks na verdade não oferecem nada para nos ajudar. Temos que construir toda essa maquinaria nós mesmos, e isso frequentemente acaba sendo muito código repetitivo/sobrecarga que repetimos para cada callback assíncrono.

O problema mais perturbador com callbacks é a *inversão de controle* que leva a um colapso completo ao longo de todas essas linhas de confiança.

Se você tem código que usa callbacks, especialmente, mas não exclusivamente, com utilitários de terceiros, e você ainda não está aplicando algum tipo de lógica de mitigação para todos esses problemas de confiança da *inversão de controle*, o seu código *tem* bugs nele agora mesmo, mesmo que eles ainda não tenham te prejudicado. Bugs latentes ainda são bugs.

O inferno, de fato.

## Tentando Salvar os Callbacks

Existem várias variações do design de callbacks que tentaram resolver alguns (não todos!) dos problemas de confiança que acabamos de ver. É um esforço valente, mas fadado ao fracasso, de salvar o padrão de callback de implodir sobre si mesmo.

Por exemplo, em relação a um tratamento de erros mais elegante, alguns designs de API fornecem callbacks divididos (um para a notificação de sucesso, um para a notificação de erro):

```js
function success(data) {
	console.log( data );
}

function failure(err) {
	console.error( err );
}

ajax( "http://some.url.1", success, failure );
```

Em APIs desse design, frequentemente o tratador de erro `failure()` é opcional e, se não for fornecido, será assumido que você quer que os erros sejam engolidos. Argh.

**Nota:** Esse design de callback dividido é o que a API de Promise do ES6 usa. Abordaremos Promises do ES6 em muito mais detalhe no próximo capítulo.

Outro padrão comum de callback é chamado de "estilo error-first" (às vezes chamado de "estilo Node", já que também é a convenção usada em quase todas as APIs do Node.js), onde o primeiro argumento de um único callback é reservado para um objeto de erro (se houver). Em caso de sucesso, esse argumento será vazio/falsy (e quaisquer argumentos subsequentes serão os dados de sucesso), mas se um resultado de erro estiver sendo sinalizado, o primeiro argumento é definido/truthy (e normalmente nada mais é passado):

```js
function response(err,data) {
	// erro?
	if (err) {
		console.error( err );
	}
	// caso contrário, assume sucesso
	else {
		console.log( data );
	}
}

ajax( "http://some.url.1", response );
```

Em ambos os casos, várias coisas devem ser observadas.

Primeiro, ele na verdade não resolveu a maioria dos problemas de confiança como pode parecer. Não há nada em nenhum dos callbacks que previna ou filtre invocações repetidas indesejadas. Além disso, as coisas estão piores agora, porque você pode receber tanto sinais de sucesso quanto de erro, ou nenhum dos dois, e você ainda tem que codificar em torno de qualquer uma dessas condições.

Além disso, não perca o fato de que, embora seja um padrão padrão que você pode empregar, é definitivamente mais verboso e cheio de código repetitivo sem muita reutilização, então você vai se cansar de digitar tudo isso para cada callback na sua aplicação.

E quanto ao problema de confiança de nunca ser chamado? Se isso é uma preocupação (e provavelmente deveria ser!), você provavelmente precisará configurar um timeout que cancela o evento. Você poderia criar um utilitário (apenas uma prova de conceito é mostrada) para ajudá-lo com isso:

```js
function timeoutify(fn,delay) {
	var intv = setTimeout( function(){
			intv = null;
			fn( new Error( "Timeout!" ) );
		}, delay )
	;

	return function() {
		// o timeout ainda não aconteceu?
		if (intv) {
			clearTimeout( intv );
			fn.apply( this, [ null ].concat( [].slice.call( arguments ) ) );
		}
	};
}
```

Veja como você o usa:

```js
// usando o design de callback "estilo error-first"
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

Outro problema de confiança é ser chamado "cedo demais". Em termos específicos da aplicação, isso pode realmente envolver ser chamado antes que alguma tarefa crítica seja concluída. Mas, de forma mais geral, o problema é evidente em utilitários que podem tanto invocar o callback que você fornece *agora* (de forma síncrona), quanto *depois* (de forma assíncrona).

Esse não determinismo em torno do comportamento síncrono ou assíncrono quase sempre vai levar a bugs muito difíceis de rastrear. Em alguns círculos, o monstro fictício indutor de insanidade chamado Zalgo é usado para descrever os pesadelos de sync/async. "Não solte o Zalgo!" é um grito comum, e leva a um conselho muito sensato: sempre invoque callbacks de forma assíncrona, mesmo que seja "imediatamente" no próximo giro do loop de eventos, para que todos os callbacks sejam previsivelmente assíncronos.

**Nota:** Para mais informações sobre o Zalgo, veja "Don't Release Zalgo!" de Oren Golan (https://github.com/oren/oren.github.io/blob/master/posts/zalgo.md) e "Designing APIs for Asynchrony" de Isaac Z. Schlueter (http://blog.izs.me/post/59142742143/designing-apis-for-asynchrony).

Considere:

```js
function result(data) {
	console.log( a );
}

var a = 0;

ajax( "..pre-cached-url..", result );
a++;
```

Este código imprimirá `0` (invocação síncrona do callback) ou `1` (invocação assíncrona do callback)? Depende... das condições.

Você pode ver quão rapidamente a imprevisibilidade do Zalgo pode ameaçar qualquer programa JS. Então o conselho de som bobo "nunca solte o Zalgo" é, na verdade, incrivelmente comum e sólido. Sempre seja assíncrono.

E se você não souber se a API em questão sempre executará de forma assíncrona? Você poderia inventar um utilitário como esta prova de conceito `asyncify(..)`:

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
		// disparando rápido demais, antes que o timer `intv` tenha
		// disparado para indicar que o giro assíncrono já passou?
		if (intv) {
			fn = orig_fn.bind.apply(
				orig_fn,
				// adiciona o `this` do wrapper aos parâmetros
				// da chamada `bind(..)`, bem como faz currying
				// de quaisquer parâmetros passados
				[this].concat( [].slice.call( arguments ) )
			);
		}
		// já é assíncrono
		else {
			// invoca a função original
			orig_fn.apply( this, arguments );
		}
	};
}
```

Você usa `asyncify(..)` assim:

```js
function result(data) {
	console.log( a );
}

var a = 0;

ajax( "..pre-cached-url..", asyncify( result ) );
a++;
```

Quer a requisição Ajax esteja no cache e resolva tentar chamar o callback imediatamente, quer precise ser buscada pela rede e, portanto, seja concluída mais tarde de forma assíncrona, este código sempre produzirá `1` em vez de `0` -- `result(..)` não tem como deixar de ser invocado de forma assíncrona, o que significa que o `a++` tem a chance de rodar antes de `result(..)`.

Oba, mais um problema de confiança "resolvido"! Mas é ineficiente, e novamente mais código repetitivo inchado para sobrecarregar o seu projeto.

Essa é simplesmente a história, repetidas vezes, com callbacks. Eles podem fazer praticamente qualquer coisa que você quiser, mas você tem que estar disposto a trabalhar duro para consegui-lo, e muitas vezes esse esforço é muito maior do que você pode ou deveria gastar raciocinando sobre tal código.

Você pode se ver desejando APIs nativas ou outros mecanismos da linguagem para resolver esses problemas. Finalmente o ES6 chegou em cena com algumas ótimas respostas, então continue lendo!

## Revisão

Callbacks são a unidade fundamental de assincronia em JS. Mas eles não são suficientes para o cenário em evolução da programação assíncrona à medida que o JS amadurece.

Primeiro, nossos cérebros planejam as coisas de maneiras semânticas sequenciais, bloqueantes e de thread única, mas callbacks expressam o fluxo assíncrono de uma forma um tanto não linear e não sequencial, o que torna o raciocínio adequado sobre tal código muito mais difícil. Código difícil de raciocinar é código ruim que leva a bugs ruins.

Precisamos de uma maneira de expressar a assincronia de uma forma mais síncrona, sequencial e bloqueante, exatamente como nossos cérebros fazem.

Segundo, e mais importante, callbacks sofrem de *inversão de controle* pois eles implicitamente entregam o controle a outra parte (frequentemente um utilitário de terceiros fora do seu controle!) para invocar a *continuação* do seu programa. Essa transferência de controle nos leva a uma lista preocupante de problemas de confiança, como se o callback é chamado mais vezes do que esperamos.

Inventar lógica ad hoc para resolver esses problemas de confiança é possível, mas é mais difícil do que deveria ser, e produz código mais desajeitado e mais difícil de manter, bem como código que provavelmente está insuficientemente protegido desses riscos até que você seja visivelmente prejudicado pelos bugs.

Precisamos de uma solução generalizada para **todos os problemas de confiança**, uma que possa ser reutilizada para quantos callbacks criarmos, sem toda a sobrecarga extra de código repetitivo.

Precisamos de algo melhor do que callbacks. Eles nos serviram bem até este ponto, mas o *futuro* do JavaScript exige padrões assíncronos mais sofisticados e capazes. Os capítulos subsequentes deste livro vão mergulhar nessas evoluções emergentes.
