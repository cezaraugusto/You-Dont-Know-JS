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

"Traceur" [^note-traceur] é um projeto mantido pelo Google, desenvolvido para *transpilar* funcionalidades ES6, de uso geral, compatíveis com ambientes pré-ES6 (em sua maioria, ES5). O comitê TC39 confia na ferramenta (e entre outras) para testar a semântica das funcionalidades que são especificadas.  

O que Traceur produz a partir do nosso trecho de código abaixo? Adivinhe só!

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

Então, com o uso dessas ferramentas, nós podemos tirar proveito do escopo de bloco independentemente se nós estamos desenvolvendo para ambientes ES6 ou não, pois `try/catch` está presente desde o ES3.

## Blocos Implícitos x Explícitos

No capítulo 3, nós identificamos potenciais problemas à manutenibilidade quando introduzimos o escopo de bloco. Será que há maneiras de tirarmos proveito do escopo de bloco reduzindo esses possíveis aspectos negativos?

Considere a seguinte alternativa de uso do `let`, chamado de "let block" ou "let statement" (ao contrário de "let declarations", como antes).

```js
let (a = 2) {
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

Em vez de atrelar-se, implicitamente, à um bloco existente, o `let` cria seu bloco explícitamente. Além do bloco explícito ser mais visível, melhorando a manutenibilidade, produz um código mais limpo, falando gramaticalmente, fazendo com que todas as declarações sejam feitas no início. Isso torna mais fácil o reconhecimento do que está vinculado ao escopo do bloco.

Como um padrão, isso reflete na abordagem de muitas pessoas que, durante o escopo de uma função, colocam todas as suas declarações de `var` no início. Contudo, a estrutura do `let` requer isso, e se você não usar `let` espalhado por aí, suas declarações de escopo de bloco serão mais fáceis de serem identificadas e mantidas.

Mas, há um problema. A estrutura do `let` apresentada não foi incluida no ES6. Nem mesmo o compilador oficial do Traceur aceita essa forma.

Então nós temos duas opções, uma seria utilizar a sintaxe válida do ES6 e um pouco de criatividade:

```js
/*let*/ { let a = 2;
	console.log( a );
}

console.log( a ); // ReferenceError
```

Mas as ferramentas são feitas para solucionar nossos problemas, então uma outra opção é escrever explicitamente a estrutura do `let`, deixando para uma ferramenta a tarefa de conversão para um código válido.

Por isso, eu desenvolvi uma ferramenta chamada "let-er" [^note-let_er]. *let-er* é um *transpilador* de código durante a fase de build, tendo como tarefa, encontrar a estrutura do `let` e *transpilá-la*. Todo o restante do seu código será isolado, incluindo declarações `let`. Você pode utilizar o *let-er* como uma primeira fase ao *transpilar*, para então encaminhar seu código à uma outra ferramenta, tal como Traceur.

Aliás, *let-er* possui uma *flag* de configuração `--es6`, que quando ativada (desativada por padrão), modifica o tipo de código produzido. Em vez de um *polyfill* `try/catch`, *let-er* produz um código totalmente compatível com ES6, sem adaptações:

```js
{
	let a = 2;
	console.log( a );
}

console.log( a ); // ReferenceError
```

Assim, você pode começar a usar *let-er* a partir de agora, atingindo todos os ambientes pré-ES6. E quando quiser produzir apenas para ES6, basta ativar a *flag*.

E o mais importante, **você pode utilizar a estrutura do `let` que preferir, mesmo que não seja parte oficial de qualquer versão ES (ainda)**.

## Performance

Let me add one last quick note on the performance of `try/catch`, and/or to address the question, "why not just use an IIFE to create the scope?"

Firstly, the performance of `try/catch` *is* slower, but there's no reasonable assumption that it *has* to be that way, or even that it *always will be* that way. Since the official TC39-approved ES6 transpiler uses `try/catch`, the Traceur team has asked Chrome to improve the performance of `try/catch`, and they are obviously motivated to do so.

Secondly, IIFE is not a fair apples-to-apples comparison with `try/catch`, because a function wrapped around any arbitrary code changes the meaning, inside of that code, of `this`, `return`, `break`, and `continue`. IIFE is not a suitable general substitute. It could only be used manually in certain cases.

The question really becomes: do you want block-scoping, or not. If you do, these tools provide you that option. If not, keep using `var` and go on about your coding!

[^note-traceur]: [Google Traceur](http://traceur-compiler.googlecode.com/git/demo/repl.html)

[^note-let_er]: [let-er](https://github.com/getify/let-er)
