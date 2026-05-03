# Extração (PostgreSQL ➔ MinIO)

Este notebook é responsável pela etapa primária de ingestão de dados (Extração). Ele se conecta ao banco de dados relacional **PostgreSQL**, identifica as tabelas disponíveis e as exporta no formato bruto (**CSV**) diretamente para a **Landing Zone** no **MinIO**.

!!! note "Objetivo Técnico"
    Realizar a extração dos dados transacionais de forma automatizada e enviá-los para o *Object Storage* (MinIO) de forma performática, utilizando buffers em memória para não gerar I/O (leitura/escrita) desnecessário no disco local.

## 1. Configuração e Conexão

O processo inicia estabelecendo a comunicação independente com as duas pontas da arquitetura: o banco de dados de origem e o *storage* de destino. Utilizamos a biblioteca `pyodbc` com o driver do PostgreSQL para a conexão relacional, e o `boto3` para a comunicação com a API S3 do MinIO.

Python
``` 
title="Conexões do Pipeline"
import pyodbc
import boto3
from botocore.client import Config

# Conexão ODBC com o banco de dados
conn = pyodbc.connect(
    f'DRIVER={{PostgreSQL Unicode}};'
    f'SERVER={DB_SERVER};'
    f'PORT={DB_PORT};'
    f'DATABASE={DB_DATABASE};'
    f'UID={DB_USER};'
    f'PWD={DB_PASSWORD};',
    autocommit=True
)
cursor = conn.cursor()

# Cliente AWS S3 apontado para o MinIO local
s3_client = boto3.client(
    's3',
    endpoint_url=MINIO_ENDPOINT,
    aws_access_key_id=MINIO_ACCESS_KEY,
    aws_secret_access_key=MINIO_SECRET_KEY,
    config=Config(signature_version='s3v4'),
    region_name='us-east-1'
)
```

## 2. Mapeamento Dinâmico de Tabelas

Para garantir que a extração seja escalável e não dependa de nomes "chumbados" no código, o notebook consulta dinamicamente a tabela de metadados de sistema do banco de dados (information_schema.tables). Isso permite que ele liste e percorra automaticamente todas as tabelas de usuário (BASE TABLE) contidas no schema public.

Python
``` 
cursor.execute("""
    SELECT table_name 
    FROM information_schema.tables 
    WHERE table_schema = 'public' 
    AND table_type = 'BASE TABLE' 
    ORDER BY table_name
""")
tabelas = [row[0] for row in cursor.fetchall()]
```

## 3. Extração e Upload em Memória

A extração de cada tabela é consolidada em um DataFrame do Pandas. Para otimizar o envio para a nuvem/storage local, o DataFrame é convertido em um arquivo CSV virtual, alocado diretamente na memória RAM usando a classe io.StringIO(). O conteúdo convertido em bytes é disparado diretamente para o MinIO.


Python
``` 
import io
import pandas as pd

for tabela in tabelas:
    # Captura os dados da tabela
    cursor.execute(f'SELECT * FROM "{tabela}"')
    columns = [desc[0] for desc in cursor.description]
    df = pd.DataFrame.from_records(cursor.fetchall(), columns=columns)

    # Cria o buffer de CSV na memória RAM
    csv_buffer = io.StringIO()
    df.to_csv(csv_buffer, index=False)
    csv_bytes = csv_buffer.getvalue().encode('utf-8')

    # Faz o Upload direto para a Landing Zone
    s3_client.put_object(
        Bucket=LANDING_BUCKET, 
        Key=f'{tabela}.csv',
        Body=csv_bytes, 
        ContentType='text/csv'
    )
```

## 4. Validação da Ingestão

Após finalizar o loop de extração, o script faz uma listagem usando s3_client.list_objects_v2 para validar o conteúdo físico presente no bucket landing-zone. Um resumo operacional (contendo total de registros e total de KB trafegados) é impresso para fins de log e monitoramento.

!!! success "Camada Raw Concluída"
Com a finalização deste notebook, os dados são retirados com segurança do ambiente transacional (PostgreSQL) e entram oficialmente no ecossistema isolado do Data Lake, prontos para a etapa de limpeza e conversão.