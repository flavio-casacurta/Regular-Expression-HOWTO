

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

Mais Metacaracteres¶
Existem alguns metacaracteres que nós ainda não vimos. A maioria deles serão referenciados
nesta seção.
Alguns dos metacaracteres restantes a serem discutidos são como uma afirmação de "largura zero" (zero-width assertions). Eles
não fazem com que o mecanismo avance pela string; ao contrário, eles não consomem
nenhum caractere, e simplesmente tem sucesso ou falha. Por exemplo, \b é
uma afirmação de que a posição atual está localizada nas bordas de uma palavra; a
posição não é alterada de nenhuma maneira por \b. Isto significa que afirmações de "largura zero"
nunca devem ser repetidas, porque se elas combinam uma vez em um
determinado local, elas podem, obviamente, combinar um número infinito de
vezes.

|
Alternância, ou operador "or". Se A e B são expressões regulares, A|B irá
corresponder com qualquer string que corresponder com A ou B. | tem uma prioridade muito baixa,
a fim de fazê-lo funcionar razoavelmente quando você está alternando entre strings de
vários caracteres. Crow|Servo irá corresponder tanto com Crow quanto com Servo, e não com Cro,
'w' ou 'S', e ervo.
Para corresponder com um '|' literal, use \|, ou coloque ele dentro de uma classe de
caracteres, como em [|].

^
Corresponde ao início de linha. A menos que a flag MULTILINE tinha sido definida,
isso só irá corresponder ao início da string. No modo MULTILINE, isso também
corresponde imediatamente após cada nova linha de dentro da string.
Por exemplo, para ter correspondência com a palavra 'From' apenas no início de uma linha, a
RE a ser usada é ^From.
>>> print re.search('^From', 'From Here to Eternity')
<_sre.SRE_Match object at 0x...>
>>> print re.search('^From', 'Reciting From Memory')
None

$
Corresponde ao fim de uma linha, que tanto é definido como o fim de uma string, ou

qualquer local seguido por um caractere de nova linha.
>>> print re.search('}$', '{block}')
<_sre.SRE_Match object at 0x...>
>>> print re.search('}$', '{block} ')
None
>>> print re.search('}$', '{block}\n')
<_sre.SRE_Match object at 0x...>


Para corresponder com um '$' literal, use '\$' ou coloque-o dentro de uma classe de
caracteres, como em [$].

\A
Corresponde apenas com o início da string. Quando não estiver em modo MULTILINE, \A
e ^ são efetivamente a mesma coisa. No modo MULTILINE, eles são diferentes: \A continua a
corresponder apenas com o início da string, mas ^ pode corresponder com qualquer localização de dentro da string, que seja posterior a um caractere nova linha.

\Z
Corresponde apenas ao final da string.

\b
Borda de palavra. Esta é uma afirmação de "largura zero" que corresponde apenas ao
início ou ao final de uma palavra. Uma palavra é definida como uma sequência de
caracteres alfanuméricos, de modo que o fim de uma palavra é indicado por espaços
em branco ou um caractere não alfanumérico.

