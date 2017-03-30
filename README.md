Hive JDBC Storage Handler
=========================

Provides support for creating read-only Hive external tables that can
read the results of a query run on an RDBMS such as MySQL. 

Usage
-----

*   Clone the git project and run `mvn clean package` to produce the storage
    handler JAR.

*   Start hive and add the JAR file to the classpath:

        ADD JAR /path/to/hive-jdbc-storage-handler-1.1.1-cdh4.3.0-SNAPSHOT-dist.jar;

*   Create an external table in Hive:



        CREATE EXTERNAL TABLE mysql_test
        (
            id STRING,
            part_name STRING,
            warehouse_id STRING,
            created STRING
        )
        STORED BY 'com.qubitproducts.hive.storage.jdbc.JdbcStorageHandler' 
        TBLPROPERTIES (
            "qubit.sql.database.type" = "MYSQL",
            "qubit.sql.jdbc.url" = "jdbc:mysql://localhost:3306/qubit?user=qubit&password=qubit",
            "qubit.sql.jdbc.driver" = "com.mysql.jdbc.Driver",
            "qubit.sql.query" = "SELECT part_id, part_name, warehouse_id, created_datetime FROM parts",
            "qubit.sql.column.mapping" = "id=part_id, created=created_datetime:date"
        );

Configuration
-------------

### Table Creation

##### Required Table Properties

*   `qubit.sql.database.type` - Currently only supports `MYSQL`, `H2` and
    `DERBY`.
*   `qubit.sql.jdbc.url` - Full JDBC connection URL.
*   `qubit.sql.jdbc.driver` - JDBC driver class name.
*   `qubit.sql.query` - Query to run against the database. The Storage
    handler is intelligent enough to determine the correct conditions to
    apply to the database query based on the hive query being executed.
    As such, please ensure that there are no `WHERE` clauses defined
    here. If column names differ between the hive table and the query,
    use the `qubit.sql.column.mapping` property to define the mappings.

##### Optional Table Properties

*   `qubit.sql.column.mapping` - If the Hive column names are different
    from the corresponding table column, use this property to indicate
    it. Format is: `hiveColumnName=dbColumnName[:type]` separated by
    commas. Currently the only accepted `type` is `date`. Sample usage:
    `"qubit.sql.column.mapping" = "id=part_id, created=created_datetime:date"`.
    Only the columns that differ need to be specified.
*   `qubit.sql.jdbc.fetch.size` - Resultset fetch size. Default value is
    1000.
*   Connections to the database are made through a DBCP connection pool.
    Specify any [DBCP configuration
    options](http://commons.apache.org/dbcp/configuration.html) by
    prefixing them with `qubit.sql.dbcp.`. Eg:
    `qubit.sql.dbcp.maxActive=5`

### Querying

The Storage handler is intelligent enough to determine the correct
conditions to apply to the database query based on the hive query being
executed. Therefore no additional steps need to be performed while
querying.

Resultset fetch size can be changed at query time:

    SET qubit.sql.jdbc.fetch.size=500;

