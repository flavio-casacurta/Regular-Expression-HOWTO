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
strings.

>>>
>>> import re
>>> p = re.compile('ab*')
>>> print p
<_sre.SRE_Pattern object at 0x...>

``re.compile()`` também aceita flags opcionais como argumentos, utilizados para
habilitar vários recursos especiais e variações de sintaxe. Nós vamos ver todas as
configurações disponíveis mais tarde, mas por agora, um único exemplo vai servir:

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


A praga da barra invertida
--------------------------

Como afirmado anteriormente, expressões regulares usam o caractere de barra
invertida ``\`` para indicar formas especiais ou para permitir que caracteres especiais
sejam usados sem invocar o seu significado especial. Isso entra em conflito com o uso
pelo Python do mesmo caractere para o mesmo propósito nas strings literais.

Vamos dizer que você quer escrever uma RE que corresponde com a string ``\section``, que
pode ser encontrada em um arquivo LaTeX. Para descobrir o que escrever no código
do programa, comece com a string que se deseja corresponder. Em seguida, você
deve preceder qualquer barra invertida e outros metacaracteres com
uma barra invertida, tendo como resultado a string ``\\section``. A string resultante que deve ser
passada para ``re.compile()`` deve ser ``\\section``. No entanto, para expressar isso
como uma string literal Python, ambas as barras invertidas devem ser precedidas com uma barra invertida
novamente.

+----------------+----------------------------------------------------------------+
|**Caracteres**  | **Etapa**                                                      |
+----------------+----------------------------------------------------------------+
|\\section       | string de texto a ser correspondida                            |
+----------------+----------------------------------------------------------------+
|\\\\section     | preceder com barra invertida para re.compile()                 |
+----------------+----------------------------------------------------------------+
|\\\\\\\\section | barras invertidas precedidas novamente para uma string literal |
+----------------+----------------------------------------------------------------+

Em suma, para corresponder com uma barra invertida literal, tem de se escrever ``\\\\``
como a string da RE, porque a expressão regular deve ser ``\\``, e cada barra invertida
deve ser expressa como ``\\`` dentro de uma string literal Python normal. Em REs que
apresentam barras invertidas repetidas vezes, isso leva a um monte de barras
invertidas repetidas e faz as strings resultantes difíceis de entender.

A solução é usar a notação de string crua (raw) do Python para expressões regulares;
barras invertidas não são tratadas de nenhuma forma especial em uma string literal
se prefixada com ``r``, então ``r"\n"`` é uma string de dois caracteres contendo ``\`` e
``n``, enquanto ``"\n"`` é uma string de um único caractere contendo uma nova linha. As
expressões regulares, muitas vezes, são escritas no código Python usando esta
notação de string crua (raw).

+-------------------+-----------------+
|**String Regular** | **String Crua** |
+-------------------+-----------------+
|"ab*"              | r"ab*"          |
+-------------------+-----------------+
|"\\\\\\\\section"  | r"\\\\section"  |
+-------------------+-----------------+
|"\\\\w+\\\\s+\\1"  | r"\\w+\\s+\\1"  |
+-------------------+-----------------+

Executando Comparações
----------------------

Uma vez que você tem um objeto que representa uma expressão regular compilada, o
que você faz com ele? Objetos padrão têm vários métodos e atributos. Apenas os
mais significativos serão vistos aqui; consulte a documentação do módulo ``re`` para uma lista
completa.

+-------------------+----------------------------------------------------------------------------------+
|**Método/Atributo**|   **Propósito**                                                                  |
+-------------------+----------------------------------------------------------------------------------+
|match()            |Determina se a RE combina com o início da string.                                 |
+-------------------+----------------------------------------------------------------------------------+
|search()           |Varre toda a string, procurando qualquer local onde esta RE tem correspondência.  |
+-------------------+----------------------------------------------------------------------------------+
|findall()          |Encontra todas as substrings onde a RE corresponde, e as retorna como uma lista.  |
+-------------------+----------------------------------------------------------------------------------+
|finditer()         |Encontra todas as substrings onde a RE corresponde, e as retorna como um iterator.|
+-------------------+----------------------------------------------------------------------------------+

``match()`` e ``search()`` retornam ``None`` se não existir nenhuma correspondência encontrada. Se tiveram sucesso,
uma instância de ``MatchObject`` é retornada, contendo informações sobre a correspondência: onde ela começa e termina, a
substring com a qual ela teve correspondência, e mais.

Você pode aprender sobre isto experimentando interativamente o módulo ``re``. Se você tiver o Tkinter disponível, você
pode também querer dar uma olhada em ``Tools/scripts/redemo.py``, um programa de demonstração incluído na
distribuição Python. Ele permite você entrar com REs e strings, e exibe se a RE tem correspondência ou falha. ``redemo.py``
pode ser bastante útil quando se tenta depurar uma RE complicada. Phil Schwartz’s ``Kodos`` é também uma ferramenta
interativa para desenvolvimento e teste de padrões RE.

Este HOWTO usa o interpretador Python padrão para seus exemplos. Primeiro, execute o interpretador Python, importe o
modulo ``re``, e compile uma RE

>>>
Python 2.7.10 (default, May 23 2015, 09:44:00) [MSC v.1500 64 bit
>>> import re
>>> p = re.compile('[a-z]+')
>>> p
<_sre.SRE_Pattern object at 0x...>
>>>

Agora, você pode tentar corresponder várias strings com a RE [a-z]+. Mas uma string
vazia não deveria corresponder com nada, desde que ``+`` significa ``uma ou mais repetições``. ``match()`` deve retornar
``None`` neste caso, o que fará com que o interpretador não imprima nenhuma saída. Você pode imprimir explicitamente o
resultado de ``match()`` para deixar isso claro.

>>>
>>> p.match("")
>>> print p.match("")
None


Agora, vamos experimentá-la em uma string que ela deve corresponder, como ``tempo``.
Neste caso, ``match()`` irá retornar um ``MatchObject``, assim que você deve armazenar
o resultado em uma variável para uso posterior.

>>>
>>> m = p.match('tempo')
>>> print m
<_sre.SRE_Match object at 0x...>

Agora você pode consultar o MatchObject para obter informações sobre a string
correspondente. Instâncias do ``MatchObject`` também tem vários métodos e atributos;
os mais importantes são os seguintes:

+-------------------+----------------------------------------------------------------------------------+
|**Método/Atributo**|   **Propósito**                                                                  |
+-------------------+----------------------------------------------------------------------------------+
|group()            |Retorna a string que corresponde com a RE                                         |
+-------------------+----------------------------------------------------------------------------------+
|start()            |Retorna a posição inicial da string correspondente                                |
+-------------------+----------------------------------------------------------------------------------+
|end()              |Retorna a posição final da string correspondente                                  |
+-------------------+----------------------------------------------------------------------------------+
|span()             |Retorna uma tupla contendo as posições (inicial, final) da string combinada       |
+-------------------+----------------------------------------------------------------------------------+

Experimentando estes métodos teremos seus significado esclarecidos:

>>>
>>> m.group()
'tempo'
>>> m.start(), m.end()
(0, 5)
>>> m.span()
(0, 5)

``group()`` retorna a substring correspondeu com a RE. ``start()`` e ``end()`` retornam
os índices inicial e o final da substring correspondente. ``span()`` retorna tanto os índices
inicial e final em uma única tupla. Como o método ``match()`` somente verifica se a RE
corresponde com o início de uma string, ``start()`` será sempre ``zero``. No entanto, o
método ``search()`` dos objetos padrão, varre toda a string, de modo que a substring correspondente
pode não iniciar em zero nesse caso.

>>>
>>> print p.match('::: message')
None
>>> m = p.search('::: message') ; print m
<_sre.SRE_Match object at 0x...>
>>> m.group()
'message'
>>> m.span()
(4, 11)

Nos programas reais, o estilo mais comum é armazenar o MatchObject em uma
variável e, em seguida, verificar se ela é 'None'. Isso geralmente se parece com:

>>>
>>> p = re.compile( ... )
>>> m = p.match( 'string goes here' )
>>> if m:
>>>     print 'Match found: ', m.group()
>>> else:
>>>     print 'No match'

Dois métodos padrão retornam todas as correspondências de um padrão. ``findall()``
retorna uma lista de strings correspondentes:

>>>
>>> p = re.compile('\d+')
>>> p.findall('12 drummREs drumming, 11 pipREs piping, 10 lords aleaping')
['12', '11', '10']

``findall()`` tem de criar a lista inteira antes que poder devolvê-la como
resultado.

O método ``finditer()`` retorna uma sequência de instâncias ``MatchObject`` como um iterator. [1]

>>>
>>> iterator = p.finditer('12 drummREs drumming, 11 ... 10 ...')
>>> iterator
<callable-iterator object at 0x401833ac>
>>> for match in iterator:
...     print match.span()
...
(0, 2)
(22, 24)
(29, 31)

Funções de Nível de Módulo
--------------------------

Você não tem que criar um objeto padrão e chamar seus métodos; o módulo
``re`` também fornece funções de nível superior chamada ``match()``, ``search()``,
``findall()``, ``sub()``, e assim por diante. Estas funções recebem os mesmos
argumentos que o método padrão correspondente, com a string RE adicionada
como o primeiro argumento, e ainda retornam ``None`` ou uma instância
``MatchObject``.

>>>
>>> print re.match(r'From\s+', 'Fromage amk')
None
>>> re.match(r'From\s+', 'From amk Thu May 14 19:12:10 1998')
<_sre.SRE_Match object at 0x...>

Sob o capô, estas funções simplesmente criam um objeto padrão para você e chamam
o método apropriado para ele. Elas também armazenam o objeto compilado em um
cache, para que futuras chamadas usando a mesma RE sejam mais rápidas.

Você deve usar essas funções de nível de módulo, ou você deve obter o padrão e
chamar seus métodos você mesmo? Essa escolha depende da frequência com que a
RE será usada, e no seu estilo de codificação pessoal. Se a RE for usada em
apenas um ponto no código, então as funções do módulo são provavelmente
mais convenientes. Se o programa contém uma grande quantidade de expressões
regulares, ou reutiliza as mesmas em vários locais, então pode valer a pena
recolher todas as definições em um lugar, em uma seção de código que compila todas
as REs antes do uso. Para dar um exemplo da biblioteca padrão, aqui está um extrato
de ``xmllib.py``:

>>>
>>> ref = re.compile( ... )
>>> entityref = re.compile( ... )
>>> charref = re.compile( ... )
>>> starttagopen = re.compile( ... )

Eu geralmente prefiro trabalhar com o objeto compilado, mesmo para usos de
uma só vez, mas poucas pessoas serão tão puristas sobre isso como eu
sou.

Flags de Compilação
-------------------

Flags de compilação permitem modificar alguns aspectos de como as expressões
regulares funcionam. Flags estão disponíveis no módulo ``rev`` sob dois nomes, um
nome longo, tal como ``IGNORECASE`` e um curto, na forma de uma letra, como ``I``. (Se você
estiver familiarizado com o padrão dos modificadores do Perl, o nome curto
usa as mesmas letras; o forma abreviada de ``re.VERBOSE`` é ``re.X``, por exemplo)
Várias flags podem ser especificadas como um vetor intercalado por ``OU(|)``; ``re.I |re.M`` define as flags
``I`` e ``M``, por exemplo.

Aqui está uma tabela das flags disponíveis, seguida por uma explicação mais detalhada
de cada uma:

+--------------+---------------------------------------------------------------------------------------------+
|Flag          | Significado                                                                                 |
+==============+=============================================================================================+
|DOTALL, S     |Faz o '.' corresponder a qualquer caractere, incluindo novas linhas                          |
+--------------+---------------------------------------------------------------------------------------------+
|IGNORECASE, I |Faz correspondências com maiúsculas e minúsculas                                             |
+--------------+---------------------------------------------------------------------------------------------+
|LOCALE, L     |Faz uma correspondência de acordo com o idioma                                               |
+--------------+---------------------------------------------------------------------------------------------+
|MULTILINE, M  |Correspondência multi-linha, afetando ^ e $                                                  |
+--------------+---------------------------------------------------------------------------------------------+
|VERBOSE, X    |Habilita REs detalhadas, que podem ser organizadas de forma mais clara e compreensível.      |
+--------------+---------------------------------------------------------------------------------------------+
|UNICODE, U    |Faz de uma letra precedida pela barra invertida ('\\') tal como \\w, \\b \\s e \\d dependente|
+--------------+---------------------------------------------------------------------------------------------+
|              |                                                 da base de dados de caracteres Unicode.     |
+--------------+---------------------------------------------------------------------------------------------+

**I**

**IGNORECASE**
Executa a correspondência com maiúsculas e minúsculas; classe de caracteres e
strings literais irão corresponder com letras ignorando serem maiúsculas ou minúsculas. Por exemplo, ``[A-Z]`` irá
corresponder com letras minúsculas também, e ``Spam`` irá corresponder com ``Spam``, ``spam``, ou
``spAM``. Este ``“lowercasing”`` não leva o idioma corrente em conta; ele irá se você também definir a
flag ``LOCALE``.

**L**

**LOCALE**
Faz ``\w, \W, \b, e \B``, dependentes do idioma corrente. ``Locale`` é um recurso da biblioteca C com o objetivo de ajudar na
criação de programas que levam em conta as diferenças linguísticas. Por exemplo, se você está processando um texto em
francês, que você gostaria de ser capaz de escrever ``\w+`` para corresponder com palavras, mas ``\w`` corresponde apenas com a
classe de caracteres ``[A-Za-z]``; ele não vai corresponder com ``é`` ou ``ç``. Se o sistema estiver configurado corretamente e
o idioma francês estiver selecionado, determinadas funções C vão dizer ao programa que ``é`` também deve ser considerada
como uma letra. Definir a flag ``LOCALE`` no momento de compilar uma expressão regular fará com que o objeto compilado resultante
use essas funções de C para ``\w``; isso causa lentidão, mas também permite que ``\w+`` corresponda com as palavras em francês,
caso seja necessário.

**M**

**MULTILINE**
