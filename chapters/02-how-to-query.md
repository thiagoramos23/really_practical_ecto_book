# How to Query

Getting data from the database is one of the crucial activities a developer need to learn when he or she is learning a new framework and since the name of this book has `practical` in it I will stop talking now and start with the examples.

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

And now if you want to get all the products, what you need to do? The answer is pretty simple you just need to use the `Repo` module and use one of the numerous functions to get all the data.

```elixir
Repo.all(Product)
```
```sql
SELECT * FROM products;
```

Throughout this book every time I show some code that gets some data from the database I will also show the related SQL query the code is producing.

Even though this is somewhat very simple and you can probably use in your real projects we have another way to get data and it is using the `Ecto.Query` module. So to get all products with `Ecto.Query` we would need to do:

```elixir
import Ecto.Query

query = from p in Product

Repo.all(query)
```
```sql
SELECT * FROM products;
```

This will get the same results. So you maybe asking yourself, well, why this people insist in having multiple ways of doing the same thing? I will tell you next. 

I've got say that I thought the same thing but there is a pretty good explanation for that and in fact the first example without `Ecto.Query` is just a kind of syntactic sugar for you so you don't need to write the query.
Ecto is a very powerful framework and you can, pretty much, do all the queries that you need to do, even the hard ones, without ever need to write the SQL queries for it, just using the `Ecto.Query` code.

## Getting data without schema

So, imagine now you don't have schemas. Imagine that you just want a simple app to get some data from the 
database and nothing more, you don't want to write a lot of modules with schemas and fields and all the code you'd need to write so you can have access to fields and stuff. Right now you just want to get some data. 
So, how do you do?

Turns out `Ecto.Query` is the perfect solution for your problem. Let's get all the data we need from the products table without even have any schema defined.

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

So, we have a small difference here, because we don't have any schema, `Ecto` can't possibly know what would be the data to return, so we need to tell it the name of the columns we should return in the query. Another thing to be aware of is that we can use other data structures in the `select` clause. If you aren't anything like me
and you paid attention to every piece of code I showed to you so far you had noticed that we are using a `List` data structure as the return value of the `select` clause. Another example using a `Map` would be:

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

So, one thing to keep in mind is that you can pretty much use the data structure you feel is the best choice for your case. This last example is using a `Tuple`:

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

As you can see the only thing that changed is that now we are using the `Ecto.Schema` instead of using a simple string, the other thing will continue to be the same.

So far I showed to you how to get all the data from a table in the database but usually we don't want to do that.
We usually only want a set of all the data that is there. In the next sections we will learn how to:
- Use the `where` clause
- How to paginate data so we can get a small set of data each time
- How to use `join` to get data from `Ecto.Schema` or direct into tables that are associated (have association between foreign keys).

Again, I didn't write this book to show my english language skills, I wrote this because I want a book that is succint and to the point. So let's go.

## Where clause

We often want to get data from one of more tables that contain a lot of rows, I mean, A LOT. It's not viable nor feasible to get all the data from a table every time. Continuing with our `products` table, let's add some new fields
to it, don't worry about how to add fields to a table or even how to create tables and schemas for now, we will learn all about this in the other chapters. For now, let's suppose our `Product` schema have these fields:

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

Let's create some data, you can enter the elixir console with `iex -S mix` inside the folder of the project and then you can copy and paste the code below to create some shoes in your database:

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

The answer to that is the `where clause`. Let's see how this how to achieve this with ecto. There are two ways to do this, the first one is using functions:

```elixir
Product
|> Ecto.Query.where(brand: "Nike")
|> Ecto.Query.where(type: "shoes")
|> Repo.all
```

If you don't want to repeat the `Ecto.Query` every time you can use the `import`.

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

Here I opted to use just one `where` and use the `and` to build the query that we need. One thing to keep in mind when using more than one `where` clause is that they all translate to `and` in the end. Next we will see how to use the the `where` clause with the `or`.

Now your manager wants to know all the nike shoes we have but also want to know all the other types of products we have that are not shoes.

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

We can also use the `or_where` and in this case it would work the same way. Just keep in mind that the `or_where` is used when we want to concatenate more than on `where` function using the `or` instead of the `and` which is the default. Here is
the [documentation](https://hexdocs.pm/ecto/Ecto.Query.html#or_where/3) about `or_where` that explains the usage. Bottom line the query above could be rewritten like this:

```elixir
import Ecto.Query

Product
|> where([p], p.brand == "Nike" and p.type == "shoes")
|> or_where([p], p.type != "shoes")
|> Repo.all
```

The result is the same and the SQL query is also the same. One thing to keep in mind is that the `where clause` can get really complex, specially when we are composing queries but I hope I can show you some techniques to make your life easier.

Ok, we did those `where` clauses but we are always using strings as the values. What if I want to a query to return products
of a particular brand and particular type but I don't want to hardcode the name of the brand neither the type in the 
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
This happens because Elixir does not forbids you to set a new value to a variable already declared. However, you can change this behavior by using the `^` pin operator. What this does? The pin operator forbids a variable to be rebound, in other words it does not let you set a new value to a variable after this variable is declared.
Remember that in elixir the `=` sign does not mean an assignment, it's a pattern matching. So:
```elixir
x = 1
```
This means that you are pattern matching x with 1. Since x didn't exist before it assumes the value of 1.
What elixir also allows is the rebound of a variable so, even though the x is 1 when you do this:
```elixir
x = 2
```
what elixir is doing is rebounding the value of x. So, although it's still a pattern matching it's now setting the x value to 2.
But what about if you have to really check the pattern matching, to check if x is 2? Hence the pin operator. With the pin operator you can do this:
```elixir
x = 2 
^x = 2 # This not fail
^x = 3 # This will raise: ** (MatchError) no match of right hand side value: 2
```
The third line in this code will raise a `MatchError` because it verified that the x value is not 3, therefore we have a pattern match error.

You may be asking. Ok, but how this is usable or viable to have? Well, imagine you have a list and you want to get the values in the list to respective variables, but you also want to check that the second item in the list is an Apple, if you do this:
```elixir
fruit = "Orange"
list = [10, "Apple", 11]
[ten, fruit, eleven] = list
```
you will be setting the value 10 to the `ten` variable, the value `"Apple"` to the `fruit` variable, and the value 11 to the `eleven` variable because elixir will rebound the value of the variable fruit.
To set the values to the variables but also check the pattern match for the `fruit` you need to use the `pin operator` like this:
```elixir
fruit = "Orange"
list = [10, "Apple", 11]
[ten, ^fruit, eleven] = list
=> ** (MatchError) no match of right hand side value: [10, "Apple", 11]
```
This will raise an error because elixir now expects that the second item in the list to have the same value as the variable
`fruit`. This is actually a powerful tool when you want to pattern match against variables and this is what Ecto uses when you need to use variables in `where` clauses.

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

by doing this you will be able to pass any brand and any type and Ecto will take care of getting the value of the variable and set that in the query. Keep in mind, all variables must be set with the `pin operator` 

So, now that you know how to use variables in an Ecto query it's time to find out how Ecto protects you against the oldest hacking in the Hack Book.

### The very bad SQL Injection

So, you probably already heard about SQL Injection but do you know how it's done? I will explain to you and I also will explain how Ecto make it easy to protect you against all of this.

I will show to you some examples of SQL injection starting by the 1=1 problem. Imagine you have some part of your code where you wrote this SQL query to get user from an ID that is informed in some form:
```sql
"SELECT * FROM users WHERE id = " + txt_id;
```

So, if the user of the form write this in the input text:

```
10 OR 1=1
```

When you run the SQL query this will be translated to:
```SQL
"SELECT * FROM users WHERE id = 10 OR 1=1";
```

This is a perfectly valid SQL query and it will return all the users from your database. And the probabilities tell us that if you are allowing this to happen chances are that the passwords in you `users` database are not encrypted. And by doing this the  person will have access to all the users in your database and, potentially, their passwords in plain text. 

Another trick in the book is the SQL injection based on `""=""` is always true. It goes like this:

Imagine you have a form with email and password so the user can login into your system. To get the user logged in you do this SQL query:
```SQL
"SELECT * FROM users WHERE email = "' + txt_email + 
'" AND password = "' + txt_password + '";
```

If the user inform normal values like an email and a password then everything will be ok. But the user can also write this in the form fields:
```
txt_email = " or ""="
txt_password = " or ""="
```

This will result in a SQL like this:
```SQL
SELECT * FROM Users WHERE Name ="" or ""="" AND Pass ="" or ""="";
```

And this will always be true for every row. Which means you will return every user from the database. 

But rest assured, if you use any ORM (Object Relation Mapper) frameworks available today will be more than safe. Does not matter if it's Active Record for Rails or Eloquent for Laravel. Those problems will probably never happen with you if you use those frameworks correctly. And if you use Ecto this problem will also not happen too. Keep in mind that I didn't put Ecto in the same place as the other ORMs because Ecto is not an ORM. Throughout the book I will explain this better but for now just trust my words.

Ok! But now you maybe asking, you explained the problem but how Ecto solves this problem for us? What Ecto does to prevent this from happening. 

**Short answer: Ecto transforms any variable in the Ecto Query to a parameterized value in the SQL query**

**Now the long answer:**

The way to not suffer from SQL Injection we use something called a prepared statement. So instead of a sql query like this:
```sql
"SELECT * FROM users WHERE email = "' + txt_email + 
'" AND password = "' + txt_password + '";
```

You write a sql query like this:
```sql
SELECT * FROM users WHERE email = ? AND password = ?
```

So, right now when you run this query you will provide the parameters and when the database runs this query it will first call a `prepare statement` code (you will usually use the code from the language or the framework will do this for you). This `prepare statement` will parse the query and add placeholders where you put `?`. So when you run this:
```sql
txt_usernam = "admin"
txt_password = "a' or '1'='1"

SELECT * FROM users WHERE username = ? AND password = ?
```
And the system will use the `txt_email` as the value for the first `?` and the `txt_password` as the value for the second `?` but because the `prepared statement` the SQL query is pre-compiled with placeholders and the data from the `txt_email` and `txt_password` is added later so the interpolation will not take any effect and the result of this query would be empty.

So, what exactly happens with Ecto. Ecto uses the same strategy. It parses the query and uses the parameterized parameters, creating placeholders in the query so it can replace the placeholders for the values in the params later on when it executing the query.

Let's get the same query and write using Ecto
```elixir
import Ecto.Query
def get_user_by_username_and_password(username, password) do
	query = from u in Users, where: u.username = ^username and
			u.password = ^password
	Repo.one(query)
end
```
Don't bother about the `Repo.one` we will talk about it later, for now just keep in mind that when you are sure the result of the query will always return one element in the result you can use `Repo.one` instead of `Repo.all`. 
So, as you can see Ecto does not let you use a query param without use the pin operator, but the way Ecto requires you to write the query forces you to set the params in a way that Ecto will be able to parse the query with placeholders and then it will add the values in the params to execute the query. The best thing here is that it does all of that behind the scene. 

##### Note
> Ecto supports the use of raw SQL queries but I don't advise to do this because it opens a port to some SQL Injection. Of course, each case is different and if you know for sure that the data being passed to the query is always validated by the code you wrote you can sleep in peace. But keep this mind!

## How to Order by

First of all why we need to order things. Maybe you are a maniac for ordering things, maybe it's because you want to put some order into this world, maybe it's because you are tired to live in a world full of disorder. :) 

Well, in a more restrict way we do want to order our queries because otherwise you can't actually rely on the order of the results because this is highly dependent on the database and how the database organize indexes and cache.

So, it's always a good idea to explicitly order your queries, but how can we do this using Ecto? 

There are some ways. First I will show you the one that is closer to the SQL writing using the keywords in Ecto Query:
```elixir
import Ecto.Query

query = from p in Product, order_by: [asc: p.name]
Repo.all(query)
```
```sql
SELECT * FROM products ORDER BY name ASC;
```

The other one could be accomplish using the Expressions:
```elixir
import Ecto.Query

Product 
|> order_by(asc: :name)
|> Repo.all()
```

The order by can be used using the `asc` or `desc` where the `asc` will order your query in an ascendent order and the `desc` in the descendent order. What does that mean?

It means that if you have a list or users with names like these:
```text
Anna
Claire
Belle
Apple
Paul
John
```

If you order them by `asc` you will get an ascendent order using the alphabet. Which means the query will first order using the first letter and if it finds more than one record with the same letter it will look at the second letter and so on and so forth. 

This list in `asc` order would be like:
```
Anna
Apple
Belle
Claire
John 
Paul
```

And it will be the inverse if you order them by `desc`.

How do we order for more than one column?

```elixir
import Ecto.Query
query = from p in Product, order_by: [asc: p.name, desc: p.color]
Repo.all(query)
```

```sql
SELECT * FROM products ORDER BY name ASC, color DESC;
```

Using the expressions would be:

```elixir
Product
|> order_by([asc: :name, desc: :color])
|> Repo.all()
```

Or:

```elixir
Product
|> order_by([p], [asc: p.name, desc: p.color])
|> Repo.all()
```

Or:

```elixir
Product
|> order_by(asc: :name)
|> order_by(desc: :color)
```

So, there are a bunch ways and permutations to use this. Let's continue, in this chapter you will also see how to create a module so we can build queries with functions and use the way real professionals use.

## How to Get only one Result

So, we already saw a little bit how to get only one result using the `Repo.one` function. But there is a caveat. 

You need to keep in mind when you are using this function that you need to be sure the query you are using is only returning one result or nothing. Because if it gets more than one result you will get an error. So, this is something you need to be aware of. Usually, in the real world you will see a code something like this:

```elixir
def get_user(%{email: email} = _user) do
	query = from u in User, where: u.email = ^email
	case Repo.one(query) do
	  nil -> {:error, :not_found}
	  user -> {:ok, user}
	end
end 
```

This code is dealing with the fact that the database could not have any user and you want to return an error tuple instead of return nil. Of course you can always return nil if you want to but in this case we are leveraging the fact that returning tuples as `{:error, THE_ERROR_MESSAGE}` or `{:ok, THE_RESULT}` is an accepted idiom for elixir programs. But hey, the function is yours you can return whatever you want to.

We can also use the `Repo.get_by` which will do the same but you can use the key that you want to search for.

```elixir
def get_user(%{email: email} = _user) do
	case Repo.get_by(User, email: ^email) do
	  nil -> {:error, :not_found}
	  user -> {:ok, user}
	end
end 
```

Using `get_by` you can also use with more than one key:

```elixir
	Repo.get_by(User, email: ^email, password: ^password)
```

And this will try to get the user with the email and the password .

And if you know the value of the primary key of the resource you want to get you can do:

```elixir
Repo.get(User, 10)
```

This will also return a nil or, in this case, the user.

So, this is how you get just one result instead of many.

## How to use Enums in the Query

First let's understand what the Ecto.Enum does. So, Ecto.Enum is responsible to convert an atom value to a string so we can store in the database and do the conversion from string to atom  when we want to use in our application. But it does not only that. With Ecto.Enum you can have a field that accepts one value or a list of values and this is done easily. 

Also, Ecto.Enum has some great integrations with Phoenix Forms as we will see later in the book when we build a simple Phoenix Application.

Let's see how it is a schema with an Ecto.Enum. Let's remember the products schema and change the field type to use the Ecto.Enum.

```elixir
defmodule Product do
  use Ecto.Schema
  
  schema "products" do
    field :name, :string
    field :color, :string
    field :type, Ecto.Enum, values: [:shoes, :shirt, :jeans, :sports]
    field :brand, :string
    field :manufactured_at, :datetime
  end
end
```

And the migration(don't worry we will talk about migrations in details in other chapters) to add this field would be something like that:
```elixir
def change do
  alter table(:products) do
    field :type, :string
  end
end
```

What will happen when we insert some data into the database? What will happen is that the values that are represented as atoms (:shoes, :shirt, :jeans, :sports) will be inserted in the database as strings. 

So, if you have a Phoenix form you can also do:

```elixir
<%= select f, :type, Ecto.Enum.values(Product, :type) %>
```

And this will list all the types that you can select in the dropdown. 
You can also want to store more than one value in the field. Let's suppose you have a product like a Nike Jordan shoe which is both a shoe and it could also be in the sports type. What do you do?

Well, it's super simple. You would change the migration to be:
```elixir
def change do
  alter table(:products) do
    field :type, {:array, :string}, default: []
  end
end
```

And now the Product schema would be like this:

```elixir
defmodule Product do
  use Ecto.Schema
  
  schema "products" do
    field :name, :string
    field :color, :string
    field :type, {:array, Ecto.Enum}, values: [:shoes, :shirt, :jeans, :sports]
    field :brand, :string
    field :manufactured_at, :datetime
  end
end
```

And now in the form you would just change to a multi select field:

```elixir
<%= multiple_select f, :type, Ecto.Enum.values(Product, :type) %>
```

And now your data will be inserted as an array in the field for the types.

So, as you can see it's valuable to use the Ecto.Enum instead of just simple strings. But now we need to get this data from the database. So, how can we do it?

// TODO: How to query with Ecto.Enum

- We can specify the acceptable values
- Has some nice helpers to be used in conjunction with Forms.
- We can also insert as an {:array, :string}
- the field would be {:array, Ecto.Enum}

















