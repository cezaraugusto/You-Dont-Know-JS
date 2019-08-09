# You Don't Know JS: *this* & Prototipagem de Objetos
# Chapter 4: Confundindo Objetos com "Classes"

Continuando com nossa exploração de objetos do capítulo anterior, é natural que agora nós demos atenção a "programação orientada a objetos (OO)", com "classes". Nós primeiro iremos olhar para "orientação à classes" como um padrão de projeto, antes de examinar as mecânicas de "classes": "instanciação", "herança" e "polimorfismo (relativo)".

Nós veremos que esses conceitos não mapeiam muito naturalmente para o mecanismo de objetos no JS e também até onde muitos desenvolvedores JavaScript vão para superar estes desafios (mixins etc.).

**Nota:** Este capítulo gasta um tempo considerável (a primeira metade!) na exploração massiva da teoria por trás da "programação orientada a objetos". Nós eventualmente relacionaremos a um código JavaScript concreto na segunda metade, quando nós falamos sobre "Mixins". Mas há uma grande quantidade de conceito e pseudocódigo para percorrer primeiro, então não se perca -- apenas fique com isso!

## Teoria das Classes

Classe/Herança descreve uma certa forma de organização de código e arquitetura -- uma maneira de modelar problemas de domínio do mundo real em nosso software.

OO ou programação orientada à classes enfatiza que os dados têm um comportamento associado intrinsecamente (claro, diferente dependendo do tipo e natureza do dado!) que opera sobre ele, então o design adequado é empacotar (também conhecido como encapsular) o dado e o comportamento juntos. Isso às vezes é chamado "estruturas de dados" na ciência da computação.

Por exemplo, uma série de caracteres que representa uma palavra ou frase geralmente é chamada de "string". Os caracteres são os dados. Mas você quase nunca se importa apenas com os dados, você geralmente quer *fazer coisas* com os dados, assim os comportamentos que podem se aplicar *para* aquele dado (calcular seu tamanho, anexar dados, busca etc.) são todos concebidos como métodos de uma classe `String`.

Qualquer string é apenas uma instância dessa classe, o que significa que é um pacote cuidadosamente organizado contendo tanto os caracteres quanto as funcionalidades que podemos executar neles.

Classe também insinuam uma maneira de *classificar* uma certa estrutura de dados. A maneira que fazemos isso é assumindo que qualquer estrutura é uma variação específica de uma definição `base` mais ampla.

Vamos explorar esse processo de classificação olhando para um exemplo comumente citado. Um *carro* pode ser descrito como uma implementação específica de uma "classe" de coisas, chamada de *veículo*.

Nós modelamos esse relacionamento em software com classes definindo uma classe `Vehicle` e uma classe `Car`.

A definição de `Vehicle` pode incluir coisas como propulsão (motores etc.), a capacidade de transportar pessoas etc., que seriam todos os comportamentos. O que nós definimos em `Vehicle` é todo o material que é comum a todos (ou a maioria) dos diferentes tipos de veículos (como os "aviões, trens e automóveis").

Pode não fazer sentido no nosso software redefinir a essência básica da "capacidade de transportar pessoas"
repetidas vezes para cada tipo diferente de veículo. Ao invés disso, nós definimos essa capacidade uma vez em `Vehicle`, e então quando definimos `Car`, nós simplesmente indicamos que ele "herda" (ou "estende") sua definição base de `Vehicle`. Diz-se que a definição de `Car` especializa a definição geral de `Vehicle`.

Enquanto `Vehicle` e `Car` definem coletivamente o comportamento por meio de métodos, os dados em uma instância seriam coisas como o "número do chassi" de um carro específico etc.

**E assim, classes, herança e instanciação surgem.**

Outro conceito chave com classes é o "polimorfismo", o qual descreve a ideia que um comportamento genérico de uma classe pai pode ser sobrescrito em uma classe filha para dar mais especificidade. De fato, o polimorfismo relativo nos permite referenciar o comportamento base a partir do comportamento sobrescrito.

A teoria de Classe sugere fortemente que uma classe pai e uma classe filha compartilhem o mesmo nome de método para um determinado comportamento, de modo que esse filho substitui o pai (diferentemente). Como nós veremos mais tarde, fazer isso no seu código JavaScript é optar pela frustração e pela fragilidade do código.

### Padrão de projeto "Classe"

Você pode nunca ter pensando sobre classes como um "padrão de projeto", já que é mais comum ver discussões sobre os populares "Padrões de projeto OO", como "Iterator", "Observer", "Factory", "Singleton" etc. Apresentado dessa maneira, é quase uma suposição de que as classes OO são a mecânica baixo nível pela qual implementamos todos os padrões de projeto (alto nível), como se OO fosse uma base para *todo* o código (apropriado).

Dependendo do seu nível de educação formal em programação, você pode ter ouvido falar sobre "programação procedural" como uma maneira de descrever o código que consiste apenas de procedimentos (também conhecido como funções) chamando outras funções, sem nenhuma maior abstração. Você pode ter sido ensinado que classes eram a maneira *adequada* de transformar o estilo procedural "código espaguete" em um código bem moldado e organizado.

Claro que, se você tem experiência com "programação funcional" (Monads etc.), você sabe muito bem que as classes são apenas um dos vários padrões de projetos comuns. Mas para outras pessoas, esta pode ser a primeira vez que você se pergunta se as classes realmente são uma base fundamental para o código, ou se elas são uma abstração opcional sobre o código.

Algumas linguagens (como Java) não te dão a opção de escolha, então não é muito *opcional* -- tudo é uma classe. Outras linguagens como C/C++ ou PHP fornecem tanto a sintaxe procedural quanto a orientada a classes deixando mais para o desenvolvedor escolher qual estilo ou mistura de estilos é apropriado.

### "Classes" JavaScript

Onde o JavaScript se enquadra nesse aspecto? JS já possuía *alguns* elementos sintáticos baseados em classes (como `new` e `instaceof`) por um tempo, e mais recentemente no ES6, foram feitas algumas adições, como a palavra chave `class` (veja o Apêndice A).

Mas isso significa que atualmente o Javascript *possui* classes? Claro e simples: **Não.**

Como as classes são um design pattern, você *pode*, com um pouco de esforço (como veremos ao longo desse capítulo), implementar aproximações de muitas funcionalidades clássicas das classes. O JS tenta satisfazer o *desejo* extremamente comum de projetar com classes, fornecendo uma sintaxe aparentemente semelhante com a de uma classe.

Embora possamos ter uma sintaxe parecida com as das classes, é como se o mecanismo do Javascript estivesse lutando contra o seu desejo de utilizar o *design pattern de classes*, porque por trás da cortina, os mecanismos que você constrói estão operando de forma bastante diferente. Açúcar sintático e (as amplamente utilizadas) bibliotecas "Class" JS percorrem um longo caminho para esconder essa realidade de você, porém mais cedo ou mais tarde você vai enfrentar o fato de que classes que você tem em outras linguagens não são como as "classes" do Javascript.

Isso resume que as classes são um pattern opcional no design de um software, e você tem a escolha de usá-los no Javascript ou não. Como muitos desenvolvedores possuem uma grande afinidade com design de software orientado a classes, passaremos o restante deste capítulo explorando o que é necessário para manter a ilusão de que o JS provem classes, e os pontos problemáticos que teremos.

## Mecânica das Classes

Na maioria das linguagens orientadas a classes, a "biblioteca padrão" provê uma "stack" de estruturas de dados (push, pop, etc) como uma classe `Stack`.
Essa classe teria um conjunto interno de variáveis que armazena dados e um conjunto de comportamentos publicamente acessíveis ("métodos") fornecidos pela classe, o que dá ao seu código a capacidade de interagir (adicionando e removendo, etc.) com os dados (ocultos).

Porém em tais linguagens você não precisa trabalhar diretamente com essa classe `Stack` (a menos que esteja fazendo uma referência a um membro de uma classe do tipo **Static**, o que está fora do escopo da nossa discussão). A classe `Stack` é apenas uma explicação abstrata do que qualquer objeto do tipo "stack" pode fazer, mas não é em si uma "stack". Você deve **instanciar** a classe `Stack` antes de ter uma estrutura de dados concreta onde possa operar.

### Construção

O pensamento metafórico tradicional baseado em "classe" e "instância" vêm da construção civil.

Uma arquiteta planeja todos as características de um edifício: quão largo, quão alto, janelas e em quais locais, até mesmo o tipo de material que será usado no teto e nas paredes. Até esse ponto, ela não necessáriamente se importa, onde o prédio será construido, nem se importa em quantas cópias do edifício serão construídas.

Ela também não se importa muito com o conteúdo do edifício -- o mobiliário, o papel de parede, ventiladores de teto, etc. - somente o tipo de estrutura que serão contidos.

Os projetos arquitetônicos que ela produz são apenas *planos* para um edifício. Eles não constituem realmente um prédio onde podemos caminhar e sentar. Nós precisamos de um construtor para essa tarefa. Um construtor irá pegar esses planos e os seguir, minuciosamente, enquanto *constrói* o prédio. Em um sentido mais real, ele está *copiando* as características dos planos para o prédio físico.

Uma vez concluído, o prédio é uma instanciação física do projeto arquitetônico, com sorte um *cópia* perfeita. E então o construtor pode se mover para o lote aberto ao lado e fazer tudo novamente, criando outra *cópia*.

O relacionamento entre o prédio e o plano arquitetônico é indireto. Você pode examinar um projeto arquitetônico para entender como o prédio foi estruturado, para todas as partes em que a inspeção direta do edifício em si fosse insuficiente. Mas se você quer abrir uma porta, você tem que ir para o prédio em si -- o projeto tem apenas linhas desenhadas em uma página *representando* onde a porta deveria estar.

Uma classe é um projeto arquitetônico. Para realmente *conseguir* um objeto com o qual podemos interagir, nós devemos construir (também conhecido como "instanciar") algo da classe. O resultado final dessa "construção" é um objeto, tipicamente chamado de "instância", no qual podemos chamar métodos diretamente e acessar quaisquer propriedades de dados públicos, conforme o necessário.

**O objeto é uma *cópia*** de todas as características descritas pela classe.

Você provavelmente não esperaria entrar em um prédio e encontrar, emoldurado na parede, uma cópia do projeto arquitetônico usado para planejar o prédio, embora projetos arquitetônicos provavelmente estejam arquivadas em um escritório de registros públicos. Similarmente, você geralmente não usa uma instância de objeto para diretamente acessar e manipular a classe, mas é geralmente possível ao menos determinar *de qual classe* um determinado objeto vem.

É mais útil considerar a relação direta entre a classe e uma instância de objeto, ao invés do relacionamento indireto entre uma instância de objeto e a classe do qual ela vem. **Uma classe é instanciada em um objeto por uma operação de cópia.**

<img src="fig1.png">

Como você pode ver, as setas se movem da esquerda para direita, e de cima para baixo, o que indica as operações de cópia que ocorrem tanto conceitualmente quanto fisicamente.

### Construtores

Instâncias das classes são construídas por um método especial da classe, que geralmente possui o mesmo nome da classe, chamado de *construtor*. O trabalho desse método é inicializar qualquer informação (estado) que a instância irá precisar.

Por exemplo, considere esse pseudo-código solto (sintaxe inventada) para as classes:

```js
class CoolGuy {
	specialTrick = nothing

	CoolGuy( trick ) {
		specialTrick = trick
	}

	showOff() {
		output( "Here's my trick: ", specialTrick )
	}
}
```

Para *criar* uma instância de `CoolGuy`, nós vamos chamar o construtor da classe:

```js
Joe = new CoolGuy( "jumping rope" )

Joe.showOff() // Here's my trick: jumping rope
```

Perceba que a classe `CoolGuy` tem um construtor `CoolGuy()`, que é o que chamamos quando dizemos `new CoolGuy(..)`. Nós temos como retorno um objeto (uma instância da nossa classe) do construtor, e nós podemos chamar o método `showOff()`, que vai mostrar o truque especial daquele determinado `CoolGuy`

*Obviamente, pular corda torna Joe um cara muito legal*

O construtor de uma classe *pertence* a classe, quase universalmente com o mesmo nome da classe. Além disso, construtores quase sempre precisam ser chamados com o operador `new` para que o motor da linguagem saiba que você quer construir uma nova instância da classe.

## Heranças de classe

Em linguagens orientadas a classes, você pode definir não somente qual classe pode ser instânciada, como também pode definir que outra classe que herda da primeira classe.

A segunda classe é também chamada de classe-filha enquanto que a primeira é chamada de classe-pai. Esses termos se originam, obviamente, de uma metáfora entre pais e filhos, embora as metáforas aqui estejam empregadas em um conceito mais amplo, como você verá em breve.

Quando um pai tem um filho/filha biológico, as características genéticas do pai são copiadas no filho. Obviamente, na maioria dos sistemas de reprodução biológicos, há dois pais que contribuem igualmente para a variabilidade genética. Mas por efeitos da metáfora, vamos considerar somente um pai.

A partir do momento que o filho existe, ele ou ela é separado do pai. O filho foi fortemente influenciado pela herança genética do seu pai, mas é único e distinto. Se uma criança tem cabelo ruivo isso não necessariamente significa que algum de seus pais tenha cabelo vermelho.

De maneira similar, uma vez que uma classe filha é definida, ela é separada e distinta da classe pai. A classe filha possui uma cópia inicial do comportamento do pai, mas elas podem sobrescrever qualquer comportamento herdado e até mesmo definir um novo comportamento.

É importante relembrar que estamos falando sobre **classes** pais e filhos, que não são coisas físicas. E é isso o que torna a metáfora de pais e filhos um tanto quanto confusas, por que o correto seria dizer que uma classe pai é na verdade o DNA de um pai, e uma classe filho seria como o DNA de um filho. Nós podemos criar (também conhecido como "instanciar") uma pessoa de cada conjunto de DNA para realmente ter uma pessoa com a qual podemos conversar.

Vamos colocar de o conceito biológico de pais e filhos, e olhar a herança através de uma lente ligeramente diferente: diferentes tipos de veículos. Essa é uma das metáforas mais canônicas (e muitas vezes dignas) para entender herança.

Vamos revisitar a discussão sobre `Vehicle` (veículo) e `Car` (carro) que tivemos anteriormente nesse capítulo. Considere esse pseudo-código solto (sintaxe inventada) para classes herdadas:

```js
class Vehicle {
	engines = 1

	ignition() {
		output( "Turning on my engine." )
	}

	drive() {
		ignition()
		output( "Steering and moving forward!" )
	}
}

class Car inherits Vehicle {
	wheels = 4

	drive() {
		inherited:drive()
		output( "Rolling on all ", wheels, " wheels!" )
	}
}

class SpeedBoat inherits Vehicle {
	engines = 2

	ignition() {
		output( "Turning on my ", engines, " engines." )
	}

	pilot() {
		inherited:drive()
		output( "Speeding through the water with ease!" )
	}
}
```

**Nota:** Para clareza e abreviação, os construtores dessas classes foram omitidos

Nós definimos a classe `Veiculo` para que ela assumisse um motor, a forma de ligar a ignição e uma maneira de dirigir. Mas você não fabricaria um veículo genérico, então até esse ponto ela é um conceito abstrato.

Então, definimos dosi tipos específicos de veículos: `Carro` e `SpeedBoat`. Ambos herdam as características gerais de `Veiculo`, mas especializam características apropriadamente para cada tipo. Um carro precisa de 4 rodas, e um SpeedBoat de dois motores, o que significa que é preciso atenção extra para ligar a ignição de ambos motores.

### Polimorfismo

`Car` define seu próprio método `drive()`, que sobrescreve o método de mesmo nome herdado da classe `Veiculo`. Mas então o método `drive()` pertencente a `Car` chama `inherited:drive()`, o que indica que o carro pode referenciar o método original `drive()` pré-sobrescrito herdado. O método `pilot()` do SpeedBoat também faz referência à sua cópia herdada do `drive()`.

Essa técnica é chamada de "polimorfismo", ou "polimorfismo virtual". Mais especificamente para nosso ponto atual, vamos chamar isso de "polimorfismo relativo".

Polimorfismo é um tópico muito mais abrangente do que vamos abordar aqui, mas nossa atual semântica "relativa" refere-se a um aspecto em particular: a ideia de que qualquer método pode referenciar outro método (com o mesmo nome, ou um nome diferente) em um nível mais alto da hierarquia de herança. Nós chamamos de "relativo" porque não estabelecemos absolutamente qual nível de herança (também conhecido como classe) queremos acessar, mas referencia-lo relativamente basicamente dizendo "procure um nível acima".

Em muitas linguagens, a palavra chave `super` é usada, no lugar de `inherited` desse exemplo, apoiando-se na ideia de que uma "superclasse" é o pai/ancestral da classe atual.

Outro aspecto do polimorfismo é que um nome específico de método pode ter mútiplas definições em diferentes níveis da cadeia de herança, e essas definiçõessão selecionadas de forma automática de acordo com os métodos que estão sendo chamados.

Nós vemos duas ocorrências desse comportamento em nosso exemplo acima: `drive()` é definido tanto em `Veiculo` como em `Car`, e `ignition()` é definido tanto em `Veiculo` como em `SpeedBoat`.

**Nota:** Outra coisa que linguagens orientadas a classes fornecem a você através do `super()` é uma forma do construtor da classe filho referenciar diretamente o construtor de sua classe pai. Isso é verdade principalmente porque, com classes reais, o construtor pertence a classe. No entanto em JS é o contrário - é mais apropriado pensar que a "classe" pertence ao construtor (as referências `Foo.prototype`). Já que em JS o relacionamento entre pai e filho existe apenas entre os dois objetos `.prototype` dos respectivos construtores, os próprios construtores não estão diretamente relacionados, e portanto, não há uma forma de referenciar um do outro (consulte o apêndice A para ver como `class` resolve isso com `super`).

Uma implicação interessante do polimorfismo pode ser visto especificamente com `ignition()`. Dentro de `pilot()`, uma referência polimórfica é feita para (a herdada) versão `drive()` de `Veiculo`. Mas esse método `drive()` faz referência ao método `ignition()` apenas pelo nome (sem termos referência relativa).

Qual versão de `ignition()` o motor da linguagem irá usar?, a de `Veiculo` ou a de `SpeedBoat`? **Ela usa a versão de `SpeedBoat` de `ignition()`.** Se você *fosse* instanciar a clase `Veiculo`, e chamar o seu método `drive()`, o motro da linguagem usaria apenas a definição do método `ignition()` pertencete a `Veiculo`.

Dito de outra forma, a definição para o método `ignition()` polimorfa (muda) de acordo com a classe (nível de herança) que a sua instância está referenciando.

Isso pode parecer um detalhe acadêmico muito profundo. Mas entender esses detalhes é necessário para diferenciar adequadamente comportamentos semelhantes (porém diferentes) no mecanismo `[[Prototype]]` do Javascript.

Quando classes são herdadas, existe uma maneira de as próprias classes (não as instâncias de objetos criadas a partir delas!) referirem-se relativamente à classe herdada, e essa referência relativa é geralmente chamada de `super()`.

Lembre se dessa figura anterior:

<img src="fig1.png">

Note como todas as instanciações (`a1`, `a2`, `b1` e `b2`) e a herança (`Bar`) indicam operações de cópia.

Conceitualmente, parece que uma classe filho `Bar` pode acessar o comportamento em sua classe pai `Foo` usando uma referência polimórfica relativa (também conhecida como `super`). No entanto, na realidade, a classe filho recebe meramente uma cópia do comportamento herdado da classe pai. Se o filho "sobrescrever" um método que ele herdou, as versões originais e sobrescritas do método são mantidas, para que ambas sejam acessíveis.

Não deixe o polimorfismo confundir você em pensar que uma classe filho é ligada a uma classe pai. Uma classe filho recebe uma cópia do que precisa da classe pai. **Herança de classes significa cópias.**

### Herança múltipla

Se lembra da nossa conversa sobre pais, filhos e DNA? Nós dizemos que a metáfora era um pouco estranha porque biologicamente a maioria dos descendentes vêm de dois pais. Se uma classe pudesse herdar de duas outras classes distintas, haveria uma maior aproximação com a metáfora de pais e filhos.

Algumas linguagens orientadas a classes permitem que você especifique mais de uma classe "pai" para "herdar". Herança múltipla significa que cada definição das classes pai foram copiadas para a classe filha.

Superficialmente, isso parece um adicional poderoso para a orientação a classes, nos dando a possibilidade de compor mais funcionalidades juntos. No entanto, há certamente algumas questões complicadas que se levantam. Se ambas as classes pai possuem um método chamado `drive()`, qual versão de `drive()` será referenciada pela classe filha? Você sempre teria que manualmente especificar qual o método pai `drive()` você quis dizer, perdendo um pouco da graça da herança polimórfica?

Há uma outra variação, o chamado "Problema do Diamante", que se refere a um cenário onde uma classe filha "D" herda de outras duas classes pai ("B" e "C"), e essas classes, por sua vez, herdam de um pai "A" comum. Se "A" provem um método `drive()`, e "B" e "C" sobrescrevem (polimorficamente) esse método, quando `D` referencia `drive()`, qual versão ele irá usar(`B:drive()` ou `C:drive()`)?

<img src="fig2.png">

Essas complicações são ainda mais profundas do que essa rápida análise. Nós as expomos aqui apenas para que possamos contrastar com o funcionamento dos mecanismos Javascript.

Javascript é mais simples: ele não fornece um mecanismo nativo para "herança múltipla". Muitos vêem isso como uma coisa boa, porque a economia de complexidade é mais compensatória do que a funcionalidade "reduzida". Mas isso não impede os programadores de fingirem fazer isso de várias formas diferentes, como vamos ver a seguir.

## Mixins

O mecânismo objeto do Javascript não executa *automaticamente* o comportamento de uma cópia quando você "herda" ou "instancia". Obviamente, não há classes no Javascript que serão instanciadas, apenas objetos. E objetos não são cópiados para outros objetos, eles são *linkados juntos* (mais sobre isso no capítulo 5).

Como comportamentos de classes observados em outras linguagens implicam em cópias, vamos examinar como desenvolvedores JS **falsificam** o comportamento de cópia *ausente* no mecânismo de classes JavaScript: Mixins. Vamos ver dois tipos de "mixin": **explicito** e **implicito**.

### Mixin Explicito

Vamos revisitar nosso exemplo anterior sobre `Vehicle` e `Car`. Como o JavaScript não irá copiar automaticamente o comportamento de `Vehicle` para `Car`, podemos criar um utilitário que copie manualmente. Tal utilidade é frequentemente chamada de `extend(..)` por muitas bibliotecas/frameworks, mas aqui vamos chamar isso de `mixin(..)` por motivos ilustrativos.

```js
// Exemplo bem simplificado de`mixin(..)`:
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		// só copie se ainda não estiver presente
		if (!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}

	return targetObj;
}

var Vehicle = {
	engines: 1,

	ignition: function() {
		console.log( "Turning on my engine." );
	},

	drive: function() {
		this.ignition();
		console.log( "Steering and moving forward!" );
	}
};

var Car = mixin( Vehicle, {
	wheels: 4,

	drive: function() {
		Vehicle.drive.call( this );
		console.log( "Rolling on all " + this.wheels + " wheels!" );
	}
} );
```

**Nota:** De maneira sutil, mas importante, nós não estamos mais lidando com classes, porque não há classes no JavaScript. `Vehicle` e `Car` são apenas objetos dos quais fazemos cópias de e para, respectivamente.

`Car` agora tem um cópia das propriedades e funções estabelecidas em `Vehicle`. Tecnicamente, as funções não são realmente duplicadas, mas as referências a elas são copiadas. Então `Car` possui agora uma propriedade chamada `ignition`, que é uma referência copiada para a função `ignition()`, assim como uma propriedade chamada `engines` com o valor copiado de `1` de `Vehicle`.

`Car` já possui uma propriedade (função) `drive()`, de modo que a referência de propriedade não foi substituida (consulte a instução if no mixin(..) acima).

#### "Polimorfismo" revisado

Vamos examinar essa instrução: `Vehicle.drive.call(this)`. Isso é o que eu chamo de "pseudo-polimorfismo explicito". Lembre-se que no nosso pseudo-código anterior, essa linha era `inherited:drive()`, que nós chamamos de "polimorfismo relativo".

O JavaScript não tem (antes do ES6; veja apêndice A) um utilitário para o polimorfismo relativo. Então, **já que `Car` e `Vehicle` tinahm uma função com o mesmo nome: `drive()`**, para distinguir uma chamada de um ou outra, temos de fazer uma referência absoluta (não relativa). Nós explicitamente especificamos o objeto `Vehicle` pelo nome, e chamamos a função `drive()` nele.

Mas se dissermos `Vehicle.drive`, o binding do `this` para essa função será o objeto `Vehicle` em vez do objeto `Car` (veja o capítulo 2), o que não é o que queremos. Então, em vez disso usamos `.call( this )`(capítulo 2) para garantir que `drive()` seja executado no contexto do objeto `Car`.

**Nota:** Se o identificador da função para `Car.drive()` não tinha se sobreposto com (também conhecido como "sombreado"; veja capítulo 5) `Vehicle.drive()`, não teríamos exercido o "polimorfismo de método". Então, uma referência para `Vehicle.drive()` teria sido copiada pela chamada `mixin(..)`, e nós poderiamos tê-la acessado diretamente através de `this.drive()`. O identificador escolhido se sobrepondo ao sombreamento é o porque de termos de usar uma abordagem mais complexa de *pseudo-polimorfismo explicito*.

Em linguagens orientadas a classes, que possuem polimorfismo relativo, a linkagem entre `Car` e `Vehicle` é estabelecida uma vez, no topo da definição da classe, o que faz apenas um lugar para manter esses tais relacionamentos.

Mas por causa das peculiaridades do JavaScript, o pseudo-polimorfismo explicito (por causa do sombreamento!) cria vinculação manual/explicita frágil **em cada função onde você precisa de uma referência (pseudo) polimórfica.** Isso pode aumentar muito o custo de manutenção. Além disso, enquanto o pseudo-polimorfismo explícito pode emular o comportamento de "herança múltipla", ele apenas aumenta a complexidade e fragilidade.

O resultado dessas abordagens é normalmente um código mais complexo, difícil de ler, *e* difícil de manter. **Pseudo polimorfismo explicito deve ser evitado sempre que possivel**, porque o custo supera o benefício na maioria dos aspectos.

#### Mesclando cópias

Relembre o utilitário `mixin(..)` acima:

```js
// Exemplo bem simplificado de`mixin(..)`:
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		// só copie se ainda não estiver presente
		if (!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}

	return targetObj;
}
```

Agora, vamos examinar como `mixin(..)` funciona. Ele itera sobre as propriedades de `sourceObj` (`Vehicle` em nosso exemplo) e se não tiver uma propriedade compatível em `targetObj` (`Car` em nosso exemplo) ele cria uma cópia. Como estamos fazendo uma cópia depois que o objeto inicial existe, tomamos o cuidado de não copiar uma propriedade de destino.

Se fazermos as cópias primeiro, antes de especificar o conteúdo específico para `Car`, poderiamos omitir essa verificação contra o `targetObj`, mas isso é um pouco mais desajeitado e menos eficiente, então geralmente é menos preferido:

```js
// alternate mixin, less "safe" to overwrites
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		targetObj[key] = sourceObj[key];
	}

	return targetObj;
}

var Vehicle = {
	// ...
};

// first, create an empty object with
// Vehicle's stuff copied in
var Car = mixin( Vehicle, { } );

// now copy the intended contents into Car
mixin( {
	wheels: 4,

	drive: function() {
		// ...
	}
}, Car );
```

Em qualquer abordagem, copiamos explicitamente o conteúdo não sobreposto do `Vehicle` no `Car`. O nome "mixin" vem de uma maneira alternativa de explicar a tarefa: O `Car` tem o conteúdo de `Vehicle` misturado a ele, assim como você mistura pedaços de chocolate em sua massa de biscoito favorita.

Como resultado da operação, `Car` irá operar de forma pouco separada de `Vehicle`, e vice versa.

**Nota:** Alguns pequenos detalhes foram desdobrados por aqui. Ainda existem algumas maneiras sutis que os dois objetos podem "afetar" uns aos outros, mesmo após a cópia, como se ambos compartilhassem uma referência para um mesmo objeto (como um array).

Como os dois objetos também compartilham referências para as suas funções comuns, isso significa que **mesmo copias manuais de funções (também conhecidos como mixins) de um objeto para outro *não emulam* a duplicação real de classe para a instância que ocorre em linguagens orientadas a classes**.

Funções JavaScript não podem ser duplicadas (de uma maneira padrão e confiável), então, o que você acaba fazendo é ma referência duplicada ao mesmo objeto de função compartilhada (funções são objetos; veja capítulo 3). Se você modificar uma das **funções objetos** compartilhadas (como `ignition()`) adicionando propriedades sobre elas, por exemplo, tanto `Vehicle` quanto `Car` seriam "afetados" por meio da referência compartilhada.

Mixins explícitos são um ótimo mecânismo em JavaScript. Mas eles parecem ser mais fortes que realmente são. Poucos benefícios são realmente derivados da cópia de uma propriedade para outra, ao invés de apenas definir as propriedades duas vezes, uma vez em cada objeto. E isso é especialmente verdadeiro, dada a nuance de referência de objeto de função que acabamos de citar.

Se você *mixar* dois ou mais objetos dentro de um objeto alvo, você pode **emular parcialmente** o comportamento de "herança multipla", mas não há uma forma direta de lidarcom colisões se o mesmo método ou propriedade está sendo copiado de mais de uma origem. Alguns desenvolvedores e bibliotecas surgiram com técnicas de "binding tardio" e outras abordagens exóticas, mas fundamentalmente esses "truques" são *geralmente* mais trabalhosos (e têm desempenho menor) do que o resultado.

Tome cuidado de apenas usar mixins explícitos onde ele realmente ajuda a tornar o código mais legível, e evite esse padrão sempre que você achar que ele está tornando o código mais difícil de rastrear, ou se achar que ele cria dependencias desnecessáriasou difíceis de manipular entre objetos.

**Se começar a ficar mais difícil usar mixins do que antes, você provavelmente deveria para de usar mixins.** De fato, se você tem que usar uma bilioteca/utilitário complexo para trabalhar todos esses detalhes, pode ser um sinal que você está seguindo pelo caminho mais difícil, talvez desnecessariamente. No capítulo 6, tentaremso destilar uma maneira mais simples que realize os resultados desejados sem toda confusão.

#### Herança Parasita

Uma variação desse padrão de mixin explícito, que é de algumas formas explícita e de outras formas implícita, é chamada de "herança parasita", popularizado principalmente por Douglas Crockford.

Aqui está como isso pode funcionar:

```js
// "Classe Tradicional JS" `Vehicle`
function Vehicle() {
	this.engines = 1;
}
Vehicle.prototype.ignition = function() {
	console.log( "Turning on my engine." );
};
Vehicle.prototype.drive = function() {
	this.ignition();
	console.log( "Steering and moving forward!" );
};

// "Classe Parasita" `Car`
function Car() {
	// primeiramente, `car` é um `Vehicle`
	var car = new Vehicle();

	// agora, vamos modificar nosso `car` para especificá-lo
	car.wheels = 4;

	// salve uma referência especial para `Vehicle::drive()`
	var vehDrive = car.drive;

	// substitua `Vehicle::drive()`
	car.drive = function() {
		vehDrive.call( this );
		console.log( "Rolling on all " + this.wheels + " wheels!" );
	};

	return car;
}

var myCar = new Car();

myCar.drive();
// Turning on my engine.
// Steering and moving forward!
// Rolling on all 4 wheels!
```

Como você pode ver, nós inicialmente fizemos uma cópia da definição da "classe pai" `Vehicle` (objeto), então misturamos nossa definição da "classe filha" (objeto) (preservando as referências especiais da classe pai conforme necessário), e passamos esse objeto composto `car` como nossa instância filha.

**Nota:** quando nós chamamos `new Car()`, um novo objeto é criado e especificado pela referência `this` de `Car` (veja o Capítulo 2). Mas uma vez que nós não usamos aquele objeto, e ao invés disso retornamos nosso próprio objeto `car`, o objeto criado inicialmente é descartado. Então, `Car()` poderia ser chamado sem a palavra-chave `new`, e o funcionamento acima seria idêntico, mas sem o desperdício da criação e garbage-collection do objeto.

### Mixins Implícitos

Mixins implícitos estão intimamente relacionados com *pseudo-polimorfismo explícito* como explicado anteriormente. Sendo assim, eles vêm com as mesmas advertências e alertas.

Considere este código:

```js
var Something = {
	cool: function() {
		this.greeting = "Hello World";
		this.count = this.count ? this.count + 1 : 1;
	}
};

Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1

var Another = {
	cool: function() {
		// mixin implícito de `Something` para `Another`
		Something.cool.call( this );
	}
};

Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1 (não compartilha estado com `Something`)
```

Com `Something.cool.call( this )`, que pode acontecer tanto numa chamada de "constructor" (mais comum), ou em uma chamada de método (mostrado aqui), nós essencialmente "emprestamos" a função `Something.cool()` e a chamamos no contexto de `Another` (via o vínculo `this`; veja o Capítulo 2) no lugar de `Something`. O resultado final é que as atribuições que `Something.cool()` faz são aplicadas no objeto `Another` ao invés do objeto `Something`.

Então, é dito que nós "misturamos" o comportamento de `Something` com (ou em) `Another`.

Enquanto esse tipo de técnica parece tirar proveito da funcionalidade de re-ligação do `this`, é na frágil chamada `Something.cool.call( this )`, que não pode ser transformada em uma referência relativa (e então, mais flexível), que você deve **prestar atenção com cuidado**. Geralmente, **evite tais construções sempre que possível** para manter um código mais limpo e mais sustentável.

## Review (TL;DR)

Classes são um design pattern. Muitas linguagens disponibilizam una sintaxe que permite um design de software orientado a classes natural. JS também tem uma sintaxe semelhante, mas ela funciona **muito diferente** do que você está acostumado com as classes nessas outras linguagens.

**Classes significa cópias**

Quando classes tradicionais são instanciadas, ocorre uma cópia do comportamento da classe para a instância. Quando uma classe é herdada, também ocorre uma cópia do comportamento de pai para filho.

Polimorfismo (diferentes funções em múltiplos níveis dentro de uma cadeia de herança com o mesmo nome) pode parecer que implica um link de referência relativa do filho de volta para o pai, mas isso ainda é o resultado da cópia do comportamento.

JavaScript **não** cria cópias entre objetos (como classes fazem) **automaticamente**.

O padrão de mixin (ambos, explícito e implícito) é geralmente usado como *uma forma* de emular o comportamento de cópia de classes, mas isso geralmente leva a sintaxes feias e frágeis como o pseudo-polimorfismo explícito (`OtherObj.methodName.call(this, ...)`), que resultam em códigos difíceis de entender e manter.

Mixins explícitos também não são exatamente iguais a uma *cópia* de classe, uma vez que objetos (e funções!) só terão compartilhado referências duplicadas, e não os objetos/funções duplicados. Não prestar atenção a tais detalhes é a fonte de uma variedade de armadilhas.

Em geral, imitar classes em JS geralmente criam mais minas terrestres para o código futuro do que soluções de problemas atuais *reais*.

