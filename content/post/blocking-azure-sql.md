+++
title = "Блокировка в Azure SQL"
date = "2017-01-15T17:59:34+03:00"

+++

Херня случается, что ж поделать, вот так работает система десятилетие и тут бац половину силектов отвалилось. Когда у вас SQL Server от Azure в виде сервиса, то поддержка части DMV's и DMF's в GUI не реализована, да и SQL не перегрузишь для временного решения этой проблемы. Необходимо разобраться кто и кого блокирует. Но если кого, можно выяснить простым запросом и понять какие таблица золочены, то узнать кем золочено можно из этого запроса:

```
WITH Blockers AS
    (select DISTINCT blocking_session_id as session_id
 from sys.dm_exec_requests
 where blocking_session_id > 0
)
SELECT 'Blocker' as type_desc
 , sys.dm_exec_sessions.session_id
 , sys.dm_exec_requests.start_time
 , sys.dm_exec_requests.status
 , sys.dm_exec_requests.command
 , sys.dm_exec_requests.wait_type
 , sys.dm_exec_requests.wait_time
 , sys.dm_exec_requests.blocking_session_id
 , '' AS stmt_text
FROM sys.dm_exec_sessions
LEFT JOIN sys.dm_exec_requests ON sys.dm_exec_requests.session_id = sys.dm_exec_sessions.session_id
INNER JOIN Blockers ON Blockers.session_id = sys.dm_exec_sessions.session_id
UNION
SELECT 'Victim' as type_desc
 , sys.dm_exec_sessions.session_id
 , sys.dm_exec_requests.start_time
 , sys.dm_exec_requests.status
 , sys.dm_exec_requests.command
 , sys.dm_exec_requests.wait_type
 , sys.dm_exec_requests.wait_time
 , sys.dm_exec_requests.blocking_session_id
 , ST.text AS stmt_text
FROM sys.dm_exec_sessions
INNER JOIN sys.dm_exec_requests ON sys.dm_exec_requests.session_id = sys.dm_exec_sessions.session_id
CROSS APPLY SYS.DM_EXEC_SQL_TEXT(sys.dm_exec_requests.sql_handle) AS ST
WHERE blocking_session_id > 0
```

В выводе ищем пару Blocker-Victim с одинаковым blocking_session_id, пишем KILL session_id и готово, блокировка снята. Далее разбирайтесь какой запрос вызвал блокировку и исправляйте проблему. В stmt_text можно увидеть запрос, который привел к блокировке, какой тип блокировки был наложен и какой был тип команды. 
