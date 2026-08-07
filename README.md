# Documentação Técnica - Dashboard de Alocação e Performance Logística B2B

![Google Sheets](https://img.shields.io/badge/Google_Sheets-Camada_de_Ingest%C3%A3o-34A853?logo=googlesheets&logoColor=white)
![Google Apps Script](https://img.shields.io/badge/Apps_Script-Automa%C3%A7%C3%A3o-4285F4?logo=googleappsscript&logoColor=white)
![Power Query](https://img.shields.io/badge/Power_Query-M_Language-F2C811?logo=powerbi&logoColor=black)
![Power BI](https://img.shields.io/badge/Power_BI-Dashboard-yellow?logo=powerbi)

> **Projeto de Portfólio:** desenvolvido com base em um contexto real de consultoria freelance. Nome da empresa, do sistema de origem, colunas e métricas sensíveis foram **fictícios/adaptados** para garantir a confidencialidade contratual das informações.

---

## Sumário

1. [Visão Geral](#1-visão-geral)
2. [O Desafio de Negócio](#2-o-desafio-de-negócio)
3. [Arquitetura da Solução & Fluxo dos Dados](#3-arquitetura-da-solução--fluxo-dos-dados)
4. [Stack Tecnológica](#4-stack-tecnológica)
5. [Orquestração e Atualização dos Dados](#5-orquestração-e-atualização-dos-dados)
6. [Dicionário de Dados (Resumido)](#6-dicionário-de-dados-resumido)
7. [Recursos e Funcionalidades do Dashboard](#7-recursos-e-funcionalidades-do-dashboard)
8. [Obstáculos Técnicos e Soluções](#8-obstáculos-técnicos-e-soluções)
9. [Galeria do Projeto](#9-galeria-do-projeto)
10. [Como Executar](#10-como-executar)
11. [Impacto e Resultados](#11-impacto-e-resultados)

---

## 1. Visão Geral

Dashboard desenvolvido em regime de consultoria freelance para uma empresa do setor de logística, que conecta entregadores autônomos (motoristas e motoboys cadastrados, sem vínculo empregatício, remunerados por corrida) a clientes PJ com demanda recorrente de coleta e entrega. O modelo de operação é comparável a um "marketplace de entregadores": cada entregador é alocado pelo time interno da empresa em uma ou duas rotas fixas, atendendo um agrupamento fixo de clientes — e não entregas avulsas para consumidor final (CPF).

A empresa nasceu em Salvador (BA) e expandiu sua operação para aproximadamente 9 estados, mas, por seu porte, ainda não possui uma área de tecnologia própria nem infraestrutura de banco de dados. O projeto entrega visibilidade sobre volume de corridas, alocação de entregadores por cliente/rota e distribuição geográfica da operação, usando exclusivamente ferramentas de baixo custo e sem dependência de infraestrutura de servidor.

## 2. O Desafio de Negócio

Antes do projeto, toda a análise da operação era feita manualmente: a equipe extraía relatórios do sistema de gestão operacional utilizado pela empresa (fornecido por um parceiro terceirizado) e realizava os cruzamentos "na unha", em planilhas Excel com tabelas dinâmicas. Esse processo era demorado, sujeito a erro humano e não escalava à medida que novos estados eram incorporados à operação.

**Desafios identificados na operação:**

- **Ausência de banco de dados e de área de TI** — toda a base vinha de relatórios extraídos manualmente do sistema do parceiro, sem uma camada de dados centralizada.
- **Análise manual e não escalável** — cruzamentos feitos em Excel/tabela dinâmica, sem padronização, o que dificultava enxergar tendências e comparar estados/rotas entre si.
- **Tomada de decisão às cegas** — sem um painel consolidado, decisões sobre alocação de entregadores, abertura de novas rotas e expansão para novos estados eram tomadas sem suporte analítico direto.
- **Sem infraestrutura de servidor** — não havia VM, banco de dados ou ambiente para rodar automações via Python, o que exigiu uma solução inteiramente baseada em ferramentas de nuvem gratuitas/low-code.

## 3. Arquitetura da Solução & Fluxo dos Dados

```text
Sistema de Gestão Operacional (parceiro terceirizado)
        │
        ▼ (Extração manual de relatórios pela equipe)
Google Drive (upload dos relatórios extraídos)
        │
        ▼ (Google Apps Script — unificação automática)
Planilha Mestra (Google Sheets)
        │
        ▼ (Publicação/exportação via link CSV com ID da planilha)
Power BI — Conector Web
        │
        ▼ (ETL 100% em linguagem M — Power Query)
Dashboard de Alocação e Performance Logística
```

Como a empresa não possui banco de dados nem servidor próprio, o Google Sheets assume o papel de camada de staging: cada extração manual do sistema é subida ao Google Drive, e uma automação em Google Apps Script lê essas extrações e as unifica em uma planilha mestra única. Essa planilha é então publicada como link CSV (o método de exportação por ID da planilha), permitindo que o Power BI consuma os dados diretamente via conector Web, sem necessidade de gateway on-premises.

## 4. Stack Tecnológica

| Camada | Tecnologia |
|---|---|
| Extração da base | Sistema de gestão operacional do parceiro (exportação manual de relatórios) |
| Staging / unificação | Google Drive + Google Apps Script |
| Fonte para o BI | Google Sheets, publicado via link CSV (exportação por ID da planilha) |
| ETL / tratamento | Power Query (linguagem M) — 100% da tratativa de dados |
| Modelagem & métricas | Power BI Desktop (DAX) |
| Publicação | Power BI Service, via conector Web |

## 5. Orquestração e Atualização dos Dados

Sem VM ou servidor disponível, a orquestração foi desenhada para depender do mínimo de intervenção manual possível:

1. **Extração manual** — a equipe interna exporta periodicamente os relatórios do sistema de gestão operacional e faz o upload para uma pasta no Google Drive.
2. **Unificação via Google Apps Script** — uma automação lê os arquivos recém-adicionados na pasta e os consolida em uma planilha mestra no Google Sheets, evitando duplicidade de registros.
3. **Publicação como CSV** — a planilha mestra é publicada/exportada via link CSV (o método de geração de link com o ID da planilha), tornando-a acessível como uma fonte de dados web pública.
4. **Atualização no Power BI Service** — como a fonte é um link web público (e não um banco atrás de uma rede corporativa), o Power BI Service consegue agendar a atualização automaticamente via conector Web, sem necessidade de On-Premises Data Gateway — contornando a ausência de infraestrutura de servidor da empresa.
5. **ETL em Power Query** — toda a limpeza, padronização e regras de negócio (classificação de rota, cliente, entregador, estado) são aplicadas em linguagem M, já que não havia viabilidade de uma execução Python agendada sem VM.

## 6. Dicionário de Dados (Resumido)

### Base unificada (Google Sheets — planilha mestra)

| Coluna | Descrição |
|---|---|
| `corrida_id` | Identificador único da corrida/entrega |
| `entregador_id` | Identificador do entregador (motorista/motoboy) alocado |
| `cliente_id` | Identificador do cliente PJ atendido (origem ou destino da coleta) |
| `rota_id` | Identificador da rota fixa associada ao agrupamento de clientes |
| `uf` | Estado (UF) onde a operação ocorreu |
| `data_corrida` | Data de realização da corrida |
| `valor_pago_entregador` | Valor pago ao entregador pela corrida (remuneração por corrida, sem vínculo CLT) |
| `status_corrida` | Status da corrida (realizada, cancelada, etc.) |

### Camada tratada (Power Query — consumida pelo dashboard)

Tabela agregada por dia/UF/rota/entregador, já com classificações de negócio aplicadas (padronização de nomenclatura entre extrações, tratamento de duplicidades e enriquecimento de status).

## 7. Recursos e Funcionalidades do Dashboard

- **Visão geral da operação** — total de corridas realizadas, entregadores ativos e clientes atendidos no período.
- **Distribuição geográfica** — visualização da operação por estado (UF), acompanhando a expansão da empresa para novas regiões.
- **Alocação de entregadores por rota** — comparação entre a demanda média de cada cliente/agrupamento e a quantidade de entregadores alocados nas rotas fixas.
- **Performance por entregador** — volume de corridas e valor pago por entregador, permitindo identificar desequilíbrios de carga entre rotas.
- **Acompanhamento temporal** — evolução do volume de corridas e da expansão de estados atendidos ao longo do tempo.
- **Indicadores financeiros** — valor total pago a entregadores por período, estado e rota.

## 8. Obstáculos Técnicos e Soluções

- **Ausência total de banco de dados** — resolvido usando o Google Sheets como camada de staging, com Google Apps Script cumprindo o papel que normalmente seria de um pipeline de ETL server-side.
- **Sem VM ou servidor para automação em Python** — todo o tratamento de dados precisou ser reescrito em linguagem M, dentro do próprio Power Query, eliminando a dependência de execução manual de scripts.
- **Extrações manuais e não padronizadas** — como os relatórios eram extraídos por diferentes pessoas em momentos distintos, foi necessário construir, em M, regras de padronização de nomenclatura e de deduplicação para evitar contagem dupla de corridas.
- **Atualização automática sem gateway** — resolvido publicando a planilha mestra como link CSV público, permitindo que o Power BI Service agendasse atualizações via conector Web sem depender de infraestrutura on-premises.

## 9. Galeria do Projeto

*(inserir prints do dashboard aqui, ex.: `assets/screenshots/logistica_a.jpeg`)*

## 10. Como Executar

Como este projeto não depende de banco de dados nem de scripts Python, a reprodução do fluxo depende principalmente da configuração da automação no Google Workspace e da conexão do Power BI à fonte web.

### Pré-requisitos

- Conta Google com acesso ao Google Drive e Google Sheets
- Power BI Desktop

### Passos

1. Crie uma pasta no Google Drive para receber as extrações manuais do sistema de origem.
2. Configure um script em Google Apps Script para ler os arquivos da pasta e consolidá-los em uma planilha mestra no Google Sheets.
3. Publique a planilha mestra na web como CSV (Arquivo → Compartilhar → Publicar na web), obtendo o link com o ID da planilha.
4. No Power BI Desktop, conecte-se à fonte via **Obter Dados → Web**, informando o link CSV publicado.
5. Aplique as transformações em Power Query (linguagem M) conforme as regras de negócio (padronização, deduplicação, classificações).
6. Publique o relatório no Power BI Service e configure a atualização agendada via conector Web.

## 11. Impacto e Resultados

- **Fim da análise "no escuro"** — pela primeira vez a empresa passou a acompanhar seus próprios números de forma consolidada, em vez de depender de cruzamentos manuais em Excel.
- **Suporte à expansão** — embora o dashboard não tenha sido a causa direta, a adoção do BI coincidiu com o período em que a empresa expandiu sua operação de Salvador para cerca de 9 estados.
- **Solução viável sem infraestrutura própria** — prova de que é possível estruturar um pipeline analítico funcional e com atualização automatizada mesmo em empresas sem área de tecnologia, banco de dados ou servidor dedicado.
