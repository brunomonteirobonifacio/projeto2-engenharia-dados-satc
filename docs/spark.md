# Apache Spark

!!! info "O que é o Apache Spark?"
    O **Apache Spark** é um sistema de processamento distribuído de código aberto, projetado para lidar com workloads de Big Data. No contexto de Data Lakes modernos, ele atua como o principal motor computacional, utilizando armazenamento em cache na memória para executar transformações e consultas pesadas de forma extremamente rápida.

## O Papel do Spark no Projeto

Neste projeto, o Spark (através do **PySpark**) atua como a espinha dorsal de todo o pipeline de ETL (Extração, Transformação e Carga). Ele foi configurado para rodar localmente sobre um ambiente Python gerenciado pelo `uv`, comunicando-se ativamente com a infraestrutura em Docker (PostgreSQL e MinIO).

Suas responsabilidades foram divididas em quatro rotinas principais (Notebooks):

!!! success "Módulos de Processamento"
    1. **Setup Transacional (`00_setup_postgres`):** Carga em lote (Bulk Insert) lendo arquivos CSV locais e escrevendo no PostgreSQL via driver JDBC para simular um banco de produção.
    2. **Extração para Landing Zone (`01_postgres_to_minio_csv`):** Leitura de dados do PostgreSQL através de queries distribuídas e gravação bruta no Object Storage (MinIO) no formato `.csv`.
    3. **Conversão para Lakehouse (`02_csv_to_delta`):** Processamento dos CSVs da Landing Zone, inferência de schemas e conversão para tabelas **Delta Lake** na camada Bronze.
    4. **Manipulação ACID (`03_dml_delta`):** Uso do Spark SQL para disparar comandos DML (Insert, Update, Delete) diretamente sobre os arquivos parquet do MinIO.

---

## Configuração da Sessão (SparkSession)

Para que o Spark consiga conversar perfeitamente com o ecossistema do projeto, a inicialização da `SparkSession` exige a injeção de bibliotecas Java (JARS) e extensões SQL específicas.

Abaixo está o detalhamento da configuração utilizada na nossa rotina principal:

```
python title="Configuração PySpark + Delta + PostgreSQL"
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("PipelineOuvidoria") \
    # 1. Dependências Maven (JARS)
    .config("spark.jars.packages", "org.postgresql:postgresql:42.7.3,io.delta:delta-spark_2.12:3.2.0,org.apache.hadoop:hadoop-aws:3.3.4") \
    # 2. Extensões SQL do Delta Lake
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    # 3. Configuração do Catálogo (Spark)
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    # 4. Credenciais e Endpoint do MinIO (S3)
    .config("spark.hadoop.fs.s3a.endpoint", "http://localhost:9020") \
    .config("spark.hadoop.fs.s3a.access.key", "minioadmin") \
    .config("spark.hadoop.fs.s3a.secret.key", "minioadmin") \
    .config("spark.hadoop.fs.s3a.path.style.access", "true") \
    .config("spark.hadoop.fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem") \
    .getOrCreate()

```
# Entendendo os Parâmetros
  
- **spark.jars.packages:** Instrução para o Spark baixar os tradutores necessários da internet:  

- Driver JDBC do PostgreSQL (org.postgresql).  

- Conector do Delta Lake (io.delta).  

- **Protocolo** AWS S3 para conexão com o MinIO (hadoop-aws).  

- **spark.sql.extensions:** Altera a engine nativa de SQL do Spark para que ele reconheça comandos específicos do Delta, como UPDATE e MERGE.

- **spark.hadoop.fs.s3a.:** Conjunto de chaves de ambiente que mascara o nosso MinIO local para que o Spark acredite estar conversando com um bucket real do Amazon S3.

!!! note "Performance em Memória"
Ao utilizar a API de DataFrames do PySpark com as configurações acima, garantimos que os dados da Ouvidoria sejam processados inteiramente em memória e paralelizados, evitando gargalos de leitura e escrita em disco.