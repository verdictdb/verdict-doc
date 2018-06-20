
Configurations
=================


Changing Verdict configuration
--------------------------------------

1. **Creating a configuration file** (i.e., :code:`verdict.properties`) in the :code:`config` directory. In order for the options in this file to be effective, Verdict should be build again using a :code:`mvn package` command.

2. **Setting environment variables**: When Verdict's Java instance is created, it scans all environment variables and stores them in Verdict's configuration if they start with :code:`verdict`.

3. **Passing parameters when making a connection**

   * **Spark**: You can pass parameters using the :code:`--conf` argument. For example::
   
      $ spark-shell --jars target/{{ site.verdict_core_jar }} --conf verdict.loglevel=warn
   
   will set the Verdict's log level to :code:`warn`.

   * **Hive, Impala**: You can pass the parameters in the JDBC connection string. For example::
   
      $ /bin/verdict-shell -h "impala://host:port/default;verdict.loglevel=warn"
   
   will set the Verdict's log level to :code:`warn`.

4. **Using a Verdict query**: You can issue a :code:`set key=value` query after a Verdict instance is created.

The higher-numbered items take higher precedences.


Verdict configuration options
--------------------------------------

Basic options
^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 15, 8, 20
   :stub-columns: 0
   :align: left
   :class: config-table

   *  -  key
      -  Default values
      -  Description
   *  -  verdict.bypass
      - false
      - If set true, Verdict simply passes all the query to the underlying database and returns the result to the user without any modifications. The only exception is `set bypass=false`.
   *  -  verdict.bypass
      -  false
      -  If set true, Verdict simply passes all the query to the underlying database and returns the result to the user without any modifications. The only exception is `set bypass=false`.
   *  -  verdict.loglevel
      -  info
      -  Determines the log level; should be one of "debug", "info", "warn", "error". `loglevel` is a synonym for `verdict.loglevel`.
   *  -  verdict.relative_target_cost
      -  10%
      -  Keeps the cost of Verdict's query processing under a certain level. Verdict chooses a set of the largest samples under this limit.

JDBC options
^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 17, 8, 20
   :stub-columns: 0
   :align: left
   :class: config-table

   *  -  Key
      -  Default values
      -  Description
   *  -  verdict.jdbc.user
      -
      -  If set, pass as a "user" value when making a JDBC connection to an underlying database.
   *  -  verdict.jdbc.password
      -
      -  If set, pass as a "password" value when making a JDBC connection to an underlying database.
   *  -  verdict.jdbc.kerberos_principal
      -  n/a
      -  If set, makes an appropriate Kerberos authentication when making a JDBC connection to an underlying database. :code:`principal` is a synonym for :code:`verdict.jdbc.kerberos_principal`.
   *  -  verdict.jdbc.ignore_user_credentials
      -  false
      -  If set to true, "user" and "password" values are not passed even if they are set in :code:`verdict.jdbc.user` and :code:`verdict.jdbc.password`.
   *  - verdict.jdbc.dbname
      -
      - One of "impala", "hive2". The database name specified in the JDBC connection string is set.
   *  -  verdict.jdbc.host
      -  127.0.0.1
      -  The host that will be used for a JDBC connection.
   *  -  verdict.jdbc.port
      -
      -  The port number that will be used for a JDBC connection.
   *  -  verdict.jdbc.schema
      -  default
      -  The default schema (or database) that a JDBC connection will connect to.
   *  -  verdict.jdbc.hive2.default_port
      -  10000
      -  The default port number that will be used for a hive2 connection if the port number is not explicitly set in the passed JDBC connection string.
   *  -  verdict.jdbc.hive2.class_name
      -  com.cloudera.hive.jdbc4.HS2Driver
      -  The class name of the JDBC driver that will be used for a hive2 connection.
   *  -  verdict.jdbc.impala.default_port
      -  21050
      -  The default port number that will be used for an impala connection if the port number is not explicitly set in the passed JDBC connection string.
   *  -  verdict.jdbc.impala.class_name
      -  com.cloudera.impala.jdbc4.Driver
      -  The class name of the JDBC driver that will be used for an impala connection.


Spark options
^^^^^^^^^^^^^^^^^^^^^^^^^
.. list-table::
   :header-rows: 1
   :widths: 15, 8, 20
   :stub-columns: 0
   :align: left
   :class: config-table

   *  -  Key
      -  Default values
      -  Description
   *  -  verdict.spark.cache_samples
      -  true
      -  (Lazily) cache the sample tables created by Verdict using Spark's `DataFrame#cache()` function. The sample tables are typically significantly smaller than the original tables. Caching sample tables will lower query response times.

