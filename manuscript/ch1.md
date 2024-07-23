# Functional-Light JavaScript 
# Capitulo 1: Por que Programação Funcional?

> Programação Funcional: (substantivo) Aquele que nomeia variáveis "x", nomeia funções "f", e nomeia padrões de código "pré-promorfismo zigohistomórfico"
>
> James Iry @jamesiry 5/13/15
>
> https://twitter.com/jamesiry/status/598547781515485184

Programação Funcional (FP) de forma alguma é um conceito novo. Ela esta presente em toda a historia da programação. Entretanto, e não tenho certeza se é justo dizer, mas... com certeza não parecia um conceito tão comum no mundo dos desenvolvedores até nos ultimos dez anos talvez. Acho que a Programação Funcional tem sido mais uma matéria acadêmica.

No entanto tudo isso esta mudando. Uma onda de interesse está crescendo em torno da Programação funcional, não apenas no nível das linguagens, mas também nas bibliotecas e ferramentas. Você pode muito bem estar lendo este livro porque finalmente entendeu que programação funcional é algo que você não pode mais ignorar. Ou talvez você seja como eu e já tentou aprender programação funcional muitas vezes mas teve dificuldades em ler todos os termos ou notações matemáticas.

O objetivo deste primeiro capítulo é responder perguntas como "Por que devo usar o estilo de (FP) no meu código?" e "Como este livro se compara ao que os outros dizem sobre (FP)?". Depois de estabelecermos uma base, ao longo do livro descobriremos, peça por peça, as técnicas e os padrões para escrever JS no estilo "Funcional Leve".

## Uma olhada

Vamos fazer uma pequena ilustração de "JavaScript Funcional-Light" com uma visão antes/depois de um código. Considere:

```js
var numbers = [4,10,0,27,42,17,15,-6,58];
var faves = [];
var magicNumber = 0;

pickFavoriteNumbers();
calculateMagicNumber();
outputMsg();                // The magic number is: 42

// ***************

function calculateMagicNumber() {
    for (let fave of faves) {
        magicNumber = magicNumber + fave;
    }
}

function pickFavoriteNumbers() {
    for (let num of numbers) {
        if (num >= 10 && num <= 20) {
            faves.push( num );
        }
    }
}

function outputMsg() {
    var msg = `The magic number is: ${magicNumber}`;
    console.log( msg );
}
```

Agora considere um estilo muito diferente que alcança exatamente o mesmo resultado:

```js
var sumOnlyFavorites = FP.compose( [
    FP.filterReducer( FP.gte( 10 ) ),
    FP.filterReducer( FP.lte( 20 ) )
] )( sum );

var printMagicNumber = FP.pipe( [
    FP.reduce( sumOnlyFavorites, 0 ),
    constructMsg,
    console.log
] );

var numbers = [4,10,0,27,42,17,15,-6,58];

printMagicNumber( numbers );        // The magic number is: 42

// ***************

function sum(x,y) { return x + y; }
function constructMsg(v) { return `The magic number is: ${v}`; }
```

Uma vez que você entende (FP) e "Funcional-Light", é provável que você *leia* e processe mentalmente esse segundo trecho:

> Primeiro estamos criando uma função chamada `sumOnlyFavorites(..)` que é uma combinação de três outras funções. Combinamos dois filtros, um verificando se o valor é maior ou igual a 10 e outro para menor ou igual a 20. Então incluímos o `sum(..)` reducer(redutor) na composição do transducer(transdutor). O resultado da função `sumOnlyFavorites(..)` é um reducer(redutor) que verifica se o valor passa em ambos os filtros e, em caso positivo, adiciona o valor a um accumulator(acumulador).
>
> Então criamos uma outra função chamada `printMagicNumber(..)` que primeiro reduces(reduz) uma lista de números favoritos usando o reducer(redutor) `sumOnlyFavorites(..)` que acabamos de definir, resultando em uma soma de apenas números que passaram na verificação de *favoritos*. Então `printMagicNumber(..)` canaliza essa soma final para `constructMsg(..)`, que cria um valor de string que finalmente irá para o `console.log(..)`.

Todas essas peças móveis *falam* com um desenvolvedor de programação funcional, de maneira que parecem bem "infamiliar" para vocês neste momento. Este livro irá ajuda-lo a *falar* este mesmo tipo de raciocínio para que este seja tão legível quanto qualquer outro código, se não até mais!

Algumas outras observações rápidas sobre esta comparação de código:

* É provável que, para muitos leitores, o primeiro trecho pareça mais confortável/legível/de fácil manutenção do que o último trecho. Está tudo bem se for este o caso. Você esta exatamente no lugar certo. Tenho certeza de que se você persistir durante todo o livro e praticar tudo o que falamos, esse segundo trecho acabará se tornando muito mais natural, talvez até preferível!

* Você pode ter executado a tarefa de forma significativa ou totalmente diferente de qualquer um dos trechos apresentados. Esta tudo OK, também. Este livro não será preescritivo ao ditar que você deve fazer algo de uma maneira específica. O objetivo é ilustrar os prós/contras de vários padrões e permitir que você tome essas decisões. Ao final deste livro, a forma como você abordaria a tarefa poderá ficar um pouco mais próxima do segundo trecho do que está agora.

* Também é possível que você já seja um desenvolvedor experiente de FP e esteja lendo o início deste livro para ver se há algo útil para você. Esse segundo trecho certamente contém algumas partes que são bastante familiares. Mas também aposto que você pensou: "Hmm, eu não teria feito isso *assim*..." algumas vezes. Esta OK, e totalmente razoável.

    Este não é um livro tradicional e canônico de FP. Às vezes parecemos bastante heréticos em nossas abordagens. Estamos buscando encontrar um equilíbrio pragmatico entre os benefícios inegáveis da FP e a necessidade de fornecer JS viável e sustentável sem ter que enfrentar uma montanha assustadora de matemática/notação/terminologia. Esta não é a *sua* FP. é "Funcional-Light JavaScript".

Quaisquer que sejam suas razões para ler este livro, seja bem-vindo!

## Confiança

Eu tenho uma premissa muito simples que fundamenta tudo que faço como professor de desenvolvimento de software (em JavaScript): código em que você não pode confiar é um código que você não entende. O inverso também é verdadeiro: código que você não entende é código em que você não pode confiar. Além disso, se você não pode confiar ou compreender seu código, não poderá ter qualquer confiança de que o código que você escreve é adequado para a tarefa. Você executa o programa e basicamente cruza os dedos.

O que quero dizer com confiança? Quero dizer que você pode verificar, lendo e raciocinando, não apenas executando, que você entende o que um trecho de código *fará*; você não está apenas confiando no que ele *deveria* fazer. Mais frequentemente do que seria prudente, tendemos a confiar na execução de conjuntos de testes para verificar a correção de nossos programas. Não quero sugerir que os testes sejam ruins. Mas acho que devemos aspirar a entender nossos códigos bem o suficiente para sabermos que o conjunto de testes será aprovado antes de ser executado.

As técnicas que constituem a base da FP são concebidas a partir  da mentatilade de ter muito mais confiança nos nossos programas apenas por lê-los. Alguém que entende de FP e que é disciplinado o suficiente para usá-la diligentemente em seus programas escreverá um código que eles **e outros** poderão ler e verificar se o programa fará o que deseja.

A confiança também aumenta quando usamos técnicas que evitam ou minimizam prováveis fontes de bugs. Esse é talvez um dos maiores argumentos de venda da FP: os programas geralmente têm menos bugs, e os bugs que exitem geralmente estão em lugares mais óbvios, por isso são mais fáceis de encontrar e corrigir. O código de programação funcional tende a ser mais resistente a bugs -- certamente não à prova de bugs.

À medida que você avança neste livro, você começará a desenvolver mais confiança no código que escreve, porque você usará padrões e práticas já comprovadas; e você evitará as causas mais comuns de bugs de programa!

## Comunicação

Por que a Programação Funcional é importante? Para responder a isso, precisamos dar um passo atrás e falar sobre por que a programação em si é importante.

Você pode ficar surpreso ao ouvir isso, mas não acredito que o código seja princimalmente um conjunto de intruções para o computador. Na verdade, acho que o fato de o código instruir o computador é quase um feliz acidente.

Eu acredito profundamente que o papel mais importante do código é como meio de comunicação com outros seres humanos.

Você provavelmente sabe por experiência própria que muito do seu tempo gasto "codando" é, na verdade, gasto na leitura de código existente. Muito poucos de nós temos o privilégio de gastar todo ou a maior parte do nosso tempo simplesmente elaborando todos os novos códigos e nunca lidando com códigos que outros (ou nossos "eus" anteriores) escreveram.

É amplamente estimado que os desenvolvedores gastam 70% do tempo de manutenção do código lendo para entendê-lo. Isso é revelador. 70%. Não é de admirar que a média global de linhas de código escritas por dia por um programador seja cerca de 10 linhas. Passamos até 7 horas do nosso dia apenas lendo o código para descobrir onde essas 10 linhas devem ir!

Precisamos nos concentrar muito mais na legibilidade do nosso código. E, a propósito, legibilidade não envolve apenas menos caracteres. A legibilidade é, na verdade, mais afetada pela familiaridade.<a href="#user-content-footnote-1"><sup>1</sup></a>

Se vamos gastar nosso tempo procupados em criar código que seja mais legível e compeensível, a FP é fundamental nesse esforço. Os princípios da FP estão bem estabelecidos, profundamente estudados e avaliados, e comprovadamente verificáveis. Reservar um tempo para aprender e empregar esses princípios de FP resultará em um código mais fácil e reconhecidamente familiar para você e outras pessoas. O aumento na familiaridade com código e a conveniência desse reconhecimento melhorarão a legibilidade do código.

Por exemplo, depois de aprender o que `map(..)` faz, você será capaz de identificá-lo e entendê-lo quase instantaneamente quando o vir em qualquer programa. Mas toda vez que você ver um loop `for`, você terá que ler o loop inteiro para entendê-lo. A sintaxe do loop `for` pode ser familiar, mas a "substância" do que ele está fazendo não é; isso tem que ser *lido*, sempre.

Ao ter mais código reconhecível à primeira vista e, assim, gastar menos tempo descobrindo o que o código está fazendo, nosso foco fica liberado para pensar nos níveis mais elevados da lógica do programa; de qualquer maneira, essa é a coisa mais importante que precisa de nossa atenção.

FP (pelo menos, sem pensar em toda a terminologia) é uma das ferramentas mais eficazes para criar código legível. *É* por isso que é tão importante.

## Legibilidade

A legibilidade não é uma característica binária. É um fator humano amplamente subjetivo que descreve nossa relação com o código. E isso irá naturalmente variar ao longo do tempo, à medida que nossas habilidades e compreensão evoluem. Experimentei efeitos semelhantes aos da figura a seguir e, curiosamente, muitos outros com quem conversei também.

<p align="center">
    <img src="images/fig17.png" width="50%">
</p>

Você pode experimentar efeitos semelhantes à medida que lê o livro. Mas tenha coragem; se você persistir, a curva volta a subir!

*Imperativo* descreve o código que a maioria de nós provavelmente já escreve naturalmente; seu foco é instruir precisamente o computador "*como*" fazer algo. Código declarativo -- o tipo que aprenderemos a escrever, que segue os princípios de FP -- é um código mais focado em descrever "*o que*" resultado.

Vamos revisitar os dois trechos de código apresentados anteriormente neste capítulo.

O primeiro trecho é imperativo, focado quase inteiramente em *como* realizar as tarefas; está repleto de intruções `if`, loops `for`, varíaveis temporárias, reatribuições, mutações de valores, chamadas de função com efeitos colaterais e fluxo de dados implícito entre funções. Você certamente *pode* rastrear sua lógica para ver como os números fluem e mudam para o estado final, mas isso não é nada claro ou direto.

O segundo trecho é mais declarativo; elimina a maioria das técnicas imperativas mencionadas acima. Observe que não há condicionais, loops, efeitos colaterais, reatribuições ou mutações explícitas; em vez disso, ele emprega padrões bem conhecidos (pelo menos para o mundo da FP!) e confiáveis, como filtragem, redução, transdução e composição. O foco muda do nível inferior de "*como*" para o nível superior de "*quais*" resultados.

Em vez de mexer com uma instrução `if` para testar um número, delegamos isso a um utilitário da FP bem conhecido como `gte(..)` (maior que ou igual a) e então nos concentramos na tarefa mais importante de combinar esse filtro com outro filtro e uma fução de soma.

Além disso, o fluxo de dados através do segundo programa é explícito:

1. Uma lista de números vai para `printMagicNumber(..)`.
2. Um de cada vez, esses números são processados por `sumOnlyFavorites(..)`, resultando em um número unico através da soma dos números favoritos.
3. Esse total é convertido em uma string de mensagem com `constructMsg(..)`.
4. A string da mensagem é impressa no console com o `console.log(..)`.

Você ainda pode achar que essa abordagem é complicada e que o trecho imperativo foi mais fácil de entender. Você está muito mais acostumado com isso; a familiaridade tem uma influência profunda em nossos julgamentos de legibilidade. No entanto, ao final deste livro, você terá internalizado os benefícios da abordagem declarativa do segundo trecho, e essa familiaridade dará vida à sua legigibilidade.

Eu sei que pedir para você acreditar nisso neste momento é um ato de fé.

É preciso muito mais esforços, e às vezes mais código, para melhorar sua legibilidade, como estou sugerindo, para minimizar ou eliminar muitos dos erros que levam a bugs. Sinceramente, quando comecei a escrever este livro, nunca poderia ter escrito (ou mesmo compreendido completamente!) aquele segundo trecho. À medida que estou avançando em minha jornada de aprendizado, é mais natural e confortável.

Se você espera a refatoração FP, como uma bala de prata mágica, transforme rapidamente seu código para ser mais gracioso, elegante, inteligente, resiliente e conciso -- e que será fácil no curto prazo -- infelizmente isso não é uma expectativa realista.

FP é uma maneira muito diferente de pensar sobre como o código deve ser estruturado, para tornar o fluxo de dados muito mais óbvio e para ajudar o leitor a seguir seu pensamento. Isso levará algum tempo. Este esforço vale eminintemente a pena, mas pode ser uma jornada árdua.

Muitas vezes ainda preciso de várias tentativas para refatorar um trecho de código imperativo em um FP mais declarativa, antes de chegar a algo que seja claro o suficiente para eu entender mais tarde. Descobri que a conversão para FP é um processo iterativo lento, em vez de uma rápida mudança binária de um paradigma para outro.

Também aplico o teste "ensine mais tarde" a cada trecho de código que escrevo. Depois de escrever um trecho de código, deixo-o sozinho por algumas horas ou dias, depois volto e tento lê-lo com novos olhos e finjo que preciso ensiná-lo ou explicá-lo para outra pessoa. Normalmente fica atrapalhado e confuso nas primeiras passagens, então eu ajusto e repito!

Não estou tentando diminuir seu ânimo. Eu realmente quero que você corte essas ervas daninha. Estou feliz por ter feito isso. Posso finalmente começar a ver a linha se curvando para cima em direção a uma melhor legibilidade. O esforço valeu a pena. Valerá para você também.

## Perspective

Most other FP texts seem to take a top-down approach, but we're going to go the opposite direction: working from the ground up, we'll uncover the basic foundational principles that I believe formal FPers would admit are the scaffolding for everything they do. But for the most part we'll stay arm's length away from most of the intimidating terminology or mathematical notation that can so easily frustrate learners.

I believe it's less important what you call something and more important that you understand what it is and how it works. That's not to say there's no importance to shared terminology -- it undoubtedly eases communication among seasoned professionals. But for the learner, I've found it can be distracting.

So this book will try to focus more on the base concepts and less on the fancy fluff. That's not to say there won't be terminology; there definitely will be. But don't get too wrapped up in the sophisticated words. Wherever necessary, look beyond them to the ideas.

I call the less formal practice herein "Functional-Light Programming" because I think where the formalism of true FP suffers is that it can be quite overwhelming if you're not already accustomed to formal thought. I'm not just guessing; this is my own personal story. Even after teaching FP and writing this book, I can still say that the formalism of terms and notation in FP is very, very difficult for me to process. I've tried, and tried, and I can't seem to get through much of it.

I know many FPers who believe that the formalism itself helps learning. But I think there's clearly a cliff where that only becomes true once you reach a certain comfort with the formalism. If you happen to already have a math background or even some flavors of CS experience, this may come more naturally to you. But some of us don't, and no matter how hard we try, the formalism keeps getting in the way.

So this book introduces the concepts that I believe FP is built on, but comes at it by giving you a boost from below to climb up the cliff wall, rather than condescendingly shouting down at you from the top, prodding you to just figure out how to climb as you go.

## How to Find Balance

If you've been around programming for very long, chances are you've heard the phrase "YAGNI" before: "You Ain't Gonna Need It". This principle primarily comes from extreme programming, and stresses the high risk and cost of building a feature before it's needed.

Sometimes we guess we'll need a feature in the future, build it now believing it'll be easier to do as we build other stuff, then realize we guessed wrong and the feature wasn't needed, or needed to be quite different. Other times we guess right, but build a feature too early, and suck up time from the features that are genuinely needed now; we incur an opportunity cost in diluting our energy.

YAGNI challenges us to remember: even if it's counterintuitive in a situation, we often should postpone building something until it's presently needed. We tend to exaggerate our mental estimates of the future refactoring cost of adding it later when it is needed. Odds are, it won't be as hard to do later as we might assume.

As it applies to functional programming, I would offer this admonition: there will be plenty of interesting and compelling patterns discussed in this text, but just because you find some pattern exciting to apply, it may not necessarily be appropriate to do so in a given part of your code.

This is where I will differ from many formal FPers: just because you *can* apply FP to something doesn't mean you *should* apply FP to it. Moreover, there are many ways to slice a problem, and even though you may have learned a more sophisticated approach that is more "future-proof" to maintenance and extensibility, a simpler FP pattern might be more than sufficient in that spot.

Generally, I'd recommend seeking balance in what you code, and to be conservative in your application of FP concepts as you get the hang of things. Default to the YAGNI principle in deciding if a certain pattern or abstraction will help that part of the code be more readable or if it's just introducing clever sophistication that isn't (yet) warranted.

> Reminder, any extensibility point that’s never used isn’t just wasted effort, it’s likely to also get in your way as well
>
> Jeremy D. Miller @jeremydmiller 2/20/15
>
> https://twitter.com/jeremydmiller/status/568797862441586688

Remember, every single line of code you write has a reader cost associated with it. That reader may be another team member, or even your future self. Neither of those readers will be impressed with overly clever, unnecessary sophistication just to show off your FP prowess.

The best code is the code that is most readable in the future because it strikes exactly the right balance between what it can/should be (idealism) and what it must be (pragmatism).

## Resources

I have drawn on a great many different resources to be able to compose this text. I believe you, too, may benefit from them, so I wanted to take a moment to point them out.

### Books

Some FP/JavaScript books that you should definitely read:

* [Professor Frisby's Mostly Adequate Guide to Functional Programming](https://mostly-adequate.gitbook.io/mostly-adequate-guide/) by [Brian Lonsdorf](https://twitter.com/drboolean)
* [JavaScript Allongé](https://leanpub.com/javascriptallongesix) by [Reg Braithwaite](https://twitter.com/raganwald)
* [Functional JavaScript](http://shop.oreilly.com/product/0636920028857.do) by [Michael Fogus](https://twitter.com/fogus)

### Blogs/sites

Some other authors and content you should check out:

* [Fun Fun Function Videos](https://www.youtube.com/watch?v=BMUiFMZr7vk) by [Mattias P Johansson](https://twitter.com/mpjme)
* [Awesome FP JS](https://github.com/stoeffel/awesome-fp-js)
* [Kris Jenkins](http://blog.jenkster.com/2015/12/what-is-functional-programming.html)
* [Eric Elliott](https://medium.com/@_ericelliott)
* [James A Forbes](https://james-forbes.com/)
* [James Longster](https://github.com/jlongster)
* [André Staltz](http://staltz.com/)
* [Functional Programming Jargon](https://github.com/hemanth/functional-programming-jargon#functional-programming-jargon)
* [Functional Programming Exercises](https://github.com/InceptionCode/Functional-Programming-Exercises)

### Libraries

The code snippets in this book largely do not rely on libraries. Each operation that we discover, we'll derive how to implement it in standalone, plain ol' JavaScript. However, as you begin to build more of your real code with FP, you'll soon want a library to provide optimized and highly reliable versions of these commonly accepted utilities.

By the way, you need to check the documentation for the library functions you use to ensure you know how they work. There will be a lot of similarities in many of them to the code we build on in this text, but there will undoubtedly be some differences, even between popular libraries.

Here are a few popular FP libraries for JavaScript that are a great place to start your exploration with:

* [Ramda](http://ramdajs.com)
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide)
* [functional.js](http://functionaljs.com/)
* [Immutable.js](https://github.com/facebook/immutable-js)

[Appendix C takes a deeper look at these libraries](apC.md/#stuff-to-investigate) and others.

## Summary

You may have a variety of reasons for starting to read this book, and different expectations of what you'll get out of it. This chapter has explained why I want you to read the book and what I want you to get out of the journey. It also helps you articulate to others (like your fellow developers) why they should come on the journey with you!

Functional programming is about writing code that is based on proven principles so we can gain a level of confidence and trust over the code we write and read. We shouldn't be content to write code that we anxiously *hope* works, and then abruptly breathe a sigh of relief when the test suite passes. We should *know* what it will do before we run it, and we should be absolutely confident that we've communicated all these ideas in our code for the benefit of other readers (including our future selves).

This is the heart of Functional-Light JavaScript. The goal is to learn to effectively communicate with our code but not have to suffocate under mountains of notation or terminology to get there.

The journey to learning functional programming starts with deeply understanding the nature of what a function is. That's what we tackle in the next chapter.

----

<a name="footnote-1"><sup>1</sup></a>Buse, Raymond P. L., and Westley R. Weimer. “Learning a Metric for Code Readability.” IEEE Transactions on Software Engineering, IEEE Press, July 2010, dl.acm.org/citation.cfm?id=1850615.
