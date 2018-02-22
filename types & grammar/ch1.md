# You Don't Know JS: Tipos e Gramática
# Capítulo 1: Tipos

Muitos desenvolvedores diriam que uma linguagem dinâmica (como JS) não possui *tipos*. Vamos ver o que a especificação ES5.1 (http://www.ecma-international.org/ecma-262/5.1/) tem a dizer sobre este tópico:

> Algorítimos dentro desta especificação manipulam valores cada um dos quais possui um tipo associado. Os tipos de valores possíveis são exatamente os definidos nessa cláusula. Tipos são ainda sub-classificados na ECMAScript como tipos de linguagem e tipos de especificação.
>
> A tipagem da linguagem ECMAScript corresponde à valores que são diretamente manipulados por um programador ECMAScript usando a linguagem ECMAScript. Os tipos existentes da linguagem ECMAScript são Undefined, Null, Boolean, String, Number, e Object.

Agora, se você é um fã de linguagens com tipagem forte (tipagem estática), você pode opôr-se a esse uso da palavra "tipo." Nessas linguagens, "tipo" significa muito *mais* do que ele faz aqui no JS.

Algumas pessoas dizem que JS não deveria possuir "tipos," antes estes deveriam ser chamados de "tags" ou talvez "subtipos".

Bah! Nós iremos usar essa grosseira definição (a mesma que parece dirigir a escrita da especificação): um *tipo* é intrínseco, construído com um conjunto de características que unicamente identifica o comportamento de um valor em particular e distingue-o de outros valores, tanto para o motor (engine) **quanto para o desenvolvedor**.

Em outras palavras, se tanto o motor (engine) quanto o desenvolvedor tratam o valor `42` (o número) diferentemente do que eles tratam o valor `"42"` (a string), então esses dois valores possuem *tipos* diferentes -- `number` e `string`, respectivamente. Quando você usa `42`, você *pretende* fazer algo numérico, como matemática. Mas quando você usa `"42"`, você está *pretendendo* fazer alguma coisa com texto/strings, como saída (output) para a página, etc. **Esses dois valores possuem tipagem diferente.**

Isso não é uma definição perfeita. Mas é bom o suficiente para essa discussão. E é consistente com a maneira que JS descreve-se.

# O que importa em um tipo é o que ele é, não pelo quê é chamado...

Além dos desacordos da definição acadêmica, por que é importante se JavaScript possui *tipos* ou não?

Ter uma compreensão adequada de cada *tipo* e seu comportamento intrínsico é absolutamente essencial para entender como converter valores corretamente e precisamente para diferentes tipos (veja Coerção (Coercion), Capítulo 4). Quase todos programas escritos em JS precisarão lidar com coerção de valores em algum tipo de formulário, assim é importante você fazê-los com responsabilidade e confiança.

Se você possui o `número (number)` com valor `42`, mas você deseja tratá-lo como uma `string`, tal que a saída dê-se como um caractere `"2"` na posição `1`, você obviamente deve primeiro converter (coerce) o valor de `número (number)` para `string`.

Isso parece bastante simples.

Mas exitem muitas maneiras diferentes em que a coerção pode acontecer. Algumas dessas maneiras são explícitas, fáceis de assimilar e confiáveis. Mas se você não for cuidadoso, a coerção (coercion) pode acontecer de formas muito estranhas e surpreendentes.

Confusão na coerção é possivelmente uma das mais profundas frustrações para desenvolvedores JavaScript. Tem sido frequentemente criticada como sendo tão *perigosa* que pode ser considerada uma falha no design da linguagem, à ser banida e evitada.

Armados com total conhecimento dos tipos do JavaScript, nosso objetivo é demonstrar por que a *má reputação* da coerção é largamente famosa e um pouco imerecida -- mudando a sua perspectiva, para ver a coerção como uma poderosa utilidade. Mas primeiro, devemos ter uma compreensão melhor sobre valores e tipos.

## Tipos nativos (built-in)

JavaScript define sete tipos nativos:

* `null`
* `undefined`
* `boolean`
* `number`
* `string`
* `object`
* `symbol` -- adicionado no ES6!

**Nota:** Todos esses tipos, exceto `object` são chamados "primitivos".

O operador `typeof` inspeciona o tipo do valor obtido, e sempre retorna um dos sete valores de string -- surpreendentemente, não há uma correspondência exata de 1-para-1 com os sete tipos nativos que listamos.

```js
typeof undefined     === "undefined"; // true (verdadeiro)
typeof true          === "boolean";   // true (verdadeiro)
typeof 42            === "number";    // true (verdadeiro)
typeof "42"          === "string";    // true (verdadeiro)
typeof { life: 42 }  === "object";    // true (verdadeiro)

// adicionado no ES6!
typeof Symbol()      === "symbol";    // true (verdadeiro)
```

Estes seis tipos listados possuem valores do tipo correspondente e retornam uma string com o mesmo nome, como mostrado. `Symbol` é um novo tipo de dados introduzido à partir do ES6, e será explicado no Capítulo 3.

Como você pode perceber, eu excluí `null` da lista acima. Ele é *especial* -- especial no sentido que é problemático quando combinado com o operador `typeof`:

```js
typeof null === "object"; // true (verdadeiro)
```

Teria sido legal (e correto!) se retornasse `"null"`, mas esse "bug" original no JS tem persistido por aproximadamente duas décadas, e provavelmente nunca será corrigido porque existe muito conteúdo na web que se baseia nesse comportamente "bugado", e corrigí-lo *criaria* mais "bugs", e causaria rupturas em uma grade quantidade de softwares web.

Se você deseja testar um valor `null` usando seu tipo, precisará de uma condição composta:

```js
var a = null;

(!a && typeof a === "object"); // true (verdadeiro)
```

`null` é o único valor primitivo que pode ser "falso" (também conhecido como "false-like"; veja o Capítulo 4) mas que também retorna `"object"` na checagem de `typeof`.

Então qual é o sétimo valor de string que `typeof` pode retornar?

```js
typeof function a(){ /* .. */ } === "function"; // true (verdadeiro)
```

É fácil pensar que `function (função)` é um tipo nativo de alto nível no JS, especialmente obtendo esse comportamento do operador `typeof`. Contudo, se você ler a especificação, verá que atualmente é um "subtipo" de objeto. Especificamente, uma função é referenciada como um "objeto chamável (callable object)" -- um objeto que possui uma propriedade `[[Call]]` interna que possibilita ao mesmo ser invocado.

O fato de funções atualmente serem objetos é muito útil. Mais importante, elas podem ter propriedades. Por exemplo:

```js
function a(b,c) {
	/* .. */
}
```

O objeto de função tem uma propriedade `length` definido com o número de parâmetros formais com o qual a função foi declarada.

```js
a.length; // 2
```

Uma vez que você tenha declarado a função com dois parâmetros formalmente nomeados (`b` e `c`), o "length (tamanho) da função" será `2`.

E quanto aos arrays? Eles são nativos para o JS, então eles são um tipo especial?

```js
typeof [1,2,3] === "object"; // true (verdadeiro)
```

Não, apenas objetos. É muito apropriado pensar sobre eles também como um "subtipo" de objeto (veja o Capítulo 3), neste caso com as características adicionais de serem numéricamente indexáveis (em oposição aos objetos simples com chaves do tipo string) e manterem uma propriedade `.length` automaticamente atualizada.

## Valores como tipos

Em JavaScript, variáveis não possuem tipos -- **valores possuem tipos**. Variáveis podem conter qualquer valor à qualquer momento.

Outra maneira de pensar sobre tipos do JS é que JS não possui "tipagem forçada (type enforcement)," pois a engine não insiste que uma *variável* sempre terá valores do *mesmo tipo inicial* com que ela começou. Uma variável pode, em uma declaração de atribuição, possuir um valor do tipo `string`, e no futuro ter um `number`, e assim por diante.

O *valor* `42` possui o tipo `number` intrínseco, e seu *tipo* não pode ser modificado. Um outro valor, como `"42"` com o tipo `string`, pode ser criado *à partir* do valor `42` de tipo `number` através de um processo chamado **coerção (coercion)** (veja o Capítulo 4).

Se você usa o `typeof` em uma variável, ele não está perguntando "qual o tipo da variável?" como pode parecer, pois variáveis JS não possuem tipos. Pelo contrário, ele está perguntando "qual é o tipo do valor *na* variável?"

```js
var a = 42;
typeof a; // "number" ("número")

a = true;
typeof a; // "boolean" ("booleano")
```

O operador `typeof` sempre retorna uma string. Assim:

```js
typeof typeof 42; // "string"
```

O primeiro `typeof 42` retorna `"number"`, e `typeof "number"` é `"string"`.

### `undefined` vs "undeclared"

Variáveis que não possuem valores *neste momento*, atualmente possuem o valor `undefined (indefinido)`. Chamar `typeof` nestas variáveis retornará `"undefined"`:

```js
var a;

typeof a; // "undefined" ("indefinido")

var b = 42;
var c;

// depois
b = c;

typeof b; // "undefined" ("indefinido")
typeof c; // "undefined" ("indefinido")
```

É tentador para muitos desenvolvedores olharem para a palavra "undefined (indefinido)" e pensar nela como sinônimo para "undeclared (não declarada)." Contudo, em JS, esses dois conceitos são bastante diferentes.

Uma variável "undefined" é aquela que foi declarada no escopo acessível, mas *neste momento* não possui nenhum outro valor. Por contraste, uma variável "undeclared" é aquela que não foi formalmente declarada no escopo acessível.

Considere:

```js
var a;

a; // undefined (indefinido)
b; // ReferenceError: b is not defined (Erro de Referência: b não está definido)
```

Uma irritante confusão é a mensagem de erro que os navegadores atribuem para essa condição. Como você pode ver, a mensagem é "b is not defined (b não está definido)," a qual é com certeza muito fácil de confundir com "b is undefined (b é indefinido)." Novamente, "undefined (indefinido)" e "is not defined (não está definido)" são coisas bem diferentes. Seria ótimo se os navegadores retornassem algo como "b is not found (b não foi encontrado)" ou "b is not declared (b não está declarado) ," para reduzir essa confusão!

Há também um comportamento especial associado com `typeof` no que se refere às variáveis não declaradas que reforça ainda mais a confusão. Considere:

```js
var a;

typeof a; // "undefined" ("indefinido")

typeof b; // "undefined" ("indefinido"?)
```

O operador `typeof` retorna `"undefined"` até para variáveis "undeclared (não declaradas)" (ou "não definidas"). Note que aqui não há erros disparados quando nós executamos `typeof b`, apesar de `b` ser uma variável "não declarada". Isso é uma questão especial de segurança no comportamento de `typeof`.

Similarmente, teria sido ótimo se `typeof` usado com uma variável não declarada retornasse "undeclared" ao invés de confundir o resultado com os diferentes tipos de "undefined".

### `typeof` Undeclared

De qualquer forma, esta proteção de segurança é útil quando lidamos com JavaScript no navegador, onde vários arquivos de script podem carregar variáveis no namespace global compartilhado.

**Nota:** Muitos desenvolvedores acreditam que jamais deverá haver variáveis no namespace global, e que tudo deve estar contido em módulos e namespaces privados/separados. Isto é ótimo em teoria, mas quase impossível na prática; Mesmo assim, é um bom objetivo para se tentar alcançar! Felizmente, o ES6 adicionou suporte de primeira classe para módulos, que eventualmente tornará isso muito mais prático.

Como um exemplo simples, imagine ter um "modo debug (depuração)" em seu programa que é controlado por uma variável global (flag) chamada `DEBUG`. Você gostaria de verificar se essa variável foi declarada antes de executar uma tarefa de depuração, como por exemplo, exibir uma mensagem de log no console. A declaração global de nível superior `var DEBUG = true` só seria incluída no arquivo "debug.js", o qual você carrega no navegador somente quando estiver em desenvolvimento/teste, mas não em produção.

Entretanto, você deve tomar cuidado com a forma na qual você verifica a variável global `DEBUG` no restante do código da sua aplicação, para que você não cause um `ReferenceError` (Erro de referência). A proteção de segurança por meio do `typeof` é nossa amiga nesse caso.

```js
// oops, isto causaria um erro!
if (DEBUG) {
	console.log( "A depuração está começando" );
}

// Esta é uma verificação de existência segura
if (typeof DEBUG !== "undefined") {
	console.log( "A depuração está começando" );
}
```

Este tipo de verificação é útil mesmo se você não estiver lidando com váriaveis definidas pelo usuário (como a `DEBUG`). Se você estiver fazendo uma verificação de recursos para integração com uma API, você também pode achar útil verificar sem lançar um erro:

```js
if (typeof atob === "undefined") {
	atob = function() { /*..*/ };
}
```

**Nota:** Se você estiver definindo um "polyfill" para um recurso que ainda não existe, você provavelmente deseja evitar o uso de `var` para fazer a declaração de `atob`. Se você declarar `var atob` dentro da instrução `if`, esta declaração será elevada (veja o livro *Escopos & Clausuras* desta série) para o topo do escopo, mesmo se a condição `if` não passar (porque a váriavel global `atob` já existe!). Em alguns navegadores e para alguns tipos especiais de variáveis construidas globalmente (geralmente chamadas de "host objects (objetos hospedeiros)"), esta declaração duplicada pode gerar erros. Omitindo o `var` impede que essa declaração seja elevada.

Outra maneira de fazer essas verificações de variáveis globais sem utilizar a proteção de segurança do `typeof` é observar que todas essas variáveis globais são propriedades do objeto global, que no navegador é basicamente o objeto `window`. Então, as verificações acima poderiam ter sido feitas (seguramente) com:

```js
if (window.DEBUG) {
	// ..
}

if (!window.atob) {
	// ..
}
```

Ao contrário de referenciar variáveis não declaradas, não ocorrerá um `ReferenceError` (Erro de referência) se você tentar acessar uma propriedade que não existe em um objeto (mesmo no objeto global `window`).

Por outro lado, referenciar manualmente variáveis globais utilizando a referência de `window` é algo que alguns desenvolvedores preferem evitar, especialmente se o seu código precisa ser executado em múltiplos ambientes JS (não apenas navegadores, mas do lado do servidor, como node.js, por exemplo), onde o objeto global pode nem sempre ser chamado `window`.

Tecnicamente, a proteção de segurança por meio do `typeof` é útil mesmo se você não estiver utilizando variáveis globais, embora essas circunstâncias sejam menos comuns, e alguns desenvolvedores podem achar essa abordagem menos aplicável. Imagine uma função de utilidade a qual você deseja que outros copiem e colem em seus programas ou módulos, onde você deseja verificar se o programa em questão contém a definição para uma certa variável (para que você possa usá-la) ou não:

```js
function doSomethingCool() {
	var helper =
		(typeof FeatureXYZ !== "undefined") ?
		FeatureXYZ :
		function() { /*.. característica padrão ..*/ };

	var val = helper();
	// ..
}
```

`doSomethingCool()` testa uma variável chamada `FeatureXYZ` e, se econtrar, utiliza ela, mas se não, utiliza sua própria definição. Agora, se alguém incluir em seu programa/módulo esta função, ele verificará com segurança se `FeatureXYZ` foi definida ou não:

```js
// uma IIFE (veja a discussão "Expressão de Função Imediatamente Invocada"
// no livro *Escopo & Clausuras* desta série)
(function(){
	function FeatureXYZ() { /*.. minha função XYZ ..*/ }

	// incluir `doSomethingCool(..)`
	function doSomethingCool() {
		var helper =
			(typeof FeatureXYZ !== "undefined") ?
			FeatureXYZ :
			function() { /*.. característica padrão ..*/ };

		var val = helper();
		// ..
	}

	doSomethingCool();
})();
```

Neste caso, `FeatureXYZ` não é uma variável global, mas assim mesmo estamos utilizando a proteção de segurança por meio do `typeof` para garantir a verificação. E o mais importante, aqui não existe objeto que possamos usar (como fizemos para variáveis globais com `window.___`) para fazer a verificação, então o `typeof` é muito útil.

Outros desenvolvedores preferem um padrão de projetos chamado "injeção de dependência", em que, ao invés de `doSomethingCool()` verificar implicitamente se `FeatureXYZ` está definida fora dela, ela precisaria ter a dependência passada explicitamente, como:

```js
function doSomethingCool(FeatureXYZ) {
	var helper = FeatureXYZ ||
		function() { /*.. característica padrão ..*/ };

	var val = helper();
	// ..
}
```

Há muitas opções para projetar essa funcionalidade. Nenhum padrão apresentado aqui é "certo" ou "errado" -- Existem vários prós e contras para cada abordagem. Mas, em geral, é bom que a proteção de segurança do `typeof` para tipos não declarados (undeclared) nos dê mais opções.

## Revisão

JavaScript tem sete *tipos* nativos: `null`, `undefined`,  `boolean`, `number`, `string`, `object`, `symbol`. Eles podem ser identificados pelo operador `typeof`.

Variáveis não possuem tipos, mas os valores sim. Estes tipos definem o comportamento intrínseco dos valores.

Muitos desenvolvedores assumirão que "undefined" e "undeclared" são, grosseiramente, a mesma coisa, mas em JavaScript, elas são bem diferentes. `undefined` é um valor que uma variável declarada pode conter. "Undeclared" significa que uma variável nunca foi declarada.

O JavaScript, infelizmente, combina esses dois termos não só nas suas mensagens de erro ("ReferenceError: a is not defined (Erro de referência: a não está definido)"), mas também no valor retornado pelo `typeof`, que é `"undefined"` para os dois casos.

No entando, a proteção de segurança (evitando um erro) do `typeof` quando utilizada em uma variável não declarada, em alguns casos pode ser útil.
