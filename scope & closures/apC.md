# You Don't Know JS: Escopos & Clausuras
# Apêndice C: Lexical-this

Embora este título não aborde o mecanismo `this` em seus detalhes, existe um tópico que se refere ao escopo léxico `this` de forma significativa, que vamos examinar rapidamente.

ES6 adiciona uma declaração especial de função chamada "arrow function". Que se parece com isto:

```js
var foo = a => {
	console.log( a );
};

foo( 2 ); // 2
```

A chamada "fat arrow" é frequentemente mencionada como um atalho para a *tediosamente detalhada* (sarcasmo) palavra chave `function`.

Mas há algo muito mais importante acontecendo com `arrow-functions` e que não tem nada a ver com escrever menos caracteres em sua declaração.

Resumidamente, este código sofre um problema:

```js

var obj = {
	id: "awesome",
	cool: function coolFn() {
		console.log( this.id );
	}
};

var id = "not awesome";

obj.cool(); // awesome

setTimeout( obj.cool, 100 ); // not awesome
```

O problema é a perda do vínculo `this` na função `cool()`.  Existem várias maneiras de lidar com esse problema, mas uma solução, muitas vezes repetida é `var self = this;`.

Que pode parecer com isto:

```js
var obj = {
	count: 0,
	cool: function coolFn() {
		var self = this;

		if (self.count < 1) {
			setTimeout( function timer(){
				self.count++;
				console.log( "awesome?" );
			}, 100 );
		}
	}
};

obj.cool(); // awesome?
```

Sem entrar em muitos detalhes aqui, a solução `var self = this` apenas dispensa todo o problema de compreensão e uso correto do vínculo `this` ao inves de tentarmos algo que talvez seja mais confortável como: `escopo léxico`. O `self` torna-se apenas um identificador que pode ser resolvido através de escopo e encerramento, e não se preocupa com o que aconteceu com o vínculo `this` ao longo do caminho.

As pessoas não gostam de escrever coisas detalhadas, especialmente quando eles fazem frequentemente.
Assim, a motivação do ES6 é para ajudar a aliviar estes cenários, e de fato, *corrigir* problemas idiomáticos comuns, como este.

A solução ES6, `arrow-function` apresenta um comportamento chamado "lexical this".

```js
var obj = {
	count: 0,
	cool: function coolFn() {
		if (this.count < 1) {
			setTimeout( () => { // arrow-function ftw?
				this.count++;
				console.log( "awesome?" );
			}, 100 );
		}
	}
};

obj.cool(); // awesome?
```

Em uma curta explicação é que `arrow-functions` não se comportam como funções normais quando se trata do vínculo `this`. Elas rejeitam todas as regras normais para o vinculo `this`, Em vez de assumir o valor do seu escopo lexico delimitador imediato. seja ele qual for.

Então, nesse trecho, a `arrow-function` não recebe seu `this` desacoplado em uma forma imprevisível, ela apenas herda o vinculo `this` da função `cool()` (que é o correto se invocarmos como o mostrado!).

Ainda que sirva para encurtar código, minha perspectiva é que as "arrow functions" são na verdade apenas codificacão de *erros* comuns do desenvolvedor na sintaxe da linguagem, que são para confundir e associar as regras do "vínculo this" com as regras de "escopo léxico".

Em outras palavras: por que usar o verboso e confuso paradigma do estilo de código `this`, apenas para encurtá-lo, misturando-o com referências léxicas. Parece natural escolher uma ou outra abordagem para diferentes pedaços de código, e não misturá-los na mesma parte.

**Nota:** uma outra depreciação das `arrow-functions` é que elas são anônimas. Veja o Capítulo 3 para as razões pelas quais funções anônimas são menos desejáveis do que as funções nomeadas.

Na minha perspectiva, uma abordagem mais adequada para este "problema", é compreender o mecanismo `this` corretamente.

```js
var obj = {
	count: 0,
	cool: function coolFn() {
		if (this.count < 1) {
			setTimeout( function timer(){
				this.count++; // `this` is safe because of `bind(..)`
				console.log( "more awesome" );
			}.bind( this ), 100 ); // look, `bind()`!
		}
	}
};

obj.cool(); // more awesome
```

Independentemente se você prefere o novo comportamento `lexical-this` das arrow-functions, ou se você prefere o testado e comprovado `bind()`, é importante notar que `arrow-functions` **não** são apenas para economizar caracteres ao digitar "function".

Elas têm uma diferença de comportamento intencional (ou se preferir: importância diferente) que devemos entender e aprender.

Agora que entendemos plenamente escopo lexico (e closure!), entender `lexical-this` deve ser uma brisa!
