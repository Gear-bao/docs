---
layout: default
description: AQL supports the following functions to operate on document values
---
Document functions
==================

AQL supports the following functions to operate on document values:

- *MATCHES(document, examples, return-index)*: Compares the document
  *document* against each example document provided in the array *examples*. 
  If *document* matches one of the examples, *true* is returned, and if there is
  no match *false* will be returned. The default return value type can be changed by
  passing *true* as the third function parameter *return-index*. Setting this
  flag will return the index of the example that matched (starting at offset 0), or 
  *-1* if there was no match.

  The comparisons will be started with the first example. All attributes of the example
  will be compared against the attributes of *document*. If all attributes match, the 
  comparison stops and the result is returned. If there is a mismatch, the function will
  continue the comparison with the next example until there are no more examples left.

  The *examples* must be an array of 1..n example documents, with any number of attributes
  each. Note: specifying an empty array of examples is not allowed.
   
  **Examples**

      MATCHES(
        { "test" : 1 }, [ 
          { "test" : 1, "foo" : "bar" }, 
          { "foo" : 1 }, 
          { "test : 1 } 
        ], true)

  This will return *2*, because the third example matches, and because the 
  *return-index* flag is set to *true*.

- *MERGE(document1, document2, ... documentN)*: Merges the documents
  in *document1* to *documentN* into a single document. If document attribute
  keys are ambiguous, the merged result will contain the values of the documents 
  contained later in the argument list.

  For example, two documents with distinct attribute names can easily be merged into one: 

      /* { "user1" : { "name" : "J" }, "user2" : { "name" : "T" } } */
      MERGE(
        { "user1" : { "name" : "J" } }, 
        { "user2" : { "name" : "T" } }
      )

  When merging documents with identical attribute names, the attribute values of the
  latter documents will be used in the end result:

      /* { "users" : { "name" : "T" } } */
      MERGE(
        { "users" : { "name" : "J" } }, 
        { "users" : { "name" : "T" } }
      )

  *MERGE* works with a single array parameter, too. This variant allows combining the 
  attributes of multiple objects from the array into a single object, e.g.

      RETURN MERGE([ 
        { foo: 'bar' }, 
        { quux: 'quetzalcoatl', ruled: true }, 
        { bar: 'baz', foo: 'done' }
      ])

  This will now return:

      {
        "foo": "done",
        "quux": "quetzalcoatl",
        "ruled": true,
        "bar": "baz"
      }

  Please note that merging will only be done for top-level attributes. If you wish to
  merge sub-attributes, you should consider using *MERGE_RECURSIVE* instead.

- *MERGE_RECURSIVE(document1, document2, ... documentN)*: Recursively
  merges the documents in *document1* to *documentN* into a single document. If
  document attribute keys are ambiguous, the merged result will contain the values of the 
  documents contained later in the argument list.

  For example, two documents with distinct attribute names can easily be merged into one: 
      
      /* { "user-1" : { "name" : "J", "livesIn" : { "city" : "LA", "state" : "CA" }, "age" : 42 } } */
      MERGE_RECURSIVE(
        { "user-1" : { "name" : "J", "livesIn" : { "city" : "LA" } } }, 
        { "user-1" : { "age" : 42, "livesIn" : { "state" : "CA" } } }
      )

  *MERGE_RECURSIVE* does not support the single array parameter variant that *MERGE* offers.


- *TRANSLATE(value, lookup, defaultValue)*: Looks up the value *value* in the *lookup*
  document. If *value* is a key in *lookup*, then *value* will be replaced with the
  lookup value found. If *value* is not present in *lookup*, then *defaultValue* will
  be returned if specified. If no *defaultValue* is specified, *value* will be returned:

      /* "France" */
      TRANSLATE("FR", { US: "United States", UK: "United Kingdom", FR: "France" })

      /* "not found!" */
      TRANSLATE(42, { foo: "bar", bar: "baz" }, "not found!")

- *HAS(document, attributename)*: Returns *true* if *document* has an
  attribute named *attributename*, and *false* otherwise.

- *ATTRIBUTES(document, removeInternal, sort)*: Returns the attribute
  names of the *document* as an array. 
  If *removeInternal* is set to *true*, then all internal attributes (such as *_id*, 
  *_key* etc.) are removed from the result. If *sort* is set to *true*, then the
  attribute names in the result will be sorted. Otherwise they will be returned in any order.

- *VALUES(document, removeInternal)*: Returns the attribute values of the *document*
  as an array. If *removeInternal* is set to *true*, then all internal attributes (such 
  as *_id*, *_key* etc.) are removed from the result. 

- *ZIP(attributes, values)*: Returns a document object assembled from the
  separate parameters *attributes* and *values*. *attributes* and *values* must be
  arrays and must have the same length. The items in *attributes* will be used for
  naming the attributes in the result. The items in *values* will be used as the
  actual values of the result.

      /* { "name" : "some user", "active" : true, "hobbies" : [ "swimming", "riding" ] } */
      ZIP([ 'name', 'active', 'hobbies' ], [ 'some user', true, [ 'swimming', 'riding' ] ])

- *UNSET(document, attributename, ...)*: Removes the attributes *attributename*
  (can be one or many) from *document*. All other attributes will be preserved.
  Multiple attribute names can be specified by either passing multiple individual string argument 
  names, or by passing an array of attribute names:

      UNSET(doc, '_id', '_key', 'foo', 'bar')
      UNSET(doc, [ '_id', '_key', 'foo', 'bar' ])

- *UNSET_RECURSIVE(document, attributename, ...)*: Recursively removes the attributes 
  *attributename* (can be one or many) from *document* and its sub-documents. All other 
  attributes will be preserved.
  Multiple attribute names can be specified by either passing multiple individual string argument 
  names, or by passing an array of attribute names:

      UNSET_RECURSIVE(doc, '_id', '_key', 'foo', 'bar')
      UNSET_RECURSIVE(doc, [ '_id', '_key', 'foo', 'bar' ])

- *KEEP(document, attributename, ...)*: Keeps only the attributes *attributename*
  (can be one or many) from *document*. All other attributes will be removed from the result.
  Multiple attribute names can be specified by either passing multiple individual string argument 
  names, or by passing an array of attribute names:

      KEEP(doc, 'firstname', 'name', 'likes')
      KEEP(doc, [ 'firstname', 'name', 'likes' ])

- *PARSE_IDENTIFIER(document-handle)*: Parses the [document handle](glossary.html#document-handle) specified in 
  *document-handle* and returns a the handle's individual parts a separate attributes.
  This function can be used to easily determine the [collection name](glossary.html#collection-name) and key from a given document.
  The *document-handle* can either be a regular document from a collection, or a document
  identifier string (e.g. *_users/1234*). Passing either a non-string or a non-document or a
  document without an *_id* attribute will result in an error.

      /* { "collection" : "_users", "key" : "my-user" } */ 
      PARSE_IDENTIFIER('_users/my-user')

      /* { "collection" : "mycollection", "key" : "mykey" } */ 
      PARSE_IDENTIFIER({ "_id" : "mycollection/mykey", "value" : "some value" })

- *IS_SAME_COLLECTION(collection, document)*: Return true if *document* has the same
  collection id as the collection specified in *collection*. *document* can either be
  a [document handle](glossary.html#document-handle) string, or a document with
  an *_id* attribute. The function does not validate whether the collection actually
  contains the specified document, but only compares the name of the specified collection
  with the collection name part of the specified document.
  If *document* is neither an object with an *id* attribute nor a *string* value,
  the function will return *null* and raise a warning.

      /* true */
      IS_SAME_COLLECTION('_users', '_users/my-user')
      IS_SAME_COLLECTION('_users', { _id: '_users/my-user' })

      /* false */ 
      IS_SAME_COLLECTION('_users', 'foobar/baz')
      IS_SAME_COLLECTION('_users', { _id: 'something/else' })

