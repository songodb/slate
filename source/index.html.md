---
title: SongoDB API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript

toc_footers:
  - <a href='https://songodb.io'>Homepage (songodb.io)</a> 
  - <a href='https://github.com/songodb'>Source Code (github.com/songodb)</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

The SongoDB API is an HTTP REST translation of a small subset the MongoDB client. It accepts and returns data in JSON-encoded format. 

```javascript
const SONGODB_BASEURL = "https://api.songodb.io/v1";
```

All URLs in this documentation will be shown relative the to the **Base URL**:

`https://api.songodb.io/v1`



<aside class="notice">
  SongoDB is in <b>ALPHA</b> and in active development.
  Unannounced breaking changes should be expected.
</aside>

# Instance

An instance functions as top-level namespace that can host multiple databases. 
A single SongoDB API endpoint can serve multiple instances.

## connect

Connect to the specific instance by the SongoDB base url with a specific instanceId.

```javascript
// Import the module
const { SongoDBClient } = require('@songodb/songodb-client');

// Construct the SongoDB connection URL
// https://api.songodb.io/v1/my-unique-instance-id
const instanceId = 'my-unique-instance-id';
let url = `${SONGODB_BASEURL}/${instanceId}` 

// Connect to SongoDB instance using the URL
let instance = await SongoDBClient.connect(`${SONGODB_BASEURL}/${instanceId}`);
```

| Name | Type | Description |
| ---- | ---- | ----------- |
| **url** | string | The base url followed by the unique ID of the instance: `https://{SONGODB_BASEURL}/{instanceId}`|

<aside class="warning" style="color:#F0F0F0;">
  There is currently <b>no authentication or authorization</b> for SongoDB.
  Anyone with the ID of the instance will be able to connect and perform all operations.
</aside>

### Returns

| Name | Type | Description |
| ---- | ---- | ----------- |
| **url** | string | The base url followed by the unique ID of the instance: `https://{SONGODB_BASEURL}/{instanceId}`|


## db

Access a specific database within the instance by name.  

```javascript
// A database for a retail store application named "store"
let db = instance.db('store')
```

| Name | Type | Description |
| ---- | ---- | ----------- |
| **name** | string | The name of the database. Must be unique within the instance. |


## listDatabases

List the available databases from a specific instance.

```javascript

// Get all the databases hosted on this instance
let result = await instance.listDatabases()

// result
{
  docs: [ 
    { name: "store" }, 
    { name: "analytics" }, 
    { name: "auth" }, 
    ... 
  ]
}
```

### Returns

| Field | Type | Description |
| ---- | ---- | ----------- |
| **docs** | array | An array of documents, each one representing a collection in the database. If `nameOnly` is `true`, then an array of strings, where each string is the name of a collection.|
| **explain** | object | An [explain](#explain-results) document which contains metadata about the execution of the operation. |

### listDatabases()


# Database

A database can store multiple collections of documents.

The name of a database must satisfy the following criteria:

* Must begin with a letter or underscore.
* Can only contain URL-friendly characters.
* It cannot start with "admin" (this is a reserved database).
* It must be 60 characters or less.

## collection

Access a specific collection within the database by name. 

```javascript
// Get a reference to the "inventory" collection
let inventory = instance.db("store").collection("inventory")
```

### collection(name)

| Argument | Type | Description |
| ---- | ---- | ----------- |
| **name** | string | The name of the collection. |

### Returns

A reference to the specific collection. If a collection with that name does not yet exist, it will be created when a document is first inserted. 

## listCollections

Return the collections contained within a specific database.

```javascript
// Return all the collections for the "store" database
let result = await instance.db("store").listCollections()

// result
{
  "docs": [
    { "name": "customers", ... },
    { "name": "employees", ... },
    { "name": "inventory", ... }
  ],
  "explain": { ... }
}

// Return just the names of the collections 
result = await instance.db("store").listCollections({ nameOnly: true })

// result
{
  "docs": [ "customers", "employees", "inventory" ],
  "explain": { ... }
}
```
### listCollections(options)

**options**

| Field | Type | Description |
| ---- | ---- | ----------- |
| **nameOnly** | boolean | Only the names of each collection.|

### Returns

| Field | Type | Description |
| ---- | ---- | ----------- |
| **docs** | array | An array of documents, each one representing a collection in the database. If `nameOnly` is `true`, then an array of strings, where each string is the name of a collection.|
| **explain** | object | An [explain](#explain-results) document which contains metadata about the execution of the operation. |

## dropDatabase

Delete the entire database including all the collections and documents it stores.

```javascript
// Drop the entire "store" database including all its collections and documents
let result = await instance.db("store").dropDatabase()

// result
{
  "deletedCount": 9,
  "dropped": true,
  "explain": { ... }
}
```
### dropDatabase()

### Returns

| Field | Type | Description |
| ---- | -- | ----------- |
| **deletedCount** | string | The number of documents deleted from the database. |
| **dropped** | boolean | `true` if the database was dropped successfully. |
| **explain** | object | An [explain](#explain-results) document which contains metadata about the execution of the operation. |

# Collection

A collection is a group of related documents identified by a name. 

```javascript
// Get the 'inventory' collection object from the 'store' database:

let inventory = instance.db('store').collection('inventory');
```

The name of a collection must satisfy the following criteria:

* Must begin with a letter or underscore.
* Can only contain URL-friendly characters.
* It cannot start with "system" (this is a reserved collection).
* It must be 60 characters or less.

<aside class="notice">
  The naming criteria for a collection in SongoDB is more restrictive than MongoDB.
  Therefore, following SongoDB naming criteria will guarantee compability with MongoDB.
</aside>

## insertOne

Add a single document to the collection. 


```javascript
// Insert a document in the "inventory" collection

let result = await inventory.insertOne({ 
  item: "canvas",
  qty: 100, 
  tags: [ "cotton" ],
  size: { 
    h: 28, 
    w: 35.5, 
    uom: "cm" 
  }
})

// result
{
  "insertedCount": 1,
  "ops": [
    {
      "item": "canvas",
      "qty": 100,
      "tags": [
        "cotton"
      ],
      "size": {
        "h": 28,
        "w": 35.5,
        "uom": "cm"
      },
      "_id": "c3a29f7a-1362-410f-ba6a-880fdf3eb570"
    }
  ],
  "insertedId": "c3a29f7a-1362-410f-ba6a-880fdf3eb570"
}
```

### insertOne(doc)

| Argument | Type | Description |
| ---- | ---- | ----------- |
| **doc** | object | The document to be inserted. If the document does not supply the `_id` field, then SongoDB add the `_id` field with a [v4 UUID](https://www.intl-spectrum.com/Article/r848/IS_UUID_V4_UUID_V4_Random_Generation). |

### Returns

| Field | Type | Description |
| ---- | ---- | ----------- |
| **insertedCount** | int | The number of documents inserted. |
| **ops** | array | An array of documents inserted. |
| **insertedId** | string | The `_id` of the document inserted. |


## insertMany

Add multiple documents to the collection.


```javascript
// Insert multiple documents in the "inventory" collection

let result = await inventory.insertMany([
  { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
  { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
  { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])

// result
{
  "insertedCount": 3,
  "ops": [
    { "item": "journal", ... },
    { "item": "mat", ... },
    { "item": "mousepad", ... }
  ],
  "insertedIds": [
    "206f5e98-cd4a-4ee5-b1a7-86e7740da387",
    "004d8446-782e-4c4b-b60e-ddf9094288c6",
    "8839df54-5105-4efb-b54f-73b3952afbc1"
  ]
}
```

### insertMany(docs) 

| Argument | Type | Description |
| ---- | ---- | ----------- |
| **docs** | array | An array of documents to be inserted. If a document does not supply the `_id` field, then SongoDB add the `_id` field with a [v4 UUID](https://www.intl-spectrum.com/Article/r848/IS_UUID_V4_UUID_V4_Random_Generation). |

### Returns 

| Field | Type | Description |
| ---- | ---- | ----------- |
| **insertedCount** | int | The number of documents inserted. |
| **ops** | array | An array of documents inserted. |
| **insertedIds** | array | Array of the `_id`s of the documents inserted. |


## find

Query and return all matching documents.

```javascript
// Find all items that have 50 or less quantity in stock
let result = await inventory.find({ qty: { "$lte": 50 } })

// result
{
  "docs": [
    {
      "_id": "3c008637-037d-49a8-9c85-24f1a734ec45",
      "item": "mousepad",
      "qty": 25,
      ...
    },
    {
      "_id": "b2d9266c-854c-4743-867a-9f5a093c175b",
      "item": "journal",
      "qty": 25,
      ...
    }
  ],
  "explain": { ... }
}
```

### find(query, options)

| Argument | Type | Description |
| ---- | -- | ----------- |
| **query** | object | A [MongoDB query document](https://docs.mongodb.com/manual/tutorial/query-documents/). Supports all operators implemented by the [Sift](https://github.com/crcn/sift.js#readme) NodeJS library.  |
| **options** | object | *Optional* |

**options**

| Field | Type | Description |
| ---- | -- | ----------- |
| **MaxKeys** | int | The total number of S3 keys for this operation to scan for a matching document. Default is `100`. |
| **ContinuationToken** | string | A obfuscated token which tells S3 to continue scanning for a matching document from where the last operation left off. |

### Returns

| Field | Type | Description |
| ---- | -- | ----------- |
| **docs** | array | An array of all matching documents. Returns an empty array `[]` if no matching documents were found. |
| **explain** | object | An [explain](#explain-results) document which contains metadata about the execution of the operation. |

## findOne

Query and return the first matching document.

```javascript
// Find the first item named "canvas"
let result = await inventory.findOne({ item: "canvas" })

// result
{
  "docs": [
    {
      "item": "canvas",
      "qty": 100,
      "tags": [
        "cotton"
      ],
      "size": {
        "h": 28,
        "w": 35.5,
        "uom": "cm"
      },
      "_id": "a499a0bf-6b65-480d-8ca2-1546e6f99425"
    }
  ],
  "explain": { ... }
}

```
### findOne(query, options)

| Argument | Type | Description |
| ---- | -- | ----------- |
| **query** | object | A [MongoDB query document](https://docs.mongodb.com/manual/tutorial/query-documents/). Supports all operators implemented by the [Sift](https://github.com/crcn/sift.js#readme) NodeJS library.  |
| **options** | object | *Optional* |

**options**

| Field | Type | Description |
| ---- | -- | ----------- |
| **MaxKeys** | int | The total number of S3 keys for this operation to scan for a matching document. Default is `100`. |
| **ContinuationToken** | string | A obfuscated token which tells S3 to continue scanning for a matching document from where the last operation left off. |

### Returns

| Field | Type | Description |
| ---- | -- | ----------- |
| **doc** | object | The first matching document or `null` if no matching document was found. |
| **explain** | object | An [explain](#explain-results) document which contains metadata about the execution of the operation. |

## updateMany

Query and update all matching documents.

```javascript
// Increase our inventory of the "journal" and "mousepad" items
let result = await inventory.updateMany(
  { item: { "$in": [ "journal", "mousepad" ] } },
  { "$inc":  100 })

// result
{
  "matchedCount": 2,
  "modifiedCount": 0,
  "upsertedCount": 0,
  "upsertedId": null,
  "explain": { ... }
}

```
### updateMany(filter, update, options)

| Argument | Type | Description |
| ---- | -- | ----------- |
| **filter** | object | All documents that satisfy the filter criteria will be updated. The same [MongoDB query document](https://docs.mongodb.com/manual/tutorial/query-documents/) format as the [find()](#find) operation. Supports all operators implemented by the [Sift](https://github.com/crcn/sift.js#readme) NodeJS library.  |
| **update** | object | The modifications to apply to the matching documents. Currently supports all field-level [MongoDB update operators](https://docs.mongodb.com/manual/reference/operator/update/#id1). |
| **options** | object | *Optional* |

**options**

| Field | Type | Description |
| ---- | -- | ----------- |
| **upsert** | boolean | When `true`, if there are no matching documents, then insert a new document with the update modifications applied |
| **MaxKeys** | int | The total number of S3 keys for this operation to scan for a matching document. Default is `100`. |
| **ContinuationToken** | string | A obfuscated token which tells S3 to continue scanning for a matching document from where the last operation left off. |

### Returns

| Field | Type | Description |
| ---- | -- | ----------- |
| **matchedCount** | int | The number of documents that matched the filter criteria. |
| **modifiedCount** | int | The number of documents that were modified from the update doc. |
| **upsertedCount** | int | `1` if the operation resulted in an upsert (no matching documents). `0` otherwise. |
| **upsertedId** | string | The `_id` of the upserted document. `null` otherwise. |
| **explain** | object | An [explain](#explain-results) document which contains metadata about the execution of the operation. |

## updateOne

Query and update the first matching document.

```javascript
// Increase our inventory of the "canvas" item
let result = await inventory.updateOne(
  { item: "canvas" }, 
  { "$inc": { qty: 25 } })

// result
{
  "matchedCount": 1,
  "modifiedCount": 1,
  "upsertedCount": 0,
  "upsertedId": null,
  "explain": { ... }
}
```

### updateMany(filter, update, options)

| Argument | Type | Description |
| ---- | -- | ----------- |
| **filter** | object | The first document that satisfies the filter criteria will be updated. The same [MongoDB query document](https://docs.mongodb.com/manual/tutorial/query-documents/) format as the [find()](#find) operation. Supports all operators implemented by the [Sift](https://github.com/crcn/sift.js#readme) NodeJS library.  |
| **update** | object | The modifications to apply to the first matching document. Currently supports all field-level [MongoDB update operators](https://docs.mongodb.com/manual/reference/operator/update/#id1). |
| **options** | object | *Optional* |

**options**

| Field | Type | Description |
| ---- | -- | ----------- |
| **upsert** | boolean | When `true`, if there are no matching documents, then insert a new document with the update modifications applied |
| **MaxKeys** | int | The total number of S3 keys for this operation to scan for a matching document. Default is `100`. |
| **ContinuationToken** | string | A obfuscated token which tells S3 to continue scanning for a matching document from where the last operation left off. |

### Returns

| Field | Type | Description |
| ---- | -- | ----------- |
| **matchedCount** | int | The number of documents that matched the filter criteria. |
| **modifiedCount** | int | The number of documents that were modified from the update doc. |
| **upsertedCount** | int | `1` if the operation resulted in an upsert (no matching documents). `0` otherwise. |
| **upsertedId** | string | The `_id` of the upserted document. `null` otherwise. |
| **explain** | object | An [explain](#explain-results) document which contains metadata about the execution of the operation. |

## replaceOne

Replace the first matching document with a new document.

```javascript
// Replace our gray mats with blue mats
let result = await inventory.replaceOne(
  { item: "mat" }, 
  { item: "mat", qty: 100, tags: ["blue"], size: { h: 10.5, w: 20.5, uom: "cm" } })

// result
{
  "matchedCount": 1,
  "modifiedCount": 1,
  "upsertedCount": 0,
  "upsertedId": null,
  "explain": { ... }
}
```

### replaceOne(filter, doc, options)

| Argument | Type | Description |
| ---- | -- | ----------- |
| **filter** | object | The first document that satisfies the filter criteria will be replaced. The same [MongoDB query document](https://docs.mongodb.com/manual/tutorial/query-documents/) format as the [find()](#find) operation. Supports all operators implemented by the [Sift](https://github.com/crcn/sift.js#readme) NodeJS library.  |
| **doc** | object | The document to replace the first matching dcoument. |
| **options** | object | *Optional* |

**options**

| Field | Type | Description |
| ---- | -- | ----------- |
| **upsert** | boolean | When `true`, if there are no matching documents, then insert a new document with the update modifications applied |
| **MaxKeys** | int | The total number of S3 keys for this operation to scan for a matching document. Default is `100`. |
| **ContinuationToken** | string | A obfuscated token which tells S3 to continue scanning for a matching document from where the last operation left off. |

### Returns

| Field | Type | Description |
| ---- | -- | ----------- |
| **matchedCount** | int | The number of documents that matched the filter criteria. |
| **modifiedCount** | int | The number of documents that were replaced. Will always be `1` if there is a matching document, or `0` if there were no matching documents.  |
| **upsertedCount** | int | `1` if the operation resulted in an upsert (no matching documents). `0` otherwise. |
| **upsertedId** | string | The `_id` of the upserted document. `null` otherwise. |
| **explain** | object | An [explain](#explain-results) document which contains metadata about the execution of the operation. |


## deleteMany

Remove all matching documents.


```javascript
// Remove our inventory of journals and mousepads 
let result = await inventory.deleteMany({ 
  item: { "$in": [ "journal", "mousepad" ] } 
})

// result
{
  "deletedCount": 2,
  "explain": { ... }
}
```

### deleteMany(filter, options)

| Argument | Type | Description |
| ---- | -- | ----------- |
| **filter** | object | All documents that satisfy the filter criteria will be deleted. The same [MongoDB query document](https://docs.mongodb.com/manual/tutorial/query-documents/) format as the [find()](#find) operation. Supports all operators implemented by the [Sift](https://github.com/crcn/sift.js#readme) NodeJS library.  |
| **options** | object | *Optional* |

**options**

| Field | Type | Description |
| ---- | -- | ----------- |
| **MaxKeys** | int | The total number of S3 keys for this operation to scan for a matching document. Default is `100`. |
| **ContinuationToken** | string | A obfuscated token which tells S3 to continue scanning for a matching document from where the last operation left off. |

### Return

| Field | Type | Description |
| ---- | -- | ----------- |
| **deletedCount** | string | The number of documents deleted. `0` if no matching documents. |
| **explain** | object | An [explain](#explain-results) document which contains metadata about the execution of the operation. |


## deleteOne

Remove the first matching document.

```javascript
// Delete the first blue item in our inventory
let result = await inventory.deleteOne({ tags: "blue" })

// result
{
  "deletedCount": 1,
  "explain": { ... }
}
```

### deleteOne(filter, options)

| Argument | Type | Description |
| ---- | -- | ----------- |
| **filter** | object | The first document that satisfies the filter criteria will be deleted. The same [MongoDB query document](https://docs.mongodb.com/manual/tutorial/query-documents/) format as the [find()](#find) operation. Supports all operators implemented by the [Sift](https://github.com/crcn/sift.js#readme) NodeJS library.  |
| **options** | object | *Optional* |

**options**

| Field | Type | Description |
| ---- | -- | ----------- |
| **MaxKeys** | int | The total number of S3 keys for this operation to scan for a matching document. Default is `100`. |
| **ContinuationToken** | string | A obfuscated token which tells S3 to continue scanning for a matching document from where the last operation left off. |

### Return

| Field | Type | Description |
| ---- | -- | ----------- |
| **deletedCount** | string | The number of documents deleted. `0` if no matching documents. |
| **explain** | object | An [explain](#explain-results) document which contains metadata about the execution of the operation. |

## drop

Delete the entire collection and all documents it contains.

```javascript
// Drop the entire inventory collection
let result = await inventory.drop()

// result
{
  "deletedCount": 4,
  "dropped": true,
  "explain": { ... }
}
```

### drop()

### Return

| Field | Type | Description |
| ---- | -- | ----------- |
| **deletedCount** | string | The number of documents deleted from the collection. |
| **dropped** | boolean | `true` if the collection was dropped successfully. |
| **explain** | object | An [explain](#explain-results) document which contains metadata about the execution of the operation. |

# Options

## MaxKeys

## ContinuationToken

# Explain 

SongoDB will return an **explain** document as a part of every successful response.
The document will contain metadata about the execution of the operation.

```javascript 
// Find all items that have 50 or less quantity in stock
let result = await inventory.find({ qty: { "$lte": 50 } })

// result
 {
  "docs": [ 
    { "item": "mousepad", ... },
    { "item": "journal", ... }
  ],
  "explain": {
    "executionStats": {
      "nReturned": 2,
      "executionTimeMillis": 857,
      "totalKeysExamined": 0,
      "totalDocsExamined": 4
    },
    "s3": {
      "KeyCount": 4,
      "MaxKeys": 100,
      "NextContinuationToken": null,
      "TimeMillis": 856
    }
  }
}
```


| Field | Type | Description |
| ---- | ---- | ----------- |
| **executionStats** | object | An embedded document that contains subset of MongoDB explain metadata (e.g. `nReturned`). |
| **s3** | object | An embedded document that contains S3 specific list metadata (e.g. `IsTruncated`)  |


### executionStats

**executionStats** is an embedded document that mimics MongoDB's [executionStats](https://docs.mongodb.com/manual/reference/explain-results/#executionstats) metadata.

| Field | Type | Description |
| ---- | ---- | ----------- |
| **nReturned** | int | The number of documents returned as a result of the operation. |
| **executionTimeMillis** | int | Total time spent executing the operation.|
| **totalKeysExamined** | int | Total number of index entries scanned. SongoDB does not use indexes so this will always be equal to `0`.|
| **totalDocsExamined** | int | Total number of S3 data documents examined during the operation. Does not include system documents (e.g. system.*, admin.*). |


### s3

The **s3** embedded document contains AWS S3 specfic metadata.
It's structure is based off the response of the [listObjectsV2](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectsV2.html) API.


| Field | Type | Description |
| ---- | ---- | ----------- |
| **KeyCount** | int | The number of S3 keys scanned in operation. |
| **MaxKeys** | int | The client-provided maximum number of S3 keys to scan. Default is `100`. |
| **IsTruncated** | boolean | `true` if there are more keys available to be scanned. |
| **ContinuationToken** | string | If a `ContinuationToken` was provided by the client, it is included in the response. |
| **NextContinuationToken** | string | If `IsTruncated` is `true` then this token can be sent in a future request to scan the next batch of keys (up to `MaxKeys`). |
| **TimeMillis** | int | The time spent executing S3 operations. |


# Client Libraries

TBD 

# SongoDB Server

TBD

# More Information

TBD
