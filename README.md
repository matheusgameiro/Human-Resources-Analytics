# Human-Resources-Analytics

## Human Resources Analytics
17 de janeiro de 2022

## Visão Geral

A documentação do projeto de data science a seguir descreve um problema de negócio onde a empresa deseja entender os motivos que influenciam na saída de seus funcionários e se existe, através dos dados, uma forma de prever (se antecipar) a saída do colaborador.

## Questões levantadas (dores do negócio):

- Quais fatores influenciam para um colaborador deixar a empresa?
- Como reter pessoas?
- Podemos nos antecipar e saber se um determinado colaborador vai sair da empresa?
- Como diminuir o turnover?
Obs: Este projeto foi realizado a partir do curso “Human Resource Analytics” ministrado pela Stack Tecnologias.

## Objetivo
Explicar para os stakeholders através dos dados os principais motivos que levam os funcionários a deixarem a empresa, e por fim disponibilizar um recurso que permita que a empresa consiga realizar a predição para verificar se um colaborador vai ou não deixar a empresa com base nas mais importantes variáveis que impactam nessa decisão, sendo elas: carga de trabalho, nível de satisfação com a empresa e resultados de performance, número de projetos envolvidos, anos de empresa e histórico de acidentes de trabalho.
Portanto, o projeto tem a finalidade de simular um problema real de Data Science, entender as expectativas e as necessidades do cliente, responder através de dados as questões levantadas (dores do negócio), testar hipóteses e colocar o modelo de Machine Learning produzido disponível de forma interativa e intuitiva para o cliente.

## Arquitetura da Solução Proposta

Para resolver esse problema de negócio, pensei em uma solução completa para armazenamento, tratamento, gestão e automatização de fluxos dos dados utilizando tecnologias como: Docker, Apache Airflow, Minio, MYSQL e Python com as seguintes bibliotecas: Seaborn, PyCaret, Scikit learn, Pandas, Streamlit, Matplotlib.

Depois da infraestrutura devidamente criada e configurada, foram criados e modelados atributos relevantes para análise utilizando fontes de dados diversas como arquivos em formato xlsx, json e dados do Banco de Dados MySQL. O projeto pode ser dividido entre as seguintes etapas:

- Configuração do Ambiente
- Estruturação/Modelagem  dos Dados
- Desenvolvimento do Modelo de Machine Learning;
- Disponibilização da Solução;

## Configuração do Ambiente

Foi utilizado o Docker a fim de virtualizar algumas aplicações utilizando o conceito de containers, o nosso banco de dados MySQL, nosso data lake Minio e nosso orquestrador de fluxos de trabalho Airflow, estão representados cada um respectivamente em um container.



*containers ativos
Estruturação/Modelagem Dos Dados
MYSQL - Banco de Dados:
Depois da infraestrutura devidamente criada e configurada, vamos alimentar nosso banco de dados MySQL com a base em formato SQL fornecido pela empresa (employees):

Além disso, também temos outras fontes de dados em diferentes formatos, como .CSV e .JSON (que foram incluídos em nosso data lake manualmente). Esses dados passarão por processos de limpeza/modelagem utilizando a linguagem Python através do orquestrador de tarefas Airflow, arquivos depois de modelados, serão enviados novamente ao nosso Data Lake MinIO, porém dessa vez em um zona (bucket) diferente.





MinIO - Data Lake:

Decidi dividir nosso data lake em três zonas diferentes (buckets):


Landing: Zona de arquivos brutos, pré-processamento
Processing: Zona de arquivos já processados, ou seja, que já passaram pelas DAGs (scripts) do    Airflow e já estão no formato desejado.
Curated: Zona de arquivos já processados e verificados e serve como zona de backup.

Airflow:

Com o objetivo de orquestrar todos as execuções a serem realizadas, utilizamos o Airflow. Para o nosso projeto, foi construído as seguintes DAGS (scripts):

Cada DAG tem o objetivo de tratar/modelar as fontes de dados a fim de levantar um atributo que será incluído no nosso modelo de Machine Learning, após a execução de cada DAG, a nossa zona Processing do Data Lake é atualizada com um arquivo de dados. A DAG etl_employees_dataset foi responsável por realizar a construção da tabela final. Cada tabela foi salva em .parquet com o objetivo de consumir menos memória e garantir a integridade das colunas.
Abaixo podemos ver a execução de cada DAG no Airflow:


Desenvolvimento do Modelo de Machine Learning:

Aqui, buscamos entender o nosso problema de forma mais direcionada a predição de dados e, a partir disso, modelar dados que são representativos para a construção do nosso modelo. Para isso, levamos em consideração o Turnover (binária) como a variável resposta do nosso modelo, a fim de antecipar possíveis saídas de funcionários, permitindo que a empresa possa tomar iniciativas prévias para reter o funcionário.
Ao decorrer da análise exploratória dos dados, obtivemos os seguintes insights:
Turnover

A empresa possui uma rotatividade de aproximadamente 24%


Nível de Satisfação do Funcionário

Empregados com o nível de satisfação em 20 ou menos tendem a deixar a empresa.
Empregados com o nível de satisfação em até 50 tem maior probabilidade de deixar a empresa.
Existe uma razão para o pico de empregados insatisfeitos?
Salário


A maioria dos empregados que saíram tinha salário baixo ou médio.
Quase nenhum empregado com alto salário deixou a empresa.
Como é o ambiente de trabalho? Isso se difere por salário?
O que faz empregados com alto salário sairem da empresa.
Nota de avaliação do funcionário
Temos uma distribuição bimodal para o conjunto que deixou a empresa.
Colaboradores com baixa performance tendem a deixar a empresa.
Colaboradores com alta performance tendem a deixar a empresa.
O ponto ideal para os funcionários que permaneceram está dentro da avaliação de 60 à 80.



Feature Selection
Esta etapa terá como objetivo selecionar a variáveis que de fato são necessárias para realizar o treinamento do nosso modelo, evitando assim, “overfitting” Para isso, utilizaremos o método DecisionTreeClassifier da biblioteca scikit-learn
Random Forest;
Quando executamos a Random Forest, ele nos diz quais atributos são mais relevantes para incluir no modelo:


	


Com os nossos dados coletados, armazenados, modelados, partimos para a etapa de construção do nosso algoritmo de predição de Turnover. Para otimizar a escolha do melhor algoritmo de machine learning ao nosso problema de negócio, usei a ferramenta PyCaret, que nos indicou que o algoritmo com melhor AUC (métrica de desempenho escolhida) é o modelo Gradient Boosting Classifier, o “gbc”, logo criaremos o modelo e o treinamos.








Disponibilização da Solução - Deploy:

Como forma de disponibilizar a solução rapidamente, foi utilizado o Streamlit. Ele permite construir um WebApp e realizar seu deploy com apenas um clique. Sendo assim, podemos visualizá-lo abaixo tanto as variáveis que serão utilizadas na etapa da predição, quanto a localização do funcionário dentro do cluster construído anteriormente.


Conclusão
Através desse projeto foi possível praticar e implementar conceitos importantes na Ciência e Engenharia de Dados e propor uma solução para um problema latente e recorrente de qualquer empresa que é a retenção de talentos através da Análise de Dados de Recursos Humanos.
Como um processo de melhoria contínua podemos desenvolver uma automação para executar não só o pipeline de coleta e transformação de dados como automatizar os passos da etapa de Machine Learning e Deploy.
