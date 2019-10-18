# You Don't Know JS: Escopos & Clausuras
# Capítulo 3: Escopo de função vs. Bloco de escopo

Conforme exploramos no Capítulo 2, o escopo consiste em uma série de "bolhas" que atuam como recipientes nos quais identificadores (variáveis, funções) são declarados. Estas bolhas aninham-se umas às outras de forma muito organizada, e este aninhamento é definido durante a escrita do código.

Mas o que exatamente faz estas bolhas? Seriam apenas as funções? Ou teríamos outras estruturas em JavaScript para criar bolhas de escopo?

## Escopo a partir de funções

A resposta mais comum para estas perguntas é que JavaScript possui escopo baseado em funções. Isto é, cada função que você declara cria uma bolha para si, mas nenhuma outra estrutura cria suas próprias bolhas de escopo. Conforme veremos em breve, isso não é bem a verdade.

Mas primeiro vamos explorar o escopo de funções e suas consequências.

Considere este código:

```js
function foo(a) {
    var b = 2

    // algum código

    function bar() {
        // ...
    }

    // mais código

    var c = 3;
}
```

Neste trecho, a bolha de escopo de `foo(..)` inclui os identificadores `a`, `b`, `c` e `bar`. **Não importa** *onde* a declaração aparece dentro do escopo, a variável ou função pertence à bolha de escopo onde está inserida, independente de qualquer coisa. Vamos explorar no próximo capítulo como exatamente *isso* acontece.

`bar(..)` tem sua própria bolha de escopo. Assim como o escopo global, que possui apenas um identificador: `foo`.

Pelo fato de `a`, `b`, `c` e `bar` pertencerem à bolha de escopo de `foo(..)`, elas não estão acessíveis fora de `foo(..)`. Isto é, o código a seguir resultará em um erro `ReferenceError`, visto que os identificadores não ficam disponíveis para o escopo global:

```js
bar(); // vai falhar

console.log( a, b, c ); // as três falham
```

Porém, todos estes identificadores (`a`, `b`, `c`, `foo` e `bar`) podem ser acessados *dentro* de `foo(..)`, assim como também ficam disponíveis dentro de `bar(..)` (partindo do princípio que não foram sombreados por identificadores declarados dentro de `bar(..)`).

Escopo de função encoraja a ideia de que todas as variáveis pertencem à função e podem ser usadas e reutilizadas por toda sua extensão (e, de fato, acessíveis inclusive para escopos aninhados). Esta abordagem de projeto pode ser muito útil e certamente pode aproveitar por completo a natureza "dinâmica" das variáveis JavaScript ao receber valores de diferentes tipos quando necessário.

Por outro lado, se você não toma algumas medidas de precaução, variáveis que existem por toda a extensão de um determinado escopo podem se transformar em armadilhas inesperadas.

## Escondendo-se em pleno escopo

A forma tradicional de pensarmos sobre funções é que você as declara e então adiciona código dentro dela. Mas o pensamento inverso é igualmente poderoso e útil: pegue um trecho qualquer de código que você escreveu e envolva uma declaração de funcão ao seu redor, isso acaba por "esconder" este código.

O resultado prático é a criação de uma bolha de escopo ao redor do código em questão, o que significa que qualquer declaração (de variáveis ou funções) neste código estará atrelada ao escopo da nova função que a envolve em vez do escopo que a envolvia anteriormente. Em outras palavras, você pode "esconder" variáveis e funções ao envolvê-las no escopo de uma função.

Mas por que "ocultar" variáveis e funções seria uma técnica útil?

Existem uma série de fatores que motivam este ocultamento baseado em escopo. Eles tendem a surgir de um princípio de projeto de software chamado "Princípio do privilégio mínimo" [^note-leastprivilege], também chamado de (Princípio da) "Menor autoridade" ou "Exposição mínima". Este princípio define que, no projeto de determinado software, como por exemplo a API de um módulo/objeto, você deve expôr somente o mínimo necessário e "esconder" todo o resto.

Este princípio se estende à escolha de qual escopo deve conter determinada variável ou função. Se todas as variáveis e funções estivessem disponíveis no escopo global, elas certamente poderiam ser acessadas por qualquer escopo aninhado. Mas isto violaria o "Princípio" pelo fato de você (provavelmente) estar expondo muitas variáveis e funções que deveriam ser mantidas ocultas, visto que um uso mais adequado para este código poderia desencorajar a utilização destas variáveis/funções.

Por exemplo:

```js
function doSomething(a) {
    b = a + doSomethingElse( a * 2 );

    console.log( b * 3 );
}

function doSomethingElse(a) {
    return a - 1;
}

var b;

doSomething( 2 ); // 15
```

Neste trecho, a variável `b` e a função `doSomethingElse(..)` são detalhes "privados" de como `doSomething(..)` faz seu trabalho. Dar "acesso" à `b` e `doSomethingElse(..)` para o escopo superior não só é desnecessário mas também possivelmente "perigoso", visto que ambas podem ser eventualmente utilizadas de maneira inesperada, intencionalmente ou não, e isso pode violar suposições predeterminadas por `doSomething(..)`.

Um projeto mais "adequado" esconderia estes detalhes privados no escopo de `doSomething(..)`, como por exemplo:

```js
function doSomething(a) {
    function doSomethingElse(a) {
        return a - 1;
    }

    var b;

    b = a + doSomethingElse( a * 2 );

    console.log( (b * 3) );
}

doSomething( 2 ); // 15
```

Agora, `b` e `doSomethingElse(..)` não estão sujeitas a nenhuma influência externa, sendo controladas apenas por `doSomething(..)`. A funcionalidade e o resultado final não foram afetados, mas o projeto manteve oculto os detalhes privados, o que normalmente é considerado como um software de melhor qualidade.

### Evitando Colisões

Outro benefício de "esconder" as variáveis e funções dentro de um escopo é para evitar a colisão não intencional entre dois identificadores diferentes com o mesmo nome, mas diferentes usos pretendidos. Colisão resulta muitas vezes em substituição inesperada de valores.

Por exemplo:

```js
function foo() {
    function bar(a) {
        i = 3; // alterando o `i` no for-loop do escopo envolvido
        console.log( a + i );
    }

    for (var i=0; i<10; i++) {
        bar( i * 2 ); // oops, loop infinito em frente!
    }
}

foo();
```

A atribuição `i = 3` dentro de `bar(..)` substitui, de forma inesperada, o `i` que foi declarado em `foo(..)` no for-loop. Neste caso, ele irá resultar em um loop infinito, porque `i` é definido para um valor fixo de 3 e que permanecerá para sempre < 10.

A atribuição dentro de `bar(..)` precisa declarar uma variável local para usar, independentemente de qual nome de identificador é escolhido. `var i = 3;` iria resolver o problema (e criaria a declaração mencionada anteriormente "variável sombreada" para i). Uma opção adicional (e não alternativa) a opção é escolher um outro nome inteiramente diferente para o identificador, como `var j = 3`. Mas seu projeto de software pode, naturalmente, chamar para o mesmo nome de identificador, portanto, utilizando o escopo para "esconder" a sua declaração interior é a sua melhor/única opção nesse caso.

#### "Namespaces" globais

Um particularmente forte exemplo de (possível) colisão variável ocorre no escopo global. Várias bibliotecas carregadas em seu programa podem facilmente colidir umas com as outras se elas não esconderem adequadamente suas funções privadas/internas e variáveis.

Tais bibliotecas normalmente irão criar uma única declaração de variável, muitas vezes, um objeto, com um nome suficientemente original, no escopo global. Este objeto é então usado como um "namespace" para aquela biblioteca, onde todas as exposições específicas de funcionalidade são feitas como propriedades fora esse objeto (namespace), e não como identificadores lexicamente escopados em alto-nível por conta própria.

Por exemplo:

```js
var MyReallyCoolLibrary = {
    awesome: "stuff",
    doSomething: function() {
        // ...
    },
    doAnotherThing: function() {
        // ...
    }
};
```

#### Gestão de módulo

Outra opção para evitar colisões é a abordagem mais moderna de "módulo", usando qualquer dos vários gestores de dependência. Usando essas ferramentas, nenhuma biblioteca adicionará quaisquer identificadores ao escopo global, mas são obrigados a ter o(s) seu(s) identificador(es) sendo importado(s) explicitamente dentro de outro escopo específico através do uso de vários mecanismos do gerenciador de dependências.

Deve ser observado que estas ferramentas não possuem funcionalidades "mágicas" que estão isentas das regras do escopo léxico. Elas simplesmente usam as regras de escopo, como explicado aqui, para impor que nenhum identificador seja injetado em qualquer escopo compartilhado, e em vez disso são mantidos privados, escopos de não colisão suscetíveis (non-collision-susceptible scopes), o que previne qualquer colisão acidental no escopo.

Como tal, você pode codificar defensivamente e alcançar os mesmos resultados que os gestores de dependência fazem sem realmente precisar usá-los, se assim você desejar. Veja o Capítulo 5 para mais informações sobre module pattern.

## Funções como Escopo

Vimos que podemos pegar qualquer trecho de código e envolver uma função ao seu redor, e que, efetivamente, "esconde" as declarações inclusas de funções ou variáveis de fora do escopo principal, dentro do interior do escopo daquela função.

Por exemplo:

```js
var a = 2;

function foo() { // <-- insira isso

    var a = 3;
    console.log( a ); // 3

} // <-- e isso
foo(); // <-- e isso

console.log( a ); // 2
```

Embora esta técnica "funcione", ela não é necessariamente ideal. Existem alguns problemas que ela introduz. A primeira, é que nós temos que declarar um função nomeada `foo()`, o que significa que o nome identificado  `foo`  "polui" o escopo envolvido (global, neste caso). Nós também temos que chamar explicitamente a função pelo nome (`foo ()`) para que o código atualmente envolvido execute.

O ideal seria se a função não precisasse de um nome (ou melhor, o nome não poluir o escopo envolvido), e se a função automaticamente pudesse ser executada.

Felizmente, JavaScript oferece uma solução para ambos os problemas.

```js
var a = 2;

(function foo() { // <-- insira isso

    var a = 3;
    console.log( a ); // 3

})(); // <-- e isso

console.log( a ); // 2
```

Vamos desvendar o que está acontecendo aqui.

Primeiro, note que o envolvimento da instrução da função começa com `(function...` ao invés de apenas `function...`. Enquanto isto deve parecer como um pequeno detalhe, ele é na realidade uma grande mudança. Ao invés de tratar a função como uma declaração padrão, a função é tratada como uma função de expressão (function-expression).

**Nota:** A maneira mais fácil de distinguir declaração vs. expressão é a posição da palavra "function" na declaração (não apenas uma linha, mas uma declaração distinta). Se "function" é a primeira coisa na instrução, então é uma declaração de função. Caso contrário, é uma expressão de função.

A principal diferença, que podemos observar aqui, entre uma declaração de função e uma expressão de função se refere a onde seu nome está vinculado como um identificador.

Compare os dois trechos (de código) anteriores. No primeiro trecho, o nome `foo` é obrigatório (bound) no escopo envolvido, e nós o chamamos diretamente com `foo()`. No segundo trecho, o nome `foo` não está vinculado no escopo envolvido, mas em vez disso está vinculado somente dentro de sua própria função.

Em outras palavras, `(function foo(){..})` como uma expressão significa que o identificador `foo` é encontrado somente no escopo onde o `{ .. }`  é indicado, não no escopo externo. Escondendo o nome `foo` dentro dele mesmo significa que não polui o escopo envolvido desnecessariamente.

### Anônimo vs. Nomeado

Você provavelmente está mais familiarizado com expressões de função como parâmetros de retorno (callbacks parameters) de chamada, tais como:

```js
setTimeout( function(){
    console.log("Eu espero 1 segundo!");
}, 1000 );
```

Isso é chamado de “expressão de função anônima”, porque `function()...` não tem um nome no identificador dele. As expressões de função podem ser anônimas, mas declarações de função não podem omitir o nome - o que seria ilegal na gramática do JS.

Expressões de função anônimas são rápidas e fáceis de digitar, e muitas bibliotecas e ferramentas tendem a encorajar este estilo idiomático de código. Entretanto, elas têm vários inconvenientes a serem considerados:

1. Funções anônimas não têm nome útil para exibir em stack traces, que podem fazer a depuração (debugging) mais difícil.

2. Sem um nome, se a função precisa se referir a si mesma, por recursão, etc., o **desaconselhado** `arguments.callee` referenciado é infelizmente necessário. Outro exemplo da necessidade de auto referência é quando uma função de manipulador de eventos (event handler function) quer desvincular-se após a execução.

3. Funções anônimas omitem um nome que muitas vezes é útil para fornecer um código mais legível/compreensível. Um nome descritivo ajuda a auto documentação do código em questão.

**Expressões de função em linha (Inline function expressions)** são poderosas e úteis - a questão de anônimo vs. nomeado não deprecia a partir disso. Fornecer um nome para a sua expressão de função de forma bastante eficaz aborda todos estes inconvenientes, mas não tem desvantagens tangíveis. A melhor prática é sempre nomear suas expressões de função:

```js
setTimeout( function timeoutHandler(){ // <-- Olha, eu tenho um nome!
    console.log( "Eu esperei um segundo!" );
}, 1000 );
```

### Invocando Expressões de função imediatamente

```js
var a = 2;

(function foo(){

    var a = 3;
    console.log( a ); // 3

})();

console.log( a ); // 2
```

Agora que temos uma função como uma expressão em virtude de envolvê-la em um par de `()`, podemos executar aquela função adicionando outro `()` no fim, como `(foo function(){..})()`. O primeiro par de inclusão `()` faz a expressão de uma função, e a segunda `()` executa a função.

Este padrão é tão comum, que há alguns anos, a comunidade aceitou um termo para isso: IIFE, que representa uma Expressão de Função Imediatamente Invocada (Immediately Invoked Function Expressions).

Claro, IIFE’s não precisam de nomes, necessariamente - a forma mais comum de uma IIFE é usar uma expressão da função anônima. Embora certamente menos comum, nomeando um IIFE tem todas as vantagens acima mencionadas sobre as expressões de função anônima, por isso é uma boa prática para adotar.

```js
var a = 2;

(function IIFE(){

    var a = 3;
    console.log( a ); // 3

})();

console.log( a ); // 2
```

Há uma ligeira variação na tradicional forma do IIFE, que alguns preferem: `(function(){..}())`. Olhe atentamente para ver a diferença. Na primeira forma, a expressão da função é envolvida por `()`, e então invocada por um par de `()` que está à direita do lado de fora após ele. Na segunda forma, o invocador par de `()` é movido para o interior do exterior par de `()` envolvido.

Estas duas formas são idênticas em termos de funcionalidade. **É puramente uma escolha estilística que você preferir.**

Outra variação sobre IIFE o qual é bastante comum, é utilizar o fato de que eles são, de fato, apenas funções de chamada e passagem para o(s) argumento(s).

Por Exemplo:

```js
var a = 2;

(function IIFE( global ){

    var a = 3;
    console.log( a ); // 3
    console.log( global.a ); // 2

})( window );

console.log( a ); // 2
```

Passamos na referência do objeto `window`, mas o nomeamos parâmetro `global`, de modo que temos um esboço estilístico claro para referências global vs. não-global. Claro, você pode passar qualquer coisa que você quiser no escopo envolvido, e você pode nomear o(s) parâmetro(s) de qualquer forma que convir. Isto é escolha apenas estilística.

Outra aplicação desse padrão aborda (nicho menor) uma preocupação que o identificador padrão `undefined` pode ter seu valor incorretamente substituído, causando resultados inesperados. Ao nomear um parâmetro `undefined`, mas não passando qualquer valor para aquele argumento, podemos garantir que o identificador `undefined` é de fato um valor indefinido em um bloco de código:

```js
undefined = true; // configurando minas terrestres para outro código! Evite!

(function IIFE( undefined ){

    var a;
    if (a === undefined) {
        console.log( "Undefined está salvo aqui!" );
    }

})();
```

Ainda que outra variação do IIFE inverta a ordem das coisas, onde a função a ser executada é dada em segundo lugar, *após* a invocação e os parâmetros passados a ele. Este padrão é utilizado no projeto UMD (Módulo de Definição Universal). Algumas pessoas acham que é um pouco mais limpo para entender, embora seja um pouco mais detalhado.

```js
var a = 2;

(function IIFE( def ){
    def( window );
})(function def( global ){

    var a = 3;
    console.log( a ); // 3
    console.log( global.a ); // 2

});
```

A expressão de função `def` é definida na segunda metade do trecho (de código), e depois passado como um parâmetro (também chamado `def`) para a função `IIFE` definida na primeira metade do trecho (de código). Finalmente, o parâmetro `def` (a função) é invocado, passando `window` como o parâmetro `global`.

## Blocos como Escopos

Enquanto as funções são as unidades mais comuns do escopo, e certamente a mais generalizada das abordagens de design na maioria dos JS em circulação, outras unidades de escopo são possíveis, e o uso dessas outras unidades de escopo podem conduzir ainda melhor, a manter um código limpo.

Muitas outras linguagens além do JavaScript suportam Blocos de escopo, e os desenvolvedores dessas linguagens estão acostumados com essa mentalidade, enquanto que aqueles que principalmente só trabalharam com JavaScript devem achar o conceito um pouco estranho.

Mas mesmo se você nunca escreveu uma única linha de código na forma de escopo do bloco, você ainda provavelmente está familiarizado com esta linguagem extremamente comum em JavaScript:

```js
for (var i=0; i<10; i++) {
    console.log( i );
}
```

Nós declaramos a variável `i` diretamente dentro do cabeçalho do for-loop, muito provavelmente porque a nossa *intenção* é usar `i` somente no contexto daquele for-loop, e essencialmente, ignorar o fato daquela variável no atual escopo para o escopo envolvido (função ou global).

Isso é tudo sobre escopo do bloco. Declarar variáveis tão próximas quanto possível, tão local quanto possível, para onde elas serão utilizadas. Outro exemplo:

```js
var foo = true;

if (foo) {
    var bar = foo * 2;
    bar = something( bar );
    console.log( bar );
}
```

Nós estamos usando uma variável `bar` apenas no contexto da declaração if, por isso, faz sentido nós declararmos dentro do bloco if. No entanto, onde nós declaramos variáveis não é relevante quando se usa `var`, porque elas vão sempre pertencer ao escopo envolvido. Este trecho é essencialmente um escopo do bloco "falso", por razões estilísticas, e contando com a auto execução para não acidentalmente usar `bar` em outro lugar naquele escopo.

Escopo de bloco é uma ferramenta para estender o anterior "Princípio da menor Exposição ~~privilegiada~~" [^ note-leastprivilege] de esconder informações em funções para esconder informações em blocos de nosso código.

Considere o exemplo for-loop novamente:

```js
for (var i=0; i<10; i++) {
    console.log( i );
}
```

Por que poluir todo o escopo de uma função com a variável `i` que só vai ser (ou só *deveria ser*, pelo menos) utilizado para o for-loop?

Mas o mais importante, os desenvolvedores devem preferir *se prevenir* contra a acidental (re)utilização de variáveis fora da sua finalidade, tais como sendo emitido um erro sobre uma variável desconhecida, se você tentar usá-la no lugar errado. Escopo do bloco (se fosse possível) para a variável `i` faria `i` disponível apenas para o for-loop, causando um erro se `i` é acessado em outros lugares na função. Isso ajuda a garantir as variáveis não serem reutilizados de maneiras confusas ou difíceis de manter.

Mas, a triste realidade é que, na superfície, JavaScript não tem facilidades para o escopo do bloco.

Isto é, até você cavar um pouco mais.

### `with`

Aprendemos sobre `with` no Capítulo 2. Embora seja uma desaprovada construção, *é* um exemplo de (uma forma de) escopo do bloco, em que o escopo que é criado a partir do objeto só existe para o tempo de vida da declaração `with`, e não no escopo envolvido.

### `try/catch`

É um fato *muito* pouco conhecido que o JavaScript no ES3 especificou a declaração de variável na cláusula `catch` de um `try / catch` para ser escopo do bloco para o bloco `catch`.

Por exemplo:

```js
try {
    undefined(); // Operação ilegal para forçar uma exceção!
}
catch (err) {
    console.log( err ); // Funciona!
}

console.log( err ); // ReferenceError: `err` not found
```

Como você pode ver, `err` só existe na cláusula `catch`, e lança um erro quando você tenta referenciá-lo em outro lugar.

**Nota:** Enquanto este comportamento foi especificado e verdadeiro em praticamente todo o ambiente JS padrão (exceto, talvez no velho IE), muitos linters parecem ainda reclamar, se você tiver duas ou mais cláusulas `catch` no mesmo escopo, e cada qual declarar sua variável de erro com o mesmo nome do identificador. Esta não é realmente uma redefinição, uma vez que as variáveis estão salvas no escopo do bloco, mas os linters ainda parecem, irritantemente, queixar-se deste fato.

Para evitar esses avisos desnecessários, alguns devs nomearão suas variáveis `catch` `err1`, `err2`, etc. Outros desenvolvedores simplesmente desligarão a verificação linting para nomes de variáveis duplicados.

A natureza escopo do bloco de `catch` deve parecer como fato acadêmico inútil, mas veja o Apêndice B para mais informações sobre o quão útil pode ser.

### `let`

Até agora, vimos que o JavaScript só tem alguns comportamentos estranhos de nicho que expõem a funcionalidade do escopo do bloco. Se fosse tudo o que tínhamos, *e foi* por muitos e muitos anos, então escopo do bloco não seria terrivelmente útil para o desenvolvedor de JavaScript.

Felizmente, ES6 muda isso, e introduziu uma nova palavra-chave `let` que fica ao lado de `var` como outra forma de declarar variáveis.

A palavra-chave `let` atribui uma declaração da variável ao escopo de qualquer bloco (normalmente um par `{ .. }`) em que está contido. Em outras palavras, `let` implicitamente sequestra qualquer escopo do bloco para a sua declaração de variável.

```js
var foo = true;

if (foo) {
    let bar = foo * 2;
    bar = something( bar );
    console.log( bar );
}

console.log( bar ); // ReferenceError
```

Usando `let` para anexar uma variável para um bloco existente é algo implícito. Ele pode confundir se você não está prestando atenção em quais blocos tem variáveis de escopo para eles, e têm o hábito de mover blocos ao redor, envolvendo-os em outros blocos, etc., da forma que você desenvolver e evoluir código.

Criando blocos explícitos para os blocos de escopo, podemos chamar a atenção para algumas dessas preocupações, tornando-a mais óbvias onde as variáveis estão, e não estão vinculadas. Normalmente, código explícito é preferível ao código implícito ou sutil. Este estilo escopo do bloco explícito é fácil de efetuar, e se encaixa mais naturalmente com a forma como o escopo do bloco funciona em outras linguagens:

```js
var foo = true;

if (foo) {
    { // <-- bloco explícito
        let bar = foo * 2;
        bar = something( bar );
        console.log( bar );
    }
}

console.log( bar ); // ReferenceError
```

Podemos criar um bloco arbitrário para `let` vincular-se simplesmente incluindo um par de `{ .. }` em qualquer lugar que uma declaração é gramaticalmente válida. Neste caso, nós fizemos um bloco explícito *dentro* da instrução if, o que deve ser mais fácil como um bloco inteiro para se mover mais tarde na refatoração, sem afetar a posição e semântica da instrução if envolvida.

**Nota:** Para uma outra maneira de expressar blocos de escopo explícitos, consulte o Apêndice B.

No capítulo 4, trataremos de hoisting, que fala sobre as declarações sendo tomadas como existentes para todo o escopo em que ocorrem.

No entanto, declarações feitas com `let` *não* irão elevar para todo o escopo do bloco que eles são apresentados. Tais declarações não observáveis "existem" no bloco até a instrução de declaração.

```js
{
    console.log( bar ); // ReferenceError!
    let bar = 2;
}
```

#### Coleta de lixo (Garbage Collection)

Outra razão do escopo do bloco ser útil, relaciona se com closures e coletas de lixo para recuperar a memória. Vamos brevemente ilustrar aqui, mas o mecanismo de closures é explicado em detalhes no Capítulo 5.

Considere:

```js
function process(data) {
    // faz alguma coisa interessante
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
    console.log("button clicked");
}, /*capturandoFrase=*/false );
```

O `click`, função manipuladora de clique de retorno de chamada, não *precisa* da variável `someReallyBigData` em tudo. Isso significa que, teoricamente, após o `process(..)` executar, a grande estrutura de dados de memória pesado poderia ser lixo coletado. No entanto, é bastante provável (embora dependente de implementação) que o motor JS ainda terá de manter a estrutura em volta, uma vez que a função `click` tem uma clausura sobre todo o escopo.

Escopo do bloco pode resolver este problema, tornando-o mais claro para o motor que ele não precisa manter `someReallyBigData` em volta:

```js
function process(data) {
    // faz alguma coisa interessante
}

// Qualquer coisa declarada dentro de bloco pode ir embora depois!
{
    let someReallyBigData = { .. };

    process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
    console.log("button clicked");
}, /*FaseDeCaptura=*/false );
```

Declarando blocos explícitos para as variáveis localmente se associarem, é uma ferramenta poderosa que você pode adicionar à sua caixa de ferramentas de código.

#### `let` Loops

Um caso particular em que `let` brilha é no caso for-loop como discutimos anteriormente.

```js
for (let i=0; i<10; i++) {
    console.log( i );
}

console.log( i ); // ReferenceError
```

Não só `let` no cabeçalho for-loop associa o `i` para o corpo for-loop, mas de fato, ele **reassocia** a cada *iteração* do loop, certificando-se de reatribuir-lhe o valor do final da iteração anterior do loop.

Aqui está outra maneira de ilustrar o comportamento de ligação por-iteração que ocorre:

```js
{
    let j;
    for (j=0; j<10; j++) {
        let i = j; // re-associa para cada interação!
        console.log( i );
    }
}
```

A razão pela qual a ligação por-interação é interessante se tornará claro no capítulo 5, quando discutirmos closures.

Porque declarações `let` anexam à blocos arbitrários em vez de ao escopo da função envolvida (ou global), pode haver armadilhas em que o código existente tem uma dependência escondida na declaração do escopo da função `var`, e substituindo a `var` com `let` deve exigir cuidados adicionais quando re-fatorando código.

Considere:

```js
var foo = true, baz = 10;

if (foo) {
    var bar = 3;

    if (baz > bar) {
        console.log( baz );
    }

    // ...
}
```

Este código é facilmente re-fatorado como:

```js
var foo = true, baz = 10;

if (foo) {
    var bar = 3;

    // ...
}

if (baz > bar) {
    console.log( baz );
}
```

Mas, seja cuidadoso com tais mudanças ao usar variáveis com escopo do bloco:

```js
var foo = true, baz = 10;

if (foo) {
    let bar = 3;

    if (baz > bar) { // <-- Não esqueça `bar` quando mover!
        console.log( baz );
    }
}
```

Veja o Apêndice B para uma alternativa de estilo (mais explícito) do escopo do bloco que pode proporcionar manter/re-fatorar, mais facilmente, o código que é mais robusto para esses cenários.

### `const`

Além de `let`, ES6 introduz `const`, que também cria uma variável com escopo do bloco, mas cujo valor é fixo (constante). Qualquer tentativa de alterar esse valor em um momento posterior resultará em um erro.

```js
var foo = true;

if (foo) {
    var a = 2;
    const b = 3; // escopo do bloco que contém `if`

    a = 3; // Tudo bem!
    b = 4; // error!
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```

## Revisão (TL;DR)

As funções são as unidades mais comuns de escopo em JavaScript. Variáveis e funções que são declaradas dentro de outra função são essencialmente "escondidas" de qualquer um dos "escopos" envolvidos, que é um princípio de design intencional de um bom software.

Mas as funções são de nenhuma maneira a única unidade do escopo. Escopo do bloco refere-se à ideia de que as variáveis e funções podem pertencer a um bloco arbitrário (geralmente, qualquer par de `{ .. }`) de código, em vez de apenas para a função envolvida.

Começando com ES3, a estrutura `try/catch` tem escopo do bloco na cláusula `catch`.

Em ES6, é introduzida a palavra-chave `let` (um primo para a palavra-chave `var`) para permitir que as declarações de variáveis em qualquer bloco arbitrário de código. `if (..) {let a = 2; }` Irá declarar uma variável `a` que, essencialmente, sequestrará o escopo do bloco `if`'s `{ .. }` e auto atribuíra-se lá.

Embora alguns parecem acreditar, o escopo do bloco não deveria ser tomado como uma substituição pura e simples de escopo da função `var`. Ambas as funcionalidades co-existem, e os desenvolvedores podem e deveriam usar ambas as técnicas de escopo da função e escopo do bloco, onde respectivamente apropriados para produzir melhor, mais código legível/sustentável.

[^note-leastprivilege]: [Principle of Least Privilege](http://en.wikipedia.org/wiki/Principle_of_least_privilege)
