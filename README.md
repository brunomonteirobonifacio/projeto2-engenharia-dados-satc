# Projeto Apache Spark + Delta Lake + MinIO + SQL Server

Projeto desenvolvido para o curso de Engenharia de Dados com Spark e Delta Lake, lendo os dados de um databases SQL Server e carregando no MinIO.

1. **Extração** de tabelas de um banco SQL Server 2025
2. **Carga** no MinIO (Object Storage compatível com S3) no formato CSV
3. **Conversão** para formato Delta Lake
4. **Manipulação** dos dados com comandos DML (INSERT, UPDATE, DELETE)

## Arquitetura

```
┌─────────────────┐     ┌──────────────────┐     ┌───────────────────┐
│   SQL Server    │────▶│   MinIO (S3)     │────▶│   MinIO (S3)      │
│   2025 Dev      │     │   landing-zone/  │     │   bronze/         │
│                 │     │   (CSVs)         │     │   (Delta Tables)  │
│   SeguroDB      │     │                  │     │                   │
│   11 tabelas    │     │   1 CSV/tabela   │     │   INSERT/UPDATE   │
│                 │     │                  │     │   DELETE/HISTORY  │
└─────────────────┘     └──────────────────┘     └───────────────────┘
     Notebook 00             Notebook 01            Notebooks 02/03
     (Setup)                 (Extração)             (Delta + DML)
```

## Pré-requisitos

- **Linux** (Ubuntu 24.04 ou WSL do Windows 11)
- **Docker** e **Docker Compose** v2+
- **Python 3.11** (PySpark 3.5.3 requer Python ≤ 3.12)
- **Java 11** (OpenJDK)
- **UV** (gerenciador de pacotes Python) — [instalação](https://github.com/astral-sh/uv)
- **ODBC Driver 18 for SQL Server** — [instalação](https://learn.microsoft.com/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server)

## Setup do Ambiente

### 1. Subir os Containers (SQL Server + MinIO)

```bash
docker compose up -d
```

**Containers criados:**

| Container       | Imagem                                      | Portas          |
|----------------|---------------------------------------------|-----------------|
| sqlserver-2025 | `mcr.microsoft.com/mssql/server:2025-latest` | `1433`          |
| minio          | `minio/minio:RELEASE.2025-02-03T21-03-04Z`  | `9020`, `9021`  |

**Credenciais:**

| Serviço     | Usuário       | Senha              |
|-------------|---------------|---------------------|
| SQL Server  | `sa`          | `SqlServer@2025!`  |
| MinIO       | `minioadmin`  | `minioadmin`       |

**Console MinIO:** http://localhost:9021

### 2. Configurar Variáveis de Ambiente

Crie o arquivo de configuração a partir do template fornecido:

```bash
cp .env.example .env
```

### 3. Configurar o Ambiente Python

```bash
uv venv
source .venv/bin/activate
uv sync
```

### 4. Instalar ODBC Driver - Client SQL Server (Ubuntu)

```bash
# Ubuntu 24.04
sudo apt install -y unixodbc-dev
curl https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc
sudo curl https://packages.microsoft.com/config/ubuntu/24.04/prod.list | sudo tee /etc/apt/sources.list.d/mssql-release.list
sudo apt update
sudo ACCEPT_EULA=Y apt install -y msodbcsql18
```
### 4.1. Validar instalação do ODBC Driver - Client SQL Server (Ubuntu)

```bash
# Ubuntu 24.04
odbcinst -q -d
```

Deve retornar `[ODBC Driver 18 for SQL Server]`.

## Executando o Projeto

Execute os notebooks **em ordem**:

| # | Notebook | Descrição |
|---|----------|-----------|
| 0 | `00_setup_sqlserver.ipynb` | Cria database `SeguroDB` e carrega 11 tabelas com dados de exemplo |
| 1 | `01_sqlserver_to_minio_csv.ipynb` | Extrai todas as tabelas do SQL Server → CSV no MinIO (bucket `landing-zone`) |
| 2 | `02_csv_to_delta.ipynb` | Lê CSVs do MinIO e converte para Delta Lake (bucket `bronze`) |
| 3 | `03_dml_delta.ipynb` | Executa comandos DML (INSERT, UPDATE, DELETE), exibe HISTORY e TIME TRAVEL |

> **Importante:** Selecione o ambiente virtual (`.venv`) como Kernel do Jupyter antes de executar.

## Estrutura do Projeto

```
spark-delta-minio-sqlserver/
├── docker-compose.yml                   # SQL Server 2025 + MinIO
├── .env.example                         # Template de variáveis de ambiente
├── pyproject.toml                       # Dependências Python (UV)
├── .python-version                      # Python 3.11
├── data/                                # CSVs de dados de exemplo
│   ├── regiao.csv
│   ├── estado.csv
│   ├── municipio.csv
│   ├── marca.csv
│   ├── modelo.csv
│   ├── cliente.csv
│   ├── endereco.csv
│   ├── telefone.csv
│   ├── carro.csv
│   ├── apolice.csv
│   └── sinistro.csv
├── notebook/                            # Notebooks Jupyter
│   ├── 00_setup_sqlserver.ipynb         # Setup: CSV → SQL Server
│   ├── 01_sqlserver_to_minio_csv.ipynb  # Extração: SQL Server → MinIO (CSV)
│   ├── 02_csv_to_delta.ipynb            # Conversão: CSV → Delta Lake
│   └── 03_dml_delta.ipynb               # DML: INSERT, UPDATE, DELETE
└── README.md
```

## Tecnologias Utilizadas

- **Apache Spark 3.5.3** (PySpark) — Motor de processamento distribuído
- **Delta Lake 3.2.0** — Formato de armazenamento com suporte ACID
- **MinIO** — Object Storage compatível com S3
- **SQL Server 2025** — Banco de dados relacional (Developer Edition)
- **Docker Compose** — Orquestração de containers
- **Python 3.11** com UV

## Conceitos Demonstrados

- **Extração de dados** de banco relacional (SQL Server)
- **Object Storage** como repositório de dados (MinIO/S3)
- **Delta Lake** como formato de armazenamento lakehouse
- **Transações ACID** em data lakes
- **DML** (INSERT, UPDATE, DELETE) em tabelas Delta
- **Versionamento** de dados (History e Time Travel)
- **Arquitetura Medalhão** (Landing Zone → Bronze)

## Links e Referências

- [Delta Lake - Releases](https://docs.delta.io/latest/releases.html)
- [Delta Spark - Maven Repository](https://mvnrepository.com/artifact/io.delta/delta-spark)
- [MinIO - Documentação](https://min.io/docs/minio/linux/index.html)
- [SQL Server 2025 - Docker](https://learn.microsoft.com/sql/linux/quickstart-install-connect-docker)
