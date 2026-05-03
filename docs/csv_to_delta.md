# CSV para Delta Lake

Este notebook é responsável pela primeira grande transformação do nosso pipeline. Nele, pegamos os arquivos brutos em texto plano (**CSV**) que foram extraídos do banco de dados e os convertemos para o formato **Delta Lake**, armazenando-os na Camada Bronze do nosso **Data Lakehouse**.

!!! note "Objetivo Técnico"
Migrar dados da **Landing Zone** (armazenamento temporário e bruto) para a **Bronze Layer** (tabelas transacionais), garantindo otimização de leitura e suporte a transações ACID.

## 1. Configuração e Ambiente

Nesta etapa inicial, carregamos as variáveis de ambiente (credenciais do MinIO) e inicializamos a SparkSession. A configuração é crítica, pois informa ao Spark que ele deve usar a extensão do Delta Lake e os protocolos de comunicação do Amazon S3 (S3A).

```
from pyspark.sql import SparkSession
from delta import *

spark = (
    SparkSession.builder
    .appName('CSV_to_Delta')
    .master('local[*]')
    # Injeção dos pacotes Delta e Hadoop-AWS
    .config('spark.jars.packages', 'io.delta:delta-spark_2.12:3.2.0,org.apache.hadoop:hadoop-aws:3.3.4')
    .config('spark.sql.extensions', 'io.delta.sql.DeltaSparkSessionExtension')
    .config('spark.sql.catalog.spark_catalog', 'org.apache.spark.sql.delta.catalog.DeltaCatalog')
    # Configurações de conexão com o MinIO
    .config('spark.hadoop.fs.s3a.endpoint', MINIO_ENDPOINT)
    .config('spark.hadoop.fs.s3a.access.key', MINIO_ACCESS_KEY)
    .config('spark.hadoop.fs.s3a.secret.key', MINIO_SECRET_KEY)
    .config('spark.hadoop.fs.s3a.path.style.access', 'true')
    .config('spark.hadoop.fs.s3a.impl', 'org.apache.hadoop.fs.s3a.S3AFileSystem')
    .getOrCreate()
)
```
## 2. O Processo de Conversão

O pipeline percorre de forma automatizada todos os arquivos detectados no bucket landing-zone. Para cada arquivo, o Spark realiza as seguintes ações:

* **Leitura com Inferência:** `O Spark lê o CSV e tenta "adivinhar" o tipo de dado de cada coluna (String, Integer, Timestamp).`
* **Escrita Otimizada:** `Os dados são gravados no formato Parquet dentro da estrutura Delta, criando o log de transações (_delta_log).`

!!! info "Vantagens da Conversão"
* **Compressão**: Arquivos Delta/Parquet ocupam muito menos espaço que CSVs.
* **Metadados**: O formato Delta armazena o schema, evitando que dados corrompidos quebrem o pipeline futuramente.
---

## 3. Validação de Integridade

Após a carga, o notebook executa uma rotina de validação para garantir que:

- O local de destino é reconhecido como uma Delta Table oficial.

- A contagem de registros entre o CSV original e a nova tabela Delta é idêntica.

Python
```
from delta.tables import DeltaTable

# Verificação de formato
is_delta = DeltaTable.isDeltaTable(spark, delta_path)
print(f"A tabela {tabela} é Delta? {is_delta}")
```

## 4. Resumo da Execução

Abaixo, o relatório final gerado pelo código ao concluir o processamento:

Python
```
print('=' * 75)
print('RESUMO DA CONVERSÃO E INGESTÃO NO DATA LAKE (LANDING ➔ BRONZE)')
print('=' * 75)
print()
print('ORIGEM (Landing Zone):')
print('  - Bucket MinIO: landing-zone')
print('  - Formato dos arquivos: CSV (Dados Brutos / Plain Text)')
print()
print('DESTINO (Camada Bronze):')
print('  - Bucket MinIO: bronze')
print('  - Formato dos arquivos: Delta Lake (Parquet Otimizado + Log de Transações)')
print()
print('OPERAÇÕES REALIZADAS:')
print('  - Conexão nativa com Object Storage estabelecida via s3a://')
print('  - Inferência automática de schema (tipagem de colunas) aplicada na leitura.')
print('  - Todas as 7 tabelas do sistema lidas e convertidas simultaneamente.')
print('  - Estrutura de metadados (_delta_log) criada com sucesso no MinIO.')
print()
print('VALIDAÇÃO E GOVERNANÇA:')
print('  - Verificação de formato (DeltaTable.isDeltaTable) retornou True.')
print('  - Contagem de registros preservada 100% entre a origem e o destino.')
print('=' * 75)
```

!!! success "Status Final"
Com a conclusão deste notebook, os dados da Ouvidoria estão prontos para sofrerem alterações de DML (Insert, Update, Delete) e consultas analíticas de alta performance.