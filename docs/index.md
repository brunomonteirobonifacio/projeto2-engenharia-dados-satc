---
hide:
  - navigation
  - toc
---

# Projeto de Engenharia de Dados SATC

Trabalho desenvolvido para a disciplina de Engenharia de Dados do curso de Engenharia de Software da **UNISATC**.

!!! abstract "Resumo do Projeto"
    Este projeto tem como objetivo demonstrar a construção de uma pipeline de engenharia de dados utilizando **Apache Spark** integrado com tecnologias de vanguarda em arquiteturas de *Data Lakehouse*.

## Objetivo

Demonstrar, na prática, operações fundamentais em tabelas de dados com suporte a versionamento (*Time Travel*) e consistência transacional (ACID) diretamente sobre o Data Lake. As operações validadas incluem:

* **DDL:** Criação de tabelas e catálogos.
* **DML:** Inserção (INSERT), Atualização (UPDATE) e Exclusão (DELETE) de dados.
* **Governança:** Consulta de histórico, versionamento e snapshots.

## Abordagens e Tecnologias

Foram utilizadas duas tecnologias principais para o gerenciamento da camada de armazenamento, ambas processadas via **PySpark** no ambiente **Jupyter**:

1. [**Apache Iceberg**](iceberg.md): Focado no controle eficiente de metadados.
2. [**Delta Lake**](delta.md): Focado em confiabilidade e alta performance.

---
!!! note "Considerações Finais"
    O projeto evidencia como tecnologias de *Data Lakehouse* permitem realizar operações transacionais em arquivos Parquet brutos, superando as limitações dos Data Lakes tradicionais e garantindo confiabilidade e rastreabilidade das informações.