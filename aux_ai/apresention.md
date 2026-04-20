# 🏦 Nubank Analytics Engineer Case
### Dividir e conquistar!

---

## 📋 Índice

- [Enunciado do Problema](#-enunciado-do-problema)
- [Principais Entregáveis](#-principais-entregáveis)
- [Solução Proposta — Modelagem de Dados](#-solução-proposta--modelagem-de-dados)
  - [financial_transactions](#1-financial_transactions)
  - [accounts](#2-accounts)
  - [customers](#3-customers)
  - [customers_address](#4-customers_address)
  - [d_time](#5-d_time)
  - [holidays](#6-holidays)
  - [actions_type](#7-actions_type)
- [Evolução do Modelo](#-evolução-do-modelo)
- [Diagrama Final](#-diagrama-final)
- [Trade-offs e Decisões](#-trade-offs-e-decisões)

---

## 📌 Enunciado do Problema

O desafio propõe uma solução para **dois públicos distintos**: negócios e técnico. Os principais pontos de atenção são:

- A solução deve ser **o mais autoexplicativa possível**
- Novos produtos **às vezes não estão relacionados a transações entre pessoas** (peer-to-peer)
- Alguns produtos podem estar **disponíveis apenas em determinados países**

---

## 🎯 Principais Entregáveis

### Proposta de modificação do modelo de dados

> A proposta deve conter:
> - **Representação visual** das mudanças propostas
> - **Análise de trade-offs** das mudanças propostas ou do que foi mantido igual

### Matrizes de origem

| Matriz | Descrição |
|--------|-----------|
| `transfer_ins` | Transferências não-PIX **recebidas** por uma conta (dinheiro entrando) |
| `transfer_outs` | Transferências não-PIX **realizadas** a partir de uma conta (dinheiro saindo) |
| `pix_movements` | Transferências **recebidas ou enviadas** por uma conta utilizando o PIX |

---

## 🛠 Solução Proposta — Modelagem de Dados

### O Problema das Tabelas Originais

**Part 1 — transfer_ins e transfer_outs:**
As tabelas `transfer_ins` e `transfer_outs` são **redundantes**: só é possível saber se trata de entradas ou saídas pelo nome das tabelas. Não há como identificar o tipo de transferência ou a direção do dinheiro de forma estruturada.

**Part 2 — pix_movements:**
A tabela `pix_movements` tem praticamente a mesma estrutura das tabelas anteriores, com uma melhoria: a coluna **`in_or_out`**, que torna possível identificar se a transação é uma entrada ou saída de dinheiro.

> ✅ **Solução:** Consolidar as três entidades em uma única tabela chamada **`financial_transactions`**, incorporando a coluna `in_or_out` da `pix_movements` para identificar a direção do dinheiro.

---

### 1. `financial_transactions`

Tabela central do modelo (fato). Registra cada evento financeiro realizado em uma conta.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | uuid | Identificador único da transação |
| `account_id` | uuid (FK) | Conta envolvida na transação |
| `action_type_id` | uuid (FK) | Tipo de ação/produto utilizado |
| `action_timestamp_tra` | datetime | Data e hora em que a ação foi registrada |
| `amount_tra` | float | Valor monetário da transação |
| `in_or_out_tra` | enum(in, out) | Indica se o valor **entrou** ou **saiu** da conta |
| `requested_at_tra` | int (FK → d_time) | Data de solicitação da transação |
| `canceled_at_tra` | int (FK → d_time) | Data de cancelamento (se houver) |
| `unavailable_at_tra` | int (FK → d_time) | Data de indisponibilidade (se houver) |
| `completed_at_tra` | int (FK → d_time) | Data de conclusão da transação |
| `status` | varchar(128) | Status atual da transação |

> 💡 As colunas `canceled_at_tra` e `unavailable_at_tra` foram adicionadas para suportar o caso em que o tipo de ação não está disponível no país do cliente — a transação é então cancelada e registrada como indisponível.

---

### 2. `accounts`

Armazena as contas bancárias cadastradas no sistema. Cada conta pertence a um cliente e pode estar ativa, inativa ou bloqueada.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `account_id` | uuid (PK) | Identificador único da conta |
| `customer_id` | uuid (FK) | Cliente titular da conta |
| `created_at_acc` | timestamp | Data e hora de criação da conta |
| `status` | enum(active, inactive, blocked) | Estado atual da conta |
| `account_branch_acc` | varchar(128) | Número da agência bancária |
| `account_check_digit_acc` | varchar(128) | Dígito verificador da conta |
| `account_number_acc` | varchar(128) | Número da conta bancária |

---

### 3. `customers`

Dados cadastrais dos clientes.

**Decisões de design:**
- A coluna `cpf` foi **renomeada para `identification_document`** e o tipo alterado para `varchar`, evitando problemas de integridade com zero à esquerda
- As colunas `country_name` e `customer_city` foram **removidas**, pois não se enquadram no contexto de uma tabela de identificação direta do cliente — essas informações agora vivem em `customers_address`

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `customer_id` | uuid (PK) | Identificador único do cliente |
| `address_id` | uuid (FK) | Endereço associado ao cliente |
| `first_name_cus` | varchar(128) | Primeiro nome |
| `last_name_cus` | varchar(128) | Sobrenome |
| `identification_document_cus` | varchar(128) | Documento de identificação (CPF, RG etc.) |

---

### 4. `customers_address`

Criada para substituir as três matrizes originais de localização (`city`, `state`, `country`), que traziam complexidade desnecessária ao modelo. Esta tabela registra as mesmas informações de forma **mais completa e enriquecedora**.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `address_id` | uuid (PK) | Identificador único do endereço |
| `country_adr` | varchar(128) | País |
| `zip_code_adr` | varchar(128) | CEP / Código Postal |
| `state_adr` | varchar(128) | Estado / Província |
| `city_adr` | varchar(128) | Cidade |
| `neighborhood_adr` | varchar(128) | Bairro |

---

### 5. `d_time`

Tabela de **dimensão temporal**, totalmente remodelada com inspiração em uma tabela de calendário do Power BI.

A ideia é ter granularidades que vão desde o dia exato da transação até a semana do mês, permitindo análises em **vários cenários diferentes** sem precisar aplicar funções de data nas queries.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `time_id` | int (PK) | Chave surrogate da data |
| `year_tim` | int | Ano |
| `day_tim` | int | Dia do mês |
| `month_tim` | int | Mês (1–12) |
| `quarter_tim` | int | Trimestre (1–4) |
| `week_year_tim` | int | Semana do ano |
| `week_day_tim` | int | Dia da semana |
| `week_month_tim` | int | Semana dentro do mês |
| `holiday_id` | int (FK) | Feriado associado à data, se houver |

---

### 6. `holidays`

Tabela auxiliar que registra se a data da transação ocorreu em um feriado. Armazena o país e o nome do feriado, além de uma breve descrição (`about_hol`) para que os usuários **não precisem pesquisar externamente** o que é cada feriado.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `holiday_id` | int (PK) | Identificador único do feriado |
| `country_hol` | varchar(128) | País onde o feriado é válido |
| `name_hol` | varchar(128) | Nome do feriado |
| `about_hol` | varchar(255) | Descrição do feriado |

---

### 7. `actions_type`

Criada para registrar **todos os tipos de produtos e transações** disponíveis na instituição bancária.

As colunas `country_off` e `city_off` registram em quais países e cidades determinado produto está **indisponível** — o que viabiliza o cancelamento automático registrado em `financial_transactions`.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `action_id` | uuid (PK) | Identificador único do tipo de ação |
| `type_act` | varchar(128) | Categoria (ex.: depósito, saque, transferência, PIX) |
| `name_act` | varchar(128) | Nome descritivo da ação |
| `country_off_act` | varchar(128) | País onde o produto está indisponível |
| `city_off_act` | varchar(128) | Cidade onde o produto está indisponível |

---

## 📈 Evolução do Modelo

### Versão 0.1 — Consolidação inicial

Primeira versão após a consolidação das três tabelas originais (`transfer_ins`, `transfer_outs`, `pix_movements`) em `financial_transactions`.

![alt text](/img/image-111.png)

### Versão Final — Modelo completo

Adição da tabela `actions_type` e enriquecimento de `financial_transactions` com:
- `action_type_id` → identifica qual produto/tipo de transação foi utilizado
- `unavailable_at_tra` e `canceled_at_tra` → suportam o fluxo de cancelamento por indisponibilidade geográfica

---

## 🗺 Diagrama Final


![alt text](/img/image.png)


---

## ⚖️ Trade-offs e Decisões

| Decisão | O que foi feito | Trade-off |
|---------|----------------|-----------|
| **Consolidação das 3 tabelas** | `transfer_ins` + `transfer_outs` + `pix_movements` → `financial_transactions` | Perde a separação semântica por nome de tabela, mas ganha em consistência e manutenibilidade |
| **Coluna `in_or_out`** | Adicionada da `pix_movements` para toda a tabela de fatos | Torna a direção do dinheiro explícita e estruturada, não dependendo mais do nome da tabela |
| **`cpf` → `identification_document`** | Renomeada e tipada como varchar | Evita perda de zeros à esquerda; suporta outros documentos de identificação internacionais |
| **Remoção de `country_name` e `customer_city` de customers** | Movidas para `customers_address` | Separação clara de responsabilidades: `customers` = quem é, `customers_address` = onde está |
| **Substituição de city/state/country por `customers_address`** | Três tabelas colapsadas em uma | Reduz JOINs desnecessários e enriquece o endereço com zip_code e neighborhood |
| **Dimensão temporal `d_time`** | Remodelada como tabela calendário com alta granularidade | Facilita análises de BI sem funções de data; custo de storage é marginal |
| **Tabela `holidays`** | Nova tabela ligada ao `d_time` | Permite identificar feriados sem enriquecer a tabela de fatos; campo `about_hol` documenta o feriado inline |
| **Tabela `actions_type`** | Nova tabela com colunas de indisponibilidade geográfica | Centraliza o catálogo de produtos e permite controle de disponibilidade sem lógica no código |

---

## 🏷️ Padrão de Nomenclatura (Rastreabilidade)

Para facilitar a descoberta de dados (Data Discovery) e a construção de queries, adotei um padrão de **sufixos de 3 letras** no final das colunas inspirado no CRM que utilizo no meu trabalho.

Esse sufixo indica exatamente a qual tabela de origem aquele campo pertence. Assim, mesmo em consultas complexas com vários `JOIN`s, o analista saberá de onde o dado veio.

| Sufixo | Tabela de Origem | Exemplo de Coluna |
|--------|-----------------|-------------------|
| `_tra` | `financial_transactions` | `amount_tra` |
| `_acc` | `accounts` | `created_at_acc` |
| `_cus` | `customers` | `first_name_cus` |
| `_adr` | `customers_address` | `city_adr` |
| `_tim` | `d_time` | `year_tim` |
| `_hol` | `holidays` | `name_hol` |
| `_act` | `actions_type` | `type_act` |