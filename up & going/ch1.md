# You Don't Know JS: Iniciando
# Capítulo 1: Iniciando

Bem vindo à série *You Don't Know JS* (*YDKJS*).

*Iniciando* é uma introdução à diversos conceitos básicos de programação -- claro que inclinado ao uso do JavaScript (muitas vezes abreviado como JS) especificamente -- e como abordar e entender o resto dos títulos nessa série. Especialmente se você está começando em programação ou JavaScript, esse livro irá, de forma breve, explorar tudo que você precisa saber para *iniciar*.

Esse livro começa explicando os princípios básicos de programação em uma camada mais alta. É mais indicado se você está começando *YDKJS* com pouca ou nenhuma experiência anterior com programação, e está procurando nesses livros uma ajuda para começar na longa jornada de entender como programar através da visão do JavaScript.

O Capítulo 1 deve ser abordado como uma visão geral de coisas que você irá querer aprender e praticar mais para *iniciar na programação*. Existem também outros recursos para entender melhor essa introdução à programação e eu encorajo você a aprender através deles em adição à este capítulo.

Uma vez que você se sentir confortável com os conceitos básicos da programação, o Capítulo 2 irá te familiarizar com o modo de programar com JavaScript. O Capítulo 2 faz uma introdução sobre do que o JavaScript é capaz, mas novamente, ele não é um guia compreensivo -- esse é a finalidade do resto da série *YDKJS*!

Se você já se sente confortável com JavaScript, dê uma olhada no Capítulo 3 como uma breve visão do que esperar com *YDKJS*, e caia de cabeça!

## Código

Vamos *começar do começo*.

Um programa, também conhecido como *código fonte* ou apenas *código*, é um conjunto de instruções especializadas para dizer ao computador quais tarefas ele deve realizar. Geralmente, códigos são salvos em arquivos de texto, apesar de que, em JavaScript, você também pode escrever códigos direto no developer console do navegador, o qual iremos cobrir em breve.

As regras para formatos e combinações de instruções é chamado *linguagem de computação*, e algumas vezes é referenciado como *sintaxe*, bem parecido como a lingua Portuguesa diz à você a forma de pronunciar palavras e de como criar sentenças válidas usando palavras e pontuações.

### Instruções

Em linguagens de computação, um grupo de palavras, números e operadores que realizam tarefas específicas são uma *instrução*. Em JavaScript, uma instrução pode ser parecida com o seguinte:

```js
a = b * 2;
```

Os caracteres `a` e `b` são chamados *variáveis* (veja "Variáveis"), que são recipientes em que você pode armazenar qualquer coisa dentro. Em programas, variáveis detém valores (como o número `42`) que serão utilizados pelo programa. Pense neles como nomes simbólicos para chamarmos os valores.

Em contrapartida, o `2` é apenas um valor, chamado *valor literal*, por que é apresentado sozinho, sem estar armazenado em uma variável.

Os caracteres `=` e `*` são *operadores* (veja "Operadores") -- eles realizam ações com os valores e variáveis como realizar uma atribuição qualquer ou uma multiplicação matemática.

A maioria das instruções em JavaScript termina com um ponto e vírgula (`;`) no final.

A instrução `a = b * 2;` diz ao computador, grosseiramente falando, para pegar o valor atual dentro da variável `b`, multiplicar esse valor por `2`, depois armazenar o resultado dentro de outra variável, chamada `a`.

Programas são apenas coleções de muitas instruções, que juntas descrevem todas as etapas que são necessárias para cumprir a finalidade do programa.

### Expressões

Instruções são feitas de uma ou mais *expressões*. Uma expressão é qualquer referência à uma variável ou valor, ou um conjunto de variáveis e valores combinados com operadores.

Por exemplo:

```js
a = b * 2;
```

Essa instrução tem quatro expressões dentro dela:

* `2` é uma *expressão de valor literal*
* `b` é uma *expressão de valor variável*, que significa que ela armazena seu valor atual
* `b * 2` é uma *expressão aritmética*, que significa execute a multiplicação
* `a = b * 2` é uma *expressão de atribuição*, que significa designar o resultado da expressão `b * 2` para a variável `a` (mais instruções depois).

Uma expressão genérica que permanece sozinha é também chamada de *instrução de expressão*, como o exemplo abaixo:

```js
b * 2;
```

Esse tipo de instrução de expressão não é muito comum ou útil, como geralmente a instrução não afeta o desenvolvimento do programa -- ela apenas pega o valor armazenado por `b` e multiplica por `2`, sem realizar nenhuma ação com esse resultado.

Uma instrução de expressão mais comum é chamada instrução de *expressão de chamada* (veja "Funções"), sendo a própria chamada da função uma instrução completa:

```js
alert( a );
```

### Executando um Programa

Como essas coleções de instruções em programação dizem ao computador o que fazer? O programa *precisa ser executado*, ou, mais comumente usado, precisamos *rodar o programa*.

Instruções como `a = b * 2` são úteis quando desenvolvedores estão lendo e escrevendo, mas não são numa forma que o computador possa entender. Sendo assim, uma ferramenta especial (tanto um *interpretador* como um *compilador*) é usado para traduzir o código que você escreveu em comandos que o computador possa entender.

Para algumas linguagens, essa tradução dos comandos é típicamente feita de cima para baixo, linha por linha, cada vez que o programa roda. Essas etapas são geralmente chamadas de *interpretação* do código.
**NT** *Definem uma linguagem interpretada.*

Para outras linguagens, a tradução é feita em tempos distintos, chamado *compilamento* do código. Dessa forma, o programa *roda* depois, ou seja: o que está rodando são as instruções prontas, já compiladas.
**NT** *Definem uma linguagem compilada.*

Tipicamente, afirma-se que o JavaScript é uma linguagem *interpretada*, porque o código é processado a cada vez que roda. Essa afirmação não é totalmente verdadeira. Na verdade, a *engine* do JavaScript *compila* o programa no mesmo instante e imediatamente roda o código compilado.

**Nota:** Para mais informações sobre compilação em JavaScript, veja os dois primeiros capítulos do livro desta série *Escopos & Clausuras*.

## Tente você mesmo

Esse capítulo irá introduzir você em cada conceito de programação com snippets simples de código, todos escritos em JavaScript (claro!).

Nunca é demais enfatizar: enquanto você lê este capítulo -- e você poderá precisar vir aqui diversas vezes -- você deve praticar cada um desses conceitos escrevendo o código você mesmo. A forma mais fácil de fazer isso é abrir o console do *developer tools* do seu navegador favorito (Firefox, Chrome, IE, etc.).

**Dica:** Geralmente, você pode abrir o console com um atalho do teclado ou um item do menu. Para informações mais detalhadas sobre abrir e utilizar o console do seu navegador favorito, veja "Mastering The Developer Tools Console" (http://blog.teamtreehouse.com/mastering-developer-tools-console). Para digitar mais de uma linha no console, use `<shift> + <enter>` para mover para próxima linha. Uma vez digitado `<enter>`, o console irá rodar tudo que você digitou.

Vamos nos familiarizar com o processo de rodar o código no console. Primeiro, sugiro que abra uma aba em branco no seu navegador. Eu prefiro fazer isso digitando `about:blank` na barra de endereços. Feito isso, abra o console, como acabamos de mencionar.

Agora, digite o código abaixo e veja como ele se comporta:

```js
a = 21;

b = a * 2;

console.log( b );
```

Digitar o código acima no console do Chrome deverá produzir algo parecido com isso:

<img src="fig1.png" width="500">

Vá em frente, tente também! A melhor forma de aprender programação é produzindo códigos!

### Output

No exemplo anterior, usamos o `console.log(..)`. Vamos, superficialmente, entender o que essa linha de código faz.

Você deve ter suspeitado: essa é exatamente a forma como imprimimos texto (também conhecido como *output*) no *console* do desenvolvedor.

Primeiro, a parte do `log( b )` é usada como uma função de chamada (veja "Funções"). O que está acontecendo é que estamos usando a variável `b` na função para pegar seu valor e imprimir no console.

Depois, a parte do `console.` é uma referência ao objeto onde a função `log(..)`está localizada. Iremos cobrir objetos e suas propriedades com mais detalhes no Capítulo 2.

Outra forma de criar um output que você possa visualizar é rodar a instrução `alert(..)`. Por exemplo:

```js
alert( b );
```

Se você rodar esse comando, irá perceber que ao invés de imprimir o resultado no console, um popup com o conteúdo da variável `b` e um botão de "OK" irão aparecer. Entretanto, usar `console.log(..)` em geral vai facilitar seu aprendizado e a forma de rodar seus programas, mais do que se estivesse usando o `alert(..)`, porque com o `console.log(..)` você pode expressar mais valores de uma vez sem interromper a interface do navegador.

Para esse livro, iremos usar sempre o `console.log(..)` para os nossos *outputs*.

### Input

Enquanto estávamos discutindo sobre o output, você deve ter se perguntado sobre o *input* (em outras palavras, receber informações do usuário).

A forma mais comum de isso acontecer é através de um formulário em uma página HTML (como caixas de texto) para o usuário inserir suas informações e depois usar JS para ler esses valores nas variáveis do seu programa.

Mas existe uma maneira ainda mais fácil de conseguir um input com a finalidade de demonstração e aprendizado como as que você irá fazer ao longo desse livro. Usando a função `prompt(..)`:

```js
age = prompt( "Please tell me your age:" );

console.log( age );
```

Como você deve ter suspeitado, a mensagem a ser passada para o `prompt(..)` -- nesse caso, `"Please tell me your age:"` -- é impresso no popup.

O resultado deve ser parecido com a imagem abaixo:

<img src="fig2.png" width="500">

Uma vez que você enviar a informação ao clicar em "OK", observe que o valor que você digitou é armazenado na variável `age`, que nós então fazemos o *output* com `console.log(..)`:

<img src="fig3.png" width="500">

Para manter as coisas simples enquanto estamos aprendendo os conceitos básicos da programação, os exemplos desse livro não irão precisar de input. Mas agora que você viu como usar o `prompt(..)`, se você quiser se desafiar, pode tentar usar input ao explorar os exemplos.

## Operadores

Operadores são como realizar uma ação em variáveis e valores. Nós já vimos até agora dois operadores em JavaScript, o `=` e o `*`.

O operador `*` realiza uma multiplicação matemática. Simples o suficiente, não?

O operador de igualdade `=` é usado para *atribuir* -- primeiro calculamos o valor do *lado da mão direita* (valor original) do `=` e então o colocamos em uma variável que especificamos no *lado da mão esquerda* (variável de destino).

**Atenção:** Essa pode parecer uma ordem reversa estranha de especificar uma atribuição. Ao invés de `a = 42`, algumas pessoas preferem inverter a ordem do valor original na esquerda e a variável de destino na direita, algo como `42 -> a` (isso não é JavaScript valido!). Infelizmente, a forma ordenada `a = 42` e variações similares, prevalece em linguagens de programação modernas. Caso pareça uma forma não-natural, tome algum tempo assimilando essa forma na sua cabeça até se sentir acostumado.

Considere:

```js
a = 2;
b = a + 1;
```

Aqui, atribuimos o valor`2` à variável `a`. Assim, pegamos o valor da variável `a` (ainda `2`), adicionamos `1` a ele, resultando no valor `3`, então armazenamos esse valor na variável `b`.

Apesar de não ser tecnicamente um operador, você irá precisar da palavra-chave `var` em cada programa, por ser o primeiro modo de *declarar* (conhecido como *criar*) *var*iáveis (veja "Variáveis").

Você sempre deve declarar a variável por nome antes de usá-la. Mas você precisa declarar a variável apenas uma vez para cada *escopo* (veja "Escopo"); ela pode ser usada depois quantas vezes forem necessárias. Por exemplo:

```js
var a = 20;

a = a + 1;
a = a * 2;

console.log( a );	// 42
```

Aqui encontram-se os operadores mais comuns em JavaScript:

* Atribuição: `=` como em `a = 2`.
* Aritmético: `+` (adição), `-` (subtração), `*` (multiplicação), e `/` (divisão), como em `a * 3`.
* Atribuição com Operação: `+=`, `-=`, `*=`, e `/=` são operadores comuns que combinam operadores aritméticos com a atribuição, como em `a += 2` (o mesmo que `a = a + 2`).
* Incremento/Decremento: `++` (incremento), `--` (decremento), como em `a++` (similar à `a = a + 1`).
* Acesso a propriedade do Objeto: `.` como em `console.log()`.

   Objetos são valores que armazenam outros valores em lugares determinados por nome, chamados propriedades. `obj.a` significa que este é um objeto chamado `obj` com uma propriedade de nome `a`. Propriedades podem, alternativamente, serem chamados por `obj["a"]`. Veja mais no capítulo 2.
* Igualdade: `==` (igualdade), `===` (igualdade estrita), `!=` (desigualdade), `!==` (desigualdade estrita), como em `a == b`.

   Veja "Valores & Tipos" e o Capítulo 2.
* Comparação: `<` (menor que), `>` (maior que), `<=` (menor ou igual), `>=` (maior ou igual), como em `a <= b`.

   Veja "Valores & Tipos" e o Capítulo 2.
* Lógicos: `&&` (e), `||` (ou), como em `a || b` que seleciona `a` *ou* `b`.

   Esses operadores são usados para expressar instruções condicionais (veja "Condicionais"), como *se* `a` *ou* `b` for verdadeiro.

**Note:** Para muito mais detalhes, e cobertura dos operadores não mencionados aqui, veja mais no Mozilla Developer Network (MDN)'s "Expressões e Operadores" (https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Guide/Expressions_and_Operators).

## Valores & Tipos

Se você abordar uma vendedora de uma loja de celulares e perguntar quanto um certo modelo custa, e ela disser "noventa e nove e noventa e nove" ($99.99), ela está fornecendo um valor numérico que representa quanto você vai precisar pagar para comprar o aparelho. Se você quiser levar dois desses celulares, você pode facilmente fazer uma conta mental e dobrar o valor para encontrar o valor $199,98.

Se a vendedora pegar outro aparelho similar e disser "é grátis", ela não está te fornecendo um valor numérico, mas está fazendo um outro tipo de representação de um valor que é esperado ($0.00) -- a palavra "grátis."

Se você depois perguntar se o aparelho vem com carregador, a resposta pode ser apenas "sim" ou "não."

De modo bem similar, quando você expressa valores em um programa, você escolhe representações diferentes para aqueles valores baseado no que você planeja fazer com eles.

Essas diferentes representações para valores são chamados *tipos* na terminologia de programação. JavaScript tem tipos pré-definidos para cada um dos chamados valores *primitivos*:

* Quando quiser fazer operações matemáticas, você vai precisar de um `number`.
* Quando você precisar imprimir um valor na tela, você precisará de uma `string` (um ou mais caracteres, palavras, sentenças).
* Quando você precisar tomar uma decisão em seu programa, você vai precisar de um `boolean` (`true` ou `false`).

Valores que são incluídos diretamente no código fonte são chamados *literais*. Literais `string` são sempre envolvidas por aspas duplas `"..."` ou aspas simples (`'...'`) -- a única diferença é a preferência estética. Literais `number` e `boolean` são apresentadas como são (exemplos: `42`, `true`, etc.).

Considere:

```js
"I am a string";
'I am also a string';

42;

true;
false;
```

Além de tipos como `string`/`number`/`boolean`, é comum para linguagens de programação proverem *arrays*, *objetos*, *funções*, e mais. Iremos cobrir muito mais sobre valores e tipos ao longo desse capítulo e também do próximo.

### Coerções entre Tipos

Se você tem um número(`number`) mas precisa imprimí-lo na tela, você precisará converter o valor para uma `string`, e em JavaScript isso é chamado de "coerção." De maneira similar, se alguém insere uma série de caracteres numéricos em um formulário de uma página de ecommerce, isso é uma `string`, mas se você precisar usar esse valor para fazer operações matemáticas, você vai precisar *converter* para um número(`number`).

O JavaScript fornece diversas facilidades para forçar a coerção entre *tipos*. Por exemplo:

```js
var a = "42";
var b = Number( a );

console.log( a );	// "42"
console.log( b );	// 42
```

Usando `Number(..)` (uma função nativa) como demonstrado, estamos realizando uma coerção *explícita* de qualquer outro tipo para o tipo `number` (número). Isso deve ser bem claro.

Um tópico controverso acontece quando você tenta comparar dois valores que ainda não são do mesmo tipo, que requer uma coerção *implícita*.

Quando comparada a string `"99.99"` com o número `99.99`, muitos concordam que elas sejam equivalentes. Mas ele não são exatamente iguais, são? É o mesmo valor em duas representações diferentes, dois *tipos* diferentes. Você poderia dizer que eles são "igualdade nao-estrita", não poderia?

Para te ajudar nessas situações, o JavaScript irá, em alguns casos, *implicitamente* converter os valores para os tipos certos.

Sendo assim, se você usar o operador de igualdade não-estrita `==`  para fazer uma comparação entre `"99.99" == 99.99`, o JavaScript vai converter o lado da mão esquerda `"99.99"` para seu número(`number`) equivalente `99.99`. A comparação então se torna `99.99 == 99.99`, que é claro, é verdadeira (`true`).

Apesar de ter sido feito para te ajudar, coerções implícitas geram confusão se você não teve tempo de aprender as regras que regem seu comportamento. A maioria dos desenvolvedores de JS nunca tiveram, então o sentimento geral é que coerções implícitas são confusas e deixam os programas com bugs inesperados, e os mesmos devem ser evitados. Em alguns casos até o design da linguagem é considerado falho.

Entretanto, coerções implícitas é um mecanismo que *pode ser aprendido*, e mais ainda *deve ser aprendido* por qualquer um que queira levar a programação em JavaScript a sério. Não apenas as coerções não são confusas, uma vez que aprendidas as regras, como pode fazer os seus programas melhores! Os esforços para aprender valerão a pena.

**Nota:** Para mais informações sobre coerções, veja o Capítulo 2 desse título (Iniciando) e o Capítulo 4 de *Tipos & Gramática* da série.

## Comentários do Código

A vendedora da loja de celulares pode escrever algumas notas sobre funcionalidades de um recém-lançado aparelho ou sobre os novos planos que a empresa oferece. Essas notas são apenas para ela ler -- elas não são feitas para os consumidores lerem. De qualquer forma, essas notas ajudam a vendedora a fazer seu trabalho por documentar os "como's" e "porquê's" do que deve-se falar aos consumidores.

Uma das lições mais importantes que você pode aprender sobre códigos é que eles não são apenas para computadores. O código é feito tanto, senão mais, para o desenvolvedor do que para o compilador.

Seu computador se importa apenas com código de máquina, uma série de binários, 0s e 1s, que vem da *compilação*. Existe uma infinidade de programas que você pode escrever que produzem as mesmas séries de 0s e 1s. As escolhas que você faz sobre como programar importam -- não apenas para você, mas para toda a equipe que você está trabalhando e até para você mesmo no futuro.

Você deve se empenhar não apenas em escrever programas que funcionam corretamente, mas programas que fazem sentido ao serem examinados. Você pode percorrer uma boa parte desse caminho começando por escolher bons nomes para variáveis (veja "Variáveis") e funções (veja "Funções").

Uma parte importante do nosso código são os comentários. Eles são blocos de texto no seu programa que são inseridos com o propósito único de explicar coisas a um humano. O interpretador/compilador sempre irá ignorar esses comentários.

Existem diversas opiniões sobre o que faz um código ser bem documentado; não podemos definir regras universais. Entretanto, algumas observações e orientações são bastante úteis:

* Códigos sem comentários não são ideais.
* Muitos comentários (um por linha, por exemplo) é provavelmente sinal de código mal escrito.
* Comentários devem explicar *porquê* e não *o quê*. Eles podem opcionalmente explicar *como* se a parte for particularmente confusa.

Em JavaScript, existem dois tipos possíveis de comentário: uma linha simples de comentário e um comentário multi-linhas.

Considere:

```js
// Esse é um comentário de linha simṕles

/* Mas esse é
       um comentário
             multi-linhas.
                      */
```

O `//` comentário de linha simples é apropriado se você está fazendo um comentário logo acima de uma instrução, ou até mesmo no final da linha. Tudo após o início da linha de `//` é tratado como um comentário (e consequentemente ignorado pelo compilador), até o final da linha. Não existem restrições sobre o que pode ser posto dentro de um comentário de linha simples.

Considere:

```js
var a = 42;		// 42 é o sentido da vida
```

O `/* .. */` comentário de multi-linhas é apropriado se você precisar de diversas linhas de explicações para fazer seu comentário.

Abaixo uma forma comum de usar um comentário de multiplas linhas:

```js
/* O valor abaixo é usado porque
   foi exposto que ele responde
   a todas as questões do universo. */
var a = 42;
```

O comentário multi-linhas pode inclusive aparecer em uma linha, ou até mesmo no meio de uma linha, porque o `*/` finaliza ele. Por exemplo:

```js
var a = /* valor arbitrário */ 42;

console.log( a );	// 42
```

Porém a única coisa que não pode aparecer dentro de um comentário multi-linhas é um`*/`, porque seria interpretado como final do comentário.

Você definitivamente irá querer começar seu aprendizado na programação com o hábito de comentar seu código. Através desse capítulo, você verá que uso comentários para explicar coisas, então faça isso com suas práticas. Confie em mim, todos que irão ler seu código vão agradecer!

## Variáveis

A maioria dos programas que são úteis, precisam rastrear um valor conforme ele se modifica ao longo do programa, realizando diferentes operações enquanto o programa o chama de acordo com determinadas tarefas.

A forma mais fácil de realizar esse procedimento é determinar um valor para um agregador simbólico, chamado *variável* -- sendo assim chamado porque o valor que carrega pode *variar* ao longo do tempo se necessário.

Em algumas linguagens de programação, você declara a variável (agregador) para manter um valor específico, como por exemplo um `number` ou `string`. A *Tipagem estática*, é geralmente citada como benéfica para o programa por previnir coerções de valores não desejadas.

Outras linguagens enfatizam tipos para valores ao invés de variáveis. A *Tipagem fraca*, também conhecida como *tipagem dinâmica*, permite que uma variável armazene qualquer tipo de valor em qualquer tempo. É tipicamente citada como benéfica para a flexibilidade de um programa, por permitir que uma simples variável tenha um valor representado independentemente do tipo e do momento dentro decorrer da lógica.

O JavaScript usa a segunda abordagem, *tipagem dinâmica*, o que significa que as variáveis podem armazenar valores de qualquer *tipo*.

Como mencionado anteriormente, nós declaramos uma variável usando a instrução `var` -- note que não existe aqui nenhuma informação sobre *tipo* nessa declaração. Considere esse simples programa:

```js
var amount = 99.99;

amount = amount * 2;

console.log( amount );		// 199.98

// converte o amount` para uma string, e
// adiciona "$" no começo
amount = "$" + String( amount );

console.log( amount );		// "$199.98"
```

A variável `amount` começa armazenando o número `99.99`, e depois armazena o resultado númerico(`number`) de `amount * 2`, que é `199.98`.

O primeiro comando e `console.log(..)` precisa *implicitamente* converter o valor de `number` para `string` para poder imprimí-lo.

A instrução `amount = "$" + String(amount)` *explicitamente* converte o valor `199.98` para uma `string` e adiciona um caractere `"$"` para o começo. Nesse ponto, `amount` agora armazena o valor em `string` de `"$199.98"`, então o segundo comando `console.log(..)` não precisa fazer nenhuma coerção para imprimir seu valor.

Os desenvolvedores JavaScript irão notar a flexibilidade de usar a variável `amount` para cada um dos valores `99.99`, `199.98` e `"$199.98"`. Entusiastas da tipagem estática irão preferir separar valores como `amountStr` para armazenar a representação do valor `"$199.98"`, por ser de um tipo diferente.

De qualquer forma, você irá notar que `amount` armazena um valor corrente que é alterado conforme o decorrer do programa, ilustrando assim o primeiro propósito das variáveis: gerenciar o *estado* do programa.

Em outras palavras, *estado* é o acompanhamento das mudanças dos valores conforme seu programa está rodando.

Outro exemplo comum de como usar uma variável é quando você deseja definir as opções de valores. Isso é geralmente chamado de *constante*, quando você declara uma variável com um valor e deseja que o valor *não mude* ao longo do programa.

Você declara essas *constantes* geralmente no início do programa, de forma a se tornar um lugar conveniente de se visitar caso deseje alterar algum valor. Por convenção, variáveis definidas como constantes em JavaSscript são geralmente capitalizadas e separadas por um sublinhado.

Abaixo um exemplo simples:

```js
var TAX_RATE = 0.08;	// 8% de taxas

var amount = 99.99;

amount = amount * 2;

amount = amount + (amount * TAX_RATE);

console.log( amount );				// 215.9784
console.log( amount.toFixed( 2 ) );	// "215.98"
```

**Nota:** Assim como `console.log(..)` tem a função `log(..)` acessada como uma propriedade do valor do objeto de `console`, `toFixed(..)`é uma função que pode ser acessada para valores `number`. O `number` em JavaScript não é automaticamente formatado para ser o valor de uma moeda -- o sistema não sabe o que se pretende fazer e não existe um tipo específico para moedas. `toFixed(..)` nos deixa especificar quantos valores decimais gostaríamos que o `number` fosse arredondado, e ele produz a `string` como necessário.

A variável `TAX_RATE` só é uma *constante* por convenção -- não existe nada especial nesse programa que não permita que ela seja alterada. Mas se a cidade aumentar o valor das taxas para 9%, nós ainda poderemos atualizar o valor de `TAX_RATE` para `0.09` no mesmo lugar, ao invés de procurar diversas ocorrências do valor `0.08` ao longo do programa e ter que atualizá-los um por um.

A nova versão do JavaScript no momento que escrevo isso (comumente chamada de "ES6") inclui uma nova forma de declarar *constantes*, usando `const` ao invés de `var`:

```js
// como em ES6:
const TAX_RATE = 0.08;

var amount = 99.99;

// ..
```

Constantes são tão úteis quanto variáveis de valores imutáveis, exceto que constantes também previnem que valores sejam alterados acidentalmente após sua configuração inicial. Se você tentar designar diferentes valores para `TAX_RATE` após sua primeira declaração, seu programa vai rejeitar a mudança (e em modo estrito, irá gerar um erro -- veja "Modo Estrito" no Capítulo 2).

Falando nisso, o tipo de "proteção" contra acidentes é similar ao de linguagens com tipagem estática, assim você vê como a tipagem estática em outras linguagens pode ser bem atrativa!

**Nota:** Para mais informações sobre os diferentes valores em variáveis que podem ser usados em seus programas, veja o título *Tipos & Gramática* dessa série.

## Blocos

A vendedora da loja de celulares precisa seguir uma série de etapas para efetivar a compra do seu novo celular.

De maneira similar, em código nós muitas vezes precisamos agrupar uma série de instruções, as quais podemos chamar de *blocos*. Em JavaScript, um bloco é definido por englobar uma ou mais instruções dentro de um par de chaves `{ .. }`. Considere:

```js
var amount = 99.99;

// um bloco qualquer
{
	amount = amount * 2;
	console.log( amount );	// 199.98
}
```

Essa forma de formatação do bloco`{ .. }` é válida, mas não é muito comum de se ver em programas em JS. Tipicamente, blocos são anexados a outros tipos de controle, como dentro de uma condicional `if` (veja "Condicionais") ou em um laço (veja "Loops"). Por exemplo:

```js
var amount = 99.99;

// amount é grande o suficiente?
if (amount > 10) {			// <-- bloco anexado ao `if`
	amount = amount * 2;
	console.log( amount );	// 199.98
}
```

Iremos explicar condicionais `if` na próxima seção, mas como você pode ver, o bloco `{ .. }`com suas duas instruções é anexado ao `if (amount > 10)`; as instruções dentro do bloco só irão ser processadas se a condicional for aceita.

**Nota:** Ao contrário da maioria das instruções como `console.log(amount);`, uma instrução de bloco não precisa de um ponto-e-vírgula (`;`) para ser concluída.

## Condicionais

"Você gostaria de adicionar protetores de tela extras à sua compra, por $9.99". A atenciosa vendedora da loja de celulares fez você tomar uma decisão. E você primeiro precisa consultar o *estado* corrente da sua conta bancária para responder à essa pergunta. Mas, obviamente, essa é uma questão de um simples "sim" ou "não".

Existem diversas formas das quais podemos expressar *condicionais* (ou decisões) em nossos programas.

A forma mais comum é a condicional `if`. Essencialmente, você está dizendo, "*se* essa condição for verdadeira, faça isso...".

```js
var bank_balance = 302.13;
var amount = 99.99;

if (amount < bank_balance) {
	console.log( "Quero comprar esse celular!" );
}
```
A condicional `if` requer uma expressão entre parênteses `( )` que pode ser definida como verdadeira (`true`) ou falsa (`false`). Nesse programa, declaramos a expressão `amount < bank_balance`, que irá determinar se o valor é `true` ou  `false`, dependendo da quantidade dentro da variável `bank_balance`.

Você pode até mesmo definir uma alternativa para se a condição não for verdadeira, uma cláusula chamada `else`. Considere:

```js
const ACCESSORY_PRICE = 9.99;

var bank_balance = 302.13;
var amount = 99.99;

amount = amount * 2;

// podemos fazer uma compra extra?
if ( amount < bank_balance ) {
	console.log( "Vou levar este acessório!" );
	amount = amount + ACCESSORY_PRICE;
}
// se não pudermos:
else {
	console.log( "Não, obrigado." );
}
```

Aqui, se `amount < bank_balance` for `true`, iremos imprimir `"Vou levar este acessório!"` e adicionar `9.99` para a nossa variável `amount`. Ou senão pudermos, a cláusula `else` diz que podemos responder, polidamente, `"Não, obrigado."` e deixar o `amount` inalterado.

Como discutimos em "Valores & Tipos" anteriormente, valores que não são de algum tipo anteriormente definido, geralmente é coergido para o novo tipo. Se a condicional `if` esperar um tipo `boolean`, mas o argumento que você passou for de algum tipo que não seja `boolean`, uma coerção irá acontecer.

O JavaScript define uma lista de valores específicos que são considerados "falsinhos" porque quando coergido para `boolean`, eles se tornam `false` -- esses valores incluem `0` e `""`. Qualquer outro valor não incluído na lista de  "falsinhos" será automaticamente definido como "verdadeirinho" -- quando coergidos para `boolean` se tornam `true`. Valores verdadeirinhos incluem coisas como `99.99` e `"free"`. Veja "Verdadeirinhos & Falsinhos" no Capítulo 2 para mais informaçoes.

*Condicionais* existem em outras formas além do `if`. Por exemplo, a instrução `switch` pode ser usada como um atalho para uma série de instruções `if..else` (veja o Capítulo 2). Os Loops (veja "Loops") usam uma *condicional* para determinar se um loop deve prosseguir rodando ou parar.

**Nota:** Para conhecer mais a fundo sobre coerções que podem ocorrer implicitamente ao testar expressões em *condicionais*, veja o Capítulo 4 do título desta série *Tipos & Gramática*.

## Loops

Durante dias movimentados, existe uma lista de espera de consumidores que precisam falar com a vendedora da loja de celulares. Enquanto houverem pessoas na lista, ela precisa continuar atendendo o próximo consumidor.

Repetir uma série de ações até que certa condição falhe -- em outras palavras, repetir apenas enquanto a condição for respeitada -- é o trabalho dos loops na programação (**NT**: traduzido também como *laços*); os loops podem ter diversas formas, mas todos eles têm esse mesmo comportamento.

Um loop inclui a condição teste, assim como o bloco (tipicamente um `{ .. }`). A cada vez que o bloco de loop é executado, damos o nome de *iteração*.

Por exemplo, o loop `while` e o loop `do..while` ilustram o conceito de repetir um bloco de instruções até uma condição deixar de ser verdadeira (`true`):

```js
while (numOfCustomers > 0) {
	console.log( "Como posso ajudar?" );

	// Ajude o consumidor...

	numOfCustomers = numOfCustomers - 1;
}

// versus:

do {
	console.log( "Como posso ajudar?" );

	// Ajude o consumidor...

	numOfCustomers = numOfCustomers - 1;
} while (numOfCustomers > 0);
```

A única diferença prática entre esses dois loops é como a condicional é executada, se antes da primeira iteração (`while`) ou após a primeira iteração (`do..while`).

De qualquer forma, se o teste da condicional retorna falso (`false`), a próxima iteração não irá rodar. Isso significa que se a condição inicial for `false`, um loop `while` nunca irá rodar, mas um loop `do..while` irá rodar apenas a primeira vez.

Às vezes você está fazendo um loop com a finalidade de contar uma certa quantidade de números, como de `0` à `9` (dez números). Você pode fazer isso definindo uma iteração utilizando uma variável `i` para o valor `0` e ir incrementando `1` para cada iteração.

**Atenção:** Por diversas questões históricas, linguagens de programação quase sempre começam a contar pelo zero, o que significa que elas começam com `0` ao invés de `1`. Se você não é familiar com essa forma de pensar, pode ser um pouco confuso no início. Tire um tempo para praticar contando a partir do `0` para se ambientar com essa forma.

A condicional é testada em cada iteração, mesmo se existir uma condicional `if` dentro do loop.

Podemos usar a instrução `break`para parar um loop. Além disso, podemos observar que é terrivelmente fácil criar um loop que pode rodar para sempre sem um mecanismo que o faça parar.

Vamos ilustrar isso:

```js
var i = 0;

// um loop `while..true` iria rodar pra sempre, certo?
while (true) {
	// parar o loop?
	if ((i <= 9) === false) {
		break;
	}

	console.log( i );
	i = i + 1;
}
// 0 1 2 3 4 5 6 7 8 9
```

**Atenção:** Essa não é necessariamente uma forma prática que você usaria para executar para seus loops. Ela é apresentada aqui com propósito ilustrativo apenas.

Enquanto os loops `while` (ou `do..while`) podem fazer a tarefa manualmente, existe outra forma, outro loop chamado `for` que serve justamente para essa finalidade:

```js
for (var i = 0; i <= 9; i = i + 1) {
	console.log( i );
}
// 0 1 2 3 4 5 6 7 8 9
```

Como você pôde ver, nos dois casos a condicional `i <= 9` foi verdadeira(`true`) para as 10 primeiras iterações (`i` para os valores de `0` até `9`) em ambas as formas do loop, mas se torna falsa(`false`) uma vez que o valor de `i` chega a `10`.

O loop `for` tem três instruções: uma atribuição inicial (`var i = 0`), um teste condicional (`i <= 9`), e uma atualização (`i = i + 1`). Sendo assim, se o que você pretende fazer com a iteração é uma contagem, `for` é a forma mais compacta e em geral mais fácil de entender e escrever.

Existem outros loops especializados que são designados a iterar sobre valores específicos, como propriedades de um objeto (veja o Capítulo 2) onde a aplicação do teste condicional é saber se todas as propriedades foram processadas. O conceito de "iterar até determinada condição falhar" permanece independentemente do formato do loop.

## Funções

A vendedora da loja de celulares provavelmente não anda com uma calculadora para saber a quantidade de taxas e o valor final do produto. Essa é uma tarefa que ela precisa definir uma vez e reusar diversas vezes. As chances são que a empresa que ela trabalha tenha um aparelho (computador, tablet, etc) que tenha, de fábrica, esse tipo de funcionalidade.

De maneira similar, o programa quase certamente irá dividir o código em
partes reusáveis, ao invés de ficar se repetindo. A forma que fazemos isso é definindo uma função (`function`).

Uma função geralmente é um bloco de código nomeado, de forma que ele possa ser "chamado" pelo nome, fazendo o código dentro dele ser acionado sempre que preciso. Considere:

```js
function printAmount() {
	console.log( amount.toFixed( 2 ) );
}

var amount = 99.99;

printAmount(); // "99.99"

amount = amount * 2;

printAmount(); // "199.98"
```

Funções podem, opcionalmente, carregar argumentos (conhecidos também como parâmetros) -- valores que você designa a ela. E eles podem, também opcionalmente, retornar um outro valor.

```js
function printAmount(amt) {
	console.log( amt.toFixed( 2 ) );
}

function formatAmount() {
	return "$" + amount.toFixed( 2 );
}

var amount = 99.99;

printAmount( amount * 2 );		// "199.98"

amount = formatAmount();
console.log( amount );			// "$99.99"
```

A função `printAmount(..)` utiliza um parâmetro que chamamos `amt`. A função `formatAmount()` retorna um valor. Claro, você também pode combinar essas duas técnicas na mesma função.

Funções são geralmente usadas para códigos que você planeja chamar diversas vezes, mas eles podem ser úteis também para organizar códigos relacionados em coleções que você possa nomear, mesmo que você só planeja chamá-los apenas uma vez.

Considere:

```js
const TAX_RATE = 0.08;

function calculateFinalPurchaseAmount(amt) {
	// calcula o novo amount adicionando a tax
	amt = amt + (amt * TAX_RATE);

	// retorne o novo amount
	return amt;
}

var amount = 99.99;

amount = calculateFinalPurchaseAmount( amount );

console.log( amount.toFixed( 2 ) );		// "107.99"
```

Apesar de `calculateFinalPurchaseAmount(..)` ser chamado apenas uma vez, organizar seu comportamento em uma função separadamente faz o código que usa sua lógica (a instrução `amount = calculateFinal...`) mais limpa. Se a função tiver mais instruções nela, os benefícios podem ser ainda maiores.

### Escopo

Se você pedir à vendedora da loja de celulares por um modelo que não está em estoque, ela não poderá te vender o celular que você quer. Ela só tem acesso aos aparelhos que estão em estoque, você terá que ir até outra loja para saber se eles têm o telefone que você deseja.

Em programação temos um termo para esse conceito: *escopo* (tecnicamente chamado *escopo léxico*). Em JavaScript, cada função tem seu próprio escopo. O escopo é basicamente uma coleção de variáveis e regras de como essas variáveis serão acessadas pelo nome. Apenas o código dentro dessa função poderá acessar as variáveis dentro daquele *escopo*.

O nome de uma variável precisa ser único dentro do escopo -- não podem haver duas variáveis diferentes com o nome `a` no mesmo escopo. Porém duas variáveis com o nome `a` em escopos diferentes podem coexistir sem problemas.

```js
function one() {
	// essa variável `a` só pertence à função `one()`
	var a = 1;
	console.log( a );
}

function two() {
	// essa variável `a` só pertence à função `two()`
	var a = 2;
	console.log( a );
}

one();		// 1
two();		// 2
```

Também podemos aninhar um escopo dentro de outro escopo, igual a um palhaço numa festa de aniversário e estoura um balão que contém outro balão dentro. Se um escopo está aninhado a outro, o código dentro do escopo interno pode acessar as variáveis do escopo mais externo.

Considere:

```js
function outer() {
	var a = 1;

	function inner() {
		var b = 2;

		// nós podemos acessar ambos `a` e `b` aqui
		console.log( a + b );	// 3
	}

	inner();

	// podemos acessar apenas `a` aqui
	console.log( a );			// 1
}

outer();
```

As regras do escopo léxico dizem que o código dentro de um escopo pode acessar variáveis tanto dele mesmo quanto de qualquer escopo fora dele.

Assim, considere que a função `inner()` tem acesso a ambas as variáveis `a` e `b`, mas o código dentro de `outer()` só tem acesso à `a` -- ele não pode acessar `b` porque a variável está dentro de `inner()`.

Voltando ao exemplo anterior:

```js
const TAX_RATE = 0.08;

function calculateFinalPurchaseAmount(amt) {
	// calcula o novo amount adicionando a tax
	amt = amt + (amt * TAX_RATE);

	// retorne o novo amount
	return amt;
}
```

A constante `TAX_RATE` (variável) é acessível dentro da função `calculateFinalPurchaseAmount(..)`, mesmo se não passarmos por ela, por conta do escopo léxico.

**Nota:** Para mais informações sobre o escopo léxico, veja os primeiros três capítulos dos títulos dessa série *Escopos & Clausuras*.

## Pratique

Não existe absolutamente nenhum substituto para prática ao aprender à programar. Nenhuma quantidade de artigos de minha parte, sozinhos, irão fazer você se tornar um programador.

Com isso em mente, vamos tentar praticar alguns conceitos que aprendemos neste capítulo. Eu irei lhe dar os "requerimentos", e você primeiro irá tentar executá-los. Depois, consulte o código listado abaixo para ver a forma que eu os resolvi.

* Escreva um programa que calcula o preço total da compra do seu celular. Você pode continuar comprando telefones (dica: loop!) até você ficar sem dinheiro na sua conta bancária. Você irá também comprar acessórios para cada telefone enquanto sua quantidade de dinheiro for menor do que seu limite mensal.
* Após calcular o valor da compra, adicione as taxas, depois imprima a quantidade total, devidamente formatada.
* Por fim, verifique o total gasto em sua conta bancária para saber se você pode comprar ou não.
* Você deve definir algumas constantes para a "taxa de imposto", "preço do telefone", "preço do acessório", e "limite de gastos", assim como variáveis para o seu "saldo bancário".
* Você deve definir funções para calcular a taxa e para formatar o preço com um "$" e arredondá-lo para duas casas decimais.
* **Desafio Extra:** Tente incorporar um input para esse programa, talvez com o`prompt(..)` que abordamos anteriormente em "Input". Você pode definir um prompt para o usuário para definir o saldo de sua conta bancária, por exemplo. Divirta-se e seja criativo!

Certo, vá em frente. Tente. Não venha ver o resultado do código que fiz enquanto você não tentar por conta própria!

**Nota:** Por este ser um livro de JavaScript, eu obviamente estarei resolvendo o exercício em JavaScript. Mas você pode fazer por enquanto em qualquer linguagem que você se sentir mais confortável.

Abaixo minha solução para esse exercício:

```js
const SPENDING_THRESHOLD = 200;
const TAX_RATE = 0.08;
const PHONE_PRICE = 99.99;
const ACCESSORY_PRICE = 9.99;

var bank_balance = 303.91;
var amount = 0;

function calculateTax(amount) {
	return amount * TAX_RATE;
}

function formatAmount(amount) {
	return "$" + amount.toFixed( 2 );
}

// continue comprando até não ter mais dinheiro
while (amount < bank_balance) {
	// compre um novo celular!
	amount = amount + PHONE_PRICE;

	// podemos comprar o acessório?
	if (amount < SPENDING_THRESHOLD) {
		amount = amount + ACCESSORY_PRICE;
	}
}

// não esqueça a fatia do governo!
amount = amount + calculateTax( amount );

console.log(
	"Sua compra: " + formatAmount( amount )
);
// Sua compra: $334.76

// Você pode pagar a conta?
if (amount > bank_balance) {
	console.log(
		"Você não pode pagar a conta. :("
	);
}
// Você não pode pagar a conta. :(
```

**Nota:** A maneira mais fácil de rodar esse programa em JavaScript é digitá-lo no console do seu navegador mais próximo.

Como você foi? Não dói se você tentar de novo agora que viu meu código. Brinque um pouco alterando algumas constantes e veja como o programa roda com valores diferentes.

## Recapitulando

Aprender a programar não precisa ser um processo complexo e cansativo. Existem apenas alguns conceitos que você precisa entender para as coisas começarem a fazer sentido.

Eles funcionam como blocos de montar. Para construir uma torre alta, você precisa primeiro colocar bloco em cima de bloco. O mesmo acontece com programação. Aqui encontram-se alguns blocos essenciais para programação:

* Você precisa de *operadores* para modificar valores.
* Você precisa de valores e *tipos* para realizar tipos diferentes de alterações como operações matemáticas em números (`number`s) ou em um output com `string`s.
* Você precisa de *variáveis* para armazenar informação (conhecido como *estado*) durante a execução de um programa.
* Você precisa de *condicionais* como `if` para tomar decisões.
* Você precisa de *loops* para repetir tarefas até uma condição deixar de ser verdadeira (`true`).
* Você precisa de *funções* (*function*) para organizar seu código de forma lógica e em componentes reutilizáveis.

Comentários no código são uma maneira efetiva de escrever um código mais legível, o que irá fazer seu programa fácil de entender, manter e consertar caso tenha algum problema no futuro.

Por fim, não esqueça o poder da prática. A melhor forma de aprender como escrever código é escrevendo código.

Estou empolgado que você está progredindo na sua caminhada para aprender a programar. Continue assim. Não esqueça de ver também outros recursos para começar a programar (blogs, outros livros, treinamento online, etc.) Este capítulo e este livro são um grande começo, mas são apenas uma breve introdução.

No próximo capítulo iremos revisar muitos dos conceitos deste capítulo, mas de uma perspectiva mais específica para o JavaScript, que irá destacar a maioria dos grandes tópicos que iremos abordar mais à fundo ao longo da série.

*Observação:*
**NT:** Nota do tradutor.
