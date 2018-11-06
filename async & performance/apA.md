# You Don't Know JS: Async e Performance
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

Uma importante deiferença de *asynquence* em comparação com Promises se dá no tratamento de erros.

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

Às vezes você quer gerenciar erros no nível dos passos e não necessariamente enviar toda a sequência para um estado de erro. *asynquence* oferece duas variaçòes de passo para estes casos.

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

#### Promise-Style Steps

If you would prefer to have, inline in your sequence, Promise-style semantics like Promises' `then(..)` and `catch(..)` (see Chapter 3), you can use the `pThen` and `pCatch` plug-ins:

```js
ASQ( 21 )
.pThen( function(msg){
	return msg * 2;
} )
.pThen( output )				// 42
.pThen( function(){
	// throw an exception
	doesnt.Exist();
} )
.pCatch( function(err){
	// caught the exception (rejection)
	console.log( err );			// ReferenceError
} )
.val( function(){
	// main sequence is back in a
	// success state because previous
	// exception was caught by
	// `pCatch(..)`
} );
```

`pThen(..)` and `pCatch(..)` are designed to run in the sequence, but behave as if it was a normal Promise chain. As such, you can either resolve genuine Promises or *asynquence* sequences from the "fulfillment" handler passed to `pThen(..)` (see Chapter 3).

### Forking Sequences

One feature that can be quite useful about Promises is that you can attach multiple `then(..)` handler registrations to the same promise, effectively "forking" the flow-control at that promise:

```js
var p = Promise.resolve( 21 );

// fork 1 (from `p`)
p.then( function(msg){
	return msg * 2;
} )
.then( function(msg){
	console.log( msg );		// 42
} )

// fork 2 (from `p`)
p.then( function(msg){
	console.log( msg );		// 21
} );
```

The same "forking" is easy in *asynquence* with `fork()`:

```js
var sq = ASQ(..).then(..).then(..);

var sq2 = sq.fork();

// fork 1
sq.then(..)..;

// fork 2
sq2.then(..)..;
```

### Combining Sequences

The reverse of `fork()`ing, you can combine two sequences by subsuming one into another, using the `seq(..)` instance method:

```js
var sq = ASQ( function(done){
	setTimeout( function(){
		done( "Hello World" );
	}, 200 );
} );

ASQ( function(done){
	setTimeout( done, 100 );
} )
// subsume `sq` sequence into this sequence
.seq( sq )
.val( function(msg){
	console.log( msg );		// Hello World
} )
```

`seq(..)` can either accept a sequence itself, as shown here, or a function. If a function, it's expected that the function when called will return a sequence, so the preceding code could have been done with:

```js
// ..
.seq( function(){
	return sq;
} )
// ..
```

Also, that step could instead have been accomplished with a `pipe(..)`:

```js
// ..
.then( function(done){
	// pipe `sq` into the `done` continuation callback
	sq.pipe( done );
} )
// ..
```

When a sequence is subsumed, both its success message stream and its error stream are piped in.

**Note:** As mentioned in an earlier note, piping (manually with `pipe(..)` or automatically with `seq(..)`) opts the source sequence out of error-reporting, but doesn't affect the error reporting status of the target sequence.

## Value and Error Sequences

If any step of a sequence is just a normal value, that value is just mapped to that step's completion message:

```js
var sq = ASQ( 42 );

sq.val( function(msg){
	console.log( msg );		// 42
} );
```

If you want to make a sequence that's automatically errored:

```js
var sq = ASQ.failed( "Oops" );

ASQ()
.seq( sq )
.val( function(msg){
	// won't get here
} )
.or( function(err){
	console.log( err );		// Oops
} );
```

You also may want to automatically create a delayed-value or a delayed-error sequence. Using the `after` and `failAfter` contrib plug-ins, this is easy:

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

You can also insert a delay in the middle of a sequence using `after(..)`:

```js
ASQ( 42 )
// insert a delay into the sequence
.after( 100 )
.val( function(msg){
	console.log( msg );		// 42
} );
```

## Promises and Callbacks

I think *asynquence* sequences provide a lot of value on top of native Promises, and for the most part you'll find it more pleasant and more powerful to work at that level of abstraction. However, integrating *asynquence* with other non-*asynquence* code will be a reality.

You can easily subsume a promise (e.g., thenable -- see Chapter 3) into a sequence using the `promise(..)` instance method:

```js
var p = Promise.resolve( 42 );

ASQ()
.promise( p )			// could also: `function(){ return p; }`
.val( function(msg){
	console.log( msg );	// 42
} );
```

And to go the opposite direction and fork/vend a promise from a sequence at a certain step, use the `toPromise` contrib plug-in:

```js
var sq = ASQ.after( 100, "Hello World" );

sq.toPromise()
// this is a standard promise chain now
.then( function(msg){
	return msg.toUpperCase();
} )
.then( function(msg){
	console.log( msg );		// HELLO WORLD
} );
```

To adapt *asynquence* to systems using callbacks, there are several helper facilities. To automatically generate an "error-first style" callback from your sequence to wire into a callback-oriented utility, use `errfcb`:

```js
var sq = ASQ( function(done){
	// note: expecting "error-first style" callback
	someAsyncFuncWithCB( 1, 2, done.errfcb )
} )
.val( function(msg){
	// ..
} )
.or( function(err){
	// ..
} );

// note: expecting "error-first style" callback
anotherAsyncFuncWithCB( 1, 2, sq.errfcb() );
```

You also may want to create a sequence-wrapped version of a utility -- compare to "promisory" in Chapter 3 and "thunkory" in Chapter 4 -- and *asynquence* provides `ASQ.wrap(..)` for that purpose:

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

**Note:** For the sake of clarity (and for fun!), let's coin yet another term, for a sequence-producing function that comes from `ASQ.wrap(..)`, like `coolUtility` here. I propose "sequory" ("sequence" + "factory").

## Iterable Sequences

The normal paradigm for a sequence is that each step is responsible for completing itself, which is what advances the sequence. Promises work the same way.

The unfortunate part is that sometimes you need external control over a Promise/step, which leads to awkward "capability extraction".

Consider this Promises example:

```js
var domready = new Promise( function(resolve,reject){
	// don't want to put this here, because
	// it belongs logically in another part
	// of the code
	document.addEventListener( "DOMContentLoaded", resolve );
} );

// ..

domready.then( function(){
	// DOM is ready!
} );
```

The "capability extraction" anti-pattern with Promises looks like this:

```js
var ready;

var domready = new Promise( function(resolve,reject){
	// extract the `resolve()` capability
	ready = resolve;
} );

// ..

domready.then( function(){
	// DOM is ready!
} );

// ..

document.addEventListener( "DOMContentLoaded", ready );
```

**Note:** This anti-pattern is an awkward code smell, in my opinion, but some developers like it, for reasons I can't grasp.

*asynquence* offers an inverted sequence type I call "iterable sequences", which externalizes the control capability (it's quite useful in use cases like the `domready`):

```js
// note: `domready` here is an *iterator* that
// controls the sequence
var domready = ASQ.iterable();

// ..

domready.val( function(){
	// DOM is ready
} );

// ..

document.addEventListener( "DOMContentLoaded", domready.next );
```

There's more to iterable sequences than what we see in this scenario. We'll come back to them in Appendix B.

## Running Generators

In Chapter 4, we derived a utility called `run(..)` which can run generators to completion, listening for `yield`ed Promises and using them to async resume the generator. *asynquence* has just such a utility built in, called `runner(..)`.

Let's first set up some helpers for illustration:

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

Now, we can use `runner(..)` as a step in the middle of a sequence:

```js
ASQ( 10, 11 )
.runner( function*(token){
	var x = token.messages[0] + token.messages[1];

	// yield a real promise
	x = yield doublePr( x );

	// yield a sequence
	x = yield doubleSeq( x );

	return x;
} )
.val( function(msg){
	console.log( msg );			// 84
} );
```

### Wrapped Generators

You can also create a self-packaged generator -- that is, a normal function that runs your specified generator and returns a sequence for its completion -- by `ASQ.wrap(..)`ing it:

```js
var foo = ASQ.wrap( function*(token){
	var x = token.messages[0] + token.messages[1];

	// yield a real promise
	x = yield doublePr( x );

	// yield a sequence
	x = yield doubleSeq( x );

	return x;
}, { gen: true } );

// ..

foo( 8, 9 )
.val( function(msg){
	console.log( msg );			// 68
} );
```

There's a lot more awesome that `runner(..)` is capable of, but we'll come back to that in Appendix B.

## Review

*asynquence* is a simple abstraction -- a sequence is a series of (async) steps -- on top of Promises, aimed at making working with various asynchronous patterns much easier, without any compromise in capability.

There are other goodies in the *asynquence* core API and its contrib plug-ins beyond what we saw in this appendix, but we'll leave that as an exercise for the reader to go check the rest of the capabilities out.

You've now seen the essence and spirit of *asynquence*. The key take away is that a sequence is comprised of steps, and those steps can be any of dozens of different variations on Promises, or they can be a generator-run, or... The choice is up to you, you have all the freedom to weave together whatever async flow control logic is appropriate for your tasks. No more library switching to catch different async patterns.

If these *asynquence* snippets have made sense to you, you're now pretty well up to speed on the library; it doesn't take that much to learn, actually!

If you're still a little fuzzy on how it works (or why!), you'll want to spend a little more time examining the previous examples and playing around with *asynquence* yourself, before going on to the next appendix. Appendix B will push *asynquence* into several more advanced and powerful async patterns.
