# Functional-Light JavaScript

# Apêndice A: Transducing (Transducing)

Transducing é uma das técnicas mais avançadas que abordamos neste livro. Ela estende muitos dos conceitos do [Capítulo 9](ch9.md) sobre operações em listas.

Eu não chamaria necessariamente esse tópico de "Functional-Light" estritamente, mas seria uma espécie de bônus. Apresentei isso como um apêndice porque você pode muito bem precisar pular a discussão por enquanto e voltar a ela quando se sentir mais confortável, e garantir que tenha praticado os conceitos principais do livro.

Para ser honesto, mesmo após ensinar transducing várias vezes e escrever este capítulo, ainda estou tentando compreender completamente essa técnica. Portanto, não se sinta mal se isso te confundir. Marque este apêndice e volte quando estiver pronto.

Transducing significa transformar com redução.

Sei que isso pode parecer um amontoado de palavras que confunde mais do que esclarece. Mas vamos dar uma olhada em como isso pode ser poderoso. Na verdade, acho que é uma das melhores ilustrações do que você pode fazer uma vez que compreende os princípios da Programação Functional-Light.

Como no restante deste livro, minha abordagem é primeiro explicar o _porquê_, depois o _como_ e finalmente simplificar em um _o que_ replicável. Isso geralmente é o oposto de como muitos ensinam, mas acho que você aprenderá o tópico mais profundamente dessa maneira.

## Primeiro, Porquê

Vamos começar estendendo um [cenário que abordamos no Capítulo 3](https://github.com/armadilha/functional-light-js/blob/main/manuscript/ch3.md), testando palavras para ver se são curtas o suficiente e/ou longas o suficiente:

```js
function isLongEnough(str) {
    return str.length >= 5;
}

function isShortEnough(str) {
    return str.length <= 10;
}
```

No [Capítulo 3, usamos essas funções de predicado](https://github.com/armadilha/functional-light-js/blob/main/manuscript/ch3.md) para testar uma única palavra. Então, no Capítulo 9, aprendemos a repetir tais testes [usando operações de lista como `filter(..)`](https://github.com/armadilha/functional-light-js/blob/main/manuscript/ch9.md/#chapter-9-list-operations). Por exemplo:

```js
var words = ["You", "have", "written", "something", "very", "interesting"];

words.filter(isLongEnough).filter(isShortEnough);
// ["written","something"]
```

Pode não ser óbvio, mas esse padrão de operações de lista adjacentes separadas tem algumas características não ideais. Quando lidamos apenas com um único array de um pequeno número de valores, está tudo bem. Mas se houver muitos valores no array, cada `filter(..)` processando a lista separadamente pode tornar as coisas um pouco mais lentas do que gostaríamos.

Um problema semelhante de desempenho surge quando nossos arrays são assíncronos/lazy (também conhecidos como Observables), processando valores ao longo do tempo em resposta a eventos (veja [Capítulo 10](https://github.com/armadilha/functional-light-js/blob/main/manuscript/ch10.md)). Nesse cenário, apenas um único valor desce no fluxo de eventos por vez, então processar esse valor discreto com duas chamadas de função `filter(..)` separadas não é realmente um grande problema.

Mas o que não é óbvio é que cada método `filter(..)` produz um observable separado. A sobrecarga de bombear um valor de um observable para outro pode realmente se acumular. Isso é especialmente verdade, já que nesses casos, não é incomum que milhares ou milhões de valores sejam processados; até mesmo pequenos custos de sobrecarga se acumulam rapidamente.

A outra desvantagem é a legibilidade, especialmente quando precisamos repetir a mesma série de operações contra múltiplas listas (ou Observables). Por exemplo:

```js
zip(
    list1.filter(isLongEnough).filter(isShortEnough),
    list2.filter(isLongEnough).filter(isShortEnough),
    list3.filter(isLongEnough).filter(isShortEnough)
);
```

Repetitivo, certo?

Não seria melhor (tanto para legibilidade quanto para desempenho) se pudéssemos combinar o predicado `isLongEnough(..)` com o predicado `isShortEnough(..)`? Você poderia fazer isso manualmente:

```js
function isCorrectLength(str) {
    return isLongEnough(str) && isShortEnough(str);
}
```

Mas isso não é o jeito FP (Functional Programming - Programação Funcional no original) !

No [Capítulo 9, falamos sobre fusão](https://github.com/armadilha/functional-light-js/blob/main/manuscript/ch9.md) -- compondo funções de mapeamento adjacentes. Lembre-se:

```js
words.map(pipe(removeInvalidChars, upper, elide));
```

Infelizmente, combinar funções de predicado adjacentes não funciona tão facilmente quanto combinar funções de mapeamento adjacentes. Para entender por quê, pense na "forma" da função de predicado -- uma maneira acadêmica de descrever a assinatura de entradas e saída. Ela recebe um único valor e retorna `true` ou `false`.

Se você tentar `isShortEnough(isLongEnough(str))`, não funcionará corretamente. `isLongEnough(..)` retornará `true`/`false`, não o valor da string que `isShortEnough(..)` está esperando. Que pena.

Uma frustração semelhante existe ao tentar compor duas funções redutoras adjacentes. A "forma" de um reducer é uma função que recebe dois valores como entrada e retorna um valor combinado. A saída de um reducer como um único valor não é adequada para a entrada de outro reducer que espera dois valores.

Além disso, o helper `reduce(..)` aceita uma entrada `initialValue` opcional. Às vezes isso pode ser omitido, mas às vezes precisa ser passado. Isso complica ainda mais a composição, já que uma redução pode precisar de um `initialValue` e outra redução pode parecer precisar de um `initialValue` diferente. Como podemos fazer isso se fizermos apenas uma chamada `reduce(..)` com algum tipo de reducer composto?

Considere uma cadeia como esta:

```js
words
    .map(strUppercase)
    .filter(isLongEnough)
    .filter(isShortEnough)
    .reduce(strConcat, "");
// "WRITTENSOMETHING"
```

Você pode imaginar uma composição que inclua todas essas etapas: `map(strUppercase)`, `filter(isLongEnough)`, `filter(isShortEnough)`, `reduce(strConcat)`? A forma de cada operador é diferente, então eles não se compõem diretamente. Precisamos dobrar suas formas um pouco para encaixá-las juntas.

Espero que essas observações tenham ilustrado por que a composição simples no estilo de fusão não é suficiente. Precisamos de uma técnica mais poderosa, e transducing é essa ferramenta.

## O Como, O Próximo

Vamos falar sobre como podemos derivar uma composição de mappers, predicados e/ou redutores.

Não fique muito sobrecarregado: você não precisará passar por todas essas etapas mentais que estamos prestes a explorar em sua própria programação. Uma vez que você entenda e possa reconhecer o problema que a transducing resolve, poderá simplesmente pular direto para usar uma utilidade `transduce(..)` de uma biblioteca FP e seguir em frente com o resto de sua aplicação!

Vamos começar.

### Expressando Map/Filter como Reduce

O primeiro truque que precisamos realizar é expressar nossas chamadas `filter(..)` e `map(..)` como chamadas `reduce(..)`. Lembre-se de [como fizemos isso no Capítulo 9](https://github.com/armadilha/functional-light-js/blob/main/manuscript/ch9.md):

```js
function strUppercase(str) {
    return str.toUpperCase();
}
function strConcat(str1, str2) {
    return str1 + str2;
}

function strUppercaseReducer(list, str) {
    list.push(strUppercase(str));
    return list;
}

function isLongEnoughReducer(list, str) {
    if (isLongEnough(str)) list.push(str);
    return list;
}

function isShortEnoughReducer(list, str) {
    if (isShortEnough(str)) list.push(str);
    return list;
}

words
    .reduce(strUppercaseReducer, [])
    .reduce(isLongEnoughReducer, [])
    .reduce(isShortEnoughReducer, [])
    .reduce(strConcat, "");
// "WRITTENSOMETHING"
```

Isso é uma melhoria decente. Agora temos quatro chamadas `reduce(..)` adjacentes em vez de uma mistura de três métodos diferentes, todos com formas diferentes. Ainda não podemos simplesmente usar o `compose(..)` com esses quatro redutores, porque eles aceitam dois argumentos em vez de um.

No [Capítulo 9, meio que trapaceamos](https://github.com/armadilha/functional-light-js/blob/main/manuscript/ch9.md) e usamos `list.push(..)` para mutar como um efeito colateral, em vez de criar um array totalmente novo para concatenar. Vamos retroceder e ser um pouco mais formais por enquanto:

```js
function strUppercaseReducer(list, str) {
    return [...list, strUppercase(str)];
}

function isLongEnoughReducer(list, str) {
    if (isLongEnough(str)) return [...list, str];
    return list;
}

function isShortEnoughReducer(list, str) {
    if (isShortEnough(str)) return [...list, str];
    return list;
}
```

Mais tarde, revisaremos se criar um novo array (por exemplo, `[...list,str]`) para concatenar é necessário aqui ou não.

### Parametrizando os Reducers

Ambos os reducers de filtro são quase idênticos, exceto que usam uma função de predicado diferente. Vamos parametrizar isso para obter uma utilidade que pode definir qualquer filtro-reducer:

```js
function filter

Reducer(predicateFn) {
    return function reducer(list,val){
        if (predicateFn( val )) return [ ...list, val ];
        return list;
    };
}

var isLongEnoughReducer = filterReducer( isLongEnough );
var isShortEnoughReducer = filterReducer( isShortEnough );
```

Vamos fazer a mesma parametrização da `mapperFn(..)` para uma utilidade que produza qualquer map-reducer:

```js
function mapReducer(mapperFn) {
    return function reducer(list, val) {
        return [...list, mapperFn(val)];
    };
}

var strUppercaseReducer = mapReducer(strUppercase);
```

Nossa cadeia ainda parece a mesma:

```js
words
    .reduce(strUppercaseReducer, [])
    .reduce(isLongEnoughReducer, [])
    .reduce(isShortEnoughReducer, [])
    .reduce(strConcat, "");
```

### Extraindo Lógica de Combinação Comum

Observe bem de perto as funções `mapReducer(..)` e `filterReducer(..)` anteriores. Você vê a funcionalidade comum compartilhada em cada uma?

Esta parte:

```js
return [ ...list, .. ];

// ou
return list;
```

Vamos definir um helper para essa lógica comum. Mas como devemos chamá-lo?

```js
function WHATSITCALLED(list, val) {
    return [...list, val];
}
```

Se você examinar o que essa função `WHATSITCALLED(..)` faz, ela pega dois valores (um array e outro valor) e os "combina" criando um novo array e concatenando o valor ao final dele. Nada criativo, poderíamos chamar isso de `listCombine(..)`:

```js
function listCombine(list, val) {
    return [...list, val];
}
```

Vamos redefinir nossos helpers reducers para usar o `listCombine(..)`:

```js
function mapReducer(mapperFn) {
    return function reducer(list, val) {
        return listCombine(list, mapperFn(val));
    };
}

function filterReducer(predicateFn) {
    return function reducer(list, val) {
        if (predicateFn(val)) return listCombine(list, val);
        return list;
    };
}
```

Nossa cadeia ainda parece a mesma (então não vamos repeti-la).

### Parametrizando a Combinação

Nossa simples utilidade `listCombine(..)` é apenas uma maneira possível de combinar dois valores. Vamos parametrizar o uso dela para tornar nossos reducers mais genéricos:

```js
function mapReducer(mapperFn, combinerFn) {
    return function reducer(list, val) {
        return combinerFn(list, mapperFn(val));
    };
}

function filterReducer(predicateFn, combinerFn) {
    return function reducer(list, val) {
        if (predicateFn(val)) return combinerFn(list, val);
        return list;
    };
}
```

Para usar esta forma dos nossos helpers:

```js
var strUppercaseReducer = mapReducer(strUppercase, listCombine);
var isLongEnoughReducer = filterReducer(isLongEnough, listCombine);
var isShortEnoughReducer = filterReducer(isShortEnough, listCombine);
```

Definir essas utilidades para aceitar dois argumentos em vez de um é menos conveniente para composição, então vamos usar nossa abordagem `curry(..)`:

```js
var curriedMapReducer = curry(function mapReducer(mapperFn, combinerFn) {
    return function reducer(list, val) {
        return combinerFn(list, mapperFn(val));
    };
});

var curriedFilterReducer = curry(function filterReducer(
    predicateFn,
    combinerFn
) {
    return function reducer(list, val) {
        if (predicateFn(val)) return combinerFn(list, val);
        return list;
    };
});

var strUppercaseReducer = curriedMapReducer(strUppercase)(listCombine);
var isLongEnoughReducer = curriedFilterReducer(isLongEnough)(listCombine);
var isShortEnoughReducer = curriedFilterReducer(isShortEnough)(listCombine);
```

Isso parece um pouco mais verboso, e provavelmente não parece muito útil.

Mas isso é realmente necessário para chegar ao próximo passo de nossa derivação. Lembre-se, nosso objetivo final aqui é ser capaz de utilizar o `compose(..)` nesses reducers. Estamos quase lá.

### Compondo Curried

Esta etapa é a mais difícil de visualizar. Então, leia devagar e preste muita atenção aqui.

Vamos considerar as funções curried de antes, mas sem a função `listCombine(..)` passada para cada uma:

```js
var x = curriedMapReducer(strUppercase);
var y = curriedFilterReducer(isLongEnough);
var z = curriedFilterReducer(isShortEnough);
```

Pense na forma de todas essas três funções intermediárias, `x(..)`, `y(..)` e `z(..)`. Cada uma espera uma única função de combinação e produz uma função reducer com ela.

Lembre-se, se quiséssemos os reducers independentes de todos esses, poderíamos fazer:

```js
var upperReducer = x(listCombine);
var longEnoughReducer = y(listCombine);
var shortEnoughReducer = z(listCombine);
```

Mas o que você obteria de volta se chamasse `y(z)`, em vez de `y(listCombine)`? Basicamente, o que acontece ao passar `z` como a função `combinerFn(..)` para a chamada `y(..)`? Essa função reducer retornada internamente parece algo assim:

```js
function reducer(list, val) {
    if (isLongEnough(val)) return z(list, val);
    return list;
}
```

Veja, a chamada interna de `z(..)` . Isso deve parecer errado para você, porque a função `z(..)` deve receber apenas um único argumento (uma `combinerFn(..)`), não dois argumentos (`list` e `val`). As formas não correspondem. Isso não funcionará.

Vamos, em vez disso, olhar para a composição `y(z(listCombine))`. Vamos dividir isso em duas etapas separadas:

```js
var shortEnoughReducer = z(listCombine);
var longAndShortEnoughReducer = y(shortEnoughReducer);
```

Criamos `shortEnoughReducer(..)`, então o passamos como a função `combinerFn(..)` para `y(..)` em vez de chamar `y(listCombine)`; essa nova chamada produz `longAndShortEnoughReducer(..)`. Releia isso algumas vezes até entender.

Agora, considere: como `shortEnoughReducer(..)` e `longAndShortEnoughReducer(..)` se parecem internamente? Você pode vê-los em sua mente?

```js
// shortEnoughReducer, da chamada z(..):
function reducer(list, val) {
    if (isShortEnough(val)) return listCombine(list, val);
    return list;
}

// longAndShortEnoughReducer, da chamada y(..):
function reducer(list, val) {
    if (isLongEnough(val)) return shortEnoughReducer(list, val);
    return list;
}
```

Você vê como `shortEnoughReducer(..)` tomou o lugar de `listCombine(..)` dentro de `longAndShortEnoughReducer(..)`? Por que isso funciona?

Porque **a forma de um `reducer(..)` e a forma de `listCombine(..)` são as mesmas.** Em outras palavras, um reducer pode ser usado como uma função de combinação para outro reducer; é assim que eles se compõem! A função `listCombine(..)` faz o primeiro reducer, então _esse redutor_ pode ser usado como a função de combinação para fazer o próximo reducer, e assim por diante.

Vamos testar nosso `longAndShortEnoughReducer(..)` com alguns valores diferentes:

```js
longAndShortEnoughReducer([], "nope");
// []

longAndShortEnoughReducer([], "hello");
// ["hello"]

longAndShortEnoughReducer([], "hello world");
// []
```

A utilidade `longAndShortEnoughReducer(..)` está filtrando tanto valores que não são longos o suficiente quanto valores que não são curtos o suficiente, e está fazendo ambos esses filtros na mesma etapa. É um reducer composto!

Tire mais um momento para deixar isso entrar na sua mente. Isso ainda meio que me impressiona.

Agora, para trazer `x(..)` (o produtor de reducer de maiúsculas) para a composição:

```js
var longAndShortEnoughReducer = y(z(listCombine));
var upperLongAndShortEnoughReducer = x(longAndShortEnoughReducer);
```

Como o nome `upperLongAndShortEnoughReducer(..)` sugere, ele realiza todas as três etapas de uma vez -- um mapeamento e dois filtros! Como isso se parece internamente:

```js
// upperLongAndShortEnoughReducer:
function reducer(list, val) {
    return longAndShortEnoughReducer(list, strUppercase(val));
}
```

Uma string `val` é passada, transformada em maiúsculas por `strUppercase(..)` e então passada para `longAndShortEnoughReducer(..)`. _Essa_ função apenas condicionalmente adiciona essa string em maiúsculas à `list` se ela for longa o suficiente e curta o suficiente. Caso contrário, `list` permanecerá inalterada.

Demorei semanas para entender completamente as implicações dessa ginástica mental. Então, não se preocupe se você precisar parar aqui e reler algumas (muitas!) vezes para entender. Vá com calma.

Agora vamos verificar:

```js
upperLongAndShortEnoughReducer([], "nope");
// []

upperLongAndShortEnoughReducer([], "hello");
// ["HELLO"]

upperLongAndShortEnoughReducer([], "hello world");
// []
```

Este reducer é a composição do mapeamento e ambos os filtros! Isso é incrível!

Vamos recapitular onde estamos até agora:

```js
var x = curriedMapReducer(strUppercase);
var y = curriedFilterReducer(isLongEnough);
var z = curriedFilterReducer(isShortEnough);

var upperLongAndShortEnoughReducer = x(y(z(listCombine)));

words.reduce(upperLongAndShortEnoughReducer, []);
// ["WRITTEN","SOMETHING"]
```

Isso é muito legal. Mas vamos melhorar ainda mais.

`x(y(z( .. )))` é uma composição. Vamos pular os nomes de variáveis intermediárias `x` / `y` / `z` e apenas expressar essa composição diretamente:

```js
var composition = compose(
    curriedMapReducer(strUppercase),
    curriedFilterReducer(isLongEnough),
    curriedFilterReducer(isShortEnough)
);

var upperLongAndShortEnoughReducer = composition(listCombine);

words.reduce(upperLongAndShortEnoughReducer, []);
// ["WRITTEN","SOMETHING"]
```

Pense no fluxo de "dados" nessa função composta:

1. O `listCombine(..)` flui como a função de combinação para fazer o reducer de filtro para `isShortEnough(..)`.
2. _Esse_ reducer resultante então flui como a função de combinação para fazer o reducer de filtro para `isLongEnough(..)`.
3. Finalmente, _esse_ reducer resultante flui como a função de combinação para fazer o reducer de mapeamento para `strUppercase(..)`.

No trecho anterior, `composition(..)` é uma função composta esperando uma função de combinação para fazer um reducer; `composition(..)` aqui tem um nome especial: transducer (transdutor). Quando se fornece a função de combinação para um transducer produz o reducer composto:

```js
var transducer = compose(
    curriedMapReducer(strUppercase),
    curriedFilterReducer(isLongEnough),
    curriedFilterReducer(isShortEnough)
);

words.reduce(transducer(listCombine), []);
// ["WRITTEN","SOMETHING"]
```

**Nota:** Devemos fazer uma observação sobre a ordem `compose(..)` nos dois trechos anteriores, porque isso pode ser confuso. Lembre-se de que em nossa cadeia de exemplos original, fizemos `map(strUppercase)`, depois `filter(isLongEnough)` e, finalmente, `filter(isShortEnough)`; essas operações de fato acontecem nessa ordem. Mas no [Capítulo 4](https://github.com/armadilha/functional-light-js/blob/main/manuscript/ch4.md), aprendemos que `compose(..)` normalmente tem o efeito de executar suas funções na ordem inversa da listagem. Então, por que não precisamos inverter a ordem _aqui_ para obter o mesmo resultado desejado? A abstração da função `combinerFn(..)` de cada reducer inverte a ordem de operações aplicada sob o capô. Então, contra-intuitivamente, ao compor um transducer, você realmente deseja listá-los na ordem desejada de execução!

#### Combinação de Lista: Pura vs. Impura

Como uma observação rápida, vamos revisitar nossa implementação da função de combinação `listCombine(..)`:

```js
function listCombine(list, val) {
    return [...list, val];
}
```

Embora essa abordagem seja pura, ela tem consequências negativas para o desempenho: para cada etapa na redução, estamos criando um array totalmente novo para anexar o valor, efetivamente descartando o array anterior. Isso é um monte de arrays sendo criados e descartados, o que não é apenas ruim para o CPU, mas também para a coleta de lixo (Garbage Colector - GC).

Por outro lado, olhe novamente para a versão melhorada, mas impura:

```js
function listCombine(list, val) {
    list.push(val);
    return list;
}
```

Pensando sobre `listCombine(..)` isoladamente, não há dúvida de que é impura e isso geralmente é algo que gostaríamos de evitar. No entanto, há um contexto maior que devemos considerar.

A `listCombine(..)` não é uma função com a qual interagimos diretamente. Não a usamos diretamente em nenhum lugar do programa; em vez disso, deixamos o processo de transducing usá-la.

No [Capítulo 5](https://github.com/armadilha/functional-light-js/blob/main/manuscript/ch5.md), afirmamos que nosso objetivo com a redução de efeitos colaterais e a definição de funções puras era apenas expor funções puras no nível da API de funções que usaremos em todo o programa. Observamos que, nos bastidores, dentro de uma função pura, ela pode trapacear para ganho de desempenho, desde que não viole o contrato externo de pureza.

A `listCombine(..)` é mais um detalhe de implementação interna da transducing -- na verdade, muitas vezes será fornecida pela biblioteca de transducing para você! -- em vez de um método de nível superior com o qual você interagiria normalmente ao longo de seu programa.

Conclusão: acho que é perfeitamente aceitável, e até aconselhável, usar a versão impura e otimizada para desempenho de `listCombine(..)`. Apenas certifique-se de documentar que ela é impura com um comentário no código!

### Combinação Alternativa

Até agora, isso é o que derivamos com transducing:

```js
words.reduce(transducer(listCombine), []).reduce(strConcat, "");
// WRITTENSOMETHING
```

Isso é muito bom, mas temos um truque final na manga com transducing. E, sinceramente, acho que essa parte é o que torna todo esse esforço mental que você fez até agora realmente valioso.

Podemos, de alguma forma, "compor" essas duas chamadas `reduce(..)` para reduzi-las a apenas uma chamada `reduce(..)`? Infelizmente, não podemos simplesmente adicionar `strConcat(..)` na chamada `compose(..)`; porque é um reducer e não uma função que espera uma combinação, sua forma não é adequada para a composição.

Mas vamos olhar para essas duas funções lado a lado:

```js
function strConcat(str1, str2) {
    return str1 + str2;
}

function listCombine(list, val) {
    list.push(val);
    return list;
}
```

Se você entrecerrar os olhos, quase pode ver como essas duas funções são intercambiáveis. Elas operam com tipos de dados diferentes, mas conceitualmente fazem a mesma coisa: combinam dois valores em um.

Em outras palavras, `strConcat(..)` é uma função de combinação!

Isso significa que podemos usá-la em vez de `listCombine(..)` se nosso objetivo final for obter uma concatenação de strings em vez de uma lista:

```js
words.reduce(transducer(strConcat), "");
// WRITTENSOMETHING
```

Boom! Isso é transducing para você. Eu não vou realmente soltar o microfone aqui, mas apenas colocá-lo suavemente...

## O Que, Finalmente

Respire fundo. Isso foi muita coisa para digerir.

Limpando nossas mentes por um minuto, vamos voltar nossa atenção para apenas usar transducing em nossas aplicações sem passar por todos esses aros mentais para derivar como ela funciona.

Lembre-se dos helpers que definimos anteriormente; vamos renomeá-los para clareza:

```js
var transduceMap = curry(function mapReducer(mapperFn, combinerFn) {
    return function reducer(list, v) {
        return combinerFn(list, mapperFn(v));
    };
});

var transduceFilter = curry(function filterReducer(predicateFn, combinerFn) {
    return function reducer(list, v) {
        if (predicateFn(v)) return combinerFn(list, v);
        return list;
    };
});
```

Também lembre-se de que os usamos assim:

```js
var transducer = compose(
    transduceMap(strUppercase),
    transduceFilter(isLongEnough),
    transduceFilter(isShortEnough)
);
```

`transducer(..)` ainda precisa de uma função de combinação (como `listCombine(..)` ou `strConcat(..)`) passada para ele para produzir uma função de reducer de transducing, que então pode ser usada (junto com um valor inicial) em `reduce(..)`.

Mas para expressar todas essas etapas de transducing de forma mais declarativa, vamos fazer uma utilidade `transduce(..)` que faça essas etapas por nós:

```js
function transduce(transducer, combinerFn, initialValue, list) {
    var reducer = transducer(combinerFn);
    return list.reduce(reducer, initialValue);
}
```

Aqui está nosso exemplo em execução, limpo:

```js
var transducer = compose(
    transduceMap(strUppercase),
    transduceFilter(isLongEnough),
    transduceFilter(isShortEnough)
);

transduce(transducer, listCombine, [], words);
// ["WRITTEN","SOMETHING"]

transduce(transducer, strConcat, "", words);
// WRITTENSOMETHING
```

Nada mal, hein!? Está vendo as funções `listCombine(..)` e `strConcat(..)` sendo usadas de forma intercambiável como funções de combinação?

### Transducers.js

Finalmente, vamos ilustrar nosso exemplo em execução usando a biblioteca [`transducers-js`](https://github.com/cognitect-labs/transducers-js):

```js
var transformer = transducers.comp(
    transducers.map(strUppercase),
    transducers.filter(isLongEnough),
    transducers.filter(isShortEnough)
);

transducers.transduce(transformer, listCombine, [], words);
// ["WRITTEN","SOMETHING"]

transducers.transduce(
    transformer,
    strConcat,

    "",
    words
);
// WRITTENSOMETHING
```

Parece quase idêntico com o de cima.

**Nota:** O trecho anterior usa `transformers.comp(..)` porque a biblioteca o fornece, mas neste caso, nossa [`compose(..)` do Capítulo 4](https://github.com/armadilha/functional-light-js/blob/main/manuscript/ch4.md) produziria o mesmo resultado. Em outras palavras, a composição em si não é uma operação sensível ao transducing.

A função composta neste trecho é chamada de `transformer` em vez de `transducer`. Isso porque, se chamarmos `transformer( listCombine )` (ou `transformer( strConcat )`), não obteremos uma função de reducer de transducing diretamente como anteriormente.

O `transducers.map(..)` e o `transducers.filter(..)` são helpers especiais que adaptam funções de predicado ou mapeamento regulares em funções que produzem um objeto de transformação especial (com a função de transducing embutida); a biblioteca usa esses objetos de transformação para transducing. As capacidades extras dessa abstração de objeto de transformação vão além do que exploraremos, então consulte a documentação da biblioteca para mais informações.

Como chamar o `transformer(..)` produz um objeto de transformação e não uma função reducer de transducing binária típica, a biblioteca também fornece `toFn(..)` para adaptar o objeto de transformação para ser utilizável pelo `reduce(..)` de array nativo:

```js
words.reduce(transducers.toFn(transformer, strConcat), "");
// WRITTENSOMETHING
```

O `into(..)` é outro helper fornecido que seleciona automaticamente uma função de combinação padrão com base no tipo de valor inicial/esvaziado especificado:

```js
transducers.into([], transformer, words);
// ["WRITTEN","SOMETHING"]

transducers.into("", transformer, words);
// WRITTENSOMETHING
```

Ao especificar um array vazio `[]`, a chamada `transduce(..)` feita por baixo dos panos usa uma implementação padrão de uma função como nosso helper `listCombine(..)`. Mas ao especificar uma string vazia `""`, algo como nosso `strConcat(..)` é usado. Legal!

Como você pode ver, a biblioteca `transducers-js` torna a transducing bastante simples. Podemos aproveitar muito efetivamente o poder dessa técnica sem entrar nos detalhes de definir todas essas utilidades intermediárias que produzem os transducers.

## Resumo

Transducing significa transformar com um reduce. Mais especificamente, um transducer é um reducer composível.

Usamos o transducing para compor operações `map(..)`, `filter(..)` e `reduce(..)` adjacentes juntas. Conseguimos isso primeiro expressando `map(..)`s e `filter(..)`s como `reduce(..)`s, e depois abstraindo a operação comum de combinação para criar funções que produzem redutores unários que são facilmente compostos.

O transducing melhora principalmente o desempenho, o que é especialmente óbvio se usado em um observable.

Mas de forma mais ampla, o transducing é como expressamos uma composição mais declarativa de funções que de outra forma não seriam diretamente composíveis. O resultado, se usado de forma apropriada como todas as outras técnicas neste livro, é um código mais claro e legível! Uma única chamada `reduce(..)` com um transducer é mais fácil de entender do que o rastreio de várias chamadas `reduce(..)`s.
