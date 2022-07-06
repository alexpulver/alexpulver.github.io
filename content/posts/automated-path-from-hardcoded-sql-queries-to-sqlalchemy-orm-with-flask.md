---
title: "Automated Path From Hardcoded SQL Queries to SQLAlchemy ORM With Flask"
date: 2020-04-11
---

Let's assume you started by developing the database schema directly on the server (e.g. using MySQL Workbench) and used hardcoded SQL queries in the code (e.g. using one of [Transaction Script, Domain Model or Logic in SQL](http://www.martinfowler.com/articles/dblogic.html) approaches).
 
Then at some point you feel confident to use [SQLAlchemy](https://www.sqlalchemy.org/)'s Object-Relational Mapping (ORM) framework. On a side note, I highly recommend to read [OrmHate](http://martinfowler.com/bliki/OrmHate.html) article by Martin Fowler to get some perspective on when to use ORM frameworks. One benefit is versioning the schema as part of business logic code. There are also many tools to perform migrations, and more importantly, downgrades of the database. The reason I use SQLAlchemy here is because it is considered as one of the most popular SQL frameworks for Python.
 
Below I show how to use [flask-sqlacodegen](https://github.com/ksindi/flask-sqlacodegen) to automatically generate domain models code from an existing database schema, to save you manual work.
It supports [Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/) extension out of the box with `--flask` option, which I found very convenient.
 
In order to generate the models for MySQL, you need to install [MySQL Connector/Python](https://dev.mysql.com/doc/connector-python/en/) (self-contained Python driver by MySQL) and then run `flask-sqlacodegen`. The below command includes `ssl_ca` parameter which specifies a path to SSL CA certificate, which is needed when connecting to database instances over TLS/SSL.

```bash
pip install mysql-connector-python flask-sqlacodegen
flask-sqlacodegen --flask 'mysql+mysqlconnector://{USERNAME}:{PASSWORD}@{HOST}:{PORT}/{DATABASE}?ssl_ca={CERTIFICATE_PATH}' > models.py
```

As easy as that!
