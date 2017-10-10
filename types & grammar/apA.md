# You Don't Know JS: Tipos e Gramática
# Apêndice A: JavaScript em Ambiente Misto

Além da mecânica principal da linguagem que exploramos completamente neste livro, há vários comportamentos diferentes que seu código JS pode apresentar quando ele roda no mundo real. Se o JS estava sendo executando puro dentro do motor, ele pode ser totalmente previsível baseado em nada além da especificação. Mas JS quase sempre roda no contexto de um ambiente hosteado, o que expôe seu código à alguns graus de imprevisibilidade.

Por exemplo, quando seu código roda simultaneamente com códigos de outras fontes, ou quando seu código roda em diferentes tipos de Motores JS (não apenas navegadores), há algumas coisas que podem se comportar diferente.

Nós vamos explorar brevemente alguns destes casos.

## Anexo B (ECMAScript)

É um pouco conhecido o fato que o nome oficial da linguagem é ECMAScript (referindo-se ao conjunto de padrões ECMA que o gerencia). O que então é "JavaScript"? JavaScript é o nome comercial da linguagem, é claro, mas mais apropriadamente, JavaScript é basicamente a implementação desta especificação nos navegadores.

A especificação oficial ECMAScript inclui o "Anexo B", que discute derivações específicas da especificação oficial para os propósitos de compatibilidaded do JS nos navegadores.

O jeito apropriado de considerar essas derivações é que elas são confiavelmente válidas/presentes somente se seu código estiver rodando em um navegador. Se seu código sempre roda em navegadores, você não verá nenhuma diferença considerável. Caso contrário (como se pudesse ser executado em node.js, Rhino, etc.), ou você não tem certeza, vá com cuidado.

As pricipais diferenças de compatibilidade:

* Números Octais são permitidos, como `0123` (decimal `83`) em non-`strict mode`.
* `window.escape(..)` e `window.unescape(..)` permitem que você escape ou não strings com `%`-sequências de escape delimitadas. Por exemplo: `window.escape( "?foo=97%&bar=3%" )` se torna `"%3Ffoo%3D97%25%26bar%3D3%25"`.
* `String.prototype.substr` é bem similiar à `String.prototype.substring`, exceto que ao invés do segundo parâmetro ser o último index (noninclusive), o segundo parâmetro é o `lenght` (números de caracteres para incluir).

### Web ECMAScript

A especificação do Web ECMAScript (http://javascript.spec.whatwg.org/) abrange as difereças entre a especificação oficial do ECMAScript e a implementação atual do JavaScript nos navegadores.

Em outras palavras, esses itens são "requisitos" dos navegadores (para serem compatíveis uns com os outros) mas não são (até o momento) listados na seção "Anexo B" das especificações oficiais:

* `<!--` and `-->` são válidos como delimitadores de cometários de única linha.
* `String.prototype` adições para retornarem strings no formato HTML: `anchor(..)`, `big(..)`, `blink(..)`, `bold(..)`, `fixed(..)`, `fontcolor(..)`, `fontsize(..)`, `italics(..)`, `link(..)`, `small(..)`, `strike(..)`, and `sub(..)`. 
**Observação:** Estes exemplos raramente são usados na prática, e geralmente não são recomendados por outras API de DOM nativas ou utilitários definidos pelo usuário.
* Extensões `RegExp`: `RegExp.$1` .. `RegExp.$9` (comninações de grupos) e `RegExp.lastMatch`/`RegExp["$&"]` (combinação mais recente).
* Adições `Function.prototype`: `Function.prototype.arguments` (apelidos internos dos `arguments` do objeto) e `Function.caller` (apelidos internos de `arguments.caller`). **Observação:** `arguments` e `arguments.caller` estão obsoletos, então você deve evitar de usa-los se porssível. Isso vale ainda mais para apelidos -- Não os use!

**Observação:** Algumas outras variações que são raramente utilizadas não foram incluídas aqui em nossa lista. Veja a documentação do "Anexo B" e "Web ECMAScript" para informações mais detalhadas se necessário.

Falando no geral, todas estas diferenças são raramente utilizadas, então as derivações da especificação não são preocupações significativas. **Apenas tenha cuidado** ao contar com algumas delas.

## Objetos Globais

As regras de boas práticas de como as variáveis se comportam em JS têm exceções quando se tratam de variáveis auto definidas, ou criadas e fornecidas de outra forma no JS pelo ambiente que hospeda o seu código (navegador, etc.) -- então chamados, "objetos globais" (no qual ambos incluem `objects` e `functions`s nativas).

Por exemplo:

```js
var a = document.createElement( "div" );

typeof a;								// "object" -- como esperado
Object.prototype.toString.call( a );	// "[object HTMLDivElement]"

a.tagName;								// "DIV"
```

`a` não é só um `object`, mas um objeto global especial porque este é um elemento do DOM. Ele tem um `[[Class]]` valor interno (`"HTMLDivElement"`) diferente e vem com propriedades pré-definidas (e frequentemente imutáveis).

Outra peculiaridade já foi abordada, na seção "Objetos Falsos"(Falsy Objects) no capítulo 4: alguns objetos podem existir mas quando forçados em `boolean` eles (de forma confusa) serão forçados para `false` ao invés de `true` como seria o esperado.

Outra variação de comportamento com objetos globais para se ter cuidado podem incluir:

* não ter acesso a propriedades nativas do `object` como `toString()`
* não ser editável
* ter certas propriedades pré-definidas com "somente leitura"
* ter métodos que não podem ter o `this` substituído por outros objetos
* e mais...

Objetos globais são fundamentais para tornar nosso código JS funcional em todo ambiente. Mas é importante lembrar quando você está interagindo com um objeto global e ter cuidado ao assumir seu comportamento, pois eles quase sempre não estão em conformiadade com ´object´s tradicionais.

Um exemplo notável de um objeto global que você provavelmente vai interagir regularmente é o objeto `console` que é fornecido pelo *ambiente global* especificamente para que seu código possa interagir com ele por várias tarefas de relacionadas ao desenvolvimento em produção.

Em navegadores, o `console` está atrelado ao console de ferramentas do desenvolvedor, enquanto em node.js e outros ambientes JS do lado do servidor, `console` é geralmente à saída padrão (`stdout`) e depurador de erros (`stderr`) do fluxo de processos do sistema de desenvolvimento em JS.

## Variáveis Globais do DOM

Você provavelmente está ciente que declarar uma variável no escopo global (com ou sem `var`) cria não só uma variável global, mas também um espelho: uma propriedade de mesmo nome no objeto `global` (`window` no navegador).

Mas o que talvez possa ser de pouco conhecimento é que (por causa do compartamento legado do navegador) criar elementos do DOM com atributos `id` cria variáveis globais com esses mesmos nomes. Por exemplo:

```html
<div id="foo"></div>
```

E:

```js
if (typeof foo == "undefined") {
	foo = 42;		// nunca vai rodar
}

console.log( foo );	// elemento HTML
```

Você talvez esteja acostumado a gerenciar variáveis globais (usando `typeof` ou checagens `.. in window`) assumindo qe somente código JS cria tais variáveis, mas como você pode ver, o conteúdo da sua página HTML Global também pode criá-los, o que pode facilmente derrubar toda sua lógica já existente se você não for cuidadoso.

Esta é mais uma razão do porque você deve, sempre que possível, evitar o uso de variáveis globais, e se for necessário, use variáveis com nomes únicos que não causarão conflitos tão facilmente. Mas você também precisa ter certeza que não vá ter conflitos com o conteúdo HTML tanto quanto com qualquer outro código.

## Prototypes Nativos

Uma das mais conhecidas e clássicas *melhores práticas* de Javascript é: **nunca extenda prototypes nativos**.

Qualquer nome de método ou propriedade que você tenha que adicionar em um `Array.prototype` que (ainda) não existe, se for uma adição útil e bem estruturada, e nomeada apropriadamente, há uma grande chance que isso *possa* eventualmente acabar sendo adicionado na especificação -- nesse caso, sua extensão está agora em conflito.

Aqui está um exemplo real do que realmente aconteceu comigo que ilustra bem esse ponto.

Eu estava construindo um widget embedado para outros websites, e meu widget dependia do JQuery (embora praticamente qualquer estrutura sofreu com essa pegadinha). Ele funcionava em quase todos os sites, mas nós nos deparamos com um que estava totalmente quebrado.

Depois de quase uma semana de análise/debbug, eu encontrei que o que o site em questão tinha, enterrado profundamente em um dos seus arquivos legados, um código parecido com isso:

```js
// Netscape 4 não possui Array.push
Array.prototype.push = function(item) {
	this[this.length] = item;
};
```

Além desse comentário louco (quem ainda liga para o netscape 4!?), isso parecia razoável, certo?

O problema é que, `Array.prototype.push` foi adicionado à especificação em um momento subsequente à esse código do nescape 4, mas o que foi adicionado não era compatível com esse código. O padrão `push(...)` permitia que múltiplos itens fossem inseridos de uma vez. Esse hack ignorava os itens subsequentes.

Basicamente todos os frameworks JS tem algum código que dependem de `push(...)``com múltiplos elementos. No meu caso, era um código ao redor da engine de seletores CSS que foi completamente bloqueado. Mas poderiam haver dúzias de outros lugares suscetíveis.

O desenvolvedor que originalmente escreveu aquele hack de `push(...)` tinha bons instintos para chamá-lo de `push`, mas não previu dar `push` em múltiplos elementos. Eles certamente agiram com boa vontade, mas eles criaram uma bomba que não explodiu até que quase 10 anos depois eu, involuntariamente, apareci.

Há várias lições para aprender de todos os lados.

Primeiro, não extenda propriedades nativas a menos que tenha absoluta certeza que seu código será o único código que vai rodar naquele ambiente. Se você não pode ter 100% de certeza, então extender propriedades nativas é perigoso. Você deve medir os riscos.

Próximo, não defina incondicionalmente as extensões (porque você pode substituir propriedades nativas acidentalmente). Nesse exemplo em particular, temos um código assim:

```js
if (!Array.prototype.push) {
	// Netscape 4 não possui Array.push
	Array.prototype.push = function(item) {
		this[this.length] = item;
	};
}
```

A declaração `if` garante que você somente definiu esse hack de `push()` em ambientes JS que ele não existe. No meu caso, isso provavelmente seria ok. Mas mesmo essa abordagem não é livre de riscos:

1. Se o código do site (por alguma razão louca!) estava dependendo de um `push(...)` que ignorou múltiplos itens, aquele código terá quebrado anos antes quando o padrão `push(..)` foi lançado.
2. Se alguma outra biblioteca apareceu e fez um hack de `push(..)` com a garantia do `if`, e o fez de forma incompatível, ela já teria quebrado o site naquele momento.

O que se destaca é uma questão interessandte que, francamente, não recebe a atenção devida dos desenvolvedores JS:
**Nunca deve-se depender de um comportamento legado** seu código está em execução em qualquer ambiente em que não seja o único código presente?

A resposta estrita é **não**, mas é terrivelmente impraticável. Seu código geralmente não pode redefinir suas próprias versões privadas ​​de todo o comportamento dependente do legado. Mesmo se *pudesse*, isso seria muito desperdício.

Então, você deveria fazer testes para comportamentos legados assim como testes de conformidade que faz o que você espera? E se esse teste falha -- seu código seria impedido de executar?

```js
// não confie no Array.prototype.push
(function(){
	if (Array.prototype.push) {
		var a = [];
		a.push(1,2);
		if (a[0] === 1 && a[1] === 2) {
			// passou no teste, seguro para usar!
			return;
		}
	}

	throw Error(
		"Array#push() está faltanto/quebrado!"
	);
})();
```

Em teoria, isso parece plausível, mas também é bem impraticável estruturar testes para cada um dos métodos legados.

Então, o que devemos fazer? Devemos *confiar mas verificar* (testes de funcionalidade e conformidade) **em tudo**? Podemos somente assumir que a existência é conformidade e deixar a quebra (causada por outros) se disseminar livremente?

Não existe uma grande resposta. O único fato observável é que extender prototypes nativos é o único meio dessas coisas te atingirem.

Se você não fizer, e nignuém mais fizer isso no código da sua aplicação, você está seguro. 
De outra forma, você deve construir um pouco de cetitcismo, pessimismo, e expectativas de possíveis rupturas.

Ter um conjunto completo de testes de unidade/regressão do seu código que executa em todos os ambientes possíveis é uma maneira de enfrentar alguns desses problemas anteriores, mas isso não faz nada para realmente protege-lo desses conflitos.

### Shims/Polyfills

É geralmente dito que o único lugar seguro para se extender uma propriedade nativa é num velho (especificação não compilada) ambiente, desde que é improvável que este mude algum dia -- novos navegadores com novas funcionalidades de especificações substituem navegadores mais antigos em vez de modificá-los.

Se você pudesse ver no futuro, e saber com certeza qual padrão o futuro terá para `Array.prototype.foobar`, aí seria totalmente seguro fazer sua própria versão compatível para utilizar, certo?

```js
if (!Array.prototype.foobar) {
	// sabe de nada, inocente
	Array.prototype.foobar = function() {
		this.push( "foo", "bar" );
	};
}
```
Se já existe uma especificação para `Array.prototype.foobar`, e o comportamento especificado é equivalente à sua lógica, você está bem seguro ao definir este snippet, e nesse caso é geralmente chamado "polyfill" (ou "shim").

Esse código é **muito** útil de se incluir em sua base de código para "corrigir" ambientes de navegadores antigos que não se atualizaram às novas especificações. Usar os polyfills é uma excelente maneira de criar códigos previsíveis em todos os seus ambientes suportados.

**Dica** ES5-Shim (https://github.com/es-shims/es5-shim) é uma coleção abrangente de shims/polyfills para trazer um projeto para a base do ES5, e similarmente, ES6-Shim (https://github.com/es-shims/es6-shim) oferece shims para novas APIs adicionadas a partir do ES6. Equanto as APIs podem ter seus shims/polyfills, novas sintaxes geralmente não. Para contornar essa divisão sintática, você também deverá usar um transpilador, como o Traceur (https://github.com/google/traceur-compiler/wiki/GettingStarted).

Se (provavelmente) surgir um novo padrão, a maioria das discussões concordará o que será chamado e como isso vai funcionar, criando um polyfill que antecipará as conformidades com padrões voltados para o futuro, isso é chamado "prollyfill"(probably-fill).

O real problema é se algum novo padrão de comportamento não puder (totalmente) ter um polyfill/prollyfill.

Há um debate na comunidade com polyfills parciais para os casos comuns em que é aceitável (documentar as partes que não podem ter um polyfill), ou se o polyfill deve ser evitado se não puder ter 100% de conformidade com a especificação.

Muitos desenvolvedores, pelo menos aceitam, alguns polyfills parciais mais comuns (como por exemplo `Object.create(..)`), porque as partes que não são suportadas, são partes que eles não têm a intenção de usar, de qualquer maneira.

Alguns desenvolvedores acreditam que o `if` que envolve um polyfill/shim deve incluir alguma forma de teste de conformidade, substituindo o método existente se este está ausente ou se os testes falharem. Essa camada extra de teste de conformiade é algumas vezes usada para distinguir "shim" (testado a conformidade) de "pollyfill" (testado a existência).

A única saída absoluta é que não há uma resposta totalmente *certa* aqui. Extender propriedades nativas, mesmo quando feito de forma *segura* em ambientes antigos, não é 100% seguro. O mesmo vale para depender de (possívelmente extendidas) propriedades nativas quando há códigos de outras pessoas.

Em qualquer um dos casos, deverá sempre ser feito com cuidado, código defensivo, e obviamente, muitas documentações sobre os riscos.

## `<script>`s

Most browser-viewed websites/applications have more than one file that contains their code, and it's common to have a few or several `<script src=..></script>` elements in the page that load these files separately, and even a few inline-code `<script> .. </script>` elements as well.

But do these separate files/code snippets constitute separate programs or are they collectively one JS program?

The (perhaps surprising) reality is they act more like independent JS programs in most, but not all, respects.

The one thing they *share* is the single `global` object (`window` in the browser), which means multiple files can append their code to that shared namespace and they can all interact.

So, if one `script` element defines a global function `foo()`, when a second `script` later runs, it can access and call `foo()` just as if it had defined the function itself.

But global variable scope *hoisting* (see the *Scope & Closures* title of this series) does not occur across these boundaries, so the following code would not work (because `foo()`'s declaration isn't yet declared), regardless of if they are (as shown) inline `<script> .. </script>` elements or externally loaded `<script src=..></script>` files:

```html
<script>foo();</script>

<script>
  function foo() { .. }
</script>
```

But either of these *would* work instead:

```html
<script>
  foo();
  function foo() { .. }
</script>
```

Or:

```html
<script>
  function foo() { .. }
</script>

<script>foo();</script>
```

Also, if an error occurs in a `script` element (inline or external), as a separate standalone JS program it will fail and stop, but any subsequent `script`s will run (still with the shared `global`) unimpeded.

You can create `script` elements dynamically from your code, and inject them into the DOM of the page, and the code in them will behave basically as if loaded normally in a separate file:

```js
var greeting = "Hello World";

var el = document.createElement( "script" );

el.text = "function foo(){ alert( greeting );\
 } setTimeout( foo, 1000 );";

document.body.appendChild( el );
```

**Note:** Of course, if you tried the above snippet but set `el.src` to some file URL instead of setting `el.text` to the code contents, you'd be dynamically creating an externally loaded `<script src=..></script>` element.

One difference between code in an inline code block and that same code in an external file is that in the inline code block, the sequence of characters `</script>` cannot appear together, as (regardless of where it appears) it would be interpreted as the end of the code block. So, beware of code like:

```html
<script>
  var code = "<script>alert( 'Hello World' )</script>";
</script>
```

It looks harmless, but the `</script>` appearing inside the `string` literal will terminate the script block abnormally, causing an error. The most common workaround is:

```js
"</sc" + "ript>";
```

Also, beware that code inside an external file will be interpreted in the character set (UTF-8, ISO-8859-8, etc.) the file is served with (or the default), but that same code in an inline `script` element in your HTML page will be interpreted by the character set of the page (or its default).

**Warning:** The `charset` attribute will not work on inline script elements.

Another deprecated practice with inline `script` elements is including HTML-style or X(HT)ML-style comments around inline code, like:

```html
<script>
<!--
alert( "Hello" );
//-->
</script>

<script>
<!--//--><![CDATA[//><!--
alert( "World" );
//--><!]]>
</script>
```

Both of these are totally unnecessary now, so if you're still doing that, stop it!

**Note:** Both `<!--` and `-->` (HTML-style comments) are actually specified as valid single-line comment delimiters (`var x = 2; <!-- valid comment` and `--> another valid line comment`) in JavaScript (see the "Web ECMAScript" section earlier), purely because of this old technique. But never use them.

## Reserved Words

The ES5 spec defines a set of "reserved words" in Section 7.6.1 that cannot be used as standalone variable names. Technically, there are four categories: "keywords", "future reserved words", the `null` literal, and the `true` / `false` boolean literals.

Keywords are the obvious ones like `function` and `switch`. Future reserved words include things like `enum`, though many of the rest of them (`class`, `extends`, etc.) are all now actually used by ES6; there are other strict-mode only reserved words like `interface`.

StackOverflow user "art4theSould" creatively worked all these reserved words into a fun little poem (http://stackoverflow.com/questions/26255/reserved-keywords-in-javascript/12114140#12114140):

> Let this long package float,
> Goto private class if short.
> While protected with debugger case,
> Continue volatile interface.
> Instanceof super synchronized throw,
> Extends final export throws.
>
> Try import double enum?
> - False, boolean, abstract function,
> Implements typeof transient break!
> Void static, default do,
> Switch int native new.
> Else, delete null public var
> In return for const, true, char
> …Finally catch byte.

**Note:** This poem includes words that were reserved in ES3 (`byte`, `long`, etc.) that are no longer reserved as of ES5.

Prior to ES5, the reserved words also could not be property names or keys in object literals, but that restriction no longer exists.

So, this is not allowed:

```js
var import = "42";
```

But this is allowed:

```js
var obj = { import: "42" };
console.log( obj.import );
```

You should be aware though that some older browser versions (mainly older IE) weren't completely consistent on applying these rules, so there are places where using reserved words in object property name locations can still cause issues. Carefully test all supported browser environments.

## Implementation Limits

The JavaScript spec does not place arbitrary limits on things such as the number of arguments to a function or the length of a string literal, but these limits exist nonetheless, because of implementation details in different engines.

For example:

```js
function addAll() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		sum += arguments[i];
	}
	return sum;
}

var nums = [];
for (var i=1; i < 100000; i++) {
	nums.push(i);
}

addAll( 2, 4, 6 );				// 12
addAll.apply( null, nums );		// should be: 499950000
```

In some JS engines, you'll get the correct `499950000` answer, but in others (like Safari 6.x), you'll get the error: "RangeError: Maximum call stack size exceeded."

Examples of other limits known to exist:

* maximum number of characters allowed in a string literal (not just a string value)
* size (bytes) of data that can be sent in arguments to a function call (aka stack size)
* number of parameters in a function declaration
* maximum depth of non-optimized call stack (i.e., with recursion): how long a chain of function calls from one to the other can be
* number of seconds a JS program can run continuously blocking the browser
* maximum length allowed for a variable name
* ...

It's not very common at all to run into these limits, but you should be aware that limits can and do exist, and importantly that they vary between engines.

## Review

We know and can rely upon the fact that the JS language itself has one standard and is predictably implemented by all the modern browsers/engines. This is a very good thing!

But JavaScript rarely runs in isolation. It runs in an environment mixed in with code from third-party libraries, and sometimes it even runs in engines/environments that differ from those found in browsers.

Paying close attention to these issues improves the reliability and robustness of your code.
