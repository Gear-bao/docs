---
layout: default
description: These functions implementthe HTTP API for modifying views
---

# Manipulating the view

These functions implement
[the HTTP API for modifying views](../http/views-modifying.html).

## ArangoDatabase.createView

`ArangoDatabase.createView(String name, ViewType type) : ViewEntity`

Creates a view of the given _type_, then returns view information from the server.

**Arguments**

- **name**: `String`

  The name of the view

- **type**: `ViewType`

  The type of the view

**Examples**

```Java
ArangoDB arango = new ArangoDB.Builder().build();
ArangoDatabase db = arango.db("myDB");
db.createView("myView", ViewType.ARANGO_SEARCH);
// the view "potatoes" now exists
```

## ArangoView.rename

`ArangoView.rename(String newName) : ViewEntity`

Renames the view.

**Arguments**

- **newName**: `String`

  The new name

**Examples**

```Java
ArangoDB arango = new ArangoDB.Builder().build();
ArangoDatabase db = arango.db("myDB");
ArangoView view = db.view("some-view");

ViewEntity result = view.rename("new-view-name")
assertThat(result.getName(), is("new-view-name");
// result contains additional information about the view
```

## ArangoView.drop

`ArangoView.drop() : void`

Deletes the view from the database.

**Examples**

```Java
ArangoDB arango = new ArangoDB.Builder().build();
ArangoDatabase db = arango.db("myDB");
ArangoView view = db.view("some-view");

view.drop();
// the view "some-view" no longer exists
```
