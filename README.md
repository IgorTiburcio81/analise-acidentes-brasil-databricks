# Projeto Datatran

**Análise de Acidentes de Trânsito no Brasil**  
Arquitetura Medallion + Modelagem Star Schema + PySpark + Delta Lake

**Autor:** Igor Maciel Tiburcio  
**Período de Desenvolvimento:** dezembro de 2025 - jnaeiro de 2026 
**Dados:** 2007-2024 (PRF/DATATRAN)

---

## Visão Geral do Projeto

O Projeto Datatran consiste em uma análise abrangente de acidentes de trânsito no Brasil utilizando dados históricos fornecidos pela Polícia Rodoviária Federal através do sistema DATATRAN. O projeto implementa uma arquitetura moderna de dados baseada no padrão Medallion combinada com modelagem dimensional Star Schema, processamento distribuído via PySpark e armazenamento em Delta Lake.

O objetivo central é estruturar e transformar dados brutos de acidentes de trânsito cobrindo o período de 2007 a 2024 em informações analíticas de alta qualidade, permitindo análises profundas sobre padrões de acidentes, taxas de letalidade e tendências temporais. A solução foi desenvolvida para lidar com desafios reais de integração de dados, incluindo estruturas de tabelas inconsistentes ao longo dos anos, identificadores duplicados e registros múltiplos para o mesmo acidente.

## Arquitetura de Dados

A arquitetura segue o padrão Medallion com quatro camadas distintas: Raw, Bronze, Silver e Gold. Cada camada possui responsabilidades específicas no processo de transformação dos dados, garantindo qualidade incremental e rastreabilidade completa do pipeline.

### Camada Raw

A camada inicial consiste na ingestão de arquivos CSV disponibilizados pela PRF diretamente no Databricks. Estes dados são convertidos em tabelas Delta Lake, garantindo propriedades ACID e versionamento. Cada ano possui sua própria tabela nesta camada, totalizando dezoito tabelas individuais (datatran_2007 até datatran_2024) que apresentam schemas heterogêneos devido a mudanças na estrutura de coleta ao longo do tempo.

### Camada Bronze - Padronização e Unificação

A camada Bronze representa o primeiro estágio de transformação dos dados, implementando um processo de unificação que consolida todas as tabelas anuais em uma única tabela. O desafio principal desta camada foi lidar com a heterogeneidade das estruturas de dados, resolvido através da padronização de todas as colunas como tipo string e utilização do método unionByName com o parâmetro allowMissingColumns, permitindo que colunas presentes em alguns anos mas ausentes em outros sejam tratadas adequadamente.

**Notebook:** `raw_to_bronze.ipynb`  
**Saída:** `workspace.projeto_datatran.datatran_bronze`

### Camada Silver - Limpeza e Tipagem

A camada Silver aplica transformações de limpeza e padronização sobre os dados brutos consolidados. Nesta etapa são realizadas conversões de tipos de dados apropriadas utilizando funções to_date para campos de data, try_to_timestamp para campos de horário e try_cast para campos numéricos, garantindo robustez contra valores inválidos. Esta camada também seleciona os atributos mais relevantes para análise e remove inconsistências identificadas durante o processo de exploração dos dados.

**Notebook:** `bronze_to_silver.sql`  
**Saída:** `workspace.projeto_datatran.datatran_silver`

### Camada Gold - Modelagem Dimensional

A camada Gold implementa uma modelagem dimensional Star Schema otimizada para análises de business intelligence. Esta camada separa os dados em tabelas de dimensões e fatos, proporcionando melhor desempenho em consultas analíticas e facilitando a construção de dashboards.

**Notebook:** `silver_to_gold.sql`

#### Tabelas de Dimensão

**gold_dim_time_sk** - Dimensão temporal contendo ano, mês, dia, trimestre e dia da semana, utilizando time_sk como chave substituta para otimização de relacionamentos.

**gold_dim_location_sk** - Dimensão geográfica consolidando UF, município, BR, quilômetro, tipo de pista e uso do solo, com lógica de deduplicação implementada para garantir unicidade por combinação de localização.

**gold_dim_conditions_sk** - Dimensão de condições ambientais e da via, armazenando condição meteorológica, fase do dia e traçado da via como combinações distintas.

#### Tabela de Fatos

**gold_fact_accident** - Cada registro representa um acidente único com suas métricas agregadas, incluindo total de veículos envolvidos, total de vítimas, número de mortos, feridos totais e ilesos. A tabela estabelece relacionamentos com todas as dimensões através de chaves substitutas, permitindo análises multidimensionais eficientes.

## Views Analíticas

O projeto inclui views especializadas que facilitam análises específicas e recorrentes sobre os dados modelados.

**vw_brasil_ano_ranking_letalidade** - Apresenta uma visão consolidada por ano dos acidentes em todo o Brasil, calculando total de acidentes, total de vítimas, total de mortos, número de acidentes fatais e taxa de letalidade.

**vw_municipios_ano_ranking_letalidade** - Oferece análise granular focada nos vinte municípios com maior número de acidentes, calculando métricas de letalidade por ano e incluindo ranking de letalidade particionado por ano para identificar municípios mais críticos.

## Stack Tecnológica

**Plataforma:** Databricks Free Edition  
**Processamento:** PySpark e Spark SQL  
**Armazenamento:** Delta Lake  
**Visualização:** Metabase (containerizado via Docker)  
**Arquitetura:** Medallion (Raw → Bronze → Silver → Gold)  
**Modelagem:** Star Schema

A escolha desta stack permite processamento distribuído de grandes volumes de dados, garantindo propriedades ACID através do Delta Lake e facilitando análises através da modelagem dimensional. O Metabase foi selecionado como ferramenta de visualização por ser uma solução open-source robusta que se conecta facilmente ao Databricks.

## Desafios Técnicos e Soluções Implementadas

O projeto enfrentou diversos desafios inerentes ao trabalho com dados do mundo real coletados ao longo de quase duas décadas. A inconsistência de nomenclatura de colunas entre diferentes anos foi resolvida através da padronização inicial como tipo string na camada Bronze e posterior tipagem apropriada na camada Silver. Identificadores duplicados e múltiplos registros para o mesmo acidente foram tratados através de agregações e deduplicações implementadas nas transformações da camada Gold.

A escala dos dados, abrangendo dezoito anos de registros de acidentes em todo o território nacional, exigiu uma arquitetura que pudesse processar eficientemente milhões de registros. A escolha do PySpark e Delta Lake garantiu que as transformações pudessem ser executadas de forma distribuída e performática, enquanto a modelagem Star Schema otimizou o desempenho das consultas analíticas.

## Resultados e Aplicações

O projeto resulta em uma base de dados estruturada e otimizada para análises sobre acidentes de trânsito no Brasil, permitindo identificar padrões temporais de acidentes, analisar taxas de letalidade por região e período, comparar o desempenho de diferentes municípios em termos de segurança viária e avaliar a influência de condições ambientais nos acidentes.

A modelagem dimensional implementada facilita a criação de relatórios dinâmicos e dashboards interativos através do Metabase, tornando as informações acessíveis para análises exploratórias, desenvolvimento de modelos preditivos e suporte à tomada de decisões em políticas públicas de trânsito e segurança viária.

## Requisitos para Execução

Para executar este projeto, é necessário ter um ambiente Spark configurado, seja através do Databricks Community Edition, instalação standalone ou cluster próprio. O Delta Lake deve estar habilitado no ambiente Spark. É necessário ter acesso às tabelas raw do DATATRAN ou aos arquivos CSV originais disponibilizados pela PRF. O ambiente deve ter Python versão 3.10 ou superior instalado.

## Instalação e Execução

Clone o repositório utilizando o comando git clone seguido da URL do repositório, navegue até o diretório do projeto e execute os notebooks na seguinte ordem: primeiro o notebook raw_to_bronze.ipynb para unificação dos dados brutos, em seguida bronze_to_silver.sql para limpeza e tipagem, e finalmente silver_to_gold.sql para criação da modelagem dimensional.

```bash
git clone https://github.com/igorTiburcio81/projeto-datatran.git
cd projeto-datatran
```

Execute os notebooks sequencialmente: raw_to_bronze.ipynb, bronze_to_silver.sql e silver_to_gold.sql.

## Dataset no Kaggle

O dataset processado nas camadas Silver e Gold está disponível para download no Kaggle, facilitando reprodução de análises e desenvolvimento de novos estudos sobre acidentes de trânsito no Brasil.

*Link será adicionado em breve*

## Próximos Passos

O desenvolvimento futuro do projeto contempla a implementação de uma fase de análise exploratória detalhada, incluindo estudo estatístico aprofundado, visualizações especializadas, detecção de outliers e identificação de padrões complexos nos dados. Posteriormente será desenvolvido um dashboard analítico completo utilizando ferramentas como Power BI ou recursos nativos do Databricks, consolidando as principais métricas e insights em uma interface interativa e acessível.

## Contribuições

Contribuições são bem-vindas e podem ser enviadas através de pull requests. Sugestões de melhorias na modelagem, otimizações no processo de limpeza de dados ou propostas de novas análises serão avaliadas e incorporadas quando apropriado.

## Contato

Para discussões sobre dados, arquitetura de dados, engenharia de dados ou análises relacionadas ao projeto, conecte-se através do LinkedIn.

**LinkedIn:** [Igor Maciel Tiburcio](https://www.linkedin.com/in/igor-maciel-tiburcio)

---

**Licença:** Este projeto é disponibilizado para fins educacionais e de portfólio.  
**Fonte dos Dados:** Polícia Rodoviária Federal (PRF) - Sistema DATATRAN
