
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

É obviamente muito mais fácil fazer referência a m.group('zonem'), do que ter que se
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

