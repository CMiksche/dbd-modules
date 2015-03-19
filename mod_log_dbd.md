# Introduction #
**mod\_log\_dbd** has one directive:

### DBDLog   FILENAME   SQL    `[`USENULLS`]` ###

**DBDLog** is always paired with a **[CustomLog](http://httpd.apache.org/docs/current/mod/mod_log_config.html#customlog)** directive using the same file name.

The **[CustomLog](http://httpd.apache.org/docs/current/mod/mod_log_config.html#customlog)** format must be a comma-separated list of [mod\_log\_config](http://httpd.apache.org/docs/current/mod/mod_log_config.html)  ["%" directives](http://httpd.apache.org/docs/current/mod/mod_log_config.html#formats).
```
CustomLog   logs/access.sql     "%h, %l, %u, %{%Y-%m-%d %H:%M:%S}t, %r, %>s, %b"
DBDLog      logs/access.sql    "INSERT INTO log_table (Host, Rname, User, Tstmp, Request, Status, Bytes) VALUES (%s, %s, %s, %s, %s, %s, %s)"
```

Only ["%" directives](http://httpd.apache.org/docs/current/mod/mod_log_config.html#formats), commas, and spaces can appear in the **[CustomLog](http://httpd.apache.org/docs/current/mod/mod_log_config.html#customlog)** format.

The **SQL** parameter markers must correspond to the **[CustomLog](http://httpd.apache.org/docs/current/mod/mod_log_config.html#customlog)** ["%" directives](http://httpd.apache.org/docs/current/mod/mod_log_config.html#formats).

Notice how in the example above: there are seven comma-separated fields in the **[CustomLog](http://httpd.apache.org/docs/current/mod/mod_log_config.html#customlog)** directive ( `%{%Y-%m-%d %H:%M:%S}t` _is a single field_) which match the seven comma-separated `%s` parameter markers in the **DBDLog** directive.

Access log records are inserted into the database.  Access log records are only written to the file `logs/access.sql` _(as SQL statements)_ if the database is inaccessible.

DBDLog can be used at the Server or the Virtual Host level.

# Details #
**FILENAME** must match the filename used in the **[CustomLog](http://httpd.apache.org/docs/current/mod/mod_log_config.html#customlog)** directive.

**SQL** is an SQL statement whose parameter markers correspond exactly to the **[CustomLog](http://httpd.apache.org/docs/current/mod/mod_log_config.html#customlog)** ["%" directives](http://httpd.apache.org/docs/current/mod/mod_log_config.html#formats).

**USENULLS** causes any parameters which are a single hyphen to be inserted into the database as NULL values. Make sure both your DBD driver and your database can handle NULL values. The [odbc-dbd driver](http://odbc-dbd.googlecode.com/) v1.0.6 or higher and the PostgreSQL DBD driver can process NULL values.

  * Apache requests will be logged to the database using the SQL statement. If the database cannot be accessed, the SQL statement (with the data inserted) is written to the file, and the error is written to the error log.

  * Do not put any characters between the format codes in the log format except a comma and (optionally) spaces. Characters inside the curly braces of `%{...}` are OK, so the date  `%{Year:%Y Month:%m Day:%d}t` is acceptable.

  * If you have literal strings in your SQL statement, always enclose them in single-quotes; even if your database allows other forms, like enclosing literals in double-quotes. Your server may be vulnerable to SQL-injection attacks if you do not use single-quotes for string literals.

  * You must have an Apache [DBD driver](http://people.apache.org/~niq/dbd.html) and [mod\_dbd](http://httpd.apache.org/docs/current/mod/mod_dbd.html) loaded and configured to use mod\_log\_dbd.

  * You must have [mod\_log\_config](http://httpd.apache.org/docs/current/mod/mod_log_config.html) loaded and configured to use mod\_log\_dbd.

  * The [LoadModule](http://httpd.apache.org/docs/current/mod/mod_so.html#loadmodule) directives for both [mod\_log\_config](http://httpd.apache.org/docs/current/mod/mod_log_config.html) and [mod\_dbd](http://httpd.apache.org/docs/current/mod/mod_dbd.html) must occur _**before**_ the [LoadModule](http://httpd.apache.org/docs/current/mod/mod_so.html#loadmodule) directive for mod\_log\_dbd in the configuration file.

  * Use [LoadModule](http://httpd.apache.org/docs/current/mod/mod_so.html#loadmodule) to enable mod\_log\_dbd, like this:
```
LoadModule log_dbd_module modules/mod_log_dbd.so 
```

  * **SQL** should return no rows.  It is typically a DML (data manipulation language) SQL statement: either an INSERT, UPDATE statement, or a call to a stored procedure.

  * Do not put a semicolon at the end of your SQL statement.

  * Do not use DBDLog with a **[CookieLog](http://httpd.apache.org/docs/current/mod/mod_log_config.html#cookielog)** directive. Use a **[CustomLog](http://httpd.apache.org/docs/current/mod/mod_log_config.html#customlog)** directive with a `%{Cookie}n` format instead.


  * **DBDLog** can use the name of a statement as its **SQL** parameter.  This name must have been previously defined by a **[DBDPrepareSQL](http://httpd.apache.org/docs/current/mod/mod_dbd.html#dbdpreparesql)** directive. For example:
```


LogFormat      "%V, %r"  dbFormat
DBDPrepareSQL  "INSERT INTO log_table (Server, Url) VALUES (%s, %s)"  dbLog
...
CustomLog logs/access.sql  dbFormat
DBDLog    logs/access.sql  dbLog  UseNULLs
```

  * Unfortunately, using a statement prepared with [DBDPrepareSQL](http://httpd.apache.org/docs/current/mod/mod_dbd.html#dbdpreparesql) precludes using **FILENAME** directly for database recovery because the original SQL statement text is no longer available to **DBDLog** when writing to **FILENAME**.  The prepared statement label is used as if it were a SQL function when writing to **FILENAME**.  You will need to edit **FILENAME** before it can be used for database recovery. Some databases (for example: [Firebird](http://firebirdsql.org)) may report errors when statements for **DBDLog** are prepared with [DBDPrepareSQL](http://httpd.apache.org/docs/current/mod/mod_dbd.html#dbdpreparesql). In this case, [DBDPrepareSQL](http://httpd.apache.org/docs/current/mod/mod_dbd.html#dbdpreparesql) cannot be used for log statements.

  * For Apache 2.2 prior to version 2.2.9, use parameter markers appropriate to your database.  For example: Use **?** for [ODBC](http://odbc-dbd.googlecode.com/) or SQLite.  Use **%s** for MySQL, etc.

  * For Apache 2.2.9+ always use **%s** as a parameter marker in your SQL statement.

  * If you do not want log entries to be saved to a file when the database is inaccessable, set **FILENAME** to **NUL** _(on Windows)_ or to **/dev/null** _(on Unix)_.

  * Setting **[LogLevel](http://httpd.apache.org/docs/current/mod/core.html#loglevel) debug** will print additional information in the Apache error log which can help to diagnose SQL query problems.

# Database Considerations #
## Suitable Databases for Logging ##

Databases which do not support concurrent updates by multiple threads, or which lock the entire database on update _(like [SQLite](http://www.sqlite.org))_ generally cannot be used successfully for logging.

If you _must_ try to use such a database, set [DBDMax](http://httpd.apache.org/docs/current/mod/mod_dbd.html#dbdmax), [DBDMin](http://httpd.apache.org/docs/current/mod/mod_dbd.html#dbdmin), and [DBDKeep](http://httpd.apache.org/docs/current/mod/mod_dbd.html#dbdkeep) to 1 to prevent concurrent update attempts from the same Apache process.

[Oracle](http://www.oracle.com/database/index.html), [MySQL](http://www.mysql.com/) _(especially with a MyISAM table)_, [PostgreSQL](http://www.postgresql.org), [Apache Derby](http://db.apache.org/derby/) _(as a Network Server)_, [SQL Server](http://www.microsoft.com/sql), [Sybase](http://www.sybase.com/products/databasemanagement), [Firebird](http://firebirdsql.org), etc. all work well for logging.

[Microsoft Access](http://office.microsoft.com/en-us/access) works surprisingly well for modest-volume Apache servers on Windows using the ODBC driver.

## Indexes ##

Database performance can be an issue when you use database logging.

It is often best to create a table with no primary key or indexes for capturing log entries.

You can move all the log records from this table into another indexed table periodically via an external process.

This will allow Apache to insert new log records quickly without the overhead of updating an index for every entry.

## Dates and Times in Databases ##
The default format for TIMESTAMP columns varies between databases.  The log format code `%{%Y-%m-%d %H:%M:%S}t` produces a string like '2007-10-29 14:22:13', which is acceptable as a TIMESTAMP or DATETIME field to most databases except Oracle.

The log format code `%{%Y-%m-%d}t` can likewise be used with most DATE columns and the format code `%{%H:%M:%S}t` with most TIME columns.

Use `%{%d-%b-%y %I.%M.%S %p}t` for Oracle TIMESTAMP columns, which look like '08-NOV-07 11.15.13.000000 AM'.

The time formats for `%{...}t` vary by platform.  The specific codes available are documented
|for Linux at|       http://www.gnu.org/software/libc/manual/html_node/Formatting-Calendar-Time.html#index-strftime-2660|
|:-----------|:---------------------------------------------------------------------------------------------------------|
|for Windows at|     http://msdn2.microsoft.com/en-us/library/fe06s4ak(VS.80).aspx|
|for Solaris at|     http://docs.oracle.com/cd/E19253-01/816-5168/strftime-3c/index.html|
|for Netware at|     http://developer.novell.com/documentation/libc/libc_vol2/data/sdk1810.html#sdk1810|

## Recovery ##
If the database is not accessible, SQL statements are be written to **FILENAME** in a format suitable for re-entering the data when the database becomes available again.