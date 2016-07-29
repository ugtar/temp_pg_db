temp_pg_db
==========

Provides a simple class for creating a temporary userland Postgres db server.

The `TempDB()` class constructor will set up the database server in a temporary
directory.  If any databases are specified they are created inside the newly
created server.  The class provides a `cleanup()` method for stopping the
server and disposing of the temporary files.

The module also provides an `init_temp_db()` function for managing the `TempDB`
class as a singleton.  This is useful for scenarios where you want to import in
more than one module, but ensure that they all use the same database (and that
the server and database are only created once).  For example, this is handy in
unit test code where you want to be able to test a single file or use a test
runner to run tests in multiple files with only a single database.

`init_temp_db()` also registers an `atexit` handler to cleanup the temp
database when the process exits.

Examples:
---------

Create a temporary server with a database called 'testdb':
```python

from temp_pg_db import TempDB
temp_db = TempDB(databases=['testdb'])

# you can connect to this database using temp_db's pg_socket_dir
connection = psycopg2.connect(host=temp_db.pg_socket_dir, database='testdb')

# ... do stuff...

temp_db.cleanup()
```

Alternatively, useful in a test runner:

```python
import temp_pg_db
temp_db = temp_pg_db.init_temp_db(databases=['testdb'])
# repeat above in multiple modules

# you can connect to this database using temp_db's pg_socket_dir
connection = psycopg2.connect(host=temp_db.pg_socket_dir, database='testdb')

# ... do stuff...
# the db is automatically cleaned up when the process exits
```

Last an interactive example:
```python
>>> import temp_pg_db
>>> import psycopg2
>>> temp_db = temp_pg_db.TempDB(databases=['testdb'])
Creating temp PG server... done
(Connect on: `psql -h /var/folders/d7/n3_h9vnn3w3bbmsnbdb73fmw0000gn/T/pg_tmp_OQMGwC/socket`)
>>> connection = psycopg2.connect(host=temp_db.pg_socket_dir, database='testdb')
>>> cur = connection.cursor()
# The rest is stolen from psycopg's documentation.  You get the idea...
>>> cur.execute("CREATE TABLE test (id serial PRIMARY KEY, num integer, data varchar);")
>>> cur.execute("INSERT INTO test (num, data) VALUES (%s, %s)",
... (100, "abc'def"))
>>> cur.execute("SELECT * FROM test;")
>>> cur.fetchone()
(1, 100, "abc'def")
>>> connection.close()
>>> temp_db.cleanup()
```
