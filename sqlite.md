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
