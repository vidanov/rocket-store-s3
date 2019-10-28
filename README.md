# Rocket-Store

**Using the AWS S3  as a searchable database.**

Rocket-Store is a high performance solution to simple data storage and retrieval. It's taking advantage of modern file system's exceptionally advanced cashing mechanisms.

It's packaged in a single file to include, with few dependencies.


## Simple to use:

```javascript
result = await rs.post("cars","Mercedes",{owner:"Lisa Simpson",reg:"N3RD"});

result = await rs.get("cars","Mercedes");

result = await rs.delete("cars","Mercedes");

```

## Features:

* Very little footprint.
* Very flexible.
* Few dependencies
* Works without configuration or setup.
* Data stored in JSON format
* Configurable


## Installation

    npm install rocket-store-s3

## Usages

```js
const rs = require('rocket-store-s3');
```

Rocket-Store requires initialization:
```js
rs3.options(
        {
            data_storage_area: "YOUR_S3_BUCKET", 
            data_storage_domain: "THE_NAME_OF_S3_FOLDER_TO_USE", 
            credentials: credentials
        }
    );
```
* credentials are optional if you use Rocket-Store-S3 with Lambda, your AWS credentials will be used.
  Do not put the credentials as a plain text in your code, use the environmental variables or the parameter store or the secrets manager.
  
  ```js
  var credentials = {credentials: {
        accessKeyId: s3accessKey,
        secretAccessKey: s3secretKey,
        region: s3region
    }};
  ```
* When trying to get a non existant collection, the reply is that no records were found.
* When posting to a non existant collection, it is created.

However you can set the storage area and data format to use, with the setOption function, before doing any operation on the data.

## Basic terminology

Rocket-Store was made to replace a more complex database, in a setting that required a low footprint and high performance.

Rocket-Store is intended to store and retrieve records/documents, organized in collections, using a key.

Terms used:

* __Collection__: name of a collections of records. (Like an SQL table)
* __Record__: the data store. (Like an SQL row)
* __Data storage area__: area/directory where collections are stored. (Like SQL data base)
* __Key__: every record has exactly one unique key, which is the same as a file name (same restrictions) and the same wildcards used in searches.

Compare Rocket-Store, SQL and file system terms:


| Rocket-Store       | SQL             | File system    |
| ------------------ | --------------- | -------------- |
| __storage area__   | database        | S3 bucket      |
| __storage domain__ | database prefix | data directory |
| __collection__     | table           | sub directory  |
| __key__            | key             | file name      |
| __record__         | row             | file contents  |

### Post

Stores a record in a collection identified by a unique key

```javascript
post(string <collection>, string <key>, mixed <record> [, integer options])
```

__Collection__ name to contain the records.

__Key__ uniquely identifying the record

No path separators or wildcards etc. are allowed in collection names and keys.
Illigal charakters are silently striped off.

__Options__

  * _ADD_AUTO_INC:  Add an auto incremented sequence to the beginning of the key
  * _ADD_GUID: Add a Globally Unique IDentifier to the key

__Returns__ an associative array containing the result of the operation:

* count : number of records affected (1 on succes)
* key:   string containing the actual key used


If the key already exists, the record will be replaced.

If no key is given, an auto-incremented sequence is used as key.

If the function fails for any reason, an error is thrown.

### Get

Find and retrieve records, in a collection.

```javascript
get([string <collection> [,string <filename> ]]])
```

__Collection__ to search. If no collection name is given, get will return a list of data base assets: collections and sequences etc.

__Key__ to search for. 

__Return__ an array of

* count   : number of records affected
* key     : array of keys
* result  : array of records

### Delete

Delete one or more records, whos key match.

```javascript
delete([string <collection> [,string <key>]])
```

__Collection__ to search. If no collection is given, **THE WHOLE DATABASE IS DELETED!**

__Key__ to search for.

__Return__ an array of

* count : number of records or collections affected


### Configuring

Configuration options is an associative array, that can be parsed during require or with the options function
The array can have these options:

#### Set data storage directory and file format to JSON

```javascript
const rs = require('rocket-store-s3');

var credentials = ''; /* {credentials: {
        accessKeyId: s3accessKey,
        secretAccessKey: s3secretKey,
        region: s3region
    }}; */
await  rs3.options(
        {
            data_storage_area: "rocketstore", // S3 Bucket
            data_storage_domain: "rocketstore_test", // S3 folder
            credentials: credentials
        }
);
```



## Examples

#### Storing records:

```javascript
// Initialize 
const rs = require('./rocket-store-s3');
await  rs3.options(
        {
            data_storage_area: "rocketstore", // S3 Bucket
            data_storage_domain: "rocketstore_test", // S3 folder
            credentials: ''
        }
);
// POST a record
result = await rs.post("cars", "Mercedes_Benz_GT_R", {owner: "Lisa Simpson"});

// GET a record
result = await rs.get("cars", "Mercedes_Benz_GT_R");

console.log(result);
```

The above example will output this:

    {
      count: 1,
      key: [ 'Mercedes_Benz_GT_R' ],
      result: [
        { owner: 'Lisa Simpson' }
      ]
    }

#### Mass insterts

```javascript
  const dataset = {
      Gregs_BMW_740li           : { owner: "Greg Onslow"  },
      Lisas_Mercedes_Benz_GT_R  : { owner: "Lisa Simpson" },
      Bills_BMW_740li           : { owner: "Bill Bo"      },
  };

  var promises = [];
  var ii = 0;
  for(let i in dataset){
    ii++;
    promises[promises.length] = rs.post("cars", i, dataset[i]);
    if(ii >= 20){
      ii = 0;
      await Promise.all(promises);
    }
  }
  if(promises.length > 0)
    await Promise.all(promises);

  result = await rs.get("cars", "Gregs_BMW_740li");

  console.log(result);
```

The above example might output this:

    { count: 1,
      key:[
       'Gregs_BMW_740li'
      ],
      result: [
        { owner: 'Greg Onslow' }
      ]
    }

#### Delete matching records from a collection

```javascript
rs.delete("cars", "Gregs_BMW_740li");
```

#### Delete a whole collection

```javascript
rs.delete("cars");
```

#### Delete the entire database

```javascript
rs.delete();
```

---

## 