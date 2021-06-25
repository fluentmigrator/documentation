Database | Identifier | Alternative identifier(s)
---------|------------|---------------------------
Microsoft SQL Server 2019 | SqlServer2016<sup>(1)</sup> | SqlServer
Microsoft SQL Server 2017 | SqlServer2016<sup>(2)</sup> | SqlServer
Microsoft SQL Server 2016 | SqlServer2016 | SqlServer
Microsoft SQL Server 2014 | SqlServer2014 | SqlServer
Microsoft SQL Server 2012 | SqlServer2012 | SqlServer
Microsoft SQL Server 2008 | SqlServer2008 | SqlServer
Microsoft SQL Server 2005 | SqlServer2005 | SqlServer
Microsoft SQL Server 2000 | SqlServer2000 | SqlServer
~~Microsoft SQL Server Compact Edition~~<sup>(3)</sup> | ~~SqlServerCe~~ | ~~SqlServer~~
PostgreSQL | Postgres | PostgreSQL
PostgreSQL 9.2 | Postgres92 | PostgreSQL92
PostgreSQL 10.0 | PostgreSQL10_0 | PostgreSQL
PostgreSQL 11.0 | PostgreSQL11_0 | PostgreSQL
MySQL 4 | MySql4 | MySql
MySQL 5 | MySql5 | MySql, MariaDB
Oracle  | Oracle |
Oracle (managed ADO.NET) | OracleManaged | Oracle
Oracle (DotConnect ADO.NET) | OracleDotConnect | Oracle
Microsoft JET Engine (Access) | Jet |
SQLite | Sqlite |
Firebird | Firebird |
Amazon Redshift | Redshift |
SAP Hana | Hana |
~~SAP SQL Anywhere~~<sup>(4)</sup> | ~~SqlAnywhere16~~ | ~~SqlAnywhere~~
DB2 | DB2 |
DB2 iSeries | DB2 iSeries | DB2

- <sup>(1)</sup> All integration tests ran without error against an SQL Server 2019 using the SqlServer2016 dialect.
- <sup>(2)</sup> All integration tests ran without error against an SQL Server 2017 using the SqlServer2016 dialect.
- <sup>(3)</sup> Support for Microsoft SQL Server Compact Edition is being dropped due to Microsoft end-of-life support date passing.
- <sup>(4)</sup> Support for SAP SQL Anywhere is being dropped due to SAP not supporting a .NET Core / .NET 5 database driver.