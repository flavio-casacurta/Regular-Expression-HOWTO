========================
Regular Expression HOWTO
========================
Author:
-------
**A.M. Kuchling <amk@amk.ca>**

Site:
-----
https://docs.python.org/2/howto/regex.html

Traduzido:
----------
Flávio Augusto Casacurta <flavio@casacurta.com>

Revisado:
---------
Herbert Parentes Fortes Neto <hpfn@ig.com.br>


Abstrato
--------
Este documento é um tutorial introdutório ao uso de expressões regulares em Python
com o módulo re. Ele fornece uma introdução mais suave do que a seção
correspondente na Biblioteca de Referência.

Introdução
----------
O módulo ``re`` foi adicionado no Python 1.5, e provê padrões de expressões regulares
no “estilo Perl”. As versões anteriores do Python vieram com o módulo regex, que
forneceu os padrões “estilo Emacs”. O módulo regex foi completamente removido
do Python 2.5.

As expressões regulares (chamadas de REs, regexes, ou padrões de regex)
são, essencialmente, uma pequena linguagem de programação, altamente
especializada, embutida dentro do Python e disponibilizadas através do módulo
re. Utilizando esta pequena linguagem, você especifica as regras para o conjunto de
possíveis strings com as quais você deseja corresponder, este conjunto pode
conter frases em inglês, endereços de e-mail, comandos TeX ou qualquer coisa que você
queira. Você pode então fazer perguntas como "Será que esta string corresponde
ao padrão?" ou "Existe uma correspondência para o padrão em qualquer lugar nesta
string?". Você também pode usar REs para modificar uma string ou dividi-la
de várias maneiras.

Padrões de expressões regulares são compilados em uma série de bytecodes que são
então executados por um mecanismo de correspondência escrito em C. Para uso avançado,
pode ser necessário prestar muita atenção à forma como o mecanismo irá executar
uma RE informada, e escrever a RE de uma certa maneira, a fim de produzir um bytecode que seja
executado de forma mais rápida. A otimização não é abordada neste documento, porque ela requer que
você tenha um bom entendimento interno do mecanismo de correspondência.

A linguagem de expressão regular é relativamente pequena e restrita, por isso nem
todas as tarefas de processamento de strings possíveis podem ser feitas usando
expressões regulares. Existem também tarefas que podem ser feitas com expressões
regulares, mas as expressões acabam por ser tornar muito complicadas. Nestes casos, pode
ser melhor para você escrever um código Python para fazer o processamento;
embora um código Python seja mais lento do que uma expressão regular elaborada,
ele provavelmente será mais compreensível.

Padrões Simples
---------------
Vamos começar por aprender sobre as expressões regulares mais simples possíveis.
Como as expressões regulares são usadas para operar em strings, vamos começar
com a tarefa mais comum: de correspondência caracteres.

Para uma explicação detalhada da ciência da computação referente a expressões
regulares (autômatos finitos determinísticos e não-determinístico), você pode consultar
a praticamente qualquer livro sobre a escrita de compiladores.

Caracteres Correspondentes
--------------------------
A maioria das letras e caracteres simplesmente irão corresponder entre si. Por exemplo, a expressão regular ``teste``
irá combinar com a string ``teste`` totalmente. (Você pode habilitar o modo de maiúsculas e minúsculas que faria com que
a RE corresponder com ``Test`` ou ``TEST`` também; veremos mais sobre isso mais adiante.)

Há exceções a essa regra, alguns caracteres são metacaracteres especiais, e não se
correspondem. Em vez disso, eles sinalizam que alguma coisa fora do normal deve
ser correspondida, ou eles afetam outras partes da RE, repetindo-as ou alterando seus
significados. Grande parte deste documento é dedicada à discussão de vários metacaracteres
e o que eles fazem.

Aqui está a lista completa dos metacaracteres; seus significados serão discutidos ao
longo deste documento.::

. ^ $ * + ? { } [ ] \ | ( )

O primeiro metacaractere que vamos olhar são os colchetes, ``[`` e ``]``. Eles são usados para
especificar uma classe de caracteres, que é um conjunto de caracteres que você
deseja corresponder. Os caracteres podem ser listados individualmente, ou um
intervalo de caracteres pode ser indicado informando dois caracteres e separando-os por
um ``-``. Por exemplo, ``[abc]`` irá corresponder a qualquer dos caracteres ``a``, ``b``, ``c`` ou, o que
é o mesmo que ``[a-c]``, que usa um intervalo de expressar o mesmo conjunto de
caracteres. Se você quiser corresponder apenas letras minúsculas, a RE seria ``[a-z]``.

Metacaracteres não são ativos dentro classes ``[ ]``. Por exemplo, ``[akm$]`` irá
corresponder a qualquer um dos caracteres ``a``, ``k``, ``m``, ou ``$``; ``$`` é geralmente um
metacaractere, mas dentro de uma classe de caracteres ele é despojado de sua natureza
especial.

Você pode combinar os caracteres ``não listados`` dentro de um classe,
através de um complemento do conjunto. Isto é indicado através da inclusão de um ``^`` como o
primeiro caractere da classe; ``^`` fora de uma classe de caracteres simplesmente
corresponder ao caractere ``^``. Mas por exemplo, ``[^5]`` irá corresponder a qualquer caractere,
exceto ``5``.

Talvez o metacaractere mais importante é a barra invertida, ``\``. Como as strings literais em
Python, a barra invertida pode ser seguida por vários caracteres para sinalizar várias
sequências especiais. Ela também é usada para ``escapar`` todos os metacaracteres,
e assim, você poder combiná-los em padrões; por exemplo, se você precisa
fazer correspondência a um ``[`` ou ``\``, você pode precedê-los com uma barra invertida para
remover seu significado especial: ``\[`` ou ``\\``.

Algumas das sequências especiais que começam com ``\`` representam conjuntos
predefinidos de caracteres que muitas vezes são úteis, como o conjunto de dígitos, o
conjunto de letras, ou o conjunto de qualquer coisa que não seja um espaço em branco.
As seguintes sequências especiais pré-definidas são um subconjunto das disponíveis. As classes
equivalentes são para padrões de "byte string" (byte string patterns). Para uma lista completa de
definições de sequências e classe estendida para os padrões string
Unicode, consulte a última parte de 'Sintaxe de Expressões Regulares'.

**\\d** corresponde a qualquer ``dígito decimal``, que é equivalente à classe ``[0-9]``.

**\\D** corresponde a qualquer caractere ``não-dígito``, o que é equivalente à classe ``[^0-9]``.

**\\s** corresponde a qualquer caractere ``espaço-em-branco``, o que é equivalente à classe ``[\t\n\r\f\v]``.

**\\S** corresponde a qualquer caractere ``não-espaço-branco``, o que é equivalente à classe ``[^\t\n\r\f\v].``

**\\w** corresponde a qualquer caractere ``alfanumérico``, o que é equivalente à classe ``[azA-Z0-9_]``.

**\\W** Corresponde a qualquer caractere ``não-alfanumérico``, o que é equivalente à classe ``[^a-zA-Z0-9_]``.

Estas sequências podem ser incluídas dentro de uma classe caractere. Por exemplo,
``[\s,.]`` É uma classe caractere que irá corresponder a qualquer caractere ``espaço-em-branco``, ou ``,`` ou ``.``.

O metacaractere final desta seção é o ``.``. Ele encontra tudo, exceto um caractere ``nova linha``, e existe um modo
alternativo (re.DOTALL), onde ele irá corresponderaté mesmo a um caractere ``nova linha``. ``.`` é , geralmente, usado
quando você quer corresponder com ``qualquer caractere``.



Repetindo Coisas
----------------

Ser capaz de corresponder com variados conjuntos de caracteres é a primeira coisa que as
expressões regulares podem fazer que ainda não é possível com os métodos disponíveis
para strings. No entanto, se essa fosse a única capacidade adicional das expressões
regulares, elas não seriam um avanço relevante. Outro recurso que você pode especificar é que
partes do RE devem ser repetidas um certo número de vezes.

O primeiro metacaractere para repetição de coisas que vamos ver é o ``*``. ``*`` Não corresponde
com o caractere literal ``*``; em vez disso, ele especifica que o caractere anterior pode ser
combinado zero ou mais vezes, em vez de apenas uma vez.

Por exemplo, ca*t irá corresponder a ``ct`` ``0 caracteres a``, ``cat`` ``1 caracter a``, ``caaat`` ``3 caracteres a``,
e assim por diante. O motor da RE tem várias limitações internas decorrentes
do tamanho do tipo int de C que irá impedi-lo de correspondência com mais de 2
bilhões de caracteres ``a``; você provavelmente não tem memória suficiente para a
construção de uma string tão grande, então você não deve chegar a esse limite.

Repetições, tais como ``*`` são gananciosas; ao repetir a RE, o motor de correspondência
vai tentar repeti-la tantas vezes quanto possível. Se porções posteriores do padrão
não corresponderem, o motor de correspondência, em seguida, volta e tenta
novamente com algumas repetições.

Um exemplo passo a passo irá tornar isso mais óbvio. Vamos considerar a expressão
``a[bcd]*b``. Isto corresponde com a letra ``a``, zero ou mais cartas da classe [bcd] e,
finalmente, termina com um ``b``. Agora imagine combinar esta RE contra a string
``abcbd``.

+------+-----------------+------------------------------------------------------------------------------------------------------+
|Passo | Correspondência | Explanação                                                                                           |
+------+-----------------+------------------------------------------------------------------------------------------------------+
|1     | a               |O caractere ``a`` na RE tem correspondência.                                                          |
+------+-----------------+------------------------------------------------------------------------------------------------------+
|2     | abcbd           |O motor corresponde com [bcd]*, indo tão longe quanto possível, que é o fim do string.                |
+------+-----------------+------------------------------------------------------------------------------------------------------+
|3     | Falha           |O motor tenta corresponder com ``b``, mas a posição corrente está no final da string, então ele falha.|
+------+-----------------+------------------------------------------------------------------------------------------------------+
|4     | abcb            |Voltando, de modo que [bcd]* corresponde a um caracter a menos.                                       |
+------+-----------------+------------------------------------------------------------------------------------------------------+
|5     | Falha           |Tenta ``b`` novamente, mas a posição corrente é a do último caractere, que é um ``d``.                |
+------+-----------------+------------------------------------------------------------------------------------------------------+
|6     | abc             |Voltando novamente, de modo que [bcd]* está correspondendo com ``bc`` somente.                        |
+------+-----------------+------------------------------------------------------------------------------------------------------+
|7     | abcb            |Tenta ``b`` novamente. Desta vez, o caractere na posição corrente é ``b``, por isso sucesso.          |
+------+-----------------+------------------------------------------------------------------------------------------------------+


O final da RE foi agora alcançado, e correspondeu com abcb. Isso demonstra como o motor
de correspondência vai tão longe quanto possível em uma primeira tentativa, e se não for encontrada
nenhuma correspondência, ele irá então progressivamente voltar e tentar novamente o resto da
RE novamente e novamente. Ele vai retornar até que não tenha
conseguido nenhum resultado para [bcd]*, e se isso falhar subsequentemente, o motor vai
concluir que a string não corresponde com o RE de nenhuma maneira.

Outro metacaractere de repetição é o ``+``, que corresponde a uma ou mais vezes. Preste
muita atenção para a diferença entre ``*`` e ``+``; ``*`` corresponde com zero ou mais vezes, assim, o que quer
que esteja sendo repetido pode não estar presente, enquanto que ``+`` requer pelo
menos uma ocorrência. Para usar um exemplo similar, ``ca+t`` vai corresponder a ``cat``
``1 a``, ``caaat`` ``3 a``, mas não corresponde com ``ct``.

Existem mais dois qualificadores de repetição. ``?`` O caractere ponto de interrogação, ``?``, que
corresponde a uma vez ou zero vezes; você pode pensar nisso como a marcação de algo
sendo opcional. Por exemplo, ``home-?brew`` corresponde tanto com ``homebrew`` quanto com ``home-brew``.

O qualificador de repetição mais complicado é o ``{m,n}``, em que ``m`` e ``n`` são números
inteiros decimais. Esta qualificação significa que deve haver pelo menos ``m`` repetições,
e no máximo ``n``. Por exemplo, ``a/{1,3}b`` irá corresponder a ``a/b``, ``a//b`` e ``a///b``. Não
vai corresponder com ``ab``, que não tem barras ou a ``a////b``, que tem quatro.

Você pode omitir tanto ``m`` quanto ``n``; nesse caso, um valor razoável é assumido para o valor em
falta. Omitir ``m`` é interpretado como o limite inferior de ``0(zero)``, enquanto omitir ``n``
resulta em um limite superior como o infinito -- na verdade, o limite superior é o limite de 2
bilhões mencionado anteriormente, mas isso poderia muito bem ser infinito.

Os leitores de uma inclinação reducionista podem notar que os três outros
qualificadores podem todos serem expressos utilizando esta notação. ``{0,}`` é o mesmo que ``*``,
``{1,}`` é equivalente a ``+``, e ``{0,1}`` é o mesmo que ``?``. É melhor usar ``*``, ``+`` ou ``?`` quando
puder, simplesmente porque eles são mais curtos e fáceis de ler.

Usando expressões regulares
---------------------------

Agora que nós vimos algumas expressões regulares simples, como nós
realmente as usamos em Python? O módulo re fornece uma interface para o mecanismo
de expressão regular, permitindo compilar REs em objetos e, em seguida,
executar comparações com eles.


Compilando Expressões Regulares
-------------------------------

As expressões regulares são compiladas em objetos padrão, que têm métodos para
várias operações, tais como a procura por padrões de correspondência ou realizar substituições de
strings.::

>>>
>>> import re
>>> p = re.compile('ab*')
>>> print p
<_sre.SRE_Pattern object at 0x...>

``re.compile()`` também aceita flags opcionais como argumentos, utilizados para
habilitar vários recursos especiais e variações de sintaxe. Nós vamos ver todas as
configurações disponíveis mais tarde, mas por agora, um único exemplo vai servir::

>>>
>>> p = re.compile('ab*', re.IGNORECASE)

A RE é passada para ``re.compile()`` como uma string. REs são tratadas como
strings porque as expressões regulares não são parte do núcleo da linguagem Python,
e nenhuma sintaxe especial foi criada para expressá-las. (Existem aplicações que não
necessitam de REs nenhuma, por isso não há necessidade de inchar a especificação
da linguagem, incluindo-as.) Em vez disso, o módulo ``re`` é simplesmente um módulo
de extensão C incluído no Python, assim como os módulos de ``socket`` ou ``zlib``.

Colocando REs em strings mantém a linguagem Python mais simples, mas tem uma
desvantagem, que é o tema da próxima seção.

