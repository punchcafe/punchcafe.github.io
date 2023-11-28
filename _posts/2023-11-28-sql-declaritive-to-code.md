# SQL: Declaritve to Code

I've been putting off getting better at SQL for about two years now. Each new year, it manages to make it's way back on to my to-do list, yet never finds the attention it deserves.
I'm a back end engineer so I really should be better with it and yet it's bizarre combinations of clauses has always left me at a loss. It's never _felt_ like coding, until I recently finally started putting that new years resolution. Skip ahead a number of Codewars kata,  _really_ y2k Oracle documentation pages and a handful of blog posts, and I feel I'm really starting to _get it_. 

I'm hoping this blog post can do the same for anyone in a similar position, and give you that ah-ha click to allow you to start thinking of SQL interms of _lines of instructions_. you have a solid coding foundation and some SQL fundamentals. `SELECT`s, `GROUP BY`s and `JOIN`s should all be things you are aware of and have used a few times.

# The Schema
Let's start out blog by introducing a fictional schema. My professional experience is Elixir and Java, so let's say we're building a database for a Coffee Shop which sells potions (of which, coffee is one). Let's introduce our first table: `potions`.

```sql
CREATE TABLE potions(
    elixir_name varchar(255),
    id int,
    price float,
    bottle_volume_ml float,
    category varchar(255)
)
```

Let's start with a simple SQL query. Our alchemist staff have limited space in their store room. Magical climate control is very expensive, so they want to know what their most expensive price-per-volume goods are: 

```sql
SELECT 
  elixir_name, 
  price,
  price / bottle_volume AS cost_per_ml
FROM
  potions
ORDER BY
  cost_per_ml
```

This works fine, but the staff want to tweak the query. They know it's a bad idea to just stock one variety of potion, so they want find _all_ potions which are above a certain cost-per-volume threshold:

```sql
SELECT 
  elixir_name, 
  price,
  price / bottle_volume AS cost_per_ml
FROM
  potions
WHERE
  cost_per_ml > 50.0
```

Huh? That doesn't work:
```
ERROR OUTPUT
```

Why was `cost_per_ml` fine when it was in the `ORDER BY` clause but not the `WHERE` clause?
The answer lies in the first major sleight-of-hand in SQL; SELECT FROM clauses have an implicit ordering of execution to their component clauses, and each execution step affects what's available to the step after it.

```
    1. FROM/JOIN
    2. WHERE
    3. GROUP BY
    4. HAVING
    5. SELECT
    6. ORDER BY
    7. LIMIT/OFFSET
```
In the above query, we introduce the `cost_per_ml` field in the `SELECT` step. The SELECT step is before the ORDER BY step, but after the WHERE step, so it makes sense that our first query succeeded, but our second query failed: the column just wasn't available in the where clause.

Let's imagine this as code.
First, the dataset:
```elixir
@type dataset :: {rows :: [tuple], available_columns :: [String.t()]}
``` 
Loosely imagine our table data as a big bunch of rows, with a set of columns. Each step in the query can make transformations, but the result will be another bunch of rows and columns.

Let's model those functions. Each function accepts a type, `dataset` (our floaty model), some other args, and returns an object of type `dataset`. The only exception to this is `FROM`, as this is our data source. So our very first query can be written like this:

```elixir
defmodule SelectQuery do

    def query() do
      "potions"
      |> from() # {[..rows..], [:elixir_name, :price, :id, :category, :bottle_volume_ml]}
      |> select(["elixir_name", "price", "price / bottle AS cost_per_ml"])
      |> order_by("cost_per_ml")
    end
    ...
  
end
```

BUT SELECT is last.
Let's illustrate this by changing the where clause.
What about order by??