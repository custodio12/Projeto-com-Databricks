# Projeto com databricks 🎭
 Projeto para criação de dois dashboards que permitem a visualização de museus brasileiros e eventos que ocorrem nesses museus.
 

O objetivo do projeto foi atender a solicitação de um cliente que gostaria de fomentar iniciativas culturais fornecendo visualização dos museus brasileiros e seus eventos de maneira dinâmica e que essas informações estivessem unificadas. A demanda foi visualizar as instituições separadas por estado e região, e os principais eventos através de uma linha do tempo. 


Para isso, foram extraídos os dados de uma API disponível em <a href="https://antigo.museus.gov.br/museus-do-brasil/">Museus do Brasil</a>. O governo federal criou uma base chamada <a href="http://museus.cultura.gov.br/">Museusbr</a> através da  <a href="https://renim.museus.gov.br/wp-content/uploads/2014/04/Portaria-n%C2%BA-6-de-9-de-janeiro-de-2017-D.O.U.-se%C3%A7%C3%A3o-1-de-10-de-janeiro-de-2017...pdf">Portaria nº 6, de 9 de janeiro de 2017</a> que é um sistema nacional de identificação de museus e uma plataforma para mapeamento colaborativo, gestão e compartilhamento de informações sobre museus brasileiros.


Os dados extraídos em formato .JSON foram armazenados inicialmente em um contâiner dentro do <a href="https://azure.microsoft.com/en-us/products/storage/blobs">Azure Blob Storage</a>, como mostra a figura abaixo:

<img width="960" alt="BlobStorage" src="https://user-images.githubusercontent.com/98350733/219026430-ef8ea359-b00e-451a-801b-db8b5d6aef2b.png">

Após a extração, foi possível identificar que os dados precisavam de um tratamento de limpeza profundo. Como essa base é alimentada por diferentes usuários, alguns padrões não eram seguidos. Para atender a solicitação do cliente, a partir dos arquivos extraídos, observou-se que a melhor maneira seria criar dentro do <a href="https://www.databricks.com/">Databricks</a> dois cadernos para tratamento separado dessas informações. Sendo assim, os dados foram transformados tratando praticamente cada coluna com suas particularidades, como por exemplo tratar o formato de chave valor oriundo do formato JSON para transformar os dados em um dataframe utilizando o <a href="https://spark.apache.org/docs/latest/api/python/">Pyspark</a>. Após tratamento, os dados foram disponibilizados no <a href="https://azure.microsoft.com/pt-br/products/azure-sql/database">Banco de dados SQL do Azure</a> para disponibilidade analítica em formato de dashboard.


Linguagem utilizada | Descrição do Projeto | Ferramentas utilizadas 
---|---|---
<a href="https://www.python.org/">Python</a> e SQL | ETL de dados e criação de Dashboards para a visualização dos museus brasileiros e os principais eventos | <a href="https://azure.microsoft.com/en-us/products/storage/blobs/?&ef_id=CjwKCAiAoL6eBhA3EiwAXDom5uK6OvefZQqZmSeysc74ATyOVgFIZCPlcBrZUXO9aggFS-3y1gSOyhoCcM8QAvD_BwE:G:s&OCID=AIDcmmzmnb0182_SEM_CjwKCAiAoL6eBhA3EiwAXDom5uK6OvefZQqZmSeysc74ATyOVgFIZCPlcBrZUXO9aggFS-3y1gSOyhoCcM8QAvD_BwE:G:s&gclid=CjwKCAiAoL6eBhA3EiwAXDom5uK6OvefZQqZmSeysc74ATyOVgFIZCPlcBrZUXO9aggFS-3y1gSOyhoCcM8QAvD_BwE">Azure Blob Storage</a>, <a href="https://azure.microsoft.com/pt-br/products/databricks">Azure Databrics</a>, <a href="https://azure.microsoft.com/pt-br/products/data-factory/">Azure Data Factory (orquestração)</a>, <a href="https://azure.microsoft.com/pt-br/services/sql-database/campaign/">Azure SQL Server</a> e <a href="https://powerbi.microsoft.com/pt-br/">Power BI</a>.


A Figura abaixo apresenta a arquitetura da solução proposta levando em consideração o levantamento de
requisitos e entendimento do negócio.


![arquitetura](https://user-images.githubusercontent.com/98350733/219031591-e44abc02-f132-4acc-a999-f4690140c7af.png)


Para o tratamento dos dados dentro dos cadernos do databricks, foi necessário criar um mount point que faz essa conexão com o blob storage. Para desenvolver o código, fiz importações de pacotes do pyspark para converter os dados para um dataframe e utilizar as bibliotecas para o tratamento dos dados bruto para dados legíveis para apresentação posterior. Além disso, faço a confguração para que o caderno seja conectado com o banco de dados do Azure SQL Server. E por fim, configuro o unmount para o encerramento entre blob e databricks.


No exemplo abaixo, crio o dataframe fazendo a leitura com pyspark do JSON, utilizando a visualização do nome das colunas:


<img width="959" alt="dataframe" src="https://user-images.githubusercontent.com/98350733/219033539-db4c60fb-257b-4084-a603-dd88fd340952.png">


Com os dados tratados, crio duas tabelas dentro do Azure SQL Server, onde os armazeno:

<img width="960" alt="tabelaMuseus" src="https://user-images.githubusercontent.com/98350733/219036054-6c57581a-3ca4-43c2-ac0c-eb32fc22f8ec.png">

<img width="960" alt="tabelaEventos" src="https://user-images.githubusercontent.com/98350733/219036078-cbaf81de-bf1e-45af-94cd-d46405c24924.png">


Para orquestrar esse processo de extração, tratamento e carregamento dos dados para o banco de dados SQL do Azure, utilizo o Data factory, criando uma pipeline de dados. Utilizo dentro do data factory dois caderno do databricks onde primeiro executo um caderno, e após a conclusão do caderno de museus, o data factory executa o caderno de eventos somente se o caderno de museus executar com sucesso. Abaixo segue a pipeline depurada com sucesso:


<img width="960" alt="datafactory" src="https://user-images.githubusercontent.com/98350733/219038717-cd8ba1a4-14ea-454f-9456-75542a33405a.png">


Com os dados disponíveis, a última etapa foi confeccionar as informações solicitadas, conforme os dois dashboards feitos com o PowerBI. Para possibilitar a modelagem multidimensional, foi utilizado o modelo "Star Schema" entre as tabelas:


<img width="473" alt="StarSchema" src="https://user-images.githubusercontent.com/98350733/219040112-8d650613-6c8d-4304-91af-7d407d915f0b.png">


<img width="682" alt="dashMuseus" src="https://user-images.githubusercontent.com/98350733/219040162-4f9f4a7c-c23d-4017-be12-27d44dd46638.png">

<img width="691" alt="DashEventos" src="https://user-images.githubusercontent.com/98350733/219040217-f5227fd5-6758-4657-8e7b-e9be8696145d.png">


Para concluir, foi entregue a documentação do projeto utilizando a plataforma <a href="https://pt.overleaf.com/">Overleaf</a>, que utiliza linguaguem <a href="https://pt.wikipedia.org/wiki/LaTeX">LaTeX</a>.
