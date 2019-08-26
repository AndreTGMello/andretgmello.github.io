---
layout: post
title:  "Aprendendo pandas num bug do Kaggle"
date:   2019-08-26 09:00:00 -0300
categories: data-science
lang: pt
lang-ref: pandas-trick-kaggle-bug
---

Tem um tempo desde que considerei criar uma conta no [Kaggle](https://kaggle.com). Na verdade, eu já tinha registrado uma conta uns três anos atrás, mas nunca mexi em nada por lá.

Caso você não conheça o Kaggle, ele é famoso principalmente pelas suas [competições](https://www.kaggle.com/competitions). Essas competições abrangem diferentes áreas, do reconhecimento de imagens à saúde e predição de vendas. Resumidamente, eles fornecem os dados, o problema a ser resolvido e, através da ciência de dados, IA e aprendizado de máquina, você tenta obter os resultados desejados. Eu nunca tive muita vontade em participar, principalmente porque não sentia que estava realmente pronto. Além disso, de um modo geral, competições nunca soaram muito atrativas para mim.

Mas eles lançaram alguns [cursos muito interessantes](https://www.kaggle.com/learn/overview) recentemente, cobrindo de [pandas](https://www.kaggle.com/learn/pandas) a [ explicabilidade no aprendizado de máquina](https://www.kaggle.com/learn/machine-learning-explainability). Eu sou uma pessoa que aprende melhor por meio de exercícios práticos, então estou apaixonado por esse novo recurso. Um desses novos cursos é o [Introdução ao SQL](https://www.kaggle.com/learn/intro-to-sql). Ele faz parte do [SQL Summer Camp](https://www.kaggle.com/sql-summer-camp).

É um curso bastante introdutório, então ele não deve ser difícil caso você já esteja familiarizado com SQL. No entanto, como parte de um Summer camp para iniciantes, achei o curso muito interessante. Existe até uma [playlist do YouTube](https://www.youtube.com/watch?v=jYQoQfFzJRw&list=PLqFaTIg4myu9neIs_wfWzgeOkKbiImXB6) pra te ajudar no aprendizado.

Os cursos são compostos por um texto que cobre um pouco da teoria seguida de uma prática sobre o tópico explorado. A sessão prática é realizada em um [kernel/notebook](https://www.kaggle.com/docs/kernels) e é muito bem feita. Os notebooks contam inclusive com um recurso bem bacana para verificar se você obteve os resultados corretos ou não, e pode até dar dicas de como melhorar sua resposta.

Porém, durante [um dos exercícios](https://www.kaggle.com/dansbecker/as-with) na Introdução ao SQL, no qual eu devia descobrir o número total de viagens de táxi de cada ano a partir de um [ conjunto de dados](https://console.cloud.google.com/marketplace/details/city-of-chicago-public-data/chicago-taxi-trips) [do bigquery](https://cloud.google.com/bigquery/), eu encontrei um bug.

A consulta é bem simples.

```
rides_per_year_query = "" "
    SELECT
        EXTRACT (ANO DE trip_start_timestamp) como ano,
        COUNT (1) como num_trips
    FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`
    GRUPO POR ANO
    PEDIDO POR ano DESC
                        "" "
```

Ela não parecia conter qualquer tipo de erro, além de parecer devolver a resposta correta. No entanto, a função de verificação de perguntas do Kaggle não gostou muito do resultado que eu obtive e me retornou [esse erro](https://pastebin.com/rePWBm5G). Fiquei muito confuso. Depois de muito quebrar a cabeça decidi dar uma olhada na solução e, pra minha surpresa, ela só diferia da minha por não ter o comando `DESC`.

Decidi relatar esse comportamento estranho [fórum](https://www.kaggle.com/product-feedback/105149) do Kaggle. Pouco tempo se passou e outro usuário comentou no meu post com uma sacada simples e interessante do que estava acontecendo.

[Alexander Savchuck](https://www.kaggle.com/amsavchuk) explicou o erro em seu comentário no meu post e também criou um [kernel](https://www.kaggle.com/amsavchuk/learntools-checking-in-sql-summer-camp) para exemplificar melhor o raciocínio dele. Você pode dar uma olhada no que ele encontrou lá, mas, se você ainda estiver um pouco perdido com todas essas informações sobre cursos, kernels e verificação de perguntas, vou falar sobre o erro e a solução aqui em detalhes.

Basicamente a função de verificação de perguntas no curso Introdução à SQL para esta pergunta funciona verificando três coisas. Primeiro, ele verifica se você obteve os nomes de coluna corretos. Isso é simples e obviamente muito importante. Em seguida, verifica o tamanho do dataframe gerado a partir da sua consulta. Por fim, ele compara o valor de uma das linhas do seu dataframe com o da resposta esperada. Isso deve ser suficiente para garantir que você tenha a resposta certa.

Mas então o bug parecia surgir quando a função tentava recuperar a resposta dos usuários da seguinte maneira

`submitted_number = int(results[results["year"]==year_to_check]["num_trips"][0])`

(você pode ver como isso é feito em detalhes no [Github do Kaggle](https://github.com/Kaggle/learntools/tree/master/learntools); mais especificamente, estou olhando para [esse código](https: / /github.com/Kaggle/learntools/blob/master/learntools/sql/ex5.py).)

O problema de verificar a resposta dessa maneira, como você pode ver na última linha do erro (`KeyError: 0`), é que o pandas não processa a indexação da mesma maneira que o python lida com uma lista ou um array.

Pode-se olhar para este trecho de código e imaginar que ao adicionar `[0]` ao final do dataframe ele estará recuperando o primeiro elemento/linha do conjunto de dados. Mas não é isso que ele faz. Adicionar `[0]` à transformação do seu pandas fará com que ele se comporte como um `loc` em vez de `iloc`.

Ok, vou voltar um pouco. O que `loc` e `iloc` fazem? [Na verdade, é bastante simples](https://stackoverflow.com/questions/31593201/how-are-iloc-ix-and-loc-different). `dataframe.iloc[0]` retornará a primeira linha no seu dataframe. Isso é basicamente o mesmo que fazer `array[0]` para recuperar o primeiro elemento de um array. Mas `dataframe[0]` será traduzido para `dataframe.loc[0]`, que fornecerá a linha cujo **rótulo** (**label**) é `0`. Portanto, `loc` não tem nada a ver com a posição da linha, apenas com sua label.

Isso torna o erro mais claro: como ordenei minha query em ordem decrescente, de 2019 a 2013, meu dataframe ficou assim

```
year  num_trips
0  2019    9843414
1  2018   20732088
2  2017   24988003
3  2016   31759339
4  2015   32385875
5  2014   37395436
6  2013   27217716
```

enquanto o resultado esperado era

```
   year  num_trips
0  2013   27217716
1  2014   37395436
2  2015   32385875
3  2016   31759339
4  2017   24988003
5  2018   20732088
6  2019    9843414
```

Então, se eu tentar rodar

`submit_number_row = results[results ["year"] == year_to_check]["num_trips"]`

com `year_to_check = 2013` no meu dataframe (que ordenei com `DESC`), o resultado seria

```
6    27217716
Name: num_trips, dtype: int64
```
(e isso é uma _series_ de pandas).

Tentar fazer `submit_number = int (submit_number_row[0])` traria o erro mencionado acima. Como isso seria traduzido para `submit_number = int(submit_number_row.loc[0])`, conforme mencionei anteriormente, o pandas procuraria a label `0`, enquanto a única linha presente no dataframe recuperado possui a label`6`.

Se eu executasse a mesma consulta (`submit_number_row = results [results["year"] == year_to_check]["num_trips"]`) no dataframe com a ordenação padrão, obteria como resultado

```
0    27217716
Name: num_trips, dtype: int64
```

e, nesse caso, nenhum erro seria gerado, pois o rótulo `0` realmente está presente nesse dataframe.

E então era esse o problema escondido naquela mensagem de erro gigantesca! Como vimos na última linha do erro, o pandas simplesmente não conseguia encontrar a label `0` no dataframe fornecido (`KeyError: 0`).

Então, Alexander e [Alexis Cook](https://www.kaggle.com/alexisbcook), membro da equipe do Kaggle responsável por este curso, apresentaram duas soluções para esse problema [no post](https://www.kaggle.com/product-feedback/105149).

Alexander simplesmente sugeriu o uso de `iloc[0]` explicitamente para recuperar o primeiro elemento do dataframe corretamente. Como em

```
submitted_number =
  int(results[results["year"]==year_to_check]["num_trips"].iloc[0])
```

Alexis optou por usar `.values`, com algumas pequenas alterações no código original

```
submitted_number =
  int(results.loc[results["year"]==year_to_check]["num_trips"].values)
```

No fim, ambas as opções resolvem o problema.

Para mim, este foi um ótimo lembrete de que você sempre pode aprender com os erros. E não apenas os seus, como também com o dos outros! E não estou de forma alguma desmerecendo o Kaggle. Eles resolveram o problema rapidamente e eu estou achando fantástico os novos cursos e todos os recursos que eles desenvolveram para tornar essa uma adição muito valiosa ao site deles.

De todo modo, a mensagem permanece. Aprenda em todas as oportunidades que tiver, principalmente através de erros.

"Aprenda com os erros dos outros. Você nunca irá viver o suficiente para cometer todos eles." [Groucho Marx](https://pinterest.com/pin/533535887077279336/), aparentemente. Ou talvez [Eleanor Roosevelt](https://pinterest.com/pin/378020962462200152/). Bem, pelo menos um dos dois deve estar errado.
