# DML, History e Time Travel no Delta Lake

Este notebook é o ápice do nosso pipeline. Em um Data Lake tradicional (apenas com CSVs ou Parquets puros), atualizar ou deletar um único registro é uma operação complexa que exige a reescrita completa do arquivo.

Com o **Delta Lake** na nossa Camada Bronze, ganhamos a habilidade de realizar transações ACID (operações seguras de DML) diretamente sobre o *Object Storage* (MinIO), usando comandos SQL familiares.

!!! note "Objetivo Técnico"
    Demonstrar operações **DML** (`INSERT`, `UPDATE`, `DELETE`) em tabelas Delta, e utilizar ferramentas de governança como **History** (Log de Auditoria) e **Time Travel** (Viagem no tempo para recuperar versões antigas).

## 1. Registro do Catálogo (Metastore)

Antes de executarmos comandos SQL puros, o Spark precisa saber onde os arquivos Delta estão morando dentro do MinIO. Nós mapeamos as pastas do bucket `bronze` para tabelas virtuais no catálogo do Spark.

Python
```
title="Registrando Tabelas Virtuais"
tabelas_delta = ['anexo', 'cidade', 'estado', 'ouvidoria', 'servico_afetado', 'tipo_ouvidoria', 'usuario']

for tabela in tabelas_delta:
    delta_path = f's3a://{BRONZE_BUCKET}/{tabela}'
    spark.sql(f"""
        CREATE TABLE IF NOT EXISTS {tabela}
        USING delta
        LOCATION '{delta_path}'
    """)
```

## 2. Transações ACID (DML)

Uma vez que as tabelas estão registradas, podemos manipulá-las livremente. O projeto demonstra duas abordagens equivalentes: o uso de Spark SQL puro e o uso da DeltaTable API nativa do Python.

### 2.1 Inserção (INSERT)

Adicionamos novas ocorrências de ouvidoria ao sistema de forma transacional. Se o servidor cair no meio da inserção, o Delta Lake garante que o arquivo não ficará corrompido (Atomicidade).

```
spark.sql("""
    INSERT INTO ouvidoria (id_ouvidoria, descricao_ouvidoria, cod_tipo, cod_servico, protocolo_ouvidoria, data_ouvidoria, cod_usuario) 
    VALUES 
        (4, 'Solicitação de maior frequência de rondas policiais.', 3, 6, '202605030001', '2026-05-03 08:00:00', 1),
        (5, 'Reclamação sobre o atraso nas obras de recapeamento.', 4, 7, '202605030002', '2026-05-03 09:15:00', 2);
    -- (Mais registros omitidos para brevidade)
""")
```

### 2.2 Atualização (UPDATE)

Se a descrição de um chamado precisar ser corrigida, não precisamos reescrever o lake inteiro.

=== "Via Spark SQL"

```
sql UPDATE ouvidoria  SET descricao_ouvidoria = 'Falta de medicamentos nos postos de saúde.'  WHERE id_ouvidoria = 1
```

=== "Via DeltaTable API"

Python
```
from pyspark.sql.functions import lit

dt_servico = DeltaTable.forPath(spark, f's3a://{BRONZE_BUCKET}/servico_afetado')
dt_servico.update(
    condition="id_servico = 1",
    set={"nome_servico": lit("Educação, Ciência e Tecnologia")}
)
```

### 2.3 Exclusão (DELETE)

Ideal para lidar com leis de proteção de dados (como o "Direito ao Esquecimento" da LGPD) ou para limpar sujeiras pontuais do banco.

```
# Limpeza de IDs nulos
spark.sql("DELETE FROM ouvidoria WHERE id_ouvidoria is null")

# Deleção via API
dt_ouvidoria = DeltaTable.forPath(spark, f's3a://{BRONZE_BUCKET}/ouvidoria')
dt_ouvidoria.delete("id_ouvidoria = 6")
```

### 3. Governança: History e Time Travel

Cada vez que aplicamos um dos comandos DML acima, o Delta Lake gera um arquivo .json escondido na pasta _delta_log. Isso nos permite fazer duas coisas incríveis:

Auditoria Completa (History)

Podemos descobrir exatamente que tipo de operação modificou a tabela.

```
# Retorna: version, timestamp, operation, operationMetrics
spark.sql('DESCRIBE HISTORY ouvidoria').show()
```

Viagem no Tempo (Time Travel)

Erramos um UPDATE ou um DELETE sem a cláusula WHERE? O Delta permite recarregar o Dataframe exatamente como ele era no passado, usando o parâmetro versionAsOf.

Python
```
# Recarrega a tabela ouvidoria exatamente como ela foi criada na Versão 0
df_v0 = spark.read.format('delta').option('versionAsOf', 0).load(f's3a://{BRONZE_BUCKET}/ouvidoria')

df_atual = spark.read.format('delta').load(f's3a://{BRONZE_BUCKET}/ouvidoria')

print(f'Versão Original: {df_v0.count()} registros')
print(f'Versão Atual: {df_atual.count()} registros')
```

## 4. Resumo da Execução

Abaixo, o relatório final impresso pelo notebook após executar e validar todas as transações:

```
print('=' * 70)
print('RESUMO DAS OPERAÇÕES DML REALIZADAS NO DATA LAKE (OUVIDORIA)')
print('=' * 70)
print()
print('INSERT:')
print('  - 5 novas ouvidorias inseridas (IDs 4, 5, 6, 7 e 8)')
print()
print('UPDATE:')
print('  - ouvidoria ID 1 -> Descrição atualizada para "Falta de medicamentos..." (via SQL)')
print('  - servico_afetado ID 5 -> Nome atualizado para "Agropecuária" (via SQL)')
print('  - servico_afetado ID 1 -> Nome atualizado para "Educação, Ciência..." (via DeltaTable API)')
print()
print('DELETE:')
print('  - ouvidorias com ID nulo (sujeira) removidas (via SQL)')
print('  - ouvidoria ID 8 (sugestão de semáforos) removida (via SQL)')
print('  - ouvidoria ID 6 (denúncia de resíduos) removida (via DeltaTable API)')
print()
print('HISTORY e TIME TRAVEL:')
print('  - Histórico completo de transações consultado (DESCRIBE HISTORY)')
print('  - Leitura do estado original dos dados (versionAsOf = 0)')
print('  - Comparação e extração de diferenças entre Versão 0 vs Versão Atual')
print('=' * 70)
```