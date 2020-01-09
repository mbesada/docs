---
title: Connect to the Database
summary: How to connect to CockroachDB from your application
toc: true
---

This page has instructions for connecting to your CockroachDB cluster.  We assume that you already have a cluster up and running using the instructions for a [manual deployment][manual] or an [orchestrated deployment][orchestrated].

The examples will show a connection string for a [secure local cluster][local_secure], which you will need to edit depending on your deployment type.

For a reference explaining all the possible cluster connection parameters, see [Connection Parameters][connection_params].

<div class="filters filters-big clearfix">
  <button class="filter-button" data-scope="sql">SQL</button>
  <button class="filter-button" data-scope="java">Java</button>
  <button class="filter-button" data-scope="python">Python</button>
  <button class="filter-button" data-scope="go">Go</button>
</div>

<section class="filter-content" markdown="1" data-scope="sql">

## SQL

{% include copy-clipboard.html %}
~~~ shell
$ cockroach sql --certs-dir=certs --host=localhost:26257
~~~

</section>

<section class="filter-content" markdown="1" data-scope="java">

## Java

{% include copy-clipboard.html %}
~~~ java
PGSimpleDataSource ds = new PGSimpleDataSource();
ds.setServerName("localhost");
ds.setPortNumber(26257);
ds.setDatabaseName("bank");
ds.setUser("maxroach");
ds.setPassword(null);
ds.setSsl(true);
ds.setSslMode("require");
ds.setSslCert("certs/client.maxroach.crt");
ds.setSslKey("certs/client.maxroach.key.pk8");
ds.setReWriteBatchedInserts(true); // add `rewriteBatchedInserts=true` to pg connection string
ds.setApplicationName("BasicExample");
~~~

</section>

<section class="filter-content" markdown="1" data-scope="python">

## Python

{% include copy-clipboard.html %}
~~~ python
conn = psycopg2.connect(
    database='bank',
    user='maxroach',
    sslmode='require',
    sslrootcert='certs/ca.crt',
    sslkey='certs/client.maxroach.key',
    sslcert='certs/client.maxroach.crt',
    port=26257,
    host='localhost'
)
~~~

</section>

<section class="filter-content" markdown="1" data-scope="go">

## Go

{% include copy-clipboard.html %}
~~~ go
db, err := sql.Open("postgres",
        "postgresql://maxroach@localhost:26257/bank?ssl=true&sslmode=require&sslrootcert=certs/ca.crt&sslkey=certs/client.maxroach.key&sslcert=certs/client.maxroach.crt")
    if err != nil {
        log.Fatal("error connecting to the database: ", err)
    }
    defer db.Close()
~~~

</section>

## See also

- [Connection parameters][connection_params]
- [Manual deployments][manual]
- [Orchestrated deployments][orchestrated]
- [Start a local cluster (secure)][local_secure]

<!-- Reference Links -->

[manual]: manual-deployment.html
[orchestrated]: orchestration.html
[local_secure]: secure-a-cluster.html
[connection_params]: connection-parameters.html