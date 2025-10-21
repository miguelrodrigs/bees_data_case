BEES Data Engineering: Pipeline de Cervejarias
1. Visão Geral e Escolhas da Stack
O objetivo deste projeto foi construir uma pipeline de dados ponta a ponta, consumindo a Open Brewery DB API e aplicando o modelo de Arquitetura Medallion (Bronze → Silver → Gold).


A meta final é entregar uma visão analítica limpa e agregada para consumo (BI).

1.1 Escolhas Técnicas

Linguagem: Optei por Python e PySpark. Python para o consumo da API (requests) e PySpark para as transformações (Silver e Gold), garantindo que a solução é escalável, mesmo que o volume de dados seja pequeno agora.


Containerização: O projeto usa Docker  para empacotar o ambiente (PySpark,jupyte), garantindo que qualquer pessoa possa rodá-lo com um único comando.

2. Fluxo da Pipeline (Medallion)
A pipeline é dividida em três estágios, onde o dado é progressivamente refinado:

Bronze Layer (Raw Data)

Ação: Coleta de dados paginada da API e persistência no formato original (JSON).

Silver Layer (Curated Data)
Ação: Leitura do JSON, transformação e limpeza dos dados.


Saída: Parquet (formato colunar para leitura eficiente).

Regras de Transformação (Explicação exigida):


Particionamento: Os dados são salvos particionados pela localização (country e state_province). Isso melhora a performance de leitura em consultas de BI.

Normalização: Apliquei lower() e trim() em campos chave (brewery_type, country, state_province) para garantir a consistência dos dados e evitar duplicação no particionamento e agregação.

Tratamento de Nulos: Implementei regras para tratar valores nulos ou strings vazias em campos essenciais, substituindo-os por um valor padrão como 'unknown' para evitar falhas no PySpark.

Gold Layer (Analytical View)
Ação: Geração da visão final para consumo de BI.


Transformação: Agregação da quantidade de cervejarias (COUNT) por tipo e localização. Esta é a única lógica de negócio aplicada nesta camada.


3. Instruções de Execução
Pré-requisitos: Docker e Docker Compose instalados.

Clonar Repositório:

Bash

git clone https://github.com/miguelrodrigs/bees_data_case
Construir e Iniciar o Ambiente:


Construir e Iniciar o Ambiente (Jupyter):
Bash
docker-compose up --build -d
Isso inicia o ambiente Jupyter Lab na porta 8888


##MONITORAMENTO E ALERTAS##

O monitoramento e alertas devem cobrir a saúde operacional (sucesso/falha, tempo de execução, retries), a qualidade dos dados (checagem de volume, valores nulos, esquema e frescor em cada camada), e problemas de infraestrutura/API (uso de recursos, latência da API de origem). O orquestrador (ex: Airflow) deve disparar alertas imediatos em caso de falha ou atraso (SLA), e as falhas de qualidade críticas devem interromper o pipeline para evitar dados ruins. Todas as métricas devem ser visualizadas em um dashboard para visibilidade completa.