# Documentação Técnica - Dashboard de Operações e Performance de Frete B2B

![Power Query](https://img.shields.io/badge/Power_Query-M_Language-F2C811?logo=powerbi&logoColor=black)
![Google Drive](https://img.shields.io/badge/Google_Drive-Reposit%C3%B3rio_CSV-4285F4?logo=googledrive&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-Dashboard-yellow?logo=powerbi)
![DAX](https://img.shields.io/badge/DAX-M%C3%A9tricas-217346?logo=powerbi&logoColor=white)

> **Projeto de Portfólio:** desenvolvido com base em um contexto real de consultoria freelance. Nome da empresa, valores e informações sensíveis foram **fictícios/adaptados** para garantir a confidencialidade contratual das informações.

---

## Sumário

1. [Visão Geral](#1-visão-geral)
2. [O Desafio de Negócio](#2-o-desafio-de-negócio)
3. [Arquitetura da Solução & Fluxo dos Dados](#3-arquitetura-da-solução--fluxo-dos-dados)
4. [Stack Tecnológica](#4-stack-tecnológica)
5. [Engenharia de Dados em Power Query](#5-engenharia-de-dados-em-power-query)
6. [Modelagem de Dados (Resumida)](#6-modelagem-de-dados-resumida)
7. [Recursos e Funcionalidades do Dashboard](#7-recursos-e-funcionalidades-do-dashboard)
8. [Obstáculos Técnicos e Soluções](#8-obstáculos-técnicos-e-soluções)
9. [Galeria do Projeto](#9-galeria-do-projeto)
10. [Como Executar](#10-como-executar)
11. [Impacto e Resultados](#11-impacto-e-resultados)

---

## 1. Visão Geral

Painel construído em regime de consultoria freelance para uma plataforma de entregas expressas B2B (modelo *last-mile*), que conecta entregadores autônomos — cadastrados e remunerados por corrida, sem vínculo empregatício — a clientes corporativos com demanda recorrente de coleta e entrega, atendendo hoje múltiplos estados a partir de Salvador (BA).

Sem infraestrutura interna de TI nem banco de dados próprio, os dados operacionais eram gerados por um sistema proprietário de terceiros e exportados periodicamente em cerca de **8 arquivos CSV distintos**, sem chave de relacionamento nativa entre eles. O projeto unificou essas fontes fragmentadas em um modelo relacional único, entregue em um layout de **página única (one-page)**, exigido pela diretoria para leitura consolidada de SLA, performance de entregadores e resultado financeiro da operação.

## 2. O Desafio de Negócio

A empresa não possuía área de tecnologia nem banco de dados intermediário: toda a base vinha de exportações manuais do sistema legado do parceiro, em múltiplos arquivos CSV sem relação direta entre si. Antes do projeto, o acompanhamento de prazos, repasses e rentabilidade era feito de forma manual e fragmentada, dificultando a visão consolidada da operação em um cenário de expansão para novos estados.

**Desafios identificados na operação:**

- **Dados fragmentados em CSV** — cerca de 8 arquivos brutos exportados periodicamente do sistema legado, sem chave de relacionamento nativa entre eles.
- **Sem banco de dados intermediário** — necessidade de unificar múltiplas fontes diretamente na camada de transformação, sem uma etapa de staging em banco.
- **Exigência de layout único** — a diretoria solicitava uma visão consolidada em página única (*single canvas*), aumentando a complexidade de organização visual de um volume grande de indicadores.

## 3. Arquitetura da Solução & Fluxo dos Dados

```text
Sistema Legado (parceiro terceirizado)
        │
        ▼ (Exportação periódica de 8+ arquivos CSV)
Google Drive (repositório centralizado dos CSVs)
        │
        ▼ (Merge de chaves, pivoteamento, limpeza de endereços — 100% em linguagem M)
Power Query (M)
        │
        ▼ (Unificação em modelo relacional)
Modelo Relacional — Star Schema
        │
        ▼ (Medidas DAX)
Dashboard One-Page (Power BI)
```

Como não havia banco de dados intermediário nem infraestrutura de servidor, o Google Drive assume o papel de repositório centralizado dos arquivos CSV exportados do sistema legado. Toda a unificação dessas mais de 8 fontes — relacionamento por chaves, pivoteamento de colunas e limpeza de dados de endereço — é feita diretamente na camada de transformação do Power BI, em linguagem M, sem passar por uma etapa de staging em banco.

## 4. Stack Tecnológica

| Camada | Tecnologia |
|---|---|
| Origem dos dados | Sistema legado do parceiro terceirizado (exportação periódica em CSV) |
| Repositório dos arquivos | Google Drive (conector nativo do Power Query) |
| ETL / unificação das fontes | Power Query (linguagem M) |
| Modelagem de dados | Star Schema (Power BI Desktop) |
| Métricas e indicadores | DAX |
| Visualização | Power BI — layout one-page |

## 5. Engenharia de Dados em Power Query

Com mais de 8 arquivos CSV como única fonte de dados — e sem qualquer camada de banco intermediário — praticamente toda a complexidade de engenharia do projeto está concentrada na linguagem M:

1. **Ingestão via conector Google Drive** — leitura direta dos arquivos CSV armazenados no repositório, sem necessidade de download manual.
2. **Relacionamento entre tabelas sem chave nativa** — construção de chaves de merge entre as diferentes exportações, que não possuíam relacionamento direto entre si no sistema de origem.
3. **Pivoteamento e limpeza** — reestruturação de colunas (pivot/unpivot) e padronização/limpeza de campos de endereço vindos de formatos inconsistentes.
4. **Padronização de timestamps** — tratamento de fusos horários e cálculo de durações exatas entre os marcos da ordem de serviço (solicitação, chegada, saída e finalização), essenciais para as métricas de SLA.
5. **Otimização de performance** — organização das etapas de transformação para evitar estourar o limite de memória na atualização, já que toda a unificação de 8+ tabelas ocorre em tempo de carga, sem staging em banco.

## 6. Modelagem de Dados (Resumida)

Modelo relacional em **Star Schema**, unificando as dimensões operacionais com a tabela fato de Ordens de Serviço:

| Tabela | Tipo | Descrição |
|---|---|---|
| `fato_ordens_servico` | Fato | Uma linha por ordem de serviço (OS): entregador, cliente, horários, distância, status, valores |
| `dim_entregadores` | Dimensão | Cadastro dos entregadores autônomos alocados nas rotas |
| `dim_clientes` | Dimensão | Clientes corporativos (parceiros comerciais) atendidos |
| `dim_regioes` | Dimensão | Estados e municípios cobertos pela operação |
| `dim_faixas_km_tempo` | Dimensão | Faixas de distância e tempo de entrega, usadas para curvas de distribuição |

## 7. Recursos e Funcionalidades do Dashboard

- **Macrométricas financeiras** — valor total movimentado, repasse a entregadores e faturamento bruto consolidados.
- **Gestão de SLA por cliente** — percentual de entregas no prazo vs. fora do prazo e tempo médio de entrega por parceiro comercial.
- **Auditoria por OS** — rastreabilidade individual de cada ordem de serviço: entregador, horários, distância e status.
- **Faixas de tempo e distância** — curvas de distribuição das entregas por faixa de tempo e quilometragem.
- **Ranking de entregadores** — tempos médios de alocação, coleta e entrega, pontualidade e valor de repasse por profissional.
- **Cobertura territorial** — mapa interativo com distribuição dos pontos de entrega por estado e município.

## 8. Obstáculos Técnicos e Soluções

- **Engenharia intensa em M** — relacionar mais de 8 tabelas CSV sem banco intermediário, otimizando as etapas de transformação para não estourar o limite de memória na atualização do modelo.
- **Padronização de timestamps** — tratamento de fusos horários e cálculo de durações exatas entre os marcos de cada ordem de serviço (solicitação, chegada, saída, finalização), fundamentais para as métricas de SLA.
- **Canvas único** — organização de um grande volume de informação técnica e gerencial em uma única página longa, sem poluição visual, atendendo à exigência da diretoria por uma visão one-page.

## 9. Como Executar

Como este projeto não depende de banco de dados nem de scripts em Python, a reprodução do fluxo se concentra na configuração do repositório de arquivos e na engenharia de transformação em Power Query.

### Pré-requisitos

- Conta Google com acesso ao Google Drive
- Power BI Desktop

### Passos

1. Centralize os arquivos CSV exportados do sistema de origem em uma pasta do Google Drive.
2. No Power BI Desktop, conecte-se à pasta via **Obter Dados → Google Drive**.
3. Em Power Query, construa as chaves de relacionamento entre as 8+ tabelas, aplique pivoteamento/limpeza de endereços e padronize os timestamps (fuso horário e cálculo de durações).
4. Monte o modelo relacional em Star Schema (dimensões de Entregadores, Clientes, Regiões e Faixas de KM/Tempo relacionadas à fato de Ordens de Serviço).
5. Construa as medidas DAX de SLA, repasse e faturamento.
6. Publique o relatório em um único canvas (one-page), organizando os componentes por densidade de informação.

## 10. Impacto e Resultados

- **Visão de SLA por cliente** — identificação de gargalos de prazo por cliente e por região.
- **Otimização de rotas** — base analítica para otimizar rotas por faixa de quilometragem.
- **Auditoria de rentabilidade** — rentabilidade real da operação auditável por região e parceiro comercial.

## 11. Galeria do Projeto

![Preview do Dashboard](assets/screenshots/credito_a.jpeg)

🔗 **[Acessar demonstração interativa no Portfólio](https://jefersonbitencourt-svg.github.io/portifolio-data-analyst/#projeto/performance-operacoes-frete)**
