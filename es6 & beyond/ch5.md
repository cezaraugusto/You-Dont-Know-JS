# You Don't Know JS: ES6 & Beyond
# Capítulo 5: Coleções

Coleções estruturadas e acesso a informações são componentes críticos de praticamente qualquer programa JS. Desde o começo da linguagem até hoje, os vetores e objetos tem sido nosso principal mecanismo para se criar estruturas de dados. É claro, muitas estruturas de alto-nível foram construídas no topo destas, como bibliotecas que rodam em modo usuário.

A partir do ES6, algumas das abstrações de estruturas de dados mais úteis (e otimizadas para performance!) foram adicionados como componentes nativos da linguagem.

Nós começaremos este capítulo observando primeiramente TypedArrays (Vetores Tipados), tecnicamente contemporânea aos esforços do ES5 já há alguns anos, mas apenas padronizada como acompanhante do WebGL e não do JavaScript. A partir do ES6, eles foram adotadas diretamente pela especificação da linguagem, o que lhes garante o status de primeira-classe.

Mapas (Maps) são como objetos (pares de chave/valor), mas ao invés de apenas uma string para a chave, você pode usar qualquer valor -- até mesmo outro objeto ou mapa! Conjuntos (Sets) são similares à vetores (listas de valores), mas os valores são únicos; caso você adicione um valor duplicado, é ignorado. Também existem as versões fracas (em relação ao coletor de memória/lixo): WeakMap e WeakSet.

## TypedArrays (Vetores Tipados)

Como nós vimos no livro *Tipos & Gramática* desta série, JS tem um conjunto de tipos embutidos (built-in), como `number` e `string`. Seria tentador olhar para uma funcionalidade chamada "vetor tipado" e assumir que isso significa um vetor com valores de um tipo específico, como um vetor que contenha apenas strings.

Entretanto, TypedArrays tem mais a ver com prover acesso estruturado à informações binárias usando uma semântica similar à de vetores (acesso indexado, etc.). O "tipo" no nome se refere a uma "view" que é colocado no tipo do balde de bits, o que é essencialmente um mapeamento se os bits devem ser vistos como uma array de 8-bits inteiros com sinais, 16-bits de inteiros com sinais, e assim por adiante.

Como se constrói esse esse tal balde de bits? Ele é chamado de "buffer," e você o constrói mais diretamente com o construtor `ArrayBuffer(..)`:

```js
var buf = new ArrayBuffer( 32 );
buf.byteLength;							// 32
```

`buf` é agora um buffer binário com tamanho de 32-bytes (256-bits), onde todos os valores são pré-inicializados para `0`. Um buffer por si só não permite qualquer interação, com a exceção de verificar qual é a sua propriedade `byteLength`.

**Dica:** Diversas funcionalidades de plataformas web usam ou retornam vetores de buffer, por exemplo `FileReader#readAsArrayBuffer(..)`, `XMLHttpRequest#send(..)`, e `ImageData` (canvas data).

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

`littleEndian` será `true`ou`false`; para a maioria dos navegadores, deve retornar `true`. Esse teste usa `DataView(..)`, o que permite um nível mais baixo, e um controle mais fino para acessar (definir/pegar) os bits da view que você colocou sobre o buffer. O terceiro parâmetro do método `setInt16(..)` no trecho anterior é para dizer para a `DataView` qual é a extremidade que você quer usar para a operação.

**Cuidado:** Não confunda a extremidade de um armazenamento binário em array de buffers com como um dado número é representado quando exposto em um programa JS. Por exemplo,(3085).toString(2)` retorna`"110000001101"`, o que com uma liderança assumida de quatro `"0"`s parece ser uma representação em big-endian. Na verdade, essa representação é baseado numa única view de 16-bits, não em uma view de dois bytes de 8-bits. O teste do `DataView` é a melhor forma de representar a extremidade do seu ambiente JS.

### Multiple Views

A single buffer can have multiple views attached to it, such as:

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

The typed array constructors have multiple signature variations. We've shown so far only passing them an existing buffer. However, that form also takes two extra parameters: `byteOffset` and `length`. In other words, you can start the typed array view at a location other than `0` and you can make it span less than the full length of the buffer.

If the buffer of binary data includes data in non-uniform size/location, this technique can be quite useful.

For example, consider a binary buffer that has a 2-byte number (aka "word") at the beginning, followed by two 1-byte numbers, followed by a 32-bit floating point number. Here's how you can access that data with multiple views on the same buffer, offsets, and lengths:

```js
var first = new Uint16Array( buf, 0, 2 )[0],
	second = new Uint8Array( buf, 2, 1 )[0],
	third = new Uint8Array( buf, 3, 1 )[0],
	fourth = new Float32Array( buf, 4, 4 )[0];
```

### TypedArray Constructors

In addition to the `(buffer,[offset, [length]])` form examined in the previous section, typed array constructors also support these forms:

* [constructor]`(length)`: Creates a new view over a new buffer of `length` bytes
* [constructor]`(typedArr)`: Creates a new view and buffer, and copies the contents from the `typedArr` view
* [constructor]`(obj)`: Creates a new view and buffer, and iterates over the array-like or object `obj` to copy its contents

The following typed array constructors are available as of ES6:

* `Int8Array` (8-bit signed integers), `Uint8Array` (8-bit unsigned integers)
	- `Uint8ClampedArray` (8-bit unsigned integers, each value clamped on setting to the `0`-`255` range)
* `Int16Array` (16-bit signed integers), `Uint16Array` (16-bit unsigned integers)
* `Int32Array` (32-bit signed integers), `Uint32Array` (32-bit unsigned integers)
* `Float32Array` (32-bit floating point, IEEE-754)
* `Float64Array` (64-bit floating point, IEEE-754)

Instances of typed array constructors are almost the same as regular native arrays. Some differences include having a fixed length and the values all being of the same "type."

However, they share most of the same `prototype` methods. As such, you likely will be able to use them as regular arrays without needing to convert.

For example:

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

**Warning:** You can't use certain `Array.prototype` methods with TypedArrays that don't make sense, such as the mutators (`splice(..)`, `push(..)`, etc.) and `concat(..)`.

Be aware that the elements in TypedArrays really are constrained to the declared bit sizes. If you have a `Uint8Array` and try to assign something larger than an 8-bit value into one of its elements, the value wraps around so as to stay within the bit length.

This could cause problems if you were trying to, for instance, square all the values in a TypedArray. Consider:

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

The `20` and `30` values, when squared, resulted in bit overflow. To get around such a limitation, you can use the `TypedArray#from(..)` function:

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

See the "`Array.from(..)` Static Function" section in Chapter 6 for more information about the `Array.from(..)` that is shared with TypedArrays. Specifically, the "Mapping" section explains the mapping function accepted as its second argument.

One interesting behavior to consider is that TypedArrays have a `sort(..)` method much like regular arrays, but this one defaults to numeric sort comparisons instead of coercing values to strings for lexicographic comparison. For example:

```js
var a = [ 10, 1, 2, ];
a.sort();								// [1,10,2]

var b = new Uint8Array( [ 10, 1, 2 ] );
b.sort();								// [1,2,10]
```

The `TypedArray#sort(..)` takes an optional compare function argument just like `Array#sort(..)`, which works in exactly the same way.

## Maps

If you have a lot of JS experience, you know that objects are the primary mechanism for creating unordered key/value-pair data structures, otherwise known as maps. However, the major drawback with objects-as-maps is the inability to use a non-string value as the key.

For example, consider:

```js
var m = {};

var x = { id: 1 },
	y = { id: 2 };

m[x] = "foo";
m[y] = "bar";

m[x];							// "bar"
m[y];							// "bar"
```

What's going on here? The two objects `x` and `y` both stringify to `"[object Object]"`, so only that one key is being set in `m`.

Some have implemented fake maps by maintaining a parallel array of non-string keys alongside an array of the values, such as:

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

Of course, you wouldn't want to manage those parallel arrays yourself, so you could define a data structure with methods that automatically do the management under the covers. Besides having to do that work yourself, the main drawback is that access is no longer O(1) time-complexity, but instead is O(n).

But as of ES6, there's no longer any need to do this! Just use `Map(..)`:

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );
m.set( y, "bar" );

m.get( x );						// "foo"
m.get( y );						// "bar"
```

The only drawback is that you can't use the `[ ]` bracket access syntax for setting and retrieving values. But `get(..)` and `set(..)` work perfectly suitably instead.

To delete an element from a map, don't use the `delete` operator, but instead use the `delete(..)` method:

```js
m.set( x, "foo" );
m.set( y, "bar" );

m.delete( y );
```

You can clear the entire map's contents with `clear()`. To get the length of a map (i.e., the number of keys), use the `size` property (not `length`):

```js
m.set( x, "foo" );
m.set( y, "bar" );
m.size;							// 2

m.clear();
m.size;							// 0
```

The `Map(..)` constructor can also receive an iterable (see "Iterators" in Chapter 3), which must produce a list of arrays, where the first item in each array is the key and the second item is the value. This format for iteration is identical to that produced by the `entries()` method, explained in the next section. That makes it easy to make a copy of a map:

```js
var m2 = new Map( m.entries() );

// same as:
var m2 = new Map( m );
```

Because a map instance is an iterable, and its default iterator is the same as `entries()`, the second shorter form is more preferable.

Of course, you can just manually specify an *entries* list (array of key/value arrays) in the `Map(..)` constructor form:

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

### Map Values

To get the list of values from a map, use `values(..)`, which returns an iterator. In Chapters 2 and 3, we covered various ways to process an iterator sequentially (like an array), such as the `...` spread operator and the `for..of` loop. Also, "Arrays" in Chapter 6 covers the `Array.from(..)` method in detail. Consider:

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

As discussed in the previous section, you can iterate over a map's entries using `entries()` (or the default map iterator). Consider:

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

### Map Keys

To get the list of keys, use `keys()`, which returns an iterator over the keys in the map:

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

To determine if a map has a given key, use `has(..)`:

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );

m.has( x );						// true
m.has( y );						// false
```

Maps essentially let you associate some extra piece of information (the value) with an object (the key) without actually putting that information on the object itself.

While you can use any kind of value as a key for a map, you typically will use objects, as strings and other primitives are already eligible as keys of normal objects. In other words, you'll probably want to continue to use normal objects for maps unless some or all of the keys need to be objects, in which case map is more appropriate.

**Warning:** If you use an object as a map key and that object is later discarded (all references unset) in attempt to have garbage collection (GC) reclaim its memory, the map itself will still retain its entry. You will need to remove the entry from the map for it to be GC-eligible. In the next section, we'll see WeakMaps as a better option for object keys and GC.

## WeakMaps

WeakMaps are a variation on maps, which has most of the same external behavior but differs underneath in how the memory allocation (specifically its GC) works.

WeakMaps take (only) objects as keys. Those objects are held *weakly*, which means if the object itself is GC'd, the entry in the WeakMap is also removed. This isn't observable behavior, though, as the only way an object can be GC'd is if there's no more references to it -- once there are no more references to it, you have no object reference to check if it exists in the WeakMap.

Otherwise, the API for WeakMap is similar, though more limited:

```js
var m = new WeakMap();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );

m.has( x );						// true
m.has( y );						// false
```

WeakMaps do not have a `size` property or `clear()` method, nor do they expose any iterators over their keys, values, or entries. So even if you unset the `x` reference, which will remove its entry from `m` upon GC, there is no way to tell. You'll just have to take JavaScript's word for it!

Just like Maps, WeakMaps let you soft-associate information with an object. But they are particularly useful if the object is not one you completely control, such as a DOM element. If the object you're using as a map key can be deleted and should be GC-eligible when it is, then a WeakMap is a more appropriate option.

It's important to note that a WeakMap only holds its *keys* weakly, not its values. Consider:

```js
var m = new WeakMap();

var x = { id: 1 },
	y = { id: 2 },
	z = { id: 3 },
	w = { id: 4 };

m.set( x, y );

x = null;						// { id: 1 } is GC-eligible
y = null;						// { id: 2 } is GC-eligible
								// only because { id: 1 } is

m.set( z, w );

w = null;						// { id: 4 } is not GC-eligible
```

For this reason, WeakMaps are in my opinion better named "WeakKeyMaps."

## Sets

A set is a collection of unique values (duplicates are ignored).

The API for a set is similar to map. The `add(..)` method takes the place of the `set(..)` method (somewhat ironically), and there is no `get(..)` method.

Consider:

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

The `Set(..)` constructor form is similar to `Map(..)`, in that it can receive an iterable, like another set or simply an array of values. However, unlike how `Map(..)` expects *entries* list (array of key/value arrays), `Set(..)` expects a *values* list (array of values):

```js
var x = { id: 1 },
	y = { id: 2 };

var s = new Set( [x,y] );
```

A set doesn't need a `get(..)` because you don't retrieve a value from a set, but rather test if it is present or not, using `has(..)`:

```js
var s = new Set();

var x = { id: 1 },
	y = { id: 2 };

s.add( x );

s.has( x );						// true
s.has( y );						// false
```

**Note:** The comparison algorithm in `has(..)` is almost identical to `Object.is(..)` (see Chapter 6), except that `-0` and `0` are treated as the same rather than distinct.

### Set Iterators

Sets have the same iterator methods as maps. Their behavior is different for sets, but symmetric with the behavior of map iterators. Consider:

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

The `keys()` and `values()` iterators both yield a list of the unique values in the set. The `entries()` iterator yields a list of entry arrays, where both items of the array are the unique set value. The default iterator for a set is its `values()` iterator.

The inherent uniqueness of a set is its most useful trait. For example:

```js
var s = new Set( [1,2,3,4,"1",2,4,"5"] ),
	uniques = [ ...s ];

uniques;						// [1,2,3,4,"1","5"]
```

Set uniqueness does not allow coercion, so `1` and `"1"` are considered distinct values.

## WeakSets

Whereas a WeakMap holds its keys weakly (but its values strongly), a WeakSet holds its values weakly (there aren't really keys).

```js
var s = new WeakSet();

var x = { id: 1 },
	y = { id: 2 };

s.add( x );
s.add( y );

x = null;						// `x` is GC-eligible
y = null;						// `y` is GC-eligible
```

**Warning:** WeakSet values must be objects, not primitive values as is allowed with sets.

## Review

ES6 defines a number of useful collections that make working with data in structured ways more efficient and effective.

TypedArrays provide "view"s of binary data buffers that align with various integer types, like 8-bit unsigned integers and 32-bit floats. The array access to binary data makes operations much easier to express and maintain, which enables you to more easily work with complex data like video, audio, canvas data, and so on.

Maps are key-value pairs where the key can be an object instead of just a string/primitive. Sets are unique lists of values (of any type).

WeakMaps are maps where the key (object) is weakly held, so that GC is free to collect the entry if it's the last reference to an object. WeakSets are sets where the value is weakly held, again so that GC can remove the entry if it's the last reference to that object.
