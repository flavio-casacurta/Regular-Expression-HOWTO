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
Herbert Parentes Fortes Neto <brpacotes@gmail.com>


Abstrato
--------
Este documento é um tutorial introdutório ao uso de expressões regulares em Python
com o módulo re. Ele fornece uma introdução mais suave do que a seção
correspondente na Biblioteca de Referência.

Introdução
----------
O módulo *re* foi adicionado no Python 1.5, e provê padrões de expressões regulares
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

¶

Vamos começar por aprender sobre as expressões regulares mais simples possíveis.
Como as expressões regulares são usadas para operar em strings, vamos começar
com a tarefa mais comum: de correspondência caracteres.
Para uma explicação detalhada da ciência da computação referente a expressões
regulares (autômatos finitos determinísticos e não-determinístico), você pode consultar
a praticamente qualquer livro sobre a escrita de compiladores.

Caracteres Correspondentes¶
A maioria das letras e caracteres simplesmente irão corresponder entre si. Por exemplo, a expressão regular 'teste'
irá combinar com a string 'teste' totalmente. (Você pode
habilitar o modo de maiúsculas e minúsculas que faria com que a RE corresponder com
Test ou TEST também; veremos mais sobre isso mais adiante.)
Há exceções a essa regra, alguns caracteres são metacaracteres especiais, e não se
correspondem. Em vez disso, eles sinalizam que alguma coisa fora do normal deve
ser correspondida, ou eles afetam outras partes da RE, repetindo-as ou alterando seus
significados. Grande parte deste documento é dedicada à discussão de vários metacaracteres
e o que eles fazem.
Aqui está a lista completa dos metacaracteres; seus significados serão discutidos ao
longo deste documento.
. ^ $ * + ? { } [ ] \ | ( )
O primeiro metacaractere que vamos olhar são os colchetes, [ e ]. Eles são usados para
especificar uma classe de caracteres, que é um conjunto de caracteres que você
deseja corresponder. Os caracteres podem ser listados individualmente, ou um
intervalo de caracteres pode ser indicado informando dois caracteres e separando-os por
um '-'. Por exemplo, [abc] irá corresponder a qualquer dos caracteres a, b, c ou, o que
é o mesmo que [a-c], que usa um intervalo de expressar o mesmo conjunto de
caracteres. Se você quiser corresponder apenas letras minúsculas, a RE seria [a-z].
Metacaracteres não são ativos dentro classes “[ ]”. Por exemplo, [akm$] irá
corresponder a qualquer um dos caracteres 'a', 'k', 'm', ou '$'; '$' é geralmente um


metacaractere, mas dentro de uma classe de caracteres ele é despojado de sua natureza
especial.
Você pode combinar os caracteres 'não listados' dentro de um classe,
através de um complemento do conjunto. Isto é indicado através da inclusão de um '^' como o
primeiro caractere da classe; '^' fora de uma classe de caracteres simplesmente
corresponder ao caractere '^'. Mas por exemplo, [^5] irá corresponder a qualquer caractere,
exceto '5'.
Talvez o metacaractere mais importante é a barra invertida, \. Como as strings literais em
Python, a barra invertida pode ser seguida por vários caracteres para sinalizar várias
sequências especiais. Ela também é usada para "escapar" todos os metacaracteres,
e assim, você poder combiná-los em padrões; por exemplo, se você precisa
fazer correspondência a um [ ou \, você pode precedê-los com uma barra invertida para
remover seu significado especial: \[ ou \\.
Algumas das sequências especiais que começam com '\' representam conjuntos
predefinidos de caracteres que muitas vezes são úteis, como o conjunto de dígitos, o
conjunto de letras, ou o conjunto de qualquer coisa que não seja um espaço em branco. As seguintes sequências especiais pré-definidas
são um subconjunto das disponíveis. As classes
equivalentes são para padrões de "byte string" (byte string patterns). Para uma lista completa de
definições de sequências e classe estendida para os padrões string
Unicode, consulte a última parte de 'Sintaxe de Expressões Regulares'.
\d corresponde a qualquer dígito decimal, que é equivalente à classe [0-9].
\D corresponde a qualquer caractere não-dígito, o que é equivalente à classe [^0-9].
\s corresponde a qualquer caractere 'espaço-em-branco', o que é equivalente à
classe [\t\n\r\f\v].
\S corresponde a qualquer caractere 'não-espaço-branco', o que é equivalente à classe
[^ \t\n\r\f\v].
\w corresponde a qualquer caractere alfanumérico, o que é equivalente à classe [azA-Z0-9_].
\W Corresponde a qualquer caractere não-alfanumérico, o que é equivalente à classe
[^a-zA-Z0-9_].
Estas sequências podem ser incluídas dentro de uma classe caractere. Por exemplo,
[\s,.] É uma classe caractere que irá corresponder a qualquer caractere 'espaço-em-branco', ou ',' ou '.'.
O metacaractere final desta seção é o '.'. Ele encontra tudo, exceto um caractere
'nova linha', e existe um modo alternativo (re.DOTALL), onde ele irá corresponder
até mesmo a um caractere 'nova linha'. '.' é , geralmente, usado quando você quer corresponder com "qualquer caractere".
