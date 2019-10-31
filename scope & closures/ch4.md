# You Don't Know JS: Escopos & Clausuras
# Capítulo 4: Hoisting

A essa altura, você deve estar relativamente confortável com a ideia de escopo, e como variáveis são anexadas a diferentes níveis de escopo  dependendo de onde e como elas são declaradas. Tanto o escopo de função quanto o escopo de bloco se comportam seguindo as mesmas regras e considerando o fato que: qualquer variável declarada em um escopo é anexada ele.

Mas existe um pequeno detalhe de como o anexo de escopo funciona com declarações que aparecem em vários lugares dentro de um escopo, e este detalhe é o que nós analisaremos aqui.

## O Ovo ou a Galinha?

Existe um hábito em pensar que todo o código que você vê em um programa JavaScript é interpretado linha por linha, na ordem de cima para baixo, assim que o programa é executado. Mesmo sendo substancialmente verdade, há uma parte dessa suposição que pode levar à uma ideia errada sobre o seu programa.

Considere esse código:

```js
a = 2;

var a;

console.log( a );
```

O que você espera que seja impresso na instrução `console.log(..)`?

Muitos desenvolvedores esperariam `undefined`, uma vez que a instrução `var a` vem depois de `a = 2`, e pareceria natural assumir que a variável é redefinida, de modo que é atribuído o valor padrão `undefined`. Entretanto, a saída será `2`.

Considere outra parte de código:

```js
console.log( a );

var a = 2;
```

Você pode ser tentado a supor que, já que o exemplo anterior mostrou um certo comportamento aparentemente não tão de-cima-pra-baixo assim, talvez nesse trecho de código, `2` também será impresso. Outros podem pensar que uma vez que variável `a` é usada antes de ser declarada, isso deve resultar em um `ReferenceError` sendo lançado.

Infelizmente, os dois palpites estão errados. `undefined` é a saída.

**Então, o que está acontecendo aqui?** Parece que temos a questão do ovo e da galinha. Quem vem primeiro, a declaração ("ovo") ou a atribuição ("galinha")?

## O Compilador ataca novamente

Para responder essa pergunta, precisamos voltar ao capítulo 1 com a discussão sobre compiladores. Relembre que o *Motor* irá, na verdade, compilar seu código JavaScript antes de interpretá-lo. Parte da fase de compilação era encontrar e associar todas as declarações com seus escopos corretos. Vimos no capítulo 2 que isso é o coração do Escopo Léxico.

Então, a melhor forma de pensar sobre como as coisas funcionam é que todas as declarações, tanto variáveis quanto funções, são processadas primeiro, antes que qualquer parte do nosso código seja executado.

Quando você vê `var a = 2;`, você provavelmente pensa nisso como uma instrução. Mas, na verdade, JavaScript pensa como sendo duas instruções: `var a;` e `a = 2;`. A primeira instrução, a declaração, é processada durante a fase de compilação. A segunda instrução, a atribuição, é deixada **no lugar** para a fase de execução.

Com isso, deveríamos pensar no nosso primeiro trecho de código como sendo tratado assim:

```js
var a;
```
```js
a = 2;

console.log( a );
```

...onde a primeira parte é a compilação e a segunda é a execução.

De maneira similar, nosso segundo trecho de código é, de fato, processado da seguinte forma:

```js
var a;
```
```js
console.log( a );

a = 2;
```

Portanto, uma maneira de pensar, meio que metaforicamente, a respeito desse processo, é que declarações de variável e função são "movidas" de onde elas aparecem, no fluxo do código, para o topo do código. Este processo dá origem ao termo Hoisting.

Em outras palavras, **o ovo (declaração) vem primeiro que a galinha (atribuição)**.

**Lembrete:** Apenas as próprias declarações são "elevadas", enquanto qualquer atribuição ou lógica executável são deixadas *no lugar*. Se Hoisting reorganizasse a lógica executável do nosso código, isso poderia causar estragos.

```js
foo();

function foo() {
	console.log( a ); // undefined

	var a = 2;
}
```

A declaração da função `foo` (na qual nesse caso *inclui* o seu valor implícito como uma função real) é "elevada", de maneira que a chamada da primeira linha está pronta para ser executada.

Também é importante entender que hoisting é **por escopo**. Portanto, enquanto nossos trechos de código anterior eram simplificados nesse ponto, eles apenas incluíam o escopo global, a própria função `foo(...)` que estamos examinando agora, mostra que `var a` é "elevada" para o topo de `foo(...)` (não para o topo do programa, obviamente). Deste modo, o programa pode ser interpretado mais precisamente dessa forma:

```js
function foo() {
	var a;

	console.log( a ); // undefined

	a = 2;
}

foo();
```

Declarações de função são "elevadas", como acabamos de ver. Mas expressões de função não são.

```js
foo(); // não é ReferenceError, mas um TypeError!

var foo = function bar() {
	// ...
};
```

O identificador da variável `foo` é "elevado" e anexado ao escopo delimitado (global) do programa, logo `foo()` não falha devido a `ReferenceError`. Mas `foo` não possui valor ainda (como teria se fosse um declaração de função real em vez de expressão). Portanto, `foo()` é tentada a invocar o valor `undefined`, que é uma operação ilegal `TypeError`.

Também lembre que apesar de ser uma expressão de função nomeada, o identificador de nome não está disponível no escopo delimitado:

```js
foo(); // TypeError
bar(); // ReferenceError

var foo = function bar() {
	// ...
};
```

Esse trecho de código é mais precisamente interpretado (com hoisting) como:

```js
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
	var bar = ...self...
	// ...
}
```


## Primeiro as Funções

Declarações de função e variável são "elevadas". Mas um detalhe  (que *pode* aparecer no código com múltiplas declarações "duplicadas") é que primeiro são "elevadas" as funções, e depois as variáveis.

Considere:

```js
foo(); // 1

var foo;

function foo() {
	console.log( 1 );
}

foo = function() {
	console.log( 2 );
};
```

`1` é impresso em vez de `2`! Esse trecho é interpretado pelo *Motor* como:

```js
function foo() {
	console.log( 1 );
}

foo(); // 1

foo = function() {
	console.log( 2 );
};
```

Note que `var foo` era a declaração duplicada (neste caso ignorada), apesar dela vir antes da declaração `function foo()...`, porque declarações de função são "elevadas" antes de variáveis normais.

Enquanto múltiplas/duplicadas declarações `var` são efetivamente ignoradas, declarações de função subsequentes *sobrescrevem* declarações anteriores.

```js
foo(); // 3

function foo() {
	console.log( 1 );
}

var foo = function() {
	console.log( 2 );
};

function foo() {
	console.log( 3 );
}
```

Embora isso tudo possa parecer nada além de algo interessantemente trivial, destaca-se o fato de que definições duplicadas no mesmo escopo são uma má ideia e muitas vezes irão levar à resultados confusos.

Declarações de função que aparecem dentro de blocos normais tipicamente "elevam" para o escopo delimitado, em vez de serem condicionais como o seguinte código sugere:

```js
foo(); // "b"

var a = true;
if (a) {
   function foo() { console.log( "a" ); }
}
else {
   function foo() { console.log( "b" ); }
}
```

Entretanto, é importante entender que esse comportamento não é confiável e está sujeito a mudanças em futuras versões do JavaScript, por isso é melhor evitar declarar funções em blocos.

## Revisão (TL;DR)

Podemos ser tentados a olhar para `var a = 2;` como sendo uma instrução, mas o *Motor* do JavaScript não vê dessa maneira. Ele vê `var a` e `a = 2` como duas instruções separadas, a primeira como uma tarefa da fase de compilação e a segunda como tarefa da fase de execução.

Isso nos leva à concluir que todas as declarações em um escopo, independente de onde elas aparecerem, são processadas *primeiro* antes do próprio código ser executado. Você pode entender esse processo como declarações sendo "movidas" para o topo de seus respectivos escopos, o qual nós chamamos de hoisting.

As próprias declarações são "elevadas", mas atribuições, mesmo atribuições de expressões de função, *não* são "elevadas".

Cuidado com declarações duplicadas, especialmente misturadas entre declarações de variável normal e de função -- há um certo perigo, caso isso aconteça!
