+++
date = "2017-01-18T12:39:04+03:00"
title = "Полная версия SQL запроса из плана выполнения"
+++

Бывает, что запросы не помещаются в стандартный вывод SMSS или XML версию плана выполнения запроса. Для этого нужно обратиться к кэшу на прямую и взять необходимые поля, в которых уж точно будет полный запрос а не его часть. 

```
SELECT 
    cp.objtype AS ObjectType,
    OBJECT_NAME(st.objectid,st.dbid) AS ObjectName,
    cp.usecounts AS ExecutionCount,
    st.TEXT AS QueryText,
    qp.query_plan AS QueryPlan
FROM 
    sys.dm_exec_cached_plans AS cp
    CROSS APPLY sys.dm_exec_query_plan(cp.plan_handle) AS qp
    CROSS APPLY sys.dm_exec_sql_text(cp.plan_handle) AS st
```