# MVP de Engenharia de Dados: Análise de Vendas para Gestão de Restaurante

## Detalhes do Projeto

* **Aluno(a):** Ursula Machado Weinstein
* **Matrícula:** 4052025000257
* **Instituição:** PUC-Rio - Pontifícia Universidade Católica do Rio de Janeiro

---

## 🎯 1. Introdução e Objetivo

Este projeto demonstra a proficiência em Engenharia de Dados, focando na construção de uma arquitetura de dados (`Data Warehouse`) que permita a **análise de negócios** para a gestão de um restaurante. O objetivo final foi entregar uma base de dados confiável (tabela `GOLD`) e responder a questões estratégicas sobre Mix de Vendas, Sazonalidade e Perfil do Cliente.

O projeto foi desenhado para responder a **8 perguntas de negócio** fundamentais:

1.  **Top Produtos:** Quais são os 10 produtos mais vendidos (em quantidade)?
2.  **Mix de Vendas:** Dentre os diversos tipos, quais Grupos geram mais receita?
3.  **Estoque Morto:** Quais são os 10 produtos menos vendidos?
4.  **Performance de PDV:** Qual o ranking de faturamento por Ponto de Venda?
5.  **Sazonalidade:** Qual é o faturamento por hora do dia (identificação de picos)?
6.  **Ticket Médio:** Qual é o valor médio gasto por nota fiscal?
7.  **Clientes VIP:** Quais são os 10 sócios que mais consumiram?
8.  **Fluxo de Caixa:** Qual a divisão entre vendas "A Faturar" vs. "Pago na Hora"?

* **Período Analisado:** 01/01/2025 a 13/11/2025.

---

## 🔄 2. Linhagem e Origem dos Dados (Data Lineage)

### 2.1. Origem dos Dados
A fonte de dados para o projeto consiste em arquivos brutos de transações de vendas extraídos do sistema de **Ponto de Venda (PDV)** da empresa onde atuo profissionalmente.

> **Nota de Consentimento:** Este projeto utiliza dados corporativos reais, cedidos pela empresa mediante **autorização expressa da Presidência**. O nome da organização foi preservado por motivos de privacidade. O dataset não contém informações sensíveis de clientes e foi utilizado exclusivamente para fins de análise técnica e demonstração acadêmica.

### 2.2. Pipeline de Transformação (Arquitetura)
A linhagem dos dados segue a arquitetura **Medallion (Bronze, Silver, Gold)** executada no ambiente **Databricks** utilizando **Spark SQL**:

1.  **Fonte (Source):** Arquivos brutos extraídos do ERP.
2.  **Camada Bronze (Raw):** Ingestão dos dados no Data Lake sem tratamento.
3.  **Camada Silver (Trusted):** Limpeza de dados, tipagem forte (casting), remoção de nulos e padronização.
4.  **Camada Gold (Refined):** Criação da tabela desnormalizada `default.gold_vendas_flat_model`, otimizada para consultas analíticas (OLAP).

---

## 📚 3. Dicionário de Dados

Especificação técnica da tabela analítica `gold_vendas_flat_model`.

| Coluna | Tipo | Descrição do Domínio | Intervalo / Valores Esperados |
| :--- | :---: | :--- | :--- |
| `DATA_HORA` | `TIMESTAMP` | Momento da transação | **Min:** 2025-01-01 / **Max:** 2025-11-13 |
| `FATURAMENTO_LIQUIDO` | `DOUBLE` | Valor líquido **do item** (R$) | **Min:** 0.00 (ver nota 1) / **Max:** [INSIRA SEU VALOR MÁXIMO] |
| `QUANTIDADE` | `INT` | Unidades vendidas | **Min:** 1 / **Max:** 600* (ver nota 2) |
| `NOME_PRODUTO` | `STRING` | Item do cardápio | *Ex: CAFE EXPRESSO, HEINEKEN, BINGO* |
| `NOME_GRUPO` | `STRING` | Categoria macro | *Ex: BUFFET, BEBIDAS, CERVEJAS* |
| `TIPO_CONSUMO` | `STRING` | Forma de pagamento | `SOCIO_A_FATURAR`, `AVULSO_PAGO_NA_HORA` |
| `NOME_PDV` | `STRING` | Local da venda | *Ex: RESTAURANTE PRINCIPAL, QUIOSQUE* |
| `ID_SOCIO` | `STRING` | Código do cliente | Números ou `null` (anônimo) |
| `NUM_NFCE` | `STRING` | Chave da Nota Fiscal | Identificador único |

> **Nota 1 (Min):** O valor R$ 0.00 refere-se a itens promocionais, cortesias ou componentes de combos. A Nota Fiscal consolidada sempre possui valor total > 0.
>
> **Nota 2 (Max):** Valores extremos na coluna `QUANTIDADE` (ex: > 100) referem-se a pacotes de festas ou eventos lançados em nota única, e não a erros de sistema.

---

## 📈 4. Resultados da Análise de Negócios

A análise foi conduzida através do **Notebook 4**, gerando os seguintes *insights* estratégicos:

### 4.1. Mix de Vendas e Estoque
* **Top Produtos:** A lista é dominada por itens de necessidade e conveniência (Café, Água), que possuem alto giro operacional mas baixo ticket unitário.
* **Grupos Fortes:** O faturamento é concentrado no grupo **BUFFET E EVENTOS** (líder isolado) e nas **Bebidas** (Alcoólicas + Não Alcoólicas), que somadas representam a segunda maior fonte de receita.
* **Estoque Morto:** Todos os 10 produtos menos vendidos registraram apenas **uma única unidade vendida** em quase 11 meses, indicando a necessidade de revisão do cardápio.

### 4.2. Sazonalidade e Operação
* **Picos de Horário:** O gráfico revela um perfil de consumo vespertino/lazer. O pico máximo de faturamento ocorre às **16:00h**, seguido das 15:00h, com queda acentuada no período noturno (após 20h).
* **Concentração de PDV:** A receita é altamente dependente do **[NOME DO PDV LÍDER]**, enquanto os demais pontos atuam apenas como satélites de apoio.

### 4.3. Perfil Financeiro e Cliente
* **Ticket Médio:** O valor médio por transação é de **R$ 121,49**.
* **Clientes VIP:** Os Top 10 Sócios possuem um volume de gastos acumulado significativamente superior à média, indicando alta fidelidade e recorrência.
* **Fluxo de Caixa:** A operação é fortemente baseada em vendas com **cobrança posterior**. A categoria `SOCIO_A_FATURAR` responde por **90.4%** do faturamento líquido, enquanto o pagamento à vista (`AVULSO_PAGO_NA_HORA`) representa apenas 9.6%.

---

## 💡 5. Conclusão Geral

Este MVP validou a capacidade de transformar dados transacionais complexos e "sujos" em informações estratégicas claras. A construção da tabela `GOLD` e a validação da qualidade dos dados (Data Profiling) permitiram à gestão identificar o **perfil de consumo vespertino** e a **dependência crítica do fluxo de caixa na liquidação de contas de sócios**, fornecendo insumos diretos para a tomada de decisão financeira e operacional.
