# cdc-debezium-engine-release

This is a repo to publish releases for project cdc-debezium-engine. Original repository is https://github.com/just-modeling/cdc-debezium-engine

## What is this?

A lightweight Java application that pushes CDC events as real-time data streaming from various of databases to the target platform. 

The implementation architecture is based on [Debezium Engine](https://debezium.io/documentation/reference/2.1/development/engine.html)

Current support target platforms are: 

* Databricks
* Apache Spark

Note **The pipelines that handle merging CDC events to target tables are developed separately.**

## Installation 

### Databricks

Authenticate to workspace where the library need to be deployed.

```sh
databricks auth login --host <workspace_url>
```

Create deployment volume in the root catalog for the environment. This must be done by user who has all privilleges on the catalog.

```sh
databricks volumes create <CATALOG_NAME> default libraries MANAGED
```

Run the following command to build the jar and deploy to Databricks workspace

```sh
sh ./make-databricks -c <CATALOG_NAME> -p <CLI-PROFILE>
```

### Apache Spark

Run the following command in your DBFS to build the jar and deploy to designated path

```sh
sh ./make-spark -c <CATALOG_NAME> -p <CLI-PROFILE>
```

## Usage

Create example config.ini file as below.

```txt
[DEBEZIUM-CONFIG]
schema.history.internal=com.justmodeling.pitcrewcdc.databricks.DbfsSchemaHistory
offset.storage=com.justmodeling.pitcrewcdc.databricks.DbfsOffsetBackingStore
connector.class=io.debezium.connector.sqlserver.SqlServerConnector
schema.history.internal.file.filename=/dbfs/cdc-checkpoint/XXX/XXX/schema.dat
offset.storage.file.filename=/dbfs/cdc-checkpoint/XXX/XXX/offsets.dat
database.hostname=SECRET|SCOPE|HOSTNAME
database.port=1433
database.user=SECRET|SCOPE|USERNAME
database.password=SECRET|SCOPE|PASSWORD
topic.prefix=<TOPIC-NAME>
snapshot.mode=initial
snapshot.isolation.mode=snapshot
tombstones.on.delete=false
name=mssql

[DATABRICKS-CONFIG]
cdc.persist.catalog=ENV|CATALOG|CATALOG-NAME
cdc.persist.database=<DB-NAME>
cdc.persist.table=<TABLE-NAME>
```

Simply run the JAR file with configuration file, called config.ini

```bash
java com.justmodeling.pitcrewcdc.databricks.MSSQLChangeCapture config.ini
```

## Connectors

### MSSQL

[Debezium SQLServer Connector](https://debezium.io/documentation/reference/2.1/connectors/sqlserver.html)

**SQL SERVER Always On**

The SQL Server connector can capture changes from an Always On read-only replica.

Prerequisites
* Change data capture is configured and enabled on the primary node. SQL Server does not support CDC directly on replicas.

* The configuration option database.applicationIntent is set to ReadOnly. This is required by SQL Server. When Debezium detects this configuration option, it responds by taking the following actions:

    * Enable snapshot isolation
    ```sql
    ALTER DATABASE <database_name>  
    SET ALLOW_SNAPSHOT_ISOLATION ON 
    ```

    * Sets snapshot.isolation.mode to snapshot, which is the only one transaction isolation mode supported for read-only replicas. 

    * Commits the (read-only) transaction in every execution of the streaming query loop, which is necessary to get the latest view of CDC data.

### MongoDB

[Debezium MongoDB Connector](https://debezium.io/documentation/reference/2.1/connectors/mongodb.html)

### MySQL

[Debezium MySQL Connector](https://debezium.io/documentation/reference/2.1/connectors/mysql.html)

### PostgreSQL

[Debezium PostgreSQL Connector](https://debezium.io/documentation/reference/2.1/connectors/postgresql.html)

### Oracle

[Debezium Oracle Connector](https://debezium.io/documentation/reference/2.1/connectors/oracle.html)