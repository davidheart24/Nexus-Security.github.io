---
title: 'PgFincore – A set of functions to manage pages in memory from PostgreSQL'
date: 2019-12-03T06:05:00+01:00
draft: false
---

![](https://avatars0.githubusercontent.com/u/57241?s=400&v=4 "GitHub - klando/pgfincore: Mirror of http://git.postgresql.org/gitweb/?p=pgfincore.git;a=summary")  

[](https://github.com/klando/pgfincore#pgfincore)PgFincore
==========================================================

* * *

[](https://github.com/klando/pgfincore#a-set-of-functions-to-manage-pages-in-memory-from-postgresql)A set of functions to manage pages in memory from PostgreSQL
----------------------------------------------------------------------------------------------------------------------------------------------------------------

A set of functions to handle low-level management of relations using mincore to explore cache memory.

[](https://github.com/klando/pgfincore#description)DESCRIPTION
--------------------------------------------------------------

With PostgreSQL, each Table or Index is splitted in segments of (usually) 1GB, and each segment is splitted in pages in memory then in blocks for the filesystem.

Those functions let you know which and how many disk block from a relation are in the page cache of the operating system. It can provide the result as a VarBit and can be stored in a table. Then using this table, it is possible to restore the page cache state for each block of the relation, even in another server, thanks to Streaming Replication.

Other functions are used to set a _POSIX\_FADVISE_ flag on the entire relation (each segment). The more usefull are probably _WILLNEED_ and _DONTNEED_ which push and pop blocks of each segments of a relation from page cache, respectively.

Each functions are call with at least a table name or an index name (or oid) as a parameter and walk each segment of the relation.

[](https://github.com/klando/pgfincore#download)DOWNLOAD
--------------------------------------------------------

You can grab the latest code with git:

```
git clone git://git.postgresql.org/git/pgfincore.git or git://github.com/klando/pgfincore.git 
```

And the project is on pgfoundry : [http://pgfoundry.org/projects/pgfincore](http://pgfoundry.org/projects/pgfincore)

[](https://github.com/klando/pgfincore#install)INSTALL
------------------------------------------------------

From source code:

```
make clean make su make install 
```

For PostgreSQL >= 9.1, log in your database and:

```
mydb=# CREATE EXTENSION pgfincore; 
```

For other release, create the functions from the sql script (it should be in your contrib directory):

```
psql mydb -f pgfincore.sql 
```

PgFincore is also shipped with Debian scripts to build your own package:

```
aptitude install debhelper postgresql-server-dev-all postgresql-server-dev-9.1 # or postgresql-server-dev-8.4|postgresql-server-dev-9.0 make deb dpkg -i ../postgresql-9.1-pgfincore_1.1.1-1_amd64.deb 
```

PgFincore is packaged for _RPM_ at [http://yum.postgresql.org/](http://yum.postgresql.org/) PgFincore is packaged for _debian_ at [http://pgapt.debian.net/](http://pgapt.debian.net/)

[](https://github.com/klando/pgfincore#examples)EXAMPLES
--------------------------------------------------------

Here are some examples of usage. If you want more details go to Documentation\_

### [](https://github.com/klando/pgfincore#get-current-state-of-a-relation)Get current state of a relation

May be useful:

```
cedric=# select * from pgfincore('pgbench_accounts'); relpath | segment | os_page_size | rel_os_pages | pages_mem | group_mem | os_pages_free | databit | pages_dirty | group_dirty --------------------+---------+--------------+--------------+-----------+-----------+---------------+---------+-------------+------------- base/11874/16447 | 0 | 4096 | 262144 | 262144 | 1 | 81016 | | 0 | 0 base/11874/16447.1 | 1 | 4096 | 65726 | 65726 | 1 | 81016 | | 0 | 0 (2 rows) Time: 31.563 ms 
```

### [](https://github.com/klando/pgfincore#load-a-table-or-an-index-in-os-page-buffer)Load a table or an index in OS Page Buffer

You may want to try to keep a table or an index into the OS Page Cache, or preload a table before your well know big query is executed (reducing the query time).

To do so, just execute the following query:

```
cedric=# select * from pgfadvise_willneed('pgbench_accounts'); relpath | os_page_size | rel_os_pages | os_pages_free --------------------+--------------+--------------+--------------- base/11874/16447 | 4096 | 262144 | 169138 base/11874/16447.1 | 4096 | 65726 | 103352 (2 rows) Time: 4462,936 ms 
```

*   The column _os\_page\_size_ report that page size is 4KB.
*   The column _rel\_os\_pages_ is the number of pages of the specified file.
*   The column _os\_pages\_free_ is the number of free pages in memory (for caching).

### [](https://github.com/klando/pgfincore#snapshot-and-restore-the-os-page-buffer-state-of-a-table-or-an-index-or-more)Snapshot and Restore the OS Page Buffer state of a table or an index (or more)

You may want to restore a table or an index into the OS Page Cache as it was while you did the snapshot. For example if you have to reboot your server, then when PostgreSQL start up the first queries might be slower because neither PostgreSQL or the OS have pages in their respective cache about the relations involved in those first queries.

Executing a snapshot and a restore is very simple:

```
-- Snapshot cedric=# create table pgfincore_snapshot as cedric-# select 'pgbench_accounts'::text as relname,*,now() as date_snapshot cedric-# from pgfincore('pgbench_accounts',true); -- Restore cedric=# select * from pgfadvise_loader('pgbench_accounts', 0, true, true, (select databit from pgfincore_snapshot where relname='pgbench_accounts' and segment = 0)); relpath | os_page_size | os_pages_free | pages_loaded | pages_unloaded ------------------+--------------+---------------+--------------+---------------- base/11874/16447 | 4096 | 80867 | 262144 | 0 (1 row) Time: 35.349 ms 
```

*   The column _pages\_loaded_ report how many pages have been read to memory (they may have already been in memoy)
*   The column _pages\_unloaded_ report how many pages have been removed from memory (they may not have already been in memoy);

[](https://github.com/klando/pgfincore#synopsis)SYNOPSIS
--------------------------------------------------------

pgsysconf(OUT os\_page\_size bigint, OUT os\_pages\_free bigint, OUT os\_total\_pages bigint) RETURNS record

pgsysconf\_pretty(OUT os\_page\_size text, OUT os\_pages\_free text, OUT os\_total\_pages text) RETURNS record

pgfadvise(IN relname regclass, IN fork text, IN action int, OUT relpath text, OUT os\_page\_size bigint, OUT rel\_os\_pages bigint, OUT os\_pages\_free bigint) RETURNS setof record

pgfadvise\_willneed(IN relname regclass, OUT relpath text, OUT os\_page\_size bigint, OUT rel\_os\_pages bigint, OUT os\_pages\_free bigint) RETURNS setof record

pgfadvise\_dontneed(IN relname regclass, OUT relpath text, OUT os\_page\_size bigint, OUT rel\_os\_pages bigint, OUT os\_pages\_free bigint) RETURNS setof record

pgfadvise\_normal(IN relname regclass, OUT relpath text, OUT os\_page\_size bigint, OUT rel\_os\_pages bigint, OUT os\_pages\_free bigint) RETURNS setof record

pgfadvise\_sequential(IN relname regclass, OUT relpath text, OUT os\_page\_size bigint, OUT rel\_os\_pages bigint, OUT os\_pages\_free bigint) RETURNS setof record

pgfadvise\_random(IN relname regclass, OUT relpath text, OUT os\_page\_size bigint, OUT rel\_os\_pages bigint, OUT os\_pages\_free bigint) RETURNS setof record

pgfadvise\_loader(IN relname regclass, IN fork text, IN segment int, IN load bool, IN unload bool, IN databit varbit, OUT relpath text, OUT os\_page\_size bigint, OUT os\_pages\_free bigint, OUT pages\_loaded bigint, OUT pages\_unloaded bigint) RETURNS setof record

pgfadvise\_loader(IN relname regclass, IN segment int, IN load bool, IN unload bool, IN databit varbit, OUT relpath text, OUT os\_page\_size bigint, OUT os\_pages\_free bigint, OUT pages\_loaded bigint, OUT pages\_unloaded bigint) RETURNS setof record

pgfincore(IN relname regclass, IN fork text, IN getdatabit bool, OUT relpath text, OUT segment int, OUT os\_page\_size bigint, OUT rel\_os\_pages bigint, OUT pages\_mem bigint, OUT group\_mem bigint, OUT os\_pages\_free bigint, OUT databit varbit, OUT pages\_dirty bigint, OUT group\_dirty bigint) RETURNS setof record

pgfincore(IN relname regclass, IN getdatabit bool, OUT relpath text, OUT segment int, OUT os\_page\_size bigint, OUT rel\_os\_pages bigint, OUT pages\_mem bigint, OUT group\_mem bigint, OUT os\_pages\_free bigint, OUT databit varbit, OUT pages\_dirty bigint, OUT group\_dirty bigint) RETURNS setof record

pgfincore(IN relname regclass, OUT relpath text, OUT segment int, OUT os\_page\_size bigint, OUT rel\_os\_pages bigint, OUT pages\_mem bigint, OUT group\_mem bigint, OUT os\_pages\_free bigint, OUT databit varbit, OUT pages\_dirty bigint, OUT group\_dirty bigint) RETURNS setof record

[](https://github.com/klando/pgfincore#documentation)DOCUMENTATION
------------------------------------------------------------------

### [](https://github.com/klando/pgfincore#pgsysconf)pgsysconf

This function output size of OS blocks, number of free page in the OS Page Buffer.

```
cedric=# select * from pgsysconf(); os_page_size | os_pages_free | os_total_pages --------------+---------------+---------------- 4096 | 80431 | 4094174 
```

### [](https://github.com/klando/pgfincore#pgsysconf_pretty)pgsysconf\_pretty

The same as above, but with pretty output.

```
cedric=# select * from pgsysconf_pretty(); os_page_size | os_pages_free | os_total_pages --------------+---------------+---------------- 4096 bytes | 314 MB | 16 GB 
```

### [](https://github.com/klando/pgfincore#pgfadvise_willneed)pgfadvise\_WILLNEED

This function set _WILLNEED_ flag on the current relation. It means that the Operating Sytem will try to load as much pages as possible of the relation. Main idea is to preload files on server startup, perhaps using cache hit/miss ratio or most required relations/indexes.

```
cedric=# select * from pgfadvise_willneed('pgbench_accounts'); relpath | os_page_size | rel_os_pages | os_pages_free --------------------+--------------+--------------+--------------- base/11874/16447 | 4096 | 262144 | 80650 base/11874/16447.1 | 4096 | 65726 | 80650 
```

### [](https://github.com/klando/pgfincore#pgfadvise_dontneed)pgfadvise\_DONTNEED

This function set _DONTNEED_ flag on the current relation. It means that the Operating System will first unload pages of the file if it need to free some memory. Main idea is to unload files when they are not usefull anymore (instead of perhaps more interesting pages)

```
cedric=# select * from pgfadvise_dontneed('pgbench_accounts'); relpath | os_page_size | rel_os_pages | os_pages_free --------------------+--------------+--------------+--------------- base/11874/16447 | 4096 | 262144 | 342071 base/11874/16447.1 | 4096 | 65726 | 408103 
```

### [](https://github.com/klando/pgfincore#pgfadvise_normal)pgfadvise\_NORMAL

This function set _NORMAL_ flag on the current relation.

### [](https://github.com/klando/pgfincore#pgfadvise_sequential)pgfadvise\_SEQUENTIAL

This function set _SEQUENTIAL_ flag on the current relation.

### [](https://github.com/klando/pgfincore#pgfadvise_random)pgfadvise\_RANDOM

This function set _RANDOM_ flag on the current relation.

### [](https://github.com/klando/pgfincore#pgfadvise_loader)pgfadvise\_loader

This function allow to interact directly with the Page Cache. It can be used to load and/or unload page from memory based on a varbit representing the map of the pages to load/unload accordingly.

Work with relation pgbench\_accounts, segment 0, arbitrary varbit map:

```
-- Loading and Unloading cedric=# select * from pgfadvise_loader('pgbench_accounts', 0, true, true, B'111000'); relpath | os_page_size | os_pages_free | pages_loaded | pages_unloaded ------------------+--------------+---------------+--------------+---------------- base/11874/16447 | 4096 | 408376 | 3 | 3 -- Loading cedric=# select * from pgfadvise_loader('pgbench_accounts', 0, true, false, B'111000'); relpath | os_page_size | os_pages_free | pages_loaded | pages_unloaded ------------------+--------------+---------------+--------------+---------------- base/11874/16447 | 4096 | 408370 | 3 | 0 -- Unloading cedric=# select * from pgfadvise_loader('pgbench_accounts', 0, false, true, B'111000'); relpath | os_page_size | os_pages_free | pages_loaded | pages_unloaded ------------------+--------------+---------------+--------------+---------------- base/11874/16447 | 4096 | 408370 | 0 | 3 
```

### [](https://github.com/klando/pgfincore#pgfincore-1)pgfincore

This function provide information about the file system cache (page cache).

```
cedric=# select * from pgfincore('pgbench_accounts'); relpath | segment | os_page_size | rel_os_pages | pages_mem | group_mem | os_pages_free | databit | pages_dirty | group_dirty --------------------+---------+--------------+--------------+-----------+-----------+---------------+---------+-------------+------------- base/11874/16447 | 0 | 4096 | 262144 | 3 | 1 | 408444 | | 0 | 0 base/11874/16447.1 | 1 | 4096 | 65726 | 0 | 0 | 408444 | | 0 | 0 
```

For the specified relation it returns:

*   relpath : the relation path
*   segment : the segment number analyzed
*   os\_page\_size : the size of one page
*   rel\_os\_pages : the total number of pages of the relation
*   pages\_mem : the total number of relation's pages in page cache. (not the shared buffers from PostgreSQL but the OS cache)
*   group\_mem : the number of groups of adjacent pages\_mem
*   os\_page\_free : the number of free page in the OS page cache
*   databit : the varbit map of the file, because of its size it is useless to output Use pgfincore('pgbench\_accounts',true) to activate it.
*   pages\_dirty : if HAVE\_FINCORE constant is define and the platorm provides the relevant information, like pages\_mem but for dirtied pages
*   group\_dirty : if HAVE\_FINCORE constant is define and the platorm provides the relevant information, like group\_mem but for dirtied pages

[](https://github.com/klando/pgfincore#debug)DEBUG
--------------------------------------------------

You can debug the PgFincore with the following error level: _DEBUG1_ and _DEBUG5_.

For example:

```
set client_min_messages TO debug1; -- debug5 is only usefull to trace each block 
```

[](https://github.com/klando/pgfincore#requirements)REQUIREMENTS
----------------------------------------------------------------

*   PgFincore needs mincore() or fincore() and POSIX\_FADVISE

[](https://github.com/klando/pgfincore#limitations)LIMITATIONS
--------------------------------------------------------------

*   PgFincore has a limited mode when POSIX\_FADVISE is not provided by the platform.
    
*   PgFincore needs PostgreSQL >= 8.3
    
*   PgFincore does not work on windows.
    

[](https://github.com/klando/pgfincore#see-also)SEE ALSO
--------------------------------------------------------

2ndQuadrant, PostgreSQL Expertise, developement, training and 24x7 support:

[http://2ndQuadrant.com](http://2ndQuadrant.com)

  
  
from Hacker News https://ift.tt/R0zaHV