
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

+-----------+----------------------------------------------------------------+
|Caracteres | Etapa                                                          |
+-----------+----------------------------------------------------------------+
|\section   | string de texto a ser correspondida                            |
+-----------+----------------------------------------------------------------+
|\\section  | preceder com barra invertida para re.compile()                 |
+-----------+----------------------------------------------------------------+
|\\\\section| barras invertidas precedidas novamente para uma string literal |
+-----------+----------------------------------------------------------------+

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

+---------------+-------------+
|String Regular | String Crua |
+---------------+-------------+
|"ab*"          | r"ab*"      |
+---------------+-------------+
|"\\\\section"  | r"\\section"|
+---------------+-------------+
|"\\w+\\s+\\1"  | r"\w+\s+\1" |
+---------------+-------------+


Executando Comparações¶
Uma vez que você tem um objeto que representa uma expressão regular compilada, o
que você faz com ele? Objetos padrão têm vários métodos e atributos. Apenas os
mais significativos serão vistos aqui; consulte a documentação do módulo re para uma lista
completa.

Método/Atributo    Propósito
match()            Determina se a RE combina com o início da string.
search()           Varre toda a string, procurando qualquer local onde esta RE
                   tem correspondência.
findall()          Encontra todas as substrings onde a RE corresponde, e
                   as retorna como uma lista.
finditer()         Encontra todas as substrings onde a RE corresponde, e
                   as retorna como um iterator.

match() e search() retornam None se não existir nenhuma correspondência
encontrada. Se tiveram sucesso, uma instância de MatchObject é retornada,
contendo informações sobre a correspondência: onde ela começa e termina, a
substring com a qual ela teve correspondência, e mais.
Você pode aprender sobre isto experimentando interativamente o
módulo re. Se você tiver o Tkinter disponível, você pode também querer dar uma olhada
em Tools/scripts/redemo.py, um programa de demonstração incluído na
distribuição Python. Ele permite você entrar com REs e strings, e exibe se a RE
tem correspondência ou falha. redemo.py pode ser bastante útil quando se tenta
depurar uma RE complicada. Phil Schwartz’s Kodos é também uma
ferramenta interativa para desenvolvimento e teste de padrões RE.
Este HOWTO usa o interpretador Python padrão para seus exemplos. Primeiro,
execute o interpretador Python, importe o modulo re, e compile uma RE:
Python 2.7.10 (default, May 23 2015, 09:44:00) [MSC v.1500 64 bit
>>> import re
>>> p = re.compile(``[a-z]+``)
>>> p
<_sre.SRE_Pattern object at 0x...>

Agora, você pode tentar corresponder várias strings com a RE [a-z]+. Mas uma string
vazia não deveria corresponder com nada, desde que ``+`` significa ``uma ou
mais repetições``. match() deve retornar None neste caso, o que fará com que o


interpretador não imprima nenhuma saída. Você pode imprimir explicitamente o
resultado de match() para deixar isso claro.
>>>
>>> p.match("")
>>> print p.match("")
None

Agora, vamos experimentá-la em uma string que ela deve corresponder, como tempo.
Neste caso, match() irá retornar um MatchObject, assim que você deve armazenar
o resultado em uma variável para uso posterior.

>>>
>>> m = p.match(``tempo``)
>>> print m
<_sre.SRE_Match object at 0x...>

Agora você pode consultar o MatchObject para obter informações sobre a string
correspondente. Instâncias do MatchObject também tem vários métodos e atributos;
os mais importantes são os seguintes:

Método/Atributo     Propósito
group()             Retorna a string que corresponde com a RE
start()             Retorna a posição inicial da string correspondente
end()               Retorna a posição final da string correspondente
span()              Retorna uma tupla contendo as posições (inicial, final) da
                    string combinada

Experimentando estes métodos teremos seus significado esclarecidos:
>>>
>>> m.group()
``tempo``
>>> m.start(), m.end()
(0, 5)
>>> m.span()
(0, 5)

group() retorna a substring correspondeu com a RE. start()e end() retornam
os índices inicial e o final da substring correspondente. span()retorna tanto os índices
inicial e final em uma única tupla. Como o método match() somente verifica se a RE
corresponde com o início de uma string, start() será sempre zero. No entanto, o

