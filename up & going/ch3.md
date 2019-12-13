# You Don't Know JS: Iniciando
# Capítulo 3: Dentro do YDKJS

Sobre o que é essa série? Simplificando, trata-se de levar a sério a tarefa de aprender *todas as partes do JavaScript*, e não apenas um subconjunto da linguagem que alguns chamam de "as partes boas", e também não apenas o mínimo que você precisa fazer para terminar o seu trabalho.

Desenvolvedores sérios em outras linguagens esperam colocar seu esforço para aprender a maioria ou a totalidade da linguagem (ou linguagens) que eles mais usam, mas os desenvolvedores JavaScript parecem se destacar da multidão no sentido de normalmente não aprender muito da linguagem em si. Esta não é uma coisa boa, e não é algo que podemos continuar a permitir ser o modelo.

A série *You Don't Know JS* (*YDKJS*) está em contraste gritante com as abordagens típicas para aprender JS, e é diferente de praticamente quaisquer outros livros sobre JS que você vai ler. Ele te desafia a sair da sua zona de conforto e de se perguntar os mais profundos "porquês" para cada comportamento que você encontrar. Você está pronto para esse desafio?

Vou usar este capítulo final para resumir o que esperar do resto dos livros da série, e como ir de forma mais eficaz sobre a construção de uma base de aprendizagem de JavaScript em cima da *YDKJS*.

## Escopos & Clausuras

Talvez uma das coisas mais fundamentais que você precisa aprender, é como o escopo de variáveis realmente funciona em JavaScript. Isso não é o suficiente para ter *opiniões* vagas sobre escopo.

O título *Escopos & Clausuras* começa por desmascarar o equívoco comum que JS é uma "linguagem interpretada" e, portanto, não compilado. Não.

O motor de JS compila seu código logo antes (e às vezes durante!) a execução. Então, faremos uso de uma abordagem mais profunda do compilador para o nosso código, para entender como ele encontra e lida com declarações de variáveis e funções. Ao longo do caminho, vemos a típica metáfora para gerenciamento de escopo de variáveis em JS, *"Hoisting"* (Elevação).

É nesta compreensão crítica do "escopo léxico" que nós iremos basear a nossa exploração de *Clausura* para o último capítulo do livro. Talvez *Clausura* seja o conceito mais importante em toda a linguagem JavaScript, mas se você primeiramente não entender firmemente como o escopo funciona, *Clausura* provavelmente permanecerá fora do seu alcance.

Uma aplicação importante de *Clausura* é o *module pattern*, como nós introduzimos brevemente neste livro, no Capítulo 2. O *module pattern* é, talvez, o padrão de organização de código que mais prevalece em todos JavaScript; a profunda compreensão disso deve ser uma de suas maiores prioridades.

## this & Prototipagem de Objetos

Talvez uma das inverdades mais comuns e persistentes sobre JavaScript é que a palavra-chave `this` se refere à função qual ela aparece. Terrível engano.

A palavra-chave `this` é dinamicamente ligada com base em como a função em questão é executada, e existem quatro regras básicas para entender e determinar plenamente a ligação do `this`.

Intimamente relacionado com a palavra-chave `this`, está o mecanismo protótipo de objeto (object prototype), que é uma cadeia de pesquisa para as propriedades, similar ao modo léxico que o escopo de variáveis é encontrado. Mas envolto nos protótipos, está outro enorme erro sobre JS: a (falsa) ideia de emular classes e (a chamada "prototipagem") herança.

Infelizmente, o desejo de trazer o design pattern (padrão de projeto) de classes e heranças para o JavaScript é simplesmente a pior coisa que você poderia tentar fazer, porque enquanto a sintaxe pode induzí-lo a pensar que há algo como classes, de fato o mecanismo de protótipo é fundamentalmente oposto no seu funcionamento.

O que está em questão é se é melhor ignorar a incompatibilidade e fingir que o que você está implementando é "herança", ou se é mais apropriado aprender e abraçar como o sistema de protótipo de objeto realmente funciona. Este último é mais apropriadamente chamado de "delegação de comportamento."

Então isso é mais sobre preferência sintática. A delegação é um sistema totalmente diferente, e mais poderoso, de design pattern, que substitui a necessidade de planejar classes e heranças. Mas essas afirmações com certeza vão pros ares frente à quase todos os outros post, livros, e conferências sobre o assunto para a duração da vida do JavaScript.

As afirmações que faço sobre delegação contra herança vem não de uma antipatia da linguagem e da sua sintaxe, mas a partir do desejo de ver a verdadeira capacidade da linguagem adequadamente aproveitada, e, a confusão sem fim e frustração indo embora.

Mas o caso que eu faço a respeito de protótipos e a delegação é muito mais do que o que eu vou falar aqui. Se você está pronto para reconsiderar tudo o que você pensa que sabe sobre as "classes" e "heranças" em JavaScript, eu lhe ofereço a oportunidade para "tomar a pílula vermelha" (*Matrix* 1999) e conferir os capítulos 4-6 do título *this & Prototipagem de Objetos* desta série.

## Tipos & Gramática

O terceiro título desta série tem foco principalmente em combater ainda outro tópico altamente controverso: a coerção de tipos. Provavelmente nenhum tópico causa mais frustração entre os desenvolvedores JS que quando você fala sobre as confusões acerca da coerção implícita.

De longe, a sabedoria popular diz que a coerção implícita é uma "parte ruim" da linguagem e deve ser evitada a qualquer custo. Na verdade, alguns tem ido tão longe ao ponto de chamá-la de falha na concepção da linguagem. Realmente, existem ferramentas que tem o foco em nada mais que escanear o código e alertar se você estiver fazendo alguma coisa parecida com a coerção.

Mas a coerção é realmente confusa, tão ruim, tão falsa, que seu código estará condenado desde o início, se você usá-la?

Eu digo que não. Depois de ter consolidado uma compreensão de como tipos e valores realmente funcionam nos capítulos 1-3, o capítulo 4 assume este debate e explicará totalmente como a coerção funciona, com todos os detalhes. Nós veremos apenas que partes da coerção realmente surpreendem e quais partes fazem completo sentido se tiver tempo para aprender.

Mas eu não estou apenas sugerindo que a coerção é sensível e pode ser aprendida, eu estou seguro que a coerção é uma ferramenta incrivelmente útil e totalmente menosprezada que *você deve usar em seu código*. Eu estou dizendo que a coerção, quando usada corretamente, não apenas funciona, mas faz seu código ser melhor. Todos os pessimistas e céticos certamente irão zombar disso, mas eu acredito que isso é uma das principais chaves para elevar seu conhecimento em JS.

Você apenas quer continuar seguindo o que a multidão diz, ou você está disposto a deixar todos os pressupostos de lado e olhar para a coerção com uma nova perspectiva? O volume *Tipos & Gramática* desta série irá te forçar a pensar.

## Async & Performance

Os três primeiros títulos desta série tem o foco nos mecanismos centrais da linguagem, mas o quarto título ramifica-se levemente para cobrir padrões no topo da linguagem para gerenciar a programação assíncrona. Programação assíncrona não é somente crítica para a performance das nossas aplicações, ela está se tornando cada vez mais o fator crítico para a sua escrita e manutenção.

O livro começa primeiramente esclarecendo um monte de conceitos e terminologias confusas acerca de coisas como "assíncrono", "paralelismo" e "simultâneo", e explica detalhadamente como essas coisas se aplicam ou não ao JS.

Em seguida, passamos a examinar os callbacks como o principal método à possibilitar programação assíncrona. Mas é aqui que vemos rapidamente que o callback em si é insuficiente para as exigências modernas de programação assíncrona. Identificamos duas principais deficiências da codificação somente callback: a perda de confiança na *Inversão de Controle* (IoC), e falta de capacidade de raciocinar linearmente.

Para tratar dessas duas importantes deficiências, o ES6 introduz dois novos mecanismos (e, na verdade, padrões): promises e generators.

Promessas (Promises) são um agregador independente, sobre um "valor futuro", que permite você pensar sobre ele e compô-los independentemente do valor estar pronto ou ainda não. Além disso, elas efetivamente resolvem os problemas de confiança na IoC roteando os callbacks através de um confiável e composto mecanismo de promise.

Os Geradores (Generators) introduzem um novo modo de execução para as funções JS, visto que o gerador pode ser pausado em pontos de `yield` e depois continuar de forma assíncrona. A capacidade de pausa-e-continua permite o síncrono, consequentemente, procura código no gerador para ser processado assincronamente nos bastidores. Ao fazer isso, nós abordamos as confusões não-linear, não-saltos-locais de callbacks, e assim, tornar nosso código assíncrono síncrono, procurando um modo de ser mais razoável.

Mas é essa combinação de promessas e geradores que "produz" nosso mais eficaz padrão de codificação assíncrona, até hoje, em JavaScript. De fato, muito do futuro da sofisticação assíncrona está por vir no ES7 e depois certamente será construída sobre esse fundamento. Para ser sério sobre programação de modo eficaz em um mundo assíncrono, você deverá se acostumar com a combinação de promessas e geradores.

Se promessas e geradores representam a expressão de padrões que permitem nossos programas rodarem ao mesmo tempo, e assim, obter mais processamento realizado em um período menor, o JS tem muitas outras facetas de desempenho que vale a pena explorar.

O Capítulo 5 explora temas como o paralelismo de programa com o Web Workers e paralelismo de dados com SIMD, bem como técnicas de otimização de baixo nível, como ASM.js. O Capítulo 6 lança um olhar sobre otimização de desempenho do ponto de vista das técnicas de avaliação comparativas adequadas, incluindo que tipo de desempenho a se preocupar e qual ignorar.

Escrever JavaScript efetivamente significa escrever código que pode quebrar as barreiras de restrição do que está sendo executado de forma dinâmica em uma ampla gama de navegadores e outros ambientes. Isso exige muito esforço e um planejamento complexo e detalhado da nossa parte, para levar um programa de "isso funciona" para "isso funciona bem".

O título *Async & Performance* é feito para te dar todas as ferramentas e habilidades que você precisa para escrever código JavaScript sensato e performático.

## ES6 & Além

Não importa o quanto você se sinta o mestre em JavaScript até aqui, a verdade é que JavaScript nunca deixará de evoluir, e além disso, a taxa de evolução está aumentando rapidamente. Este fato é quase uma metáfora para o espírito desta série, compreender que nós nunca vamos *saber* tudo sobre JS, porque tão logo você dominar tudo, surgirão coisas novas que irão para a fila do que você precisa aprender.

Este título é dedicado tanto para visões de curto e médio prazo para onde a linguagem caminha, não apenas as coisas *conhecidas* como ES6 mas também as coisas *prováveis* que estão por vir.

Enquanto todos os títulos desta série compreendem o estado do JavaScript do momento em que foram escritos, que está na metade do caminho para a adoção do ES6, o foco primário desta série tem sido mais no ES5. Agora, nós queremos voltar nossa atenção para o ES6, ES7, e...

Já que o ES6 está quase completo, no momento da redação deste texto, *ES6 & Além* começa dividindo o matérial sólido da paisagem do ES6 em diversas categorias chave, incluindo a nova sintaxe, as novas (coleções de) estruturas de dados, e a nova capacidade de processamento e APIs. Nós cobriremos cada uma dessas novas características do ES6, em diferentes níveis de detalhe, incluindo a revisão de detalhes que já foram citados em outros livros desta série.

Algumas coisas interessantes do ES6 para acompanhar e ler sobre: desestruturação, parâmetros com valores padrão, símbolos, métodos concisos, propriedades calculadas, arrow functions, bloco de escopo, promessas, geradores, iterators, módulos, proxies, weakmaps, e mais, muito mais! Ufa, o ES6 tem muito poder!

A primeira parte do livro é um roteiro para todas as coisas que você precisa aprender para se preparar para o novo e melhorado JavaScript que você vai escrever e explorar pelos próximos anos.

A última parte do livro volta e foca rapidamente nas coisas que nós provavelmente podemos esperar ver no futuro do JavaScript. A realização mais importante aqui é o pós-ES6, o JS provavelmente vai evoluir funcionalidade por funcionalidade a cada versão, o que significa que podemos esperar para ver coisas num futuro próximo vindo mais cedo do que você pode imaginar.

O futuro para o JavaScript é brilhante. Não é a hora de começarmos a aprendê-lo!?

## Revisão

A série *YDKJS* é dedicada à questão de que todos os desenvolvedores JS podem e devem aprender todas as partes desta grandiosa linguagem. Nem a opinião de alguém, nem framework's, nem prazos de projetos devem ser desculpa para você deixar de aprender e compreender JavaScript profundamente.

Nós pegamos cada área importante da linguagem, focamos e dedicamos um livro curto, mas denso para explorar plenamente todas as partes que você talvez pensou que sabia, mas não provavelmente em sua totalidade.

"You Don't Know JS" ("Você Não Sabe JS") não é uma crítica ou um insulto. Isso é uma percepção que todos nós, inclusive eu, devemos ter. Aprender JavaScript não é um objetivo final, mas um processo. Nós não sabemos JavaScript ainda, mas vamos!
