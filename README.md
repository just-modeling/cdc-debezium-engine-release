# cdc-debezium-engine-release

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
PROFILE=dev
databricks auth login --host <workspace_url> -p ${PROFILE}
```

Create deployment volume in the root catalog for the environment. This must be done by user who has all privilleges on the catalog.

```sh
databricks volumes create <CATALOG_NAME> default libraries MANAGED
```

Download the jar and run the following command to deploy the jar to Databricks workspace

```sh
TAG=2.1.4-1.2.2
CATALOG=dev
PROFILE=dev
databricks fs cp justmodeling-pitcrew-cdc-${TAG}-jar-with-dependencies.jar dbfs:/Volumes/${CATALOG}/default/libraries/justmodeling-pitcrew-cdc-0.1.0-jar-with-dependencies.jar --overwrite -p ${PROFILE}
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

Optionally, instead of hard code the catalog or credentials, you can leverage Databricks secret vault to 
pass values to the config.ini. If you have "my_username" stored in secret vault under scope "mysecret", configure 
the config.ini like below:

database.user=SECRET|mysecret|my_username

this instructs the engine to lookup secret called "my_username" under scope "mysecret". Similary you can inject environment
variables in a similar way

cdc.persist.catalog=ENV|catalog_name|us_dev

this instructe the gine to lookup environment vars called "catalog_name", if it cannot find one, default it to the value "us_dev"


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
