---
date: 2020-03-25 13:17:00
layout: post
title: Bulk insert into database table using placeholders, Postgres and Python 3
subtitle:
description: 
image: https://images.unsplash.com/photo-1468070454955-c5b6932bd08d?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
<!-- optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559825288/theme17_nlndhx.jpg -->
category: startup
tags:
  - database
  - bulk insert
  - python 3
  - postgresql
author: estambolieva
---

I was refactoring and optimizing the code we have for writing into our database today.
At some point I had problems bulk inserting into a table while inserting values some of which we [UUIDs](https://en.wikipedia.org/wiki/Universally_unique_identifier).
This is what I did after that:

### INSERT INTO

Standard `psql` syntax for inserting data into a db table looks like:
```psql
INSERT INTO
services (service_id, service_name)
VALUES ('86a1bf1d-523f-4b64-a333-ce8e1b6d8c56', 'Deliver Food');
```
This inserts a new service to the *services* table the the db, which has an unique id '86a1bf1d-523f-4b64-a333-ce8e1b6d8c56' and is written in the *service_id* column in the table. The service name - 'Deliver Food', is written in the *service_name* column of the table. 

### Bulk INSERT INTO

To insert more than one services into the table, we need to call:
```psql
INSERT INTO 
services (service_id, service_name) 
VALUES 
('86a1bf1d-523f-4b64-a333-ce8e1b6d8c56', 'Deliver Food'), 
('2b051dd2-70cc-4f06-a27f-767cb6309533', 'Deliver Drinks');
```

### Psycopg2: how to handle UUIDs instead of converting them to strings

Simply call the method `register_uuid()` before you start generating and handling UUIDs:

```python
import psycopg2.extras
psycopg2.extras.register_uuid()
```

This solved this error:
```sh
can't adapt type 'UUID'

```

### Python method to bulk insert using placeholders

This [Stackoverflow answer](https://stackoverflow.com/questions/8134602/psycopg2-insert-multiple-rows-with-one-query?answertab=votes#tab-top) helped me discover a quick way to programatically bulk insert formatting the values inserted in the table using `cur.mogrify` (docs), however it was producing a binary string to be concatenated to the `command` string which yielded the following error:

```sh
sequence item 0: expected str instance, bytes found
```

To solve it, I needed to cast the binary string to a normal one adding `decode('urf-8')` to `cur.mogrify` (see method below). 

And putting it al together:
```python
def bulk_insert_into_table(cur, table_name, column_names, data):
    """Build a psql string command to bulk insert into a table

                :param cur: the db cursor
                :type id: cursor
                :param table_name: the name of the existing table in which the insertion happens
                :type name: str
                :param column_names: a list of strings -> the strings are the column names of the table
                :type name: str
                :param data: a list of tuples, each tuple holding the values to the inserted into the table
                :type name: str
    """
    column_names = ','.join(column_names)

    args_str = b','.join(cur.mogrify("(%s,%s)", x) for x in data).decode("utf-8")
    # decode was needed as args_str was a binary string which failed to be inserted into the command string

    command = """ INSERT INTO %s (%s) VALUES %s;""" % (table_name, column_names, args_str)

    return command
```

```python
conn = psycopg2.connect(database = psql_db, user = psql_user, password = psql_ps,
                            host = psql_url, port = psql_port)
cur = conn.cursor()
psycopg2.extras.register_uuid()

table_name = 'services'
column_names = ["service_id", "service_name"]
data = [(uuid4(), 'Deliver Food'),
        (uuid4(), 'Deliver Drinks')]

command = bulk_insert_into_table(cur, table_name, column_names, data)
cur.execute(command)

cur.close()
conn.commit() # <--- makes sure the change is shown in the database
conn.close()
```