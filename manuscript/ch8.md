# Functional-Light JavaScript
# Capítulo 8: Recursão
Você se divertiu no nosso pequeno mergulho em closures/objetos no capítulo anterior? Bem-vindo de volta!

Na próxima página, vamos mergulhar no tópico de recursão.

<hr>

*(resto da página intencionalmente deixado em branco)*

<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>

<div style="page-break-after: always;"></div>

Vamos falar sobre recursão. Antes de mergulharmos, consulte a página anterior para a definição formal.

Piada fraca, eu sei. :)

Recursão é uma daquelas técnicas de programação que a maioria dos desenvolvedores admite que pode ser muito poderosa, mas também a maioria não gosta de usá-la. Eu colocaria na mesma categoria que expressões regulares, nesse sentido. Poderosa, mas confusa, e portanto, visto como *não valendo o esforço*.

Eu sou um grande fã da recursão, e você também pode ser! Infelizmente, muitos exemplos de recursão concentram-se em tarefas acadêmicas triviais, como gerar sequências de Finonacci.
Se você precisa desses tipos de números em seu programa -- e sejamos sinceros, isso não é muito comum! -- você provavelmente perderá a visão geral.

De fato, a recursão é uma das maneiras mais importantes que os desenvolvedores de PF (Programação Funcional) usam para evitar loops imperativos e reatribuições, delegando os detalhes de implementação à linguagem e ao mecanismo. Quando usada corretamente, a recursão é poderosamente declaritiva para problemas complexos.

Infelizmente, a recursão recebe muito menos atenção, especialmente em JS, do que deveria, em grande parte devido a algumas limitações reais de desempenho (velociadade e memória). Nosso objetivo neste capítulo é aprofundar e encontrar razões práticas pelas quais a recursão deve estar em destaque na nossa PF.

## Definição

Recursão é quando uma função chama a si mesma, e essa chamada faz o mesmo, e seu ciclo continua até que uma condição base seja satisfeita e o loop de chamada desdobre.

**Aviso:** Se você não garantir que uma condição base seja *eventualmente* atendida, a recursão rodará indefinidamente e causará uma falha ou travará seu programa; a condição base é muito importante para acertar!

Mas... essa definição é muito confusa em sua forma escrita. Podemos fazer melhor. Considere esta função recursirva:


```js
function foo(x) {
    if (x < 5) return x;
    return foo( x / 2 );
}
```

Vamos visualizar o que acontece com esta função quando chamamos `foo( 16 )`:

<p align="center">
    <img src="images/fig13.png">
</p>

No passo 2, `x / 2` produz `8`, e isso é passado como argumento para uma chamada recursiva `foo(..)`.
No passo 3, a mesma coisa, `x / 2` produz `4`, e isso é passado como argumento para outra chamada `foo(..)`. Esperamos que essa parte seja bastante direta.

Mas onde alguém pode frenquentemente se confundir é o que acontece no passo 4. Uma vez que a condição base `x` (valor `4`) é `< 5` é satisfeita, não fazemos mais chamadas recursivas, e simplesmente (efetivamente) fazemos `return 4`. Especificamente, a linha pontilhada de retorno de `4` nesta figura simplifica o que está acontecendo ali, então vamos detalhar esse último passo e visualizá-lo com estes três sub-passos:

<p align="center">
    <img src="images/fig14.png">
</p>

Uma vez que a condição base é satisfeita, o valor retornado se propaga de volta por todas as chamadas de função atuais (e, portanto, `return`s), eventualmente retornando o resultado final

Outra maneira de visualizar essa recursão é considerando as chamadas de função na ordem em que acontecem (comumente referido como a pilha de chamadas):

<p align="center">
    <img src="images/fig19.png" width="30%">
</p>

Mais sobre a pilha de chamadas mais adiante neste capítulo.

Outro exemplo de recursão:

```js
function isPrime(num, divisor = 2){
    if (num < 2 || (num > 2 && num % divisor == 0)) {
        return false;
    }
    if (divisor <= Math.sqrt( num )) {
        return isPrime( num, divisor + 1 );
    }

    return true;
}
```

Essa verificação de primos funciona basicamente tentando cada inteiro de `2` até a raiz quadrada do `num` sendo verificado, para ver se algum deles divide igualmente (`%` mod retornando `0`) o número. Se algum o fizer, não é um primo. Caso contrário, deve ser primo. O `divisor + 1` usa a recursão para iterar através de cada valor possível de `divisor`.

Um dos exemplos mais famosos de recursão é calcular um número de Fibonacci, onde a sequência é definida como:

```txt
fib( 0 ): 0
fib( 1 ): 1
fib( n ):
    fib( n - 2 ) + fib( n - 1 )
```

**Nota:** Os primeiros vários números dessa sequência são: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34... Cada número é a adição dos dois números anteriores na sequência.

A definição de Fibonacci expressa diretamente em código:

```js
function fib(n) {
    if (n <= 1) return n;
    return fib( n - 2 ) + fib( n - 1 );
}
```

`fib(..)` chama a si mesma recursivamente duas vezes, o que é tipicamente referido como recursão binária. Falaremos mais sobre recursão binária mais adiante.

Usaremos `fib(..)` de várias maneiras ao longo deste capítulo para ilustrar ideias sobre recursão, mas uma desvantagem dessa forma particular é que há um monte de trabalho duplicado. `fib(n-1)` e `fib(n-2)` não compartilham nenhum de seu trabalho com o outro, mas se sobrepõem quase totalmente, em todo o espaço inteiro até `0`.

Tocamos brevemente em memoização no [Capítulo 5, "Efeitos de perfomance"](ch5.md/#performance-effects). Aqui, a memoização permitiria que o `fib(..)` de qualquer número dado fosse computado apenas uma vez, em vez de ser recomputado várias vezes. Não vamos aprofundar mais nesse tópico aqui, mas essa ressalva de desempenho é importante de se ter em mente com qualquer algoritmo, recursivo ou não.

### Recursão Mútua

Quando uma função chama a si mesma, isso é chamado de recursão direta. Foi isso que vimos na seção anterior com `foo(..)`, `isPrime(..)`, and `fib(..)`. Quando duas ou mais funções se chamam mutuamente em um ciclo recursivo, isso é chamado de recursão mútua.

Essas duas funções são recursivas mutuamente:

```js
function isOdd(v) {
    if (v === 0) return false;
    return isEven( Math.abs( v ) - 1 );
}

function isEven(v) {
    if (v === 0) return true;
    return isOdd( Math.abs( v ) - 1 );
}
```

Sim, essa é uma maneira boba de calcular se um número é ímpar ou par. Mas ilustra a ideia de que certos algoritmos podem ser definidos em termos de recursão mútua.

Lembre-se do `fib(..)` recursivo binário da seção anterior; poderíamos, em vez disso, tê-lo expressado com recursão mútua:

```js
function fib_(n) {
    if (n == 1) return 1;
    else return fib( n - 2 );
}

function fib(n) {
    if (n == 0) return 0;
    else return fib( n - 1 ) + fib_( n );
}
```

**Nota:** Esta implementação recursiva mútua de `fib(..)` foi adaptada de uma pesquisa apresentada em ["Números de Fibonacci usando recursão mútua"](https://www.researchgate.net/publication/246180510_Fibonacci_Numbers_Using_Mutual_Recursion).

Embora os exemplos de recursão mútua mostrados sejam um tanto artificiais, há casos de uso mais complexos onde a recursão mútua pode ser muito útil. Contar o número de folhas em uma estrutura de dados em árvore é um exemplo, e a análise de descida recursiva (de código-fonte, por um compilador) é outro.


### Por que Recursão?

Agora que definimos e ilustramos a recursão, devemos examinar por que ela é útil.

A razão mais citada para que a recursão se encaixe no espírito da programação funcional (FP) é porque ela troca (muito da) rastreabilidade explícita de estado por estado implícito na pilha de chamadas. Normalmente, a recursão é mais útil quando o problema exige ramificação condicional e retrocesso, e gerenciar esse tipo de estado em um ambiente puramente iterativo pode ser bastante complicado; no mínimo, o código é altamente imperativo e mais difícil de ler e verificar. Mas rastrear cada nível de ramificação como seu próprio escopo na pilha de chamadas frequentemente melhora significativamente a legibilidade do código.

Algoritmos iterativos simples podem ser trivialmente expressos como recursão:

```js
function sum(total,...nums) {
    for (let num of nums) {
        total = total + num;
    }

    return total;
}

// vs

function sum(num1,...nums) {
    if (nums.length == 0) return num1;
    return num1 + sum( ...nums );
}
```

Não é apenas que o loop `for` é eliminado em favor da pilha de chamadas, mas que as somas parciais incrementais (o estado intermitente de `total`) são rastreadas implicitamente através dos `return`s da pilha de chamadas em vez de reatribuir `total` a cada iteração. Programadores funcionais frequentemente preferem evitar a reatribuição de variáveis locais quando possível.

Em um algoritmo básico como esta soma, essa diferença é menor e sutil. Mas quanto mais sofisticado for seu algoritmo, mais provável será que você veja os benefícios da recursão em vez do rastreamento imperativo de estado.

## Recursão Declarativa

Matemáticos usam o símbolo **Σ** como um espaço reservado para representar a soma de uma lista de números. A principal razão para isso é que é mais trabalhoso (e menos legível!) se eles estiverem lidando com fórmulas mais complexas e tiverem que escrever a soma manualmente, como `1 + 3 + 5 + 7 + 9 + ..`. Usar a notação é matemática declarativa!

A recursão é declarativa para algoritmos no mesmo sentido que **Σ** é declarativo para a matemática. A recursão expressa que uma solução para o problema existe, mas não exige necessariamente que o leitor do código entenda como essa solução funciona. Vamos considerar duas abordagens para encontrar o maior número par passado como argumento:

```js
function maxEven(...nums) {
    var maxNum = -Infinity;

    for (let num of nums) {
        if (num % 2 == 0 && num > maxNum) {
            maxNum = num;
        }
    }

    if (maxNum !== -Infinity) {
        return maxNum;
    }
}
```

Esta implementação não é particularmente intransigente, mas também não é imediatamente claro quais são suas sutilezas. Quão óbvio é que `maxEven()`, `maxEven(1)`, e `maxEven(1,13)` todos retornam `undefined`? Fica claro rapidamente por que a última instrução `if` é necessária?

Vamos considerar uma abordagem recursiva para comparação. Podemos notar a recursão da seguinte maneira:

```txt
maxEven( nums ):
    maxEven( nums.0, maxEven( ...nums.1 ) )
```

Em outras palavras, podemos definir o maior par de uma lista de números como o maior par do primeiro número comparado ao maior par dos demais números. Por exemplo:

```txt
maxEven( 1, 10, 3, 2 ):
    maxEven( 1, maxEven( 10, maxEven( 3, maxEven( 2 ) ) )
```

Para implementar essa definição recursiva em JS, uma abordagem é:

```js
function maxEven(num1,...restNums) {
    var maxRest = restNums.length > 0 ?
            maxEven( ...restNums ) :
            undefined;

    return (num1 % 2 != 0 || num1 < maxRest) ?
        maxRest :
        num1;
}
```

Então, quais são as vantagens dessa abordagem?

Primeiro, a assinatura é um pouco diferente da anterior. Eu intencionalmente chamei o primeiro argumento de `num1` coletando os demais argumentos em `restNums`. Mas por quê? Poderíamos apenas ter coletado todos em um único array `nums` e depois referir a `nums[0]`.

Essa assinatura de função é uma dica intencional para a definição recursiva. Ela lê assim:

```txt
maxEven( num1, ...restNums ):
    maxEven( num1, maxEven( ...restNums ) )
```

Você vê a simetria entre a assinatura e a definição recursiva?

Quando podemos tornar a definição recursiva mais aparente mesmo na assinatura da função, melhoramos a declaratividade da função. E se pudermos então espelhar a definição recursiva da assinatura para o corpo da função, fica ainda melhor.

Mas eu diria que a melhoria mais óbvia é que a distração do loop `for` imperativo é suprimida. Toda a lógica de looping é abstraída na pilha de chamadas recursivas, para que essas partes não poluam o código. Então, podemos focar na lógica de encontrar um par máximo comparando dois números de cada vez — que é a parte importante, de qualquer forma!

Mentalmente, o que está acontecendo é semelhante ao quando um matemático usa uma soma **Σ** em uma equação maior. Estamos dizendo: "o maior par do restante da lista é calculado por `maxEven(...restNums), então vamos assumir essa parte e seguir em frente."

Além disso, reforçamos essa noção com a guarda `restNums.length > 0`, porque se não há mais números a considerar, o resultado natural é que `maxRest` seria `undefined`. Não precisamos dedicar atenção mental extra a essa parte da lógica. Essa condição base (não há mais números a considerar) é claramente evidente.

Em seguida, voltamos nossa atenção para verificar `num1` em relação a `maxRest` — a lógica principal do algoritmo é como determinar qual dos dois números, se houver, é o maior par. Se `num1` não é par (`num1 % 2 != 0`), ou é menor que `maxRest`, então `maxRest` deve ser retornado, mesmo que seja `undefined`. Caso contrário, `num1` é a resposta.

O argumento que estou fazendo é que esse raciocínio ao ler uma implementação é mais direto, com menos nuances ou ruído para nos distrair, do que a abordagem imperativa; é **mais declarativo** do que a versão com `for` e `-Infinity`.

**Dica:** Devemos observar que outra (provavelmente melhor!) forma de modelar isso além da iteração manual ou recursão seria com operações de lista (Veja [Capítulo 9](ch9.md)), com um `filter(..)` para incluir apenas os pares e depois um `reduce(..)` para encontrar o máximo. Apenas usamos este exemplo para ilustrar a natureza mais declarativa da recursão em relação à iteração manual.

### Recursão em Árvore Binária

Aqui está outro exemplo de recursão: calcular a profundidade de uma árvore binária. Na verdade, quase toda operação que você faz com árvores é implementada mais facilmente com recursão, porque rastrear manualmente a pilha para cima e para baixo é altamente imperativo e propenso a erros.

A profundidade de uma árvore binária é o caminho mais longo para baixo (para a esquerda ou para a direita) através dos nós da árvore. Outra maneira de definir isso é recursivamente — a profundidade de uma árvore em qualquer nó é 1 (o nó atual) mais o maior das profundidades das suas subárvores esquerda ou direita:

```txt
depth( node ):
    1 + max( depth( node.left ), depth( node.right ) )
```

Traduzindo isso diretamente para uma função binária recursiva:

```js
function depth(node) {
    if (node) {
        let depthLeft = depth( node.left );
        let depthRight = depth( node.right );
        return 1 + max( depthLeft, depthRight );
    }

    return 0;
}
```

Não vou listar a forma imperativa desse algoritmo, mas acredite, é muito mais confusa. Essa abordagem recursiva é de forma elegante e declarativa. Ela segue a definição recursiva do algoritmo de forma muito próxima, com muito pouca distração.

Nem todos os problemas são limpos e recursivos. Isso não é uma solução mágica que você deve tentar aplicar em todos os lugares. Mas a recursão pode ser muito eficaz para evoluir a expressão de um problema de um formato mais imperativo para um formato mais declarativo.

## Pilha

Vamos revisar a recursão `isOdd(..)` / `isEven(..)` de antes:

```js
function isOdd(v) {
    if (v === 0) return false;
    return isEven( Math.abs( v ) - 1 );
}

function isEven(v) {
    if (v === 0) return true;
    return isOdd( Math.abs( v ) - 1 );
}
```

Na maioria dos navegadores, se você tentar isso, você receberá um erro:

```js
isOdd( 33333 );         // RangeError: Tamanho máximo da pilha de chamadas excedido
```

O que está acontecendo com esse erro? O mecanismo lança esse erro porque está tentando proteger seu programa de esgotar a memória do sistema. Para explicar isso, precisamos dar uma olhada um pouco abaixo da superfície sobre o que está acontecendo no mecanismo JS quando as chamadas de função ocorrem.

Cada chamada de função reserva um pequeno pedaço de memória chamado de quadro de pilha (ou stack frame). O quadro de pilha armazena informações importantes sobre o estado atual do processamento das instruções em uma função, incluindo os valores em quaisquer variáveis. A razão para essas informações precisarem ser armazenadas em memória (em um quadro de pilha) é porque a função pode chamar outra função, o que pausa a função atual. Quando a outra função termina, o mecanismo precisa retomar o estado exato de quando foi pausado.

Quando a segunda chamada de função começa, ela também precisa de um quadro de pilha, aumentando a contagem para 2. Se essa função chama outra, precisamos de um terceiro quadro de pilha. E assim por diante. A palavra "pilha" refere-se ao conceito de que cada vez que uma função é chamada a partir da função anterior, o próximo quadro é *empilhado* sobre o anterior. Quando uma chamada de função termina, seu quadro é retirado da pilha.

Considere este programa:

```js
function foo() {
    var z = "foo!";
}

function bar() {
    var y = "bar!";
    foo();
}

function baz() {
    var x = "baz!";
    bar();
}

baz();
```

Visualizando os quadros de pilha deste programa passo a passo:

<p align="center">
    <img src="images/fig15.png" width="80%">
</p>

**Nota:** Se essas funções não chamassem umas às outras, mas fossem chamadas sequencialmente — como `baz(); bar(); foo();`, onde cada uma termina antes que a próxima comece — os quadros não seriam empilhados; cada chamada de função termina e remove seu quadro da pilha antes que o próximo seja adicionado.

Ok, então um pouco de memória é necessário para cada chamada de função. Não é um grande problema na maioria das condições normais de programa, certo? Mas rapidamente se torna um grande problema quando você introduz a recursão. Embora você quase certamente nunca empilhe manualmente milhares (ou mesmo centenas!) de chamadas de diferentes funções em uma única pilha de chamadas, você verá facilmente dezenas de milhares ou mais chamadas recursivas se empilhando.

O emparelhamento `isOdd(..)`/`isEven(..)` lança um `RangeError` porque o mecanismo intervém em um limite arbitrário quando acha que a pilha de chamadas cresceu demais e deve ser interrompida. Isso não é provavelmente um limite baseado em níveis reais de memória se aproximando de zero, mas sim uma previsão do mecanismo de que, se esse tipo de programa fosse deixado rodando, o uso da memória se tornaria descontrolado. É impossível saber ou provar que um programa eventualmente parará, então o mecanismo tem que fazer uma suposição informada.

Esse limite é dependente da implementação. A especificação não diz nada sobre isso, então não é *obrigatório*. Mas praticamente todos os mecanismos JS têm um limite, porque não ter limite criaria um dispositivo instável suscetível a código mal escrito ou malicioso. Cada mecanismo em cada ambiente de dispositivo diferente vai impor seus próprios limites, então não há como prever ou garantir quão longe podemos rodar na pilha de chamadas de função.

O que esse limite significa para nós, como desenvolvedores, é que há uma limitação prática na utilidade da recursão para resolver problemas em conjuntos de dados não triviais. Na verdade, eu acho que essa limitação pode ser a razão mais importante para que a recursão seja uma cidadã de segunda classe na caixa de ferramentas dos desenvolvedores. Lamentavelmente, a recursão é um pensamento posterior em vez de uma técnica primária.

### Chamadas de Cauda

A recursão é muito anterior ao JS, assim como essas limitações de memória. Na década de 1960, os desenvolvedores queriam usar a recursão e enfrentavam limites rígidos de memória dos dispositivos que eram muito mais baixos do que temos em nossos relógios hoje.

Felizmente, uma observação poderosa foi feita naqueles primeiros dias que ainda oferece esperança. A técnica é chamada de *chamadas de cauda*.

A ideia é que se uma chamada da função `baz()` para a função `bar()` acontecer no final da execução da função `baz()` — conhecida como chamada de cauda — o quadro de pilha para `baz()` não é mais necessário. Isso significa que a memória pode ser recuperada ou, ainda melhor, simplesmente reutilizada para lidar com a execução da função `bar()`. Visualizando:

<p align="center">
    <img src="images/fig16.png" width="80%">
</p>

Chamadas de cauda não estão realmente diretamente relacionadas à recursão; essa noção se aplica a qualquer chamada de função. Mas suas pilhas de chamadas não-recursivas manuais são improváveis de ir além de talvez 10 níveis de profundidade na maioria dos casos, então as chances de chamadas de cauda impactarem a memória do seu programa são bastante baixas.

Chamadas de cauda realmente brilham no caso da recursão, porque isso significa que uma pilha recursiva poderia rodar "para sempre", e a única preocupação de desempenho seria a computação, não as limitações fixas de memória. Recursão de chamada de cauda pode rodar com uso de memória fixo `O(1)`.

Esses tipos de técnicas são frequentemente referidos como Otimizações de Chamadas de Cauda (OCC), mas é importante distinguir a capacidade de detectar uma chamada de cauda para rodar em espaço de memória fixo, das técnicas que otimizam essa abordagem. Tecnicamente, chamadas de cauda não são uma otimização de desempenho como a maioria das pessoas pensa, pois podem na verdade rodar mais lentamente do que chamadas normais. TCO é sobre otimizar chamadas de cauda para rodar mais eficientemente.

### Chamadas de Cauda Adequadas (CCA)

JavaScript nunca exigiu (nem proibiu) chamadas de cauda, até o ES6. O ES6 manda o reconhecimento das chamadas de cauda, de uma forma específica chamada Chamadas de Cauda Adequadas (CCA), e a garantia de que o código no formato CCA rodará sem crescimento de memória de pilha não controlado. Na prática, isso significa que não devemos receber `RangeErrors` lançados se aderirmos ao CCA.

Primeiro, CCA em JavaScript requer o modo estrito. Você já deve estar usando o modo estrito, mas se não estiver, esta é mais uma razão para já estar usando o modo estrito. Eu mencionei, ainda, que você deveria estar usando o modo estrito!?

Em segundo lugar, a *chamada de cauda adequada* se parece com isso:

```js
return foo( .. );
```

Em outras palavras, a chamada de função é a última coisa a ser executada na função envolvente, e qualquer valor que ela retorna é explicitamente retornado. Dessa forma, o JS pode garantir absolutamente que o quadro de pilha atual não será mais necessário.

Estas *não são* CCA:

```js
foo();
return;

// or

var x = foo( .. );
return x;

// or

return 1 + foo( .. );
```

**Nota:** Um mecanismo JS, ou um transpiler inteligente, poderia reorganizar o código para tratar `var x = foo(); return x;` efetivamente da mesma forma que `return foo();`, o que então o tornaria elegível para CCA. Mas isso não é exigido pela especificação.

A parte `1 +` definitivamente é processada *depois* que `foo(..)` termina, então o quadro de pilha precisa ser mantido.

No entanto, isso *é* CCA:

```js
return x ? foo( .. ) : bar( .. );
```

Depois que a condição `x` é computada, ou `foo(..)` ou `bar(..)` será executada, e em qualquer caso, o valor de retorno será sempre retornado. Isso é forma de CCA.

Recursão binária (ou múltipla) — como mostrado anteriormente, duas (ou mais!) chamadas recursivas feitas em cada nível — nunca pode ser totalmente CCA como está, porque toda a recursão deve estar em posição de chamada de cauda para evitar o crescimento da pilha; no máximo, apenas uma chamada recursiva pode aparecer em posição de CCA.

Anteriormente, mostramos um exemplo de refatoração de recursão binária para recursão mútua. Pode ser possível alcançar Chamadas de Cauda Adequadas (PTC) a partir de um algoritmo com múltiplas recursões, dividindo cada uma em chamadas de funções separadas, onde cada uma é expressa, respectivamente, em forma de PTC. No entanto, esse tipo de refatoração intrincada é altamente dependente do cenário e está além do escopo do que podemos cobrir neste texto.

## Reestruturando Recursão

Se você deseja usar recursão, mas seu conjunto de problemas pode crescer o suficiente para eventualmente exceder o limite de pilha do motor JS, você precisará reestruturar suas chamadas recursivas para aproveitar o CCA (ou evitar chamadas aninhadas totalmente). Existem várias estratégias de refatoração que podem ajudar, mas, claro, há compensações a serem consideradas.

Como um aviso, sempre tenha em mente que a legibilidade do código é nosso objetivo mais importante. Se a recursão, juntamente com alguma combinação das estratégias descritas aqui, resultar em um código que é mais difícil de ler/compreender, **não use recursão**; encontre uma abordagem mais legível.

### Substituindo a Pilha

O principal problema com a recursão é seu uso de memória, mantendo os frames de pilha para rastrear o estado de uma chamada de função enquanto ela despacha para a próxima iteração da chamada recursiva. Se conseguirmos descobrir como reestruturar nosso uso de recursão para que o frame de pilha não precise ser mantido, então podemos expressar a recursão com CCA e aproveitar o tratamento otimizado de chamadas de cauda pelo motor JS.

Vamos relembrar o exemplo de soma de antes:

```js
function sum(num1,...nums) {
    if (nums.length == 0) return num1;
    return num1 + sum( ...nums );
}
```

Isso não está na forma CCA porque, após a chamada recursiva para `sum(...nums)` ser concluída, a variável `total` é adicionada ao resultado. Portanto, o frame de pilha deve ser preservado para acompanhar o resultado parcial de `total` enquanto o restante da recursão prossegue.

O ponto chave para essa estratégia de refatoração é que podemos remover nossa dependência da pilha fazendo a adição *agora* em vez de *depois*, e então passar esse resultado parcial como argumento para a chamada recursiva. Em outras palavras, em vez de manter `total` no frame de pilha da função atual, empurre-o para o frame de pilha da próxima chamada recursiva; isso libera o frame de pilha atual para ser removido/reutilizado.

Para começar, poderíamos alterar a assinatura da nossa função `sum(..)` para ter um novo primeiro parâmetro como o resultado parcial:

```js
function sum(result,num1,...nums) {
    // ..
}
```

Agora, devemos pré-calcular a adição de `result` e `num1`, e passar isso adiante:

```js
"use strict";

function sum(result,num1,...nums) {
    result = result + num1;
    if (nums.length == 0) return result;
    return sum( result, ...nums );
}
```

Agora nosso `sum(..)` está na forma CCA! Yay!

Mas o lado negativo é que agora alteramos a assinatura da função, tornando seu uso mais estranho. O chamador essencialmente tem que passar `0` como o primeiro argumento antes dos outros números que deseja somar:

```js
sum( /*initialResult=*/0, 3, 1, 17, 94, 8 );        // 123
```

Isso é lamentável.

Normalmente, as pessoas resolvem isso nomeando sua função recursiva com assinatura estranha de forma diferente, e então definindo uma função de interface que oculta a estranheza:

```js
"use strict";

function sumRec(result,num1,...nums) {
    result = result + num1;
    if (nums.length == 0) return result;
    return sumRec( result, ...nums );
}

function sum(...nums) {
    return sumRec( /*initialResult=*/0, ...nums );
}

sum( 3, 1, 17, 94, 8 );                             // 123
```

Isso é melhor. Ainda é lamentável que agora criamos várias funções em vez de apenas uma. Às vezes, você verá desenvolvedores "ocultando" a função recursiva como uma função interna, assim:

```js
"use strict";

function sum(...nums) {
    return sumRec( /*initialResult=*/0, ...nums );

    function sumRec(result,num1,...nums) {
        result = result + num1;
        if (nums.length == 0) return result;
        return sumRec( result, ...nums );
    }
}

sum( 3, 1, 17, 94, 8 );                             // 123
```

O lado negativo aqui é que recriaremos essa função interna `sumRec(..)` cada vez que a função externa `sum(..)` for chamada. Então, podemos voltar a tê-las como funções lado a lado, mas escondê-las dentro de um IIFE, e expor apenas a que queremos:

```js
"use strict";

var sum = (function IIFE(){

    return function sum(...nums) {
        return sumRec( /*initialResult=*/0, ...nums );
    }

    function sumRec(result,num1,...nums) {
        result = result + num1;
        if (nums.length == 0) return result;
        return sumRec( result, ...nums );
    }

})();

sum( 3, 1, 17, 94, 8 );                             // 123
```

OK, temos CCA e temos uma assinatura limpa para nosso `sum(..)` que não requer que o chamador conheça os detalhes da nossa implementação. Yay!

Mas... uau, nossa função recursiva simples tem muito mais ruído agora. A legibilidade definitivamente foi reduzida. Isso é lamentável, para dizer o mínimo. Às vezes, isso é o melhor que podemos fazer.

Felizmente, em alguns outros casos, como o presente, há uma maneira melhor. Vamos voltar a essa versão:

```js
"use strict";

function sum(result,num1,...nums) {
    result = result + num1;
    if (nums.length == 0) return result;
    return sum( result, ...nums );
}

sum( /*initialResult=*/0, 3, 1, 17, 94, 8 );        // 123
```

O que você pode observar é que `result` é um número assim como `num1`, o que significa que podemos sempre tratar o primeiro número da nossa lista como nosso total corrente; isso inclui até mesmo a primeira chamada. Tudo o que precisamos é renomear esses parâmetros para deixar isso claro:

```js
"use strict";

function sum(num1,num2,...nums) {
    num1 = num1 + num2;
    if (nums.length == 0) return num1;
    return sum( num1, ...nums );
}

sum( 3, 1, 17, 94, 8 );                             // 123
```

Ótimo. Isso é muito melhor, não é!? Acho que esse padrão alcança um bom equilíbrio entre declarativo/razoável e performático.

Vamos tentar refatorar com CCA mais uma vez, revisitando nosso `maxEven(..)` anterior (atualmente não está em CCA). Vamos observar que, semelhante a manter a soma como o primeiro argumento, podemos reduzir a lista de números um por um, mantendo o primeiro argumento como o maior par que encontramos até agora.

Para clareza, a estratégia do algoritmo (semelhante ao que discutimos antes) que podemos usar:

1. Comece comparando os dois primeiros números, `num1` e `num2`.
2. `num1` é par e maior que `num2`? Se sim, mantenha `num1.
3. Se `num2` for par, mantenha-o (armazenado em `num1`).
4. Caso contrário, recorra a `undefined` (armazenado em `num1`).
5. Se houver mais `nums` a considerar, compare-os recursivamente com `num1`.
6. Finalmente, retorne o valor restante em `num1`.

Nosso código pode seguir esses passos quase exatamente:

```js
"use strict";

function maxEven(num1,num2,...nums) {
    num1 =
        (num1 % 2 == 0 && !(maxEven( num2 ) > num1)) ?
            num1 :
            (num2 % 2 == 0 ? num2 : undefined);

    return nums.length == 0 ?
        num1 :
        maxEven( num1, ...nums )
}
```

**Nota:** A primeira chamada `maxEven(..)` não está na posição CCA, mas como ela só passa `num2`, ela só recursiona um nível e então retorna de volta; isso é apenas um truque para evitar repetir a lógica `%`. Como tal, essa chamada não aumentará o crescimento da pilha recursiva, assim como se essa chamada fosse para uma função totalmente diferente. A segunda chamada `maxEven(..)` é a chamada recursiva legítima, e está de fato na posição CCA, o que significa que nossa pilha não crescerá conforme a recursão prossegue.

Vale repetir que esse exemplo é apenas para ilustrar a abordagem de mover a recursão para a forma CCA para otimizar o uso da pilha (memória). A maneira mais direta de expressar um algoritmo de máximo par pode de fato ser uma filtragem da lista `nums` para pares primeiro, seguida então por uma maximização ou até mesmo uma ordenação.

Refatorar a recursão para a forma CCA é, admitidamente, um pouco intrusivo na forma declarativa simples, mas ainda assim faz o trabalho de forma razoável. Infelizmente, alguns tipos de recursão não funcionarão bem mesmo com uma função de interface, então precisaremos de estratégias diferentes.

### Estilo de Passagem de Continuação (EPC)

Em JavaScript, a palavra *continuação* é frequentemente usada para se referir a uma função de callback que especifica o próximo passo(s) a ser(em) executado(s) após uma certa função terminar seu trabalho. Organizar o código de modo que cada função receba outra função para executar ao final é conhecido como Estilo de Passagem de Continuação (EPC).

Algumas formas de recursão não podem ser praticamente refatoradas para a forma pura de CCA, especialmente a recursão múltipla. Lembre-se da função `fib(..)` mencionada anteriormente, e até mesmo da forma de recursão mútua que derivamos. Em ambos os casos, há múltiplas chamadas recursivas, o que efetivamente impede a otimização de memória CCA.

No entanto, você pode realizar a primeira chamada recursiva e envolver as chamadas recursivas subsequentes em uma função de continuação para passar para essa primeira chamada. Embora isso signifique que muitas mais funções precisarão ser executadas na pilha, desde que todas elas, incluindo as continuações, estejam na forma CCA, o uso de memória da pilha não crescerá de forma ilimitada.

Podemos fazer isso para `fib(..)`:

```js
"use strict";

function fib(n,cont = identity) {
    if (n <= 1) return cont( n );
    return fib(
        n - 2,
        n2 => fib(
            n - 1,
            n1 => cont( n2 + n1 )
        )
    );
}
```

Preste atenção no que está acontecendo aqui. Primeiro, definimos a função de continuação `cont(..)` como nossa função [`identity(..)` utilitária do Capítulo 3](ch3.md/#one-on-one); lembre-se, ela simplesmente retorna o que é passado para ela.

Além disso, não é apenas uma, mas duas funções de continuação são adicionadas à mistura. A primeira recebe o argumento `n2`, que eventualmente recebe o valor computado de `fib(n-2)`. A próxima continuação interna recebe o argumento `n1`, que eventualmente é o valor de `fib(n-1)`. Uma vez que os valores de `n2` e `n1` são conhecidos, eles podem ser somados (`n2 + n1`), e esse valor é passado para o próximo passo de continuação `cont(..)`.

Talvez isso ajude a entender mentalmente o que está acontecendo: assim como na discussão anterior, quando passamos resultados parciais em vez de retorná-los após a pilha recursiva, estamos fazendo o mesmo aqui, mas cada passo é envolvido em uma continuação, que adia sua computação. Esse truque nos permite realizar múltiplos passos onde cada um está na forma CCA.

Em linguagens estáticas, o EPS muitas vezes é uma oportunidade para chamadas de cauda que o compilador pode identificar automaticamente e reorganizar o código recursivo para aproveitar isso. Infelizmente, isso não se aplica realmente à natureza do JS.

Em JavaScript, você provavelmente precisará escrever a forma EPS você mesmo. É mais complicado, com certeza; a forma declarativa foi certamente obscurecida. Mas, no geral, essa forma ainda é mais declarativa do que a implementação imperativa com `for`.

**Aviso:** Uma grande ressalva a ser observada é que, no EPS, criar as funções de continuação internas extras ainda consome memória, mas de um tipo diferente. Em vez de acumular frames de pilha, as closures consomem memória livre (tipicamente, da heap). Os motores não parecem aplicar os limites de `RangeError` nesses casos, mas isso não significa que seu uso de memória está fixo em escala.

### Trampolins

Onde o EPS cria continuações e as passa adiante, outra técnica para aliviar a pressão da memória é chamada de trampolins. Neste estilo de código, continuações semelhantes às do EPS são criadas, mas em vez de serem passadas, são retornadas de forma superficial.

Em vez de funções chamando funções, a pilha nunca vai além da profundidade de um, porque cada função simplesmente retorna a próxima função a ser chamada. Um loop simplesmente continua executando cada função retornada até que não haja mais funções para executar.

Uma vantagem dos trampolins é que você não está limitado a ambientes que suportam CCA; outra é que cada chamada de função é regular, não otimizada para CCA, então pode ser mais rápida.

Vamos esboçar uma utilidade `trampoline(..)`:

```js
function trampoline(fn) {
    return function trampolined(...args) {
        var result = fn( ...args );

        while (typeof result == "function") {
            result = result();
        }

        return result;
    };
}
```

Enquanto uma função é retornada, o loop continua, executando essa função e capturando seu retorno, e depois verificando seu tipo. Uma vez que um valor não-função é retornado, o trampolim assume que a chamada de função está completa e apenas devolve o valor.

Como cada continuação precisa retornar outra continuação, precisaremos usar o truque anterior de passar o resultado parcial como argumento. Aqui está como podemos usar essa utilidade com nosso exemplo anterior de soma de uma lista de números:

```js
var sum = trampoline(
    function sum(num1,num2,...nums) {
        num1 = num1 + num2;
        if (nums.length == 0) return num1;
        return () => sum( num1, ...nums );
    }
);

var xs = [];
for (let i=0; i<20000; i++) {
    xs.push( i );
}

sum( ...xs );                   // 199990000
```

A desvantagem é que um trampolim exige que você envolva sua função recursiva na função de controle de trampolim; além disso, assim como no EPS, closures são criadas para cada continuação. No entanto, ao contrário do EPS, cada função de continuação retornada é executada e finalizada imediatamente, então o motor não precisa acumular uma quantidade crescente de memória de closure enquanto a profundidade da pilha de chamadas do problema é esgotada.

Além do desempenho de execução e memória, a vantagem dos trampolins sobre o EPS é que eles são menos intrusivos na forma declarativa de recursão, já que você não precisa alterar a assinatura da função para receber um argumento de função de continuação. Trampolins não são ideais, mas podem ser eficazes na sua tentativa de equilibrar entre código imperativo com loops e recursão declarativa.

## Resumo

Recursão é quando uma função chama a si mesma recursivamente. Heh. Uma definição recursiva para recursão. Entendeu!?

Recursão direta é uma função que faz pelo menos uma chamada a si mesma, e continua se chamando até satisfazer uma condição base. A recursão múltipla (como a recursão binária) é quando uma função se chama múltiplas vezes. Recursão mútua é quando duas ou mais funções se chamam recursivamente de forma *mútua*.

O lado positivo da recursão é que ela é mais declarativa e, portanto, tipicamente mais legível. O lado negativo é geralmente o desempenho, mas mais restrições de memória do que a velocidade de execução.

Chamadas de cauda aliviam a pressão da memória ao reutilizar/descartar frames da pilha. O JavaScript requer modo estrito e chamadas de cauda apropriadas (CCA) para aproveitar essa "otimização". Existem várias técnicas que podemos combinar para refatorar uma função recursiva não-CCA para a forma CCA, ou pelo menos evitar as restrições de memória achatando a pilha.

Lembre-se: a recursão deve ser usada para tornar o código mais legível. Se você usar ou abusar da recursão, a legibilidade acabará pior do que a forma imperativa. Não faça isso!
