# SQL-Server-Maintenance              <img align="right" src="https://visitor-badge.laobi.icu/badge?page_id=noetovar5.SQL-Server-Maintenance"/>
SQL Server Maintenance Queries



find out SQL server version
SELECT SERVERPROPERTY('productversion') as Version, SERVERPROPERTY ('productlevel') as Level, SERVERPROPERTY ('edition') as Edition, @@servername 'Server Name', substring(@@version,1,charindex('-',@@version)-1) +convert(varchar(100),SERVERPROPERTY('edition'))+ ' '+ +convert(varchar(100),SERVERPROPERTY('productlevel'))'Server Version'



Show all DB files sizes
SELECT DB_NAME(database_id) AS Database_Name, Name AS File_Name, Physical_Name AS Physical_Path, (size*8)/1024 Size_MB FROM sys.master_files order by Size_MB desc



Reindex DB (MANDATORY if you shrink log files or DB)
EXEC sp_MSforeachtable 'dbcc dbreindex("?")'



Show DBs with exceptional compatibility level
select name as 'DB name', compatibility_level as 'DB compatibility' , 'Server version' = (select cmptlevel from master.dbo.sysdatabases where name = db_name() ) from sys.databases where compatibility_level not in (select cmptlevel from master.dbo.sysdatabases where name = db_name() )


Rebuild all indexes on selected databases
/*rebuild all indexes*/
/*Adjusted by Dotan on 2021-11-23 from the XMPDB2 stored procedure: SP_CleanupAndMaintain*/
RAISERROR('Status: rebuilding indexes

',0,1) WITH NOWAIT
DECLARE @affectedRows bigint
DECLARE @affectedRowsTemp bigint
DECLARE @Database NVARCHAR(255)
DECLARE @Table NVARCHAR(255)
DECLARE @cmd NVARCHAR(1000)
SET @affectedRows = 0
DECLARE DatabaseCursor CURSOR READ_ONLY FOR
SELECT name FROM master.sys.databases


