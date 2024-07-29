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

Ambos os utilitários armazenam um valor em uma fonte de dados. Essa é a generalidade. A especialidade é que um deles coloca o valor no final de um array, enquanto o outro define o valor no nome de uma propriedade de um objeto.

Então vamos abstrair:

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

A tarefa geral de referenciar uma propriedade em um objeto (ou array, graças à conveniente sobrecarga do operador `[ ]` do JS) e definir seu valor é abstraída em sua própria função `storeData(..)`. Embora este utilitário tenha apenas uma única linha de código no momento, pode-se imaginar outro comportamento geral que era comum em ambas as tarefas, como gerar um ID numérico exclusivo ou armazenar um carimbo de data/hora com o valor.

Se repetirmos o comportamento geral comum em vários lugares, corremos o risco de manutenção de alterar algumas instâncias, mas esquecer de alterar outras. Há um princípio em jogo nesse tipo de abstração, muitas vezes referido como “Don't Repeat Yourself” (DRY).

DRY se esforça para ter apenas uma definição em um programa para qualquer tarefa. Um aforismo alternativo para motivar a codificação DRY é que os programadores geralmente são preguiçosos e não querem fazer trabalho desnecessário.

A abstração pode ser levada ainda mais longe. Considere:


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

Em um esforço para ser DRY e evitar a repetição de uma instrução `if`, movemos a condicional para a abstração geral. Também presumimos que *podemos* ter verificações de strings não vazias ou valores não `indefinidos` em outras partes do programa no futuro, então podemos ser "DRY" também!

Este código *é* mais "DRY", mas de forma exagerada. Os programadores devem ter o cuidado de aplicar os níveis apropriados de abstração a cada parte do seu programa, nem mais, nem menos.

Com relação à nossa discussão mais ampla sobre composição de funções neste capítulo, pode parecer que seu benefício é esse tipo de abstração DRY. Mas não vamos tirar conclusões precipitadas, porque acho que a composição na verdade serve a um propósito mais importante em nosso código.

Além disso, **a composição é útil mesmo se houver apenas uma ocorrência de algo** (sem repetição para DRY).

### Separação permite foco

Além de generalização versus especialização, acho que há outra definição mais útil para abstração, conforme revelado por esta citação:

> ... abstração é um processo pelo qual o programador associa um nome a um fragmento de programa potencialmente complicado, que pode então ser pensado em termos de seu propósito de função, e não em termos de como essa função é alcançada. Ao ocultar detalhes irrelevantes, a abstração reduz a complexidade conceitual, possibilitando ao programador focar em um subconjunto gerenciável do texto do programa a qualquer momento específico.
>
> Michael L. Scott, Programming Language Pragmatics<a href="#user-content-footnote-1"><sup>1</sup></a>

O que esta citação enfatiza é que a abstração -- geralmente, extrai algum pedaço de código em sua própria função -- serve ao propósito principal de separar duas partes de funcionalidade para que seja possível focar em cada parte independentemente da outra.

Observe que a abstração, nesse sentido, não tem a intenção de *ocultar* detalhes, como se tratasse as coisas como caixas pretas que *nunca* examinamos.

Nesta citação, “irrelevante”, em termos do que está oculto, não deve ser pensado como um julgamento qualitativo absoluto, mas sim relativo ao que se deseja focar em um determinado momento. Ou seja, quando separamos X de Y, se quero focar em X, Y é irrelevante naquele momento. Em outro momento, se eu quiser focar em Y, X é irrelevante naquele momento.

**Não estamos abstraindo para ocultar detalhes; estamos separando detalhes para melhorar o foco.**

Lembre-se de que no início deste livro afirmei que o objetivo do FP é criar código que seja mais legível e compreensível. Uma maneira eficaz de fazer isso é desembaraçar código complexo (leia-se: fortemente trançado, como em fios de corda) em pedaços de código separados e mais simples (leia-se: fracamente ligados). Dessa forma, o leitor não se distrai com os detalhes de uma parte enquanto procura os detalhes da outra.

Nosso objetivo maior não é implementar algo apenas uma vez, como acontece com a mentalidade DRY. Na verdade, às vezes nos repetiremos no código.

Como [afirmamos no Capítulo 3](ch3.md/#why-currying-and-partial-application), o principal objetivo da abstração é implementar coisas separadas, separadamente. Estamos tentando melhorar o foco, porque isso melhora a legibilidade.

Ao separar duas ideias, inserimos uma fronteira semântica entre elas, o que nos permite focar em cada lado independentemente do outro. Em muitos casos, esse limite semântico é algo como o nome de uma função. A implementação da função está focada em *como* calcular algo, e o site de chamada que usa essa função pelo nome está focado em *o que* fazer com sua saída. Abstraímos o *como* do *o quê* para que sejam separados e separadamente razoáveis.

Outra maneira de descrever esse objetivo é com estilo de programação imperativo versus declarativo. O código imperativo se preocupa principalmente em declarar explicitamente *como* realizar uma tarefa. O código declarativo declara *qual* deve ser o resultado e deixa a implementação para alguma outra responsabilidade.

O código declarativo abstrai o *o quê* do *como*. Normalmente, a codificação declarativa é favorecida em termos de legibilidade em vez da imperativa, embora nenhum programa (exceto, é claro, o código de máquina 1s e 0s) seja inteiramente um ou outro. O programador deve buscar o equilíbrio entre eles.

ES6 adicionou muitas possibilidades sintáticas que transformam antigas operações imperativas em formas declarativas mais recentes. Talvez uma das mais claras seja a desestruturação. A desestruturação é um padrão de atribuição que descreve como um valor composto (objeto, matriz) é desmontado em seus valores constituintes.

Aqui está um exemplo de array desestruturado:

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

O *o que* está atribuindo o primeiro valor da matriz `a` e o quarto valor da ` b`. O *como* está obtendo uma referência à matriz (`tmp`) e referenciando manualmente os índices` 0` e `3` nas atribuições para` a` e `b`, respectivamente.

A desestruturação da matriz oculta a atribuição? Depende da sua perspectiva. Estou afirmando que ela simplesmente separa o *o que* do *como*. O mecanismo JS ainda faz as tarefas, isso evita que você tenha que se distrair com *como* as coisas são feitas.

Em vez disso, você lê `[ a ,,, b ] = ..` e pode ver o padrão de atribuição meramente dizendo a você *o que* vai acontecer. A desestruturação de array é um exemplo de abstração declarativa.

### Composição como abstração

O que tudo isso tem a ver com composição de função? Composição de função também é abstração declarativa.

Lembre-se do exemplo `shorterWords(..)` anterior. Vamos comparar uma definição imperativa e declarativa para ele:

```js
// imperative
function shorterWords(text) {
    return skipLongWords( unique( words( text ) ) );
}

// declarative
var shorterWords = compose( skipLongWords, unique, words );
```

A forma declarativa foca no *o que* -- essas três funções canalizam dados de uma string para uma lista de palavras mais curtas -- e deixa o *como* para os internos de `compose(..)`.

Em um sentido mais amplo, a linha `shorterWords = compose(..)` explica o *como* para definir um utilitário `shorterWords(..)`, deixando essa linha declarativa em algum outro lugar no código para focar apenas no *o que*:

```js
shorterWords( text );
```

Abstrações de composição obtêm uma lista de palavras mais curtas a partir dos passos necessários para fazer isso.

Em contraste, e se não tivéssemos usado abstração de composição?

```js
var wordsFound = words( text );
var uniqueWordsFound = unique( wordsFound );
skipLongWords( uniqueWordsFound );
```

Ou mesmo:

```js
skipLongWords( unique( words( text ) ) );
```

Qualquer uma dessas duas versões demonstra um estilo mais imperativo em oposição ao estilo declarativo anterior. O foco do leitor nesses dois trechos está inextricavelmente ligado ao *como* e menos ao *o quê*.

A composição de funções não é apenas sobre salvar código com DRY. Mesmo que o uso de `shorterWords(..)` ocorra apenas em um lugar -- então não há repetição a evitar! -- separar o *como* do *o que* ainda melhora nosso código.

A composição é uma ferramenta poderosa para abstração que transforma código imperativo em código declarativo mais legível.

## Revendo os pontos

Agora que cobrimos completamente a composição (um truque que será imensamente útil em muitas áreas do FP), vamos observá-lo em ação revisitando o estilo sem pontos do [Capítulo 3, "Sem pontos"](ch3.md/#no-points) com um cenário que é um pouco mais complexo de refatorar:

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

Os "pontos" que gostaríamos de remover são as referências de parâmetros `order` e `person`.

Vamos começar tentando obter o "ponto" `person` da função `personFound(..)`. Para fazer isso, vamos primeiro definir:

```js
function extractName(person) {
    return person.name;
}
```

Considere que essa operação poderia ser expressa em termos genéricos: extrair qualquer propriedade por nome de qualquer objeto. Vamos chamar esse utilitário de `prop(..)`:

```js
function prop(name,obj) {
    return obj[name];
}

// or the ES6 => form
var prop =
    (name,obj) =>
        obj[name];
```

Enquanto lidamos com propriedades de objetos, vamos também definir o utilitário oposto: `setProp(..)` para definir um valor de propriedade em um objeto.

No entanto, queremos ter cuidado para não apenas mutar um objeto existente, mas sim criar um clone do objeto para fazer a alteração e, então, retorná-lo. As razões para tal cuidado serão discutidas em detalhes no [Capítulo 5](ch5.md).

<a name="setprop"></a>

```js
function setProp(name,obj,val) {
    var o = Object.assign( {}, obj );
    o[name] = val;
    return o;
}
```

Agora, para definir um `extract Name(..)` que extrai uma propriedade `"name"` de um objeto, aplicaremos parcialmente `prop(..)`:

```js
var extractName = partial( prop, "name" );
```

**Observação:** Não esqueça que `extractName(..)` aqui ainda não extraiu nada. Aplicamos parcialmente `prop(..)` para criar uma função que está esperando para extrair a propriedade `"name"` de qualquer objeto que passarmos para ela. Também poderíamos ter feito isso com `curry(prop)("name")`.

Em seguida, vamos restringir o foco nas chamadas de pesquisa aninhadas do nosso exemplo para isto:

```js
getLastOrder( function orderFound(order){
    getPerson( { id: order.personId }, outputPersonName );
} );
```

Como podemos definir `outputPersonName(..)`? Para visualizar o que precisamos, pense no fluxo de dados desejado:

```txt
output <-- extractName <-- person
```

`outputPersonName(..)` precisa ser uma função que pega um valor (objeto), passa para `extractName(..)`, então passa esse valor para `output(..)`.

Espero que você tenha reconhecido isso como uma operação `compose(..)`. Então podemos definir `outputPersonName(..)` como:

```js
var outputPersonName = compose( output, extractName );
```

A função `outputPersonName(..)` que acabamos de criar é o retorno de chamada fornecido para `getPerson(..)`. Então podemos definir uma função chamada `processPerson(..)` que predefine o argumento de retorno de chamada, usando `partialRight(..)`:

```js
var processPerson = partialRight( getPerson, outputPersonName );
```

Vamos reconstruir o exemplo de pesquisas aninhadas novamente com nossa nova função:

```js
getLastOrder( function orderFound(order){
    processPerson( { id: order.personId } );
} );
```

Ufa, estamos fazendo um bom progresso!

Mas precisamos continuar e remover o "ponto" `order`. O próximo passo é observar que `personId` pode ser extraído de um objeto (como `order`) via `prop(..)`, assim como fizemos com `name` no objeto `person`:

```js
var extractPersonId = partial( prop, "personId" );
```

Para construir o objeto (do formato `{ id: .. }`) que precisa ser passado para `processPerson(..)`, vamos criar outro utilitário para encapsular um valor em um objeto em um nome de propriedade especificado, chamado `makeObjProp(..)`:

```js
function makeObjProp(name,value) {
    return setProp( name, {}, value );
}

// or the ES6 => form
var makeObjProp =
    (name,value) =>
        setProp( name, {}, value );
```

**Dica:** Este utilitário é conhecido como `objOf(..)` na biblioteca Ramda.

Assim como fizemos com `prop(..)` para fazer `extractName(..)`, aplicaremos parcialmente `makeObjProp(..)` para construir uma função `personData(..)` que faz nosso objeto de dados:

```js
var personData = partial( makeObjProp, "id" );
```

Para usar `processPerson(..)` para executar a pesquisa de uma pessoa anexada a um valor `order`, o fluxo conceitual de dados por meio de operações que precisamos é:

```txt
processPerson <-- personData <-- extractPersonId <-- order
```

Então usaremos `compose(..)` novamente para definir um utilitário `lookupPerson(..)`:

```js
var lookupPerson =
    compose( processPerson, personData, extractPersonId );
```

E... é isso! Montando o exemplo todo de volta sem nenhum "ponto":

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

Uau. Sem pontos. E `compose(..)` acabou sendo realmente útil em dois lugares!

Acho que neste caso, embora os passos para derivar nossa resposta final tenham sido um pouco longos, o resultado final é um código muito mais legível, porque acabamos chamando explicitamente cada passo.

E mesmo que você não goste de ver/nomear todos esses passos intermediários, você pode preservar a ausência de pontos, mas conectar as expressões sem variáveis ​​individuais:

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

Este snippet é menos verboso, com certeza, mas acho que é menos legível do que o snippet anterior, onde cada operação é sua própria variável. De qualquer forma, a composição nos ajudou com nosso estilo sem pontos.

## Resumo

Composição de funções é um padrão para definir uma função que roteia a saída de uma chamada de função para outra chamada de função, e sua saída para outra, e assim por diante.

Como as funções JS só podem retornar valores únicos, o padrão essencialmente determina que todas as funções na composição (exceto talvez a primeira chamada) precisam ser unárias, recebendo apenas uma única entrada da saída da função anterior.

Em vez de listar cada etapa como uma chamada discreta em nosso código, a composição de funções usando um utilitário como `compose(..)` ou `pipe(..)` abstrai os detalhes da implementação para que o código seja mais legível, permitindo-nos focar em *o que* a composição será usada para realizar, não em *como* ela será executada.

Composição é um fluxo de dados declarativo, o que significa que nosso código descreve o fluxo de dados de forma explícita, óbvia e legível.

De muitas maneiras, a composição é o padrão fundamental mais importante, em grande parte porque é a única maneira de rotear dados por meio de nossos programas, além de usar efeitos colaterais; o próximo capítulo explora por que isso deve ser evitado sempre que possível.

----

<a name="footnote-1"><sup>1</sup></a>Scott, Michael L. “Capítulo 3: Nomes, escopos e vinculações.” Programming Language Pragmatics, 4th ed., Morgan Kaufmann, 2015, pp. 115.
