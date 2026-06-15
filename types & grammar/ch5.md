# You Don't Know JS: Tipos & Gramática
# Capítulo 5: Gramática

O último tópico a ser abordado é a sintaxe do JavaScript (também conhecida como sendo sua gramática). Você pode pensar que sabe escrever JS, mas há muitas nuances em sua gramática que levam ao equívoco, então queremos nos aprofundar nessas partes e esclarecê-las.

**Nota:** O termo "gramática" pode ser um pouco menos familiar do que o termo "sintaxe". De muitas formas, eles são termos semelhantes, e descrevem as *regras* de como a linguagem funciona.
Existem diferenças sutis entre os dois termos, porém, na maioria das vezes, não há contribuição para a discussão. A gramática do JS é uma maneira estruturada de descrever como a sintaxe (operadores, keywords, etc.) se encaixam para formar programas estruturados. Em outras palavras, discutir a sintaxe sem gramática deixaria de fora muitos detalhes importantes. Portanto, o foco da nossa discussão é a *gramática*, mesmo sendo a sintaxe da linguagem o que os desenvolvedores interagem diretamente.

## Instruções & Expressões

É comum desenvolvedores assumirem que os termos "instruções" e "expressões" são equivalentes. Porém precisamos distinguí-los, pois existem algumas diferenças muito importantes em nossos programas JS.

Para exemplificar essa distinção, usaremos a terminologia com a qual você pode estar mais familiarizado: O idioma inglês.

Uma "sentença" é uma junção de palavras que expressa um pensamento. É composta por uma ou mais "frases", cada uma das quais pode ser conectada com sinais de pontuação ou por palavras de conjunção ("e", "ou", etc). Uma frase pode ser composta por frases menores. Algumas delas são incompletas e não tem muito sentido por si só, enquanto outras podem se sustentar sozinhas. Esse conjunto de regras é chamado de *gramática* da língua inglesa.

E assim acontece com a gramática de JavaScript. Instruções são sentenças, expressões são frases e operadores são conjuções e pontuações.

Todas expressões em JS podem ser avaliadas em um único resultado. Por exemplo:

```js
var a = 3 * 6;
var b = a;
b;
```

Nesse trecho, `3 * 6` é uma expressão (avaliada no valor `18`). Mas `a` na segunda linha é também uma expressão, assim como `b` na terceira linha. As expressões `a` e `b` avaliam os valores armazenados nessas variáveis naquele momnento, que também passa a ser `18`.

Além disso, cada uma das três linhas é uma instrução contendo expressões. `var a = 3 * 6` e `var b = a` são chamados de 'instruções de declaração', pois cada uma declara uma variável ( e opcionalmente atribui um valor a elas). As atribuições `a = 3 * 6` e `b = a` são chamadas de expressões de atribuições.

A terceira linha contém apenas a expressão `b`, que também é uma instrução por si só (embora não seja uma muito interessante!). Esse tipo, geralmente, é chamado de "instrução de expressão".

### Valores de Conclusão de Instruções

É um fato pouco conhecido que todas as instruções têm valores de conclusão (mesmo que esse valor seja apenas `undefined`).

Como você faria para ver o valor de conclusão de uma instrução?

A resposta mais óbvia é digitar a instrução no console de desenvolvedor do seu navegador, porque quando você a executa, o console por padrão reporta o valor de conclusão da instrução mais recente que executou.

Vamos considerar `var b = a`. Qual é o valor de conclusão dessa instrução?

A expressão de atribuição `b = a` resulta no valor que foi atribuído (`18` acima), mas a própria instrução `var` resulta em `undefined`. Por quê? Porque as instruções `var` são definidas dessa forma na especificação. Se você colocar `var a = 42;` no seu console, você verá `undefined` reportado de volta em vez de `42`.

**Nota:** Tecnicamente, é um pouco mais complexo do que isso. Na especificação ES5, seção 12.2 "Variable Statement," o algoritmo `VariableDeclaration` na verdade *retorna* um valor (uma `string` contendo o nome da variável declarada -- estranho, não!?), mas esse valor é basicamente engolido (exceto para uso pelo laço `for..in`) pelo algoritmo `VariableStatement`, que força um valor de conclusão vazio (também conhecido como `undefined`).

Na verdade, se você já fez muitos experimentos de código no seu console (ou em um REPL de ambiente JavaScript -- ferramenta read/evaluate/print/loop), você provavelmente já viu `undefined` reportado após muitas instruções diferentes, e talvez nunca tenha percebido por que ou o que era aquilo. Em poucas palavras, o console está apenas reportando o valor de conclusão da instrução.

Mas o que o console imprime para o valor de conclusão não é algo que possamos usar dentro do nosso programa. Então como podemos capturar o valor de conclusão?

Essa é uma tarefa muito mais complicada. Antes de explicarmos *como*, vamos explorar *por que* você iria querer fazer isso?

Precisamos considerar outros tipos de valores de conclusão de instruções. Por exemplo, qualquer bloco regular `{ .. }` tem um valor de conclusão igual ao valor de conclusão da última instrução/expressão contida nele.

Considere:

```js
var b;

if (true) {
	b = 4 + 38;
}
```

Se você digitasse isso no seu console/REPL, provavelmente veria `42` reportado, já que `42` é o valor de conclusão do bloco `if`, que assumiu o valor de conclusão da sua última instrução de expressão de atribuição `b = 4 + 38`.

Em outras palavras, o valor de conclusão de um bloco é como um *retorno implícito* do valor da última instrução no bloco.

**Nota:** Isso é conceitualmente familiar em linguagens como CoffeeScript, que têm valores de `return` implícitos de `function`s que são os mesmos do valor da última instrução na função.

Mas há um problema óbvio. Esse tipo de código não funciona:

```js
var a, b;

a = if (true) {
	b = 4 + 38;
};
```

Não podemos capturar o valor de conclusão de uma instrução e atribuí-lo a outra variável de nenhuma maneira sintática/gramatical fácil (pelo menos não ainda!).

Então, o que podemos fazer?

**Aviso**: Apenas para fins de demonstração -- não faça realmente o seguinte no seu código de verdade!

Poderíamos usar a tão difamada função `eval(..)` (às vezes pronunciada "evil", "maligna" em inglês) para capturar esse valor de conclusão.

```js
var a, b;

a = eval( "if (true) { b = 4 + 38; }" );

a;	// 42
```

Pooooois é. Isso é terrivelmente feio. Mas funciona! E ilustra o ponto de que valores de conclusão de instruções são uma coisa real que pode ser capturada não apenas no nosso console, mas também nos nossos programas.

Há uma proposta para o ES7 chamada "do expression". Veja como ela poderia funcionar:

```js
var a, b;

a = do {
	if (true) {
		b = 4 + 38;
	}
};

a;	// 42
```

A expressão `do { .. }` executa um bloco (com uma ou várias instruções nele), e o valor de conclusão da instrução final dentro do bloco torna-se o valor de conclusão *da* expressão `do`, que pode então ser atribuído a `a` como mostrado.

A ideia geral é poder tratar instruções como expressões -- elas podem aparecer dentro de outras instruções -- sem precisar envolvê-las em uma expressão de função inline e executar um `return ..` explícito.

Por enquanto, valores de conclusão de instruções não passam de curiosidades. Mas provavelmente vão assumir mais importância à medida que o JS evolui, e esperançosamente as expressões `do { .. }` reduzirão a tentação de usar coisas como `eval(..)`.

**Aviso:** Repetindo minha advertência anterior: evite `eval(..)`. Sério. Veja o título *Scope & Closures* desta série para mais explicações.

### Efeitos Colaterais de Expressões

A maioria das expressões não têm efeitos colaterais. Por exemplo:

```js
var a = 2;
var b = a + 3;
```

A expressão `a + 3` não teve *em si* um efeito colateral, como por exemplo alterar `a`. Ela teve um resultado, que é `5`, e esse resultado foi atribuído a `b` na instrução `b = a + 3`.

O exemplo mais comum de uma expressão com (possíveis) efeitos colaterais é uma expressão de chamada de função:

```js
function foo() {
	a = a + 1;
}

var a = 1;
foo();		// resultado: `undefined`, efeito colateral: alterou `a`
```

Há outras expressões com efeitos colaterais, porém. Por exemplo:

```js
var a = 42;
var b = a++;
```

A expressão `a++` tem dois comportamentos distintos. *Primeiro*, ela retorna o valor atual de `a`, que é `42` (que então é atribuído a `b`). Mas *em seguida*, ela altera o valor de `a` em si, incrementando-o em um.

```js
var a = 42;
var b = a++;

a;	// 43
b;	// 42
```

Muitos desenvolvedores acreditariam erroneamente que `b` tem o valor `43` assim como `a` tem. Mas a confusão vem de não considerar plenamente o *quando* dos efeitos colaterais do operador `++`.

O operador de incremento `++` e o operador de decremento `--` são ambos operadores unários (veja o Capítulo 4), que podem ser usados tanto na posição pós-fixa ("depois") quanto na posição pré-fixa ("antes").

```js
var a = 42;

a++;	// 42
a;		// 43

++a;	// 44
a;		// 44
```

Quando `++` é usado na posição pré-fixa como `++a`, seu efeito colateral (incrementar `a`) acontece *antes* de o valor ser retornado da expressão, em vez de *depois* como em `a++`.

**Nota:** Você acharia que `++a++` seria sintaxe válida? Se você tentar, vai receber um erro `ReferenceError`, mas por quê? Porque operadores com efeitos colaterais **requerem uma referência a variável** para direcionar seus efeitos colaterais. Para `++a++`, a parte `a++` é avaliada primeiro (por causa da precedência de operadores -- veja abaixo), o que devolve o valor de `a` _antes_ do incremento. Mas então ele tenta avaliar `++42`, o que (se você tentar) dá o mesmo erro `ReferenceError`, já que `++` não pode ter um efeito colateral diretamente sobre um valor como `42`.

Às vezes pensa-se erroneamente que você pode encapsular o efeito colateral *posterior* de `a++` envolvendo-o em um par `( )`, como:

```js
var a = 42;
var b = (a++);

a;	// 43
b;	// 42
```

Infelizmente, `( )` em si não define uma nova expressão envolvida que seria avaliada *depois* do *efeito colateral posterior* da expressão `a++`, como poderíamos ter esperado. Na verdade, mesmo que definisse, `a++` retorna `42` primeiro, e a menos que você tenha outra expressão que reavalie `a` após o efeito colateral do `++`, você não vai obter `43` dessa expressão, então `b` não será atribuído `43`.

Existe uma opção, porém: o operador vírgula `,` de série de instruções. Esse operador permite que você encadeie múltiplas instruções de expressão independentes em uma única instrução:

```js
var a = 42, b;
b = ( a++, a );

a;	// 43
b;	// 43
```

**Nota:** Os `( .. )` em torno de `a++, a` são obrigatórios aqui. A razão é a precedência de operadores, que abordaremos mais adiante neste capítulo.

A expressão `a++, a` significa que a segunda instrução de expressão `a` é avaliada *depois* dos *efeitos colaterais posteriores* da primeira instrução de expressão `a++`, o que significa que ela retorna o valor `43` para atribuição a `b`.

Outro exemplo de operador com efeito colateral é `delete`. Como mostramos no Capítulo 2, `delete` é usado para remover uma propriedade de um `object` ou um slot de um `array`. Mas geralmente é apenas chamado como uma instrução independente:

```js
var obj = {
	a: 42
};

obj.a;			// 42
delete obj.a;	// true
obj.a;			// undefined
```

O valor de resultado do operador `delete` é `true` se a operação solicitada for válida/permitida, ou `false` caso contrário. Mas o efeito colateral do operador é que ele remove a propriedade (ou slot do array).

**Nota:** O que queremos dizer com válida/permitida? Propriedades inexistentes, ou propriedades que existem e são configuráveis (veja o Capítulo 3 do título *this & Object Prototypes* desta série) retornarão `true` do operador `delete`. Caso contrário, o resultado será `false` ou um erro.

Um último exemplo de operador com efeito colateral, que pode ser ao mesmo tempo óbvio e não óbvio, é o operador de atribuição `=`.

Considere:

```js
var a;

a = 42;		// 42
a;			// 42
```

Pode não parecer que `=` em `a = 42` seja um operador com efeito colateral para a expressão. Mas se examinarmos o valor de resultado da instrução `a = 42`, é o valor que acabou de ser atribuído (`42`), então a atribuição desse mesmo valor a `a` é essencialmente um efeito colateral.

**Dica:** O mesmo raciocínio sobre efeitos colaterais vale para os operadores de atribuição composta como `+=`, `-=`, etc. Por exemplo, `a = b += 2` é processado primeiro como `b += 2` (que é `b = b + 2`), e o resultado *dessa* atribuição `=` é então atribuído a `a`.

Esse comportamento de que uma expressão (ou instrução) de atribuição resulta no valor atribuído é principalmente útil para atribuições encadeadas, como:

```js
var a, b, c;

a = b = c = 42;
```

Aqui, `c = 42` é avaliado como `42` (com o efeito colateral de atribuir `42` a `c`), então `b = 42` é avaliado como `42` (com o efeito colateral de atribuir `42` a `b`), e finalmente `a = 42` é avaliado (com o efeito colateral de atribuir `42` a `a`).

**Aviso:** Um erro comum que os desenvolvedores cometem com atribuições encadeadas é algo como `var a = b = 42`. Embora isso pareça a mesma coisa, não é. Se essa instrução acontecesse sem também haver um `var b` separado (em algum lugar do escopo) para declarar formalmente `b`, então `var a = b = 42` não declararia `b` diretamente. Dependendo do modo `strict`, isso ou lançaria um erro ou criaria uma global acidental (veja o título *Scope & Closures* desta série).

Outro cenário a considerar:

```js
function vowels(str) {
	var matches;

	if (str) {
		// extrai todas as vogais
		matches = str.match( /[aeiou]/g );

		if (matches) {
			return matches;
		}
	}
}

vowels( "Hello World" ); // ["e","o","o"]
```

Isso funciona, e muitos desenvolvedores preferem assim. Mas usando uma expressão idiomática onde aproveitamos o efeito colateral da atribuição, podemos simplificar combinando as duas instruções `if` em uma só:

```js
function vowels(str) {
	var matches;

	// extrai todas as vogais
	if (str && (matches = str.match( /[aeiou]/g ))) {
		return matches;
	}
}

vowels( "Hello World" ); // ["e","o","o"]
```

**Nota:** Os `( .. )` em torno de `matches = str.match..` são obrigatórios. A razão é a precedência de operadores, que abordaremos na seção "Precedência de Operadores" mais adiante neste capítulo.

Eu prefiro esse estilo mais curto, pois acho que deixa mais claro que os dois condicionais estão de fato relacionados em vez de separados. Mas, como na maioria das escolhas estilísticas em JS, é puramente questão de opinião qual é *melhor*.

### Regras Contextuais

Há vários lugares nas regras de gramática do JavaScript onde a mesma sintaxe significa coisas diferentes dependendo de onde/como é usada. Esse tipo de coisa pode, isoladamente, causar bastante confusão.

Não vamos listar exaustivamente todos esses casos aqui, mas apenas destacar alguns dos mais comuns.

#### Chaves `{ .. }`

Há dois lugares principais (e mais virão à medida que o JS evolui!) em que um par de chaves `{ .. }` aparecerá no seu código. Vamos dar uma olhada em cada um deles.

##### Literais de Objeto

Primeiro, como um literal de `object`:

```js
// suponha que exista uma função `bar()` definida

var a = {
	foo: bar()
};
```

Como sabemos que isso é um literal de `object`? Porque o par `{ .. }` é um valor que está sendo atribuído a `a`.

**Nota:** A referência `a` é chamada de "l-value" (também conhecido como left-hand value, valor do lado esquerdo) já que é o alvo de uma atribuição. O par `{ .. }` é um "r-value" (também conhecido como right-hand value, valor do lado direito) já que é usado *apenas* como um valor (neste caso como a origem de uma atribuição).

##### Rótulos

O que acontece se removermos a parte `var a =` do trecho acima?

```js
// suponha que exista uma função `bar()` definida

{
	foo: bar()
}
```

Muitos desenvolvedores assumem que o par `{ .. }` é apenas um literal de `object` independente que não é atribuído a lugar nenhum. Mas na verdade é algo completamente diferente.

Aqui, `{ .. }` é apenas um bloco de código regular. Não é muito idiomático em JavaScript (muito mais em outras linguagens!) ter um bloco `{ .. }` independente assim, mas é gramática JS perfeitamente válida. Pode ser especialmente útil quando combinado com declarações de escopo de bloco `let` (veja o título *Scope & Closures* desta série).

O bloco de código `{ .. }` aqui é funcionalmente bem idêntico ao bloco de código que é anexado a alguma instrução, como um laço `for`/`while`, condicional `if`, etc.

Mas se é um bloco de código normal, o que é aquela sintaxe `foo: bar()` de aparência bizarra, e como isso é legal?

É por causa de um recurso pouco conhecido (e, francamente, desencorajado) no JavaScript chamado "instruções rotuladas". `foo` é um rótulo para a instrução `bar()` (que omitiu seu `;` final -- veja "Ponto e Vírgula Automático" mais adiante neste capítulo). Mas qual é o propósito de uma instrução rotulada?

Se o JavaScript tivesse uma instrução `goto`, você teoricamente poderia dizer `goto foo` e fazer a execução pular para aquela localização no código. `goto`s geralmente são considerados péssimas práticas de programação, pois tornam o código muito mais difícil de entender (também conhecido como "código espaguete"), então é uma *coisa muito boa* que o JavaScript não tenha um `goto` geral.

Entretanto, o JS *suporta* uma forma limitada e especial de `goto`: saltos rotulados. Tanto a instrução `continue` quanto a `break` podem opcionalmente aceitar um rótulo especificado, caso em que o fluxo do programa "salta" mais ou menos como um `goto`. Considere:

```js
// laço rotulado `foo`
foo: for (var i=0; i<4; i++) {
	for (var j=0; j<4; j++) {
		// sempre que os laços se encontrarem, continua o laço externo
		if (j == i) {
			// salta para a próxima iteração do
			// laço rotulado `foo`
			continue foo;
		}

		// pula os múltiplos ímpares
		if ((j * i) % 2 == 1) {
			// `continue` normal (não rotulado) do laço interno
			continue;
		}

		console.log( i, j );
	}
}
// 1 0
// 2 0
// 2 1
// 3 0
// 3 2
```

**Nota:** `continue foo` não significa "vá para a posição rotulada 'foo' para continuar", mas sim, "continue o laço que está rotulado como 'foo' com sua próxima iteração." Então, não é *realmente* um `goto` arbitrário.

Como você pode ver, pulamos a iteração de múltiplo ímpar `3 1`, mas o salto do laço rotulado também pulou as iterações `1 1` e `2 2`.

Talvez uma forma um pouco mais útil do salto rotulado seja com `break __` de dentro de um laço interno onde você quer sair do laço externo. Sem um `break` rotulado, essa mesma lógica às vezes poderia ser bem desajeitada de escrever:

```js
// laço rotulado `foo`
foo: for (var i=0; i<4; i++) {
	for (var j=0; j<4; j++) {
		if ((i * j) >= 3) {
			console.log( "stopping!", i, j );
			// sai do laço rotulado `foo`
			break foo;
		}

		console.log( i, j );
	}
}
// 0 0
// 0 1
// 0 2
// 0 3
// 1 0
// 1 1
// 1 2
// stopping! 1 3
```

**Nota:** `break foo` não significa "vá para a posição rotulada 'foo' para continuar," mas sim, "saia do laço/bloco que está rotulado como 'foo' e continue *depois* dele." Não é exatamente um `goto` no sentido tradicional, não é?

A alternativa de `break` não rotulado para o caso acima provavelmente precisaria envolver uma ou mais funções, acesso a variável de escopo compartilhado, etc. Seria bem provavelmente mais confusa do que o `break` rotulado, então aqui usar um `break` rotulado é talvez a melhor opção.

Um rótulo pode se aplicar a um bloco que não seja laço, mas apenas `break` pode referenciar tal rótulo de não-laço. Você pode fazer um `break ___` rotulado para sair de qualquer bloco rotulado, mas não pode fazer `continue ___` em um rótulo de não-laço, nem pode fazer um `break` não rotulado para sair de um bloco.

```js
function foo() {
	// bloco rotulado `bar`
	bar: {
		console.log( "Hello" );
		break bar;
		console.log( "never runs" );
	}
	console.log( "World" );
}

foo();
// Hello
// World
```

Laços/blocos rotulados são extremamente incomuns, e frequentemente mal vistos. É melhor evitá-los se possível; por exemplo usando chamadas de função em vez dos saltos de laço. Mas talvez haja alguns casos limitados onde possam ser úteis. Se você for usar um salto rotulado, certifique-se de documentar o que você está fazendo com muitos comentários!

É uma crença muito comum que JSON é um subconjunto próprio de JS, então uma string de JSON (como `{"a":42}` -- note as aspas em torno do nome da propriedade como o JSON requer!) é considerada um programa JavaScript válido. **Não é verdade!** Tente colocar `{"a":42}` no seu console JS, e você receberá um erro.

Isso acontece porque rótulos de instrução não podem ter aspas em torno deles, então `"a"` não é um rótulo válido, e portanto `:` não pode vir logo depois dele.

Então, JSON é verdadeiramente um subconjunto da sintaxe do JS, mas o JSON não é gramática JS válida por si só.

Um equívoco extremamente comum nesse sentido é que, se você carregasse um arquivo JS em uma tag `<script src=..>` que tem apenas conteúdo JSON nele (como de uma chamada de API), os dados seriam lidos como JavaScript válido mas simplesmente inacessíveis ao programa. JSON-P (a prática de envolver os dados JSON em uma chamada de função, como `foo({"a":42})`) é geralmente dito resolver essa inacessibilidade enviando o valor para uma das funções do seu programa.

**Não é verdade!** O valor JSON totalmente válido `{"a":42}` por si só na verdade lançaria um erro JS porque seria interpretado como um bloco de instrução com um rótulo inválido. Mas `foo({"a":42})` é JS válido porque nele, `{"a":42}` é um valor de literal de `object` sendo passado para `foo(..)`. Então, dito apropriadamente, **JSON-P transforma JSON em gramática JS válida!**

##### Blocos

Outra pegadinha JS comumente citada (relacionada à coerção -- veja o Capítulo 4) é:

```js
[] + {}; // "[object Object]"
{} + []; // 0
```

Isso parece sugerir que o operador `+` dá resultados diferentes dependendo de se o primeiro operando é o `[]` ou o `{}`. Mas na verdade isso não tem nada a ver com isso!

Na primeira linha, `{}` aparece na expressão do operador `+`, e é portanto interpretado como um valor real (um `object` vazio). O Capítulo 4 explicou que `[]` é coagido para `""` e portanto `{}` também é coagido para um valor `string`: `"[object Object]"`.

Mas na segunda linha, `{}` é interpretado como um bloco vazio `{}` independente (que não faz nada). Blocos não precisam de ponto e vírgula para terminá-los, então a falta de um aqui não é problema. Por fim, `+ []` é uma expressão que *coage explicitamente* (veja o Capítulo 4) o `[]` para um `number`, que é o valor `0`.

##### Desestruturação de Objeto

A partir do ES6, outro lugar onde você verá pares `{ .. }` aparecendo é com "atribuições de desestruturação" (veja o título *ES6 & Beyond* desta série para mais informações), especificamente desestruturação de `object`. Considere:

```js
function getData() {
	// ..
	return {
		a: 42,
		b: "foo"
	};
}

var { a, b } = getData();

console.log( a, b ); // 42 "foo"
```

Como você provavelmente já percebeu, `var { a , b } = ..` é uma forma de atribuição de desestruturação do ES6, que é aproximadamente equivalente a:

```js
var res = getData();
var a = res.a;
var b = res.b;
```

**Nota:** `{ a, b }` é na verdade uma abreviação de desestruturação do ES6 para `{ a: a, b: b }`, então qualquer uma funcionará, mas espera-se que a forma mais curta `{ a, b }` se torne a forma preferida.

A desestruturação de objeto com um par `{ .. }` também pode ser usada para argumentos de função nomeados, que é açúcar sintático para esse mesmo tipo de atribuição implícita de propriedade de objeto:

```js
function foo({ a, b, c }) {
	// não há necessidade de:
	// var a = obj.a, b = obj.b, c = obj.c
	console.log( a, b, c );
}

foo( {
	c: [1,2,3],
	a: 42,
	b: "foo"
} );	// 42 "foo" [1, 2, 3]
```

Então, o contexto em que usamos pares `{ .. }` determina inteiramente o que eles significam, o que ilustra a diferença entre sintaxe e gramática. É muito importante entender essas nuances para evitar interpretações inesperadas pelo motor JS.

#### `else if` E Blocos Opcionais

É um equívoco comum que o JavaScript tenha uma cláusula `else if`, porque você pode fazer:

```js
if (a) {
	// ..
}
else if (b) {
	// ..
}
else {
	// ..
}
```

Mas há uma característica oculta da gramática JS aqui: não existe `else if`. Mas as instruções `if` e `else` têm permissão para omitir os `{ }` em torno do bloco anexado se eles contiverem apenas uma única instrução. Você já viu isso muitas vezes antes, sem dúvida:

```js
if (a) doSomething( a );
```

Muitos guias de estilo JS insistirão que você sempre use `{ }` em torno de um bloco de instrução única, como:

```js
if (a) { doSomething( a ); }
```

Entretanto, a exata mesma regra gramatical se aplica à cláusula `else`, então a forma `else if` que você provavelmente sempre codificou é *na verdade* analisada como:

```js
if (a) {
	// ..
}
else {
	if (b) {
		// ..
	}
	else {
		// ..
	}
}
```

O `if (b) { .. } else { .. }` é uma única instrução que segue o `else`, então você pode colocar os `{ }` envolventes ou não. Em outras palavras, quando você usa `else if`, você está tecnicamente quebrando aquela regra comum de guia de estilo e apenas definindo seu `else` com uma única instrução `if`.

Claro, a expressão idiomática `else if` é extremamente comum e resulta em um nível a menos de indentação, então é atrativo. De qualquer forma que você faça, apenas declare explicitamente no seu próprio guia de estilo/regras e não assuma que coisas como `else if` são regras gramaticais diretas.

## Precedência de Operadores

Como abordamos no Capítulo 4, a versão do JavaScript de `&&` e `||` é interessante por selecionar e retornar um de seus operandos, em vez de simplesmente resultar em `true` ou `false`. Isso é fácil de raciocinar se houver apenas dois operandos e um operador.

```js
var a = 42;
var b = "foo";

a && b;	// "foo"
a || b;	// 42
```

Mas e quando há dois operadores envolvidos, e três operandos?

```js
var a = 42;
var b = "foo";
var c = [1,2,3];

a && b || c; // ???
a || b && c; // ???
```

Para entender no que essas expressões resultam, vamos precisar entender quais regras governam como os operadores são processados quando há mais de um presente em uma expressão.

Essas regras são chamadas de "precedência de operadores."

Aposto que a maioria dos leitores sente que tem um domínio decente sobre precedência de operadores. Mas como tudo o mais que abordamos nesta série de livros, vamos cutucar e provocar esse entendimento para ver quão sólido ele realmente é, e esperançosamente aprender algumas coisas novas ao longo do caminho.

Relembre o exemplo de cima:

```js
var a = 42, b;
b = ( a++, a );

a;	// 43
b;	// 43
```

Mas o que aconteceria se removêssemos os `( )`?

```js
var a = 42, b;
b = a++, a;

a;	// 43
b;	// 42
```

Espere! Por que isso mudou o valor atribuído a `b`?

Porque o operador `,` tem precedência menor do que o operador `=`. Então, `b = a++, a` é interpretado como `(b = a++), a`. Porque (como explicamos anteriormente) `a++` tem *efeitos colaterais posteriores*, o valor atribuído a `b` é o valor `42` antes de o `++` alterar `a`.

Isso é apenas uma simples questão de precisar entender precedência de operadores. Se você for usar `,` como operador de série de instruções, é importante saber que ele na verdade tem a menor precedência. Todo outro operador se ligará mais firmemente do que o `,`.

Agora, relembre este exemplo de cima:

```js
if (str && (matches = str.match( /[aeiou]/g ))) {
	// ..
}
```

Dissemos que os `( )` em torno da atribuição são obrigatórios, mas por quê? Porque `&&` tem precedência maior do que `=`, então sem os `( )` para forçar a ligação, a expressão seria em vez disso tratada como `(str && matches) = str.match..`. Mas isso seria um erro, porque o resultado de `(str && matches)` não vai ser uma variável, mas sim um valor (neste caso `undefined`), e portanto não pode ser o lado esquerdo de uma atribuição `=`!

OK, então você provavelmente acha que dominou essa coisa de precedência de operadores.

Vamos avançar para um exemplo mais complexo (que carregaremos ao longo das próximas várias seções deste capítulo) para *realmente* testar seu entendimento:

```js
var a = 42;
var b = "foo";
var c = false;

var d = a && b || c ? c || b ? a : c && b : a;

d;		// ??
```

OK, maligno, eu admito. Ninguém escreveria uma cadeia de expressões como essa, certo? *Provavelmente* não, mas vamos usá-la para examinar várias questões em torno do encadeamento de múltiplos operadores juntos, o que *é* uma tarefa muito comum.

O resultado acima é `42`. Mas isso não é nem de longe tão interessante quanto como podemos descobrir essa resposta sem simplesmente jogá-la em um programa JS para deixar o JavaScript resolvê-la.

Vamos cavar.

A primeira pergunta -- pode nem ter te ocorrido perguntar -- é, a primeira parte (`a && b || c`) se comporta como `(a && b) || c` ou como `a && (b || c)`? Você sabe com certeza? Você consegue ao menos se convencer de que elas são de fato diferentes?

```js
(false && true) || true;	// true
false && (true || true);	// false
```

Então, há prova de que são diferentes. Mas ainda assim, como `false && true || true` se comporta? A resposta:

```js
false && true || true;		// true
(false && true) || true;	// true
```

Então temos nossa resposta. O operador `&&` é avaliado primeiro e o operador `||` é avaliado em segundo.

Mas isso é só por causa do processamento da esquerda para a direita? Vamos inverter a ordem dos operadores:

```js
true || false && false;		// true

(true || false) && false;	// false -- não
true || (false && false);	// true -- ganhador, ganhador!
```

Agora provamos que `&&` é avaliado primeiro e depois `||`, e neste caso isso foi na verdade contrário ao processamento da esquerda para a direita geralmente esperado.

Então o que causou esse comportamento? **Precedência de operadores**.

Toda linguagem define sua própria lista de precedência de operadores. É desanimador, porém, quão incomum é que desenvolvedores JS tenham lido a lista do JS.

Se você a conhecesse bem, os exemplos acima não teriam te derrubado nem um pouco, porque você já saberia que `&&` tem mais precedência que `||`. Mas aposto que uma boa quantidade de leitores teve que pensar um pouco sobre isso.

**Nota:** Infelizmente, a especificação JS não tem realmente sua lista de precedência de operadores em um único local conveniente. Você tem que analisar e entender todas as regras gramaticais. Então vamos tentar dispor as partes mais comuns e úteis aqui em um formato mais conveniente. Para uma lista completa de precedência de operadores, veja "Operator Precedence" no site da MDN (* https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence).

### Curto-circuito

No Capítulo 4, mencionamos numa nota lateral a natureza de "curto-circuito" de operadores como `&&` e `||`. Vamos revisitar isso em mais detalhe agora.

Para ambos os operadores `&&` e `||`, o operando do lado direito **não será avaliado** se o operando do lado esquerdo for suficiente para determinar o resultado da operação. Daí o nome "curto-circuito" (no sentido de que, se possível, ele pegará um atalho antecipado para sair).

Por exemplo, com `a && b`, `b` não é avaliado se `a` for falsy, porque o resultado do operando `&&` já é certo, então não há motivo em se incomodar em verificar `b`. Da mesma forma, com `a || b`, se `a` for truthy, o resultado do operando já é certo, então não há razão para verificar `b`.

Esse curto-circuito pode ser muito útil e é comumente usado:

```js
function doSomething(opts) {
	if (opts && opts.cool) {
		// ..
	}
}
```

A parte `opts` do teste `opts && opts.cool` atua como uma espécie de guarda, porque se `opts` não estiver definido (ou não for um `object`), a expressão `opts.cool` lançaria um erro. A falha do teste `opts` somada ao curto-circuito significa que `opts.cool` nem será avaliado, portanto nenhum erro!

De forma semelhante, você pode usar o curto-circuito de `||`:

```js
function doSomething(opts) {
	if (opts.cache || primeCache()) {
		// ..
	}
}
```

Aqui, estamos verificando `opts.cache` primeiro, e se ele estiver presente, não chamamos a função `primeCache()`, evitando assim trabalho potencialmente desnecessário.

### Ligação Mais Firme

Mas vamos voltar nossa atenção àquele exemplo de instrução complexa anterior com todos os operadores encadeados, especificamente as partes do operador ternário `? :`. O operador `? :` tem mais ou menos precedência do que os operadores `&&` e `||`?

```js
a && b || c ? c || b ? a : c && b : a
```

Isso é mais parecido com isto:

```js
a && b || (c ? c || (b ? a : c) && b : a)
```

ou isto?

```js
(a && b || c) ? (c || b) ? a : (c && b) : a
```

A resposta é a segunda. Mas por quê?

Porque `&&` tem mais precedência que `||`, e `||` tem mais precedência que `? :`.

Então, a expressão `(a && b || c)` é avaliada *primeiro* antes do `? :` do qual ela participa. Outra forma de explicar isso comumente é que `&&` e `||` "se ligam mais firmemente" do que `? :`. Se o contrário fosse verdade, então `c ? c...` se ligaria mais firmemente, e se comportaria (como a primeira escolha) como `a && b || (c ? c..)`.

### Associatividade

Então, os operadores `&&` e `||` se ligam primeiro, depois o operador `? :`. Mas e quanto a múltiplos operadores de mesma precedência? Eles sempre processam da esquerda para a direita ou da direita para a esquerda?

Em geral, operadores são ou associativos à esquerda ou associativos à direita, referindo-se a se o **agrupamento acontece pela esquerda ou pela direita**.

É importante notar que associatividade *não* é a mesma coisa que processamento da esquerda para a direita ou da direita para a esquerda.

Mas por que importa se o processamento é da esquerda para a direita ou da direita para a esquerda? Porque expressões podem ter efeitos colaterais, como por exemplo com chamadas de função:

```js
var a = foo() && bar();
```

Aqui, `foo()` é avaliado primeiro, e então possivelmente `bar()` dependendo do resultado da expressão `foo()`. Isso definitivamente poderia resultar em comportamento de programa diferente do que se `bar()` fosse chamado antes de `foo()`.

Mas esse comportamento é *apenas* processamento da esquerda para a direita (o comportamento padrão em JavaScript!) -- não tem nada a ver com a associatividade de `&&`. Naquele exemplo, já que há apenas um `&&` e portanto nenhum agrupamento relevante aqui, a associatividade nem entra em jogo.

Mas com uma expressão como `a && b && c`, o agrupamento *irá* acontecer implicitamente, significando que ou `a && b` ou `b && c` será avaliado primeiro.

Tecnicamente, `a && b && c` será tratado como `(a && b) && c`, porque `&&` é associativo à esquerda (assim como `||`, a propósito). Entretanto, a alternativa associativa à direita `a && (b && c)` se comporta de forma observável da mesma maneira. Para os mesmos valores, as mesmas expressões são avaliadas na mesma ordem.

**Nota:** Se hipoteticamente `&&` fosse associativo à direita, ele seria processado da mesma forma como se você manualmente usasse `( )` para criar o agrupamento como `a && (b && c)`. Mas isso ainda **não significa** que `c` seria processado antes de `b`. Associatividade à direita **não** significa avaliação da direita para a esquerda, significa **agrupamento** da direita para a esquerda. De qualquer forma, independentemente do agrupamento/associatividade, a ordem estrita de avaliação será `a`, depois `b`, depois `c` (também conhecido como da esquerda para a direita).

Então não importa muito que `&&` e `||` sejam associativos à esquerda, exceto para ser preciso em como discutimos suas definições.

Mas nem sempre é assim. Alguns operadores se comportariam de forma muito diferente dependendo de associatividade à esquerda vs. associatividade à direita.

Considere o operador `? :` ("ternário" ou "condicional"):

```js
a ? b : c ? d : e;
```

`? :` é associativo à direita, então qual agrupamento representa como ele será processado?

* `a ? b : (c ? d : e)`
* `(a ? b : c) ? d : e`

A resposta é `a ? b : (c ? d : e)`. Diferentemente de `&&` e `||` acima, a associatividade à direita aqui na verdade importa, já que `(a ? b : c) ? d : e` *vai* se comportar de forma diferente para algumas (mas não todas!) combinações de valores.

Um exemplo disso:

```js
true ? false : true ? true : true;		// false

true ? false : (true ? true : true);	// false
(true ? false : true) ? true : true;	// true
```

Diferenças ainda mais sutis se escondem com outras combinações de valores, mesmo que o resultado final seja o mesmo. Considere:

```js
true ? false : true ? true : false;		// false

true ? false : (true ? true : false);	// false
(true ? false : true) ? true : false;	// false
```

A partir desse cenário, o mesmo resultado final implica que o agrupamento é irrelevante. Entretanto:

```js
var a = true, b = false, c = true, d = true, e = false;

a ? b : (c ? d : e); // false, avalia apenas `a` e `b`
(a ? b : c) ? d : e; // false, avalia `a`, `b` E `e`
```

Então, provamos claramente que `? :` é associativo à direita, e que isso de fato importa em relação a como o operador se comporta se encadeado consigo mesmo.

Outro exemplo de associatividade à direita (agrupamento) é o operador `=`. Relembre o exemplo de atribuição encadeada do início do capítulo:

```js
var a, b, c;

a = b = c = 42;
```

Afirmamos anteriormente que `a = b = c = 42` é processado avaliando primeiro a atribuição `c = 42`, depois `b = ..`, e finalmente `a = ..`. Por quê? Por causa da associatividade à direita, que na verdade trata a instrução assim: `a = (b = (c = 42))`.

Lembra do nosso exemplo de expressão de atribuição complexa do início do capítulo?

```js
var a = 42;
var b = "foo";
var c = false;

var d = a && b || c ? c || b ? a : c && b : a;

d;		// 42
```

Munidos do nosso conhecimento de precedência e associatividade, devemos agora ser capazes de decompor o código em seu comportamento de agrupamento assim:

```js
((a && b) || c) ? ((c || b) ? a : (c && b)) : a
```

Ou, para apresentá-lo indentado se isso for mais fácil de entender:

```js
(
  (a && b)
    ||
  c
)
  ?
(
  (c || b)
    ?
  a
    :
  (c && b)
)
  :
a
```

Vamos resolvê-lo agora:

1. `(a && b)` é `"foo"`.
2. `"foo" || c` é `"foo"`.
3. Para o primeiro teste `?`, `"foo"` é truthy.
4. `(c || b)` é `"foo"`.
5. Para o segundo teste `?`, `"foo"` é truthy.
6. `a` é `42`.

É isso, terminamos! A resposta é `42`, exatamente como vimos antes. Na verdade não foi tão difícil, foi?

### Desambiguação

Você deve agora ter um domínio muito melhor sobre precedência de operadores (e associatividade) e se sentir muito mais confortável em entender como o código com múltiplos operadores encadeados vai se comportar.

Mas uma pergunta importante permanece: devemos todos escrever código entendendo e confiando perfeitamente em todas as regras de precedência/associatividade de operadores? Devemos usar agrupamento manual `( )` apenas quando for necessário para forçar uma ligação/ordem de processamento diferente?

Ou, por outro lado, devemos reconhecer que, embora tais regras *de fato sejam* aprendíveis, há pegadinhas suficientes para justificar ignorar precedência/associatividade automáticas? Se sim, devemos então sempre usar agrupamento manual `( )` e remover toda dependência desses comportamentos automáticos?

Esse debate é altamente subjetivo, e fortemente simétrico ao debate do Capítulo 4 sobre coerção *implícita*. A maioria dos desenvolvedores sente o mesmo sobre ambos os debates: ou aceitam ambos os comportamentos e codificam esperando por eles, ou descartam ambos os comportamentos e se atêm a idiomas manuais/explícitos.

Claro, não posso responder essa pergunta definitivamente para o leitor aqui mais do que pude no Capítulo 4. Mas apresentei a você os prós e contras, e esperançosamente encorajei entendimento profundo suficiente para que você possa tomar decisões informadas em vez de movidas por hype.

Na minha opinião, há um importante meio-termo. Devemos misturar tanto precedência/associatividade de operadores *quanto* agrupamento manual `( )` em nossos programas -- argumento da mesma forma no Capítulo 4 a favor do uso saudável/seguro da coerção *implícita*, mas certamente não a endosso exclusivamente sem limites.

Por exemplo, `if (a && b && c) ..` está perfeitamente OK para mim, e eu não faria `if ((a && b) && c) ..` apenas para destacar explicitamente a associatividade, porque acho que é excessivamente verboso.

Por outro lado, se eu precisasse encadear dois operadores condicionais `? :` juntos, eu certamente usaria agrupamento manual `( )` para deixar absolutamente claro qual é a minha lógica pretendida.

Assim, meu conselho aqui é semelhante ao do Capítulo 4: **use precedência/associatividade de operadores onde isso leve a um código mais curto e mais limpo, mas use agrupamento manual `( )` em lugares onde ele ajude a criar clareza e reduzir confusão.**

## Ponto e Vírgula Automático

ASI (Automatic Semicolon Insertion, Inserção Automática de Ponto e Vírgula) é quando o JavaScript assume um `;` em certos lugares do seu programa JS mesmo que você não tenha colocado um lá.

Por que ele faria isso? Porque se você omitir até mesmo um único `;` obrigatório seu programa falharia. Não muito tolerante. O ASI permite que o JS seja tolerante em certos lugares onde `;` comumente não são considerados necessários.

É importante notar que o ASI só terá efeito na presença de uma nova linha (também conhecida como quebra de linha). Pontos e vírgulas não são inseridos no meio de uma linha.

Basicamente, se o analisador (parser) JS analisa uma linha onde ocorreria um erro de analisador (um `;` esperado faltando), e ele pode razoavelmente inserir um, ele o faz. O que é razoável para inserção? Apenas se não houver nada além de espaço em branco e/ou comentários entre o fim de alguma instrução e a nova linha/quebra de linha daquela linha.

Considere:

```js
var a = 42, b
c;
```

O JS deveria tratar o `c` na próxima linha como parte da instrução `var`? Certamente trataria se um `,` tivesse aparecido em algum lugar (mesmo em outra linha) entre `b` e `c`. Mas já que não há um, o JS assume em vez disso que há um `;` implícito (na nova linha) depois de `b`. Assim, `c;` é deixado como uma instrução de expressão independente.

De forma semelhante:

```js
var a = 42, b = "foo";

a
b	// "foo"
```

Esse ainda é um programa válido sem erro, porque instruções de expressão também aceitam ASI.

Há certos lugares onde o ASI é útil, como por exemplo:

```js
var a = 42;

do {
	// ..
} while (a)	// <-- ; esperado aqui!
a;
```

A gramática requer um `;` depois de um laço `do..while`, mas não depois de laços `while` ou `for`. Mas a maioria dos desenvolvedores não se lembra disso! Então, o ASI prestativamente intervém e insere um.

Como dissemos antes neste capítulo, blocos de instrução não requerem terminação com `;`, então o ASI não é necessário:

```js
var a = 42;

while (a) {
	// ..
} // <-- nenhum ; esperado aqui
a;
```

O outro caso principal onde o ASI entra em ação é com as keywords `break`, `continue`, `return`, e (ES6) `yield`:

```js
function foo(a) {
	if (!a) return
	a *= 2;
	// ..
}
```

A instrução `return` não atravessa a nova linha até a expressão `a *= 2`, pois o ASI assume o `;` terminando a instrução `return`. Claro, instruções `return` *podem* facilmente quebrar em múltiplas linhas, só não quando não há nada depois de `return` além da nova linha/quebra de linha.

```js
function foo(a) {
	return (
		a * 2 + 3 / 12
	);
}
```

Raciocínio idêntico se aplica a `break`, `continue`, e `yield`.

### Correção de Erro

Uma das *guerras religiosas* mais acaloradas na comunidade JS (além de tabs vs. espaços) é se deve-se confiar fortemente/exclusivamente no ASI ou não.

A maioria, mas não todos, dos pontos e vírgulas é opcional, mas os dois `;` no cabeçalho do laço `for ( .. ) ..` são obrigatórios.

Do lado a favor desse debate, muitos desenvolvedores acreditam que o ASI é um mecanismo útil que lhes permite escrever código mais conciso (e mais "bonito") omitindo todos os `;` exceto os estritamente obrigatórios (que são pouquíssimos). É frequentemente afirmado que o ASI torna muitos `;` opcionais, então um programa corretamente escrito *sem eles* não é diferente de um programa corretamente escrito *com eles*.

Do lado contra do debate, muitos outros desenvolvedores afirmarão que há *lugares demais* que podem ser pegadinhas acidentais, especialmente para desenvolvedores mais novos e menos experientes, onde `;` involuntários sendo magicamente inseridos mudam o significado. De forma semelhante, alguns desenvolvedores argumentarão que se eles omitem um ponto e vírgula, é um erro descarado, e querem que suas ferramentas (linters, etc.) o detectem antes que o motor JS *corrija* o erro por baixo dos panos.

Deixe-me apenas compartilhar minha perspectiva. Uma leitura estrita da especificação implica que o ASI é uma rotina de "correção de erro". Que tipo de erro, você pode perguntar? Especificamente, um **erro de analisador**. Em outras palavras, numa tentativa de fazer o analisador falhar menos, o ASI deixa-o ser mais tolerante.

Mas tolerante de quê? No meu ponto de vista, a única forma de um **erro de analisador** ocorrer é se ele recebe um programa incorreto/com erro para analisar. Então, embora o ASI esteja estritamente corrigindo erros de analisador, a única forma de ele poder obter tais erros é se houvesse primeiro erros de autoria do programa -- omitir pontos e vírgulas onde as regras gramaticais os requerem.

Então, para colocar de forma mais contundente, quando ouço alguém afirmar que quer omitir "pontos e vírgulas opcionais," meu cérebro traduz essa afirmação para "quero escrever o programa mais quebrado em termos de analisador que ainda funcione."

Acho essa posição ridícula de se tomar e os argumentos de economizar digitação e ter "código mais bonito" são fracos na melhor das hipóteses.

Além disso, não concordo que isso seja a mesma coisa que o debate de espaços vs tabs -- que seja puramente cosmético -- mas sim acredito que seja uma questão fundamental de escrever código que adere aos requisitos gramaticais vs. código que depende de exceções gramaticais para passar raspando.

Outra forma de ver isso é que confiar no ASI é essencialmente considerar as novas linhas como "espaço em branco" significativo. Outras linguagens como Python têm espaço em branco verdadeiramente significativo. Mas é realmente apropriado pensar no JavaScript como tendo novas linhas significativas tal como ele está hoje?

Minha posição: **use pontos e vírgulas onde você sabe que eles são "obrigatórios," e limite suas suposições sobre o ASI ao mínimo.**

Mas não acredite só na minha palavra. Lá em 2012, o criador do JavaScript Brendan Eich disse (http://brendaneich.com/2012/04/the-infernal-semicolon/) o seguinte:

> A moral dessa história: o ASI é (falando formalmente) um procedimento de correção de erro sintático. Se você começar a codificar como se ele fosse uma regra universal de nova linha significativa, você vai se meter em encrenca.
> ..
> Eu queria ter tornado as novas linhas mais significativas no JS lá naqueles dez dias de maio de 1995.
> ..
> Tenha cuidado para não usar o ASI como se ele desse ao JS novas linhas significativas.

## Erros

O JavaScript não só tem diferentes *subtipos* de erros (`TypeError`, `ReferenceError`, `SyntaxError`, etc.), mas também a gramática define que certos erros sejam impostos em tempo de compilação, em comparação a todos os outros erros que acontecem em tempo de execução.

Em particular, há muito tempo uma série de condições específicas que devem ser detectadas e reportadas como "erros precoces" (durante a compilação). Qualquer erro de sintaxe puro é um erro precoce (por exemplo, `a = ,`), mas também a gramática define coisas que são sintaticamente válidas mas mesmo assim proibidas.

Já que a execução do seu código ainda não começou, esses erros não são capturáveis com `try..catch`; eles simplesmente farão a análise/compilação do seu programa falhar.

**Dica:** Não há requisito na especificação sobre exatamente como os navegadores (e ferramentas de desenvolvedor) devem reportar erros. Então você pode ver variações entre navegadores nos exemplos de erro a seguir, em qual subtipo específico de erro é reportado ou qual será o texto da mensagem de erro incluída.

Um exemplo simples é com a sintaxe dentro de um literal de expressão regular. Não há nada de errado com a sintaxe JS aqui, mas o regex inválido lançará um erro precoce:

```js
var a = /+foo/;		// Error!
```

O alvo de uma atribuição deve ser um identificador (ou uma expressão de desestruturação ES6 que produza um ou mais identificadores), então um valor como `42` naquela posição é ilegal e pode ser reportado imediatamente:

```js
var a;
42 = a;		// Error!
```

O modo `strict` do ES5 define ainda mais erros precoces. Por exemplo, no modo `strict`, nomes de parâmetros de função não podem ser duplicados:

```js
function foo(a,b,a) { }					// tudo bem

function bar(a,b,a) { "use strict"; }	// Error!
```

Outro erro precoce do modo `strict` é um literal de objeto ter mais de uma propriedade com o mesmo nome:

```js
(function(){
	"use strict";

	var a = {
		b: 42,
		b: 43
	};			// Error!
})();
```

**Nota:** Semanticamente falando, tais erros não são tecnicamente erros de *sintaxe* mas mais erros de *gramática* -- os trechos acima são sintaticamente válidos. Mas já que não há um tipo `GrammarError`, alguns navegadores usam `SyntaxError` em vez disso.

### Usando Variáveis Cedo Demais

O ES6 define um novo conceito (francamente de nome confuso) chamado TDZ ("Temporal Dead Zone", Zona Morta Temporal).

A TDZ se refere a lugares no código onde uma referência a variável ainda não pode ser feita, porque ela não atingiu sua inicialização obrigatória.

O exemplo mais claro disso é com o escopo de bloco `let` do ES6:

```js
{
	a = 2;		// ReferenceError!
	let a;
}
```

A atribuição `a = 2` está acessando a variável `a` (que está de fato com escopo de bloco para o bloco `{ .. }`) antes de ela ter sido inicializada pela declaração `let a`, então ela está na TDZ para `a` e lança um erro.

Interessantemente, embora `typeof` tenha uma exceção para ser seguro com variáveis não declaradas (veja o Capítulo 1), nenhuma exceção de segurança desse tipo é feita para referências da TDZ:

```js
{
	typeof a;	// undefined
	typeof b;	// ReferenceError! (TDZ)
	let b;
}
```

## Argumentos de Função

Outro exemplo de violação da TDZ pode ser visto com valores de parâmetro padrão do ES6 (veja o título *ES6 & Beyond* desta série):

```js
var b = 3;

function foo( a = 42, b = a + b + 5 ) {
	// ..
}
```

A referência `b` na atribuição aconteceria na TDZ para o parâmetro `b` (não puxa a referência `b` externa), então ela lançará um erro. Entretanto, o `a` na atribuição está OK já que naquele momento ele já passou da TDZ para o parâmetro `a`.

Ao usar os valores de parâmetro padrão do ES6, o valor padrão é aplicado ao parâmetro se você ou omite um argumento, ou passa um valor `undefined` em seu lugar:

```js
function foo( a = 42, b = a + 1 ) {
	console.log( a, b );
}

foo();					// 42 43
foo( undefined );		// 42 43
foo( 5 );				// 5 6
foo( void 0, 7 );		// 42 7
foo( null );			// null 1
```

**Nota:** `null` é coagido para um valor `0` na expressão `a + 1`. Veja o Capítulo 4 para mais informações.

Da perspectiva dos valores de parâmetro padrão do ES6, não há diferença entre omitir um argumento e passar um valor `undefined`. Entretanto, há uma forma de detectar a diferença em alguns casos:

```js
function foo( a = 42, b = a + 1 ) {
	console.log(
		arguments.length, a, b,
		arguments[0], arguments[1]
	);
}

foo();					// 0 42 43 undefined undefined
foo( 10 );				// 1 10 11 10 undefined
foo( 10, undefined );	// 2 10 11 10 undefined
foo( 10, null );		// 2 10 null 10 null
```

Mesmo que os valores de parâmetro padrão sejam aplicados aos parâmetros `a` e `b`, se nenhum argumento foi passado naqueles slots, o array `arguments` não terá entradas.

Inversamente, se você passa um argumento `undefined` explicitamente, uma entrada existirá no array `arguments` para aquele argumento, mas será `undefined` e não (necessariamente) a mesma que o valor padrão que foi aplicado ao parâmetro nomeado para aquele mesmo slot.

Embora os valores de parâmetro padrão do ES6 possam criar divergência entre o slot do array `arguments` e o parâmetro nomeado correspondente, essa mesma desconexão também pode ocorrer de formas traiçoeiras no ES5:

```js
function foo(a) {
	a = 42;
	console.log( arguments[0] );
}

foo( 2 );	// 42 (vinculado)
foo();		// undefined (não vinculado)
```

Se você passa um argumento, o slot de `arguments` e o parâmetro nomeado são vinculados para sempre ter o mesmo valor. Se você omite o argumento, nenhuma vinculação desse tipo ocorre.

Mas no modo `strict`, a vinculação não existe independentemente:

```js
function foo(a) {
	"use strict";
	a = 42;
	console.log( arguments[0] );
}

foo( 2 );	// 2 (não vinculado)
foo();		// undefined (não vinculado)
```

É quase certamente uma má ideia confiar em qualquer vinculação desse tipo, e na verdade a própria vinculação é uma abstração com vazamento que está expondo um detalhe de implementação subjacente do motor, em vez de um recurso projetado apropriadamente.

O uso do array `arguments` foi descontinuado (especialmente em favor dos parâmetros rest `...` do ES6 -- veja o título *ES6 & Beyond* desta série), mas isso não significa que seja tudo ruim.

Antes do ES6, `arguments` é a única forma de obter um array de todos os argumentos passados para repassar a outras funções, o que acaba sendo bem útil. Você também pode misturar parâmetros nomeados com o array `arguments` e estar seguro, desde que siga uma regra simples: **nunca se refira a um parâmetro nomeado *e* seu slot `arguments` correspondente ao mesmo tempo.** Se você evita essa má prática, nunca exporá o comportamento de vinculação com vazamento.

```js
function foo(a) {
	console.log( a + arguments[1] ); // seguro!
}

foo( 10, 32 );	// 42
```

## `try..finally`

Você provavelmente está familiarizado com como o bloco `try..catch` funciona. Mas você já parou para considerar a cláusula `finally` que pode ser pareada com ele? Na verdade, você sabia que o `try` só requer ou `catch` ou `finally`, embora ambos possam estar presentes se necessário.

O código na cláusula `finally` *sempre* roda (não importa o quê), e ele sempre roda logo após o `try` (e o `catch` se presente) terminarem, antes de qualquer outro código rodar. Em certo sentido, você pode mais ou menos pensar no código numa cláusula `finally` como estando numa função de callback que sempre será chamada independentemente de como o resto do bloco se comporta.

Então o que acontece se houver uma instrução `return` dentro de uma cláusula `try`? Ela obviamente retornará um valor, certo? Mas o código chamador que recebe esse valor roda antes ou depois do `finally`?

```js
function foo() {
	try {
		return 42;
	}
	finally {
		console.log( "Hello" );
	}

	console.log( "never runs" );
}

console.log( foo() );
// Hello
// 42
```

O `return 42` roda imediatamente, o que configura o valor de conclusão da chamada `foo()`. Essa ação completa a cláusula `try` e a cláusula `finally` roda imediatamente em seguida. Só então a função `foo()` está completa, para que seu valor de conclusão seja retornado de volta para a instrução `console.log(..)` usar.

O exato mesmo comportamento é verdadeiro para um `throw` dentro do `try`:

```js
 function foo() {
	try {
		throw 42;
	}
	finally {
		console.log( "Hello" );
	}

	console.log( "never runs" );
}

console.log( foo() );
// Hello
// Uncaught Exception: 42
```

Agora, se uma exceção é lançada (acidentalmente ou intencionalmente) dentro de uma cláusula `finally`, ela substituirá como a conclusão primária daquela função. Se um `return` anterior no bloco `try` tinha configurado um valor de conclusão para a função, esse valor será abandonado.

```js
function foo() {
	try {
		return 42;
	}
	finally {
		throw "Oops!";
	}

	console.log( "never runs" );
}

console.log( foo() );
// Uncaught Exception: Oops!
```

Não deveria ser surpreendente que outras instruções de controle não lineares como `continue` e `break` exibam comportamento semelhante a `return` e `throw`:

```js
for (var i=0; i<10; i++) {
	try {
		continue;
	}
	finally {
		console.log( i );
	}
}
// 0 1 2 3 4 5 6 7 8 9
```

A instrução `console.log(i)` roda no fim da iteração do laço, o que é causado pela instrução `continue`. Entretanto, ela ainda roda antes da instrução de atualização da iteração `i++`, que é por isso que os valores impressos são `0..9` em vez de `1..10`.

**Nota:** O ES6 adiciona uma instrução `yield`, em geradores (veja o título *Async & Performance* desta série) que de certas formas pode ser vista como uma instrução `return` intermediária. Entretanto, diferentemente de um `return`, um `yield` não está completo até o gerador ser retomado, o que significa que um `try { .. yield .. }` não foi completado. Então uma cláusula `finally` anexada não rodará logo após o `yield` como acontece com `return`.

Um `return` dentro de um `finally` tem a habilidade especial de substituir um `return` anterior do `try` ou da cláusula `catch`, mas apenas se `return` for explicitamente chamado:

```js
function foo() {
	try {
		return 42;
	}
	finally {
		// nenhum `return ..` aqui, então nenhuma substituição
	}
}

function bar() {
	try {
		return 42;
	}
	finally {
		// substitui o `return 42` anterior
		return;
	}
}

function baz() {
	try {
		return 42;
	}
	finally {
		// substitui o `return 42` anterior
		return "Hello";
	}
}

foo();	// 42
bar();	// undefined
baz();	// "Hello"
```

Normalmente, a omissão de `return` numa função é o mesmo que `return;` ou até `return undefined;`, mas dentro de um bloco `finally` a omissão de `return` não age como um `return undefined` que substitui; ela apenas deixa o `return` anterior valer.

Na verdade, podemos realmente aumentar a loucura se combinarmos `finally` com `break` rotulado (discutido anteriormente no capítulo):

```js
function foo() {
	bar: {
		try {
			return 42;
		}
		finally {
			// sai do bloco rotulado `bar`
			break bar;
		}
	}

	console.log( "Crazy" );

	return "Hello";
}

console.log( foo() );
// Crazy
// Hello
```

Mas... não faça isso. Sério. Usar um `finally` + `break` rotulado para efetivamente cancelar um `return` é fazer o seu melhor para criar o código mais confuso possível. Eu apostaria que nenhuma quantidade de comentários redimiria esse código.

## `switch`

Vamos explorar brevemente a instrução `switch`, uma espécie de abreviação sintática para uma cadeia de instruções `if..else if..else..`.

```js
switch (a) {
	case 2:
		// faz alguma coisa
		break;
	case 42:
		// faz outra coisa
		break;
	default:
		// recai aqui
}
```

Como você pode ver, ela avalia `a` uma vez, depois compara o valor resultante com cada expressão `case` (apenas expressões de valor simples aqui). Se uma correspondência é encontrada, a execução começará naquele `case` correspondente, e irá ou até um `break` ser encontrado ou até o fim do bloco `switch` ser alcançado.

Isso pode não te surpreender, mas há vários caprichos sobre o `switch` que você pode não ter notado antes.

Primeiro, a correspondência que ocorre entre a expressão `a` e cada expressão `case` é idêntica ao algoritmo `===` (veja o Capítulo 4). Muitas vezes os `switch`es são usados com valores absolutos nas instruções `case`, como mostrado acima, então a correspondência estrita é apropriada.

Entretanto, você pode querer permitir igualdade coercitiva (também conhecida como `==`, veja o Capítulo 4), e para fazer isso você precisará meio que "hackear" um pouco a instrução `switch`:

```js
var a = "42";

switch (true) {
	case a == 10:
		console.log( "10 or '10'" );
		break;
	case a == 42:
		console.log( "42 or '42'" );
		break;
	default:
		// nunca chega aqui
}
// 42 or '42'
```

Isso funciona porque a cláusula `case` pode ter qualquer expressão (não apenas valores simples), o que significa que ela comparará estritamente o resultado dessa expressão com a expressão de teste (`true`). Já que `a == 42` resulta em `true` aqui, a correspondência é feita.

Apesar do `==`, a própria correspondência do `switch` ainda é estrita, entre `true` e `true` aqui. Se a expressão `case` resultasse em algo que fosse truthy mas não estritamente `true` (veja o Capítulo 4), não funcionaria. Isso pode te morder se você por exemplo estiver usando um "operador lógico" como `||` ou `&&` na sua expressão:

```js
var a = "hello world";
var b = 10;

switch (true) {
	case (a || b == 10):
		// nunca chega aqui
		break;
	default:
		console.log( "Oops" );
}
// Oops
```

Já que o resultado de `(a || b == 10)` é `"hello world"` e não `true`, a correspondência estrita falha. Neste caso, a correção é forçar a expressão explicitamente para ser um `true` ou `false`, como `case !!(a || b == 10):` (veja o Capítulo 4).

Por fim, a cláusula `default` é opcional, e ela não tem necessariamente que vir no fim (embora essa seja a forte convenção). Mesmo na cláusula `default`, as mesmas regras se aplicam sobre encontrar um `break` ou não:

```js
var a = 10;

switch (a) {
	case 1:
	case 2:
		// nunca chega aqui
	default:
		console.log( "default" );
	case 3:
		console.log( "3" );
		break;
	case 4:
		console.log( "4" );
}
// default
// 3
```

**Nota:** Como discutido anteriormente sobre `break`s rotulados, o `break` dentro de uma cláusula `case` também pode ser rotulado.

A forma como esse trecho processa é que ele passa por toda a correspondência das cláusulas `case` primeiro, não encontra nenhuma correspondência, depois volta para a cláusula `default` e começa a executar. Já que não há `break` ali, ele continua executando no bloco `case 3` já pulado, antes de parar ao atingir aquele `break`.

Embora esse tipo de lógica circular seja claramente possível em JavaScript, não há quase nenhuma chance de que ela resulte em código razoável ou compreensível. Seja muito cético se você se pegar querendo criar tal fluxo de lógica circular, e se você realmente fizer, certifique-se de incluir muitos comentários de código para explicar o que você está aprontando!

## Revisão

A gramática do JavaScript tem bastante nuance à qual nós, como desenvolvedores, deveríamos dedicar um pouco mais de tempo prestando atenção mais de perto do que tipicamente fazemos. Um pouquinho de esforço rende muito na solidificação do seu conhecimento mais profundo da linguagem.

Instruções e expressões têm análogos no idioma inglês -- instruções são como sentenças e expressões são como frases. Expressões podem ser puras/autocontidas, ou podem ter efeitos colaterais.

A gramática do JavaScript dispõe regras de uso semântico (também conhecidas como contexto) sobre a sintaxe pura. Por exemplo, pares `{ }` usados em vários lugares do seu programa podem significar blocos de instrução, literais de `object`, atribuições de desestruturação (ES6), ou argumentos de função nomeados (ES6).

Todos os operadores do JavaScript têm regras bem definidas para precedência (quais se ligam primeiro antes de outros) e associatividade (como múltiplas expressões de operador são implicitamente agrupadas). Uma vez que você aprenda essas regras, cabe a você decidir se precedência/associatividade são *implícitas demais* para o seu próprio bem, ou se ajudarão a escrever um código mais curto e mais claro.

O ASI (Automatic Semicolon Insertion, Inserção Automática de Ponto e Vírgula) é um mecanismo de correção de erro de analisador embutido no motor JS, que lhe permite sob certas circunstâncias inserir um `;` assumido em lugares onde ele é obrigatório, foi omitido, *e* onde a inserção corrige o erro do analisador. O debate se acalora sobre se esse comportamento implica que a maioria dos `;` é opcional (e pode/deve ser omitida para um código mais limpo) ou se significa que omiti-los é cometer erros que o motor JS apenas limpa para você.

O JavaScript tem vários tipos de erros, mas é menos conhecido que ele tem duas classificações para erros: "precoces" (lançados pelo compilador, não capturáveis) e "de tempo de execução" (capturáveis com `try..catch`). Todos os erros de sintaxe são obviamente erros precoces que param o programa antes de ele rodar, mas há outros também.

Argumentos de função têm uma relação interessante com seus parâmetros nomeados formalmente declarados. Especificamente, o array `arguments` tem uma série de pegadinhas de comportamento de abstração com vazamento se você não tomar cuidado. Evite `arguments` se puder, mas se precisar usá-lo, de toda forma evite usar o slot posicional em `arguments` ao mesmo tempo que usa um parâmetro nomeado para aquele mesmo argumento.

A cláusula `finally` anexada a um `try` (ou `try..catch`) oferece alguns caprichos muito interessantes em termos de ordem de processamento da execução. Alguns desses caprichos podem ser úteis, mas é possível criar muita confusão, especialmente se combinados com blocos rotulados. Como sempre, use `finally` para tornar o código melhor e mais claro, não mais esperto ou confuso.

O `switch` oferece uma boa abreviação para instruções `if..else if..`, mas tenha cuidado com muitas suposições simplificadoras comuns sobre seu comportamento. Há vários caprichos que podem te derrubar se você não tomar cuidado, mas há também alguns truques ocultos legais que o `switch` tem na manga!
