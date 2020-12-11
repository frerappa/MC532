# Lab08 - PubChem e DRON com XQuery/XPath

## Tarefas com Publicações

## Questão 1
Construa uma comando SELECT que retorne dados equivalentes a este XPath
~~~xquery
//individuo[idade>20]/@nome
~~~

### Resolução
~~~sql
SELECT nome FROM Individuo WHERE idade > 20;
~~~

## Questão 2
Qual a outra maneira de escrever esta query sem o where?

~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
 
for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~
### Resolução
~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
 
for $i in ($fichariodoc//individuo[idade>17])
return {data($i/@nome)}
~~~

## Questão 3
Escreva uma consulta SQL equivalente ao XQuery:
~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')

for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~

### Resolução
~~~sql
SELECT nome FROM Individuo WHERE idade > 17;
~~~

## Questão 4
Retorne quantas publicações são posteriores ao ano de 2011.

### Resolução
~~~xquery
let $doc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')

let $pub := $doc//publication
return count($pub[year>2011])
~~~

## Questão 5
Retorne a categoria cujo `<label>` em inglês seja 'e-Science Domain'.

### Resolução
~~~xquery
let $doc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')

for $cat in ($doc//categories)
where $cat/@catkey = "subject"
for $cat2 in $cat//category
where $cat2//label/@lang = "en-US"
return {$cat2[label = 'e-Science Domain']}
~~~

## Questão 6
Retorne as publicações associadas à categoria cujo `<label>` em inglês seja 'e-Science Domain'. A associação entre o label e a key da categoria deve ser feita na consulta.

### Resolução
~~~xquery
let $doc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')


for $cat in ($doc//categories)
where $cat/@catkey = "subject"
for $cat2 in $cat//category
where $cat2//label/@lang = "en-US"
let $key := data($cat2[label = 'e-Science Domain']/@key)
for $l in ($doc//publication)
let $chave := $l//key
where data($chave) = $key
return {data($l//title), '&#xa;'}
~~~

## Tarefas com DRON e PubChem

## Questão 1

Liste o nome de todas as classificações que estão apenas dois níveis imediatamente abaixo da raiz.

### Resolução
~~~xquery
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
for $droga in ($dron/drug/drug/drug)
return {data($droga/@name), '&#xa;'}
~~~

## Questão 2

Apresente todas as classificações de um componente a sua escolha (diferente de `Acetylsalicylic Acid`) que estejam hierarquicamente dois níveis acima. Note que no exemplo dado em sala foi considerado um nível hierárquico acima.

### Resolução
~~~xquery
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
for $droga in ($dron//drug[drug/drug/@name="CLOBETASOL"])
let $nome := $droga/@name
group by $nome
return {$nome, '&#xa;'}
~~~

## Questão 3

### Questão 3.1

Liste todos os códigos ChEBI dos componentes disponíveis.

#### Resolução
~~~xquery
let $src := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')

for $Information in $src//InformationList//Synonym
where substring(data($Information), 1, 5) = 'CHEBI'
return {data($Information), '&#xa;'}
~~~

### Questão 3.2

Liste todos os códigos ChEBI e primeiro nome (sinônimo) de cada um dos componentes disponíveis.

#### Resolução
~~~xquery
let $src := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')

for $Information in $src//InformationList
for $Synonym in $Information//Synonym
where substring(data($Synonym), 1, 5) = 'CHEBI'
return {'ChEBI: ', data($Synonym), ', Nome: ', data($Information//Synonym[1]), '&#xa;'}
~~~

### Questão 3.3

Para cada código ChEBI, liste os sinônimos e o nome que aparece para o mesmo componente no DRON (se existir). Para isso faça um JOIN com o DRON.

#### Resolução
~~~xquery
let $componentes := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')

for $Information in $componentes//InformationList
for $Synonym in $Information//Synonym
where substring(data($Synonym), 1, 5) = 'CHEBI'
let $chebi := substring(data($Synonym), 7)
for $droga in ($dron//drug)
where concat('http://purl.obolibrary.org/obo/CHEBI_', $chebi) = $droga/@id
let $nome := $droga/@name
group by $nome
return {'Nome DRON: ', data($nome), 'ChEBI: ', data($chebi), 'Sinonimo: ', data($Information//Synonym), '&#xa;'}
~~~