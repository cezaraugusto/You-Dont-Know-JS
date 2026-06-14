# You Don't Know JS: Async & Performance
# Capítulo 5: Desempenho de Programas

Até aqui, este livro tratou inteiramente de como aproveitar os padrões de assincronia de forma mais eficaz. Mas não abordamos diretamente por que a assincronia realmente importa para o JS. A razão explícita mais óbvia é **performance**.

Por exemplo, se você tem duas requisições Ajax a fazer, e elas são independentes, mas você precisa esperar que ambas terminem antes de executar a próxima tarefa, você tem duas opções para modelar essa interação: serial e concorrente.

Você poderia fazer a primeira requisição e esperar para iniciar a segunda até que a primeira termine. Ou, como vimos tanto com promises quanto com geradores, você poderia fazer ambas as requisições "em paralelo", e expressar o "portão" para esperar por ambas antes de seguir em frente.

Claramente, a última geralmente será mais performática que a primeira. E uma performance melhor geralmente leva a uma experiência de usuário melhor.

É até possível que a assincronia (concorrência intercalada) possa melhorar apenas a percepção de performance, mesmo que o programa como um todo ainda leve a mesma quantidade de tempo para ser concluído. A percepção de performance pelo usuário é tão importante quanto -- se não mais! -- a performance real mensurável.

Queremos agora ir além dos padrões de assincronia localizados para falar sobre alguns detalhes de performance de visão mais ampla, no nível do programa.

**Nota:** Você pode estar se perguntando sobre questões de micro-performance, como se `a++` ou `++a` é mais rápido. Veremos esses tipos de detalhes de performance no próximo capítulo sobre "Benchmarking & Tuning".

## Web Workers

Se você tem tarefas intensivas em processamento, mas não quer que elas rodem na thread principal (o que pode deixar o navegador/UI mais lento), talvez já tenha desejado que o JavaScript pudesse operar de maneira multithreaded.

No Capítulo 1, falamos em detalhes sobre como o JavaScript é single threaded. E isso ainda é verdade. Mas uma única thread não é a única forma de organizar a execução do seu programa.

Imagine dividir seu programa em duas partes, e rodar uma dessas partes na thread principal de UI, e rodar a outra parte em uma thread inteiramente separada.

Que tipos de preocupações uma arquitetura assim levantaria?

Para começar, você gostaria de saber se rodar em uma thread separada significaria que ela rodaria em paralelo (em sistemas com múltiplas CPUs/cores), de tal forma que um processo de longa duração nessa segunda thread **não** bloquearia a thread principal do programa. Caso contrário, "threading virtual" não traria muito benefício em relação ao que já temos no JS com concorrência assíncrona.

E você gostaria de saber se essas duas partes do programa têm acesso ao mesmo escopo/recursos compartilhados. Se tiverem, então você tem todas as questões com que as linguagens multithreaded (Java, C++, etc.) lidam, como a necessidade de travamento cooperativo ou preemptivo (mutexes, etc.). Isso é muito trabalho extra, e não deve ser empreendido levianamente.

Alternativamente, você gostaria de saber como essas duas partes poderiam "se comunicar" caso não pudessem compartilhar escopo/recursos.

Todas essas são ótimas questões a considerar enquanto exploramos um recurso adicionado à plataforma web por volta do HTML5 chamado "Web Workers". Este é um recurso do navegador (também conhecido como ambiente hospedeiro) e na verdade quase não tem nada a ver com a linguagem JS em si. Ou seja, o JavaScript não tem *atualmente* nenhum recurso que dê suporte à execução em threads.

Mas um ambiente como o seu navegador pode facilmente fornecer múltiplas instâncias do motor JavaScript, cada uma em sua própria thread, e permitir que você rode um programa diferente em cada thread. Cada uma dessas partes do seu programa, separadas em threads, é chamada de "(Web) Worker". Esse tipo de paralelismo é chamado de "paralelismo de tarefas", já que a ênfase está em dividir pedaços do seu programa para rodar em paralelo.

A partir do seu programa JS principal (ou de outro Worker), você instancia um Worker assim:

```js
var w1 = new Worker( "http://some.url.1/mycoolworker.js" );
```

A URL deve apontar para a localização de um arquivo JS (não uma página HTML!) que se destina a ser carregado em um Worker. O navegador então iniciará uma thread separada e deixará esse arquivo rodar como um programa independente naquela thread.

**Nota:** O tipo de Worker criado com tal URL é chamado de "Dedicated Worker". Mas, em vez de fornecer uma URL para um arquivo externo, você também pode criar um "Inline Worker" fornecendo uma Blob URL (outro recurso do HTML5); essencialmente é um arquivo inline armazenado em um único valor (binário). Entretanto, Blobs estão além do escopo do que discutiremos aqui.

Workers não compartilham nenhum escopo ou recurso entre si nem com o programa principal -- isso traria todos os pesadelos da programação com threads para o primeiro plano -- mas, em vez disso, têm um mecanismo básico de mensagens por eventos conectando-os.

O objeto Worker `w1` é um event listener e disparador, que permite que você se inscreva em eventos enviados pelo Worker, bem como envie eventos para o Worker.

Veja como escutar eventos (na verdade, o evento fixo `"message"`):

```js
w1.addEventListener( "message", function(evt){
	// evt.data
} );
```

E você pode enviar o evento `"message"` para o Worker:

```js
w1.postMessage( "something cool to say" );
```

Dentro do Worker, as mensagens são totalmente simétricas:

```js
// "mycoolworker.js"

addEventListener( "message", function(evt){
	// evt.data
} );

postMessage( "a really cool reply" );
```

Note que um dedicated Worker tem uma relação de um-para-um com o programa que o criou. Ou seja, o evento `"message"` não precisa de nenhuma desambiguação aqui, porque temos certeza de que ele só poderia ter vindo dessa relação de um-para-um -- ou veio do Worker ou da página principal.

Normalmente, a aplicação da página principal cria os Workers, mas um Worker pode instanciar seu(s) próprio(s) Worker(s) filho(s) -- conhecidos como subworkers -- conforme necessário. Às vezes é útil delegar tais detalhes a uma espécie de Worker "mestre" que gera outros Workers para processar partes de uma tarefa. Infelizmente, no momento em que isto é escrito, o Chrome ainda não dá suporte a subworkers, enquanto o Firefox dá.

Para matar um Worker imediatamente a partir do programa que o criou, chame `terminate()` no objeto Worker (como `w1` nos trechos anteriores). Encerrar abruptamente uma thread de Worker não lhe dá nenhuma chance de finalizar seu trabalho ou limpar quaisquer recursos. É semelhante a você fechar uma aba do navegador para matar uma página.

Se você tiver duas ou mais páginas (ou múltiplas abas com a mesma página!) no navegador que tentem criar um Worker a partir da mesma URL de arquivo, esses na verdade acabarão sendo Workers completamente separados. Em breve, discutiremos uma forma de "compartilhar" um Worker.

**Nota:** Pode parecer que um programa JS malicioso ou ignorante poderia facilmente realizar um ataque de negação de serviço em um sistema gerando centenas de Workers, aparentemente cada um com sua própria thread. Embora seja verdade que existe uma espécie de garantia de que um Worker acabará em uma thread separada, essa garantia não é ilimitada. O sistema é livre para decidir quantas threads/CPUs/cores realmente quer criar. Não há como prever ou garantir quantos você terá acesso, embora muitas pessoas presumam que seja pelo menos tantos quanto o número de CPUs/cores disponíveis. Acho que a suposição mais segura é que existe pelo menos uma outra thread além da thread principal de UI, mas é mais ou menos isso.

### Ambiente do Worker

Dentro do Worker, você não tem acesso a nenhum dos recursos do programa principal. Isso significa que você não pode acessar nenhuma de suas variáveis globais, nem pode acessar o DOM da página ou outros recursos. Lembre-se: é uma thread totalmente separada.

Você pode, entretanto, realizar operações de rede (Ajax, WebSockets) e definir temporizadores. Além disso, o Worker tem acesso à sua própria cópia de várias variáveis/recursos globais importantes, incluindo `navigator`, `location`, `JSON` e `applicationCache`.

Você também pode carregar scripts JS extras no seu Worker, usando `importScripts(..)`:

```js
// dentro do Worker
importScripts( "foo.js", "bar.js" );
```

Esses scripts são carregados de forma síncrona, o que significa que a chamada `importScripts(..)` bloqueará o restante da execução do Worker até que o(s) arquivo(s) terminem de carregar e executar.

**Nota:** Houve também algumas discussões sobre expor a API `<canvas>` aos Workers, o que, combinado com o fato de os canvases serem Transferables (veja a seção "Data Transfer"), permitiria aos Workers realizar processamento gráfico off-thread mais sofisticado, o que pode ser útil para jogos de alta performance (WebGL) e outras aplicações similares. Embora isso ainda não exista em nenhum navegador, é provável que aconteça em um futuro próximo.

Quais são alguns usos comuns para Web Workers?

* Cálculos matemáticos intensivos em processamento
* Ordenação de grandes conjuntos de dados
* Operações de dados (compressão, análise de áudio, manipulações de pixels de imagens, etc.)
* Comunicações de rede de alto tráfego

### Transferência de Dados

Você pode notar uma característica comum à maioria desses usos, que é o fato de exigirem que uma grande quantidade de informação seja transferida através da barreira entre as threads usando o mecanismo de eventos, talvez em ambas as direções.

Nos primeiros dias dos Workers, serializar todos os dados em um valor string era a única opção. Além da penalidade de velocidade das serializações em duas vias, o outro grande aspecto negativo era que os dados estavam sendo copiados, o que significava uma duplicação do uso de memória (e o subsequente churn da coleta de lixo).

Felizmente, agora temos algumas opções melhores.

Se você passa um objeto, um chamado "Structured Cloning Algorithm" (https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/The_structured_clone_algorithm) é usado para copiar/duplicar o objeto do outro lado. Esse algoritmo é bastante sofisticado e pode até lidar com a duplicação de objetos com referências circulares. A penalidade de performance de to-string/from-string não é paga, mas ainda temos duplicação de memória usando essa abordagem. Há suporte para isso no IE10 e acima, bem como em todos os outros navegadores principais.

Uma opção ainda melhor, especialmente para conjuntos de dados maiores, são os "Transferable Objects" (http://updates.html5rocks.com/2011/12/Transferable-Objects-Lightning-Fast). O que acontece é que a "propriedade" do objeto é transferida, mas os dados em si não são movidos. Uma vez que você transfere um objeto para um Worker, ele fica vazio ou inacessível no local de origem -- isso elimina os perigos da programação com threads sobre um escopo compartilhado. Claro, a transferência de propriedade pode ocorrer em ambas as direções.

Na verdade, não há muito que você precise fazer para optar por um Transferable Object; qualquer estrutura de dados que implemente a interface Transferable (https://developer.mozilla.org/en-US/docs/Web/API/Transferable) será automaticamente transferida dessa forma (suporte Firefox & Chrome).

Por exemplo, typed arrays como `Uint8Array` (veja o título *ES6 & Beyond* desta série) são "Transferables". Veja como você enviaria um Transferable Object usando `postMessage(..)`:

```js
// `foo` é um `Uint8Array`, por exemplo

postMessage( foo.buffer, [ foo.buffer ] );
```

O primeiro parâmetro é o buffer bruto e o segundo parâmetro é uma lista do que transferir.

Navegadores que não dão suporte a Transferable Objects simplesmente degradam para clonagem estruturada, o que significa redução de performance em vez de quebra total do recurso.

### Shared Workers

Se o seu site ou app permite o carregamento de múltiplas abas da mesma página (um recurso comum), você pode muito bem querer reduzir o uso de recursos do sistema impedindo dedicated Workers duplicados; o recurso limitado mais comum nesse aspecto é uma conexão de rede via socket, já que os navegadores limitam o número de conexões simultâneas a um único host. Claro, limitar múltiplas conexões a partir de um cliente também alivia as exigências de recursos do seu servidor.

Nesse caso, criar um único Worker centralizado que todas as instâncias de página do seu site ou app possam *compartilhar* é bastante útil.

Isso é chamado de `SharedWorker`, que você cria assim (o suporte para isso é limitado a Firefox e Chrome):

```js
var w1 = new SharedWorker( "http://some.url.1/mycoolworker.js" );
```

Como um shared Worker pode estar conectado a, ou a partir de, mais de uma instância de programa ou página do seu site, o Worker precisa de uma forma de saber de qual programa uma mensagem vem. Essa identificação única é chamada de "port" -- pense em portas de socket de rede. Então o programa chamador deve usar o objeto `port` do Worker para a comunicação:

```js
w1.port.addEventListener( "message", handleMessages );

// ..

w1.port.postMessage( "something cool" );
```

Além disso, a conexão da port deve ser inicializada, assim:

```js
w1.port.start();
```

Dentro do shared Worker, um evento extra deve ser tratado: `"connect"`. Esse evento fornece o `object` port para aquela conexão específica. A forma mais conveniente de manter múltiplas conexões separadas é usar closure (veja o título *Scope & Closures* desta série) sobre a `port`, como mostrado a seguir, com a escuta e transmissão de eventos para aquela conexão definidas dentro do handler para o evento `"connect"`:

```js
// dentro do shared Worker
addEventListener( "connect", function(evt){
	// a port atribuída para esta conexão
	var port = evt.ports[0];

	port.addEventListener( "message", function(evt){
		// ..

		port.postMessage( .. );

		// ..
	} );

	// inicializa a conexão da port
	port.start();
} );
```

Fora essa diferença, shared e dedicated Workers têm as mesmas capacidades e semânticas.

**Nota:** Shared Workers sobrevivem ao encerramento de uma conexão de port se outras conexões de port ainda estiverem ativas, ao passo que dedicated Workers são encerrados sempre que a conexão com o programa que os iniciou é encerrada.

### Polyfill de Web Workers

Web Workers são muito atraentes em termos de performance para rodar programas JS em paralelo. No entanto, você pode estar em uma posição em que seu código precisa rodar em navegadores mais antigos que não têm suporte. Como Workers são uma API e não uma sintaxe, eles podem ser polyfilled, até certo ponto.

Se um navegador não dá suporte a Workers, simplesmente não há maneira de simular multithreading do ponto de vista de performance. Comumente se pensa que iframes fornecem um ambiente paralelo, mas em todos os navegadores modernos eles na verdade rodam na mesma thread que a página principal, então não são suficientes para simular paralelismo.

Como detalhamos no Capítulo 1, a assincronicidade do JS (não o paralelismo) vem da fila de loop de eventos, então você pode forçar Workers simulados a serem assíncronos usando temporizadores (`setTimeout(..)`, etc.). Daí você só precisa fornecer um polyfill para a API do Worker. Há alguns listados aqui (https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills#web-workers), mas, francamente, nenhum deles parece ótimo.

Escrevi um esboço de um polyfill para `Worker` aqui (https://gist.github.com/getify/1b26accb1a09aa53ad25). É básico, mas deve dar conta do recado para suporte simples a `Worker`, dado que as mensagens em duas vias funcionam corretamente, bem como o tratamento de `"onerror"`. Você provavelmente também poderia estendê-lo com mais recursos, como `terminate()` ou Shared Workers simulados, conforme achar conveniente.

**Nota:** Você não pode simular bloqueio síncrono, então este polyfill simplesmente proíbe o uso de `importScripts(..)`. Outra opção poderia ter sido analisar e transformar o código do Worker (uma vez carregado por Ajax) para lidar com a reescrita para alguma forma assíncrona de um polyfill de `importScripts(..)`, talvez com uma interface que reconheça promises.

## SIMD

Single instruction, multiple data (SIMD) é uma forma de "paralelismo de dados", em contraste com o "paralelismo de tarefas" dos Web Workers, porque a ênfase não está realmente em paralelizar pedaços da lógica do programa, mas sim em múltiplos bits de dados sendo processados em paralelo.

Com SIMD, threads não fornecem o paralelismo. Em vez disso, CPUs modernas fornecem capacidade SIMD com "vetores" de números -- pense: arrays especializados em tipos -- bem como instruções que podem operar em paralelo sobre todos os números; essas são operações de baixo nível que aproveitam o paralelismo no nível de instrução.

O esforço para expor a capacidade SIMD ao JavaScript é liderado principalmente pela Intel (https://01.org/node/1495), a saber, por Mohammad Haghighat (no momento em que isto é escrito), em cooperação com as equipes do Firefox e do Chrome. SIMD está em uma trilha de padronização inicial com uma boa chance de entrar em uma futura revisão do JavaScript, provavelmente no período do ES7.

O SIMD JavaScript propõe expor tipos de vetores curtos e APIs ao código JS, que, nesses sistemas habilitados para SIMD, mapeariam as operações diretamente para os equivalentes da CPU, com fallback para "shims" de operação não paralelizada em sistemas sem SIMD.

Os benefícios de performance para aplicações intensivas em dados (análise de sinais, operações com matrizes em gráficos, etc.) com tal processamento matemático paralelo são bastante óbvios!

As formas iniciais da proposta da API SIMD, no momento em que isto é escrito, são assim:

```js
var v1 = SIMD.float32x4( 3.14159, 21.0, 32.3, 55.55 );
var v2 = SIMD.float32x4( 2.1, 3.2, 4.3, 5.4 );

var v3 = SIMD.int32x4( 10, 101, 1001, 10001 );
var v4 = SIMD.int32x4( 10, 20, 30, 40 );

SIMD.float32x4.mul( v1, v2 );	// [ 6.597339, 67.2, 138.89, 299.97 ]
SIMD.int32x4.add( v3, v4 );		// [ 20, 121, 1031, 10041 ]
```

Mostrados aqui estão dois tipos de dados de vetor diferentes, números de ponto flutuante de 32 bits e números inteiros de 32 bits. Você pode ver que esses vetores são dimensionados exatamente para quatro elementos de 32 bits, já que isso corresponde aos tamanhos de vetor SIMD (128 bits) disponíveis na maioria das CPUs modernas. Também é possível que vejamos uma versão `x8` (ou maior!) dessas APIs no futuro.

Além de `mul()` e `add()`, muitas outras operações provavelmente serão incluídas, como `sub()`, `div()`, `abs()`, `neg()`, `sqrt()`, `reciprocal()`, `reciprocalSqrt()` (aritméticas), `shuffle()` (rearranjar elementos do vetor), `and()`, `or()`, `xor()`, `not()` (lógicas), `equal()`, `greaterThan()`, `lessThan()` (comparação), `shiftLeft()`, `shiftRightLogical()`, `shiftRightArithmetic()` (deslocamentos), `fromFloat32x4()` e `fromInt32x4()` (conversões).

**Nota:** Existe um "prollyfill" oficial (um polyfill esperançoso, expectante, voltado para o futuro) para a funcionalidade SIMD disponível (https://github.com/johnmccutchan/ecmascript_simd), que ilustra muito mais da capacidade SIMD planejada do que ilustramos nesta seção.

## asm.js

"asm.js" (http://asmjs.org/) é um rótulo para um subconjunto altamente otimizável da linguagem JavaScript. Ao evitar cuidadosamente certos mecanismos e padrões que são *difíceis* de otimizar (coleta de lixo, coerção, etc.), código no estilo asm.js pode ser reconhecido pelo motor JS e receber atenção especial com otimizações agressivas de baixo nível.

Diferentemente de outros mecanismos de performance de programa discutidos neste capítulo, asm.js não é necessariamente algo que precisa ser adotado na especificação da linguagem JS. *Existe* uma especificação asm.js (http://asmjs.org/spec/latest/), mas ela serve principalmente para rastrear um conjunto acordado de inferências candidatas para otimização, e não um conjunto de requisitos para os motores JS.

Atualmente, não há nenhuma nova sintaxe sendo proposta. Em vez disso, asm.js sugere maneiras de reconhecer a sintaxe JS padrão existente que esteja em conformidade com as regras do asm.js e deixar os motores implementarem suas próprias otimizações de acordo.

Houve alguma discordância entre os fornecedores de navegadores sobre exatamente como o asm.js deveria ser ativado em um programa. Versões iniciais do experimento asm.js exigiam um pragma `"use asm";` (similar ao `"use strict";` do strict mode) para ajudar a dar a pista ao motor JS de que ele deveria procurar oportunidades e dicas de otimização asm.js. Outros afirmaram que asm.js deveria ser apenas um conjunto de heurísticas que os motores reconhecem automaticamente sem que o autor tenha que fazer nada extra, o que significa que programas existentes poderiam teoricamente se beneficiar de otimizações no estilo asm.js sem fazer nada de especial.

### Como Otimizar com asm.js

A primeira coisa a entender sobre as otimizações asm.js gira em torno de tipos e coerção (veja o título *Types & Grammar* desta série). Se o motor JS tem que rastrear múltiplos tipos diferentes de valores em uma variável através de várias operações, de modo a poder lidar com coerções entre tipos conforme necessário, isso é muito trabalho extra que mantém a otimização do programa abaixo do ideal.

**Nota:** Vamos usar código no estilo asm.js aqui para fins de ilustração, mas esteja ciente de que normalmente não se espera que você escreva tal código à mão. asm.js destina-se mais a ser um alvo de compilação a partir de outras ferramentas, como o Emscripten (https://github.com/kripken/emscripten/wiki). É claro que é possível escrever seu próprio código asm.js, mas isso normalmente é uma má ideia porque o código é de muito baixo nível e gerenciá-lo pode ser muito demorado e propenso a erros. Mesmo assim, pode haver casos em que você queira ajustar manualmente seu código para fins de otimização asm.js.

Existem alguns "truques" que você pode usar para dar a dica a um motor JS que reconhece asm.js sobre qual é o tipo pretendido para variáveis/operações, de modo que ele possa pular essas etapas de rastreamento de coerção.

Por exemplo:

```js
var a = 42;

// ..

var b = a;
```

Nesse programa, a atribuição `b = a` deixa a porta aberta para divergência de tipos nas variáveis. No entanto, ela poderia, em vez disso, ser escrita como:

```js
var a = 42;

// ..

var b = a | 0;
```

Aqui, usamos o `|` ("OR binário") com o valor `0`, o que não tem efeito sobre o valor além de garantir que ele seja um inteiro de 32 bits. Esse código, rodado em um motor JS normal, funciona perfeitamente, mas, quando rodado em um motor JS que reconhece asm.js, ele *pode* sinalizar que `b` deve sempre ser tratado como um inteiro de 32 bits, de modo que o rastreamento de coerção possa ser pulado.

De forma similar, a operação de adição entre duas variáveis pode ser restrita a uma adição de inteiros mais performática (em vez de ponto flutuante):

```js
(a + b) | 0
```

Novamente, o motor JS que reconhece asm.js pode ver essa dica e inferir que a operação `+` deve ser uma adição de inteiros de 32 bits porque o resultado final de toda a expressão seria, de qualquer forma, automaticamente conformado a um inteiro de 32 bits.

### Módulos asm.js

Um dos maiores detratores da performance em JS gira em torno da alocação de memória, da coleta de lixo e do acesso a escopo. asm.js sugere que uma das formas de contornar essas questões é declarar um "módulo" asm.js mais formalizado -- não confunda esses com módulos ES6; veja o título *ES6 & Beyond* desta série.

Para um módulo asm.js, você precisa passar explicitamente um namespace estritamente conformado -- isso é referido na especificação como `stdlib`, já que deve representar as bibliotecas padrão necessárias -- para importar os símbolos necessários, em vez de apenas usar globais via escopo léxico. No caso base, o objeto `window` é um objeto `stdlib` aceitável para fins de módulo asm.js, mas você poderia, e talvez devesse, construir um ainda mais restrito.

Você também deve declarar um "heap" -- que é apenas um termo elegante para um local reservado na memória onde variáveis já podem ser usadas sem pedir mais memória ou liberar memória previamente usada -- e passá-lo, de modo que o módulo asm.js não precise fazer nada que cause churn de memória; ele pode apenas usar o espaço pré-reservado.

Um "heap" é provavelmente um `ArrayBuffer` tipado, como:

```js
var heap = new ArrayBuffer( 0x10000 );	// heap de 64k
```

Usando esse espaço binário de 64k pré-reservado, um módulo asm.js pode armazenar e recuperar valores nesse buffer sem nenhuma penalidade de alocação de memória ou de coleta de lixo. Por exemplo, o buffer `heap` poderia ser usado dentro do módulo para dar suporte a um array de valores float de 64 bits assim:

```js
var arr = new Float64Array( heap );
```

OK, então vamos fazer um exemplo rápido e bobo de um módulo no estilo asm.js para ilustrar como essas peças se encaixam. Vamos definir um `foo(..)` que recebe um inteiro de início (`x`) e fim (`y`) para um intervalo, e calcula todas as multiplicações internas adjacentes dos valores no intervalo, e então, finalmente, calcula a média desses valores:

```js
function fooASM(stdlib,foreign,heap) {
	"use asm";

	var arr = new stdlib.Int32Array( heap );

	function foo(x,y) {
		x = x | 0;
		y = y | 0;

		var i = 0;
		var p = 0;
		var sum = 0;
		var count = ((y|0) - (x|0)) | 0;

		// calcula todas as multiplicações internas adjacentes
		for (i = x | 0;
			(i | 0) < (y | 0);
			p = (p + 8) | 0, i = (i + 1) | 0
		) {
			// armazena o resultado
			arr[ p >> 3 ] = (i * (i + 1)) | 0;
		}

		// calcula a média de todos os valores intermediários
		for (i = 0, p = 0;
			(i | 0) < (count | 0);
			p = (p + 8) | 0, i = (i + 1) | 0
		) {
			sum = (sum + arr[ p >> 3 ]) | 0;
		}

		return +(sum / count);
	}

	return {
		foo: foo
	};
}

var heap = new ArrayBuffer( 0x1000 );
var foo = fooASM( window, null, heap ).foo;

foo( 10, 20 );		// 233
```

**Nota:** Este exemplo asm.js é escrito à mão para fins de ilustração, então não representa o mesmo código que seria produzido por uma ferramenta de compilação que tem asm.js como alvo. Mas ele de fato mostra a natureza típica do código asm.js, especialmente as dicas de tipo e o uso do buffer `heap` para armazenamento temporário de variáveis.

A primeira chamada a `fooASM(..)` é o que configura nosso módulo asm.js com sua alocação de `heap`. O resultado é uma função `foo(..)` que podemos chamar quantas vezes forem necessárias. Essas chamadas a `foo(..)` deveriam ser especialmente otimizadas por um motor JS que reconhece asm.js. É importante notar que o código anterior é JS completamente padrão e rodaria perfeitamente bem (sem otimização especial) em um motor que não reconhece asm.js.

Obviamente, a natureza das restrições que tornam o código asm.js tão otimizável reduz significativamente os usos possíveis para tal código. asm.js não será necessariamente um conjunto de otimizações de uso geral para qualquer programa JS dado. Em vez disso, ele se destina a fornecer uma forma otimizada de lidar com tarefas especializadas, como operações matemáticas intensivas (por exemplo, aquelas usadas no processamento gráfico de jogos).

## Revisão

Os quatro primeiros capítulos deste livro baseiam-se na premissa de que padrões de codificação assíncrona lhe dão a capacidade de escrever código mais performático, o que geralmente é uma melhoria muito importante. Mas o comportamento assíncrono só leva você até certo ponto, porque ele ainda está fundamentalmente vinculado a uma única thread de loop de eventos.

Então, neste capítulo, cobrimos vários mecanismos no nível do programa para melhorar a performance ainda mais.

Web Workers permitem que você rode um arquivo JS (também conhecido como programa) em uma thread separada, usando eventos assíncronos para trocar mensagens entre as threads. Eles são maravilhosos para descarregar tarefas de longa duração ou intensivas em recursos para uma thread diferente, deixando a thread principal de UI mais responsiva.

SIMD propõe mapear operações matemáticas paralelas no nível da CPU para APIs JavaScript, para operações de paralelismo de dados de alta performance, como processamento de números em grandes conjuntos de dados.

Por fim, asm.js descreve um pequeno subconjunto do JavaScript que evita as partes difíceis de otimizar do JS (como coleta de lixo e coerção) e permite que o motor JS reconheça e rode tal código através de otimizações agressivas. asm.js poderia ser escrito à mão, mas isso é extremamente tedioso e propenso a erros, semelhante a escrever linguagem assembly à mão (daí o nome). Em vez disso, a principal intenção é que asm.js seja um bom alvo para compilação cruzada a partir de outras linguagens de programação altamente otimizadas -- por exemplo, o Emscripten (https://github.com/kripken/emscripten/wiki) transpilando C/C++ para JavaScript.

Embora não coberto explicitamente neste capítulo, há ideias ainda mais radicais sob discussão muito inicial para o JavaScript, incluindo aproximações de funcionalidade direta de threads (não apenas escondida atrás de APIs de estruturas de dados). Quer isso aconteça explicitamente, quer apenas vejamos mais paralelismo se infiltrando no JS nos bastidores, o futuro de uma performance de programa mais otimizada no JS parece realmente *promissor*.
