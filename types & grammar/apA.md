# You Don't Know JS: Tipos e Gramática
# Apêndice A: JavaScript em Ambiente Misto

Além da mecânica principal da linguagem que exploramos completamente neste livro, há vários comportamentos diferentes que seu código JS pode apresentar quando ele roda no mundo real. Se o JS estava sendo executando puramente dentro do motor, ele pode ser totalmente previsível baseado em nada além da especificação. Mas JS quase sempre roda no contexto de um ambiente hosteado, o que expôe seu código à alguns graus de imprevisibilidade.

Por exemplo, quando seu código roda simultaneamente com códigos de outras fontes, ou quando seu código roda em diferentes tipos de Motores JS (não apenas navegadores), há algumas coisas que podem se comportar diferente.

Nós vamos explorar brevemente alguns destes casos.

## Anexo B (ECMAScript)

É um fato pouco conhecido que o nome oficial da linguagem é ECMAScript (referindo-se ao conjunto de padrões ECMA que o gerencia). O que então é "JavaScript"? JavaScript é o nome comercial da linguagem, é claro, mas mais apropriadamente, JavaScript é basicamente a implementação desta especificação nos navegadores.

A especificação oficial ECMAScript inclui o "Anexo B", que discute derivações específicas da especificação oficial para os propósitos de compatibilidade do JS nos navegadores.

O jeito apropriado de considerar essas derivações é que elas são confiavelmente válidas/presentes somente se seu código estiver rodando em um navegador. Se seu código sempre roda em navegadores, você não verá nenhuma diferença considerável. Caso contrário (como se pudesse ser executado em node.js, Rhino, etc.), ou você não tem certeza, vá com cuidado.

As pricipais diferenças de compatibilidade:

* Números Octais são permitidos, como `0123` (decimal `83`) em modo não estrito(*non-`strict-mode`*).
* `window.escape(..)` e `window.unescape(..)` permitem que você escape ou não strings com `%`-sequências de escape delimitadas. Por exemplo: `window.escape( "?foo=97%&bar=3%" )` se torna `"%3Ffoo%3D97%25%26bar%3D3%25"`.
* `String.prototype.substr` é bem similiar à `String.prototype.substring`, exceto que ao invés do segundo parâmetro ser o último index (não-inclusivo), o segundo parâmetro é o `lenght` (números de caracteres para incluir).

### Web ECMAScript

A especificação do Web ECMAScript (http://javascript.spec.whatwg.org/) abrange as difereças entre a especificação oficial do ECMAScript e a implementação atual do JavaScript nos navegadores.

Em outras palavras, esses itens são "requisitos" dos navegadores (para serem compatíveis uns com os outros) mas não são (até o momento) listados na seção "Anexo B" das especificações oficiais:

* `<!--` and `-->` são válidos como delimitadores de cometários de única linha.
* `String.prototype` adições para retornarem strings no formato HTML: `anchor(..)`, `big(..)`, `blink(..)`, `bold(..)`, `fixed(..)`, `fontcolor(..)`, `fontsize(..)`, `italics(..)`, `link(..)`, `small(..)`, `strike(..)`, and `sub(..)`. 
**Observação:** Estes exemplos raramente são usados na prática, e geralmente não são recomendados por outras API de DOM nativas ou utilitários definidos pelo usuário.
* Extensões `RegExp`: `RegExp.$1` .. `RegExp.$9` (comninações de grupos) e `RegExp.lastMatch`/`RegExp["$&"]` (combinação mais recente).
* Adições `Function.prototype`: `Function.prototype.arguments` (apelidos internos dos `arguments` do objeto) e `Function.caller` (apelidos internos de `arguments.caller`). **Observação:** `arguments` e `arguments.caller` estão obsoletos, então você deve evitar de usa-los se possível. Isso vale ainda mais para apelidos -- Não os use!

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

`a` não é só um `object`, mas um objeto global especial porque este é um elemento do DOM. Ele tem um valor interno diferente para `[[Class]]` (seu valor é `"HTMLDivElement"`) e vem com propriedades pré-definidas (e frequentemente imutáveis).

Outra peculiaridade já foi abordada, na seção "Objetos Falsos"(Falsy Objects) no capítulo 4: alguns objetos podem existir mas quando convertidos em `boolean` eles (de forma confusa) serão convertidos para `false` ao invés de `true` como seria o esperado.

Outra variação de comportamento com objetos globais para se ter cuidado podem incluir:

* não ter acesso a propriedades nativas do `object` como `toString()`
* não ser editável
* ter certas propriedades pré-definidas com "somente leitura"
* ter métodos que não podem ter o `this` substituído por outros objetos
* e mais...

Objetos globais são fundamentais para tornar nosso código JS funcional em todo ambiente. Mas é importante lembrar quando você está interagindo com um objeto global e ter cuidado ao assumir seu comportamento, pois eles quase sempre não estão em conformidade com ´object´s tradicionais.

Um exemplo notável de um objeto global que você provavelmente vai interagir regularmente é o objeto `console` functions (`log(..)`, `error(..)`, etc.). O objeto `console` é fornecido pelo *ambiente global* especificamente para que seu código possa interagir com ele para várias tarefas relacionadas ao output do desenvolvimento.

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

Depois de quase uma semana de análise/debbug, eu encontrei que o site em questão tinha, enterrado profundamente em um dos seus arquivos legados, um código parecido com isso:

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

O que se destaca é uma questão interessante que, francamente, não recebe a atenção devida dos desenvolvedores JS:
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

A maioria dos websites/aplicações web têm mais de um arquivo que contém seus códigos, e é comum ter poucos ou muitos elementos `<script src=..></script>` na página que carrega os arquivos separadamente, e até um pouco de código inline dentro do `<script> .. </script>` também.

Mas estes snippets de arquivos/códigos separados constituem programas separados ou eles são, coletivamente, um só programa em JS?

A (talvez surpreendente) realidade é que eles agem mais como programas JS independentes na maior parte das vezes, mas nem sempre, isso é respeitado.

A única coisa que eles *compartilham* é um único objeto `global` (`window` no navegador), o que significa que múltiplos arquivos podem anexar seu código nesse espaço compartilhado e todos podem interagir·

Então, se um elemento de um `script` definir uma função global `foo()`, quando um segundo `script` executar depois, ele poderá acessar e chamar `foo()` como se ele próprio tivesse definido essa função.

Mas a *elevação* do escopo de variáveis globais (veja em *Escopo & Clausuras* dessa série) não ocorre através desse limite, então o código seguinte não funcionará (porque a declaração de `foo()` ainda não foi declarada), independentemente de terem (como mostrado) elementos `<script> .. </script>` inline ou arquivos carregados externamente `<script src=..></script>`:

```html
<script>foo();</script>

<script>
  function foo() { .. }
</script>
```

Mas qualquer um desses *poderia* funcionar no lugar:

```html
<script>
  foo();
  function foo() { .. }
</script>
```

Ou:

```html
<script>
  function foo() { .. }
</script>

<script>foo();</script>
```

Também, se acontecer um erro num elemento `script` (inline ou externo), como um programa independente e separado, ele irá falhar e parar, mas qualquer `script`s subsequente irá executar (continua com `global` compartilhado) sem obstáculos.

Você pode criar elementos `script` dinamicamente no seu código, e injetá-los no DOM da página, e o código nele vai se comportar, basicamente, como se tivesse carregado normalmente em um arquivo separado:

```js
var greeting = "Hello World";

var el = document.createElement( "script" );

el.text = "function foo(){ alert( greeting );\
 } setTimeout( foo, 1000 );";

document.body.appendChild( el );
```

**Observação** é claro, se você tentar o snippet acima mas definir `el.src` para algum arquivo em URL externa em vez de definir `el.text` para os conteúdos do código, você estará criando um elemento de carregamento de `<script src=..></script>` externo dinamicamente.

Uma diferença entre o código em um bloco de código inline e esse mesmo código em um arquivo externo é que aquele no bloco de código inline, a sequência dos caracteres `</script>` não podem aparecer juntos, assim (independentemente de onde aparecerá) ele vai ser interpretado como o final do bloco de código. Então, cuidado com códigos como esse:

```html
<script>
  var code = "<script>alert( 'Hello World' )</script>";
</script>
```

Parece inofensivo, mas o `</script>` aparecendo dentro de uma `string` literal terminará o bloco de script anormalmente, causando um erro. A solução alternativa mais comum é:  

```js
"</sc" + "ript>";
```

Também, tenha cuidado com códigos dentro de arquivos externos que serão interpretados com conjuntos de catacteres (UTF-8, ISO-8859-8, etc.) o arquivo é servido com (ou o padrão), mas esse mesmo código em um `script` inline elemento na sua página HTML será interpretado pelo conjunto de caracteres da página (ou seu padrão).

**Atenção** O atributo `charset` não funciona em elementos de script inline.

Outra prática obsoleta com elementos de `script` inline é incluir comentários HTML-style or X(HT)ML-style no código inline, dessa forma:

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

Os dois casos são totalmente desnecessários agora, então se você continua fazendo isso, pare!

**Observação:** Ambos `<!--` e `-->` (comentários HTML) são atualmente especificados como delimitadores de linha única válidos (`var x = 2; <!-- comentário válido` e `--> outra linha de comentário válida`) no Javascript (veja a seção anterior "Web ECMAScript"), puramente por causa dessa técnica antiga. Mas nunca use eles.

## Palavras Reservadas

A especificação ES5 define um conjunto de "palavras reservadas" na seção 7.6.1 que não podem ser usadas como nomes de variáveis independentes. Tecnicamente, há quatro categorias: "palavras-chave", "palavras reservadas para o futuro", o literal `null`, e os literais booleanos `true` / `false`.

Palavras-chave são as mais óbvias, como `function` e `switch`. Palavras reservadas para o futuro incluem coisas como `enum`, embora muitas outras (`class`, `extends`, etc.) são agora utilizadas pela ES6; há outro modo estrito que apenas reservam palavras como `interface`.

O usuário "art4theSould" do StackOverflow, criativamente, trabalhou todas essas palavras reservadas em um pequeno e engraçado poema:
(http://stackoverflow.com/questions/26255/reserved-keywords-in-javascript/12114140#12114140)

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

**Observação:** Esse poema inclui palavras que foram reservadas na ES3 (`byte`, `long`, etc.) que não são mais reservadas na ES5.

Antes do ES5, as palavras reservadas também não podiam ser nomes de propriedades e chaves em objetos literais, mas essa restrição não existe mais.

Então, isso não é permitido:

```js
var import = "42";
```

Mas isso é permitido:

```js
var obj = { import: "42" };
console.log( obj.import );
```

Você deve tomar cuidado com algumas versões antigas de navegadores (principalmente IE antigos) essas regras não estão totalmente aplicadas de forma consistente, então há lugares em que usar palavras reservadas em nomes de propriedades de objetos ainda podem causar conflitos. Cuidadosamente teste todos os ambientes de navagadores suportados.

## Limites de Implementação

A especificação do Javascript não coloca arbritariamente limites nas coisas, como o número de argumentos para uma função ou comprimento de uma string literal, no entanto, esses limites existem por causa dos detalhes de implementação em diferentes motores.

Por exemplo:

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
addAll.apply( null, nums );		// deverá ser: 499950000
```

Em alguns motores JS, você receberá a resposta correta de `499950000`, mas em outros (como o Safari 6.x), você receberá o erro: "RangeError: Maximum call stack size exceeded".

Exemplos de outros limites conhecidos:

* número máximo de caracteres permitidos para uma string literal (não apenas o valor da string)
* tamanho (bytes) dos dados que podem ser enviados no argumento para a chamada de uma função
* número de parâmetros na declaração de uma função
* profundidade máxima da pilha de chamadas não otimizadas (i.e., com recursão): quanto tempo a chamada de um para o outro uma cadeia de função pode ter
* número de segundos que um programa JS pode executar continuamente bloqueando o navegador
* tamanho máximo permitido para o nome de uma variável
* ...

## Revisão

Sabemos que podemos confiar que a própria linguagem JS tem um padrão e é previsivelmente implementada por todos os navagadores/motores modernos. Isso é uma coisa muito boa!

Mas o javascript raramente executa isoladamente. Ele roda em um abiente misto e com códigos de bibliotecas terceiras, e algumas vezes até roda em motores/ambientes que diferem daqueles encontrados nos navagadores.

Prestar muita atenção para essas questões melhora a confiabilidade e robustez do seu código.
