---
title: Make Queries Fast
summary: How to make your queries run faster during application development
toc: true
---

This page describes how to optimize OLTP application queries for CockroachDB.  To get good performance from CockroachDB, you need to answer questions on several layers, like an onion.

1. **Will the SQL queries you are executing perform well?** Specifically, are they returning too many rows?  Are they using the wrong type of [`JOIN`](joins.html)?  Are they using the right [index](indexes.html)?  Any index at all?  See [SQL query performance](#sql-query-performance) below.
2. **Will your schema design perform well under your workload?** Even if you write good SQL, you may be querying a schema that will lead to contention under your workload.  For example, you may be trying to run a write-heavy workload against a schema that uses sequential primary keys, which leads to write hotspots.  See [Schema design](#schema-design) below.
3. **Does the cluster topology you are using match your use case?**  Are you picking the right point in the latency vs. resiliency tradeoff? See [Cluster topology](#cluster-topology) below.

## SQL query performance

To get good SQL query performance, follow these rules (in order of importance):

1. Make sure your query [scans at most a few dozen rows](#scan-at-most-a-few-dozen-rows) (several hundred rows at the absolute maximum).
2. [Use the right join type](#use-the-right-join-type) for the tables you are querying.
3. [Use the right index](#use-the-right-index).

### Scan at most a few dozen rows

To determine how many rows a query is scanning, you have the following options:

1. Look at the output of [`EXPLAIN`](explain.html).  This can work if you know how to read `EXPLAIN` output and already know about how much data is in each table.
2. Break apart the query and and run each piece to understand how many rows are being scanned.

Option # 2 requires you to have a test environment available, but is generally easier.

For example, see the differences in output from these two [pagination](query-data.html#using-pagination) queries:

- One is using keyset pagination, scans a few dozen rows, and is fast:

XXX: YOU ARE HERE

- One is using `LIMIT`/`OFFSET` to do pagination, scans the whole table, and is 10x slower:

~~~ sql
SELECT * FROM employees AS OF SYSTEM TIME '-1m' LIMIT 25 OFFSET 200024;
~~~

~~~
... output snipped ...
Time: 118.114ms
~~~

For more information about pagination, and this specific data set, see [Using Pagination](query-data.html#using-pagination).

### Use the right join type

General rules for join types:

1. If one side A is much smaller than the other side B, use a lookup join with A on the left side.  This will ensure that the query touches only the smaller number of rows in A.

`SELECT * FROM A LOOKUP JOIN B ON A.id = B.id ...`

2. Hash join and merge join both iterate over each row on the left hand side, and then over each row on the right hand side.  They should be avoided in cases when A is much smaller than B.

For more details, see the [join reference documentation](joins.html).

### Use the right index

## Schema design

The most important factor to consider when designing your schema is your workload's data access patterns.  In particular:

- Keep records that are likely to be accessed together near each other
- Keep records that are not likely to be accessed together far from each other
- Make sure you have indexes you need for your application

For more information about how to put these rules into practice, see:

- [Choosing Index Keys for CockroachDB](https://www.cockroachlabs.com/blog/how-to-choose-db-index-keys/)
- [Unique ID Best Practices](performance-best-practices-overview.html#unique-id-best-practices)

## Cluster topology

The most important factor to consider in your choice of cluster topology is the latency vs. resiliency tradeoff.  In general, more resiliency will cost you in higher latency, since it requires more nodes in your cluster, spread over more availability zones.

Depending on your use case, there are a number of design options in this space to choose from.  To find the cluster topology that meets the needs of your use case, see [this list of topology patterns](topology-patterns.html).

## Foo

3 pronged approach, listed in order of most to least control by application developer (probably):
Write good SQL
Follow Rules of thumb from Optimizing OLTP Application Queries:
Queries should scan no more than dozens of rows (couple of hundred at most).
Create the right index, use the right join so that DB has to touch a handful of rows to get the results.
Pointers to SQL tuning with EXPLAIN, etc.
Use good schema design
This can list the highlights and point the user elsewhere for details
Especially important to avoid transaction contention if at all possible.
Developer may not have control of schema depending on organization / application maturity / etc.
Use the right cluster topology
Give the highlights, then punt to Topology Patterns.
Ideally, Ops team should have already handled this so you don’t have to care, but be aware as it affects query performance.  Link to ‘Troubleshooting cluster problems’ doc in this guide.
TODO: look at SQL best practices page to see if something is missing
TODO: talk about secondary indexes - does your query have covering indexes

## See also

Reference information:

- [CockroachDB Performance](cockroachdb-performance.html)
- [SQL Performance Best Practices](sql-performance-best-practices.html)
- [Topology Patterns](topology-patterns.html)
- [SQL Tuning with `EXPLAIN`](sql-tuning-with-explain.html)
- [Joins](joins.html)

Specific tasks:

- [Connect to the Database](connect-to-the-database.html)
- [Insert Data](insert-data.html)
- [Query Data](query-data.html)
- [Update Data](update-data.html)
- [Delete Data](delete-data.html)
- [Run Multi-Statement Transactions](run-multi-statement-transactions.html)
- [Hello World Example apps](hello-world-example-apps.html)

