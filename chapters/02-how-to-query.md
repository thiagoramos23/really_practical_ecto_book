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

We often want to get data from one of more tables that contain a lot of rows, I mean, A LOT. It's not viable nor 
feasable to get all the data from a table everytime. Continuing with our `products` table, let's add some new fields
to it, don't worry about how to add fields to a table or even how to create tables and schemas for now, we will learn
all about this in the other chapters. For now, let's suppose our `Product` schema have these fields:

```elixir
defmodule Product do
  use Ecto.Schema
  
  schema "products" do
    field :name, :string
    field :color, :string
    field :type, :string
    field :brand, :string
    field :manufactured_at, :datetime
  end
end
```

Let's create some data, you can enter the elixir console with `iex -S mix` inside the folder of the project and then you
can copy and paste the code below to create some shoes in your database:

```elixir
colors = ["black", "orange", "green", "white"]
product_list = [{"Nike 1", "Nike", "shoes"}, {"Nike Air", "Nike", "shoes"}, {"Nike Jordan", "Nike", "shoes"}, 
              {"Adidas Run", "Adidas", "shoes"}, {"Olympikus Running", "Olympikus", "shoes"}, 
              {"Under Armour Charged Fleet", "Under Armour", "shoes"}, {"Adidas Life Racer", "Adidas", "shoes"}, 
              {"Mizuno Wave", "Mizuno", "shoes"}, {"Mizuno Prime 9", "Mizuno", "shoes"},
              {"Michael Jordan Shorts", "Nike", "sportswear"}, {"Mike Tyson Gloves", "Adidas", "sportswear"}]

Enum.each(product_list, fn {name, brand, type} ->
  product = %Product{name: name, color: Enum.random(colors), type: type, brand: brand}
  Repo.insert(product)
end)
```

Now, let's say your new manager is asking you to send him a report showing all the shoes the company sells from Nike.
What do you do?

The answer to that is the `where clause`. Let's see how this how to achieve this with ecto. There are two ways to do this,
the first one is using functions:

```elixir
Product
|> Ecto.Query.where(brand: "Nike")
|> Ecto.Query.where(type: "shoes")
|> Repo.all
```

If you don't want to repeat the `Ecto.Query` everytime you can use the `import`.

```elixir
import Ecto.Query
Product
|> where(brand: "Nike")
|> where(type: "shoes")
|> Repo.all
```

This is exactly the same and now you don't need to write `Ecto.Query` for every `where` clause you need to write.
The other way of getting the same data is using the `Ecto.Query` to build the query you want:

```elixir
import Ecto.Query
query = from p in Product, 
        where: p.brand == "Nike",
        where: p.type == "shoes"
        
Repo.all(query)
```

You can also do the same using the more functional form:

```elixir
import Ecto.Query

Product
|> where([p], p.brand == "Nike" and p.type == "shoes")
|> Repo.all
```
```sql
SELECT * FROM products WHERE products.brand = 'Nike' AND products.type = 'shoes'
```

Here I opted to use just one `where` and use the `and` to build the query that we need. One thing to keep in mind when using 
more than one `where` clause is that they all translate to `and` in the end. Next we will see how to use the the `where` clause 
with the `or`.

Now your manager wants to know all the nike shoes we have but also want to know all the other types of products we have
that are not shoes.

```elixir
import Ecto.Query

Product
|> where([p], (p.brand == "Nike" and p.type == "shoes") or (p.type != "shoes"))
|> Repo.all
```
```sql
SELECT * 
FROM products 
WHERE (products.brand = 'Nike' AND products.type = 'shoes') 
OR products.type <> 'shoes'
```

We can also use the `or_where` and in this case it would work the same way. Just keep in mind that the `or_where` is used 
when we want to concatenate more than on `where` function using the `or` instead of the `and` which is the default. Here is
the [documentation](https://hexdocs.pm/ecto/Ecto.Query.html#or_where/3) about `or_where` that explains the usage. Bottom line the query above could be rewrited like this:

```elixir
import Ecto.Query

Product
|> where([p], p.brand == "Nike" and p.type == "shoes")
|> or_where([p], p.type != "shoes")
|> Repo.all
```

The result is the same and the SQL query is also the same. One thing to keep in mind is that the `where clause` can get 
really complex, specially when we are composing queries but I hope I can show you some techniques to make your life easier.

Ok, we did those `where` clauses but we are always using strings as the values. What if I want to a query to return products
of a particular brand and and particular type but I don't want to hardcode the name of the brand neither the type in the 
query.

### Meet the Pin Operator

So, although Elixir is an immutable language, variables in Elixir can be rebound. I can perfectly do this:
```elixir
x = 1
x = 2

IO.puts(x) 
=> 2
```

As you can see I can change the value of a variable even though I had already declared the same variable before. 
This happens because Elixir does not forbids you to set a new value to a variable already declared. However, you can
change this behavior by using the `^` pin operator. What this does? The pin operator forbids a variable to be rebound, 
in other words it does not let you set a new value to a variable after this variable is declared.
Remember that in elixir the `=` sign does not mean an assigment, it's a pattern matching. So:
```elixir
x = 1
```
This means that you are pattern matching x with 1. Since x didn't exist before it assumes the value of 1.
What elixir also allows is the rebound of a variable so, even though the x is 1 when you do this:
```elixir
x = 2
```
what elixir is doing is rebounding the value of x. So, although it's still a pattern matching it's now setting the 
x value to 2.
But what about if you have to really check the pattern matching, to check if x is 2? Hence the pin operator. With 
the pin operator you can do this:
```elixir
x = 2 
^x = 2 # This not fail
^x = 3 # This will raise: ** (MatchError) no match of right hand side value: 2
```
The third line in this code will raise a `MatchError` because it verified that the x value is not 3, therefore we have a
pattern match error.

You may be asking. Ok, but how this is usable or viable to have? Well, imagine you have a list and you want to get the values
in the list to respective variables, but you also want to check that the second item in the list is an Apple, if you do this:
```elixir
fruit = "Orange"
list = [10, "Apple", 11]
[ten, fruit, eleven] = list
```
you will be setting the value 10 to the `ten` variable, the value `"Apple"` to the `fruit` varible, and the value 11 to the `eleven`
variable because elixir will rebound the value of the variable fruit.
To set the values to the variables but also check the pattern match for the `fruit` you need to use the `pin operator` like this:
```elixir
fruit = "Orange"
list = [10, "Apple", 11]
[ten, ^fruit, eleven] = list
=> ** (MatchError) no match of right hand side value: [10, "Apple", 11]
```
This will raise an error because elixir now expects that the second item in the list to have the same value as the variable
`fruit`. This is actually a powerfull tool when you want to pattern match against variables and this is what Ecto uses when 
you need to use variables in `where` clauses.

Now that you know how the pin operator works we can understand how to pass a variable to the `where` clause. So, let's say
I want a function where I can pass a brand and a type and it will return for me all the products for that brand and that type. 
How can I do this? Using the pin operator.
```elixir
import Ecto.Query

def products_by_brand_and_type(brand, type) do
  Product
  |> where([p], p.brand == ^brand and p.type == ^type)
  |> Repo.all
end
```

by doing this you will be able to pass any brand and any type and Ecto will take care of getting the value of the variable 
and set that in the query. Keep in mind, all variables must be set with the `pin operator` 

So, now that you know how to use variables in an Ecto query it's time to find out how Ecto protects you against the oldest
hacking in the Hack Book.

### The very bad SQL Injection

So, you probably already heard about SQL Injection but do you know how it's done? I will explain to you and I also will 
explain how Ecto make it easy to protect you against all of this.

TODO: Example of SQL injection.

TODO: Explain how ecto transforms, when generating the SQL query transforms any variable in the Ecto Query to a 
      [parameterized](parameterized) value in the SQL query.

