# Apache Iceberg

!!! info "O que é o Apache Iceberg?"
    O **Apache Iceberg** é um formato de tabela aberto e de alto desempenho para conjuntos de dados analíticos muito grandes. Diferente de focar apenas em pastas (como no Hive), o Iceberg rastreia dados no nível do arquivo, oferecendo melhor performance e operações ACID.

## Objetivo da implementação

Demonstrar o uso do Apache Iceberg com Apache Spark, com foco na sua gestão por catálogos e snapshots.

## Estrutura utilizada

* **Catálogo:** `local`
* **Database:** `db_teste`
* **Tabela:** `tabela_v1`

---

## Operações Realizadas

### 1. Criação da database
Criando a base de dados dentro do catálogo configurado no Spark.

SQL
```CREATE DATABASE IF NOT EXISTS local.db_teste;```

2. Criação da tabela e carga inicial
Os dados iniciais foram criados utilizando a API de DataFrame do PySpark:

Python
```
data = [
    ("Engenharia", 1),
    ("Dados", 2),
    ("Big Data", 3)
]
```
# Criação do DataFrame
```df = spark.createDataFrame(data, ["palavra", "valor"])```

# Escrita utilizando o formato Iceberg
```
df.writeTo("local.db_teste.tabela_v1").createOrReplace()
```
!!! successo "Visualização Inicial"
Após a execução do PySpark, um ```SELECT * FROM local.db_teste.tabela_v1;``` retornará os registros mapeados pelo catálogo do Iceberg.

### 1. Inserção (INSERT)
SQL
```

INSERT INTO local.db_teste.tabela_v1 VALUES ('Spark', 4);
```

### 2. Atualização (UPDATE)
SQL
```
UPDATE local.db_teste.tabela_v1
SET valor = 99
WHERE palavra = 'Dados';
```

### 3. Exclusão (DELETE)
SQL
```
DELETE FROM local.db_teste.tabela_v1
WHERE palavra = 'Engenharia';
```

### 4. Versionamento (Snapshots)
O Iceberg trabalha com o conceito de Snapshots (fotografias do estado dos dados). Podemos consultar o histórico de alterações através da tabela de metadados:

SQL
```
SELECT * FROM local.db_teste.tabela_v1.snapshots;
```