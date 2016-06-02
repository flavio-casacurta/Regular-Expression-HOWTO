

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
e assim por diante.
O motor da RE tem várias limitações internas decorrentes
do tamanho do tipo int de C que irá impedi-lo de correspondência com mais de 2
bilhões de caracteres ``a``; você provavelmente não tem memória suficiente para a
construção de uma string tão grande, então você não deve chegar a esse limite.

Repetições, tais como ``*`` são gananciosas; ao repetir a RE, o motor de correspondência
vai tentar repeti-la tantas vezes quanto possível. Se porções posteriores do padrão
não corresponderem, o motor de correspondência, em seguida, volta e tenta
novamente com algumas repetições.
Um exemplo passo a passo irá tornar isso mais óbvio. Vamos considerar a expressão
a[bcd]*b. Isto corresponde com a letra ``a``, zero ou mais cartas da classe [bcd] e,
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
menos uma ocorrência. Para usar um exemplo similar, ca+t vai corresponder a cat
(1 ``a``), caaat (3 ``a``), mas não corresponde com ct.
Existem mais dois qualificadores de repetição. ? O caractere ponto de interrogação, ``?``, que
corresponde a uma vez ou zero vezes; você pode pensar nisso como a marcação de algo
sendo opcional. Por exemplo, home-?brew corresponde tanto com homebrew quanto com
home-brew.
O qualificador de repetição mais complicado é o {m,n}, em que ``m`` e ``n`` são números
inteiros decimais. Esta qualificação significa que deve haver pelo menos ``m`` repetições,
e no máximo ``n``. Por exemplo, ``a/{1,3}b`` irá corresponder a ``a/b``, ``a//b`` e ``a///b``. Não
vai corresponder com ``ab``, que não tem barras ou a ``a////b``, que tem quatro.
Você pode omitir tanto ``m`` quanto ``n``; nesse caso, um valor razoável é assumido para o valor em
falta. Omitir ``m`` é interpretado como o limite inferior de 0(zero), enquanto omitir ``n``
resulta em um limite superior como o infinito -- na verdade, o limite superior é o limite de 2
bilhões mencionado anteriormente, mas isso poderia muito bem ser infinito.
Os leitores de uma inclinação reducionista podem notar que os três outros
qualificadores podem todos serem expressos utilizando esta notação. {0,} é o mesmo que ``*``,
{1,} é equivalente a ``+``, e {0,1} é o mesmo que ``?``. É melhor usar ``*``, ``+`` ou ``?`` quando
puder, simplesmente porque eles são mais curtos e fáceis de ler.

Usando expressões regulares

¶

Agora que nós vimos algumas expressões regulares simples, como nós
realmente as usamos em Python? O módulo re fornece uma interface para o mecanismo
de expressão regular, permitindo compilar REs em objetos e, em seguida,
executar comparações com eles.


Compilando Expressões Regulares

¶

As expressões regulares são compiladas em objetos padrão, que têm métodos para
várias operações, tais como a procura por padrões de correspondência ou realizar substituições de
strings.
>>>
>>> import re
>>> p = re.compile(``ab*``)
>>> print p
<_sre.SRE_Pattern object at 0x...>
re.compile()também aceita flags opcionais como argumentos, utilizados para
habilitar vários recursos especiais e variações de sintaxe. Nós vamos ver todas as
configurações disponíveis mais tarde, mas por agora, um único exemplo vai servir:
>>>
>>> p = re.compile(``ab*``, re.IGNORECASE)
A RE é passada para re.compile()como uma string. REs são tratadas como
strings porque as expressões regulares não são parte do núcleo da linguagem Python,
e nenhuma sintaxe especial foi criada para expressá-las. (Existem aplicações que não
necessitam de REs nenhuma, por isso não há necessidade de inchar a especificação
da linguagem, incluindo-as.) Em vez disso, o módulo re é simplesmente um módulo
de extensão C incluído no Python, assim como os módulos de socket ou zlib.
Colocando REs em strings mantém a linguagem Python mais simples, mas tem uma
desvantagem, que é o tema da próxima seção.

