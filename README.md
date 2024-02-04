# Estudo-de-Caso-Covid
Estudo de caso utilizando como base os boletins informativos do coronavírus por município por dia utilizando as ferramentas do Google Cloud Plataform (GCP).

Dados foram incluídos de forma manualmente dentro da plataforma de BigQuery.

**ETAPA 1**

Criação do Cloud Storage: 

https://console.cloud.google.com/storage/browser/upload_covid_data2;tab=objects?forceOnBucketsSortingFiltering=true&project=dazzling-tiger-413212&prefix=&forceOnObjectsSortingFiltering=false


![image](https://github.com/RaissaYrina/Estudo-de-Caso-Covid/assets/127452513/3ecb8594-b58d-4864-8d05-5ab21e1f32e3)

Criação da BigQuery
Arquivos bases foram imputados manualmente.

https://console.cloud.google.com/bigquery?project=dazzling-tiger-413212&ws=!1m4!1m3!3m2!1sdazzling-tiger-413212!2scasos_covid19
![image](https://github.com/RaissaYrina/Estudo-de-Caso-Covid/assets/127452513/f771f65e-4b08-4318-9a49-481f1a9cb4e8)


**Etapa 2**

1.	Qual o total de casos confirmados?
Query:
```SQL
SELECT SUM(confirmed) AS total_ultima_data
FROM `dazzling-tiger-413212.casos_covid19.caso`
WHERE date = (SELECT MAX(date) FROM `dazzling-tiger-413212.casos_covid19.caso`);
```
Resultado: 
29.849.740

2.	Qual a quantidade de casos confirmados por Estado, classificando os 5 primeiros estados com mais casos
Query:
```SQL
SELECT state, SUM(confirmed) AS casos
FROM  `dazzling-tiger-413212.casos_covid19.caso`
WHERE date = (SELECT MAX(date) FROM `dazzling-tiger-413212.casos_covid19.caso`)
GROUP BY state
ORDER BY casos DESC
LIMIT 5;
```

Resultado
| Linha | state |   casos   |
| ----- | ----- | --------- | 
|   1	  |   SP	| 5.232.374 |
|   2	  |   MG	| 3.317.401 |
|   3	  |   PR	| 2.407.960 |
|   4	  |   RS	| 2.263.880 |
|   5	  |   RJ	| 2.078.817 |


3.	Qual a Letalidade em % (mortes/casos confirmados) por estado classificando os 5 primeiros estados com maior letalidade.

Query
```SQL
SELECT state, 
ROUND((SUM (Deaths) / NULLIF(SUM (confirmed),0)) * 100,3) as letalidade
FROM `dazzling-tiger-413212.casos_covid19.caso`
WHERE date = (SELECT MAX(date) FROM `dazzling-tiger-413212.casos_covid19.caso`)
GROUP BY state
ORDER BY letalidade DESC
LIMIT 5;
```

Resultado
|  state	| letalidade |
| ------- | ---------- | 
|   RJ	  |    3.497   |
|   SP	  |    3.194   |
|   MA	  |    2.562   |
|   AM	  |    2.435   |
|   PA    |	   2.406   |

4.	Qual a Taxa de Óbito por cada mil habitantes, por estado, listar os 5 primeiros estados com maior concentração de óbitos a cada mil habitantes (população)?

Query:
```SQL
SELECT state, 
ROUND((SUM (Deaths) / NULLIF(SUM (estimated_population),0)) * 1000,3) as taxa_obito_por_mil_habitantes
FROM `dazzling-tiger-413212.casos_covid19.caso`
GROUP BY state
ORDER BY taxa_obito_por_mil_habitantes DESC
LIMIT 5;
```
Resultado:

Linha	state	
|  state	| taxa_obito_por_mil_habitantes |
| ------- | ----------------------------- | 
|	   MT	  |              2.364            |
|    AM	  |              2.047            |   
|  	 RJ	  |              1.958            |
|	   RO	  |              1.871            |
| 	 DF   |	             1.864            |

5.	Qual a porcentagem de municípios que registraram óbito em relação ao total de municípios da amostra?
Query:
```SQL
SELECT 
    ROUND(COUNT(DISTINCT CASE WHEN deaths > 0 THEN city END) / NULLIF(COUNT(DISTINCT city), 0) * 100,2) AS porcentagem_municipios_com_obito
 FROM `dazzling-tiger-413212.casos_covid19.caso`;
```

Resultado:
99,62%


6.	Qual a população total por estado, o município mais populoso de cada estado e a representatividade de concentração populacional em porcentagem deste município em relação ao total de habitantes do estado?
Query:
```SQL
SELECT 
    populacao_por_estado.estado,
    populacao_total,
    municipio_mais_populoso,
    populacao_municipio_mais_populoso,
    ROUND((populacao_municipio_mais_populoso / NULLIF(populacao_total, 0)) * 100, 2) AS representatividade_percentual
FROM (
    SELECT 
        estado,
        SUM(populacao) AS populacao_total
    FROM 
        `dazzling-tiger-413212.casos_covid19.cidades_estado`
    GROUP BY 
        estado
) AS populacao_por_estado
JOIN (
    SELECT 
        estado,
        municipio AS municipio_mais_populoso,
        populacao AS populacao_municipio_mais_populoso,
        ROW_NUMBER() OVER (PARTITION BY estado ORDER BY populacao DESC) AS ranking
    FROM 
        `dazzling-tiger-413212.casos_covid19.cidades_estado`
) AS municipio_mais_populoso_por_estado
ON populacao_por_estado.estado = municipio_mais_populoso_por_estado.estado
WHERE ranking = 1;
```
Resultado:

|  estado	|   populacao_total	| municipio_mais_populoso	| populacao_municipio_mais_populoso| representatividade_percentual|
| ------- | ----------------- | ----------------------- | -------------------------------- | ---------------------------- |
|    AP   |  	    733508	    |           Macapá	      |               442.933	           |               60.39          |
|    PI	  |       3269200	    |          Teresina	      |               866.300	           |               26.5           |
|    MT	  |       3658813     |	          Cuiabá	      |               650.912	           |               17.79          |
|    TO	  |       1511459     |          	Palmas	      |               302.692	           |               20.03          |
|    SC	  |       7609601	    |          Joinville	    |               616.323            |               	8.1           |
|    RO	  |       1581016	    |          Porto Velho	  |               460.413	           |               29.12          |
|    DF	  |       2817068	    |           Brasília	    |              2.817.068           |             	 100.0          |
|    MS	  |       2756700	    |         Campo Grande	  |               897.938	           |               32.57          |
|    PA	  |       8116132	    |             Belém	      |              1.303.389	         |               16.06          |
|    SP	  |       44420459	  |          São Paulo	    |              11.451.245	         |               25.78          |
|    MG	  |       20538718	  |        Belo Horizonte	  |               2.315.560	         |               11.27          |
|    RS	  |       10880506	  |        Porto Alegre     |   	          1.332.570          |               12.25          |
|    MA	  |       6775152	    |          São Luís	      |               1.037.775	         |               15.32          |
|    BA	  |       14136417	  |           Salvador      |               2.418.005	         |               17.1           |
|    AM	  |       3941175	    |            Manaus	      |               2.063.547	         |               52.36          |
|    AC	  |       830026	    |          Rio Branco     |                364.756	         |               43.95          |
|    RR	  |       636303	    |           Boa Vista	    |                413.486	         |               64.98          |
|    RJ   |   	  16054524	  |       Rio de Janeiro    |               6.211.423	         |               38.69          |
|    CE   |       8791688	    |           Fortaleza     |              	2.428.678	         |               27.62          |
|    ES	  |       3833486	    |            Serra        |                520.649	         |               13.58          |
|    PE	  |       9058155	    |           Recife	      |               1.488.920	         |               16.44          |
|    RN	  |       3302406	    |            Natal	      |                751.300           |         	     22.75          |


**ETAPA 3**
Criação de um Dashboard utilizando as bases de dados anteriores.

*Link*: https://lookerstudio.google.com/reporting/85984641-a85a-47d9-b6d2-04795b80291a/page/k1LpD

![image](https://github.com/RaissaYrina/Estudo-de-Caso-Covid/assets/127452513/dddd1efe-c815-429e-bf8c-a71ca45c249f)


*Dentro do dashboard, foi incorporada a análise de dois gráficos: um que exibe a distribuição da quantidade de casos confirmados por estado e outro que ilustra a letalidade por estado, calculada como o número de mortes dividido pelo número de casos confirmados. A inclusão desses gráficos visa destacar que a região com o maior número de casos confirmados nem sempre corresponde àquela com o maior número de mortes. Isso ajuda a proporcionar uma visão mais abrangente da situação epidemiológica, permitindo a identificação de disparidades entre a incidência de casos e a gravidade da doença em diferentes localidades.
Além disso, essa abordagem oferece uma análise mais aprofundada e cuidadosa, especialmente para regiões com um número menor de casos, mas que podem apresentar uma alta taxa de mortalidade devido à falta de infraestrutura hospitalar adequada, condições de higiene precárias, acesso limitado a cuidados de saúde ou outros fatores socioeconômicos. Isso destaca a importância de não apenas considerar o volume de casos confirmados, mas também avaliar a qualidade e a capacidade do sistema de saúde local para lidar com a doença, fornecendo insights valiosos para direcionar recursos e intervenções de saúde de forma mais eficaz e equitativa.*

![image](https://github.com/RaissaYrina/Estudo-de-Caso-Covid/assets/127452513/2c43a7d1-0b6d-4cbe-9aae-daf504654c37)
