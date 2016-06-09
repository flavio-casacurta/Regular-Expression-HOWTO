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
de cada uma.

+--------------+-------------------------------------------------------------------------------------------------------+
|Flag		   |Significado                                                                                            |
+--------------+-------------------------------------------------------------------------------------------------------+
|DOTALL, S	   |Faz o '.' corresponder a qualquer caractere, incluindo novas linhas                                    |
+--------------+-------------------------------------------------------------------------------------------------------+
|IGNORECASE, I |Faz correspondências com maiúsculas e minúsculas                                                       |
+--------------+-------------------------------------------------------------------------------------------------------+
|LOCALE, L	   |Faz uma correspondência de acordo com o idioma                                                         |
+--------------+-------------------------------------------------------------------------------------------------------+
|MULTILINE, M  |Correspondência multi-linha, afetando ^ e $                                                            |
+--------------+-------------------------------------------------------------------------------------------------------+
|VERBOSE, X	   |Habilita REs detalhadas, que podem ser organizadas de forma mais clara e compreensível.                |
+--------------+-------------------------------------------------------------------------------------------------------+
|UNICODE, U	   |Faz de uma letra precedida pela barra invertida ('\') tal como \w, \b \s e \d dependente da base de    |
+--------------+-------------------------------------------------------------------------------------------------------+
|              |                                                                           dados de caracteres Unicode.|
+--------------+-------------------------------------------------------------------------------------------------------+

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
como uma letra. Definir a flag ``LOCALE``` no momento de compilar uma expressão regular fará com que o objeto compilado resultante
use essas funções de C para ``\w``; isso causa lentidão, mas também permite que ``\w+`` corresponda com as palavras em francês,
caso seja necessário.

**M**

**MULTILINE**
