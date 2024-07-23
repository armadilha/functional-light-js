# Functional-Light JavaScript
# Capítulo 4: Compondo funções

Por hora, espero que você esteja se sentindo bem mais a vontade com o significado do uso das funções para programação funcional.

Um programador ou uma programadora funcionais vêem toda função do seu programa como uma simples peça de lego. Eles reconhecem a peça azul 2x2 em um bloco à primeira vista e sabem exatamente como funciona e o que pode ser feito.

Mas as vezes você pega a peça azul 2x2, a cinza 4x1 e as coloca juntas em um formato específico, e aí percebe, "essa é uma peça que pode ser usada com frequência".

E agora combinando duas peças, você criou uma nova "peça", e sempre que precisar poderá usá-la. É mais fácil usar essa peça composta por cinza e azul onde necessário, do que usá-las separadamente a cada vez.

As funções existem de vários formatos e tamanho. E nós podemos definir uma determinada combinação para formar uma função composta que irá ser útil em várias partes do programa.
Este processo de utilização conjunta de funções é chamado de composição.

Composição é como um FPer modela o fluxo de dados através do programa. De alguma forma, este é o conceito mais fundamental em todo o FP, porque sem ele, você não pode modelar dados declarativamente e nem alterar estado. Em outras palavras, todo o resto em FP entraria em colapso sem composição.

## Saída para entrada

Nós já vimos alguns poucos exemplos de composição. Por exemplo, nossa discussão de [`unary(..)` no capítulo 3](ch3.md/#user-content-unary) incluiu esta expressão: [`[..].map(unary(parseInt))`](ch3.md/#user-content-mapunary). Reflita sobre o que acontece aqui.

Para compor duas funções juntas, passe a saída da primeira chamada de função como a entrada da segunda chamada de função. Em `map(unary(parseInt))`, o `unary(parseInt)` chamado retorna um valor (uma função); que o valor é diretamente passado como argumento para `map(..)`, que retorna um array.
Para dar um passo atrás e visualizar o fluxo conceitual de dados, considere:

```txt
arrayValue <-- map <-- unary <-- parseInt
```

`parseInt` é a entrada para `unary(..)`. a saída de `unary(..)` é a entrada para `map(..)`. A saída de `map(..)` é `arrayValue`. Isto é a composição de `map(..)` e `unary(..)`.

**Observação:** A orientação da direita para a esquerda aqui é proposital, embora possa parecer estranha neste ponto do seu aprendizado. Voltaremos para explicar isso mais detalhadamente mais tarde.

Pense neste fluxo de dados como uma esteira em uma loja de doces, onde cada operação é um passo no processo de resfriamento, corte e embalagem dos doces. Iremos usar a metáfora da loja de doces ao longo deste capítulo para explicar o que é composição.

<p align="center">
    <img src="images/fig2.png">
</p>

Vamos examinar a composição em ação um passo de cada vez. Considere estas duas utilidades que você pode ter no seu programa:

```js
function words(str) {
    return String( str )
        .toLowerCase()
        .split( /\s|\b/ )
        .filter( function alpha(v){
            return /^[\w]+$/.test( v );
        } );
}

function unique(list) {
    var uniqList = [];

    for (let v of list) {
        // valor ainda não está na nova lista?
        if (uniqList.indexOf( v ) === -1 ) {
            uniqList.push( v );
        }
    }

    return uniqList;
}
```

`words(..)` divide uma sequência dentro de um array de palavras. `unique(..)` pega uma lista de palavras e filtra para não ter nenhuma palavra repetida.           
Para usar essas duas opções para analisar uma sequência de texto:

```js
var text = "Para compor duas funções juntas, passe a \
saída da primeira chamada de função como a entrada da \
segunda chamada de função.";


var wordsFound = words( text );
var wordsUsed = unique( wordsFound );

wordsUsed;
// ["to","compose","two","functions","together","pass",
// "the","output","of","first","function","call","as",
// "input","second"]
```

Chamamos a saída do array de `words(..)`como `wordsFound`. A entrada de `unique(..)` que é também um array, para que possamos passar o `wordsFound` pra ele.


De volta à linha de produção da fábrica de doces: a primeira máquina tem como "entrada"(Input) o chocolate derretido, e a "saída"(Output) é um pedaço de chocolate formado e resfriado. A próxima máquina um pouco abaixo da linha de produção tem como "entrada"(Input) o pedaço de chocolate formado e resfriado, e a "saída"(Output) é um pedaço recortado de doce de chocolate. A seguir, uma máquina na linha pega pequenos pedaços de doce de chocolate da esteira e embrulha para saírem prontos para encaixotar e transportar.

<img src="images/fig3.png" align="right" width="9%" hspace="20">

A fábrica de doces tem muito sucesso nesse processo, mas como em todo negócio, a chefia continua buscando por maneiras de crescer.


Para aumentar a demanda da produção de doces, eles decidem retirar a esteira e simplesmente empilhar as três máquinas umas sobre as outras, de modo que a válvula de saída de uma seja conectada diretamente à válvula de entrada da outra abaixo dela. Não há mais espaço desperdiçado onde um pedaço de chocolate desce lenta e ruidosamente por uma esteira rolante da primeira para a segunda máquina.

Esta inovação economiza um grande espaço na fábrica, e então a chefia está feliz da vida porque irá conseguir fazer mais doces por dia!


O código equivalente a esta nova configuração da fábrica de doces precisa pular a etapa intermediária (a variável `wordsFound` no trecho anterior) e apenas usar as duas chamadas de função juntas:

```js
var wordsUsed = unique( words( text ) );
```

**Observação:** Apesar de escrevermos uma chamada de função da esquerda para a direita -- `unique(..)` e então `words(..)` -- a ordem das operações serão na verdade mais da direita para a esquerda, ou de dentro para fora. `words(..)` irá rodar primeiro e depois `unique(..)`. Mais tarde falaremos sobre um padrão que combina a ordem de execução com a nossa leitura natural da esquerda para a direita, chamado `pipe(..)`.

As máquinas empilhadas estão funcionando bem, mas é meio desorganizado termos os fios pendurados por todo lado. Quanto mais máquinas empilhadas eles criarem, mais desordenado fica o andar da fábrica. E o esforço para montar e manter todas essas pilhas de máquinas consome muito tempo.

<img src="images/fig4.png" align="left" width="15%" hspace="20">

Em uma manhã, uma engenheira da fábrica de doces tem uma excelente idéia. Ela pensa que será bem mais eficiente se ela fizer uma caixa externa para acomodar os cabos; por dentro, todas as três máquinas estão interligadas e, por fora, tudo está limpo e arrumado. No topo desta nova máquina sofisticada há uma válvula para despejar o chocolate derretido e na parte inferior há uma válvula que cospe bombons de chocolate embrulhados. Brilhante!

Esta máquina de composto único é muito mais fácil de transportar e instalar em qualquer lugar da fábrica. Os trabalhadores deste piso da fábrica estãoainda mais contentes pois já não precisam operar três máquinas diferentes individualmente; eles preferem usar uma única máquina mais sofisticada.

De volta ao código: nós percebemos que a combinação de `words(..)` e `unique(..)` nesta ordem de execução específica (pense: lego composto) é algo que poderíamos usar em várias outras partes da nossa aplicação. Então, vamos definir uma função composta que os combine:

```js
function uniqueWords(str) {
    return unique( words( str ) );
}
```

`uniqueWords(..)` pega uma string e retorna um array. Isto é uma composição de duas funções: `unique(..)` e `words(..)`; isto cria este fluxo de dados:

```txt
wordsUsed <-- unique <-- words <-- text
```

Provavelmente agora você já tenha reconhecido isso: a revolução que se desenrola no design de fábricas de doces é a composição de funções.

### Criando as máquinas

A fábrica de doces está funcionando muito bem e, graças a todo o espaço economizado, eles agora têm muito espaço para experimentar fazer novos tipos de doces. Com base no sucesso anterior, a administração está empenhada em continuar a inventar novas máquinas sofisticadas de compostos para a sua crescente variedade de doces.

Mas a batalha dos engenheiros da fábrica continua, porque cada vez que uma nova máquina composta sofisticada precisa ser criada, eles gastam bastante tempo fazendo uma nova caixa externa e encaixando as máquinas individuais nela.

Por isso os engenheiros contactaram um fornecedor de máquinas industriais para os ajudar. Eles ficaram surpresos quando descobriram que este fornecedor possui uma máquina **fazedora de máquinas**! Por mais incrível que pareça, eles compraram uma máquina que pode acomodar algumas das máquinas menores da fábrica -- a do resfriamento do chocolate e a do corte, por exemplo -- e conectá-las automaticamente, mesmo alocando-os em uma caixa maior, bonita e limpa. Isso certamente fará com que a fábrica de doces realmente decole!

<p align="center">
    <img src="images/fig5.png" width="50%">
</p>

De volta ao código, vamos considerar um utilitário chamado `compose2(..)` que cria uma composição de duas funções automaticamente, exatamente da mesma forma que fizemos manualmente:

```js
function compose2(fn2,fn1) {
    return function composed(origValue){
        return fn2( fn1( origValue ) );
    };
}

// or the ES6 => form
var compose2 =
    (fn2,fn1) =>
        origValue =>
            fn2( fn1( origValue ) );
```

Você percebeu que nós definimos a ordem dos parâmetros com `fn2, fn1`, e além disso é a segunda função listada (`fn1`) que roda primeiro e depois a primeira função listada (`fn2`)? Em outras palavras, a função se compõe da direita para a esquerda.

Isso pode parecer uma escolha estranha, mas existem alguns motivos pra isso. A maioria das bibliotecas FP típicas definem seus `compose(..)` trabalhar da direita para a esquerda em termos de ordenação, então vamos manter essa convenção.

Mas porquê? Penso que a explicação mais fácil (mas talves não a mais historicamente precisa) é que os estamos listando para corresponder à ordem em que são escritos manualmente no código, ou mais precisamente, a ordem em que os encontramos ao ler da esquerda para a direita.

`unique(words(str))` lista as funções da esquerda pra direita `unique, words`, então nós fazemos com que o nosso utilitário `compose(..)` os aceitem nesta ordem também. A ordem de execução é da direita para a esquerda, mas a ordem do código é da esquerda para a direita. Preste muita atenção nisso para mantê-los distintos na sua mente.

Agora, a definição mais eficiente da máquina de fazer doces é:

```js
var uniqueWords = compose2( unique, words );
```

### Variação de composição

Pode parecer que a combinação `<-- unique <-- words` é a única ordem em que estas duas funções podem ser compostas. Mas poderíamos na verdade, compor isto na ordem oposta para criar um utilitário com um propósito um pouco diferente:

```js
var letters = compose2( words, unique );

var chars = letters( "Como vai você, Henry?" );
chars;
// ["h","o","w","a","r","e","y","u","n"]
```

Isto funciona porque o utilitário `words(..)`, por uma questão de segurança do tipo valor, primeiro força sua entrada para uma string usando `String(..)`. Então o array que `unique(..)` retorna -- agora a entrada para `words(..)` -- se torna a string `"H,o,w, ,a,r,e,y,u,n ,?"`, e então o resto do comportamento em `words(..)` processa essa string no array `chars`.

É certo que este é um exemplo inventado. Mas a questão é que as composições de funções nem sempre são unidirecionais. Às vezes colocamos o bloco do Lego cinza em cima do bloco azul e às vezes colocamos o azul por cima.

É melhor a fábrica de doces ter cuidado se tentarem colocar os doces embrulhados na máquina que mistura e resfriamento do chocolate!


## Composição geral

Se definirmos a composição de duas funções, podemos continuar apoiando a composição de qualquer número de funções. O fluxo geral de visualização de dados para qualquer número de funções que estão sendo compostas é assim:

```txt
finalValue <-- func1 <-- func2 <-- ... <-- funcN <-- origValue
```

<p align="center">
    <img src="images/fig6.png" width="50%">
</p>

Agora a fábrica de doces possui a melhor máquina de todas: uma máquina que pode pegar qualquer número de máquinas menores separada e produzir uma máquina grande e sofisticada que executa cada etapa em ordem.
Essa é uma operação incrível de doces! É o sonho de Willy Wonka!

Podemos implementar um utilitário `compose(..)` como este:

<a name="Composição geral"></a>

```js
function compose(...fns) {
    return function composed(result){
        // copiar o array das funções
        var list = [...fns];

        while (list.length > 0) {
            // pegar a última função do final da lista
            // e executar
            result = list.pop()( result );
        }

        return result;
    };
}

// or the ES6 => form
var compose =
    (...fns) =>
        result => {
            var list = [...fns];

            while (list.length > 0) {
                // pegar a última função do final da lista
                // e executar
                result = list.pop()( result );
            }

            return result;
        };
```

**Aviso:** `fns` é um array coletado de argumentos, não um array passado e, como tal, é local para `compose(..)`. Pode ser tentador pensar que `[...fns]` seria, portanto, desnecessário. Entretanto, nessa implementação em particular, `.pop()` dentro da função interna `composed(..)` está alterando a lista, então se não fizermos uma cópia de cada vez, a função composta retornada só poderá ser usada de forma confiável uma vez. Iremos abordar esse perigo mais uma vez no [Capítulo 6](ch6.md/#user-content-hiddenmutation).

Agora vamos olhar para um exemplo composto por mais de duas funções. Recordando nosso exemplo de composição `uniqueWords(..)`, adicionamos um `skipShortWords(..)` à mistura:

```js
function skipShortWords(words) {
    var filteredWords = [];

    for (let word of words) {
        if (word.length > 4) {
            filteredWords.push( word );
        }
    }

    return filteredWords;
}
```

Vamos definir `biggerWords(..)` que inclui `skipShortWords(..)`. A composição manual equivalente é `skipShortWords( unique( words( text ) ) )`, então vamos fazer isso com `compose(..)`:

```js
var text = "Para compor duas funções juntas, passe a \
saída da primeira chamada de função como a entrada da \
segunda chamada de função.";

var biggerWords = compose( skipShortWords, unique, words );

var wordsUsed = biggerWords( text );

wordsUsed;
// ["compose","functions","together","output","first",
// "function","input","second"]
```

Para fazer alguma coisa mais interessante com a composição, vamos usar [`partialRight(..)`, que vimos pela primeira vez no Capítulo 3](ch3.md/#user-content-partialright). Podemos construir uma aplicação parcial à direita do próprio `compose(..)`, pré-especificando o segundo e terceiro argumentos (`unique(..)` e `words(..)`, respectivamente); vamos chamá-lo de `filterWords(..)`.

Então, podemos completar a composição várias vezes chamando `filterWords(..)`, mas com diferentes primeiros argumentos respectivamente:

```js
// Nota: usar um check `<= 4` ao invés de `> 4`
// este `skipShortWords(..)` usa
function skipLongWords(list) { /* .. */ }

var filterWords = partialRight( compose, unique, words );

var biggerWords = filterWords( skipShortWords );
var shorterWords = filterWords( skipLongWords );

biggerWords( text );
// ["compose","functions","together","output","first",
// "function","input","second"]

shorterWords( text );
// ["to","two","pass","the","of","call","as"]
```

Tire um momento para considerar o que a aplicação parcial à direita `compose(..)` nos dá. Ela nos permite especificar antecipadamente a(s) primeira(s) etapa(s) de uma composição e, em seguida, criar variações especializadas dessa composição com diferentes etapas subsequentes (`biggerWords(..)` e `shorterWords(..)`). Esse é um dos truques mais poderosos do FP!

Você pode também `curry(..)` uma composição ao invés da aplicação parcial, embora por causa da ordem da direita para a esquerda, você possa querer `curry( reverseArgs(compose), ..)` com mais frequência em vez de apenas `curry ( compor, ..)` em si.


**Observação:** Porque `curry(..)` (pelo menos [a maneira como o implementamos no Capítulo 3](ch3.md/#user-content-curry)) depende da detecção da aridade (`length` ) ou tê-lo especificado manualmente, e `compose(..)` é uma função variada, você precisará especificar manualmente a aridade pretendida como `curry(.. , 3)`.


### Implementações Alternativas

Embora você possa nunca implementar seu próprio `compose(..)` para usar na produção, e sim apenas usar a implementação de uma biblioteca conforme fornecida, descobri que entender como funciona nos bastidores, na verdade ajuda a solidificar conceitos gerais de FP muito bem.

Então, vamos examinar algumas opções diferentes de implementação para `compose(..)`. Veremos também que há alguns prós/contras em cada implementação, especialmente no desempenho.

Veremos o utilitário [`reduce(..)` em detalhes no Capítulo 9](ch9.md/#reduce), mas por enquanto, saiba apenas que ele reduz uma lista (array) a um único valor finito . É como um loop sofisticado.

Por exemplo, se você fizesse um addition-reduction em uma lista de números (como `[1,2,3,4,5,6]`), você faria um loop sobre eles, adicionando-os conforme avança. A redução adicionaria `1` a `2`, e adicionaria esse resultado a `3`, e então adicionaria esse resultado a `4`, e assim por diante, resultando na soma final: `21`.

A versão original de `compose(..)` usa um loop e calcula ansiosamente (também conhecido como, imediatamente) o resultado de uma chamada para passar para a próxima chamada. Esta é uma redução de uma lista de funções, então podemos fazer a mesma coisa com `reduce(..)`:

<a name="composereduce"></a>

```js
function compose(...fns) {
    return function composed(result){
        return [...fns].reverse().reduce( function reducer(result,fn){
            return fn( result );
        }, result );
    };
}

// or the ES6 => form
var compose = (...fns) =>
    result =>
        [...fns].reverse().reduce(
            (result,fn) =>
                fn( result )
            , result
        );
```

**Observação:** Esta implementação de `compose(..)` usa `[...fns].reverse().reduce(..)` para reduzir da direita para a esquerda. Iremos [revisitar `compose(..)` no Capítulo 9](ch9.md/#user-content-composereduceright), em vez de usar `reduceRight(..)` para esse propósito.

Observe que o loop `reduce(..)` acontece cada vez que a função `composed(..)` final é executada, e que cada `result(..)` intermediário é passado para a próxima iteração como a entrada para o próxima chamada.

A vantagem desta implementação é que o código é mais conciso e também utiliza uma construção FP bem conhecida: `reduce(..)`. E o desempenho desta implementação também é semelhante ao da versão original do loop `for`.

No entanto, esta implementação é limitada porque a função composta externa (também conhecida como a primeira função na composição) só pode receber um único argumento. A maioria das outras implementações passa todos os argumentos para a primeira chamada. Se todas as funções na composição forem unárias, isso não é grande coisa. Mas se você precisar passar vários argumentos para a primeira chamada, você vai preferir uma implementação diferente.

Para corrigir a limitação de argumento único da primeira chamada, ainda podemos usar `reduce(..)`, mas produzir um encapsulamento de função de avaliação lenta:

```js
function compose(...fns) {
    return fns.reverse().reduce( function reducer(fn1,fn2){
        return function composed(...args){
            return fn2( fn1( ...args ) );
        };
    } );
}

// or the ES6 => form
var compose =
    (...fns) =>
        fns.reverse().reduce( (fn1,fn2) =>
            (...args) =>
                fn2( fn1( ...args ) )
        );
```

Observe que retornamos o resultado da chamada `reduce(..)` diretamente, que é em si uma função, não um resultado computado. *Essa* função nos permite passar quantos argumentos quisermos, passando-os todos ao longo da linha para a primeira chamada de função na composição e, em seguida, aumentando cada resultado em cada chamada subsequente.

Em vez de calcular o resultado da execução e transmiti-lo à medida que o loop `reduce(..)` prossegue, esta implementação executa o loop `reduce(..)` **uma vez** antecipadamente no momento da composição e adia toda a função cálculos de chamada - chamados de cálculo lento. Cada resultado parcial da redução é uma função sucessivamente mais encapsulada.

Quando você chama a função composta final e fornece um ou mais argumentos, todos os níveis da grande função aninhada, da chamada mais interna à mais externa, são executados em sucessão inversa (não por meio de um loop).

As características de desempenho serão potencialmente diferentes das da implementação anterior baseada em `reduce(..)`. Aqui, `reduce(..)` é executado apenas uma vez para produzir uma grande função composta, e então esta chamada de função composta simplesmente executa todas as suas funções aninhadas a cada chamada. Na versão anterior, `reduce(..)` seria executado para cada chamada.

Sua milhagem pode variar dependendo de qual implementação é melhor, mas tenha em mente que esta última implementação não é limitada na contagem de argumentos como a anterior.

Também poderíamos definir `compose(..)` usando recursão. A definição recursiva para `compose(fn1,fn2, .. fnN)` seria semelhante a:

```txt
compose( compose(fn1,fn2, .. fnN-1), fnN );
```

**Observação:** Abordaremos a recursão mais detalhadamente no [Capítulo 8](ch8.md), portanto, se essa abordagem parecer confusa, não se preocupe por enquanto. Ou leia aquele capítulo e depois volte e releia esta nota. :)

Aqui está como implementamos `compose(..)` com recursão:

```js
function compose(...fns) {
    // pull off the last two arguments
    var [ fn1, fn2, ...rest ] = fns.reverse();

    var composedFn = function composed(...args){
        return fn2( fn1( ...args ) );
    };

    if (rest.length == 0) return composedFn;

    return compose( ...rest.reverse(), composedFn );
}

// or the ES6 => form
var compose =
    (...fns) => {
        // retira os dois últimos argumentos
        var [ fn1, fn2, ...rest ] = fns.reverse();

        var composedFn =
            (...args) =>
                fn2( fn1( ...args ) );

        if (rest.length == 0) return composedFn;

        return compose( ...rest.reverse(), composedFn );
    };
```

Acho que o benefício de uma implementação recursiva é principalmente conceitual. Pessoalmente, acho muito mais fácil pensar em uma ação repetitiva em termos recursivos, em vez de em um loop onde preciso rastrear o resultado da execução, por isso prefiro que o código o expresse dessa forma.

Outros acharão a abordagem recursiva um pouco mais difícil de fazer malabarismos mentais. Convido você a fazer suas próprias avaliações.

## Composição Reordenada

Falamos anteriormente sobre a ordenação da direita para a esquerda das implementações padrão `compose(..)`. A vantagem está em listar os argumentos (funções) na mesma ordem em que apareceriam se a composição fosse feita manualmente.

A desvantagem é que eles estão listados na ordem inversa em que são executados, o que pode ser confuso. Também foi mais estranho ter que usar `partialRight(compose, ..)` para pré-especificar a(s) *primeira(s) função(ões) a serem executadas na composição.

A ordem inversa, composta da esquerda para a direita, tem um nome comum: `pipe(..)`. Diz-se que esse nome vem da terra Unix/Linux, onde vários programas são encadeados "pipe" (operador `|`) a saída do primeiro como a entrada do segundo, e assim por diante (ou seja, `ls -la | grep "foo" | less`)

`pipe(..)` é idêntico a `compose(..)` exceto que processa a lista de funções na ordem da esquerda para a direita:

```js
function pipe(...fns) {
    return function piped(result){
        var list = [...fns];

        while (list.length > 0) {
            // take the first function from the list
            // and execute it
            result = list.shift()( result );
        }

        return result;
    };
}
```

Na verdade, poderíamos apenas definir `pipe(..)` como a reversão de argumentos de `compose(..)`:

```js
var pipe = reverseArgs( compose );
```

Essa foi fácil!

Lembre-se deste exemplo da composição geral anterior:

```js
var biggerWords = compose( skipShortWords, unique, words );
```

Para expressar isso com `pipe(..)`, apenas invertemos a ordem em que os listamos:

```js
var biggerWords = pipe( words, unique, skipShortWords );
```

A vantagem de `pipe(..)` é que ele lista as funções em ordem de execução, o que às vezes pode reduzir a confusão do leitor. Pode ser mais simples ler o código: `pipe(words, unique, skipShortWords )`, e reconhecer que ele está executando `words(..)` primeiro, depois `unique(..)`, e finalmente `skipShortWords(.. )`.

`pipe(..)` também é útil se você estiver em uma situação em que deseja aplicar parcialmente a(s) *primeira(s) função(ões) executada(s). Anteriormente fizemos isso com a aplicação parcial à direita de `compose(..)`.

Compare:

```js
var filterWords = partialRight( compose, unique, words );

// vs

var filterWords = partial( pipe, words, unique );
```

Como você deve se lembrar de nossa primeira implementação de [`partialRight(..)` no Capítulo 3](ch3.md/#user-content-partialright), ele usa `reverseArgs(..)` nos bastidores, assim como nosso `pipe(..)` agora funciona. Portanto, obtemos o mesmo resultado de qualquer maneira.

*Neste caso específico*, a pequena vantagem de desempenho de usar `pipe(..)` é que, como não estamos tentando preservar a ordem dos argumentos da direita para a esquerda de `compose(..)`, não é necessário reverter a ordem dos argumentos, como fazemos dentro de `partialRight(..)`. Portanto `partial(pipe, ..)` é um pouco mais eficiente aqui do que `partialRight(compose, ..)`.

## Abstração

A abstração desempenha um papel importante em nosso raciocínio sobre composição, então vamos examiná-la com mais detalhes.

Semelhante a como a aplicação parcial e o currying (consulte o [Capítulo 3](ch3.md/#some-now-some-later)) permitem uma progressão de funções generalizadas para funções especializadas, podemos abstrair extraindo a generalidade entre duas ou mais tarefas . A parte geral é definida uma vez, para evitar repetições. Para realizar a especialização de cada tarefa, a parte geral é parametrizada.

Por exemplo, considere este código (obviamente inventado):

```js
function saveComment(txt) {
    if (txt != "") {
        comments[comments.length] = txt;
    }
}

function trackEvent(evt) {
    if (evt.name !== undefined) {
        events[evt.name] = evt;
    }
}
```

Both of these utilities are storing a value in a data source. That's the generality. The specialty is that one of them sticks the value at the end of an array, while the other sets the value at a property name of an object.

So let's abstract:

```js
function storeData(store,location,value) {
    store[location] = value;
}

function saveComment(txt) {
    if (txt != "") {
        storeData( comments, comments.length, txt );
    }
}

function trackEvent(evt) {
    if (evt.name !== undefined) {
        storeData( events, evt.name, evt );
    }
}
```

The general task of referencing a property on an object (or array, thanks to JS's convenient operator overloading of `[ ]`) and setting its value is abstracted into its own function `storeData(..)`. While this utility only has a single line of code right now, one could envision other general behavior that was common across both tasks, such as generating a unique numeric ID or storing a timestamp with the value.

If we repeat the common general behavior in multiple places, we run the maintenance risk of changing some instances but forgetting to change others. There's a principle at play in this kind of abstraction, often referred to as "don't repeat yourself" (DRY).

DRY strives to have only one definition in a program for any given task. An alternative aphorism to motivate DRY coding is that programmers are just generally lazy and don't want to do unnecessary work.

Abstraction can be taken too far. Consider:

```js
function conditionallyStoreData(store,location,value,checkFn) {
    if (checkFn( value, store, location )) {
        store[location] = value;
    }
}

function notEmpty(val) { return val != ""; }

function isUndefined(val) { return val === undefined; }

function isPropUndefined(val,obj,prop) {
    return isUndefined( obj[prop] );
}

function saveComment(txt) {
    conditionallyStoreData( comments, comments.length, txt, notEmpty );
}

function trackEvent(evt) {
    conditionallyStoreData( events, evt.name, evt, isPropUndefined );
}
```

In an effort to be DRY and avoid repeating an `if` statement, we moved the conditional into the general abstraction. We also assumed that we *may* have checks for non-empty strings or non-`undefined` values elsewhere in the program in the future, so we might as well DRY those out, too!

This code *is* more DRY, but to an overkill extent. Programmers must be careful to apply the appropriate levels of abstraction to each part of their program, no more, no less.

Regarding our greater discussion of function composition in this chapter, it might seem like its benefit is this kind of DRY abstraction. But let's not jump to that conclusion, because I think composition actually serves a more important purpose in our code.

Moreover, **composition is helpful even if there's only one occurrence of something** (no repetition to DRY out).

### Separation Enables Focus

Aside from generalization vs. specialization, I think there's another more useful definition for abstraction, as revealed by this quote:

> ... abstraction is a process by which the programmer associates a name with a potentially complicated program fragment, which can then be thought of in terms of its purpose of function, rather than in terms of how that function is achieved. By hiding irrelevant details, abstraction reduces conceptual complexity, making it possible for the programmer to focus on a manageable subset of the program text at any particular time.
>
> Michael L. Scott, Programming Language Pragmatics<a href="#user-content-footnote-1"><sup>1</sup></a>

The point this quote makes is that abstraction -- generally, pulling out some piece of code into its own function -- serves the primary purpose of separating apart two pieces of functionality so that it's possible to focus on each piece independently of the other.

Note that abstraction in this sense is not really intended to *hide* details, as if to treat things as black boxes we *never* examine.

In this quote, "irrelevant", in terms of what is hidden, shouldn't be thought of as an absolute qualitative judgement, but rather relative to what you want to focus on at any given moment. In other words, when we separate X from Y, if I want to focus on X, Y is irrelevant at that moment. At another time, if I want to focus on Y, X is irrelevant at that moment.

**We're not abstracting to hide details; we're separating details to improve focus.**

Recall that at the outset of this book I stated that FP's goal is to create code that is more readable and understandable. One effective way of doing that is untangling complected (read: tightly braided, as in strands of rope) code into separate, simpler (read: loosely bound) pieces of code. In that way, the reader isn't distracted by the details of one part while looking for the details of the other part.

Our higher goal is not to implement something only once, as it is with the DRY mindset. As a matter of fact, sometimes we'll actually repeat ourselves in code.

As we [asserted in Chapter 3](ch3.md/#why-currying-and-partial-application), the main goal with abstraction is to implement separate things, separately. We're trying to improve focus, because that improves readability.

By separating two ideas, we insert a semantic boundary between them, which affords us the ability to focus on each side independent of the other. In many cases, that semantic boundary is something like the name of a function. The function's implementation is focused on *how* to compute something, and the call-site using that function by name is focused on *what* to do with its output. We abstract the *how* from the *what* so they are separate and separately reason'able.

Another way of describing this goal is with imperative vs. declarative programming style. Imperative code is primarily concerned with explicitly stating *how* to accomplish a task. Declarative code states *what* the outcome should be, and leaves the implementation to some other responsibility.

Declarative code abstracts the *what* from the *how*. Typically declarative coding is favored in readability over imperative, though no program (except of course machine code 1s and 0s) is ever entirely one or the other. The programmer must seek balance between them.

ES6 added many syntactic affordances that transform old imperative operations into newer declarative forms. Perhaps one of the clearest is destructuring. Destructuring is a pattern for assignment that describes how a compound value (object, array) is taken apart into its constituent values.

Here's an example of array destructuring:

```js
function getData() {
    return [1,2,3,4,5];
}

// imperative
var tmp = getData();
var a = tmp[0];
var b = tmp[3];

// declarative
var [ a ,,, b ] = getData();
```

The *what* is assigning the first value of the array to `a` and the fourth value to `b`. The *how* is getting a reference to the array (`tmp`) and manually referencing indexes `0` and `3` in assignments to `a` and `b`, respectively.

Does the array destructuring *hide* the assignment? Depends on your perspective. I'm asserting that it simply separates the *what* from the *how*. The JS engine still does the assignments, but it prevents you from having to be distracted by *how* it's done.

Instead, you read `[ a ,,, b ] = ..` and can see the assignment pattern merely telling you *what* will happen. Array destructuring is an example of declarative abstraction.

### Composition as Abstraction

What's all this have to do with function composition? Function composition is also declarative abstraction.

Recall the `shorterWords(..)` example from earlier. Let's compare an imperative and declarative definition for it:

```js
// imperative
function shorterWords(text) {
    return skipLongWords( unique( words( text ) ) );
}

// declarative
var shorterWords = compose( skipLongWords, unique, words );
```

The declarative form focuses on the *what* -- these three functions pipe data from a string to a list of shorter words -- and leaves the *how* to the internals of `compose(..)`.

In a bigger sense, the `shorterWords = compose(..)` line explains the *how* for defining a `shorterWords(..)` utility, leaving this declarative line somewhere else in the code to focus only on the *what*:

```js
shorterWords( text );
```

Composition abstracts getting a list of shorter words from the steps it takes to do that.

By contrast, what if we hadn't used composition abstraction?

```js
var wordsFound = words( text );
var uniqueWordsFound = unique( wordsFound );
skipLongWords( uniqueWordsFound );
```

Or even:

```js
skipLongWords( unique( words( text ) ) );
```

Either of these two versions demonstrates a more imperative style as opposed to the prior declarative style. The reader's focus in those two snippets is inextricably tied to the *how* and less on the *what*.

Function composition isn't just about saving code with DRY. Even if the usage of `shorterWords(..)` only occurs in one place -- so there's no repetition to avoid! -- separating the *how* from the *what* still improves our code.

Composition is a powerful tool for abstraction that transforms imperative code into more readable declarative code.

## Revisiting Points

Now that we've thoroughly covered composition (a trick that will be immensely helpful in many areas of FP), let's watch it in action by revisiting point-free style from [Chapter 3, "No Points"](ch3.md/#no-points) with a scenario that's a fair bit more complex to refactor:

```js
// given: ajax( url, data, cb )

var getPerson = partial( ajax, "http://some.api/person" );
var getLastOrder = partial( ajax, "http://some.api/order", { id: -1 } );

getLastOrder( function orderFound(order){
    getPerson( { id: order.personId }, function personFound(person){
        output( person.name );
    } );
} );
```

The "points" we'd like to remove are the `order` and `person` parameter references.

Let's start by trying to get the `person` "point" out of the `personFound(..)` function. To do so, let's first define:

```js
function extractName(person) {
    return person.name;
}
```

Consider that this operation could instead be expressed in generic terms: extracting any property by name off of any object. Let's call such a utility `prop(..)`:

```js
function prop(name,obj) {
    return obj[name];
}

// or the ES6 => form
var prop =
    (name,obj) =>
        obj[name];
```

While we're dealing with object properties, let's also define the opposite utility: `setProp(..)` for setting a property value onto an object.

However, we want to be careful not to just mutate an existing object but rather create a clone of the object to make the change to, and then return it. The reasons for such care will be discussed at length in [Chapter 5](ch5.md).

<a name="setprop"></a>

```js
function setProp(name,obj,val) {
    var o = Object.assign( {}, obj );
    o[name] = val;
    return o;
}
```

Now, to define an `extractName(..)` that pulls a `"name"` property off an object, we'll partially apply `prop(..)`:

```js
var extractName = partial( prop, "name" );
```

**Note:** Don't miss that `extractName(..)` here hasn't actually extracted anything yet. We partially applied `prop(..)` to make a function that's waiting to extract the `"name"` property from whatever object we pass into it. We could also have done it with `curry(prop)("name")`.

Next, let's narrow the focus on our example's nested lookup calls to this:

```js
getLastOrder( function orderFound(order){
    getPerson( { id: order.personId }, outputPersonName );
} );
```

How can we define `outputPersonName(..)`? To visualize what we need, think about the desired flow of data:

```txt
output <-- extractName <-- person
```

`outputPersonName(..)` needs to be a function that takes an (object) value, passes it into `extractName(..)`, then passes that value to `output(..)`.

Hopefully you recognized that as a `compose(..)` operation. So we can define `outputPersonName(..)` as:

```js
var outputPersonName = compose( output, extractName );
```

The `outputPersonName(..)` function we just created is the callback provided to `getPerson(..)`. So we can define a function called `processPerson(..)` that presets the callback argument, using `partialRight(..)`:

```js
var processPerson = partialRight( getPerson, outputPersonName );
```

Let's reconstruct the nested lookups example again with our new function:

```js
getLastOrder( function orderFound(order){
    processPerson( { id: order.personId } );
} );
```

Phew, we're making good progress!

But we need to keep going and remove the `order` "point". The next step is to observe that `personId` can be extracted from an object (like `order`) via `prop(..)`, just like we did with `name` on the `person` object:

```js
var extractPersonId = partial( prop, "personId" );
```

To construct the object (of the form `{ id: .. }`) that needs to be passed to `processPerson(..)`, let's make another utility for wrapping a value in an object at a specified property name, called `makeObjProp(..)`:

```js
function makeObjProp(name,value) {
    return setProp( name, {}, value );
}

// or the ES6 => form
var makeObjProp =
    (name,value) =>
        setProp( name, {}, value );
```

**Tip:** This utility is known as `objOf(..)` in the Ramda library.

Just as we did with `prop(..)` to make `extractName(..)`, we'll partially apply `makeObjProp(..)` to build a function `personData(..)` that makes our data object:

```js
var personData = partial( makeObjProp, "id" );
```

To use `processPerson(..)` to perform the lookup of a person attached to an `order` value, the conceptual flow of data through operations we need is:

```txt
processPerson <-- personData <-- extractPersonId <-- order
```

So we'll just use `compose(..)` again to define a `lookupPerson(..)` utility:

```js
var lookupPerson =
    compose( processPerson, personData, extractPersonId );
```

And... that's it! Putting the whole example back together without any "points":

```js
var getPerson = partial( ajax, "http://some.api/person" );
var getLastOrder =
    partial( ajax, "http://some.api/order", { id: -1 } );

var extractName = partial( prop, "name" );
var outputPersonName = compose( output, extractName );
var processPerson = partialRight( getPerson, outputPersonName );
var personData = partial( makeObjProp, "id" );
var extractPersonId = partial( prop, "personId" );
var lookupPerson =
    compose( processPerson, personData, extractPersonId );

getLastOrder( lookupPerson );
```

Wow. Point-free. And `compose(..)` turned out to be really helpful in two places!

I think in this case, even though the steps to derive our final answer were a bit drawn out, the end result is much more readable code, because we've ended up explicitly calling out each step.

And even if you didn't like seeing/naming all those intermediate steps, you can preserve point-free but wire the expressions together without individual variables:

```js
partial( ajax, "http://some.api/order", { id: -1 } )
(
    compose(
        partialRight(
            partial( ajax, "http://some.api/person" ),
            compose( output, partial( prop, "name" ) )
        ),
        partial( makeObjProp, "id" ),
        partial( prop, "personId" )
    )
);
```

This snippet is less verbose for sure, but I think it's less readable than the previous snippet where each operation is its own variable. Either way, composition helped us with our point-free style.

## Summary

Function composition is a pattern for defining a function that routes the output of one function call into another function call, and its output to another, and so on.

Because JS functions can only return single values, the pattern essentially dictates that all functions in the composition (except perhaps the first called) need to be unary, taking only a single input from the output of the previous function.

Instead of listing out each step as a discrete call in our code, function composition using a utility like `compose(..)` or `pipe(..)` abstracts that implementation detail so the code is more readable, allowing us to focus on *what* the composition will be used to accomplish, not *how* it will be performed.

Composition is declarative data flow, meaning our code describes the flow of data in an explicit, obvious, and readable way.

In many ways, composition is the most important foundational pattern, in large part because it's the only way to route data through our programs aside from using side effects; the next chapter explores why such should be avoided wherever possible.

----

<a name="footnote-1"><sup>1</sup></a>Scott, Michael L. “Chapter 3: Names, Scopes, and Bindings.” Programming Language Pragmatics, 4th ed., Morgan Kaufmann, 2015, pp. 115.
