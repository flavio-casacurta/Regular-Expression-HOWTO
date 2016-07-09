

(0, 5)
>>> print re.match('super', 'insuperable')
None

Por outro lado, ``search()`` fará a varredura percorrendo a string e relatando a
primeira correspondência que encontrar.

>>>
>>> print re.search('super', 'super.stition').span()
(0, 5)
>>> print re.search('super', 'insuperable').span()
(2, 7)

Às vezes, você vai ficar tentado a continuar usando ``re.match()``, e apenas adicionar ``.*`` ao início de sua RE.
Resista a essa tentação e use ``re.search()`` em vez disso. O
compilador de expressão regular faz alguma análise das REs, a fim de acelerar o
processo de procura de uma correspondência. Tal análise descobre o que o primeiro
caractere de uma string deve ser; por exemplo, um padrão começando com ``Crow``
deve corresponder com algo iniciando com ``C``. A análise permite que o mecanismo faça a varredura rapidamente através
da string a procura do caractere inicial, apenas tentando a combinação completa se um ``C`` for encontrado.

Adicionar um ``.*`` evita essa otimização, sendo necessário a varredura até o final da string e, em seguida, retroceder
para encontrar uma correspondência para o resto da RE. Use ``re.search()`` em vez disso.

Gulosos versus não Gulosos
--------------------------

Ao repetir uma expressão regular, como em ``a*``, a ação resultante é consumir o tanto do
padrão quanto possível. Este fato, muitas vezes derruba você quando você está tentando
corresponder com um par de delimitadores balanceados, tal como os colchetes que cercam uma tag
HTML. O padrão ingênuo para combinar uma única tag HTML não funciona por causa
da natureza gulosa de ``.*``.

>>>
>>> s = '<html><head><title>Title</title>'
>>> len(s)
32
>>> print re.match('<.*>', s).span()
(0, 32)
>>> print re.match('<.*>', s).group()
<html><head><title>Title</title>


A RE corresponde a ``<`` em ``<html>``, e o ``.*`` consome o resto da string. Existe ainda
mais coisa existente na RE, no entanto, e o ``>`` pode não corresponder com o final da string,
de modo que o mecanismo de expressão regular tem que recuar caractere por
caractere até encontrar uma correspondência para a ``>``. A correspondência final se
estende do ``<`` em ``<html>`` ao ``>`` em ``</title>``, que não é o que você quer.

Neste caso, a solução é usar os qualificadores não-gulosos ``*?, +?,??``, or ``{m,n}?``,
que corresponde com o mínimo de texto possível. No exemplo acima, o ``>`` é tentado
imediatamente após a primeira correspondência de ``<``, e quando ele falhar, o mecanismo avança
um caractere de cada vez, experimentado ``>`` a cada passo. Isso produz justamente o
resultado correto:

>>>
>>> print re.match('<.*?>', s).group()
<html>

(Note que a análise de HTML ou XML com expressões regulares é dolorosa. Padrões
"sujos e rápidos" irão lidar com casos comuns, mas HTML e XML tem casos especiais
que irão quebrar expressões regulares óbvias; com o tempo, expressões regulares que você venha a escrever para lidar com todos os casos possíveis, se tornarão um padrão muito
complicado. Use um módulo de análise de HTML ou XML para tais tarefas.)

Usando re.VERBOSE
-----------------

Nesse momento, você provavelmente deve ter notado que as expressões regulares são de uma notação
muito compacta, mas não é possível dizer que são legíveis. REs de complexidade
moderada podem se tornar longas coleções de barras invertidas, parênteses e
metacaracteres, fazendo com que se tornem difíceis de ler e compreender.

Para tais REs, especificar a flag ``re.VERBOSE`` ao compilar a expressão regular
pode ser útil, porque permite que você formate a expressão regular de forma mais clara.

A flag ``re.VERBOSE`` produz vários efeitos. Espaço em branco na expressão regular que
não está dentro de uma classe de caracteres é ignorado. Isto significa que uma
expressão como ``dog | cat`` é equivalente ao menos legível ``dog|cat``, mas ``[a b]``
ainda vai coincidir com os caracteres ``a``, ``b``, ou um ``espaço``. Além disso, você
também pode colocar comentários dentro de uma RE; comentários se estendem de um
caractere ``#`` até a próxima nova linha. Quando usados junto com strings de aspas
triplas, isso permite as REs serem formatadas mais ordenadamente::

    pat = re.compile(r"""
    \s*                 # Skip leading whitespace
    (?P<header>[^:]+)   # Header name
    \s* :               # Whitespace, and a colon
    (?P<value>.*?)      # The header's value -- *? used to
                        # lose the following trailing whitespace
    \s*$                # Trailing whitespace to end-of-line
    """, re.VERBOSE)

Isso é muito mais legível do que::

    pat = re.compile(r"\s*(?P<header>[^:]+)\s*:(?P<value>.*?)\s*$")

Comentários
-----------

Expressões regulares são um tópico complicado. Esse documento ajudou você a compreendê-las?
Existem partes que foram pouco claras, ou situações que você vivenciou
que não foram abordadas aqui? Se assim for, por favor, envie sugestões de melhorias
para o autor.
O livro mais completo sobre expressões regulares é quase certamente o Mastering Regular Expressions de Jeffrey Friedl’s,
publicado pela O'Reilly. Infelizmente, ele se concentra exclusivamente em sabores de expressões regulares do Perl e
do Java, e não contém qualquer material relativo a Python, por isso não vai ser útil como uma referência para a
programação em Python. (A primeira edição cobre o módulo regex agora removido do
Python, o que não vai te ajudar muito.) Considere removê-lo de sua biblioteca.

N.T.
----

    Meu primeiro contato com expressões regulares foi com o livro Expressões
    Regulares - Guia de Consulta Rápida por Aurelio Marinho Jargas e Editora
    Novatec, ©2001, é uma abordagem divertida sobre expressões regulares
    embora não seja específico sobre Python, você aprende brincando com REs. E
    mesclando com este HOWTO dará a você uma boa base de conhecimentos
    sobre RE.

    http://aurelio.net/regex/guia/

Alguns bons recursos para testar suas RE s, são:
http://www.pyregex.com/
http://tools.lymas.com.br/regexp_br.php#


Rodapé
[1] Introduced in Python 2.2.2.


