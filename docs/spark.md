
# Apache Spark

!!! info "O que é o Apache Spark?"
    O **Apache Spark** é um sistema de processamento distribuído de código aberto usado para workloads de big data. O sistema usa armazenamento em cache na memória e execução otimizada de consultas para oferecer consultas analíticas rápidas de dados de qualquer tamanho.

## Objetivo da implementação

Prover a engine de processamento distribuído responsável por executar operações analíticas e transacionais sobre dados armazenados em formatos de tabela modernos (Delta Lake e Iceberg).

## Estrutura utilizada

* **Diretório raiz de armazenamento dos dados:** `data`
* **Notebooks:** `notebooks/`
* O Spark lê e escreve dados a partir do diretório base configurado

---

## Operações Realizadas

### 1. Sessão Spark
Comando em Python para inicializar uma sessão do Spark com PySpark

```python
spark = SparkSession.builder \
    .appName("App_Name") \
    # ... configurações do Delta Lake / Apache Iceberg
    .getOrCreate()

```
### 2. Configurações relevantes
Configurações omitidas na seção acima, permitem a integrçaão com Delta Lake e Apache Iceberg.

Configurações utilizadas em ambos Apache Iceberg e Delta Lake:
```
spark.jars.packages
spark.sql.extensions
```

Configurações específicas do Delta Lake:
```
spark.sql.catalog.spark_catalog
```

Configurações específicas do Apache Iceberg:
```
spark.sql.catalog.local
spark.sql.catalog.local.type
spark.sql.catalog.local.warehouse
```
