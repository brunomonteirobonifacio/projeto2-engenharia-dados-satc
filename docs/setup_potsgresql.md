# Setup e Carga Inicial (PostgreSQL)

Este notebook é o ponto de partida do nosso projeto. Para que o nosso pipeline de Engenharia de Dados faça sentido, precisamos de um "Sistema de Origem" que simule um ambiente de produção real.

Nesta etapa, conectamos ao nosso contêiner do **PostgreSQL**, criamos a modelagem relacional do sistema de **Ouvidoria** e populamos as tabelas com os dados brutos armazenados localmente.

!!! note "Objetivo Técnico"
    Provisionar automaticamente o banco de dados relacional (`sistema_ouvidoria`), aplicar a modelagem física (DDL) com integridade referencial e realizar a carga em lote (Bulk Insert) a partir de arquivos `.csv`.

## 1. Instalação de Drivers e Conexão Base

Como o PostgreSQL está rodando em um contêiner Docker isolado, o nosso ambiente Python (gerenciado pelo UV) precisa dos drivers adequados do sistema operacional para "falar" com o banco via protocolo ODBC.

Python
```
title="Configuração do ODBC"
# Instalação via sistema (Linux/WSL) dos pacotes necessários para o pyodbc
!apt-get update && apt-get install -y unixodbc unixodbc-dev odbc-postgresql
```

Com o driver instalado, o script estabelece uma conexão primária com o banco postgres (master) apenas para verificar a existência e criar, se necessário, o nosso banco de dados alvo: sistema_ouvidoria.

## 2. Modelagem Relacional (DDL)

Com o banco de dados criado, o script executa um bloco de instruções DDL (Data Definition Language). Ele constrói a estrutura de 7 tabelas normalizadas, respeitando a ordem de dependência das Chaves Estrangeiras (Foreign Keys).

As tabelas criadas formam o seguinte domínio de negócio:

- **estado** (Tabela domínio)

- **cidade** (Depende de estado)

- **usuario** (Depende de cidade)

- **tipo_ouvidoria** (Tabela domínio)

- **servico_afetado** (Tabela domínio)

- **ouvidoria** (Tabela Fato - Depende de usuário, tipo e serviço)

- **anexo** (Depende de ouvidoria)

## 3. Carga de Dados (Bulk Insert)

Esta é a etapa mais pesada do notebook. Lemos os arquivos .csv da pasta /data e inserimos os registros nas tabelas recém-criadas.

Para garantir a eficiência e a qualidade dos dados, utilizamos a biblioteca Pandas:

- **Limpeza Prévia:** O Pandas varre todas as colunas de texto (strings) e aplica um .strip() para remover espaços em branco invisíveis que poderiam causar problemas de join no futuro.

- **Tratamento de Nulos:** Valores NaN do Pandas são convertidos nativamente para None do Python, garantindo que o PostgreSQL receba registros NULL verdadeiros.

- **Inserção em Lotes:** Para não estourar a memória ou a conexão, usamos o executemany do cursor ODBC, enviando os dados em lotes (batches) de 2.000 registros por vez.

```
# Inserir em lotes de 2000 registros para otimizar a rede e o banco
batch_size = 2000
for i in range(0, len(data), batch_size):
    batch = data[i:i+batch_size]
    cursor.executemany(insert_sql, batch)

```

!!! tip "Idempotência"
O script foi desenhado para ser idempotente. Antes de iniciar a carga de uma tabela, ele executa um SELECT COUNT(*). Se a tabela já possuir registros, ele pula a carga daquele arquivo. Isso evita a duplicação de dados se o notebook for executado mais de uma vez.

4. Validação

A última célula do notebook realiza uma auditoria básica para garantir que a carga foi bem-sucedida. Ele percorre as 7 tabelas, conta o número exato de linhas inseridas no PostgreSQL e exibe uma amostra (Top 5) de tabelas cruciais, como cidade, servico_afetado e ouvidoria.

!!! success "Ambiente Pronto"
Após a execução deste notebook, o ambiente de produção fictício está 100% operacional. A partir daqui, o pipeline de dados entra em ação para extrair e migrar esses dados para o Data Lake.
