# You Don't Know JS: Up & Going
# Capítulo 3: Dentro do YDKJS

Sobre o que é essa série? Simplificando, trata-se de levar a sério a tarefa de aprender *todas as partes do JavaScript*, e não apenas um subconjunto da linguagem que alguns chamam de "as partes boas", e também não apenas o mínimo que você precisa fazer para terminar o seu trabalho.

Desenvolvedores sérios em outras linguagens esperam colocar seu esforço para aprender a maioria ou a totalidade da linguagem (ou linguagens) que eles mais usam, mas os desenvolvedores JavaScript parecem se destacar da multidão no sentido de normalmente não aprender muito da linguagem em si. Esta não é uma coisa boa, e não é algo que podemos continuar a permitir a ser o modelo.

A série *You Don't Know JS* (*YDKJS*) está em contraste gritante com as abordagens típicas para aprender JS, e é diferente de praticamente qualquer outros livros sobre JS que você vai ler. Ele te desafia a sair da sua zona de conforto e de se perguntar os mais profundos "porquês" para cada comportamento que você encontrar. Você está pronto para esse desafio?

Vou usar este capítulo final para resumir o que esperar do resto dos livros da série, e como ir de forma mais eficaz sobre a construção de uma base de aprendizagem de JavaScript em cima da *YDKJS*.

## Escopo & Closures

Talvez uma das coisas mais fundamentais que você precisa para chegar rapidamente a um entendimento, é como o escopo de variáveis realmente funciona em JavaScript. Isso não é o suficiente para ter *opiniões* vagas sobre escopo.

O título *Scope & Closures* começa por desmascarar o equívoco comum que JS é uma "linguagem interpretada" e, portanto, não compilado. Não.

O motor de JS compila seu código logo antes (e às vezes durante!) a execução. Então, nós usamos um pouco de compreensão mais profunda da abordagem do compilador para o nosso código, para entender como ele encontra e lida com declarações de variáveis e funções. Ao longo do caminho, vemos a típica metáfora para gerenciamento de escopo de variáveis em JS, *"Hoisting"* (Elevação).

É nesta compreensão crítica do "escopo léxico" que nós iremos basear a nossa exploração de *Closure* para o último capítulo do livro. Talvez *Closure* seja o conceito mais importante em toda a linguagem JavaScript, mas se você primeiramente não entender firmemente como o escopo funciona, *Closure* provavelmente permanecerá fora do seu alcance.

Uma aplicação importante de *Closure* é o *module pattern*, como nós introduzimos brevemente neste livro, no Capítulo 2. O *module pattern* (módulo padrão) é, talvez, o padrão de organização de código que mais prevalece em todos JavaScript; a profunda compreensão disso, deve ser uma de suas maiores prioridades.

## this & Object Prototypes

Talvez uma das inverdades mais comuns e persistentes sobre JavaScript é que a palavra-chave `this` se refere à função qual ela aparece. Terrivelmente enganado.

A palavra-chave `this` é dinamicamente ligada com base em como a função em questão é executada, e existem quatro regras básicas para entender e determinar plenamente a ligação do `this`.

Intimamente relacionado com a palavra-chave `this`, está o mecanismo object prototype (protótipo de objeto), que é uma cadeia de pesquisa para as propriedades, similar ao modo léxico que o escopo de variáveis é encontrado. Mas envolto nos prototypes, está outro enorme erro sobre JS: a (falsa) ideia de emular classes e (a chamada "prototipagem") herança.

Infelizmente, o desejo de trazer o design pattern (padrão de projeto) de classes e heranças para o JavaScript é apenas a pior coisa que você poderia tentar fazer, porque enquanto a sintaxe pode induzi-lo a pensar que há algo como classes, de fato o mecanismo de prototype é fundamentalmente oposto em o seu funcionamento.

O que está em questão é se é melhor ignorar a incompatibilidade e fingir que o que você está implementando é "herança", ou se é mais apropriado aprender e abraçar como o sistema de prototype de objeto realmente funciona. Este último é mais apropriadamente chamado de "delegação de funcionamento."

Então isso é mais sobre preferência sintática. A delegação é um sistema totalmente diferente, e mais poderoso, de design pattern, que substitui a necessidade de planejar classes e heranças. Mas essas afirmações com certeza vão pros ares frente à quase todos os outros post, livro, e conferências sobre o assunto para a duração da vida do JavaScript.

As afirmações que faço sobre delegação contra herança vem não de uma antipatia da linguagem e da sua sintaxe, mas a partir do desejo de ver a verdadeira capacidade da linguagem adequadamente aproveitada, e a confusão sem fim e frustração indo embora.

Mas o caso que eu faço a respeito de prototypes e a delegação é muito mais do que o que eu vou falar aqui. Se você está pronto para reconsiderar tudo o que você pensa que sabe sobre as "classes" e "heranças" em JavaScript, eu lhe ofereço a oportunidade para "tomar a pílula vermelha" (*Matrix* 1999) e conferir os capítulos 4-6 do título *this & Object Prototypes* desta série.

## Types & Grammar

The third title in this series primarily focuses on tackling yet another highly controversial topic: type coercion. Perhaps no topic causes more frustration with JS developers than when you talk about the confusions surrounding implicit coercion.

By far, the conventional wisdom is that implicit coercion is a "bad part" of the language and should be avoided at all costs. In fact, some have gone so far as to call it a "flaw" in the design of the language. Indeed, there are tools whose entire job is to do nothing but scan your code and complain if you're doing anything even remotely like coercion.

But is coercion really so confusing, so bad, so treacherous, that your code is doomed from the start if you use it?

I say no. After having built up an understanding of how types and values really work in Chapters 1-3, Chapter 4 takes on this debate and fully explains how coercion works, in all its nooks and crevices. We see just what parts of coercion really are surprising and what parts actually make complete sense if given the time to learn.

But I'm not merely suggesting that coercion is sensible and learnable, I'm asserting that coercion is an incredibly useful and totally underestimated tool that *you should be using in your code.* I'm saying that coercion, when used properly, not only works, but makes your code better. All the naysayers and doubters will surely scoff at such a position, but I believe it's one of the main keys to upping your JS game.

Do you want to just keep following what the crowd says, or are you willing to set all the assumptions aside and look at coercion with a fresh perspective? The *Types & Grammar* title of this series will coerce your thinking.

## Async & Performance

The first three titles of this series focus on the core mechanics of the language, but the fourth title branches out slightly to cover patterns on top of the language mechanics for managing asynchronous programming. Asynchrony is not only critical to the performance of our applications, it's increasingly becoming *the* critical factor in writability and maintainability.

The book starts first by clearing up a lot of terminology and concept confusion around things like "async," "parallel," and "concurrent," and explains in depth how such things do and do not apply to JS.

Then we move into examining callbacks as the primary method of enabling asynchrony. But it's here that we quickly see that the callback alone is hopelessly insufficient for the modern demands of asynchronous programming. We identify two major deficiencies of callbacks-only coding: *Inversion of Control* (IoC) trust loss and lack of linear reason-ability.

To address these two major deficiencies, ES6 introduces two new mechanisms (and indeed, patterns): promises and generators.

Promises are a time-independent wrapper around a "future value," which lets you reason about and compose them regardless of if the value is ready or not yet. Moreover, they effectively solve the IoC trust issues by routing callbacks through a trustable and composable promise mechanism.

Generators introduce a new mode of execution for JS functions, whereby the generator can be paused at `yield` points and be resumed asynchronously later. The pause-and-resume capability enables synchronous, sequential looking code in the generator to be processed asynchronously behind the scenes. By doing so, we address the non-linear, non-local-jump confusions of callbacks and thereby make our asynchronous code sync-looking so as to be more reason-able.

But it's the combination of promises and generators that "yields" our most effective asynchronous coding pattern to date in JavaScript. In fact, much of the future sophistication of asynchrony coming in ES7 and later will certainly be built on this foundation. To be serious about programming effectively in an async world, you're going to need to get really comfortable with combining promises and generators.

If promises and generators are about expressing patterns that let our programs run more concurrently and thus get more processing accomplished in a shorter period, JS has many other facets of performance optimization worth exploring.

Chapter 5 delves into topics like program parallelism with Web Workers and data parallelism with SIMD, as well as low-level optimization techniques like ASM.js. Chapter 6 takes a look at performance optimization from the perspective of proper benchmarking techniques, including what kinds of performance to worry about and what to ignore.

Writing JavaScript effectively means writing code that can break the constraint barriers of being run dynamically in a wide range of browsers and other environments. It requires a lot of intricate and detailed planning and effort on our parts to take a program from "it works" to "it works well."

The *Async & Performance* title is designed to give you all the tools and skills you need to write reasonable and performant JavaScript code.

## ES6 & Beyond

No matter how much you feel you've mastered JavaScript to this point, the truth is that JavaScript is never going to stop evolving, and moreover, the rate of evolution is increasing rapidly. This fact is almost a metaphor for the spirit of this series, to embrace that we'll never fully *know* every part of JS, because as soon as you master it all, there's going to be new stuff coming down the line that you'll need to learn.

This title is dedicated to both the short- and mid-term visions of where the language is headed, not just the *known* stuff like ES6 but the *likely* stuff beyond.

While all the titles of this series embrace the state of JavaScript at the time of this writing, which is mid-way through ES6 adoption, the primary focus in the series has been more on ES5. Now, we want to turn our attention to ES6, ES7, and ...

Since ES6 is nearly complete at the time of this writing, *ES6 & Beyond* starts by dividing up the concrete stuff from the ES6 landscape into several key categories, including new syntax, new data structures (collections), and new processing capabilities and APIs. We cover each of these new ES6 features, in varying levels of detail, including reviewing details that are touched on in other books of this series.

Some exciting ES6 things to look forward to reading about: destructuring, default parameter values, symbols, concise methods, computed properties, arrow functions, block scoping, promises, generators, iterators, modules, proxies, weakmaps, and much, much more! Phew, ES6 packs quite a punch!

The first part of the book is a roadmap for all the stuff you need to learn to get ready for the new and improved JavaScript you'll be writing and exploring over the next couple of years.

The latter part of the book turns attention to briefly glance at things that we can likely expect to see in the near future of JavaScript. The most important realization here is that post-ES6, JS is likely going to evolve feature by feature rather than version by version, which means we can expect to see these near-future things coming much sooner than you might imagine.

The future for JavaScript is bright. Isn't it time we start learning it!?

## Review

The *YDKJS* series is dedicated to the proposition that all JS developers can and should learn all of the parts of this great language. No person's opinion, no framework's assumptions, and no project's deadline should be the excuse for why you never learn and deeply understand JavaScript.

We take each important area of focus in the language and dedicate a short but very dense book to fully explore all the parts of it that you perhaps thought you knew but probably didn't fully.

"You Don't Know JS" isn't a criticism or an insult. It's a realization that all of us, myself included, must come to terms with. Learning JavaScript isn't an end goal but a process. We don't know JavaScript, yet. But we will!
