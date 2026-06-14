# You Don't Know JS: Async & Performance
# Apêndice B: Padrões Assíncronos Avançados

O apêndice A introduziu a biblioteca *asynquence* para controle de fluxo assíncrono sequencial primariamente baseada em Promises e geradores.
  
Agora exploraremos outros padrões assíncronos avançados construidos a partir desta compreensão e funcionalidade existente, e veremos como *asynquence* torna técnicas de assíncronismo sofisticadas facilmente combináveis com nossos programas sem a necessidade de diversas bibliotecas diferentes.

## sequências iteráveis

Nós introduzimos sequências iteráveis no *asynquence** no apêndice anterior, mas queremos revisitá-lo em mais detalhes.

Para relembrar:

```js
var domready = ASQ.iterable();

// ..

domready.val( function(){
	// DOM is ready
} );

// ..

document.addEventListener( "DOMContentLoaded", domready.next );
```

Agora vamos definir uma sequência de múltiplos passos como uma sequência iterável:

```js
var steps = ASQ.iterable();

steps
.then( function STEP1(x){
	return x * 2;
} )
.steps( function STEP2(x){
	return x + 3;
} )
.steps( function STEP3(x){
	return x * 4;
} );

steps.next( 8 ).value;	// 16
steps.next( 16 ).value;	// 19
steps.next( 19 ).value;	// 76
steps.next().done;		// true
```

Como podemos ver, uma sequência iterável é um *iterator* compátivel com padrões (Veja capitulo 4).
Portanto pode ser iterado com o loop `for..of` da ES6, assim como um gerador (ou qualquer outro *iterável*) pode:

```js
var steps = ASQ.iterable();

steps
.then( function STEP1(){ return 2; } )
.then( function STEP2(){ return 4; } )
.then( function STEP3(){ return 6; } )
.then( function STEP4(){ return 8; } )
.then( function STEP5(){ return 10; } );

for (var v of steps) {
	console.log( v );
}
// 2 4 6 8 10
```

Além do exemplo de encadear eventos mostrada no apêndice anterior, sequências iteráveis são interessantes porque em essência podem ser vistas como substituto para geradores ou encadeamentos de Promises, mas com ainda mais flexibilidade

Considere o exemplo de uma requisição múltipla Ajax -- Nós já vimos o mesmo cenário no capitulo 3 e 4, tanto com encadeamento de promises quanto como geradores sendo expressados como uma sequência iterável

```js
// sequence-aware ajax
var request = ASQ.wrap( ajax );

ASQ( "http://some.url.1" )
.runner(
	ASQ.iterable()

	.then( function STEP1(token){
		var url = token.messages[0];
		return request( url );
	} )

	.then( function STEP2(resp){
		return ASQ().gate(
			request( "http://some.url.2/?v=" + resp ),
			request( "http://some.url.3/?v=" + resp )
		);
	} )

	.then( function STEP3(r1,r2){ return r1 + r2; } )
)
.val( function(msg){
	console.log( msg );
} );
```

A sequência iterável expressa uma série sequêncial de passos (síncronos ou assíncronos) que aparentam ser extremamente similares a um encadeamento de Promises, em outras palavras, são muito mais limpos que apenas callbacks puramente aninhados, mas não tão bons como a sintaxe sequêncial de `yield`s de geradores.

Nós passamos a sequência iterável no `ASQ#runner(..)`, que roda até sua complitude, da mesma forma como seria com geradores. O fato de que uma sequência iterável se comporta essêncialmente da mesma forma que geradores e chamam atenção por uma série de razões.

Primeiro, sequências iteráveis são meio que um equivalente pré-ES6 a um certo sub-conjunto de geradores ES6, o que significa que você pode tanto criá-los diretamente (para rodar em qualquer lugar), ou pode criá-los como geradores do ES6 para então transpilar/converter em sequências iteráveis (ou como encadeamento de Promise com essa finalidade!).

Pensar em um gerador async-roda-até-completar como apenas um _syntatic sugar_ para um encadeamento de Promise é importante para reconhecer sua relação isomórfica.

Antes de irmos em frente, devemos notar que poderiamos ter expressado o trecho de código anterior em *asynquence* como:

```js
ASQ( "http://some.url.1" )
.seq( /*STEP 1*/ request )
.seq( function STEP2(resp){
	return ASQ().gate(
		request( "http://some.url.2/?v=" + resp ),
		request( "http://some.url.3/?v=" + resp )
	);
} )
.val( function STEP3(r1,r2){ return r1 + r2; } )
.val( function(msg){
	console.log( msg );
} );
```

Alem disso, o passo 2 pode ser expressado como:

```js
.gate(
	function STEP2a(done,resp) {
		request( "http://some.url.2/?v=" + resp )
		.pipe( done );
	},
	function STEP2b(done,resp) {
		request( "http://some.url.3/?v=" + resp )
		.pipe( done );
	}
)
```

Então porque passamos pelo problema de expressar nosso controle de fluxo como uma sequência iterável em um passo de `ASQ#runner(..)` quando um encadeamento de *asynquence* aparenta muito mais simples e plano faz o trabalho bem?

Pois a forma da sequência iterável tem uma carta na manga que nos dá ainda mais capacidade. Leia mais.

### Estendendo Sequências Iteráveis

Geradores, sequências *asynquence* normais e encadeamentos de Promise são todos **avaliados de forma adiantada (eager)** -- qualquer controle de fluxo expressado inicialmente *é* o fluxo fixo que será seguido.

Entretanto, sequências iteráveis são **avaliadas de forma preguiçosa (lazy)**, o que significa que durante a execução da sequência iterável, você pode estender a sequência com mais passos, se desejar.

**Nota:** Você só pode anexar ao final de uma sequência iterável, não injetar no meio da sequência.

Vamos primeiro olhar um exemplo mais simples (síncrono) dessa capacidade para nos familiarizarmos com ela:

```js
function double(x) {
	x *= 2;

	// devemos continuar estendendo?
	if (x < 500) {
		isq.then( double );
	}

	return x;
}

// configura uma sequência iterável de passo único
var isq = ASQ.iterable().then( double );

for (var v = 10, ret;
	(ret = isq.next( v )) && !ret.done;
) {
	v = ret.value;
	console.log( v );
}
```

A sequência iterável começa com apenas um passo definido (`isq.then(double)`), mas a sequência continua estendendo a si mesma sob certas condições (`x < 500`). Tanto sequências *asynquence* quanto encadeamentos de Promise tecnicamente *podem* fazer algo similar, mas veremos em breve por que sua capacidade é insuficiente.

Embora este exemplo seja bastante trivial e pudesse, em outras circunstâncias, ser expressado com um loop `while` em um gerador, consideraremos casos mais sofisticados.

Por exemplo, você poderia examinar a resposta de uma requisição Ajax e, se ela indicar que mais dados são necessários, você condicionalmente insere mais passos na sequência iterável para fazer a(s) requisição(ões) adicional(is). Ou você poderia adicionar condicionalmente um passo de formatação de valor ao final do seu tratamento de Ajax.

Considere:

```js
var steps = ASQ.iterable()

.then( function STEP1(token){
	var url = token.messages[0].url;

	// foi fornecido um passo de formatação adicional?
	if (token.messages[0].format) {
		steps.then( token.messages[0].format );
	}

	return request( url );
} )

.then( function STEP2(resp){
	// adicionar outra requisição Ajax à sequência?
	if (/x1/.test( resp )) {
		steps.then( function STEP5(text){
			return request(
				"http://some.url.4/?v=" + text
			);
		} );
	}

	return ASQ().gate(
		request( "http://some.url.2/?v=" + resp ),
		request( "http://some.url.3/?v=" + resp )
	);
} )

.then( function STEP3(r1,r2){ return r1 + r2; } );
```

Você pode ver em dois lugares diferentes onde estendemos `steps` condicionalmente com `steps.then(..)`. E para rodar essa sequência iterável `steps`, nós apenas a conectamos ao fluxo principal do nosso programa com uma sequência *asynquence* (chamada `main` aqui) usando `ASQ#runner(..)`:

```js
var main = ASQ( {
	url: "http://some.url.1",
	format: function STEP4(text){
		return text.toUpperCase();
	}
} )
.runner( steps )
.val( function(msg){
	console.log( msg );
} );
```

A flexibilidade (comportamento condicional) da sequência iterável `steps` pode ser expressada com um gerador? Mais ou menos, mas temos que rearranjar a lógica de uma forma um pouco esquisita:

```js
function *steps(token) {
	// **STEP 1**
	var resp = yield request( token.messages[0].url );

	// **STEP 2**
	var rvals = yield ASQ().gate(
		request( "http://some.url.2/?v=" + resp ),
		request( "http://some.url.3/?v=" + resp )
	);

	// **STEP 3**
	var text = rvals[0] + rvals[1];

	// **STEP 4**
	// foi fornecido um passo de formatação adicional?
	if (token.messages[0].format) {
		text = yield token.messages[0].format( text );
	}

	// **STEP 5**
	// precisa de outra requisição Ajax adicionada à sequência?
	if (/foobar/.test( resp )) {
		text = yield request(
			"http://some.url.4/?v=" + text
		);
	}

	return text;
}

// nota: `*steps()` pode ser rodado pela mesma sequência `ASQ`
// que `steps` foi anteriormente
```

Deixando de lado os benefícios já identificados da sintaxe sequencial, com aparência síncrona, dos geradores (veja o Capítulo 4), a lógica de `steps` teve que ser reordenada na forma de gerador `*steps()`, para fingir o dinamismo da sequência iterável extensível `steps`.

Mas e quanto a expressar a funcionalidade com Promises ou sequências? Você *pode* fazer algo assim:

```js
var steps = something( .. )
.then( .. )
.then( function(..){
	// ..

	// estendendo o encadeamento, certo?
	steps = steps.then( .. );

	// ..
})
.then( .. );
```

O problema é sutil mas importante de compreender. Então, considere tentar conectar nosso encadeamento de Promise `steps` ao fluxo principal do nosso programa -- desta vez expressado com Promises em vez de *asynquence*:

```js
var main = Promise.resolve( {
	url: "http://some.url.1",
	format: function STEP4(text){
		return text.toUpperCase();
	}
} )
.then( function(..){
	return steps;			// dica!
} )
.val( function(msg){
	console.log( msg );
} );
```

Você consegue identificar o problema agora? Olhe de perto!

Há uma condição de corrida (race condition) na ordenação dos passos da sequência. Quando você faz `return steps`, naquele momento `steps` *pode* ser o encadeamento de promise originalmente definido, ou pode agora apontar para o encadeamento de promise estendido via a chamada `steps = steps.then(..)`, dependendo da ordem em que as coisas acontecem.

Aqui estão os dois resultados possíveis:

* Se `steps` ainda for o encadeamento de promise original, uma vez que ele seja posteriormente "estendido" por `steps = steps.then(..)`, essa promise estendida no final do encadeamento **não** é considerada pelo fluxo `main`, pois ele já se conectou ao encadeamento `steps`. Esta é a infelizmente limitante **avaliação adiantada (eager evaluation)**.
* Se `steps` já for o encadeamento de promise estendido, ele funciona como esperamos, no sentido de que a promise estendida é à qual `main` se conecta.

Além do fato óbvio de que uma condição de corrida é intolerável, o primeiro caso é a preocupação; ele ilustra a **avaliação adiantada (eager evaluation)** do encadeamento de promise. Em contraste, nós estendemos facilmente a sequência iterável sem tais problemas, porque sequências iteráveis são **avaliadas de forma preguiçosa (lazily evaluated)**.

Quanto mais dinâmico você precisar que seja seu controle de fluxo, mais as sequências iteráveis brilharão.

**Dica:** Confira mais informações e exemplos de sequências iteráveis no site do *asynquence* (https://github.com/getify/asynquence/blob/master/README.md#iterable-sequences).

## Event Reactive

Deveria ser óbvio (pelo menos!) a partir do Capítulo 3 que Promises são uma ferramenta muito poderosa na sua caixa de ferramentas assíncronas. Mas uma coisa que claramente falta é a capacidade delas de lidar com fluxos (streams) de eventos, já que uma Promise só pode ser resolvida uma vez. E, francamente, esta exata mesma fraqueza é verdadeira para sequências *asynquence* simples também.

Considere um cenário em que você quer disparar uma série de passos toda vez que um certo evento for disparado. Uma única Promise ou sequência não pode representar todas as ocorrências daquele evento. Então, você tem que criar um encadeamento de Promise (ou sequência) inteiramente novo para *cada* ocorrência do evento, tal como:

```js
listener.on( "foobar", function(data){

	// cria um novo encadeamento de promise de tratamento de evento
	new Promise( function(resolve,reject){
		// ..
	} )
	.then( .. )
	.then( .. );

} );
```

A funcionalidade básica de que precisamos está presente nesta abordagem, mas ela está longe de ser uma forma desejável de expressar nossa lógica pretendida. Há duas capacidades separadas misturadas neste paradigma: a escuta do evento e a resposta ao evento; a separação de responsabilidades nos imploraria para separar essas capacidades.

O leitor cuidadosamente observador verá este problema como algo simétrico aos problemas que detalhamos com callbacks no Capítulo 2; é uma espécie de problema de inversão de controle.

Imagine desinverter este paradigma, assim:

```js
var observable = listener.on( "foobar" );

// depois
observable
.then( .. )
.then( .. );

// em outro lugar
observable
.then( .. )
.then( .. );
```

O valor `observable` não é exatamente uma Promise, mas você pode *observá-lo* muito parecido com a forma como você pode observar uma Promise, então ele é estreitamente relacionado. De fato, ele pode ser observado muitas vezes, e enviará notificações toda vez que seu evento (`"foobar"`) ocorrer.

**Dica:** Este padrão que acabei de ilustrar é uma **simplificação massiva** dos conceitos e motivações por trás da programação reativa (também conhecida como RP), que tem sido implementada/expandida por vários projetos e linguagens excelentes. Uma variação da RP é a programação reativa funcional (FRP), que se refere à aplicação de técnicas de programação funcional (imutabilidade, integridade referencial, etc.) a fluxos de dados. "Reativo" refere-se a espalhar essa funcionalidade ao longo do tempo em resposta a eventos. O leitor interessado deveria considerar estudar "Reactive Observables" na fantástica biblioteca "Reactive Extensions" ("RxJS" para JavaScript) da Microsoft (http://rxjs.codeplex.com/); ela é muito mais sofisticada e poderosa do que acabei de mostrar. Além disso, Andre Staltz tem um excelente artigo (https://gist.github.com/staltz/868e7e9bc2a7b8c1f754) que expõe a RP de forma pragmática em exemplos concretos.

### Observables do ES7

No momento em que isto é escrito, há uma proposta inicial para o ES7 de um novo tipo de dado chamado "Observable" (https://github.com/jhusain/asyncgenerator#introducing-observable), que em espírito é similar ao que expusemos aqui, mas é definitivamente mais sofisticado.

A noção desse tipo de Observable é que a forma como você "se inscreve" (subscribe) nos eventos de um stream é passar um gerador -- na verdade o *iterator* é a parte interessada -- cujo método `next(..)` será chamado para cada evento.

Você poderia imaginá-lo mais ou menos assim:

```js
// `someEventStream` é um stream de eventos, como de
// cliques do mouse, e coisas do tipo.

var observer = new Observer( someEventStream, function*(){
	while (var evt = yield) {
		console.log( evt );
	}
} );
```

O gerador que você passa irá fazer `yield` para pausar o loop `while` esperando o próximo evento. O *iterator* anexado à instância do gerador terá seu `next(..)` chamado toda vez que `someEventStream` tiver um novo evento publicado, e assim aqueles dados do evento irão retomar seu gerador/*iterator* com os dados `evt`.

Na funcionalidade de inscrição em eventos aqui, é a parte do *iterator* que importa, não o gerador. Então, conceitualmente, você poderia passar praticamente qualquer iterável, incluindo sequências iteráveis `ASQ.iterable()`.

Curiosamente, há também adaptadores propostos para tornar fácil construir Observables a partir de certos tipos de streams, tais como `fromEvent(..)` para eventos do DOM. Se você olhar uma implementação sugerida de `fromEvent(..)` na proposta do ES7 linkada anteriormente, ela se parece muito com o `ASQ.react(..)` que veremos na próxima seção.

Claro, essas são todas propostas iniciais, então o que sair disso pode muito bem parecer/se comportar de forma diferente do mostrado aqui. Mas é empolgante ver os alinhamentos iniciais de conceitos entre diferentes bibliotecas e propostas de linguagem!

### Sequências Reativas

Com aquele resumo brevíssimo e maluco de Observables (e F/RP) como nossa inspiração e motivação, agora ilustrarei uma adaptação de um pequeno subconjunto de "Reactive Observables", que eu chamo de "Sequências Reativas".

Primeiro, vamos começar com como criar um Observable, usando um utilitário plug-in do *asynquence* chamado `react(..)`:

```js
var observable = ASQ.react( function setup(next){
	listener.on( "foobar", next );
} );
```

Agora, vamos ver como definir uma sequência que "reage" -- em F/RP, isto é tipicamente chamado de "se inscrever" (subscribing) -- àquele `observable`:

```js
observable
.seq( .. )
.then( .. )
.val( .. );
```

Então, você apenas define a sequência encadeando a partir do Observable. Isso é fácil, hein?

Em F/RP, o stream de eventos tipicamente passa por um conjunto de transformações funcionais, como `scan(..)`, `map(..)`, `reduce(..)`, e assim por diante. Com sequências reativas, cada evento passa por uma nova instância da sequência. Vamos olhar um exemplo mais concreto:

```js
ASQ.react( function setup(next){
	document.getElementById( "mybtn" )
	.addEventListener( "click", next, false );
} )
.seq( function(evt){
	var btnID = evt.target.id;
	return request(
		"http://some.url.1/?id=" + btnID
	);
} )
.val( function(text){
	console.log( text );
} );
```

A porção "reativa" da sequência reativa vem de atribuir um ou mais tratadores de evento (event handlers) para invocar o gatilho do evento (chamando `next(..)`).

A porção "sequência" da sequência reativa é exatamente como as sequências que já exploramos: cada passo pode ser qualquer técnica assíncrona que faça sentido, de callback de continuação a Promise a gerador.

Uma vez que você configura uma sequência reativa, ela continuará a iniciar instâncias da sequência enquanto os eventos continuarem disparando. Se você quiser parar uma sequência reativa, você pode chamar `stop()`.

Se uma sequência reativa for parada com `stop()`, você provavelmente vai querer que o(s) tratador(es) de evento sejam desregistrados também; você pode registrar um tratador de desmontagem (teardown) para este propósito:

```js
var sq = ASQ.react( function setup(next,registerTeardown){
	var btn = document.getElementById( "mybtn" );

	btn.addEventListener( "click", next, false );

	// será chamado assim que `sq.stop()` for chamado
	registerTeardown( function(){
		btn.removeEventListener( "click", next, false );
	} );
} )
.seq( .. )
.then( .. )
.val( .. );

// depois
sq.stop();
```

**Nota:** A referência de binding do `this` dentro do tratador `setup(..)` é a mesma sequência reativa `sq`, então você pode usar a referência `this` para adicionar à definição da sequência reativa, chamar métodos como `stop()`, e assim por diante.

Aqui está um exemplo do mundo do Node.js, usando sequências reativas para lidar com requisições HTTP que chegam:

```js
var server = http.createServer();
server.listen(8000);

// observador reativo
var request = ASQ.react( function setup(next,registerTeardown){
	server.addListener( "request", next );
	server.addListener( "close", this.stop );

	registerTeardown( function(){
		server.removeListener( "request", next );
		server.removeListener( "close", request.stop );
	} );
});

// responde às requisições
request
.seq( pullFromDatabase )
.val( function(data,res){
	res.end( data );
} );

// desmontagem do node
process.on( "SIGINT", request.stop );
```

O gatilho `next(..)` também pode se adaptar facilmente a streams do node, usando `onStream(..)` e `unStream(..)`:

```js
ASQ.react( function setup(next){
	var fstream = fs.createReadStream( "/some/file" );

	// canaliza o evento "data" do stream para `next(..)`
	next.onStream( fstream );

	// escuta o fim do stream
	fstream.on( "end", function(){
		next.unStream( fstream );
	} );
} )
.seq( .. )
.then( .. )
.val( .. );
```

Você também pode usar combinações de sequências para compor múltiplos streams de sequências reativas:

```js
var sq1 = ASQ.react( .. ).seq( .. ).then( .. );
var sq2 = ASQ.react( .. ).seq( .. ).then( .. );

var sq3 = ASQ.react(..)
.gate(
	sq1,
	sq2
)
.then( .. );
```

A principal lição é que `ASQ.react(..)` é uma adaptação leve de conceitos de F/RP, possibilitando a conexão de um stream de eventos a uma sequência, daí o termo "sequência reativa". Sequências reativas são geralmente capazes o suficiente para usos reativos básicos.

**Nota:** Aqui está um exemplo de uso de `ASQ.react(..)` no gerenciamento de estado de UI (http://jsbin.com/rozipaki/6/edit?js,output), e outro exemplo de tratamento de streams de requisição/resposta HTTP com `ASQ.react(..)` (https://gist.github.com/getify/bba5ec0de9d6047b720e).

## Corrotina de Gerador (Generator Coroutine)

Espero que o Capítulo 4 tenha ajudado você a ficar bem familiarizado com os geradores do ES6. Em particular, queremos revisitar a discussão sobre "Concorrência de Geradores" e levá-la ainda mais longe.

Nós imaginamos um utilitário `runAll(..)` que poderia receber dois ou mais geradores e rodá-los concorrentemente, permitindo que eles cooperativamente fizessem `yield` do controle de um para o próximo, com passagem opcional de mensagens.

Além de ser capaz de rodar um único gerador até a conclusão, o `ASQ#runner(..)` que discutimos no Apêndice A é uma implementação similar dos conceitos de `runAll(..)`, que pode rodar múltiplos geradores concorrentemente até a conclusão.

Então vamos ver como podemos implementar o cenário de Ajax concorrente do Capítulo 4:

```js
ASQ(
	"http://some.url.2"
)
.runner(
	function*(token){
		// transfere o controle
		yield token;

		var url1 = token.messages[0]; // "http://some.url.1"

		// limpa as mensagens para começar do zero
		token.messages = [];

		var p1 = request( url1 );

		// transfere o controle
		yield token;

		token.messages.push( yield p1 );
	},
	function*(token){
		var url2 = token.messages[0]; // "http://some.url.2"

		// passa a mensagem e transfere o controle
		token.messages[0] = "http://some.url.1";
		yield token;

		var p2 = request( url2 );

		// transfere o controle
		yield token;

		token.messages.push( yield p2 );

		// passa adiante os resultados para o próximo passo da sequência
		return token.messages;
	}
)
.val( function(res){
	// `res[0]` vem de "http://some.url.1"
	// `res[1]` vem de "http://some.url.2"
} );
```

As principais diferenças entre `ASQ#runner(..)` e `runAll(..)` são as seguintes:

* Cada gerador (corrotina) recebe um argumento que chamamos de `token`, que é o valor especial para fazer `yield` quando você quer transferir explicitamente o controle para a próxima corrotina.
* `token.messages` é um array que guarda quaisquer mensagens passadas do passo anterior da sequência. É também uma estrutura de dados que você pode usar para compartilhar mensagens entre corrotinas.
* Fazer `yield` de um valor de Promise (ou sequência) não transfere o controle, mas em vez disso pausa o processamento da corrotina até que aquele valor esteja pronto.
* O último valor `return`ado ou `yield`ado da execução de processamento da corrotina será passado adiante para o próximo passo na sequência.

Também é fácil colocar camadas de helpers em cima da funcionalidade base do `ASQ#runner(..)` para se adequar a diferentes usos.

### Máquinas de Estado (State Machines)

Um exemplo que pode ser familiar a muitos programadores são as máquinas de estado. Você pode, com a ajuda de um simples utilitário cosmético, criar um processador de máquina de estado fácil de expressar.

Vamos imaginar tal utilitário. Vamos chamá-lo de `state(..)`, e passaremos a ele dois argumentos: um valor de estado e um gerador que trata aquele estado. `state(..)` fará o trabalho sujo de criar e retornar um gerador adaptador para passar ao `ASQ#runner(..)`.

Considere:

```js
function state(val,handler) {
	// cria um tratador de corrotina para este estado
	return function*(token) {
		// tratador de transição de estado
		function transition(to) {
			token.messages[0] = to;
		}

		// define o estado inicial (se nenhum foi definido ainda)
		if (token.messages.length < 1) {
			token.messages[0] = val;
		}

		// continua até que o estado final (false) seja alcançado
		while (token.messages[0] !== false) {
			// o estado atual corresponde a este tratador?
			if (token.messages[0] === val) {
				// delega ao tratador do estado
				yield *handler( transition );
			}

			// transfere o controle para outro tratador de estado?
			if (token.messages[0] !== false) {
				yield token;
			}
		}
	};
}
```

Se você olhar de perto, verá que `state(..)` retorna um gerador que aceita um `token`, e então configura um loop `while` que vai rodar até a máquina de estado alcançar seu estado final (que arbitrariamente escolhemos como o valor `false`); é exatamente o tipo de gerador que queremos passar ao `ASQ#runner(..)`!

Nós também arbitrariamente reservamos o slot `token.messages[0]` como o lugar onde o estado atual da nossa máquina de estado será rastreado, o que significa que podemos até semear o estado inicial como o valor passado do passo anterior na sequência.

Como usamos o helper `state(..)` junto com o `ASQ#runner(..)`?

```js
var prevState;

ASQ(
	/* opcional: valor de estado inicial */
	2
)
// roda nossa máquina de estado
// transições: 2 -> 3 -> 1 -> 3 -> false
.runner(
	// tratador do estado `1`
	state( 1, function *stateOne(transition){
		console.log( "in state 1" );

		prevState = 1;
		yield transition( 3 );	// vai para o estado `3`
	} ),

	// tratador do estado `2`
	state( 2, function *stateTwo(transition){
		console.log( "in state 2" );

		prevState = 2;
		yield transition( 3 );	// vai para o estado `3`
	} ),

	// tratador do estado `3`
	state( 3, function *stateThree(transition){
		console.log( "in state 3" );

		if (prevState === 2) {
			prevState = 3;
			yield transition( 1 ); // vai para o estado `1`
		}
		// tudo pronto!
		else {
			yield "That's all folks!";

			prevState = 3;
			yield transition( false ); // estado terminal
		}
	} )
)
// máquina de estado completa, então prossiga
.val( function(msg){
	console.log( msg );	// That's all folks!
} );
```

É importante notar que os próprios geradores `*stateOne(..)`, `*stateTwo(..)` e `*stateThree(..)` são reinvocados cada vez que aquele estado é entrado, e eles terminam quando você faz `transition(..)` para outro valor. Embora não mostrado aqui, é claro que esses tratadores de gerador de estado podem ser pausados assincronamente fazendo `yield` de Promises/sequências/thunks.

Os geradores ocultos por baixo produzidos pelo helper `state(..)` e que de fato são passados ao `ASQ#runner(..)` são os que continuam a rodar concorrentemente durante toda a duração da máquina de estado, e cada um deles trata de cooperativamente fazer `yield` do controle para o próximo, e assim por diante.

**Nota:** Veja este exemplo de "ping pong" (http://jsbin.com/qutabu/1/edit?js,output) para mais ilustração do uso de concorrência cooperativa com geradores conduzidos pelo `ASQ#runner(..)`.

## Communicating Sequential Processes (CSP)

"Communicating Sequential Processes" (CSP) foi descrito pela primeira vez por C. A. R. Hoare em um artigo acadêmico de 1978 (http://dl.acm.org/citation.cfm?doid=359576.359585), e depois em um livro de 1985 (http://www.usingcsp.com/) de mesmo nome. CSP descreve um método formal para "processos" concorrentes interagirem (ou seja, "se comunicarem") durante o processamento.

Você deve se lembrar que examinamos "processos" concorrentes lá no Capítulo 1, então nossa exploração de CSP aqui irá se construir sobre aquele entendimento.

Como a maioria dos grandes conceitos da ciência da computação, CSP está fortemente impregnado de formalismo acadêmico, expressado como uma álgebra de processos. Entretanto, suspeito que teoremas de álgebra simbólica não farão muita diferença prática para o leitor, então vamos querer encontrar alguma outra forma de envolver nossos cérebros em torno do CSP.

Deixarei grande parte da descrição formal e da prova do CSP para a escrita de Hoare, e para muitos outros escritos fantásticos desde então. Em vez disso, tentaremos apenas explicar brevemente a ideia do CSP de uma forma tão não acadêmica e, espero, intuitivamente compreensível quanto possível.

### Passagem de Mensagens (Message Passing)

O princípio central no CSP é que toda comunicação/interação entre processos de outra forma independentes deve acontecer através da passagem formal de mensagens. Talvez contrário às suas expectativas, a passagem de mensagens no CSP é descrita como uma ação síncrona, onde o processo emissor e o processo receptor têm que estar mutuamente prontos para a mensagem ser passada.

Como tal mensageria síncrona poderia possivelmente estar relacionada à programação assíncrona em JavaScript?

A concretude da relação vem da natureza de como os geradores do ES6 são usados para produzir ações de aparência síncrona que, por baixo dos panos, podem de fato ser tanto síncronas quanto (mais provavelmente) assíncronas.

Em outras palavras, dois ou mais geradores rodando concorrentemente podem parecer enviar mensagens síncronas uns aos outros, ao mesmo tempo em que preservam a assincronia fundamental do sistema, porque o código de cada gerador está pausado (ou seja, "bloqueado") esperando a retomada de uma ação assíncrona.

Como isso funciona?

Imagine um gerador (ou seja, "processo") chamado "A" que quer enviar uma mensagem para o gerador "B". Primeiro, "A" faz `yield` da mensagem (pausando assim "A") para ser enviada a "B". Quando "B" estiver pronto e pegar a mensagem, "A" é então retomado (desbloqueado).

Simetricamente, imagine um gerador "A" que quer uma mensagem **de** "B". "A" faz `yield` da sua requisição (pausando assim "A") pela mensagem de "B", e uma vez que "B" envia uma mensagem, "A" pega a mensagem e é retomado.

Uma das expressões mais populares dessa teoria de passagem de mensagens do CSP vem da biblioteca core.async do ClojureScript, e também da linguagem *go*. Essas interpretações do CSP incorporam as semânticas de comunicação descritas em um conduto que é aberto entre processos chamado de "canal" (channel).

**Nota:** O termo *canal* é usado em parte porque há modos em que mais de um valor pode ser enviado de uma vez para dentro do "buffer" do canal; isso é similar ao que você pode pensar como um stream. Não vamos nos aprofundar nisso aqui, mas pode ser uma técnica muito poderosa para gerenciar streams de dados.

Na noção mais simples de CSP, um canal que criamos entre "A" e "B" teria um método chamado `take(..)` para bloquear até receber um valor, e um método chamado `put(..)` para bloquear até enviar um valor.

Isso pode parecer com:

```js
var ch = channel();

function *foo() {
	var msg = yield take( ch );

	console.log( msg );
}

function *bar() {
	yield put( ch, "Hello World" );

	console.log( "message sent" );
}

run( foo );
run( bar );
// Hello World
// "message sent"
```

Compare esta interação estruturada e de passagem de mensagens síncrona(-aparente) com o compartilhamento de mensagens informal e não estruturado que o `ASQ#runner(..)` fornece através do array `token.messages` e do `yield` cooperativo. Em essência, `yield put(..)` é uma única operação que tanto envia o valor quanto pausa a execução para transferir o controle, enquanto nos exemplos anteriores fizemos isso como passos separados.

Além disso, o CSP enfatiza que você não realmente "transfere o controle" explicitamente, mas em vez disso você projeta suas rotinas concorrentes para bloquear esperando ou um valor recebido do canal, ou para bloquear esperando tentar enviar uma mensagem no canal. O bloqueio ao redor do recebimento ou envio de mensagens é como você coordena o sequenciamento de comportamento entre as corrotinas.

**Nota:** Aviso justo: este padrão é muito poderoso, mas também é um pouco confuso mentalmente para se acostumar a princípio. Você vai querer praticar isto um pouco para se acostumar a esta nova forma de pensar sobre coordenar sua concorrência.

Há várias bibliotecas excelentes que implementaram este sabor de CSP em JavaScript, mais notavelmente a "js-csp" (https://github.com/ubolonton/js-csp), que James Long (http://twitter.com/jlongster) fez fork (https://github.com/jlongster/js-csp) e sobre a qual escreveu extensivamente (http://jlongster.com/Taming-the-Asynchronous-Beast-with-CSP-in-JavaScript). Além disso, não dá para enfatizar o suficiente quão incríveis são os muitos escritos de David Nolen (http://twitter.com/swannodette) sobre o tópico de adaptar o CSP estilo-go do core.async do ClojureScript para geradores do JS (http://swannodette.github.io/2013/08/24/es6-generators-and-csp/).

### Emulação de CSP no asynquence

Como estivemos discutindo padrões assíncronos aqui no contexto da minha biblioteca *asynquence*, você pode estar interessado em ver que podemos facilmente adicionar uma camada de emulação em cima do tratamento de geradores do `ASQ#runner(..)` como uma portabilidade quase perfeita da API e comportamento do CSP. Esta camada de emulação é distribuída como uma parte opcional do pacote "asynquence-contrib" junto ao *asynquence*.

Muito similar ao helper `state(..)` de antes, `ASQ.csp.go(..)` recebe um gerador -- em termos de go/core.async, ele é conhecido como uma goroutine -- e o adapta para uso com o `ASQ#runner(..)` retornando um novo gerador.

Em vez de receber um `token`, sua goroutine recebe um canal inicialmente criado (`ch` abaixo) que todas as goroutines nesta execução irão compartilhar. Você pode criar mais canais (o que frequentemente é bem útil!) com `ASQ.csp.chan(..)`.

No CSP, modelamos toda assincronia em termos de bloqueio em mensagens de canal, em vez de bloqueio esperando uma Promise/sequência/thunk completar.

Então, em vez de fazer `yield` da Promise retornada de `request(..)`, `request(..)` deveria retornar um canal do qual você faz `take(..)` de um valor. Em outras palavras, um canal de valor único é aproximadamente equivalente, neste contexto/uso, a uma Promise/sequência.

Vamos primeiro fazer uma versão de `request(..)` ciente de canais:

```js
function request(url) {
	var ch = ASQ.csp.channel();
	ajax( url ).then( function(content){
		// `putAsync(..)` é uma versão de `put(..)` que
		// pode ser usada fora de um gerador. Ela retorna
		// uma promise para a conclusão da operação. Nós
		// não usamos essa promise aqui, mas poderíamos se
		// precisássemos ser notificados quando o valor
		// tivesse sido pego com `take(..)`.
		ASQ.csp.putAsync( ch, content );
	} );
	return ch;
}
```

Do Capítulo 3, "promisory" é um utilitário produtor de Promise, "thunkory" do Capítulo 4 é um utilitário produtor de thunk, e finalmente, no Apêndice A inventamos "sequory" para um utilitário produtor de sequência.

Naturalmente, precisamos cunhar um termo simétrico aqui para um utilitário produtor de canal. Então, sem surpresa, vamos chamá-lo de "chanory" ("channel" + "factory"). Como exercício para o leitor, tente definir um utilitário `channelify(..)` similar a `Promise.wrap(..)`/`promisify(..)` (Capítulo 3), `thunkify(..)` (Capítulo 4) e `ASQ.wrap(..)` (Apêndice A).

Agora considere o exemplo de Ajax concorrente usando CSP no sabor *asynquence*:

```js
ASQ()
.runner(
	ASQ.csp.go( function*(ch){
		yield ASQ.csp.put( ch, "http://some.url.2" );

		var url1 = yield ASQ.csp.take( ch );
		// "http://some.url.1"

		var res1 = yield ASQ.csp.take( request( url1 ) );

		yield ASQ.csp.put( ch, res1 );
	} ),
	ASQ.csp.go( function*(ch){
		var url2 = yield ASQ.csp.take( ch );
		// "http://some.url.2"

		yield ASQ.csp.put( ch, "http://some.url.1" );

		var res2 = yield ASQ.csp.take( request( url2 ) );
		var res1 = yield ASQ.csp.take( ch );

		// passa adiante os resultados para o próximo passo da sequência
		ch.buffer_size = 2;
		ASQ.csp.put( ch, res1 );
		ASQ.csp.put( ch, res2 );
	} )
)
.val( function(res1,res2){
	// `res1` vem de "http://some.url.1"
	// `res2` vem de "http://some.url.2"
} );
```

A passagem de mensagens que troca as strings de URL entre as duas goroutines é bem direta. A primeira goroutine faz uma requisição Ajax para a primeira URL, e essa resposta é colocada (put) no canal `ch`. A segunda goroutine faz uma requisição Ajax para a segunda URL, então pega a primeira resposta `res1` do canal `ch`. Naquele ponto, ambas as respostas `res1` e `res2` estão completas e prontas.

Se houver quaisquer valores restantes no canal `ch` ao final da execução das goroutines, eles serão passados adiante para o próximo passo na sequência. Então, para passar mensagem(ns) para fora da goroutine final, faça `put(..)` delas em `ch`. Como mostrado, para evitar o bloqueio daqueles `put(..)`s finais, nós colocamos `ch` em modo de buffering definindo seu `buffer_size` como `2` (padrão: `0`).

**Nota:** Veja muito mais exemplos de uso de CSP no sabor *asynquence* aqui (https://gist.github.com/getify/e0d04f1f5aa24b1947ae).

## Revisão

Promises e geradores fornecem os blocos de construção fundamentais sobre os quais podemos construir uma assincronia muito mais sofisticada e capaz.

O *asynquence* tem utilitários para implementar *sequências iteráveis*, *sequências reativas* (ou seja, "Observables"), *corrotinas concorrentes* e até *goroutines CSP*.

Esses padrões, combinados com as capacidades de callback de continuação e de Promise, dão ao *asynquence* uma mistura poderosa de diferentes funcionalidades assíncronas, todas integradas em uma abstração de controle de fluxo assíncrono limpa: a sequência.
