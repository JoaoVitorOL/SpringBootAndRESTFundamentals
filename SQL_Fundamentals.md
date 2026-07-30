# 🗄️ Guia Técnico: SQL do Zero ao Avançado

> **Nível:** Zero ao Avançado  
> **Linguagem:** SQL  
> **Fontes principais:** [ISO/IEC 9075-1:2023 — SQL/Framework](https://www.iso.org/standard/76583.html), [ISO/IEC 9075-2:2023 — SQL/Foundation](https://www.iso.org/standard/76584.html) e documentação oficial dos produtos comparados  
> **Versão de referência:** SQL:2023, distinguindo explicitamente padrão, dialeto, recurso do SGBD e ferramenta cliente  
> **Dialeto dos exemplos:** núcleo portável; exemplos específicos recebem rótulo do produto  
> **Atualizado em:** 21/07/2026

---

## Prefácio

[⬆️ Voltar ao Sumário](#sumário)

SQL costuma ser apresentado como uma sequência de comandos para “buscar dados”. Isso ensina a escrever um primeiro `SELECT`, mas não prepara alguém para modelar integridade, compreender `NULL`, controlar concorrência, interpretar um plano de execução, migrar um schema ou impedir que uma consulta correta produza um sistema lento e inseguro.

Este guia trata SQL como uma linguagem declarativa de bancos relacionais e como parte de um sistema maior. A linguagem descreve o resultado e as restrições desejados; o SGBD decide estratégias físicas; o driver transporta parâmetros e resultados; transações estabelecem fronteiras de atomicidade e isolamento; índices, estatísticas, locks, versões de linhas e logs sustentam a execução.

O texto segue o padrão didático de `C#_Fundamentals.md`, `Java_Fundamentals.md` e `C++_Fundamentals.md`:

- definição e modelo mental antes da sintaxe;
- exemplos pequenos com **leitura guiada**;
- tabelas para comparar semântica, custo e dialetos;
- atenção a integridade, concorrência, segurança e operação;
- catálogos para descobrir recursos prontos;
- referências oficiais próximas das afirmações.

SQL não é sinônimo de PostgreSQL, SQL Server, MySQL, Oracle Database ou SQLite. O padrão ISO define uma família ampla; cada implementação seleciona recursos, acrescenta extensões e possui comportamento operacional próprio. Este guia ensina primeiro o conceito estável e depois mostra onde confirmar a sintaxe real.

---

## Como usar este guia

[⬆️ Voltar ao Sumário](#sumário)

Há três trilhas:

1. **Trilha iniciante:** Partes 1–16, executando as consultas sobre um schema pequeno e prevendo o resultado antes de rodar.
2. **Trilha profissional:** Partes 17–27, com transações, concorrência, planos, segurança, integração, operação e mudanças de schema.
3. **Trilha de consulta:** Partes 28–29, anexos e glossário para localizar statements, funções, metadados, produtos e ferramentas.

Para cada consulta, responda:

1. Qual conjunto de linhas cada etapa lógica produz?
2. O que acontece quando um valor é `NULL`?
3. Que restrição protege a regra mesmo fora da aplicação?
4. A operação é atômica e qual isolamento ela exige?
5. O predicado permite acesso eficiente ou força processar muitas linhas?
6. A sintaxe é do padrão ou de um dialeto?
7. Os valores externos chegaram como parâmetros ou foram concatenados ao SQL?

> **Regra de laboratório:** execute exemplos destrutivos apenas em banco descartável. `DROP`, `TRUNCATE`, `DELETE` e migrations podem causar perda de dados; backup não conta até que restore tenha sido testado.

---

<a id="sumário"></a>

## Sumário

### Como o conteúdo está organizado

| Bloco | Partes | Assuntos centrais | Resultado esperado | Comece por |
|---|---:|---|---|---|
| 1. Fundamentos e modelo relacional | 1–4 | padrão, dialetos, sintaxe, relações, chaves, tipos e `NULL` | ler SQL sem confundir linguagem, produto e ferramenta | [Parte 1](#parte-1--introdução-contexto-e-padrões) |
| 2. Definição e manipulação | 5–8 | DDL, inserção, atualização, remoção, `SELECT` e predicados | criar estruturas e consultar/modificar dados com segurança | [Parte 5](#parte-5--definição-e-evolução-de-schemas) |
| 3. Composição de consultas | 9–14 | joins, agregação, subqueries, CTEs, conjuntos, janelas e funções | expressar consultas analíticas e relacionais corretamente | [Parte 9](#parte-9--junções-e-composição-de-relações) |
| 4. Integridade, transações e desempenho | 15–19 | constraints, views, ACID, isolamento, locks, índices e planos | preservar regras e diagnosticar custo/conflitos | [Parte 15](#parte-15--integridade-constraints-e-regras-no-banco) |
| 5. Segurança, programação e metadados | 20–22 | privilégios, injection, routines, triggers e `INFORMATION_SCHEMA` | construir fronteiras seguras e inspecionáveis | [Parte 20](#parte-20--segurança-autorização-e-sql-injection) |
| 6. Aplicações e arquitetura de dados | 23–25 | drivers, pooling, migrations, modelagem, temporal, OLTP e analytics | integrar SQL a software e escolher modelos adequados | [Parte 23](#parte-23--sql-em-aplicações-e-apis-de-acesso) |
| 7. Operação e produção | 26–27 | backup, HA, observabilidade, testes, performance e change management | operar e evoluir bancos com risco controlado | [Parte 26](#parte-26--operação-backup-recuperação-e-alta-disponibilidade) |
| 8. Catálogo e ecossistema | 28–29 | statements, palavras-chave, funções, SGBDs e ferramentas | descobrir o que já existe e verificar portabilidade | [Parte 28](#parte-28--catálogo-da-linguagem-sql) |
| 9. Revisão | Anexos | fontes oficiais, trilhas e glossário | aprofundar na especificação e nos manuais | [Anexo A](#anexo-a--trilhas-oficiais-de-estudo-e-prática) |

### Atalhos por pergunta prática

| Se você quer saber... | Consulte primeiro |
|---|---|
| como SQL, SGBD, database, schema, driver e cliente se relacionam | [Partes 1](#parte-1--introdução-contexto-e-padrões) e [23](#parte-23--sql-em-aplicações-e-apis-de-acesso) |
| como modelar tabelas, chaves e relacionamentos | [Partes 3](#parte-3--modelo-relacional-tabelas-e-chaves), [15](#parte-15--integridade-constraints-e-regras-no-banco) e [24](#parte-24--modelagem-e-arquitetura-relacional) |
| como `NULL` altera comparações e filtros | [Partes 4](#parte-4--tipos-domínios-null-e-lógica-de-três-valores) e [8](#parte-8--filtros-predicados-e-lógica-ternária) |
| como combinar tabelas sem duplicar ou perder linhas | [Parte 9](#parte-9--junções-e-composição-de-relações) |
| como resumir, ranquear e calcular acumulados | [Partes 10](#parte-10--agregação-e-agrupamento) e [13](#parte-13--funções-de-janela) |
| como usar subqueries e CTEs sem transformar tudo em uma consulta ilegível | [Parte 11](#parte-11--subqueries-ctes-e-recursão) |
| como alterar dados concorrentemente sem corromper regras | [Partes 17](#parte-17--transações-e-acid) e [18](#parte-18--concorrência-isolamento-locks-e-mvcc) |
| por que uma consulta está lenta | [Parte 19](#parte-19--índices-otimizador-e-planos-de-execução) |
| como evitar SQL injection e aplicar menor privilégio | [Parte 20](#parte-20--segurança-autorização-e-sql-injection) |
| como integrar SQL a C#, Java, C++ ou outra aplicação | [Parte 23](#parte-23--sql-em-aplicações-e-apis-de-acesso) |
| qual statement, função, tipo, SGBD ou ferramenta já existe | [Partes 28](#parte-28--catálogo-da-linguagem-sql) e [29](#parte-29--sgbds-ecossistema-e-ferramentas-externas) |

### Índice detalhado

**Bloco 1 — Fundamentos e modelo relacional (Partes 1–4)**

- **[Parte 1 — Introdução, Contexto e Padrões](#parte-1--introdução-contexto-e-padrões)**
  - [1.0 Nomenclatura e estilo](#10-nomenclatura-e-estilo)
  - [1.1 O que é SQL?](#11-o-que-é-sql)
  - [1.2 SQL:2023 e a família ISO/IEC 9075](#12-sql2023-e-a-família-isoiec-9075)
  - [1.3 Linguagem declarativa e processamento em conjuntos](#13-linguagem-declarativa-e-processamento-em-conjuntos)
  - [1.4 SGBD, database, schema, sessão e cliente](#14-sgbd-database-schema-sessão-e-cliente)
  - [1.5 DDL, DML, controle transacional e autorização](#15-ddl-dml-controle-transacional-e-autorização)
  - [1.6 Modelo de exemplo e mapa do iniciante](#16-modelo-de-exemplo-e-mapa-do-iniciante)
- **[Parte 2 — Sintaxe, Identificadores e Portabilidade](#parte-2--sintaxe-identificadores-e-portabilidade)**
  - [2.1 Tokens, comentários e terminadores](#21-tokens-comentários-e-terminadores)
  - [2.2 Identificadores regulares, delimitados e qualificados](#22-identificadores-regulares-delimitados-e-qualificados)
  - [2.3 Literais, parâmetros e casts](#23-literais-parâmetros-e-casts)
  - [2.4 Palavras reservadas e case](#24-palavras-reservadas-e-case)
  - [2.5 Dialetos e matriz de portabilidade](#25-dialetos-e-matriz-de-portabilidade)
- **[Parte 3 — Modelo Relacional, Tabelas e Chaves](#parte-3--modelo-relacional-tabelas-e-chaves)**
  - [3.1 Relação, tabela, linha, coluna e domínio](#31-relação-tabela-linha-coluna-e-domínio)
  - [3.2 Chaves candidatas, primária, natural e substituta](#32-chaves-candidatas-primária-natural-e-substituta)
  - [3.3 Chaves estrangeiras e cardinalidade](#33-chaves-estrangeiras-e-cardinalidade)
  - [3.4 Entidades associativas e relacionamentos N:N](#34-entidades-associativas-e-relacionamentos-nn)
  - [3.5 Integridade e normalização em uma primeira visão](#35-integridade-e-normalização-em-uma-primeira-visão)
- **[Parte 4 — Tipos, Domínios, NULL e Lógica de Três Valores](#parte-4--tipos-domínios-null-e-lógica-de-três-valores)**
  - [4.1 Tipos numéricos e precisão](#41-tipos-numéricos-e-precisão)
  - [4.2 Texto, caracteres e collations](#42-texto-caracteres-e-collations)
  - [4.3 Datas, horas, timestamps e intervalos](#43-datas-horas-timestamps-e-intervalos)
  - [4.4 Booleanos, binários, UUID, JSON e extensões](#44-booleanos-binários-uuid-json-e-extensões)
  - [4.5 `NULL`, `UNKNOWN` e testes corretos](#45-null-unknown-e-testes-corretos)
  - [4.6 Conversão, coerção e domains](#46-conversão-coerção-e-domains)

**Bloco 2 — Definição e manipulação (Partes 5–8)**

- **[Parte 5 — Definição e Evolução de Schemas](#parte-5--definição-e-evolução-de-schemas)**
  - [5.1 `CREATE SCHEMA` e `CREATE TABLE`](#51-create-schema-e-create-table)
  - [5.2 Constraints na definição](#52-constraints-na-definição)
  - [5.3 Identity, sequences e valores gerados](#53-identity-sequences-e-valores-gerados)
  - [5.4 `ALTER`, `DROP` e `TRUNCATE`](#54-alter-drop-e-truncate)
  - [5.5 Tabelas temporárias e objetos auxiliares](#55-tabelas-temporárias-e-objetos-auxiliares)
  - [5.6 Migrations compatíveis e mudanças online](#56-migrations-compatíveis-e-mudanças-online)
- **[Parte 6 — Inserção, Atualização e Remoção de Dados](#parte-6--inserção-atualização-e-remoção-de-dados)**
  - [6.1 `INSERT` e inserção em lote](#61-insert-e-inserção-em-lote)
  - [6.2 `UPDATE` seguro e expressões baseadas no valor atual](#62-update-seguro-e-expressões-baseadas-no-valor-atual)
  - [6.3 `DELETE`, retenção e remoção em cascata](#63-delete-retenção-e-remoção-em-cascata)
  - [6.4 `MERGE` e famílias de upsert](#64-merge-e-famílias-de-upsert)
  - [6.5 Linhas afetadas, `RETURNING` e `OUTPUT`](#65-linhas-afetadas-returning-e-output)
  - [6.6 Operações idempotentes e concorrência](#66-operações-idempotentes-e-concorrência)
- **[Parte 7 — Consultas com `SELECT`](#parte-7--consultas-com-select)**
  - [7.1 Projeção, aliases e expressões](#71-projeção-aliases-e-expressões)
  - [7.2 `DISTINCT` e multiplicidade](#72-distinct-e-multiplicidade)
  - [7.3 Ordem lógica de processamento](#73-ordem-lógica-de-processamento)
  - [7.4 `ORDER BY` e determinismo](#74-order-by-e-determinismo)
  - [7.5 Paginação: `OFFSET/FETCH`, `LIMIT` e keyset](#75-paginação-offsetfetch-limit-e-keyset)
  - [7.6 `CASE` e construção condicional](#76-case-e-construção-condicional)
- **[Parte 8 — Filtros, Predicados e Lógica Ternária](#parte-8--filtros-predicados-e-lógica-ternária)**
  - [8.1 Comparações e operadores lógicos](#81-comparações-e-operadores-lógicos)
  - [8.2 `BETWEEN`, `IN`, `LIKE` e escaping](#82-between-in-like-e-escaping)
  - [8.3 `EXISTS`, quantificadores e semijoins](#83-exists-quantificadores-e-semijoins)
  - [8.4 `NULL`, `NOT IN` e armadilhas](#84-null-not-in-e-armadilhas)
  - [8.5 Predicados sargable e filtros opcionais](#85-predicados-sargable-e-filtros-opcionais)

**Bloco 3 — Composição de consultas (Partes 9–14)**

- **[Parte 9 — Junções e Composição de Relações](#parte-9--junções-e-composição-de-relações)**
  - [9.1 `INNER JOIN`](#91-inner-join)
  - [9.2 `LEFT`, `RIGHT` e `FULL OUTER JOIN`](#92-left-right-e-full-outer-join)
  - [9.3 Predicados em `ON` versus `WHERE`](#93-predicados-em-on-versus-where)
  - [9.4 `CROSS JOIN`, self join e `LATERAL`/`APPLY`](#94-cross-join-self-join-e-lateralapply)
  - [9.5 Cardinalidade, duplicatas e diagnósticos](#95-cardinalidade-duplicatas-e-diagnósticos)
- **[Parte 10 — Agregação e Agrupamento](#parte-10--agregação-e-agrupamento)**
  - [10.1 `COUNT`, `SUM`, `AVG`, `MIN` e `MAX`](#101-count-sum-avg-min-e-max)
  - [10.2 `GROUP BY` e granularidade](#102-group-by-e-granularidade)
  - [10.3 `HAVING` e filtros de grupo](#103-having-e-filtros-de-grupo)
  - [10.4 Agregação condicional e `FILTER`](#104-agregação-condicional-e-filter)
  - [10.5 `ROLLUP`, `CUBE` e `GROUPING SETS`](#105-rollup-cube-e-grouping-sets)
- **[Parte 11 — Subqueries, CTEs e Recursão](#parte-11--subqueries-ctes-e-recursão)**
  - [11.1 Subqueries escalares, tabulares e correlacionadas](#111-subqueries-escalares-tabulares-e-correlacionadas)
  - [11.2 CTEs com `WITH`](#112-ctes-com-with)
  - [11.3 CTE recursiva e condição de parada](#113-cte-recursiva-e-condição-de-parada)
  - [11.4 Hierarquias, grafos e detecção de ciclos](#114-hierarquias-grafos-e-detecção-de-ciclos)
- **[Parte 12 — Operações de Conjuntos](#parte-12--operações-de-conjuntos)**
  - [12.1 `UNION` e `UNION ALL`](#121-union-e-union-all)
  - [12.2 `INTERSECT` e `EXCEPT`](#122-intersect-e-except)
  - [12.3 Compatibilidade, precedência e ordenação final](#123-compatibilidade-precedência-e-ordenação-final)
- **[Parte 13 — Funções de Janela](#parte-13--funções-de-janela)**
  - [13.1 `OVER`, `PARTITION BY` e `ORDER BY`](#131-over-partition-by-e-order-by)
  - [13.2 Ranking: `ROW_NUMBER`, `RANK` e `DENSE_RANK`](#132-ranking-row_number-rank-e-dense_rank)
  - [13.3 `LAG`, `LEAD`, primeiros e últimos valores](#133-lag-lead-primeiros-e-últimos-valores)
  - [13.4 Frames `ROWS`, `RANGE` e `GROUPS`](#134-frames-rows-range-e-groups)
  - [13.5 Padrões analíticos e armadilhas](#135-padrões-analíticos-e-armadilhas)
- **[Parte 14 — Funções, Operadores e Expressões Essenciais](#parte-14--funções-operadores-e-expressões-essenciais)**
  - [14.1 Texto](#141-texto)
  - [14.2 Números e arredondamento](#142-números-e-arredondamento)
  - [14.3 Datas, horas e intervalos](#143-datas-horas-e-intervalos)
  - [14.4 `COALESCE`, `NULLIF` e tratamento de nulos](#144-coalesce-nullif-e-tratamento-de-nulos)
  - [14.5 JSON, XML, arrays, regex e full-text](#145-json-xml-arrays-regex-e-full-text)
  - [14.6 Funções determinísticas, volatilidade e collation](#146-funções-determinísticas-volatilidade-e-collation)

**Bloco 4 — Integridade, transações e desempenho (Partes 15–19)**

- **[Parte 15 — Integridade, Constraints e Regras no Banco](#parte-15--integridade-constraints-e-regras-no-banco)**
  - [15.1 `NOT NULL`, `CHECK` e `DEFAULT`](#151-not-null-check-e-default)
  - [15.2 `PRIMARY KEY` e `UNIQUE`](#152-primary-key-e-unique)
  - [15.3 `FOREIGN KEY` e ações referenciais](#153-foreign-key-e-ações-referenciais)
  - [15.4 Constraints diferíveis e validação tardia](#154-constraints-diferíveis-e-validação-tardia)
  - [15.5 Regras que exigem trigger ou transação](#155-regras-que-exigem-trigger-ou-transação)
- **[Parte 16 — Views e Abstrações de Consulta](#parte-16--views-e-abstrações-de-consulta)**
  - [16.1 Views comuns](#161-views-comuns)
  - [16.2 Views atualizáveis e `WITH CHECK OPTION`](#162-views-atualizáveis-e-with-check-option)
  - [16.3 Materialized views e indexed views](#163-materialized-views-e-indexed-views)
  - [16.4 Segurança, contratos e dependências](#164-segurança-contratos-e-dependências)
- **[Parte 17 — Transações e ACID](#parte-17--transações-e-acid)**
  - [17.1 Atomicidade, consistência, isolamento e durabilidade](#171-atomicidade-consistência-isolamento-e-durabilidade)
  - [17.2 `BEGIN`, `COMMIT` e `ROLLBACK`](#172-begin-commit-e-rollback)
  - [17.3 Savepoints](#173-savepoints)
  - [17.4 Autocommit, erros e fronteiras](#174-autocommit-erros-e-fronteiras)
  - [17.5 Transações distribuídas e sagas](#175-transações-distribuídas-e-sagas)
- **[Parte 18 — Concorrência, Isolamento, Locks e MVCC](#parte-18--concorrência-isolamento-locks-e-mvcc)**
  - [18.1 Anomalias de concorrência](#181-anomalias-de-concorrência)
  - [18.2 Níveis de isolamento](#182-níveis-de-isolamento)
  - [18.3 Locks e leitura para atualização](#183-locks-e-leitura-para-atualização)
  - [18.4 MVCC e versões de linha](#184-mvcc-e-versões-de-linha)
  - [18.5 Deadlocks, serialization failures e retry](#185-deadlocks-serialization-failures-e-retry)
  - [18.6 Concorrência otimista e version columns](#186-concorrência-otimista-e-version-columns)
- **[Parte 19 — Índices, Otimizador e Planos de Execução](#parte-19--índices-otimizador-e-planos-de-execução)**
  - [19.1 O que um índice oferece](#191-o-que-um-índice-oferece)
  - [19.2 B-tree, hash, bitmap, inverted e spatial](#192-b-tree-hash-bitmap-inverted-e-spatial)
  - [19.3 Índices compostos, covering, parciais e por expressão](#193-índices-compostos-covering-parciais-e-por-expressão)
  - [19.4 Estatísticas, cardinalidade e custo](#194-estatísticas-cardinalidade-e-custo)
  - [19.5 `EXPLAIN` e plano real](#195-explain-e-plano-real)
  - [19.6 Antipadrões de consulta e tuning orientado por evidência](#196-antipadrões-de-consulta-e-tuning-orientado-por-evidência)

**Bloco 5 — Segurança, programação e metadados (Partes 20–22)**

- **[Parte 20 — Segurança, Autorização e SQL Injection](#parte-20--segurança-autorização-e-sql-injection)**
  - [20.1 Usuários, roles e privilégios](#201-usuários-roles-e-privilégios)
  - [20.2 `GRANT`, `REVOKE` e menor privilégio](#202-grant-revoke-e-menor-privilégio)
  - [20.3 SQL injection e parâmetros](#203-sql-injection-e-parâmetros)
  - [20.4 Identificadores dinâmicos e allowlists](#204-identificadores-dinâmicos-e-allowlists)
  - [20.5 Row-level security, views e masking](#205-row-level-security-views-e-masking)
  - [20.6 Criptografia, segredos e auditoria](#206-criptografia-segredos-e-auditoria)
- **[Parte 21 — Procedures, Functions, Triggers e SQL Procedural](#parte-21--procedures-functions-triggers-e-sql-procedural)**
  - [21.1 SQL/PSM e dialetos procedurais](#211-sqlpsm-e-dialetos-procedurais)
  - [21.2 Procedures e functions](#212-procedures-e-functions)
  - [21.3 Variáveis, controle de fluxo e exceptions](#213-variáveis-controle-de-fluxo-e-exceptions)
  - [21.4 Triggers](#214-triggers)
  - [21.5 Cursors e processamento linha a linha](#215-cursors-e-processamento-linha-a-linha)
  - [21.6 SQL dinâmico](#216-sql-dinâmico)
- **[Parte 22 — Metadados, Catálogos e Diagnóstico](#parte-22--metadados-catálogos-e-diagnóstico)**
  - [22.1 `INFORMATION_SCHEMA`](#221-information_schema)
  - [22.2 Catálogos específicos](#222-catálogos-específicos)
  - [22.3 Constraints, dependências e linhagem](#223-constraints-dependências-e-linhagem)
  - [22.4 Sessões, statements ativos e locks](#224-sessões-statements-ativos-e-locks)
  - [22.5 SQLSTATE, warnings e diagnostics](#225-sqlstate-warnings-e-diagnostics)
- [Checkpoint — Fundamentos de SQL](#checkpoint--fundamentos-de-sql-partes-122)

**Bloco 6 — Aplicações e arquitetura de dados (Partes 23–25)**

- **[Parte 23 — SQL em Aplicações e APIs de Acesso](#parte-23--sql-em-aplicações-e-apis-de-acesso)**
  - [23.1 ODBC, JDBC, ADO.NET e drivers nativos](#231-odbc-jdbc-adonet-e-drivers-nativos)
  - [23.2 Prepared statements e binding](#232-prepared-statements-e-binding)
  - [23.3 Connections, pooling e transações](#233-connections-pooling-e-transações)
  - [23.4 Result sets, streaming e tipos](#234-result-sets-streaming-e-tipos)
  - [23.5 ORMs, query builders e N+1](#235-orms-query-builders-e-n1)
  - [23.6 Migrations e compatibilidade entre aplicação e schema](#236-migrations-e-compatibilidade-entre-aplicação-e-schema)
- **[Parte 24 — Modelagem e Arquitetura Relacional](#parte-24--modelagem-e-arquitetura-relacional)**
  - [24.1 Dependências funcionais e formas normais](#241-dependências-funcionais-e-formas-normais)
  - [24.2 Desnormalização consciente](#242-desnormalização-consciente)
  - [24.3 Agregados, invariantes e fronteiras transacionais](#243-agregados-invariantes-e-fronteiras-transacionais)
  - [24.4 Dados temporais, histórico e soft delete](#244-dados-temporais-histórico-e-soft-delete)
  - [24.5 Multitenancy](#245-multitenancy)
  - [24.6 IDs, clocks e distribuição](#246-ids-clocks-e-distribuição)
- **[Parte 25 — Analytics, Warehousing e Arquiteturas de Dados](#parte-25--analytics-warehousing-e-arquiteturas-de-dados)**
  - [25.1 OLTP versus OLAP](#251-oltp-versus-olap)
  - [25.2 Modelagem dimensional](#252-modelagem-dimensional)
  - [25.3 Particionamento e pruning](#253-particionamento-e-pruning)
  - [25.4 Replicação, CDC e event streaming](#254-replicação-cdc-e-event-streaming)
  - [25.5 Bancos distribuídos e consistência](#255-bancos-distribuídos-e-consistência)
  - [25.6 SQL para grafos e SQL/PGQ](#256-sql-para-grafos-e-sqlpgq)

**Bloco 7 — Operação e produção (Partes 26–27)**

- **[Parte 26 — Operação, Backup, Recuperação e Alta Disponibilidade](#parte-26--operação-backup-recuperação-e-alta-disponibilidade)**
  - [26.1 Backup lógico e físico](#261-backup-lógico-e-físico)
  - [26.2 Restore, PITR, RPO e RTO](#262-restore-pitr-rpo-e-rto)
  - [26.3 Replicação e failover](#263-replicação-e-failover)
  - [26.4 Maintenance, vacuum, statistics e storage](#264-maintenance-vacuum-statistics-e-storage)
  - [26.5 Observabilidade e capacity planning](#265-observabilidade-e-capacity-planning)
- **[Parte 27 — Engenharia de SQL para Produção](#parte-27--engenharia-de-sql-para-produção)**
  - [27.1 Testes de schema, queries e migrations](#271-testes-de-schema-queries-e-migrations)
  - [27.2 Revisão, lint e formatação](#272-revisão-lint-e-formatação)
  - [27.3 Performance e regressão de planos](#273-performance-e-regressão-de-planos)
  - [27.4 Privacidade, retenção e compliance](#274-privacidade-retenção-e-compliance)
  - [27.5 Deploy progressivo e rollback](#275-deploy-progressivo-e-rollback)
  - [27.6 Runbooks e resposta a incidentes](#276-runbooks-e-resposta-a-incidentes)

**Bloco 8 — Catálogo e ecossistema (Partes 28–29)**

- **[Parte 28 — Catálogo da Linguagem SQL](#parte-28--catálogo-da-linguagem-sql)**
  - [28.1 Categorias de statements](#281-categorias-de-statements)
  - [28.2 Palavras-chave e por que não há lista universal única](#282-palavras-chave-e-por-que-não-há-lista-universal-única)
  - [28.3 Tipos padronizados e famílias de tipos](#283-tipos-padronizados-e-famílias-de-tipos)
  - [28.4 Operadores e predicados](#284-operadores-e-predicados)
  - [28.5 Funções prontas por domínio](#285-funções-prontas-por-domínio)
  - [28.6 Objetos de schema](#286-objetos-de-schema)
  - [28.7 Recursos analíticos e relacionais](#287-recursos-analíticos-e-relacionais)
  - [28.8 Metadados e diagnóstico](#288-metadados-e-diagnóstico)
  - [28.9 Matriz rápida de dialetos](#289-matriz-rápida-de-dialetos)
  - [28.10 Como verificar disponibilidade e semântica](#2810-como-verificar-disponibilidade-e-semântica)
- **[Parte 29 — SGBDs, Ecossistema e Ferramentas Externas](#parte-29--sgbds-ecossistema-e-ferramentas-externas)**
  - [29.1 Padrão, produto, serviço e ferramenta](#291-padrão-produto-serviço-e-ferramenta)
  - [29.2 SGBDs relacionais gerais](#292-sgbds-relacionais-gerais)
  - [29.3 Bancos embarcados](#293-bancos-embarcados)
  - [29.4 Cloud data warehouses e lakehouses](#294-cloud-data-warehouses-e-lakehouses)
  - [29.5 Distributed SQL e NewSQL](#295-distributed-sql-e-newsql)
  - [29.6 Clientes e IDEs](#296-clientes-e-ides)
  - [29.7 Migration, transformação e qualidade](#297-migration-transformação-e-qualidade)
  - [29.8 Observabilidade e segurança](#298-observabilidade-e-segurança)
  - [29.9 Como escolher e adotar](#299-como-escolher-e-adotar)

**Anexos e consulta rápida**

- [Anexo A — Trilhas Oficiais de Estudo e Prática](#anexo-a--trilhas-oficiais-de-estudo-e-prática)
- [Anexo B — Referências Oficiais Consultadas](#anexo-b--referências-oficiais-consultadas)
- [Glossário](#glossário)

---

## Parte 1 — Introdução, Contexto e Padrões

[⬆️ Voltar ao Sumário](#sumário)

Esta parte separa a linguagem SQL do produto que a executa e estabelece o vocabulário usado no restante do guia.

---

### 1.0 Nomenclatura e estilo

[⬆️ Voltar ao Sumário](#sumário)

O padrão define como identificadores são reconhecidos, mas não impõe a convenção visual de uma equipe. Um estilo consistente reduz quoting, ambiguidades e diferenças entre sistemas.

| Elemento | Convenção prática deste guia | Motivo |
|---|---|---|
| palavras-chave | `SELECT`, `FROM`, `WHERE` | separa linguagem de nomes do domínio |
| tabelas e colunas | `snake_case` sem aspas | evita depender de case preservado |
| constraints | `pk_pedido`, `fk_pedido_pessoa` | facilita diagnóstico e migrations |
| aliases | curtos, mas sem mistério: `p`, `ip` | consultas permanecem legíveis |
| parâmetros | `:pessoa_id` na documentação | diferencia valor externo de literal |

```sql
SELECT p.id,
       p.nome,
       p.criado_em
FROM pessoa AS p
WHERE p.ativo = 1
ORDER BY p.nome, p.id;
```

**Leitura guiada:** `p` qualifica as colunas e evita ambiguidade quando outra tabela for adicionada. O `id` no final do `ORDER BY` desempata nomes iguais. O valor `1` é didático; produtos com tipo booleano podem usar `TRUE`.

Evite palavras reservadas, caracteres especiais e identificadores que só funcionam quando delimitados. Singular versus plural é decisão do projeto; consistência importa mais que preferência.

> **Fontes oficiais:** [ISO/IEC 9075-2:2023 — SQL/Foundation](https://www.iso.org/standard/76584.html), [PostgreSQL — Lexical Structure](https://www.postgresql.org/docs/current/sql-syntax-lexical.html)

---

### 1.1 O que é SQL?

[⬆️ Voltar ao Sumário](#sumário)

SQL é a linguagem padronizada para definir, consultar, modificar e controlar dados em implementações SQL. Seu núcleo é declarativo: a consulta especifica relações e condições; o SGBD escolhe um plano físico que preserve a semântica.

SQL reúne mais que consultas:

- definição de schemas, tabelas, views e constraints;
- consulta e transformação de conjuntos de linhas;
- inserção, atualização e remoção;
- controle de transações e sessões;
- autorização e metadados;
- módulos armazenados e extensões previstas em outras partes da família 9075.

SQL não é uma linguagem de propósito geral e não define sozinho arquivos de dados, rede, processo servidor, replicação ou formato de backup. Esses contratos pertencem à implementação e à plataforma.

> **Fontes oficiais:** [ISO/IEC 9075-1:2023 — escopo e framework](https://www.iso.org/standard/76583.html), [ISO/IEC 9075-2:2023 — Foundation](https://www.iso.org/standard/76584.html)

---

### 1.2 SQL:2023 e a família ISO/IEC 9075

[⬆️ Voltar ao Sumário](#sumário)

Em julho de 2026, a edição internacional publicada é SQL:2023. “SQL:2023” não é um executável nem uma versão que o servidor habilita com uma flag; é uma família de documentos normativos.

| Parte | Papel resumido |
|---|---|
| 9075-1 | framework, termos e organização |
| 9075-2 | SQL/Foundation, núcleo amplo da linguagem |
| 9075-3 | Call-Level Interface — SQL/CLI |
| 9075-4 | Persistent Stored Modules — SQL/PSM |
| 9075-9 | Management of External Data — SQL/MED |
| 9075-10 | Object Language Bindings |
| 9075-11 | Information and Definition Schemas |
| 9075-13 | Routines and Types Using Java |
| 9075-14 | XML-Related Specifications |
| 9075-15 | Multidimensional Arrays |
| 9075-16 | Property Graph Queries — SQL/PGQ |

Uma feature aparecer no padrão não significa que todos os produtos a implementem, nem que usem sintaxe idêntica. Consulte primeiro o padrão para semântica e depois o manual da versão do produto para disponibilidade.

> **Fontes oficiais:** [ISO SQL/Framework](https://www.iso.org/standard/76583.html), [SQL/Foundation](https://www.iso.org/standard/76584.html), [SQL/PSM](https://www.iso.org/standard/76585.html), [SQL/Schemata](https://www.iso.org/standard/76586.html), [SQL/PGQ](https://www.iso.org/standard/79473.html)

---

### 1.3 Linguagem declarativa e processamento em conjuntos

[⬆️ Voltar ao Sumário](#sumário)

Em código imperativo, é comum pensar “abra cada linha e faça X”. Em SQL, formule “qual relação resulta destas fontes, condições e expressões?”. O otimizador pode trocar join order, escolher índice, paralelizar ou transformar subqueries desde que preserve o resultado permitido.

```sql
SELECT departamento_id,
       COUNT(*) AS quantidade_pessoas
FROM pessoa
WHERE ativo = 1
GROUP BY departamento_id;
```

**Leitura guiada:** a consulta não descreve um loop. `WHERE` restringe linhas, `GROUP BY` forma um grupo por departamento e `COUNT(*)` conta linhas de cada grupo. A ordem física de leitura não faz parte do resultado.

Set-based não significa que duplicatas desaparecem. Resultados SQL normalmente têm semântica de **bag**: linhas equivalentes podem aparecer várias vezes até que uma operação como `DISTINCT`, uma chave ou uma operação de conjunto as controle.

> **Fonte oficial:** [PostgreSQL — Queries](https://www.postgresql.org/docs/current/queries.html), [ISO/IEC 9075-2:2023](https://www.iso.org/standard/76584.html)

---

### 1.4 SGBD, database, schema, sessão e cliente

[⬆️ Voltar ao Sumário](#sumário)

| Termo | Responsabilidade |
|---|---|
| SGBD/DBMS | implementação que armazena, executa e coordena operações |
| database/catalog | conjunto nomeado de schemas e dados, conforme o produto |
| schema | namespace de objetos SQL e fronteira de organização/autorização |
| tabela/view | objetos relacionais consultáveis |
| sessão/conexão | contexto de usuário, transação e configurações |
| driver | API que envia statements, parâmetros e recebe resultados |
| cliente | `psql`, `sqlcmd`, aplicação, IDE ou ferramenta administrativa |

```text
aplicação/cliente
      ↓ protocolo + driver
sessão no SGBD
      ↓ parse → bind/resolve → optimize → execute
catálogo, dados, índices, locks/versões e log
```

**Leitura guiada:** o cliente não acessa arquivos diretamente: o driver envia uma instrução à sessão, e o SGBD valida nomes/tipos, escolhe um plano e coordena storage e concorrência.

Um ponto e vírgula pode separar statements no cliente sem ser o protocolo de rede. Um comando de cliente como `\d` no `psql`, `.schema` no `sqlite3` ou `GO` em ferramentas SQL Server não é automaticamente SQL padronizado.

> **Fontes oficiais:** [PostgreSQL — Architecture](https://www.postgresql.org/docs/current/tutorial-arch.html), [SQL Server — Database Engine](https://learn.microsoft.com/en-us/sql/database-engine/sql-database-engine), [SQLite — Command-Line Shell](https://sqlite.org/cli.html)

---

### 1.5 DDL, DML, controle transacional e autorização

[⬆️ Voltar ao Sumário](#sumário)

As siglas abaixo são didáticas e variam entre materiais; não trate “DQL” como uma divisão normativa universal.

| Grupo informal | Exemplos | Intenção |
|---|---|---|
| DDL | `CREATE`, `ALTER`, `DROP` | definir objetos e regras |
| consulta | `SELECT`, `VALUES`, set operations | construir resultados |
| DML | `INSERT`, `UPDATE`, `DELETE`, `MERGE` | modificar dados |
| TCL | `START TRANSACTION`, `COMMIT`, `ROLLBACK` | controlar unidade de trabalho |
| DCL | `GRANT`, `REVOKE` | controlar privilégios |

O comportamento transacional de DDL varia: PostgreSQL aceita muitas mudanças de schema transacionais; MySQL documenta statements que causam implicit commit; outros produtos têm regras próprias. Nunca deduza rollback de DDL por hábito de outro banco.

> **Fontes oficiais:** [PostgreSQL — SQL Commands](https://www.postgresql.org/docs/current/sql-commands.html), [MySQL 8.4 — Statements That Cause an Implicit Commit](https://dev.mysql.com/doc/refman/8.4/en/implicit-commit.html), [SQL Server — Transactions](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/transactions-transact-sql)

---

### 1.6 Modelo de exemplo e mapa do iniciante

[⬆️ Voltar ao Sumário](#sumário)

Grande parte dos exemplos usa este domínio reduzido:

```sql
CREATE TABLE departamento (
    id          INTEGER     NOT NULL,
    nome        VARCHAR(80) NOT NULL,
    CONSTRAINT pk_departamento PRIMARY KEY (id),
    CONSTRAINT uq_departamento_nome UNIQUE (nome)
);

CREATE TABLE pessoa (
    id               INTEGER        NOT NULL,
    departamento_id  INTEGER,
    nome             VARCHAR(120)   NOT NULL,
    email            VARCHAR(254)   NOT NULL,
    salario          DECIMAL(12, 2),
    ativo            SMALLINT       DEFAULT 1 NOT NULL,
    criado_em        TIMESTAMP      DEFAULT CURRENT_TIMESTAMP NOT NULL,
    CONSTRAINT pk_pessoa PRIMARY KEY (id),
    CONSTRAINT uq_pessoa_email UNIQUE (email),
    CONSTRAINT ck_pessoa_ativo CHECK (ativo IN (0, 1)),
    CONSTRAINT fk_pessoa_departamento
        FOREIGN KEY (departamento_id) REFERENCES departamento (id)
);
```

**Leitura guiada:** a chave primária identifica cada linha; `email` é uma chave alternativa; `departamento_id` pode ser `NULL`, portanto o relacionamento é opcional; `CHECK` limita `ativo`; defaults são aplicados quando a coluna é omitida, não quando se envia `NULL` explicitamente.

Partes posteriores introduzem tabelas auxiliares pequenas — como `produto`, `pedido`, `item_pedido`, `medicao`, `documento` e `participante_projeto` — no próprio contexto. Cada leitura guiada informa as colunas relevantes; esses trechos são exemplos focados, não um único script cumulativo e portável entre todos os dialetos.

O roteiro mental do iniciante é: schema → constraints → dados de teste → `SELECT` simples → filtros → joins → agregação → transações → planos. Aprender tuning antes de entender cardinalidade apenas troca erro lógico por superstição de performance.

> **Fontes oficiais:** [ISO/IEC 9075-2:2023](https://www.iso.org/standard/76584.html), [PostgreSQL — Data Definition](https://www.postgresql.org/docs/current/ddl.html)

---

## Parte 2 — Sintaxe, Identificadores e Portabilidade

[⬆️ Voltar ao Sumário](#sumário)

SQL possui estrutura lexical própria. Quoting, case, parâmetros e terminadores são fontes frequentes de falsa portabilidade.

---

### 2.1 Tokens, comentários e terminadores

[⬆️ Voltar ao Sumário](#sumário)

```sql
-- Comentário até o fim da linha.
SELECT id, nome
FROM pessoa
WHERE ativo = 1; /* comentário em bloco */
```

**Leitura guiada:** keywords, identificadores, vírgulas, operadores e literais formam tokens. O ponto e vírgula termina o statement no texto enviado pelo cliente; algumas APIs aceitam um statement sem ele. Comentário inserido no lugar errado ainda pode separar tokens e alterar parsing.

Não copie delimitadores de batch do cliente para migrations sem confirmar: `GO`, `/`, `DELIMITER` e meta-comandos iniciados por barra ou ponto pertencem a ferramentas/dialetos específicos.

> **Fontes oficiais:** [ISO SQL/Foundation](https://www.iso.org/standard/76584.html), [PostgreSQL — Lexical Structure](https://www.postgresql.org/docs/current/sql-syntax-lexical.html), [MySQL — Comments](https://dev.mysql.com/doc/refman/8.4/en/comments.html)

---

### 2.2 Identificadores regulares, delimitados e qualificados

[⬆️ Voltar ao Sumário](#sumário)

O delimitador padrão de identificador é aspas duplas. Aspas simples delimitam strings.

```sql
SELECT p.nome,
       d.nome AS nome_departamento,
       p."coluna Com Espaço"
FROM cadastro.pessoa AS p
JOIN cadastro.departamento AS d ON d.id = p.departamento_id;
```

**Leitura guiada:** `cadastro.pessoa` qualifica objeto pelo schema; `p.nome` qualifica coluna pelo alias. `"coluna Com Espaço"` preserva case e espaço e deverá ser citado consistentemente. O exemplo pressupõe que essa coluna incomum exista; o guia recomenda não criá-la.

MySQL usa backticks por padrão e SQL Server aceita brackets, mas ambos possuem modos/opções e suporte a aspas duplas em contextos determinados. Prefira nomes regulares portáveis no modelo compartilhado.

> **Fontes oficiais:** [PostgreSQL — Identifiers and Key Words](https://www.postgresql.org/docs/current/sql-syntax-lexical.html), [MySQL — Schema Object Names](https://dev.mysql.com/doc/refman/8.4/en/identifiers.html), [SQL Server — Database Identifiers](https://learn.microsoft.com/en-us/sql/relational-databases/databases/database-identifiers)

---

### 2.3 Literais, parâmetros e casts

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT CAST('2026-07-21' AS DATE) AS data_referencia,
       CAST('19.90' AS DECIMAL(10, 2)) AS valor,
       'D''Ávila' AS sobrenome;
```

**Leitura guiada:** duas aspas simples representam uma aspas dentro do literal. `CAST` declara o tipo desejado e é mais portável que operadores de cast do dialeto. Formato de data aceito e coerções ainda devem ser conferidos na implementação.

Valores externos não devem ser interpolados. Use placeholders e binding do driver: `?`, `$1`, `:nome` ou `@nome` variam conforme API/produto. Parâmetro representa valor, não nome de tabela, coluna ou keyword.

> **Fontes oficiais:** [PostgreSQL — Value Expressions](https://www.postgresql.org/docs/current/sql-expressions.html), [SQLite — SQL Parameters](https://sqlite.org/lang_expr.html#parameters), [SQL Server — Data Type Conversion](https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-type-conversion-database-engine)

---

### 2.4 Palavras reservadas e case

[⬆️ Voltar ao Sumário](#sumário)

SQL possui keywords reservadas e não reservadas, e a classificação muda por edição e produto. `SELECT`, `FROM`, `WHERE`, `JOIN`, `GROUP`, `ORDER`, `USER` e `CURRENT_DATE` são exemplos de nomes arriscados para objetos.

Identificadores regulares são case-insensitive segundo regras da implementação; identificadores delimitados preservam distinções definidas pelo produto. Dados textuais obedecem ao tipo e à collation, não à regra dos nomes.

Nunca dependa de “funcionou sem aspas” como prova de portabilidade. Consulte a lista de keywords da versão e automatize lint de migrations.

> **Fontes oficiais:** [PostgreSQL — SQL Key Words](https://www.postgresql.org/docs/current/sql-keywords-appendix.html), [MySQL 8.4 — Keywords and Reserved Words](https://dev.mysql.com/doc/refman/8.4/en/keywords.html), [SQLite — Keywords](https://sqlite.org/lang_keywords.html)

---

### 2.5 Dialetos e matriz de portabilidade

[⬆️ Voltar ao Sumário](#sumário)

| Intenção | Forma padrão/portável | Extensões comuns |
|---|---|---|
| limitar linhas | `FETCH FIRST n ROWS ONLY` | `LIMIT`, `TOP` |
| coluna gerada de identidade | `GENERATED ... AS IDENTITY` | `AUTO_INCREMENT`, propriedades `IDENTITY` |
| devolver linhas alteradas | depende do recurso suportado | `RETURNING`, `OUTPUT` |
| upsert | `MERGE` quando implementado com contrato adequado | `ON CONFLICT`, `ON DUPLICATE KEY` |
| concatenar texto | `\|\|` | `CONCAT`, `+` em contextos específicos |
| data atual | `CURRENT_DATE` | funções próprias |
| booleano | `BOOLEAN`, `TRUE`, `FALSE` | `BIT`, números, regras de affinity |

Até uma sintaxe compartilhada pode ter semântica distinta para `NULL`, collation, overflow, DDL transacional, time zone ou concorrência. Portabilidade exige testes na matriz real, não apenas parser compatível.

> **Fontes oficiais:** [PostgreSQL SQL Language](https://www.postgresql.org/docs/current/sql.html), [SQL Server T-SQL Reference](https://learn.microsoft.com/en-us/sql/t-sql/language-reference), [MySQL 8.4 Reference](https://dev.mysql.com/doc/refman/8.4/en/), [Oracle AI Database SQL Reference](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/), [SQLite SQL](https://sqlite.org/lang.html)

---

## Parte 3 — Modelo Relacional, Tabelas e Chaves

[⬆️ Voltar ao Sumário](#sumário)

Um schema útil não é uma coleção de planilhas: chaves, domínios e constraints tornam relações e regras verificáveis pelo banco.

---

### 3.1 Relação, tabela, linha, coluna e domínio

[⬆️ Voltar ao Sumário](#sumário)

No modelo relacional, uma relação é formada por atributos definidos sobre domínios e por tuples. Uma tabela SQL é a representação operacional mais próxima, mas admite características como `NULL` e duplicatas em resultados que exigem cuidado conceitual.

| Conceito | Pergunta de projeto |
|---|---|
| coluna/atributo | qual fato atômico e qual tipo? |
| linha/tuple | que ocorrência do domínio representa? |
| domínio | quais valores são válidos? |
| chave | como a ocorrência é identificada? |
| constraint | qual regra sempre deve valer? |

Cada coluna deve representar um fato na granularidade declarada. Guardar lista separada por vírgula em uma coluna transforma relação em parsing de texto e enfraquece integridade, joins e índices.

> **Fonte oficial:** [ISO/IEC 9075-1:2023](https://www.iso.org/standard/76583.html), [PostgreSQL — Table Basics](https://www.postgresql.org/docs/current/ddl-basics.html)

---

### 3.2 Chaves candidatas, primária, natural e substituta

[⬆️ Voltar ao Sumário](#sumário)

Uma chave candidata identifica unicamente uma linha e é mínima. A primary key é a candidata escolhida para identificação principal; outras continuam protegidas por `UNIQUE` quando representam identidade do domínio.

```sql
CREATE TABLE pais (
    id       INTEGER     NOT NULL,
    codigo   CHAR(2)     NOT NULL,
    nome     VARCHAR(80) NOT NULL,
    CONSTRAINT pk_pais PRIMARY KEY (id),
    CONSTRAINT uq_pais_codigo UNIQUE (codigo)
);
```

**Leitura guiada:** `id` é chave substituta; `codigo` continua sendo chave natural e não pode perder a constraint. Sem `UNIQUE`, duas linhas poderiam representar o mesmo país mesmo que o ID técnico fosse diferente.

Chave substituta não corrige ausência de regra de negócio. Escolha tipo e estratégia de geração considerando coordenação, tamanho de índices, exposição e distribuição.

> **Fonte oficial:** [PostgreSQL — Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html), [SQL Server — Primary and Foreign Key Constraints](https://learn.microsoft.com/en-us/sql/relational-databases/tables/primary-and-foreign-key-constraints)

---

### 3.3 Chaves estrangeiras e cardinalidade

[⬆️ Voltar ao Sumário](#sumário)

Foreign key exige que valores não nulos da coluna filha encontrem chave compatível na relação referenciada. Ela protege existência; não carrega automaticamente regras de quantidade além do que outras constraints expressam.

| Relação | Modelagem comum |
|---|---|
| 1:N obrigatório no lado N | FK `NOT NULL` na tabela N |
| 1:N opcional | FK nullable na tabela N |
| 1:1 | FK + `UNIQUE`, com nulabilidade conforme participação |
| N:N | tabela associativa com duas FKs |

Índice na chave referenciada é necessário para unicidade. Índice na FK filha frequentemente ajuda joins e ações referenciais, mas a criação automática depende do produto.

> **Fontes oficiais:** [PostgreSQL — Foreign Keys](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK), [MySQL — InnoDB Foreign Key Constraints](https://dev.mysql.com/doc/refman/8.4/en/innodb-foreign-key-constraints.html)

---

### 3.4 Entidades associativas e relacionamentos N:N

[⬆️ Voltar ao Sumário](#sumário)

```sql
CREATE TABLE pessoa_projeto (
    pessoa_id  INTEGER       NOT NULL,
    projeto_id INTEGER       NOT NULL,
    papel      VARCHAR(40)   NOT NULL,
    alocacao   DECIMAL(5, 2) NOT NULL,
    CONSTRAINT pk_pessoa_projeto PRIMARY KEY (pessoa_id, projeto_id),
    CONSTRAINT ck_alocacao CHECK (alocacao > 0 AND alocacao <= 100),
    CONSTRAINT fk_pp_pessoa FOREIGN KEY (pessoa_id) REFERENCES pessoa (id),
    CONSTRAINT fk_pp_projeto FOREIGN KEY (projeto_id) REFERENCES projeto (id)
);
```

**Leitura guiada:** a chave composta impede repetir a mesma pessoa no mesmo projeto. `papel` e `alocacao` pertencem ao relacionamento, portanto ficam na entidade associativa. O exemplo pressupõe a tabela `projeto` já criada.

Se uma pessoa puder exercer vários papéis no mesmo projeto, `papel` talvez participe da chave; a decisão vem da regra, não da conveniência do ORM.

> **Fonte oficial:** [PostgreSQL — Multicolumn Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)

---

### 3.5 Integridade e normalização em uma primeira visão

[⬆️ Voltar ao Sumário](#sumário)

Normalização reduz anomalias ao fazer cada fato depender da chave apropriada. Uma primeira leitura:

| Forma | Ideia operacional simplificada |
|---|---|
| 1NF | valores no domínio esperado, sem grupos repetidos em uma coluna |
| 2NF | atributo não chave depende da chave inteira |
| 3NF | atributo não chave não depende transitivamente de outro não chave |
| BCNF | todo determinante relevante é superchave |

Não normalize por memorização de siglas. Identifique dependências funcionais e as anomalias: inserir, atualizar ou remover um fato não deveria exigir alterar cópias não controladas.

Desnormalização é duplicação deliberada com dono, sincronização e medição; não é ausência acidental de modelagem.

> **Fonte oficial prática:** [PostgreSQL — Data Definition](https://www.postgresql.org/docs/current/ddl.html), [Microsoft — Database Normalization Description](https://learn.microsoft.com/en-us/office/troubleshoot/access/database-normalization-description)

---

## Parte 4 — Tipos, Domínios, NULL e Lógica de Três Valores

[⬆️ Voltar ao Sumário](#sumário)

Tipo não serve apenas para storage: participa de validação, comparação, arredondamento, protocolo do driver e escolha de índices.

---

### 4.1 Tipos numéricos e precisão

[⬆️ Voltar ao Sumário](#sumário)

| Família | Uso | Cuidado |
|---|---|---|
| `SMALLINT`, `INTEGER`, `BIGINT` | contagens e inteiros | faixa varia pelo tipo; overflow depende do produto |
| `DECIMAL(p,s)`/`NUMERIC` | valor decimal exato | defina precisão e escala de domínio |
| `REAL`, `DOUBLE PRECISION`, `FLOAT` | aproximação científica | igualdade e arredondamento binário |

```sql
CREATE TABLE pagamento (
    id      BIGINT         NOT NULL PRIMARY KEY,
    valor   DECIMAL(19, 4) NOT NULL,
    taxa    DECIMAL(9, 6)  NOT NULL,
    CHECK (valor >= 0),
    CHECK (taxa >= 0)
);
```

**Leitura guiada:** dinheiro permanece decimal exato; a escala quatro permite subunidades definidas pelo domínio. O tipo não impede negativos sozinho, por isso há `CHECK`. Escolha escala a partir da regra contábil e documente arredondamento.

Não use floating point para valores que exigem igualdade decimal exata. Não escolha `BIGINT` para todo campo sem considerar driver, índice e semântica.

> **Fontes oficiais:** [PostgreSQL — Numeric Types](https://www.postgresql.org/docs/current/datatype-numeric.html), [MySQL — Numeric Data Types](https://dev.mysql.com/doc/refman/8.4/en/numeric-types.html), [SQL Server — Data Types](https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-types-transact-sql)

---

### 4.2 Texto, caracteres e collations

[⬆️ Voltar ao Sumário](#sumário)

`CHAR(n)` representa comprimento fixo segundo regras do produto; `VARCHAR(n)` limita comprimento; `CHARACTER LARGE OBJECT`/CLOB atende textos grandes quando suportado. Bytes, caracteres, code points e grapheme clusters não são sinônimos.

Collation controla comparação e ordenação: case, acentos, equivalências linguísticas e versão da biblioteca podem alterar unicidade e resultados.

```sql
SELECT nome
FROM pessoa
WHERE nome >= 'A' AND nome < 'B'
ORDER BY nome;
```

**Leitura guiada:** o intervalo depende da collation. Ele não é uma definição universal de “nomes que começam por A” em todos os idiomas. Índices também são construídos segundo operator class/collation da implementação.

Defina encoding e collation no nível correto e teste dados reais. Alterar collation pode exigir reconstruir índices ou revisar constraints de unicidade.

> **Fontes oficiais:** [PostgreSQL — Character Types](https://www.postgresql.org/docs/current/datatype-character.html), [PostgreSQL — Collation Support](https://www.postgresql.org/docs/current/collation.html), [MySQL — Character Sets and Collations](https://dev.mysql.com/doc/refman/8.4/en/charset.html)

---

### 4.3 Datas, horas, timestamps e intervalos

[⬆️ Voltar ao Sumário](#sumário)

| Conceito | Tipo comum | Pergunta obrigatória |
|---|---|---|
| data civil | `DATE` | qual calendário e regra de negócio? |
| hora local | `TIME` | há offset/time zone? |
| instante | timestamp com time zone ou representação equivalente | como normalizar e exibir? |
| data/hora local | timestamp sem time zone | em qual zona será interpretada? |
| duração | `INTERVAL` quando suportado | meses civis ou segundos exatos? |

```sql
SELECT CURRENT_DATE AS hoje,
       CURRENT_TIMESTAMP AS agora;
```

**Leitura guiada:** os valores vêm do contexto temporal do SGBD/sessão segundo o produto. Eles não substituem um clock explicitamente injetado em testes nem resolvem sozinhos time zone de negócio.

“Guardar UTC” é um começo para instantes, mas recorrência, data civil, zona original e mudanças de regras de fuso podem exigir campos adicionais.

> **Fontes oficiais:** [PostgreSQL — Date/Time Types](https://www.postgresql.org/docs/current/datatype-datetime.html), [SQL Server — Date and Time Types](https://learn.microsoft.com/en-us/sql/t-sql/data-types/date-and-time-types), [Oracle — Datetime Data Types](https://docs.oracle.com/en/database/oracle/oracle-database/26/nlspg/datetime-data-types-and-time-zone-support.html)

---

### 4.4 Booleanos, binários, UUID, JSON e extensões

[⬆️ Voltar ao Sumário](#sumário)

O padrão possui booleano, mas produtos diferem: PostgreSQL oferece `BOOLEAN`; SQL Server usa frequentemente `BIT`; SQLite aplica typing dinâmico/affinity; Oracle e MySQL têm seus contratos por versão.

Binários, UUID, JSON, arrays, ranges, geospatial e vector types também variam. Não transforme tipo proprietário em contrato público portável sem adapter/migration.

| Necessidade | Primeiro procure | Observação |
|---|---|---|
| flag | booleano nativo ou domínio controlado | proteja com constraint |
| identificador UUID | tipo nativo quando disponível | representação textual custa mais |
| documento JSON | tipo/feature JSON do produto | ainda modele constraints e paths |
| arquivo grande | LOB ou object storage | backup, streaming e transação importam |

> **Fontes oficiais:** [PostgreSQL — Data Types](https://www.postgresql.org/docs/current/datatype.html), [SQLite — Datatypes](https://sqlite.org/datatype3.html), [MySQL — JSON Data Type](https://dev.mysql.com/doc/refman/8.4/en/json.html), [SQL Server — JSON Data](https://learn.microsoft.com/en-us/sql/relational-databases/json/json-data-sql-server)

---

### 4.5 `NULL`, `UNKNOWN` e testes corretos

[⬆️ Voltar ao Sumário](#sumário)

`NULL` marca ausência do valor SQL; não é zero, string vazia ou um valor igual a outro `NULL`. Comparações comuns com `NULL` produzem `UNKNOWN`.

| Expressão conceitual | Resultado |
|---|---|
| `1 = 1` | `TRUE` |
| `1 = 2` | `FALSE` |
| `1 = NULL` | `UNKNOWN` |
| `NULL = NULL` | `UNKNOWN` |
| `NULL IS NULL` | `TRUE` |

```sql
SELECT id, nome
FROM pessoa
WHERE departamento_id IS NULL;
```

**Leitura guiada:** `IS NULL` testa o marcador de ausência. `departamento_id = NULL` não selecionaria essas linhas, pois `WHERE` mantém somente condições `TRUE`; `FALSE` e `UNKNOWN` são descartados.

Antes de tornar coluna nullable, diga o que ausência significa. Estados diferentes — desconhecido, não aplicável, ainda não informado — podem exigir modelagem distinta.

> **Fontes oficiais:** [PostgreSQL — Comparison Functions and Operators](https://www.postgresql.org/docs/current/functions-comparison.html), [SQL Server — NULL and UNKNOWN](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/null-and-unknown-transact-sql)

---

### 4.6 Conversão, coerção e domains

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT CAST(salario AS DECIMAL(14, 2)) AS salario_normalizado
FROM pessoa
WHERE salario IS NOT NULL;
```

**Leitura guiada:** o cast muda o tipo da expressão no resultado; não altera a definição da coluna. A precisão maior evita reduzir a faixa neste exemplo, mas regras de arredondamento continuam pertencendo à implementação.

Coerções implícitas podem escolher tipo inesperado, impedir uso de índice ou falhar apenas para certos dados. Converta na fronteira apropriada e alinhe tipos de PK/FK e parâmetros.

SQL domains permitem nomear tipo com constraints em implementações que os suportam. Produtos também oferecem user-defined types distintos; confirme portabilidade e suporte do driver.

> **Fontes oficiais:** [PostgreSQL — Type Conversion](https://www.postgresql.org/docs/current/typeconv.html), [PostgreSQL — CREATE DOMAIN](https://www.postgresql.org/docs/current/sql-createdomain.html), [Oracle — Data Conversion](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Data-Type-Comparison-Rules.html)

---

## Parte 5 — Definição e Evolução de Schemas

**Objetivo da parte:**

Transformar o modelo lógico em objetos persistentes, com tipos e restrições explícitos, e evoluí-lo sem tratar DDL como um conjunto de comandos improvisados.

---

### 5.1 `CREATE SCHEMA` e `CREATE TABLE`

[⬆️ Voltar ao Sumário](#sumário)

Um *schema* é um namespace do catálogo. Qualificar objetos como `cadastro.pessoa` reduz ambiguidades, mas o modo de selecionar o schema padrão varia por produto. Em SQL Server, *schema* também é uma fronteira importante de autorização; no MySQL, `SCHEMA` é sinônimo de `DATABASE`.

```sql
CREATE SCHEMA cadastro;

CREATE TABLE cadastro.departamento (
    id INTEGER NOT NULL,
    nome VARCHAR(80) NOT NULL,
    CONSTRAINT pk_departamento PRIMARY KEY (id),
    CONSTRAINT uq_departamento_nome UNIQUE (nome)
);
```

**Leitura guiada:** `CREATE SCHEMA` cria o namespace; o nome antes do ponto qualifica a tabela. As constraints receberam nomes estáveis, úteis em diagnósticos e migrations. A ordem física das linhas não é definida pela ordem das colunas nem pela inserção.

Ao projetar uma tabela:

1. dê nome ao conceito de negócio, não à tela atual;
2. use o tipo mais estreito que represente todo o domínio legítimo;
3. declare nulabilidade e constraints deliberadamente;
4. escolha a chave antes de espalhar referências;
5. documente unidade, timezone e significado de códigos.

> **Fontes oficiais:** [ISO — SQL/Foundation](https://www.iso.org/standard/76584.html), [PostgreSQL — Data Definition](https://www.postgresql.org/docs/current/ddl.html), [SQL Server — CREATE SCHEMA](https://learn.microsoft.com/en-us/sql/t-sql/statements/create-schema-transact-sql)

---

### 5.2 Constraints na definição

[⬆️ Voltar ao Sumário](#sumário)

Constraints fazem o banco rejeitar estados inválidos independentemente de qual aplicação escreva. Prefira uma regra declarativa quando ela puder expressar corretamente a invariável.

```sql
CREATE TABLE produto (
    id INTEGER NOT NULL,
    sku VARCHAR(40) NOT NULL,
    nome VARCHAR(160) NOT NULL,
    preco DECIMAL(12, 2) NOT NULL,
    estoque INTEGER DEFAULT 0 NOT NULL,
    CONSTRAINT pk_produto PRIMARY KEY (id),
    CONSTRAINT uq_produto_sku UNIQUE (sku),
    CONSTRAINT ck_produto_preco CHECK (preco >= 0),
    CONSTRAINT ck_produto_estoque CHECK (estoque >= 0)
);
```

**Leitura guiada:** `NOT NULL` exige presença; `DEFAULT` fornece um valor quando a coluna é omitida, mas não transforma `NULL` explícito em zero; `UNIQUE` protege o SKU; `CHECK` limita valores. A chave primária identifica uma linha.

Uma `CHECK` aceita a linha quando sua condição é `TRUE` **ou** `UNKNOWN`; portanto, `CHECK (preco >= 0)` não substitui `NOT NULL`. Regras entre tabelas normalmente pedem FK, transação ou desenho diferente, não uma `CHECK` que consulta outras tabelas.

> **Fontes oficiais:** [PostgreSQL — Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html), [SQL Server — Unique Constraints and Check Constraints](https://learn.microsoft.com/en-us/sql/relational-databases/tables/unique-constraints-and-check-constraints), [MySQL — CHECK Constraints](https://dev.mysql.com/doc/refman/8.4/en/create-table-check-constraints.html)

---

### 5.3 Identity, sequences e valores gerados

[⬆️ Voltar ao Sumário](#sumário)

SQL padroniza colunas identity; PostgreSQL, SQL Server, Oracle e DB2 oferecem mecanismos relacionados, enquanto MySQL usa `AUTO_INCREMENT` e SQLite possui regras próprias para `INTEGER PRIMARY KEY`. Sintaxe e garantias não são intercambiáveis.

```sql
CREATE TABLE pedido (
    id BIGINT GENERATED ALWAYS AS IDENTITY,
    pessoa_id INTEGER NOT NULL,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL,
    total DECIMAL(14, 2) DEFAULT 0 NOT NULL,
    CONSTRAINT pk_pedido PRIMARY KEY (id),
    CONSTRAINT fk_pedido_pessoa
        FOREIGN KEY (pessoa_id) REFERENCES pessoa (id),
    CONSTRAINT ck_pedido_total CHECK (total >= 0)
);
```

**Leitura guiada:** o SGBD gera `id`; `ALWAYS` comunica que a aplicação normalmente não deve fornecê-lo. Identity gera um identificador, não uma sequência sem lacunas: rollback, cache e concorrência podem pular valores.

Sequences são objetos independentes e podem alimentar várias colunas, mas seu avanço em geral não deve ser confundido com estado transacional do negócio. Valores gerados a partir de expressões também têm restrições específicas por produto.

> **Fontes oficiais:** [PostgreSQL — Identity Columns](https://www.postgresql.org/docs/current/ddl-identity-columns.html), [SQL Server — IDENTITY](https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql-identity-property), [Oracle — CREATE SEQUENCE](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/CREATE-SEQUENCE.html), [SQLite — AUTOINCREMENT](https://sqlite.org/autoinc.html)

---

### 5.4 `ALTER`, `DROP` e `TRUNCATE`

[⬆️ Voltar ao Sumário](#sumário)

```sql
ALTER TABLE pessoa
    ADD COLUMN telefone VARCHAR(30);

ALTER TABLE pessoa
    ADD CONSTRAINT ck_pessoa_telefone
    CHECK (telefone IS NULL OR CHAR_LENGTH(telefone) >= 8);
```

**Leitura guiada:** a primeira instrução amplia o schema; a segunda protege um requisito mínimo sem proibir a ausência. Em bases grandes, até uma alteração aparentemente simples pode adquirir locks, reescrever dados ou bloquear tráfego.

| Comando | Intenção | Cuidado principal |
|---|---|---|
| `ALTER` | mudar definição | lock, validação e possível rewrite |
| `DROP` | remover objeto | dependências e perda de dados |
| `TRUNCATE` | esvaziar rapidamente | semântica transacional, FKs e identities variam |
| `DELETE` | remover linhas selecionadas | custo, log e concorrência |

`CASCADE` é conveniente e perigoso: ele amplia o conjunto de objetos afetados. Inspecione dependências e ensaie restore antes de remoções relevantes. O comportamento transacional de DDL varia entre SGBDs; não presuma rollback portátil.

> **Fontes oficiais:** [PostgreSQL — ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html), [PostgreSQL — Dependency Tracking](https://www.postgresql.org/docs/current/ddl-depend.html), [MySQL — Atomic Data Definition Statement Support](https://dev.mysql.com/doc/refman/8.4/en/atomic-ddl.html)

---

### 5.5 Tabelas temporárias e objetos auxiliares

[⬆️ Voltar ao Sumário](#sumário)

Tabelas temporárias materializam dados intermediários com um ciclo de vida ligado à sessão ou transação, conforme declaração e produto. Podem ajudar quando o mesmo conjunto caro é reutilizado, precisa de índice intermediário ou cruza várias instruções.

```sql
CREATE TEMPORARY TABLE ids_processados (
    pessoa_id INTEGER NOT NULL,
    CONSTRAINT pk_ids_processados PRIMARY KEY (pessoa_id)
);

INSERT INTO ids_processados (pessoa_id)
SELECT id
FROM pessoa
WHERE ativo = 1;
```

**Leitura guiada:** a tabela temporária recebe apenas os IDs ativos. Isso não é automaticamente mais rápido que uma CTE ou subquery: houve escrita, catalogação e talvez logging. Compare planos e tempo sob dados representativos.

CTEs, views, materialized views e tabelas permanentes de staging resolvem problemas diferentes. Escolha pelo ciclo de vida, reutilização, isolamento, volume e necessidade de índices, não por hábito.

> **Fontes oficiais:** [PostgreSQL — CREATE TABLE](https://www.postgresql.org/docs/current/sql-createtable.html), [SQL Server — Temporary Tables](https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql#temporary-tables), [Oracle — CREATE TABLE / temporary tables](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/CREATE-TABLE.html)

---

### 5.6 Migrations compatíveis e mudanças online

[⬆️ Voltar ao Sumário](#sumário)

Uma migration é uma transformação versionada do schema — e, às vezes, dos dados. Em deploy contínuo, aplicação antiga e nova podem coexistir. Uma estratégia segura comum é *expand/contract*:

1. **expandir:** adicionar a nova estrutura de forma compatível;
2. publicar código que escreva/leia o formato novo;
3. executar backfill em lotes observáveis;
4. validar invariantes e mudar leitores;
5. **contrair:** remover a estrutura antiga em outro deploy.

Evite renomear/remover coluna no mesmo instante em que todo código supostamente mudou. Definir `NOT NULL` ou validar FK em milhões de linhas pode precisar de recurso específico do SGBD, janela operacional ou validação faseada.

Toda migration precisa de: revisão do plano, limite de lock, estratégia de retomada, métrica de progresso, backup/restore ensaiado quando pertinente e plano de compatibilidade. Um script de “down” não consegue recuperar dados já destruídos.

> **Fontes oficiais:** [PostgreSQL — ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html), [SQL Server — Transaction Locking and Row Versioning Guide](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide), [MySQL — Online DDL Operations](https://dev.mysql.com/doc/refman/8.4/en/innodb-online-ddl-operations.html)

---

## Parte 6 — Inserção, Atualização e Remoção de Dados

**Objetivo da parte:**

Modificar dados com predicados explícitos, tratamento correto de concorrência e retorno observável do que realmente aconteceu.

---

### 6.1 `INSERT` e inserção em lote

[⬆️ Voltar ao Sumário](#sumário)

```sql
INSERT INTO departamento (id, nome)
VALUES
    (10, 'Engenharia'),
    (20, 'Financeiro');

INSERT INTO pessoa
    (id, departamento_id, nome, email, salario, ativo)
VALUES
    (1, 10, 'Ana Lima', 'ana@example.com', 8500.00, 1),
    (2, 20, 'Bruno Reis', 'bruno@example.com', 7200.00, 1);
```

**Leitura guiada:** a lista explícita de colunas desacopla o comando da ordem física da tabela. Cada tupla de `VALUES` cria uma linha. Defaults só atuam nas colunas omitidas ou marcadas com `DEFAULT`, conforme a sintaxe aceita.

`INSERT ... SELECT` copia um resultado relacional para outra tabela. Para grandes cargas, APIs de bulk load do produto podem ser mais adequadas, mas constraints, transação, logging e tratamento de rejeições continuam necessários.

> **Fontes oficiais:** [PostgreSQL — INSERT](https://www.postgresql.org/docs/current/sql-insert.html), [SQL Server — INSERT](https://learn.microsoft.com/en-us/sql/t-sql/statements/insert-transact-sql), [MySQL — INSERT](https://dev.mysql.com/doc/refman/8.4/en/insert.html)

---

### 6.2 `UPDATE` seguro e expressões baseadas no valor atual

[⬆️ Voltar ao Sumário](#sumário)

```sql
UPDATE produto
SET estoque = estoque - 2
WHERE id = 101
  AND estoque >= 2;
```

**Leitura guiada:** `estoque - 2` é calculado a partir do valor da linha; o segundo predicado impede estoque negativo. A aplicação deve conferir se uma linha foi afetada: zero pode significar ID inexistente ou saldo insuficiente.

Antes de um `UPDATE` ad hoc, execute o mesmo `WHERE` em um `SELECT`, conte o conjunto, abra a transação quando apropriado e preserve evidência. Um `UPDATE` sem `WHERE` é válido e altera todas as linhas.

Não faça leitura, cálculo no cliente e escrita cega quando uma expressão atômica no banco resolve. Se o novo valor depender de regras mais amplas, use transação, lock/isolamento ou controle de versão.

> **Fontes oficiais:** [PostgreSQL — UPDATE](https://www.postgresql.org/docs/current/sql-update.html), [SQL Server — UPDATE](https://learn.microsoft.com/en-us/sql/t-sql/queries/update-transact-sql), [SQLite — UPDATE](https://sqlite.org/lang_update.html)

---

### 6.3 `DELETE`, retenção e remoção em cascata

[⬆️ Voltar ao Sumário](#sumário)

```sql
DELETE FROM pessoa
WHERE ativo = 0
  AND criado_em < TIMESTAMP '2020-01-01 00:00:00';
```

**Leitura guiada:** somente pessoas inativas anteriores ao limite são candidatas. O literal timestamp segue a forma tipada do SQL; suporte e precisão exatos devem ser confirmados no dialeto.

Retenção é uma política, não apenas um comando. Considere obrigações legais, auditoria, anonimização, dependências e backups. *Soft delete* (`excluido_em`) conserva a linha, mas obriga todas as consultas e unicidades a entenderem esse estado; não é substituto automático para histórico temporal.

`ON DELETE CASCADE` mantém referências ao remover o pai, porém pode atingir um grafo grande. `RESTRICT`/`NO ACTION`, `SET NULL` e `SET DEFAULT` expressam outros contratos e têm diferenças de timing por produto.

> **Fontes oficiais:** [PostgreSQL — DELETE](https://www.postgresql.org/docs/current/sql-delete.html), [PostgreSQL — Foreign Keys](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK), [SQL Server — Primary and Foreign Key Constraints](https://learn.microsoft.com/en-us/sql/relational-databases/tables/primary-and-foreign-key-constraints)

---

### 6.4 `MERGE` e famílias de upsert

[⬆️ Voltar ao Sumário](#sumário)

`MERGE` compara uma origem com um destino e escolhe ações para linhas correspondentes ou não correspondentes. Apesar do nome comum, recursos e semânticas variam entre SQL:2023, PostgreSQL, SQL Server, Oracle e outros.

```sql
MERGE INTO departamento AS d
USING (VALUES (10, 'Engenharia de Produto')) AS origem (id, nome)
   ON d.id = origem.id
WHEN MATCHED THEN
    UPDATE SET nome = origem.nome
WHEN NOT MATCHED THEN
    INSERT (id, nome) VALUES (origem.id, origem.nome);
```

**Leitura guiada:** a origem contém uma linha; `ON` decide se há correspondência. Em caso positivo, atualiza; caso contrário, insere. Esse formato é ilustrativo e deve ser testado no SGBD-alvo.

PostgreSQL e SQLite também possuem `INSERT ... ON CONFLICT`; MySQL possui `INSERT ... ON DUPLICATE KEY UPDATE`. Não são uma única API portátil. Sob concorrência, apoie a decisão em uma constraint única e conheça o contrato de atomicidade, triggers e múltiplas correspondências.

> **Fontes oficiais:** [ISO — SQL/Foundation](https://www.iso.org/standard/76584.html), [PostgreSQL — MERGE](https://www.postgresql.org/docs/current/sql-merge.html), [SQL Server — MERGE](https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql), [SQLite — UPSERT](https://sqlite.org/lang_upsert.html)

---

### 6.5 Linhas afetadas, `RETURNING` e `OUTPUT`

[⬆️ Voltar ao Sumário](#sumário)

A contagem de linhas afetadas faz parte do resultado lógico da escrita. Ela permite detectar conflito otimista, predicado que não encontrou dados e operações inesperadamente amplas.

```sql
-- PostgreSQL; SQLite também possui RETURNING com contrato próprio.
UPDATE pessoa
SET salario = salario * 1.05
WHERE id = 1
RETURNING id, salario;
```

**Leitura guiada:** a modificação e a obtenção dos valores acontecem em uma instrução. Isso evita uma segunda consulta que poderia observar outro estado. Não use `RETURNING *` como contrato estável se o schema pode evoluir.

SQL Server oferece `OUTPUT`, Oracle tem `RETURNING ... INTO`, e drivers também expõem generated keys. Capture apenas campos necessários e confirme o comportamento com triggers e múltiplas linhas.

> **Fontes oficiais:** [PostgreSQL — Returning Data from Modified Rows](https://www.postgresql.org/docs/current/dml-returning.html), [SQL Server — OUTPUT Clause](https://learn.microsoft.com/en-us/sql/t-sql/queries/output-clause-transact-sql), [SQLite — RETURNING](https://sqlite.org/lang_returning.html)

---

### 6.6 Operações idempotentes e concorrência

[⬆️ Voltar ao Sumário](#sumário)

Uma operação idempotente pode ser repetida sem aplicar o efeito de negócio duas vezes. Retries acontecem após timeout quando o cliente não sabe se o servidor confirmou.

Padrão frequente:

1. cliente gera uma chave de idempotência;
2. banco impõe `UNIQUE` nessa chave;
3. gravação do pedido e da chave ocorre na mesma transação;
4. repetição recupera o resultado já associado.

Um “consultar e depois inserir” sem proteção ainda possui corrida. A constraint única arbitra concorrentes; a aplicação trata o conflito conforme o contrato. Idempotência não substitui isolamento e não torna toda falha automaticamente repetível — operações externas exigem coordenação própria.

> **Fontes oficiais:** [PostgreSQL — Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html), [SQL Server — Transactions](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/transactions-transact-sql), [MySQL — InnoDB Transaction Model](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-model.html)

---

## Parte 7 — Consultas com `SELECT`

**Objetivo da parte:**

Construir resultados previsíveis entendendo projeção, multiplicidade, ordem lógica, ordenação e paginação.

---

### 7.1 Projeção, aliases e expressões

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT p.id,
       p.nome,
       p.salario,
       p.salario * 12 AS salario_anual
FROM pessoa AS p;
```

**Leitura guiada:** `FROM` introduz a fonte e o alias `p`; `SELECT` projeta três colunas armazenadas e uma expressão calculada. O alias `salario_anual` nomeia a coluna do resultado, não cria coluna na tabela.

Evite `SELECT *` em contratos de aplicação: ele transfere dados desnecessários, torna o resultado sensível a mudanças de schema e cria colisões em joins. Em investigação interativa, pode ser conveniente.

SQL descreve uma tabela-resultado. A apresentação final — moeda, data localizada, quebra de linha — costuma pertencer à aplicação, salvo quando a consulta é deliberadamente um relatório.

> **Fontes oficiais:** [PostgreSQL — SELECT Lists](https://www.postgresql.org/docs/current/queries-select-lists.html), [SQL Server — SELECT](https://learn.microsoft.com/en-us/sql/t-sql/queries/select-transact-sql), [SQLite — SELECT](https://sqlite.org/lang_select.html)

---

### 7.2 `DISTINCT` e multiplicidade

[⬆️ Voltar ao Sumário](#sumário)

Resultados SQL preservam duplicatas por padrão — comportam-se como *bags*, não como conjuntos matemáticos puros. `DISTINCT` elimina linhas duplicadas depois de formar a projeção.

```sql
SELECT DISTINCT departamento_id
FROM pessoa
WHERE departamento_id IS NOT NULL;
```

**Leitura guiada:** duas pessoas do mesmo departamento geram um único ID no resultado. `DISTINCT` considera a combinação de todas as expressões projetadas.

Não use `DISTINCT` para esconder um join incorreto. Primeiro determine a cardinalidade esperada e por que as linhas se multiplicaram. Eliminar duplicatas exige trabalho e pode mascarar perda de semântica.

> **Fontes oficiais:** [PostgreSQL — DISTINCT](https://www.postgresql.org/docs/current/queries-select-lists.html#QUERIES-DISTINCT), [MySQL — SELECT](https://dev.mysql.com/doc/refman/8.4/en/select.html), [Oracle — SELECT](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/SELECT.html)

---

### 7.3 Ordem lógica de processamento

[⬆️ Voltar ao Sumário](#sumário)

Uma forma didática de raciocinar sobre um bloco de consulta é:

1. `FROM` e joins produzem linhas;
2. `WHERE` filtra linhas;
3. `GROUP BY` forma grupos;
4. `HAVING` filtra grupos;
5. `SELECT` calcula a projeção;
6. `DISTINCT` remove duplicatas;
7. `ORDER BY` ordena;
8. paginação limita o resultado.

Essa é uma **ordem lógica**, não a ordem física executada. O otimizador pode reescrever operações preservando a semântica. Ela explica por que, em muitos dialetos, um alias criado no `SELECT` não está disponível ao `WHERE` do mesmo nível.

```sql
SELECT departamento_id,
       AVG(salario) AS media_salarial
FROM pessoa
WHERE ativo = 1
GROUP BY departamento_id
HAVING AVG(salario) >= 7000
ORDER BY media_salarial DESC;
```

**Leitura guiada:** primeiro entram pessoas ativas; depois elas são agrupadas; `HAVING` testa a média do grupo; o alias é exibido e usado na ordenação quando o dialeto permite essa referência.

> **Fontes oficiais:** [PostgreSQL — Table Expressions](https://www.postgresql.org/docs/current/queries-table-expressions.html), [SQL Server — SELECT / logical processing order](https://learn.microsoft.com/en-us/sql/t-sql/queries/select-transact-sql#logical-processing-order-of-the-select-statement), [MySQL — SELECT](https://dev.mysql.com/doc/refman/8.4/en/select.html)

---

### 7.4 `ORDER BY` e determinismo

[⬆️ Voltar ao Sumário](#sumário)

Sem `ORDER BY`, nenhuma ordem é garantida — mesmo que um teste pareça devolver linhas por PK. Índices, paralelismo, estatísticas e versão podem alterar o plano.

```sql
SELECT id, nome, criado_em
FROM pessoa
ORDER BY criado_em DESC, id DESC;
```

**Leitura guiada:** registros recentes vêm primeiro; `id` desempata timestamps iguais. Uma ordenação total e estável é essencial para paginação reproduzível.

Collation define comparação e ordenação textual. A posição de `NULL` varia por produto e pode ser controlada por `NULLS FIRST/LAST` apenas onde suportado. Para contratos portáveis, torne a regra explícita ou normalize na camada apropriada.

> **Fontes oficiais:** [PostgreSQL — Sorting Rows](https://www.postgresql.org/docs/current/queries-order.html), [SQL Server — ORDER BY](https://learn.microsoft.com/en-us/sql/t-sql/queries/select-order-by-clause-transact-sql), [SQLite — SELECT / ORDER BY](https://sqlite.org/lang_select.html#orderby)

---

### 7.5 Paginação: `OFFSET/FETCH`, `LIMIT` e keyset

[⬆️ Voltar ao Sumário](#sumário)

SQL padroniza `OFFSET ... FETCH`; PostgreSQL/MySQL/SQLite também usam `LIMIT`, SQL Server possui `OFFSET ... FETCH`, e versões modernas do Oracle aceitam row limiting.

```sql
SELECT id, nome
FROM pessoa
ORDER BY id
OFFSET 20 ROWS
FETCH FIRST 10 ROWS ONLY;
```

**Leitura guiada:** pula vinte linhas da ordem definida e solicita dez. Sem `ORDER BY`, “página 3” não possui significado estável. Escritas concorrentes podem mover itens entre páginas.

Offsets altos ainda obrigam o banco a localizar/descartar muitas linhas. Para navegação sequencial, *keyset pagination* usa o último valor visto:

```sql
SELECT id, nome
FROM pessoa
WHERE id > :ultimo_id
ORDER BY id
FETCH FIRST 10 ROWS ONLY;
```

**Leitura guiada:** `:ultimo_id` é um parâmetro; o índice da chave pode retomar perto da posição. Para ordenação composta, o cursor deve carregar todos os campos de desempate e respeitar `NULL`/collation.

> **Fontes oficiais:** [PostgreSQL — LIMIT and OFFSET](https://www.postgresql.org/docs/current/queries-limit.html), [SQL Server — ORDER BY/OFFSET](https://learn.microsoft.com/en-us/sql/t-sql/queries/select-order-by-clause-transact-sql), [Oracle — SELECT row limiting](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/SELECT.html)

---

### 7.6 `CASE` e construção condicional

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT nome,
       CASE
           WHEN salario IS NULL THEN 'não informado'
           WHEN salario >= 10000 THEN 'faixa A'
           WHEN salario >= 5000 THEN 'faixa B'
           ELSE 'faixa C'
       END AS faixa_salarial
FROM pessoa;
```

**Leitura guiada:** as condições são testadas em ordem; a primeira verdadeira determina o resultado. O teste de `NULL` vem explicitamente. Todas as alternativas precisam resultar em tipos compatíveis segundo as regras do SGBD.

`CASE` é uma expressão e pode aparecer em projeções, ordenações, agregações e atualizações. Não o transforme em uma linguagem procedural oculta: regras complexas podem merecer tabela de decisão, constraint ou código de domínio.

> **Fontes oficiais:** [PostgreSQL — Conditional Expressions](https://www.postgresql.org/docs/current/functions-conditional.html), [SQL Server — CASE](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/case-transact-sql), [Oracle — CASE Expressions](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/CASE-Expressions.html)

---

## Parte 8 — Filtros, Predicados e Lógica Ternária

**Objetivo da parte:**

Expressar seleção corretamente, especialmente diante de `NULL`, listas, padrões e requisitos de desempenho.

---

### 8.1 Comparações e operadores lógicos

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT id, nome, salario
FROM pessoa
WHERE ativo = 1
  AND salario >= 7000
  AND (departamento_id = 10 OR departamento_id = 20);
```

**Leitura guiada:** todos os predicados ligados por `AND` precisam resultar em `TRUE`; os parênteses tornam a intenção do `OR` explícita. Sem eles, precedência pode produzir outro conjunto.

Operadores comuns incluem `=`, `<>`, `<`, `<=`, `>` e `>=`. Algumas implementações aceitam `!=`, mas `<>` é a grafia SQL tradicional. `AND`, `OR` e `NOT` operam sobre `TRUE`, `FALSE` e `UNKNOWN`.

Não dependa de avaliação curto-circuitada para impedir erro ou efeito: o otimizador pode reorganizar expressões. Use construções semanticamente seguras, como `NULLIF`, casts tolerantes do produto ou validação anterior.

> **Fontes oficiais:** [PostgreSQL — Comparison Operators](https://www.postgresql.org/docs/current/functions-comparison.html), [SQL Server — Logical Operators](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/logical-operators-transact-sql), [Oracle — Conditions](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Conditions.html)

---

### 8.2 `BETWEEN`, `IN`, `LIKE` e escaping

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT id, nome
FROM pessoa
WHERE departamento_id IN (10, 20)
  AND nome LIKE 'Ana%' ESCAPE '\';
```

**Leitura guiada:** `IN` testa pertença à lista; no padrão de `LIKE`, `%` significa zero ou mais caracteres e `_`, exatamente um. `ESCAPE` define como tratar esses símbolos literalmente; a representação da barra no cliente também precisa ser considerada.

`BETWEEN a AND b` inclui os dois limites. Para intervalos temporais, o padrão semiaberto costuma ser mais seguro:

```sql
SELECT id, criado_em
FROM pessoa
WHERE criado_em >= TIMESTAMP '2026-07-01 00:00:00'
  AND criado_em <  TIMESTAMP '2026-08-01 00:00:00';
```

**Leitura guiada:** o limite final exclusivo inclui qualquer precisão dentro de julho e evita inventar “23:59:59.999...”. Calcule limites com timezone e calendário corretos.

> **Fontes oficiais:** [PostgreSQL — Pattern Matching](https://www.postgresql.org/docs/current/functions-matching.html), [PostgreSQL — Row and Array Comparisons](https://www.postgresql.org/docs/current/functions-comparisons.html), [SQL Server — LIKE](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/like-transact-sql)

---

### 8.3 `EXISTS`, quantificadores e semijoins

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT d.id, d.nome
FROM departamento AS d
WHERE EXISTS (
    SELECT 1
    FROM pessoa AS p
    WHERE p.departamento_id = d.id
      AND p.ativo = 1
);
```

**Leitura guiada:** para cada departamento, a subquery procura ao menos uma pessoa ativa relacionada. `SELECT 1` comunica que os valores internos não importam; `EXISTS` testa existência e não duplica a linha do departamento.

`NOT EXISTS` expressa um antijoin e lida melhor com `NULL` do que várias formulações com `NOT IN`. SQL também define comparações quantificadas com `ANY`/`SOME` e `ALL`; a leitura deve considerar conjunto vazio e `UNKNOWN`.

> **Fontes oficiais:** [PostgreSQL — Subquery Expressions](https://www.postgresql.org/docs/current/functions-subquery.html), [SQL Server — EXISTS](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/exists-transact-sql), [Oracle — EXISTS Condition](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/EXISTS-Condition.html)

---

### 8.4 `NULL`, `NOT IN` e armadilhas

[⬆️ Voltar ao Sumário](#sumário)

Se a subquery de `NOT IN` devolver um `NULL`, comparações podem resultar em `UNKNOWN` e eliminar todas as linhas esperadas.

```sql
SELECT d.id, d.nome
FROM departamento AS d
WHERE NOT EXISTS (
    SELECT 1
    FROM pessoa AS p
    WHERE p.departamento_id = d.id
);
```

**Leitura guiada:** a consulta seleciona departamentos sem pessoa relacionada. A correlação compara IDs; um `departamento_id` nulo em outra linha não contamina o teste.

`IS DISTINCT FROM` oferece comparação null-safe no padrão e em produtos como PostgreSQL; MySQL possui `<=>`; outros dialetos usam formulações diferentes. Não reescreva automaticamente igualdade de negócio sem decidir se dois ausentes devem ser considerados iguais.

> **Fontes oficiais:** [PostgreSQL — Subquery Expressions / NOT IN](https://www.postgresql.org/docs/current/functions-subquery.html#FUNCTIONS-SUBQUERY-NOTIN), [PostgreSQL — IS DISTINCT FROM](https://www.postgresql.org/docs/current/functions-comparison.html), [MySQL — Comparison Functions](https://dev.mysql.com/doc/refman/8.4/en/comparison-operators.html)

---

### 8.5 Predicados sargable e filtros opcionais

[⬆️ Voltar ao Sumário](#sumário)

Um predicado é chamado informalmente de *sargable* quando pode orientar uma busca eficiente, frequentemente por índice. Aplicar função à coluna pode ocultar a faixa pesquisável.

```sql
-- Geralmente mais favorável a um índice em criado_em:
SELECT id
FROM pessoa
WHERE criado_em >= TIMESTAMP '2026-07-21 00:00:00'
  AND criado_em <  TIMESTAMP '2026-07-22 00:00:00';
```

**Leitura guiada:** a coluna permanece sem transformação e é comparada com dois limites. `CAST(criado_em AS DATE) = ...` pode exigir índice de expressão específico ou impedir um acesso já disponível, dependendo do produto.

O padrão `WHERE (:p IS NULL OR coluna = :p)` é prático para filtros opcionais, porém um único plano pode servir mal a combinações diferentes. SQL dinâmico **parametrizado**, consultas especializadas ou recompilação controlada podem produzir planos melhores. Meça; “sargable” não garante que o índice será escolhido.

> **Fontes oficiais:** [PostgreSQL — Indexes and WHERE](https://www.postgresql.org/docs/current/indexes.html), [SQL Server — Query Processing Architecture](https://learn.microsoft.com/en-us/sql/relational-databases/query-processing-architecture-guide), [MySQL — Optimization and Indexes](https://dev.mysql.com/doc/refman/8.4/en/optimization-indexes.html)

---

## Parte 9 — Junções e Composição de Relações

**Objetivo da parte:**

Combinar tabelas sem perder de vista cardinalidade, preservação de linhas, `NULL` introduzido pelo join e custo do plano.

---

### 9.1 `INNER JOIN`

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT p.id,
       p.nome,
       d.nome AS departamento
FROM pessoa AS p
INNER JOIN departamento AS d
        ON d.id = p.departamento_id;
```

**Leitura guiada:** `ON` define pares correspondentes; somente pessoas com departamento existente aparecem. A FK assegura a validade de valores não nulos, mas pessoas com `departamento_id IS NULL` continuam fora do resultado.

Em joins N:1, cada linha do lado N encontra no máximo um pai se a coluna referenciada for única. Antes de escrever, diga a cardinalidade esperada: “uma linha por pessoa”, “uma por pedido” ou “uma por item”.

> **Fontes oficiais:** [PostgreSQL — Joined Tables](https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-JOIN), [SQL Server — Joins](https://learn.microsoft.com/en-us/sql/relational-databases/performance/joins), [Oracle — Joins](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Joins.html)

---

### 9.2 `LEFT`, `RIGHT` e `FULL OUTER JOIN`

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT d.id,
       d.nome,
       p.nome AS pessoa
FROM departamento AS d
LEFT JOIN pessoa AS p
       ON p.departamento_id = d.id
ORDER BY d.id, p.id;
```

**Leitura guiada:** todos os departamentos são preservados. Quando não existe pessoa correspondente, as colunas de `p` aparecem como `NULL`. Várias pessoas multiplicam a linha do departamento.

`RIGHT JOIN` preserva o lado direito e geralmente pode ser reescrito invertendo as tabelas. `FULL OUTER JOIN` preserva ambos os lados e é útil para reconciliação; nem todo produto o implementa — SQLite, por exemplo, passou a suportar `RIGHT` e `FULL` em versões modernas, mas confirme a versão implantada.

> **Fontes oficiais:** [PostgreSQL — Joined Tables](https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-JOIN), [SQLite — SELECT / join operators](https://sqlite.org/lang_select.html), [MySQL — JOIN Clause](https://dev.mysql.com/doc/refman/8.4/en/join.html)

---

### 9.3 Predicados em `ON` versus `WHERE`

[⬆️ Voltar ao Sumário](#sumário)

Em inner joins, muitos filtros podem ser movidos entre `ON` e `WHERE` sem mudar o resultado. Em outer joins, essa mudança pode eliminar justamente as linhas preservadas.

```sql
SELECT d.nome AS departamento,
       p.nome AS pessoa_ativa
FROM departamento AS d
LEFT JOIN pessoa AS p
       ON p.departamento_id = d.id
      AND p.ativo = 1;
```

**Leitura guiada:** o filtro de atividade participa da correspondência; departamentos sem pessoa ativa ainda aparecem. Se `p.ativo = 1` estivesse no `WHERE`, as linhas com `p` introduzido como `NULL` seriam descartadas, aproximando o resultado de um inner join.

Use `ON` para dizer como/lado em que as relações combinam; use `WHERE` para filtrar a linha já formada. A distinção semântica vale mais que uma regra estética.

> **Fontes oficiais:** [PostgreSQL — JOIN Conditions](https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-JOIN), [SQL Server — FROM plus JOIN](https://learn.microsoft.com/en-us/sql/t-sql/queries/from-transact-sql), [Oracle — Joins](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Joins.html)

---

### 9.4 `CROSS JOIN`, self join e `LATERAL`/`APPLY`

[⬆️ Voltar ao Sumário](#sumário)

`CROSS JOIN` produz o produto cartesiano. É correto para todas as combinações deliberadas, mas perigoso quando um predicado foi esquecido.

```sql
SELECT p.nome AS pessoa,
       gestor.nome AS gestor
FROM pessoa AS p
LEFT JOIN pessoa AS gestor
       ON gestor.id = p.gestor_id;
```

**Leitura guiada:** este *self join* pressupõe uma FK nullable `pessoa.gestor_id → pessoa.id`. Aliases representam papéis diferentes da mesma tabela.

`LATERAL` permite que uma subquery no `FROM` referencie fontes anteriores; SQL Server oferece a família `CROSS APPLY`/`OUTER APPLY`. Serve para top-N por pai, expansão de coleções e funções tabulares, com sintaxe específica.

> **Fontes oficiais:** [PostgreSQL — LATERAL Subqueries](https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-LATERAL), [SQL Server — FROM / APPLY](https://learn.microsoft.com/en-us/sql/t-sql/queries/from-transact-sql), [Oracle — SELECT](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/SELECT.html)

---

### 9.5 Cardinalidade, duplicatas e diagnósticos

[⬆️ Voltar ao Sumário](#sumário)

Antes de culpar “duplicatas”, compare contagens:

```sql
SELECT p.id,
       COUNT(*) AS linhas_apos_join
FROM pessoa AS p
JOIN pedido AS pe
  ON pe.pessoa_id = p.id
GROUP BY p.id
HAVING COUNT(*) > 1;
```

**Leitura guiada:** uma pessoa com vários pedidos legitimamente aparece várias vezes no join. A agregação mostra essa multiplicidade; não prova duplicação inválida.

Perguntas de diagnóstico:

- A condição usa todas as colunas de uma chave composta?
- A origem deveria ser única e possui constraint que prova isso?
- Houve join N:N não intencional?
- A consulta pede entidades ou ocorrências relacionadas?
- `DISTINCT` está ocultando um erro de modelagem?

Estimativas ruins de cardinalidade também afetam planos. Mantenha estatísticas e escreva predicados que expressem relações reais.

> **Fontes oficiais:** [PostgreSQL — Planner Statistics](https://www.postgresql.org/docs/current/planner-stats.html), [SQL Server — Cardinality Estimation](https://learn.microsoft.com/en-us/sql/relational-databases/performance/cardinality-estimation-sql-server), [MySQL — Optimizer Statistics](https://dev.mysql.com/doc/refman/8.4/en/optimizer-statistics.html)

---

## Parte 10 — Agregação e Agrupamento

**Objetivo da parte:**

Resumir conjuntos sem confundir linhas, grupos, valores nulos e granularidade do resultado.

---

### 10.1 `COUNT`, `SUM`, `AVG`, `MIN` e `MAX`

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT COUNT(*) AS pessoas,
       COUNT(salario) AS salarios_informados,
       SUM(salario) AS folha,
       AVG(salario) AS media
FROM pessoa;
```

**Leitura guiada:** `COUNT(*)` conta linhas; `COUNT(salario)` ignora `NULL`; as demais agregações também ignoram nulos, salvo funções/contratos específicos. Em conjunto vazio, `COUNT` retorna zero, enquanto várias agregações retornam `NULL`.

O tipo do acumulador/resultado pode diferir do tipo de entrada. Para dinheiro e medições, valide precisão, arredondamento e overflow do produto.

Agregações com `DISTINCT`, por exemplo `COUNT(DISTINCT departamento_id)`, deduplicam somente a expressão indicada e podem exigir memória ou ordenação adicional.

> **Fontes oficiais:** [PostgreSQL — Aggregate Functions](https://www.postgresql.org/docs/current/functions-aggregate.html), [SQL Server — Aggregate Functions](https://learn.microsoft.com/en-us/sql/t-sql/functions/aggregate-functions-transact-sql), [Oracle — Aggregate Functions](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Aggregate-Functions.html)

---

### 10.2 `GROUP BY` e granularidade

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT departamento_id,
       COUNT(*) AS quantidade,
       AVG(salario) AS salario_medio
FROM pessoa
GROUP BY departamento_id;
```

**Leitura guiada:** o resultado possui uma linha por valor de `departamento_id`; todos os `NULL` formam um grupo para esse propósito. A granularidade deixou de ser “pessoa” e passou a ser “departamento associado”.

Toda expressão projetada que não é agregada precisa ser compatível com o agrupamento segundo as regras do padrão/produto. Modos permissivos que escolhem um valor arbitrário produzem consultas frágeis.

> **Fontes oficiais:** [PostgreSQL — GROUP BY](https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-GROUP), [MySQL — GROUP BY Handling](https://dev.mysql.com/doc/refman/8.4/en/group-by-handling.html), [SQLite — Aggregate Queries](https://sqlite.org/lang_select.html#bareagg)

---

### 10.3 `HAVING` e filtros de grupo

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT departamento_id,
       COUNT(*) AS quantidade
FROM pessoa
WHERE ativo = 1
GROUP BY departamento_id
HAVING COUNT(*) >= 5;
```

**Leitura guiada:** `WHERE` remove pessoas inativas antes do agrupamento; `HAVING` mantém apenas grupos com pelo menos cinco linhas restantes.

Filtros que independem de agregação normalmente pertencem ao `WHERE`, reduzindo dados cedo e comunicando melhor a intenção. `HAVING` sem `GROUP BY` pode tratar todo o resultado como um grupo, conforme a consulta e o dialeto.

Se a pergunta for “quais pessoas?”, use `WHERE`; se for “quais departamentos cuja contagem...?”, a condição agregada pertence a `HAVING`.

> **Fontes oficiais:** [PostgreSQL — GROUP BY and HAVING](https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-GROUP), [SQL Server — HAVING](https://learn.microsoft.com/en-us/sql/t-sql/queries/select-having-transact-sql), [MySQL — SELECT](https://dev.mysql.com/doc/refman/8.4/en/select.html)

---

### 10.4 Agregação condicional e `FILTER`

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT departamento_id,
       COUNT(*) AS total,
       SUM(CASE WHEN ativo = 1 THEN 1 ELSE 0 END) AS ativos
FROM pessoa
GROUP BY departamento_id;
```

**Leitura guiada:** cada linha ativa contribui com 1 e cada inativa com 0; `SUM` calcula a quantidade. Essa forma é amplamente portável quando o tipo booleano não pode ser somado.

O padrão e produtos como PostgreSQL oferecem `aggregate(...) FILTER (WHERE condição)`, mais direto para várias métricas condicionais. SQL Server e alguns outros exigem `CASE` ou sintaxe própria. Evite fazer N consultas quando uma passagem agrupada resolve de modo legível.

> **Fontes oficiais:** [PostgreSQL — Aggregate Expressions / FILTER](https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-AGGREGATES), [SQLite — Aggregate Function Invocation](https://sqlite.org/lang_aggfunc.html), [SQL Server — CASE](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/case-transact-sql)

---

### 10.5 `ROLLUP`, `CUBE` e `GROUPING SETS`

[⬆️ Voltar ao Sumário](#sumário)

`GROUPING SETS` calcula várias granularidades numa consulta; `ROLLUP` produz uma hierarquia de subtotais e `CUBE`, combinações de dimensões.

```sql
SELECT departamento_id,
       ativo,
       COUNT(*) AS quantidade
FROM pessoa
GROUP BY GROUPING SETS (
    (departamento_id, ativo),
    (departamento_id),
    ()
);
```

**Leitura guiada:** o primeiro conjunto detalha departamento e atividade; o segundo gera subtotal por departamento; `()` produz o total geral. `NULL` no resultado pode significar subtotal ou valor real ausente; use a função `GROUPING` do dialeto para distinguir.

> **Fontes oficiais:** [PostgreSQL — GROUPING SETS](https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-GROUPING-SETS), [SQL Server — GROUP BY](https://learn.microsoft.com/en-us/sql/t-sql/queries/select-group-by-transact-sql), [Oracle — GROUP BY](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/SELECT.html)

---

## Parte 11 — Subqueries, CTEs e Recursão

**Objetivo da parte:**

Decompor consultas, expressar existência e navegar hierarquias sem perder correlação, cardinalidade ou condição de parada.

---

### 11.1 Subqueries escalares, tabulares e correlacionadas

[⬆️ Voltar ao Sumário](#sumário)

Uma subquery escalar deve devolver no máximo uma linha e uma coluna; mais de uma linha é erro. Subqueries tabulares alimentam `FROM`, `IN`, `EXISTS` e outras construções.

```sql
SELECT p.id,
       p.nome,
       p.salario
FROM pessoa AS p
WHERE p.salario > (
    SELECT AVG(p2.salario)
    FROM pessoa AS p2
    WHERE p2.departamento_id = p.departamento_id
);
```

**Leitura guiada:** a subquery correlaciona `p2` à linha externa `p` e calcula a média do respectivo departamento. Pessoas com departamento/salário nulo exigem decisão de negócio explícita.

O texto correlacionado não obriga execução “uma vez por linha”; o otimizador pode decorrelacionar. Compare planos se a forma estiver cara.

> **Fontes oficiais:** [PostgreSQL — Subquery Expressions](https://www.postgresql.org/docs/current/functions-subquery.html), [SQL Server — Subqueries](https://learn.microsoft.com/en-us/sql/relational-databases/performance/subqueries), [Oracle — Using Subqueries](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Using-Subqueries.html)

---

### 11.2 CTEs com `WITH`

[⬆️ Voltar ao Sumário](#sumário)

```sql
WITH ativos AS (
    SELECT id, departamento_id, salario
    FROM pessoa
    WHERE ativo = 1
),
resumo AS (
    SELECT departamento_id,
           COUNT(*) AS quantidade,
           AVG(salario) AS media
    FROM ativos
    GROUP BY departamento_id
)
SELECT departamento_id, quantidade, media
FROM resumo
ORDER BY departamento_id;
```

**Leitura guiada:** `ativos` nomeia o primeiro resultado; `resumo` o reutiliza logicamente; o `SELECT` final define a interface. CTE melhora decomposição, mas não é função nem tabela persistente.

Materialização/inlining variam por produto, versão e hints. Não presuma que uma CTE sempre otimiza ou sempre penaliza; leia o plano.

CTEs também ajudam a testar cada estágio isoladamente: execute primeiro a consulta interna, confirme sua granularidade e então componha o próximo nível.

> **Fontes oficiais:** [PostgreSQL — WITH Queries](https://www.postgresql.org/docs/current/queries-with.html), [SQL Server — CTE](https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql), [MySQL — CTEs](https://dev.mysql.com/doc/refman/8.4/en/with.html)

---

### 11.3 CTE recursiva e condição de parada

[⬆️ Voltar ao Sumário](#sumário)

```sql
WITH RECURSIVE numeros (n) AS (
    SELECT 1
    UNION ALL
    SELECT n + 1
    FROM numeros
    WHERE n < 5
)
SELECT n
FROM numeros
ORDER BY n;
```

**Leitura guiada:** o termo âncora produz 1; o termo recursivo lê o resultado anterior e soma 1; `n < 5` encerra a expansão. O resultado é 1 a 5.

SQL Server omite a palavra `RECURSIVE`; limites de recursão e sintaxe de busca/ciclo variam. Uma condição de parada defeituosa pode consumir recursos. Teste profundidade, ciclos e volume.

O tipo de cada coluna recursiva é determinado pela âncora e pelas regras de compatibilidade; faça casts explícitos quando caminhos ou contadores puderem crescer.

> **Fontes oficiais:** [PostgreSQL — Recursive Queries](https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-RECURSIVE), [SQL Server — Recursive CTEs](https://learn.microsoft.com/en-us/sql/t-sql/queries/recursive-common-table-expression-transact-sql), [SQLite — Recursive CTEs](https://sqlite.org/lang_with.html#recursive_common_table_expressions)

---

### 11.4 Hierarquias, grafos e detecção de ciclos

[⬆️ Voltar ao Sumário](#sumário)

Uma adjacency list armazena `pai_id` na própria tabela. A CTE recursiva pode produzir descendentes, profundidade e caminho, mas dados cíclicos quebram a suposição de árvore.

Alternativas como materialized path, nested sets e closure table favorecem operações diferentes e aumentam o custo de outras. SQL/PGQ, publicado na família SQL:2023, padroniza consultas de property graphs; suporte de produto ainda deve ser verificado.

Para produção:

- imponha FK e, quando possível, regras contra autorreferência trivial;
- limite profundidade/linhas;
- mantenha um conjunto de visitados ou use recursos `CYCLE`;
- escolha `UNION ALL` versus `UNION` conscientemente;
- teste grafos desconectados e ciclos.

> **Fontes oficiais:** [ISO — SQL/PGQ](https://www.iso.org/standard/79473.html), [PostgreSQL — Recursive Query Cycle Detection](https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-CYCLE), [Oracle — Hierarchical Queries](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Hierarchical-Queries.html)

---

## Parte 12 — Operações de Conjuntos

**Objetivo da parte:**

Combinar resultados compatíveis entendendo duplicatas, precedência, nomes de colunas e custo.

---

### 12.1 `UNION` e `UNION ALL`

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT email
FROM pessoa
WHERE ativo = 1
UNION ALL
SELECT email
FROM cliente_importado;
```

**Leitura guiada:** os dois ramos precisam ter o mesmo número de colunas e tipos compatíveis. `UNION ALL` preserva repetições; os nomes do resultado vêm normalmente do primeiro ramo.

Use `UNION` somente quando a semântica exigir eliminação de duplicatas. Se fontes são disjuntas por definição, `UNION ALL` comunica isso e evita o trabalho de deduplicação.

Aliases e tipos pertencem ao resultado combinado; se cada ramo representa uma categoria, adicione uma coluna literal de origem para preservar proveniência.

> **Fontes oficiais:** [PostgreSQL — Combining Queries](https://www.postgresql.org/docs/current/queries-union.html), [SQL Server — UNION](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/set-operators-union-transact-sql), [MySQL — Set Operations](https://dev.mysql.com/doc/refman/8.4/en/set-operations.html)

---

### 12.2 `INTERSECT` e `EXCEPT`

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT pessoa_id
FROM participante_projeto
WHERE projeto_id = 100
INTERSECT
SELECT pessoa_id
FROM participante_projeto
WHERE projeto_id = 200;
```

**Leitura guiada:** devolve pessoas presentes nos dois projetos. `EXCEPT` devolveria as do primeiro resultado que não aparecem no segundo. Oracle tradicionalmente usa `MINUS` para a diferença, além da evolução recente do produto.

Versões `ALL` preservam multiplicidades segundo regras de bags, onde suportadas. `EXISTS`/joins podem expressar alternativas, mas compare semântica de `NULL` e duplicatas.

Esses operadores são especialmente legíveis em reconciliação: “presentes nos dois”, “somente na origem” e “somente no destino” viram consultas auditáveis.

> **Fontes oficiais:** [PostgreSQL — UNION, INTERSECT and EXCEPT](https://www.postgresql.org/docs/current/queries-union.html), [SQL Server — EXCEPT and INTERSECT](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/set-operators-except-and-intersect-transact-sql), [Oracle — Set Operators](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/The-UNION-ALL-INTERSECT-MINUS-Operators.html)

---

### 12.3 Compatibilidade, precedência e ordenação final

[⬆️ Voltar ao Sumário](#sumário)

Parênteses tornam explícita a associação entre operações. `INTERSECT` pode ter precedência diferente de `UNION`/`EXCEPT`; não obrigue o leitor a memorizar quando a intenção é importante.

```sql
(SELECT id, nome FROM pessoa WHERE ativo = 1)
UNION ALL
(SELECT id, nome FROM pessoa_arquivada)
ORDER BY id;
```

**Leitura guiada:** `ORDER BY` ordena o resultado combinado, não cada ramo. Ordenação interna só faz sentido em contextos específicos, como limitação top-N, e precisa obedecer ao dialeto.

Converta tipos deliberadamente se os ramos usam domínios distintos. Uma coerção implícita para texto ou precisão menor pode esconder defeito.

> **Fontes oficiais:** [PostgreSQL — Combining Queries](https://www.postgresql.org/docs/current/queries-union.html), [SQL Server — UNION](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/set-operators-union-transact-sql), [SQLite — Compound Select Statements](https://sqlite.org/lang_select.html#compound)

---

## Parte 13 — Funções de Janela

**Objetivo da parte:**

Calcular rankings, totais e comparações preservando cada linha, com frames e desempates explícitos.

---

### 13.1 `OVER`, `PARTITION BY` e `ORDER BY`

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT id,
       departamento_id,
       salario,
       AVG(salario) OVER (
           PARTITION BY departamento_id
       ) AS media_departamento
FROM pessoa;
```

**Leitura guiada:** `PARTITION BY` cria grupos analíticos, mas não colapsa linhas como `GROUP BY`. Cada pessoa conserva sua linha e recebe a média do departamento.

O `ORDER BY` dentro de `OVER` define ordem da janela; o `ORDER BY` externo define apresentação. São responsabilidades independentes.

Se `PARTITION BY` for omitido, todas as linhas pertencem à mesma partição; se o `ORDER BY` interno faltar, funções dependentes de posição não têm sequência útil.

> **Fontes oficiais:** [PostgreSQL — Window Functions](https://www.postgresql.org/docs/current/tutorial-window.html), [SQL Server — OVER Clause](https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql), [Oracle — Analytic Functions](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Analytic-Functions.html)

---

### 13.2 Ranking: `ROW_NUMBER`, `RANK` e `DENSE_RANK`

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT nome,
       departamento_id,
       salario,
       ROW_NUMBER() OVER (
           PARTITION BY departamento_id
           ORDER BY salario DESC, id
       ) AS posicao_unica,
       DENSE_RANK() OVER (
           PARTITION BY departamento_id
           ORDER BY salario DESC
       ) AS faixa
FROM pessoa;
```

**Leitura guiada:** `ROW_NUMBER` entrega números únicos e usa `id` para desempatar; `DENSE_RANK` dá a mesma posição a salários iguais e não deixa lacunas. `RANK` deixaria lacunas depois de empates.

Para “top 3 por departamento”, calcule a janela numa CTE/subquery e filtre no nível externo, salvo se o produto oferecer `QUALIFY`.

Escolha ranking pela pergunta: posição física única (`ROW_NUMBER`), classificação esportiva com lacunas (`RANK`) ou classes de empate consecutivas (`DENSE_RANK`).

> **Fontes oficiais:** [PostgreSQL — Window Functions](https://www.postgresql.org/docs/current/functions-window.html), [SQL Server — Ranking Functions](https://learn.microsoft.com/en-us/sql/t-sql/functions/ranking-functions-transact-sql), [MySQL — Window Function Descriptions](https://dev.mysql.com/doc/refman/8.4/en/window-function-descriptions.html)

---

### 13.3 `LAG`, `LEAD`, primeiros e últimos valores

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT pessoa_id,
       data_medicao,
       valor,
       LAG(valor) OVER (
           PARTITION BY pessoa_id
           ORDER BY data_medicao
       ) AS valor_anterior
FROM medicao;
```

**Leitura guiada:** `LAG` lê a linha anterior na ordem de cada pessoa sem self join. Na primeira linha da partição, retorna `NULL` ou o default opcional.

`FIRST_VALUE` e `LAST_VALUE` operam dentro do frame, não necessariamente na partição inteira. `LAST_VALUE` surpreende quando o frame padrão termina na linha atual; declare o frame desejado.

`LEAD` faz o movimento oposto a `LAG` e permite calcular diferença até a próxima observação sem deslocar dados no cliente.

> **Fontes oficiais:** [PostgreSQL — Window Functions](https://www.postgresql.org/docs/current/functions-window.html), [SQL Server — LAG](https://learn.microsoft.com/en-us/sql/t-sql/functions/lag-transact-sql), [Oracle — LAG](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/LAG.html)

---

### 13.4 Frames `ROWS`, `RANGE` e `GROUPS`

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT data_medicao,
       valor,
       AVG(valor) OVER (
           ORDER BY data_medicao
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS media_movel_3
FROM medicao;
```

**Leitura guiada:** o frame contém a linha atual e até duas linhas físicas anteriores na ordem, portanto calcula média móvel de no máximo três observações. Datas ausentes não são preenchidas automaticamente.

`ROWS` conta linhas; `RANGE` agrupa pares segundo valores da ordenação e limites suportados; `GROUPS` conta grupos de pares. Defaults e suporte variam. Declare frame quando o resultado depende dele.

> **Fontes oficiais:** [PostgreSQL — Window Function Calls](https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS), [SQLite — Window Frames](https://sqlite.org/windowfunctions.html), [SQL Server — OVER Clause](https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql)

---

### 13.5 Padrões analíticos e armadilhas

[⬆️ Voltar ao Sumário](#sumário)

Padrões úteis incluem acumulado, percentual do total, top-N por grupo, detecção de lacunas e comparação com período anterior.

```sql
SELECT data_medicao,
       valor,
       SUM(valor) OVER (
           ORDER BY data_medicao
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS acumulado
FROM medicao;
```

**Leitura guiada:** cada linha soma desde o início até a posição atual. Se `data_medicao` não é única, adicione desempate para resultado determinístico e decida se pares devem avançar juntos.

Janelas podem exigir sort grande; partições amplas consomem memória/temp space. Filtre na fase correta, indexe conforme padrão de acesso e examine spill/plan.

> **Fontes oficiais:** [PostgreSQL — Window Functions](https://www.postgresql.org/docs/current/tutorial-window.html), [MySQL — Window Function Optimization](https://dev.mysql.com/doc/refman/8.4/en/window-function-optimization.html), [SQL Server — Showplan](https://learn.microsoft.com/en-us/sql/relational-databases/showplan-logical-and-physical-operators-reference)

---

## Parte 14 — Funções, Operadores e Expressões Essenciais

**Objetivo da parte:**

Conhecer o vocabulário pronto mais frequente sem confundir recursos do padrão com bibliotecas específicas de cada SGBD.

---

### 14.1 Texto

[⬆️ Voltar ao Sumário](#sumário)

| Intenção | Forma comum/padrão | Variação relevante |
|---|---|---|
| comprimento | `CHAR_LENGTH(texto)` | `LEN` no SQL Server tem contrato próprio |
| concatenar | `a \|\| b` | SQL Server usa `+`/`CONCAT`; MySQL privilegia `CONCAT` |
| recortar | `SUBSTRING(...)` | gramática varia |
| caixa | `UPPER`, `LOWER` | depende de collation/Unicode |
| remover bordas | `TRIM` | opções variam |
| localizar | `POSITION` | produtos oferecem `CHARINDEX`, `INSTR`, etc. |

```sql
SELECT UPPER(TRIM(nome)) AS nome_normalizado,
       CHAR_LENGTH(nome) AS caracteres
FROM pessoa;
```

**Leitura guiada:** `TRIM` remove caracteres de borda conforme a forma usada; `UPPER` aplica regras do SGBD/collation; `CHAR_LENGTH` conta caracteres, não necessariamente bytes nem grafemas percebidos pelo usuário.

Não aplique normalização destrutiva sem preservar a forma necessária. Busca linguística, case folding e acentos exigem collation e requisitos explícitos.

> **Fontes oficiais:** [PostgreSQL — String Functions](https://www.postgresql.org/docs/current/functions-string.html), [SQL Server — String Functions](https://learn.microsoft.com/en-us/sql/t-sql/functions/string-functions-transact-sql), [Oracle — Character Functions](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Functions.html)

---

### 14.2 Números e arredondamento

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT preco,
       ROUND(preco * 1.075, 2) AS preco_com_acrescimo,
       ABS(preco - 100) AS distancia_de_100
FROM produto;
```

**Leitura guiada:** `ROUND` pede duas casas neste dialeto/forma; `ABS` produz magnitude. Tipo do argumento, regra de empate e tipo do retorno podem mudar o resultado.

Funções comuns incluem `CEILING`/`CEIL`, `FLOOR`, `POWER`, `SQRT`, `MOD` e funções trigonométricas, com disponibilidade variável. Não use ponto flutuante para valores que exigem aritmética decimal exata.

Divisão entre inteiros e módulo merecem teste no dialeto: tipo do resultado, sinal do resto e divisão por zero não devem ser inferidos de outra linguagem.

> **Fontes oficiais:** [PostgreSQL — Mathematical Functions](https://www.postgresql.org/docs/current/functions-math.html), [SQL Server — Mathematical Functions](https://learn.microsoft.com/en-us/sql/t-sql/functions/mathematical-functions-transact-sql), [MySQL — Mathematical Functions](https://dev.mysql.com/doc/refman/8.4/en/mathematical-functions.html)

---

### 14.3 Datas, horas e intervalos

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT CURRENT_DATE AS data_corrente,
       CURRENT_TIMESTAMP AS instante_da_sessao;
```

**Leitura guiada:** são expressões SQL de contexto temporal. A estabilidade dentro de uma instrução/transação e a zona associada seguem o padrão e o contrato do produto.

Somar intervalos, truncar período, extrair campos e converter timezone diferem bastante: `EXTRACT` e tipos `INTERVAL` são conceitos SQL; `DATEADD`, `DATEDIFF`, `DATE_TRUNC`, `TIMESTAMPADD` e `AT TIME ZONE` possuem contratos particulares. “Diferença em dias” pode significar fronteiras civis ou duração de 24 horas.

> **Fontes oficiais:** [PostgreSQL — Date/Time Functions](https://www.postgresql.org/docs/current/functions-datetime.html), [SQL Server — Date and Time Functions](https://learn.microsoft.com/en-us/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql), [MySQL — Date and Time Functions](https://dev.mysql.com/doc/refman/8.4/en/date-and-time-functions.html)

---

### 14.4 `COALESCE`, `NULLIF` e tratamento de nulos

[⬆️ Voltar ao Sumário](#sumário)

```sql
SELECT nome,
       COALESCE(salario, 0) AS salario_para_exibicao,
       100 / NULLIF(:divisor, 0) AS quociente
FROM pessoa;
```

**Leitura guiada:** `COALESCE` retorna o primeiro argumento não nulo, após resolver um tipo comum; ele não altera o dado armazenado. `NULLIF` retorna `NULL` quando o divisor é zero, evitando a divisão nessa formulação — mas a aplicação deve decidir o significado do resultado ausente.

`ISNULL`, `NVL` e `IFNULL` são atalhos de produtos com diferenças de tipo e avaliação. Prefira `COALESCE`/`NULLIF` quando atendem à semântica, e não substitua ausência por zero apenas para facilitar uma tela.

> **Fontes oficiais:** [PostgreSQL — Conditional Expressions](https://www.postgresql.org/docs/current/functions-conditional.html), [SQL Server — COALESCE](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/coalesce-transact-sql), [Oracle — COALESCE](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/COALESCE.html)

---

### 14.5 JSON, XML, arrays, regex e full-text

[⬆️ Voltar ao Sumário](#sumário)

Essas famílias são valiosas e pouco portáveis em detalhes:

| Família | Boa aplicação | Pergunta obrigatória |
|---|---|---|
| JSON | atributos/documentos variáveis e integração | quais paths precisam de constraint/índice? |
| XML | documentos com ecossistema XML | schema, namespaces e custo? |
| arrays | coleções pequenas/acopladas | consulta relacional seria mais íntegra? |
| regex | validação/busca textual avançada | dialeto e collation são compatíveis? |
| full-text | relevância linguística | idioma, tokenizer e ranking? |

SQL:2023 reúne recursos JSON no padrão Foundation, mas operadores, índices e funções de PostgreSQL, SQL Server, MySQL, Oracle e SQLite não são equivalentes. Encapsule dependências deliberadas e teste upgrades.

> **Fontes oficiais:** [PostgreSQL — JSON Functions and Operators](https://www.postgresql.org/docs/current/functions-json.html), [SQL Server — JSON Data](https://learn.microsoft.com/en-us/sql/relational-databases/json/json-data-sql-server), [MySQL — JSON Functions](https://dev.mysql.com/doc/refman/8.4/en/json-function-reference.html), [SQLite — JSON Functions](https://sqlite.org/json1.html)

---

### 14.6 Funções determinísticas, volatilidade e collation

[⬆️ Voltar ao Sumário](#sumário)

Uma função pode depender apenas dos argumentos, da instrução, da sessão, do relógio ou de estado externo. Essa classificação afeta generated columns, índices de expressão, cache, replicação e liberdade do otimizador.

Não envolva coluna indexada em uma função “inofensiva” sem avaliar:

- se a função é imutável/determinística para o produto;
- se o índice usa a mesma expressão e collation;
- se timezone, locale ou configuração da sessão alteram o resultado;
- se upgrade muda regras Unicode.

Collation participa de igualdade, ordenação e unicidade textual. Duas strings visualmente semelhantes também podem possuir sequências Unicode diferentes; SQL não substitui uma política explícita de normalização.

> **Fontes oficiais:** [PostgreSQL — Function Volatility Categories](https://www.postgresql.org/docs/current/xfunc-volatility.html), [SQL Server — Deterministic and Nondeterministic Functions](https://learn.microsoft.com/en-us/sql/relational-databases/user-defined-functions/deterministic-and-nondeterministic-functions), [MySQL — Character Sets and Collations](https://dev.mysql.com/doc/refman/8.4/en/charset.html)

---

## Parte 15 — Integridade, Constraints e Regras no Banco

**Objetivo da parte:**

Distribuir invariantes entre tipos, constraints, transações e código de modo que estados inválidos sejam impossíveis ou detectados na fronteira mais confiável.

---

### 15.1 `NOT NULL`, `CHECK` e `DEFAULT`

[⬆️ Voltar ao Sumário](#sumário)

`NOT NULL` protege presença, `CHECK` protege um predicado da linha e `DEFAULT` calcula um valor quando a coluna é omitida. Default não é validação nem backfill automático universal.

```sql
CREATE TABLE item_pedido (
    pedido_id BIGINT NOT NULL,
    produto_id INTEGER NOT NULL,
    quantidade INTEGER DEFAULT 1 NOT NULL,
    preco_unitario DECIMAL(12, 2) NOT NULL,
    CONSTRAINT pk_item_pedido PRIMARY KEY (pedido_id, produto_id),
    CONSTRAINT ck_item_quantidade CHECK (quantidade > 0),
    CONSTRAINT ck_item_preco CHECK (preco_unitario >= 0),
    CONSTRAINT fk_item_pedido FOREIGN KEY (pedido_id) REFERENCES pedido (id),
    CONSTRAINT fk_item_produto FOREIGN KEY (produto_id) REFERENCES produto (id)
);
```

**Leitura guiada:** a PK composta impede o mesmo produto duas vezes no pedido, decisão que deve corresponder ao domínio; quantidade e preço são obrigatórios e limitados; as FKs exigem pais válidos.

> **Fontes oficiais:** [PostgreSQL — Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html), [SQL Server — Specify Default Values](https://learn.microsoft.com/en-us/sql/relational-databases/tables/specify-default-values-for-columns), [Oracle — Constraint](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/constraint.html)

---

### 15.2 `PRIMARY KEY` e `UNIQUE`

[⬆️ Voltar ao Sumário](#sumário)

Uma tabela tem no máximo uma PK, mas pode ter várias chaves candidatas protegidas por `UNIQUE`. Uma PK implica unicidade e não nulabilidade; não significa necessariamente ordem física.

`UNIQUE` e `NULL` diferem entre padrão/produtos: alguns permitem múltiplos nulos, outros oferecem opções de nulls not distinct, filtered indexes ou comportamentos próprios. Se “email presente deve ser único” é regra, modele e teste exatamente essa frase.

Surrogate keys simplificam referências, mas não dispensam a constraint na chave natural que define duplicidade de negócio. Uma tabela com `id` e nenhuma outra unicidade pode aceitar duas representações da mesma entidade.

> **Fontes oficiais:** [PostgreSQL — Unique and Primary Keys](https://www.postgresql.org/docs/current/ddl-constraints.html), [SQL Server — Primary and Foreign Key Constraints](https://learn.microsoft.com/en-us/sql/relational-databases/tables/primary-and-foreign-key-constraints), [MySQL — InnoDB Index Types](https://dev.mysql.com/doc/refman/8.4/en/innodb-index-types.html)

---

### 15.3 `FOREIGN KEY` e ações referenciais

[⬆️ Voltar ao Sumário](#sumário)

FK exige que cada valor não nulo corresponda a uma chave referenciada. Ela protege integridade, não cria automaticamente um índice eficiente no lado filho em todos os produtos.

| Ação | Ideia | Uso cauteloso |
|---|---|---|
| `NO ACTION`/`RESTRICT` | impedir mudança incompatível | diferenças de timing existem |
| `CASCADE` | propagar delete/update | árvore de efeitos pode ser ampla |
| `SET NULL` | desfazer vínculo | coluna precisa aceitar nulo |
| `SET DEFAULT` | usar default | default precisa referenciar valor válido |

Em chave composta, ordem e tipos das colunas devem corresponder. Declare índice no filho quando joins/delete do pai o exigirem e valide pelo plano.

> **Fontes oficiais:** [PostgreSQL — Foreign Keys](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK), [MySQL — InnoDB Foreign Keys](https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html), [SQLite — Foreign Key Support](https://sqlite.org/foreignkeys.html)

---

### 15.4 Constraints diferíveis e validação tardia

[⬆️ Voltar ao Sumário](#sumário)

Uma constraint diferível pode ser verificada ao fim da transação em vez de ao fim da instrução. Isso viabiliza trocas de chaves/posições e ciclos controlados, mas adia o feedback e não é suportado igualmente.

PostgreSQL e Oracle oferecem contratos robustos para constraints deferidas; SQL Server e MySQL não reproduzem a mesma semântica geral. Alguns produtos também permitem adicionar constraint sem validar linhas antigas imediatamente, útil em migrations faseadas, mas novos dados e validação posterior precisam de plano explícito.

Use apenas quando a invariável precisa ser temporariamente violada dentro de uma transação. Não transforme deferred validation em dívida permanente.

> **Fontes oficiais:** [PostgreSQL — SET CONSTRAINTS](https://www.postgresql.org/docs/current/sql-set-constraints.html), [Oracle — SET CONSTRAINTS](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/SET-CONSTRAINTS.html), [SQL Server — Disable Foreign Key Constraints](https://learn.microsoft.com/en-us/sql/relational-databases/tables/disable-foreign-key-constraints-with-insert-and-update-statements)

---

### 15.5 Regras que exigem trigger ou transação

[⬆️ Voltar ao Sumário](#sumário)

Exclusividade por intervalo, saldo agregado entre tabelas e limites dependentes de várias linhas podem exceder uma `CHECK`. Antes de criar trigger, procure constraint nativa, exclusion constraint do produto, índice único por expressão/filtro ou redesenho.

Quando a regra depende de leitura e escrita:

1. determine o conjunto que precisa permanecer consistente;
2. escolha isolamento/lock que impeça corridas;
3. atualize e valide na mesma transação;
4. trate conflito/retry;
5. mantenha uma última defesa no banco quando expressável.

Triggers são úteis, mas efeitos invisíveis aumentam acoplamento. Documente ordem, recursão, execução por linha/instrução, identidade do executor e comportamento em carga em lote.

> **Fontes oficiais:** [PostgreSQL — Triggers](https://www.postgresql.org/docs/current/triggers.html), [SQL Server — DML Triggers](https://learn.microsoft.com/en-us/sql/relational-databases/triggers/dml-triggers), [Oracle — Triggers](https://docs.oracle.com/en/database/oracle/oracle-database/26/lnpls/plsql-triggers.html)

---

## Parte 16 — Views e Abstrações de Consulta

**Objetivo da parte:**

Criar interfaces relacionais estáveis sem confundir abstração lógica, materialização, autorização e compatibilidade.

---

### 16.1 Views comuns

[⬆️ Voltar ao Sumário](#sumário)

```sql
CREATE VIEW vw_pessoa_ativa AS
SELECT id, departamento_id, nome, email
FROM pessoa
WHERE ativo = 1;
```

**Leitura guiada:** a view armazena a definição da consulta, não necessariamente suas linhas. Consumidores veem somente as colunas declaradas, mas mudanças nas tabelas-base ainda podem quebrar dependências.

Views encapsulam joins e contratos de leitura, porém camadas de views aninhadas podem dificultar tuning. Nomeie colunas explicitamente e trate a view como API versionada quando vários consumidores dependem dela.

Consultar uma view não garante um plano previamente materializado: o otimizador normalmente integra sua definição à consulta externa conforme as regras da engine.

> **Fontes oficiais:** [PostgreSQL — CREATE VIEW](https://www.postgresql.org/docs/current/sql-createview.html), [SQL Server — Views](https://learn.microsoft.com/en-us/sql/relational-databases/views/views), [MySQL — Using Views](https://dev.mysql.com/doc/refman/8.4/en/views.html)

---

### 16.2 Views atualizáveis e `WITH CHECK OPTION`

[⬆️ Voltar ao Sumário](#sumário)

Views simples podem aceitar `INSERT`, `UPDATE` ou `DELETE`; joins, agregações e expressões frequentemente impedem atualização automática. As regras variam.

```sql
CREATE VIEW vw_pessoa_ativa_editavel AS
SELECT id, departamento_id, nome, email, ativo
FROM pessoa
WHERE ativo = 1
WITH CHECK OPTION;
```

**Leitura guiada:** `WITH CHECK OPTION` impede que uma escrita por essa view produza uma linha que deixe de satisfazer `ativo = 1`, segundo escopo e contrato do produto. Constraints da tabela continuam valendo.

Permissões de leitura e escrita podem ser diferentes; confirme também se triggers `INSTEAD OF` ou regras do produto alteram a atualização aparente.

> **Fontes oficiais:** [PostgreSQL — CREATE VIEW](https://www.postgresql.org/docs/current/sql-createview.html), [SQL Server — Modify Data Through a View](https://learn.microsoft.com/en-us/sql/relational-databases/views/modify-data-through-a-view), [Oracle — CREATE VIEW](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/CREATE-VIEW.html)

---

### 16.3 Materialized views e indexed views

[⬆️ Voltar ao Sumário](#sumário)

Uma materialized view armazena resultado e precisa de refresh. SQL Server oferece indexed views com requisitos específicos; MySQL não possui materialized view nativa equivalente; produtos cloud têm variações.

Escolha após responder:

- qual atraso é aceitável?
- refresh completo ou incremental existe?
- escritas na origem serão encarecidas?
- consultas podem tolerar dado obsoleto?
- como índices, estatísticas e autorização serão mantidos?

Não chame cache de “tempo real” sem medir freshness.

> **Fontes oficiais:** [PostgreSQL — Materialized Views](https://www.postgresql.org/docs/current/rules-materializedviews.html), [SQL Server — Create Indexed Views](https://learn.microsoft.com/en-us/sql/relational-databases/views/create-indexed-views), [Oracle — Materialized Views](https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/basic-materialized-views.html)

---

### 16.4 Segurança, contratos e dependências

[⬆️ Voltar ao Sumário](#sumário)

Uma view pode expor somente colunas/linhas necessárias e receber privilégios próprios. Isso não garante segurança sozinho: ownership chaining, security invoker/definer, RLS, funções chamadas e inferência por agregados dependem do produto.

Evite `SELECT *` na definição. Para breaking changes, publique uma nova versão, migre consumidores e só então remova a anterior. Consulte catálogos de dependência, mas saiba que SQL dinâmico e referências externas podem não aparecer.

> **Fontes oficiais:** [PostgreSQL — CREATE VIEW security options](https://www.postgresql.org/docs/current/sql-createview.html), [SQL Server — Ownership Chains and Context Switching](https://learn.microsoft.com/en-us/sql/relational-databases/tutorial-ownership-chains-and-context-switching?view=sql-server-ver17), [Oracle — Managing Views](https://docs.oracle.com/en/database/oracle/oracle-database/26/admin/managing-views-sequences-and-synonyms.html)

---

## Parte 17 — Transações e ACID

**Objetivo da parte:**

Definir unidades corretas de trabalho e compreender o que ACID garante — e o que depende do isolamento, storage e arquitetura externa.

---

### 17.1 Atomicidade, consistência, isolamento e durabilidade

[⬆️ Voltar ao Sumário](#sumário)

| Propriedade | Significado prático |
|---|---|
| atomicidade | a unidade confirma ou desfaz suas mudanças |
| consistência | invariantes válidas antes/depois; depende das regras modeladas |
| isolamento | concorrentes observam efeitos conforme o nível escolhido |
| durabilidade | após confirmação, dados sobrevivem às falhas cobertas pelo contrato |

ACID não garante regra que não foi expressa, resposta de serviço externo ou ausência de perda sob qualquer desastre. Durabilidade depende de configuração, hardware, replicação e acknowledgment; consistência não é uma magia do SGBD.

> **Fontes oficiais:** [PostgreSQL — Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html), [SQL Server — Transactions](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/transactions-transact-sql), [MySQL — InnoDB Transaction Model](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-model.html)

---

### 17.2 `BEGIN`, `COMMIT` e `ROLLBACK`

[⬆️ Voltar ao Sumário](#sumário)

```sql
BEGIN;

UPDATE produto
SET estoque = estoque - 1
WHERE id = 101
  AND estoque >= 1;

INSERT INTO item_pedido
    (pedido_id, produto_id, quantidade, preco_unitario)
VALUES
    (5001, 101, 1, 49.90);

COMMIT;
```

**Leitura guiada:** estoque e item confirmam juntos. A aplicação precisa validar que o `UPDATE` afetou uma linha; se não, deve executar `ROLLBACK`, não inserir o item. `BEGIN` pode ser `START TRANSACTION` ou outra API no dialeto/driver.

Mantenha transações curtas, sem esperar usuário ou rede externa. Sempre encerre no caminho de sucesso e erro.

As duas instruções devem usar a mesma conexão e o mesmo objeto transacional no driver; abrir outra conexão pode quebrar a atomicidade pretendida.

> **Fontes oficiais:** [PostgreSQL — BEGIN](https://www.postgresql.org/docs/current/sql-begin.html), [SQL Server — BEGIN TRANSACTION](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/begin-transaction-transact-sql), [SQLite — Transactions](https://sqlite.org/lang_transaction.html)

---

### 17.3 Savepoints

[⬆️ Voltar ao Sumário](#sumário)

```sql
BEGIN;
SAVEPOINT antes_do_item;

-- operação que pode falhar

ROLLBACK TO SAVEPOINT antes_do_item;
COMMIT;
```

**Leitura guiada:** o rollback parcial desfaz mudanças posteriores ao savepoint e mantém a transação externa. A sintaxe de `RELEASE` e o estado após erro variam.

“Nested transactions” de frameworks muitas vezes são savepoints, contadores lógicos ou simplesmente participam da transação existente. Conheça a implementação antes de prometer atomicidade independente.

Após `ROLLBACK TO`, operações anteriores ao ponto continuam pendentes; ainda é necessário confirmar ou desfazer a transação externa inteira.

> **Fontes oficiais:** [PostgreSQL — SAVEPOINT](https://www.postgresql.org/docs/current/sql-savepoint.html), [SQL Server — SAVE TRANSACTION](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/save-transaction-transact-sql), [SQLite — SAVEPOINT](https://sqlite.org/lang_savepoint.html)

---

### 17.4 Autocommit, erros e fronteiras

[⬆️ Voltar ao Sumário](#sumário)

Drivers normalmente iniciam em autocommit ou em um modo configurável. Em autocommit, cada statement bem-sucedido forma sua própria transação, inadequado quando várias instruções compõem uma invariável.

Após um erro, PostgreSQL deixa a transação abortada até rollback; outros produtos podem desfazer apenas a instrução ou toda a transação conforme erro/configuração. A biblioteca deve ter um padrão `try/commit` e `catch/rollback`, devolvendo a conexão limpa ao pool.

A fronteira transacional pertence ao caso de uso, não a cada repository isolado. Evite APIs internas que confirmam sem saber se a operação maior terminou.

> **Fontes oficiais:** [PostgreSQL — Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html), [JDBC — Transactions](https://docs.oracle.com/en/java/javase/25/docs/api/java.sql/java/sql/Connection.html), [ADO.NET — Local Transactions](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/local-transactions)

---

### 17.5 Transações distribuídas e sagas

[⬆️ Voltar ao Sumário](#sumário)

Two-phase commit coordena participantes preparados, mas aumenta estados operacionais e pode reter recursos. Suporte de XA/DTC/prepared transactions varia e requer procedimentos para coordenador indisponível.

Uma saga divide a operação em transações locais com mensagens e compensações. Não oferece isolamento ACID global: estados intermediários são visíveis e compensar não equivale a apagar a história.

Padrões como transactional outbox gravam mudança e evento na mesma transação local; um relay publica depois de modo ao menos uma vez, exigindo consumidores idempotentes.

> **Fontes oficiais:** [PostgreSQL — Two-Phase Transactions](https://www.postgresql.org/docs/current/two-phase.html), [SQL Server — Distributed Transactions](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/transactions-always-on-availability-and-database-mirroring), [Jakarta Transactions Specification](https://jakarta.ee/specifications/transactions/)

---

## Parte 18 — Concorrência, Isolamento, Locks e MVCC

**Objetivo da parte:**

Projetar comportamento correto com usuários simultâneos, sabendo quais anomalias o nível de isolamento permite e como tratar conflitos esperados.

---

### 18.1 Anomalias de concorrência

[⬆️ Voltar ao Sumário](#sumário)

| Anomalia | Resumo |
|---|---|
| dirty read | ler mudança ainda não confirmada |
| nonrepeatable read | reler linha e observar valor confirmado diferente |
| phantom | repetir predicado e observar conjunto diferente |
| lost update | uma escrita sobrescrever decisão concorrente |
| write skew | transações preservam checks locais, mas quebram invariável conjunta |

Os nomes do padrão não descrevem toda anomalia possível. O resultado depende do nível e da implementação; “não vi dirty read” não prova que o caso de uso é serializável.

> **Fontes oficiais:** [PostgreSQL — Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html), [SQL Server — Locking and Row Versioning](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide), [MySQL — Transaction Isolation Levels](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html)

---

### 18.2 Níveis de isolamento

[⬆️ Voltar ao Sumário](#sumário)

SQL nomeia `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ` e `SERIALIZABLE`. Produtos podem implementar níveis mais fortes que o mínimo, snapshot isolation separado ou mapas diferentes.

`SERIALIZABLE` busca resultado equivalente a alguma ordem serial, frequentemente abortando uma transação quando não pode garantir isso. Aplicação deve aceitar retries. Snapshot isolation oferece visão consistente por versões, mas pode permitir write skew se não for serializável.

Escolha pelo caso de uso e invariável, não por uma tabela genérica de desempenho. Teste concorrência real no SGBD-alvo.

> **Fontes oficiais:** [PostgreSQL — Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html), [SQL Server — SET TRANSACTION ISOLATION LEVEL](https://learn.microsoft.com/en-us/sql/t-sql/statements/set-transaction-isolation-level-transact-sql), [Oracle — Data Concurrency and Consistency](https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/data-concurrency-and-consistency.html)

---

### 18.3 Locks e leitura para atualização

[⬆️ Voltar ao Sumário](#sumário)

```sql
BEGIN;

SELECT estoque
FROM produto
WHERE id = 101
FOR UPDATE;

UPDATE produto
SET estoque = estoque - 1
WHERE id = 101;

COMMIT;
```

**Leitura guiada:** no PostgreSQL/MySQL/Oracle, `FOR UPDATE` solicita lock apropriado nas linhas selecionadas; SQL Server usa hints/semântica própria. O lock vive até o fim da transação e pode bloquear ou entrar em deadlock.

Locks podem ser de linha, página, tabela, metadata e range/key, com compatibilidade específica. Índice inadequado pode ampliar linhas examinadas/bloqueadas.

Variantes `NOWAIT` e `SKIP LOCKED` resolvem filas e baixa latência em produtos compatíveis, mas mudam o conjunto observado e precisam de contrato explícito.

> **Fontes oficiais:** [PostgreSQL — Explicit Locking](https://www.postgresql.org/docs/current/explicit-locking.html), [SQL Server — Locking Guide](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide), [MySQL — Locking Reads](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html)

---

### 18.4 MVCC e versões de linha

[⬆️ Voltar ao Sumário](#sumário)

MVCC mantém versões para que leitores e escritores interfiram menos. PostgreSQL armazena versões no heap e precisa de vacuum; InnoDB usa undo; Oracle e SQL Server possuem mecanismos próprios.

MVCC não significa “sem locks”: writers concorrem, DDL bloqueia metadata e versões antigas consomem storage. Transações longas atrasam limpeza e podem causar bloat/pressão de versão.

Monitore idade de transações, version store/undo, replicação e sessões “idle in transaction”. O detalhe operacional é parte da correção.

> **Fontes oficiais:** [PostgreSQL — MVCC](https://www.postgresql.org/docs/current/mvcc.html), [SQL Server — Row Versioning](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide), [MySQL — InnoDB MVCC](https://dev.mysql.com/doc/refman/8.4/en/innodb-multi-versioning.html)

---

### 18.5 Deadlocks, serialization failures e retry

[⬆️ Voltar ao Sumário](#sumário)

Deadlock é um ciclo de espera; o SGBD escolhe uma vítima. Evite-o acessando recursos em ordem consistente e mantendo transações curtas, mas ainda trate a falha.

Retry robusto:

1. repete a **transação inteira**, não apenas o último statement;
2. reconhece SQLSTATE/código transitório específico;
3. usa backoff e jitter com limite;
4. respeita cancelamento/deadline;
5. garante idempotência dos efeitos externos;
6. registra tentativas e causa final.

Não repita constraint violation ou erro de sintaxe como se fossem transitórios.

> **Fontes oficiais:** [PostgreSQL — Deadlocks](https://www.postgresql.org/docs/current/explicit-locking.html#LOCKING-DEADLOCKS), [SQL Server — Deadlocks Guide](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-deadlocks-guide), [MySQL — Deadlocks](https://dev.mysql.com/doc/refman/8.4/en/innodb-deadlocks.html)

---

### 18.6 Concorrência otimista e version columns

[⬆️ Voltar ao Sumário](#sumário)

```sql
UPDATE documento
SET conteudo = :novo_conteudo,
    versao = versao + 1
WHERE id = :id
  AND versao = :versao_lida;
```

**Leitura guiada:** somente a versão observada pelo cliente pode ser atualizada. Uma linha afetada significa sucesso; zero significa inexistência ou conflito, que precisa de mensagem, merge ou retry consciente.

SQL Server oferece `rowversion`, que não é data/hora; PostgreSQL expõe metadados internos que não devem substituir levianamente um contrato de domínio. Uma coluna inteira explícita é portável e testável, mas overflow e mudanças fora da aplicação precisam ser considerados.

> **Fontes oficiais:** [ADO.NET — Optimistic Concurrency](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/optimistic-concurrency), [SQL Server — rowversion](https://learn.microsoft.com/en-us/sql/t-sql/data-types/rowversion-transact-sql), [Jakarta Persistence — Version](https://jakarta.ee/specifications/persistence/)

---

## Parte 19 — Índices, Otimizador e Planos de Execução

**Objetivo da parte:**

Otimizar por evidência: entender estruturas de acesso, estimativas, planos e o custo que cada índice impõe às escritas.

---

### 19.1 O que um índice oferece

[⬆️ Voltar ao Sumário](#sumário)

Um índice é uma estrutura redundante que pode localizar, ordenar ou provar unicidade sem ler toda a tabela. Ele ocupa espaço, deve ser atualizado em cada escrita e precisa de manutenção/estatísticas.

```sql
CREATE INDEX ix_pessoa_departamento_ativo
    ON pessoa (departamento_id, ativo);
```

**Leitura guiada:** o índice começa por `departamento_id` e depois `ativo`; pode ajudar filtros/joins alinhados a esse prefixo. Não garante uso nem é equivalente a um índice iniciado por `ativo`.

PK/UNIQUE geralmente criam estrutura de suporte, mas detalhes físicos variam. Não duplique índice já coberto sem medir.

> **Fontes oficiais:** [PostgreSQL — Indexes](https://www.postgresql.org/docs/current/indexes.html), [SQL Server — Index Architecture](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide), [MySQL — Optimization and Indexes](https://dev.mysql.com/doc/refman/8.4/en/optimization-indexes.html)

---

### 19.2 B-tree, hash, bitmap, inverted e spatial

[⬆️ Voltar ao Sumário](#sumário)

| Família | Favorece | Observação |
|---|---|---|
| B-tree | igualdade, faixa e ordem | default frequente |
| hash | igualdade | suporte/persistência variam |
| bitmap | baixa cardinalidade analítica | concorrência OLTP pode ser inadequada |
| inverted | termos, arrays, JSON | GIN/full-text são específicos |
| spatial | relações geométricas | operador e SRID precisam corresponder |

O padrão SQL não define uma implementação física universal de índice. Escolha o access method do produto conforme operadores reais e distribuição dos dados.

> **Fontes oficiais:** [PostgreSQL — Index Types](https://www.postgresql.org/docs/current/indexes-types.html), [Oracle — Indexes](https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/indexes-and-index-organized-tables.html), [SQL Server — Index Design Guide](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide)

---

### 19.3 Índices compostos, covering, parciais e por expressão

[⬆️ Voltar ao Sumário](#sumário)

Índice composto deve refletir igualdade, faixas, ordenação e seletividade sem aplicar uma fórmula cega. Colunas incluídas podem cobrir projeções no SQL Server/PostgreSQL; contratos diferem.

```sql
-- PostgreSQL: índice parcial.
CREATE INDEX ix_pessoa_ativa_email
    ON pessoa (email)
    WHERE ativo = 1;
```

**Leitura guiada:** somente linhas ativas entram. A consulta precisa ter predicado que o otimizador reconheça como compatível. SQL Server possui filtered indexes; Oracle/MySQL oferecem alternativas diferentes.

Índices por expressão exigem correspondência semântica da expressão e função permitida. Muitos índices sobrecarregam inserts/updates e o trabalho de vacuum/rebuild.

> **Fontes oficiais:** [PostgreSQL — Partial Indexes](https://www.postgresql.org/docs/current/indexes-partial.html), [PostgreSQL — Indexes on Expressions](https://www.postgresql.org/docs/current/indexes-expressional.html), [SQL Server — Included Columns](https://learn.microsoft.com/en-us/sql/relational-databases/indexes/create-indexes-with-included-columns)

---

### 19.4 Estatísticas, cardinalidade e custo

[⬆️ Voltar ao Sumário](#sumário)

O otimizador estima linhas e custo para comparar planos. Estatísticas incluem distribuição, nulos, valores distintos e correlações conforme produto; dados correlacionados ou parâmetros atípicos podem enganar o modelo.

Estimativa errada perto da base do plano multiplica erros acima. Compare “estimated rows” e “actual rows”, confirme freshness de estatísticas e investigue skew, parâmetros e predicados não modelados antes de forçar um join.

Custo é unidade interna, não milissegundos. O plano mais barato segundo o modelo pode variar entre execuções/configurações.

> **Fontes oficiais:** [PostgreSQL — Statistics Used by the Planner](https://www.postgresql.org/docs/current/planner-stats.html), [SQL Server — Statistics](https://learn.microsoft.com/en-us/sql/relational-databases/statistics/statistics), [MySQL — Optimizer Statistics](https://dev.mysql.com/doc/refman/8.4/en/optimizer-statistics.html)

---

### 19.5 `EXPLAIN` e plano real

[⬆️ Voltar ao Sumário](#sumário)

```sql
EXPLAIN
SELECT p.id, p.nome
FROM pessoa AS p
WHERE p.departamento_id = 10
  AND p.ativo = 1;
```

**Leitura guiada:** `EXPLAIN` mostra o plano estimado no PostgreSQL/SQLite/MySQL com formatos distintos. `EXPLAIN ANALYZE`/plano real **executa** a instrução em produtos como PostgreSQL; não o use descuidadamente em escrita ou produção.

Leia de dentro para fora: accesses, join algorithms, sorts/aggregates, estimativas versus reais, loops, buffers/I/O, memória e spills. Uma seq scan pode ser ótima para grande fração de tabela.

> **Fontes oficiais:** [PostgreSQL — Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html), [SQL Server — Display Execution Plans](https://learn.microsoft.com/en-us/sql/relational-databases/performance/display-and-save-execution-plans), [MySQL — EXPLAIN](https://dev.mysql.com/doc/refman/8.4/en/explain.html), [SQLite — EXPLAIN QUERY PLAN](https://sqlite.org/eqp.html)

---

### 19.6 Antipadrões de consulta e tuning orientado por evidência

[⬆️ Voltar ao Sumário](#sumário)

Sinais frequentes, não regras absolutas:

- N+1 round-trips;
- função/conversão na coluna filtrada;
- paginação com offset enorme;
- `OR` opcional para dezenas de combinações;
- `SELECT *` e transferência excessiva;
- índice para cada query sem medir escrita;
- estatísticas antigas;
- transações longas;
- hints permanentes para um sintoma passageiro.

Fluxo: defina SLO → capture consulta/parâmetros/plano → reproduza com volume e distribuição realistas → altere uma variável → compare CPU, I/O, locks e latência em percentis → monitore regressão. Uma query “mais curta” não é necessariamente mais barata.

> **Fontes oficiais:** [PostgreSQL — Performance Tips](https://www.postgresql.org/docs/current/performance-tips.html), [SQL Server — Query Processing Architecture](https://learn.microsoft.com/en-us/sql/relational-databases/query-processing-architecture-guide), [MySQL — Optimization](https://dev.mysql.com/doc/refman/8.4/en/optimization.html)

---

## Parte 20 — Segurança, Autorização e SQL Injection

**Objetivo da parte:**

Reduzir superfície de ataque combinando identidade, menor privilégio, consultas parametrizadas, proteção de dados e auditoria verificável.

---

### 20.1 Usuários, roles e privilégios

[⬆️ Voltar ao Sumário](#sumário)

SGBDs distinguem login, usuário, role, schema e ownership de maneiras próprias. Modele identidades para pessoas, aplicações, migrations, observabilidade e jobs — não compartilhe uma conta superuser.

Roles agrupam privilégios; usuários/roles recebem membership. Autenticação prova identidade, enquanto autorização decide ações. Integrações com Kerberos, IAM, certificados e diretórios são extensões do produto/plataforma.

Separe ao menos:

- runtime de leitura/escrita necessária;
- migration/DDL;
- operação/backup;
- análise read-only;
- break-glass auditado.

> **Fontes oficiais:** [PostgreSQL — Database Roles](https://www.postgresql.org/docs/current/user-manag.html), [SQL Server — Principals](https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/principals-database-engine), [MySQL — Access Control](https://dev.mysql.com/doc/refman/8.4/en/access-control.html), [Oracle — Users and Security](https://docs.oracle.com/en/database/oracle/oracle-database/26/dbseg/)

---

### 20.2 `GRANT`, `REVOKE` e menor privilégio

[⬆️ Voltar ao Sumário](#sumário)

```sql
GRANT SELECT, INSERT, UPDATE ON pessoa TO app_cadastro;
REVOKE DELETE ON pessoa FROM app_cadastro;
```

**Leitura guiada:** a role recebe somente três ações e tem remoção explícita. A sintaxe de role, schema, default privileges e grant option varia; o exemplo é conceitual.

Menor privilégio exige inventário contínuo. Considere privileges no schema/sequence/function, grants herdados, `PUBLIC`, ownership e capacidade de executar SQL com privilégios do definidor. Teste acesso negado no CI, não apenas o happy path.

> **Fontes oficiais:** [PostgreSQL — GRANT](https://www.postgresql.org/docs/current/sql-grant.html), [SQL Server — GRANT](https://learn.microsoft.com/en-us/sql/t-sql/statements/grant-transact-sql), [MySQL — GRANT](https://dev.mysql.com/doc/refman/8.4/en/grant.html), [Oracle — GRANT](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/GRANT.html)

---

### 20.3 SQL injection e parâmetros

[⬆️ Voltar ao Sumário](#sumário)

SQL injection ocorre quando dados não confiáveis alteram a estrutura da instrução. A defesa principal é prepared statement/parameterized query, não escaping manual.

```csharp
const string sql = "SELECT id, nome FROM pessoa WHERE email = @email";
using var command = new SqlCommand(sql, connection);
command.Parameters.Add("@email", SqlDbType.NVarChar, 254).Value = email;
```

**Leitura guiada:** o texto SQL é fixo; `@email` recebe valor e tipo separadamente. Mesmo que `email` contenha aspas ou palavras SQL, o driver o envia como dado. `Add` com tipo/tamanho explícitos também evita inferências indesejadas.

Parâmetros protegem **valores**, não identificadores ou palavras-chave. Stored procedure só é segura se não reconstruir SQL vulnerável internamente. Use também least privilege e limites de resultado como defesa em profundidade.

> **Fontes oficiais:** [ADO.NET — Commands and Parameters](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/commands-and-parameters), [JDBC — PreparedStatement](https://docs.oracle.com/en/java/javase/25/docs/api/java.sql/java/sql/PreparedStatement.html), [OWASP — SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)

---

### 20.4 Identificadores dinâmicos e allowlists

[⬆️ Voltar ao Sumário](#sumário)

Uma aplicação não pode normalmente bindar nome de tabela, coluna ou direção `ASC/DESC`. Mapeie opções externas para tokens internos permitidos:

```csharp
string orderBy = ordenacao switch
{
    "nome" => "nome ASC, id ASC",
    "recentes" => "criado_em DESC, id DESC",
    _ => "id ASC"
};

string sql = $"SELECT id, nome FROM pessoa ORDER BY {orderBy}";
```

**Leitura guiada:** a interpolação recebe somente uma das strings constantes criadas pelo programa; entrada do usuário nunca é copiada como SQL. Valores de filtros continuariam parametrizados.

Quoting de identificador não valida autorização nem intenção. Para DDL administrativo, use API oficial de quoting do driver/produto e allowlist de objetos.

> **Fontes oficiais:** [OWASP — Allow-list Input Validation](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html#allow-list-input-validation), [PostgreSQL — Lexical Structure](https://www.postgresql.org/docs/current/sql-syntax-lexical.html), [SQL Server — QUOTENAME](https://learn.microsoft.com/en-us/sql/t-sql/functions/quotename-transact-sql)

---

### 20.5 Row-level security, views e masking

[⬆️ Voltar ao Sumário](#sumário)

Row-level security (RLS) adiciona políticas que limitam linhas conforme identidade/contexto. PostgreSQL e SQL Server oferecem modelos diferentes; Oracle possui Virtual Private Database. Teste owners, bypass roles, funções definer e operações de escrita.

Views podem reduzir colunas/linhas expostas. Dynamic data masking costuma ser proteção de apresentação para usuários não privilegiados, não criptografia nem barreira completa contra inferência.

Em multitenancy, RLS é defesa em profundidade; mantenha `tenant_id` nas chaves/constraints relevantes e contexto da sessão resistente a pooling/leak.

> **Fontes oficiais:** [PostgreSQL — Row Security Policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html), [SQL Server — Row-Level Security](https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security), [Oracle — Virtual Private Database](https://docs.oracle.com/en/database/oracle/oracle-database/26/dbseg/using-oracle-vpd-to-control-data-access.html)

---

### 20.6 Criptografia, segredos e auditoria

[⬆️ Voltar ao Sumário](#sumário)

Proteja conexões com TLS validado, dados em repouso com mecanismo apropriado e campos extremamente sensíveis com estratégia que preserve chaves fora do banco quando necessário. TDE protege mídias/arquivos, mas um usuário SQL autorizado ainda lê plaintext.

Nunca armazene senha recuperável: use algoritmo de password hashing adequado na camada de identidade. Rotacione credenciais, prefira identidades gerenciadas quando disponíveis e não grave secrets em SQL, migration ou connection string versionada.

Auditoria deve responder quem, o quê, quando, origem e resultado, com acesso/retention próprios. Logs não devem vazar parâmetros sensíveis. Aplique legislação e políticas organizacionais com especialistas responsáveis.

> **Fontes oficiais:** [PostgreSQL — SSL Support](https://www.postgresql.org/docs/current/ssl-tcp.html), [SQL Server — Encryption](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/sql-server-encryption), [MySQL — Security Guidelines](https://dev.mysql.com/doc/refman/8.4/en/security-guidelines.html), [Oracle — Database Security Guide](https://docs.oracle.com/en/database/oracle/oracle-database/26/dbseg/)

---

## Parte 21 — Procedures, Functions, Triggers e SQL Procedural

**Objetivo da parte:**

Reconhecer quando levar lógica para o servidor, quais construções já existem e como evitar efeitos ocultos ou processamento linha a linha desnecessário.

---

### 21.1 SQL/PSM e dialetos procedurais

[⬆️ Voltar ao Sumário](#sumário)

ISO/IEC 9075-4:2023 define Persistent Stored Modules (SQL/PSM). Implementações usam dialetos: PL/pgSQL, T-SQL, PL/SQL e SQL/PSM do MySQL não são portáveis entre si.

O SQL procedural acrescenta blocos, variáveis, condições, loops, handlers e rotinas. É adequado para lógica próxima dos dados, manutenção e APIs server-side; aumenta lock-in, exige testes/deploy próprios e pode esconder custo.

> **Fontes oficiais:** [ISO — SQL/PSM](https://www.iso.org/standard/76585.html), [PostgreSQL — PL/pgSQL](https://www.postgresql.org/docs/current/plpgsql.html), [SQL Server — T-SQL Reference](https://learn.microsoft.com/en-us/sql/t-sql/language-reference), [Oracle — PL/SQL](https://docs.oracle.com/en/database/oracle/oracle-database/26/lnpls/)

---

### 21.2 Procedures e functions

[⬆️ Voltar ao Sumário](#sumário)

Functions retornam valor/conjunto e podem participar de expressões conforme categoria; procedures são invocadas como operação e podem possuir controle transacional específico. A separação varia por SGBD.

Contratos devem declarar tipos, nulabilidade, efeitos, transação, segurança invoker/definer e estabilidade. Uma função chamada para cada linha pode dominar o custo e impedir otimizações; uma set-returning/table-valued function pode ser alternativa, dependendo do produto.

Versione assinatura e comportamento como qualquer API. Não dê a uma rotina `SECURITY DEFINER` mais poder do que necessário; fixe search path/contexto com segurança.

> **Fontes oficiais:** [PostgreSQL — User-Defined Functions](https://www.postgresql.org/docs/current/xfunc.html), [SQL Server — Stored Procedures](https://learn.microsoft.com/en-us/sql/relational-databases/stored-procedures/stored-procedures-database-engine), [Oracle — PL/SQL Subprograms](https://docs.oracle.com/en/database/oracle/oracle-database/26/lnpls/plsql-subprograms.html)

---

### 21.3 Variáveis, controle de fluxo e exceptions

[⬆️ Voltar ao Sumário](#sumário)

Declaração, atribuição, `IF`, loops e tratamento de erro pertencem ao dialeto procedural, não ao núcleo portável de uma query. Use set-based SQL quando a transformação cabe em uma instrução; use controle de fluxo para orquestrar conjuntos, não simular uma linguagem geral linha a linha.

Tratamento de exception precisa preservar diagnóstico e estado da transação. Capturar “qualquer erro” e continuar pode confirmar dados parciais. Use SQLSTATE/códigos, rethrow contextualizado e logging sem secrets.

> **Fontes oficiais:** [PostgreSQL — PL/pgSQL Control Structures](https://www.postgresql.org/docs/current/plpgsql-control-structures.html), [SQL Server — Control-of-Flow](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/control-of-flow), [Oracle — PL/SQL Error Handling](https://docs.oracle.com/en/database/oracle/oracle-database/26/lnpls/plsql-error-handling.html)

---

### 21.4 Triggers

[⬆️ Voltar ao Sumário](#sumário)

Triggers executam em eventos de dados ou schema. Podem ser `BEFORE`, `AFTER` ou `INSTEAD OF`, por linha ou instrução, segundo produto.

Boas aplicações incluem auditoria próxima ao dado, manutenção de invariável não declarativa e compatibilidade temporária. Riscos: ordem implícita, recursão, escrita adicional, falha longe da origem, comportamento incorreto em lote e deadlocks.

Nunca presuma uma linha: no SQL Server, `inserted`/`deleted` podem conter muitas; em PostgreSQL/Oracle há distinção row/statement. Teste `INSERT ... SELECT`, updates multirow, cascades e rollback.

> **Fontes oficiais:** [PostgreSQL — Trigger Behavior](https://www.postgresql.org/docs/current/trigger-definition.html), [SQL Server — Create DML Triggers](https://learn.microsoft.com/en-us/sql/relational-databases/triggers/create-dml-triggers), [MySQL — Trigger Syntax](https://dev.mysql.com/doc/refman/8.4/en/trigger-syntax.html), [Oracle — Triggers](https://docs.oracle.com/en/database/oracle/oracle-database/26/lnpls/plsql-triggers.html)

---

### 21.5 Cursors e processamento linha a linha

[⬆️ Voltar ao Sumário](#sumário)

Cursor percorre um result set. É necessário para streaming por drivers e alguns algoritmos procedurais, mas loops de update frequentemente são substituíveis por uma operação set-based mais curta e consistente.

Cursores variam em scrollability, sensitivity e updatability. Um cursor aberto pode manter snapshot, locks e recursos; feche-o e limite transações.

Se processamento em lote é inevitável, use chave estável, lotes limitados, checkpoint/idempotência e métricas. `OFFSET` crescente pode pular/repetir linhas sob mudanças.

> **Fontes oficiais:** [PostgreSQL — Cursors](https://www.postgresql.org/docs/current/plpgsql-cursors.html), [SQL Server — Cursors](https://learn.microsoft.com/en-us/sql/relational-databases/cursors), [Oracle — Cursors](https://docs.oracle.com/en/database/oracle/oracle-database/26/lnpls/cursors-overview.html)

---

### 21.6 SQL dinâmico

[⬆️ Voltar ao Sumário](#sumário)

SQL dinâmico é apropriado para objetos/colunas escolhidos em runtime, geração administrativa e filtros estruturais complexos. Separe:

- texto estrutural vindo de templates/allowlists;
- valores enviados por parâmetros;
- identificadores quoted pela API correta;
- contexto de autorização conhecido.

No servidor, PostgreSQL usa `EXECUTE ... USING`, SQL Server `sp_executesql`, Oracle `EXECUTE IMMEDIATE` com binds e MySQL prepared statements. Concatenar entrada continua vulnerável dentro de procedure.

> **Fontes oficiais:** [PostgreSQL — Executing Dynamic Commands](https://www.postgresql.org/docs/current/plpgsql-statements.html#PLPGSQL-STATEMENTS-EXECUTING-DYN), [SQL Server — sp_executesql](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-executesql-transact-sql), [Oracle — Native Dynamic SQL](https://docs.oracle.com/en/database/oracle/oracle-database/26/lnpls/native-dynamic-sql.html)

---

## Parte 22 — Metadados, Catálogos e Diagnóstico

**Objetivo da parte:**

Inspecionar o banco como sistema vivo: schema, dependências, sessões, locks e erros por interfaces estáveis sempre que possível.

---

### 22.1 `INFORMATION_SCHEMA`

[⬆️ Voltar ao Sumário](#sumário)

`INFORMATION_SCHEMA` é o conjunto padronizado de views de metadados, coberto por SQL/Schemata. Produtos implementam subconjuntos/extensões e normalmente mostram apenas objetos visíveis ao usuário.

```sql
SELECT table_schema,
       table_name,
       column_name,
       data_type,
       is_nullable
FROM information_schema.columns
WHERE table_name = 'pessoa'
ORDER BY ordinal_position;
```

**Leitura guiada:** a consulta descobre colunas da tabela pelo catálogo lógico. Filtre também `table_schema` em bancos com nomes repetidos. Campos e representação de tipos podem não capturar extensões proprietárias completas.

Essa abordagem é adequada a geradores e verificações portáveis; para detalhes físicos ou tipos proprietários, complemente com o catálogo específico da engine.

> **Fontes oficiais:** [ISO — SQL/Schemata](https://www.iso.org/standard/76586.html), [PostgreSQL — Information Schema](https://www.postgresql.org/docs/current/information-schema.html), [SQL Server — System Information Schema Views](https://learn.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/system-information-schema-views-transact-sql)

---

### 22.2 Catálogos específicos

[⬆️ Voltar ao Sumário](#sumário)

Catálogos específicos expõem índices, estatísticas, storage, configurações e performance que o padrão não cobre:

| Produto | Família principal |
|---|---|
| PostgreSQL | `pg_catalog`, views estatísticas |
| SQL Server | catalog views, DMVs/DMFs |
| MySQL | data dictionary, Performance Schema, `sys` |
| Oracle | static/dynamic data dictionary views |
| SQLite | `sqlite_schema` e PRAGMAs |

Essas interfaces também possuem compatibilidade/permissions. Use views documentadas, não tabelas internas não suportadas.

DMVs e views de desempenho podem ser cumulativas desde o startup ou reset; registre a janela de medição antes de comparar números.

> **Fontes oficiais:** [PostgreSQL — System Catalogs](https://www.postgresql.org/docs/current/catalogs.html), [SQL Server — Catalog Views](https://learn.microsoft.com/en-us/sql/relational-databases/system-catalog-views/catalog-views-transact-sql), [MySQL — Data Dictionary](https://dev.mysql.com/doc/refman/8.4/en/data-dictionary.html), [Oracle — Data Dictionary](https://docs.oracle.com/en/database/oracle/oracle-database/26/refrn/)

---

### 22.3 Constraints, dependências e linhagem

[⬆️ Voltar ao Sumário](#sumário)

Inventarie PKs, FKs, checks, defaults, views e routines antes de uma migration. Catálogos revelam dependências registradas; SQL dinâmico, arquivos ETL, BI e consumidores externos exigem catalogação/telemetria adicional.

Linhagem responde de onde um dado veio e quais transformações sofreu. O catálogo SQL oferece parte da resposta, enquanto jobs, streams e código precisam propagar metadata. Não confunda dependency graph sintático com impacto de negócio.

> **Fontes oficiais:** [PostgreSQL — Dependency Tracking](https://www.postgresql.org/docs/current/ddl-depend.html), [SQL Server — Track Dependencies](https://learn.microsoft.com/en-us/sql/relational-databases/views/get-information-about-a-view), [MySQL — INFORMATION_SCHEMA Referential Constraints](https://dev.mysql.com/doc/refman/8.4/en/information-schema-referential-constraints-table.html)

---

### 22.4 Sessões, statements ativos e locks

[⬆️ Voltar ao Sumário](#sumário)

Diagnóstico operacional correlaciona sessão, usuário/aplicação, query, duração, wait event, transação, blocker e objeto. Uma query longa pode estar calculando, bloqueada ou aguardando I/O — “matar” sem entender pode causar rollback ainda mais caro.

Configure application name/tags, timeouts e tracing. Para incidente:

1. capture estado antes de intervir;
2. identifique head blocker e fronteira transacional;
3. avalie impacto de cancel versus terminate;
4. preserve plano/parâmetros sanitizados;
5. corrija causa e alerte recorrência.

> **Fontes oficiais:** [PostgreSQL — Monitoring Database Activity](https://www.postgresql.org/docs/current/monitoring-stats.html), [SQL Server — Dynamic Management Views](https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views), [MySQL — Performance Schema](https://dev.mysql.com/doc/refman/8.4/en/performance-schema.html), [Oracle — Dynamic Performance Views](https://docs.oracle.com/en/database/oracle/oracle-database/26/refrn/dynamic-performance-views.html)

---

### 22.5 SQLSTATE, warnings e diagnostics

[⬆️ Voltar ao Sumário](#sumário)

SQLSTATE é um código de cinco caracteres: os dois primeiros formam a classe. Exemplos comuns incluem classe `23` para integrity constraint violation e `40` para transaction rollback; subclasses/produtos detalham a causa.

Aplicações devem usar propriedades estruturadas do driver — SQLSTATE, vendor code, constraint name — em vez de analisar mensagem localizada. Classifique retryable, conflito de domínio, autenticação, indisponibilidade e defeito de programação.

Warnings e notices também importam em truncamento, conversão ou plano. Não exponha mensagem interna bruta ao usuário; preserve correlação para diagnóstico.

> **Fontes oficiais:** [PostgreSQL — Error Codes](https://www.postgresql.org/docs/current/errcodes-appendix.html), [SQL Server — Database Engine Events and Errors](https://learn.microsoft.com/en-us/sql/relational-databases/errors-events/database-engine-events-and-errors), [Oracle — Error Help](https://docs.oracle.com/en/error-help/db/), [ISO — SQL/Foundation](https://www.iso.org/standard/76584.html)

---

## Checkpoint — Fundamentos de SQL (Partes 1–22)

[⬆️ Voltar ao Sumário](#sumário)

Antes de avançar, você deve conseguir:

- modelar tabelas com PK, UQ, FK, nullability e checks coerentes;
- explicar bag semantics, `NULL`/`UNKNOWN` e ordem lógica de `SELECT`;
- escolher join pela cardinalidade, não por tentativa;
- usar agregação, subquery, CTE, conjunto e janela sem confundir granularidade;
- delimitar transações e prever concorrência/isolamento;
- ler um plano e medir antes de criar índice;
- parametrizar valores e aplicar menor privilégio;
- consultar metadados e classificar erros estruturados.

Exercício integrador: modele pedidos, produtos e estoque; implemente criação do pedido com idempotência, concorrência e constraints; gere relatório por departamento com ranking e acumulado; explique o plano; documente migration e privilégios mínimos.

---

## Parte 23 — SQL em Aplicações e APIs de Acesso

**Objetivo da parte:**

Usar SQL por APIs reais com parâmetros, pooling, tipos e transações corretos, sem deixar abstrações esconderem custo ou concorrência.

---

### 23.1 ODBC, JDBC, ADO.NET e drivers nativos

[⬆️ Voltar ao Sumário](#sumário)

| API | Ecossistema | Abstrações centrais |
|---|---|---|
| ODBC | C/C++ e interoperabilidade ampla | environment, connection, statement |
| JDBC | Java | `DataSource`, `Connection`, `PreparedStatement`, `ResultSet` |
| ADO.NET | .NET | `DbConnection`, `DbCommand`, `DbDataReader`, `DbTransaction` |
| driver nativo | qualquer | recursos e tipos específicos do produto |

SQL é enviado por um protocolo/driver. Placeholder, binding, batch, cancelamento, timeout, encoding e type mapping fazem parte do resultado correto. Use a versão de driver suportada pelo servidor/runtime e leia suas release notes.

> **Fontes oficiais:** [Microsoft — ODBC Programmer's Reference](https://learn.microsoft.com/en-us/sql/odbc/reference/odbc-programmer-s-reference), [Oracle Java — JDBC Basics](https://docs.oracle.com/javase/tutorial/jdbc/basics/), [Microsoft — ADO.NET](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/)

---

### 23.2 Prepared statements e binding

[⬆️ Voltar ao Sumário](#sumário)

```java
String sql = "SELECT id, nome FROM pessoa WHERE email = ?";
try (PreparedStatement ps = connection.prepareStatement(sql)) {
    ps.setString(1, email);
    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) {
            long id = rs.getLong("id");
            String nome = rs.getString("nome");
        }
    }
}
```

**Leitura guiada:** `?` é placeholder posicional; `setString` vincula o valor sem mudar o SQL; os dois recursos são fechados por try-with-resources. `getLong` exige verificar `wasNull()` se a coluna puder ser nula — aqui `id` não pode.

Preparar não significa necessariamente cache server-side; drivers podem emular, preparar após limiar ou usar protocolo estendido. O benefício de segurança vem da separação estrutura/valor. Defina tamanho/precision e UTC/timezone conforme o tipo.

> **Fontes oficiais:** [JDBC — PreparedStatement](https://docs.oracle.com/en/java/javase/25/docs/api/java.sql/java/sql/PreparedStatement.html), [ADO.NET — Configuring Parameters](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/configuring-parameters-and-parameter-data-types), [ODBC — Binding Parameters](https://learn.microsoft.com/en-us/sql/odbc/reference/develop-app/binding-parameters-odbc)

---

### 23.3 Connections, pooling e transações

[⬆️ Voltar ao Sumário](#sumário)

Uma connection física é cara e finita; pools emprestam conexões. Fechar normalmente devolve ao pool. Defina máximo pelo que o banco suporta, não pelo número de threads da aplicação.

Ao devolver conexão, nenhuma transação, cursor, temp object relevante ou configuração de sessão deve vazar. Poolers/drivers resetam parte do estado; a aplicação ainda deve finalizar transação e usar timeouts.

Não mantenha connection durante chamada HTTP ou interação humana. Propague deadline/cancelamento e diferencie connection timeout, command timeout e lock timeout.

> **Fontes oficiais:** [JDBC — DataSource](https://docs.oracle.com/en/java/javase/25/docs/api/java.sql/javax/sql/DataSource.html), [ADO.NET — SQL Server Connection Pooling](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/sql-server-connection-pooling), [PostgreSQL — Client Connection Defaults](https://www.postgresql.org/docs/current/runtime-config-client.html)

---

### 23.4 Result sets, streaming e tipos

[⬆️ Voltar ao Sumário](#sumário)

Result sets têm metadata, ordem de colunas, nulabilidade prática e ciclo de vida ligado à conexão/statement. Ler milhões de linhas para uma lista em memória causa pressão no cliente; use streaming/fetch size conforme o driver e mantenha a transação pelo menor tempo possível.

Mapeamentos exigem atenção:

- `DECIMAL/NUMERIC` → decimal arbitrário/`BigDecimal`, não `double`;
- timestamp com/sem zona → tipo temporal equivalente, não string;
- `NULL` → nullable/optional e APIs `wasNull`;
- LOB → stream e descarte correto;
- UUID/JSON/arrays → suporte específico ou conversão explícita.

> **Fontes oficiais:** [JDBC — ResultSet](https://docs.oracle.com/en/java/javase/25/docs/api/java.sql/java/sql/ResultSet.html), [ADO.NET — Retrieving Data Using a DataReader](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/retrieving-data-using-a-datareader), [ODBC — Retrieving Results](https://learn.microsoft.com/en-us/sql/odbc/reference/develop-app/retrieving-results-basic)

---

### 23.5 ORMs, query builders e N+1

[⬆️ Voltar ao Sumário](#sumário)

ORM mapeia objetos e relações, acompanha mudanças e gera SQL. Query builder compõe estrutura de maneira programática. Ambos reduzem boilerplate, mas não eliminam modelagem, transações, índices ou leitura do SQL.

N+1 ocorre quando uma consulta carrega N entidades e o acesso a cada relação dispara outra. Corrija com eager loading seletivo, join/projeção, batch loading ou consulta específica — evitando produto cartesiano gigantesco.

Monitore número de round-trips, linhas/bytes, plano e duração. Desative lazy loading onde ele torna I/O invisível; use SQL manual para operações set-based complexas quando mais claro.

> **Fontes oficiais:** [EF Core — Efficient Querying](https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying), [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/), [Hibernate ORM — User Guide](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)

---

### 23.6 Migrations e compatibilidade entre aplicação e schema

[⬆️ Voltar ao Sumário](#sumário)

Controle migrations em ordem, com checksum/estado, ownership claro e execução única coordenada. Geração automática precisa de revisão: rename pode aparecer como drop+create e perder dados.

Contrato de deploy:

- release N funciona com schema antes/depois da expansão;
- backfill é retomável e limitado;
- release N+1 muda leitura/escrita;
- telemetria prova que formato antigo não é usado;
- contração ocorre depois.

Não execute DDL pesado silenciosamente no startup de todas as réplicas. Separe job de migration com credencial própria e observabilidade.

> **Fontes oficiais:** [EF Core — Migrations](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/), [Jakarta Persistence — Schema Generation](https://jakarta.ee/specifications/persistence/), [PostgreSQL — DDL](https://www.postgresql.org/docs/current/ddl.html)

---

## Parte 24 — Modelagem e Arquitetura Relacional

**Objetivo da parte:**

Projetar schemas que preservem significado ao longo do tempo, com redundância controlada e fronteiras transacionais explícitas.

---

### 24.1 Dependências funcionais e formas normais

[⬆️ Voltar ao Sumário](#sumário)

Uma dependência funcional `X → Y` diz que valores de X determinam Y. Ela fundamenta chaves e normalização.

| Forma | Intuição resumida |
|---|---|
| 1NF | atributos pertencem ao modelo relacional, sem grupos repetitivos na mesma linha |
| 2NF | não há dependência parcial de parte de chave candidata composta |
| 3NF | atributos não-chave não dependem transitivamente da chave |
| BCNF | todo determinante não trivial é superchave |

Esses resumos não substituem definições formais. Normalizar reduz anomalias de insert/update/delete; decomposição deve preservar dependências e, idealmente, ser lossless.

> **Fontes oficiais/primárias:** [IBM — Database Normalization](https://www.ibm.com/docs/en/db2/11.5.x?topic=design-database-normalization), [PostgreSQL — Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html), [Codd — A Relational Model (ACM)](https://dl.acm.org/doi/10.1145/362384.362685)

---

### 24.2 Desnormalização consciente

[⬆️ Voltar ao Sumário](#sumário)

Desnormalização duplica ou pré-calcula dados para um objetivo mensurável: latência, custo de join, relatório ou disponibilidade. Ela cria obrigação de sincronização e reconciliação.

Antes: meça plano e workload, tente índice/query/materialized view/cache, defina source of truth e tolerância de staleness. Depois: atualize atomicamente ou via pipeline idempotente, monitore drift e mantenha rebuild.

Copiar `preco_unitario` para `item_pedido` pode ser **modelagem temporal correta**, não mera desnormalização: registra o preço contratado naquele evento, diferente do preço atual do produto.

> **Fontes oficiais:** [PostgreSQL — Materialized Views](https://www.postgresql.org/docs/current/rules-materializedviews.html), [Oracle — Data Warehousing Guide](https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/), [Microsoft — Data Warehousing and Analytics](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/data/data-warehouse)

---

### 24.3 Agregados, invariantes e fronteiras transacionais

[⬆️ Voltar ao Sumário](#sumário)

Agrupe numa transação as mudanças que devem confirmar juntas. Se invariável cruza milhares de linhas/serviços, talvez precise de reserva, ledger, serialização por chave ou consistência assíncrona verificável.

Não confunda “aggregate” de domain-driven design com `GROUP BY`. O primeiro é fronteira de consistência de domínio; o segundo é operador SQL.

Constraints locais continuam essenciais dentro do agregado. Para contadores derivados, decida entre calcular, manter atomicamente ou reconciliar; cada escolha tem custo e falhas.

> **Fontes oficiais:** [PostgreSQL — Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html), [SQL Server — Transactions](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/transactions-transact-sql), [MySQL — InnoDB Transaction Model](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-model.html)

---

### 24.4 Dados temporais, histórico e soft delete

[⬆️ Voltar ao Sumário](#sumário)

Tempo pode significar:

- **valid time:** quando o fato valeu no negócio;
- **transaction/system time:** quando o banco registrou a versão;
- **event time:** quando o evento ocorreu;
- **processing time:** quando foi processado.

SQL possui recursos temporais e produtos oferecem system-versioned/temporal tables com diferenças. Um histórico manual precisa de intervalos semiabertos, não sobreposição e política para correções tardias.

Soft delete ajuda recuperação/workflow, mas complica FK, unique, RLS e toda query. Para auditoria imutável, use mecanismo separado e protegido.

> **Fontes oficiais:** [SQL Server — Temporal Tables](https://learn.microsoft.com/en-us/sql/relational-databases/tables/temporal-tables), [Oracle — Flashback](https://docs.oracle.com/en/database/oracle/oracle-database/26/adfns/flashback.html), [MariaDB — System-Versioned Tables](https://mariadb.com/docs/server/reference/sql-structure/temporal-tables/system-versioned-tables)

---

### 24.5 Multitenancy

[⬆️ Voltar ao Sumário](#sumário)

| Modelo | Isolamento | Operação |
|---|---|---|
| database por tenant | alto | muitas instâncias/migrations |
| schema por tenant | intermediário | catálogo e pooling complexos |
| tabelas compartilhadas | menor | escala simples, exige `tenant_id` onipresente |

Em tabela compartilhada, chaves e unicidades de negócio frequentemente incluem `tenant_id`; FKs compostas evitam referência cruzada. RLS pode reforçar, mas contexto precisa ser definido/resetado com segurança no pool.

Escolha por requisitos legais, noisy neighbor, customização, restore por tenant e volume — não apenas custo inicial.

> **Fontes oficiais:** [Azure SQL — SaaS Tenancy Patterns](https://learn.microsoft.com/en-us/azure/azure-sql/database/saas-tenancy-app-design-patterns), [PostgreSQL — Row Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html), [SQL Server — Row-Level Security](https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security)

---

### 24.6 IDs, clocks e distribuição

[⬆️ Voltar ao Sumário](#sumário)

Identity/sequence, UUID e IDs distribuídos trocam tamanho, locality, coordenação, ordenação e informação vazada. UUID aleatório reduz coordenação, mas amplia índices; UUID time-ordered melhora locality com novas considerações de privacidade/clock e suporte de versão.

Não use ID para inferir tempo ou ordem de negócio sem contrato. Clocks recuam, hosts divergem e timestamp igual não desempata eventos. Para ordenação causal, use versão/sequence por agregado, log position ou protocolo adequado.

> **Fontes oficiais:** [PostgreSQL — UUID Type](https://www.postgresql.org/docs/current/datatype-uuid.html), [SQL Server — uniqueidentifier](https://learn.microsoft.com/en-us/sql/t-sql/data-types/uniqueidentifier-transact-sql), [Oracle — Sequences](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/CREATE-SEQUENCE.html)

---

## Parte 25 — Analytics, Warehousing e Arquiteturas de Dados

**Objetivo da parte:**

Entender como SQL muda de papel em workloads analíticos, particionados, replicados, distribuídos e orientados a grafo.

---

### 25.1 OLTP versus OLAP

[⬆️ Voltar ao Sumário](#sumário)

| Dimensão | OLTP | OLAP |
|---|---|---|
| consultas | curtas e seletivas | scans, joins e agregações amplas |
| escrita | concorrente e frequente | cargas em lote/stream |
| modelo | normalizado por invariantes | dimensional/colunar por análise |
| latência | ms por transação | segundos/minutos por análise |

São extremos de um espectro. HTAP e warehouses modernos misturam recursos, mas isolamento de workload, freshness e custo continuam decisões. Não rode relatório sem limite no primário crítico sem avaliar impacto.

> **Fontes oficiais:** [Oracle — Data Warehousing Guide](https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/), [Microsoft — Data Warehousing and Analytics](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/data/data-warehouse), [PostgreSQL — Parallel Query](https://www.postgresql.org/docs/current/parallel-query.html)

---

### 25.2 Modelagem dimensional

[⬆️ Voltar ao Sumário](#sumário)

Um fato registra evento/medida em uma granularidade declarada; dimensões descrevem entidades/contexto. Star schema reduz joins e favorece BI; surrogate keys permitem histórico de dimensão.

Dimensões conformadas reutilizam a mesma definição entre fatos, permitindo que relatórios comparem processos sem reinterpretar cliente, produto ou calendário.

Declare antes da tabela fato: “uma linha por item de pedido confirmado”. Medidas podem ser aditivas, semiaditivas ou não aditivas. Slowly changing dimensions têm estratégias distintas; escolha pelo tipo de mudança e consulta histórica.

> **Fontes oficiais:** [Microsoft — Star Schema Guidance for Power BI](https://learn.microsoft.com/en-us/power-bi/guidance/star-schema), [Oracle — Data Warehousing Guide](https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/), [Google Cloud — BigQuery Schema Design](https://cloud.google.com/bigquery/docs/best-practices-performance-nested)

---

### 25.3 Particionamento e pruning

[⬆️ Voltar ao Sumário](#sumário)

Particionamento divide logicamente uma tabela por range, list ou hash. Ajuda manutenção, retenção, paralelismo e pruning quando o predicado contém a chave compatível. Não substitui índice e adiciona metadata/planejamento.

Escolha chave estável, alinhada a acesso e ciclo de vida. Muitas partições pequenas pioram overhead; uma partição “default” pode esconder falhas de roteamento. Constraints únicas globais nem sempre estão disponíveis.

> **Fontes oficiais:** [PostgreSQL — Table Partitioning](https://www.postgresql.org/docs/current/ddl-partitioning.html), [SQL Server — Partitioned Tables and Indexes](https://learn.microsoft.com/en-us/sql/relational-databases/partitions/partitioned-tables-and-indexes), [MySQL — Partitioning](https://dev.mysql.com/doc/refman/8.4/en/partitioning.html), [Oracle — Partitioning](https://docs.oracle.com/en/database/oracle/oracle-database/26/vldbg/)

---

### 25.4 Replicação, CDC e event streaming

[⬆️ Voltar ao Sumário](#sumário)

Replicação mantém cópias para HA/leitura; CDC publica mudanças do log; event streaming transporta eventos. Eles não têm o mesmo contrato.

Réplicas podem atrasar: read-after-write não é automático ao redirecionar leitura. CDC normalmente entrega ao menos uma vez e pode exigir ordering/dedup por chave/position. DDL/schema evolution precisa acompanhar consumidores.

Não use log de banco como API sem conector suportado. Proteja PII replicada e dimensione retenção do log conforme consumidores lentos.

> **Fontes oficiais:** [PostgreSQL — Logical Decoding](https://www.postgresql.org/docs/current/logicaldecoding.html), [SQL Server — Change Data Capture](https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-data-capture-sql-server), [MySQL — Replication](https://dev.mysql.com/doc/refman/8.4/en/replication.html), [Oracle — Data Guard](https://docs.oracle.com/en/database/oracle/oracle-database/26/sbydb/)

---

### 25.5 Bancos distribuídos e consistência

[⬆️ Voltar ao Sumário](#sumário)

Distribuir SQL exige replicar dados, eleger líderes, rotear transações e lidar com partições de rede. Produtos oferecem contratos diferentes de serializability, snapshot, follower reads, geo-partitioning e DDL.

Evite slogans de CAP como escolha binária cotidiana. Pergunte: quais operações atravessam shards/regiões, qual stale read é permitido, como clocks entram no protocolo, o que ocorre na indisponibilidade e como restore preserva consistência.

Chave de distribuição determina locality e hotspots; joins/transações cross-shard custam mais. Meça com latência entre regiões real.

> **Fontes oficiais:** [Google Cloud Spanner — Transactions](https://cloud.google.com/spanner/docs/transactions), [CockroachDB — Architecture Overview](https://www.cockroachlabs.com/docs/stable/architecture/overview), [Citus — Data Modeling for Distributed Databases](https://docs.citusdata.com/en/stable/sharding/data_modeling.html)

---

### 25.6 SQL para grafos e SQL/PGQ

[⬆️ Voltar ao Sumário](#sumário)

SQL/PGQ adiciona property graph queries ao SQL:2023. Um property graph mapeia tabelas para vértices/arestas e permite padrões de caminho, mantendo integração com resultados tabulares.

Isso não significa suporte universal nem sintaxe idêntica em extensões anteriores. Para árvore simples, CTE recursiva pode bastar; para travessias variáveis e padrões de caminhos, recursos de grafo podem comunicar melhor.

> **Fontes oficiais:** [ISO/IEC 9075-16:2023 — SQL/PGQ](https://www.iso.org/standard/79473.html), [Oracle — Graph Reference](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/graph-reference.html), [PostgreSQL — Recursive Queries](https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-RECURSIVE)

---

## Parte 26 — Operação, Backup, Recuperação e Alta Disponibilidade

**Objetivo da parte:**

Operar o banco como componente crítico, com recuperação ensaiada, manutenção, failover e capacidade mensuráveis.

---

### 26.1 Backup lógico e físico

[⬆️ Voltar ao Sumário](#sumário)

Backup lógico exporta objetos/linhas e favorece migração seletiva; físico copia páginas/arquivos em formato da versão/plataforma. Snapshot de volume só é consistente se coordenado com o SGBD.

Defina escopo: dados, schema, roles, extensions, encryption keys, jobs e configurações. Proteja backup como produção — ele contém os mesmos segredos. Use checksums, retenção imutável/off-site e inventário.

Export lógico pode ser consistente por snapshot, mas isso precisa ser pedido/garantido pela ferramenta; copiar arquivos de um servidor ativo sem protocolo é insuficiente.

> **Fontes oficiais:** [PostgreSQL — Backup and Restore](https://www.postgresql.org/docs/current/backup.html), [SQL Server — Back Up and Restore](https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases), [MySQL — Backup and Recovery](https://dev.mysql.com/doc/refman/8.4/en/backup-and-recovery.html), [Oracle — Backup and Recovery](https://docs.oracle.com/en/database/oracle/oracle-database/26/bradv/)

---

### 26.2 Restore, PITR, RPO e RTO

[⬆️ Voltar ao Sumário](#sumário)

- **RPO:** quanto dado a organização aceita perder no tempo.
- **RTO:** quanto tempo aceita ficar sem o serviço.
- **PITR:** restaurar base + logs até um ponto/posição.

Backup só é confiável após restore automatizado e validação semântica. Ensaie corrupção, exclusão lógica, perda de região e chaves indisponíveis. Meça tempo de download, replay, checks e reentrada de tráfego.

PITR restaura o banco inteiro ao passado; recuperar uma linha/tabela pode exigir instância auxiliar e reconciliação.

> **Fontes oficiais:** [PostgreSQL — Continuous Archiving and PITR](https://www.postgresql.org/docs/current/continuous-archiving.html), [SQL Server — Restore and Recovery Overview](https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/restore-and-recovery-overview-sql-server), [Oracle — Recovery Concepts](https://docs.oracle.com/en/database/oracle/oracle-database/26/bradv/rman-backup-concepts.html)

---

### 26.3 Replicação e failover

[⬆️ Voltar ao Sumário](#sumário)

Replicação não é backup: exclusão/corrupção pode replicar. Síncrona reduz RPO ao custo de latência/disponibilidade; assíncrona aceita lag. Failover automático precisa de quorum/fencing para evitar split-brain.

Teste failover e **failback**, connection retry, DNS/endpoint, transações em voo e capacidade da réplica. Read replicas exigem política para staleness e consistency.

Depois da promoção, impeça o antigo primário de aceitar escrita até provar a nova topologia; esse fencing evita histórias divergentes difíceis de reconciliar.

> **Fontes oficiais:** [PostgreSQL — High Availability](https://www.postgresql.org/docs/current/high-availability.html), [SQL Server — Always On Availability Groups](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server), [MySQL — Group Replication](https://dev.mysql.com/doc/refman/8.4/en/group-replication.html), [Oracle — Data Guard](https://docs.oracle.com/en/database/oracle/oracle-database/26/sbydb/)

---

### 26.4 Maintenance, vacuum, statistics e storage

[⬆️ Voltar ao Sumário](#sumário)

Cada engine exige manutenção: vacuum/analyze no PostgreSQL, statistics/index maintenance no SQL Server, purge/analyze no InnoDB, optimizer stats/segment management no Oracle. Serviços gerenciados automatizam parte, não a observação.

Monitore bloat/dead tuples, fragmentation relevante, estatísticas, log/undo, tablespaces/files, IOPS e crescimento. Evite jobs universais de rebuild: manutenção também gera I/O, log, locks e replicação lag.

> **Fontes oficiais:** [PostgreSQL — Routine Vacuuming](https://www.postgresql.org/docs/current/routine-vacuuming.html), [SQL Server — Index Maintenance](https://learn.microsoft.com/en-us/sql/relational-databases/indexes/reorganize-and-rebuild-indexes), [MySQL — InnoDB Storage Engine](https://dev.mysql.com/doc/refman/8.4/en/innodb-storage-engine.html), [Oracle — Optimizer Statistics](https://docs.oracle.com/en/database/oracle/oracle-database/26/tgsql/toc.htm)

---

### 26.5 Observabilidade e capacity planning

[⬆️ Voltar ao Sumário](#sumário)

Observe latência p50/p95/p99, throughput, erros, pool wait, sessões, locks/deadlocks, CPU, I/O, cache, memória/temp, log, replication lag, storage e crescimento. Métrica sem workload/tag dificulta atribuição.

Logs de queries precisam de sampling/redaction. Query fingerprints agrupam literais distintos; planos e wait events explicam a causa. Capacidade deve incluir picos, maintenance, failover N-1 e crescimento, não só média.

> **Fontes oficiais:** [PostgreSQL — Monitoring](https://www.postgresql.org/docs/current/monitoring.html), [SQL Server — Monitor and Tune](https://learn.microsoft.com/en-us/sql/relational-databases/performance/monitor-and-tune-for-performance), [MySQL — Performance Schema](https://dev.mysql.com/doc/refman/8.4/en/performance-schema.html), [Oracle — Performance Tuning Guide](https://docs.oracle.com/en/database/oracle/oracle-database/26/tgdba/)

---

## Parte 27 — Engenharia de SQL para Produção

**Objetivo da parte:**

Transformar SQL, schema e operação em artefatos testados, revisáveis, implantáveis e sustentáveis.

---

### 27.1 Testes de schema, queries e migrations

[⬆️ Voltar ao Sumário](#sumário)

Teste em um SGBD real da mesma família/versão, não somente mock:

- constraints e `NULL`/Unicode/timezone;
- cardinalidades vazia, uma e muitas;
- concorrência e retry;
- migrations sobre snapshot representativo;
- permissões negativas;
- plano/tempo com distribuição próxima da produção;
- restore e compatibilidade de rollback.

Testcontainers/containers ajudam integração, mas extensões/configuração/volume também importam. Dados sintéticos devem incluir skew e extremos.

> **Fontes oficiais:** [Testcontainers — Databases](https://testcontainers.com/modules/?category=database), [PostgreSQL — Regression Tests](https://www.postgresql.org/docs/current/regress.html), [SQL Server — SQL Database Projects](https://learn.microsoft.com/en-us/sql/tools/sql-database-projects/sql-database-projects)

---

### 27.2 Revisão, lint e formatação

[⬆️ Voltar ao Sumário](#sumário)

Uma revisão SQL verifica semântica antes de estilo: granularidade, `NULL`, determinismo, concorrência, constraints, segurança, impacto de DDL e plano. Formatação consistente torna joins/predicados auditáveis.

Lint pode detectar `SELECT *`, alias ambíguo e dialeto inválido, mas não conhece a invariável. Configure dialeto/versão e trate formatter como transformação mecânica revisada.

Inclua no pull request: objetivo, antes/depois, volume, plano, locks esperados, migration/backfill, rollback compatível e métricas.

> **Fontes oficiais:** [PostgreSQL — SQL Syntax](https://www.postgresql.org/docs/current/sql-syntax.html), [Microsoft — T-SQL Reference](https://learn.microsoft.com/en-us/sql/t-sql/language-reference), [Oracle — SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/)

---

### 27.3 Performance e regressão de planos

[⬆️ Voltar ao Sumário](#sumário)

Planos mudam com dados, estatísticas, parâmetros, versão e configuração. Capture baseline de queries críticas, planos e métricas; canary upgrades e compare regressões.

Plan forcing/query store/hints podem estabilizar emergência, mas congelam uma decisão e exigem expiração/revisão. Corrija estatística, índice, query ou modelagem quando possível.

Teste throughput sob concorrência: uma query 2× mais rápida isolada pode aumentar locks/CPU total e piorar o sistema.

> **Fontes oficiais:** [SQL Server — Query Store](https://learn.microsoft.com/en-us/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store), [PostgreSQL — pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html), [MySQL — Performance Schema Statement Tables](https://dev.mysql.com/doc/refman/8.4/en/performance-schema-statement-tables.html)

---

### 27.4 Privacidade, retenção e compliance

[⬆️ Voltar ao Sumário](#sumário)

Classifique dados, minimize coleta, defina base/finalidade, acesso, retenção e descarte. Pseudonimização reduz vínculo direto, mas pode continuar sendo dado pessoal; anonimização é requisito forte e contextual.

Propague deleção/retention a réplicas, caches, warehouse, logs e backups conforme política e lei. Separe auditoria necessária de logging indiscriminado. Consulte profissionais jurídicos/segurança para o contexto aplicável.

Crie inventário de campos sensíveis e testes de acesso/expiração; uma cláusula em política sem execução técnica não reduz o dado retido.

> **Fontes oficiais:** [ANPD — Guias e orientações](https://www.gov.br/anpd/pt-br/documentos-e-publicacoes), [NIST — Privacy Framework](https://www.nist.gov/privacy-framework), [PostgreSQL — Row Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)

---

### 27.5 Deploy progressivo e rollback

[⬆️ Voltar ao Sumário](#sumário)

Use expand/contract, feature flags e canary. Backfills devem processar lotes por chave, limitar impacto, registrar checkpoint, ser idempotentes e pausar sob lag/locks.

Rollback de aplicação não implica rollback de schema. Depois que código novo escreveu novo formato, versão antiga talvez não o entenda. Prefira roll-forward compatível; reserve restauração para perda/corrupção com procedimento ensaiado.

Cada etapa deve expor uma métrica de adoção — por exemplo, percentual de linhas preenchidas e leitores ainda usando a coluna antiga.

> **Fontes oficiais:** [PostgreSQL — ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html), [MySQL — Online DDL](https://dev.mysql.com/doc/refman/8.4/en/innodb-online-ddl-operations.html), [SQL Server — Online Index Operations](https://learn.microsoft.com/en-us/sql/relational-databases/indexes/perform-index-operations-online)

---

### 27.6 Runbooks e resposta a incidentes

[⬆️ Voltar ao Sumário](#sumário)

Runbook acionável contém sintomas/alerta, impacto, acessos, comandos read-only primeiro, critérios de decisão, riscos, rollback, comunicação e escalonamento. Teste-o em game days.

Para lentidão: confirme escopo → pool/sessões → waits/locks → queries/plan → recursos → mudanças recentes. Para perda: congele destruição, preserve evidência, determine RPO/ponto e restaure em ambiente isolado antes de reconciliar.

Pós-incidente sem culpa deve gerar correções verificáveis: constraint, teste, alerta, automação, capacity ou simplificação — não apenas “ter mais cuidado”.

> **Fontes oficiais:** [PostgreSQL — Monitoring](https://www.postgresql.org/docs/current/monitoring.html), [SQL Server — Troubleshoot Performance](https://learn.microsoft.com/en-us/troubleshoot/sql/database-engine/performance/troubleshoot-slow-running-queries), [Google SRE — Incident Response](https://sre.google/sre-book/managing-incidents/)

---

## Parte 28 — Catálogo da Linguagem SQL

**Objetivo da parte:**

Servir como mapa de consulta do que já existe na linguagem e nos principais produtos, evitando reinventar operadores, estruturas e serviços prontos.

---

### 28.1 Categorias de statements

[⬆️ Voltar ao Sumário](#sumário)

As siglas abaixo são vocabulário didático/industrial; a classificação normativa da família ISO é mais detalhada e alguns comandos cruzam categorias.

| Família informal | Finalidade | Exemplos |
|---|---|---|
| definição (DDL) | criar/evoluir objetos | `CREATE`, `ALTER`, `DROP` |
| manipulação (DML) | consultar/modificar dados | `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE` |
| transação (TCL) | delimitar unidade | `START TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |
| autorização (DCL) | conceder/remover acesso | `GRANT`, `REVOKE` |
| sessão/conexão | alterar contexto | `SET`, `CONNECT`, formas do produto |
| diagnóstico | obter status/erros | `GET DIAGNOSTICS`, catálogos do produto |
| procedural | módulos/rotinas | `CREATE PROCEDURE`, `CALL`, blocos do dialeto |

Nem toda palavra é statement isolado: `WHERE`, `JOIN` e `OVER` são partes de gramáticas maiores. Consulte o índice de comandos do SGBD-alvo.

> **Fontes oficiais:** [ISO — SQL/Foundation](https://www.iso.org/standard/76584.html), [PostgreSQL — SQL Commands](https://www.postgresql.org/docs/current/sql-commands.html), [SQL Server — T-SQL Reference](https://learn.microsoft.com/en-us/sql/t-sql/language-reference), [MySQL — SQL Statements](https://dev.mysql.com/doc/refman/8.4/en/sql-statements.html)

---

### 28.2 Palavras-chave e por que não há lista universal única

[⬆️ Voltar ao Sumário](#sumário)

O padrão possui palavras reservadas e não reservadas; cada implementação adiciona palavras e decide em quais contextos um nome exige delimitador. Versões novas também reservam termos.

| Grupo de uso | Exemplos representativos |
|---|---|
| consulta | `SELECT`, `FROM`, `WHERE`, `GROUP`, `HAVING`, `ORDER`, `FETCH` |
| composição | `JOIN`, `ON`, `UNION`, `INTERSECT`, `EXCEPT`, `WITH`, `RECURSIVE` |
| expressão | `CASE`, `WHEN`, `THEN`, `ELSE`, `END`, `CAST`, `NULL`, `TRUE`, `FALSE` |
| definição | `CREATE`, `TABLE`, `VIEW`, `SCHEMA`, `DOMAIN`, `ALTER`, `DROP` |
| integridade | `PRIMARY`, `FOREIGN`, `REFERENCES`, `UNIQUE`, `CHECK`, `DEFAULT` |
| transação | `START`, `TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`, `ISOLATION` |
| autorização | `GRANT`, `REVOKE`, `ROLE`, `USER` |
| análise | `OVER`, `PARTITION`, `ROWS`, `RANGE`, `GROUPS` |

Esta lista é intencionalmente um mapa, não a lista normativa completa. Use nomes simples não reservados e verifique o catálogo oficial da versão; não tente memorizar centenas de termos.

> **Fontes oficiais:** [PostgreSQL — SQL Key Words](https://www.postgresql.org/docs/current/sql-keywords-appendix.html), [SQL Server — Reserved Keywords](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/reserved-keywords-transact-sql), [MySQL — Keywords and Reserved Words](https://dev.mysql.com/doc/refman/8.4/en/keywords.html), [SQLite — Keywords](https://sqlite.org/lang_keywords.html)

---

### 28.3 Tipos padronizados e famílias de tipos

[⬆️ Voltar ao Sumário](#sumário)

| Família | Tipos/formas comuns | Para quê |
|---|---|---|
| inteiros | `SMALLINT`, `INTEGER`, `BIGINT` | contagens/IDs dentro da faixa |
| exatos | `NUMERIC(p,s)`, `DECIMAL(p,s)` | dinheiro e medidas decimais |
| aproximados | `REAL`, `FLOAT`, `DOUBLE PRECISION` | ciência/medição aproximada |
| caracteres | `CHAR`, `VARCHAR`, `CLOB` | texto |
| binários | `BINARY`, `VARBINARY`, `BLOB` | bytes/LOB |
| booleano | `BOOLEAN` | verdade SQL ternária |
| temporais | `DATE`, `TIME`, `TIMESTAMP`, `INTERVAL` | datas, horários, instantes/durações |
| estruturados | row, array, multiset, XML/JSON conforme feature | valores compostos/semiestruturados |
| domínio | `CREATE DOMAIN` | tipo nomeado com constraints |

Tipos com mesmo nome podem ter faixa, encoding, timezone ou storage diferentes. Tipos prontos do produto — UUID, spatial, ranges, vector, money, enums — devem ser escolhidos pelo contrato, driver e portabilidade.

> **Fontes oficiais:** [ISO — SQL/Foundation](https://www.iso.org/standard/76584.html), [PostgreSQL — Data Types](https://www.postgresql.org/docs/current/datatype.html), [SQL Server — Data Types](https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-types-transact-sql), [Oracle — Data Types](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Data-Types.html)

---

### 28.4 Operadores e predicados

[⬆️ Voltar ao Sumário](#sumário)

| Intenção | Vocabulário pronto |
|---|---|
| aritmética | `+`, `-`, `*`, `/`, operações módulo do dialeto |
| comparação | `=`, `<>`, `<`, `<=`, `>`, `>=` |
| nulidade | `IS NULL`, `IS NOT NULL`, `IS [NOT] DISTINCT FROM` |
| lógica | `NOT`, `AND`, `OR`, `IS TRUE/FALSE/UNKNOWN` |
| intervalo/lista | `BETWEEN`, `IN` |
| padrão | `LIKE`, `SIMILAR TO` onde suportado |
| quantificação | `EXISTS`, `ANY`/`SOME`, `ALL` |
| conjuntos | `UNION`, `INTERSECT`, `EXCEPT` e versões `ALL` |
| relações | `JOIN`, `NATURAL JOIN`, lateral/aplicação do produto |

Precedência, coerção e overload podem variar. Use parênteses quando misturar lógica e nunca use `NATURAL JOIN` em contratos duradouros: uma nova coluna homônima pode mudar o join sem alterar a query.

> **Fontes oficiais:** [PostgreSQL — Functions and Operators](https://www.postgresql.org/docs/current/functions.html), [SQL Server — Operators](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/operators-transact-sql), [Oracle — Operators](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Operators.html)

---

### 28.5 Funções prontas por domínio

[⬆️ Voltar ao Sumário](#sumário)

Antes de criar uma UDF, procure no padrão/produto:

| Domínio | Funções/construções frequentes |
|---|---|
| texto | `CHAR_LENGTH`, `SUBSTRING`, `TRIM`, `UPPER`, `LOWER`, `POSITION`, `CONCAT` |
| número | `ABS`, `ROUND`, `FLOOR`, `CEILING`, `POWER`, `SQRT`, `MOD` |
| temporal | `CURRENT_DATE`, `CURRENT_TIMESTAMP`, `EXTRACT`, interval arithmetic |
| nulos/condição | `COALESCE`, `NULLIF`, `CASE` |
| conversão | `CAST`, `TRY_CAST`/formas tolerantes do produto |
| agregação | `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, string/JSON aggregate do produto |
| janela | `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, `LEAD`, `FIRST_VALUE`, `LAST_VALUE` |
| estatística | desvio, variância, correlação/regressão conforme feature/produto |
| JSON/XML | constructors, query/value/exists/aggregate conforme dialeto |
| segurança/contexto | current user/role/schema/session conforme produto |

O mesmo nome não prova mesma semântica. Verifique `NULL`, collation, tipo do retorno, determinismo e versão.

> **Fontes oficiais:** [PostgreSQL — Functions and Operators](https://www.postgresql.org/docs/current/functions.html), [SQL Server — Built-in Functions](https://learn.microsoft.com/en-us/sql/t-sql/functions/functions), [MySQL — Functions and Operators](https://dev.mysql.com/doc/refman/8.4/en/functions.html), [Oracle — Functions](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Functions.html), [SQLite — Built-In Functions](https://sqlite.org/lang_corefunc.html)

---

### 28.6 Objetos de schema

[⬆️ Voltar ao Sumário](#sumário)

| Objeto | Papel |
|---|---|
| schema/catalog | namespace e organização |
| table | relação persistente/temporária |
| view/materialized view | abstração lógica/resultado armazenado |
| constraint | invariável declarativa |
| domain/type | vocabulário de valores |
| sequence/identity | geração numérica |
| routine/module | operação armazenada |
| trigger | reação a evento |
| index | estrutura física de acesso; em geral extensão de implementação |
| role/privilege | autorização |
| collation | regras de comparação textual |

Synonyms, packages, tablespaces, filegroups, publications e extensions pertencem a produtos. Trate objetos como código versionado e ownership como segurança.

> **Fontes oficiais:** [ISO — SQL/Schemata](https://www.iso.org/standard/76586.html), [PostgreSQL — Data Definition](https://www.postgresql.org/docs/current/ddl.html), [SQL Server — Database Engine Objects](https://learn.microsoft.com/en-us/sql/relational-databases/databases/databases), [Oracle — Database Objects](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Database-Objects.html)

---

### 28.7 Recursos analíticos e relacionais

[⬆️ Voltar ao Sumário](#sumário)

Não reinvente no cliente o que o banco já executa em conjuntos:

- joins internos/externos e semijoin por `EXISTS`;
- grouping sets, rollup e cube;
- window ranking, navegação, frames e acumulados;
- CTEs recursivas para hierarquias;
- `MERGE`/upsert conforme contrato;
- `MATCH_RECOGNIZE` para row pattern recognition onde suportado;
- temporal, JSON, XML e property graph conforme feature;
- pivot/unpivot como extensões de alguns produtos.

“Disponível” não significa que deve ser usado: legibilidade, índice, volume, portabilidade e operação decidem.

> **Fontes oficiais:** [ISO — SQL/Foundation](https://www.iso.org/standard/76584.html), [PostgreSQL — Queries](https://www.postgresql.org/docs/current/queries.html), [Oracle — Pattern Matching](https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/sql-pattern-matching-data-warehouses.html), [SQL Server — PIVOT and UNPIVOT](https://learn.microsoft.com/en-us/sql/t-sql/queries/from-using-pivot-and-unpivot)

---

### 28.8 Metadados e diagnóstico

[⬆️ Voltar ao Sumário](#sumário)

| Pergunta | Interface preferida |
|---|---|
| quais tabelas/colunas existem? | `INFORMATION_SCHEMA` |
| quais índices/planos/estatísticas? | catálogo/DMV do produto |
| quem bloqueia quem? | activity/lock views do produto |
| qual erro ocorreu? | SQLSTATE + vendor code + diagnostics |
| qual versão/feature? | função/view oficial de versão/features |
| qual sessão/configuração? | contexto e settings documentados |

Ferramentas gráficas leem essas interfaces; saiba consultar a fonte quando a UI omite detalhe. Restrinja metadata sensível e sanitize query text.

> **Fontes oficiais:** [ISO — SQL/Schemata](https://www.iso.org/standard/76586.html), [PostgreSQL — System Catalogs](https://www.postgresql.org/docs/current/catalogs.html), [SQL Server — System Catalog Views](https://learn.microsoft.com/en-us/sql/relational-databases/system-catalog-views/catalog-views-transact-sql), [MySQL — INFORMATION_SCHEMA](https://dev.mysql.com/doc/refman/8.4/en/information-schema.html)

---

### 28.9 Matriz rápida de dialetos

[⬆️ Voltar ao Sumário](#sumário)

| Necessidade | PostgreSQL | SQL Server | MySQL | Oracle | SQLite |
|---|---|---|---|---|---|
| limite comum | `LIMIT`/`FETCH` | `TOP`/`OFFSET FETCH` | `LIMIT` | `FETCH` | `LIMIT` |
| auto-ID comum | identity/sequence | `IDENTITY`/sequence | `AUTO_INCREMENT` | identity/sequence | `INTEGER PRIMARY KEY` |
| upsert comum | `ON CONFLICT`/`MERGE` | `MERGE` ou padrão transacional | `ON DUPLICATE KEY` | `MERGE` | `ON CONFLICT` |
| retorno de DML | `RETURNING` | `OUTPUT` | recursos/versão específicos | `RETURNING INTO` | `RETURNING` |
| string aggregate | `string_agg` | `STRING_AGG` | `GROUP_CONCAT` | `LISTAGG` | `group_concat`/`string_agg` |
| procedure dialect | PL/pgSQL etc. | T-SQL | SQL/PSM próprio | PL/SQL | sem stored procedures nativas |
| catálogos | `pg_catalog` | catalog views/DMVs | dictionary/Performance Schema | dictionary views | `sqlite_schema`/PRAGMA |

Esta matriz orienta pesquisa, não garante equivalência nem disponibilidade em qualquer versão. Consulte a página oficial indicada em 28.10.

> **Fontes oficiais:** [PostgreSQL — SQL Commands](https://www.postgresql.org/docs/current/sql-commands.html), [SQL Server — T-SQL](https://learn.microsoft.com/en-us/sql/t-sql/language-reference), [MySQL 8.4 Reference](https://dev.mysql.com/doc/refman/8.4/en/), [Oracle 26 SQL Reference](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/), [SQLite — SQL Syntax](https://sqlite.org/lang.html)

---

### 28.10 Como verificar disponibilidade e semântica

[⬆️ Voltar ao Sumário](#sumário)

Para qualquer recurso:

1. identifique produto, edição, versão e compatibility mode;
2. abra a referência oficial dessa versão;
3. confirme gramática, tipo de retorno, `NULL`, concorrência e privilégios;
4. verifique release notes/deprecation;
5. construa um teste mínimo com casos extremos;
6. examine o plano quando desempenho importa;
7. registre a dependência de dialeto no código/documentação.

Não use resposta genérica da internet como contrato de produção. SQL padrão, implementação e driver são três camadas diferentes.

> **Fontes oficiais:** [PostgreSQL — Documentation](https://www.postgresql.org/docs/), [SQL Server — Documentation](https://learn.microsoft.com/en-us/sql/), [MySQL — Reference Manuals](https://dev.mysql.com/doc/), [Oracle Database — Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/), [SQLite — Documentation](https://sqlite.org/docs.html)

---

## Parte 29 — SGBDs, Ecossistema e Ferramentas Externas

**Objetivo da parte:**

Mapear produtos e ferramentas ao redor de SQL sem apresentar ecossistema como parte da linguagem nem produzir uma lista de recomendações sem critérios.

---

### 29.1 Padrão, produto, serviço e ferramenta

[⬆️ Voltar ao Sumário](#sumário)

| Camada | Exemplo | Quem define o contrato |
|---|---|---|
| padrão | ISO/IEC 9075 | ISO/IEC |
| SGBD | PostgreSQL, SQL Server, MySQL, Oracle, SQLite | fornecedor/projeto |
| serviço gerenciado | Azure SQL, Amazon RDS, Cloud SQL | provedor + engine |
| driver/API | JDBC, ODBC, ADO.NET/provider | especificação + driver |
| ferramenta | migration, IDE, observabilidade | projeto/fornecedor |

Um serviço gerenciado não elimina limites da engine e acrescenta quotas, rede, HA, backup e versões próprias. Registre exatamente a camada escolhida.

> **Fontes oficiais:** [ISO — SQL](https://www.iso.org/standard/76583.html), [PostgreSQL — What is PostgreSQL?](https://www.postgresql.org/about/), [Microsoft — SQL Documentation](https://learn.microsoft.com/en-us/sql/), [SQLite — About](https://sqlite.org/about.html)

---

### 29.2 SGBDs relacionais gerais

[⬆️ Voltar ao Sumário](#sumário)

| Produto/projeto | Perfil resumido | Documentação oficial |
|---|---|---|
| PostgreSQL | open source extensível, forte SQL e tipos | [Manual](https://www.postgresql.org/docs/current/) |
| SQL Server/Azure SQL | ecossistema Microsoft, T-SQL e tooling | [Docs](https://learn.microsoft.com/en-us/sql/) |
| MySQL | open source/comercial, amplo uso web, InnoDB | [8.4 Reference](https://dev.mysql.com/doc/refman/8.4/en/) |
| MariaDB | fork independente com dialeto divergente | [Docs](https://mariadb.com/docs/) |
| Oracle Database | plataforma comercial, SQL/PLSQL e recursos enterprise | [26ai Docs](https://docs.oracle.com/en/database/oracle/oracle-database/26/) |
| IBM Db2 | SQL empresarial e várias plataformas | [Docs](https://www.ibm.com/docs/en/db2) |

Não escolha por ranking. Compare workload, equipe, operação, licença, HA, extensões, compatibilidade, suporte e custo total com prova de conceito.

---

### 29.3 Bancos embarcados

[⬆️ Voltar ao Sumário](#sumário)

SQLite é biblioteca serverless/in-process e arquivo único, apropriada para apps locais, dispositivos, testes e vários serviços de baixa/moderada concorrência. Seu typing, locking e deployment diferem de servidores cliente-servidor.

DuckDB é banco analítico in-process orientado a workloads OLAP e integração com arquivos/frames; não substitui automaticamente um OLTP multiusuário. H2/Derby são opções Java com contratos próprios e não simulam perfeitamente PostgreSQL/Oracle em testes.

> **Fontes oficiais:** [SQLite — Appropriate Uses](https://sqlite.org/whentouse.html), [DuckDB — Documentation](https://duckdb.org/docs/stable/), [Apache Derby — Documentation](https://db.apache.org/derby/manuals/), [H2 — Documentation](https://h2database.com/html/main.html)

---

### 29.4 Cloud data warehouses e lakehouses

[⬆️ Voltar ao Sumário](#sumário)

BigQuery, Snowflake, Amazon Redshift, Azure Synapse e Databricks SQL oferecem dialetos e arquiteturas analíticas. Cobrança por bytes/compute, separation of storage/compute, clustering, caching e concurrency mudam o tuning.

Lakehouse combina formatos de tabela em object storage com engines de consulta; transações e schema evolution pertencem ao formato/catalog/engine. “SQL compatível” não significa que indexes, constraints ou procedures de OLTP existam.

> **Fontes oficiais:** [BigQuery — GoogleSQL Reference](https://cloud.google.com/bigquery/docs/reference/standard-sql), [Snowflake — SQL Reference](https://docs.snowflake.com/en/sql-reference), [Amazon Redshift — SQL Reference](https://docs.aws.amazon.com/redshift/latest/dg/c_SQL_reference.html), [Databricks SQL — Language Reference](https://docs.databricks.com/aws/en/sql/language-manual/)

---

### 29.5 Distributed SQL e NewSQL

[⬆️ Voltar ao Sumário](#sumário)

Google Cloud Spanner, CockroachDB, YugabyteDB e arquiteturas com Citus distribuem dados preservando graus distintos de SQL/transações. Diferenças aparecem em chaves, locality, isolation, sequences, DDL e recursos do dialeto.

“PostgreSQL-compatible” costuma significar protocolo/sintaxe parcial, não compatibilidade binária/semântica total. Rode suite real e leia a matriz de features.

Teste também comportamento de falha: mudança de líder, partição de rede, transação cross-region e rebalanceamento costumam revelar o custo arquitetural real.

> **Fontes oficiais:** [Cloud Spanner — SQL](https://cloud.google.com/spanner/docs/reference/standard-sql/overview), [CockroachDB — SQL Feature Support](https://www.cockroachlabs.com/docs/stable/sql-feature-support), [YugabyteDB — YSQL](https://docs.yugabyte.com/stable/api/ysql/), [Citus — Documentation](https://docs.citusdata.com/)

---

### 29.6 Clientes e IDEs

[⬆️ Voltar ao Sumário](#sumário)

Ferramentas oficiais incluem `psql`/pgAdmin, SQL Server Management Studio, Visual Studio Code com a extensão MSSQL, MySQL Shell/Workbench, SQLcl/SQL Developer e `sqlite3`. O Azure Data Studio foi aposentado em 28 de fevereiro de 2026 e não recebe correções; a Microsoft orienta migrar para VS Code/MSSQL ou usar outra ferramenta suportada. Clientes externos incluem DBeaver, DataGrip e extensões de editores.

Avalie suporte ao dialeto, explain visual, diff seguro, SSH/TLS, vault, redaction, export e licenciamento. Configure produção read-only por padrão e destaque ambiente visualmente; UI não substitui revisão de comando destrutivo.

> **Fontes oficiais:** [PostgreSQL — psql](https://www.postgresql.org/docs/current/app-psql.html), [Microsoft — aposentadoria do Azure Data Studio](https://learn.microsoft.com/en-us/sql/tools/whats-happening-azure-data-studio?view=sql-server-ver17), [Microsoft — SQL Tools](https://learn.microsoft.com/en-us/sql/tools/overview-sql-tools), [MySQL Shell](https://dev.mysql.com/doc/mysql-shell/8.4/en/), [Oracle SQLcl](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/), [SQLite CLI](https://sqlite.org/cli.html)

---

### 29.7 Migration, transformação e qualidade

[⬆️ Voltar ao Sumário](#sumário)

| Categoria | Exemplos externos | Uso |
|---|---|---|
| migration | Flyway, Liquibase, DbUp, Alembic | versionar DDL/DML de evolução |
| transformação | dbt | modelos analíticos e testes no warehouse |
| lint/format | SQLFluff e ferramentas do dialeto | consistência/análise estática |
| data quality | Great Expectations, Soda | expectativas e perfil de dados |

Ferramenta não substitui migration compatível nem constraint. Fixe versão, configure dialeto, revise SQL gerado e saiba sair do produto.

> **Documentações oficiais:** [Flyway](https://documentation.red-gate.com/flyway), [Liquibase](https://docs.liquibase.com/), [dbt](https://docs.getdbt.com/docs/introduction), [SQLFluff](https://docs.sqlfluff.com/), [Great Expectations](https://docs.greatexpectations.io/)

---

### 29.8 Observabilidade e segurança

[⬆️ Voltar ao Sumário](#sumário)

APM/OpenTelemetry correlaciona trace da aplicação com chamada SQL; exporters/agentes coletam métricas; DAM/auditoria observa acesso. Ferramentas externas precisam de privilégio mínimo e tratamento de query text/PII.

Recursos nativos — Query Store, `pg_stat_statements`, Performance Schema, AWR/licenciamento aplicável — frequentemente dão a evidência mais próxima. Centralize alertas sem perder detalhes do engine.

Defina retenção e acesso da telemetria: statements e bind values podem conter dados pessoais, tokens ou filtros comercialmente sensíveis.

> **Fontes oficiais:** [OpenTelemetry — Database Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/database/), [PostgreSQL — pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html), [SQL Server — Query Store](https://learn.microsoft.com/en-us/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store), [MySQL — Performance Schema](https://dev.mysql.com/doc/refman/8.4/en/performance-schema.html)

---

### 29.9 Como escolher e adotar

[⬆️ Voltar ao Sumário](#sumário)

Use uma matriz ponderada e uma prova de conceito:

1. correção transacional e recursos SQL necessários;
2. latência/throughput com dados e concorrência realistas;
3. HA, backup/restore e disaster recovery;
4. segurança, compliance e residência;
5. experiência/contratação e ecossistema;
6. portabilidade e custo de saída;
7. licença, suporte e custo total;
8. roadmap/deprecation e cadência de upgrades.

Padronize versões, configuração, templates de schema, observabilidade, backup, migration e resposta a incidentes. Adotar “SQL” sem adotar operação deixa a parte mais arriscada sem dono.

> **Fontes oficiais:** [PostgreSQL — Supported Versions](https://www.postgresql.org/support/versioning/), [SQL Server — Lifecycle](https://learn.microsoft.com/en-us/sql/sql-server/sql-server-get-help), [MySQL — Release Model](https://dev.mysql.com/doc/refman/8.4/en/mysql-releases.html), [Oracle — Database Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/)

---

## Anexo A — Trilhas Oficiais de Estudo e Prática

[⬆️ Voltar ao Sumário](#sumário)

## Trilha A1 — Primeira semana

1. Partes 1–4: linguagem, modelo, chaves, tipos e `NULL`.
2. Partes 5–8: criar schema, gravar e consultar com filtros.
3. Parte 9–10: joins e agregação com cardinalidade declarada.
4. Prática: reproduzir o modelo em SQLite/PostgreSQL e escrever vinte consultas pequenas.

Materiais: [PostgreSQL Tutorial](https://www.postgresql.org/docs/current/tutorial.html), [SQLite Quickstart](https://sqlite.org/quickstart.html), [Microsoft — Query Data with T-SQL](https://learn.microsoft.com/en-us/training/paths/get-started-querying-with-transact-sql/).

## Trilha A2 — Desenvolvimento profissional

1. Partes 11–16: composição, janelas, integridade e views.
2. Partes 17–20: transações, concorrência, planos e segurança.
3. Parte 23: driver da sua stack e integração testada.
4. Prática: implementar pedido concorrente, paginação keyset e migration expand/contract.

Materiais: [PostgreSQL SQL Language](https://www.postgresql.org/docs/current/sql.html), [SQL Server Database Engine](https://learn.microsoft.com/en-us/sql/database-engine/), [MySQL Tutorial](https://dev.mysql.com/doc/refman/8.4/en/tutorial.html), [Oracle SQL Reference](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/).

## Trilha A3 — Produção e arquitetura

1. Partes 24–27: modelagem, analytics, operação e engenharia.
2. Escolha um SGBD e estude locking, backup/PITR, replication e monitoring na documentação daquela versão.
3. Faça restore drill, failover drill e load test.
4. Construa runbooks e SLOs com evidência.

## Projeto final sugerido

Construa um pequeno sistema de pedidos com:

- schema normalizado e constraints nomeadas;
- migration compatível e seed de teste;
- escrita idempotente e concorrência de estoque;
- consultas de relatório com CTE/janela;
- acesso parametrizado por uma das três APIs;
- roles runtime/migration/read-only;
- plano de índices comprovado por `EXPLAIN`;
- backup restaurado e runbook de incidente.

---

## Anexo B — Referências Oficiais Consultadas

[⬆️ Voltar ao Sumário](#sumário)

## Padrão internacional

- [ISO/IEC 9075-1:2023 — SQL/Framework](https://www.iso.org/standard/76583.html)
- [ISO/IEC 9075-2:2023 — SQL/Foundation](https://www.iso.org/standard/76584.html)
- [ISO/IEC 9075-4:2023 — SQL/PSM](https://www.iso.org/standard/76585.html)
- [ISO/IEC 9075-11:2023 — SQL/Schemata](https://www.iso.org/standard/76586.html)
- [ISO/IEC 9075-16:2023 — SQL/PGQ](https://www.iso.org/standard/79473.html)

## Implementações de referência prática

- [PostgreSQL — documentação atual](https://www.postgresql.org/docs/current/)
- [Microsoft SQL / T-SQL — documentação](https://learn.microsoft.com/en-us/sql/)
- [MySQL 8.4 Reference Manual](https://dev.mysql.com/doc/refman/8.4/en/)
- [Oracle AI Database 26ai — SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/)
- [SQLite — SQL Language](https://sqlite.org/lang.html)

## APIs e segurança

- [JDBC API](https://docs.oracle.com/en/java/javase/25/docs/api/java.sql/module-summary.html)
- [ADO.NET](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/)
- [ODBC Reference](https://learn.microsoft.com/en-us/sql/odbc/reference/)
- [OWASP — SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)

> Links do guia apontam para páginas específicas e têm precedência sobre este índice resumido. Para produção, fixe a documentação da versão realmente implantada.

---

## Glossário

[⬆️ Voltar ao Sumário](#sumário)

| Termo | Definição didática |
|---|---|
| ACID | atomicidade, consistência, isolamento e durabilidade de transações |
| agregação | redução de linhas a métricas por conjunto/grupo |
| alias | nome temporário dado a tabela, coluna ou expressão |
| antijoin | linhas da esquerda sem correspondência; frequentemente `NOT EXISTS` |
| autocommit | modo em que cada statement forma sua própria transação |
| backup lógico | exportação em objetos/instruções/linhas, independente do layout físico |
| backup físico | cópia consistente de páginas/arquivos da engine |
| bag semantics | semântica SQL que preserva duplicatas por padrão |
| binding | associação de um valor tipado a um parâmetro |
| B-tree | estrutura de índice comum para igualdade, faixas e ordenação |
| cardinalidade | quantidade de linhas ou multiplicidade de um relacionamento |
| catálogo | metadados mantidos pelo SGBD sobre objetos e estado |
| CDC | captura de mudanças, normalmente a partir do log transacional |
| chave candidata | conjunto mínimo de atributos que identifica uma linha |
| chave estrangeira (FK) | constraint que exige referência válida a chave de outra relação |
| chave natural | chave derivada do domínio de negócio |
| chave primária (PK) | chave escolhida como identificador principal da tabela |
| chave substituta | identificador artificial sem significado de negócio intrínseco |
| collation | regras de comparação e ordenação de texto |
| commit | confirmação de uma transação |
| connection pool | conjunto de conexões reutilizadas por aplicações |
| constraint | regra declarativa que restringe estados do banco |
| covering index | índice que contém dados necessários para responder à consulta sem outro acesso, conforme engine |
| CTE | resultado nomeado com `WITH`, recursivo ou não |
| cursor | mecanismo para percorrer um result set |
| DDL | rótulo informal para statements de definição de dados |
| deadlock | ciclo de espera entre transações/recursos |
| desnormalização | redundância deliberada para requisito mensurado |
| dialeto | extensão/variação de SQL implementada por um produto |
| DML | rótulo informal para consulta e modificação de dados |
| domain | tipo nomeado com restrições no modelo SQL |
| driver | componente que traduz API da aplicação para protocolo do SGBD |
| execution plan | estratégia física escolhida para executar uma instrução |
| expand/contract | evolução de schema em fases compatíveis de adição e remoção |
| fan-out | multiplicação de linhas causada pela cardinalidade de um join |
| frame | subconjunto da partição usado por uma função de janela |
| full outer join | join que preserva linhas sem par dos dois lados |
| função de janela | cálculo analítico que preserva cada linha via `OVER` |
| generated column | coluna calculada pelo SGBD a partir de expressão |
| granularidade | o que exatamente cada linha do resultado representa |
| idempotência | propriedade de repetir operação sem duplicar efeito de negócio |
| identity | mecanismo de geração de valores de coluna, frequentemente IDs |
| índice | estrutura auxiliar para acesso/ordem/unicidade, com custo de escrita |
| inner join | join que conserva apenas pares correspondentes |
| isolation level | contrato de visibilidade/interferência entre transações |
| keyset pagination | paginação que retoma a partir da última chave ordenada |
| lateral | subquery no `FROM` que pode referenciar itens anteriores |
| left join | join que preserva todas as linhas da esquerda |
| lock | proteção concorrente sobre recurso lógico/físico |
| materialized view | resultado de consulta armazenado e atualizado por refresh/manutenção |
| migration | transformação versionada do schema/dados |
| MVCC | controle concorrente por múltiplas versões de dados |
| normalização | decomposição guiada por dependências para reduzir anomalias |
| `NULL` | marcador SQL de ausência do valor |
| OLAP | workload analítico de scans, agregações e grandes volumes |
| OLTP | workload transacional de operações curtas e concorrentes |
| otimizador | componente que compara estratégias e escolhe um plano |
| outer join | join que preserva linhas sem correspondência de um ou ambos os lados |
| parâmetro | placeholder cujo valor é vinculado separadamente do SQL |
| partição de janela | grupo independente sobre o qual uma janela calcula |
| particionamento | divisão de tabela/índice por chave e regras físicas/lógicas |
| PITR | recuperação até um ponto no tempo usando backup e logs |
| predicate | expressão que resulta em `TRUE`, `FALSE` ou `UNKNOWN` |
| prepared statement | instrução cuja estrutura é preparada e valores são vinculados |
| primary key | veja chave primária |
| pruning | eliminação de partições irrelevantes pelo otimizador |
| query | expressão/instrução que produz um resultado relacional |
| RLS | row-level security, política de acesso por linha |
| rollback | desfazimento de uma transação ou até savepoint |
| RPO | perda máxima de dados aceitável medida no tempo |
| RTO | tempo máximo aceitável para restaurar o serviço |
| sargable | predicado que pode orientar acesso eficiente, termo informal |
| savepoint | ponto nomeado para rollback parcial dentro de transação |
| schema | namespace de objetos no catálogo; significado exato varia por produto |
| semijoin | linhas da esquerda com ao menos uma correspondência, como `EXISTS` |
| sequence | objeto que gera valores numéricos, geralmente independente da tabela |
| serialization failure | abort necessário para preservar isolamento serializável |
| SGBD | sistema gerenciador de banco de dados |
| SQL | linguagem padronizada da família ISO/IEC 9075 |
| SQL injection | alteração maliciosa da estrutura SQL por entrada tratada como código |
| SQLSTATE | código estruturado de cinco caracteres para condição SQL |
| statement | unidade sintática executável da linguagem |
| statistics | metadata sobre distribuição usada pelo otimizador |
| surrogate key | veja chave substituta |
| three-valued logic | lógica SQL com `TRUE`, `FALSE` e `UNKNOWN` |
| transaction | unidade de trabalho confirmada ou desfeita conforme contrato |
| trigger | rotina disparada automaticamente por evento no banco |
| two-phase commit | protocolo de preparação e confirmação entre participantes |
| unique constraint | regra de unicidade sobre uma chave candidata |
| upsert | operação que insere ou trata conflito com atualização/ação |
| vacuum | manutenção de versões/visibilidade e espaço no PostgreSQL/SQLite, com contratos distintos |
| view | consulta nomeada que atua como relação virtual |
| window frame | veja frame |

---

> **Encerramento:** dominar SQL é aprender semântica relacional, dados ausentes, concorrência, desempenho e operação — não apenas memorizar comandos. Comece pelo padrão mental comum, escolha deliberadamente um dialeto e valide cada contrato na documentação oficial da versão em produção.
