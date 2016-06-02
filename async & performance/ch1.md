# You Don't Know JS: Async e Performance
# Capítulo 1: Assíncronia: Agora & Depois

Uma das partes mais importantes e mais mal compreendidas da programação em uma linguagem como JavaScript, é como expressar e
manipular comportamentos de um programa que é esticado através de um período de tempo.

Não se trata apenas sobre o que acontece entre início e o fim de um loop `for`, o que é claro leva *algum tempo* (microsegundos à milisegundos) para completar. Se trata do que acontece quando parte do seu programa é executada *agora*, e outra parte do seu programa é executada *depois*  -- existe um vão entre *agora* e *depois* onde seu programa não está sendo executado ativamente.

Praticamente todos os programas não triviais (especialmente em JS) tem de alguma forma ou outra que lidar com esse vão, seja ao esperar pelo input do usuário, requisitando dados de um banco de dados ou sistema de arquivos, enviando dados através da rede e esperando por uma resposta, ou repetindo uma tarefa em um período de tempo intervalado (como em uma animação). De todas essas formas, seu programa tem que gerenciar estado através do vão no tempo. Como famigeradamente se diz em Londres (sobre o vão entre o vagão e a plataforma): "cuide o vão".

Na verdade, o relacionamento entre as partes *agora* e *depois* do seu programa estão no coração da programação assíncrona.

Programação assíncrona já existia desde o início de JS, claro. Mas a maioria dos desenvolvedores JS nunca consideraram com cuidado como e por que ela acontece repentinamente em seus programas, ou exploraram diversas *outras* maneiras de lidar com ela. O método "bom o suficiente" sempre foi a humilde função de retorno (callback). Até hoje, muitos insistem que elas são boas o suficiente.

Mas enquanto JS continua a crescer em escopo e complexidade, para alcançar as demandas sempre crescentes de uma linguagem de programação de primeira classe que roda em navegadores e servidores e em todos os dispositivos concebíveis entre eles, as dores com as quais lidamos com assincronia estão ficando cada vez mais debilitantes, e elas suplicam por métodos que são mais capazes e razoáveis.

Enquanto isso tudo pode parecer abstrato agora, eu garanto que vamos cuidar disso enquanto prosseguimos através do livro. Exploraremos uma variedade de técnicas emergentes para JavaScript assíncrono nos próximos capítulos.

Mas antes de podermos chegar lá, vamos ter que entender de uma forma mais aprofundada o que é assincronia e como opera em JS.

## Um Programa em Pedaços

Você pode escrever seu programa JS em um arquivo *.js*, mas ele é certamente feito de diversos pedaços, do qual apenas um pedaço é executado *agora*, sendo o restante executado *depois*. A unidade mais comum do *pedaço* é a `função`.

O problema que a maioria do desenvolvedores novatos em JS parecem ter, é que *depois* não acontece estritamente e imediatamente depois de *agora*. Em outras palavras, tarefas que não podem completar *agora* serão, por definição, completas de forma assíncrona, não possuindo assim comportamento bloqueante que você intuitivamente espera ou quer.

Considere:

```js
// ajax(..) é uma função Ajax arbitrária fornecida por uma biblioteca
var data = ajax( "http://alguma.url.1" );

console.log( data );
// Oops! `data` geralmente não conterá os resultados do Ajax
```

Você provavelmente está ciente de que requisições Ajax não completam de forma síncrona, o que significa que a função `ajax(...)` ainda não possui nenhum valor para retornar, para então ser atribuido a variável `data`. Se `ajax(...)` *pudesse* bloquear a execução até o retorno da resposta, então a atribuição `data = ..` funcionaria sem problemas.

Mas não é assim que se faz Ajax. Fazemos uma requisição assíncrona *agora*, e não possuiremos o resultado de volta até *depois*.

A forma mais simples (mas definitivamente não a única, ou nem mesmo a melhor!) de "esperar" de *agora* até *depois* é usando uma função, comumente conhecida como função de callback (retorno).

```js
// ajax(..) é uma função Ajax arbitrária fornecida por uma biblioteca
ajax( "http://alguma.url.1", function minhaFuncaoDeRetorno(data) {
	console.log(data); // Yay, temos dados!`!
});
```
**Atenção:** você pode ter ouvido que é possível fazer requisições Ajax síncronas. Enquanto isso é tecnicamente verdade, você não deveria fazer isso nunca, em nenhuma circunstância, por que tranca a interface do usuário no navegador (botões, menus, rolamento, etc.) e previne qualquer interação. Essa é uma péssima ideia, e deve sempre ser evitada.

Antes que você proteste em discordância, não, seu desejo de evitar a confusão dos callbacks *não* é justificativa para Ajax síncrono bloqueante.

Por exemplo, considere esse código

```js
function agora () {
  return 21;
}

function depois () {
  resposta = resposta * 2;
  console.log( "Significado da vida:", resposta);
}

var resposta = agora();

setTimeout( depois, 1000); // Significado da vida: 42
```

Existem duas partes para esse programa: a coisa que vai executar *agora*, e a coisa que vai executar *depois*. Está bem óbvio qual é qual, mas vamos ser super explícitos:

Agora:
```js
function agora() {
	return 21;
}

function depois() { .. }

var resposta = agora();

setTimeout( depois, 1000 );
```

Depois:
```js
resposta = resposta * 2;
console.log( "Significado da vida:", resposta );
```

O pedaço *agora* roda de imediato, tão cedo você executar o programa. Mas `setTimeout(...)` também define um evento (um tempo limite) para acontecer *depois*, de maneira que os conteúdos da função `depois()` serão executados posteriormente (1.000 milisegundos a partir de agora). 

Toda vez que você encapsula uma porção de código numa `função` e especifica que ela deve ser executada como resposta a algum evento (timer, clique do mouse, resposta do Ajax, etc.), você está criando um pedaço *posterior (depois)* do seu código, assim introduzindo assincronia ao seu programa.

### Async Console

Não existe especificação ou grupo de requerimentos sobre como os métodos `console.*` funcionam -- eles não são parte oficial do JavaScript, mas ao invés disso são adicionados ao JS pelo *ambiente hospedeiro* (veja o título *Tipos & Gramática* dessa série de livros).

Então, diferentes navegadores e ambientes JS fazem da maneira que preferem, o que pode as vezes causar comportamento confuso.

Em particular, existem alguns navegadores e condições nas quais `console.log(..)` não retorna imediatamente o que é dado. 
O principal motivo pelo qual isso pode acontecer é por que o I/O é uma parte lenta e bloqueante de muitos programas (não apenas JS). Então, pode ser melhor (do ponto de vista página/UI) para um navegador lidar com o I/O do `console` de maneira assíncrona no plano de fundo, sem que você talvez sequer saiba o que ocorreu.

Um cenário improvável, mas possível, de onde isso pode ser *observado* (não através do código em si, mas de fora):

```js
var a = {
  index: 1
};

//depois 
console.log(a); //?

//mais depois
a.index++;

```

Normalmente esperaríamos ver o objeto `a` fotografado no exato momento da instrução, imprimindo algo como `{ index: 1 }`, tal qual na próxima instrução quando `a.index++` acontece, está modificando algo diferente, ou estritamente depois da saída de `a`.

Na maior parte do tempo, o código precedente provavelmente vai produzir uma representação de um objeto no console da seção de ferramentas do desenvolvedor do seu navegador (developer tools) da forma como você esperaria. Mas é possível que o mesmo código execute em uma situação onde o navegador sentiu que precisafa deferir o I/O do console para o plano de fundo, e que nesse caso seja *possível* que no momento em que o objeto tenha sido representado no console do navegador, `a.index++` já tenha acontecido, e mostre `{ index: 2 }`.

As condições exatas nas quais o I/O do `console` será deferido, ou mesmo se será observável ou não é oscilante. Apenas se lembre dessa possível "assincronicidade" no I/O no caso de você se deparar com problemas ao tentar debugar o código em situações onde o objeto foi modificado *depois* da instrução `console.log(..)`, e ainda assim as modificações inesperadas apareçam.

**Nota:** Se você se deparar com esse cenário raro, a melhor opção é usar pontos de quebra no seu debugados JS ao invés de contar com a saída do `console`. A segunda melhor opção seria forçar uma fotografia do objeto em questão ao serializá-lo em uma `string`, como com 'JSON.stringidy(..)`.

## Laço de Eventos (Event Loop)

Vamos fazer uma afirmação (talvez chocante): apesar de claramente permitir código assíncrono (como o timeout que acabamos de ver), até recentmente (ES6), JavaScript nunca de verdade teve uma noção direta de assíncronia intrínseca.

**O que!?** Isso parece uma afirmação louca, certo? Na verdade, é verdade. O motor JS nunca fez nada além de executar um pedaço único de código do seu programa em um dado momento qualquer, quando solicitado.

"Quando solicitado." Por quem? Essa é a parte importante!

O motor JS não roda isolado. Roda dentro de um *ambiente hospedeiro*, que é para muitos desenvolvedores o típico navegador da web.
Através do últimos muitos anos (de nenhuma forma exclusivamente), JS expandiou além do navegador em outros ambientes, tal qual servidores, através de coisas como Node.js. Na verdade, JavaScript é anexado nos mais variados tipos de dispositivos hoje em dia, de robos à lampadas.

Mas uma parte comum de todos esses ambientes é que eles tem um mecanismo que lida com a execução de múltiplos pedaços do seu programa *ao longo do tempo*, a cada momento invocando o motor JS chamado "loop de eventos".

Em outras palavras, o motor JS não tem senso inato de *tempo*, mas está ao invés disso, em um ambiente de execução on-demand para qualquer código arbitrário de JS. É o ambiente periférico que tem sempre *agendado* os "eventos" (execuções de código JS).

Então, por exemplo, quando seu programa JS faz uma requisição Ajax para pegar algum dado de um servidor, você define o código de "resposta" em uma função (comumente chamada de "callback"), e o motor JS informa ao ambiente hospedeiro, "Ei, eu vou suspender a execução agora, mas assim que você terminar com aquela requisição na rede, e você tiver algum dado, por favor *chame* (call), essa função *de volta* (back)."

O navegador então fica a postos escutando pela resposta do servidor, e quando tem algo para te dar, agenda a função de callback (retorno) para ser executada ao inserí-la no *laço de eventos* (event loop).

Então o que é o *laço de eventos* (event loop)?

Vamos contextualizar através de um (meio que) pseudo código:

```js
 // eventLoop é uma array que age como uma lista de espera (primeiro a entrar, primeiro a sair)
 var eventLoop = [];
 var event;
 
 // continua indo "para sempre"
 while (true) {
   //faz a verificação
   if(eventLoop.length > 0) {
   	// insere o próximo evento na fila
   	event = eventLoop.shift();
   	
   	// agora, executa o próximo evento
   	try {
   	   event();
   	} 
   	catch (err) {
   	   reportError(err);
   	}
   }
 }
```

Esse é um pseudo código muito simplificado para ilustrar o conceito. Mas deve ser suficiente para ajudar a ter um entendimento melhor.

Como você pode ver, existe um laço contínuo representado pelo laço `while`, e cada iteração desse laço é chamada de um "tick". Para cada tick, se um evento está esperando na fila, é removido e executado. Esses eventos são as suas funções de callback (retorno).

É importante notar que `setTimeout(..)` não coloca sua função de callback na lista de espera do laço de eventos (event loop). O que faz é definir um contador; quando o contador expira, o ambiente coloca sua função de callback no laço de eventos, de modo que um tick futuro vai pegá-lo e executá-lo.

E se já existem 20 itens no laço de eventos naquele momento? Sua função de callback espera. Fica na fila atrás das outras -- normalmente não existe uma maneira de antecipar a espera e pular na frente da fila. Isso explica porque temporizadores `setTimeout(..)` podem não acionar com precisão temporal perfeita. É garantido (de maneira geral) que a sua função de retorno não vai acionar *antes* do tempo especificado, mas pode acontecer depois, dependendo do estado da fila de eventos.

Então, em outras palavras, seu programa é geralmente quebrado em diversos pequenos pedaços, que podem acontecer um após o outro na lista de espera do laço de eventos. E tecnicamente, outros eventos não relacionados diretamente com o seu programa também podem ser alternados dentro da fila.


**Nota:** Mencionamos "até recentemente" em relação ao ES6 mudando a natureza de onde laço de eventos é gerenciado. É em sua maioria uma tecnicalidade formal, mas o ES6 agora especifica como o laço de eventos funciona, o que significa que está dentro do alcance do motor JS, ao invés de apenas no *ambiente hospedeiro*. Um motivo principal para essa mudança é a introdução das Promises do ES6, as quais discutiremos no Capítulo 3, por que elas requerem a habilidade de ter controle direto e preciso do agendamento de operações na lista de espera do laço de eventos.

<hr>

**Note:** We mentioned "up until recently" in relation to ES6 changing the nature of where the event loop queue is managed. It's mostly a formal technicality, but ES6 now specifies how the event loop works, which means technically it's within the purview of the JS engine, rather than just the *hosting environment*. One main reason for this change is the introduction of ES6 Promises, which we'll discuss in Chapter 3, because they require the ability to have direct, fine-grained control over scheduling operations on the event loop queue (see the discussion of `setTimeout(..0)` in the "Cooperation" section).

## Parallel Threading

It's very common to conflate the terms "async" and "parallel," but they are actually quite different. Remember, async is about the gap between *now* and *later*. But parallel is about things being able to occur simultaneously.

The most common tools for parallel computing are processes and threads. Processes and threads execute independently and may execute simultaneously: on separate processors, or even separate computers, but multiple threads can share the memory of a single process.

An event loop, by contrast, breaks its work into tasks and executes them in serial, disallowing parallel access and changes to shared memory. Parallelism and "serialism" can coexist in the form of cooperating event loops in separate threads.

The interleaving of parallel threads of execution and the interleaving of asynchronous events occur at very different levels of granularity.

For example:

```js
function later() {
	answer = answer * 2;
	console.log( "Meaning of life:", answer );
}
```

While the entire contents of `later()` would be regarded as a single event loop queue entry, when thinking about a thread this code would run on, there's actually perhaps a dozen different low-level operations. For example, `answer = answer * 2` requires first loading the current value of `answer`, then putting `2` somewhere, then performing the multiplication, then taking the result and storing it back into `answer`.

In a single-threaded environment, it really doesn't matter that the items in the thread queue are low-level operations, because nothing can interrupt the thread. But if you have a parallel system, where two different threads are operating in the same program, you could very likely have unpredictable behavior.

Consider:

```js
var a = 20;

function foo() {
	a = a + 1;
}

function bar() {
	a = a * 2;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

In JavaScript's single-threaded behavior, if `foo()` runs before `bar()`, the result is that `a` has `42`, but if `bar()` runs before `foo()` the result in `a` will be `41`.

If JS events sharing the same data executed in parallel, though, the problems would be much more subtle. Consider these two lists of pseudocode tasks as the threads that could respectively run the code in `foo()` and `bar()`, and consider what happens if they are running at exactly the same time:

Thread 1 (`X` and `Y` are temporary memory locations):
```
foo():
  a. load value of `a` in `X`
  b. store `1` in `Y`
  c. add `X` and `Y`, store result in `X`
  d. store value of `X` in `a`
```

Thread 2 (`X` and `Y` are temporary memory locations):
```
bar():
  a. load value of `a` in `X`
  b. store `2` in `Y`
  c. multiply `X` and `Y`, store result in `X`
  d. store value of `X` in `a`
```

Now, let's say that the two threads are running truly in parallel. You can probably spot the problem, right? They use shared memory locations `X` and `Y` for their temporary steps.

What's the end result in `a` if the steps happen like this?

```
1a  (load value of `a` in `X`   ==> `20`)
2a  (load value of `a` in `X`   ==> `20`)
1b  (store `1` in `Y`   ==> `1`)
2b  (store `2` in `Y`   ==> `2`)
1c  (add `X` and `Y`, store result in `X`   ==> `22`)
1d  (store value of `X` in `a`   ==> `22`)
2c  (multiply `X` and `Y`, store result in `X`   ==> `44`)
2d  (store value of `X` in `a`   ==> `44`)
```

The result in `a` will be `44`. But what about this ordering?

```
1a  (load value of `a` in `X`   ==> `20`)
2a  (load value of `a` in `X`   ==> `20`)
2b  (store `2` in `Y`   ==> `2`)
1b  (store `1` in `Y`   ==> `1`)
2c  (multiply `X` and `Y`, store result in `X`   ==> `20`)
1c  (add `X` and `Y`, store result in `X`   ==> `21`)
1d  (store value of `X` in `a`   ==> `21`)
2d  (store value of `X` in `a`   ==> `21`)
```

The result in `a` will be `21`.

So, threaded programming is very tricky, because if you don't take special steps to prevent this kind of interruption/interleaving from happening, you can get very surprising, nondeterministic behavior that frequently leads to headaches.

JavaScript never shares data across threads, which means *that* level of nondeterminism isn't a concern. But that doesn't mean JS is always deterministic. Remember earlier, where the relative ordering of `foo()` and `bar()` produces two different results (`41` or `42`)?

**Note:** It may not be obvious yet, but not all nondeterminism is bad. Sometimes it's irrelevant, and sometimes it's intentional. We'll see more examples of that throughout this and the next few chapters.

### Run-to-Completion

Because of JavaScript's single-threading, the code inside of `foo()` (and `bar()`) is atomic, which means that once `foo()` starts running, the entirety of its code will finish before any of the code in `bar()` can run, or vice versa. This is called "run-to-completion" behavior.

In fact, the run-to-completion semantics are more obvious when `foo()` and `bar()` have more code in them, such as:

```js
var a = 1;
var b = 2;

function foo() {
	a++;
	b = b * a;
	a = b + 3;
}

function bar() {
	b--;
	a = 8 + b;
	b = a * 2;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

Because `foo()` can't be interrupted by `bar()`, and `bar()` can't be interrupted by `foo()`, this program only has two possible outcomes depending on which starts running first -- if threading were present, and the individual statements in `foo()` and `bar()` could be interleaved, the number of possible outcomes would be greatly increased!

Chunk 1 is synchronous (happens *now*), but chunks 2 and 3 are asynchronous (happen *later*), which means their execution will be separated by a gap of time.

Chunk 1:
```js
var a = 1;
var b = 2;
```

Chunk 2 (`foo()`):
```js
a++;
b = b * a;
a = b + 3;
```

Chunk 3 (`bar()`):
```js
b--;
a = 8 + b;
b = a * 2;
```

Chunks 2 and 3 may happen in either-first order, so there are two possible outcomes for this program, as illustrated here:

Outcome 1:
```js
var a = 1;
var b = 2;

// foo()
a++;
b = b * a;
a = b + 3;

// bar()
b--;
a = 8 + b;
b = a * 2;

a; // 11
b; // 22
```

Outcome 2:
```js
var a = 1;
var b = 2;

// bar()
b--;
a = 8 + b;
b = a * 2;

// foo()
a++;
b = b * a;
a = b + 3;

a; // 183
b; // 180
```

Two outcomes from the same code means we still have nondeterminism! But it's at the function (event) ordering level, rather than at the statement ordering level (or, in fact, the expression operation ordering level) as it is with threads. In other words, it's *more deterministic* than threads would have been.

As applied to JavaScript's behavior, this function-ordering nondeterminism is the common term "race condition," as `foo()` and `bar()` are racing against each other to see which runs first. Specifically, it's a "race condition" because you cannot predict reliably how `a` and `b` will turn out.

**Note:** If there was a function in JS that somehow did not have run-to-completion behavior, we could have many more possible outcomes, right? It turns out ES6 introduces just such a thing (see Chapter 4 "Generators"), but don't worry right now, we'll come back to that!

## Concurrency

Let's imagine a site that displays a list of status updates (like a social network news feed) that progressively loads as the user scrolls down the list. To make such a feature work correctly, (at least) two separate "processes" will need to be executing *simultaneously* (i.e., during the same window of time, but not necessarily at the same instant).

**Note:** We're using "process" in quotes here because they aren't true operating system–level processes in the computer science sense. They're virtual processes, or tasks, that represent a logically connected, sequential series of operations. We'll simply prefer "process" over "task" because terminology-wise, it will match the definitions of the concepts we're exploring.

The first "process" will respond to `onscroll` events (making Ajax requests for new content) as they fire when the user has scrolled the page further down. The second "process" will receive Ajax responses back (to render content onto the page).

Obviously, if a user scrolls fast enough, you may see two or more `onscroll` events fired during the time it takes to get the first response back and process, and thus you're going to have `onscroll` events and Ajax response events firing rapidly, interleaved with each other.

Concurrency is when two or more "processes" are executing simultaneously over the same period, regardless of whether their individual constituent operations happen *in parallel* (at the same instant on separate processors or cores) or not. You can think of concurrency then as "process"-level (or task-level) parallelism, as opposed to operation-level parallelism (separate-processor threads).

**Note:** Concurrency also introduces an optional notion of these "processes" interacting with each other. We'll come back to that later.

For a given window of time (a few seconds worth of a user scrolling), let's visualize each independent "process" as a series of events/operations:

"Process" 1 (`onscroll` events):
```
onscroll, request 1
onscroll, request 2
onscroll, request 3
onscroll, request 4
onscroll, request 5
onscroll, request 6
onscroll, request 7
```

"Process" 2 (Ajax response events):
```
response 1
response 2
response 3
response 4
response 5
response 6
response 7
```

It's quite possible that an `onscroll` event and an Ajax response event could be ready to be processed at exactly the same *moment*. For example let's visualize these events in a timeline:

```
onscroll, request 1
onscroll, request 2          response 1
onscroll, request 3          response 2
response 3
onscroll, request 4
onscroll, request 5
onscroll, request 6          response 4
onscroll, request 7
response 6
response 5
response 7
```

But, going back to our notion of the event loop from earlier in the chapter, JS is only going to be able to handle one event at a time, so either `onscroll, request 2` is going to happen first or `response 1` is going to happen first, but they cannot happen at literally the same moment. Just like kids at a school cafeteria, no matter what crowd they form outside the doors, they'll have to merge into a single line to get their lunch!

Let's visualize the interleaving of all these events onto the event loop queue.

Event Loop Queue:
```
onscroll, request 1   <--- Process 1 starts
onscroll, request 2
response 1            <--- Process 2 starts
onscroll, request 3
response 2
response 3
onscroll, request 4
onscroll, request 5
onscroll, request 6
response 4
onscroll, request 7   <--- Process 1 finishes
response 6
response 5
response 7            <--- Process 2 finishes
```

"Process 1" and "Process 2" run concurrently (task-level parallel), but their individual events run sequentially on the event loop queue.

By the way, notice how `response 6` and `response 5` came back out of expected order?

The single-threaded event loop is one expression of concurrency (there are certainly others, which we'll come back to later).

### Noninteracting

As two or more "processes" are interleaving their steps/events concurrently within the same program, they don't necessarily need to interact with each other if the tasks are unrelated. **If they don't interact, nondeterminism is perfectly acceptable.**

For example:

```js
var res = {};

function foo(results) {
	res.foo = results;
}

function bar(results) {
	res.bar = results;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

`foo()` and `bar()` are two concurrent "processes," and it's nondeterminate which order they will be fired in. But we've constructed the program so it doesn't matter what order they fire in, because they act independently and as such don't need to interact.

This is not a "race condition" bug, as the code will always work correctly, regardless of the ordering.

### Interaction

More commonly, concurrent "processes" will by necessity interact, indirectly through scope and/or the DOM. When such interaction will occur, you need to coordinate these interactions to prevent "race conditions," as described earlier.

Here's a simple example of two concurrent "processes" that interact because of implied ordering, which is only *sometimes broken*:

```js
var res = [];

function response(data) {
	res.push( data );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

The concurrent "processes" are the two `response()` calls that will be made to handle the Ajax responses. They can happen in either-first order.

Let's assume the expected behavior is that `res[0]` has the results of the `"http://some.url.1"` call, and `res[1]` has the results of the `"http://some.url.2"` call. Sometimes that will be the case, but sometimes they'll be flipped, depending on which call finishes first. There's a pretty good likelihood that this nondeterminism is a "race condition" bug.

**Note:** Be extremely wary of assumptions you might tend to make in these situations. For example, it's not uncommon for a developer to observe that `"http://some.url.2"` is "always" much slower to respond than `"http://some.url.1"`, perhaps by virtue of what tasks they're doing (e.g., one performing a database task and the other just fetching a static file), so the observed ordering seems to always be as expected. Even if both requests go to the same server, and *it* intentionally responds in a certain order, there's no *real* guarantee of what order the responses will arrive back in the browser.

So, to address such a race condition, you can coordinate ordering interaction:

```js
var res = [];

function response(data) {
	if (data.url == "http://some.url.1") {
		res[0] = data;
	}
	else if (data.url == "http://some.url.2") {
		res[1] = data;
	}
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

Regardless of which Ajax response comes back first, we inspect the `data.url` (assuming one is returned from the server, of course!) to figure out which position the response data should occupy in the `res` array. `res[0]` will always hold the `"http://some.url.1"` results and `res[1]` will always hold the `"http://some.url.2"` results. Through simple coordination, we eliminated the "race condition" nondeterminism.

The same reasoning from this scenario would apply if multiple concurrent function calls were interacting with each other through the shared DOM, like one updating the contents of a `<div>` and the other updating the style or attributes of the `<div>` (e.g., to make the DOM element visible once it has content). You probably wouldn't want to show the DOM element before it had content, so the coordination must ensure proper ordering interaction.

Some concurrency scenarios are *always broken* (not just *sometimes*) without coordinated interaction. Consider:

```js
var a, b;

function foo(x) {
	a = x * 2;
	baz();
}

function bar(y) {
	b = y * 2;
	baz();
}

function baz() {
	console.log(a + b);
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

In this example, whether `foo()` or `bar()` fires first, it will always cause `baz()` to run too early (either `a` or `b` will still be `undefined`), but the second invocation of `baz()` will work, as both `a` and `b` will be available.

There are different ways to address such a condition. Here's one simple way:

```js
var a, b;

function foo(x) {
	a = x * 2;
	if (a && b) {
		baz();
	}
}

function bar(y) {
	b = y * 2;
	if (a && b) {
		baz();
	}
}

function baz() {
	console.log( a + b );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

The `if (a && b)` conditional around the `baz()` call is traditionally called a "gate," because we're not sure what order `a` and `b` will arrive, but we wait for both of them to get there before we proceed to open the gate (call `baz()`).

Another concurrency interaction condition you may run into is sometimes called a "race," but more correctly called a "latch." It's characterized by "only the first one wins" behavior. Here, nondeterminism is acceptable, in that you are explicitly saying it's OK for the "race" to the finish line to have only one winner.

Consider this broken code:

```js
var a;

function foo(x) {
	a = x * 2;
	baz();
}

function bar(x) {
	a = x / 2;
	baz();
}

function baz() {
	console.log( a );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

Whichever one (`foo()` or `bar()`) fires last will not only overwrite the assigned `a` value from the other, but it will also duplicate the call to `baz()` (likely undesired).

So, we can coordinate the interaction with a simple latch, to let only the first one through:

```js
var a;

function foo(x) {
	if (a == undefined) {
		a = x * 2;
		baz();
	}
}

function bar(x) {
	if (a == undefined) {
		a = x / 2;
		baz();
	}
}

function baz() {
	console.log( a );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

The `if (a == undefined)` conditional allows only the first of `foo()` or `bar()` through, and the second (and indeed any subsequent) calls would just be ignored. There's just no virtue in coming in second place!

**Note:** In all these scenarios, we've been using global variables for simplistic illustration purposes, but there's nothing about our reasoning here that requires it. As long as the functions in question can access the variables (via scope), they'll work as intended. Relying on lexically scoped variables (see the *Scope & Closures* title of this book series), and in fact global variables as in these examples, is one obvious downside to these forms of concurrency coordination. As we go through the next few chapters, we'll see other ways of coordination that are much cleaner in that respect.

### Cooperation

Another expression of concurrency coordination is called "cooperative concurrency." Here, the focus isn't so much on interacting via value sharing in scopes (though that's obviously still allowed!). The goal is to take a long-running "process" and break it up into steps or batches so that other concurrent "processes" have a chance to interleave their operations into the event loop queue.

For example, consider an Ajax response handler that needs to run through a long list of results to transform the values. We'll use `Array#map(..)` to keep the code shorter:

```js
var res = [];

// `response(..)` receives array of results from the Ajax call
function response(data) {
	// add onto existing `res` array
	res = res.concat(
		// make a new transformed array with all `data` values doubled
		data.map( function(val){
			return val * 2;
		} )
	);
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

If `"http://some.url.1"` gets its results back first, the entire list will be mapped into `res` all at once. If it's a few thousand or less records, this is not generally a big deal. But if it's say 10 million records, that can take a while to run (several seconds on a powerful laptop, much longer on a mobile device, etc.).

While such a "process" is running, nothing else in the page can happen, including no other `response(..)` calls, no UI updates, not even user events like scrolling, typing, button clicking, and the like. That's pretty painful.

So, to make a more cooperatively concurrent system, one that's friendlier and doesn't hog the event loop queue, you can process these results in asynchronous batches, after each one "yielding" back to the event loop to let other waiting events happen.

Here's a very simple approach:

```js
var res = [];

// `response(..)` receives array of results from the Ajax call
function response(data) {
	// let's just do 1000 at a time
	var chunk = data.splice( 0, 1000 );

	// add onto existing `res` array
	res = res.concat(
		// make a new transformed array with all `chunk` values doubled
		chunk.map( function(val){
			return val * 2;
		} )
	);

	// anything left to process?
	if (data.length > 0) {
		// async schedule next batch
		setTimeout( function(){
			response( data );
		}, 0 );
	}
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

We process the data set in maximum-sized chunks of 1,000 items. By doing so, we ensure a short-running "process," even if that means many more subsequent "processes," as the interleaving onto the event loop queue will give us a much more responsive (performant) site/app.

Of course, we're not interaction-coordinating the ordering of any of these "processes," so the order of results in `res` won't be predictable. If ordering was required, you'd need to use interaction techniques like those we discussed earlier, or ones we will cover in later chapters of this book.

We use the `setTimeout(..0)` (hack) for async scheduling, which basically just means "stick this function at the end of the current event loop queue."

**Note:** `setTimeout(..0)` is not technically inserting an item directly onto the event loop queue. The timer will insert the event at its next opportunity. For example, two subsequent `setTimeout(..0)` calls would not be strictly guaranteed to be processed in call order, so it *is* possible to see various conditions like timer drift where the ordering of such events isn't predictable. In Node.js, a similar approach is `process.nextTick(..)`. Despite how convenient (and usually more performant) it would be, there's not a single direct way (at least yet) across all environments to ensure async event ordering. We cover this topic in more detail in the next section.

## Jobs

As of ES6, there's a new concept layered on top of the event loop queue, called the "Job queue." The most likely exposure you'll have to it is with the asynchronous behavior of Promises (see Chapter 3).

Unfortunately, at the moment it's a mechanism without an exposed API, and thus demonstrating it is a bit more convoluted. So we're going to have to just describe it conceptually, such that when we discuss async behavior with Promises in Chapter 3, you'll understand how those actions are being scheduled and processed.

So, the best way to think about this that I've found is that the "Job queue" is a queue hanging off the end of every tick in the event loop queue. Certain async-implied actions that may occur during a tick of the event loop will not cause a whole new event to be added to the event loop queue, but will instead add an item (aka Job) to the end of the current tick's Job queue.

It's kinda like saying, "oh, here's this other thing I need to do *later*, but make sure it happens right away before anything else can happen."

Or, to use a metaphor: the event loop queue is like an amusement park ride, where once you finish the ride, you have to go to the back of the line to ride again. But the Job queue is like finishing the ride, but then cutting in line and getting right back on.

A Job can also cause more Jobs to be added to the end of the same queue. So, it's theoretically possible that a Job "loop" (a Job that keeps adding another Job, etc.) could spin indefinitely, thus starving the program of the ability to move on to the next event loop tick. This would conceptually be almost the same as just expressing a long-running or infinite loop (like `while (true) ..`) in your code.

Jobs are kind of like the spirit of the `setTimeout(..0)` hack, but implemented in such a way as to have a much more well-defined and guaranteed ordering: **later, but as soon as possible**.

Let's imagine an API for scheduling Jobs (directly, without hacks), and call it `schedule(..)`. Consider:

```js
console.log( "A" );

setTimeout( function(){
	console.log( "B" );
}, 0 );

// theoretical "Job API"
schedule( function(){
	console.log( "C" );

	schedule( function(){
		console.log( "D" );
	} );
} );
```

You might expect this to print out `A B C D`, but instead it would print out `A C D B`, because the Jobs happen at the end of the current event loop tick, and the timer fires to schedule for the *next* event loop tick (if available!).

In Chapter 3, we'll see that the asynchronous behavior of Promises is based on Jobs, so it's important to keep clear how that relates to event loop behavior.

## Statement Ordering

The order in which we express statements in our code is not necessarily the same order as the JS engine will execute them. That may seem like quite a strange assertion to make, so we'll just briefly explore it.

But before we do, we should be crystal clear on something: the rules/grammar of the language (see the *Types & Grammar* title of this book series) dictate a very predictable and reliable behavior for statement ordering from the program point of view. So what we're about to discuss are **not things you should ever be able to observe** in your JS program.

**Warning:** If you are ever able to *observe* compiler statement reordering like we're about to illustrate, that'd be a clear violation of the specification, and it would unquestionably be due to a bug in the JS engine in question -- one which should promptly be reported and fixed! But it's vastly more common that you *suspect* something crazy is happening in the JS engine, when in fact it's just a bug (probably a "race condition"!) in your own code -- so look there first, and again and again. The JS debugger, using breakpoints and stepping through code line by line, will be your most powerful tool for sniffing out such bugs in *your code*.

Consider:

```js
var a, b;

a = 10;
b = 30;

a = a + 1;
b = b + 1;

console.log( a + b ); // 42
```

This code has no expressed asynchrony to it (other than the rare `console` async I/O discussed earlier!), so the most likely assumption is that it would process line by line in top-down fashion.

But it's *possible* that the JS engine, after compiling this code (yes, JS is compiled -- see the *Scope & Closures* title of this book series!) might find opportunities to run your code faster by rearranging (safely) the order of these statements. Essentially, as long as you can't observe the reordering, anything's fair game.

For example, the engine might find it's faster to actually execute the code like this:

```js
var a, b;

a = 10;
a++;

b = 30;
b++;

console.log( a + b ); // 42
```

Or this:

```js
var a, b;

a = 11;
b = 31;

console.log( a + b ); // 42
```

Or even:

```js
// because `a` and `b` aren't used anymore, we can
// inline and don't even need them!
console.log( 42 ); // 42
```

In all these cases, the JS engine is performing safe optimizations during its compilation, as the end *observable* result will be the same.

But here's a scenario where these specific optimizations would be unsafe and thus couldn't be allowed (of course, not to say that it's not optimized at all):

```js
var a, b;

a = 10;
b = 30;

// we need `a` and `b` in their preincremented state!
console.log( a * b ); // 300

a = a + 1;
b = b + 1;

console.log( a + b ); // 42
```

Other examples where the compiler reordering could create observable side effects (and thus must be disallowed) would include things like any function call with side effects (even and especially getter functions), or ES6 Proxy objects (see the *ES6 & Beyond* title of this book series).

Consider:

```js
function foo() {
	console.log( b );
	return 1;
}

var a, b, c;

// ES5.1 getter literal syntax
c = {
	get bar() {
		console.log( a );
		return 1;
	}
};

a = 10;
b = 30;

a += foo();				// 30
b += c.bar;				// 11

console.log( a + b );	// 42
```

If it weren't for the `console.log(..)` statements in this snippet (just used as a convenient form of observable side effect for the illustration), the JS engine would likely have been free, if it wanted to (who knows if it would!?), to reorder the code to:

```js
// ...

a = 10 + foo();
b = 30 + c.bar;

// ...
```

While JS semantics thankfully protect us from the *observable* nightmares that compiler statement reordering would seem to be in danger of, it's still important to understand just how tenuous a link there is between the way source code is authored (in top-down fashion) and the way it runs after compilation.

Compiler statement reordering is almost a micro-metaphor for concurrency and interaction. As a general concept, such awareness can help you understand async JS code flow issues better.

## Review

A JavaScript program is (practically) always broken up into two or more chunks, where the first chunk runs *now* and the next chunk runs *later*, in response to an event. Even though the program is executed chunk-by-chunk, all of them share the same access to the program scope and state, so each modification to state is made on top of the previous state.

Whenever there are events to run, the *event loop* runs until the queue is empty. Each iteration of the event loop is a "tick." User interaction, IO, and timers enqueue events on the event queue.

At any given moment, only one event can be processed from the queue at a time. While an event is executing, it can directly or indirectly cause one or more subsequent events.

Concurrency is when two or more chains of events interleave over time, such that from a high-level perspective, they appear to be running *simultaneously* (even though at any given moment only one event is being processed).

It's often necessary to do some form of interaction coordination between these concurrent "processes" (as distinct from operating system processes), for instance to ensure ordering or to prevent "race conditions." These "processes" can also *cooperate* by breaking themselves into smaller chunks and to allow other "process" interleaving.
