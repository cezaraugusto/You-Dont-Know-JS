# You Don't Know JS: Tipos e Gramática
# Capitulo 4: Coerção

Agora que nós entendemos melhor os tipos e valores do JavaScript, vamos voltar nossa atenção para um tópico muito controverso: coerção.

Como mencionado no Capítulo 1, os debates sobre se coerção é um recurso útil ou uma falha no design da linguagem (ou algo no meio disso!) têm ocorrido desde o primeiro dia. Se você já leu outros livros sobre JS, você sabe que a *mensagem* prevalentemente esmagadora é que coerção é mágica, má, confusa, e francamente uma ideia ruim.

Seguindo o mesmo espírito dessa série de livros, ao invés de fugir da coerção porque todo mundo faz isso, ou porque você tem algum capricho, eu acho que você deve seguir em frente e procurar entender mais claramente.

Nosso objetivo é entender plenamente os prós e contras (sim, *existem* prós!) da coerção, então você poderá estar bem informado para tomar a decisão de como ela se adequa ao seu programa.

## Convertendo Valores

Converter um valor de um tipo para outro é geralmente chamado de "type casting", quando explicitamente, e "coerção" quando feito implicitamente (forçado pelas regras de como um valor é utilizado).

**Nota:** Pode não ser óbvio, mas as coerções do JavaScript sempre resultam em um dos primitivos escalares (veja Capítulo 2), como `string`, `number`, ou `boolean`. Não existe coerção que resulte em um valor complexo como `object` ou `function`. O Capítulo 3 aborda "boxing", que envolve os valores primitivos em seus `object` homólogos, mas isso não é realmente coerção em seu sentido correto.

Pode-se distinguir esses de ainda outra maneira: "type casting" (ou coversão de tipos) ocorrem em tempo de compilação em linguagem staticamente tipadas, enquanto "coerção" é uma conversão que ocorre em tempo de execução em linguagens dinamicamente tipadas.

Entretanto, em JavaScript, a maioria das pessoas chamam todos esse tipos de *coerção*, por isso a maneira que prefiro chamar de "coerção implícita" vs. "coerção explícita".

A diferença deve ser óbvia: "coerção explícita" é quando consegue-se olhar para o código e ver que a coversão de tipos está ocorrendo intencionalmente, enquanto que a "coerção implícita" é quando o conversão de tipos ocorre de maneira óbvia como um efeito colateral de outra operação intencional.

Por exemplo, considere essas duas abordagens de coerção:

```js
var a = 42;

var b = a + "";			// coerção implícita

var c = String( a );	// coerção explícita
```

Para `b`, a coerção ocorre implicitamente, porque o operador `+` combinado com um dos operandos sendo uma `string` (`""`) vai insistir que a operação seja uma concatenação de strings (juntando duas strings), que, *como um efeito colateral (oculto)* vai forçar o valor `42` em `a` a ser convertido a seu equivalente em `string`: `"42"`.

Em contrapartida, a função `String(..)` torna bastante óbvio que o valor de `a` é convertido em sua representação em `string`.

As duas abordagens têm o mesmo efeito: `42` vira `"42"`. Mas é o *como* que é o coração dos debates mais acalorados sobre coerção no JavaScript.

**Note:** Tecnicamente, há pequenas nuances no comportamento que vão além da estética. Nós abordaremos isso em mais detalhes mais tarde neste capítulo, na seção "Implicitamente: Strings <--> Number".

Os termos "explícito" e "implícito", ou "óbvio" e "efeito colateral oculto", são *relativos*.

Se você sabe exatamente o que `a + ""` está fazendo e você está intencionalmente convertendo o valor para uma `string`, você pode achar a operção suficientemente "explícita". Por outro lado, se você nunca viu a função `String(..)` usada para coerção de strings, o seu comportamento pode parecer oculto o suficiente para ser "implícito" para você.

Mas nós estamos discutindo "explícito" vs. "implícito" baseados nas prováveis opiniões de um desenvolvedor *mediano, razoavelmente informado, mas não especialista ou devoto pela especificação do JS*. Se você se encaixa de alguma forma nesse grupo, você terá de ajustar a sua perspectiva para estar de acordo com as nossas observações.

Relembrando: é pouco provável que após escrevermos o nosso código, nós sejamos os únicos que vão lê-lo. Mesmo que você saiba todos os prós e contras no JS, considere como um colega de trabalho com menos experiência vai ser se sentir quando ler o seu código. Vai ser "explícito" ou "implícito" para eles da mesma maneira que é para você?

## Operações de valor abstrato

Antes de explorarmos a coerção *explícita* e *implícita*, nós precisamos aprender as regras básicas que governam como os valores *tornam-se* uma `string`, `number` ou `boolean`. A seção 9 da especificação ES5 define várias "operações abstratas" (nome técnico para "operação interna") com as regras de conversão de valor. Nós vamos prestar atenção, especificamente, em: `ToString`, `ToNumber` e `ToBoolean`, e menos na extensão  `ToPrimitive`.

### `ToString`

Quando um valor não-`string` é convertido para uma representação `string`, a conversão é manipulada pela operação abstrata `ToString` na seção 9.8 da especificação.

Valores primitivos nativos têm stringficação natural: `null` torna-se `"null"`, `undefined` torna-se `"undefined"` e `true` torna-se `"true"`. `numbers` são geralmente expressos de forma natural como você esperava, mas como discutimos no Capítulo 2, `numbers` muito pequenos ou muito grandes são representados na forma expoente:

```js
// multiplicando `1.07` por `1000`, sete vezes mais
var a = 1.07 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000;

// sete vezes três dígitos => 21 digits
a.toString(); // "1.07e21"
```

Para objetos regulares, a menos que você mesmo especifique, o padrão `toString()` (localizado em Object.prototype.toString()`) vai retornar uma *`[[Class]]` interna* (veja o capítulo 3), como por exemplo `"[object Object]"`.

Mas como mostrado anteriormente, se um objeto tem seu próprio método `toString()`, e se você usa esse objeto em um tipo `string`, o `toString()` é o que será chamado automaticamente, e o resultado da `string` dessa chamada é o que vai ser usado no lugar.

**Observação:** A forma que um objeto é convertido em uma `string` tecnicamente passa através da operação abstrata `toPrimitive` (seção 9.1 da especificação ES5), mas essas nuances serão abordadas com mais detalhes na seção `ToNumber`, mais tarde nesse capítulo, então vamos pulá-lo aqui.

Arrays têm um padrão `toString()` substituído que stringfica a (string) concatenação de todos esses valores (cada um stringficando a si mesmo), com `","` entre cada valor:

```js
var a = [1,2,3];

a.toString(); // "1,2,3"
```

Novamente, `toString()` pode tanto ser chamada explicitamente, ou ela vai ser chamada automaticamente se uma não-`string` for usada em um contexto de `string`.

#### Stringficação do JSON

Outra tarefa que parece terrível relacionada a `ToString` é quando você usa a funcionalidade  `JSON.stringify(..)` para serializar um valor para um valor `string` compatível com JSON.

É importante notar que essa stringficação não é exatamente a mesma que a coerção. Mas como ela é relacionada às regras de `ToString` acima, nós faremos uma leve diversificação para abordar os comportamentos de stringficação JSON aqui.

Por mais simples que sejam o valores, a stringficação JSON se comporta basicamente da mesma forma que conversões `toString()`, exceto que a serialização resulta *sempre como uma `string`*:

```js
JSON.stringify( 42 );	// "42"
JSON.stringify( "42" );	// ""42"" (uma string com um valor dentro de aspas)
JSON.stringify( null );	// "null"
JSON.stringify( true );	// "true"
```

Qualuqer valor *seguro para JSON* pode ser stringficada com `JSON.stringify(..)`. Mas o que é *seguro para JSON (JSON-safe)* ? Qualquer valor que pode ser representado em uma representação JSON válida.

Pode ser mais fácil considerar valores que **não** são seguros para JSON. Alguns exemplos: `undefined`s, `function`s, (ES6+) `symbol`s, e `object`s com referências circulares (onde as referências de propriedade em uma estrutura de objeto criam um ciclo interminável entre si). Todos esses são valores ilegais para uma estrutura JSON padrão, principalmente porque ela não têm portabilidade para outras linguagens que consumem valores JSON.

A funcionalidade `JSON.stringify(..)` vai omitir automaticamente valores `undefined`, `function` e `symbol` quando cruzar com eles. Se o valor em questão for encontrado em um `array`, esse valor é substituído por `null` (então aquela posição da informação do array não é alterada). Se for encontrado como propriedade de um objeto, essa propriedade vai simplesmente ser excluída.

Considere:

```js
JSON.stringify( undefined );					// undefined
JSON.stringify( function(){} );					// undefined

JSON.stringify( [1,undefined,function(){},4] );	// "[1,null,null,4]"
JSON.stringify( { a:2, b:function(){} } );		// "{"a":2}"
```

Mas se você tentar fazer um `JSON.stringify(..)` em um `object` com referência(s) circular nele, um erro vai ser lançado.

Stringficação JSON tem um comportamente especial, que se o valor de um `object` tem um método `toJSON()` definido, esse método vai ser chamado primeiro para pegar um valor a ser usado para serialização.

Você tem a intenção de stringficar um objeto JSON que pode conter valores JSON ilegais, ou se você apenas têm, valores no `object` que não são apropriados para a serialização, você deveria definir um método `toJSON()`para que ele retorne à uma versão *segura para JSON* do `object`.

Por exemplo:

```js
var o = { };

var a = {
	b: 42,
	c: o,
	d: function(){}
};

// cria uma referência circular dentro de `a`
o.e = a;

// vai lançar um erro na referência circular
// JSON.stringify( a );

// define um valor de serialização JSON personalizado
a.toJSON = function() {
	// apenas inclui a propriedade `b` para serialização
	return { b: this.b };
};

JSON.stringify( a ); // "{"b":42}"
```

É bem comum o equívoco que `toJSON()` deveria retornar uma representação stringficada de JSON. Isso está provavelmente incorreto, a menos que você queira realmente stringficar a própria `string` (geralmente não!). `toJSON()` deve retornar o valor regular atual (de qualquer tipo) seria apropriado, e o próprio `JSON.stringify(..)` vai manipular a stringficação.

Em outras palavras, `toJSON()` deve ser interpretado como "adequado para stringficação para um valor seguro para JSON", não "para uma string JSON" como muitos desenvolvedores assumem errôneamente.

Considere:

```js
var a = {
	val: [1,2,3],

	// provavelmente correto!
	toJSON: function(){
		return this.val.slice( 1 );
	}
};

var b = {
	val: [1,2,3],

	// provavelmente incorreto!
	toJSON: function(){
		return "[" +
			this.val.slice( 1 ).join() +
		"]";
	}
};

JSON.stringify( a ); // "[2,3]"

JSON.stringify( b ); // ""[2,3]""
```

Na segunda chamada, nós stringficamos o retorno `string` ao invés do próprio `array`, o que provavelmente não é o que queríamos fazer.

Enquanto estamos falando de `JSON.stringify(..)`, vamos discutor algumas funcionalidades pouco conhecidas que continuam a ser bem úteis.

Um segundo argumento opcional pode ser passado para `JSON.stringify(..)` que é chamado *substituto (replacer)*. Esse argumento pode tanto ser um `array` ou uma `function`. É usado para personalizar a serialização recursiva de um `object` fornecendo um mecanismo de filtro no qual propriedades podem ou não serem incluídas, em uma maneira similar de como o `toJSON()` pode preparar um valor para serialização.

Se um *substituto* é um `array`, ele deve ser um `array` de `strings`, no qual cada um vai especificar uma nome de propriedade que é permitida para ser incluída na serialização do `object`. Se uma propriedade que existe não está nessa lista, ela será ignorada.

Se o *substituto* é uma `function`, ele será chamado uma vez pelo próprio `object`, e então uma vez para cada propriedade no `object`, e cada vez que ele passar dois argumentos, *chave* e *valor*. Para ignorar uma *chave* na serialização, retorne `undefined`. Do contrário, retorne o *valor* fornecido.

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"

JSON.stringify( a, function(k,v){
	if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"
```

**Observação:** No caso do *substituto* da `function`, o argumento chave `k` é `undefined` na primeira chamada (onde o próprio objeto `a` está sendo passado). A declaração `if` **filtra** a propriedade nomeada de `"c"`. Stringficação é recursiva, então o array `[1,2,3]` tem cada um dos seus valores (`1`, `2`, e `3`) passados como `v` para o *substituto*, com índices (`0`, `1`, and `2`) como `k`.

Um terceiro argumento opcional também pode ser passado para `JSON.stringify(..)`, chamado *espaço (space)*, no qual é usado como indentação para deixar a saída mais bonita e amigável. *espaço* pode ser um intermediador positivo para indicar quantos espaços de caracteres devem ser usados em cada nível de identação. Ou, *espaço* pode ser uma `string`, que nesse caso até os primeiros dez caracteres do seu valor serão usados para cada nível de identação.

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, null, 3 );
// "{
//    "b": 42,
//    "c": "42",
//    "d": [
//       1,
//       2,
//       3
//    ]
// }"

JSON.stringify( a, null, "-----" );
// "{
// -----"b": 42,
// -----"c": "42",
// -----"d": [
// ----------1,
// ----------2,
// ----------3
// -----]
// }"
```

Lembre-se, `JSON.stringify(..)` não é uma forma direta de coerção. Nós o abordamos aqui, porém, por duas razões que seu comportamento está relacionado com coerção `ToString`:

1. Valores `string`, `number`, `boolean`, e `null` todos podem ser stringficados para JSON basicamente o mesmo como a forma que eles convertem valores `string` através das regras da operação abstrata `ToString`.
2. Se você passou o uma valor de `object` para `JSON.stringify(..)`, e esse `object` tem um método `toJSON()` nele, `toJSON()` é chamado automaticamente para (tipo que) "converter" o valor para ser *seguro para JSON* antes da stringficação.

### `ToNumber`

Se qualquer valor não-`number` é usado de uma forma que exige que seja um `number`, como uma operação matemática, a especificação ES5 define, na seção 9.3, a operação abstrata `ToNumber`.

Por exemplo, `true` torna-se `1` e `false` torna-se `0`. `undefined` torna-se `NaN`, mas (curiosamente) `null` torna-se `0`.

`ToNumber` para valor `string` essencialmente funciona para a maioria das partes como regras/sintaxe para numéricos literais (Veja o Capítulo 3). Se isso falhar, o resultado é `NaN` (ao invés de um erro de sintaxe com `numbers` literais). Um exemplo da diferença é que `0`-números octais pré fixados não são manipulados como octais (apenas como decimais normais) nessa operação, portanto esses octais são válidos como `numbers` literais (veja o Capítulo 2).

**Observação:** As diferenças entre a gramática de `number` literal  e `ToNumber` em um valor de uma `string` são sutis e altamente matizados, e por isso não serão mais abordados aqui. Consulte a seção 9.3.1 da especificação ES5 para mais informações.

Objetos (e arrays) vão primeiro ser convertidos para seus valores primitivos equivalentes, e o valor resultado (se for primitivo mas ainda não um `number`) é convertido para um `number` de acordo com as regras de `ToNumber` mencionadas.

Para converter para seu valor primitivo equivalente, a operação abstrata`ToPrimitive` (seção 9.1 da especificação ES5) irá consultar o valor (usando a operação interna `DefaultValue` -- seção 8.12.8 da especificação ES5) em questão para ver se ele tem um método `valueOf()`. Se o `valueOf()` estiver disponível e ele retornar um valor primitivo, *aquele* valor é usado para coerção. Do contrário, mas se `toString()` está disponível, ele vai fornecer o valor para a coerção.

Se nenhuma das operações pode fornecer um valor primitivo, um `TypeError` é lançado.

A partir de ES5, você pode criar certos objetos não coercivos -- um sem `valueOf()` e `toString()` -- se ele tiver um valor `null` para seu `[[Prototype]]`, geralmente criado com `Object.create(null)`. Veja o título *this & Object Prototypes* desa série para mais informações de `[[Prototype]]`s.

**Observação:** Nós abordamos como converter para `number`s em detalhes mais tarde nesse capítulo, mas para esse próximo trecho de código, apenas suponha que a função `Number(..)` faz isso.

Considere:

```js
var a = {
	valueOf: function(){
		return "42";
	}
};

var b = {
	toString: function(){
		return "42";
	}
};

var c = [4,2];
c.toString = function(){
	return this.join( "" );	// "42"
};

Number( a );			// 42
Number( b );			// 42
Number( c );			// 42
Number( "" );			// 0
Number( [] );			// 0
Number( [ "abc" ] );	// NaN
```

### `ToBoolean`

A seguir, vamos ter uma pequena conversa sobre como `boolean`s se comportam em JS. Há **muita confusão e equívoco** em torno desse tópico, então preste bastante atenção!

Em primeiro lugar, JS tem as palavras-chave atuais `true` e `false`, e elas se comportam exatamente como você esperaria de valores `boolean`. É um equívoco comum que os valores `1` e `0` sejam idênticos à `true/false`. Enquanto isso pode ser verdadeiro em outras linguagens, em JS os `number`s são `number`s e os `boolean`s são `boolean`s. Você pode converter `1` para `true` (e vice-versa) ou `0` para `false` (e vice versa). Mas eles não são os mesmos.

#### Valores Falsos (Falsy)

Mas esse não é o fim da história. Nós precisamos discutir como outros valores além dos dois `boolean`s se comportam independentemente de você converter *para* seus equivalentes `boolean`.

Todos os valores JavaScript podem ser divididos em duas categorias:

1. Valores que irão se tornar `false` se convertidos para `boolean`
2. Todo o resto (o que vai obviamente se tornar `true`)

Eu não estou apenas sendo engraçado. A especificação JS define uma específica e estreita lista dos valores que poderão tornar-se `false` quando convertidos para um valor `boolean`.

Como sabemos qual é essa lista de valores? Na seção 9.2 da especificação ES5, é definido uma operação abstrata `ToBoolean`, na que diz exatamente o que aconteceria para todos os valores possíveis quando você tenta converte-los "para boolean".

A partir dessa tabela, obtemos o seguinte da chamada lista de valores "falsos":

* `undefined`
* `null`
* `false`
* `+0`, `-0`, e `NaN`
* `""`

É isso. Se um valor não está nessa lista, é um valor "falso" (falsy), e ele não vai ser convertido para `false` se você forçar uma coerção `boolean` nele.

Por conclusão lógica, se um valor *não* está nessa lista, ele deve estar em *outra lista*, na qual nós chamamos de lista de valores "verdadeiros". Mas o JS realmente não define uma lista de valores "verdadeiros" por si só. Ele dá alguns exemplos, assim como dizemos explicitamente que todos os objetos são verdadeiros, mas principalmente a especificação apenas implica que: **qualquer coisa que não esteja explicitamente na lista falsa, é portanto, verdadeira.**

#### Objetos Falsos (Falsy Objects)

Espere um minuto, aquele títulos de seção soa até contraditório. Eu *apenas disse* literalmente que a especificação chama todos os objetos de verdadeiro, certo? Não deveria existir tal coisa como um "objeto falso".

O que isso possivelmente pode significar?

Você deve estar tentado a pensar que isso significa um *object wrapper* (veja o capítulo 3) em torno de um valor falso (como `""`, `0` ou `false`). Mas não caia nessa *armadilha*.

**Observação** 

Wait a minute, that section title even sounds contradictory. I literally *just said* the spec calls all objects truthy, right? There should be no such thing as a "falsy object."

What could that possibly even mean?

You might be tempted to think it means an object wrapper (see Chapter 3) around a falsy value (such as `""`, `0` or `false`). But don't fall into that *trap*.

Essa é uma piada de especificação sutil que alguns de vocês podem ter sacado.

Considere:

```js
var a = new Boolean( false );
var b = new Number( 0 );
var c = new String( "" );
```

Nós sabemos que todos os três valores são *objects wraper* (veja o capítulo 3) em torno de valores obviamente falsos. Mas esses objetos se comportam como `true` ou como `false`? Essa é fácil de responder:

```js
var d = Boolean( a && b && c );

d; // true
```

Então, todos os três se comportam como `true`, como essa é a única maneira de `d` acabar como `true`.

**Dica:** Note que o wrapped `Boolean(..)` está em torno da expressão `a && b && c` -- você deve estar se perguntando porque isso está ali. Nós vamos voltar mais tarde nesse capítulo, então faça um nota mental disso. Para um pequeno exercício, procure por si mesmo o que `d` será se você apenas fizer `d = a && b && c` sem a chamada `Boolean(..)`!

Então, se "objetos falsos" **não são apenas objetos embrulhados em torno de valores falsos**, o que diabos eles são?

A parte complicada é que eles podem aparecer no seu programa JS, mas eles na verdade não são parte do próprio JavaScript.

**O quê?!**

Há certos casos em que navegadores criam seus próprios tipos de comportamentos *exóticos* de valores, nomeando essa ideia de "objetos falsos", no topo da semântica regular do JS.

Um "objeto falso" é um valor que parece e age como um objeto normal (propriedades, etc.), mas quando você converte eles para um `boolean`, ele faz a coerção para um valor `false`.

**Por quê?!**

O caso mais conhecido é `document.all`: um tipo array (objeto) fornecido pelo seu programa JS *pelo DOM* (não pelo próprio motor JS), que expõe elementos na sua página para seu programa JS. Ele *costuma* se comportar como um objeto normal -- isso seria verdadeiro. Mas não mais.

O próprio `document.all` nunca foi realmente "padrão" e há muito tempo ficou obsoleto/abandonado.

"Eles não podem apenas remover isso então?" Desculpe, boa tentativa. Gostaria que pudessem. Mas há muita base de código JS legado por aí que dependem desse uso.

Então, porque fazer ele agir como falso? Porque coerções de `document.all` para `boolean` (assim como nas declarações `if`) foram quase sempre usadas como um meio de detectar o IE antigo e não padronizado.

O IE há muito tempo vem se aproximando dos padrões e, em muitos casos, vem empurrando a Web para frente tanto ou mais do que qualquer outro navegador. Mas todos aqueles códigos `if` antigos (document.all){ /* it's IE */ }` antigos contiuam por aí, e muito deles, provavelmente, nunca irão embora. Todos esses códigos legados continuam supondo que estão sendo executados em IE antigos, o que leva a más experiências de navegação para usuários IE.

Então, nós não podemos remover `document.all` completamente, mas o IE não quer que códigos `if (document.all) { .. }` funcionem mais, então esses usuários em IE modernos terão novas lógicas de código compatível com os padrões.

"O que devemos fazer?" **"Já sei! Vamos degradar o sistema de tipo do JS e fingir que `document.all` é falso!"

Eca. Isso é uma merda. É um macete louco que a maioria dos desenvolvedores não entendem. Mas a alternativa (fazer nada sobre os problemas sem solução acima) fede *um pouquinho mais*.

Então...é isso que temos: "objetos falsos" loucos e fora do padrão adicionados ao JS pelos navegadores. Ebaa!

#### Valores verdadeiros (truthy)

De volta para a lista verdadeira. O que exatamente são valores verdadeiros? Lembre-se: ** um valor é verdadeiro se ele não está na lista falsa **.

Considere:

```js
var a = "false";
var b = "0";
var c = "''";

var d = Boolean( a && b && c );

d;
```

Qual valor você espera que `d` tenha aqui? Deve ser ou `true` ou `false`.

É `true`. Porquê? Porque apesar dos conteúdos dos valores daquelas `string` parecerem como valores falsos, os próprios valores da `string` são verdadeiros, porque `""` é o único valor de `string` na lista falsa.

E esses?

```js
var a = [];				// array vazio -- verdadeiro ou falso?
var b = {};				// object vazio --  verdadeiro ou falso?
var c = function(){};	// function vazio --  verdadeiro ou falso?

var d = Boolean( a && b && c );

d;
```

Sim, você acertou, `d` continua `true` aqui. Porque? Mesma razão de antes. Apesar do que possa parecer, `[]`, `{}`, e `function(){}` *não* estão na lista falsa, e, portanto, são valores verdadeiros.

Em outras palavras, a lista de verdadeiros é infinitamente longa. É impossível de fazer tal lista. Você apenas pode fazer uma lista falsa e *cosultá-la*.

Pegue cinco minutos, escreva a lista falsa em um post-it para o monitor do seu computador, ou memorize-a se preferir. De qualquer forma, você facilmente poderá construir uma lista falsa virtual sempre que precisar simplesmente perguntando se está na lista falsa ou não.

A importância de verdadeiro e falso no entendimento de como um valor vai se comportar quando você coverte-lo (explicitamente ou implicitamente) para uma valor `boolean`. Agora que você tem essas duas listas em mente, nós podemos mergulhar nos exemplos de coerção.

## Coerção Explícita

Coerção *explícita* refere-se à conversões de tipos que são óbvias e explícitas. Há uma ampla variedade de uso de conversões de tipo que caeem na categoria de coerção *explícita* para a maioria dos desenvolvedores.

O objetivo aqui é indetificar padrões em seu código que nós podemos deixar claro e óbvio que nós estamos convertendo um valor de um tipo para o outro, para não deixar buracos para os futuros desenvolvedores tropeçarem. Quanto mais explícitos somos, mais provável que alguém mais tarde consiga ler nosso código e entender sem esforço excessivo qual era a nossa intenção.

Seria difícil encontrar quaisquer discordâncias salientes com coerção *explícita*, uma vez que se alinha mais de perto com o modo como a prática comumente aceita de conversão de tipos funciona em linguagens com tipagem estática. Como tal, vamos tomar como certo (por enquanto) que a coerção *explícita* pode ser aceita como não maligna ou controversa. No entando, nós vamos revisitar isso mais tarde.

### Explicitamente: Strings <--> Numbers

Nós vamos começar com a operação de coerção mais simples, e talvez, mais comum: converter valores entre uma representação `string` e `number`.

Para converter `string`s em `number`s,  nós usamos as funções nativas String(..)` e `Number(..)` (nas quais nos referimos no Capítulo 3 como "construtores nativos"), mas **muito importante**, nós não usamos a palavra-chave `new` na frente delas. Assim sendo, não estamos criando object wrappers.

Em vez disso, nós na verdade criamos uma *coerção explícita* entre os dois tipos:

```js
var a = 42;
var b = String( a );

var c = "3.14";
var d = Number( c );

b; // "42"
d; // 3.14
```

`String(..)` converte qualquer outro valor para um valor primitivo `string`, usando as regras da operação `ToString` discutida anteriormente. `Number(..)` coverte qualquer outro valor para um valor primitivo `number`, usando as regras da operação `ToNumber` discutida anteriormente.

Eu chamo isso de coerção *explícita* porque em geral, é bastante óbvio para a maioria dos desenvolvedores que o resultado final dessas operações é a aplicação da conversão de tipo.

Na verdade, a forma como isso é usado, realmente parece muito em como isso é feito em algumas outras linguagem de tipagem estática.

Por exemplo, em C/C++, você pode dizer tanto `(int)x` ou `int(x)`, que ambas irão converter o valor em `x` para um número inteiro. Ambas formas são válidas, mas muitos preferem a última, que meio que parece a chamda de uma função. No JavaScript, quando você diz `Number(x)`, isso parece terrivelmente igual. Importa se isso é *de verdade* uma chamada de função no JS? Na verdade, não.

Além de `String(..)` e `Number(..)`, há outras formas de converter "explicitamente" esses valores entre `string` e `number`:

```js
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c;

b; // "42"
d; // 3.14
```

Chamar `a.toString()` é ostensivamente explícito (está bem claro que "toString" significa "para uma string"), mas existe alguma implicidade. `toString()` não pode ser chamado em um valor *primitivo* como `42`. Então o JS automaticamente "encaixota" (veja o Capítulo 3) `42` em um object wrapper, então `toString()` pode ser chamada contra o objeto. Em outras palavras, você pode chamar isso de "explicitamente implícito".

Aqui o `+c` esta mostrando o *operador unário* (operador com somente um operando) do operador `+`. Em vez de realizar uma adição matemática (ou concatenação de string -- veja abaixo), o unário `+` converte explicitamente seu operando (`c`) em um valor `number`.

`c+` é uma coerção explícita? Depende da sua experiêcnia e perspectiva. Se você sabe (e voce, agora, sabe!) que o unário `+` é explicitamente destinado para uma coerção `number`, então é bem explícito e óbvio. Porém, se você nunca viu isso antes, isso pode ser terrivelmente confuso, implícito, com efeitos colaterais ocultos, etc.

**Observação:** A perspectiva geralmente aceita na comunidade open source JS pe que o unário `+` é uma forma aceita de coerção *explícita*.

Mesmo se você realmente goste da forma `+c`, há, definitivamente, alguns lugares onde isso pode parecer terrivelmente confuso, Considere:

```js
var c = "3.14";
var d = 5+ +c;

d; // 8.14
```

O operador unário `-` também converte da mesma forma que o `+` faz, mas ele também inverte o sinal do número. No entanto, você não pode colocar dois `--` próximo um do outro para desvirar o sinal, como isso é feito com o operador de decremnto. Em vez disso, você vai precisar fazer: `- -"3.14"` com espaço no meio, e isso vai resultar na coerção para `3.14`.

Você provavelmente pode inventar todo o tipo de combinações horríveis de operadores binários (como `+` para adição) ao lado da forma  unária de um operador. Aqui está outro exemplo maluco:

```js
1 + - + + + - + 1;	// 2
```

Você deve considerar fortemente evitar corção unária com `+` (ou `-`) quando ela é imediatamentamenta adjacente de outro operador. Enquando o de cima funciona, é quase que universalmente considerado uma má ideia. Mesmo `d = +c` (ou `d =+ c` também!) pode muito facilmente ser confudido com `d += c`, o que é inteiramente diferente!

**Observação:** Outro lugar extremamente confuso para usar o unário `+` é adjacente à outro operador que pode ser, `++` operador de incremento e `--` operador de decremento. Por exemplo: `a +++b`, `a + ++b`, and `a + + +b`. Veja "Efeitos colaterais de expressões" no Capítulo 5 para mais sobre `++`.

Lembre-se, nós estamos tentando ser explícitos e **reduzir** a confusão, não torná-la muito pior!

#### `Date` para `number`

Outro uso comum do operador unário `+` é converter um objeto `Date` para um `number`, porque o resultado é um valor timestamp unix (milisegundos passados desde 1 de Janeiro 1970 1970 00:00:00 UTC) que é a representação da data/hora.

```js
var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );

+d; // 1408369986000
```

O uso mais comum dessa representação é pegar o tempo *atual* como uma timestamp, assim:

```js
var timestamp = +new Date();
```

**Nota:** Alguns desenvolvedores estão cientes do "truque" sintático peculiar no JavaScript, é que o `()` definido em uma chamada de construtor (uma função chamada com `new`) é *opcional* se não há argumentos para passar. Então você pode se deparar com a forma `var timestamp = +new Date;`. No entanto, nem todos os desenvolvedores concordam que omitir o `()` melhora a legibilidade, pois é uma exceção de sintaxe incomum que se aplica apenas a forma de chamada `new fn()` e não a forma de chamada tradicional `fn()`.

Mas coerção não é a única forma de pegar o timestamp de uma objeto `Date`. A abordagem de não coerção talvez seja preferível, já que é mais explícita:

```js
var timestamp = new Date().getTime();
// var timestamp = (new Date()).getTime();
// var timestamp = (new Date).getTime();
```

Mas uma opção de não coerção *ainda mais* preferível é usar a função estática adicionada no ES5 `Date.now()`:

```js
var timestamp = Date.now();
```

E se você quiser fazer o pollyfill de `Date.now()` para navegadores antigos, é bem simples:

```js
if (!Date.now) {
	Date.now = function() {
		return +new Date();
	};
}
```

Eu recomendaria pular as formas de coerção relacionadas à datas. Use `Date.now()` para timestamps *atuais* e `new Date(..).getTime()` para pergar um timestamp para uma data/hora *não atual* específica que você precise.

#### O curioso caso do `~`

Um operador coercivo JS que é frequentemente negligenciado e geralmente muito confudido é o operador til `~` (também conhecido como "operador bit a bit NOT"). Muitos dos que até compreendem o que ele faz, vão muitas vezes continuar a evitá-lo. Mas se mantendo no espírito do nossa abordagem nesse livro e série, vamos cavar isso e descobrir se o `~` tem algo de útil para nos dar.

Na seção "inteiros de 32-bit (signed)" do Capítulo 2, nós abordamos como operadores bit a bit em JS são definidos apenas por operações de 32-bit, o que sifnifica que eles forçam seus operando a entrarem em conformidade com representações de valores 32-bit. As regras para como isso acontece são controladas pela operação abstrata `ToInt32` (Especificação ES5, seção 9.5).

`ToInt32` primeiro faz uma coerção para um `ToNumber`, o que significa que se o valor é `"123"`, ele vai primeiro se tornar `123` antes das regras de `ToInt32` serem aplicadas.

Enquanto não é *tecnicamente* uma coerção em si (desde que o type não mude!), o uso de operadores bit a bit (como `|` ou `~`) com um certo valor `number` especial produz um efeito coercivo que resulta em um valor `number` diferente.

Por exemplo, primeiro vamos considerar o `|` "operador bit a bit OU" usado de outra forma em um idioma não-op `0 | x`, que (como o capítulo 2 mostrou) essencialmente apenas faz a conversão `ToInt32`:

```js
0 | -0;			// 0
0 | NaN;		// 0
0 | Infinity;	// 0
0 | -Infinity;	// 0
```

Esse números especiais não são representações 32-bit (desde que eles venham do padrão 64-bit IEEE 754 -- veja o Capítulo 2), então `ToInt32` apenas especifica `0` como resultado para esses valores.

É discutível se `0 | __` é uma forma *explícita* dessa operação coerciva `ToInt32` ou se ela é mais *implícita*. Pela perspectiva da especificação, é inquestionavelmente *explícita*, mas se você não compreende operações bit a bit nesse nível, isso pode parecer uma mágica mais *implícita*. No entanto, de acordo com outras afirmações nesse capítulo, nós vamos chamá-la de *explícita*.

Então vamos voltar nossa atenção para o `~`. O operador `~` primeiro faz a "coerção" para um valor `number` de 32-bit , e então realiza uma negativa bit a bit (lançando a paridade de cada bit).

**Observação:** Isso é bem similar em como `!` não apenas faz a coerção de seus valores para `boolean` mas também lança sua paridade (veja a discussão dos "unários `!`" depois).

Mas...o quê!? Porquê nos importamos com bits sendo lançados? Isso é algo bem específico, algo com muitas nuances. É bem raro que os desenvolvedores JS precisem raciocinar sobre bits individuais.

Outra forma de pensar sobre a definição de `~` vem da ciência da computação/Matemática old-school: `~` realiza dois complementos. Ótimo, obrigado, isso está totalmente claro!

Vamos tentar de novo: `~x` é aproximadamente o mesmo que `-(x+1)`. Isso é estranho, mas um pouco mais fácil de racionalizar. Então:

```js
~42;	// -(42+1) ==> -43
```

Você provavelmente continua imaginando o que diabos é toda essa coisa com o `~`, ou porque isso realmente importa para uma discussão sobre coerção. Vamos chegar ao ponto rapidamente.

Considere `-(x+1)`. Qual é o único valor que você pode realizar essa operação no qual ele irá produzir um resultado `0` (ou, tecnicamente, `-0`)? `-1`. Em outras palavras, `~` usado com uma gama de valores `number` produzirá um valor falso (facilmente coercível para `false`) `0` para o valor de entrada `-1`, e, de outra forma, para qualquer outro valor verdadeiro.

Por que isso é relevante?

`-1` é comumente chamado de um "sentinel value", o que basicamente significa um valor no qual é dado um significado semântico arbitrário dentro do conjunto maior de valores do primeiro tipo (`number`s). A linguagem C usa o sentinel value `-1` para muitas funções que retornam valores `>=0` para "sucesso" e `-1` para "falha".

O JavaScript adotou esse precedente ao definir a operação `string` de `indexOf(..)`, que busca por uma substring e, se encontrada, retorna sua posição de índice inicial, ou `-1` se não encontrada.

É bem comum tentar usar `indexOf(..)` não apenas como uma operação para pegar a posição, mas como uma verificação `boolean` que verifica a presença/ausência de uma sibstring em outra `string`. Veja agora como como desenvolvedores realizam essas verificações:

```js
var a = "Hello World";

if (a.indexOf( "lo" ) >= 0) {	// true
	// encontrado!
}
if (a.indexOf( "lo" ) != -1) {	// true
	// encontrado!
}

if (a.indexOf( "ol" ) < 0) {	// true
	// não encontrado!
}
if (a.indexOf( "ol" ) == -1) {	// true
	// não encontrado!
}
```

Eu acho um pouco grosseiro olhar para `>= 0` ou `== -1`. É basicamente uma "abstração vazada", na medida em que está vazando o comportamento de implementação subjacente -- o uso da sentinela `-1` para "falha" -- no meu código. Eu preferiria esconder tal detahe.

E agora, finalmente, nós vemos porque `~` pode nos ajudar! Usar `~` com `indexOf()` realiza a "coerção" (na verdade apenas transforma) o valor **para ser um `boolean` apropriado para coerção**:

```js
var a = "Hello World";

~a.indexOf( "lo" );			// -4   <-- verdadeiro!

if (~a.indexOf( "lo" )) {	// true
	// encontrado!
}

~a.indexOf( "ol" );			// 0    <-- falso!
!~a.indexOf( "ol" );		// true

if (!~a.indexOf( "ol" )) {	// true
	// não encontrado!
}
```

`~` pega o valor retornado de `indexOf(..)` e o transforma: em caso de "falha" `-1` nós teremos o falso `0`, e todo outro valor é verdadeiro.

**Observação:** O pseudo-algoritmo `-(x+1)` para `~` implicaria que `~-1` é `-0`, mas na verdade ele produz `0` porque a operação subjacente é na verdade bit a bit, não matemática.

Tecnicamente, `if (~a.indexOf(..))` ainda está confiando na coerção *implícita* da resultante `0` para `false` ou diferente de zero para `true`. Mas no geral, `~` ainda me parece mais como um mecanismo de coerção *explícita*, desde que você saiba o que pretende fazer nessa linguagem.

Eu acho esse é um código mais limpo do que o desorganizado `>= 0` / `== -1`.

##### Truncando bits

Há mais um lugar que `~` pode aparecer em um código: alguns desenvolvedores usam o til duplo `~~` para truncar a parte decimal de um `number` (aplicar "coerção" para um número "inteiro"). É comum (embora erroneamente) dizer que este é o mesmo resultado que chamar `Math.floor(..)`.

Como `~~` funciona, é que o primeiro `~` aplica a coerção `ToInt32` e faz o lançamento do bit e, em seguida, o segundo` ~ `faz outro lançamento de bit a bit, folheando todos os bits de volta para o estado original. O resultado final é apenas a "coerção" `ToInt32` (também conhecida como truncamento).

**Observação:** O lançamento bit a bit duplo de `~~` é muito parecido com o comportamento de paridade da negativa dupla `!!`, explicada mais tarde na seção "Explicitamente: * --> Boolean.

Porém, `~~` precisa de algum cuidado/esclarecimento. Primeiro, ele apenas funciona dependente de valores em 32-bit. Mas, mais importante, ele não funciona da mesma forma em números negativos como o `Math.floor(..)` faz!

```js
Math.floor( -49.6 );	// -50
~~-49.6;				// -49
```

Definindo o `Math.floor(..)`, apesar das diferenças, `~~x` pode truncar para um inteiro (32-bit). Mas o `x | 0` também faz, e aparentemente com (ligeiramente) *menos esforço*.

Então, porque você escolheria `~~x` em vez de `x | 0`? Precedência de operador (veja o Capítulo 5):

```js
~~1E20 / 10;		// 166199296

1E20 | 0 / 10;		// 1661992960
(1E20 | 0) / 10;	// 166199296
```

Assim como todos os outros conselhos aqui, use `~` e `~~`` como mecanismos explícitos para "coerção" e transformação de valores somente se todos que lêem/escrevem o código em questão estão propriamente cientes de como esses operadores funcionam!

### Explicitamente: Parseando strings numéricas

Um resultado semelhante para coagir uma `string` para um` number` pode ser conseguido parseando um `number` de um conteúdo de caracteres de uma `string`. Há no entanto, diferenças distintas entre esse parseamento e a conversão de tipo que examinamos acima. 

Considere:

```js
var a = "42";
var b = "42px";

Number( a );	// 42
parseInt( a );	// 42

Number( b );	// NaN
parseInt( b );	// 42
```

Parsear um valor numérico de uma string é *tolerante* para caracteres não numéricos -- isso apenas para de parsear da esquerda para a direita quando encontrado -- enquanto a coerção é *não tolerante* e falha, resultando no valor 'NaN`.

Parseamento não deve ser visto como um substituto para coerção. Essas duas tarefas, mesmo similares, têm propósitos diferentes. Realize o parser de uma `string` como um `number` quando você não sabe/se importa com outros caracteres não numéricos que possam existir no lado direito. Fazer a coerção de um `string` (para um `number`) quando os únicos valores aceitáveis são numéricos e algo como "42px" deve ser rejeitado como um `number`.

**Dica:** `parseInt(..)` tem um irmão gêmeo, `parseFloat(..)`, que (como parece) tira um número de ponto flutuante de uma string.

Não esqueça que `parseInt(..)` opera em valores `string`. Não faz absolutamente nenhum sentido passar um valor `number` para `parseInt(..)`. Nem faria sentido passar nenhum outro tipo de valor, como `true`, `function(){..}` ou `[1,2,3]`.

Se você passar uma não `string`, o valor que você passar vai automaticamente sofrer coerção para uma `string` primeiro (veja "`ToString`" anteriormente), o que vai claramente ser um tipo de coerção *implícita* oculta. É realmente uma péssima ideia confiar em tal comportamento no seu programa, então nunca use `parseInt(..)` em um valor que não seja uma `string`.

Antes do ES5, outra pegadinha existia com `parseInt(..)`, a qual era fonte de muitos bugs de programas JS. Se você não passasse um segundo argumento para indicar qual base numérica (conhecida como radix) usar para interpretar o conteúdo numérico da `string`, `parseInt(..)` iria olhar para os caracteres iniciais e adivinhar.

Se os primeiros dois caracteres fossem `"0x"` ou `"0X"`, o palpite (por convenção) era que você queria interpretar a `string` como um `number` de base hexadecimal(base-16). Por outro lado, se o primeiro caractere fosse `"0"`, o palpite (novamente, por convenção) era que você queria interpretar a `string` como um `number` de base octal (base-8).

`string`s hexadecimais (com iniciais `0x` ou `0X`) não são extremamente fáceis de se misturar. Mas a adivinhação do número octal mostrou-se diabolicamente comum. Por exemplo:

```js
var hour = parseInt( selectedHour.value );
var minute = parseInt( selectedMinute.value );

console.log( "The time you selected was: " + hour + ":" + minute);
```

Parece inofensivo, certo? Tente selecionar `08` para hora e `09` para os minutos. Você vai ter `0:0`. Por quê? porque nem `8` nem `9` são caracteres válidos em octais base-8.

A correção pré-ES5 foi simples, mas muito fácil de esquecer: **sempre passar `10` como o segundo argumento**. Isso era totalmente seguro:

```js
var hour = parseInt( selectedHour.value, 10 );
var minute = parseInt( selectedMiniute.value, 10 );
```

A partir da ES5, `parseInt(..)` não adivinhava mais octais. A menos que você diga o contrário, ele supõe caracteres base-10 (ou base-16 para prefixos `"0"`). Isso é muito melhor. Apenas tenha cuidado se seu código tenha que rodar em ambientes pré-ES5, que nesse caso você ainda vai precisar passar `10` para o radix.

#### Parseando não-Strings

Um exemplo um pouco infame do comportamento do `parseInt(..)` é destacado em uma publicação com uma piada sarcástica alguns anos atrás, tirando sarro desse comportamento JS:

```js
parseInt( 1/0, 19 ); // 18
```
A afirmação pretensiosa (mas totalmente inválida) foi: "Se eu passar infinito e parsear um número inteiro disso, eu deveria recuperar o infinito, não 18." Certamente, JS deve estar louco por esse resultado, certo?

Embora este exemplo seja obviamente artificial e irreal, vamos entrar na loucura por um momento e examinar se JS realmente é tão louco.

Primeiramente, o pecado mais óbvio cometido aqui é passar uma não-`string` para `parseInt(..)`. Não, não, não. Faça isso e você estará pedindo por problemas. Mas mesmo se você fizer, o JS, educadamente, faz a coerção do que você passa em uma `string` que pode tentar parsear.

Alguns poderão argumentar que esse é um comportamento irracional, e que `parseInt(..)` deveria operar em um valor não-`string`. Isso deveria lançar um erro? Isso seria muito a cara do Java, francamente. Eu estremeço ao pensar que JS deveria começar a lançar erros em todo o lugar para que o `try..catch` seja necessário em quase todas as linhas.

Ele deveria retornar `NaN`? Talvez. Mas... que tal:

```js
parseInt( new String( "42") );
```

Isso deveria falhar também? É um valor não-`string`. Se você quer que o wrapper de objeto `String` seja desenpacotado para `"42"`, então é realmente tão incomum que o `42` se torne primeiro `"42"` para que `42` possa ser analisado de volta?

Eu argumentaria que essa coerção meio *explícita*, meio *implícita* que pode ocorrer pode ser uma coisa muito útil. Por Exemplo:

```js
var a = {
	num: 21,
	toString: function() { return String( this.num * 2 ); }
};

parseInt( a ); // 42
```

O fato de que `parseInt (...)` forçe a coerção de seu valor para um `string` para realizar um parse é bastante sensato. Se você passar lixo, e você receber lixo de volta, não culpe a lata de lixo -- ela só fez seu trabalho fielmente.

Então, se você passar um valor como `Infinity` (o resultado de `1 / 0` obviamente), que tipo de representação de `string` você faria mais sentido para essa coerção? Apenas duas escolhas racionais vêm à mente: `"Infinity"` and `"∞"`. O JS escolhe `"Infinity"`. E sou grato por ele escolher isso.

Eu acho que é uma coisa boa que **todos os valores** em JS tenham algum tipo de representação de `string` padrão, assim eles não são misteriosas caixas preta que nós não podemos debugar e pensar sobre.

Agora, e sobre caracteres base-19? Obviamente, completamente falso e artificial. Nenhum programa JS real usa base-19. É um absurdo. Mas, de novo, vamos curtir o ridículo. Em base-19, os caracteres numéricos válidos são `0` - `9` e `a` - `i` (case insensitive).

Então, de volta para nosso exemplo `parseInt( 1/0, 19 )`. Isso é essencialmente `parseInt( "Infinity", 19 )`. Como ele irá parsear? O primeiro caractere é o `"I"`, no qual é valor `18` na boba base-19. O segundo caractere `"n"` não está no conjuntos de caracteres válidos, e como tal, o parse simplesmente pára, assim como quando ele cruzar com `"p"` em `"42px"`.

O resultado? `18`. Exatamente como ele, sensatamente, deve ser. Os comportamentos envolvidos para nos trazer até aqui, e não para um próprio erro `Infinity`, são **muito importantes** para o JS, e não devem ser descartados tão facilmente.

Outros examplos desse comportamento com `parseInt(..)` que podem ser surpreendentes mas são bastante sensatos incluem:

```js
parseInt( 0.000008 );		// 0   ("0" de "0.000008")
parseInt( 0.0000008 );		// 8   ("8" de "8e-7")
parseInt( false, 16 );		// 250 ("fa" de "false")
parseInt( parseInt, 16 );	// 15  ("f" de "function..")

parseInt( "0x10" );			// 16
parseInt( "103", 2 );		// 2
```

Na verdade `parseInt(..)` é bem previsível e consistente em seu comportamento. Se voce usa-lo corretamente, você terá resultados sensatos. Se você usa-lo incorretamente, o resultado maluco que você terá não é culpa do JavaScript.

### Explicitamente: * --> Boolean

Agora, vamos examinar a coerção de qualquer outro valor não `boolean` para um `boolean`.

Assim como com `String(..)` e `Number(..)` acima, `Boolean(..)` (sem o `new`, claro!) é uma forma explícita de forçar a coerção `ToBoolean`:

```js
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

Boolean( a ); // true
Boolean( b ); // true
Boolean( c ); // true

Boolean( d ); // false
Boolean( e ); // false
Boolean( f ); // false
Boolean( g ); // false
```

Enquanto `Boolean(..)` é claramente explícita, ela não é tão comum ou idiomática.

Assim como o operador unário `+` faz a coerção de um valor para um `number` (veja acima), o operador unário de negação `!` faz a coerção explicitamente de um valor para um `boolean`. O *problema* é que ele também inverte o valor de verdadeiro para falso ou vice versa. Então a forma mais comum em que desenvolvedores JS fazem a coerção explícita para `boolean` é usando o operador de negação duplo `!!`, porque o segundo `!` vai inverter a paridade de volta à original:

```js
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

!!a;	// true
!!b;	// true
!!c;	// true

!!d;	// false
!!e;	// false
!!f;	// false
!!g;	// false
```

Qualquer uma dessas coerções `ToBoolean` podem acontecer *implicitamente* sem o `Boolean(..)` ou `!!`, se usado em um contexto `boolean` assim como uma declaração `if(..) ..`. Mas o objetivo é forçar explicitamente o valor para um `boolean` para deixar claro que a coerção `ToBoolean` é intencional.

Outro caso de uso para coerção explícita `ToBoolean` é se você quer forçar uma coerção de valor `true`/`false` em uma serialização JSON de uma estrutura de dados:

```js
var a = [
	1,
	function(){ /*..*/ },
	2,
	function(){ /*..*/ }
];

JSON.stringify( a ); // "[1,null,2,null]"

JSON.stringify( a, function(key,val){
	if (typeof val == "function") {
		// força a coerção `ToBoolean` da função
		return !!val;
	}
	else {
		return val;
	}
} );
// "[1,true,2,true]"
```

Se você veio para o JavaScript do Java, você deve reconhecer essa linguagem:

```js
var a = 42;

var b = a ? true : false;
```

O operador ternário `? :` vai testar `a` para verdadeiro, e baseado nesse teste atribuirá `true` ou `false` para `b`, em conformidade.

Nessa superfície, essa linguagem é uma forma *explícita* de coerção do tipo `ToBoolean`, uma vez que é óbvio que apenas `true` ou `false` saem da operação.

Porém, há uma coerção *implícita* oculta, aquela expressão `a` deve primeiro sofrer a coerção para `boolean` para executar o teste de verdade. Eu chamaria essa linguagem de "explicitamente implícita". Além disso, eu sugiro que **você evite essa linguagem completamente** no JavaScript. Ela não oferece benefícios reais, e pior, mascara algo que não é.

`Boolean(a)` e `!!a` são de longe melhores as opções para coerção *explícita*.

## Coerção Implícita

Coerção *implícita* se refere à tipos de conversões que são ocultas, com efeitos colaterais não óbvios que implicitamente ocorrem por outras ações. Em outras palavras, *coerções implicitas* são qualquer tipo de conversões que não são óbvias (para você).

Enquanto está claro qual é o objetivo de coerção *explícita* (tornar o código explícito e compreensível), pode ser *muito* óbvio que coerção *implícita* tenha o objetivo oposto: tornar o código mais difícil de se entender.

Confiar de olhos fechados, acredito que é aí que grande parte da raiva de coerções vêm. A maioria das reclamações sobre "coerções JavaScript" têm, na verdade, como alvo (eles percebendo ou não) coerções *implícitas*.

**Observação:** Douglas Crockford, autor de *"JavaScript: The Good Parts"*, afirmou em muitas palestras de conferências e artigos que coerção JavaScript deve ser evitada. Mas o que ele pareceu falar é que coerção *implícita* é ruim (na opnião dele). Porém, se você ler seu próprio código, você irá achar muitos exemplos de coerção, ambas *implícita* e *explícita*! Na verdade, a raiva dele parece ser primeiramente destinada para a operação `==`, mas você verá nesse capítulo, essa é apenas uma parte do mecanismo de coerção.

Então, a **coerção implícita é** maligna? Ela é perigosa? É uma falha no design do JavaScript? Nós devemos evitá-la a todo custo?

Aposto que a maioria de vocês, leitores, está inclinada a torcer com entusiasmo, "Sim!"

**Não tão rápido**. Me dê ouvidos.

Vamos assumir uma perspectiva diferente do que é coerção *implícita*, e pode ser, do que apenas que é "o oposto do bom tipo explícito de coerção". Isso é muito estreiro e perde uma nuance importante.

Vamos definir o objetivo de coerção *implícita* como: reduzir a verbosidade, boilerplate, e/ou detalhes de implentação desnecessários que encobre nosso código com ruído que nos distrai da intenção mais importante.

### Simplificando a Implicidade

Antes até de nós chegarmos ao JavaScript, deixe-me sugerir algum pseudo-código de uma linguagem teórica fortemente tipada para ilustrar:

```js
SomeType x = SomeType( AnotherType( y ) )
```

Nesse exemplo, eu tenho tenho um tipo de valor arbitrário em `y` que eu quero converter para o tipo `SomeType`. O problema é, essa linguagem não pode ir diretamente de qualquer coisa que `y` é pra `SomeType`. Ele precisa de um passo intermediário, onde ele primeiro converte para `AnotherType`, e então de `AnotherType` para `SomeType`.

Agora, e se a linguagem (ou definição que você mesmo pode criar com a linguagem) deixasse você *dizer*:

```js
SomeType x = SomeType( y )
```

Você não concorda que nós simplificamos o tipo de conversão aqui para reduzir o "ruído" desnecessário do passo de conversão intermediária? Quero dizer, isso é *realmente* tão importante, aqui mesmo nesse ponto do código, para ver e lidar com o fato que `y` vai primeiro para `AnotherType` antes e então vai para `SomeType`?

Alguns argumentariam, pelo menos em algumas circunstâncias, sim. Mas eu acho que um argumento equivalente pode ser feito de várias outras cinscunstâncias que aqui, a simplificação **na verdade ajuda na legibilidade do código** absorvendo ou escondendo tais detalhes, tanto na própria linguagem ou nas suas próprias abstrações.

Sem dúvidas, nos bastidores, em algum lugar, a conversão intermediária continua acontecendo. Mas se esse detalhe é oculto da view, nós apenas podemos raciocinar sobre pegar `y` para o tipo `SomeType` como uma operação genérica e enconder os detalhes bangunçados.

Embora não seja uma analogia perfeita, o que eu vou argumentar em todo o resto desse capítulo é que coerção *implícita* JS pode ser considerada como fornecedora de uma ajuda similar para seu código.

Mas, **e isso é muito importante**, essa não é uma declaração absoluta e ilimitada. Há definitivamente uma abundância de *males* que espreitam a coerção *implícita*, que prejudicará seu código muito mais do que qualquer potencial melhoria de legibilidade. Claramente, nós teremos que aprender como evitar certas construções para que não envenenemos nosso código com todas as formas de bugs.

Muitos desenvolvedores acreditam que se um mecanismo pode fazer algo últil **A** mas também pode ser abusado ou mal usado para fazer algo terrível **Z**, então nós devemos descartar completamente esse mecanismo, apenas por segurança.

Meu conselho para você é: não se conforme com isso. Não "mate uma mosca com uma bala de canhão". Não assuma coerção *implícita* é de toda ruim porque tudo que você acha que já viu são "partes ruins". Eu penso que há "partes boas" aqui, e eu quero ajudar e inspirar você para acha-las e absorve-las.

### Implicitamente: Strings <--> Numbers

Mais cedo nesse capítulo, nós exploramos a coerção *implícita* entre valores `string` e `number`. Agora, vamos explorar a mesma tarefa mas com abordagem de coerção *implícita*. Mas antes, nós temos que examinar algumas nuances de operações que vão forçar a coerção *implícita*.

O operador `+` é encarregado de servir propósitos tanto de adição de `number` como concatenação de `string`. Então como o JS sabe qual tipo de operação você quer usar? Considere:

```js
var a = "42";
var b = "0";

var c = 42;
var d = 0;

a + b; // "420"
c + d; // 42
```

Qual a diferença que causa `"420"` vs `42`? É um equívoco comum que a diferença é se um ou ambos os operadores são uma `string`, pois isso significa que `+` assumirá a concatenação `string`. Enquando isso é parcialmente verdade, é mais complicado que isso.

Considere:

```js
var a = [1,2];
var b = [3,4];

a + b; // "1,23,4"
```

Nenhum desses operandos é uma `string`, mas claramente ambos sofrem coerção para `string`s e então a concatenação `string` pula dentro. Então o que realmente stá acontecendo?

(**Atenção**: terrível e profunda linguagem de especificação abaixo, então pule os próximos dois parágrafos se isso intimida você!)

-----

De acordo com  a seção 11.6.1 da especificação ES5, o algoritmo `+` (quando um valor `object` é um operando) vai concatenar se um dos operandos já for uma `string`, ou se os passos seguintes produzirem uma representação `string`. Então quando o `+` recebe um `object` (incluindo `array`) para ambos operandos, ele primeiro chama a operação abstrata `ToPrimitive` (seção 9.1) no valor, o que então chama o algoritmo `[[DefaultValue]]` (section 8.12.8) com um contexto `number`.

Se você está prestando bastante atenção, você irá notar que essa operação é agora indêntica a como a operação abstrata `ToNumber` maneja `object` (veja a seção anterior "`ToNumber`"). A operação `valueOf()` no `array` vai falhar em produzir um primitivo simples, então ela cai na representação `toString()`. Os dois `array`s irão então se tornar `"1,2"` and `"3,4"`, respectivamente. Agora, `+` concatena as duas `string` como você espera: `"1,23,4"`.

-----

Vamos olhar além desses detalhes confusos e voltar para uma explicação simplificada: se o operando para `+` é uma `string` (ou torna-se uma com os passos acima!), a operação será concatenação de `string`. Do contrário, ela sempre será adição numérica.

**Observação** uma pegadinha de coerção comumente citada é `[] + {}` vs. `{} + []`, como essas duas expressões resultam, repectivamente, `"[object Object]"` e `0`. Há ainda mais, e cobrimos esses detalhes em "Blocos" no capítulo 5.

O que isso significa para coerção *implícita*?

Voce pode fazer a coerção de um `number` para uma `string` simplismente "adicionando" o `number` e a `string` vazia `""`:

```js
var a = 42;
var b = a + "";

b; // "42"
```

**Dica:** Adições numéricas com o operador `+` é comutativa, o que significa que `2 + 3` é o mesmo que `3 + 2`. Concatenação de String com `+` obviamente não é comutativa geralmente, **mas** com o caso específico do `""`, ela é efetivamente comutativa, assim como `a + ""` e `"" + a` irão produzir o mesmo resultado.

É extremamente comum/idiomático fazer a coerção (*implicitamente*) de `number` para `string` com uma operação `+""`. De fato, é interessante que, mesmo alguns dos maiores críticos da coerção *implícita* ainda usam essa abordagem em seu próprio código, em vez de uma das suas alternativas *explícitas*.

**Eu acho que esse é um grande exemplo** de formas úteis na coerção *implícita*, apesar do quão frequentemente o mecanismo recebe críticas.

Comparando essa coerção *implícita* de `a + ""` com nosso exemplo anterior de coerção *explícita* `String(a)`, há uma peculiaridade adicional para ter cuidado. Por causa de como a operação abstrata `ToPrimitive` funciona, `a + ""` invoca `valueOf()` no valor de `a`, no qual o valor de retorno é então finalmente convertido para uma `string` via operação abstrata interna `ToString`. Mas `String(a)` apenas invoca `toString()` diretamente.

Ambas abordagens vão resultar em uma `string` no final, mas se você está usando um `object` em vez de um valor primitivo `number` normal, você pode, não necessariamente, ter o *mesmo* valor de `string`!

Considere:

```js
var a = {
	valueOf: function() { return 42; },
	toString: function() { return 4; }
};

a + "";			// "42"

String( a );	// "4"
```

Geralmente, esse tipo de pegadinha não vai te pegar a menos que você realmente esteja tentando criar estruturas de dados e operações confusas, mas você deve ter cuidado se você está definindo métodos próprios `valueOf()` e `toString()` para algum `object`, pois a forma de fazer a coerção pode afetar o resultado.

E a outra direção? Como podemos fazer a *coerção implícita* de `string` para `number`?

```js
var a = "3.14";
var b = a - 0;

b; // 3.14
```

O operador `-` é definido apenas para subtrações numéricas, então `a - 0` força o valor `a` a sofrer coerção para `number`. Embora muito menos comum, `a * 1` or `a / 1` realizarão o mesmo resultado, já que esses operadores também são definidos apenas para operações numéricas.

E os valores `object` com o operador `-`? Mesma história que para o `+` acima:

```js
var a = [3];
var b = [1];

a - b; // 2
```

Ambos valores `array` precisam se tornar `number`s, mas eles terminam primeiro sofrendo a coerção para `string` (usando a serialização esperada `toString()`), e então sofrem a coerção para `number`s, para a substração `-` seja aplicada.

Então, a coerção *implícita* de valores `string` e `number` são tão maléficas sobre a qual você sempre ouviu histórias de terror? Pessoalmente, eu não acho.

Compare `b = String(a)` (*explícita*) com `b = a + ""` (*implícita*). Eu acho que casos podem ser feitos para que ambas abordagens sejam úteis para seu código. Certamente `b = a + ""` é um pouco mais comum em programas JS, provendo sua própria utilidade independentemente de *sentimentos* sobre os méritos e perigos da coerção *implícita* em geral.

### Implicitamente: Booleans --> Numbers

Eu acho que um caso onde coerção *implícita* pode realmente brilhar é em simplificar certos tipos de lógicas `boolean` complicadas em simples adições numéricas. Claro, essa não é uma técnica com propósito geral, mas uma solução específica para casos específicos.

Considere:

```js
function onlyOne(a,b,c) {
	return !!((a && !b && !c) ||
		(!a && b && !c) || (!a && !b && c));
}

var a = true;
var b = false;

onlyOne( a, b, b );	// true
onlyOne( b, a, b );	// true

onlyOne( a, b, a );	// false
```

Esse utilitário `onlyOne(..)` apenas deve retornar `true` se exatamente um dos argumentos for `true` / verdadeiro. Ele está usando coerção *implícita* nas validações verdadeiras e coerção *explícita* nas outras, incluindo o valor final retornado.

Mas e se precisamos que esse utilitário seja capaz de gerenciar quatro, cinco ou vinte flags da mesma forma? É bem difícil imaginar implementar um código que seja capaz de gerenciar todas essas permutações de comparações.

Mas aqui está onde fazer a coerção de valores `boolean` para `number`s (`0` ou `1`, obviamente) pode ajudar muito:

```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		// pula os valores falsos. mesmo que tratar
		// eles como 0's, mas evita os NaN's.
		if (arguments[i]) {
			sum += arguments[i];
		}
	}
	return sum == 1;
}

var a = true;
var b = false;

onlyOne( b, a );				// true
onlyOne( b, a, b, b, b );		// true

onlyOne( b, b );				// false
onlyOne( b, a, b, b, b, a );	// false
```

**Obeservação:** Claro, em vez do loop `for` em `onlyOne(..)`, você pode usar a tarefa do ES5 `reduce(..)`, mas eu não queria obscurecer os conceitos.

O que estamos fazendo aqui é relacionado com coerção de `1` para `true`/verdadeiro, e adionando todos numericamente. `sum += arguments[i]` usa coerção *implícita* para fazer isso acontecer. Se um e apenas um valor na lista de `arguments` é `true`, então a soma numérica vai ser `1`, do contrário a soma não será `1` e portanto a condição desejada não será atendida.

Nós podemos claro fazer isso com coerção *implícita* no lugar:

```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		sum += Number( !!arguments[i] );
	}
	return sum === 1;
}
```

Nós primeiro usamos `!!arguments[i]` para forçar a coerção dos valores para `true` ou `false`. Só assim você poderia passar os valores `boolean`, como `onlyOne( "42", 0 )`, e isso ainda continuará funcionando como esperado (do contrário você vai terminar com uma concatenação `string` e a lógica será incorreta).

Uma vez que temos certeza que é um `boolean`, nós fazemos outra coerção *explícita* com `Number(..)` para ter certeza que os valores são `0` ou `1`.

As formas de coerção *explícita* desse utilitário são "melhores"? Ela evita o `NaN` como explicado nos comentários do código. Mas, ultimamente, isso depende da sua necessidade. Eu pessoalmente acho que a forma anterior, confiando em coerção *implícita* é mais elegante (se você não tiver passando `undefined` ou `NaN`), e a versão *explícita* é desnecessariamente mais verbosa.

Mas assim como tudo o que discutimos aqui, é uma escolha.

**Observação:** Independentemente de abordagem *implícita* ou *explícita*, você pode facilmente fazer variações `onlyTwo(..)` ou `onlyFive(..)` simplismente mudando a comparação final de `1`, para `2` ou `5`, respectivamente. Isso é drasticamente mais fácil do que adicionar um monte de expressões `&&` e `||`. Então, geralmente, coerção é muito útil nesse caso.

### Implicitamente: * --> Boolean

Agora, vamos voltar nossa atenção para coerção *implícita* de valores `boolean`, como isso é de longe o mais comum e também de longe o mais potencialmente problemático.

Lembre-se, coerção *implícita* é o que entra quando você usa um valor de tal forma que ele força o valor a ser convertido. Para operações numéricas e de `string`, é bem fácil de ver como as coerções podem acontecer.

Mas, que tipo de expressões de operação requerem/forçam (*implicitamente*) uma coerção `boolean`?

1. A expressão test em uma declaração `if(..)`.
2. A expressão test (segunda cláusula) em um header `for ( .. ; .. ; .. )`.
3. A expressão test em loops `while (..)` e `do..while(..)`.
4. A expressão test (primeira cláusula) em expressões ternárias `? :`.
5. O operando *left-hand* (que serve uma expressão test -- veja abaixo!) para os operadores `||` ("lógico OU") e `&&` ("lógico E").

Qualquer valor usado nesse contexto que já não seja um `boolean` vai sofrer coerção *implícita* para um `boolean` usando as regras da operação abstrata `ToBoolean` abordada anteriormente nesse capítulo.

Vamos ver alguns exemplos:

```js
var a = 42;
var b = "abc";
var c;
var d = null;

if (a) {
	console.log( "yep" );		// yep
}

while (c) {
	console.log( "nope, never runs" );
}

c = d ? a : b;
c;								// "abc"

if ((a && d) || c) {
	console.log( "yep" );		// yep
}
```

Em todos estes contextos, os valores não `boolean`s sofrem coerção *implícita* para seus equivalentes `boolean` para fazer decisões de teste.

### Operadores `||` e `&&`

É bem provável que você já tenha visto os operadores `||` ("lógico OU") e `&&` ("lógico E") na maioria ou em todas outras linguagens que você já usou. Então seria natural presumir que eles trabalham basicamente da mesma forma no JavaScript como nas outras linguagens similares.

Há aqui uma nuance pouco conhecida, mas muito importante.

Na verdade, eu argumentaria que esses operadores nem sequer deveriam ser chamados de "operadores___lógicos", pois esse nome é incompleto ao descrever o que eles fazem. Se eu fosse dar à eles um nome mais preciso (se mais desajeitado), eu os chamaria de "operadores de seletores", ou mais completo, "operadores de seletor de operandos".

Por quê? Porque eles, na verdade, não resultam em um valor *lógico* (também conhecido como `boolean`) no JavaScript, como eles fazem em algumas outras linguagens.

Então qual o *resultado* deles? Eles retornam o valor de um (e apenas um) de seus dois operandos. Em outras palavras, **eles selecionam um dos dois valores de operandos**.

Citação da seção 11.11 da especificação ES5:

> O valor produzido pelo operador && ou || não pe necessariamente do tipo Boolean. O valor prodizido sempre será o valor de uma das duas empressões de operandos.

Vamos ilustrar:

```js
var a = 42;
var b = "abc";
var c = null;

a || b;		// 42
a && b;		// "abc"

c || b;		// "abc"
c && b;		// null
```

**Espera, o quê?** Pense nisso. Em liunguagens como C e PHP, essas expressões resultam em `true` ou `false`, mas em JS (e Python e Ruby, aliás!), o resultado vem dos próprios valores.

Ambos operadores, `||` e `&&` fazem um teste `boolean` no **primeiro operando** (`a` ou `c`). Se o operando já não for um `boolean` (que no caso, não é), uma coerção `ToBoolean` normal acontece, então o teste pode ser feito.

Para o operador `||`, se o teste é `true`, a expressão `||` resulta no valor do *primeiro operando* (`a` ou `c`). Se o teste é `false`, a expressão `||` resulta no valor do *segundo operando* (`b`).

Inversamente, para o operador `&&`, se o teste é `true`, a expressão `&&` resulta no valor do *segundo operando* (`b`) . Se o teste é `false`, a expressão `&&` resulta no valor do *primeiro operando* (`a` ou `c`).

O resultado das expressões `||` ou `&&` é sempre o valor de um dos operandos, **não** o resultado (possivelmente convertido) do teste. Em `c && b`, `c` é `null`, e portanto falso. Mas a própria expressão `&&` resulta em `null` (o valor em `c`), não no `false` convertido usado no teste.

Viu como esses operaodres agem como "seletores de operandos" agora?

Outra forma de pensar sobre esses operadores:

```js
a || b;
// aproximadamente equivalente à:
a ? a : b;

a && b;
// aproximadamente equivalente à:
a ? b : a;
```

**Observação:** Eu chamo `a || b` de "aproximadamente equivalente" à `a ? a : b` porque a saída é idêntica, mas há uma diferença de nuance. Em `a ? a : b`, se `a` era uma expressão mais complexa (como por exemplo uma que pode ter efeitos colaterais como chamar uma `function`, etc..), então a expressão `a` vai possivelmente ser avaliada duas vezes (se a primeira avaliação for verdadeira). Por contraste, para `a || b`, a expressão `a` é avalidada apenas uma vez, e esse valor é usado tanto para o teste coercivo como para o valor do resultado (se apropriado). A mesma nuance se aplica para as expressões `a && b` and `a ? b : a`.

Um uso extremamente comum e útil desse comportamento, em que há uma grande chance de você já ter usado isso antes e não entendido completamente é:

```js
function foo(a,b) {
	a = a || "hello";
	b = b || "world";

	console.log( a + " " + b );
}

foo();					// "hello world"
foo( "yeah", "yeah!" );	// "yeah yeah!"
```

O idioma `a = a || "hello"` (às vezes se diz que é a versão do JavaScript do "null coalescing operator" do C#) testa `a` e se ele não tem valor (ou apenas um valor falso indesejável), provê um valor padrão de backup (`"hello"`).

No entanto, **tenha cuidado!**

```js
foo( "That's it!", "" ); // "That's it! world" <-- Opa!
```

Viu o problema? `""` como segundo argumento é uma valor falso (veja `ToBoolean` anteriormente nesse capítulo), então o teste `b = b || "world"` falha, e o valor padrão `"world"` é substituído, mesmo quando a intenção era, provavelmente, passar explicitamente que `""` seja o valor atribuído para `b`.

Essa linguagem `||` é extremamente comum, e bem útil, mas você tem que usá-la somente em casos onde *todos os valores falsos* devem ser ignorados. Do contrário, você precisará ser mais explícito no seu teste, e provalmente usar um ternário `? :` no lugar.

Essa *atribuição de valor padrão* é tão comum (e útil!) que até mesmo aqueles que veementemente e publicamente condenam a coerção JavaScript, freuqnetemente a utilizam em seu pŕoprio código!

E o `&&`?

Esse é outra linguagem que é bem menos comum, mas que é usada por minificadores JS frequentemente. O operador `&&` "seleciona" o segundo operando se, e apenas se, o primeiro teste  do operando for verdadeiro, e esse uso é chamado algumas vezes de "operador guarda" (veja também "Curto-circuito" no capítulo 5) -- o primeiro teste de expressão "guarda" a segunda expressão:

```js
function foo() {
	console.log( a );
}

var a = 42;

a && foo(); // 42
```

`foo()` é chamada apenas porque o teste de `a` é verdadeiro. Se esse teste falha, essa declaração de expressão `a && foo()` vai apenas parar silenciosamente -- isso é conhecido como "curto-circuito" -- e nunca chamar `foo()`.

De novo, não é muito comum que as pessoas criem essas coisas. Normalmente, elas fazem `if (a) { foo(); }` no lugar. Mas os minificadores JS escolhem `a && foo()` porque é muito mais curto. Então, agora, se você alguma vez tiver que decifrar tal código, você saberá o que ele está fazendo e por quê.

Ok, então `||` e `&&` têm alguns truques na manga, com tanto que você queira permitir a coerção *implícita* nessa mistura.

**Observação:** Ambos, `a = b || "something"` e `a && b()` referem-se ao comportamento de circuitos curtos, que nos abordamos com mais detalhes no capítulo 5.

O fato desses operadores. na verdade, não resultarem em `true` e `false` possivelmente mexerá um pouco com a sua cabeça agora. Você provavelmente está se perguntando como todos suas declarações `if` e seus loops `for` funcionavam, se eles incluíram expressões lógicas compostas como `a && (b || c)`.

Não se preocupe! o céu não está desabando. Seu código está (provavelmente) bem. É que você provavelmente nunca percebeu antes que havia uma coerção *implícita* para `boolean` acontecendo **depois** que a expresão composta era analisada.

Considere:

```js
var a = 42;
var b = null;
var c = "foo";

if (a && (b || c)) {
	console.log( "yep" );
}
```

Esse código continua funcionando da forma que você sempre achou que funcionava, exceto por um detalhe sutil. A expressão `a && (b || c)` *na verdade* resulta em `"foo"`, não `true`. Portanto, a declaração `if` *então* força o valor `"foo"` a sofrer coerção para `boolean`, o que é claro será `true`.

Viu? não há razão para entrar em pânico. Seu código, provavelmente, está à salvo. Mas agora você sabe mais sobre como isso faz o que faz.

E agora você também percebeu que tal código usa coerção *implícita*. Se você ainda está no time "evite coerção (implícita)", você terá que voltar e fazer todos aqueles testes *explicitamente*:

```js
if (!!a && (!!b || !!c)) {
	console.log( "yep" );
}
```

Boa sorte com isso! ... Desculpe, apenas provocando.

### Coerção de Symbols

Até esse ponto, não houve quase nenhuma diferença de resultado observável entre coerção *explícita* e *implícita* -- apenas a legibilidade do código está em jogo.

Mas os Symbols do ES6 introduzem uma pegadinha no sistema de coerção que nós precisamos discutir brevemente. Por razões que vão bem além do escopo do que nós vamos discutir nesse livro, coerção *explícita* de um `symbol` para uma `string` é permitida, mas coerção *implícita* do mesmo não é permitida e lançará um erro.

Considere:

```js
var s1 = Symbol( "cool" );
String( s1 );					// "Symbol(cool)"

var s2 = Symbol( "not cool" );
s2 + "";						// TypeError
```

Valores `symbol` não fazem coerção para `number` de nenhuma forma (lança um erro de qualquer jeito), mas estranhamente ambas podem fazer coerção *explícita* e *implícita* para `boolean` (sempre `true`).

Consistências são sempre fáceis de aprender, e exceções nunca são divertidas de lidar, mas nós apenas precisamos ter cuidado com os novos valores `symbol` do ES6 e como nós fazemos coerção nelas.

A boa notícia: provavelmente será extremamente raro você precisar fazer coerção de uma valor `symbol`. A maneira como eles são normalmente usados (veja o capítulo 3), provavelmente não exigirá coerção em uma base normal.

## Igualdade Ampla vs. Igualdade Estrita

Igualdade ampla é o operador `==`, e igualdade estrita é o operador `===`. Ambos operadores são usados para comparar dois valores para "igualdade", mas o "amplo" vs. "estrito" indica uma diferença de comportamento **muito importante** entre os dois, especificamente em como eles decidem a "igualdade".

Um equívico muito comum sobre esses dois operadores é: `==` verifica igualdade de valores e `===` verifica igualdade de ambos, valores e tipos. Enquanto isso parece sensato, é impreciso. Incontáveis livros bem respeitados de JavaScript e blogs disseram exatamente isso, mas infelizmente eles estão todos *errados*.

A descrição correta é: "`==` permite coerção na comparação da igualdade e `===` não permite."

### Desempenho da Igualdade

Pare e pense sobre a diferença entre a primeira explicação (imprecisa) e esta segunda (precisa).

Na primeira explicação, parece óbvio que `===` está *fazendo mais trabalho* que `==`, porque ele precisa *também* verificar o tipo. Na segunda explicação, `==` é o que está *fazendo mais trabalho* porque ele precisa seguir através dos passos da coerção se os tipos são diferentes.

Não caia na armadilha, como muitos fazem, de pensar que isso tem alguma coisa a ver com performance, como se `==` fosse ser mais lento que `===` de qualquer maneira relevante. Embora seja mensurável que a coerção tome *um pouco mais* de tempo de processamento, são meros microsegundos (sim, isso é um milionésimo de segundo!).

Se você está comprando dois valores do mesmo tipo, `==` e `===` usam o algoritmo idêntico, e algumas outras diferenças mínimas na implementação do motor, eles devem fazer o mesmo trabalho.

Se você está comparando dois valores de tipos diferentes, a performance não é o fator importante. O que você deveria se perguntar é: ao comparar esses dois valores, eu quero a coerção ou não?

Se você quer a coerção, use `==` igualdade ampla, mas se você não quer coerção, use `===` igualdade estrita.

**Observação:** a implicação aqui é que ambos `==` e `===` verifiquem os tipos dos seus operandos. A diferença é em como eles irão responder se os tipos não coincidem.

### Igualdade abstrata

O comportamento do operador `==` é definido como "O algoritmo de comparação de igualdade abstrata" na seção 11.9.3 da especificação ES5. O que está listado lá é um algoritmo abrangente, mas simples, que declara explicitamente todas as combinações possíveis de tipos, e como as coerções (se necessárias) devem acontecer para cada combinação.

**Atenção:** Quando (*implicitamente*) a coerção é vista como sendo muito complicada e também defeituosa para ser uma *boa parte útil*, são essas regras de "igualdade abstrata" que estão sendo condenadas. Geralmente, elas são ditas como muito complexas e não intuitivas para desenvolvedores aprenderem e usá-las na prática, e que elas mais causam bugs nos programas JS do que provêm grande legibilidade do código. Eu acredito que essa é uma premissa defeituosa -- que vocês leitores são desenvolvedores competentes que escrevem (e leêm e entendem!) algoritmos (códigos) durante todo o dia. Então o que se segue é uma plano de exposição das "igualdades abstratas" em termos simples. Mas eu imploro para que você também leia a seção 11.9.3 da especificação ES5. Eu acho qe você ficará surpreso do quão sensata ela é.

Basicamente, a primeira cláusula (11.9.3.1) diz, se os dois valores que estão sendo comparados são do mesmo tipo, eles são simplesmente e naturalmente comparados via identidade como você esperava. Por exemplo, `42` só é igual a `42`, e `"abc"` é apenas igual à `"abc"`.

Algumas pequenas exceções à expectativa normal para estar ciente são:

* `NaN` nunca é igual a ela mesma (veja Capítulo 2)
* `+0` e `-0` são iguais entre si (veja Capítulo 2)

A última provisão na cláusula 11.9.3.1 é para comparação de igualdade ampla `==` com `object`s (incluindo `function`s e `array`s). Tais valores são apenas *iguais* se ambos referenciam para *exatamente o mesmo valor*. Não ocorre coerção aqui.

**Observação:** A comparação de igualdade estrita `===` é definida identicamente para 11.9.3.1, incluindo a provisão sobre dois valores de `objects`. É um fato pouco conhecido que **`==` e `===` se comportam de forma idêntica** no caso inde dois `objects`s estão sendo comparados.

O resto do algoritmo em 11.9.3. especifica qur se você usar igualdade ampla `==` para comparar dois valores de tipos diferentes, um ou ambos os valores precisarão sofrer coerção *implícita*. Essa coerção acontece para que ambos valores eventualmente terminem com o mesmo tipo, no qual possam ser comparados pela igualdade usando valores de identidade simples.

**Observação:** A operação de não-igualdade ampla `!=` é definida exatamente como você esperava, na medida em que é literalmente a comparação da operação `==` realizada na sua totalidade, e então a negação do resultado. O mesmo vale para a operação de não-igualdade estrita `!==`.

#### Comparando: `string`s com `number`s

Para ilustrar a coerção `==`, vamos primeiro desconstruir os exemplos anteriores `string` e `number` desse capítulo:

```js
var a = 42;
var b = "42";

a === b;	// false
a == b;		// true
```

Como esperávamos, `a===b` falha, porque nenhuma coerção é permitida, e de fato, os valores `42` e `"42"` são diferentes.

No entanto, a segunda comparação `a == b` usa igualdade ampla, que significa que se acontecer dos tipos serem diferentes, a comparação do algoritmo vai fazer uma coerção *implícita* em um ou ambos os valores.

Mas qual, exatamente, é o tipo de coerção que acontece aqui? O valor `a` de `42` torna-se uma `string`, ou o valor `b` de `"42"` torna-se um `number`?

Na cláusula 11.9.3.4-5 da especificação ES5 diz:

> 4. Se o Type(x) é um Number e o Type(y) é uma String,
>    retorna o resultado da comparação x == ToNumber(y).
> 5. Se o Type(x) é uma String e Type(y) ié um Number,
>    retorna o resultado da comparação ToNumber(x) == y.

**Atenção:** A especificação usa `Number` e `String` como nomes formais para os tipos, enquanto esse livro prefere `number` e `string` para tipos primitivos. Não deixe a capitalização de `Number` na especificação te confundir com a função nativa `Number()`. Para nossos propósitos, a capitalização do nome do tipo é irrelevante -- eles têm, basicamente, o mesmo significado.

Claramente, a especificação diz que o valor `"42"` sofre coerção para um `number` na comparação. O *como* dessa coerção já foi abordada anteriormente, especificamente com a operação abstrata `ToNumber`. Nesse caso, é bem óbvio que os dois valores `42` resultantes são iguais.

#### Comparando: qualquer coisa com `boolean`

Uma das maiores pegadinhas com a coerção *implícita* da igualdade ampla `==` aparecem quando você tenta comparar um valor diretamente com `true` ou `false`.

Considere:

```js
var a = "42";
var b = true;

a == b;	// falso
```

Espera, o que aconteceu aqui? Nós sabemos que `"42"` é um valor verdadeiro/*thruthy* (veja anteriormente neste capítulo). Então, como ele não é `==`, igualdade ampla à `true`?

A razão é simples e enganosamente complicada. É tão fácil de cometer um equívico, muitos desenvolvedores JS nunca prestam atenção suficiente para compreendê-la.

Vamos citar novamente a especificação, cláusula 11.9.3.6-7:

> 6. Se Type(x) é Boolean,
>    retorna o valor da comparação ToNumber(x) == y.
> 7. Se Type(y) é Boolean,
>    retorna o valor da comparação x == ToNumber(y).

Vamos acabar com isso. Primeiro:

```js
var x = true;
var y = "42";

x == y; // false
```

O `Type(x)` é de fato `Boolean`, então ele executa `ToNumber(x)`, que faz a coerção de `true` para `1`. Agora `1 == "42"` é avaliado. Os tipos continuam diferentes, então (essencialmente recursivamente) nós reconsultamos o algoritmo, que assim como acima, irá fazer a coerção de `"42"` para `42`, e `1 == 42` é claramente `false`.

Reverta isso, e nós teremos a mesma saída:

```js
var x = "42";
var y = false;

x == y; // false
```

O `Type(y)` é `Boolean` dessa vez, então `ToNumber(y)` custa `0`. `"42" == 0` recursivamente torna-se `42 == 0`, que claro, é `false`.

Em outras palavras, **o valor `"42"` não é nem `== true` nem `== false`.** Primeiramente, essa declaração pode parecer loucura. Como pode um valor não ser nem verdadeiro nem falso?

Mas esse é o problema! Você está fazendo a pergunta, totalmente errada. Não é sua culpa, de verdade. Seu cérebro está te enganando.

`"42"` é de fato verdadeiro/*truthy*, mas `"42" == true` **não está executando um teste/coerção de boolean** de maneira nenhuma, não importa o que seu cérebro diga. `"42"` não está sofrendo coerção para um `boolean` (`true`), mas, em vez disso, `true` é que está sofrendo coerção para `1`, e então `"42"` estão sofrendo coerção para `42`.

Quer nos agrade ou não, `ToBoolean` nem está envolvido aqui, então a verdade ou falsidade de `"42"` é irrelevante para a operação `==`!

O que *é* relevante, é entender como o algoritmo de comparação `==` se comporta com todas as diferentes combinações. No que se refere à um valor `boolean` de qualquer lado do `==`, um `boolean` sempre sofre coerção para um `number` *primeiro*.

Se isso parece estranho para você, você não está sozinho. Eu pessoalmente recomendaria à nunca, nunca mesmo, em nenhuma circunstância, usar `== true` ou `== false`. Nunca.

Mas lembre-se, eu estou falando somente do `==` aqui. `=== true` e `=== false` não permitirá a coerção, então você está seguro dessa coerção `ToNumber` oculta.

Considere:

```js
var a = "42";

// péssimo (vai falhar!):
if (a == true) {
	// ..
}

// ruim também (vai falhar!):
if (a === true) {
	// ..
}

// bom o bastante (funciona implicitamente):
if (a) {
	// ..
}

// melhor (funciona explicitamente):
if (!!a) {
	// ..
}

// ótimo também (funciona explicitamente):
if (Boolean( a )) {
	// ..
}
```

Se você sempre evita usar `== true` ou `== false` (também conhecido como igualdade ampla com `boolean`s) no seu código, você nunca terá que se preocupar sobre essa pegadinha mental de verdadeiro/falso.

#### Comparando: `null`s com `undefined`s

Outro exemplo de coerção *implícita* pode ser visto com `==` igualdade ampla entre valores `null` e `undefined`. Citando de novo a especificação ES5, cláusula 11.9.3.2-3:

> 2. Se x é null e y é undefined, retorna true.
> 3. Se x é undefined e y é null, retorna true.

`null` e `undefined` quando comparados com `==` igualdade ampla, equipararm-se (fazem coerção) uns nos outros (assim como neles próprios, obviamente), e nenhum outro valor em toda a linguagem.

Isso significa que `null` e `undefined` podem ser tratados sem distinção para propósitos de comparação, se você usar o operador `==` igualdade ampla para permitir coerção *implícita* mútua.

```js
var a = null;
var b;

a == b;		// true
a == null;	// true
b == null;	// true

a == false;	// false
b == false;	// false
a == "";	// false
b == "";	// false
a == 0;		// false
b == 0;		// false
```

A coerção entre `null` e `undefined` é segura e previsível e nenhum outro valor pode dar falsos positivos em tal teste. Eu recomendo usar essa coerção para permitir que `null` e `undefined` sejam indistinguíveis e assim tratados como o mesmo valor.

Por exemplo:

```js
var a = doSomething();

if (a == null) {
	// ..
}
```

A verificação `a == null` vai passar somente se `doSomething()` retornar ambos, `null` ou `undefined`, e vai falhar com qualquer outro valor, mesmo outro valor falso como `0`, `false` e `""`.

A forma de verificação *explícita*, que não permite nenhum tipo de coerção, é (eu acho) desnecessariamente muito mais feia (e talvez um pouco menos performática!):

```js
var a = doSomething();

if (a === undefined || a === null) {
	// ..
}
```

Na minha opinião, a forma `a == null` é ainda outro exemplo de onde a coerção *implícita* melhora a legibilidade do código, mas faz isso de uma maneira confiável e segura.

#### Comparando: `object`s com não-`object`s

Se um `object`/`function`/`array` é comparado com um escalar primitivo simples (`string`, `number` ou `boolean`), a especificação ES5 diz na cláusula 11.9.3.8-9:

> 8. Se Type(x) é tanto uma String ou um Number e Type(y) é um Object,
>    retorna o resultado da comparação x == ToPrimitive(y).
> 9. Se Type(x) é um Object e Type(y) tanto uma String ou um Number,
>    retorna o resultado da comparação ToPrimitive(x) == y.

**Observação:** Você pode notar que essas cláusulas apenas mencionam `String` e `Number`, mas não `Boolean`. Isso porque, como dito antes, a clásula 11.9.3.6-7 trata da coerção de qualquer operando `Boolean` apresentado para um `Number` primeiro.

Considere:

```js
var a = 42;
var b = [ 42 ];

a == b;	// true
```

O valor `[42]` tem sua operação abstrata `ToPrimitive` chamada (veja a seção anterior "Valores de operações abstratas"), que resulta no valor `"42"`. Daqui em diante, é apenas `42 == "42"`, que como já abordamos, torna-se `42 == 42`, então `a` e `b` são coercitivamente iguais.

**Dica:** Todos os quirks da operação abstrata `ToPimitive` que nós discutimos anteriormente nesse capítulo (`toString()`, `valueOf()`) são aplicados aqui como nós esperávamos. Isso pode ser bem útil se você tiver uma estrutura de dados complexa que você quer definir um método personalizado em `valueOf()`, para fornecer uma valor simples para propósitos de comparação de igualdade.

No capítulo 3, nós abordamos "unboxing", onde um `object` wrapper em torno de uma valor primitivo (como de `new String("abc")`, por exemplo) é desencapsulado, e o valor primitivo subjacente ("abc") é retornado. Esse comportamento está relacionado à coerção `ToPrimitive` no algoritmo `==` :

```js
var a = "abc";
var b = Object( a );	// Mesmo que `new String( a )`

a === b;				// false
a == b;					// true
```
`a == b` é `true` porque `b` sofre coerção (ou "unboxed", desencapsulado) via `ToPrimitive` para seu valor escalar primitivo "abc" subjacente, que é o mesmo que o valor em `a`.

Há alguns valores onde isso não é o caso, por conta de outras regras primárias do algoritmo de `==`. Considere:

```js
var a = null;
var b = Object( a );	// Mesmo que `Object()`
a == b;					// false

var c = undefined;
var d = Object( c );	// Mesmo que `Object()`
c == d;					// false

var e = NaN;
var f = Object( e );	// Mesmo que `new Number( e )`
e == f;					// false
```

Os valores `null` e `undefined` não podem ser encapsulados (*boxed*) -- eles não tem um object wrapper equivalente -- Então o `Object(null)` é como o `Object()` em que ambos apenas produzem um objeto normal.

`NaN` pode ser encapsulado no seu object wrapper `Number` equivalente, mas quando `==` causa uma desencapsulamento, a comparação `NaN == NaN` falha porque `NaN` nunca é igual a si mesmo (veja o capítulo 2).

### Casos à parte

Agora que nós examinamos completamente como a coerção *implícita* de `==` igualdade ampla funciona (tanto na maneira sensível como na surpreendente), vamos tentar chamar os piores e mais loucos casos para que possamos ver o que precisamos evitar para não ser pego com bugs de coerção.

Primeiro, vamos examinar como modificar prototypes nativos podem produzir resultados loucos:

#### Um número por outro valor seria...

```js
Number.prototype.valueOf = function() {
	return 3;
};

new Number( 2 ) == 3;	// true
```

**Atenção:** `2 == 3` não teria caído nessa armadilha, porque nem `2` nem `3` teria invocado o método nativo `Number.prototype.valueOf()` porque ambos já são valores primitivos `number` e podem ser comparados diretamente. No entanto, `new Number(2)` deve passar pela coerção `ToPrimitive`, e por isso invocar `valueOf()`.

Maldade né? É claro que é. Ninguém nunca deveria fazer algo assim. O fato de que você *pode* fazer isso é usado como crítica da coerção e `==`. Mas isso é uma frustação mal direcionada. JavaScript não é *ruim* por que você pode fazer tais coisas, um desenvolvedor é *ruim* **se eles fizerem tais coisas**. Não caia na falácia "minha linguagem de programação deveria me proteger de mim mesmo".

Próximo vamos considerar outro exemplo complicado, o que leva a maldade do exemplo anterior para outro nível:

```js
if (a == 2 && a == 3) {
	// ..
}
```

Você pode pensar que isso seria impossível, porque `a` nunca deveria ser igual a ambos `2` e `3` *ao mesmo tempo*. Mas "ao mesmo tempo" é impreciso, já que a primeira expressão `a == 2`, acontece estritamente *antes* de `a == 3`.

Então, e se nós fizermos com que `a.valueOf()` tivesse efeitos colaterais toda vez que fosse chamado, de modo que na primeira vez retorna `2` e na segunda vez que for chamada retorne `3`? Muito fácil:

```js
var i = 2;

Number.prototype.valueOf = function() {
	return i++;
};

var a = new Number( 42 );

if (a == 2 && a == 3) {
	console.log( "Yep, this happened." );
}
```

De novo, esses são truques maldosos. Não faça-os. Mas também não os use como queixas contra a coerção. Abusos potenciais dos mecanismos não são evidências suficientes para condenar o mecanismo. Apenas evite esses truques malucos, e mantenha-se com o uso válido e apropriado da coerção.

#### Comparações False-y

A queixa mais comum contra coerção *implícita* na comparação `==` ver de quão surpreendentes os valores *falsy* se comportam quando comparados entre si.

Para ilustrar, vamos olhar para a lista de casos à parte sobre comparação de valores *falsy*, para ver quais são os razoáveis e os problemáticos:

```js
"0" == null;			// false
"0" == undefined;		// false
"0" == false;			// true -- UH OH!
"0" == NaN;				// false
"0" == 0;				// true
"0" == "";				// false

false == null;			// false
false == undefined;		// false
false == NaN;			// false
false == 0;				// true -- UH OH!
false == "";			// true -- UH OH!
false == [];			// true -- UH OH!
false == {};			// false

"" == null;				// false
"" == undefined;		// false
"" == NaN;				// false
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
"" == {};				// false

0 == null;				// false
0 == undefined;			// false
0 == NaN;				// false
0 == [];				// true -- UH OH!
0 == {};				// false
```

Nessa lista de 24 comparações, 17 delas são bem razoáveis e previsíveis. Por exemplo, nós sabemos que `""` e `NaN` não são valores iguais, e mesmo eles não sofrem coerção para serem igualdades amplas, considerando que `"0"` e `0` são razoavelmente igualáveis e *vão* sofrer coerção como igualdade ampla.

No entanto, sete destas comparaçãoes estão marcadas com "UH OH!" porque como falsos positivos, elas mais provavelmente são pegadinhas que podem te enganar. `""` e `0` são valores definitivamente distintos, e é raro que você queira tratá-los como iguais, então a coerção mútua é problemática. Note que não há nenhum falso negativo aqui.

#### Os Loucos 

No entanto, nós não temos que parar aqui. Nós podemos continuar procurando por mais coerções problemáticas:

```js
[] == ![];		// true
```

Oooo, isso parece estar em um nível mais alto de loucura, certo!? Seu cérebro pode estar te enganando que você está comparando um valor verdadeiro com falso, então o resultado `true` é surpreendente, como nós *sabemos*, um valor nunca pode ser verdadeiro e falso ao mesmo tempo!

Mas, na verdade, não é isso que está acontecendo. Vamos destrinchar isso. O que sabemos sobre o operador unário `!`? Ele aplica a coerção explícita para um `boolean` usando as regras de `ToBoolean` (e também troca a paridade). Então antes de `[] == ![]` sequer ser processado, ele já traduziu para `[] == false`. Nós já vimos essa forma em nossa lista acima (`false == []`), então seu resultado surpreendente *não é novo* para nós.

E sobre esses outros casos?

```js
2 == [2];		// true
"" == [null];	// true
```

Como nós dissemos anteriormente em nossa discussão `ToNumber`, o lado direito dos valores `[2]` e `[null]` vão passar pela coerção `ToPrimitive` e então eles podem ser comparados mais prontamente aos primitivos simples (`2` e `""`, respectivamente) no lado esquerdo. Desde que `valueOf()` para o próprio valor `array`, a coerção falha na stringficação do `array`.

`[2]` torna-se `"2"`, o que então sofre coerção `ToNumber` para `2` para o valor do lado direito na primeira comparação. `[null]` apenas continua sendo `""`.

Então, `2 == 2` e `"" ==""` são completamente compreensíveis.

Se seu instinto é continuar desgostando destes resultados, sua frustação, na verdade, não é com a coerção, como provavelmente você pensa que é. É na verdade uma queixa contra o comportamento padrão de valores `array` `ToPrimitive` de uma coerção de `[2]` e então `"2"`, exceto talvez `"[2]"` -- mais isso pode ser muito estranho em outros contextos!

Você poderia justificar que, desde que `String(null)` torna-se `"null"`, então `String([null])` deverá também tornar-se `"null"`. Essa é uma afirmação razoável. Então, esse é o verdadeiro culpado.

Coerção *implícita* por si só não é a vilã aqui. Até mesmo uma coerção *explícita* de `[null]` para uma `string` resulta em `""`. O que está em contradição é se é sensato para um valor `array` stringficar para um equivalente de seu conteúdo, e exatamente como isso acontece. Então, direcione sua frustação para as regras de `String( [..] )`, porque é de onde a loucura vem. Talvez não deva mesmo acontecer a stringficação de um `array`? Mas isso teria muitas outras desvantagens em outras partes da linguagem.

Outra pegadinha famosa citada:

```js
0 == "\n";		// true
```

Como discutimos antes com `""`, `"\n"` (ou `" "` ou qualquer outra combinação de espaços vazios) sofre coerção via `ToNumber`, e o resultado é `0`. Que outro valor de `number` você espera que um espaço vazio seja convertido? Te encomoda que `Number(" ")` retorne `0`?

Realmente o único outro `number` razoável no qual strings vazias ou espaços em brancos possam sofrer coerção é o `NaN`. Mas isso seria *realmente* melhor? A comparação `" " == NaN` vai certamente falhar, mas não está claro que teríamos realmente *corrigido* qualquer uma das preocupações subjacentes.

As chances de que um programa JS real falhe porque `0 == "\n"` são terrivelmente raras, e tais casos podem facilmente ser evitados.

Conversões de tipo **sempre** tem casos à parte, em qualquer linguagem -- nada especificamente para coerção. Os problemas aqui são sobre adivinhar um certo conjunto de casos à parte (e, talvez, corretamente!), mas esse não pe um argumento saliente contra o mecanismo geral de coerção.

Quase qualquer coerção louca entre *valores normais* que você provavelmente irá encontrar (além de hacks intencionalmente complicados `valueOf()` or `toString()` como anteriores) se resumirão a esta lista curta de sete coerções que nós identificamos acima.

Para contrastar contra estes 24 suspeitos prováveis para pegadinhas de coerção, considere outra lista como esta:

```js
42 == "43";							// false
"foo" == 42;						// false
"true" == true;						// false

42 == "42";							// true
"foo" == [ "foo" ];					// true
```

Nesses casos não falsos, não à parte (e há literalmente um número infinito de comparações que podemos colocar nesta lista), os resultados da coerção são totalmente seguros, razoáveis ​​e explicáveis.

#### Teste de Sanidade

OK, nós definitivamente achamos algumas coisas loucas quando nós olhamos a fundo na coerção *implícita*. Não é à toa que a maioria dos desenvolvedores afirmam que coerção é ruim e deve ser evitada, certo!?

Mas vamos voltar um passo e fazer um teste de sanidade.

Para fins de comparações de magnitude, nós temos *uma lista* de sete pegadinhas de coerção problemáticas, mas nós temos *outra lista* de (ao menos 17, mas atualmente infinita) coerções que são totalmente sensatas e explicáveis.

Se você está buscando por um exemplo de texto para "matar uma mosca com um canhão", é isto: descartando a totalidade da coerção (a infinitamente larga lista de comportamentos seguros e úteis) por causa de uma lista de, literalmente, sete pegadinhas.

A reação mais prudente seria perguntar, "como eu posso usar incontáveis *partes boas* da coerção, mas evitar as poucas *partes ruins*?

Vamos dar uma olhada novamente na lista *ruim*:

```js
"0" == false;			// true -- UH OH!
false == 0;				// true -- UH OH!
false == "";			// true -- UH OH!
false == [];			// true -- UH OH!
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
0 == [];				// true -- UH OH!
```

Quatro dos sete itens dessa lista envolvem a comparação `== false`, que nós dissemos anteriormente que você deve **sempre**, **sempre** evitar. Essa é uma regra bem fácil de lembrar.

Agora a lista caiu para três.

```js
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
0 == [];				// true -- UH OH!
```

São essas coerções razoáveis que você faria em um programa normal de JavaScript? Em quais condições elas realmente aconteceriam?

Eu não acho que é absurdamente provável que você use `== []` em um teste `boolean` no seu programa, ao menos não se você sabe o que está fazendo. Você propavelmente faria `== ""` ou `== 0` no lugar, como:

```js
function doSomething(a) {
	if (a == "") {
		// ..
	}
}
```

Você teria um Oops se você acidentalmente chamasse `doSomething(0)` ou `doSomething([])`. 
Outro cenário:

```js
function doSomething(a,b) {
	if (a == b) {
		// ..
	}
}
```

Novamente, isso pode quebrar se você fizesse algo como `doSomething("",0)` ou `doSomething([],"")`.


Então, embora *possam* existir situações em que essas coerções vão te pegar, e você provavelmente vai querer ter cuidado com elas, elas provavelmente não são super comuns em toda sua base de código.

#### Usando coerção implícita com segurança

O conselho mais importante que posso te dar: examine seu programa e razões sobre quais valores podem aparecer em ambos lados de uma comparação `==`. Para efetivamente evitar problemas com tais comparações, aqui estão algumas heurísticas para seguir:

1. Se ambos lados de uma comparação pode ter valores `true` ou `false`, nunca, NUNCA use `==`.

2. Se ambos lados da comparação pode ter valores `[]`, `""`, ou `0`, considere seriamente em não usar `==`.

Nesses cenários é quase sempre melhor usar `===` em vez de `==`, para evitar coerções indesejadas. Siga essas duas regras simples e basicamente todas as pegadinhas de coerção que poderiam te afetar serão efetivamente evitadas.

**Ser mais explícito/verboso nesses casos irá te salvar de muitas dores de cabeça.**

A questão de `==` vs. `===` é apropriadamente enquadrada como: você deve permitir coerção para uma comparação ou não?

Há muitos casos que tal coerção pode ser útil, permitindo que você expresse mais tersamente alguma lógica de comparação (como com `null` e `undefined`, por exemplo).

No geral, há relativamente poucos casos onde coerção *implícita* é verdadeiramente perigosa. Mas nesses lugares, por segurança, definitivamente use `===`

**Dica:** Outro lugar onde é garatido que a coerção não te prejudique é com o operador `typeof`. `typeof` sempre irá te retornar uma de sete strings (veja o Capítulo 1), e nenhuma delas serão strings vazias `""`. Sendo assim, não há nenhum caso onde checar o tipo de algum valor irá executar uma coerção *implícita*. `typeof x == "function"` é 100% tão seguro quanto `typeof x === "function"`. Literalmente, a especificação diz que o algoritmo será idêntico nesse caso. Então, não use `===` cegamente em todo lugar porque aquilo é o que as ferramentas do seu código dizem para fazer, ou (pior ainda) porque você viu em algum livro para **não pensar nisso**. A qualidade do seu código é sua.

A coerção *implícita* é maligna e perigosa? Em alguns casos, sim, mas geralmente, não.

Seja um desenvolvedor maduro e responsável. Aprenda como usar o poder da coerção (tanto *explícita* como *implícita*) efetivamente e com segurança. E ensine aqueles à sua volta a fazer o mesmo.

Aqui está uma tabela útil feita pelo Alex Dorey (@dorey no GitHub) para visualizar uma variedade de conversões:

<img src="fig1.png" width="600">

Source: https://github.com/dorey/JavaScript-Equality-Table

## Comparação Relacional Abstrata

Enquanto essa parte da coerção implícita geralmente recebe bem menos atenção, é importante pensar no que acontece com comparações `a < b` (similar à `a == b` que já examinamos em profundidade).

O algoritmo da "Comparação relacional abstrata" na seção 11.8.5 do ES5 essencialmente se divide em duas partes: o que fazer se a comparação envolve ambos valores `string` (segunda metade), ou qualquer outra coisa (primeira metade).

**Observação:** O algoritmo é apenas definido por  `a < b`. Então, `a > b` é manipulado como `b < a`.

O algoritmo primeiro chama a coerção `ToPrimitive` de ambos os valores, e se o resultado de qualquer uma das chamadas não for uma `string`, então ambos valores são convertidos para valores `number` usando as regras de operação `ToNumber`, e comparado numericamente.

Por exemplo:

```js
var a = [ 42 ];
var b = [ "43" ];

a < b;	// true
b < a;	// false
```

**Observação:** As ressalvas semelhantes para `-0` e `NaN` aplicam-se aqui como feitas no algoritmo `==` discutido anteriormente.

Entretanto, se ambos os valores são `string` para a comparação `<`, a comparação lexográfica simples (alfabético natural) é performada nos caracteres:

```js
var a = [ "42" ];
var b = [ "043" ];

a < b;	// false
```

`a` e `b` não são convertidos para `number`, porque ambos terminam como `string` depois da conversão `ToPrimitive` nos dois `array`s. Então, `"42"` é comparado caractere por caractere com `"043"`, começando com os primeiros caracteres `"4"` e `"0"`, respectivamente. Como `"0"` é lexicograficamente *menor que* `"4"`, a comparação retorna `false`.

Exatamente o mesmo comportamento e objetivo acontece para:

```js
var a = [ 4, 2 ];
var b = [ 0, 4, 3 ];

a < b;	// false
```

Aqui, `a` torna-se `"4,2"` e `b` torna-se `"0,4,3"`, e esses se comparam lexicograficamente de forma idêntica ao trecho anterior.

E sobre:

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// ??
```

`a < b` também é `false`, porque `a` torna-se `[object Object]` e `b` torna-se `[object Object]`, e então claramente `a` não é lexograficamente menor que `b`.

Mas estranhamente:

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// false
a == b;	// false
a > b;	// false

a <= b;	// true
a >= b;	// true
```

Por que `a == b` não é `true`? Eles são o mesmo valor de `string` (`"[object Object]"`), então parece que eles deveriam ser iguais, certo? Não. Relembre a discussão anterior sobre como `==` funciona com referêcias de `object`.

Mas então como `a <= b` e `a >= b` resultam em `true`, se `a < b` **e** `a == b` **e** `a > b` são todos `false`?

Porque a especificação diz que para `a <= b`, ele vai na verdade avaliar primeiro `b < a`, e então negar esse resultado. Desde que `b < a` seja *também* `false`, o resultado de `a <= b` é `true`.

Isso provavelmente é muito contrário de como você teria explicado o que o `<=` faz até agora, o que provavelmente teria sido o literal: "menor que *ou* igual a." O JS mais precisamente considera `<=` como "não é maior que" (`!(a > b)`, que o JS trata como `!(b < a)`). Além disso, `a >= b` é explicado primeiro considerando-o como `b <= a`, e então aplicando o mesmo reciocícnio.

Infelizmente, não existe "comparação relacional estrita" como é para a igualdade. Em outras palavras, não há como prevenir coerção *implícita* de ocorrer com comparações relacionais como `a < b`, além de garantir que `a` e `b` são explicitamente do mesmo tipo antes de ser feita a comparação.

Use o mesmo raciocínio para nossa discussão anterior sobre teste de sanidade `==` vs. `===`. Se a coerção é útil e razoavelmente segura, como em uma comparação de `42 < "43"`, **use-a**. Por outro lado, se você precisa ter certeza sobre uma comparação relacional, faça a *coerção explícita* dos valores primeiro, antes de usar `<` (ou suas contrapartes).

```js
var a = [ 42 ];
var b = "043";

a < b;						// false -- comparação de string!
Number( a ) < Number( b );	// true -- comparação de number!
```

## Revisão

Nesse capítulo, nós voltamos nossa atenção para como as conversões de tipos acontecem no JavaScript, chamadas **coerção**, na qual pode ser caracterizada como *explícita* ou *implícita*.

Coerção tem uma má reputação, mas ela é na verdade bastante útil em muitos casos. Uma tarefa importante para um desenvolvedor JavaScript responsável é tirar um tempo para aprender todas as entradas e saídas da coerção para decidir quais partes irão ajudar a melhorar seu código, e quais partes ele realmente deve evitar.

Coerção *explícita* é o código que é óbvio que a intenção é converter um valor de um tipo para outro. O benefício é a melhora na legibilidade e manutenabilidade do código reduzindo a confusão.

Coerção *implícita* é a coerção que está "escondida" como um efeito colateral de alguma outra operação, onde não é tão óbvio o tipo de conversão que vai acontecer. Enquanto parece que a coerção *implícita* é o oposto da *explícita*, e portanto, é ruim (e de fato, muitos pensam que sim!), na verdade, coerção *implícita* é também sobre melhorar a legibilidade do código.

Especialmente para *implícita*, coerção deve ser usada com responsabilidade e conscientemente. Saber por que você está escrevendo o código que está escrevendo, e como ele funciona. Esforce-se para escrever códigos que outros facilmente possam aprender e entender também.
