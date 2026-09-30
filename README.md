# Blog_data
[![Medium Blog Stats](https://mediumblog-cards.vercel.app/getMediumBlogs?username=wondts)](https://medium.com/@wondts)
USE [YourActualDatabaseName]; -- Change to your target DB
GO

SELECT 
    q.query_id,
    p.plan_id,
    OBJECT_NAME(q.object_id) AS proc_name,
    SUBSTRING(t.query_sql_text, (q.statement_sql_handle_start_offset/2)+1,   
        (((CASE q.statement_sql_handle_end_offset   
          WHEN -1 THEN DATALENGTH(t.query_sql_text)  
         ELSE q.statement_sql_handle_end_offset  
         END) - q.statement_sql_handle_start_offset)/2) + 1) AS statement_text,
    CAST(p.query_plan AS XML) AS query_plan_xml,
    rs.count_executions,
    rs.avg_duration / 1000.0 AS avg_duration_ms,
    rs.avg_cpu_time / 1000.0 AS avg_cpu_ms,
    rs.avg_logical_io_reads,
    rs.avg_rowcount,
    rs.last_execution_time
FROM sys.query_store_query q
JOIN sys.query_store_plan p 
    ON q.query_id = p.query_id
JOIN sys.query_store_query_text t 
    ON q.query_text_id = t.query_text_id
LEFT JOIN sys.query_store_runtime_stats rs 
    ON p.plan_id = rs.plan_id
WHERE q.object_id = OBJECT_ID('dbo.usp_YourProcName') -- Put your SP name here
ORDER BY rs.avg_duration DESC;



SELECT 
    r.session_id,
    r.status,
    r.blocking_session_id,
    r.wait_type,
    r.wait_time / 1000 AS wait_time_sec,
    r.cpu_time,
    r.total_elapsed_time / 1000 AS elapsed_sec,
    r.reads,
    r.writes,
    SUBSTRING(st.text, (r.statement_start_offset/2)+1,   
        (((CASE r.statement_end_offset   
          WHEN -1 THEN DATALENGTH(st.text)  
         ELSE r.statement_end_offset  
         END) - r.statement_start_offset)/2) + 1) AS active_statement_text,
    CAST(qp.query_plan AS XML) AS live_query_plan_xml
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) st
CROSS APPLY sys.dm_exec_text_query_plan(r.plan_handle, r.statement_start_offset, r.statement_end_offset) qp
WHERE r.session_id <> @@SPID;

SELECT TOP 10
    q.query_id,
    p.plan_id,
    t.query_sql_text,
    CAST(p.query_plan AS XML) AS query_plan_xml
FROM sys.query_store_query q
JOIN sys.query_store_plan p ON q.query_id = p.query_id
JOIN sys.query_store_query_text t ON q.query_text_id = t.query_text_id
WHERE t.query_sql_text LIKE '%UniqueTableNameOrColumn%'
ORDER BY q.last_execution_time DESC;

USE [YourActualDatabaseName]; -- Change to your DB
GO

DECLARE @ProcName NVARCHAR(256) = 'dbo.usp_YourProcName'; -- Include schema if not dbo

SELECT 
    q.query_id,
    p.plan_id,
    OBJECT_NAME(q.object_id) AS proc_name,
    SUBSTRING(t.query_sql_text, (q.statement_sql_handle_start_offset/2)+1,   
        (((CASE q.statement_sql_handle_end_offset   
          WHEN -1 THEN DATALENGTH(t.query_sql_text)  
         ELSE q.statement_sql_handle_end_offset  
         END) - q.statement_sql_handle_start_offset)/2) + 1) AS statement_text,
    CAST(p.query_plan AS XML) AS query_plan_xml,
    rs.count_executions,
    rs.avg_duration / 1000.0 AS avg_duration_ms,
    rs.avg_cpu_time / 1000.0 AS avg_cpu_ms,
    rs.avg_logical_io_reads,
    rs.avg_rowcount,
    rs.last_execution_time
FROM sys.query_store_query q
JOIN sys.query_store_plan p 
    ON q.query_id = p.query_id
JOIN sys.query_store_query_text t 
    ON q.query_text_id = t.query_text_id
LEFT JOIN sys.query_store_runtime_stats rs 
    ON p.plan_id = rs.plan_id
WHERE q.object_id = OBJECT_ID(@ProcName)
ORDER BY rs.last_execution_time DESC;

USE [YourActualDatabaseName];
GO

DECLARE @ProcName NVARCHAR(256) = 'usp_YourProcName'; -- Enter your SP name here
DECLARE @ObjectID INT = OBJECT_ID(@ProcName);

SELECT 
    q.query_id,
    p.plan_id,
    OBJECT_NAME(q.object_id) AS stored_proc_name,
    SUBSTRING(t.query_sql_text, (q.statement_sql_handle_start_offset/2)+1,   
        (((CASE q.statement_sql_handle_end_offset   
          WHEN -1 THEN DATALENGTH(t.query_sql_text)  
         ELSE q.statement_sql_handle_end_offset  
         END) - q.statement_sql_handle_start_offset)/2) + 1) AS statement_text,
    CAST(p.query_plan AS XML) AS query_plan_xml,
    rs.count_executions,
    rs.avg_duration / 1000.0 AS avg_duration_ms,
    rs.avg_cpu_time / 1000.0 AS avg_cpu_ms,
    rs.avg_logical_io_reads,
    rs.avg_rowcount,
    rs.last_execution_time
FROM sys.query_store_query q
JOIN sys.query_store_plan p 
    ON q.query_id = p.query_id
JOIN sys.query_store_query_text t 
    ON q.query_text_id = t.query_text_id
JOIN sys.query_store_runtime_stats rs 
    ON p.plan_id = rs.plan_id
WHERE q.object_id = @ObjectID
ORDER BY rs.avg_duration DESC;
