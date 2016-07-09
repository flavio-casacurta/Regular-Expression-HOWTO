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

**I - IGNORECASE**

Executa a correspondência com maiúsculas e minúsculas; classe de caracteres e
strings literais irão corresponder com letras ignorando serem maiúsculas ou minúsculas. Por exemplo, ``[A-Z]`` irá
corresponder com letras minúsculas também, e ``Spam`` irá corresponder com ``Spam``, ``spam``, ou
``spAM``. Este ``“lowercasing”`` não leva o idioma corrente em conta; ele irá se você também definir a
flag ``LOCALE``.

**L - LOCALE**
Faz ``\w, \W, \b, e \B``, dependentes do idioma corrente. ``Locale`` é um recurso da biblioteca C com o objetivo de ajudar na
criação de programas que levam em conta as diferenças linguísticas. Por exemplo, se você está processando um texto em
francês, que você gostaria de ser capaz de escrever ``\w+`` para corresponder com palavras, mas ``\w`` corresponde apenas com a
classe de caracteres ``[A-Za-z]``; ele não vai corresponder com ``é`` ou ``ç``. Se o sistema estiver configurado corretamente e
o idioma francês estiver selecionado, determinadas funções C vão dizer ao programa que ``é`` também deve ser considerada
como uma letra. Definir a flag ``LOCALE`` no momento de compilar uma expressão regular fará com que o objeto compilado resultante
use essas funções de C para ``\w``; isso causa lentidão, mas também permite que ``\w+`` corresponda com as palavras em francês,
caso seja necessário.

**M - MULTILINE**



(``^`` e ``$`` ainda não foram explicados, eles serão comentados na seção ``Mais
Metacaracteres``.)

Normalmente ``^`` corresponde apenas ao início da string e ``$`` corresponde apenas ao
final da string, e imediatamente antes da nova linha (se existir) no final da string.
Quando esta flag é especificada, o ``^`` corresponde ao início da string e ao início de
cada linha dentro da string, imediatamente após cada nova linha. Da mesma
forma, o metacaractere ``$`` corresponde tanto ao final da string e ao final de cada linha
(imediatamente antes de cada nova linha).


**S - DOTALL**

Faz o caractere especial ``.`` corresponder com qualquer caractere que seja, incluindo o
nova linha; sem esta flag, ``.`` irá corresponder a qualquer coisa, exceto o nova linha.


**U - UNICODE**

Faz ``\w``, ``\W``, ``\b``, ``\B``, ``\d``, ``\D``, ``\s`` e ``\S`` dependentes das propriedades dos caracteres
Unicode do banco de dados.

**X - VERBOSE**

Esta flag permite escrever expressões regulares mais legíveis,
permitindo mais flexibilidade na maneira de formatá-la. Quando esta flag
é especificada, o espaço em branco dentro da string RE é ignorado, exceto quando o
espaço em branco está em uma classe de caracteres ou precedido por uma barra
invertida não "escapada"; isto permite organizar e formatar a RE de maneira mais clara. Esta
flag também permite que se coloque comentários dentro de uma RE que serão ignorados pelo
mecanismo; os comentários são marcados por um "#" que não está nem em uma classe de
caracteres nem precedido por uma barra invertida não "escapada".
Por exemplo, aqui está uma RE que usa re.VERBOSE; veja, o quanto mais fácil
de ler é ?

>>> charref = re.compile(r"""
    &[#]             # Início a uma referência a uma entidade numérica
    (
       0[0-7]+       # Formato Octal
     | [0-9]+        # Formato Decimal
     | x[0-9a-fA-F]+ # Formato Hexadecimal
    )
     ;                # ponto vírgula final
     """, re.VERBOSE)

Sem o "verbose" definido, A RE iria se parecer como isto:

>>> charref = re.compile("&#(0[0-7]+"
                         "|[0-9]+"
                         "|x[0-9a-fA-F]+);")

No exemplo acima, a concatenação automática de strings literais em Python foi
usada para quebrar a RE em partes menores, mas ainda é mais difícil de entender
do que a versão que usa ``re.VERBOSE``.

Mais Poder dos Padrões
----------------------

Até agora, cobrimos apenas uma parte dos recursos das expressões regulares.
Nesta seção, vamos abordar alguns metacaracteres novos, e como usar grupos para
recuperar partes do texto que teve correspondência.

Mais Metacaracteres
-------------------

Existem alguns metacaracteres que nós ainda não vimos. A maioria deles serão referenciados
nesta seção.

Alguns dos metacaracteres restantes a serem discutidos são como uma afirmação de ``largura zero`` (zero-width assertions). Eles
não fazem com que o mecanismo avance pela string; ao contrário, eles não consomem
nenhum caractere, e simplesmente tem sucesso ou falha. Por exemplo, ``\b`` é
uma afirmação de que a posição atual está localizada nas bordas de uma palavra; a
posição não é alterada de nenhuma maneira por ``\b``. Isto significa que afirmações de ``largura zero``
nunca devem ser repetidas, porque se elas combinam uma vez em um
determinado local, elas podem, obviamente, combinar um número infinito de
vezes.


**|**

Alternância, ou operador ``or``. Se ``A`` e ``B`` são expressões regulares, ``A|B`` irá
corresponder com qualquer string que corresponder com ``A`` ou ``B``. ``|`` tem uma prioridade muito baixa,
a fim de fazê-lo funcionar razoavelmente quando você está alternando entre strings de
vários caracteres. ``Crow|Servo`` irá corresponder tanto com ``Crow`` quanto com ``Servo``, e não com ``Cro``,
``w`` ou ``S``, e ``ervo``.

Para corresponder com um ``|`` literal, use ``\|``, ou coloque ele dentro de uma classe de
caracteres, como em ``[|]``.


**^**

Corresponde ao início de linha. A menos que a flag MULTILINE tenha sido definida,
isso só irá corresponder ao início da string. No modo MULTILINE, isso também
corresponde imediatamente após cada nova linha de dentro da string.
Por exemplo, para ter correspondência com a palavra ``From`` apenas no início de uma linha, a
RE a ser usada é ``^From``.

>>> print re.search('^From', 'From Here to Eternity')
<_sre.SRE_Match object at 0x...>
>>> print re.search('^From', 'Reciting From Memory')
None


**$**

Corresponde ao fim de uma linha, que tanto é definido como o fim de uma string, ou qualquer local seguido por um
caractere de nova linha.

>>> print re.search('}$', '{block}')
<_sre.SRE_Match object at 0x...>
>>> print re.search('}$', '{block} ')
None
>>> print re.search('}$', '{block}\n')
<_sre.SRE_Match object at 0x...>


Para corresponder com um ``$`` literal, use ``\$`` ou coloque-o dentro de uma classe de
caracteres, como em ``[$]``.


**\\A**

Corresponde apenas com o início da string. Quando não estiver em modo MULTILINE, ``\A``
e ``^`` são efetivamente a mesma coisa. No modo MULTILINE, eles são diferentes: ``\A`` continua a
corresponder apenas com o início da string, mas ``^`` pode corresponder com qualquer localização de dentro da string, que
seja posterior a um caractere nova linha.


**\\Z**

Corresponde apenas ao final da string.


**\\b**

Borda de palavra. Esta é uma afirmação de ``largura zero`` que corresponde apenas ao
início ou ao final de uma palavra. Uma palavra é definida como uma sequência de
caracteres alfanuméricos, de modo que o fim de uma palavra é indicado por espaços
em branco ou um caractere não alfanumérico.

O exemplo a seguir corresponde a ``class`` apenas quando é a palavra exata; ele
não irá corresponder quando for contido dentro de uma outra palavra.

>>>
>>> p = re.compile(r'\bclass\b')
>>> print p.search('no class at all')
<_sre.SRE_Match object at 0x...>
>>> print p.search('the declassified algorithm')
None
>>> print p.search('one subclass is')
None

Há duas sutilezas você deve lembrar ao usar essa sequência especial. Em primeiro
lugar, esta é a pior colisão entre strings literais do Python e sequências de expressão
regular. Nas strings literais do Python, ``\b`` é o caractere backspace, o valor ASCII 8. Se
você não estiver usando strings cruas (raw), então Python irá converter o ``\b`` em um
backspace e sua RE não irá funcionar da maneira que você espera. O exemplo a
seguir parece igual a nossa RE anterior, mas omite o ``r`` na frente da string RE.

>>>
>>> p = re.compile('\bclass\b')
>>> print p.search('no class at all')
None
>>> print p.search('\b' + 'class' + '\b')
<_sre.SRE_Match object at 0x...>

Além disso, dentro de uma classe de caracteres, onde não há nenhum uso para esta
afirmação, ``\b`` representa o caractere backspace, para compatibilidade com strings
literais do Python.

**\\B**

Outra afirmação de ``largura zero``; isto é o oposto de ``\b``, correspondendo apenas quando
a posição corrente não é de uma borda de palavra.

Agrupamento
-----------

Frequentemente é necessário obter mais informações do que apenas se a RE
teve correspondência ou não. As expressões regulares são muitas vezes utilizadas para
dissecar strings escrevendo uma RE dividida em vários subgrupos que correspondem a
diferentes componentes de interesse. Por exemplo, uma linha de cabeçalho RFC-822
é dividida em um nome de cabeçalho e um valor, separados por um ``:``, como essa:

::

    From: author@example.com
    User-Agent: Thunderbird 1.5.0.9 (X11/20061227)
    MIME-Version: 1.0
    To: editor@example.com

Isto pode ser gerenciado ao escrever uma expressão regular que corresponde com uma
linha inteira de cabeçalho, e tem um grupo que corresponde ao nome do cabeçalho, e
um outro grupo, que corresponde ao valor do cabeçalho.
Os grupos são marcados pelos metacaracteres ``(`` e ``)``. ``(`` e ``)`` têm muito do
mesmo significado que eles têm em expressões matemáticas; eles agrupam as
expressões contidas dentro deles, e você pode repetir o conteúdo de um grupo com
um qualificador de repetição, como ``*``, ``+``, ``?``, ou ``{m,n}``. Por exemplo, ``(ab)*`` irá
corresponder a zero ou mais repetições de ``ab``.

>>>
>>> p = re.compile('(ab)*')
>>> print p.match('ababababab').span()
(0, 10)

Grupos indicados com ``(`` e ``)`` também capturam o índice inicial e final do texto que
eles correspondem; isso pode ser obtido por meio da passagem de um argumento para
``group()``, ``start()``, ``end()``, e ``span()``. Os grupos são numerados começando com
0. O grupo 0 está sempre presente; é toda a RE, logo, todos os métodos MatchObject têm
o grupo 0 como seu argumento padrão. Mais tarde veremos como expressar
grupos que não capturam a extensão de texto com a qual eles correspondem.

>>>
>>> p = re.compile('(a)b')
>>> m = p.match('ab')
>>> m.group()
'ab'
>>> m.group(0)
'ab'

Subgrupos são numerados a partir da esquerda para a direita, de forma crescente a partir de 1.
Os grupos podem ser aninhados; para determinar o número, basta contar os
caracteres de abertura de parêntese - ``(``, indo da esquerda para a direita.

>>>
>>> p = re.compile('(a(b)c)d')
>>> m = p.match('abcd')
>>> m.group(0)
'abcd'
>>> m.group(1)
'abc'
>>> m.group(2)
'b'

``group()`` pode receber vários números de grupos de uma vez, e nesse caso
ele irá retornar uma tupla contendo os valores correspondentes desses grupos.

>>>
>>> m.group(2,1,2)
('b', 'abc', 'b')

O método ``groups()`` retorna uma tupla contendo as strings de todos os subgrupos, de
1 até o último. Independente da quantidade de subgrupos informada.

>>>
>>> m.groups()
('abc', 'b')

Referências anteriores em um padrão permitem que você especifique que o conteúdo
de um grupo capturado anteriormente também deve ser encontrado na posição
atual na sequência. Por exemplo, ``\1`` terá sucesso se o conteúdo exato do grupo 1
puder ser encontrado na posição atual, e falhar caso contrário. Lembre-se que as strings
literais do Python também usam a barra invertida seguida por números para
permitir a inclusão de caracteres arbitrários em uma string, por isso certifique-se de usar
strings cruas (raw) ao incorporar referências anteriores em uma RE.

Por exemplo, a seguinte RE detecta palavras duplicadas em uma string.

>>>
>>> p = re.compile(r'(\b\w+)\s+\1')
>>> p.search('Paris in the the spring').group()
'the the'

Referências anteriores como esta não são, geralmente, muito úteis apenas para fazer pesquisa percorrendo
uma string — existem alguns formatos de texto que repetem dados dessa forma —
mas em breve você irá descobrir que elas são muito úteis para realizar substituições de
strings.


Não captura e Grupos Nomeados
-----------------------------

REs elaboradas podem usar muitos grupos, tanto para capturar substrings de
interesse, quanto para agrupar e estruturar a própria RE. Em REs complexas, torna-se difícil
manter o controle dos números dos grupos. Existem dois recursos que ajudam
a lidar com esse problema. Ambos usam uma sintaxe comum para extensões de expressão
regular, então vamos olhar para isso em primeiro lugar.

Perl 5 acrescentou vários recursos adicionais para expressões regulares padrão, e o
módulo ``re`` do Python suporta a maioria deles. Teria sido difícil escolher novos
metacaracteres de uma única tecla ("single-keystroke") ou novas sequências especiais começando com ``\``
para representar os novos recursos sem fazer as expressões regulares do Perl
confusamente diferente das REs padrão. Se você escolher ``&`` como um novo
metacaractere, por exemplo, velhas expressões estariam assumindo que o ``&`` era um
caractere comum e não teriam que ``escapá-lo`` escrevendo ``\&`` ou ``[&]``.

A solução escolhida pelos desenvolvedores do Perl foi usar ``(?...)`` como uma sintaxe de
extensão. Um ``?`` imediatamente após um parêntese era um erro de sintaxe porque o ``?``
não teria nada a repetir, de modo que isso não introduz quaisquer problemas de
compatibilidade. Os caracteres imediatamente após um ``?`` indicam que a extensão está
sendo usada, então ``(?=foo)`` é uma coisa (uma afirmação ``lookahead`` positiva) e
``(?:foo)`` é outra coisa (um grupo de não captura contendo a subexpressão ``foo``).

Python adiciona uma sintaxe de extensão a sintaxe de extensão do Perl. Se o
primeiro caractere após o ponto de interrogação é um ``P``, você sabe que é uma
extensão que é específica para Python. Atualmente, existem duas dessas extensões :
``(?P<name>...)`` define um grupo nomeado e ``(?P=name)`` é uma referência anterior a
um grupo chamado. Se futuras versões de Perl 5 adicionarem funcionalidades
semelhantes, utilizando uma sintaxe diferente, o módulo ``rev vai ser alterado para
suportar a nova sintaxe, enquanto que preserva a sintaxe específica do Python, para ter boa
compatibilidade.

Agora que nós vimos a sintaxe geral de extensão, podemos voltar para as
características que simplificam o trabalho com grupos em REs complexas. Como os
grupos são numerados da esquerda para a direita e uma expressão complexa
pode usar muitos grupos, pode tornar-se difícil manter o controle da numeração
correta. Modificar uma RE tão complexa também é irritante: se inserir um novo grupo
perto do início você tem que alterar os números de tudo o que se segue.

Às vezes você vai querer usar um grupo para coletar uma parte de uma expressão
regular, mas não está interessado em recuperar o conteúdo do grupo. Você pode
fazer este fato explícito usando um grupo de não-captura: ``(?:...)``, onde você pode
substituir o ``...`` por qualquer outra expressão regular.

>>>
>>> m = re.match("([abc])+", "abc")
>>> m.groups()
('c',)
>>> m = re.match("(?:[abc])+", "abc")
>>> m.groups()
()

Exceto pelo fato de que não é possível recuperar o conteúdo sobre o qual o grupo
corresponde, um grupo de não captura se comporta exatamente da mesma forma que um
grupo de captura; você pode colocar qualquer coisa dentro dele, repeti-lo com um
metacaractere de repetição, como o '*', e aninhá-lo dentro de outros grupos (de captura ou
não captura). ``(?:...)`` é particularmente útil para modificar um padrão existente,
já que você pode adicionar novos grupos sem alterar a forma como todos os
outros grupos estão numerados. Deve ser mencionado que não há diferença de desempenho na
busca entre grupos de captura e grupos de não captura; uma forma não é mais rápida
que outra.

Uma característica mais significativa são os grupos nomeados: em vez de se referir a
eles por números, os grupos podem ser referenciados por um nome.
A sintaxe de um grupo nomeado é uma das extensões específicas do Python:
``(?P<name>...)``. ``name`` é, obviamente, o nome do grupo. Os grupos nomeados
também se comportam exatamente como os grupos de captura, e, adicionalmente,
associam um nome a um grupo. Todos os métodos ``MatchObject`` que lidam com grupos
de captura aceitam tanto inteiros que se referem ao grupo por número ou strings
que contêm o nome do grupo desejado. Os grupos nomeados ainda recebem
números, então você pode recuperar informações sobre um grupo de duas maneiras:

>>>
>>> p = re.compile(r'(?P<word>\b\w+\b)')
>>> m = p.search( '(((( Lots of punctuation )))' )
>>> m.group('word')
'Lots'
>>> m.group(1)
'Lots'

Os grupos nomeados são úteis porque eles permitem que você use nomes de fácil
lembrança, em vez de ter que lembrar de números. Aqui está um exemplo de RE usando o
módulo ``imaplib``::

    InternalDate = re.compile(r'INTERNALDATE "'
                              r'(?P<day>[ 123][0-9])-(?P<mon>[A-Z][a-z][a-z])-'
                              r'(?P<year>[0-9][0-9][0-9][0-9])'
                              r' (?P<hour>[0-9][0-9]):(?P<min>[0-9][0-9]):(?P<sec>[0-9][09])'
                              r' (?P<zonen>[-+])(?P<zoneh>[0-9][0-9])(?P<zonem>[0-9][0-9])'
                              r'"')

É obviamente muito mais fácil fazer referência a ``m.group('zonem')``, do que ter que se
lembrar de capturar o grupo 9.

A sintaxe para referências anteriores em uma expressão, tal como ``(...)\1``, faz referência ao número do grupo. Existe,
naturalmente, uma variante que usa o nome do grupo em
vez do número. Isto é outra extensão Python: ``(?P=name)`` indica que o conteúdo do
grupo chamado ``name`` deve, novamente, ser correspondido no ponto atual. A expressão
regular para encontrar palavras duplicadas, ``(\b\w+)\s+\1``, também pode ser escrita
como ``(?P<word>\b\w+)\s+(?P=word)``:

>>>
>>> p = re.compile(r'(?P<word>\b\w+)\s+(?P=word)')
>>> p.search('Paris in the the spring').group()
'the the'

Afirmação Lookahead
-------------------

Outra afirmação de "largura zero" é a afirmação lookahead. Afirmações LookAhead
estão disponíveis tanto na forma positiva quanto na negativa, e se parece com isto:

**(?=...)**

Afirmação lookahead positiva. Retorna sucesso se a expressão regular informada, aqui
representada por ``...``, corresponde com o conteúdo da localização atual, e retorna falha caso contrário.
Mas, uma vez que a expressão informada tenha sido testada, o mecanismo de correspondência não
faz qualquer avanço; o resto do padrão é tentado no mesmo local de onde a afirmação foi iniciada.

**(?!...)**

Afirmação lookahead negativa. É o oposto da afirmação positiva; será bem-sucedida se
a expressão informada não corresponder com o conteúdo da posição atual na string.

Para tornar isto concreto, vamos olhar para um caso em que um lookahead é útil.
Considere um padrão simples para corresponder com um nome de arquivo e divida-o em pedaços,
um nome base e uma extensão, separados por um ``.``. Por exemplo, em
``news.rc,news`` é o nome base, e ``rc`` é a extensão do nome de arquivo.

O padrão para corresponder com isso é muito simples::

 .*[.].*$

Observe que o ``.`` precisa ser tratado de forma especial, porque é um metacaractere;
Eu coloquei ele dentro de uma classe de caracteres. Note também o ``$`` no final; ele é
adicionado para garantir que todo o resto da string deve ser incluído na extensão.
Esta expressão regular corresponde com: ``foo.bar``, ``autoexec.bat``, ``sendmail.cf`` e
``printers.conf``.


Agora, considere complicar um pouco o problema; e se você desejar
corresponder com nomes de arquivos onde a extensão não é ``bat``? Algumas tentativas
incorretas::

    .*[.][^b].*$

A primeira tentativa acima tenta excluir bat, exigindo que o primeiro caractere da
extensão não é um b. Isso é errado, porque o padrão também não corresponde
``foo.bar``::

    .*[.]([^b]..|.[^a].|..[^t])$

A expressão fica mais confusa se você tentar remendar a primeira solução,
exigindo que uma das seguintes situações corresponda: o primeiro caractere da extensão não é ``b``; o
segundo caractere não é ``a``; ou o terceiro caractere não é ``t``. Isso aceita
``foo.bar`` e rejeita ``autoexec.bat``, mas requer uma extensão de três letras e não
aceitará um nome de arquivo com uma extensão de duas letras, tal como ``sendmail.cf``.
Nós iremos complicar o padrão novamente em um esforço para corrigi-lo::

    .*[.]([^b].?.?|.[^a]?.?|..?[^t]?)$

Na terceira tentativa, a segunda e terceira letras são todas consideradas opcionais, a fim de
permitir correspondência com as extensões mais curtas do que três caracteres, tais como
``sendmail.cf``.

O padrão está ficando realmente muito complicado agora, o que faz com que seja difícil de ler e
compreender. Pior ainda, se o problema mudar e você quiser excluir tanto ``bat`` quanto ``exe``
como extensões, o padrão iria ficar ainda mais complicado e confuso.

Um lookahead negativo elimina toda esta confusão::

    .*[.](?!bat$).*$

O lookahead negativo significa: se a expressão ``bat`` não corresponder até este momento,
tente o resto do padrão; se ``bat$`` tem correspondência, todo o padrão irá falhar. O
final ``$`` é necessário para garantir que algo como ``sample.batch``, onde a extensão
só começa com o ``bat``, será permitido.

Excluir uma outra extensão de nome de arquivo agora é fácil; basta fazer a adição de uma
alternativa dentro da afirmação. O padrão a seguir exclui os nomes de arquivos que
terminam com ``bat`` ou ``exe``::

    .*[.](?!bat$|exe$).*$


Modificando Strings
-------------------

Até este ponto, nós simplesmente realizamos pesquisas em uma string estática. As
expressões regulares também são comumente usadas para modificar strings através de várias
maneiras, usando os seguintes métodos padrão:

+----------------+-----------------------------------------------------------------------------------------------------+
|Método/Atributo | Propósito                                                                                           |
+================+=====================================================================================================+
|split()         | Divide a string em uma lista, dividindo-a onde quer que haja correspondência com a RE               |
+----------------+-----------------------------------------------------------------------------------------------------+
|sub()           | Encontra todas as substrings que correspondem com a RE e faz a substituição por uma string diferente|
+----------------+-----------------------------------------------------------------------------------------------------+
|subn()          | Faz a mesma coisa que sub(), mas retorna a nova string e o número de substituições                  |
+----------------+-----------------------------------------------------------------------------------------------------+

Dividindo as Strings
--------------------

O método ``split()`` de um padrão divide uma string em pedaços onde quer que a RE
corresponda, retornando uma lista formada por esses pedaços. É semelhante ao método ``split()`` de
strings, mas oferece muito mais generalidade nos delimitadores, e assim, você pode fazer disso para fazer a
divisão; ``split()`` só suporta a divisão de espaço em branco ou por uma string
fixa. Como você deve deduzir, existe também uma função a nível de módulo ``re.split()``.

**.split(string[, maxsplit=0])**

Divide a string usando a correspondência com uma expressão regular. Se os parênteses de
captura forem utilizados na RE, então seu conteúdo também será retornado como
parte da lista resultante. Se maxsplit é diferente de zero, um número de divisões
``maxsplit`` será executado.

Você pode limitar o número de divisões feitas, passando um valor para maxsplit.
Quando ``maxsplit`` é diferente de zero, um determinado número de divisões ``maxsplit`` será executado, e o
restante da string é retornado como o elemento final da lista. No exemplo a seguir, o
delimitador é qualquer sequência de caracteres não alfanuméricos.

>>>
>>> p = re.compile(r'\W+')
>>> p.split('This is a test, short and sweet, of split().')
['This', 'is', 'a', 'test', 'short', 'and', 'sweet', 'of', 'split',
'']
>>> p.split('This is a test, short and sweet, of split().', 3)
['This', 'is', 'a', 'test, short and sweet, of split().']


Às vezes, você não está apenas interessado no que o texto que está entre
delimitadores contém, mas também precisa saber qual o delimitador foi usado. Se os parênteses
de captura são utilizados na RE, então os respectivos valores são também
retornados como parte da lista. Compare as seguintes chamadas:

>>>
>>> p = re.compile(r'\W+')
>>> p2 = re.compile(r'(\W+)')
>>> p.split('This... is a test.')
['This', 'is', 'a', 'test', '']
>>> p2.split('This... is a test.')
['This', '... ', 'is', ' ', 'a', ' ', 'test', '.', '']

A função de nível de módulo re.split() adiciona a RE a ser utilizada como o
primeiro argumento, mas é, em determinadas circunstâncias, a mesma.

>>>
>>> re.split('[\W]+', 'Words, words, words.')
['Words', 'words', 'words', '']
>>> re.split('([\W]+)', 'Words, words, words.')
['Words', ', ', 'words', ', ', 'words', '.', '']
>>> re.split('[\W]+', 'Words, words, words.', 1)
['Words', 'words, words.']


Busca e Substituição
--------------------

Outra tarefa comum é encontrar todas as combinações para um padrão e substituí-las
por uma string diferente. O método ``sub()`` recebe um valor de substituição, que pode
ser uma string ou uma função, e a string a ser processada.

**.sub(replacement, string[, count=0])**

Retorna a string obtida substituindo as ocorrências mais à esquerda não sobrepostas
da RE em ``string`` pela substituição ``replacement``. Se o padrão não for encontrado, a
``string`` é retornada inalterada.

O argumento opcional ``count`` é o número máximo de ocorrências do padrão a ser
substituído; ``count`` deve ser um número inteiro não negativo. O valor padrão ``0``
significa para substituir todas as ocorrências.

Aqui está um exemplo simples do uso do método ``sub()``. Ele substitui nomes de
cores pela palavra ``colour``:

>>>
>>> p = re.compile( '(blue|white|red)')
>>> p.sub( 'colour', 'blue socks and red shoes')
'colour socks and colour shoes'
>>> p.sub( 'colour', 'blue socks and red shoes', count=1)
'colour socks and red shoes'

O método ``subn()`` faz o mesmo trabalho, mas retorna uma tupla com duas informações; contém uma string
com novo valor e o número de substituições que foram realizadas:

>>>
>>> p = re.compile( '(blue|white|red)')
>>> p.subn( 'colour', 'blue socks and red shoes')
('colour socks and colour shoes', 2)
>>> p.subn( 'colour', 'no colours at all')
('no colours at all', 0)

Correspondências vazias somente são substituídas quando não estão adjacente (próxima) a
uma correspondência anterior.

>>>
>>> p = re.compile('x*')
>>> p.sub('-', 'abxd')
'-a-b-d-'

Se a substituição (replacement) é uma string, qualquer barra invertida é interpretada e
processada. Isto é, ``\n`` é convertido a um único caractere de nova linha, ``\r`` é
convertido em um retorno do carro, e assim por diante. Casos desconhecidos, como
``\j`` são ignorados. Referências anteriores (retrovisor - Aurelio), como ``\6``, são substituídas com a
substring correspondida pelo grupo correspondente na RE. Isso permite que você
incorpore partes do texto original na string de substituição resultante.

Este exemplo corresponde com a palavra ``section``, seguida por uma string colocada entre
``{, }`` e altera ``section`` para ``subsection``:

>>>
>>> p = re.compile('section{ ( [^}]* ) }', re.VERBOSE)
>>> p.sub(r'subsection{\1}','section{First} section{second}')
'subsection{First} subsection{second}'

Há também uma sintaxe para se referir a grupos nomeados como definido pela
sintaxe ``(?P<name>...)``. ``\g<name>`` usará a substring correspondida pelo grupo
com nome ``name`` e ``\g<number>`` utiliza o número do grupo correspondente. ``.\g<2>`` é,
portanto, equivalente a ``\2``, mas não é ambígua em uma string de substituição (replacement), tal
como ``\g<2>0``. (``\20`` seria interpretado como uma referência ao grupo de ``20``, e não
uma referência ao grupo ``2`` seguido pelo caractere literal ``0``). As seguintes


substituições são todas equivalentes, são usadas todas as três variações da string de
substituição (replacement).

>>>
>>> p = re.compile('section{ (?P<name> [^}]* ) }', re.VERBOSE)
>>> p.sub(r'subsection{\1}','section{First}')
'subsection{First}'
>>> p.sub(r'subsection{\g<1>}','section{First}')
'subsection{First}'
>>> p.sub(r'subsection{\g<name>}','section{First}')
'subsection{First}'

A substituição (replacement) também pode ser uma função, o que dá a você ainda mais controle. Se a
substituição (replacement) é uma função, a função é chamada para cada ocorrência não
sobreposta de padrão. Em cada chamada, a função passa o argumento
MatchObject para a correspondência e pode usar esta informação para calcular a
string de substituição desejada e fazer seu retorno.

No exemplo a seguir, a função de substituição (replacement) traduz decimal em hexadecimal:

>>>
>>> def hexrepl( match ):
...     """Return the hex string for a decimal number"""
...     value = int( match.group() )
...     return hex(value)
...
>>> p = re.compile(r'\d+')
>>> p.sub(hexrepl, 'Call 65490 for printing, 49152 for user code.')
'Call 0xffd2 for printing, 0xc000 for user code.'

Ao utilizar a função de nível de módulo ``re.sub()``, o padrão é passado como o primeiro
argumento. O padrão pode ser fornecido como um objeto ou como uma string; se
você precisa especificar flags de expressões regulares, você deve usar um objeto
padrão como o primeiro parâmetro, ou usar modificadores embutidos na string
padrão, por exemplo, ``sub("(?i)b+", "x", "bbbb BBBB")`` retorna ``x x``.

Problemas Comuns
----------------

Expressões regulares são uma ferramenta poderosa para algumas aplicações, mas de
certa forma o seu comportamento não é intuitivo, e às vezes, as RE não se comportam
da maneira que você espera que elas se comportem. Esta seção irá apontar algumas das
armadilhas mais comuns.

Use String Methods
------------------

Às vezes, usar o módulo ``re`` é um equívoco. Se você está fazendo correspondência com uma string
fixa, ou uma classe de caractere única, e você não está usando nenhum recurso de ``re``
como a flag **IGNORECASE**, então pode não ser necessário todo o poder das
expressões regulares. Strings possui vários métodos para executar operações com
strings fixas e eles são, geralmente, muito mais rápidos, porque a implementação é um
único e pequeno laço (loop) de C que foi otimizado para esse propósito, em vez do grande e mais generalizado mecanismo
das expressões regulares.

Um exemplo pode ser a substituição de uma string fixa única por outra; por exemplo,
você pode substituir ``word`` por ``deed``. ``re.sub()`` parece ser a função a ser usada para
isso, mas considere o método ``replace()``. Note que ``replace()`` também irá
substituir ``word`` dentro de palavras, transformando ``swordfish`` em ``sdeedfish``, mas
uma ingênua RE teria feito isso também. (Para evitar a realização da substituição
de partes de palavras, o padrão teria que ser ``\bword\b``, a fim de exigir que word
tenha um limite de palavra em ambos os lados (o recurso borda). Isso leva o tarefa para além da
capacidade de ``replace()``.)

Outra tarefa comum é apagar todas as ocorrências de um único caractere de uma
string ou substitui-lo por um outro caractere único. Você pode fazer isso com algo como
``re.sub('\n', ' ', S)``, mas ``translate()`` é capaz de fazer ambas as tarefas e
será mais rápida do que qualquer operação de expressão regular pode ser.

Em suma, antes de recorrer ao o módulo ``re``, considere se o seu problema pode ser
resolvido com um método string mais rápido e mais simples.

match() versus search()
-----------------------

A função ``match()`` somente verifica se a ``RE`` corresponde ao início da string, enquanto
``search()`` fará a varredura através na string procurando por uma correspondência. É
importante manter esta distinção em mente. Lembre-se, ``match()`` só irá relatar uma correspondência bem-sucedida
que começa em ``0``; se a correspondência não começar em ``zero``, ``match()`` não vai reportá-la.

>>>
>>> print re.match('super', 'superstition').span()

