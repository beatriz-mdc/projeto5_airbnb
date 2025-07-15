# Etapas do projeto

***1. Processamento e preparação da base de dados***

Os dados foram disponibilizados pela Laboratoria em pasta zipada com 3 planilhas CSV nomeadas “rooms” (tabela dimensão), “hosts” (tabela dimensão) e "reviews" (tabela fato). 
Esses arquivos contém informações sobre os donos dos anúncios, quartos disponíveis e as avaliações dos clientes.

Foi realizado o upload dos arquivos no bigquery. 

***Identificar e tratar valores nulos***

Foram encontrados nulos nas 3 tabelas. Foi decidido excluir os dados nulos, uma vez que esses dados faltantes não contribuem para a compreesão do projeto.
Abaixo queries:

```sql
DELETE FROM `projeto-5-463516.airbnb.rooms`
WHERE neighbourhood_group is null;
```

```sql
DELETE FROM `projeto-5-463516.airbnb.reviews`
WHERE reviews_per_month is null;
```

```sql
DELETE FROM `projeto-5-463516.airbnb.hosts`
WHERE nome_do_host IS NULL;
```

***Verificar e alterar o tipo de dados***

Os dados "id" e "host_id" da tabela "reviews" e "id" da tabela "rooms" foram alterados de INTEREGER para STRING.
Abaixo queries:

```sql
CREATE OR REPLACE TABLE `projeto-5-463516.airbnb.reviews` AS
SELECT
CAST(id AS STRING) AS id,
CAST(host_id AS STRING) AS host_id,
price,
number_of_reviews,
last_review,
reviews_per_month,
calculated_host_listings_count,
availability_365
FROM `projeto-5-463516.airbnb.reviews`
```

```sql
CREATE OR REPLACE TABLE `projeto-5-463516.airbnb.rooms` AS
SELECT
CAST(id AS STRING) AS id,
name,
neighbourhood,
neighbourhood_group,
latitude,
longitude,
room_type,
minimum_nights
FROM `projeto-5-463516.airbnb.rooms`
```

***Relacionar tabelas***

Após o BigQuery, as tabelas foram importadas e relacionadas dentro do Power BI.
As tabelas dimensão foram conectadas à tabela fato através da cardinalidade "muitos para um".

***Criar novas variáveis***

Uma nova coluna calculada foi criada na tabela "reviews". A nova variável calcula o total que um quarto pode gerar com base no preço e na quantidade de dias disponíveis em um ano. Abaixo cálculo:

```
renda_quarto = reviews[price]*reviews[availability_365]
```

***Utilizar fórmulas DAX***

Fórmulas DAX foram utilizadas para calcular as seguintes variáveis:

- O número total de hóspedes que um bairro pode receber em um ano (assumindo que todos os quartos estão disponíveis todos os dias do ano) de acordo com o número mínimo de noites permitidas para reservar cada quarto.
- A % de quartos disponíveis por bairro.

Abaixo os cálculos:

```
Reservas_Quarto = 
IF(
    rooms[minimum_nights] > 0,
    365 / rooms[minimum_nights],
    0
)
```

```
Pct_Quartos_Disponiveis = 
DIVIDE(
    CALCULATE(
        COUNTROWS(reviews),
        reviews[availability_365] > 0
    ),
    COUNTROWS(reviews)
)
```

***2. Análise exploratória***

***Agrupar e visualizar as variáveis categóricas***

As variáveis categóricas agrupadas e visualizadas em gráficos conforme abaixo:

<img width="704" height="252" alt="image" src="https://github.com/user-attachments/assets/21712490-4e4e-462c-ad82-06f070ffd08a" />

***Aplicar medidas de tendência central***

Foram calculadas a média, mediana e desvio padrão para a variável numérica "price" que informa o valor de aluguel dos quartos. Os valores estão resumidos na tabela abaixo:

<img width="315" height="75" alt="image" src="https://github.com/user-attachments/assets/db6168d7-df2f-42b2-968a-76f41969a50b" />

Conseguimos ver que há uma diferença considerável entre média e mediana, o que indica que há valores altos puxando a média para cima.
Podemos confirmar isso com o desvio padrão que indica alta dispersão, ou seja, os preços variam bastante em torno da média. Isso pode ocorrer por diferença de localização, tipo de imóvel, época do ano etc.

***Visualizar distribuição***

A distribuição foi analisada através de histograma conforme abaixo:

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('reviews.csv', delimiter=';')

plt.hist(df['price'], bins=20, color='orange', edgecolor='black')
plt.title('Histograma da Coluna price')
plt.xlabel('Valor')
plt.ylabel('Frequência')
plt.show()
```

<img width="700" height="455" alt="image" src="https://github.com/user-attachments/assets/d3457ba5-1c6a-4788-8274-17cc517fbd15" />

O histograma tem distribuição assimétrica com cauda longa à direita, indicando que a maior parte dos preços está fortemente concentrada nas faixas mais baixas. Existem alguns poucos valores muito altos, mas eles são raros e por isso, quase não aparecem visualmente no gráfico, apesar de estarem incluídos.

***Ver o comportamento dos dados ao longo do tempo***

Abaixo é possível ver a distribuição das avaliações dos quartos ao longo do tempo:

<img width="587" height="446" alt="image" src="https://github.com/user-attachments/assets/7f94c1cb-c9bf-4359-bba1-fee76f344110" />

***3. Resumir informações em um dashboard***

Abaixo o dashboard desenvolvido para o projeto:

<img width="884" height="498" alt="image" src="https://github.com/user-attachments/assets/b26e04be-e837-4b4e-8303-4aaf6581b39a" />





