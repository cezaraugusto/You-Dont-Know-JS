# You Don't Know JS: Escopos & Clausuras
# Apêndice A: Escopo Dinâmico

No Capítulo 2, falamos sobre "Escopo Dinâmico" como um contraste ao modelo de "Escopo Léxico", que é como os escopos funcionam em JavaScript (e, de fato, na maioria das linguagens).

Iremos brevemente examinar o escopo dinâmico, para reforçar o contraste entre eles. Porém, algo bem mais importante, o escopo dinâmico é na verdade um primo próximo de outro mecanismo (`this`) em JavaScript, que cobrimos no título "*this & Prototipagem de Objetos*" desta série.

Como vimos no Capítulo 2, o escopo léxico é um grupo de regras sobre como a *Engrenagem* pode verificar uma variável e onde ela irá encontrá-lo. As características-chave do escopo léxico é que ela é definida no tempo do autor (author-time), quando o código é escrito (assumindo que você não trapaceie com `eval()` ou `with`).

Escopos dinâmicos parecem deduzir, por uma boa causa, que existe um modelo pelo qual o escopo pode ser determinado dinamicamente no tempo de execução (*runtime*), ao invés de estaticamente no momento que o código é escrito (author-time). Este é de fato o caso. Vamos ilustrar por código:

```js
function foo() {
	console.log( a ); // 2
}

function bar() {
	var a = 3;
	foo();
}

var a = 2;

bar();
```

O escopo léxico mantém que a referência RHS para `a` em `foo()` será resolvida para a variável global `a`, que irá resultar no valor `2`.

O escopo dinâmico, por contraste, não se importa em como e onde funções e escopos são declarados, mas sim com **de onde eles são chamados**. Em outras palavras, a cadeia do escopo é baseada na pilha de chamadas (call-stack), não no aninhamento de escopos no código.

Sendo assim, se o JavaScript tem um escopo dinâmico, quando `foo()` é executado, **teoricamente** o código abaixo deveria resultar em `3` como resultado.

```js
function foo() {
	console.log( a ); // 3  (não 2!)
}

function bar() {
	var a = 3;
	foo();
}

var a = 2;

bar();
```

Como pode ser assim? Por conta de `foo()` não poder resolver a referência da variável para `a`, ao invés de passar para a cadeia de escopo aninhada (léxico), ele percorre a pilha de chamadas (call-stack), para achar *de onde* `foo()` *foi chamado*. Já que `foo()` foi chamado pelo `bar()`, ele verifica as variáveis no escopo de `bar()`, e encontra um `a` com o valor `3`.

Estranho? Você deve estar pensando, no momento.

Mas isso apenas porque você provavelmentes só trabalhou (ou pelo menos considerou profundamente) em códigos que são lexicamente escopados. Por isso o escopo dinâmico parece estranho. Se você só escreveu códigos em uma linguagem de escopo dinâmico, isso te pareceu bem natural, e o escopo léxico seria a parte estranha.

Para ser claro, o JavaScript **não tem, de fato, um escopo dinâmico**. Ele tem escopo léxico. Pleno e simples. Mas o mecanismo de `this` é meio que parecido com o escopo dinâmico.

O contraste chave **escopo léxico rodam no tempo de escrita (write-time), e o escopo dinâmico (e `this`!) ocorrem no tempo de execução (runtime)**. Escopo léxico se importa com a questão *de onde a função foi declarada*, mas o escopo dinâmico se importa com a questão *de onde ela foi chamada*.

Finalmente, `this` se importa com *como a função foi chamada*, o que mostra o quão próxima é a relação do mecanismo `this` com a ideia do escopo dinâmico. Para saber mais sobre `this`, leia o título *this & Prototipagem de Objetos*.
