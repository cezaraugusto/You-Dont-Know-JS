# You Don't Know JS: ES6 & Além
# Chapter 1: ES? Presente e Futuro

Antes de mergulhar neste livro, você deve ter uma sólida proficiência de trabalho sobre o JavaScript até o padrão mais recente (no momento da redação deste artigo), que é comumente chamado de *ES5* (tecnicamente `ES 5.1`). Aqui, planejamos falar diretamente sobre o próximo *ES6*, também como moldar nossa visão além de entender como o JavaScript irá evoluir.

Se você está "entrando de cara" com o JavaScript, eu recomendo que você leia os outros títulos desta série primeiro:

* *Iniciando*: Você é novo em programação e JavaScript? Este é o roteiro que você precisa consultar enquanto inicia sua jornada de aprendizado.
* *Escopos & Encerramentos*: Você sabia que o escopo lexical do JS é baseado em uma semântica de compilador (não de intérprete)? Você pode explicar como os fechamentos são um resultado direto do escopo lexical e funções como valores?
* *this & Prototipagem de Objetos*: Você pode recitar as quatro regras simples de como o `this` está ligado? Você tem ficado confuso com falsas "classes" em JavaScript em vez de adotar o padrão de design de "delegação de comportamento" mais simples? Já ouviu falar de *objetos ligados a outros objetos*(OLOO)?
* *Tipos & Gramática*: Do you know the built-in types in JS, and more importantly, do you know how to properly and safely use coercion between types? How comfortable are you with the nuances of JS grammar/syntax?
* *Async & Performance*: Você ainda está usando callbacks para gerenciar sua assincronia? Você pode explicar o que é uma promessa e por que/como ele resolve o "inferno de retorno de chamada"? Você sabe como usar geradores para melhorar a legibilidade do código assíncrono? O que exatamente constitui a otimização madura de programas JavaScript e operações individuais?

Se você já leu todos esses títulos e se sente muito confortável com os tópicos que eles cobrem, é hora de mergulhar na evolução da JS para explorar todas as mudanças que vêm não só em breve, mas distante sobre o horizonte.

Ao contrário do ES5, o ES6 não é apenas um conjunto modesto de novas APIs adicionadas ao idioma. Ele incorpora um conjunto de novas formas sintáticas, algumas das quais podem levar um pouco de se acostumar. Há também uma variedade de novos formulários organizacionais e novos auxiliares de APIs para vários tipos de dados.

ES6 é um salto radical para a linguagem. Mesmo se você acha que sabe JavaScript no ES5, o ES6 está cheio de coisas novas que você *ainda não sabe*, então prepare-se! Este livro explora todos os principais temas do ES6 que você precisa se acostumar, e até mesmo dá-lhe um vislumbre de futuros recursos que você deve estar ciente.

**AVISO:** Todo o código neste livro possui um ambiente *ES6+*. No momento em que este texto foi escrito, O suporte ao ES6 varia muito nos navegadores e nos ambientes JavaScript (como o Node.js), de modo que sua "quilometragem" pode variar.

## Versionamento

O padrão JavaScript é referido oficialmente como "ECMAScript" (abreviado como "ES"), e até recentemente tem sido versionado inteiramente por um número ordinal (ou seja, "5" para "5 ª edição").

As primeiras versões, ES1 e ES2, não foram amplamente conhecidas ou implementadas. ES3 foi o primeira patamar generalizado para o JavaScript e constitui o padrão JavaScript para navegadores como IE6-8 e navegadores móveis Android 2.x mais antigos. Por razões políticas para além do que abordaremos aqui, o mal sucedido ES4 nunca surgiu.

Em 2009, ES5 foi oficialmente finalizado (mais tarde ES5.1 em 2011), e se estabeleceu como o padrão geral do JS para a revolução moderna e explosão de navegadores, como o Firefox, Chrome, Opera, Safari e muitos outros.

Levando até a esperada *próxima* versão do JavaScript (saltou de 2013 para 2014, e depois 2015), o rótulo óbvio e comum no discurso tem sido o ES6.

No entanto, no final do cronograma de especificações do ES6, surgiram sugestões de que o controle de versão pode no futuro, mudar para um esquema baseado no ano, como ES2016 (também conhecido como ES7) para se referir a qualquer versão da especificação finalizada antes do final de 2016. Alguns discordam, mas o ES6 provavelmente manterá sua dominante `mindshare` sobre o substituto de mudança final, ES2015. No entanto, o ES2016 de fato pode sinalizar a esquema baseado em ano novo.

Também foi observado que o ritmo de evolução do JavaScript é muito mais rápido do que o versionamento de um ano. Assim que uma idéia começa a progredir por meio de discussões sobre padrões, os navegadores começam a prototipar o recurso e os adotantes iniciais, experimentam o código.

Normalmente, bem antes de haver um selo oficial de aprovação, um recurso é de fato padronizado em virtude desta prototipagem inicial de motor/ferramentas. Por isso, também é válido considerar o futuro do versionamento de JavaScript, como sendo por recurso em vez de por coleção arbitrária de recursos principais (como é agora) ou mesmo por ano (como pode ser).

O `takeaway` é que os rótulos de versionamento deixam de ser tão importante e, o JavaScript começa a ser visto mais como um perene padrão de vida. A melhor maneira de lidar com isso é parar de pensar em sua base de código como sendo "baseado em ES6", por exemplo, em vez disso, considerar o recurso por recurso para suporte.

## Transpilando

Made even worse by the rapid evolution of features, a problem arises for JS developers who at once may both strongly desire to use new features while at the same time being slapped with the reality that their sites/apps may need to support older browsers without such support.

The way ES5 appears to have played out in the broader industry, the typical mindset was that code bases waited to adopt ES5 until most if not all pre-ES5 environments had fallen out of their support spectrum. As a result, many are just recently (at the time of this writing) starting to adopt things like `strict` mode, which landed in ES5 over five years ago.

It's widely considered to be a harmful approach for the future of the JS ecosystem to wait around and trail the specification by so many years. All those responsible for evolving the language desire for developers to begin basing their code on the new features and patterns as soon as they stabilize in specification form and browsers have a chance to implement them.

So how do we resolve this seeming contradiction? The answer is tooling, specifically a technique called *transpiling* (transformation + compiling). Roughly, the idea is to use a special tool to transform your ES6 code into equivalent (or close!) matches that work in ES5 environments.

For example, consider shorthand property definitions (see "Object Literal Extensions" in Chapter 2). Here's the ES6 form:

```js
var foo = [1,2,3];

var obj = {
	foo		// means `foo: foo`
};

obj.foo;	// [1,2,3]
```

Mas (aproximadamente) aqui está como isso transpila:

```js
var foo = [1,2,3];

var obj = {
	foo: foo
};

obj.foo;	// [1,2,3]
```

This is a minor but pleasant transformation that lets us shorten the `foo: foo` in an object literal declaration to just `foo`, if the names are the same.

Transpilers perform these transformations for you, usually in a build workflow step similar to how you perform linting, minification, and other similar operations.

### Shims/Polyfills

Not all new ES6 features need a transpiler. Polyfills (aka shims) are a pattern for defining equivalent behavior from a newer environment into an older environment, when possible. Syntax cannot be polyfilled, but APIs often can be.

For example, `Object.is(..)` is a new utility for checking strict equality of two values but without the nuanced exceptions that `===` has for `NaN` and `-0` values. The polyfill for `Object.is(..)` is pretty easy:

```js
if (!Object.is) {
	Object.is = function(v1, v2) {
		// test for `-0`
		if (v1 === 0 && v2 === 0) {
			return 1 / v1 === 1 / v2;
		}
		// test for `NaN`
		if (v1 !== v1) {
			return v2 !== v2;
		}
		// everything else
		return v1 === v2;
	};
}
```

**Tip:** Pay attention to the outer `if` statement guard wrapped around the polyfill. This is an important detail, which means the snippet only defines its fallback behavior for older environments where the API in question isn't already defined; it would be very rare that you'd want to overwrite an existing API.

There's a great collection of ES6 shims called "ES6 Shim" (https://github.com/paulmillr/es6-shim/) that you should definitely adopt as a standard part of any new JS project!

It is assumed that JS will continue to evolve constantly, with browsers rolling out support for features continually rather than in large chunks. So the best strategy for keeping updated as it evolves is to just introduce polyfill shims into your code base, and a transpiler step into your build workflow, right now and get used to that new reality.

If you decide to keep the status quo and just wait around for all browsers without a feature supported to go away before you start using the feature, you're always going to be way behind. You'll sadly be missing out on all the innovations designed to make writing JavaScript more effective, efficient, and robust.

## Revisando

ES6 (alguns podem tentar chamá-lo de ES2015) é apenas o pouso a partir do momento em que este está sendo escrito, e tem muitas coisas novas que você precisa aprender!

Mas é ainda mais importante mudar sua mentalidade para alinhar com a nova maneira que o JavaScript vai evoluir. Não é apenas esperar por anos para algum documento oficial obter um voto de aprovação, como muitos fizeram no passado.

Now, JavaScript features land in browsers as they become ready, and it's up to you whether you'll get on the train early or whether you'll be playing costly catch-up games years from now.

Whatever labels that future JavaScript adopts, it's going to move a lot quicker than it ever has before. Transpilers and shims/polyfills are important tools to keep you on the forefront of where the language is headed.

If there's any narrative important to understand about the new reality for JavaScript, it's that all JS developers are strongly implored to move from the trailing edge of the curve to the leading edge. And learning ES6 is where that all starts!
