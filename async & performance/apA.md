# You Don't Know JS: Async & Performance
# Apêndice A: Biblioteca *asynquence*

Os capítulos 1 e 2 trouxeram em detalhes padrões típicos da programação assíncrona e como estes se baseiam em callbacks. Mas também vimos que callbacks são fatalmente limitados em termos de capacidade, o que nos levou aos capítulos 3 e 4, com Promises e generators oferecendo uma base muito mais sólida, confiável e compreensível para construção de sua assincronia.

Referenciei minha própria biblioteca assíncrona *asynquence* (http://github.com/getify/asynquence) -- "async" + "sequence" (sequência) = "asynquence" -- diversas vezes neste livro, e agora gostaria de explicar brevemente como ela funciona e por que a forma única com que foi projetada é importante e útil.

No próximo apêndice exploraremos alguns padrões `async` avançados, mas você provavelmente irá querer uma biblioteca para torná-los palatáveis o suficiente para serem úteis. Utilizaremos *asynquence* para expressar estes padrões, portanto você irá querer passar algum tempo por aqui para conhecê-la antes de mais nada.

*asynquence* obviamente não é a única opção para boas implementações `async`; certamente existem ótimas bibliotecas por aí. Mas *asynquence* oferece uma perspectiva única ao combinar o melhor destes padrões em uma única biblioteca e, além disso, foi criada sobre uma única abstração: a sequência (assíncrona).

Minha premissa é que programas JS sofisticados necessitam, com frequência, de porções de diversos padrões assíncronos entrelaçados, e isso normalmente fica a cargo de cada desenvolvedor(a) descobrir. Em vez de incluirmos duas ou mais bibliotecas diferentes que focam em diferentes aspectos da assincronia, *asynquence* unifica-as em uma sequência variada de passos, com uma única biblioteca para aprender e implantar.

Acredito no valor agregado por *asynquence* na facilidade de se obter uma semântica de programação baseada em Promises para controle de fluxo assíncrono e é por isso que focaremos exclusivamente nesta biblioteca aqui.

Para começar, vou explicar os princípios por trás de *asynquence* e então nós ilustraremos o funcionamento de sua API com exemplos de código.

## Sequências, princípio de abstração

A compreensão de *asynquence* inicia com a compreensão de uma abstração fundamental: qualquer série de passos para execução de uma tarefa, sejam eles individualmente síncronos ou assíncronos, podem, coletivamente, ser pensados como uma "sequência". Em outras palavras, uma sequência é um container que representa uma tarefa e é constituído por passos individuais (potencialmente assíncronos) para completá-la.

Cada passo na sequência é controlado internamente por Promises (veja Capítulo 3). Isto é, cada passo que você adiciona à sequência cria implicitamente uma Promise que esta ligada ao seu (antigo) último passo. Por conta da semântica de Promises, cada avanço de passos em uma sequência é assíncrono, mesmo se este passo for completado de forma síncrona.

Além disso, uma sequência sempre avançará linearmente de passo em passo, de modo que o passo 2 sempre vem após o término do passo 1 e assim por diante.

Obviamente, é possível criar uma nova sequência a partir da bifurcação de uma sequência existente, de modo que a nova sequência somente iniciará no momento que a sequência principal atingir o ponto de bifurcação do fluxo. Sequências também podem ser combinadas de várias formas, inclusive incluir uma sequência em outra em algum ponto do fluxo.

Uma sequência é como uma cadeia de Promises. Porém, em uma cadeia de Promises não temos uma "alça" para nos segurarmos que referencie a cadeia por completo. Qualquer Promise para a qual você possua uma referência representa apenas o passo atual na cadeia e mais alguns passos subsequentes. Essencialmente você não pode ter uma referência para uma cadeia de Promises a não ser que você referencie a primeira Promise da cadeia.

Existem muitos casos em que torna-se útil ter esta referência para a sequência como um todo, como em situações de interrupção/cancelamento. Conforme cobrimos extensivamente no Capítulo 3, Promises em si não devem nunca ser canceladas pois isto viola um princípio imperativo fundamental: imutabilidade externa.

Mas sequências não possuem este princípio de imutabilidade por definição, muito pelo fato de não serem enviadas de um lado para o outro como containers de valores futuros que carecem de uma semântica de imutabilidade. Portanto sequências representam um nível de abstração adequado para manipulação de comportamentos relacionados à interrupções/cancelamentos. Sequências *asynquence* podem ser `abort()`adas a qualquer momento e a sequência será interrompida no ponto que estiver e não irá adiante por nenhuma razão.

Existem muitas outras razões para se preferir a sequência como abstração em relação à corrente de Promises para controle de fluxo.

Primeiramente, o processo de encadeamento de Promises é bastante manual -- e pode tornar-se bastante tedioso assim que você começa a criar e encadear Promises por uma faixa muito ampla de seus programas -- e este tédio pode tornar-se improdutivo ao dissuadir o(a) desenvolvedor(a) de utilizar Promises em locais onde seria bastante apropriado.

Abstrações tem por objetivo reduzir repetição de código e tédio, portanto a sequência como abstração é uma boa solução para este problema. Com Promises, seu foco é no passo individual e não se assume que uma corrente será formada. Uma abordagem oposta é tomada no caso das sequências, onde assumimos que esta possuirá mais passos indefinidamente.

A redução de complexidade desta abstração é especialmente poderosa quando começamos a pensar em padrões que utilizam Promises de alta ordem (além de `race([..])` e `all([..])`).

Por exemplo, talvez você queira, no meio de uma sequência, expressar um passo similar a um bloco `try..catch` onde sempre é retornado sucesso, seja pelo sucesso de fato ou pelo envio de um sinal positivo nos casos que apanhamos um erro. Ou talvez você queira expressar um passo que funciona como um loop _retry/until_, onde o mesmo passo ocorre repetidas vezes até que se obtenha sucesso.

Estes tipos de abstrações não são trivialmente expressadas utilizando-se apenas Promises nativas, e aplicá-las em uma cadeia de Promises já existente não é bonito. Mas se você abstrair seu pensamento para uma sequência e considerar um passo como um envólucro de uma Promises, este passo envólucro pode esconder estes detalhes, liberando você para pensar sobre o controle de fluxo de forma mais sensata sem se incomodar com os detalhes.

Em segundo lugar, e talvez o mais importante, pensar em controle de fluxo assíncrono em termos de passos em uma sequência permite que você abstraia detalhes de quais tipos de assincronia são envolvidos em cada passo individualmente. Por baixo dos panos, uma Promise sempre controlará um passo, mas, "por cima dos panos", este passo pode ser visto como um callback de continuidade (o padrão simples), uma Promise real, como um _generator_ em modo run-to-completion, ou... acho que você compreende.

Em terceiro, sequências podem ser alteradas mais facilmente para adaptarem-se a diferentes formas de pensar, como programação baseada em eventos, streams ou reativa. *asynquence* provê um padrão que chamo de "sequências reativas" (as quais cobriremos mais adiante) como uma variação da ideia de "observável reativo" (_reactive observable_) em RxJS ("Reactive Extensions"), que permite que um evento recorrente inicie uma nova sequência a cada ocorrência. Promises são um tiro único, portanto é um pouco estranho expressar assincronia repetitiva apenas com Promises.

Uma outra forma de pensar inverte a capacidade de resolução/controle em um padrão que chamo de "sequências iteráveis". Ao invés de cada passo controlar individualmente e internamente sua completude (e portanto o avanço da sequência), a sequência é invertida de modo que o controle de avanço se dê através de um iterador externo e cada passo na *sequência iterável* apenas responde ao controle `next(..)` do *iterador*.

Vamos explorar todas as diferentes variações na medida que avançarmos por este apêndice, portanto não se preocupe se fomos muito rápidos até o momento.

O mais importante é a ideia de que sequências são uma abstração mais poderosa e sensata para assincronia complexa do que apenas Promises (cadeias de Promises) ou *generators*, e *asynquence* foi projetada para expressar esta abstração com o nível exato de praticidade para tornar a programação assíncrona mais compreensível e prazerosa.

## API *asynquence*

Para começar, a forma com que você cria uma sequência (uma instância *asynquence*) é com a função `ASQ(..)`. Um chamada para `ASQ()` sem parâmetros cria uma sequência inicial vazia, enquanto que se passarmos um ou mais valores ou funções para `ASQ(..)` a sequência é inicializada utilizando cada um de seus argumentos como um passo.

**Nota:** Utilizarei o identificador *asynquence* global para browsers `ASQ` para todos os exemplos de código aqui. Se você incluir *asynquence* através de um sistema de módulos (browser ou server), você certamente pode definir o identificador que preferir que *asynquence* não se importará!

Muitos dos métodos da API discutidos aqui foram construídos no núcleo de *asynquence*, mas outros são providos através da inclusão do pacote de plugins "contrib". Veja a documentação de *asynquence* para identificar se um método é nativo ou se foi definido através de um plugin: http://github.com/getify/asynquence

### Passos

Se uma função representa um passo normal em uma sequência, esta função é invocada recebendo como primeiro parâmetro o callback de continuação e os parâmetros subsequentes são quaisquer mensagens transmitidas pelo passo anterior. O passo não será concluído até que o callback de continuação seja chamado. Assim que chamado, qualquer argumento passado para ele será enviado como mensagem para o próximo passo da sequência.

Para incluir um passo adicional à sequência basta chamar `then(..)` (que possui exatamente a mesma semântica de `ASQ(..)`):

```js
ASQ(
	// passo 1
	function(done){
		setTimeout( function(){
			done( "Hello" );
		}, 100 );
	},
	// passo 2
	function(done,greeting) {
		setTimeout( function(){
			done( greeting + " World" );
		}, 100 );
	}
)
// passo 3
.then( function(done,msg){
	setTimeout( function(){
		done( msg.toUpperCase() );
	}, 100 );
} )
// passo 4
.then( function(done,msg){
	console.log( msg );			// HELLO WORLD
} );
```

**Nota:** Embora o nome `then(..)` seja idêntico ao da API nativa de Promises, este `then(..)` é diferente. Você pode passar quantas funções ou valores quiser para `then(..)` e cada um é recebido como um passo separado. Não existe a semantica de dois callbacks realizado/rejeitado.

Diferentemente das Promises, onde para encadearmos uma Promise na próxima temos que criar e também retornar (`return`) esta Promise no handler de sucesso enviado para `then(..)`. Com *asynquence*, tudo que você precisa fazer é chamar o callback de continuação -- eu sempre o chamo de `done()` mas vocês pode chamá-lo como achar melhor -- e opcionalmente passar para ele mensagens como argumentos.

Cada passo definido por `then(..)` é assumido como assíncrono. Se você tem um passo que é síncrono, você pode chamar `done(..)` imediatamente ou chamar um utilitário mais simples invocando `val(..)`:

```js
// passo 1 (síncrono)
ASQ( function(done){
	done( "Hello" );	// manualmente síncrono
} )
// passo 2 (síncrono)
.val( function(greeting){
	return greeting + " World";
} )
// passo 3 (assíncrono)
.then( function(done,msg){
	setTimeout( function(){
		done( msg.toUpperCase() );
	}, 100 );
} )
// passo 4 (síncrono)
.val( function(msg){
	console.log( msg );
} );
```

Como você pode ver, passos invocados através de `val(..)` não recebem o callback de continuação pois isto é feito internamente para você -- e a lista de parâmetros fica menos bagunçada como resultado! Para enviar uma mensagem ao próximo passo, basta utilizar `return`.

Pense em `val(..)` como a representação de um passo síncrono contendo apenas um valor, o que é útil para operações com valores síncronos, logging e afins.

### Erros

Uma importante diferença de *asynquence* em comparação com Promises se dá no tratamento de erros.

Com Promises, cada Promise (passo) em uma cadeia pode ter seu próprio erro e cada paso subsequente tem a opção de manipulá-lo ou não. A principal razão desta semântica vem (novamente) do foco em Promises como unidades individuais e não como uma cadeia (sequência).

Acredito que, na maior parte do tempo, um erro em uma parte de uma sequência é irrecuperável, portanto os passos subsequentes da sequência são discutíveis e devem ser ignorados. Portanto, por padrão, um erro em qualquer passo de uma sequência passa toda a sequência para um estado de erro e o restante dos passos são ignorados.

Se você *precisa* de um passo onde um erro é recuperável, existem diferentes métodos da API que podem auxiliar, como `try(..)` -- anteriormente mencionado como um tipo de passo `try..catch` -- ou `until(..)` -- um loop de tentativas que fica repetindo o passo até que obtenha sucesso ou que você chame `break()` manualmente dentro do loop. *asynquence* possui também os métodos `pThen(..)` e `pCatch(..)` que funcionam de forma idêntica aos métodos `then(..)` e `catch(..)` de uma Promise (veja o Capítulo 3) para que você possa tratar erros no meio de uma sequência se assim desejar.

O ponto é que você tem ambas opções mas a mais comum na minha experiência é a padrão. Com Promises, para que uma cadeia de passos ignore todos os passos caso um erro ocorra você deve tomar o cuidado de não registrar um handler de rejeição em nenhum dos passos; caso contrário, este erro será desaparece como se fosse tratado e a sequência pode continuar (talvez de forma inesperada). Este tipo de comportamento, quando desejado, é um pouco estranho de se manipular adequada e confiavelmente.

Para registrar um handler de notificação de sequências com erro, *asynquence* provê o método de sequência `or(..)`, o qual possui um alias `onerror(..)`. Você pode chamar este método em qualquer ponto da sequência e você pode registrar quantos handlers achar necessário. Isso torna mais fácil para múltiplos (e diferentes) consumidores saberem se uma sequ%encia falhou ou não; é como se fosse um handler de um evento de erro.

Assim como com Promises, toda exceção JS tornam-se erros da sequência, ou você pode sinalizar um erro na sequência programaticamente:

```js
var sq = ASQ( function(done){
	setTimeout( function(){
		// sinaliza um erro na sequência
		done.fail( "Oops" );
	}, 100 );
} )
.then( function(done){
	// nunca chegará aqui
} )
.or( function(err){
	console.log( err );			// Oops
} )
.then( function(done){
	// não chegará aqui também
} );

// depois

sq.or( function(err){
	console.log( err );			// Oops
} );
```

Outra importante diferença na manipulação de erros de *asynquence* em relação a Promises nativas é o comportamento padrão de "exceções não manipuladas" (_unhandled exceptions_). Como dicutimos massivamente no Capítulo 3, uma Promise rejeitada que não possui um handler de rejeição registrado irá prender silenciosamente (também referido como "engolir") o erro; você deve lembrar-se de sempre finalizar uma corrente com um `catch(..)`.

Em *asynquence* esta suposição é invertida.

Se um erro ocorre em uma sequência e ela **até este momento** não possui um handler de erro registrado, o erro é reportado para o `console`. Em outras palavras, rejeições não manipuladas são, por padrão, reportadas de modo que não sejam engolidas ou perdidas.

Assim que um handler de error for registrado em uma sequência, a sequência para de reportar erros da forma mencionada anteriormente para evitar a duplicação/ruído.

Podem haver, de fato, casos onde você quer criar uma sequência que pode ir para um estado de erro antes de você ter a chance de registrar um handler. Isto não é comum mas pode acontecer de tempos em tempos.

Nestes casos, você pode **optar por não reportar erros desta sequência** chamando `defer()`. Você somente deve fazer isso se você tem certeza que eventualmente irá manipular estes erros:

```js
var sq1 = ASQ( function(done){
	doesnt.Exist();			// vai lançar uma exceção no console
} );

var sq2 = ASQ( function(done){
	doesnt.Exist();			// vai lançar um erro apenas na sequência
} )
// optando por não reportar erros
.defer();

setTimeout( function(){
	sq1.or( function(err){
		console.log( err );	// ReferenceError
	} );

	sq2.or( function(err){
		console.log( err );	// ReferenceError
	} );
}, 100 );

// ReferenceError (from sq1)
```

Esta é uma forma de manipulação de erros melhor do que em Promises por se tratar do Poço do Sucesso e não do Poço da Falha (veja o Capítulo 3).

**Nota:** Se uma sequência é canalizada (ou incluída em) outra sequência -- veja "Combinando Sequências" para uma descrição completa -- então a sequência de origem opta automaticamente por não reportar erros, embora agora a notificação ou não de erros da sequência de destino deva ser considerada.

### Passos paralelos

Nem todos os passos em sua sequência terão apenas uma única tarefa (assíncrona) para executar; alguns precisarão executar múltiplos passos "em paralelo" (ao mesmo tempo). Um passo em uma sequência no qual múltiplos sub-passos são processados ao mesmo tempo é chamado de `gate(..)` -- existe um alias `all(..)` se você preferir -- e é diretamente simétrico ao `Promise.all([..])` nativo.

Se todos os passos em `gate(..)` completam com sucesso, todas as mensagens de sucesso serão passadas para o próximo passo da sequência. Se algum deles gerar um erro, a sequência inteira passa para um estado de erro.

Considere:

```js
ASQ( function(done){
	setTimeout( done, 100 );
} )
.gate(
	function(done){
		setTimeout( function(){
			done( "Hello" );
		}, 100 );
	},
	function(done){
		setTimeout( function(){
			done( "World", "!" );
		}, 100 );
	}
)
.val( function(msg1,msg2){
	console.log( msg1 );	// Hello
	console.log( msg2 );	// [ "World", "!" ]
} );
```

Para ilustrarmos, vamos comparar este exemplo com Promises nativas:

```js
new Promise( function(resolve,reject){
	setTimeout( resolve, 100 );
} )
.then( function(){
	return Promise.all( [
		new Promise( function(resolve,reject){
			setTimeout( function(){
				resolve( "Hello" );
			}, 100 );
		} ),
		new Promise( function(resolve,reject){
			setTimeout( function(){
				// nota: precisamos de um [ ] array aqui
				resolve( [ "World", "!" ] );
			}, 100 );
		} )
	] );
} )
.then( function(msgs){
	console.log( msgs[0] );	// Hello
	console.log( msgs[1] );	// [ "World", "!" ]
} );
```

Eca! Promises necessitam de muita duplicação para expressar o mesmo controle de fluxo assíncrono. Esta é uma boa forma de ilustrar que a API e abstração de *asynquence* tornam a manipulação de Promises muito mais agradáveis. E isso só melhora na medida que a complexidade de sua assincronia aumenta.

#### Variações de passos

Existem diversas variações nos plug-ins `contrib` para o passo `gate(..)` de *asynquence* que podem ser muito úteis:

* `any(..)` é como `gate(..)`, exceto que apenas um segmento deve obter sucesso para darmos prosseguimento à sequência principal.
* `first(..)` é como `any(..)`, exceto que assim que qualquer segmento obtenha sucesso a sequência principal é continuada (ignorando resultados de outros segmentos).
* `race(..)` (simétrico ao `Promise.race([..])`) é como `first(..)`, exceto que a sequência principal prossegue assim que qualquer segmento se completa (seja em caso de sucesso ou falha).
* `last(..)` é como `any(..)`, exceto que apenas o último segmento a completar com sucesso enviará adiante sua(s) mensagem(ns) para a sequência principal.
* `none(..)` é o inverso de `gate(..)`: a sequência principal prossegue apenas se todos os segmentos falharem (tendo as mensagens de erro de todos os segmentos convertidas em mensagens de sucesso e vice versa).

Vamos definir algumas funções auxiliares para tornar a ilustração mais clara:

```js
function success1(done) {
	setTimeout( function(){
		done( 1 );
	}, 100 );
}

function success2(done) {
	setTimeout( function(){
		done( 2 );
	}, 100 );
}

function failure3(done) {
	setTimeout( function(){
		done.fail( 3 );
	}, 100 );
}

function output(msg) {
	console.log( msg );
}
```

Agora vamos demonstrar estas variações do passo `gate(..)`:

```js
ASQ().race(
	failure3,
	success1
)
.or( output );		// 3


ASQ().any(
	success1,
	failure3,
	success2
)
.val( function(){
	var args = [].slice.call( arguments );
	console.log(
		args		// [ 1, undefined, 2 ]
	);
} );


ASQ().first(
	failure3,
	success1,
	success2
)
.val( output );		// 1


ASQ().last(
	failure3,
	success1,
	success2
)
.val( output );		// 2

ASQ().none(
	failure3
)
.val( output )		// 3
.none(
	failure3
	success1
)
.or( output );		// 1
```

Outra variação de passo é `map(..)`, que permite que você mapeie assincronamente valores de um array para valores diferentes, e o passo não completa até que todo mapeamento esteja completo. `map(..)` é muito parecido com `gate(..)`, exceto que recebe os valores iniciais de um array em vez de receber funções separadamente, e também porque você define uma única função callback para operar em cada valor:

```js
function double(x,done) {
	setTimeout( function(){
		done( x * 2 );
	}, 100 );
}

ASQ().map( [1,2,3], double )
.val( output );					// [2,4,6]
```

Além disso, `map(..)` pode receber qualquer um dos seus parâmetros (array ou callback) a partir de mensagens enviadas por passos anteriores:

```js
function plusOne(x,done) {
	setTimeout( function(){
		done( x + 1 );
	}, 100 );
}

ASQ( [1,2,3] )
.map( double )			// recebe a mensagem `[1,2,3]`
.map( plusOne )			// recebe a mensagem `[2,4,6]`
.val( output );			// [3,5,7]
```

Outra variação é `waterfall(..)`, que é como uma mistura do comportamento de acumular mensagens de `gate(..)` com o processamento sequencial de `then(..)`.

Passo 1 é executado e sua mensagem de sucesso é enviada para o passo 2, então ambas mensagens de sucesso são enviadas para o passo 3, e as três mensagens de sucesso são enviadas para o passo 4 e assim por diante, de modo que as mensagens são acumuladas e "descem" pela "cascata" (_waterfall_).

Considere:

```js
function double(done) {
	var args = [].slice.call( arguments, 1 );
	console.log( args );

	setTimeout( function(){
		done( args[args.length - 1] * 2 );
	}, 100 );
}

ASQ( 3 )
.waterfall(
	double,					// [ 3 ]
	double,					// [ 6 ]
	double,					// [ 6, 12 ]
	double					// [ 6, 12, 24 ]
)
.val( function(){
	var args = [].slice.call( arguments );
	console.log( args );	// [ 6, 12, 24, 48 ]
} );
```

Se em qualquer ponto da "cascata" ocorrer um erro, toda sequência imediatamente passa para um estado de erro.

#### Tolerância a erro

Às vezes você quer gerenciar erros no nível dos passos e não necessariamente enviar toda a sequência para um estado de erro. *asynquence* oferece duas variações de passo para estes casos.

`try(..)` tenta executar um passo e, em caso de sucesso, a sequência prossegue normalmente. Mas se o passo falhar, a falha é convertida em uma mensagem de sucesso formatada como `{ catch: .. }` contendo a(s) mensagem(ns) de erro:

```js
ASQ()
.try( success1 )
.val( output )			// 1
.try( failure3 )
.val( output )			// { catch: 3 }
.or( function(err){
	// nunca chega aqui
} );
```

Em vez disso, você poderia configurar um loop de tentativas utilizando `until(..)`, que tenta executar um passo e, se ele falhar, executa o passo novamente no próximo instante (_tick_) do loop de eventos (_event loop_) e assim por diante.

Este loop de tentativas pode continuar indefinidamente, mas se você quiser sair do loop, você pode chamar o método `break()` no callback de continuação, que envia toda sequência principal para um estado de erro:

```js
var count = 0;

ASQ( 3 )
.until( double )
.val( output )					// 6
.until( function(done){
	count++;

	setTimeout( function(){
		if (count < 5) {
			done.fail();
		}
		else {
			// sai do loop de tentativas de `until(..)`
			done.break( "Oops" );
		}
	}, 100 );
} )
.or( output );					// Oops
```

#### Passos no Estilo Promise

Se você preferir ter, embutida na sua sequência, uma semântica no estilo Promise como o `then(..)` e o `catch(..)` das Promises (veja Capítulo 3), você pode usar os plug-ins `pThen` e `pCatch`:

```js
ASQ( 21 )
.pThen( function(msg){
	return msg * 2;
} )
.pThen( output )				// 42
.pThen( function(){
	// lança uma exceção
	doesnt.Exist();
} )
.pCatch( function(err){
	// capturou a exceção (rejeição)
	console.log( err );			// ReferenceError
} )
.val( function(){
	// a sequência principal está de volta
	// a um estado de sucesso porque a exceção
	// anterior foi capturada por
	// `pCatch(..)`
} );
```

`pThen(..)` e `pCatch(..)` são projetados para rodar na sequência, mas se comportam como se fosse uma cadeia de Promises normal. Dessa forma, você pode resolver tanto Promises genuínas quanto sequências *asynquence* a partir do tratador de "cumprimento" passado para `pThen(..)` (veja Capítulo 3).

### Bifurcando Sequências

Um recurso que pode ser bastante útil em relação às Promises é que você pode anexar múltiplos registros de tratadores `then(..)` à mesma promise, efetivamente "bifurcando" o controle de fluxo naquela promise:

```js
var p = Promise.resolve( 21 );

// bifurcação 1 (a partir de `p`)
p.then( function(msg){
	return msg * 2;
} )
.then( function(msg){
	console.log( msg );		// 42
} )

// bifurcação 2 (a partir de `p`)
p.then( function(msg){
	console.log( msg );		// 21
} );
```

A mesma "bifurcação" é fácil em *asynquence* com `fork()`:

```js
var sq = ASQ(..).then(..).then(..);

var sq2 = sq.fork();

// bifurcação 1
sq.then(..)..;

// bifurcação 2
sq2.then(..)..;
```

### Combinando Sequências

O inverso de `fork()`ar: você pode combinar duas sequências incorporando uma na outra, usando o método de instância `seq(..)`:

```js
var sq = ASQ( function(done){
	setTimeout( function(){
		done( "Hello World" );
	}, 200 );
} );

ASQ( function(done){
	setTimeout( done, 100 );
} )
// incorpora a sequência `sq` nesta sequência
.seq( sq )
.val( function(msg){
	console.log( msg );		// Hello World
} )
```

`seq(..)` pode aceitar tanto uma sequência em si, como mostrado aqui, quanto uma função. Se for uma função, espera-se que a função, quando chamada, retorne uma sequência, então o código anterior poderia ter sido feito com:

```js
// ..
.seq( function(){
	return sq;
} )
// ..
```

Além disso, esse passo poderia ter sido realizado, em vez disso, com um `pipe(..)`:

```js
// ..
.then( function(done){
	// canaliza `sq` para o callback de continuação `done`
	sq.pipe( done );
} )
// ..
```

Quando uma sequência é incorporada, tanto o seu fluxo de mensagens de sucesso quanto o seu fluxo de erros são canalizados.

**Nota:** Como mencionado em uma nota anterior, canalizar (manualmente com `pipe(..)` ou automaticamente com `seq(..)`) faz a sequência de origem optar por não relatar erros, mas não afeta o status de relato de erros da sequência de destino.

## Sequências de Valor e de Erro

Se qualquer passo de uma sequência for apenas um valor normal, esse valor é simplesmente mapeado para a mensagem de conclusão daquele passo:

```js
var sq = ASQ( 42 );

sq.val( function(msg){
	console.log( msg );		// 42
} );
```

Se você quiser criar uma sequência que entra automaticamente em erro:

```js
var sq = ASQ.failed( "Oops" );

ASQ()
.seq( sq )
.val( function(msg){
	// não chegará aqui
} )
.or( function(err){
	console.log( err );		// Oops
} );
```

Você também pode querer criar automaticamente uma sequência de valor-atrasado ou de erro-atrasado. Usando os plug-ins contrib `after` e `failAfter`, isso é fácil:

```js
var sq1 = ASQ.after( 100, "Hello", "World" );
var sq2 = ASQ.failAfter( 100, "Oops" );

sq1.val( function(msg1,msg2){
	console.log( msg1, msg2 );		// Hello World
} );

sq2.or( function(err){
	console.log( err );				// Oops
} );
```

Você também pode inserir um atraso no meio de uma sequência usando `after(..)`:

```js
ASQ( 42 )
// insere um atraso na sequência
.after( 100 )
.val( function(msg){
	console.log( msg );		// 42
} );
```

## Promises e Callbacks

Eu acho que sequências *asynquence* agregam muito valor sobre as Promises nativas e, na maior parte do tempo, você achará mais agradável e mais poderoso trabalhar nesse nível de abstração. Entretanto, integrar *asynquence* com outro código que não seja *asynquence* será uma realidade.

Você pode facilmente incorporar uma promise (ou seja, um thenable -- veja Capítulo 3) em uma sequência usando o método de instância `promise(..)`:

```js
var p = Promise.resolve( 42 );

ASQ()
.promise( p )			// poderia também: `function(){ return p; }`
.val( function(msg){
	console.log( msg );	// 42
} );
```

E para ir na direção oposta e bifurcar/fornecer uma promise a partir de uma sequência em um determinado passo, use o plug-in contrib `toPromise`:

```js
var sq = ASQ.after( 100, "Hello World" );

sq.toPromise()
// isto agora é uma cadeia de promises padrão
.then( function(msg){
	return msg.toUpperCase();
} )
.then( function(msg){
	console.log( msg );		// HELLO WORLD
} );
```

Para adaptar *asynquence* a sistemas que usam callbacks, há várias facilidades auxiliares. Para gerar automaticamente um callback "estilo error-first" a partir da sua sequência para conectar a um utilitário orientado a callbacks, use `errfcb`:

```js
var sq = ASQ( function(done){
	// nota: esperando um callback "estilo error-first"
	someAsyncFuncWithCB( 1, 2, done.errfcb )
} )
.val( function(msg){
	// ..
} )
.or( function(err){
	// ..
} );

// nota: esperando um callback "estilo error-first"
anotherAsyncFuncWithCB( 1, 2, sq.errfcb() );
```

Você também pode querer criar uma versão de um utilitário encapsulada em sequência -- compare com "promisory" no Capítulo 3 e "thunkory" no Capítulo 4 -- e *asynquence* fornece `ASQ.wrap(..)` para esse propósito:

```js
var coolUtility = ASQ.wrap( someAsyncFuncWithCB );

coolUtility( 1, 2 )
.val( function(msg){
	// ..
} )
.or( function(err){
	// ..
} );
```

**Nota:** Por uma questão de clareza (e por diversão!), vamos cunhar mais um termo, para uma função produtora de sequência que vem de `ASQ.wrap(..)`, como `coolUtility` aqui. Eu proponho "sequory" ("sequence" + "factory").

## Sequências Iteráveis

O paradigma normal para uma sequência é que cada passo é responsável por completar a si mesmo, o que é o que avança a sequência. Promises funcionam da mesma forma.

A parte infeliz é que, às vezes, você precisa de controle externo sobre uma Promise/passo, o que leva a uma estranha "extração de capacidade".

Considere este exemplo com Promises:

```js
var domready = new Promise( function(resolve,reject){
	// não queremos colocar isto aqui, porque
	// logicamente pertence a outra parte
	// do código
	document.addEventListener( "DOMContentLoaded", resolve );
} );

// ..

domready.then( function(){
	// o DOM está pronto!
} );
```

O anti-padrão de "extração de capacidade" com Promises se parece com isto:

```js
var ready;

var domready = new Promise( function(resolve,reject){
	// extrai a capacidade `resolve()`
	ready = resolve;
} );

// ..

domready.then( function(){
	// o DOM está pronto!
} );

// ..

document.addEventListener( "DOMContentLoaded", ready );
```

**Nota:** Esse anti-padrão é um estranho code smell, na minha opinião, mas alguns desenvolvedores gostam dele, por razões que não consigo entender.

*asynquence* oferece um tipo de sequência invertida que eu chamo de "sequências iteráveis", que externaliza a capacidade de controle (é bastante útil em casos de uso como o `domready`):

```js
// nota: `domready` aqui é um *iterador* que
// controla a sequência
var domready = ASQ.iterable();

// ..

domready.val( function(){
	// o DOM está pronto
} );

// ..

document.addEventListener( "DOMContentLoaded", domready.next );
```

Há muito mais sobre sequências iteráveis do que vemos neste cenário. Voltaremos a elas no Apêndice B.

## Executando Generators

No Capítulo 4, derivamos um utilitário chamado `run(..)` que pode executar generators até a conclusão, escutando por Promises retornadas via `yield` e usando-as para retomar o generator de forma assíncrona. *asynquence* tem exatamente esse utilitário embutido, chamado `runner(..)`.

Vamos primeiro configurar alguns auxiliares para ilustração:

```js
function doublePr(x) {
	return new Promise( function(resolve,reject){
		setTimeout( function(){
			resolve( x * 2 );
		}, 100 );
	} );
}

function doubleSeq(x) {
	return ASQ( function(done){
		setTimeout( function(){
			done( x * 2)
		}, 100 );
	} );
}
```

Agora, podemos usar `runner(..)` como um passo no meio de uma sequência:

```js
ASQ( 10, 11 )
.runner( function*(token){
	var x = token.messages[0] + token.messages[1];

	// faz yield de uma promise real
	x = yield doublePr( x );

	// faz yield de uma sequência
	x = yield doubleSeq( x );

	return x;
} )
.val( function(msg){
	console.log( msg );			// 84
} );
```

### Generators Encapsulados

Você também pode criar um generator autoempacotado -- isto é, uma função normal que executa o generator que você especificou e retorna uma sequência para a sua conclusão -- fazendo `ASQ.wrap(..)` nele:

```js
var foo = ASQ.wrap( function*(token){
	var x = token.messages[0] + token.messages[1];

	// faz yield de uma promise real
	x = yield doublePr( x );

	// faz yield de uma sequência
	x = yield doubleSeq( x );

	return x;
}, { gen: true } );

// ..

foo( 8, 9 )
.val( function(msg){
	console.log( msg );			// 68
} );
```

Há muito mais coisas incríveis que `runner(..)` é capaz de fazer, mas voltaremos a isso no Apêndice B.

## Revisão

*asynquence* é uma abstração simples -- uma sequência é uma série de passos (assíncronos) -- sobre as Promises, com o objetivo de tornar o trabalho com vários padrões assíncronos muito mais fácil, sem nenhum comprometimento na capacidade.

Há outras vantagens na API central da *asynquence* e em seus plug-ins contrib além do que vimos neste apêndice, mas deixaremos isso como um exercício para o leitor ir conferir o resto das capacidades.

Você agora viu a essência e o espírito da *asynquence*. O ponto-chave a ser absorvido é que uma sequência é composta de passos, e esses passos podem ser qualquer uma das dezenas de diferentes variações sobre Promises, ou podem ser uma execução de generator, ou... A escolha é sua, você tem toda a liberdade de tecer junto qualquer lógica de controle de fluxo assíncrono que seja apropriada para suas tarefas. Chega de trocar de biblioteca para capturar diferentes padrões assíncronos.

Se esses trechos de *asynquence* fizeram sentido para você, você agora está bem atualizado em relação à biblioteca; na verdade, não é preciso tanto para aprendê-la!

Se você ainda está um pouco confuso sobre como ela funciona (ou por quê!), você vai querer passar um pouco mais de tempo examinando os exemplos anteriores e brincando com a *asynquence* você mesmo, antes de prosseguir para o próximo apêndice. O Apêndice B levará a *asynquence* a vários padrões assíncronos mais avançados e poderosos.
