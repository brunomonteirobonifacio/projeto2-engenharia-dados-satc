---
hide:
  - navigation
  - toc
---

# Projeto 2 de Engenharia de Dados - SATC

Trabalho desenvolvido para a disciplina de Engenharia de Dados do curso de Engenharia de Software da **UNISATC**.

!!! abstract "Resumo do Projeto"
    Este projeto implementa um pipeline completo de Engenharia de Dados utilizando o conceito de **Data Lakehouse**. A partir de um banco de dados relacional que simula um sistema transacional de **Ouvidoria Pública**, os dados são extraídos, ingeridos em um *Object Storage* e transformados em tabelas transacionais otimizadas para análise.

## Objetivo

O objetivo principal é demonstrar, na prática, o ciclo de vida do dado desde sua origem até a camada analítica (Bronze), garantindo a confiabilidade dos dados através de transações ACID e controle de versionamento (*Time Travel*).

As operações validadas neste pipeline incluem:

* **Conectividade:** Extração JDBC via Apache Spark de um banco PostgreSQL.
* **Ingestão (Landing Zone):** Carga inicial de dados brutos (CSV) no MinIO (S3-compatible).
* **Lakehouse (Bronze Layer):** Conversão dos dados para o formato **Delta Lake**.
* **Manipulação de Dados (DML):** Testes de `INSERT`, `UPDATE` e `DELETE` direto no Data Lake.
* **Governança:** Consulta de metadados e histórico de alterações estruturais.

## Domínio dos Dados (Ouvidoria)

O ecossistema simula um sistema de ouvidoria, composto por **7 tabelas relacionais** que se conectam para mapear chamados da população:

1. `estado`
2. `cidade`
3. `servico_afetado`
4. `tipo_ouvidoria`
5. `usuario`
6. `ouvidoria` *(Tabela Fato)*
7. `anexo`

## Arquitetura do Pipeline

O projeto segue os princípios da **Arquitetura Medalhão**, focando nas etapas iniciais de ingestão:

```
graph LR
  A[(PostgreSQL<br>SeguroDB)] -->|Extração Spark| B[(MinIO S3<br>Landing Zone)]
  B -->|Conversão| C[(MinIO S3<br>Bronze Layer)]
  
  style A fill:#336791,stroke:#fff,stroke-width:2px,color:#fff
  style B fill:#C73A49,stroke:#fff,stroke-width:2px,color:#fff
  style C fill:#CD7F32,stroke:#fff,stroke-width:2px,color:#fff
```
## Stack Tecnológica

- **Processamento**: Apache Spark 3.5.3 (PySpark)

- **Storage / Lakehouse**: MinIO & Delta Lake 3.2.0

- **Banco de Dados (Origem)**: PostgreSQL 14+

- **Infraestrutura**: Docker & Docker Compose

- **Gestão de Ambiente**: Python 3.11 + UV

- **Documentação**: MkDocs Material

!!! tip "Navegue pelo menu superior para entender a fundo como configuramos a engine do Apache Spark e como o Delta Lake garante a consistência das nossas tabelas."


