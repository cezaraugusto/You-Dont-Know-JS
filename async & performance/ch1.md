# You Don't Know JS: Async e Performance
# Capítulo 1: Assincronia: Agora & Depois

Um dos assuntos mais importantes e ainda assim muitas vezes mal interpretado ao programar em uma linguagem como JavaScript, se trata de expressar e manipular a propagação do comportamento do programa executado durante um certo espaço de tempo.

Isso não significa saber o que acontece entre início e o fim de um loop `for`, o que, é claro, leva *algum tempo* (microsegundos à milisegundos) para completar. Assincronia diz mais a respeito do que acontece quando parte do seu programa é executada *agora*, e outra parte do seu programa é executada *depois*  -- existe um vão entre *agora* e *depois* onde seu programa não está sendo executado ativamente.

Praticamente todos os programas não-triviais (especialmente em JS) precisam lidar com esse vão de alguma forma, seja ao esperar pelo input do usuário, requisitando dados de um banco de dados ou sistema de arquivos, enviando dados através da rede e esperando por uma resposta ou repetindo uma tarefa em um período de tempo intervalado (como em uma animação). De todas essas formas, seu programa tem que gerenciar o estado através do vão ao longo do tempo. Assim como uma frase recorrente em Londres diz (em relação ao espaço entre o vagão do trem metropolitano e a plataforma de embarque): "cuidado com o vão".

Na verdade, o relacionamento entre as partes *agora* e *depois* do seu programa estão no coração da programação assíncrona.

Programação assíncrona já existia desde o início de JS, claro. Mas a maioria dos desenvolvedores JS nunca estudaram com cuidado como e por que ela acontece repentinamente em seus programas ou exploraram diversas *outras* maneiras de lidar com ela. O método "bom o suficiente" empregado sempre foi a humilde função de retorno (callback). Até hoje, muitos insistem que os callbacks são bons o suficiente.

Mas, enquanto JS continua a crescer em escopo e complexidade para alcançar as demandas sempre crescentes de uma linguagem de programação de primeira classe que é executada em navegadores, servidores e em todos os dispositivos concebíveis entre eles, as dificuldades que lidamos com a assincronia estão ficando cada vez mais debilitantes, e, sendo assim, elas suplicam por abordagens que são mais capazes e sensatas.

Ao passo que isso tudo pode parecer abstrato agora, garanto que vamos cuidar disso enquanto prosseguimos através do livro. Exploraremos uma variedade de técnicas emergentes para JavaScript assíncrono nos próximos capítulos.

Porém, antes de podermos chegar lá, vamos ter que entender de uma forma mais aprofundada o que é assincronia e como ela opera em JS.

## Um Programa em Pedaços

Você pode escrever seu programa JS em um arquivo *.js*, mas ele é certamente feito de diversos pedaços, do qual apenas um pedaço é executado *agora*, sendo o restante executado *depois*. A unidade mais comum do *pedaço* é a `função`.

O problema que a maioria do desenvolvedores novatos em JS parecem ter é que *depois* não acontece estritamente e imediatamente depois de *agora*. Em outras palavras, as tarefas que não podem completar *agora* serão, por definição, completas de forma assíncrona, não possuindo assim o comportamento bloqueante que você intuitivamente espera ou deseja que ela possua.

Observe:

```js
// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
var data = ajax( "http://some.url.1" );

console.log( data );
// Oops! `data` geralmente não conterá os resultados do Ajax
```

Você provavelmente está ciente de que requisições Ajax não completam de forma síncrona, o que significa que a função `ajax(...)` ainda não possui nenhum valor para retornar, para então ser atribuido a variável `data`. Se `ajax(...)` *pudesse* bloquear a execução até o retorno da resposta, então a atribuição `data = ..` funcionaria sem problemas.

Mas, não é assim que se faz Ajax. Fazemos uma requisição assíncrona *agora* e não possuiremos o resultado de volta até *depois*.

A forma mais simples (mas definitivamente não a única, ou nem mesmo a melhor) de "esperar" de *agora* até *depois* é usando uma função, comumente conhecida como função de callback (retorno).

```js
// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
ajax( "http://some.url.1", function myCallbackFunction(data){
	console.log(data); // Yay, temos dados!`!
});
```
**Atenção:** você pode ter ouvido que é possível fazer requisições Ajax síncronas. Enquanto isso é tecnicamente verdade, você não deveria fazer isso nunca, em nenhuma circunstância, pois isso trava a interface do usuário no navegador (botões, menus, rolamento etc.) e impede qualquer interação. Essa é uma péssima ideia e deve sempre ser evitada.

Antes que você proteste em discordância, não, sua vontade de evitar a confusão dos callbacks *não* é justificativa para usar Ajax de forma bloqueante e síncrona.

Por exemplo, observe esse código:

```js
function now() {
  return 21;
}

function later() {
  a = a * 2;
  console.log( "Signficado da vida:", a );
}

var answer = now();

setTimeout( later, 1000 ); // Signifcado da vida: 42
```

Existem duas partes para esse programa: o trecho que vai executar *agora* e o trecho que vai executar *depois*. É bem óbvio qual é qual, mas vamos ser super explícitos:

Agora:
```js
function now() {
  return 21;
}

function later() { .. }

var answer = now();

setTimeout( later, 1000 );
```

Depois:
```js
answer = answer * 2;
console.log( "Meaning of life:", answer );
```


O pedaço *agora* roda imediatamente, assim que você executar o programa. Mas `setTimeout(...)` também define um evento (um tempo limite) para acontecer *depois*, de maneira que os conteúdos da função `depois()` serão executados posteriormente (1.000 milisegundos a partir de agora). 

Toda vez que você encapsula uma porção de código numa `função` e especifica que ela deve ser executada como resposta a algum evento (timer, clique do mouse, resposta do Ajax etc.), você está criando um pedaço que será incluido posteriormente no seu código e assim introduzindo assincronia ao seu programa.

### Async Console

Não existe especificação ou grupo de requerimentos sobre como os métodos `console.*` funcionam -- eles não são parte oficial do JavaScript, mas, ao invés disso, são adicionados ao JS pelo *ambiente hospedeiro* (veja o título *Tipos & Gramática* dessa série de livros).

Sendo assim, diferentes navegadores e ambientes JS fazem da maneira que preferirem, o que às vezes pode causar um comportamento confuso.

Em particular, existem alguns navegadores e condições nas quais `console.log(..)` não retorna imediatamente o que é dado. O principal motivo pelo qual isso pode acontecer é por que o I/O (input/output) é uma parte lenta e bloqueante de muitos programas (não apenas JS). Então, pode ser melhor (do ponto de vista página/UI) para um navegador lidar com o I/O do `console` de maneira assíncrona no plano de fundo, sem que você talvez sequer saiba o que ocorreu.

Um cenário improvável, mas possível, de onde isso pode ser *observado* (não através do código em si, mas de fora):

```js
var a = {
  index: 1
};

//depois 
console.log(a); //?

//e depois
a.index++;

```

Normalmente esperaríamos ver o objeto `a` ser capturado no exato momento da instrução, imprimindo algo como `{ index: 1 }`, de forma que na próxima instrução quando `a.index++` acontecer, estaria modificando algo diferente de, ou estritamente depois da saída de `a`.

Na maior parte do tempo, o código acima provavelmente produzirá uma representação de um objeto no console da seção de ferramentas do desenvolvedor do seu navegador (developer tools) da forma como você esperaria. Mas, é possível que o mesmo código execute em uma situação onde o navegador sentiu que precisava deferir o I/O do console para o plano de fundo e, nesse caso, seja *possível* que no momento em que o objeto tenha sido representado no console do navegador, `a.index++` já tenha acontecido, e mostre `{ index: 2 }`.

As condições exatas nas quais o I/O do `console` será deferido, ou mesmo se será observável ou não, é oscilante. Apenas lembre-se dessa possível "assincronicidade" no I/O caso você se depare com problemas ao tentar debugar o código em situações onde o objeto foi modificado *depois* da instrução `console.log(..)` e ainda assim as modificações inesperadas ocorram.

**Nota:** Se você se deparar com esse cenário raro, a melhor opção é usar _breakpoints_ no seu debugger JS ao invés de contar com a saída do `console`. A segunda melhor opção seria forçar uma captura do objeto em questão ao serializá-lo em uma `string`, como com `'JSON.stringify(..)`.

## Loop de Eventos

Vamos fazer uma afirmação (talvez chocante): apesar de claramente permitir código assíncrono (como o timeout que acabamos de ver), até recentemente (ES6), JavaScript nunca teve de verdade uma noção direta de assíncronia nativa.

**O quê!?** Isso parece loucura, certo? E é verdade. O motor JS nunca fez nada além de executar um único pedaço de código do seu programa em um dado momento qualquer, quando solicitado.

"Quando solicitado". Por quem? Essa é a parte importante!

O motor JS não roda isolado. Ele é executado dentro de um *ambiente hospedeiro*, que é para muitos desenvolvedores o habitual navegador web.
Através do últimos anos (mas não necessariamente só nesses), JS alcançou outros ambientes além do navegador, tal qual servidores, através de iniciativas como Node.js. Na verdade, hoje em dia o JavaScript está acoplado nos mais variados tipos de dispositivos, de robôs à lâmpadas.

Entretanto, o "fio" (essa não é uma piadinha não tão sutil com assincronia) comum entre todos esses ambientes é que eles possuem um mecanismo que lida com a execução de múltiplos pedaços do seu programa *ao longo do tempo*, a cada momento invocando o motor JS, chamado "loop de eventos".

Em outras palavras, o motor JS não possui senso inato de *tempo*, de tal forma que está disponível para a execução sob demanda de qualquer código comum de JS. O ambiente periférico é que sempre possui os eventos *agendados* (execuções de código JS).

Então, por exemplo, quando seu programa JS faz uma requisição Ajax para pegar algum dado de um servidor, você define o código de "resposta" em uma função (comumente chamada de "callback") e o motor JS informa ao ambiente hospedeiro, "Ei, eu vou suspender a execução agora, mas assim que você terminar com aquela requisição na rede e você tiver algum dado, por favor *chame* (call) essa função *de volta* (back)."

O navegador então fica a postos escutando pela resposta do servidor. Quando ele possui algo para te entregar, ele agenda a função de callback (retorno) para ser executada ao inserí-la no *loop de eventos*.

Então, o que é o *loop de eventos*?

Vamos contextualizar através de um código de exemplo:

```js
 // eventLoop é uma array que age como uma fila de espera (primeiro a entrar, primeiro a sair)
 var eventLoop = [];
 var event;
 
 // continua indo "para sempre"
 while (true) {
   //faz a verificação
   if(eventLoop.length > 0) {
   	// pega o próximo evento na fila
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

Essa é uma versão bem simplificada do código original, criada apenas para ilustrar os conceitos. No entanto, ela deve ser suficiente para obter uma melhor compreensão da ideia.

Como você pode ver, existe um loop contínuo representado pelo loop `while` e cada iteração desse loop é chamada de "tick". Para cada tick, se um evento está esperando na fila, o evento é removido e executado. Esses eventos são as suas funções de callback (retorno).

É importante notar que `setTimeout(..)` não coloca sua função de callback na fila de espera do loop de eventos (event loop). O que ele faz é definir um contador; quando o contador expira, o ambiente coloca sua função de callback no loop de eventos, de modo que um tick futuro vai pegá-lo e executá-lo.

E se já existem 20 itens no loop de eventos naquele momento? Sua função de callback espera. Ela fica na fila atrás das outras -- normalmente não existe uma maneira de antecipar a espera e pular na frente da fila. Isso explica porque temporizadores `setTimeout(..)` podem não acionar com precisão temporal perfeita. É garantido (de maneira geral) que a sua função de retorno não vai acionar *antes* do tempo especificado, ainda assim, pode ser acionada depois do tempo, dependendo do estado da fila de eventos.

Então, em outras palavras, seu programa é geralmente quebrado em diversos pequenos pedaços, que podem acontecer um após o outro na fila de espera do loop de eventos. E tecnicamente, outros eventos não relacionados diretamente com o seu programa também podem ser alternados dentro da fila.

**Nota:** Mencionamos "até recentemente" em relação ao ES6 mudando a natureza de onde loop de eventos é gerenciado. Na maior parte do tempo, isso é uma tecnicalidade formal. Entretanto, agora o ES6 especifica como o loop de eventos funciona, o que significa que está sob o alcance do motor JS, ao invés de estar apenas no *ambiente hospedeiro*. A razão principal para essa mudança é a introdução das Promises do ES6, as quais discutiremos no Capítulo 3, pois elas requerem a habilidade de ter controle direto e preciso do agendamento de operações na fila de espera do loop de eventos. (veja a discussão do `setTimeout(..0)` na seção "Cooperação").

## Threading Paralelo

É muito comum confundir os termos "assíncrono" (async) e "paralelo" (parallel). Apesar disso, eles possuem significados bem diferentes. Lembre-se, "assíncrono" trata-se do vão entre *agora* e *depois*. "Paralelo" refere-se a coisas (tarefas, funções etc.) que são capazes de ocorrer simultaneamente.

As ferramentas mais comuns para computação paralela são processos e threads. Processos e threads executam de forma independente e podem executar simultaneamente: em processadores, ou mesmo em computadores diferentes, mas inúmeras threads podem compartilhar a memória de um único processo.

Um loop de eventos, por sua vez, divide seu trabalho em tarefas e os executa em série, não permitindo acessos e mudanças paralelas à memória compartilhada. Paralelismo e "serialismo" podem coexistir em loops de evento que cooperam em threads separadas.

A alternância de threads paralelas de execução e a alternância de eventos assíncronos ocorrem em diferentes níveis de granularidade.

Por exemplo:

```js
  function depois() {
     resposta = resposta * 2;
     console.log("Significado da vida:", resposta);
  }
```

Enquanto o conteúdo inteiro de `depois()` seria visto como um único item na fila do loop de eventos, quando pensado sobre a thread no qual esse código seria executado, existe, na verdade, talvez uma dúzia de operações em camadas mais profundas do código (low-level). Por exemplo, `resposta = resposta * 2` requer carregar o valor atual de `resposta`, armazenar o valor `2` em algum lugar, efetuar a multiplicação para obter o resultado final e armazenar de volta em `resposta`.

Em um ambiente de thread única (single-threaded), não importa que os itens na fila da thread são operações low-level, porque nada pode interromper a thread. Contudo, se você está em um sistema paralelo, onde duas threads diferentes estão operando em um mesmo programa, muito provavelmente você poderia ter algum tipo de comportamento imprevisível.

Observe:

```js
  var a = 20;
  
  function foo() {
      a = a + 1;
  }
  
  function bar() {
      a = a * 2;
  }
  
  // ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
  ajax("http://some.url.1", foo);
  ajax("http://some.url.2", bar);
  
```

 No comportamento de thread única do JavaScript, se `foo()` é executado antes de `bar()`, o resultado é que `a` é `42`, mas se `bar()` for executado antes de `foo()` o resultado em `a` será `41`.
 
 Caso os eventos JS compartilhem o mesmo dado executado em paralelo, os problemas seriam mais sutis. veja essas duas listas de tarefas em pseudocódigo como as threads que poderiam rodar respectivamente em `foo()` e `bar()` e leve em conta o que acontece se elas forem executados ao mesmo tempo:
 
 Thread 1 (`X` e `Y` são locações temporárias de memória):
 ```
    foo():
    a. carrega valor de `a` em `X`
    b. guarda `1` em `Y`
    c. soma `X` e `Y`, guarda resultado em `X`
    d. guarda valor de `X` em `a`
 ```
 
 Thread 2 (`X` e `Y` são locações temporárias de memória):
```
   bar():
    a. carrega valor de `a` em `X`
    b. guarda `2` em `Y`
    c. multiplica `X` e `Y`, guarda resultado em `X`
    d. guarda valor de `X` em `a`
```

Agora, digamos que as duas threads estão realmente executando em paralelo. Você consegue perceber o problema, certo? Eles usam locações de memória compartilhada `X` e `Y` para seus passos temporários.

Qual é o resultado final em `a` se os passos ocorrem dessa maneira?

```
1a (carrega o valor de `a` em `X` ==> `20`)
2a (carrega o valor de `a` em `X` ==> `20`)
1b (guarda `1` em `Y` ==> `1`)
2b (guarda `2` em `Y` ==> `2`)
1c (adiciona `X` e `Y`, guarda resultado em `X` ==> `22`)
1d (guarda valor de `X` em `a` ==> `22` )
2c (multiplica `X` e `Y`, guarda resultado em `X` ==> `44`)
2d (guarda valor de `X` em `a` ===> `44`)
```

O resultado em `a` será `44`. E nessa ordem?

```
1a  (carrega valor de `a` em `X` ==> `20`)
2a  (carrega valor de `a` em `X` ==> `20`)
2b  (guarda `2` em `Y`   ==> `2`)
1b  (guarda `1` em `Y`   ==> `1`)
2c  (multiplica `X` e `Y`, guarda resultado em `X`   ==> `20`)
1c  (soma `X` e `Y`, guarda resultado em `X`   ==> `21`)
1d  (guarda valor de `X` em `a`   ==> `21`)
2d  (guarda valor de `X` em `a`   ==> `21`)
```

O resultado em `a` será `21`.

Então, programação com threads é bem capciosa, porque se você não toma precauções para evitar que esse tipo de interrupção/alternância aconteça, você pode ser pego de surpresa com o comportamento indeterminado que frequentemente traz consigo algumas dores de cabeça.

O JavaScript nunca compartilha informações entre threads, o que significa que *esse* nível de indeterminância não é um problema. Mas isso não significa que JS é sempre determinista. Lembre-se de mais cedo, quando a ordem de `foo()` e `bar()` produzia dois resultados diferentes (`41` ou `42`)?

**Nota:** pode não ser óbvio até então, mas nem todo indeterminismo é ruim. As vezes é irrelevante, e algumas vezes é intencional. Veremos mais exemplos disso ao longo desse e dos próximos capítulos.

### Run-to-Completion

Por causa da thread única do JavaScript, o código dentro de `foo()` (e `bar()`) é atômico, o que significa que uma vez que `foo()` começa a ser executado, a completude do código será finalizada antes do código em `bar()` possa ser executado, ou vice-versa. Isso é chamado comportamento "run-to-completion".

Na verdade, a semântica de "run-to-completion" é mais óbvia quando `foo()` e `bar()` tem código neles, como em:

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

// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

Dado que `foo()` não pode ser interrompido por `bar()` e `bar()` não pode ser interrompido por `foo()`, esse programa só possui duas saídas possíveis, dependendo de quem começa a rodar primeiro -- se o threading estivesse presente, e as instruções individuais em `foo()` e `bar()` pudessem ser alternadas, o número de saídas possíveis seria consideravelmente aumentado!

O Pedaço 1 é síncrono (acontece *agora*), mas 2 e 3 são assíncronos (acontecem *depois*), o que significa que a sua execução será separada por um vão de tempo.

Pedaço 1:
```js
var a = 1;
var b = 2;
```

Pedaço 2 (`foo()`):
```js
a++;
b = b * a;
a = b + 3;
```

Pedaço 3 (`bar()`):
```js
b--;
a = 8 + b;
b = a * 2;
```

Os Pedaços 2 e 3 podem acontecer primeiro em qualquer ordem, então existem duas saídas possíveis para esse programa, como ilustrado aqui:

Saída 1:
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

Saída 2:
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

Duas saídas do mesmo código significa que ainda temos indeterminância! Entretanto, é em nível de ordenamento de função (evento), ao invés de estar no nível de ordenamento de instrução (ou, na verdade, no nível de ordenamento da operação da expressão), como é com threads. Em outras palavras, é *mais determinístico* do que threads seriam.

Aplicado ao comportamento JavaScript, essa indeterminância de ordem de função é a famosa expressão "condição de corrida", sendo que `foo()` e `bar()` estão correndo um contra o outro para ver qual é executado primeiro. Especificamente, se trata "condição de corrida" porque você não pode prever confiavelmente como `a` e `b` vão se sair.

**Nota:** Se houvesse uma função em JS que de alguma forma não tivesse comportamento run-to-completion, poderíamos ter muitas possíveis saídas diferentes, certo? Acontece que ES6 nos traz exatamente isso (veja Capítulo 4 "Geradores"), mas, não se preocupe por enquanto, voltaremos a isso!

## Concorrência
Vamos imaginar um site que mostre uma lista de atualizações de status (como atualizações de uma rede social) que progressivamente carrega ao passo que o usuário rola para baixo. Para fazer tal aplicação funcionar corretamente, pelo menos dois "processos" separados vão precisar ser executados *simultaneamente* (isto é, durante a mesma janela de tempo, mas não necessariamente ao mesmo instante).

**Nota:** Usamos "processo" em aspas aqui por que eles não são processos verdadeiros em nível de sistema operacional no sentido da ciência da computação. Eles são processos virtuais, ou tarefas, que representam uma série sequencial de operações logicamente conectadas. Vamos preferir "processo" ao invés de "tarefa", pois a terminologia corresponde com a definição dos conceitos que estamos explorando.

O primeiro "processo" responderá a eventos `onscroll` (fazendo requisições Ajax por novo conteúdo) enquanto são acionadas quando o usuário rola continuamente para baixo. O segundo "processo" receberá respostas do Ajax (para renderizar o conteúdo na página).

Obviamente, se o usuário rola a página rapidamente, você poderá ver dois ou mais eventos `onscroll` acionados durante o tempo que leva para receber e processar a primeira resposta, portanto você terá eventos `onscroll` e eventos de resposta Ajax acionando rapidamente, alternando entre si.

Concorrência é quando dois ou mais "processos" são executados simultaneamente no mesmo periodo, independentemente de se seus constituintes individuais ocorrem *em paralelo* (ao mesmo instante em processadores ou núcleos separados) ou não. Você pode pensar em concorrência como paralelismo em nível de "processo" (ou nível de tarefa), ao contrário de paralelismo em nível de operação (threads em processadores diferentes).

**Nota:** Concorrência também introduz uma noção opcional desses "processos" interagindo entre si. Voltaremos a isso mais tarde.

Para uma dada janela de tempo (poucos segundos de rolagem por parte do usuário), vamos visualizar cada "processo" independente, como uma série de eventos/operações.

"Processo" 1 (eventos `onscroll`):

```
onscroll, requisição 1
onscroll, requisição 2
onscroll, requisição 3
onscroll, requisição 4
onscroll, requisição 5
onscroll, requisição 6
onscroll, requisição 7
```

"Processo" 2 (eventos de resposta Ajax):
```
resposta 1
resposta 2
resposta 3
resposta 4
resposta 5
resposta 6
resposta 7
```

É bem possível que um evento `onscroll` e uma resposta Ajax possam estar prontos para serem processados no exato mesmo *momento*. 
Por exemplo, vamos visualizar esses eventos em uma linha do tempo:

```
onscroll, requisição 1
onscroll, requisição 2          resposta 1
onscroll, requisição 3          resposta 2
resposta 3
onscroll, requisição 4
onscroll, requisição 5
onscroll, requisição 6          resposta 4
onscroll, requisição 7
resposta 6
resposta 5
resposta 7
```

Mas, voltando para a nossa noção do loop de eventos mais cedo no capítulo, JS só poderá manusear um evento por vez, então `onscroll, requisição 2` ou `resposta 1` irá acontecer primeiro e eles não podem acontecer literalmente no mesmo momento. Tal qual crianças na cantina da escola, não importa a multidão que se forme em volta do balcão, eles terão que se juntar em uma fila única para pegar seus almoços!

Vamos visualizar o intercalamento de todos esses eventos na lista de loop de eventos.

Lista de Loop de Eventos:
```
onscroll, requisição 1   <--- Processo 1 começa
onscroll, requisição 2
resposta 1            <--- Processo 2 começa
onscroll, requisição 3
resposta 2
resposta 3
onscroll, requisição 4
onscroll, requisição 5
onscroll, requisição 6
resposta 4
onscroll, requisição 7   <--- Processo 1 termina
resposta 6
resposta 5
resposta 7            <--- Processo 2 termina
```

"Processo 1" e "Processo 2" são executados simultaneamente (paralelismo em nível de tarefa), mas seus eventos individuais correm em sequência na lista do loop de eventos.

Aliás, percebeu como `resposta 6` e `resposta 5` voltaram fora da ordem esperada? 

O loop de eventos em thread única é uma expressão de concorrência (existem certamente outras, dos quais retornaremos mais tarde).

### Não-interação

Conforme dois ou mais "processos" intercalam seus passos/eventos concorrentemente dentro do mesmo programa, eles não necessariamente precisam interagir entre si se as tarefas não são relacionadas. **Se eles não interagem, o indeterminismo é perfeitamente aceitável.**

Por exemplo:

```js
var res = {};

function foo(resultados) {
	res.foo = resultados;
}

function bar(resultados) {
	res.bar = resultados;
}

// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

`foo()` e `bar()` são dois "processos" concorrentes e é indeterminado em qual ordem serão acionados. Entretanto, construímos o programa de uma maneira que não importa a ordem que são executados, porque eles agem independentemente e assim não necessitam interagir.

Esse não é um bug de "condição de corrida", pois o código vai sempre funcionar corretamente, independente da ordem.

### Interação

Em geral, "processos" concorrentes vão interagir por necessidade, indiretamente através do escopo e/ou do DOM. Quando tal interação ocorrer, você precisa coordená-los para prevenir "condição de corrida", como descrito previamente.

Abaixo há um exemplo simples de dois "processos" concorrentes que interagem por causa da ordem implícita, que é apenas *algumas vezes quebrada*:

```js
var res = [];

function answer(data) {
	res.push( data );
}

// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

Os "processos" concorrentes são as duas chamadas de `response()` que serão feitas para lidar com as respostas do Ajax. Elas podem acontecer em qualquer ordem.

Vamos presumir que o comportamento esperado é que `res[0]` possua o resultado da chamada de `"http://some.url.1"`, e `res[1]` possua o resultado da chamada de `"http://some.url.2"`. Algumas vezes a situação será assim, mas, dependendo de qual finalize primeiro, os resultados estarão em ordem inversa. Existe uma boa probabilidade de que esse indeterminismo cause um bug de "condição de corrida".

**Nota:** Seja extremamente cauteloso em presunções que você tenda a fazer nessas situações. Por exemplo, não é incomum para um desenvolvedor observar que `"http://some.url.2"` é "sempre" mais lento em responder do que `"http://some.url.1"`, talvez pela natureza das tarefas que eles fazem (por exemplo, um fazendo uma consulta em banco, e o outro apenas buscando um arquivo estático), de forma que a ordem observada pareça sempre a esperada. Mesmo se duas requisições vão para o mesmo servidor e intencionalmente respondam em uma ordem certa, não existe garantia *real* de qual ordem as respostas chegarão de volta no navegador.

Então, para resolver tal condição de corrida, você pode coordenar interação de ordenamento:

```js
var res = [];

function answer(data) {
	if (data.url == "http://some.url.1") {
		res[0] = data;
	}
	else if (data.url == "http://some.url.2") {
		res[1] = data;
	}
}

// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
ajax( "http://some.url.1", answer );
ajax( "http://some.url.2", answer );
```

Independentemente de qual resposta Ajax voltará primeiro, inspecionamos o `data.url` (supondo que seja retornado do servidor, claro!) para descobrirmos qual posição o dado de resposta deve ocupar na array `res`. `res[0]` sempre conterá os resultados de `"http://some.url.1"` e `res[1]` sempre conterá os resultados de `"http://some.url.2"`. Através de uma coordenação simples eliminamos o indeterminismo da condição de corrida.

A mesma lógica desse cenário se aplicaria se múltiplas chamadas de função concorrente estivessem interagindo entre si através do DOM compartilhado, como uma atualizando o conteúdo de um `<div>` e a outra atualizando o estilo ou atributos do `<div>` (por exemplo, tornando o elemento visível uma vez que possua conteúdo). Você provavelmente não iria querer mostrar o elemento do DOM antes de ter conteúdo, então a coordenação deve garantir ordem de interação apropriada.

Alguns cenários de concorrência são *sempre quebrados* (não apenas *às vezes*) sem interação coordenada. Note: 

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

// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
Nesse exemplo, seja `foo()` ou `bar()` que execute primeiro, sempre fará com que `baz()` seja executado muito cedo (`a` ou `b` ainda serão `undefined`), mas a segunda invocação de `baz()` funcionará, já que ambos `a` e `b` estarão disponíveis.

Existem maneiras diferentes de direcionar essa condição. Aqui vai uma simples:

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

// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

O condicional `if (a && b)` ao redor da chamada de `baz()` é tradicionalmente chamado de "portão", porque não sabemos em qual ordem `a` e `b` chegarão, mas esperamos por ambos chegarem lá antes de precedermos a abrir o portão (chamar a função `baz()`).

Outra condição de interação de concorrência que você pode se deparar é chamada às vezes de "corrida", no entanto, é melhor definida como "tranca". Sua particularidade é o comportamento "apenas o primeiro ganha". Aqui, indeterminismo é aceitável e você diz abertamente que está tudo bem que a "corrida" até a linha de chegada possua apenas um vencedor.

Observe esse código quebrado:

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

// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

Qualquer dos dois (`foo()` ou `bar()`) que executar por último, não sobrescreverá o valor atribuído de `a` do outro, mas também duplicará a chamada (provavelmente indesejeada) à `baz()`.

Então, podemos coordenar a interação com uma simples tranca, que permitirá que apenas o primeiro passe:

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

// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

O condicional `if (a == undefined)` permite apenas que o primeiro entre  `foo()` ou `bar()` passe, ignorando a segunda chamada (muito menos qualquer outra subsequente). Não há nenhum mérito em chegar na segunda posição!

**Nota:** Em todos esses cenários usamos as variáveis globais por motivos de ilustração simples, mas não há nada em nossa lógica que sejam necessárias. Enquanto as funções em questão podem acessar as variáveis (através do escopo), elas funcionarão como pretendem. Depender de variáveis escopadas lexicamente (veja o título *Scope & Closures* dessa série de livros) e de variáveis globais como nestes exemplos é um aspecto negativo desses métodos de coordenação de concorrência. Ao avançarmos nos próximos capítulos, veremos outros métodos de coordenação que são muito mais organizados nesse aspecto.

### Cooperação

Outra expressão de coordenação de concorrência é chamada "concorrência cooperativa". Aqui, o foco não é tanto interagir compartilhando valores no escopo (apesar disso ainda ser obviamente permitido!). O objetivo é pegar um "processo" e quebrá-lo em passos ou lotes para que outros "processos" concorrentes tenham a chance de intercalar suas operações na fila do Loop de Eventos.

Por exemplo, observe uma função de callback (retorno) que precise vasculhar uma longa lista de resultados para transformar seus valores. Usaremos `Array#map` para manter o código curto:

```js
var res = [];

// `answer(..)` recebe uma array de resultados da chamada do Ajax
function answer(data) {
	// adiciona na array `res` já existente
	res = res.concat(
		// cria uma nova array transformada com todos os dados `data` dobrados
		data.map( function(val){
			return val * 2;
		} )
	);
}

// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
ajax( "http://some.url.1", answer );
ajax( "http://some.url.2", answer );
```

Se `"http://some.url.1"` recebe seu resultado primeiro, a lista inteira será mapeada dentro de `res` de uma vez. Se possuir poucos milhares ou menos registros, geralmente não é um problema. Mas, supondo que possua 10 milhões de registros, a execução pode demorar um pouco (diversos segundos em um laptop poderoso, ainda mais em um dispositivo móvel, etc).

Quando tal "processo" executa, nada mais pode acontecer na página, incluindo outras chamadas à `response(..)`, sem atualizações na interface gráfica, nem mesmo eventos como rolagem, digitação, cliques e similares. Isso é um pé no saco.

Então, para fazer um sistema de concorrência mais cooperativo, um que seja mais amigável e não monopolize a fila do loop de eventos, você pode processar os resultados em lotes assíncronos, uma após a outra de volta ao loop de eventos para dar a vez para outros eventos acontecerem.

Uma solução bem simples:

```js
var res = [];

// `answer(..)` recebe uma array de resultados da chamada Ajax
function answer(data) {
	// vamos fazer apenas 1000 por vez
	var chunk = data.splice( 0, 1000 );

	// adiciona a array `res` existente
	res = res.concat(
		// cria uma nova array transformada com todos os valores `chunk` dobrados
		chunk.map( function(val){
			return val * 2;
		} )
	);

	// ainda falta processar algo?
	if (data.length > 0) {
    // agenda o próximo lote
		setTimeout( function(){
			answer( data );
		}, 0 );
	}
}

// ajax(..) é uma função Ajax eventual fornecida por uma biblioteca
ajax( "http://some.url.1", answer );
ajax( "http://some.url.2", answer );
```
Processamos o conjunto de dados em pedaços com um tamanho máximo de 1000 itens. Dessa maneira, garantimos um processo de execução curta, mesmo que isso signifique mais "processos" subsequentes, ao passo que a alternância na fila do loop de eventos vai nos dar uma aplicação com o desempenho muito mais responsivo. 

É certo que não estamos coordenando a interação do ordenamento de nenhum desses "processos", portanto, a ordem dos resultados em `res` não será previsível. Se o ordenamento fosse necessário, você precisaria usar técnicas de interação como aquelas mencionadas anteriormente ou algumas que cobriremos nos próximos capítulos deste livro.

Nós usaremos o `setTimeout(..0)` (hack) para agendamento assíncrono, que basicamente significa apenas "coloque essa função no fim da fila atual do loop de eventos".

**Nota:** `setTimeout(..0)` não é tecnicamente inserir um item diretamente dentro da da fila do loop de eventos. O temporizador vai inserir o evento na próxima oportunidade. Por exemplo, duas chamadas `setTimeout(..0)` subsequentes não serão necessariamente processadas em ordem de chamada, então *é* possível ver várias condições estilo timer onde a ordem de tais eventos não é previsível. Em Node.js, uma solução mais simples é `process.nextTick(..)`. Apesar de quão conveniente (e usualmente mais rápido) seja, não existe uma única direção (ao menos até então) através de todos os ambientes para garantir ordenamento de eventos assíncronos. Cobriremos esse tópico com mais detalhes na próxima seção.

## Fila de Tarefas

A partir do ES6, surge um novo conceito situado no topo da fila do loop de eventos chamado fila de tarefas ("job queue"). O contato mais provável que você terá com ela será com o comportamento assíncrono das Promises (veja o capítulo 3).

Infelizmente, no momento é apenas um mecanismo sem a API aberta, e assim demonstrando que é um pouco mais complexo do que o habitual. Portanto, teremos que apenas descrevê-lo conceitualmente para que quando discutirmos comportamento assíncrono com Promises no capítulo 3, você conseguir entender como essas ações estão sendo agendadas e processadas.

Logo, a melhor perspectiva que eu encontrei para explicar é que a "fila de tarefas" é uma fila pendurada no fim de todo tick na fila do loop de eventos. Algumas ações presumidamente implícitas que podem ocorrer durante um tick não causarão a adição de um novo evento completo na fila do loop de eventos, mas vão ao invés disso adicionar um item (também conhecido como tarefa) ao fim do tick atual na fila de tarefas.

É como dizer "ó, aqui está essa outra coisa que eu preciso fazer *depois*, mas garanta que aconteça logo antes do que qualquer outra coisa possa acontecer".

Ou, usando uma metáfora: a fila do loop de eventos é como um carrinho no parque de diversões, onde toda vez que você termina a corrida, você tem que voltar para o fim da fila para brincar de novo. Mas a fila de tarefas é como terminar a corrida, mas furar a fila entrando novamente.

Uma tarefa também pode fazer com que mais tarefas sejam adicionadas ao fim da mesma fila. Então, é teoricamente possível que um loop de tarefas (uma tarefa que continua adicionando outra, etc) pode girar indefinidamente, assim privando o programa da habilidade de prosseguir para o próximo tick do loop de eventos. Isso seria conceitualmente quase a mesma coisa a expressar um loop infinito (como `while(true)..`) no seu código.

Tarefas são como o espírito da gambiarra do `setTimeout(..0)`, mas implementados de uma forma a ter um controle muito mais bem definido e com ordenamento garantido: **depois, mas o mais cedo possível**. 

Vamos imaginar uma API para agendamento de tarefas (diretamente, sem gambiarras), e chamá-la de `schedule(..)`. Note:

```js
console.log( "A" );

setTimeout( function(){
	console.log( "B" );
}, 0 );

// "API Tarefa" teórico
schedule( function(){
	console.log( "C" );

	schedule( function(){
		console.log( "D" );
	} );
} );
```
Você poderia esperar que isso imprimisse `A B C D`, mas ao invés disso, imprimiria `A C D B`, por que tarefas acontecem no fim de cada tick do loop de eventos, e o temporizador engatilha o angendamento para o *próximo* tick (se disponível!).

No capítulo 3, veremos que os comportamentos assíncronos das Promises são baseadas em tarefas, então é importante manter claro como isso se relaciona com o comportamento do loop de eventos.

## Ordenamento de Instruções

A ordem na qual expressamos instruções no nosso código não é necessariamente a mesma ordem da qual o motor JS vai executá-las. Pode parecer uma asserção estranha de se fazer, então vamos explorá-la brevemente.

Mas antes disso, devemos estar absolutamente claros em uma coisa: as regras/gramática da linguagem (veja o título *Tipos e Gramática* dessa série de livros) dita um comportamento bastante previsível e confiável para ordenamento de instruções do ponto de vista do programa. Então o que discutiremos a seguir **não são coisas que você deveria poder observar** no seu programa JS.

**Cuidado:** Se em algum momento você puder *observar* reordenamento de instruções no compilador como estamos quase ilustrando, isso seria uma clara violação da especificação, e seria inquestionavelmente em virtude de algum bug no motor JS em questão -- um que deveria ser reportado e consertado prontamente! Mas é muito mais comum que você *suspeite* que algo louco está acontecendo no motor JS, quando na verdade é só um bug (provavelmente uma "condição de corrida") no seu código -- então verifique lá primeiro, e denovo e denovo. O _debugger_ JS, usando _breakpoints_ e avançando através do código linha por linha, será sua ferramenta mais poderosa para detectar tais bugs no *seu código*.

Observe:

```js
var a, b;

a = 10;
b = 30;

a = a + 1;
b = b + 1;

console.log( a + b ); // 42
```

Esse código não tem assincronia expressa nele (além do raro I/O do `console` async discutido antes), então a presunção mais provavel é que seria processado linha por linha de cima para baixo.

Mas é *possível* que o motor JS, depois de compilar esse código (sim, JS é compilado -- veja o título *Scope & Closures* dessa série de livros!) possa encontrar de rodar esse código mais rápido ao reordenar (com segurança) a ordem das instruções. Essencialmente, desde que você não consiga observar o reordenamento, vale tudo.

Por exemplo, o motor pode achar mais rápido executar o código assim:

```js
var a, b;

a = 10;
a++;

b = 30;
b++;

console.log( a + b ); // 42
```

Ou assim:

```js
var a, b;

a = 11;
b = 31;

console.log( a + b ); // 42
```

Ou mesmo:

```js
// por que `a` e `b` não são mais usados,
// podemos removê-los e nem precisaremos mais deles!
console.log( 42 ); // 42
```
Em todos esses casos, o motor JS está fazendo otimizações seguras durante sua compilação, enquanto o fim *observável* será o mesmo.

Mas aqui temos um cenário onde essas otimizações específicas seriam inseguras e portanto não seriam permitidas (claro, sem mencionar que isso não seria otimizar de forma alguma).

```js
var a, b;

a = 10;
b = 30;

// precisamos `a` e `b` em seus estados pré-incrementados!
console.log( a * b ); // 300

a = a + 1;
b = b + 1;

console.log( a + b ); // 42
```

Outros exemplos onde o reordenamento de compilação pode criar efeitos colaterais indesejados (e portanto devem ser desabilitados) incluiriam coisas como qualquer chamada de função com efeitos colaterais (especialmente funções `getter`), ou objetos Proxy do ES6 (veja o título *ES6 e Além* dessa série de livros).

Observe:

```js
function foo() {
	console.log( b );
	return 1;
}

var a, b, c;

// Síntaxe literal ES5.1
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
Se não fosse pelo `console.log(..)` nesse código (usado apenas como uma forma conveniente de observação de efeito colateral para essa ilustração), o motor JS provavelmente estaria livre, se quisesse (quem sabe se iria querer!?), para reordenar o código para:

```js
// ...

a = 10 + foo();
b = 30 + c.bar;

// ...
```

Enquanto a semântica JS afortunadamente nos protege dos pesadelos observáveis que o ordenamento da instrução do compilador pode parecer nos causar, ainda é importante entendermos quão tênue é a ligação que existe entre a forma como o código é auditado (de cima para baixo) e a forma como é executado após a compilação.

Reordenamento da instrução do compilador é quase uma micro-metáfora para concorrência e interação. Como conceito geral, tal entendimento pode ajudar a comprender melhor os problemas do código assíncrono no JS.

## Revisão

Um programa JavaScript é (praticamente) sempre quebrado em dois ou mais pedaços, onde o primeiro pedaço roda *agora* e o próximo roda *depois* em resposta a um evento. Apesar do programa ser executado pedaço a pedaço, todos eles compartilham o mesmo acesso ao estado e escopo do programa, então cada modificação ao estado é feito em cima do estado anterior.

Sempre que existem eventos a serem executados, o *loop de eventos* roda até a fila estar vazia. Cada iteração do loop de eventos é um "tick". Interação do usuário, IO, e temporizadores enfileiram eventos na fila de eventos.

Em qualquer momento, apenas um evento pode ser processado da lista por vez. Enquanto um evento executa, o processamento pode, direta ou indiretamente, causar um ou mais eventos subsequentes.

Concorrência é quando duas ou mais cadeias de eventos se intercalam ao longo do tempo, de maneira que de uma perspectiva ampla, elas aparentem estar rodando *simultaneamente* (apesar de que em qualquer momento apenas um evento é processado).

Frequentemente é necessário fazer algum tipo de coordenação de interação entre "processos" concorrentes (diferentes de processos de sistema operacional), por exemplo, para garantir ordenamento ou prevenir "condição de corrida". Tais "processos" também podem *cooperar* ao se separar em pedaços menores e permitir que outros "processos" intercalem.