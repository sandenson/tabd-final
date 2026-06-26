# Cloud Analytics com Olist

<div style='text-align: justify'>
O olist é a maior loja de departamentos dos marketplaces. Possui um catálogo com mais de 950 mil produtos, centenas de milhares de pedidos e uma rede de mais de 9 mil lojistas parceiros espalhados por todas as regiões do Brasil. Entendemos que a área de dados e inteligência é uma das principais alavancas de crescimento do negócio, por isso buscamos profissionais apaixonados por dados para integrar a nossa equipe de Business Science e Analytics (BSA). <br><br>

**Objetivo**: Desenvolver uma arquitetura moderna de dados na nuvem baseada nos dados públicos da Olist. O desafio envolve modelar um Data Warehouse em um banco PostgreSQL na nuvem (**Supabase**), construir o pipeline de integração local/nuvem utilizando **Apache Hop**, e entregar as análises de negócio em uma plataforma SaaS de BI (**[Preset.io](https://preset.io)**)  **Apache Superset**.

**Dataset:** <https://github.com/olist/work-at-olist-data><br>
**Grupo:** 4 pessoas

- Ilky André Ramos de Lima
- Luiz Gabriel Correia Sandes
- Helder Vinícius Soares Lira do Nascimento
- Vitoria Regina dos Santos Melo

## Pré-requisitos e Infraestrutura Cloud

Em vez de provisionar toda a infraestrutura localmente via contêineres, este projeto exige a orquestração de serviços em nuvem (utilizando os planos gratuitos - *Free Tiers* - de cada plataforma).

**Desafio de Setup:**

1. **Supabase (Data Warehouse):** Criar uma conta e um novo projeto. Obter as credenciais de conexão do banco de dados PostgreSQL (Host, Porta, User, Password).
1. **Preset.io (Visualização):** Criar um *Workspace* gratuito (Starter Plan) para hospedar o Apache Superset em nuvem.
1. **Apache Hop (ETL):** Instalação local (ou via contêiner isolado) para executar a orquestração e a movimentação dos dados.

## Etapa 1: Modelagem Dimensional

Os dados brutos fornecidos no repositório da Olist estão em um modelo relacional transacional. A missão é desenhar um Star Schema focado em performance analítica e implementá-lo no Supabase.

### 1.1 Modelo Dimensional de Alto Nível

Crie o modelo dimensional de alto nível do Data Warehouse com base nas tabelas relacionais fornecidas. O modelo deve conter:

- Escopo do projeto
- Granularidade da tabela de fatos
- Diagrama inicial do modelo dimensional sem muitos detalhes que mostram as tabelas e relacionamentos

### 1.2 Modelo Dimensional Detalhado

Nesta etapa serão definidos os atributos, os valores de domínio, as fontes, os relacionamentos, as questões de qualidade de dados e as transformações.

<br>&emsp;**A. Documentação Detalhada**

- Cada dimensão e tabela de fatos devem ser documentada em uma planilha
separada
- As informações de suporte necessárias incluem:
  - Nome do atributo / fato;
  - Descrição;
  - Valores de amostra; e
  - Um indicador de tipo de dimensão que muda lentamente para cada atributo de dimensão

&emsp;**B. Diagrama Entidade-Relacionamento (DER):**

<div style='padding-left: 1.25rem'>
    Crie o diagrama final do modelo dimensional que representa visualmente todo o detalhamento feito na documentação. O diagrama deve incluir:
</div>

- Todas as tabelas (fato e dimensões);
- Todos os atributos de cada tabela;
- As chaves primárias (PK) e estrangeiras (SK) que definem os relacionamentos. Cada FK na tabela fato irá conter o valor da SK da sua dimensão correspondente, formando assim o relacionamento do esquema estrela

## Etapa 2: Pipeline de ETL (Local para Nuvem) com Apache Hop

O desafio aqui é ler os arquivos CSV locais, recuperados do repositório indicado e carregá-los no banco em nuvem, lidando com a latência de rede.

- **Pipelines (.hpl):**
  - Configurar a conexão JDBC do PostgreSQL apontando para a URI do Supabase.○
  - **Extração e Tratamento:** Ler os CSVs e realizar o tratamento dos dados.
  - **Surrogate Keys:** Gerar chaves substitutas no Hop antes da carga nas dimensões.
  - **Carga em Nuvem (Load):** Inserir os dados no Supabase. *Atenção especial ao tamanho do lote de inserção (commit size) no componente de Table Output, pois cargas para a nuvem linha a linha podem ser extremamente lentas.*
- **Workflow (.hwf):** Configurar o workflow das dimensões, workflow das tabelas fatos e por fim workflow das com os dois workflows criados.

## Etapa 3: Visualização e Insights no Preset.io

Com o Supabase populado, a etapa final é plugar a ferramenta de BI e responder às perguntas de negócio.

- **Integração:**
  - No Preset, adicionar um novo Database selecionando PostgreSQL.
  - Inserir as credenciais do Supabase. *(Dica: pode ser necessário desativar o SSL ou configurar o modo "Require" dependendo das políticas do Supabase)*.
- **Desenvolvimento do Dashboard:**
  - Importar as tabelas do Star Schema como *Datasets* no Preset.
  - Para construir os dashboard e aplicar filtros com as dimensões será necessário criar as queries para usar como dataset.
  - Construa o painel e utilize as dimensões para aplicar filtros nas visões por gráficos.
- O que o dashboard diz: definam e documentem 5 perguntas de negócio que o dashboard visa responder.

## Etapa 4: Entregas

### 4.1 Organização do Repositório

- **Etapa 1 - Modelagem:** Arquivo README.md (ou documento específico da etapa) dedicado à documentação arquitetural. Inclui os diagramas do Modelo Relacional e Dimensional (visões conceitual e física), além dos links diretos para o mapeamento das planilhas de Fatos e Dimensões.
  - [Documento de Modelagem](docs/modeling.md)
- **Etapa 2 - ETL (Apache Hop):** Pasta contendo os scripts de definição de dados (DDL) e os artefatos de integração do Apache Hop (arquivos .hpl e .hwf). Todos os artefatos desta etapa estão devidamente referenciados e linkados no documento .md principal.
  - **Pipelines (.hpl)**
    - [Fato Vendas](apache-hop/fato_vendas.hpl)
    - [Dimensão Vendedor](apache-hop/dim_vendedor.hpl)
    - [Dimensão Produto](apache-hop/dim_produto.hpl)
    - [Dimensão Pagamento](apache-hop/dim_pagamento.hpl)
    - [Dimensão Tempo](apache-hop/dim_tempo.hpl)
    - [Stage Orders](apache-hop/stage_orders.hpl)
  - **Workflows (.hwf)**
    - [Workflow Fatos](apache-hop/workflow_fatos.hwf)
    - [Workflow Dimensões](apache-hop/workflow_dimensoes.hwf)
    - [Workflow Principal](apache-hop/workflow_principal.hwf)
- **Etapa 3 - BI e Análise de Dados:** Acesso ao painel interativo desenvolvido no Preset.io (credenciais de *Viewer* disponibilizadas) e um relatório consolidado em PDF. O relatório inclui uma análise executiva destacando os principais gargalos logísticos e oportunidades de vendas identificadas na operação. Os links para o ambiente do Preset e para o PDF constam na documentação principal.
  - [Relatório em PDF](docs/report.pdf)

</div>
