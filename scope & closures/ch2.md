# You Don't Know JS: Escopos e Clausuras
# Chapter 2: Escopo Léxico

No Capítulo 1, definimos "escopo" como o conjunto de regras que dita a forma com que o *Motor* poderá buscar e eventualmente localizar variáveis através de seus identificadores, tanto no *Escopo* atual quando nos *Escopos aninhados* que este possa estar inserido.

Existem dois modelos principais para a definição de funcionamento do escopo. O primeiro e mais comum, utilizado pela grande maioria das linguagens de programação, é chamado de **Escopo Léxico**, e vamos examiná-lo em profundidade. O outro modelo, que ainda é utilizado em algumas linguagens como Bash scripting e alguns modos de Perl, é chamado de **Escopo Dinâmico**.

Escopo Dinâmico é tratado no Apêndice A. Menciono esta informação aqui apenas para definir um contraste em relação ao Escopo Léxico, que é o modelo utilizado pelo JavaScript.

## Hora do léxico

Conforme discutimos no Capítulo 1, a primeira etapa da compilação de linguagens tradicionais é chamada de Análise Léxica (ou Tokenização). Caso não se lembre, o processo de Análise Léxica examina uma sequência de caractéres do código fonte e atribui um significado semântico para estes símbolos (tokens) como resultado de uma análise stateful.

Este é o conceito que provê as bases para compreensão do que é o Escopo Léxico e a origem do seu nome.

Para uma definição de certa forma redundante, o Escopo Léxico é o escopo definido durante a etapa de Análise Léxica. Em outras palavras, o Escopo Léxico baseia-se no local onde variáveis e blocos de escopo são criados por você durante a escrita do código, portanto (normalmente) já definidos no momento que o analisador léxico processa seu código.

**Nota:** Veremos em alguns instantes que existem formas de enganar o Escopo Léxico, e assim sendo modificá-lo após sua passagem pelo analisador léxico, mas isso é, de certa forma, mal visto. É considerado boa pratica tratar o escopo léxico como, de fato, léxico, e portanto inteiramente associado ao momento em que foi definido pelo autor do código.

Consideremos o seguinte bloco de código:

```js
function foo(a) {

	var b = a * 2;

	function bar(c) {
		console.log( a, b, c );
	}

	bar(b * 3);
}

foo( 2 ); // 2 4 12
```

Existem três escopos distintos neste exemplo de código. Talvez facilite pensar neles como bolhas dentro umas das outras.

<img src="fig2.png" width="500">

**Bolha 1** representa o escopo global, e possui apenas um identificador: `foo`.

**Bolha 2** representa o escopo de `foo`, que por sua vez possui três identificadores: `a`, `bar` e `b`.

**Bolha 3** representa o escopo de `bar`, que possui apenas um identificador: `c`.

Estas bolhas são definidas pelo local onde o escopo foi definido, cada um deles aninhado com outro e assim por diante. No próximo capítulo, vamos discutir diferentes unidades de escopo, mas, por ora, vamos assumir que cada função cria uma nova bolha de escopo.

A bolha de `bar` está contida na bolha de `foo`, porque (e somente por isso) foi o local que optamos por declarar a função `bar`.

Observe que estas bolhas são estritamente aninhadas. Não estamos falando de um Diagrama de Venn, onde as fronteiras (dos conjuntos matemáticos) podem ser atravessadas (para definição de interseções). Em outras palavras, e diferente dos conjuntos, não é possível que a bolha de escopo de uma função esteja presente simultaneamente em duas outras bolhas de escopo, assim como não é possível que uma mesma função seja declarada parte em uma função e parte em outra.

### Consultas

A estrutura e a relação de posicionamento destas bolhas de escopo descreve para o *Motor* todos os locais nos quais deve consultar para localizar um identificador.

No trecho de código acima, o *Motor* executa a instrução `console.log(..)` e vai em busca das três variáveis referenciadas `a`, `b` e `c`. Ele inicia com o escopo mais interno, o escopo da função `bar`. Não encontrará `a` por lá, então sobe um nível para a bolha de escopo mais próxima, o escopo de `foo(..)`. Lá ele localiza `a`, e utiliza este `a`. A mesma coisa ocorre com `b`. Porém, no caso de `c`, ele localiza dentro de `bar(..)`.

Caso houvesse um `c` definido em `bar(..)` e outro em `foo(..)`, a instrução `console.log(..)` teria localizado e utilizado o `c` definido em `bar(..)` e nunca chegaria até o valor definido em `foo(..)`.

**A consulta de escopo se encerra no momento que uma ocorrência é localizada**. Um mesmo identificador pode ser definido em diferentes camadas de escopo aninhadas, o que é chamado de "sombreamento" (*shadowing* -- o identificador interno "põe sombra" sobre o identificador externo). Independente do sombreamento, a consulta de escopo sempre se inicia no escopo mais próximo do ponto de execução atual, e segue seu caminho para fora/cima até a localização de uma ocorrência, quando se encerra.

**Nota:** Variáveis globais tornam-se automaticamente propriedades do objeto global (`window` nos navegadores, etc.), portanto *é possível* referenciar uma variável global de forma direta através de seu nome léxico, mas também de forma indireta ao referenciar uma propriedade do objeto global.

```js
window.a
```

Esta técnica garante o acesso a uma variável global que não poderia ser acessada por conta de um eventual sombreamento. Entretanto, variáveis não-globais e que foram sombreadas não podem ser acessadas.

Não importa o *local* onde uma função é invocada, ou até mesmo *como* é invocada, seu escopo léxico será definido **apenas** pelo local onde a função foi declarada.

O processo de consulta ao escopo léxico ocorre *apenas" em identificadores de primeira classe, como `a`, `b` e `c`. Se você tivesse uma referência para `foo.bar.baz` em um trecho de código, ocorreria uma consulta ao escopo léxico para localizar o identificador `foo`, mas no momento que esta variável é localizada, regras de acesso à propriedades de objetos assumem o comando para resolução das propriedades `bar` e `baz`, respectivamente.

## Trapaceando o Léxico

Se o escopo léxico é de fato definido apenas pelo local onde uma função é declarada e este local é escolhido no momento da escrita do código, como pode haver uma maneira de "modificar" (ou trapacear) o escopo léxico em tempo de execução?

JavaScript possui dois mecanismos para isso. Ambos são vistos como má prática e igualmente (e amplamente!) desencorajados pela comunidade de modo geral, embora os argumentos que sustentam esta opinião normalmente não trazem consigo o ponto mais relevante: **trapacear o escopo léxico leva a um pior desempenho.**

Antes de explicar a questão da performance, porém, vamos olhar a forma com que estes dois mecanismos funcionam.

### `eval`

A função `eval(..)` em JavaScript recebe uma string como argumento e trata o conteúdo desta string como se tivesse de fato sido programado pelo autor do código naquele ponto do programa. Em outras palavras, você pode gerar código dinamicamente dentro do seu programa e executar este código como se estivesse lá desde o momento da programação.

Colocando desta forma, deve estar claro como `eval(..)` permite a você modificar o ambiente do escopo léxico ao trapacear e portanto fingir que aquilo que foi gerado dinamicamente estava lá desde o momento da escrita do código.

Durante a execução das linhas que sucedem a chamada para `eval(..)`, o *Motor* não vai "saber" ou "se importar" se o código em questão foi interpretado dinamicamente e portanto modificou o ambiente do escopo léxico. O *Motor* vai seguir efetuando suas consultas ao escopo léxico da mesma forma de sempre.

Considere o código a seguir:

```js
function foo(str, a) {
	eval( str ); // trapaça!
	console.log( a, b );
}

var b = 2;

foo( "var b = 3;", 1 ); // 1, 3
```

A string `"var b = 3;"` é tratada, naquele ponto onde `eval(..)` é chamado, como um código que esteve lá desde o princípio. Pelo fato deste código declarar uma nova variável `b`, ele modifica o atual escopo léxico de `foo(..)`. O que ocorre, como mencionado acima, é que este código literalmente cria a variável `b` dentro de `foo(..)`, o que acaba por sombrear a variável `b` que foi declarada no escopo externo (neste caso, o global).

Quando a chamada para `console.log(..)` ocorre, são encontradas tanto `a` quanto `b` no escopo de `foo(..)`, e portanto nunca a variável `b` externa. Sendo assim, imprimimos "1, 3" em vez de "1, 2" como normalmente ocorreria.

**Nota:** Neste exemplo, por questões de simplificação, a string de "código" que interpretamos possui um valor fixo, mas poderia facilmente ter sido gerada dinamicamente a partir de fragmentos obtidos pela lógica do seu programa. `eval(..)`é normalmente utilizada para executar código gerado dinamicamente, afinal não há qualquer benefício em interpretar dinamicamente um trecho de código estático a partir de uma string se você pode adicionar este mesmo trecho no momento da escrita do código.

Por padrão, se uma string de código executada via `eval(..)` possui uma ou mais declarações (seja de variáveis ou funções), esta ação modifica o escopo léxico no qual esta chamada para `eval(..)` se encontra. Tecnicamente, `eval(..)` pode ser invocada "indiretamente" por meio de vários truques (os quais vão além da nossa discussão), o que faz com que seja executada no contexto do escopo global, e assim sendo, modificando-o. Mas de qualquer maneira, `eval(..)` pode em tempo de execução modificar um escopo léxico definido durante a escrita do código.

**Note:** Quando utilizada em um programa em Modo estrito (strict mode), `eval(..)` opera em seu próprio escopo léxico, o que significa que as declarações efetuadas dentro de `eval()` não modificam o escopo superior.

```js
function foo(str) {
   "use strict";
   eval( str );
   console.log( a ); // ReferenceError: a is not defined
}

foo( "var a = 2" );
```

Javascript provê outras maneiras de se obter resultados similares aos de `eval(..)`. `setTimeout(..)` e `setInterval(..)` *podem* receber uma string como primeiro argumento, conteúdo este que será interpretado por `eval(..)` como o código de uma função gerada dinamicamente. Isto é um comportamento velho, legado, e desaconselhado há muito tempo. Não faça isso!

O construtor de função `new Function(..)`, de forma similar, recebe uma string de código como seu **último** argumento para torná-la uma função gerada dinamicamente -- o(s) primeiro(s) argumento(s), se existir(em), nomeia(m) o(s) parâmetro(s) da nova função. Ainda assim, isso deve ser evitado em seu código.

Os casos de uso para geração dinâmica de código são incrivelmente raros, visto que as perdas de performance quase nunca tornam esta prática vantajosa.

### `with`

A outra funcionalidade mal vista (e agora desaconselhada!) em JavaScript e com a qual se pode trapacear o escopo léxico é a keyword `with`. Existem muitas maneiras válidas de se explicar `with`, mas vou escolher explicar sob a óptica de como este mecanismo interage e afeta o escopo léxico.

`with` é comumente definido como um "atalho" para a criação de diversas referências à propriedades de um determinado objeto sem precisarmos referenciá-lo em cada uma delas.

Por exemplo:

```js
var obj = {
	a: 1,
	b: 2,
	c: 3
};

// forma mais "chata", repetindo "obj"
obj.a = 2;
obj.b = 3;
obj.c = 4;

// "atalho" mais fácil
with (obj) {
	a = 3;
	b = 4;
	c = 5;
}
```

Entretanto, há muito mais coisas acontecendo por aqui do que a simples conveniência de acesso às propriedades de um objeto. Considerando:

```js
function foo(obj) {
	with (obj) {
		a = 2;
	}
}

var o1 = {
	a: 3
};

var o2 = {
	b: 3
};

foo( o1 );
console.log( o1.a ); // 2

foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2 -- Opa, "vazou" para o escopo global!
```

No código deste exemplo, dois objetos `o1` e `o2` são criados. Um possui uma propriedade `a` e o outro não. A função `foo(...)` recebe a referência de um objeto `obj` como argumento, e chama `with (obj) { .. }` com esta referência. Dentro do bloco `with`, criamos o que parece se tratar de uma referência léxica comum para a variável `a`, uma referência LHS para ser mais exato (veja o Capítulo 1), de forma a atribuir-lhe o valor 2.

Quando passamos `o1`, a atribuição `a = 2` encontra a propriedade `o1.a` e atribui-lhe o valor `2`, conforme podemos observar na instrução `console.log(o1.a)` logo a seguir. Porém, quando passamos `o2`, tendo em vista que este não possui uma propriedade `a`, nenhuma propriedade é criada e `o2.a` segue sendo `undefined`.

Então percebemos um efeito colateral peculiar, o fato de que a variável global `a` foi criada pela atribuição `a = 2`. Como isso pode ter acontecido?

A instrução `with` recebe um objeto com zero ou mais propriedades, **trata *este* objeto como se fosse um escopo léxico à parte** e portanto suas propriedades são tratadas como identificadores definidos de forma lexical neste "escopo".

**Nota:** Embora um bloco `with` trate um objeto como um escopo léxico, uma declaração `var` dentro deste bloco não terá seu escopo atrelado ao bloco `with`, mas sim ao escopo no qual este bloco se encontra.

Enquanto a função `eval(..)` pode modificar o escopo léxico ao receber uma string com um código que possua uma ou mais declarações, a instrução `with`, por sua vez, cria um **escopo léxico totalmente novo** a partir do objeto que você passou.

Entendido desta forma, o "escopo" declarado pela instrução `with` quando passamos `o1` era `o1`, e aquele "escopo" possuía um "identificador" que correspondia à propriedade `o1.a`. Mas quando utilizamos `o2` como "escopo", este não possuía um "identificador" `a`, então se aplicam as regras normais de uma busca LHS (veja o Capítulo 1).

O identificador `a` não pode ser achado no escopo de `o2`, no escopo de `foo(...) nem no escopo global, então quando `a = 2` é executado, resulta na criação da variável global, já que não estamos em Modo estrito (strict mode).

É um pouco alucinante pernsarmos no bloco `with` tornando, em tempo de execução, um objeto e suas propriedades em um "escopo" *com* "identificadores". Mas é a forma mais clara que eu tenho para apresentar os resultados que vemos.

**Nota:** Somando-se ao fato de não ser uma boa ideia utilizá-las, `eval(..)` e `with` são afetadas (restringidas) pelo Modo Estrito (strict mode). `with` é totalmente proibida, ao passo que várias formas indiretas ou inseguras de se utilizar `eval(..)` são proibidas ainda que sua funcionalidade central seja mantida.

### Performance

Both `eval(..)` and `with` cheat the otherwise author-time defined lexical scope by modifying or creating new lexical scope at runtime.

So, what's the big deal, you ask? If they offer more sophisticated functionality and coding flexibility, aren't these *good* features? **No.**

The JavaScript *Engine* has a number of performance optimizations that it performs during the compilation phase. Some of these boil down to being able to essentially statically analyze the code as it lexes, and pre-determine where all the variable and function declarations are, so that it takes less effort to resolve identifiers during execution.

But if the *Engine* finds an `eval(..)` or `with` in the code, it essentially has to *assume* that all its awareness of identifier location may be invalid, because it cannot know at lexing time exactly what code you may pass to `eval(..)` to modify the lexical scope, or the contents of the object you may pass to `with` to create a new lexical scope to be consulted.

In other words, in the pessimistic sense, most of those optimizations it *would* make are pointless if `eval(..)` or `with` are present, so it simply doesn't perform the optimizations *at all*.

Your code will almost certainly tend to run slower simply by the fact that you include an `eval(..)` or `with` anywhere in the code. No matter how smart the *Engine* may be about trying to limit the side-effects of these pessimistic assumptions, **there's no getting around the fact that without the optimizations, code runs slower.**

## Review (TL;DR)

Lexical scope means that scope is defined by author-time decisions of where functions are declared. The lexing phase of compilation is essentially able to know where and how all identifiers are declared, and thus predict how they will be looked-up during execution.

Two mechanisms in JavaScript can "cheat" lexical scope: `eval(..)` and `with`. The former can modify existing lexical scope (at runtime) by evaluating a string of "code" which has one or more declarations in it. The latter essentially creates a whole new lexical scope (again, at runtime) by treating an object reference *as* a "scope" and that object's properties as scoped identifiers.

The downside to these mechanisms is that it defeats the *Engine*'s ability to perform compile-time optimizations regarding scope look-up, because the *Engine* has to assume pessimistically that such optimizations will be invalid. Code *will* run slower as a result of using either feature. **Don't use them.**
