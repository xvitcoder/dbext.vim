# dbext.vim

This is a fork of https://github.com/vim-scripts/dbext.vim

This plugin contains functions/mappings/commands to enable Vim to access several databases.

Current databases supported are:
------------
- ODBC / Perl DBI (Any database with a Perl DBI driver)
- MySQL
- Oracle
- Oracle Rdb (VMS)
- SAP HANA
- SAP Sybase SQL Anywhere (SA/ASA)
- SAP Sybase IQ (ASA)
- SAP Sybase Adaptive Server Enterprise (ASE)
- SAP Sybase UltraLite (UL)
- Microsoft SQL Server
- IBM DB2
- Interbase
- SQLite
- PostgreSQL
- Ingres
- Firebird
- Crate.IO


For Perl's DBI layer if the database you are using is not *natively* supported by dbext, but has a DBI interface, dbext's standard feature set is available.  For those already using dbext, the DBI interface should provide a performance boost when running statements against your database.  DBI also provides an ODBC bridge, therefore any ODBC compliant database is also accessible.

> NOTE: As of version 4.0 this plugin requires Vim 7.

> Version 5.0 supports Vim 7's autoload feature.

dbext provides a common interface between your editor and a database.  If your company/project moves onto a new database platform, there is no need to learn the new databases tools.  While editing your SQL (and without leaving Vim) you can execute database commands, run queries, display results, and view database objects.  dbext understands various programming languages, and can parse and prompt the user for [host] variables and execute the resulting statement.  See below for more details.

Adds a menu for the common commands for gvim users.

Some of the features that are supported:

Tutorial
-----------
A tutorial has been added to help you become familiar with the features of the plugin, `:h dbext-tutorial`.
If you dislike reading docs, then at a minimum follow the tutorial.  It will give you the basics of the features and introduce some "best" practices, like creating connection profiles.

Connection Profiles
-----------------------------
You can create as many profiles as you like in your vimrc.  Each profile specifies various connection information.  Each buffer can be connected to a different database.   The plugin will automatically prompt the user for connection information.  If you have defined profiles in your vimrc, for ease of use,  you can choose from a numbered list.

Adding connection profiles is the best way to use dbext, `:h dbext.txt` has lots of examples of different profiles for different databases.

```vim
let g:dbext_default_profile_myASA = 'type=ASA:user=DBA:passwd=SQL'
let g:dbext_default_profile_mySQLServer = 'type=SQLSRV:integratedlogin=1:srvname=mySrv:dbname=myDB'
let g:dbext_default_profile_mySQL = 'type=MYSQL:user=root:passwd=whatever:dbname=mysql'
let g:dbext_default_profile_mySQL_DBI = 'type=DBI:user=root:passwd=whatever:driver=mysql:conn_parms=database=mysql;host=localhost'
let g:dbext_default_profile_myORA = 'type=ORA:srvname=zzz42ga:user=john:passwd=whatever'
let g:dbext_default_profile_myPSG='type=pgsql:host=localhost:user=root:dsnname=myPSG:dbname=myPSG:passwd=whatever'
```

Assuming you work on many different projects, you can automatically have dbext choose the correct database connection profile by adding autocmds that use the filesystem path to choose the correct profile:
```vim
augroup project1
    au!
    " Automatically choose the correct dbext profile 
    autocmd BufRead */projectX/sql/* DBSetOption profile=myASA
augroup end

augroup project2
    au!
    " Automatically choose the correct dbext profile 
    autocmd BufRead */projectY/* DBSetOption profile=myORA
augroup end
```
Or from the menu or the maps created you can choose a profile at any time.

SQL History
-----------
As of version 3.0, dbext maintains a history file which is shared between multiple instances of Vim.  A statement added in one instance of Vim will be immediately available in a different instance of Vim on the same computer.  To re-run a statement you can either press <enter> on the line, or if you prefer the mouse you can double click on the statement.

Modeline Support
---------------------------
Similar to Vim modelines, you can specify connection information as comments within your buffers.  To prevent sensitive information (i.e. passwords) from being visible, you can specify a connection profile as part of your modeline.

Object Completion
----------------------------
dbext ties into Vim dictionary feature.  You can complete table names, procedure names and view names using the `i_CTRL-X_CTRL-K` feature.

Viewing Lists of Objects
------------------------------------
You can browse through the various objects in the database you are connected to and specify wildcard information.  For example you can say, "Show me all tables beginning with 'ml_' ".  These objects are currently supported: Tables, Procedures, Views,  Columns (for a table).

Result Buffer
-------------------
The output from any of the commands is placed into a new buffer called Result.  In the event of an error, both the error and the command line is included for inspection.

Mappings
----------------
There are many maps created for convenience.  They exist for most modes (command, visual and insert).

Place the cursor on a word, and invoke the default mapping (or menu) and a Result buffer will open with the contents of the table displayed (i.e. select * from <word>.  Optionally you can be prompted for the table name, and a WHERE clause.

Describe a table (see column names and datatypes).

Describe a stored procedure (see input and output datatypes).

Visually highlight statements and execute them against the database.

Parsing Statements
-----------------------------
By default any statement will be parsed looking for input parameters (host variables), if any are found you are prompted to enter a suitable value for the parameter.  This option can be turned off either globally or on a per
buffer basis.

```sql
SELECT first_name, city
  FROM customer
 WHERE last_name = @name
```
In the case you will be asked for a value for `@name`.  The rules for defining input parameters are customizable either globally or on a per buffer basis.  The rules for finding these variables can be setup as standard Vim regular expressions.  So if you can find the variables using /, you can easily customize your own settings using your own naming conventions.  See help file for more details.

FileType Support
--------------------------
SQL can be used from a variety of languages.  Each development language (PHP, Perl, Java, ...) language has different syntax for creating SQL statements that are sent to the database.  dbext has support for several
different filetypes, so that it can understand and correctly parse a SQL statement.

The current supported languages are:
        `PHP, Java, JSP, JavaScript, JProperties, Perl, SQL, Vim`

For example assume you had the following Java code:

```java
String mySQL =
    "SELECT s.script, ts.event, t.name, s.script_language, sv.name                " +
    "    FROM ml_script s, ml_table_script ts, ml_table t, ml_script_version sv   " +
    "    WHERE s.script_id = " + script_version                                     +
    "        AND ts.version_id = " + obj.method()                                   +
    "        AND ts.table_id = t.table_id                                       ";
```

If you visually select from the "SELECT ... to the "; and ran
 `:'<,'>DBExecSQL`    (or used the default map `<Leader>se`)

The Java filetype support would concatenate each individual string into one
single string.  In this case it removed the " + " and concatenated  the
lines to result in the following (assuming this is on one line):

```sql
SELECT s.script, ts.event, t.name , s.script_language, sv.name
    FROM ml_script s, ml_table_script ts, ml_table t , ml_script_version sv
    WHERE s.script_id = " + script_version + "
        AND ts.version_id = " + obj.method() + "
        AND ts.table_id = t.table_id
```

Next, it will prompt you for replacement values for the various variables or  objects you used in the string.
Assuming you had the default behaviour turned on, you would be prompted  to supply a value for:  

- `" + script_version + "`
- `" + obj.method() + "`

So assuming you entered:
- `100`
- `'Project_Yahoo'`

Then the resulting string sent to your database would be (again, this would technically be on one line):

```sql
SELECT s.script, ts.event, t.name , s.script_language, sv.name
    FROM ml_script s, ml_table_script ts, ml_table t, ml_script_version sv
    WHERE s.script_id     = 100
        AND ts.version_id = 'Project_Yahoo'
        AND ts.table_id   = t.table_id
```

Benefit:
You did not have to test your SQL by cutting and pasting it into a separate tool and replacing all the object and host variables yourself.  Just by visually selecting the string and running the command `DBExecSQL` (or the default mapping `<Leader>se`) the SQL statement was executed against the database and allowed to you enter host variables.

Additional Commands
---------------------------------
- `DBExecSQL` - Enter any command you want sent to the database
- `DBExecRangeSQL` - Enter any range of statements you want executed
- `Select`  - Enter the remainder of a select (ie :Select * from customer)
- `Call`  - Call a stored procedure
- `Update`  - Enter the remainder of an update
- `Insert`  - Enter the remainder of an insert
- `Delete`  - Enter the remainder of an delete
- `Drop`    - Enter the remainder of a drop
- `Alter`   - Enter the remainder of an alter
- `Create`  - Enter the remainder of a create
