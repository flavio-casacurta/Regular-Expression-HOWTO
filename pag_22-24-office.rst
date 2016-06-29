
Agora, considere complicar um pouco o problema; e se você desejar
corresponder com nomes de arquivos onde a extensão não é ``bat``? Algumas tentativas
incorretas::

    .*[.][^b].*$

A primeira tentativa acima tenta excluir bat, exigindo que o primeiro caractere da
extensão não é um b. Isso é errado, porque o padrão também não corresponde
``foo.bar``.

    .*[.]([^b]..|.[^a].|..[^t])$

A expressão fica mais confusa se você tentar remendar a primeira solução,
exigindo que uma das seguintes situações corresponda: o primeiro caractere da extensão não é ``b``; o
segundo caractere não é ``a``; ou o terceiro caractere não é ``t``. Isso aceita
``foo.bar`` e rejeita ``autoexec.bat``, mas requer uma extensão de três letras e não
aceitará um nome de arquivo com uma extensão de duas letras, tal como ``sendmail.cf``.
Nós iremos complicar o padrão novamente em um esforço para corrigi-lo.

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
sub()            | Encontra todas as substrings que correspondem com a RE e faz a substituição por uma string diferente|
+----------------+-----------------------------------------------------------------------------------------------------+
subn()           | Faz a mesma coisa que sub(), mas retorna a nova string e o número de substituições                  |
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
Quando ``maxsplit`` é diferente de zero, um determinado número de divisões maxsplit será executado, e o
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

Busca e Substituição¶
Outra tarefa comum é encontrar todas as combinações para um padrão e substituí-las
por uma string diferente. O método sub() recebe um valor de substituição, que pode
ser uma string ou uma função, e a string a ser processada.

.sub(replacement, string[, count=0])
Retorna a string obtida substituindo as ocorrências mais à esquerda não sobrepostas
da RE em ‘string’ pela substituição 'replacement'. Se o padrão não for encontrado, a
string é retornada inalterada.
O argumento opcional 'count' é o número máximo de ocorrências do padrão a ser
substituído; 'count' deve ser um número inteiro não negativo. O valor padrão 0
significa para substituir todas as ocorrências.
Aqui está um exemplo simples do uso do método sub(). Ele substitui nomes de
cores pela palavra 'colour':

>>>
>>> p = re.compile( '(blue|white|red)')
>>> p.sub( 'colour', 'blue socks and red shoes')

