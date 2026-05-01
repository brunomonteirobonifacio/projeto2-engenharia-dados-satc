
# Delta Lake

!!! info "O que é o Delta Lake?"
    O **Delta Lake** é uma camada de armazenamento open-source que traz confiabilidade para os Data Lakes. Ele fornece transações ACID, escalabilidade para tratamento de metadados e unifica o processamento de dados em batch e streaming.

## Objetivo da implementação

Demonstrar o uso do Delta Lake integrado ao Apache Spark para operações transacionais.

## Estrutura utilizada

* **Tabela:** `delta_table`
* **Local de armazenamento:** `data/data_lake/delta_table`

---

## Operações Realizadas

### 1. Criação da tabela
Comando DDL para criar a tabela e definir o local de armazenamento.

```sql
CREATE TABLE delta_table (
    palavra STRING,
    valor INT
)
USING DELTA
LOCATION 'data/delta_table';
```
### 2. Inserção de dados (INSERT)
Carga inicial de registros na tabela Delta.

SQL
```INSERT INTO delta_table VALUES
('Engenharia', 1),
('Dados', 2),
('Big Data', 3);```
!!! success "Visualização Inicial"
Após o insert, um SELECT * FROM delta_table; retornará os três registros acima de forma consistente.

### 3. Inserção incremental
Adicionando um novo registro posteriormente.

SQL
```INSERT INTO delta_table VALUES ('Spark', 4);```

### 4. Atualização de registros (UPDATE)
Modificando um valor existente, operação antes complexa em Data Lakes tradicionais

SQL
```
UPDATE delta_table
SET valor = 99
WHERE palavra = 'Dados';

### 5. Exclusão de dados (DELETE)
Removendo registros específicos da base.
```
SQL
```DELETE FROM delta_table
WHERE palavra = 'Engenharia';```

### 6. Consulta de histórico (Time Travel)
O Delta Lake mantém um log de transações. O comando abaixo permite ver todas as alterações feitas na tabela, quem fez e quando fez.

SQL

```DESCRIBE HISTORY delta_table;```