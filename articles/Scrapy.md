# Web Scraping

Web Scraping é basicamente a extração de elementos de dados de páginas web.

Vamos começar explorando a página: http://quotes.toscrape.com/random

Nosso objetivo é extrair o **texto**, o **nome do autor** e as **tags** do *quote* apresentando na página.

Cada vez que carregarmos essa página, ela nos fornecerá um *quote* aleatório de algum autor, então cada vez que fizermos o *Scraping* nessa página, devemos esperar um resultado diferente.

Antes de iniciarmos o *Scraping*, é importante observarmos a estrutura da página que desejamos extrair os dados. Para essa função podemos utilizar a ferramenta *Inspect Element* do nosso navegador.

- O **texto** está representado com um elemento `<span>` e uma classe `text`
- O **nome do autor** está representado com um elemento `<small>` e uma classe `author`
- As **tags** são representadas contidas dentro de um elemento `<div>` com a classe `tags` que por sua vez contém elementos `<a>` com a classe `tag` que representa respectivamente cada **tag** única

Esse é o primeiro passo em nossa jornada, identificar a localização dos dados que desejamos na página web. Agora vejamos como podemos utilizar o framework Scrapy para fazermos a requisição da página e a extração dos dados.

Caso queira saber mais detalhes sobre o framework: https://scrapy.org/ 

Para fazermos a instalação, podemos usar `pip`:

```python
>>> pip install scrapy
```

## Scrapy Shell

Scrapy Shell é uma ferramenta que nos possibilita experimentar as diferentes possibilidades de como podemos extrair os dados que desejamos

```python
>>> scrapy shell http://quotes.toscrape.com/random
```

Scrapy Shell irá fazer o download da página da URL que passamos e nos fornecerá um objeto *response* que nós podemos utilizar para extrair dados da página. Por exemplo:

```python
>>> print(response.text) # Nos fornece todo o conteúdo da página
```

Podemos utilizar o método **response.css()** para selecionarmos a parte da página que desejamos, passamos um **seletor CSS** como parâmetro e ele retornará os elementos correspondentes com o seletor

```python
>>> response.css('small.author')
```

Como você pode perceber, o método retornou uma lista de objetos contendo os objetos selecionados. Podemos utilizar o método **extract()** para obtermos os dados HTML que realmente queremos

```python
>>> response.css('small.author').extract()
```

Como podemos ver, recebemos uma lista de *strings* como resultado, entretando é necessário mais dois passos a fazermos, primeiramente queremos nos livrar das tags HTML envolvendo o nome do autor, para isso devemos mudar um pouco o nosso **seletor**

```python
>>> response.css('small.author::text').extract()
```

Veja que agora ele nos retorna uma lista com o **nome do autor**, para obtermos uma *string* é muito simples, basta acessarmos o primeiro elemento da lista ou executar o método **extract_first()**

```python
>>> response.css('small.author::text')[0].extract()
```

```python
>>> response.css('small.author::text').extract_first()
```

Agora que conseguimos obter o **nome do autor** com sucesso, voltamos à página web e vamos buscar o **texto** do *quote*

```python
>>> response.css('span.text::text').extract_first()
```

Agora vamos em busca das **tags**

```python
>>> response.css('a.tag::text').extract_first()
```

## Criando Nosso Primeiro Spider

Em nossa linha de comando, vamos utilizar o comando `scrapy genspider quotes toscrape.com` para gerarmos o esqueleto de nosso Spider. Veja que o primeiro parâmetro é o nome do nosso Spider e o segundo é o domínio do website.

Nos será retornado e um arquivo `quotes.py` será criado

```
Created spider 'quotes' using template 'basic'
```

Executando nosso spider

```python 
>>> scrapy runspider quotes.py
```

Podemos salvar nossos resultados em um arquivo, vejamos um exemplo utilizando a extensão `.json`

```python 
>>> scrapy runspider quotes.py -o items.json
```

## Extraindo Múltiplos Items de uma Página

Vamos agora utilizar a página http://quotes.toscrape.com/ que lista 10 *quotes* de uma vez

Começaremos executando `scrapy shell 'http://quotes.toscrape.com/'`

Vamos aplicar a mesma lógica de seleção que utilizamos antes para selecionar o **autor**

```python 
>>> response.css('small.author::text').extract_first()
```

Veja que nos retornou apenas um autor, para obtermos todos devemos usar o método **extract()**

```python 
>>> response.css('small.author::text').extract()
```

Vamos selecionar agora os *quotes*

```python 
>>> response.css('span.text::text').extract()
```

E as **tags**

```python 
>>> response.css('a.tag::text').extract()
```

Perceba que há um problema, recebemos uma gigante lista de **tags** e não sabemos com qual texto essas **tags** correspondem. Para isso teremos que extrair os dados de um *quote* por vez

Observando a estrutura da página, podemos percerbar que cada *quote* está contido em uma `<div>` com a classe `quote`. Vamos tentar trabalhar com esses elementos em nossa shell

Para selecionarmos todos os *quotes* podemos utilizar o comando

```python 
>>> response.css('div.quote')
```

Nos será retornado uma lista de objetos seletores, vamos selecionar o primeiro *quote* e armazenar em uma variável chamada **quote**

```python 
>>> quote = response.css('div.quote')[0]
>>> print(quote)
# <Selector xpath="descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>
```

Agora temos o objeto seletor para o primeiro *quote*, a partir deles vamos tentar selecionar o **autor**, o **texto** e as **tags**

```python 
>>> quote.css('small.author::text').extract_first()
>>> quote.css('span.text::text').extract_first()
>>> quote.css('a.tag::text').extract_first()
```

Agora que vimos como extrair de cada elemento, vamos extender essa funcionalidade para buscarmos todos eles, para essa tarefa utilizaremos o **for** loop

```python 
for quote in response.css('div.quote'):
	item = {
		'author_name': quote.css('small.author::text').extract_first(),
		'text': quote.css('span.text::text').extract_first(),
		'tags': quote.css('a.tag::text').extract(),
	}
	print(item)
```

Veja que nos será todos os *quotes* da primeira página.

## Seguindo os Links de Paginação com Scrapy

Vamos iniciar novamente nosso `scrapy shell 'http://quotes.toscrape.com/'`

Agora vamos construir nosso seletor, estamos buscando por todos os items contidos no elemento `<li>` com a classe `next` 

```python 
>>> response.css('li.next')
```

Para obtermos especificamente o link

```python 
>>> response.css('li.next > a')
```

Obtendo apenas a string

```python 
>>> response.css('li.next > a').extract_first()
```

Selecionando apenas o que está contido no atributo **href**

```python 
>>> response.css('li.next > a::attr(href)').extract_first()
```

Observe que temos apenas uma URL relativa e nós precisamos da URL absoluta, para isso vamos usar um método específico para gerar essa URL absoluta

```python 
>>> next_page_url = response.css('li.next > a::attr(href)').extract_first()
>>> response.urljoin(next_page_url)
```

O método **urljoin** faz a união da URL base da resposta com a URL que passamos como parâmetro, veja que agora temos a URL correta

## Extraindo Páginas de Detalhes sobre os Autores

Nosso objetivo agora é extrair dados sobre os autores desses *quotes*, se olharmos com cuidado o website poderemos ver que cada *quote* contém um link para uma página de detalhes sobre o autor, nesse caso específico vamos coletar o **nome do autor** e sua **data de nascimento**

Nosso Spider funcionará da seguinte forma, primeiro ele iŕa obter a página principal contendo a lista de *quotes* então ele buscará todos os links para as páginas dos autores e para cada um deles irá criar uma nova **requisição**, quando obtivermos a **resposta** dessa **requisição** nosso Spider irá fazer o *parsing* das páginas de detalhes e irá extrair os dados, depois disso o Spider seguirá o link de página e repetirá novamente todo o processo.

Novamente vamos ao nosso `scrapy shell 'http://quotes.toscrape.com/'`

Selecionando os links das páginas de detalhes

```python 
>>> response.css('div.quote > span > a::attr(href)').extract()
```

Carregando uma página de detalhes em nossa shell

```python 
>>> fetch('http://quotes.toscrape.com/author/Jane-Austen/')
```

Extraindo o **nome do autor**

```python 
>>> response.css('h3.author-title::text').extract_first()
```

Extraindo a **data de nascimento**

```python 
>>> response.css('span.author-born-date::text').extract_first()
```

## Extraindo Dados de Páginas com Infinite Scrolling

Dessa vez vamos acessar a URL: http://quotes.toscrape.com/scroll

Ao inspecionarmos a página web podemos observar que ela faz uma requisição para a seguinte API `Request URL:http://quotes.toscrape.com/api/quotes?page=3`

Novamente vamos ao Scrapy Shell

```python 
scrapy shell 'http://quotes.toscrape.com/api/quotes?page=5'
```

O corpo da resposta está no formato `.json`

```python 
>>> print(response.text)
```

Podemos carregá-lo em um dicionário Python

```python 
>>> import json
>>> data = json.loads(response.text)
```

Verificando as chaves

```python 
>>> data.keys()
```

Lendo os *quotes*

```python 
>>> data['quotes']
>>> data['quotes'][0]
>>> data['quotes'][0]['author']
>>> data['quotes'][0]['author']['name']
```

Veja que temos também um atributo chamado **has_next**, ele nos irá indicar quando devemos parar de fazer as requisições, e o atributo **page** que nos diz em qual página atualmente estamos

```python 
>>> data['has_next']
>>> data['page']
```
