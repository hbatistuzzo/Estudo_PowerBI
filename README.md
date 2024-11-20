
<div align="center">
  <img src="powerbi.png" alt="powerbi" width="66%>
  <img src="dsa.png" alt="Page 2 Overview" width="33%">
</div>

<div align="center">

</div>

Este repositório estrutura o conteúdo da segunda metade do curso "Business Intelligence e Data Science" da Data Science Academy (~ 70 horas de duração, nível intermediário).  inclui um portfolio de projetos construídos com base na primeira metade do mesmo curso.

### Qual o porquê da separação?
A primeira metade do curso é voltada para a funcionalidade do Power BI: visualizações, linguagem DAX e M, funcionalidades nativas, etc.
**Já a segunda metade, observada neste repositório, possui como foco:**
1. A integração do Power BI com Bancos de dados;
2. O uso de SQL Analytics neste contexto;
3. Aplicação de técnicas de _Machine Learning_ para segmentação e detecção de anomalias;
4. O uso de Inteligência Artificial para análise de séries temporais no Power BI; e
5. Um estudo de caso

A oportunidade é fortuita para o aprendizado de algumas aplicações acessórias como ODBC e SQLite.

Em conjunto, estes exercícios começam a integrar as diferentes ferramentas com as quais possuo familiaridade na área de ciência de dados. 


---
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

## Introduzindo Machine Learning para segmentação de clientes no Power BI

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
Imagine que umaempresa tenha dados históricosde clientes que fizeram compras de produtos ou serviços. Os dados incluem,para cada cliente: idade, renda anual e uma pontuação de gasto(poder de compra do cliente).A empresa gostaria de segmentar esses clientes em 3 grupos de acordo com similaridades a fim de personalizar as campanhas de Marketing.O gestor da área de Marketing espera receber um relatório com os 3 segmentos e para cada segmento a média de idade, renda anual e pontuação de gastos dos clientes.

![Abhinandan Trilokia](https://raw.githubusercontent.com/Trilokia/Trilokia/379277808c61ef204768a61bbc5d25bc7798ccf1/bottom_header.svg)