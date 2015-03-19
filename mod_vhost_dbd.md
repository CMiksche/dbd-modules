# Introduction #
mod\_vhost\_dbd has one directive:

#### DBDocRoot    SQL    `[` PARAM ... `]` ####

mod\_vhost\_dbd overrides the [DocumentRoot](http://httpd.apache.org/docs/current/mod/core.html#documentroot) directory using an SQL query.

The query returns the new root directory as the first column of a single result row.  If no rows are returned by the query, the original [DocumentRoot](http://httpd.apache.org/docs/current/mod/core.html#documentroot) directory is not replaced.
```
DBDocRoot "SELECT WebDir FROM WebTable WHERE WebName = %s"  HOSTNAME
```
DBDocRoot can be used at the Server or the Virtual Host level.


# Details #
**SQL** is an SQL statement which returns no more than one row, with the directory in the first column.  This directory is used instead of the existing document root set by the [DocumentRoot](http://httpd.apache.org/docs/current/mod/core.html#documentroot) directive.

**PARAM ...** arguments are inserted into the SQL statement at request time.  Use one **PARAM** for each parameter marker in your SQL statement.  In the examples, **%s** is the parameter marker.  **PARAM** may be any of these words (space separated, not case sensitive):
| **HOSTNAME**|requested hostname| _may be NULL if there is no Host header. e.g. HTTP 1.0 or FTP requests_ |
|:------------|:-----------------|:------------------------------------------------------------------------|
| **IP**|server IP address| _as a string, never NULL_ |
| **PORT**|server port number| _as string, never NULL_ |
| **URI**|The request URI| _never NULL_ |
| **URI`n`** _where_ **`n`** _is 1-9_ |Pass only the first `n` segments of the URI to the query| _never NULL_ |

  * You must have an Apache [DBD driver](http://people.apache.org/~niq/dbd.html) and [mod\_dbd](http://httpd.apache.org/docs/current/mod/mod_dbd.html) loaded and configured to use mod\_vhost\_dbd.

  * Use [LoadModule](http://httpd.apache.org/docs/current/mod/mod_so.html#loadmodule) to enable mod\_vhost\_dbd.
```
LoadModule vhost_dbd_module modules/mod_vhost_dbd.so 
```

  * If your query returns no rows, the existing document root is used.  If your query returns more than one row, the request is rejected with a _500 Internal Server Error_.

  * Do not put a semicolon at the end of your SQL statement.

  * **PARAM** names can be repeated as necessary. For example: to only look up hosts containing ".acme." for port 80 requests:
```
        DBDocRoot "SELECT DocRoot FROM myTable WHERE %s LIKE '%%.acme.%%' AND Host = %s AND %s = 80" HOSTNAME HOSTNAME PORT
```

  * **DBDocRoot** can use the name of a statement previously defined with the [DBDPrepareSQL](http://httpd.apache.org/docs/current/mod/mod_dbd.html#dbdpreparesql) directive as the **SQL** parameter. For example:
```
DBDPrepareSQL "SELECT WebDir FROM WebTable WHERE WebName = %s"  HostLookup
...
DBDocRoot HostLookup  HOSTNAME
```

  * Some databases (for example: Firebird) may report errors when statements for **DBDocRoot** are prepared with [DBDPrepareSQL](http://httpd.apache.org/docs/2.3/mod/mod_dbd.html#dbdpreparesql). In this case, [DBDPrepareSQL](http://httpd.apache.org/docs/current/mod/mod_dbd.html#dbdpreparesql) cannot be used for virtual host statements.

  * For Apache 2.2 prior to version 2.2.9, use parameter markers appropriate to your database.  For example: Use **?** for [ODBC](http://odbc-dbd.googlecode.com/) or SQLite.  Use **%s** for MySQL, etc.

  * For Apache 2.2.9+ always use **%s** as a parameter marker in your SQL statement.  If your SQL statement contains other **%** characters (for example, in a LIKE clause), use **%%** to prevent them from being misinterpreted as parameter markers.

  * For Apache 2.2.9+, any additional columns which are returned will set an [environment variable](http://httpd.apache.org/docs/current/env.html) with the same name as the column name. If the column value is NULL, any existing [environment variable](http://httpd.apache.org/docs/current/env.html) which matches the column name will be unset.

  * Using a URI **PARAM** increases the number of database queries required because each request requires a new query. Using only the leading portion of the URI can reduce the number of database queries required. When multiple requests are made on the same keep-alive connection which have the same URI`n` value, only one query is performed.

  * If the URI is /alpha/beta/gamma/delta/index.html:
```
   URI1 is /alpha
   URI2 is /alpha/beta
   URI3 is /alpha/beta/gamma
   ...
   URI5 is /alpha/beta/gamma/delta/index.html
   URI6 is /alpha/beta/gamma/delta/index.html
   ...
   URI9 is /alpha/beta/gamma/delta/index.html
   URI  is /alpha/beta/gamma/delta/index.html
```


  * If [mod\_ftp](http://httpd.apache.org/mod_ftp/) is used, an additional **PARAM** name is available.  **FTPUSER** can be specified to pass the logged-in FTP user name.  This parameter is not available for HTTP requests.

  * Setting **[LogLevel](http://httpd.apache.org/docs/current/mod/core.html#loglevel) debug** will print additional information in the Apache error log which can help to diagnose SQL query problems.

  * The httpd server variable: **document\_root** is not changed by mod\_vhost\_dbd.  It remains as the value set by the **[DocumentRoot ](http://httpd.apache.org/docs/current/mod/core.html#documentroot)** directive in the httpd configuration file.  For example, these variables will contain the directory path set in the httpd configuration file - not the directory path set by **DBDocRoot**:
|PHP | `$_SERVER['DOCUMENT_ROOT']` |
|:---|:----------------------------|
|CGI | environment variable: `DOCUMENT_ROOT` |
|J2EE | `GetServletContext().GetRealPath("/")` |