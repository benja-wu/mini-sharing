
## Background 
* MongoDB was first released at 2009 

## User Cases

* **Content Management Systems and Mobile Apps**: Due to its flexibility and ability to handle large amounts of data, MongoDB is well-suited for content management systems and mobile applications where the information structure can vary greatly.

* **Real-Time Analytics and High-Speed Logging, Caching, and Real-Time Analytics**: MongoDB's horizontal scalability and performance capabilities make it a good choice for these applications. The ability to store and process large amounts of data in real time is essential in areas such as IoT, analytics, and log processing.

* **Catalogs and User Data Management**: MongoDB's flexible, schema-less data model is well-suited for evolving data requirements in applications such as product catalogs and user data management. It can handle diverse data types and manage applications more efficiently.

* **Geographically Distributed Applications**: MongoDB has built-in support for location-based querying and indexing. Its distributed data model also allows you to spread data across regions to reduce latency, comply with data locality regulations, and increase resilience.


 

## Popup Problems
1. Why MongoDB store things in a very dynamic version, but still get the basic SQL-like CRUD capability? Does it mean MongoDB will get the whole collections from hard disk and filtering for the range searching or dedicated searching purpose?

   ``` 
   MongoDB use index/cache/partitioning&sharding for handling the desired query without whole collection scan.
   
   ```

2. The Pros/Cons of using the JavaScript in Mongo CLI
   JavaScript supports chained function calls makes query with extract operation more cool, such as 

   ```javascript
   > var cursor = db.foo.find().sort({"x" : 1}).limit(1).skip(10);
   ```

   ```
   Pros: familiar syntax in NodeJs based app(many use cases)/ allow to load dynamic JavaScript script/ uese to use 
         
   Cons: performance / concurrency issues/ Not Fully-featured 
   ```

   
3. How can I migrate my exist data in traditional relation based database? Are  there any official tools or some features build-in from MongoDB? 
   
   There are many ways to migrate data from RDBMS to  MongoDB, such as This
The table below:  

   | Name                                           | Description                                                                                                                                                                                                                 | Official |
   | ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
   | MongoDB compass                                | MongoDB's official GUI, Compass, has an import/export feature that can be used to migrate data to and from MongoDB. This method may be feasible for small datasets, but could be cumbersome for larger ones.                | Yes      |
   | MongoDB Altas' Live import feature             | MongoDB Atlas, the DBaaS (Database as a Service) provided by MongoDB, includes a Live Import feature that can migrate data from an existing MongoDB replica set (either on-premises or in the cloud) into an Atlas cluster. | Yes      |
   | Mogify                                         | a Ruby gem for mapping data and copy<br /> https://github.com/anlek/mongify                                                                                                                                                 | No       |
   | customized ETL tools(Extract, Transform, Load) | DIY, using your familiar programming language                                                                                                                                                                               | No       |


4. It that true, doc has not uniq schema can achieve isolation? Better for performance, only affects itself if the doc becomes larger, e.g., inserting many comments into one doc's comment filed array? 

   ```
   MongoDB's operation is atomic at the doc level. Too large doc also hurts performance(limited to 16MB default). In a doc and comments scenario,  embedded comments grow without bounds, which can slow down this doc's query. So better to move comments into a separate collection. It's similar to the normalized model in relational databases.  
   ```

## Regular basic usage learning 
### Basic concepts

**Database**: Similar to RDBMS's database concept, one MongoDB instance can host many databases. (In different age, back in old days IO/CPU is expensive, commit to disk architecture, nowadays resource is far more, commit to network architecture, design principles is changing)

**Collections**: A group of MongoDB documents inside one MongoDB database. Not same schema required inside one collection. 

**Document**: A record in collection called document, can compare it to one data row in RDBMS' table

**BSON**: BSON (Binary JSON) is a binary-encoded serialization of JSON-like documents. MongoDB can handle complicated data type than JSON thanks to BSON's help. 

normal form is good for writing but bad for reading
NoSQL is good for reading but expensive for writing

Morden app is fast reading required more than writing. 

**Index**: Similar concepts to RDBMS'. It also prefer left-most-prefix principle, filtering target small subset index firstly principle, and it also supports **Partial Indexing** as well(Cool! )

**Cursor**: A pointer of a query result set in MongoDB. We can iterate it for each wanted docs or use `explain` to know how MongoDB executing this query.

**Sharding**: Break a huge data set into pieces and stored them distributed in the MongoDB cluster, for performance purpose. Scalability 

Scaling is requires separate into different IO thread

MongoDB is using instance level partition, Divide-and-conquer algorithm.

**Replica Set**: A replica set in MongoDB is a group of `mongod` processes (recommend to separate them into different instances) that maintain the same data set. Replica sets provide redundancy and high availability and are the basis for all production deployments.

Replica is for HA



### 2.1 Get it run
using docker and forwarding its port to host 

```bash
➜  ~ docker run -d -p 27017:27017 --name mongodb mongo 
a9a73f908e97af3e34616dc91fd6c135d908ba11401f0138d17b122cfa4478f0
➜  ~ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS        PORTS                                           NAMES
a9a73f908e97   mongo     "docker-entrypoint.s…"   2 seconds ago   Up 1 second   0.0.0.0:27017->27017/tcp, :::27017->27017/tcp   mongodb

➜  ~ docker exec -it a9a73f908e97 /bin/bash
root@a9a73f908e97:/# ls
bin   data  docker-entrypoint-initdb.d	home	    lib    lib64   media  opt	root  sbin  sys  usr
boot  dev   etc				js-yaml.js  lib32  libx32  mnt	  proc	run   srv   tmp  var
root@a9a73f908e97:/# cd /bin
root@a9a73f908e97:/bin# ./mongosh 
Current Mongosh Log ID:	649d000ff327b0101703a774
Connecting to:		mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+1.8.2
Using MongoDB:		6.0.5
Using Mongosh:		1.8.2
...

test> show databases;
admin   40.00 KiB
config  12.00 KiB
local   40.00 KiB
test> 
```

### 2.2 CRUD
How to switch to another database? 

```javascript
test> show databases;
admin   40.00 KiB
config  12.00 KiB
local   40.00 KiB
test> use movie
switched to db movie

```

How to create one collection? 
```javascript
> db.createCollection('newCollection') 
or u can insert one record into the 'newCollection', then it will create it automattly
> 
```

How to create on doc in the collection?
```javascript
movie> movieTwo= {"Title":"Starwars: Episode VIII – The Last Jedi", "Date": new Date(2017, 11,15 )}
{
  Title: 'Starwars: Episode VIII – The Last Jedi',
  Date: ISODate("2017-12-15T00:00:00.000Z")
}
movie> db.movies.insertOne(movieTwo)
{
  acknowledged: true,
  insertedId: ObjectId("649d1438f327b0101703a776")
}
movie> db.movies.findOne()
{
  _id: ObjectId("649d1438f327b0101703a776"),
  Title: 'Starwars: Episode VIII – The Last Jedi',
  Date: ISODate("2017-12-15T00:00:00.000Z")
}
```

How to fetch one doc from collections?
```javascript
movie> db.movies.findOne()
{
  _id: ObjectId("649d1438f327b0101703a776"),
  Title: 'Starwars: Episode VIII – The Last Jedi',
  Date: ISODate("2017-12-15T00:00:00.000Z")
}
```

How to fetch wanted docs fro collection? 
```javascript
movie> movieThree={"Title":"Starwars:Episode IX – The Rise of Skywalker ", "Date":new Date(2019, 11,20)}
{
  Title: 'Starwars:Episode IX – The Rise of Skywalker ',
  Date: ISODate("2019-12-20T00:00:00.000Z")
}
movie> db.movies.insertOne(movieThree)
{
  acknowledged: true,
  insertedId: ObjectId("649d14bef327b0101703a777")
}

// find the movie only launch after 2019/01/01
movie> db.movies.find({"Date":{"$gte":new Date("01/01/2019")}})
[
  {
    _id: ObjectId("649d14bef327b0101703a777"),
    Title: 'Starwars:Episode IX – The Rise of Skywalker ',
    Date: ISODate("2019-12-20T00:00:00.000Z")
  }
]
movie> 
```

How to insert one filed value into the doc?
```javascript
// insert the favor filed into one doc
movie> db.movies.updateOne( 
... { "Date":{"$gte":new Date("01/01/2019")}}, 
...  {$set: {favor: true}}
... )
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
movie> db.movies.find({})
[
  {
    _id: ObjectId("649d1438f327b0101703a776"),
    Title: 'Starwars: Episode VIII – The Last Jedi',
    Date: ISODate("2017-12-15T00:00:00.000Z")
  },
  {
    _id: ObjectId("649d14bef327b0101703a777"),
    Title: 'Starwars:Episode IX – The Rise of Skywalker ',
    Date: ISODate("2019-12-20T00:00:00.000Z"),
    favor: true
  }
]
```

How to update one filed in the exist doc?

```javascript
movie> db.movies.updateMany({}, { $set: { watchTime: 0 } })
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 2,
  modifiedCount: 1,
  upsertedCount: 0
}

movie> db.movies.find({})
[
  {
    _id: ObjectId("649d1438f327b0101703a776"),
    Title: 'Starwars: Episode VIII – The Last Jedi',
    Date: ISODate("2017-12-15T00:00:00.000Z"),
    watchTime: 0
  },
  {
    _id: ObjectId("649d14bef327b0101703a777"),
    Title: 'Starwars:Episode IX – The Rise of Skywalker ',
    Date: ISODate("2019-12-20T00:00:00.000Z"),
    favor: true,
    watchTime: 0
  }
]

movie> db.movies.updateOne({"Date":{"$gte":new Date("01/01/2019")}},
... {"$inc":{"watchTime":2}}
... )
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
movie> db.movies.find({})
[
  {
    _id: ObjectId("649d1438f327b0101703a776"),
    Title: 'Starwars: Episode VIII – The Last Jedi',
    Date: ISODate("2017-12-15T00:00:00.000Z"),
    watchTime: 0
  },
  {
    _id: ObjectId("649d14bef327b0101703a777"),
    Title: 'Starwars:Episode IX – The Rise of Skywalker ',
    Date: ISODate("2019-12-20T00:00:00.000Z"),
    favor: true,
    watchTime: 2
  }
]

// the watchTime of Starwars IX is set to 2 
```

How to delete one doc? 

```javascript
movie> db.movies.deleteOne({"Date":{"$gte":new Date("01/01/2019")}})
{ acknowledged: true, deletedCount: 1 }
movie> db.movies.find({})
[
  {
    _id: ObjectId("649d1438f327b0101703a776"),
    Title: 'Starwars: Episode VIII – The Last Jedi',
    Date: ISODate("2017-12-15T00:00:00.000Z"),
    watchTime: 0
  }
]
```

### 2.3 Indexing

How to create a index for one collection? 
```javascript
// insert 200,000 data into db 
movie> for (i=0; i<200,000; i++) {
... db.users.insertOne( {
...  "num": i ,
...  "name": "user_"+i
... ,
...  "age" : Math.floor(Math.random()*110),
...  "created" : new Date()
... }
... );
... }

movie> db.users.createIndex({"name":1})
name_1

movie> db.users.getIndexes()
[
  { v: 2, key: { _id: 1 }, name: '_id_' },
  { v: 2, key: { name: 1 }, name: 'name_1' }
]

```

How to create compound index? 

```javascript
movie> db.users.createIndex({name :1, age :1})
name_1_age_1
movie> db.users.getIndexes()
[
  { v: 2, key: { _id: 1 }, name: '_id_' },
  { v: 2, key: { name: 1 }, name: 'name_1' },
  { v: 2, key: { name: 1, age: 1 }, name: 'name_1_age_1' }
]
```

How can I confirm the query is using index or not? 
```javascript
Using the explian, and see the query wining plan for `IXSCAN` keyword. such as
movie> db.users.find({"name":"user_1"}).explain("executionStats")
{
  explainVersion: '1',
  queryPlanner: {
    namespace: 'movie.users',
    indexFilterSet: false,
    parsedQuery: { name: { '$eq': 'user_1' } },
    queryHash: '64908032',
    planCacheKey: 'A6C0273F',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      stage: 'FETCH',
      inputStage: {
        stage: 'IXSCAN',
        keyPattern: { name: 1 },
        indexName: 'name_1',
        isMultiKey: false,
        multiKeyPaths: { name: [] },
        isUnique: false,
        isSparse: false,
        isPartial: false,
        indexVersion: 2,
        direction: 'forward',
        indexBounds: { name: [ '["user_1", "user_1"]' ] }
      }
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 1,
    executionTimeMillis: 2,
    totalKeysExamined: 1,
    totalDocsExamined: 1,
    executionStages: {
      stage: 'FETCH',
      nReturned: 1,
      executionTimeMillisEstimate: 0,
      works: 2,
      advanced: 1,
      needTime: 0,
      needYield: 0,
      saveState: 0,
      restoreState: 0,
      isEOF: 1,
      docsExamined: 1,
      alreadyHasObj: 0,
      inputStage: {
        stage: 'IXSCAN',
        nReturned: 1,
        executionTimeMillisEstimate: 0,
        works: 2,
        advanced: 1,
        needTime: 0,
        needYield: 0,
        saveState: 0,
        restoreState: 0,
        isEOF: 1,
        keyPattern: { name: 1 },
        indexName: 'name_1',
        isMultiKey: false,
        multiKeyPaths: { name: [] },
        isUnique: false,
        isSparse: false,
        isPartial: false,
        indexVersion: 2,
        direction: 'forward',
        indexBounds: { name: [ '["user_1", "user_1"]' ] },
        keysExamined: 1,
        seeks: 1,
        dupsTested: 0,
        dupsDropped: 0
      }
    }
  },
  command: { find: 'users', filter: { name: 'user_1' }, '$db': 'movie' },
  serverInfo: {
    host: 'a9a73f908e97',
    port: 27017,
    version: '6.0.5',
    gitVersion: 'c9a99c120371d4d4c52cbb15dac34a36ce8d3b1d'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
  },
  ok: 1
}


```

## Takeaways

1. MongoDB is built on a distributed architecture designed for massive volumes and varied structures of data generated by large-scale, modern application. 

2. For applications that require complex transactions with multiple operations or queries that return multiple documents, a traditional relational database might be a better choice