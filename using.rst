
Using Verdict
=================


In terminal
-------------------

Please see our :doc:`start`. for the instructions on connecting to Verdict in terminal. The page includes starting Verdict on top of Apache Hive, Apache Impala, and Apache Spark (and PySpark) in terminal.

.. _jdbc-connections:

JDBC in Java/Python applications
--------------------------------------

Verdict ships with a JDBC driver. Using the driver, you can use Verdict on top of your existing JDBC-supported database systems. Currently, Verdict supports the JDBC connections to Apache Hive and Apache Impala. Contact us if you want to use Verdict on other database systems. We need to add a small driver (due to non-standard SQL features used by different databases).

1. Class name for Verdict's JDBC driver: :code:`edu.umich.verdict.jdbc.Driver`

1. Connection string for Verdict's JDBC driver:

   * Hive:   :code:`jdbc:verdict:hive2://host:port/default_database;key1=value1;key2=value2;...`

   * Impala: :code:`jdbc:verdict:impala://host:port/default_database;key1=value1;key2=value2;...`
   
   * Redshift: :code:`jdbc:verdict:redshift://endpoint:port/database;key1=value1;key2=value2;...`

To enable the Kerberos authentication, add :code:`principal=user/host@domain` pair in the key-value pairs of the JDBC connection string. You can also pass Verdict configuration options in key-value pairs. For example, to change Verdict's log level to :code:`DEBUG`, add :code:`verdict.loglevel=debug` in the key-value pairs. See :doc:`configuration` for more configuration options for Verdict.


**Details**

The rule for Verdict's connection string is to include the :code:`verdict:` after :code:`jdbc:`. Internally, Verdict communicates with the target database (e.g., Hive, Impala, etc.) using a JDBC connection as well. For this, Verdict uses the passed JDBC connection string after removing (1) the :code:`verdict:` keyword and (2) configuration options for Verdict. This means you can pass any existing JDBC options to Verdict as they are.


Java example
^^^^^^^^^^^^^^^

Below example connects to Verdict-on-Impala and runs a count query::

  import java.sql.Connection;
  import java.sql.DriverManager;
  import java.sql.ResultSet;
  import java.sql.SQLException;
  import java.sql.Statement;

  public class VerdictJdbcExample {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
      Class.forName("edu.umich.verdict.jdbc.Driver");
      String url = "jdbc:verdict:impala://host:port/default";
      Connection conn = DriverManager.getConnection(url);
      Statement stmt = conn.createStatement();
      String sql2 = "select count(*) from orders";
      ResultSet rs = stmt.executeQuery(sql2);
    }
  }

Of course, the Verdict's JDBC driver (:code:`target/verdict-core-version-with-dependencies.jar`) must be included in the Java class path when the above code is compiled and run.

.. _python-example:

Python example
^^^^^^^^^^^^^^^^^^^^^

Below example connects to Verdict-on-Impala and runs a simple count query::

  import jaydebeapi
  import jpype
  import pandas as pd
  from jpype import *

  class_name = 'edu.umich.verdict.jdbc.Driver'
  url = 'jdbc:verdict:impala://host:port/default'
  jvm_path = jpype.getDefaultJVMPath()
  jpype.startJVM(jvm_path, "-Djava.class.path=%s" % classpath)
  conn = jaydebeapi.connect(jclassname = class_name, url = url)
  curs = conn.cursor()
  curs.execute('select count(*) from orders')
  columns = [desc[0] for desc in curs.description]       # getting column headers
  pd.DataFrame(curs.fetchall(), columns = columns)

Note that the Python libraries needed for creating a JVM instance (:code:`jpype`) and making a JDBC connection (:code:`jaydebeapi`) must be installed.


In Apache Zeppelin
--------------------------------------

Apache Zeppelin is a notebook-based application that runs in a web browser. It can connect to Apache Spark by default and also to other database systems through the JDBC interface. Verdict can also easily integrate with Apache Zeppelin.

On Spark
^^^^^^^^^^^^^^^^^^^^^


Zeppelin includes the Spark interpreter by default. For Verdict to work in Spark interpreter, it is enough to include Verdict's core jar file as a dependency. First, go to Zeppelin's interpreter setting, and find the interpreter for Spark. Click the edit button and add the path to Verdict's core jar file (which is created in the :code:`target` directory when Verdict's source code is built) as a dependency.

Now you can import Verdict and run queries as follows::

  import edu.umich.verdict.VerdictSparkHiveContext

  val vc = new VerdictSparkHiveContext(sc)               // sc: SparkContext
  vc.sql("show databases").show(false)
  df = vc.sql("select count(*) from instacart.orders")   // returns a Spark DataFrame
  df.show(false)


On Hive, Impala
^^^^^^^^^^^^^^^^^^^^^

Zeppelin can connect to Verdict on any database (including Hive, Impala) using the JDBC interface. For this, go to Zeppelin's interpreter setting, and click the **create** button. Enter "verdict-impala" (or any name you want) in "Interpreter Name", choose "jdbc" in "Interpreter Group", enter :code:`edu.umich.verdict.jdbc.Driver` for :code:`default.driver`, and enter :code:`jdbc:verdict:impala://host:port/schema` for :code:`default.url` (change the database name appropriately according to :ref:`jdbc-connections`).

You may need to set :code:`default.user` or other authentication fields as needed for your existing database connection (Verdict will pass those parameters as it makes another connection internally to your existing database).

Under **dependencies**, add the (preferably absolute) paths to all JDBC drivers for your existing databases and :code:`verdict-core-version-with-dependencies.jar` (which is created in the :code:`target` directory when Verdict is built).


In Jupyter
--------------------------------------

On PySpark
^^^^^^^^^^^^^^^^^^^^^

You can use Verdict in Jupyter (that connects to PySpark) by following the similar approach as in :ref:`verdict-on-pyspark`. In other words, simply include the path to the Verdict's core jar file as a Driver's Java class path when starting a Jupyter notebook server::

  $ export PYSPARK_DRIVER_PYTHO = "path-to-jupyter"
  $ export PYSPARK_DRIVER_PYTHON_OPTS = "notebook"
  $ export PYTHONPATH = $(pwd)/python:$PYTHONPATH
  $ pyspark --driver-class-path $(pwd)/target/{{ site.verdict_core_jar }}

The above command will start the Jupyter server in which you can import PySpark modules.


On Hive, Impala, Amazon Redshift
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can connect to Verdict on any database that support JDBC connections (including Hive, Impala) as in :ref:`python-example`.


In Hue
--------------------------------------

Hue supports custom JDBC connections. Please see `this page <http://gethue.com/custom-sql-query-editors/>`_ for instructions.

In Google Cloud Dataproc
--------------------------------------
You can use Verdict in Spark running on Google Cloud Dataproc by following a similar approach as just in Spark. First use gcloud command to SSH into Cloud Dataproc cluster master node, in which you should have Verdict's core jar file saved.  Now you can launch :code:`spark-shell` with core jar file specified :code:`spark-shell --jars target/verdict-core-0.3.0-snapshot-jar-with-dependencies.jar`. Then you can import Verdict and run queries as usual::

  import edu.umich.verdict.VerdictSparkHiveContext

  val vc = new VerdictSparkHiveContext(sc)
  vc.sql("show databases").show(false)
  df = vc.sql("select count(*) from instacart.orders")
  df.show(false)
One thing to notice is that in order for Spark 1.6 to work on Cloud Dataproc, you need to select image version to be 1.0 when creating the cluster (default is 1.2, which supports Spark 2.2).
