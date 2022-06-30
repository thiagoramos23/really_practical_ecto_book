# How to Query

Getting data from the database is one of the crucials activities a developer need to learn when he or she
is learning a new framework and since the name of this book has `practical` in it I will stop talking now 
and start with the examples.

So, the first thing you need to learn is how get all the data from a table, in this case from an `Ecto.Schema` 
So, suppose you have this ecto schema:

```elixir
defmodule Product do
  use Ecto.Schema
  
  schema "products" do
    field :name, :string
    field :color, :string
  end
end
```

And now if you want to get all the products, what you need to do? The answer is pretty simple you just need
to use the `Repo` module and use one of the numerous functions to get all the data.

```elixir
Repo.all(Product)
```
```sql
SELECT * FROM products;
```

Throughout this book every time I show some code that gets some data from the database I will also show the 
related SQL query the code is producing.

Even though this is somewhat very simple and you can probably use in your real projects we have another way to 
get data and it is using the `Ecto.Query` module. So to get all products with `Ecto.Query` we would need to do:

```elixir
import Ecto.Query

query = from p in Product

Repo.all(query)
```
```sql
SELECT * FROM products;
```

This will get the same results. So you maybe asking yourself, well, why this people insist in having multiple
ways of doing the same thing? I will tell you next. 

I've got say that I thought the same thing but there is a pretty good explanation for that and in fact the 
first example without `Ecto.Query` is just a kind of sintactic sugar for you so you don't need to write the query.
Ecto is a very powerfull framework and you can, pretty much, do all the queries that you need to do, even 
the hard ones, without ever need to write the SQL queries for it, just using the `Ecto.Query` code.

## Getting data without schema

So, imagine now you don't have schemas. Imagine that you just want a simple app to get some data from the 
database and nothing more, you don't want to write a lot of modules with schemas and fields and all the code
you'd need to write so you can have access to fields and stuff. Right now you just want to get some data. 
So, how do you do?

Turns out `Ecto.Query` is the perfect solution for your problem. Let's get all the data we need from the products
table without even have any schema defined.

```elixir
import Ecto.Query

query = 
  from p in "products",
  select: [p.name, p.color]

Repo.all(query)
```

If you run this in your shell, the query will return:

```bash
[debug] QUERY OK source="products" db=1.8ms idle=1903.5ms
SELECT p0."name", p0."color" FROM "products" AS p0 []
↳ :erl_eval.do_apply/6, at: erl_eval.erl:685
[["Samsung TV", "Black"], ["MacBook Pro 13", "Space gray"]]
````

```sql
SELECT name, color FROM products
```

So, we have a small difference here, because we don't have any schema, `Ecto` can't possibly know what would be 
the data to return, so we need to tell it the name of the columns we should return in the query. Another thing
to be aware of is that we can use other data structures in the `select` clause. If you aren't anything like me
and you paid attention to every piece of code I showed to you so far you had noticed that we are using a `List`
data structure as the return value of the `select` clause. Another example using a `Map` would be:

```elixir
import Ecto.Query

query = 
  from p in "products",
  select: %{name: p.name, color: p.color}

Repo.all(query)
```

And now, if you run this in your shell you would see:

```bash
[debug] QUERY OK source="products" db=1.1ms queue=1.7ms idle=1703.2ms
SELECT p0."name", p0."color" FROM "products" AS p0 []
↳ :erl_eval.do_apply/6, at: erl_eval.erl:685
[
  %{color: "Black", name: "Samsung TV"},
  %{color: "Space gray", name: "MacBook Pro 13"}
]
```

So, one thing to keep in mind is that you can pretty much use the data structure you feel is the best choice for
your case. This last example is using a `Tuple`:

```elixir
import Ecto.Query

query = 
  from p in "products",
  select: {p.name, p.color}

Repo.all(query)
```

And now, if you run this in your shell you would see:

```bash
[debug] QUERY OK source="products" db=0.9ms queue=1.5ms idle=1613.5ms
SELECT p0."name", p0."color" FROM "products" AS p0 []
↳ :erl_eval.do_apply/6, at: erl_eval.erl:685
[{"Samsung TV", "Black"}, {"MacBook Pro 13", "Space gray"}]
```

One of the many reasons I love ecto is because it is very explicit.

And now, what would be the same example but now using the `Product `schema? Here it goes:

```elixir
import Ecto.Query

query = 
  from p in Product,
  select: {p.name, p.color}

Repo.all(query)
```

As you can see the only thing that changed is that now we are using the `Ecto.Schema` instead of using a simple
string, the other thing will continue to be the same.

So far I showed to you how to get all the data from a table in the database but usually we don't want to do that.
We usually only want a set of all the data that is there. In the next sections we will learn how to:
- Use the `where` clause
- How to paginate data so we can get a small set of data each time
- How to use `join` to get data from `Ecto.Schema` or direct into tables that are associated (have association 
  between foreign keys).

Again, I didn't write this book to show my english language skills, I wrote this because I want a book that is
succint and to the point. So let's go.

## Where clause

# TODO: Examples of where clause
