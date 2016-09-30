# You Don't Know JS: ES6 & Beyond
# Capítulo 5: Coleções

Coleções estruturadas e acesso a informações são componentes críticos de praticamente qualquer programa JS. Desde o começo da linguagem até hoje, os arrays e objetos tem sido nosso principal mecanismo para se criar estruturas de dados. É claro, muitas estruturas de alto-nível foram construídas no topo destas, como bibliotecas que rodam em modo usuário.

A partir do ES6, algumas das abstrações de estruturas de dados mais úteis (e otimizadas para performance!) foram adicionados como componentes nativos da linguagem.

Nós começaremos este capítulo observando primeiramente TypedArrays (Arrays Tipados), tecnicamente contemporânea aos esforços do ES5 já há alguns anos, mas apenas padronizada como acompanhante do WebGL e não do JavaScript. A partir do ES6, eles foram adotadas diretamente pela especificação da linguagem, o que lhes garante o status de primeira-classe.

Maps (Mapas) são como objetos (pares de chave/valor), mas ao invés de apenas uma string para a chave, você pode usar qualquer valor -- até mesmo outro objeto ou mapa! Sets (Conjuntos) são similares à arrays (listas de valores), mas os valores são únicos; caso você adicione um valor duplicado, é ignorado. Também existem as versões fracas (em relação ao coletor de memória/lixo): WeakMap e WeakSet.

## TypedArrays (Arrays Tipados)

Como nós vimos no livro *Tipos & Gramática* desta série, JS tem um conjunto de tipos embutidos (built-in), como `number` e `string`. Seria tentador olhar para uma funcionalidade chamada "vetor tipado" e assumir que isso significa um vetor com valores de um tipo específico, como um vetor que contenha apenas strings.

Entretanto, TypedArrays tem mais a ver com prover acesso estruturado à informações binárias usando uma semântica similar à de arrays (acesso indexado, etc.). O "tipo" no nome se refere a uma "view" que é colocado no tipo do balde de bits, o que é essencialmente um mapeamento se os bits devem ser vistos como uma array de 8-bits inteiros com sinais, 16-bits de inteiros com sinais, e assim por adiante.

Como se constrói esse esse tal balde de bits? Ele é chamado de "buffer," e você o constrói mais diretamente com o construtor `ArrayBuffer(..)`:

```js
var buf = new ArrayBuffer( 32 );
buf.byteLength;							// 32
```

`buf` é agora um buffer binário com tamanho de 32-bytes (256-bits), onde todos os valores são pré-inicializados para `0`. Um buffer por si só não permite qualquer interação, com a exceção de verificar qual é a sua propriedade `byteLength`.

**Dica:** Diversas funcionalidades de plataformas web usam ou retornam arrays de buffer, por exemplo `FileReader#readAsArrayBuffer(..)`, `XMLHttpRequest#send(..)`, e `ImageData` (canvas data).

Mas por cima deste vetor de buffer, você pode então colocar uma "view," que vem na forma de uma TypedArray. Considere:

```js
var arr = new Uint16Array( buf );
arr.length;							// 16
```

`arr` é uma TypedArray de inteiros sem sinais mapeados através do buffer de 256-bit chamado `buf`, isso quer dizer que você tem 16 elementos.

### Extremidade

É muito importante entender que `arr` é mapeado se usando uma configuração endian (big-endian ou little-endian) da plataforma que o JS está rodando. Isso pode ser um problema se os dados binários forem criados em uma extremidade mas interpretados numa plataforma que possuí a extremidade inversa.

Endian significa se o byte de menor ordem (coleção de 8-bits) de um número de vários bytes -- como aqueles de 16-bits inteiros e sem sinais que criamos no trecho anterior -- está à direita ou à esquerda dos bytes do número.

Por exemplo, vamos imaginar o número na base 10 `3085`, que precisa de 16-bits para ser representado. Se você tiver apenas um container para números de 16-bits, ele seria representado como binário por `0000110000001101` (hexadecimal `0c0d`) independentemente da extremidade.

Mas se `3085` fosse representado por dois números de 8-bits, a extremidade afetaria significantemente seu armazenamentos na memória:

* `0000110000001101` / `0c0d` (big endian)
* `0000110100001100` / `0d0c` (little endian)

Se você recebesse os bits de `3085` como `0000110100001100` de um sistema little-endian, mas tivesse colocado uma view por cima dele em um sistema big-endian, você teria visto o valor `3340` (base-10) e `0d0c` (base-16).

Little endian é a representação mais comum na web atualmente, mas definitivamente existem navegadores onde isto não é verdade. É importante que você entenda a extremidade tanto do produtor quanto do consumidor de blocos de dados binários.

Do MDN, aqui esta uma maneira rápida de testar a extremidade do seu JavaScript:

```js
var littleEndian = (function() {
	var buffer = new ArrayBuffer( 2 );
	new DataView( buffer ).setInt16( 0, 256, true );
	return new Int16Array( buffer )[0] === 256;
})();
```

`littleEndian` será `true` ou `false`; para a maioria dos navegadores, deve retornar `true`. Esse teste usa `DataView(..)`, o que permite um nível mais baixo, e um controle mais fino para acessar (definir/pegar) os bits da view que você colocou sobre o buffer. O terceiro parâmetro do método `setInt16(..)` no trecho anterior é para dizer para a `DataView` qual é a extremidade que você quer usar para a operação.

**Cuidado:** Não confunda a extremidade de um armazenamento binário em array de buffers com como um dado número é representado quando exposto em um programa JS. Por exemplo,(3085).toString(2)` retorna`"110000001101"`, o que com uma liderança assumida de quatro `"0"`s parece ser uma representação em big-endian. Na verdade, essa representação é baseada numa única view de 16-bits, não em uma view de dois bytes de 8-bits. O teste do `DataView` é a melhor forma de representar a extremidade do seu ambiente JS.

### Views Múltiplas

Um único buffer pode ter múltiplas views associadas à ele, tal como:

```js
var buf = new ArrayBuffer( 2 );

var view8 = new Uint8Array( buf );
var view16 = new Uint16Array( buf );

view16[0] = 3085;
view8[0];						// 13
view8[1];						// 12

view8[0].toString( 16 );		// "d"
view8[1].toString( 16 );		// "c"

// swap (as if endian!)
var tmp = view8[0];
view8[0] = view8[1];
view8[1] = tmp;

view16[0];						// 3340
```

Os contrutores dos arrays tipados possuem múltiplas variações de assinaturas. Nós mostramos até agora apenas passando estas assinaturas para buffers já existentes. Entretanto, esse método também aceita dois parâmetros extras: `byteOffset` e `length`. Em outras palavras, você pode inicializar uma view de um vetor tipado em uma localização que não seja `0` e pode fazer com que o alcance dela seja menor que o tamanho completo do buffer.

Se o buffer de dados binários incluem dados de tamanhos/localizações não uniformes, esta técnica pode ser bem útil.

Por exemplo, considere o buffer binário que possui um número de 2-bytes (por exemplo "word") no começo, seguido de dois números de 1-byte, seguido por um número ponto flutuante de 32-bits. Você pode acessar os dados com múltiplas views no mesmo buffer, offsets, e tamanhos da seguinte forma:

```js
var first = new Uint16Array( buf, 0, 2 )[0],
	second = new Uint8Array( buf, 2, 1 )[0],
	third = new Uint8Array( buf, 3, 1 )[0],
	fourth = new Float32Array( buf, 4, 4 )[0];
```

### Construtores de TypedArrays

Com adiação à forma `(buffer,[offset, [length]])` examinada na seção anterior, construtores de arrays tipados também suportam esse formato:

* [constructor]`(length)`: Cria uma nova view sobre o novo buffer de `length` (tamanho) bytes
* [constructor]`(typedArr)`: Cria uma nova view e um novo buffer, e copia seu conteúdo para a view `typedArr`
* [constructor]`(obj)`: Cria uma nova view e um novo buffer, e itera sobre o objeto ou objeto parecido com vetor `obj` para copiar seu conteúdo

Os seguintes construtores de arrays tipados estão disponíveis no ES6:

* `Int8Array` (8-bits inteiros sinalizados), `Uint8Array` (8-bits inteiros não sinalizados)
	- `Uint8ClampedArray` (8-bits inteiros não sinalizados, cada valor é travado na configuração para o intervalo de `0`-`255`)
* `Int16Array` (16-bits inteiros sinalizados), `Uint16Array` (16-bits inteiros não sinalizados)
* `Int32Array` (32-bits inteiros sinalizados), `Uint32Array` (32-bits inteiros não sinalizado)
* `Float32Array` (32-bits ponto flutuante, IEEE-754)
* `Float64Array` (64-bits ponto flutuante, IEEE-754)

Instâncias do construtor de arrays tipados são quase iguais as de arrays comuns. Algumas diferenças incluem ter um tamanho fixo e os valores serem todos do mesmo "tipo."

Entretanto, eles compartilham a maoria dos mesmos métodos `prototype`. Assim, você provavelmente vai conseguir usá-los como arrays regulares sem a necessidade de convertê-las.

Por exemplo:

```js
var a = new Int32Array( 3 );
a[0] = 10;
a[1] = 20;
a[2] = 30;

a.map( function(v){
	console.log( v );
} );
// 10 20 30

a.join( "-" );
// "10-20-30"
```

**Cuidado**: Você não pode usar certos métodos `Array.prototype` com TypedArrays que não fazem sentido, como por exemplo os mutadores (`splice(..)`, `push(..)`, etc.) e `concat(..)`.

Esteja ciente que os elementos contidos em TypedArrays são realmente forçados a declarar o tamanho dos bits. Se você tiver um `Uint8Array` e tentar atribuí-lo à algo maior do que o valor de 8-bits em um de seus elementos, o valor será envolvido para que se mantenha no tamanho do bit.

Isso pode causar problemas se você estiver tentando, por exemplo, elevar ao quadrado todos os valores de uma TypedArray. Considere:

```js
var a = new Uint8Array( 3 );
a[0] = 10;
a[1] = 20;
a[2] = 30;

var b = a.map( function(v){
	return v * v;
} );

b;				// [100, 144, 132]
```

Os valores `20` e `30`, quando elevados ao quadrado, resultaram num overflow (transbordeamento) de bits. Para evitar esta limitação, você pode usar a função `TypedArray#from(..)`:

```js
var a = new Uint8Array( 3 );
a[0] = 10;
a[1] = 20;
a[2] = 30;

var b = Uint16Array.from( a, function(v){
	return v * v;
} );

b;				// [100, 400, 900]
```

Veja a seção "`Array.from(..)` Função Estática" do Capítulo 6 para mais informações sobre `Array.from(..)`, que é compartilhado com TypedArrays. Espeficicamente, a seção "Mapeamento" explica a função de mapeamento que é aceita como seu segundo argumento.

Um comportamento interessante para se considerar é que TypedArrays tem um método `sort(..)` bem parecido com arrays normais, mas nesse caso o padrão é fazer comparações númericas ao invés de efetuar coerção nos valores para string, para então efetuar comparações lexícográficas. Por exemplo:

```js
var a = [ 10, 1, 2, ];
a.sort();								// [1,10,2]

var b = new Uint8Array( [ 10, 1, 2 ] );
b.sort();								// [1,2,10]
```

O `TypedArray#sort(..)` recebe de forma opcional uma função de comparação como argumento da mesma forma que o `Array#sort(..)`, que funciona exatamente da mesma maneira.

## Maps (Mapas)

Se você tem muita experiência com JS, você sabe que objetos são o mecanismo primário para se criar estruturas de dados não-ordenadas com pares de chave/valores, também conhecidas como mapas. Entretanto,  a maior desvantagem em se tratar objetos como mapas é a inabilidade de se utilizar um valor que não seja uma string como sua chave.

Por exemplo, considere:

```js
var m = {};

var x = { id: 1 },
	y = { id: 2 };

m[x] = "foo";
m[y] = "bar";

m[x];							// "bar"
m[y];							// "bar"
```

O que está acontecendo aqui? Os dois objetos `x` and `y` se transformam para a string `"[object Object]"`, de tal forma que apenas uma chave está sendo definida em `m`.

Algumas pessoas implementaram mapas falsos mantendo uma array paralela de chaves que não sejam strings em conjunto com uma array dos seus respectivos valores, por exemplo:

```js
var keys = [], vals = [];

var x = { id: 1 },
	y = { id: 2 };

keys.push( x );
vals.push( "foo" );

keys.push( y );
vals.push( "bar" );

keys[0] === x;					// true
vals[0];						// "foo"

keys[1] === y;					// true
vals[1];						// "bar"
```

É claro que, você não quer ter que gerenciar essas arrays paralelas sozinho, por isso você poderia definir uma estrutura de dados com métodos que irão automaticamente fazer o gerenciamento por debaixo dos panos. Além de ter que fazer este trabalho sozinho, a maior desvantagem é que o acesso não é mais O(1) em complexidade de tempo, mas sim O(n).

Mas a partir do ES6, não existe mais a necessidade de fazer isso! Basta usar `Map(..)`:

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );
m.set( y, "bar" );

m.get( x );						// "foo"
m.get( y );						// "bar"
```

O único empecilho é que você não pode usar a sintaxe de colchetes `[ ]` para definir e acessar valores. Mas `get(..)` e `set(..)` funcionam de forma perfeitamente adequada como uma alternativa.

Para deletar um elemento de um mapa, não use o operador `delete`, mas sim o método `delete(..)`:

```js
m.set( x, "foo" );
m.set( y, "bar" );

m.delete( y );
```

Você pode limpar todo o conteúdo de um mapa com `clear()`. Para pegar o tamanho de um mapa (i.e., o número de chaves), use a propriedade `size` (não `length`):

```js
m.set( x, "foo" );
m.set( y, "bar" );
m.size;							// 2

m.clear();
m.size;							// 0
```

O construtor `Map(..)` também pode receber um iterador (veja "Iteradores" no Capítulo 3), o que deve produzir uma lista de arrays, onde o primeiro item de cada array é a chave e o segundos item é o valor. Esse formado de iteração é idêntico ao produzido pelo método `entries()`, explicado na próxima seção. Isso torna mais fácil fazer a cópia de um mapa:

```js
var m2 = new Map( m.entries() );

// same as:
var m2 = new Map( m );
```

Pelo fato de uma instância de um mapa ser iterável, e seu iterador padrão ser o mesmo que o do `entries()`, a segunda forma mais curta (mostrada acima) é preferível.

É claro que, você pode especificar manualmente uma lista de *entradas* (entries) (array de arrays de chaves/valores) na forma do construtor `Map(..)`:

```js
var x = { id: 1 },
	y = { id: 2 };

var m = new Map( [
	[ x, "foo" ],
	[ y, "bar" ]
] );

m.get( x );						// "foo"
m.get( y );						// "bar"
```

### Valores do Map

Para pegar a lista de valores de um mapa, use `values(..)`, que retorna um iterador. Nos capítulos 2 e 3, nós vimos várias formas de processar um iterador sequencialmente (como uma array), como por exemplo o operador spread `...` e o laço `for..of`. Além disso, "Arrays" no capítulo 6 cobre o método `Array.from(..)` com detalhe. Considere:

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );
m.set( y, "bar" );

var vals = [ ...m.values() ];

vals;							// ["foo","bar"]
Array.from( m.values() );		// ["foo","bar"]
```

Como discutido na seção anterior, você pode iterar sobre as entradas de uma mapa usando `entries()` (ou o iterador padrão do mapa). Considere:

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );
m.set( y, "bar" );

var vals = [ ...m.entries() ];

vals[0][0] === x;				// true
vals[0][1];						// "foo"

vals[1][0] === y;				// true
vals[1][1];						// "bar"
```

### Chaves do Map

Para pegar a lista de chaves, use `keys()`, que retorna um iterador sobre as chaves do mapa:

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );
m.set( y, "bar" );

var keys = [ ...m.keys() ];

keys[0] === x;					// true
keys[1] === y;					// true
```

Para determinar se um mapa possui uma determinada chave, use `has(..)`:

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );

m.has( x );						// true
m.has( y );						// false
```

Mapas essencialmente deixam você associar pedaços extras de informação (o valor) com um objeto (a chave) sem efetivamente ter que colocar a informação no próprio objeto.

Embora você possa usar qualquer tipo de valor como uma chave para seu mapa, você tipicamente irá usar objetos, pois strings e outras primitivas já podem ser usadas como chaves de objetos normais. Em outras palavras, você provavelmente irá querer continuar o uso de objetos normais para mapas a não ser que alguns ou todas as chaves precisam ser objetos, pois nesse caso um mapa é mais apropriado.

**Cuidado:** Se você usar um objeto como uma chave de uma mapa e esse objeto mais tarde for descartado (todas suas referências sumiram) numa tentativa do coletor de lixo (GC - garbage collector) recuperar sua memória, o mapa ainda assim terá sua entrada. Você precisará remover a entrada do mapa para que ela possa ser coletada pelo GC. Na próxima seção, você vai ver WeakMaps como uma melhor opção para chaves de objetos e o coletor de lixo.

## WeakMaps (Mapas Fracos)

WeakMaps são uma variação dos mapas, que possuem a grande maioria do comportamento externo iguais aos dos mapas, mas que se diferenciam por baixo em como a alocação de memória (especificamente o coletor de lixo) funciona.

WeakMaps aceitam (apenas) objetos como chaves. Esses objetos são mantidos *fracamente*, o que significa que se o objeto é passível de ser coletado pelo coletor de lixo, a sua entrada no WeakMap também é removida. Esse não é um comportamento observável, ainda que, a única forma de um objeto ser coletado é se não houver mais nenhuma referência para ele -- uma vez que não haja mais referências para ele, você não tem nenhuma referência ao objeto para checar se ele existe no WeakMap.

De qualquer maneira, a API para o WeakMap é similar, porém mais limitada:

```js
var m = new WeakMap();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );

m.has( x );						// true
m.has( y );						// false
```

WeakMaps não possuem a propriedade `size` ou o método `clear()`, nem expõem seus iteradores sobre as chaves, valores, ou entradas. Então, mesmo que você remova a referência para `x`, o que irá remover sua entrada de `m` pelo coletor de lixo, não haverá nenhuma forma de saber. Você terá que acreditar na palavra de honra do Javascript!

Assim como mapas, WeakMaps deixam você associar levemente informações com um objeto. Mas eles são particularmente úteis se o objeto é um que não está inteiramente sobre seu controle, como elementos do DOM por exemplo. Se o objeto que você está usando como uma chave em seu mapa pode ser deletada e deve ser passível de coleta quando for, então um WeakMap é a opção mais apropriada.

É importante notar que um WeakMap só mantêm suas *chaves* fracamente, não seus valores. Considere:

```js
var m = new WeakMap();

var x = { id: 1 },
	y = { id: 2 },
	z = { id: 3 },
	w = { id: 4 };

m.set( x, y );

x = null;						// { id: 1 } é passível de coleta
y = null;						// { id: 2 } é passível de coleta
								// apenas porque o { id: 1 } é

m.set( z, w );

w = null;						// { id: 4 } não é passível de coleta
```

Por essa razão, WeakMaps em minha opinião poderiam ter um melhor nome: "WeakKeyMaps."

## Sets (Conjuntos)

Um conjunto é uma coleção de valores únicos (duplicados são ignorados).

A API para um conjunto é similar ao do mapa. O método `add(..)` toma o lugar do método `set(..)` (meio ironicamente), e não existe método `get(..)`.

Considere:

```js
var s = new Set();

var x = { id: 1 },
	y = { id: 2 };

s.add( x );
s.add( y );
s.add( x );

s.size;							// 2

s.delete( y );
s.size;							// 1

s.clear();
s.size;							// 0
```

O construtor `Set(..)` tem forma similar ao do `Map(..)`, já que pode receber um iterador, como outro conjunto ou simplesmente um array de valores. Entretanto, diferentemente da maneira que `Map(..)` espera uma lista de *entradas* (array de array de chaves/valores), `Set(..)` espera uma lista de *valores* (array de valores):

```js
var x = { id: 1 },
	y = { id: 2 };

var s = new Set( [x,y] );
```

Um conjunto não precisa de um `get(..)` porque você não recupera um valor de um conjunto, mas sim testa para ver se ele está presente ou não:

```js
var s = new Set();

var x = { id: 1 },
	y = { id: 2 };

s.add( x );

s.has( x );						// true
s.has( y );						// false
```

**Nota:** O algoritmo de comparação do `has(..)` é praticamente idêntico ao do `Object.is(..)` (veja Capítulo 6), com exceção de que `-0` e `0` são tratados como a mesma coisa ao invés de distintos.

### Iteradores do Set

Conjuntos possuem os mesmos métodos de iteradores que os mapas. Seus comportamentos são diferentes para conjuntos, mas simétricos com o comportamento de iteradores de mapa. Considere:

```js
var s = new Set();

var x = { id: 1 },
	y = { id: 2 };

s.add( x ).add( y );

var keys = [ ...s.keys() ],
	vals = [ ...s.values() ],
	entries = [ ...s.entries() ];

keys[0] === x;
keys[1] === y;

vals[0] === x;
vals[1] === y;

entries[0][0] === x;
entries[0][1] === x;
entries[1][0] === y;
entries[1][1] === y;
```

Os iteradores `keys()` e `values()` ambos produzem uma lista com os valores únicos no conjunto. O iterador `entries()` produz uma lista com arrays de entrada, onde ambos os items da array são os valores únicos do conjunto. O iterador padrão de um conjunto é o seu iterador `values()`.

A singularidade inerente de um conjunto é sua característica mais útil. Por exemplo:

```js
var s = new Set( [1,2,3,4,"1",2,4,"5"] ),
	uniques = [ ...s ];

uniques;						// [1,2,3,4,"1","5"]
```

A singularidade do conjunto não suporta coerção, então `1` e `"1"` são considerados valores distintos.

## WeakSets (Conjuntos Fracos)

Enquanto que um WeakMap mantêm suas chaves fracamente (mas seus valores fortemente), um WeakSet mantêm seus valores fracamente (não existem chaves na verdade).

```js
var s = new WeakSet();

var x = { id: 1 },
	y = { id: 2 };

s.add( x );
s.add( y );

x = null;						// `x` é passível de ser coletado
y = null;						// `y` é passível de ser coletado
```

**Cuidado:** Os valores do WeakSet devem ser objetos, valores primitivos não são permitidos por conjuntos.

## Revisão

O ES6 define uma série de coleções úteis que tornam trabalhar com dados de maneiras estruturadas que são mais eficientes e efetivas.

TypedArrays fornecem "view"s de buffers de dados binários que se alinham com vários tipos de inteiros, como inteiros não sinalizados de 8-bits e pontos flutuantes de 32-bits. O acesso à dados binários através de arrays tornam as operações mais fáceis de se expressar e manter, o que lhe permite trabalhar mais facilmente com dados complexos como vídeo, áudio, dados de canvas, e assim por diante.

Mapas são pares de chave-valores onde a chave pode ser um objeto ao invés de apenas uma string/primitiva. Conjuntos são listas de valores únicos (de qualquer tipo).

WeakMaps são mapas onde a chave (objeto) é guardada fracamente, de tal forma que o coletor de lixo é livre para coletar a entrada caso seja a última referência para um objeto. WeakSets são conjuntos onde o valor é fracamente mantido, novamente, para que o coletor de lixo possa remover a entrada caso ela seja a última referência ao objeto.
