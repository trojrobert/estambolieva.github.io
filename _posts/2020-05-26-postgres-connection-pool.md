---
date: 2020-05-26 10:0:00
layout: post
title: Postgress Connection Pool with Gunicorn and Connexion
subtitle:
description: 
image: https://images.unsplash.com/photo-1534224039826-c7a0eda0e6b3?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
category: programming
tags:
  - python
  - connexion
  - gunicorn
  - psycopg2
author: estambolieva
---

I have been working on an API I am developing for several months. It is a [swagger](https://swagger.io/)-generated API running on a [gunicorn](https://gunicorn.org/) WSGI production server.

The API reads data from a [Postgresql](https://www.postgresql.org/) database and delivers it to my web app. Initially, I had few endpoints which I connected to rarely. Each endpoint would then open a connection to the database, read and deliver the data, and close the connection to the database. 

This is how the main app code looked like (wsgi.py) - simple and neat: 

```python
import connexion
from flask_cors import CORS
from swagger_server import encoder

application = connexion.App(__name__, specification_dir='./swagger/')
CORS(application.app)
application.app.json_encoder = encoder.JSONEncoder
application.add_api('swagger.yaml', arguments={'title': 'My Simple API'})

if __name__ == '__main__':
    application.run(port=8080)
```

My web app's demand to consume data grew with time, and it made no sense any more to open and close so many connections to the database - this only slowed my code and the responsiveness of the web app. Thus it is better for the API to keep one or more connections open to the database at all times so the data retrieval speeds up. This is achieved by creating a [connection pool](https://en.wikipedia.org/wiki/Connection_pool).

*Note*: It is advisable to use frameworks like [Django](https://www.djangoproject.com/) instead of [Flask](https://flask.palletsprojects.com/en/1.1.x/) in production environments which manage automatically the connection pools instead of implementing this yourselves. In case you do want to use this as a quick fix, here is how I modified my code: 


#### Connection Pool

(citing Wikipedia here) A connection pool is a cache of database connections maintained so that the connections can be reused when future requests to the database are required.

This means that when we create the `application` in `wsgi.py` we also need to:
* read db name, URL, port and connection credentials from a `config.ini` file using `ConfigParser`
* create a connection pool
* make sure we put conditions to close the db connection - *tear down*
* make the connection pool a global Flask variable

a. Read db name, URL, port and connection credentials from a `config.ini` file using `ConfigParser`

```python
config = configparser.ConfigParser()
config.read_file(open('path_to_config.ini_file'))
postgres_url = config.get('POSTGRES', 'URL')
postgres_port = config.get('POSTGRES', 'PORT')
postgres_user = config.get('POSTGRES', 'USER')
postgres_pwd = config.get('POSTGRES', 'PW')
postgres_db = config.get('POSTGRES', 'DB')
```

b. Create a connection pool

```python
application.app.config['postgreSQL_pool'] = 
	psycopg2.pool.ThreadedConnectionPool(1, 20,
        user = postgres_user,
        password = postgres_pwd,
        host = postgres_url,
        port = postgres_port,
        database = postgres_db)
```

c. Make sure we put conditions to close the db connection - *tear down*

```python
@application.app.teardown_appcontext
def close_conn(e):
    print('CLOSING CONN')
    db = g.pop('db', None)
    if db is not None:
        application.config['postgreSQL_pool'].putconn(db)
```

d. Make the connection pool a global Flask variable under `g` - so it can be accessed by the controllers:

```python
from flask import g

@application.app.before_request
def before_request():
    g.db = application.app.config['postgreSQL_pool'].getconn()
```

#### Putting it all together

Find all the `wsgi.py` in thei gist on Github [here](https://gist.github.com/estambolieva/32cac557eb3fcf44ac968ed23be3157e). 


#### Calling the conn from the Controllers

In swagger, the controllers are in charge of defining what happens in every endpoint. Usually, all controllers are in the `controllers` directory, which is at the same level as the `models` and `swagger` directories. 

To use the open database connections in the connection pool, we need to get this open connection from `g`.

```python
# base_controller.py
from flask import g

def get_from_db():
	conn = g.db
    cur = conn.cursor()

    data= None
    #ToDo: DO SOMETHING - usually retrieve data from the database using the cursor
    cur.close()

    return data
```

#### How does the connection pool look like


I queried the open connections to the database to test how things would work out using this `psql` command:

```sh
SELECT * FROM pg_stat_activity;
```

I executed this in [DBevaer Community](https://dbeaver.io/download/) at 10:39 on 27 May 2020. We can see 2 of the connections that DBevaer opened to the database as number 4 and number 5 in the screenshot below:

![Connection Pool as seen on DBevaer](https://raw.githubusercontent.com/estambolieva/estambolieva.github.io/master/assets/img/uploads/postgress_connection_pool/psql_conn_pool_example.png)

Then I ran my swagger API on locahlost at 10:41 which opened 4 connection as predicted for the connection pool. I use the command below to start the API from my terminal:

```sh
gunicorn "swagger_server.wsgi" -w 4 -b 0.0.0.0:8080
```

It created 4 db connections for the connection pool, listed in line number 6 to 9 - see starting time in the column titled *backend_start*. 

Then I called the *base_controller.py* 8 times - and I could see that the connections were kept open from 10:41 while executing at the different times I called the controller - at 10:46 and 10:47 - see column *query_start* in the screenshot.


#### Further Reading

a. this is a useful link which helped me write the code above: [https://gist.github.com/vulcan25/55ce270d76bf78044d067c51e23ae5ad](https://gist.github.com/vulcan25/55ce270d76bf78044d067c51e23ae5ad)

b. DB connection pool was undefined when called from `g` in the `base_controller.py`. This is a similar error to what there was which shows how to use `before_request`. Read [here](https://stackoverflow.com/questions/21138025/attributeerror-appctxglobals-object-has-no-attribute-user-in-flask)