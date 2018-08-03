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

Agravado pela rápida evolução de funcionalidades, surge um problema para os desenvolvedores JS que desejam muito utilizar as novas funcionalidades, ao mesmo tempo que precisam enfrentar a realidade de que seus sites/aplicativos precisam ser suportados por navegadores mais antigos que não possuem suporte.

A maneira que o ES5 parece ter ficado fora de moda por grande parte da indústria, o mindset típico era que a codebase esperasse adotarem o ES5 até que a maioria, se não todos, os ambientes pré-ES5 deixassem de ser suportados. Como resultado disso, muitos só recentemente (até o no momento que estou escrevendo) começaram a adotar coisas como o modo estrito (strict mode), que chegou ao ES5 há cinco anos.

Em geral, é considerado uma abordagem prejudicial para o futuro do ecossistema JS esperar e seguir a especificação por muitos anos. Todos os responsáveis pela evolução da linguagem desejam que os desenvolvedores comecem a basear seus códigos nos novas funcionalidades e padrões assim que se estabilizarem em forma de especificação e os navegadores tiverem a chance de implementá-los.

Então, como resolvemos essa aparente contradição? A resposta é tooling, especificamente uma técnica chamada *transpiling* (transformação + compilação). A grosso modo, a ideia é usar uma ferramenta especial para transformar seu código ES6 em equivalente (ou parecido!) que funcionem em ambientes ES5.

Por exemplo, considere definições de propriedades simplificadas (consulte "Extensões de Objetos Literais" no Capítulo 2). Aqui está o formulário ES6:

```js
var foo = [1,2,3];

var obj = {
	foo		// significa `foo: foo`
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

Está é uma pequena, mas agradável, transformação que nos permite encurtar o ‘foo: foo’ em uma declaração literal do objeto para apenas ‘foo’, se os nomes forem os mesmos.

Transpiladores realizam essas transformações para você, geralmente em uma etapa do workflow semelhante à forma como você realiza o lint, minificação e outras operações similares.

### Shims/Polyfills

Nem todas as novas funcionalidades do ES6 precisam de um transpilador. Polyfills (também conhecido como Shims) são um padrão para definir o comportamento equivalente de um ambiente mais novo em um ambiente antigo, quando possível. A sintaxe não pode ser *polyfizada*, mas as APIs geralmente podem ser.

Por exemplo, `Object.is (..)` é uma nova maneira para verificar a igualdade estrita de dois valores, mas sem as exceções com diferenças que `===` tem para os valores `NaN` e` -0`. O polyfill para `Object.is (..)` é bastante simples:

```js
if (!Object.is) {
	Object.is = function(v1, v2) {
		// teste para `-0`
		if (v1 === 0 && v2 === 0) {
			return 1 / v1 === 1 / v2;
		}
		// teste para `NaN`
		if (v1 !== v1) {
			return v2 !== v2;
		}
		// todo o resto
		return v1 === v2;
	};
}
```

**Dica:** Preste atenção no lado de fora da declaração do `if` em volta do polyfill. Esse é um detalhe importante, que significa que o trecho define apenas o comportamento de fallback para ambientes mais antigos em que a API em questão ainda não está definida; seria muito raro você querer sobrescrever uma API existente.

Há uma grande coleção de ES6 shims chamada "ES6 Shim" (https://github.com/paulmillr/es6-shim/) que você deve definitivamente adotar como parte padrão de qualquer novo projeto JavaScript!

Presume-se que o javaScript continuará evoluindo constantemente, com os navegadores implementando suporte para funcionalidades continuamente, em vez de em grandes blocos. Portanto, a melhor estratégia para se manter atualizado à medida que evolui é simplesmente inserir polyfill shims no seu codebase, e um passo para o transpilador em seu workflow de desenvolvimento, agora mesmo, e se acostume a essa nova realidade.

Se você decidir deixar como está e esperar que todos os navegadores sem as funcionalidades sejam suportados antes de você começar a usar a funcionalidade, você sempre estará muito atrasado. Infelizmente, você perderá todas as inovações projetadas para tornar a escrita do JavaScript mais eficaz, eficiente e robusta.

## Revisando

ES6 (alguns podem tentar chamá-lo de ES2015) é apenas o pouso a partir do momento em que este está sendo escrito, e tem muitas coisas novas que você precisa aprender!

Mas é ainda mais importante mudar sua mentalidade para alinhar com a nova maneira que o JavaScript vai evoluir. Não é apenas esperar por anos para algum documento oficial obter um voto de aprovação, como muitos fizeram no passado.

Agora, as funcionalidades do JavaScript aparecem nos navegadores à medida que se tornam prontos, e cabe a você decidir se embarcará cedo no trem ou se irá vai ficar correndo atrás do prejuízo daqui em diante.

Independente dos rótulos que o futuro adote, ele vai se mover muito mais rápido do que nunca. Transpiladores e shims/polyfills são ferramentas importantíssimas para manter você à frente de onde a linguagem é direcionada.

Se existe alguma narrativa importante para entender sobre a nova realidade para JavaScript, é que todos os desenvolvedores de JS estão fortemente implorando para sair da borda da curva para a borda de ataque. E aprender ES6 é onde tudo começa!
