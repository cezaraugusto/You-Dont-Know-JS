# You Don't Know JS: Async & Performance
# Capítulo 6: Benchmarking & Ajustes

Como os quatro primeiros capítulos deste livro trataram de performance como um padrão de codificação (assincronia e concorrência), e o Capítulo 5 tratou de performance no nível macro da arquitetura de programas, este capítulo aborda o tema da performance no nível micro, focando em expressões/instruções isoladas.

Uma das áreas mais comuns de curiosidade -- de fato, alguns desenvolvedores podem ficar bastante obcecados por isso -- é analisar e testar várias opções de como escrever uma linha ou um bloco de código, e qual delas é mais rápida.

Vamos analisar algumas dessas questões, mas é importante entender desde o início que este capítulo **não** é sobre alimentar a obsessão pelo ajuste de microperformance, como se determinado motor JS consegue executar `++a` mais rápido que `a++`. O objetivo mais importante deste capítulo é descobrir quais tipos de performance em JS importam e quais não importam, *e como diferenciá-los*.

Mas, antes mesmo de chegarmos lá, precisamos explorar como testar a performance JS da forma mais precisa e confiável, porque há toneladas de equívocos e mitos que inundaram nossa base coletiva de conhecimento popular. Temos que peneirar toda essa porcaria para encontrar alguma clareza.

## Benchmarking

OK, hora de começar a dissipar alguns equívocos. Eu apostaria que a grande maioria dos desenvolvedores JS, se solicitados a medir a velocidade (tempo de execução) de uma certa operação, inicialmente fariam algo mais ou menos assim:

```js
var start = (new Date()).getTime();	// ou `Date.now()`

// faça alguma operação

var end = (new Date()).getTime();

console.log( "Duration:", (end - start) );
```

Levante a mão se foi mais ou menos isso que veio à sua mente. Sim, eu imaginei. Há muita coisa errada com essa abordagem, mas não se sinta mal; **todos nós já passamos por isso.**

O que exatamente essa medição lhe disse? Entender o que ela diz e o que não diz sobre o tempo de execução da operação em questão é fundamental para aprender a medir performance de forma apropriada em JavaScript.

Se a duração reportada for `0`, você pode ser tentado a acreditar que levou menos de um milissegundo. Mas isso não é muito preciso. Algumas plataformas não têm precisão de um único milissegundo, mas em vez disso atualizam o cronômetro apenas em incrementos maiores. Por exemplo, versões mais antigas do windows (e portanto do IE) tinham apenas 15ms de precisão, o que significa que a operação tem que levar pelo menos esse tempo para que algo diferente de `0` seja reportado!

Além disso, qualquer que seja a duração reportada, a única coisa que você realmente sabe é que a operação levou aproximadamente esse tempo naquela única execução exata. Você tem confiança próxima de zero de que ela sempre rodará nessa velocidade. Você não tem ideia se o motor ou o sistema sofreram algum tipo de interferência naquele exato momento, e que em outras vezes a operação poderia rodar mais rápido.

E se a duração reportada for `4`? Você está mais seguro de que levou cerca de quatro milissegundos? Não. Pode ter levado menos tempo, e pode ter havido algum outro atraso em obter os timestamps de `start` ou `end`.

Mais preocupante ainda, você também não sabe se as circunstâncias deste teste de operação não são excessivamente otimistas. É possível que o motor JS tenha descoberto uma forma de otimizar seu caso de teste isolado, mas em um programa mais real tal otimização seria diluída ou impossível, de modo que a operação rodaria mais devagar do que no seu teste.

Então... o que sabemos? Infelizmente, com essas constatações expostas, **sabemos muito pouco.** Algo de tão baixa confiança não é nem remotamente bom o suficiente para basear suas conclusões. Seu "benchmark" é basicamente inútil. E pior, é perigoso na medida em que transmite uma falsa confiança, não apenas para você mas também para outros que não pensam criticamente sobre as condições que levaram a esses resultados.

### Repetição

"OK," você diz agora, "Apenas coloque um loop em volta disso para que o teste inteiro demore mais." Se você repetir uma operação 100 vezes, e esse loop inteiro reportadamente leva um total de 137ms, então você pode simplesmente dividir por 100 e obter uma duração média de 1,37ms para cada operação, certo?

Bem, não exatamente.

Uma média matemática pura por si só definitivamente não é suficiente para fazer julgamentos sobre performance que você pretende extrapolar para a amplitude de toda a sua aplicação. Com uma centena de iterações, mesmo um par de valores discrepantes (altos ou baixos) pode distorcer a média, e então, quando você aplica essa conclusão repetidamente, você infla ainda mais a distorção para além da credibilidade.

Em vez de simplesmente rodar por um número fixo de iterações, você pode optar por rodar o loop de testes até que uma certa quantidade de tempo tenha passado. Isso pode ser mais confiável, mas como você decide por quanto tempo rodar? Você pode supor que deveria ser algum múltiplo de quanto tempo sua operação deveria levar para rodar uma vez. Errado.

Na verdade, a duração de tempo a repetir deveria ser baseada na precisão do cronômetro que você está usando, especificamente para minimizar as chances de imprecisão. Quanto menos preciso o seu cronômetro, mais tempo você precisa rodar para ter certeza de que minimizou o percentual de erro. Um cronômetro de 15ms é muito ruim para benchmarking preciso; para minimizar sua incerteza (também conhecida como "taxa de erro") para menos de 1%, você precisa rodar cada ciclo de iterações de teste por 750ms. Um cronômetro de 1ms só precisa de um ciclo rodando por 50ms para obter a mesma confiança.

Mas então, isso é apenas uma única amostra. Para ter certeza de que está descontando a distorção, você vai querer muitas amostras para tirar a média. Você também vai querer entender algo sobre quão lenta é a pior amostra, quão rápida é a melhor amostra, quão distantes estavam esses melhores e piores casos, e assim por diante. Você vai querer saber não apenas um número que lhe diz quão rápido algo rodou, mas também ter alguma medida quantificável de quão confiável esse número é.

Além disso, você provavelmente vai querer combinar essas diferentes técnicas (assim como outras), para obter o melhor equilíbrio de todas as abordagens possíveis.

Isso tudo é o mínimo necessário só para começar. Se você tem abordado o benchmarking de performance com algo menos sério do que aquilo que acabei de pincelar, bem... "you don't know: benchmarking apropriado."

### Benchmark.js

Qualquer benchmark relevante e confiável deve ser baseado em práticas estatisticamente sólidas. Não vou escrever um capítulo sobre estatística aqui, então vou só acenar de leve para alguns termos: desvio padrão, variância, margem de erro. Se você não sabe o que esses termos realmente significam -- eu fiz uma disciplina de estatística lá na faculdade e ainda estou um pouco confuso sobre eles -- você não está realmente qualificado para escrever sua própria lógica de benchmarking.

Felizmente, pessoas inteligentes como John-David Dalton e Mathias Bynens de fato entendem esses conceitos, e escreveram uma ferramenta de benchmarking estatisticamente sólida chamada Benchmark.js (http://benchmarkjs.com/). Então posso acabar com o suspense simplesmente dizendo: "apenas use essa ferramenta."

Não vou repetir toda a documentação deles sobre como o Benchmark.js funciona; eles têm uma fantástica documentação de API (http://benchmarkjs.com/docs) que você deveria ler. Também há ótimos (http://calendar.perfplanet.com/2010/bulletproof-javascript-benchmarks/) artigos (http://monsur.hossa.in/2012/12/11/benchmarkjs.html) com mais detalhes e metodologia.

Mas apenas para fins de ilustração rápida, aqui está como você poderia usar o Benchmark.js para rodar um teste rápido de performance:

```js
function foo() {
	// operação(ões) a testar
}

var bench = new Benchmark(
	"foo test",				// nome do teste
	foo,					// função a testar (apenas o conteúdo)
	{
		// ..				// opções extras opcionais (veja a documentação)
	}
);

bench.hz;					// número de operações por segundo
bench.stats.moe;			// margem de erro
bench.stats.variance;		// variância entre as amostras
// ..
```

Há *muito* mais para aprender sobre o uso do Benchmark.js além dessa olhadela que estou incluindo aqui. Mas o ponto é que ele lida com toda a complexidade de configurar um benchmark de performance justo, confiável e válido para um determinado trecho de código JavaScript. Se você vai tentar testar e medir seu código, essa biblioteca é o primeiro lugar para onde você deveria recorrer.

Estamos mostrando aqui o uso para testar uma única operação como X, mas é bastante comum querer comparar X com Y. Isso é fácil de fazer simplesmente configurando dois testes diferentes em uma "Suite" (um recurso organizacional do Benchmark.js). Então, você os roda lado a lado, e compara as estatísticas para concluir se X ou Y foi mais rápido.

O Benchmark.js pode, é claro, ser usado para testar JavaScript em um navegador (veja a seção "jsPerf.com" mais adiante neste capítulo), mas ele também pode rodar em ambientes fora do navegador (Node.js, etc.).

Um caso de uso amplamente inexplorado do Benchmark.js é usá-lo em seus ambientes de Dev ou QA para rodar testes automatizados de regressão de performance contra partes do caminho crítico do JavaScript da sua aplicação. De forma semelhante a como você pode rodar suítes de testes unitários antes do deploy, você também pode comparar a performance contra benchmarks anteriores para monitorar se você está melhorando ou degradando a performance da aplicação.

#### Setup/Teardown

No trecho de código anterior, passamos por cima do objeto de "opções extras" `{ .. }`. Mas há duas opções que devemos discutir: `setup` e `teardown`.

Essas duas opções permitem definir funções a serem chamadas antes e depois da execução do seu caso de teste.

É incrivelmente importante entender que seu código de `setup` e `teardown` **não roda a cada iteração do teste**. A melhor forma de pensar sobre isso é que há um loop externo (ciclos que se repetem), e um loop interno (iterações de teste que se repetem). `setup` e `teardown` são executados no começo e no fim de cada iteração do loop *externo* (também conhecido como ciclo), mas não dentro do loop interno.

Por que isso importa? Vamos imaginar que você tenha um caso de teste parecido com isto:

```js
a = a + "w";
b = a.charAt( 1 );
```

Então, você configura seu `setup` de teste da seguinte forma:

```js
var a = "x";
```

Sua tentação é provavelmente acreditar que `a` está começando como `"x"` a cada iteração do teste.

Mas não está! Está começando `a` como `"x"` a cada ciclo de teste, e então suas concatenações repetidas de `+ "w"` farão um valor de `a` cada vez maior, mesmo que você só esteja acessando o caractere `"w"` na posição `1`.

Onde isso mais comumente te morde é quando você faz mudanças com efeito colateral em algo como o DOM, como anexar um elemento filho. Você pode pensar que seu elemento pai está sendo configurado como vazio a cada vez, mas na verdade ele está recebendo muitos elementos adicionados, e isso pode influenciar significativamente os resultados dos seus testes.

## O Contexto é Rei

Não se esqueça de checar o contexto de um determinado benchmark de performance, especialmente uma comparação entre as tarefas X e Y. Só porque seu teste revela que X é mais rápido que Y não significa que a conclusão "X é mais rápido que Y" seja de fato relevante.

Por exemplo, digamos que um teste de performance revele que X roda 10.000.000 de operações por segundo, e Y roda a 8.000.000 de operações por segundo. Você poderia afirmar que Y é 20% mais lento que X, e estaria matematicamente correto, mas sua afirmação não tem tanta sustentação quanto você pensaria.

Vamos pensar sobre os resultados de forma mais crítica: 10.000.000 de operações por segundo são 10.000 operações por milissegundo, e 10 operações por microssegundo. Em outras palavras, uma única operação leva 0,1 microssegundo, ou 100 nanossegundos. É difícil conceber quão pequeno é 100ns, mas para comparação, é frequentemente citado que o olho humano geralmente não é capaz de distinguir nada inferior a 100ms, o que é um milhão de vezes mais lento que a velocidade de 100ns da operação X.

Mesmo estudos científicos recentes mostrando que talvez o cérebro consiga processar tão rápido quanto 13ms (cerca de 8x mais rápido do que afirmado anteriormente) significariam que X ainda está rodando 125.000 vezes mais rápido do que o cérebro humano consegue perceber algo distinto acontecendo. **X está indo muito, muito rápido.**

Mas, mais importante, vamos falar sobre a diferença entre X e Y, a diferença de 2.000.000 de operações por segundo. Se X leva 100ns, e Y leva 80ns, a diferença é de 20ns, o que, no melhor caso, ainda é um 650-milésimo do intervalo que o cérebro humano consegue perceber.

Qual é o meu ponto? **Nada dessa diferença de performance importa, em absoluto!**

Mas espere, e se essa operação for acontecer um monte de vezes seguidas? Então a diferença poderia se acumular, certo?

OK, então o que estamos perguntando, então, é: quão provável é que a operação X seja rodada repetidamente, uma logo após a outra, e que isso tenha que acontecer 650.000 vezes só para obter uma fração da esperança de que o cérebro humano consiga percebê-lo? Mais provavelmente, teria que acontecer de 5.000.000 a 10.000.000 de vezes juntas em um loop apertado para sequer se aproximar da relevância.

Embora o cientista da computação dentro de você possa protestar que isso é possível, a voz mais alta do realismo dentro de você deveria fazer uma verificação de sanidade sobre quão provável ou improvável isso realmente é. Mesmo que seja relevante em ocasiões raras, é irrelevante na maioria das situações.

A grande maioria dos seus resultados de benchmark em operações minúsculas -- como o mito do `++x` vs `x++` -- **são simplesmente totalmente falsos** para sustentar a conclusão de que X deveria ser preferido em vez de Y com base em performance.

### Otimizações do Motor

Você simplesmente não pode extrapolar de forma confiável que, se X foi 10 microssegundos mais rápido que Y no seu teste isolado, isso significa que X é sempre mais rápido que Y e deveria sempre ser usado. Não é assim que a performance funciona. É muito mais complicado.

Por exemplo, vamos imaginar (puramente hipotético) que você teste algum comportamento de microperformance como comparar:

```js
var twelve = "12";
var foo = "foo";

// teste 1
var X1 = parseInt( twelve );
var X2 = parseInt( foo );

// teste 2
var Y1 = Number( twelve );
var Y2 = Number( foo );
```

Se você entende o que `parseInt(..)` faz comparado a `Number(..)`, você pode intuir que `parseInt(..)` potencialmente tem "mais trabalho" a fazer, especialmente no caso de `foo`. Ou você pode intuir que eles deveriam ter a mesma quantidade de trabalho a fazer no caso de `foo`, já que ambos deveriam ser capazes de parar no primeiro caractere `"f"`.

Qual intuição está correta? Honestamente não sei. Mas vou defender que não importa qual é a sua intuição. Quais poderiam ser os resultados quando você o testa? Novamente, estou inventando uma hipótese pura aqui, não tentei de fato, nem me importo.

Vamos fingir que o teste retorna que `X` e `Y` são estatisticamente idênticos. Você então confirmou sua intuição sobre a coisa do caractere `"f"`? Não.

É possível, na nossa hipótese, que o motor reconheça que as variáveis `twelve` e `foo` só estão sendo usadas em um lugar em cada teste, e então ele pode decidir fazer o inline desses valores. Então ele pode perceber que `Number( "12" )` pode simplesmente ser substituído por `12`. E talvez ele chegue à mesma conclusão com `parseInt(..)`, ou talvez não.

Ou a heurística de remoção de código morto de um motor poderia entrar em ação, e ele poderia perceber que as variáveis `X` e `Y` não estão sendo usadas, então declará-las é irrelevante, então ele acaba não fazendo nada em nenhum dos testes.

E tudo isso é apenas com a mentalidade de suposições sobre uma única execução de teste. Mecanismos modernos são fantasticamente mais complicados do que estamos intuindo aqui. Eles fazem todo tipo de truque, como rastrear e acompanhar como um trecho de código se comporta ao longo de um curto período de tempo, ou com um conjunto particularmente restrito de entradas.

E se o motor otimiza de uma certa forma por causa da entrada fixa, mas no seu programa real você dá entradas mais variadas e as decisões de otimização se resolvem de forma diferente (ou de forma alguma!)? Ou e se o motor aciona otimizações porque vê o código sendo rodado dezenas de milhares de vezes pelo utilitário de benchmarking, mas no seu programa real ele só vai rodar uma centena de vezes em proximidade, e sob essas condições o motor determina que as otimizações não valem a pena?

E todas aquelas otimizações que acabamos de hipotetizar poderiam acontecer no nosso teste restrito, mas talvez o motor não as fizesse em um programa mais complexo (por várias razões). Ou poderia ser o contrário -- o motor poderia não otimizar um código tão trivial, mas poderia estar mais inclinado a otimizá-lo de forma mais agressiva quando o sistema já está mais sobrecarregado por um programa mais sofisticado.

O ponto que estou tentando defender é que você realmente não sabe ao certo exatamente o que está acontecendo por baixo dos panos. Todas as suposições e hipóteses que você consiga reunir não acrescentam quase nada de concreto para realmente tomar tais decisões.

Isso significa que você não pode realmente fazer nenhum teste útil? **Definitivamente não!**

O que isso resume é que testar código *não real* lhe dá resultados *não reais*. Na medida do possível e do prático, você deveria testar trechos reais e não triviais do seu código, e sob as melhores condições reais que você de fato pode esperar. Só então os resultados que você obtém terão uma chance de se aproximar da realidade.

Microbenchmarks como `++x` vs `x++` são tão incrivelmente prováveis de serem falsos, que mais vale assumirmos categoricamente que são.

## jsPerf.com

Embora o Benchmark.js seja útil para testar a performance do seu código em qualquer ambiente JS em que você esteja rodando, não dá para enfatizar o suficiente que você precisa compilar resultados de testes de muitos ambientes diferentes (navegadores de desktop, dispositivos móveis, etc.) se quiser ter alguma esperança de conclusões de teste confiáveis.

Por exemplo, o Chrome em uma máquina desktop de alto desempenho provavelmente não vai ter desempenho nem perto do mesmo que o Chrome mobile em um smartphone. E um smartphone com a bateria totalmente carregada provavelmente não vai ter desempenho nem perto do mesmo que um smartphone com 2% de bateria restante, quando o dispositivo está começando a desligar o rádio e o processador.

Se você quiser fazer afirmações como "X é mais rápido que Y" em qualquer sentido razoável em mais do que apenas um único ambiente, você vai precisar de fato testar o maior número possível desses ambientes do mundo real. Só porque o Chrome executa alguma operação X mais rápido que Y não significa que todos os navegadores o façam. E é claro que você também provavelmente vai querer cruzar os resultados de várias execuções de teste em navegadores com a demografia dos seus usuários.

Há um site incrível para esse propósito chamado jsPerf (http://jsperf.com). Ele usa a biblioteca Benchmark.js sobre a qual falamos antes para rodar testes estatisticamente precisos e confiáveis, e disponibiliza o teste em uma URL abertamente acessível que você pode repassar a outros.

Cada vez que um teste é rodado, os resultados são coletados e persistidos junto ao teste, e os resultados cumulativos dos testes são plotados em um gráfico na página para qualquer um ver.

Ao criar um teste no site, você começa com dois casos de teste para preencher, mas pode adicionar quantos precisar. Você também tem a capacidade de configurar código de `setup` que roda no começo de cada ciclo de teste e código de `teardown` que roda no fim de cada ciclo.

**Nota:** Um truque para fazer apenas um caso de teste (se você está medindo uma única abordagem em vez de uma comparação direta) é preencher as caixas de entrada do segundo teste com texto de preenchimento na primeira criação, depois editar o teste e deixar o segundo em branco, o que vai apagá-lo. Você sempre pode adicionar mais casos de teste depois.

Você pode definir a configuração inicial da página (importando bibliotecas, definindo funções auxiliares utilitárias, declarando variáveis, etc.). Há também opções para definir o comportamento de setup e teardown se necessário -- consulte a seção "Setup/Teardown" na discussão sobre o Benchmark.js anteriormente.

### Verificação de Sanidade

O jsPerf é um recurso fantástico, mas há uma quantidade enorme de testes publicados que, quando você os analisa, são bastante falhos ou falsos, por qualquer uma de uma variedade de razões delineadas até aqui neste capítulo.

Considere:

```js
// Caso 1
var x = [];
for (var i=0; i<10; i++) {
	x[i] = "x";
}

// Caso 2
var x = [];
for (var i=0; i<10; i++) {
	x[x.length] = "x";
}

// Caso 3
var x = [];
for (var i=0; i<10; i++) {
	x.push( "x" );
}
```

Algumas observações a ponderar sobre este cenário de teste:

* É extremamente comum que devs coloquem seus próprios loops dentro dos casos de teste, e eles esquecem que o Benchmark.js já faz toda a repetição de que você precisa. Há uma chance realmente forte de que os loops `for` nestes casos sejam ruído totalmente desnecessário.
* A declaração e inicialização de `x` está incluída em cada caso de teste, possivelmente de forma desnecessária. Lembre-se de antes que, se `x = []` estivesse no código de `setup`, ele não rodaria de fato antes de cada iteração do teste, mas sim uma vez no começo de cada ciclo. Isso significa que `x` continuaria crescendo bastante, não apenas o tamanho `10` implicado pelos loops `for`.

   Então a intenção é garantir que os testes fiquem restritos apenas a como o motor JS se comporta com arrays muito pequenos (tamanho `10`)? Essa *poderia* ser a intenção, mas se for, você tem que considerar se isso não está focando demais em detalhes nuançados de implementação interna.

   Por outro lado, a intenção do teste abraça o contexto de que os arrays de fato vão crescer bastante? O comportamento dos motores JS com arrays maiores é relevante e preciso quando comparado com o uso pretendido no mundo real?

* A intenção é descobrir o quanto `x.length` ou `x.push(..)` adicionam à performance da operação de anexar ao array `x`? OK, isso pode ser algo válido de testar. Mas, por outro lado, `push(..)` é uma chamada de função, então é claro que vai ser mais lento que o acesso `[..]`. Pode-se argumentar que os casos 1 e 2 são mais justos que o caso 3.


Aqui está outro exemplo que ilustra uma falha comum de comparar coisas incomparáveis (maçãs com laranjas):

```js
// Caso 1
var x = ["John","Albert","Sue","Frank","Bob"];
x.sort();

// Caso 2
var x = ["John","Albert","Sue","Frank","Bob"];
x.sort( function mySort(a,b){
	if (a < b) return -1;
	if (a > b) return 1;
	return 0;
} );
```

Aqui, a intenção óbvia é descobrir o quanto o comparador customizado `mySort(..)` é mais lento que o comparador padrão embutido. Mas ao especificar a função `mySort(..)` como uma expressão de função inline, você criou um teste injusto/falso. Aqui, o segundo caso não está apenas testando uma função JS customizada do usuário, **mas também está testando a criação de uma nova expressão de função a cada iteração.**

Te surpreenderia descobrir que, se você rodar um teste semelhante mas atualizá-lo para isolar apenas a criação de uma expressão de função inline versus o uso de uma função pré-declarada, a criação da expressão de função inline pode ser de 2% a 20% mais lenta!?

A menos que a sua intenção com este teste *seja* considerar o "custo" da criação da expressão de função inline, um teste melhor/mais justo colocaria a declaração de `mySort(..)` no setup da página -- não a coloque no `setup` do teste, pois isso é uma redeclaração desnecessária a cada ciclo -- e simplesmente a referenciaria pelo nome no caso de teste: `x.sort(mySort)`.

Com base no exemplo anterior, outra armadilha está em, de forma opaca, evitar ou adicionar "trabalho extra" a um caso de teste, o que cria um cenário de maçãs-com-laranjas:

```js
// Caso 1
var x = [12,-14,0,3,18,0,2.9];
x.sort();

// Caso 2
var x = [12,-14,0,3,18,0,2.9];
x.sort( function mySort(a,b){
	return a - b;
} );
```

Deixando de lado a armadilha da expressão de função inline mencionada anteriormente, o `mySort(..)` do segundo caso funciona neste caso porque você forneceu números, mas é claro que teria falhado com strings. O primeiro caso não lança um erro, mas ele de fato se comporta de forma diferente e tem um resultado diferente! Deveria ser óbvio, mas: **um resultado diferente entre dois casos de teste quase certamente invalida o teste inteiro!**

Mas, para além dos resultados diferentes, neste caso, o comparador do `sort(..)` embutido está de fato fazendo "trabalho extra" que o `mySort()` não faz, na medida em que o embutido coage os valores comparados para strings e faz comparação lexicográfica. O primeiro trecho resulta em `[-14, 0, 0, 12, 18, 2.9, 3]` enquanto o segundo trecho resulta (provavelmente de forma mais precisa com base na intenção) em `[-14, 0, 0, 2.9, 3, 12, 18]`.

Então esse teste é injusto porque ele não está de fato fazendo a mesma tarefa entre os casos. Quaisquer resultados que você obtenha são falsos.

Essas mesmas armadilhas podem até ser muito mais sutis:

```js
// Caso 1
var x = false;
var y = x ? 1 : 2;

// Caso 2
var x;
var y = x ? 1 : 2;
```

Aqui, a intenção pode ser testar o impacto na performance da coerção para um Boolean que o operador `? :` vai fazer se a expressão `x` ainda não for um Boolean (veja o título *Types & Grammar* desta série de livros). Então, você aparentemente está OK com o fato de que há trabalho extra para fazer a coerção no segundo caso.

O problema sutil? Você está definindo o valor de `x` no primeiro caso e não o definindo no outro, então você está de fato fazendo trabalho no primeiro caso que não está fazendo no segundo. Para eliminar qualquer distorção potencial (ainda que menor), tente:

```js
// Caso 1
var x = false;
var y = x ? 1 : 2;

// Caso 2
var x = undefined;
var y = x ? 1 : 2;
```

Agora há uma atribuição em ambos os casos, então a coisa que você quer testar -- a coerção de `x` ou não -- provavelmente foi isolada e testada de forma mais precisa.

## Escrevendo Bons Testes

Deixe-me ver se consigo articular o ponto maior que estou tentando defender aqui.

A boa autoria de testes requer um pensamento analítico cuidadoso sobre quais diferenças existem entre dois casos de teste e se as diferenças entre eles são *intencionais* ou *não intencionais*.

Diferenças intencionais são, é claro, normais e OK, mas é fácil demais criar diferenças não intencionais que distorcem seus resultados. Você tem que ser muito, muito cuidadoso para evitar essa distorção. Além disso, você pode pretender uma diferença, mas ela pode não ser óbvia para outros leitores do seu teste qual era sua intenção, então eles podem duvidar (ou confiar!) no seu teste de forma incorreta. Como você corrige isso?

**Escreva testes melhores e mais claros.** Mas também, dedique tempo a documentar (usando o campo "Description" do jsPerf.com e/ou comentários no código) exatamente qual é a intenção do seu teste, até o detalhe mais nuançado. Aponte as diferenças intencionais, o que ajudará os outros e o seu eu futuro a identificar melhor as diferenças não intencionais que poderiam estar distorcendo os resultados do teste.

Isole as coisas que não são relevantes para o seu teste, pré-declarando-as nas configurações de setup da página ou do teste, para que fiquem fora das partes cronometradas do teste.

Em vez de tentar focar em um trecho minúsculo do seu código real e medir apenas aquela parte fora de contexto, testes e benchmarks são melhores quando incluem um contexto maior (embora ainda relevante). Esses testes também tendem a rodar mais devagar, o que significa que quaisquer diferenças que você detectar são mais relevantes no contexto.

## Microperformance

OK, até agora estivemos dançando em torno de várias questões de microperformance e geralmente olhando para elas com desfavor, em relação à obsessão por elas. Quero dedicar apenas um momento para abordá-las diretamente.

A primeira coisa com a qual você precisa se sentir mais confortável ao pensar sobre benchmarking de performance do seu código é que o código que você escreve nem sempre é o código que o motor de fato roda. Olhamos brevemente para esse tema lá no Capítulo 1 quando discutimos o reordenamento de instruções pelo compilador, mas aqui vamos sugerir que o compilador às vezes pode decidir rodar um código diferente do que você escreveu, não apenas em ordens diferentes mas diferente em substância.

Vamos considerar este trecho de código:

```js
var foo = 41;

(function(){
	(function(){
		(function(baz){
			var bar = foo + baz;
			// ..
		})(1);
	})();
})();
```

Você pode pensar que a referência a `foo` na função mais interna precisa fazer uma busca de escopo em três níveis. Cobrimos no título *Scope & Closures* desta série de livros como o escopo léxico funciona, e o fato de que o compilador geralmente faz cache de tais buscas, de modo que referenciar `foo` a partir de escopos diferentes não realmente "custa" nada extra na prática.

Mas há algo mais profundo a considerar. E se o compilador perceber que `foo` não é referenciado em nenhum outro lugar a não ser naquela única localização, e ele ainda notar que o valor nunca é nada além do `41` mostrado?

Não seria bastante possível e aceitável que o compilador JS pudesse decidir simplesmente remover a variável `foo` por completo, e fazer o *inline* do valor, como isto:

```js
(function(){
	(function(){
		(function(baz){
			var bar = 41 + baz;
			// ..
		})(1);
	})();
})();
```

**Nota:** É claro que o compilador também poderia provavelmente fazer uma análise e reescrita semelhantes com a variável `baz` aqui, também.

Quando você começa a pensar no seu código JS como sendo uma dica ou sugestão para o motor sobre o que fazer, em vez de uma exigência literal, você percebe que muito da obsessão por minúcias sintáticas discretas é muito provavelmente infundada.

Outro exemplo:

```js
function factorial(n) {
	if (n < 2) return 1;
	return n * factorial( n - 1 );
}

factorial( 5 );		// 120
```

Ah, o bom e velho algoritmo do "fatorial"! Você pode supor que o motor JS vai rodar esse código mais ou menos como está. E, para ser honesto, ele pode -- eu não tenho certeza.

Mas, como uma anedota, o mesmo código expresso em C e compilado com otimizações avançadas resultaria no compilador percebendo que a chamada `factorial(5)` pode simplesmente ser substituída pelo valor constante `120`, eliminando a função e a chamada por completo!

Além disso, alguns motores têm uma prática chamada "desenrolamento de recursão" (unrolling recursion), na qual ele pode perceber que a recursão que você expressou pode na verdade ser feita "mais facilmente" (ou seja, de forma mais ótima) com um loop. É possível que o código anterior pudesse ser *reescrito* por um motor JS para rodar como:

```js
function factorial(n) {
	if (n < 2) return 1;

	var res = 1;
	for (var i=n; i>1; i--) {
		res *= i;
	}
	return res;
}

factorial( 5 );		// 120
```

Agora, vamos imaginar que no trecho anterior você estivesse preocupado se `n * factorial(n-1)` ou `n *= factorial(--n)` roda mais rápido. Talvez você até tenha feito um benchmark de performance para tentar descobrir qual era melhor. Mas você perde o fato de que, no contexto maior, o motor pode não rodar nenhuma das linhas de código porque ele pode desenrolar a recursão!

Falando em `--`, `--n` versus `n--` é frequentemente citado como um daqueles lugares onde você pode otimizar ao escolher a versão `--n`, porque teoricamente ela requer menos esforço lá embaixo no nível de processamento de assembly.

Esse tipo de obsessão é basicamente um disparate no JavaScript moderno. Esse é o tipo de coisa que você deveria deixar o motor cuidar. Você deveria escrever o código que faz mais sentido. Compare estes três loops `for`:

```js
// Opção 1
for (var i=0; i<10; i++) {
	console.log( i );
}

// Opção 2
for (var i=0; i<10; ++i) {
	console.log( i );
}

// Opção 3
for (var i=-1; ++i<10; ) {
	console.log( i );
}
```

Mesmo que você tenha alguma teoria de que a segunda ou terceira opção é mais performática que a primeira opção por uma minúscula fração, o que é duvidoso na melhor das hipóteses, o terceiro loop é mais confuso porque você tem que começar com `-1` para `i` para levar em conta o fato de que o pré-incremento `++i` é usado. E a diferença entre a primeira e a segunda opções é realmente bastante irrelevante.

É inteiramente possível que um motor JS possa ver um lugar onde `i++` é usado e perceber que ele pode com segurança substituí-lo pelo equivalente `++i`, o que significa que o tempo que você gastou decidindo qual escolher foi completamente desperdiçado e o resultado é discutível.

Aqui está outro exemplo comum de obsessão boba por microperformance:

```js
var x = [ .. ];

// Opção 1
for (var i=0; i < x.length; i++) {
	// ..
}

// Opção 2
for (var i=0, len = x.length; i < len; i++) {
	// ..
}
```

A teoria aqui diz que você deveria fazer cache do tamanho do array `x` na variável `len`, porque supostamente ele não muda, para evitar pagar o preço de `x.length` ser consultado a cada iteração do loop.

Se você rodar benchmarks de performance em torno do uso de `x.length` comparado a fazer cache dele em uma variável `len`, você vai descobrir que, embora a teoria pareça boa, na prática quaisquer diferenças medidas são estatisticamente completamente irrelevantes.

De fato, em alguns motores como o v8, pode-se demonstrar (http://mrale.ph/blog/2014/12/24/array-length-caching.html) que você poderia tornar as coisas ligeiramente piores ao fazer o pré-cache do tamanho em vez de deixar o motor descobrir isso por você. Não tente ser mais esperto que o seu motor JavaScript, você provavelmente vai perder quando o assunto é otimizações de performance.

### Nem Todos os Motores São Iguais

Os diferentes motores JS em vários navegadores podem todos estar "em conformidade com a spec" enquanto têm formas radicalmente diferentes de lidar com o código. A especificação JS não exige nada relacionado a performance -- bem, exceto a "Tail Call Optimization" do ES6 abordada mais adiante neste capítulo.

Os motores são livres para decidir que uma operação receberá sua atenção para ser otimizada, talvez trocando isso por menor performance em outra operação. Pode ser muito tênue encontrar uma abordagem para uma operação que sempre roda mais rápido em todos os navegadores.

Há um movimento entre alguns na comunidade de devs JS, especialmente aqueles que trabalham com Node.js, de analisar os detalhes específicos de implementação interna do motor JavaScript v8 e tomar decisões sobre escrever código JS que é adaptado para tirar o melhor proveito de como o v8 funciona. Você pode de fato alcançar um grau surpreendentemente alto de otimização de performance com tais empreitadas, então o retorno pelo esforço pode ser bastante alto.

Alguns exemplos comumente citados (https://github.com/petkaantonov/bluebird/wiki/Optimization-killers) para o v8:

* Não passe a variável `arguments` de uma função para qualquer outra função, pois tal "vazamento" deixa a implementação da função mais lenta.
* Isole um `try..catch` em sua própria função. Navegadores têm dificuldade em otimizar qualquer função com um `try..catch` dentro dela, então mover essa construção para sua própria função significa que você contém o dano da des-otimização enquanto deixa o código ao redor ser otimizável.

Mas em vez de focar nessas dicas especificamente, vamos fazer uma verificação de sanidade da abordagem de otimização específica do v8 em um sentido geral.

Você está genuinamente escrevendo código que só precisa rodar em um único motor JS? Mesmo que seu código seja inteiramente destinado ao Node.js *agora*, a suposição de que o v8 *sempre* será o motor JS usado é confiável? É possível que algum dia, daqui a alguns anos, haja outra plataforma JS do lado do servidor além do Node.js na qual você escolha rodar seu código? E se aquilo para o qual você otimizou antes for agora uma forma muito mais lenta de fazer aquela operação no novo motor?

Ou e se o seu código sempre continuar rodando no v8 daqui em diante, mas o v8 decidir em algum ponto mudar a forma como algum conjunto de operações funciona, de modo que o que costumava ser rápido agora é lento, e vice-versa?

Esses cenários também não são apenas teóricos. Costumava ser que era mais rápido colocar múltiplos valores de string em um array e então chamar `join("")` no array para concatenar os valores do que simplesmente usar concatenação com `+` diretamente com os valores. A razão histórica para isso é nuançada, mas tem a ver com detalhes de implementação interna sobre como valores de string eram armazenados e gerenciados na memória.

Como resultado, o conselho de "boa prática" da época se disseminou por toda a indústria sugerindo que os desenvolvedores sempre usassem a abordagem do `join(..)` de array. E muitos seguiram.

Só que, em algum ponto do caminho, os motores JS mudaram as abordagens para gerenciar strings internamente, e especificamente colocaram otimizações para a concatenação com `+`. Eles não deixaram o `join(..)` mais lento, por si só, mas colocaram mais esforço em ajudar o uso do `+`, já que ele ainda era bastante mais difundido.

**Nota:** A prática de padronizar ou otimizar alguma abordagem particular baseando-se principalmente em seu uso já difundido é frequentemente chamada (metaforicamente) de "pavimentar o caminho do gado" (paving the cowpath).

Uma vez que essa nova abordagem para lidar com strings e concatenação se firmou, infelizmente todo o código por aí que estava usando `join(..)` de array para concatenar strings ficou então sub-ótimo.

Outro exemplo: certa vez, o navegador Opera diferia de outros navegadores na forma como lidava com o boxing/unboxing de objetos wrapper de primitivos (veja o título *Types & Grammar* desta série de livros). Como tal, o conselho deles para desenvolvedores era usar um objeto `String` em vez do valor primitivo `string` se propriedades como `length` ou métodos como `charAt(..)` precisassem ser acessados. Esse conselho pode ter sido correto para o Opera na época, mas era literalmente o completo oposto para outros grandes navegadores contemporâneos, já que eles tinham otimizações especificamente para os primitivos `string` e não para seus contrapartes wrapper de objeto.

Eu acho que essas várias pegadinhas são pelo menos possíveis, se não prováveis, para código até mesmo hoje. Então sou muito cauteloso em fazer otimizações de performance de amplo alcance no meu código JS baseando-me puramente em detalhes de implementação de motores, **especialmente se esses detalhes só forem verdadeiros para um único motor**.

O inverso também é algo a ter cautela: você não deveria necessariamente mudar um trecho de código para contornar a dificuldade de um motor em rodar um trecho de código de uma forma aceitavelmente performática.

Historicamente, o IE tem sido o alvo de muitas dessas frustrações, dado que houve muitos cenários em versões mais antigas do IE em que ele tinha dificuldade com algum aspecto de performance com o qual outros grandes navegadores da época pareciam não ter muito problema. A discussão sobre concatenação de strings que acabamos de ter era de fato uma preocupação real lá nos dias do IE6 e IE7, quando era possível obter melhor performance com `join(..)` do que com `+`.

Mas é problemático sugerir que o problema de performance de apenas um navegador seja justificativa para usar uma abordagem de código que muito possivelmente poderia ser sub-ótima em todos os outros navegadores. Mesmo que o navegador em questão tenha uma grande participação de mercado para o público do seu site, pode ser mais prático escrever o código apropriado e confiar que o navegador se atualizará com melhores otimizações eventualmente.

"Não há nada mais permanente do que um hack temporário." É provável que o código que você escreve agora para contornar algum bug de performance vá sobreviver ao próprio bug de performance no navegador.

Nos dias em que um navegador só se atualizava uma vez a cada cinco anos, essa era uma decisão mais difícil de tomar. Mas como estão as coisas agora, os navegadores em geral estão se atualizando em um intervalo muito mais rápido (embora obviamente o mundo mobile ainda fique para trás), e todos eles estão competindo para otimizar recursos da web cada vez melhor.

Se você se deparar com um caso em que um navegador *de fato* tem uma verruga de performance que outros não sofrem, certifique-se de reportá-la a eles através de quaisquer meios que você tenha disponíveis. A maioria dos navegadores tem rastreadores de bugs públicos e abertos, adequados para esse propósito.

**Dica:** Eu só sugeriria contornar um problema de performance em um navegador se ele fosse um impedimento realmente drástico, não apenas um incômodo ou frustração. E eu seria muito cuidadoso em verificar que o hack de performance não tivesse efeitos colaterais negativos perceptíveis em outro navegador.

### O Panorama Geral

Em vez de nos preocuparmos com todas essas nuances de microperformance, deveríamos em vez disso olhar para tipos de otimizações de visão ampla.

Como você sabe o que é visão ampla ou não? Você tem que primeiro entender se o seu código está rodando em um caminho crítico ou não. Se não estiver no caminho crítico, é provável que suas otimizações não valham muito.

Já ouviu a admoestação, "isso é otimização prematura!"? Ela vem de uma famosa citação de Donald Knuth: "otimização prematura é a raiz de todo o mal.". Muitos desenvolvedores citam essa frase para sugerir que a maioria das otimizações são "prematuras" e portanto são um desperdício de esforço. A verdade é, como de costume, mais nuançada.

Aqui está a citação de Knuth, em contexto:

> Programadores desperdiçam enormes quantidades de tempo pensando, ou se preocupando, com a velocidade de partes **não críticas** de seus programas, e essas tentativas de eficiência na verdade têm um forte impacto negativo quando a depuração e a manutenção são consideradas. Deveríamos esquecer pequenas eficiências, digamos cerca de 97% do tempo: otimização prematura é a raiz de todo o mal. Ainda assim, não deveríamos deixar passar nossas oportunidades naqueles **críticos** 3%. [ênfase adicionada]

(http://web.archive.org/web/20130731202547/http://pplab.snu.ac.kr/courses/adv_pl05/papers/p261-knuth.pdf, Computing Surveys, Vol 6, No 4, December 1974)

Acredito que é uma paráfrase justa dizer que Knuth *quis dizer*: "otimização de caminho não crítico é a raiz de todo o mal." Então a chave é descobrir se o seu código está no caminho crítico -- você deveria otimizá-lo! -- ou não.

Eu iria até tão longe a ponto de dizer isto: nenhuma quantidade de tempo gasta otimizando caminhos críticos é desperdiçada, não importa quão pouco seja economizado; mas nenhuma quantidade de otimização em caminhos não críticos é justificada, não importa quanto seja economizado.

Se o seu código está no caminho crítico, como um trecho de código "quente" (hot) que vai ser rodado repetidas vezes, ou em lugares críticos de UX onde os usuários vão notar, como um loop de animação ou atualizações de estilo CSS, então você não deveria poupar esforços em tentar empregar otimizações relevantes e mensuravelmente significativas.

Por exemplo, considere um loop de animação de caminho crítico que precisa coagir um valor de string para um número. Há, é claro, múltiplas formas de fazer isso (veja o título *Types & Grammar* desta série de livros), mas qual delas, se alguma, é a mais rápida?

```js
var x = "42";	// precisa do número `42`

// Opção 1: deixar a coerção implícita acontecer automaticamente
var y = x / 2;

// Opção 2: usar `parseInt(..)`
var y = parseInt( x, 0 ) / 2;

// Opção 3: usar `Number(..)`
var y = Number( x ) / 2;

// Opção 4: usar o operador unário `+`
var y = +x / 2;

// Opção 5: usar o operador unário `|`
var y = (x | 0) / 2;
```

**Nota:** Vou deixar como exercício para o leitor configurar um teste se você estiver interessado em examinar as diferenças minúsculas de performance entre essas opções.

Ao considerar essas diferentes opções, como dizem, "uma dessas coisas não é como as outras." `parseInt(..)` faz o trabalho, mas também faz muito mais -- ele faz o parsing da string em vez de apenas coagir. Você provavelmente consegue adivinhar, corretamente, que `parseInt(..)` é uma opção mais lenta, e você provavelmente deveria evitá-la.

É claro que, se `x` puder ser um valor que **precisa de parsing**, como `"42px"` (como de uma busca de estilo CSS), então `parseInt(..)` realmente é a única opção adequada!

`Number(..)` também é uma chamada de função. De uma perspectiva comportamental, ele é idêntico à opção do operador unário `+`, mas pode de fato ser um pouco mais lento, requerendo mais maquinário para executar a função. É claro que também é possível que o motor JS reconheça essa simetria comportamental e simplesmente faça o inline do comportamento de `Number(..)` (também conhecido como `+x`) para você!

Mas lembre-se, obcecar-se por `+x` versus `x | 0` é na maioria dos casos provavelmente um desperdício de esforço. Essa é uma questão de microperformance, e uma que você não deveria deixar ditar/degradar a legibilidade do seu programa.

Embora a performance seja muito importante em caminhos críticos do seu programa, ela não é o único fator. Entre várias opções que são aproximadamente semelhantes em performance, a legibilidade deveria ser outra preocupação importante.

## Otimização de Chamada de Cauda (TCO)

Como mencionamos brevemente antes, o ES6 inclui uma exigência específica que se aventura no mundo da performance. Ela está relacionada a uma forma específica de otimização que pode ocorrer com chamadas de função: a *tail call optimization*.

Em resumo, uma "tail call" é uma chamada de função que aparece na "cauda" (tail) de outra função, de modo que, depois que a chamada termina, não há mais nada a fazer (exceto talvez retornar seu valor de resultado).

Por exemplo, aqui está uma configuração não recursiva com tail calls:

```js
function foo(x) {
	return x;
}

function bar(y) {
	return foo( y + 1 );	// tail call
}

function baz() {
	return 1 + bar( 40 );	// não é tail call
}

baz();						// 42
```

`foo(y+1)` é uma tail call em `bar(..)` porque, depois que `foo(..)` termina, `bar(..)` também está terminada, exceto, neste caso, por retornar o resultado da chamada `foo(..)`. No entanto, `bar(40)` *não* é uma tail call porque, depois que ela se completa, seu valor de resultado deve ser somado a `1` antes que `baz()` possa retorná-lo.

Sem entrar em detalhes minuciosos demais, chamar uma nova função requer uma quantidade extra de memória reservada para gerenciar a pilha de chamadas, chamada de "stack frame" (quadro de pilha). Então o trecho anterior geralmente exigiria um stack frame para cada um de `baz()`, `bar(..)` e `foo(..)` todos ao mesmo tempo.

No entanto, se um motor com capacidade de TCO conseguir perceber que a chamada `foo(y+1)` está em *posição de cauda* (tail position), o que significa que `bar(..)` está basicamente completa, então, ao chamar `foo(..)`, ele não precisa criar um novo stack frame, mas pode em vez disso reutilizar o stack frame existente de `bar(..)`. Isso não é apenas mais rápido, mas também usa menos memória.

Esse tipo de otimização não é grande coisa em um trecho simples, mas se torna *uma coisa muito mais importante* ao lidar com recursão, especialmente se a recursão pudesse ter resultado em centenas ou milhares de stack frames. Com TCO o motor pode realizar todas essas chamadas com um único stack frame!

A recursão é um tema cabeludo em JS porque, sem TCO, os motores têm tido que implementar limites arbitrários (e diferentes!) para quão fundo eles deixam a pilha de recursão chegar antes de pará-la, para evitar ficar sem memória. Com TCO, funções recursivas com chamadas em *posição de cauda* podem essencialmente rodar de forma ilimitada, porque nunca há nenhum uso extra de memória!

Considere aquele `factorial(..)` recursivo de antes, mas reescrito para torná-lo amigável a TCO:

```js
function factorial(n) {
	function fact(n,res) {
		if (n < 2) return res;

		return fact( n - 1, n * res );
	}

	return fact( n, 1 );
}

factorial( 5 );		// 120
```

Esta versão de `factorial(..)` ainda é recursiva, mas também é otimizável com TCO, porque ambas as chamadas internas de `fact(..)` estão em *posição de cauda*.

**Nota:** É importante notar que TCO só se aplica se realmente houver uma tail call. Se você escrever funções recursivas sem tail calls, a performance ainda vai recair na alocação normal de stack frames, e os limites dos motores sobre tais pilhas de chamadas recursivas ainda vão se aplicar. Muitas funções recursivas podem ser reescritas como acabamos de mostrar com `factorial(..)`, mas isso requer atenção cuidadosa aos detalhes.

Uma razão pela qual o ES6 exige que os motores implementem TCO, em vez de deixar a critério deles, é porque a *falta de TCO* na verdade tende a reduzir as chances de que certos algoritmos sejam implementados em JS usando recursão, por medo dos limites da pilha de chamadas.

Se a falta de TCO no motor apenas degradasse graciosamente para uma performance mais lenta em todos os casos, provavelmente não teria sido algo que o ES6 precisasse *exigir*. Mas como a falta de TCO pode de fato tornar certos programas impraticáveis, é mais um recurso importante da linguagem do que apenas um detalhe de implementação oculto.

O ES6 garante que, de agora em diante, os desenvolvedores JS poderão confiar nesta otimização em todos os navegadores compatíveis com ES6+. Isso é uma vitória para a performance JS!

## Revisão

Medir efetivamente a performance de um trecho de código, especialmente para compará-lo a outra opção para aquele mesmo código a fim de ver qual abordagem é mais rápida, requer atenção cuidadosa aos detalhes.

Em vez de criar sua própria lógica de benchmarking estatisticamente válida, apenas use a biblioteca Benchmark.js, que faz isso por você. Mas tenha cuidado com a forma como você escreve os testes, porque é fácil demais construir um teste que parece válido mas que na verdade é falho -- até mesmo diferenças minúsculas podem distorcer os resultados a ponto de torná-los completamente não confiáveis.

É importante obter o máximo de resultados de teste do maior número possível de ambientes diferentes para eliminar o viés de hardware/dispositivo. O jsPerf.com é um site fantástico para fazer crowdsourcing de execuções de benchmark de performance.

Muitos testes de performance comuns infelizmente se obcecam por detalhes irrelevantes de microperformance, como `x++` versus `++x`. Escrever bons testes significa entender como focar em preocupações de visão ampla, como otimizar no caminho crítico, e evitar cair em armadilhas como os detalhes de implementação de diferentes motores JS.

A tail call optimization (TCO) é uma otimização exigida a partir do ES6 que tornará alguns padrões recursivos práticos em JS onde eles teriam sido impossíveis de outra forma. A TCO permite que uma chamada de função na *posição de cauda* de outra função seja executada sem precisar de nenhum recurso extra, o que significa que o motor não precisa mais impor restrições arbitrárias sobre a profundidade da pilha de chamadas para algoritmos recursivos.
