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

## Perspectiva

A maioria dos outros textos de FP parecem adotar uma abordagem de cima para baixo, mas iremos na direção oposta: trabalhando desde o chão, descobriremos os princípios básicos fundamentais que acredito que os "FPers" formais admitiriam serem a base para tudo que eles fazem. Mas, na maioria das vezes, ficaremos distantes da maior parte da terminologia intimidadora ou da notação matemática que pode facilmente frustar os alunos.

Acredito que é menos importante como você chama algo e mais importante que você entenda o que é e como funciona. Isso não quer dizer que não haja importância na terminologia compartilhada -- ela sem dúvida facilita a comunicação entre profissionais experientes. Mas para o aluno, descobri que iso pode distrair.

Portanto, este livro tentará focar mais nos conceitos básicos e menos nos detalhes sofisticados. Isso não quer dizer que não haverá terminologia; definitivamente haverá. Mas não fique muito envolvido com palavras sofisticadas. Sempre que necessário, olhe além delas para as ideias.

Eu chamo a prática menos formal aqui de "Funcional-Light Programming" porque acho que onde o formalismo do verdadeiro FP sofre é que ele pode ser bastante opressor se você ainda não estiver acostumado com o pensamento formal. Não estou apenas supondo; esta é a minha história pessoal. Mesmo depois de ensionar FP e escrever este livro, ainda posso dizer que o formalismo dos termos e da notação em FP é muito, muito difícil de processar. Eu tentei, e tentei, e não consigo passar muito disso.

Eu conheço muitos "FPers" que acreditam que o próprio formalismo ajuda no aprendizado. Mas acho que há claramente um precipício onde isso só se torna verdade quando você alcança um certo conforto com o formalismo. Se acontecer de você já ter formação em matemática ou até mesmo alguma experiência em CS, isso pode ser mais natural para você. Mas alguns de nós não, e por mais que tentemos, o formalismo continua a atrapalhar.

Portanto, este livro apresenta os conceitos sobre os quais acredito que a FP se baseia, mas aborda isso dando-lhe um impulso de baixo para escalar a parede do penhasco, em vez de gritar condescendentemente com você do topo, incitando-o a descobrir como para subir conforme você avança.

## Como encontrar o equilíbrio

Se você já programa há muito tempo, é provável que já tenha ouvido a frase "YAGNI" antes: "You Ain't Gonna Need It" ("Você não vai precisar disso"). Este princípio vem principalmente da programação extrema e enfatiza o alto risco e custo de construir um recurso antes que ele seja necessário.

Às vezes achamos que precisaremos de um recurso no futuro, construímos ele agora acreditando que será mais fácil de fazer à medida que construímos outras coisas, então percebemos que acreditamos errado e que o recurso não era necessário ou precisava ser bem diferente. Outras vezes, acertamos, mas criamos um recurso muito cedo e perdemos tempo com os recursos que são realmente necessários agora; incorremos num custo de oportunidade ao diluir a nossa energia.

YAGNI desafia-nós a lembrar: mesmo que seja contra-intuitivo numa situação, muitas vezes devemos adiar a construção de algo até que seja necessário no momento. Tendemos a exagerar nossas estimativas mentais do custo futuro de refatoração, adicionando-o mais tarde, quando for necessário. As probabilidades são de que não será tão dificíl de fazer mais tarde como podemos supor.

No que se refere à programação funcional, eu faria esta advertência: haverá muitos padrões interessantes e convinventes discutidos neste texto, mas só porque você acha interessante aplicar algum padrão, pode não ser necessariamente apropriado fazê-lo em uma determinada parte do seu código.

É aqui que vou diferir de muitos "FPers" formais: só porque você *pode* aplicar FP a algo, não significa que você *deve* aplicar FP a isso. Além disso, há muitas maneiras de dividir um problema, e mesmo que você possa ter aprendido uma abordagem mais sofisticada que seja por mais "preparada para o futuro" para manutenção e extensilibidade, um padrão FP mais simples pode ser mais que suficiente nesse caso.

Geralmente, eu recomendo buscar equilíbrio no que você codifica e ser conservador na aplicação dos conceitos de FP à medida que você pega o jeito. O padrão é o princípio YAGNI para decidir se um determinado padrão ou abstração ajudará aquela parte do código a ser mais legível ou se está apenas introduzindo uma sofisticação inteligente que (ainda) não é garantida.

> Lembrete: qualquer ponto de extensibilidade que nunca é usado não é apenas um esforço desperdiçado, é provável que também atrapalhe
>
> Jeremy D. Miller @jeremydmiller 2/20/15
>
> https://twitter.com/jeremydmiller/status/568797862441586688

Lembre-se de que cada linha de código que você escreve tem um custo de leitura associado. Esse leitor pode ser outro membro da equipe ou até mesmo seu futuro eu. Nenhum desses leitores ficará impressionado com a sofisticação excessivamente inteligente e desnecessária apenas para mostar suas proezas em FP.

O melhor código é aquele que será mais legível no futuro porque atinge exatamente o equilíbrio certo entre o que pode/deveria ser (idealismo) e o que deve ser (pragmatismo).

## Recursos

Eu recorri a muitos recurso diferentes para poder compor este texto. Acredito que você também pode se beneficiar com eles, então gostaria de reservar um momento para referenciá-los.

### Livros

Alguns livros JavaScript/FP que você definitivamente deveria ler:

* [Professor Frisby's Mostly Adequate Guide to Functional Programming](https://mostly-adequate.gitbook.io/mostly-adequate-guide/) by [Brian Lonsdorf](https://twitter.com/drboolean)
* [JavaScript Allongé](https://leanpub.com/javascriptallongesix) by [Reg Braithwaite](https://twitter.com/raganwald)
* [Functional JavaScript](http://shop.oreilly.com/product/0636920028857.do) by [Michael Fogus](https://twitter.com/fogus)

### Blogs/sites

Alguns outros autores e conteúdos que você deve conferir:

* [Fun Fun Function Videos](https://www.youtube.com/watch?v=BMUiFMZr7vk) by [Mattias P Johansson](https://twitter.com/mpjme)
* [Awesome FP JS](https://github.com/stoeffel/awesome-fp-js)
* [Kris Jenkins](http://blog.jenkster.com/2015/12/what-is-functional-programming.html)
* [Eric Elliott](https://medium.com/@_ericelliott)
* [James A Forbes](https://james-forbes.com/)
* [James Longster](https://github.com/jlongster)
* [André Staltz](http://staltz.com/)
* [Functional Programming Jargon](https://github.com/hemanth/functional-programming-jargon#functional-programming-jargon)
* [Functional Programming Exercises](https://github.com/InceptionCode/Functional-Programming-Exercises)

### Bibliotecas

Os trechos de código deste livro não dependem em grande parte de bibliotecas. Cada operação que descobrirmos, descobriremos como implementá-la em JavaScript simples e independente. No entanto, à medida que você começa a construir mais código real com FP, em breve você desejará uma biblioteca que forneça versões otimizadas e altamente confiáveis desses utilitários comumente aceitos.

A propósito você precisa verificar a documentação das funções da biblioteca que você usa para garantir que sabe como elas funcionam. Haverá muitas semelhanças em muitos deles com o código que construírmos neste texto, mas sem dúvida haverá algumas diferenças, mesmo entre bibliotecas populares.

Aqui estão algumas bibliotecas FP populares para JavaScript que são um ótimo lugar para começar sua exploração:

* [Ramda](http://ramdajs.com)
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide)
* [functional.js](http://functionaljs.com/)
* [Immutable.js](https://github.com/facebook/immutable-js)

[Appendix C takes a deeper look at these libraries](apC.md/#stuff-to-investigate) e outras.

## Resumo

Você pode ter vários motivos para começar a ler este livro e diferentes expectativas sobre o que obterá com ele. Este capítulo explicou por que quero que você leia o livro e o que quero que você obtenha com a jornada. Também ajuda você a articular com outras pessoas (como seus colegas desenvolvedores) por que eles deveriam vir nessa jornada com você!

A programação funcional trata de escrever código baseado em princípios comprovados para que possamos ganhar um nível de confiança e segurança sobre o código que escrevemos e lemos. Não deveríamos nos contentar em escrever códigos que *esperamos* ansiosamente que funcionem e, então, respirar aliviados quando o conjunto de testes for aprovado. Devemos *saber* o que ele fará antes de executá-lo e devemos estar absolutamente confiantes de que comunicamos todas essas ideias em nosso código para o benefício de outros leitores (incuindo nós mesmos no futuro).

Este é o coração do Functional-Light JavaScript. O objetivo é aprender a se comunicar de forma eficaz com nosso código, mas sem ter que sufocar sob montanhas de notação ou terminologia para chegar lá.

A jornada para aprender programação funcional começa com a compreensão profunda da natureza do que é uma função. É isso que abordaremos no próximo capítulo.

----

<a name="footnote-1"><sup>1</sup></a>Buse, Raymond P. L., and Westley R. Weimer. “Learning a Metric for Code Readability.” IEEE Transactions on Software Engineering, IEEE Press, July 2010, dl.acm.org/citation.cfm?id=1850615.
