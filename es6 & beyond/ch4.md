# You Don't Know JS: ES6 e além
# Capítulo 4: Controle de fluxo assíncrono

Se você já escreveu uma quantidade significativa de JavaScript, não é segredo que programação assíncrona é uma habilidade necessária. Até o momento, o mecanismo principal para gerenciar assincronicidade tem sido o uso de funções *callback*.

No entanto, o ES6 adiciona um novo recurso que ajuda a resolver as deficiências significativas de assincronicidade com *callbacks*: *Promises*. Ainda, podemos revisar o capítulo anterior sobre `generators` e ver um padrão que combina os dois que é melhoria ao programar controles de fluxo assíncrono no JavaScript.

## Promises

Vamos esclarecer alguns equívocos: *Promises* não substituem *callbacks*. *Promises* provêm uma intermediação confiável -- isso é, entre a execução do seu código e o código assíncrono que processará a tarefa -- para gerenciar *callbacks*.

Outra forma de pensar a respeito de uma *promise* é como sendo um *event listener*, onde você registra um evento que te informa quando uma tarefa foi concluída. É um evento que será disparado apenas uma vez, mas mesmo assim podemos imaginá-lo como se fosse um evento.

*Promises* podem ser encadeadas em conjunto, as quais podem sequenciar uma série de passos que completam-se assincronamente. Juntas com uma abstração de alto nível como o método `all(..)` (em termo clássico, uma porta) e o método `race(..)` (em um termo clássico, um cadeado), o encadeamento de uma *promise* proporciona um mecanismo para controle de fluxo assíncrono.

Ainda, mais uma forma de se entender uma *promise*, é que ela tem um valor futuro, um recipiente em torno de um valor que é independente do tempo envolvido. Esse recipiente pode ser subentendido de forma idêntica, independentemente se o seu valor é ou não o final, observando-se que a resolução de uma *promise* extrai este valor uma vez que ele estiver disponível. Em outras palavras, uma *promise* é dita como a versão assíncrona de valores retornados por funções síncronas.

Uma *promise* pode possuir apenas uma, das duas possíveis resoluções: concluída ou rejeitada, com um valor único opcional. Se uma *promise* for cumprida, o valor final é chamado de cumprimento. Se for rejeitada, o valor final é denominado de motivo (como sendo, o "motivo da rejeição"). *Promises* podem apenas ser resolvidas (cumpridas ou rejeitadas) uma única vez. Qualquer tentativa futura de cumpri-las ou rejeitá-las são simplesmente ignoradas. Assim, uma vez que uma *promise* for resolvida, seu valor se torna imutável, e não pode ser alterado.

Claramente, há muitas maneiras diferentes de imaginar o que uma *promise* é. Nenhuma das perspectivas é completamente suficiente, mas cada uma provê um aspecto diferente do todo. Podemos concluir que *Promises* oferecem melhorias significantes a respeito da utilização de *callbacks* para código assíncrono, visto que elas promovem ordem, previsibilidade e confiabilidade.

### Fazendo e Usando Promises

Para instanciar uma *promise*, use seu construtor `Promise(..)`:

```js
var p = new Promise( function pr(resolve,reject){
  // ..
} );
```

O construtor `Promise(..)` utiliza uma função (`pr(..)`), que é chamada imediatamente e recebe duas funções de controle como argumentos, normalmente chamadas de `resolve(..)` e `reject(..)`. Elas são usadas assim:

* Se você invocar `reject(..)`, a *promise* é rejeitada, e se algum valor for passado para `reject(..)`, ele será considerado como o motivo da rejeição.
* Se você invocar `resolve(..)` sem passar um valor, ou algum valor que não seja uma *promise*, a *promise* será cumprida.
* Se você invocar `resolve(..)` e passar uma outra *promise*, a primeira *promise* simplesmente adota o estado -- sendo imediato ou eventual -- da segunda *promise* informada (sendo ela cumprida ou rejeitada).

Aqui está como você normalmente utiliza uma *promise* para refatorar uma chamada de função *callback*. Vamos iniciar com uma função utilitária `ajax(..)` que espera ser capaz de primeiramente tratar algum erro no *callback*:

```js
function ajax(url,cb) {
  // faz uma requisição, ao final invoca `cb(..)`
}

// ..

ajax( "http://some.url.1", function handler(err,contents){
  if (err) {
    // manipula o erro no retorno do ajax
  }
  else {
    // manipula o `conteúdo` no sucesso
  }
} );
```

Você pode converter para:

```js
function ajax(url) {
  return new Promise( function pr(resolve,reject){
    // faz a requisição, no final invoca
    // `resolve(..)` ou `reject(..)`
  } );
}

// ..

ajax( "http://some.url.1" )
.then(
  function fulfilled(contents){
    // manipula o `conteúdo` no sucesso
  },
  function rejected(reason){
    // manipula a razão do erro da requisição
  }
);
```

*Promises* possuem um método chamado `then(..)` que aceita uma ou duas funções como *callback*. A primeira função (se presente) é tratada como o manipulador de uma *promise* cumprida com sucesso. A segunda função (se presente) é tratada como o manipulador se a *promise* for explicitamente rejeitada, ou se algum erro ou exceção ocorrer durante a resolução.

Se um dos argumentos for omitido ou se não for uma função válida -- normalmente você usará `null` em vez disso -- um valor padrão equivalente é usado. O *callback* padrão de sucesso passa seu valor de cumprimento adiante e a função *callback* padrão de erro propaga seu valor de rejeição adiante.

O atalho para chamar o método `then(null,handleRejection)` é `catch(handleRejection)`.

Ambos `then(..)` e `catch(..)` automaticamente constroem e retornam uma instância de outra *promise*, que está preparada para receber a resolução de qualquer que seja o valor de retorno da *promise* original, cumprimento ou rejeição (seja como for realmente chamado). Considere:

```js
ajax( "http://some.url.1" )
.then(
  function fulfilled(contents){
    return contents.toUpperCase();
  },
  function rejected(reason){
    return "DEFAULT VALUE";
  }
)
.then( function fulfilled(data){
  // manipula data da promise original
} );
```

Nesse trecho de código, estamos retornando um valor a partir de qualquer um dos métodos `fulfilled(..)` ou `rejected(..)`, que então é recebido no próximo `fulfilled(..)` do segundo `then(..)`. Se ao invés disso retornarmos uma nova *promise*, essa nova *promise* é adotada como a resolução:

```js
ajax( "http://some.url.1" )
.then(
  function fulfilled(contents){
    return ajax(
      "http://some.url.2?v=" + contents
    );
  },
  function rejected(reason){
    return ajax(
      "http://backup.url.3?err=" + reason
    );
  }
)
.then( function fulfilled(contents){
  // `contents` vem de uma das chamadas subsequentes de `ajax(..)`
} );
```

É importante notar que uma exceção (ou uma *promise* rejeitada) no primeiro `fulfilled(..)` *não* resultará na primeira execução do método `rejected(..)`, já que esse manipulador responde apenas à resolução da *promise* original. Ao invés disso, a segunda *promise*, cujo segundo `then(..)` é chamado de encontro, recebe a rejeição.

No trecho de código anterior, não estamos ouvindo a rejeição, o que significa que ela será silenciosamente guardada para futuras observações. Se você não lidar com a rejeição utilizando `then(..)` ou `catch(..)`, ela não será processada. Os consoles de alguns navegadores podem detectar essas rejeições não processadas e reportá-las, mas isso não é garantido; você deveria sempre observar a rejeição de uma *promise*.

**Nota:** Isso foi apenas uma visão geral e breve da teoria e comportamento em respeito de uma *promise*. Para uma exploração detalhada, veja o Capítulo 3 do título dessa série *Async & Performance*.

### Thenables

*Promises* são instâncias do construtor `Promise(..)`. Contudo, há objetos *semelhantes à promise* chamados *thenables* que geralmente podem trabalhar em conjunto com mecanismos *Promise*

Qualquer objeto (ou função) com um método `then(..)` é tido como um *thenable*. Qualquer lugar onde mecanismos *Promise* podem aceitar e adotar o estado de uma *promise* genuína, ele pode também lidar com um *thenable*.

*Thenables* são basicamente rótulos genéricos para qualquer valor *semelhante à promise* que pode ter sido criado por outro mecanismo diferente do atual construtor `Promise()`. Nessa perspectiva, um *thenable* é geralmente menos confiável que uma *Promise* genuína. Considere esse *thenable* inapropriado, por exemplo:

```js
var th = {
  then: function thener( fulfilled ) {
    // chama `fulfilled(..)` uma vez a cada 100ms eternamente.
    setInterval( fulfilled, 100 );
  }
};
```

Se você recebeu esse *thenable* e o encadeou utilizando `th.then(..)`, você provavelmente se surpreendeu pelo manipulador de cumprimento ter sido chamado repetidamente, enquanto em *Promises* originais espera-se que sejam resolvidas apenas uma única vez.

Geralmente, você não deve confiar cegamente se você está esperando receber o que se propõe a ser uma *promise* ou *thenable* de algum outro sistema. Na próxima seção, nós veremos uma vantagem incluída nas Promises do ES6 que ajuda a endereçar essas preocupações com a confiabilidade.

Mas para entender melhor os perigos desse assunto, considere que *qualquer* objeto em *qualquer* pedaço de código que foi definido para ter um método chamado `then(..)` pode ser potencialmente confundido com um *thenable* -- se usado como *Promises*, claro -- mesmo se esse código nem remotamente tiver sido destinado a ser um código assíncrono *estilo Promise*.

Antes do ES6, nunca houve qualquer método reservado chamado `then(..)`, e como você pode imaginar há ao menos alguns casos onde esse nome foi escolhido para um método antes das *Promises* terem entrado em cena. O equívoco mais provável de *thenable* seriam bibliotecas assíncronas que usam `then(..)` mas que não são estritamente compatíveis com Promises -- existem muitas por aí.

A responsabilidade será sua de se proteger de usar diretamente valores com mecanismo *Promise* que poderiam ser erradamente assumidos como *thenable*.

### *promise* API

A API *promise* também provê alguns métodos estáticos para trabalhar com *Promises*

`Promise.resolve(..)` cria uma `promise` resolvida para o valor informado. Vamos comparar como isso funciona em uma abordagem manual:

```js
var p1 = Promise.resolve( 42 );

var p2 = new Promise( function pr(resolve){
  resolve( 42 );
} );
```

`p1` e `p2` terão essencialmente comportamentos idênticos. O mesmo vale para resolver uma `promise`:

```js
var theP = ajax( .. );

var p1 = Promise.resolve( theP );

var p2 = new Promise( function pr(resolve){
  resolve( theP );
} );
```

**Dica** `Promise.resolve(..)` é uma solução para problemas de confiabilidade de *thenable* abordados na seção anterior. Qualquer valor que você ainda não está seguro se é uma promise confiável -- mesmo podendo ser um valor imediato -- pode ser normalizado passando-o como argumento para `Promise.resolve(..)`. Se o valor já for uma *promise* reconhecida ou *thenable*, seu estado/resolução será simplesmente adotado, prevenindo-o de comportamentos não esperados. E mesmo sendo um valor imediato, o mesmo será envolvido em uma *promise* genuína, normalizando assim seu comportamento para ser assíncrono.

`Promise.reject(..)` cria imediatamente uma `promise` rejeitada, o mesmo que seu construtor `Promise(..)`: 

```js
var p1 = Promise.reject( "Oops" );

var p2 = new Promise( function pr(resolve,reject){
  reject( "Oops" );
} );
```

Enquanto `resolve(..)` e `Promise.resolve(..)` podem aceitar uma promise e adotar seu estado/resolução, `reject(..)` e `Promise.reject(..)` não diferenciam qual valor eles recebem. Então, se você rejeitar com uma `promise` ou `thenable`, a `promise/thenable` por si só será setada como a razão da rejeição, e não o seu valor subjacente.

`Promise.all([ .. ])` aceita um array de um ou mais valores (ex.: valores imediatos, `promises`, `thenables`). Ele retorna uma promise que será cumprida se todos os valores se cumprirem, ou rejeitada imediatamente assim que o primeiro de um deles for rejeitado.

Iniciando com esses valores/`promises`:

```js
var p1 = Promise.resolve( 42 );
var p2 = new Promise( function pr(resolve){
  setTimeout( function(){
    resolve( 43 );
  }, 100 );
} );
var v3 = 44;
var p4 = new Promise( function pr(resolve,reject){
  setTimeout( function(){
    reject( "Oops" );
  }, 10 );
} );
```

Vamos entender como `Promise.all([ .. ])` funciona com a combinação desses valores:

```js
Promise.all( [p1,p2,v3] )
.then( function fulfilled(vals){
  console.log( vals );      // [42,43,44]
} );

Promise.all( [p1,p2,v3,p4] )
.then(
  function fulfilled(vals){
    // never gets here
  },
  function rejected(reason){
    console.log( reason );    // Oops
  }
);
```
Enquanto `Promise.all([ .. ])` espera por todos os cumprimentos (ou a primeira rejeição), `Promise.race([ .. ])` aguarda apenas pelo primeiro cumprimento ou rejeição. Considere: 

```js
// NOTE: re-setup all test values to
// avoid timing issues misleading you!

Promise.race( [p2,p1,v3] )
.then( function fulfilled(val){
  console.log( val );        // 42
} );

Promise.race( [p2,p4] )
.then(
  function fulfilled(val){
    // never gets here
  },
  function rejected(reason){
    console.log( reason );    // Oops
  }
);
```

**Atenção:** Enquanto `Promise.all([])` se cumprirá imediatamente (sem valores), `Promise.race([])` aguardará para sempre. Essa é uma estranha inconsistência, e sugere que você nunca deveria usar esses métodos com arrays vazios.

## Generators + Promises

*É* possível expressar séries de `promises` em cadeia para representar o fluxo assíncrono do seu código. Considere:

```js
step1()
.then(
  step2,
  step1Failed
)
.then(
  function step3(msg) {
    return Promise.all( [
      step3a( msg ),
      step3b( msg ),
      step3c( msg )
    ] )
  }
)
.then(step4);
```

Contudo, há uma opção muito melhor para expressar controle de fluxo assíncrono, e provavelmente será muito mais preferível em termos de estilo de codificação do que longas cadeias de `promise`. Nós podemos usar o que aprendemos no *Capítulo 3* sobre `generators` para expressar nosso controle de fluxo assíncrono.

Um importante padrão a se reconhecer: um `generator` pode produzir uma `promise`, e essa promise pode então ser ligada para retomar o `generator` com seu valor de cumprimento.

Considere o controle de fluxo assíncrono no trecho de código anterior escrito com um `generator`.

```js
function *main() {

  try {
    var ret = yield step1();
  }
  catch (err) {
    ret = yield step1Failed( err );
  }

  ret = yield step2( ret );

  // step 3
  ret = yield Promise.all( [
    step3a( ret ),
    step3b( ret ),
    step3c( ret )
  ] );

  yield step4( ret );
}
```

À primeira vista, o trecho de código pode parecer mais verboso que a cadeia de `promise` equivalente no outro trecho de código. Contudo, isso oferece um estilo de código de aparência síncrona muito mais atrativo -- e o mais importante, mais compreensível -- (com a instrução `=` para retornos de valores, etc.) É especialmente verdade que o manipulador de erros `try catch` pode ser usado através dessas fronteiras ocultas assíncronas.

Por que estamos utilizando `Promises` com `generator`? É totalmente possível escrever `generators` assíncronos sem `Promises`.

`Promises` são sistemas confiáveis que desfazem a inversão de controle de `callbacks` ou `thunks` (veja o título *Async & Performance* dessa série). Então, combinando a confiabilidade das `Promises` e a sincronicidade de código dos `generators` trata efetivamente todas as maiores deficiências das `callbacks`. Também, funções utilitárias como `Promise.all([..])` expressam de forma clara e agradável concorrência em um único passo `yield` dos `generators`.

Então, como essa mágica funciona? Nós precisaremos de um *runner* que executa nosso `generator`, recebe uma `promise` `yield`, e os liga para formar um `generator` com ambos, o valor de cumprimento, ou lança um erro no `generator` com o motivo da rejeição.

Muitas bibliotecas/utilitários capazes de rodar o código assíncrono incluem um "runner";
por exemplo, `Q.spawn(..)` e meu plugin de sequência assíncrona `runner(..)`. Mas aqui está um runner stand-alone ilustrando como o processo funciona:

```js
function run(gen) {
  var args = [].slice.call( arguments, 1), it;

  it = gen.apply( this, args );

  return Promise.resolve()
    .then( function handleNext(value){
      var next = it.next( value );

      return (function handleResult(next){
        if (next.done) {
          return next.value;
        }
        else {
          return Promise.resolve( next.value )
            .then(
              handleNext,
              function handleErr(err) {
                return Promise.resolve(
                  it.throw( err )
                )
                .then( handleResult );
              }
            );
        }
      })( next );
    } );
}
```

**Nota:** Para uma versão melhor comentada dessa `utility`, veja o capítulo *Async & Performance* dessa série. Também, a `utilidade de execução` provida em várias bibliotecas assíncronas é frequentemente mais poderosa/capaz do que as que nós mostramos aqui. Por exemplo, `*asynquence's*` `runner(..)` pode lidar com promises `yield`, sequences, thunks, e valores (que não são promises) imediatos, dando a você muita flexibilidade.

Então executar `*main()` conforme listado no trecho de código anterior é simples como:

```js
run( main )
.then(
  function fulfilled(){
    // `*main()` executada com sucesso
  },
  function rejected(reason){
    // Oops, alguma coisa está errada
  }
);
```

Essencialmente, qualquer lugar que possua mais do que dois passos assíncronos de controle de fluxo lógico em seu programa, você pode *e deveria* usar um gerador `promise-yielding` como uma função utilitária para expressar o controle de fluxo de forma síncrona. Isso tornará muito mais fácil de entender e manter o código.

Esse padrão yield-a-promise-resume-the-generator irá ser muito comum e poderoso, a próxima geração do JavaScript pós ES6 quase certamente introduzirá um novo tipo de função que fará isso automaticamente sem precisar do utilitário de execução. Explicaremos `funções assíncronas` (como esperamos que elas sejam chamadas) no Capítulo 8.

## Revisão

Como JavaScript continua amadurecendo e crescendo com uma adoção bem difundida, programação assíncrona está sendo cada vez mais o centro das atenções. *Callbacks* não são totalmente suficientes para essas tarefas, e são totalmente ineficientes para necessidades mais sofisticadas.

Ainda bem, ES6 traz *Promises* para tratar uma das maiores deficiências das `callbacks`: falta de confiança em comportamentos previsíveis. `Promises` representam o valor futuro de uma tarefa potencialmente assíncrona, normalizando comportamentos síncronos e assíncronos.

Mas é a combinação de *Promises* com *generators* que melhor demonstra os benefícios de rearranjar nosso código de controle de fluxo assíncrono para desestimular e abstrair aquela horrível sopa de `callbacks` (aka "hell").

Até então, podemos gerenciar essas interações com a ajuda de várias bibliotecas assíncronas, mas JavaScript eventualmente suportará esses padrões de interação em sua própria sintaxe.
