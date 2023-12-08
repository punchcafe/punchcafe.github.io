For a long time, "getting" SQL has eluded me.
I could statements and understood underlying basics around performance, but I couldn't have said it was a _problem solving tool_ to me.
It was more just a way to save and retrieve data in my applications, and anything more complex than that required working out the right combination of arcane incantations to paste together the query I needed. A few recent serendipidous events at work and in my own research has led me to finally have that a-ha moment where SQL has gone from something I kinda know, to something I kinda get, so I wanted to share that in case it makes anyone else's advantures in SQL easier.

Enough introduction. Let's get going with the biggest ah-ha inducer: getting the SELECT FROM WHERE statement. 

Let's introduce our hypothetical business which needs to use SQL for all it's business needs. The Apocafe is a Apocathary selling a variety of potions, of which Coffee is their most popular. Their database consists of just two tables, `potions` and `orders`. Let's peek at the schemas of those tables quickly: 

SCHEMA

A problem the cafe is facing is very limited storage space for stock. Climate control for potions is expensive! The cafe decides it's going to only stock products which have the highest price/volume. Sabrina, their Data Analyst witch, is asked to write an SQL query to return a list of products ordered by their price/volume.


BUT why doesn't it work??

There's two parts to that answer: First is that each clause (SELECT, FROM, WHERE...) in the query is actually executed in a specifc order. Second is that the execution of each of those steps can **transform the data passed to the next step.**

Firstly, ordering. Each SELECT - FROM - WHERE statement is made up of clauses. The [Oracle documentation](https://docs.oracle.com/javadb/10.8.3.0/ref/rrefselectexpression.html#rrefselectexpression) makes it easy to see what our anatomy of the clause is:
```
SELECT [ DISTINCT | ALL ] SelectItem [ , SelectItem ]*
FROM clause
[ WHERE clause ]
[ GROUP BY clause ]
[ HAVING clause ]
[ ORDER BY clause ]
[ result offset clause ]
[ fetch first clause ]
``` 
For the scope of this blog, we're going to ignore the result off set and fetch first fields, and focus entirely on `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING` and `ORDER BY`. They have the following order:
```
1. FROM/JOIN
2.   WHERE
3.   GROUP BY
4.   HAVING
5.  SELECT
6.   ORDER BY
```
Remember what I said a minute ago? Each of these steps will **transform** the data sent to the next step. So let's think of our original queries again, but this time think of them as a pipeline of **ordered** transformations:

```
FROM(products) -> SELECT(price / bottle_volume AS cost_per_ml) -> ORDER BY(cost_per_ml)
FROM(products) -> WHERE(cost_per_ml > 5) -> SELECT(price / bottle_volume AS cost_per_ml)
```
That starts to make a bit more intuitive sense. The WHERE clause is trying to filter on something which isn't introduced until the select clause.

Let's start trying to build a mental model of our "data" and "transformations" actually happening here. We can think of data as the contents of a table to begin with. This isn't always the case, but the model of "data" being a series of rows with named columns is a good start to understand how it's transformed. Breaking down the first pipeline above:
```
FROM products
```
We start off with our vanilla table in SQL, which has all the original columns:

[ ] TABLE

Then it goes into the select clause...
```
SELECT ...
```
This takes **all** the rows in the table, only keeps the `x`, `y`, columns, and also creates a new column for each row, `price_per_ml`:

[ ] TABLE


Finally it goes through the ORDER BY clause:
```
ORDER BY
```
This just orders all by the new `price_per_ml` column. At this point, the select has omitted origal columns, so let's see what happens if we try and order by something not on it:

TEST


Query concept: https://docs.oracle.com/javadb/10.8.3.0/ref/rrefsqlj21571.html#rrefsqlj21571
SELECT FROM WHERE expression: https://docs.oracle.com/javadb/10.8.3.0/ref/rrefselectexpression.html#rrefselectexpression