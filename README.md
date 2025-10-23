# Visão Geral e Detalhamento dos Casos de Uso

## 1. Visão Geral dos Casos de Uso

A tabela a seguir apresenta um resumo dos principais casos de uso (UCs) do sistema, organizados por módulo:

| ID | Módulo | Nome do Caso de Uso | Objetivo Principal |
| :--- | :--- | :--- | :--- |
| **UC1** | Acesso | Realizar Login | Acessar o sistema com segurança. |
| **UC2** | Acesso | Registrar-se | Criar uma nova conta de usuário. |
| **UC3** | Gerenciamento | Gerenciar Categorias | Classificar receitas e despesas. |
| **UC4** | Gerenciamento | Registrar Entrada (Receita) | Inserir novos valores positivos. |
| **UC5** | Gerenciamento | Registrar Saída (Despesa) | Inserir novos valores negativos. |
| **UC6** | Gerenciamento | Visualizar e Editar Transações | Consultar e corrigir o histórico de lançamentos. |
| **UC7** | Análise | Visualizar Saldo Atual | Exibir o valor total disponível. |
| **UC8** | Análise | Consultar Gráfico de Fluxo de Caixa por Período | Analisar tendências de entrada e saída no tempo. |
| **UC9** | Análise | Consultar Distribuição de Despesas por Categoria | Identificar onde o dinheiro está sendo gasto. |

---

## 2. Detalhamento dos Casos de Uso

### Módulo 1: Acesso ao Sistema

#### UC1: Realizar Login

* **Ator Principal:** Usuário
* **Pré-Condição:** O Usuário deve estar cadastrado no sistema.
* **Pós-Condição:** O Usuário está autenticado, e o Sistema gerou um **token de sessão (JWT)**.
* **Requisitos Chave (Backend):** Implementação de **hashing de senha** (Ex: bcrypt) e geração/validação de JSON Web Token (JWT).

#### UC2: Registrar-se

* **Ator Principal:** Usuário
* **Pré-Condição:** O E-mail fornecido não pode estar em uso.
* **Pós-Condição:** O Usuário é criado e persistido no banco de dados.
* **Requisitos Chave (Backend):** Persistência do usuário e garantia de que a senha é armazenada criptografada.

### Módulo 2: Gerenciamento de Dados (CRUD)

#### UC4: Registrar Entrada (Receita)

* **Ator Principal:** Usuário
* **Objetivo:** Inserir um valor de receita no registro financeiro.
* **Fluxo Básico:** O Usuário fornece: Descrição, Valor (**positivo**), Data e uma Categoria associada (Ex: "Salário"). O Sistema registra a transação como tipo **INCOME**.
* **Requisitos Chave (Banco de Dados):** Necessidade de campo `user_id` e campo `type` (`INCOME` / `EXPENSE`) na tabela de Transações.

#### UC5: Registrar Saída (Despesa)

* **Ator Principal:** Usuário
* **Objetivo:** Inserir um valor de despesa no registro financeiro.
* **Fluxo Básico:** O Usuário fornece: Descrição, Valor (**positivo**), Data e uma Categoria associada (Ex: "Alimentação"). O Sistema registra a transação como tipo **EXPENSE**.

#### UC6: Visualizar e Editar Transações

* **Ator Principal:** Usuário
* **Objetivo:** Gerenciar o histórico de transações.
* **Fluxo Básico:** O Sistema permite filtrar a lista de transações por período, tipo ou categoria. O Usuário pode selecionar qualquer transação para **atualizar** seus dados (`PUT`) ou **excluí-la** (`DELETE`).

### Módulo 3: Análise e Visualização (Dashboard)

#### UC7: Visualizar Saldo Atual

* **Ator Principal:** Usuário
* **Objetivo:** Exibir o saldo consolidado do usuário.
* **Cálculo (Backend):** O Sistema deve executar uma **consulta de agregação (SUM)** que calcule a soma total das receitas e subtraia a soma total das despesas (`SUM(INCOMES) - SUM(EXPENSES)`).

#### UC8: Consultar Gráfico de Fluxo de Caixa por Período

* **Ator Principal:** Usuário
* **Objetivo:** Apresentar a evolução mensal (ou semanal) de Receitas e Despesas.
* **Lógica da Consulta (Backend):**
    1.  Filtrar transações pelo período solicitado pelo Usuário.
    2.  Agrupar as transações por **Mês/Ano**.
    3.  Calcular a soma total das Receitas e a soma total das Despesas para cada Mês/Ano.
    4.  Retornar os dados para a renderização do gráfico de linhas/barras.

#### UC9: Consultar Distribuição de Despesas por Categoria

* **Ator Principal:** Usuário
* **Objetivo:** Visualizar o percentual de gastos em cada categoria.
* **Lógica da Consulta (Backend):**
    1.  Filtrar apenas transações do tipo **EXPENSE** no período desejado.
    2.  Agrupar as despesas por `category_id`.
    3.  Calcular a soma dos valores para cada categoria e o total geral de despesas.
    4.  Retornar os dados para a renderização do gráfico de pizza/rosquinha (incluindo o cálculo percentual de cada fatia).

    ---

## 3. Modelagem do Banco de Dados (DER)

Este projeto utiliza um modelo relacional simples e coeso, focado em garantir a integridade e o isolamento dos dados financeiros por usuário.

### 3.1. Diagrama de Entidade-Relacionamento (DER)

O diagrama abaixo ilustra as quatro entidades principais e suas relações no banco de dados, utilizando a sintaxe Mermaid para ER Diagram:

```mermaid
erDiagram
    USERS ||--o{ ACCOUNTS : possui
    USERS ||--o{ CATEGORIES : possui
    ACCOUNTS ||--o{ TRANSACTIONS : tem
    CATEGORIES ||--o{ TRANSACTIONS : classifica

    USERS {
        int id PK
        string email UK "Chave Única"
        string password_hash
        string nome
        datetime created_at
    }

    ACCOUNTS {
        int id PK
        int user_id FK
        string nome "Ex: Conta Corrente, Poupança"
        decimal saldo_inicial
    }

    CATEGORIES {
        int id PK
        int user_id FK
        string nome "Ex: Salário, Aluguel, Alimentação"
        string tipo ENUM "INCOME, EXPENSE"
    }

    TRANSACTIONS {
        int id PK
        int user_id FK
        int account_id FK
        int category_id FK
        string type ENUM "INCOME, EXPENSE"
        decimal valor
        string descricao
        date data
        datetime created_at
    }
```

### 3.2. Descrição das Entidades

| Entidade | Descrição | Chave Principal | Relacionamentos Chave |
| :--- | :--- | :--- | :--- |
| **USERS** | Armazena dados de autenticação e perfil do usuário. | `id` (PK) | Associa Contas, Categorias e Transações. |
| **ACCOUNTS** | Representa as contas financeiras (carteiras) do usuário. | `id` (PK) | Liga-se a `USERS` e `TRANSACTIONS`. |
| **CATEGORIES** | Define os rótulos de classificação das transações (Receita ou Despesa). | `id` (PK) | Liga-se a `USERS` e `TRANSACTIONS`. |
| **TRANSACTIONS** | Núcleo do sistema. Registra todas as Entradas (`INCOME`) e Saídas (`EXPENSE`). | `id` (PK) | Possui 3 Chaves Estrangeiras (`user_id`, `account_id`, `category_id`). |

### 3.3. Detalhes Cruciais da Estrutura

* **Chaves Estrangeiras (FK):** As tabelas `ACCOUNTS`, `CATEGORIES` e `TRANSACTIONS` possuem referências (`FK`) para a tabela `USERS`, garantindo que os dados sejam isolados e pertencentes apenas ao usuário logado.
* **Armazenamento de Valor:** O campo `valor` na tabela `TRANSACTIONS` é **sempre armazenado como um número positivo**. O campo `type` (`INCOME` ou `EXPENSE`) é que define se o valor deve ser somado ou subtraído no cálculo do saldo (UC7).
* **Consultas de Agregação:** O campo `data` na tabela `TRANSACTIONS` é fundamental para os Casos de Uso UC8 (Fluxo de Caixa) e UC9 (Distribuição de Despesas), pois permite a agregação e agrupamento por período.
