SQLite Crib Sheet
=================

SQLite Command Line Shell
-------------------------

See <http://www.sqlite.org/sqlite.html>

Start using:

    $ sqlite3 DATABASE_FILE

If `DATABASE_FILE` doesn't exist then a new SQLite database will be created.

Special commands (use `.help` to get a full list):

    .tables       : show a list of tables
    .dump ?TABLE? : dumps database contents to SQL
    .exit
    .quit         : exits the command line shell

To dump the database to a file as SQL directly from the bash prompt do e.g.:

    $ echo .dump | sqlite3 DATABASE_FILE > SQL_FILE

Python SQLite interface basics
------------------------------

To acquire the SQLite module:
```python
import sqlite3
```

Connect to SQLite database (file):
```python
cx = sqlite3.connect('db.sqlite')
```

Create cursor object to execute SQL commands:
```python
cu = cx.cursor()
```

Create a table by executing SQL commands:
```python
sql = """CREATE TABLE blah (
           id INTEGER PRIMARY KEY,
           data CHAR NOT NULL,
           more_data CHAR NOT NULL)"""
try:
    cu.execute(sql)
    cx.commit()
except sqlite3.Error,ex:
    print "Error making table: %s" % ex
```

Insert data into a table:
```python
sql = """INSERT INTO blah (data,more_data) VALUES (?,?)"""
try:
    cu.execute(sql,value1,value2)
except sqlite3.Error,ex:
    print "Exception: %s" % ex
```

Do a single commit (nb commits are expensive as they write to file, so it's probably
best not to commit for every insert or update operation):
```python
cx.commit()
```

Perform "select" query:
```python
sql = """SELECT DISTINCT hg_probeset,rn_probeset FROM hg_genes_to_probesets
             INNER JOIN hg_to_rn_genes
             ON hg_genes_to_probesets.hg_gene=hg_to_rn_genes.hg_gene
             INNER JOIN rn_probesets_to_rn_genes
             ON hg_to_rn_genes.rn_gene=rn_probesets_to_rn_genes.rn_gene"""
try:
    cu.execute(sql)
    for row in cu.fetchall():
        ...do stuff with row elements indexed 0,1,...
    except sqlite3.Error,ex:
        print "Error selecting data: %s" % ex
```

SQLite Tools
------------

SQLite Manager in Firefox (add-on): https://addons.mozilla.org/en-US/firefox/addon/sqlite-manager/
