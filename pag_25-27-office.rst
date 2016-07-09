

'colour socks and colour shoes'
>>> p.sub( 'colour', 'blue socks and red shoes', count=1)
'colour socks and red shoes'

O método subn() faz o mesmo trabalho, mas retorna uma tupla com duas informações; contém uma string
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

