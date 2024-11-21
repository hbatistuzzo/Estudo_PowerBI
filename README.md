
<div align="center">
  <img src="media/powerbi.png" alt="11" width="70%">
  <img src="media/dsa.png" alt="22" width="20%">
</div>

Este repositório estrutura o conteúdo da segunda metade do curso "Business Intelligence e Data Science" da Data Science Academy (~ 70 horas de duração, nível intermediário).  inclui um portfolio de projetos construídos com base na primeira metade do mesmo curso.

### Qual o porquê da separação?
A primeira metade do curso é voltada para a funcionalidade do Power BI: visualizações, linguagem DAX e M, funcionalidades nativas, etc.
**Já a segunda metade, observada neste repositório, possui como foco:**
1. A integração do Power BI com Bancos de dados;
2. O uso de SQL Analytics neste contexto;
3. Aplicação de técnicas de _Machine Learning_ para segmentação e detecção de anomalias, com o auxílio de linguagens Python e R;
4. O uso de Inteligência Artificial para análise de séries temporais no Power BI; e
5. Um estudo de caso

A oportunidade é fortuita para o aprendizado de algumas aplicações acessórias como ODBC e SQLite.

Em conjunto, estes exercícios começam a integrar as diferentes ferramentas com as quais possuo familiaridade na área de ciência de dados.

<br>

---

<h1 align="center">1. A integração do Power BI com Bancos de Dados</h1>

Embora eu esteja familiarizado com DBMS's como PostgreSQL/MySQL e suas respectivas utilizações em GUI's como pgAdmin4/MySQL Workbench, o projeto atual permite a interação com duas novas ferramentas cuja interação foge do par usual DMBS <--> Interface Gráfica:

- o **SQLite** é um sistema de banco de dados leve e sem servidor.
  - Diferentemente do PostgreSQL/MySQL, o SQLite não exige um servidor dedicado; o banco de dados é apenas um único arquivo no disco.
  - Frequentemente usado em projetos pequenos, sistemas embarcados ou situações onde a simplicidade é mais importante do que a escalabilidade. Comum, por exemplo, em aplicações de smartphones.
  - O SQLite não possui uma ferramenta GUI nativa como o pgAdmin4 ou MySQL Workbench, mas pode ser usado com ferramentas de terceiros como o DB Browser for SQLite ou DBeaver.

- O **ODBC (Open Database Connectivity)** é uma API padronizada para acessar diferentes sistemas de banco de dados.
  - Ele funciona como uma ponte entre aplicações e DBMS's, permitindo que interajam *independentemente* do tipo de banco de dados subjacente.
  - O ODBC **não** é um sistema de gerenciamento de banco de dados, mas um meio de conectar-se a um.
  - Ou seja, é um intermediário que permite que diferentes aplicações se comuniquem com bancos de dados. Ele não é diretamente comparável aos pares mencionados acima, mas complementa o SQLite (ou qualquer banco de dados) ao fornecer uma maneira padronizada de se conectar a ele.

  ---

O ODBC Driver existe na versão Trial, no momento de criação deste projeto, com licença de 30 dias. Devido ao propósito didático do projeto, isto não configura um problema.

<div align="center">
  <img src="media/a1.png" alt="a1">
</div>

<br>
A conexão é, de fato, realiada da maneira mais sucinta possível, seguindo o paradigma enxuto do SQLite. Não há servidor dedicado de hospedagem, a base de dados é armazenada em um único arquivo em disco:

<br>

<div align="center">
  <img src="media/a2.png" alt="a2">
</div>

<br>

Após a conexão com o SQLite e a ingestão dos dados, podemos utilizar instruções SQL para realizar filtros ou gerar novas vistas. Algo como:
```
SELECT Nome_Produto,
        ROUND(MIN(Valor_Venda, 2) AS valor_mínimo),
        ROUND(MAX(Valor_Venda, 2) AS valor_máximo),
        ROUND(AVG(Valor_Venda, 2) AS valor_médio),
        ROUND(SUM(Valor_Venda, 2) AS valor_total),
        COUNT(Valor_Venda) AS contagem,
        Ano
FROM TB_DSA_VENDAS, TB_DSA_PRODUTOS, TB_DSA_PEDIDOS,
WHERE TB_DSA_VENDAS.produto = TB_DSA_PRODUTOS.id_produto
AND TBD_DSA_VENDAS.pedido = TB_DSA_PEDIDOS.id_pedido
GROUP BY Produto, Ano
ORDER BY Contagem DESC
LIMIT 1000
```

Caso contrário, as tabelas são carregadas no ambiente do Power BI e seguimos com o tratamento padrão:

<br>

<div align="center">
  <img src="media/a3.png" alt="a3">
</div>

<br>

Embora o Power BI não seja uma ferramenta dedicada para a modelagem de dados estruturados (como o pgModeler, por exemplo), ele é capaz de realizar a manutenção básica. Veja na figura abaixo que o Power BI reconhece o modelo semântico que estrutura as diferentes tabelas envolvidas, bem como as 3 relações "1-para-muitos" que conectam as chaves primárias (em pedidos, produtos e clientes) à tabela central "vendas".

<br>

<div align="center">
  <img src="media/a4.png" alt="a4">
</div>

<br>

## Preâmbulo sobre Machine Learning

O aprendizado de máquina pode ser dividido em três categorias principais:

1. Aprendizado Supervisionado: no qual o algoritmo é treinado com um conjunto de dados rotulados, ou seja, com entradas e saídas conhecidas. O algoritmo utiliza esses dados  para  aprender  a  mapear  as  entradas  nas  saídas  corretas.  Exemplos comuns incluem classificação de imagens e previsão de preços

2. Aprendizado Não Supervisionado: Aqui, o algoritmo é treinado com um conjunto de dados não rotulados, e seu objetivo é encontrar padrões e estruturas subjacentes nos dados. Exemplos comuns incluem agrupamento e redução de dimensionalidade.

3. Aprendizado Por Reforço: Neste tipo de aprendizado, o algoritmo, chamado de agente, aprende a tomar decisões com base em recompensas e punições. O agente interage com um ambiente e ajusta suas ações para maximizar as recompensas a longo prazo. Exemplos comuns incluem jogos e robótica.

Neste projeto, será realizado um processo de _clusterização_, ou seja, **aprendizado não supervisionado**. Os detalhes são descritos abaixo

---


## Segmentação de clientes com Machine Learning em linguagem Python

A segmentação de clientes é o processo de dividir a base de clientes de uma empresa em grupos  distintos  com  base  em  características  comuns,  necessidades,  comportamentos  ou preferências.  O  objetivo  da  segmentação  é  entender  melhor  as  necessidades  e  desejos  de diferentes  grupos  de  clientes  e,  assim,adaptar  as  estratégias  de  marketing,  comunicação  e vendas para atender a essas necessidades de maneira mais eficaz e personalizada.

A segmentação pode ser feita com base em diversos critérios, como:
1. Demográficos:  Idade,  sexo,  estado  civil,  renda,  ocupação,  nível  de  educação  e tamanho da família;
2. Geográficos:  Localização,  clima,  densidade  populacional  e  fronteiras  políticas  ou culturais;
3. Psicográficos: Estilo de vida, personalidade, valores, atitudes e interesses;
4. Comportamentais:  Padrões  de  compra,  frequência  de  uso,  lealdade  à  marca, preferências e atitudes em relação a produtos e serviços.

A segmentação de clientes pode beneficiar as empresas de várias maneiras, incluindo:
1. Compreender melhor as necessidades e expectativas de diferentes grupos de clientes;
2. Desenvolver campanhas de marketing e comunicação mais eficazes e personalizadas;
3. Identificar oportunidades de mercado e nichos ainda não explorados;
4. Melhorar a satisfação e retenção de clientes, oferecendo produtos e serviços mais adequados às suas necessidades; e
4. Otimizar a alocação de recursos, concentrando-se nos segmentos de clientes mais lucrativos ou com maior potencial de crescimento.

As empresas podem utilizar técnicas de análise de dados e aprendizado de máquina para segmentar sua base de clientes de forma mais precisa e sofisticada, identificando padrões e relações complexas entre diferentes variáveis e comportamentos.

Nesta seção, o aprendizado de máquina será utilizado para segmentar clientes de acordo com determinadas características. O problema é proposto sucintamente da seguinte forma:

Imagine que uma empresa tenha dados históricos de clientes que fizeram compras de produtos ou serviços. Os dados incluem,para cada cliente: idade, renda anual e uma pontuação de gasto(poder de compra do cliente). A empresa gostaria de segmentar esses clientes em 3 grupos de acordo com similaridades a fim de personalizar as campanhas de Marketing. O gestor da área de Marketing espera receber um relatório com os 3 segmentos e para cada segmento a média de idade, renda anual e pontuação de gastos dos clientes.

---

Realizada uma análise exploratória dos dados, inicia-se o aprendizado de máquina. É necessário, antes de mais nada, realizar o pré-processamento dos dados. O algoritmo de aprendizado não-supervisionado KMeans (escolhido nesse caso devido ao problema de negócios, o qual envolve clusterização) espera receber dados padronizados i.e. na mesma escala. Como não iremos avaliar o modelo, não haverá divisão em grupos treino/teste. O processo é sucintamente descrito no código abaixo:

```
# Imports
import pandas as pd
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

df_dsa = pd.read_csv('recursos/dados/dados_clientes.csv') # Carrega os dados

df_dsa[['idade', 'renda_anual', 'pontuacao_gastos']].describe() # Resumo estatístico

padronizador = StandardScaler() # Criando o padronizador dos dados

# Aplica o padronizador somente nas colunas de interesse
dados_padronizados = padronizador.fit_transform(df_dsa[['idade', 'renda_anual', 'pontuacao_gastos']])

k = 3 # Definindo o número de clusters (k)

kmeans = KMeans (n_clusters = k) # o modelo é criado

kmeans.fit(dados_padronizados) # treinamento do modelo com dados padronizados

df_dsa['cluster'] = kmeans.labels_ # atribuímos os rótulos dos clusters aos clientes
```

O número de clusters _k_ é o número de agrupamentos que serão realizados sobre os dados; há uma série de técnicas para estimar o melhor valor de k, o que não é abordado aqui. A aplicação do modelo resulta em uma nova coluna, "cluster", indicando a que grupo pertence determinado indivíduo.

<br>

<div align="center">
  <img src="media/kmeans1.png" alt="a4">
</div>

<br>

Dentro de cada grupo, eu possuo a **maior** similaridade possível.
Entre os grupos, eu possuo a **menor** similaridade possível.

Em linhas gerais: o KMeans compara as distâncias entre cada vetor de dados (as linhas da matriz dados_padronizados), iterativamente, o que permite caracterizá-los.

Esta é uma oportunidade fortuita para demonstrar a funcionalidade de integração de relatórios PowerBI em ambiente Jupyter Notebook, realizada através do pacote _powerbiclient_; este e outros recursos (como a publicação online do relatório) podem ser utilizados através do acesso à Microsoft com uma conta de email institucional. No caso, estou utilizando a minha conta _alumni_ de ex-estudantes da USP. As etapas de construção podem ser verificadas no caderno "projeto_cluster.ipynb" deste repositório. Uma das funcionalidades exibidas é o QuickVisualize, o qual cria _internamente_ no caderno um relatório .pbix; 

```
from powerbiclient import QuickVisualize, get_dataset_config, Report
from powerbiclient.authentication import DeviceCodeLoginAuthentication

device_auth = DeviceCodeLoginAuthentication() # Define a autenticação no Power BI Service

relatorio_PBI = QuickVisualize(get_dataset_config(df_dsa), auth = device_auth) # Cria o relatório no Power BI
```

o PowerBI tenta organizar as informações consideradas mais importantes em uma estrutura de visualizações, geralmente acompanhadas de uma narrativa inteligente. O resultado do código acima é a visualização abaixo:

<div align="center">
  <img src="media/pbixcliente.gif" alt="a1">
</div>

<br>

Este recurso, embora interessante para uma rápida visualização, como o nome diz, não necessariamente gera informações relevantes automaticamente (o que pode vir a mudar no futuro recente com a implementação do _Copilot_ em ferramentas como o PowerBI): repare, por exemplo, que o comportamento automático do PowerBI ao lidar com variáveis numéricas é realizar a soma (note os títulos das visualizações, "sum of renda_anual by idade", "sum of id by idade", etc), o que não faz sentido dependendo do contexto, como no caso da idade. Todas estas informações devem ser verificadas e retrabalhadas quando necessário. O "quick_visualizer", no entanto, ao menos da um pontapé inicial e pode vir a ser útil em uma primeira inspeção.

Neste caso específico, é vantajoso retrabalhar as visualizações e a formatação para melhor atender às perguntas realizadas no início da seção. A grande vantagem do QuickVisualize, neste caso, é que podemos publicar o relatório na nuvem e baixar um arquivo pbix já com todas as informações engatilhadas. Não é necessário começar o trabalho do zero. Realizadas as modificações, o resultado final pode ser visualizado no relatório abaixo:

<div align="center">
  <img src="media/relatorio.png" alt="a1">
</div>

<br>

Agora, é mais fácil extrair as informações mais relevantes de toda esta análise:

1. O cabeçalho exibe as métricas globais;
2. A média de pontuação de gastos por segmento indica que o segmento 1 deve receber uma atenção maior pela equipe de marketing (são aqueles que mais adquirem produtos/serviços desta empresa).
3. O total de clientes por segmento é balanceado, o que é esperado em uma tarefa de machine learning para segmentação.
4. A média de rendimento anual por segmento indica que o grupo 0 é o que possui maior renda; é interessante notar portanto que uma maior renda não se traduz em uma maior pontuação de gastos.
5. As médias de idade para os segmentos 0 e 1 são quase coincidentes: 53 e 54 anos. O segmento 2 é o que mais destoa neste quesito, com média de 27 anos.

Esta análise pode ser devolvida agora ao departamento de marketing, o qual poderá refinar suas campanhas utilizando como alvo os 3 clusters analisados aqui. Cada campanha, espera-se, será customizada para as características de cada grupo.

---


## Detecção de anomalias com Machine Learning em linguagem R

A detecção de anomalias, também conhecida como detecção de outliers, é uma técnica em  Machine  Learning  e Estatística  que  visa  identificar  padrões  incomuns,  inesperados  ou anômalos nos dados. Esses padrões podem ser diferentes das observações normais de várias maneiras,  como  magnitude,  frequência  ou  comportamento.  A  detecção  de  anomalias  é importante porque as anomalias podem indicar problemas, erros, falhas, fraudes ou atividades maliciosas e, em muitos casos, é crucial identificar e analisar esses eventos anômalos para tomar decisões informadas e apropriadas.

Existem várias abordagens para detectar anomalias em Machine Learning, algumas das quais incluem:
1. **Métodos Estatísticos**: baseiam-se na análise estatística dos dados, como testes de hipóteses, distribuições de probabilidade e medidas de dispersão (por exemplo, desvio padrão e intervalos interquartis). Observações que estão significativamente distantes da média ou fora dos intervalos esperados são consideradas anômalas.
2. **Aprendizado Supervisionado**:  nesta  abordagem,  um  modelo  de  Machine  Learning  é treinado usando um conjunto de dados rotulado, que inclui exemplos de observações normais e anômalas. O modelo aprende a distinguir entre as duas classes e, em seguida, pode ser usado para  classificar  novas  observações  como  normais  ou  anômalas.
3. **Aprendizado Não Supervisionado**: neste caso, os algoritmos de Machine Learning são usados para analisar dados não rotulados e identificar padrões ou agrupamentos naturais neles. As anomalias são identificadas como pontos de dados que não se encaixam bem em nenhum desses agrupamentos ou que estão significativamente distantes de outros pontos de dados. Alguns exemplos de algoritmos de aprendizado não supervisionado usados para detecção de anomalias incluem clustering (por exemplo, K-means) e técnicas de redução de dimensionalidade (por exemplo, PCA).
4. **Aprendizado Semi-Supervisionado**: esta abordagem combina elementos de aprendizado supervisionado e não supervisionado. Os algoritmos são treinados em um conjunto de dados parcialmente rotulado, que contém exemplos de observações normais e um pequeno número de  exemplos anômalos. O  modelo aprende a distinguir entre as classes e identificar novas anomalias com base nos padrões aprendidos.
5. **Métodos Baseados em Densidade**: esses métodos identificam anomalias como pontos de dados que estão localizados em áreas de baixa densidade do espaço de recursos (atributos). Um exemplo popular de algoritmo de detecção de anomalias baseado em densidade é o DBSCAN (_Density-Based Spatial Clustering of Applications with Noise_)
6. **Métodos Baseados em Vizinhança**: esses métodos comparam a distância ou similaridade entre pontos de dados e seus vizinhos para identificar anomalias. Os pontos de dados que têm vizinhos significativamente diferentes de si mesmos são considerados anômalos. Exemplos de algoritmos que empregam essa abordagem incluem o k-NN (k-Nearest Neighbors) e o LOF (Local Outlier Factor). 

A escolha do método mais adequado para detecção de anomalias depende do contexto, da natureza dos dados, do conhecimento técnico do profissional de análise e do tipo de problema que desejamos resolver.

---

O problema proposto é descrito a seguir:
- Imagine que uma empresa da área financeira tenha dados históricos de clientes com duas transações financeiras (aqui chamadas de “transacao1” e “transacao2”). Os gestores acreditam que algumas dessas transações possam ser fraudulentas e gostariam de identificar as eventuais anomalias. Os gestores não fazem ideia do que seria uma anomalia e pediram sua ajuda para encontrar uma solução. De fato, eles não sabem se anomalias realmente ocorreram. Usando dados fictícios usaremos Machine Learning para agrupar os dados de transações financeiras dos clientes e então detectar e definir as anomalias (se existirem). O resultado deve ser entregue no formato visual através de gráficos no Power BI.



![Abhinandan Trilokia](https://raw.githubusercontent.com/Trilokia/Trilokia/379277808c61ef204768a61bbc5d25bc7798ccf1/bottom_header.svg)