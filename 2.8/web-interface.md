---
layout: default
description: ArangoDB comes with a built-in web interface for administration
---
Accessing the Web Interface
===========================

ArangoDB comes with a built-in web interface for administration. The web 
interface can be accessed via the URL

    http://localhost:8529

assuming you are using the standard port and no user routings. If you
have any service installed, the home page might point to that
service instead. In this case use

  http://localhost:8529/_admin/aardvark/index.html

(note: _aardvark_ is the web interface's internal name).

If no database name is specified in the URL, you will in most cases get
routed to the web interface for the *_system* database. To access the web 
interface for any other ArangoDB database, put the database name into the
request URI path as follows:
  
  http://localhost:8529/_db/mydb/

The above will load the web interface for the database *mydb*.

To restrict access to the web interface, use 
[ArangoDB's authentication feature](general-http.html#authentication).

Select Functionality provided by the Web Interface
--------------------------------------------------

The following sections provide a very brief overview of some features offered
in the web interface. Please note that this is not a complete list of features.

Dashboard Tab
-------------

The *Dashboard* tab provides statistics which are polled regularly from the
ArangoDB server.

Collections Tab
---------------

The *Collections* tab shows an overview of the loaded and unloaded
collections present in ArangoDB. System collections (i.e. collections
whose names start with an underscore) are not shown by default.

The list of collections can be restricted using the search bar or by
using the filtering at the top. The filter can also be used to show or
hide system collections.

Clicking on a collection will show the documents contained in it. 
Clicking the small icon on a collection's badge will bring up a dialog
that allows loading/unloading, renaming and deleting the collection.

Please note that you should not change or delete system collections.

In the list of documents of a collection, you can click on the *Add document*
line to add a new document to the collection. The document will be created
instantly, with a system-defined key. The key and all other attributes of the
document can be adjusted in the following view.

Services Tab
------------

The *Services* tab provides a list of installed Foxx services ( also called applications). The view
is divided into lists of installed and services that are available for
installation.

Please note that ArangoDB's web interface (_aardvark_) is a Foxx service 
itself. Please also note that installed services will be listed in both
the *installed* and the *available* section. This is intentional because each
service can be installed multiple times using different mount points.

Graphs Tab
----------

The *Graphs* tab provides a viewer facility for graph data stored in ArangoDB. It
allows browsing ArangoDB graphs stored in the *_graphs* system collection or a
graph consisting of an arbitrary vertex and [edge collection](glossary.html#edge-collection). 

Please note that the graph viewer requires client-side SVG and that you need a
browser capable of rendering that. Especially Internet Explorer browsers older
than version 9 are likely to not support this. 

AQL Editor Tab
--------------

The *AQL Editor* tab allows to execute ad-hoc AQL queries.

Type in a query in the bottom box and execute it by pressing the *Submit* button.
The query result will be shown in the box at the top.
The editor provides a few example queries that can be used as templates.

There is also the option to add own frequently used queries here. Note that own 
queries will be stored in the browser's local storage and the web interface has
no control over when the browser's local storage is cleared.

Tools Tab
---------

The Tools tab contains a JavaScript shell that can be used to run commands on
the ArangoDB server, a log viewer and a link to the description of ArangoDB's 
REST API.

The *JS Shell* menu item provides access to a JavaScript shell that connects to the
database server.

Any valid JavaScript code can be executed inside the shell. The code will be
executed inside your browser. To contact the ArangoDB server you can use the
*db* object, for example as follows:

```js
    JSH> db._create("mycollection");
    JSH> db.mycollection.save({ _key: "test", value: "something" });
```

You can use the *Logs* menu item allows browsing the most recent log entries provided by the
ArangoDB database server.

Note that the server only keeps a limited number of log entries. For
real log analyses write the logs to disk using syslog or a similar
mechanism. ArangoDB provides several startup options for this.

The *Logs* menu item will only be shown for the `_system` database, and is disabled for
any other databases.

The *API* menu item provides an overview of ArangoDB's built-in HTTP REST API, with
documentation and examples. It should be consulted when there is doubt about API
URLs, parameters etc.
