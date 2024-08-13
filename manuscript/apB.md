# Funcional-Light JavaScritp

# Apêndice B: A humilde monada
Deixe-me começar este apêndice admitindo: eu não sabia muito sobre o que era um Monoide antes de começar a escrever este apêndice. E cometi muitos erros até conseguir algo sensato. Se você não acredita em mim, dê uma olhada no histórico de commits deste apêndice no [repositório do Github deste livro](https://github.com/getify/Functional-Light-JS)!
Estou incluindo o tópico dos Monoides no livro porque faz parte da jornada que todo desenvolvedor encontrará enquanto aprende Programação Funcional (Functional Programming FP), assim como eu tive ao escrever este livro.
Estamos basicamente terminando este livro com uma breve visão sobre os monoide, enquanto a maioria da literatura sobre FP quase sempre começa com os monoide! Não encontro muita necessidade de pensar explicitamente em termos de monoide na minha programação "Funcional-Light", por isso este material é mais um bônus do que parte do conteúdo principal. Mas isso não significa que os monoides não sejam úteis ou prevalentes -- eles são muito úteis e prevalentes.
Há uma piada no mundo da FP em JavaScript de que praticamente todo mundo tem que escrever seu próprio tutorial ou post de blog sobre o que é um monoide, como se escrever sobre isso fosse um rito de passagem. Ao longo dos anos, os monoides têm sido representados de várias maneiras, como burritos, cebolas e todos os tipos de outras abstrações conceituais malucas. Espero que não haja nada dessas bobagens aqui!
> Um monoide é apenas um monoide na categoria dos endofuntores.
Começamos o prefácio com esta citação, então parece apropriado voltarmos a ela aqui. Mas não, não vamos falar sobre monoides, endofuntores ou teoria das categorias. Essa citação não é apenas condescendente, mas totalmente inútil.
Minha única esperança com esta discussão é que você não tenha mais medo do termo monoide ou do conceito -- eu tive, por anos! -- e que seja capaz de reconhecê-los quando os vir. Você pode até quem sabe, usá-los ocasionalmente.
## Tipo
Há uma enorme área de interesse em FP que basicamente evitamos completamente ao longo deste livro: a teoria dos tipos. Não vou entrar muito fundo na teoria dos tipos, porque, francamente, não sou qualificado para isso. E você não apreciaria mesmo que eu fosse.
Mas o que eu vou dizer é que um monoide é basicamente um tipo de valor.
O número `42` tem um tipo de valor (número!) que traz consigo certas características e capacidades das quais dependemos. A string `"42"` pode parecer muito semelhante, mas tem um propósito diferente em nosso programa.
Na programação orientada a objetos, quando você tem um conjunto de dados (mesmo um único valor discreto) e algum comportamento que deseja agrupar com ele, você cria um objeto/classe para representar esse "tipo". As instâncias são então membros desse tipo. Esta prática é geralmente conhecida como "estruturas de dados".
Vou usar a noção de estruturas de dados de forma muito livre aqui e afirmar que podemos achar útil em um programa definir um conjunto de comportamentos e restrições para um determinado valor e agrupá-los com esse valor em uma única abstração. Dessa forma, ao trabalharmos com um ou mais desses tipos de valores em nosso programa, seus comportamentos vêm de graça, tornando o trabalho com eles mais conveniente. E por conveniente, quero dizer mais declarativo e acessível para o leitor do seu código!
Um monoide é uma estrutura de dados. É um tipo. É um conjunto de comportamentos especificamente projetados para tornar o trabalho com um valor previsível.
Lembre-se de que no [Capítulo 9 falamos sobre functors(ch9.md/#a-word-functors): um valor junto com uma utilidade semelhante a um mapa para realizar uma operação em todos os seus membros constituintes de dados. Um monad é um functor que inclui algum comportamento adicional.
## Interface Flexível
Na verdade, um monoide não é um único tipo de dado, é mais como uma coleção relacionada de tipos de dados. É como uma interface que é implementada de maneira diferente dependendo das necessidades de diferentes valores. Cada implementação é um tipo diferente de monoide.
Por exemplo, você pode ler sobre o "Identity Monad", o "IO Monad", o "Maybe Monad", o "Either Monad" ou uma variedade de outros. Cada um deles tem o comportamento básico do monoide definido, mas estende ou substitui as interações de acordo com os casos de uso para cada tipo diferente de monoide.
É um pouco mais do que uma interface, pois não é apenas a presença de certos métodos de API que torna um objeto um monoide. Há um conjunto específico de garantias sobre as interações desses métodos que é necessário para ser monádico. Esses invariantes bem conhecidos são críticos para o uso dos monoides, melhorando a legibilidade pela familiaridade; caso contrário, é apenas uma estrutura de dados ad hoc que deve ser totalmente lida para ser compreendida pelo leitor.
Na verdade, não há nem mesmo um acordo único e unificado sobre os nomes desses métodos monádicos, da forma que uma verdadeira interface exigiria; um monoide é mais como uma interface flexível. Algumas pessoas chamam um certo método de `bind(..)`, outras de `chain(..)`, outras de `flatMap(..)`, e assim por diante.
Portanto, um monoide é uma estrutura de dados de objeto com métodos suficientes (de praticamente qualquer nome ou tipo) que, no mínimo, satisfaçam os principais requisitos comportamentais da definição de monoide. Cada tipo de monoide tem um tipo diferente de extensão acima do mínimo. Mas, como todos têm uma sobreposição de comportamento, usar dois tipos diferentes de monoides juntos ainda é direto e previsível.
É nesse sentido que os monoides são como uma interface.
## Apenas um Monoide
Um monoide primitivo básico subjacente a muitos outros monoides que você encontrará é chamado Apenas (Just). É *apenas* um simples encapsulador monádico para qualquer valor regular (ou seja, não vazio).
Como um monoide é um tipo, você pode pensar que definiríamos `Just` como uma classe a ser instanciada. Essa é uma maneira válida de fazer isso, mas isso introduz problemas de vinculação de `this` nos métodos que eu não quero lidar; em vez disso, vou seguir com apenas uma abordagem de função simples.
Aqui está uma implementação básica:
```js
function Just(val) {
    return { map, chain, ap, inspect };

    // *********************

    function map(fn) { return Just( fn( val ) ); }

    // aka: bind, flatMap
    function chain(fn) { return fn( val ); }

    function ap(anotherMonad) { return anotherMonad.map( val ); }

    function inspect() {
        return `Just(${ val })`;
    }
}
```
**Nota:** O método `inspeção(..)` está incluído aqui apenas para fins de demonstração. Ele não desempenha nenhum papel direto no sentido monádico.
Você perceberá que qualquer valor `val` que uma instância `Apenas(..)` contenha, nunca é alterado. Todos os métodos monádicos criam novas instâncias de monoides em vez de mutar o próprio valor do monoide.
Não se preocupe se a maior parte disso não fizer sentido até agora. Não vamos nos preocupar muito com os detalhes ou com a matemática/teoria por trás do design do monoide. Em vez disso, vamos nos concentrar mais em ilustrar o que podemos fazer com eles.
### Trabalhando com Métodos de Monoides
Todas as instâncias de monoides terão os métodos `map(..)`, `chain(..)` (também chamado de `bind(..)` ou `flatMap(..)`), e `ap(..)`. O propósito desses métodos e seu comportamento é fornecer uma maneira padronizada para várias instâncias de monoides interagirem entre si.
Vamos olhar primeiro para a função monádica `map(..)`. Como `map(..)` em um array (veja [Capítulo 9](ch9.md/#map)) que chama uma função mapper com seu(s) valor(es) e produz um novo array, o `map(..)` de um monoide chama uma função mapper com o valor do monoide, e o que quer que seja retornado é encapsulado em uma nova instância do monoide Apenas
```js
var A = Just( 10 );
var B = A.map( v => v * 2 );

B.inspect();                // Just(20)
```
O método monádico `chain(..)` faz algo semelhante ao `map(..)`, mas ele meio que desembrulha o valor resultante de seu novo monoide. No entanto, em vez de pensar informalmente sobre "desembrulhar" um monoide, a explicação mais formal seria que `chain(..)` achata o monoide. Considere:
```js
var A = Just( 10 );
var eleven = A.chain( v => v + 1 );

eleven;                     // 11
typeof eleven;              // "number"
```
`onze` é o número primitivo `11` real, não um monoide contendo esse valor.
Para conectar conceitualmente esse método `chain(..)` com coisas que já aprendemos, apontamos que muitas implementações de monoides nomeiam esse método como `flatMap(..)`. Agora, lembre-se do [Capítulo 9 sobre o que `flatMap(..)`](ch9.md/#user-content-flatmap) faz (em comparação com `map(..)`) com um array:
```js
var x = [3];

map( v => [v,v+1], x );         // [[3,4]]
flatMap( v => [v,v+1], x );     // [3,4]
```
Veja a diferença? A função mapper `v => [v,v+1]` resulta em um array `[3,4]`, que acaba na primeira posição única do array externo, então obtemos `[[3,4]]`. Mas `flatMap(..)` achata o array interno no array externo, então obtemos apenas `[3,4]`.
É esse mesmo tipo de coisa que acontece com o `chain(..)` de um monoide (frequentemente referido como `flatMap(..)`). Em vez de obter um monoide contendo o valor como `map(..)` faz, `chain(..)` adicionalmente achata o monoide no valor subjacente. Na verdade, em vez de criar esse monoide intermediário apenas para achatá-lo imediatamente, `chain(..)` é geralmente implementado de maneira mais eficiente para simplesmente tomar um atalho e não criar o monoide em primeiro lugar. De qualquer forma, o resultado final é o mesmo.
Uma maneira de ilustrar `chain(..)` dessa forma é em combinação com a utilidade `identity(..)` (veja [Capítulo 3](ch3.md/#one-on-one)), para extrair efetivamente um valor de um monoide:
```js
var identity = v => v;

A.chain( identity );        // 10
```
`A.chain(..)` chama `identidade(..)` com o valor em `A`, e qualquer valor que `identidade(..)` retorna (`10` neste caso) simplesmente sai sem nenhum monoide intermediário. Em outras palavras, daquele código anterior de `Apenas(..)`, na verdade não precisaríamos incluir aquele auxiliar opcional `inspect(..)`, pois `chain(identidade)` alcança o mesmo objetivo; é puramente para facilitar a depuração enquanto aprendemos sobre monoide.
Neste ponto, espero que tanto `map(..)` quanto `chain(..)` pareçam bastante razoáveis para você.
Por outro lado, o método `ap(..)` de um monoide provavelmente será muito menos intuitivo à primeira vista. Ele parecerá uma estranha contorção de interação, mas há um raciocínio profundo e importante por trás do design. Vamos reservar um momento para explicá-lo.
`ap(..)` pega o valor encapsulado em um monoide e o "aplica" a outro monoide usando o `map(..)` desse outro monoide. Ok, até aqui tudo bem.
No entanto, `map(..)` sempre espera uma função. Isso significa que o monoide em que você chama `ap(..)` precisa realmente conter uma função como seu valor, para passar para o `map(..)` desse outro monoide.
Confuso? Sim, não é o que você poderia esperar. Vamos tentar esclarecer brevemente, mas espere que essas coisas sejam confusas por um tempo até que você tenha tido muito mais exposição e prática com monoides.
Vamos definir `A` como um monoide que contém um valor `10`, e `B` como um monoide que contém o valor `3`:

```js
var A = Just( 10 );
var B = Just( 3 );

A.inspect();                // Just(10)
B.inspect();                // Just(3)
```
Agora, como poderíamos criar um novo monoide onde os valores `10` e `3` foram somados, digamos via uma função `sum(..)`? Acontece que `ap(..)` pode ajudar.
Para usar `ap(..)`, dissemos que primeiro precisamos construir um monoide que contenha uma função. Especificamente, precisamos de um que contenha uma função que por si só mantenha (lembre via fechamento) o valor em `A`. Deixe isso se firmar por um momento.
Para criar um monoide a partir de `A` que contenha uma função com valor, chamamos `A.map(..)`, fornecendo uma função curried que "lembra" aquele valor extraído (veja [Capítulo 3](ch3.md/#one-at-a-time)) como seu primeiro argumento. Chamaremos esse novo monoide contendo função de `C`:
```js
function sum(x,y) { return x + y; }

var C = A.map( curry( sum ) );

C.inspect();
// Just(function curried...)
```
Pense em como isso funciona. A função curried `sum(..)` espera dois valores para fazer seu trabalho, e damos o primeiro desses valores ao ter `A.map(..)` extraindo `10` e passando-o. `C` agora contém a função que lembra `10` via fechamento.
Agora, para obter o segundo valor (`3` dentro de `B`) passado para a função curried esperando em `C`:
```js
var D = C.ap( B );

D.inspect();                // Just(13)
```
O valor `10` saiu de `C`, e `3` saiu de `B`, e `sum(..)` os somou para `13` e encapsulou isso no monoide `D`. Vamos juntar os dois passos para que você possa ver a conexão mais claramente:
```js
var D = A.map( curry( sum ) ).ap( B );

D.inspect();                // Just(13)
```
Para ilustrar com o que `ap(..)` está nos ajudando, poderíamos ter alcançado o mesmo resultado desta maneira:
```js
var D = B.map( A.chain( curry( sum ) ) );

D.inspect();                // Just(13);
```
E isso, claro, é apenas uma composição (veja [Capítulo 4](ch4.md)):
```js
var D = compose( B.map, A.chain, curry )( sum );

D.inspect();                // Just(13)
```
Legal, né!?
Se o *como* dessa discussão sobre métodos de monoides ainda não estiver claro, volte e releia. Se o *porquê* estiver elusivo, apenas persista. Monoides confundem facilmente os desenvolvedores, é *apenas* assim que é!
## Talvez
É muito comum no material de FP cobrir monoides bem conhecidas como Talvez (Maybe). Na verdade, o monoide Talvez (Maybe) é um emparelhamento particular de dois outros monoides mais simples: Apenas (Just) e Nada (Nothing).
Já vimos Apenas (Just); Nada (Nothing) é um monoide que contém um valor vazio. Talvez (Maybe) é um monoide que contém um apenas (Just) ou um Nada (Nothing).
Aqui está uma implementação mínima de Talvez (Maybe):
```js
var Maybe = { Just, Nothing, of/* aka: unit, pure */: Just };

function Just(val) { /* .. */ }
```javascript
function Nothing() {
    return { map: Nothing, chain: Nothing, ap: Nothing, inspect };
// *********************

    function inspect() {
        return "Nothing";
    }
}
```
**Nota:** `Maybe.of(..)` (às vezes referido como `unit(..)` ou `pure(..)`) é um alias de conveniência para `Just(..)`.
Em contraste com instâncias de `Just()`, instâncias de `Nothing()` têm definições no-op para todos os métodos monádicos. Então, se uma instância desse monoide aparece em qualquer operação monádica, ela basicamente tem o efeito de encurtar a execução para não realizar nenhuma ação. Note que aqui não há imposição do que "vazio" significa - seu código decide isso. Mais sobre isso depois.
No Maybe, se um valor não é vazio, é representado por uma instância de `Just(..)`; se é vazio, é representado por uma instância de `Nothing()`.
Mas a importância desse tipo de representação monádica é que, seja uma instância de `Just(..)` ou de `Nothing()`, usaremos os métodos da API da mesma maneira.
O poder da abstração Maybe é encapsular implicitamente essa dualidade de comportamento/no-op.
### Diferentes Maybes
Muitas implementações de um monoide Maybe em JavaScript incluem uma verificação (geralmente em `map(..)`) para ver se o valor é `null`/`undefined`, e pulam o comportamento se for. Na verdade, o Maybe é valorizado precisamente porque ele meio que automaticamente interrompe seu comportamento com a verificação de valor vazio encapsulada.
Veja como o Maybe é geralmente ilustrado:
```js
// em vez de usar o inseguro `console.log( someObj.something.else.entirely)`:
Maybe.of( someObj )
.map( prop( "something" ) )
.map( prop( "else" ) )
.map( prop( "entirely" ) )
.map( console.log );
```
Em outras palavras, se em qualquer ponto da cadeia obtivermos um valor `null`/`undefined`, o Maybe magicamente muda para o modo no-op - agora é uma instância de `Nothing()`! - e para de fazer qualquer coisa pelo resto da cadeia. Isso torna o acesso a propriedades aninhadas seguro contra a geração de exceções JS se alguma propriedade estiver ausente/vazia. Isso é legal, e uma abstração certamente útil!
Mas... ***essa abordagem ao Maybe não é um monoide pura.***
O espírito central de uma Monoide diz que ela deve ser válida para todos os valores e não pode fazer nenhuma inspeção do valor, de jeito nenhum — nem mesmo uma verificação de nulo. Portanto, essas outras implementações estão cortando caminhos por conveniência. Não é um grande problema, mas quando se trata de aprender algo, você deve aprender na sua forma mais pura antes de começar a flexibilizar as regras.
A implementação anterior do monoide Maybe que forneci difere de outras Maybe principalmente por não ter a verificação de vazio. Além disso, apresentamos `Maybe` apenas como uma combinação solta de `Just(..)`/`Nothing()`.
Então espere. Se não obtemos o encurtamento automático, por que o Maybe é útil? Parece que esse é ponto.
Não tema! Podemos simplesmente fornecer a verificação de vazio externamente, e o resto do comportamento de encurtamento da monoide Maybe funcionará perfeitamente. Veja como você poderia fazer o acesso a propriedades aninhadas (`algumObj.algo.a.mais.totalmente `) de antes, mas de forma mais "correta":
```js
function isEmpty(val) {
    return val === null || val === undefined;
}

var safeProp = curry( function safeProp(prop,obj){
    if (isEmpty( obj[prop] )) return Maybe.Nothing();
    return Maybe.of( obj[prop] );
} );

Maybe.of( someObj )
.chain( safeProp( "something" ) )
.chain( safeProp( "else" ) )
.chain( safeProp( "entirely" ) )
.map( console.log );
```
Criamos um `safeProp(..)` que faz a verificação de vazio e seleciona uma instância de monoide `Nothing()` se for o caso, ou envolve o valor em uma instância de `Just(..)` (via `Maybe.of(..)`). Então, em vez de `map(..)`, usamos `chain(..)` que sabe como "desembrulhar" a monoide que `safeProp(..)` retorna.
Obtemos o mesmo encurtamento de cadeia ao encontrar um valor vazio. Só não incorporamos essa lógica no Maybe.
O benefício do monoide, e do Maybe especificamente, é que nossos métodos `map(..)` e `chain(..)` têm uma interação consistente e previsível, independentemente de qual tipo de monoide retorna. Isso é bem legal!
## Humilde
Agora que entendemos um pouco mais sobre Maybe e o que ele faz, vou dar uma pequena reviravolta — e adicionar um pouco de humor autodepreciativo à nossa discussão — ao inventar o monoide MaybeHumble (TalvezHumilde). Tecnicamente, `MaybeHumble(..)` não é uma monoide em si, mas uma função fábrica que produz uma instância do monoide Maybe.
Humilde (Humble) é uma estrutura de dados assumidamente artificial que usa Maybe para rastrear o status de um número `nivelEgo`. Especificamente, as instâncias do monoide produzidas por `MaybeHumilde(..)` só operam afirmativamente se o valor do seu nível de ego for baixo o suficiente (menor que `42`!) para ser considerado humilde; caso contrário, é um `Nothing()` sem ação. Isso deve soar muito parecido com Maybe; é bem similar!
Aqui está a função fábrica para nosso monoide Maybe+Humilde:
```js
function MaybeHumble(egoLevel) {
    // accept anything other than a number that's 42 or higher
    return !(Number( egoLevel ) >= 42) ?
        Maybe.of( egoLevel ) :
        Maybe.Nothing();
}
```
Você notará que esta função fábrica é meio parecida com `safeProp(..)`, na medida em que usa uma condição para decidir se deve escolher a parte `Just(..)` ou `Nothing()` do Maybe.

```js
function winAward(ego) {
    return MaybeHumble( ego + 3 );
}

alice = alice.chain( winAward );
alice.inspect();            // Nothing
```

A chamada `MaybeHumilde(39 + 3)` cria uma instância do monoide `Nothing()` para retornar da chamada `chain(..)`, então agora Alice não se qualifica mais como humilde.
Agora, vamos usar alguns monoide juntos:
```js
var bob = MaybeHumble( 41 );
var alice = MaybeHumble( 39 );

var teamMembers = curry( function teamMembers(ego1,ego2){
    console.log( `Our humble team's egos: ${ego1} ${ego2}` );
} );

bob.map( teamMembers ).ap( alice );
// Our humble team's egos: 41 39
```

bob.map(teamMembers).ap(alice);
// O ego do nosso humilde time: 41 39
```
Lembrando do uso de `ap(..)` de antes, agora podemos explicar como esse código funciona.
Como `teamMembers(..)` é curryed, a chamada `bob.map(..)` passa o nível de ego de `bob` (`41`) e cria uma instância de monoide com a função restante encapsulada. Chamar `ap(alice)` *naquele* monoide chama `alice.map(..)` e passa a função do monoide para ela. O efeito é que os valores numéricos dos monoides `bob` e `alice` foram fornecidos para a função `teamMembers(..)`, exibindo a mensagem mostrada.
No entanto, se um ou ambos os monoides forem instâncias `Nothing()` (porque o nível de ego deles era muito alto):

```js
var frank = MaybeHumble( 45 );

bob.map( teamMembers ).ap( frank );
// ..no output..

frank.map( teamMembers ).ap( bob );
// ..no output..
```
`teamMembers(..)` nunca é chamada (e nenhuma mensagem é exibida) porque `frank` é uma instância `Nothing()`. Esse é o poder do monoide Maybe, e nossa fábrica `MaybeHumble(..)` nos permite selecionar com base no nível de ego. Legal!
### Humildade
Mais um exemplo para ilustrar os comportamentos de nossa estrutura de dados Maybe+Humble:
```js
function introduction() {
    console.log( "I'm just a learner like you! :)" );
}

var egoChange = curry( function egoChange(amount,concept,egoLevel) {
    console.log( `${amount > 0 ? "Learned" : "Shared"} ${concept}.` );
    return MaybeHumble( egoLevel + amount );
} );

var learn = egoChange( 3 );

var learner = MaybeHumble( 35 );

learner
.chain( learn( "closures" ) )
.chain( learn( "side effects" ) )
.chain( learn( "recursion" ) )
.chain( learn( "map/reduce" ) )
.map( introduction );
// Learned closures.
// Learned side effects.
// Learned recursion.
// ..nothing else..
```

Infelizmente, o processo de aprendizado parece ter sido interrompido. Veja, eu descobri que aprender um monte de coisas sem compartilhar com os outros inflaciona muito o seu ego e não é bom para suas habilidades.
Vamos tentar uma abordagem melhor para o aprendizado:
```js
var share = egoChange( -2 );

learner
.chain( learn( "closures" ) )
.chain( share( "closures" ) )
.chain( learn( "side effects" ) )
.chain( share( "side effects" ) )
.chain( learn( "recursion" ) )
.chain( share( "recursion" ) )
.chain( learn( "map/reduce" ) )
.chain( share( "map/reduce" ) )
.map( introduction );
// Learned closures.
// Shared closures.
// Learned side effects.
// Shared side effects.
// Learned recursion.
// Shared recursion.
// Learned map/reduce.
// Shared map/reduce.
// I'm just a learner like you! :)
```
Compartilhar enquanto você aprende. Essa é a melhor maneira de aprender mais e aprender melhor.
## Resumo
O que é uma monoide, afinal? Um monoide é um tipo de valor, uma interface, uma estrutura de dados com comportamentos encapsulados.
Mas nenhuma dessas definições é particularmente útil. Aqui está uma tentativa de algo melhor: **um monoide é como você organiza comportamentos em torno de um valor de uma forma mais declarativa.**
Como com tudo mais neste livro, use monoides onde eles são úteis, mas não os use apenas porque todo mundo fala sobre elas em FP. Monoides não são uma solução universal, mas oferecem alguma utilidade quando usadas de forma conservadora.





