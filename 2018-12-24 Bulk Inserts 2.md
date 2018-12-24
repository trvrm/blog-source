Title: Efficient Postgres Bulk Inserts Take 2
Category: Database
Tags: Python, Postgres
Author: Trevor
Date: 2018-12-24


In a previous [post](/bulk-psycopg2-inserts.html) I outlined a technique for achieving highly efficient 
bulk inserts from Python into a Postgres database.

The heart of this technique relies on passing multiple rows to postgres as a *single* 
parameter, and using the `unnest` function to convert that parameter from an
array into a set of rows:


```SQL
INSERT INTO upload_time_test(text,properties)
  SELECT unnest( %(texts)s ) ,
         unnest( %(properties)s)  
```

More recently, I've been using an even more expressive technique that relies on
`jsonb_array_elements`. Like `unnest`, this function takes a single parameter and unrolls
it into multiple rows, but unlike `unnest` we only need to use the function once, 
rather than once per column.

For example, imagine we have a table like this:

```SQL
create table test(id int primary key, firstname text, lastname text, age int);
```

We *could* insert values into it one at a time like this:

```SQL
    INSERT INTO test (id, firstname,lastname,age) 
         VALUES (%(id)s, %(firstname)s, %(lastname)s, %(age)s)

```

and run a Python loop over the rows, calling this insert once for every row:

```python
    INSERT = """
        INSERT INTO test (id, firstname,lastname,age) 
             VALUES (%(id)s, %(firstname)s, %(lastname)s, %(age)s)
    """

    with engine.connect() as connection:
        for row in rows:
            connection.execute(INSERT,row)
```

In testing, it took about 8 seconds to insert 10,000 rows using this technique.
This clearly doesn't scale to larger datasets. We need a way of inserting 
multiple rows simultaneously.

Enter `jsonb_array_elements`:

```python
    INSERT = """
        INSERT INTO test (id, firstname,lastname,age) 
            SELECT 
                (el->>'id')::int,
                el->>'firstname',
                el->>'lastname',
                (el->>'age')::int
              FROM (
                    SELECT jsonb_array_elements(%(data)s) el
              ) a;
    """
    
    with engine.connect() as connection:
        connection.execute(INSERT,data=Json(rows))
        
```

This code took only 70 milliseconds to insert 10,000 rows, representing 
a 100-fold speedup!

A full demonstration of this technique is available at [https://github.com/trvrm/bulktest](https://github.com/trvrm/bulktest)


