# You Don't Know JS: ES6 & Além
# Capítulo 8: ES6 & Além

No momento atual em que escrevo, o rascunho final do ES6 (*ECMAScript 2015*) está perto de ser encaminhado ao último voto oficial de aprovação pelo ECMA. Entretanto, ao mesmo tempo em que o ES6 está sendo finalizado, o comitê TC39 já está trabalhando duro nas funcionalidades para o ES7/2016 e além. 

Como discutimos no Capítulo 1, espera-se que o ritmo de progresso do JS será mais acelerado, e, ao invés de ser atualizado uma vez a cada vários anos, terá uma versão oficial uma vez por ano (por isso a nomenclatura baseada no ano). Isso, por si só, já mudará radicalmente como os desenvolvedores JS aprendem e se mantêm atualizado com a linguagem.

Ainda mais importante que isso, o comitê passará em realidade a trabalhar em uma funcionalidade de cada vez. Assim que a especificação de uma funcionalidade esteja completa e todas suas excentricidades tiverem sido testadas em experimentos de implementação em alguns navegadores, ela será considerada estável suficiente para uso. Somos encorajados a adotar funcionalidades uma vez que estejam prontas ao invés de esperar por uma votação de padrões oficiais. Se você ainda não aprendeu ES6, já está mais que na hora de fazer isso! 

No momento atual em que escrevo, uma lista de propostas futuras e seus status podem ser encontrados aqui (https://github.com/tc39/ecma262#current-proposals).

Transpilers e polyfills serão utilizados por nós como ponte para essas novas funcionalidade, antes mesmo que todos os navegadores que apoiamos as tenham implementado. Babel, Traceur, e vários outros grandes transpilers já tem suporte para algumas dessas funcionalidades pós-ES6 que são mais prováveis de serem estabilizadas. 

Com isso em mente, é hora de dar uma olhada em algumas dessas funcionalidades. Vamos lá! 

**Atenção:** Essas funcionalidades estão em diferentes estágios de desenvolvimento. Apesar de ser provável que elas venham a existir, e provavelmente parecerão similares, leia o conteúdo desse capítulo com bastante cautela. Esse capítulo evoluirá em edições futuras desse livro a medida que essas (e outras!) funcionalidades estejam finalizadas.

## `funções async`

Na seção “Generators + Promises” do Capítulo 4, mencionamos que existe uma proposta para suporte sintático direto para o padrão de *generators* que entregam (`YIELD`) *promises* à uma utilidade do tipo *runner* que irá retomá-lo uma vez a *promise* seja completada. Vamos dar uma olhada rápida nessa funcionalidade proposta, chamada de `função async`. 

Lembre-se desse exemplo de *generator* do Capítulo 4: 

```js
run( function *main() {
	var ret = yield step1();

	try {
		ret = yield step2( ret );
	}
	catch (err) {
		ret = yield step2Failed( err );
	}

	ret = yield Promise.all([
		step3a( ret ),
		step3b( ret ),
		step3c( ret )
	]);

	yield step4( ret );
} )
.then(
	function fulfilled(){
		// `*main()` foi completada com sucesso
	},
	function rejected(reason){
		// Oops, algo deu errado
	}
);
```
A sintaxe proposta para `função async` pode expressar essa mesma lógica de controle de fluxo sem precisar da utilidade `run(..)`, porque o JS saberá automaticamente como buscar *promises* para esperar e retomar. Considere:

```js
async function main() {
	var ret = await step1();

	try {
		ret = await step2( ret );
	}
	catch (err) {
		ret = await step2Failed( err );
	}

	ret = await Promise.all( [
		step3a( ret ),
		step3b( ret ),
		step3c( ret )
	] );

	await step4( ret );
}

main()
.then(
	function fulfilled(){
		// `main()` completada com sucesso
	},
	function rejected(reason){
		// Oops, algo deu errado 
	}
);
```
Ao invés da declaração `function *main() { ..`, declaramos com o formato `async function main() { ..`. E ao invés de entregar (`yield`) uma *promise*, nós a esperamos (`await`). A chamada para executar a função `main()` em realidade retorna uma *promise* que podemos observar diretamente. É o equivalente à *promise* que recebemos de volta da chamada de `run(main)`.

Você consegue ver a simeteria? A `função async` é basicamente açúcar sintático para padrões como os de *generators* + *promises* + `run(..)`; por detrás dos panos, funciona da mesma maneira!

Se você é um desenvolvedor C# e `async`/`await` parece familiar, é porque essa funcionalidade foi diretamente inspirada por uma de C#. É bom ver precedência de linguagem formando convergência. 

Babel, Traceur e outros transpilers já tem um suporte antecipado para o status atual das `funções async`, então você já poderia começar a usá-las. Entretanto, na próxima seção "Ressalvas" veremos porque talvez você não deveria pular nesse barco por agora.

**Nota:** Há também uma proposta para `função* async`, que seria chamada de "generator async." Você poderia usar `yield` e `await` no mesmo código e até mesmo combinar essas operações em uma mesma instrução: `x = await yield y`. Essa proposta de "generator async" parece estar ainda em curso – ou seja, o valor de retorno não está completamente definido ainda. Algumas pessoas pensam que deveria ser *observável*, o que seria como a combinação de um iterador e uma promise. Por agora, não iremos entrar em mais detalhes sobre esse tópico, mas fique atento à medida que evolua. 

### Ressalvas

Um ponto de discórdia não resolvido com a `função async` se deve ao fato de que ela só retorna uma *promise*, e não é possível cancelar uma `função async` desde fora dela. Isso pode ser um problema se a operação *async* utiliza recursos de maneira intensiva e você quisesse liberar os recursos assim que você tivesse certeza que o resultado não fosse mais necessário. 

Por exemplo: 

```js
async function request(url) {
	var resp = await (
		new Promise( function(resolve,reject){
			var xhr = new XMLHttpRequest();
			xhr.open( "GET", url );
			xhr.onreadystatechange = function(){
				if (xhr.readyState == 4) {
					if (xhr.status == 200) {
						resolve( xhr );
					}
					else {
						reject( xhr.statusText );
					}
				}
			};
			xhr.send();
		} )
	);

	return resp.responseText;
}

var pr = request( "http://some.url.1" );

pr.then(
	function fulfilled(responseText){
		// sucesso do ajax
	},
	function rejected(reason){
		// Oops, algo deu errado
	}
);
```

Essa função `request(..)` que concebi é de certa forma parecida com a utilidade `fetch(..)` que foi recentemente proposta de ser incluída na plataforma web. A preocupação então é: o que acontece se você quer usar o valor `pr` para, de alguma maneira, indicar que você quer cancelar um pedido de Ajax de longa-duração, por exemplo?

*Promises* não são canceláveis (pelo menos não no momento em que escrevo). Na minha opinião, assim como de muitas outras pessoas, elas nunca deveriam ser (veja o título *Async e Performance* dessa série). E mesmo se uma *promise* tivesse um método de cancelar como `cancel()`, isso deveria mesmo significar que chamar  `pr.cancel()` propagaria o sinal de cancelamento de volta por todo o caminho da sequência até a função `async`?

Várias resoluções possíveis a esse debate surgiram: 

* `funções async` não seriam canceláveis de maneira alguma (status quo)
* Um "token de cancelamento" poderia ser passado como argumento a uma função async na hora da função ser chamada
* O valor retornado se tornaria um tipo de *promise* cancelável que é adicionado 
* O valor retornado se tornaria algo que não uma *promise* (por exemplo: observável, ou um token de controle com capacidades de *promise* e cancelamento) 

No momento em que escrevo, `funções async` devolvem *promises* normais, então é menos provável que o valor retornado irá mudar completamente. Entretanto, é muito cedo para saber como as coisas vão terminar. Fique de olho nessa discussão.

## `Object.observe(..)`

Um dos cálices sagrados do desenvolvimento front-end é data binding -- escutar por atualizações de um objeto e sincronizar a representação do DOM de acordo com aquele dado. A maioria dos frameworks JS disponibilizam algum mecanismo para esse tipo de operação.

Provavelmente depois do ES6, nós vamos ver esse suporte diretamente na linguagem, via um utilitário chamado `Object.observe(..)`. Em essência, a ideia é que você pode configurar um listener para observar uma mudança de objeto, e chamar um callback toda vez que isso acontecer. Você pode então atualizar o DOM, por exemplo.

Há seis tipos de mudanças que você pode observar:

* add
* update
* delete
* reconfigure
* setPrototype
* preventExtensions

Por padrão, você será notificada de todos os tipos de mudanças, mas você pode filtrar apenas aqueles que te interessa.

Considere:

```js
var obj = { a: 1, b: 2 };

Object.observe(
	obj,
	function(changes){
		for (var change of changes) {
			console.log( change );
		}
	},
	[ "add", "update", "delete" ]
);

obj.c = 3;
// { name: "c", object: obj, type: "add" }

obj.a = 42;
// { name: "a", object: obj, type: "update", oldValue: 1 }

delete obj.b;
// { name: "b", object: obj, type: "delete", oldValue: 2 }
```

Além dos principais tipos de mudança `"add"`, `"update"`, e `"delete"`:

* O evento de mudança `"reconfigure"` é disparado se uma das propriedades do objecto é refonfigurada com `Object.defineProperty(..)`, como por exemplo mudar o atributo `writable`. Veja o título *this e Prototipagem de Objetos* dessa série para mais informações.

* O evento de mudança `"preventExtensions"` é disparado se tornamos o objeto não extensível via `Object.preventExtensions(..)`.
Dado que `Object.seal(..)` e `Object.freeze(..)` também implicam em `Object.preventExtensions(..)`, eles também vão disparar o evento de mudança correspondente. Além disso, os eventos de mudança `"reconfigure"` também vão ser disparados para cada propriedade no objeto.

* O evento de mudança `"setPrototype"` é disparado se o `[[Prototype]]` de um objeto é modificado, tanto configurando com o setter `__proto__`, ou usando `Object.setPrototypeOf(.)`.

Note que esses eventos de mudança são notificados imediatamente depois da mudança. Não confunda com proxies (veja o Capítulo 7) onde você pode interceptar as ações antes delas acontecerem. Object observation te deixa responder depois que uma ação (ou uma série de ações) ocorre.


### Eventos de Mudança Customizados

Além dos seis tipos de eventos nativos, você também pode ouvir e disparar eventos customizados.

Considere:

```js
function observer(changes){
	for (var change of changes) {
		if (change.type == "recalc") {
			change.object.c =
				change.object.oldValue +
				change.object.a +
				change.object.b;
		}
	}
}

function changeObj(a,b) {
	var notifier = Object.getNotifier( obj );

	obj.a = a * 2;
	obj.b = b * 3;

	// queue up change events into a set
	notifier.notify( {
		type: "recalc",
		name: "c",
		oldValue: obj.c
	} );
}

var obj = { a: 1, b: 2, c: 3 };

Object.observe(
	obj,
	observer,
	["recalc"]
);

changeObj( 3, 11 );

obj.a;			// 12
obj.b;			// 30
obj.c;			// 3
```

A série de mudanças (evento customizado `"recalc"`) foi colocado na fila para ser entregue ao observer, mas ainda não foi entregue, por isso `obj.c` ainda é `3`.

As mudanças são por padrão entregues no final do laço de evento atual (veja o título *Async & Performance* dessa série). Se você quer entregá-las imediatamente, use `Object.deliverChangeRecords(observer)`. Uma vez que os eventos de mudança são entregues, você pode observar `obj.c` atualizado como esperado:

```js
obj.c;			// 42
```

No exemplo anterior, nós chamamos `notifier.notify(..)` com registro completo do evento de mudanças. Uma forma alternativa de colocar os registros de mudança na fila é usar o `performChange(..)`, que separa o tipo de evento do resto das propriedades de registro (via uma função callback). Considere:

```js
notifier.performChange( "recalc", function(){
	return {
		name: "c",
		// `this` is the object under observation
		oldValue: this.c
	};
} );
```

Em algumas circunstâncias, a separação de responsabilidades pode resultar em mais clareza para seu padrão de uso.

### Parando Observações

Assim como eventos de escuta normais, você também pode querer parar de observar a mudança de evento de um objeto. Para isso, use `Object.unobserve(..)`.

Por exemplo:

```js
var obj = { a: 1, b: 2 };

Object.observe( obj, function observer(changes) {
	for (var change of changes) {
		if (change.type == "setPrototype") {
			Object.unobserve(
				change.object, observer
			);
			break;
		}
	}
} );
```

Nesse exemplo simples, escutamos por eventos de mudança até ver o evento `"setPrototype"` aparecer, e aí paramos de observar qualquer mudança de eventos.

## Operador de Exponenciação

Um operador foi proposto ao JavaScript para aumentar a performance de exponenciação da mesma maneira que o `Math.pow(..)` faz. Considere:

```js
var a = 2;

a ** 4;			// Math.pow( a, 4 ) == 16

a **= 3;		// a = Math.pow( a, 3 )
a;				// 8
```

**Nota:** `**` é essencialmente o mesmo que aparece em Python, Ruby, Perl, e outras linguagens.

## Propriedades de Objetos e `...`

Como vimos na seção "Muito, Pouco, Suficiente" do capítulo 2, o operador `...` é bem óbvio em como ele se relaciona com o spreading ou gathering arrays. Mas e os objetos?

Essa funcionalidade foi considerada para ES6, mas foi postergada para ser considerada depois do ES6 (conhecido como "ES7" ou "ES2016" ou ...). Aqui está como provavelmente irá funcionar nos tempos "além do ES6":

```js
var o1 = { a: 1, b: 2 },
	o2 = { c: 3 },
	o3 = { ...o1, ...o2, d: 4 };

console.log( o3.a, o3.b, o3.c, o3.d );
// 1 2 3 4
```

O operador `...` provavelmente também será usado para juntar as propriedades de um objeto desestruturado em um objeto:

```js
var o1 = { b: 2, c: 3, d: 4 };
var { b, ...o2 } = o1;

console.log( b, o2.c, o2.d );		// 2 3 4
```

Aqui, o `...o2` recolhe as propriedades desestruturadas `c` e `d` para um objeto `o2` (`o2` não tem uma propriedade `b` como `o1` tem).

De novo, essas são apenas propostas sendo consideradas no pós-ES6. Mas será bem legal se elas realmente existirem.

## `Array#includes(..)`

Uma tarefa extremamente comum que pessoas desenvolvedoras JS precisam fazer é procurar por um valor dentro de um array de valores. A maneira que elas tem feito é:

```js
var vals = [ "foo", "bar", 42, "baz" ];

if (vals.indexOf( 42 ) >= 0) {
	// found it!
}
```

A razão para a checagem `>= 0` é porque `indexOf(..)` retorna um valor numérico de `0` ou maior se encontrado, ou `-1` se não encontrado. Em outras palavras, estamos usando um índex do retorno da função em um contexto booleano. Mas já que `-1` é verdadeiro ao invés de falso, temos que ser mais manuais nas nossas checagens.

No título *Tipos e Gramática* dessa série, eu explorei outro padrão que eu prefiro:

```js
var vals = [ "foo", "bar", 42, "baz" ];

if (~vals.indexOf( 42 )) {
	// found it!
}
```

O operador `~` aqui adapta o valor de retorno do `indexOf(..)` para um intervalo de valor que é facilmente convertível para booleano. Isso é, `01` produz 0 (falso), e qualquer outra coisa produz um valor não-zero (verdadeiro), que é o que usamos para decidir se encontramos o valor ou não.

Enquanto eu acho que isso é um avanço, outras pessoas discordam fortemente. Porém, ninguém pode dizer que a lógica de busca do `indexOf(..)` é perfeita. Ela falha em encontrar valores `NaN` no array, por exemplo.

Então apareceu uma proposta e ganhou muito suporte porque adiciona um retorno booleano real no método de busca, chamado `includes(..)`:

```js
var vals = [ "foo", "bar", 42, "baz" ];

if (vals.includes( 42 )) {
	// found it!
}
```

**Nota:** `Array#includes(..)` usa logica de combinação que vai encontrar valores `NaN`, mas não vai distinguir entre `-0` e `0` (veja o título *Tipos e Gramática* dessa série). Se você não se importa com os valores `-0` nos seus programas, isso provavelmente vai ser exatamente o que você está esperando. Se você *sim* se importa com o `-0`, você vai precisar fazer a sua própria lógica de busca, provavelmente usando o utilitário `Object.is(..)` (veja o Capítulo 6). 

## SIMD

Nós abordamos a Instrução Única, Múltiplos Dados (SIMD - Single Instruction, Multiple Data) em mais detalhes no título *Async e Performance* dessa série, mas essa é uma rápida menção, já que é uma das funcionalidades que provavelmente serão lançadas em um futuro JS.

A SIMD API expõe várias instruções de baixo nível (CPU) que podem operar em mais de um único valor ao mesmo tempo. Por exemplo, você poderá especificar dois *vetores* de 4 ou 8 números cada, e multiplicar os respectivos elementos de uma vez só (paralelismo de dados!).

Considere:

```js
var v1 = SIMD.float32x4( 3.14159, 21.0, 32.3, 55.55 );
var v2 = SIMD.float32x4( 2.1, 3.2, 4.3, 5.4 );

SIMD.float32x4.mul( v1, v2 );
// [ 6.597339, 67.2, 138.89, 299.97 ]
```

SIMD irá incluir várias outras operações além de `mul(..)` (multiplicação), como `sub()`, `div()`, `abs()`, `neg()`, `sqrt()`, e muito mais. 

Operações matemáticas paralelas são critícas para a próxima geração de aplicações JS de alta performance.

## WebAssembly (WASM)

Brendan Eich fez um anúncio quente tardio próximo da finalização da primeira edição desse título que tem potencial para impactar significantemente o futuro do JavaScript: WebAssembly (WASM). Nós não vamos conseguir cobrir WASM em detalhes aqui, já que está muito recente no momento atual que escrevo. Mas esse título estaria incompleto sem ao menos uma menção do assunto.

Uma das pressões mais fortes nas mudanças de desenho recentes (e futuro próximo) da linguagem tem sido o desejo de que ela se torne um alvo mais apropriado para a transpilação/cross-compilação vindo de outras linguagens (como C/C++, ClojureScript, etc.). Óbvio, a performance do código rodando em JavaScript tem sido uma preocupação primária.

Como discutido no título *Async e Performance* dessa série, há alguns anos um grupo de desenvolvedores na Mozilla introduziu uma ideia ao JavaScript chamada ASM.js. ASM.js é um subconjunto de JS válido que basicamente restringe algumas ações que deixam o código dificíl para a engine JS otimizar. O resultado é que o código compatível ASM.js rodando em uma engine ASM-consciente pode rodar notavelmente mais rápido, junto com código nativo e otimizado em C e equivalentes. Muitas pessoas viram ASM.js como muito provavelmente backbone em que aplicações necessitadas por performance iriam alavancar em JavaScript.

Em outras palavras, todos os caminhos para rodar código no navegador *liderado pelo JavaScript*.

Isso é, antes do anúncio do WASM. WASM provê um caminho alternativo para outras linguagens mirarem no ambiente runtime do navegador sem ter que passar pelo JavaScript. Em essência, se o WASM pegar, os motores JS vão ter uma capacidade extra de executar um formato binário de código que pode ser visto como algo similar ao bytecode (parecido com o que roda na JVM).

WASM propõe um formato para uma representação binária de uma AST (sintaxe de árvore) altamente compactada de código, que pode então dar instruções diretamente para o motor de JS e sua base, sem ter que ser parseado para JS, ou até mesmo se comportar como as regras de JS. Linguagens como C ou C++ podem ser compiladas diretamente para o formato WASM ao invés de ASM.js e ganhar velocidade extra e a vantagem de pular o parseamento para JS.

O curto prazo para WASM é estar similar ao ASM.js e o JS real. Mas eventualmente, é esperado que WASM possa ter novas capacidades que passam qualquer coisa que o JS pode fazer. Por exemplo, a pressão para o JS desenvolver funcionalidades radicais como threads -- uma mudança que certamente enviam uma onda de choque para o ecossistema JavaScript -- tem um futuro mais provável como uma extensão futura do WASM, aliviando a pressão de mudança do JS.

Na verdade, esse novo roteiro abre muitos novos caminhos para várias linguagens atingirem a web runtime. Esse é um caminho empolgante para a plataforma web! 

E o que isso significa para o JS? JS vai se tornar irrelevante ou "morrer"? Claro que não. ASM.js provavelmente não vai ter muito futuro nos próximos anos, mas a maior parte do JS está segura e ancorada na história da plataforma web.

Defensores do WASM sugerirem o seu sucesso vai significar que o desenho do JS vai ser protegido de pressões que eventualmente teriam gerado um stress além de presumir uma ruptura de razoabilidade. De acordo com as previsões WASM vai se tornar o alvo preferido para partes de aplicações que precisam de alta performance, como acontece em várias outras linguagens.

Curiosamente, JavaScript é uma das linguagens que menos visam atingir WASM no futuro. Haverão outras mudanças futuras que constroem possibilidades do JS ser sustentável para esse fim, mas esse caminho não parece estar no topo da lista de prioridades.

Já que JavaScript não vai ser um funil de WASM, os códigos JS e WASM serão capazes de interoperar de várias formas, assim como interações entre módulos. Você pode imaginar chamar uma função JS como `foo()` e na verdade invocar uma função WASM com esse nome com o poder de rodar bem tirando as preocupações do restante do seu código JS.

As coisas que atualmente são escritas em JS provavelmente continuarão sendo escritas em JS, pelo menos pelo futuro próximo. As que são transpiladas para JS vão provavelmente eventualmente ao menos considerar usar WASM. Paras as coisas que precisam de muita performance com uma tolerância mínica para camadas de abstração, a escolha mais comum vai ser achar uma línguagem não-JS adequada para trabalhar, e então usar WASM.

Tem chances de que a mudança será devagar, e vai levar anos em contrução. WASM ser lançada na maioria das plataformas de navegadores provavelmente vai levar alguns anos, sendo otimista. Enquanto isso, o projeto WASM (https://github.com/WebAssembly) tem um polyfill prévio para demonstrar a prova de conceito para os princípios básicos.

Mas com o tempo, e enquanto WASM aprende novos truques que não são do JS, não é muito difíficl imaginar algumas aplicações que estão em JS atualmente sendo refatoradas para uma linguagem que é compatível com WASM. Por exemplo, as partes sensíveis de performance de frameworks, motores de jogos e outras ferramentas pesadas provavelmente terão benefícios com essas mudanças. Pessoas desenvolvedoras usando essas ferramentas em suas aplicações provavelmente não verão muita diferença no uso ou na integração, mas automaticamente terão as vantagens de performance e capacidades.

O que é certo é que quanto mais WASM se torna real, mais significante fica para a trajetória e desenho do JavaScript. Esse é provavelmente um dos tópicos mais importantes do "além do ES6" que as pessoas desenvolvedoras deveriam ficar de olho.

## Revisão

Se todos os outros livros nessa série essencialmente propõem esse desafio, "você (provavelmente) não sabe JS (tanto quanto você achava)", esse livro na verdade sugere que "você não sabe mais JS". O livro cobriu uma série de novas coisas adicionadas à linguagem em ES6. Essa é uma coleção emocionante de novas funcionalidades da linguagem e paradigmas que vão melhorar seus programas JS para sempre.

Mas JS não é só ES6! Nem perto disso. Já existem algumas novas funcionalidades em vários estágios do desenvolvimento para os tempos de "além do ES6". Nesse capítulo, nós brevemente vimos alguns dos mais prováveis candidatos a se juntar ao JS em breve.

`async function`s são açucar sintáticos poderosos além dos padrões de generators + promises (veja o Capítulo 4). `Object.observe(..)` adiciona suporte diretamente nativo para observar mudanças de eventos em objetos, o que é crítico para implementação de data binding. O operador de exponenciação `**`, o `...` para propriedades de objetos, e `Array#includes(..)` são todas melhoras simples mas úteis para os mecanismos existentes. Finalmente, SIMD segue em uma nova era de evolução de JS com alta performance.

Por mais que soe cliche, o futuro do JS é realmente brilhante! O desafio dessa série, e de fato desse livro, está estabelecida em todas as pessoas leitoras agora. O que você está esperando? É hora de aprender e explorar!
