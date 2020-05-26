---
date: 2020-05-26 10:0:00
layout: post
title: Postgress Connection Pool with Gunicorn and Connexion
subtitle:
description: 
image: https://unsplash.com/photos/ImcUkZ72oUs
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
* create a simple `\` endpoint in which the connection pool is initialized

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

d. Make the connection pool a global Flask variable under `g`

```python
from flask import g

def get_db():
    print ('GETTING CONN')
    if 'db' not in g:
        g.db = application.config['postgreSQL_pool'].getconn()
    return g.db
```

e. Create a simple `\` endpoint in which the connection pool is initialized

```python
from flask import jsonify

@application.app.route('/')
def index():
    db = get_db()
    cursor = db.cursor()

    cursor.execute("select 1;")
    result = cursor.fetchall()
    print(result)

    cursor.close()
    return jsonify(result)
```

#### Putting it all together

Find all the `wsgi.py` in thei gist on Github [here](https://gist.github.com/estambolieva/32cac557eb3fcf44ac968ed23be3157e). 



#### Further Reading

a. this is a useful link which helped me write the code above: [https://gist.github.com/vulcan25/55ce270d76bf78044d067c51e23ae5ad](https://gist.github.com/vulcan25/55ce270d76bf78044d067c51e23ae5ad)