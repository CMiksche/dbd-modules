## _Several Apache modules to allow Apache 2.2+ to access databases using DBD._ ##

These modules work on Windows or Unix with any of the DBD drivers included in the Apache 2 [DBD Framework](http://people.apache.org/~niq/dbd.html).

See the [Building Wiki](http://code.google.com/p/dbd-modules/wiki/Building) for instructions to build these modules for Unix or Windows.

Windows users can get binaries built with Visual Studio 2008 (a.k.a. VC9) from the [Apache Lounge](http://www.apachelounge.com/download/).


---


## mod\_log\_dbd ##
#### Log web requests to an SQL database ####
```
LoadModule log_dbd_module modules/mod_log_dbd.so
...
CustomLog   logs/access.sql     "%h, %l, %u, %{%Y-%m-%d %H:%M:%S}t, %r, %>s, %b"
DBDLog      logs/access.sql    "INSERT INTO LogTable (Host, Rname, User, Tstmp, Request, Status, Bytes) \
                                VALUES (%s, %s, %s, %s, %s, %s, %s)"
```
See the **mod\_log\_dbd [Wiki](mod_log_dbd.md)** for details.


---


## mod\_vhost\_dbd ##
#### Override the document root directory from an SQL database ####
```
LoadModule vhost_dbd_module modules/mod_vhost_dbd.so
...
DBDocRoot "SELECT DocRoot FROM WebTable WHERE Host = %s"  HOSTNAME
```
See the **mod\_vhost\_dbd [Wiki](mod_vhost_dbd.md)** for details.


---


### Release History ###
  * **Version 1.0.6 February 19, 2012** - mod\_log\_dbd prints error message instead of crashing if mod\_log\_config is not loaded.  mod\_vhost\_dbd always executes before mod\_rewrite if mod\_rewrite is loaded.
  * **Version 1.0.5 May 17, 2009** - fix mod\_vhost\_dbd [issue #2](https://code.google.com/p/dbd-modules/issues/detail?id=#2) - Some DBD drivers depend on output-argument initial values; set the result pointer to NULL before calling apr\_dbd\_pselect, and set the row pointer to NULL before the first call to apr\_dbd\_get\_row.
  * **Version 1.0.4 August 2, 2008** - mod\_vhost\_dbd retains environment vars correctly on APR 1.3+.
  * **Version 1.0.3 January 1, 2008** - mod\_log\_dbd better fallback file when DBDPrepareSQL is used.
  * **Version 1.0.2 November 26, 2007** - mod\_log\_dbd fallback file initialized correctly