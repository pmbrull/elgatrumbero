title: Revisar l'Execution Plan
date: 2021-01-19 17:10:00
categories:
  - SQL
---

Quan tenim alguna consulta a la base de dades que no performa com esperem, una bona estratègia és revisar l'Execution Plan per a què sigui SQL qui ens digui què no acaba de funcionar.

A part de les eines de monitorització i anàlisis que ja té internes el *Microsoft SQL Studio Management*, una altra bona alternativa és [SentryOne](https://www.sentryone.com/plan-explorer).

Amb aquesta eina gratuïta podem analitzar qualsevol query passant-li el pla en format XML. Per a extreure el pla, només cal córrer (per MS SQL) la següent consulta:

```sql
SELECT
[qsq].[query_id],
[qsp].[plan_id],
[rs].[last_execution_time],
[rs].[avg_duration],
[rs].[avg_logical_io_reads],
[qst].[query_sql_text],
TRY_CONVERT(XML, [qsp].[query_plan]) AS [QueryPlan_XML]
FROM [sys].[query_store_query] [qsq]
JOIN [sys].[query_store_query_text] [qst]
ON [qsq].[query_text_id] = [qst].[query_text_id]
JOIN [sys].[query_store_plan] [qsp]
ON [qsq].[query_id] = [qsp].[query_id]
JOIN [sys].[query_store_runtime_stats] [rs]
ON [qsp].[plan_id] = [rs].[plan_id]
WHERE [qst].[query_sql_text] LIKE '%<troç de la query a analitzar>%'
order by last_execution_time desc
;
```
