From: https://elixirforum.com/t/difference-in-between-utc-datetime-and-naive-datetime-in-ecto/12551
```
You should use :utc_datetime unless you have a reason not to. The reason why Ecto’s timestamps() 
are naive by default is backwards compatibility. Since they are always UTC anyway, you should use 
:utc_datetime for them (and I believe the default will be changed to that in the future).
The difference between :utc_datetime and :naive_datetime is that the former will ensure that 
you can only insert UTC DateTimes and will read back UTC DateTime structs from the DB, 
whereas the latter will remove timezone information when writing and return NaiveDateTimes 
when you read from the DB.
```
---

From: https://www.w3schools.com/sql/sql_injection.asp
SQL Injection examples