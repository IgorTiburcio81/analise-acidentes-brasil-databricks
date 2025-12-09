# Projeto_datatran
Análise de acidentes de trânsito no Brasil.
🚧 Projeto de Análise de Acidentes de Trânsito no Brasil
📊 Arquitetura Medallion + Modelagem Star Schema + PySpark + Delta Lake

Este projeto tem como objetivo construir uma arquitetura de dados completa para analisar acidentes de trânsito no Brasil, utilizando dados públicos do DATATRAN (PRF).
A solução envolve desde o tratamento inicial de dados brutos (raw) até a criação de uma camada gold modelada em Star Schema, passando por conversão, padronização e enriquecimento das informações.

📁 Arquitetura Geral do Projeto

A arquitetura segue o padrão Medallion:

RAW → BRONZE → SILVER → GOLD

E na camada GOLD utilizamos uma arquitetura dimensional (Star Schema) para otimizar análises.

🛠️ Tecnologias Utilizadas

PySpark / Spark

Delta Lake

Databricks / SparkSQL

Arquitetura Medallion

Modelagem Dimensional (Star Schema)

📂 Estrutura das Camadas
1️⃣ Raw (Fonte)

Arquivos originais do DATATRAN, separados por ano (datatran_2007 … datatran_2024)

Dados heterogêneos: schemas diferentes entre anos

2️⃣ Bronze – Padronização e Unificação

Notebook: raw_to_bronze.ipynb

Principais passos:

Leitura de todas as tabelas anuais

Conversão de todos os campos para string

União com unionByName(allowMissingColumns=True)

Persistência em Delta Lake

Saída:
📌 workspace.projeto_datatran.datatran_unificada

3️⃣ Silver – Limpeza, Tipagem e Tratamento

Notebook: bronze_to_silver.sql

Processos realizados:

Conversão de datas (to_date)

Conversão de horários (try_to_timestamp)

Tipagem numérica segura (try_cast)

Seleção dos atributos mais relevantes

Remoção de inconsistências

Resultado:
📌 workspace.projeto_datatran.datatran_silver

4️⃣ Gold – Modelagem Dimensional (Star Schema)

Notebook: silver_to_gold.sql

A camada GOLD é composta por:

📘 Dimensões

gold_dim_time
Contém granularidade de data, mês, trimestre, dia da semana.

gold_dim_location
Informações sobre UF, município, BR, KM, tipo de pista, etc.

gold_dim_conditions
Condições do acidente como clima, fase do dia, traçado da via.

📕 Fato

gold_fact_victim
Cada registro representa uma vítima envolvida em um acidente.

🔎 Desafio importante:
A criação de IDs consistentes por vítima exigiu o uso de informações como pesid, id_acidente e id_veiculo, resultando em uma estrutura relacional mais confiável.

📌 Objetivo do Projeto

Construir uma base analítica escalável e confiável para:

✔️ Entender padrões de acidentes
✔️ Mapear locais críticos e faixas horárias
✔️ Analisar perfis de vítimas
✔️ Facilitar a construção de dashboards e modelos preditivos

📊 Próximos Passos

🚀 Fase 2 – Análise Exploratória (EDA)

Estudo estatístico e visual

Detecção de outliers

Identificação de padrões

🚀 Fase 3 – Dashboard Analítico
Ferramentas previstas:

Power BI

Looker Studio

Databricks SQL Visualization

🏷️ Requisitos para Execução

Spark configurado (Databricks, standalone ou cluster)

Delta Lake habilitado

Acesso às tabelas RAW (ou aos arquivos CSV DATATRAN)

Python 3.10+

📦 Instalação e Execução

Clone o repositório:

git clone https://github.com/seu-user/projeto-datatran.git
cd projeto-datatran


Execute os notebooks na ordem:

1. raw_to_bronze.ipynb
2. bronze_to_silver.sql
3. silver_to_gold.sql

🤝 Contribuições

Contribuições são bem-vindas!
Sugestões de modelagem, limpeza ou novas análises podem ser enviadas via pull request.

📬 Contato

Se quiser conversar sobre dados, arquitetura, engenharia ou análises, me encontre no LinkedIn:
👉 [Seu Nome Aqui]



