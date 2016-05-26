# You Don't Know JS: Escopos e Encerramentos
# Capítulo 1: O que é Escopo?

Um dos paradigmas mais fundamentais de quase todas as linguagens de programação é a capacidade de armazenar valores em variáveis e, posteriormente, obter ou modificar esses valores. Na verdade, a habilidade de armazenar e pegar valores de variáveis é o que fornece *estado* a um programa.

Sem esse conceito, um programa pode realizar algumas tarefas, mas elas seriam extremamente limitadas e extremamente desinteressantes.

Mas a inclusão de variáveis em nossos programas gera perguntas mais interessantes como: onde essas variáveis *vivem*? Em outras palavras, onde elas são armazenadas? E, mais importante, como nossos programas as encontram quando eles precisam delas?

Estas perguntas mostram a necessidade de um conjunto bem definido de regras para armazenar variáveis em algum lugar, e para encontrar essas variáveis posteriormente. Nós iremos chamar esse conjunto de regras: *Escopo*.

Mas, onde e como essas regras de *Escopo* são definidas?

## Teoria de Compiladores

Talvez seja evidente, ou pode ser que seja uma novidade, dependendo do seu nível de interação com linguagens diversas, mas apesar do fato de Javascript ser geralmente colocada na categoria de linguagens "dinâmicas" ou "interpretadas", ela é de fato uma linguagem compilada. Ela *não* é compilada com muita antecedência, como são muitas outras linguagens tradicionalmente compiladas, e nem os resultados da compilação são portáteis entre vários sistemas distribuídos.

Mas, mesmo assim, o mecanismo do Javascript realiza muitos passos semelhantes, embora de maneiras mais sofisticadas do que nós estamos acostumados a ver na maioria dos compiladores tradicionais.

Em um processo tradicional de uma linguagem compilada, um pedaço de código fonte, seu programa, vai tipicamente passar por três passos *antes* de ser executado, grosseiramente chamado "compilação":

1. **Tokenizing/Lexing:** quebrar uma string de caracteres em pedaços com algum significado (para a linguagem), chamados tokens. Por exemplo, considere o programa: `var a = 2;`. Esse programa provavelmente seria ser quebrado nos seguintes tokens: `var`, `a`, `=`, `2`, e `;`. Espaço em branco pode ou não ser mantido como um token, dependendo se tem ou não algum significado.

    **Nota:** A diferença entre tokenizing e lexing é sutil e teórica, mas centraliza-se no fato desses tokens serem ou não identificados de uma maneira *stateless* ou *stateful*. Colocando de maneira simples, se o tokenizer fosse invocar regras de análise stateful para saber se `a` deve ser considerado um token distinto ou apenas parte de outro token, *isso* seria **lexing**.

2. **Parsing:** pegar um conjunto (array) de tokens e transformar isso numa árvore de elementos aninhados, que juntos representam a estrutura gramática do programa. Essa árvore é conhecida como "AST" (<b>A</b>bstract <b>S</b>yntax <b>T</b>ree, que, em tradução livre, significa: Árvore Sintática Abstrata).

    A árvore para `var a = 2;` pode começar com um nó de nível superior chamado `VariableDeclaration`, que tem um nó filho chamado `Identifier` (cujo valor é `a`), e outro nó filho chamado `AssignmentExpression` que por sua vez tem um filho chamado `NumericLiteral` (cujo valor é `2`).

3. **Geração de código:** o processo de obter uma AST e transformar isso em código executável. Esta parte varia muito dependendo da linguagem, da plataforma-alvo, etc.

    Então, em vez de focar em detalhes, nós vamos apenas olhar superficialmente e dizer que existe uma forma de obter nossa AST descrita acima para `var a = 2;` e transformá-la em instruções de máquina para de fato *criar* uma variável chamada `a` (incluindo a reserva de memória, etc), e então armazenar um valor em `a`.

    **Nota:** Os detalhes de como o mecanismo administra recursos do sistema estão além do que iremos cobrir, então nós vamos apenas considerar que esse mecanismo é capaz de criar e armazenar variáveis conforme necessário.

O mecanismo de Javascript é vastamente mais complexo do que *apenas* aqueles três passos, da mesma maneira do que outros compiladores de linguagem. Por exemplo, no processo de análise e geração de código, há com certeza passos para otimizar o desempenho da execução, incluindo tratar elementos redundantes, etc.

Sendo assim, eu estou mostrando de forma bem grosseira aqui. Mas eu acho que vocês verão rapidamente porque *esses* detalhes que nós *cobrimos*, mesmo que superficialmente, são relevantes.

Por um lado, o mecanismo de Javascript não tem o luxo (como compiladores de outras linguagens) de ter uma grande disponibilidade de tempo para otimização, porque a compilação de Javascript não acontece numa etapa de preparação anterior, como em outras linguagens.

Para Javascript, a compilação que ocorre acontece, em muitos casos, somente alguns microsegundos (ou menos!) antes do código ser executado. Para garantir o mais alto desempenho, o mecanismo JS utiliza todos os tipos de truques (como JITs, que compilam de maneira preguiçosa e até mesmo recompilam rapidamente, etc.) que são além do escopo da nossa discussão aqui.

Vamos apenas dizer, para fins de simplicidade, que qualquer pedaço de Javascript tem que ser compilado antes (geralmente *logo antes* como dito anteriormente!) de ser executado. Sendo assim, o compilador JS vai obter o programa `var a = 2;` e compilá-lo *antes*, e então estará pronto para executá-lo, geralmente de imediato.

## Entendendo Escopo

A forma como nós vamos abordar o aprendizado sobre escopo é imaginar o processo como uma conversa. Mas, *quem* está tendo essa conversa?

### O Elenco

Vamos conhecer o elenco dos personagens que interagem para processar o programa `var a = 2;`, assim entenderemos a conversa que vamos ouvir em breve:

1. *Mecanismo*: responsável pela compilação do começo ao fim e pela execução do nosso programa Javascript.

2. *Compilador*: um dos amigos do *Mecanismo*; gerencia todo o trabalho sujo da análise e da geração de código (veja a seção anterior).

3. *Escopo*: outro amigo do *Mecanismo*; coleta e mantém uma lista de consultas a todos os identificadores declarados (variáveis), e impõe um rigoroso conjunto de regras sobre a maneira como estes identificadores são acessíveis pelo código que está em execução.

Para o seu *completo entendimento* sobre como Javascript funciona, você precisa começar a *pensar* como o *Mecanismo* (e seus amigos) pensa, fazer as perguntas que eles fazem, e responder essas perguntas.

### Para a frente e para trás

Quando você vê o programa `var a = 2;`, você provavelmente pensa nele como uma instrução. Mas não é como nosso novo amigo *Mecanismo* o vê. Na verdade, o *Mecanismo* vê duas instruções distintas, uma que o *Compilador* vai gerenciar durante a compilação, e uma que o *Mecanismo* vai gerenciar durante a execução.

Então, vamos expandir em como o *Mecanismo* e seus amigos vão abordar o programa `var a = 2;`

A primeira coisa que o *Compilador* vai fazer com este programa é realizar uma análise léxica para quebrá-los em tokens, que ele vai transformar numa árvore. Mas quando o *Compilador* chega à geração de código, ele vai tratar este programa um pouco diferente do que você talvez tenha suposto.

Uma suposição provável seria o *Compilador* produzir códigos que poderiam ser resumidos pelo pseudo-código: "Reserve memória para uma variável, rotule-a como `a`, então ponha o valor `2` nessa variável". Mas infelizmente, isso não é tão preciso.

Em vez disso, o *Compilador* vai proceder como:

1. Encontrando `var a`, o *Compilador* pede ao *Escopo* para ver se alguma variável `a` já existe para o conjunto particular deste escopo. Se já existe, o *Compilador* ignora essa declaração e segue em frente. Caso contrário, o *Compilador* pede ao *Escopo* para declarar uma nova variável chamada `a` para o conjunto deste escopo.

2. o *Compilador* então produz código para que o *Mecanismo* execute mais tarde, para gerenciar a atribuição `a = 2`. O código que o *Mecanismo* executa vai primeiro perguntar ao *Escopo* se existe uma variável chamada `a` acessível no conjunto do escopo atual. Se já existe, o *Mecanismo* usa esta variável. Se não existe, o *Mecanismo* procura *em outro lugar* (veja abaixo a seção *Escopos* aninhados).

Se o *Mecanismo* eventualmente encontra uma variável, ele atribui o valor `2` a ela. Se não, o *Mecanismo* vai levantar a mão e gritar que há um erro!

Para resumir: duas ações distintas são tomadas para uma atribuição a uma variável: Primeiro, o *Compilador* declara uma variável (se não foi anteriormente declarada no escopo atual), e segundo, quando executa, o *Mecanismo* busca a variável no *Escopo* e atribui a ela, se encontrada.

### O compilador fala

Nós precisamos de um pouco mais de termilonogias do compilador para seguir com a compreensão.

Quando o *Mecanismo* executa o código que o *Compilador* produziu no passo (2), ele tem que buscar pela variável `a` para saber se ela foi declarada, e essa busca é consultando o *Escopo*. Mas o tipo de busca que o *Mecanismo* efetua afeta o resultado da mesma.

No nosso caso, é dito que o *Mecanismo* faria uma busca "LHS" para a variável `a`. O outro tipo de busca é chamado de "RHS".

Eu aposto que você pode adivinhar o que o "L" e o "R" significam. Esses termos significam "Left-hand Side" (que em tradução livre, significa: "Lado esquerdo") e "Right-hand Side" (que em tradução livre, significa: "Lado direito").

Lado... de que? **Da operação de atribuição.**

Em outras palavras, uma busca LHS é feita quando uma variável aparece do lado esquerdo da operação de atribuição, e uma busca RHS é feita quando uma variável aparece do lado direito de uma operação de atribuição.

Na verdade, vamos ser um pouco mais precisos. Uma busca RHS é imperceptível, para nossos propósitos, simplesmente uma busca do valor de alguma variável, enquanto que, a busca LHS está tentando encontrar o recipiente por si só, então pode atribuir. Dessa maneira, RHS não significa *exatamente* "lado direito de uma atribuição", isso somente, de forma mais precisa, significa "não estar do lado esquerdo".

Sendo ligeiramente mais simplista por um momento, você poderia também imaginar que "RHS", em vez de "obtenha sua fonte (valor)", significa "vai buscar o valor de...".

Vamos um pouco mais fundo nisso.

Quando eu digo:

```js
console.log( a );
```

A referência para `a` é uma referência RHS porque nada está sendo atribuído a `a` aqui. Em vez disso, nós estamos buscando capturar o valor de `a`, para que o valor possa ser passado para `console.log(..)`.

Em compensação:

```js
a = 2;
```

A referência para `a` aqui é uma referência LHS, porque na verdade não ligamos para qual seja o valor atual, nós simplesmente queremos encontrar a variável como um alvo para a operação de atribuição `= 2`.

**Nota:** o significado "lado esquerdo/direito de uma atribuição" em LHS e RHS não necessariamente quer dizer "lado esquerdo/direito do operador de atribuição `=`". Existem muitas outras maneiras dessas atribuições acontecerem, então é melhor pensar conceitualmente sobre isso como: "quem é o alvo da atribuição (LHS) e quem é a fonte da atribuição (RHS)".

Considere este programa, que tem ambas as referências LHS e RHS:

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```
A última linha que invoca `foo(..)` como uma chamada de função requer uma referências RHS para `foo`, significando, "vá buscar o valor de `foo` e entregue para mim". Além disso, `(..)` significa que o valor de `foo` deve ser executado, então é melhor que seja realmente uma função!

There's a subtle but important assignment here. **Did you spot it?**

You may have missed the implied `a = 2` in this code snippet. It happens when the value `2` is passed as an argument to the `foo(..)` function, in which case the `2` value is **assigned** to the parameter `a`. To (implicitly) assign to parameter `a`, an LHS look-up is performed.

There's also an RHS reference for the value of `a`, and that resulting value is passed to `console.log(..)`. `console.log(..)` needs a reference to execute. It's an RHS look-up for the `console` object, then a property-resolution occurs to see if it has a method called `log`.

Finally, we can conceptualize that there's an LHS/RHS exchange of passing the value `2` (by way of variable `a`'s RHS look-up) into `log(..)`. Inside of the native implementation of `log(..)`, we can assume it has parameters, the first of which (perhaps called `arg1`) has an LHS reference look-up, before assigning `2` to it.

**Note:** You might be tempted to conceptualize the function declaration `function foo(a) {...` as a normal variable declaration and assignment, such as `var foo` and `foo = function(a){...`. In so doing, it would be tempting to think of this function declaration as involving an LHS look-up.

However, the subtle but important difference is that *Compiler* handles both the declaration and the value definition during code-generation, such that when *Engine* is executing code, there's no processing necessary to "assign" a function value to `foo`. Thus, it's not really appropriate to think of a function declaration as an LHS look-up assignment in the way we're discussing them here.

### Engine/Scope Conversation

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

Let's imagine the above exchange (which processes this code snippet) as a conversation. The conversation would go a little something like this:

> ***Engine***: Hey *Scope*, I have an RHS reference for `foo`. Ever heard of it?

> ***Scope***: Why yes, I have. *Compiler* declared it just a second ago. He's a function. Here you go.

> ***Engine***: Great, thanks! OK, I'm executing `foo`.

> ***Engine***: Hey, *Scope*, I've got an LHS reference for `a`, ever heard of it?

> ***Scope***: Why yes, I have. *Compiler* declared it as a formal parameter to `foo` just recently. Here you go.

> ***Engine***: Helpful as always, *Scope*. Thanks again. Now, time to assign `2` to `a`.

> ***Engine***: Hey, *Scope*, sorry to bother you again. I need an RHS look-up for `console`. Ever heard of it?

> ***Scope***: No problem, *Engine*, this is what I do all day. Yes, I've got `console`. He's built-in. Here ya go.

> ***Engine***: Perfect. Looking up `log(..)`. OK, great, it's a function.

> ***Engine***: Yo, *Scope*. Can you help me out with an RHS reference to `a`. I think I remember it, but just want to double-check.

> ***Scope***: You're right, *Engine*. Same guy, hasn't changed. Here ya go.

> ***Engine***: Cool. Passing the value of `a`, which is `2`, into `log(..)`.

> ...

### Quiz

Check your understanding so far. Make sure to play the part of *Engine* and have a "conversation" with the *Scope*:

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).

2. Identify all the RHS look-ups (there are 4!).

**Note:** See the chapter review for the quiz answers!

## Nested Scope

We said that *Scope* is a set of rules for looking up variables by their identifier name. There's usually more than one *Scope* to consider, however.

Just as a block or function is nested inside another block or function, scopes are nested inside other scopes. So, if a variable cannot be found in the immediate scope, *Engine* consults the next outer containing scope, continuing until found or until the outermost (aka, global) scope has been reached.

Consider:

```js
function foo(a) {
	console.log( a + b );
}

var b = 2;

foo( 2 ); // 4
```

The RHS reference for `b` cannot be resolved inside the function `foo`, but it can be resolved in the *Scope* surrounding it (in this case, the global).

So, revisiting the conversations between *Engine* and *Scope*, we'd overhear:

> ***Engine***: "Hey, *Scope* of `foo`, ever heard of `b`? Got an RHS reference for it."

> ***Scope***: "Nope, never heard of it. Go fish."

> ***Engine***: "Hey, *Scope* outside of `foo`, oh you're the global *Scope*, ok cool. Ever heard of `b`? Got an RHS reference for it."

> ***Scope***: "Yep, sure have. Here ya go."

The simple rules for traversing nested *Scope*: *Engine* starts at the currently executing *Scope*, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or not.

### Building on Metaphors

To visualize the process of nested *Scope* resolution, I want you to think of this tall building.

<img src="fig1.png" width="250">

The building represents our program's nested *Scope* rule set. The first floor of the building represents your currently executing *Scope*, wherever you are. The top level of the building is the global *Scope*.

You resolve LHS and RHS references by looking on your current floor, and if you don't find it, taking the elevator to the next floor, looking there, then the next, and so on. Once you get to the top floor (the global *Scope*), you either find what you're looking for, or you don't. But you have to stop regardless.

## Errors

Why does it matter whether we call it LHS or RHS?

Because these two types of look-ups behave differently in the circumstance where the variable has not yet been declared (is not found in any consulted *Scope*).

Consider:

```js
function foo(a) {
	console.log( a + b );
	b = a;
}

foo( 2 );
```

When the RHS look-up occurs for `b` the first time, it will not be found. This is said to be an "undeclared" variable, because it is not found in the scope.

If an RHS look-up fails to ever find a variable, anywhere in the nested *Scope*s, this results in a `ReferenceError` being thrown by the *Engine*. It's important to note that the error is of the type `ReferenceError`.

By contrast, if the *Engine* is performing an LHS look-up and arrives at the top floor (global *Scope*) without finding it, and if the program is not running in "Strict Mode" [^note-strictmode], then the global *Scope* will create a new variable of that name **in the global scope**, and hand it back to *Engine*.

*"No, there wasn't one before, but I was helpful and created one for you."*

"Strict Mode" [^note-strictmode], which was added in ES5, has a number of different behaviors from normal/relaxed/lazy mode. One such behavior is that it disallows the automatic/implicit global variable creation. In that case, there would be no global *Scope*'d variable to hand back from an LHS look-up, and *Engine* would throw a `ReferenceError` similarly to the RHS case.

Now, if a variable is found for an RHS look-up, but you try to do something with its value that is impossible, such as trying to execute-as-function a non-function value, or reference a property on a `null` or `undefined` value, then *Engine* throws a different kind of error, called a `TypeError`.

`ReferenceError` is *Scope* resolution-failure related, whereas `TypeError` implies that *Scope* resolution was successful, but that there was an illegal/impossible action attempted against the result.

## Review (TL;DR)

Scope is the set of rules that determines where and how a variable (identifier) can be looked-up. This look-up may be for the purposes of assigning to the variable, which is an LHS (left-hand-side) reference, or it may be for the purposes of retrieving its value, which is an RHS (right-hand-side) reference.

LHS references result from assignment operations. *Scope*-related assignments can occur either with the `=` operator or by passing arguments to (assign to) function parameters.

The JavaScript *Engine* first compiles code before it executes, and in so doing, it splits up statements like `var a = 2;` into two separate steps:

1. First, `var a` to declare it in that *Scope*. This is performed at the beginning, before code execution.

2. Later, `a = 2` to look up the variable (LHS reference) and assign to it if found.

Both LHS and RHS reference look-ups start at the currently executing *Scope*, and if need be (that is, they don't find what they're looking for there), they work their way up the nested *Scope*, one scope (floor) at a time, looking for the identifier, until they get to the global (top floor) and stop, and either find it, or don't.

Unfulfilled RHS references result in `ReferenceError`s being thrown. Unfulfilled LHS references result in an automatic, implicitly-created global of that name (if not in "Strict Mode" [^note-strictmode]), or a `ReferenceError` (if in "Strict Mode" [^note-strictmode]).

### Quiz Answers

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).

	**`c = ..`, `a = 2` (implicit param assignment) and `b = ..`**

2. Identify all the RHS look-ups (there are 4!).

    **`foo(2..`, `= a;`, `a + ..` and `.. + b`**


[^note-strictmode]: MDN: [Strict Mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode)
