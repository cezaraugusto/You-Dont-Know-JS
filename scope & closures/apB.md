# You Don't Know JS: Escopos e Encerramentos
# Apêndice B: Polyfilling Escopo de Bloco

No Capítulo 3, nós exploramos o Escopo de Bloco. Nós vimos que as cláusulas `with` e `catch` são, ambas, pequenos exemplos de escopo de bloco que existem em JavaScript desde de, pelo menos, a introdução ao ES3.

Mas foi com a introdução da cláusula `let` no ES6, que finalmente nos deu a capacidade completa e sem restrições de escopo de bloco ao nosso código. Há muitas coisas interessantes que agora serão permitidas, ambas funcionais e que agregam ao estilo do código.

Mas e se quiséssemos usar o escopo de bloco em ambientes anteriores ao ES6?

Considere este código:

```js
{
	let a = 2;
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

Em ambientes ES6 isso funcionará perfeitamente. Mas e para ambientes anteriores ao ES6? `catch` é a resposta.

```js
try{throw 2}catch(a){
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

Nossa! O código ficou estranho, um pouco feio. Contudo, podemos notar o uso de um `try/catch` para forçar o lançamento de um erro, no entanto o "erro" lançado é exatamente o valor 2, assim a declaração da variável que está dentro da cláusula `catch(a)` receberá tal valor. "Nossa mente: Boom!"

Está certo, a cláusula `catch` que forneceu o escopo de bloco ao código, então isso significa que podemos usá-la como uma técnica para ambientes pré-ES6.

"Mas...", você diz. "...Ninguém quer escrever códigos feios como esse!" É verdade. Ninguém escreve (alguns) o código resultante pelo compilador do CoffeeScript, afinal. Mas esse não é o ponto.

O ponto é: as ferramentas podem *transpilar* códigos ES6 para trabalhar em ambientes pré-ES6. Você pode escrever códigos utilizando escopo de bloco, se beneficiando de tais funcionalidades, e deixar que as ferramentas, em sua fase de build, se preocupem em produzir códigos que realmente funcionem quando implantados.

Este é na verdade o caminho preferido por todos (quer dizer, pela maioria) durante a migração de ambientes pré-ES6 para ES6: usar um *transpilador* de código para produzir códigos compatíveis com ambientes pré-ES6 a partir do ES6.

## Traceur

Google maintains a project called "Traceur" [^note-traceur], which is exactly tasked with transpiling ES6 features into pre-ES6 (mostly ES5, but not all!) for general usage. The TC39 committee relies on this tool (and others) to test out the semantics of the features they specify.

What does Traceur produce from our snippet? You guessed it!

```js
{
	try {
		throw undefined;
	} catch (a) {
		a = 2;
		console.log( a );
	}
}

console.log( a );
```

So, with the use of such tools, we can start taking advantage of block scope regardless of if we are targeting ES6 or not, because `try/catch` has been around (and worked this way) from ES3 days.

## Implicit vs. Explicit Blocks

In Chapter 3, we identified some potential pitfalls to code maintainability/refactorability when we introduce block-scoping. Is there another way to take advantage of block scope but to reduce this downside?

Consider this alternate form of `let`, called the "let block" or "let statement" (contrasted with "let declarations" from before).

```js
let (a = 2) {
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

Instead of implicitly hijacking an existing block, the let-statement creates an explicit block for its scope binding. Not only does the explicit block stand out more, and perhaps fare more robustly in code refactoring, it produces somewhat cleaner code by, grammatically, forcing all the declarations to the top of the block. This makes it easier to look at any block and know what's scoped to it and not.

As a pattern, it mirrors the approach many people take in function-scoping when they manually move/hoist all their `var` declarations to the top of the function. The let-statement puts them there at the top of the block by intent, and if you don't use `let` declarations strewn throughout, your block-scoping declarations are somewhat easier to identify and maintain.

But, there's a problem. The let-statement form is not included in ES6. Neither does the official Traceur compiler accept that form of code.

We have two options. We can format using ES6-valid syntax and a little sprinkle of code discipline:

```js
/*let*/ { let a = 2;
	console.log( a );
}

console.log( a ); // ReferenceError
```

But, tools are meant to solve our problems. So the other option is to write explicit let statement blocks, and let a tool convert them to valid, working code.

So, I built a tool called "let-er" [^note-let_er] to address just this issue. *let-er* is a build-step code transpiler, but its only task is to find let-statement forms and transpile them. It will leave alone any of the rest of your code, including any let-declarations. You can safely use *let-er* as the first ES6 transpiler step, and then pass your code through something like Traceur if necessary.

Moreover, *let-er* has a configuration flag `--es6`, which when turned on (off by default), changes the kind of code produced. Instead of the `try/catch` ES3 polyfill hack, *let-er* would take our snippet and produce the fully ES6-compliant, non-hacky:

```js
{
	let a = 2;
	console.log( a );
}

console.log( a ); // ReferenceError
```

So, you can start using *let-er* right away, and target all pre-ES6 environments, and when you only care about ES6, you can add the flag and instantly target only ES6.

And most importantly, **you can use the more preferable and more explicit let-statement form** even though it is not an official part of any ES version (yet).

## Performance

Let me add one last quick note on the performance of `try/catch`, and/or to address the question, "why not just use an IIFE to create the scope?"

Firstly, the performance of `try/catch` *is* slower, but there's no reasonable assumption that it *has* to be that way, or even that it *always will be* that way. Since the official TC39-approved ES6 transpiler uses `try/catch`, the Traceur team has asked Chrome to improve the performance of `try/catch`, and they are obviously motivated to do so.

Secondly, IIFE is not a fair apples-to-apples comparison with `try/catch`, because a function wrapped around any arbitrary code changes the meaning, inside of that code, of `this`, `return`, `break`, and `continue`. IIFE is not a suitable general substitute. It could only be used manually in certain cases.

The question really becomes: do you want block-scoping, or not. If you do, these tools provide you that option. If not, keep using `var` and go on about your coding!

[^note-traceur]: [Google Traceur](http://traceur-compiler.googlecode.com/git/demo/repl.html)

[^note-let_er]: [let-er](https://github.com/getify/let-er)
