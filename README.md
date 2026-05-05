# Projeto Apache Spark + Delta Lake + MinIO + PostgreSQL

Projeto desenvolvido para a disciplina de Engenharia de Dados, focado na construção de um pipeline de dados utilizando Apache Spark e Delta Lake, realizando a extração de um banco relacional PostgreSQL e o armazenamento em MinIO (Object Storage).

Extração de tabelas de um banco relacional PostgreSQL.

Carga no MinIO (S3 compatível) no formato CSV (Landing Zone).

Conversão dos dados brutos para o formato Delta Lake (Bronze Layer).

Manipulação de dados utilizando comandos DML (INSERT, UPDATE, DELETE) com suporte a transações ACID.

## Arquitetura
```
┌─────────────────┐     ┌──────────────────┐     ┌───────────────────┐
│   PostgreSQL    │────▶│   MinIO (S3)     │────▶│   MinIO (S3)      │
│    Database     │     │   landing-zone/  │     │   bronze/         │
│                 │     │   (CSVs)         │     │   (Delta Tables)  │
│    SeguroDB     │     │                  │     │                   │
│    7 tabelas    │     │   1 CSV/tabela   │     │   INSERT/UPDATE   │
│                 │     │                  │     │   DELETE/HISTORY  │
└─────────────────┘     └──────────────────┘     └───────────────────┘
Notebook 00             Notebook 01            Notebooks 02/03
(Setup)                 (Extração)             (Delta + DML)
```

## Pré-requisitos
Linux (Ubuntu 24.04 ou WSL2 no Windows 11)

Docker e Docker Compose v2+

Python 3.11

Java 11 (OpenJDK)

UV (Gerenciador de pacotes e ambientes Python)

## Setup do Ambiente1. 

Subir os Containers (PostgreSQL + MinIO)Bash
```
sudo docker compose up -d
```

### Containers criados:

Container, Imagem, Portas

projeto2-eng-dados-postgres, postgres:14.22-trixie, 5432
projeto2-eng-dados-minio, minio/minio:RELEASE.2025-02-03T21-03-04Z, "9020, 9021"

### Credenciais:

Serviço, Usuário, Senha

PostgreSQL, postgres, mysecretpassword
MinIO, minioadmin, minioadmin

Console MinIO: http://localhost:9021

Configurar o Ambiente Python

Utilizamos o UV para garantir um ambiente rápido e isolado:

# Cria o ambiente virtual
```
uv venv
```

# Ativa o ambiente
```
source .venv/bin/activate
```

# Instala as dependências (Spark, Delta, etc)
```
uv sync
```

# Instala as dependências (Spark, Delta, etc)
```
uv sync
```

## Executando o Projeto

Execute os notebooks presentes na pasta notebook/ seguindo a ordem lógica do pipeline:


| # | Notebook | Descrição |
|---|----------|-----------|
| 0 | `00_setup_sqlserver.ipynb` | Cria database `SeguroDB` e carrega 11 tabelas com dados de exemplo |
| 1 | `01_sqlserver_to_minio_csv.ipynb` | Extrai todas as tabelas do SQL Server → CSV no MinIO (bucket `landing-zone`) |
| 2 | `02_csv_to_delta.ipynb` | Lê CSVs do MinIO e converte para Delta Lake (bucket `bronze`) |
| 3 | `03_dml_delta.ipynb` | Executa comandos DML (INSERT, UPDATE, DELETE), exibe HISTORY e TIME TRAVEL |

> **Importante:** Selecione o ambiente virtual (`.venv`) como Kernel do Jupyter antes de executar.

## Estrutura do Projeto

```
projeto2-engenharia-dados-satc/
├── docker-compose.yml           # Infraestrutura (Postgres + MinIO)
├── pyproject.toml               # Configuração de dependências (UV)
├── .env                         # Variáveis de ambiente (credenciais)
├── data/                        # Dados brutos para o setup inicial
│   ├── anexo.csv
│   ├── cidade.csv
│   ├── estado.csv
│   ├── ouvidoria.csv
│   ├── servico_afetado.csv
│   ├── tipo_ouvidoria.csv
│   └── usuario.csv
├── notebook/                    # Processamento Spark e Delta Lake
│   ├── 00_setup_postgres.ipynb
│   ├── 01_postgres_to_minio_csv.ipynb
│   ├── 02_csv_to_delta.ipynb
│   └── 03_dml_delta.ipynb
└── README.md                    # Documentação principal
```

## Tecnologias Utilizadas

- **Apache Spark 3.5.3** (PySpark) — Motor de processamento distribuído
- **Delta Lake 3.2.0** — Formato de armazenamento com suporte ACID
- **MinIO** — Object Storage compatível com S3
- **PostgreSQL** — Banco de dados relacional 
- **Docker Compose** — Orquestração de containers
- **Python 3.11** com UV

## Conceitos Demonstrados

- **Ingestão de Dados**: Carga de arquivos flat para banco relacional.

- **Extração via JDBC**: Spark conectando e lendo dados de um banco de dados.

- **Object Storage**: Armazenamento distribuído seguindo padrões de nuvem.

- **Lakehouse**: Implementação de tabelas Delta com suporte a transações ACID.

- **Arquitetura Medalhão**: Organização dos dados em camadas (Landing Zone e Bronze).

- **Governança**: Auditoria de dados via DESCRIBE HISTORY.

## Links e Referências

- [Delta Lake - Releases](https://docs.delta.io/latest/releases.html)
- [Delta Spark - Maven Repository](https://mvnrepository.com/artifact/io.delta/delta-spark)
- [MinIO - Documentação](https://min.io/docs/minio/linux/index.html)
- [PostgreSQL - Docker Hub](https://hub.docker.com/_/postgres)
