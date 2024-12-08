# MongoDB Interview Questions for SDE Profile 

This README contains MongoDB interview questions and answers for a Software Developer profile with These questions cover various MongoDB concepts, including basic operations, data modeling, aggregation, performance optimization, and more.

## Topics Covered:
1. [MongoDB Basics](#t1)
2. [Data Modeling](#t2)
3. [Indexes](#t3)
4. [Querying in MongoDB](#t4)
5. [Aggregation](#t5)
6. [Performance Optimization](#t6)
7. [Replication and Sharding](#t7)
8. [Transactions](#t8)
9. [Data Integrity and Consistency](#t9)
10. [Backup and Restore](#t10)
11. [Security](#t11)
12. [Monitoring and Debugging](#t12)

---

<div id="t1"></div>

## 1. MongoDB Basics [↑](#topics-covered)

MongoDB is a NoSQL database that stores data in a flexible, document-oriented format (BSON). Key features include:
- **Scalability**: MongoDB supports horizontal scaling through sharding.
- **Flexible Schema**: Documents in collections can have different structures.
- **High Availability**: MongoDB has built-in replication for high availability.
- **Aggregation Framework**: For complex queries and data transformations.
- **Rich Query Language**: Support for a variety of queries, including geospatial queries.

---
<div id="t2"></div>

## 2. Data Modeling [↑](#topics-covered)

Data modeling in MongoDB is about designing schemas that suit your application needs. It often involves trade-offs between:

- **Embedding**: Storing related data in a single document.
- **Referencing**: Using references between documents.
- **Denormalization**: Storing redundant data for read optimization.

### Best Practices:
1. Optimize for query patterns and access frequency.
2. Keep document size under 16MB.
3. Leverage indexes for faster queries.

### Example of Embedding Data:
```
db.orders.insertOne({
  "orderId": 12345,
  "customer": { "name": "John Doe", "email": "john@example.com" },
  "items": [
    { "productId": "p1", "quantity": 2 },
    { "productId": "p2", "quantity": 1 }
  ]
})
```

### Example of Referencing Data:
```
db.customers.insertOne({ "_id": "c1", "name": "John Doe", "email": "john@example.com" })
db.orders.insertOne({
  "orderId": 12345,
  "customerId": "c1",
  "items": [
    { "productId": "p1", "quantity": 2 },
    { "productId": "p2", "quantity": 1 }
  ]
})
```

<div id="t3"></div>

## 3. Indexes [↑](#topics-covered)

 
Indexes are crucial in MongoDB because they improve the speed of query operations by reducing the number of documents MongoDB needs to scan. Without indexes, MongoDB performs a full collection scan for queries, which can be very slow for large datasets.

Types of indexes in MongoDB include:

- **Single Field Indexes**: These indexes are created on a single field, making queries on that field faster. For example, if you're querying the `sku` field frequently, an index on `sku` would speed up such queries.

  Example:
  ```
  db.products.createIndex({ "sku": 1 })
  ```

- **Compound Indexes**: These indexes are created on multiple fields. MongoDB uses them when queries involve more than one field. Compound indexes can optimize queries that filter on multiple fields.

  Example:
  ```
  db.products.createIndex({ "sku": 1, "location.city": 1 })
  ```

  This index is useful for queries that filter by both `sku` and `location.city`.

- **Multikey Indexes**: These are used when a field contains an array, and MongoDB needs to index each element of the array. This is useful for queries that match on array elements.

  Example:
  ```
  db.products.createIndex({ "tags": 1 })
  ```

- **Text Indexes**: MongoDB supports text search on string fields using text indexes. This allows you to search for words or phrases within a text field.

  Example:
  ```
  db.products.createIndex({ "description": "text" })
  ```

  This is useful for searching product descriptions for keywords.

- **Hashed Indexes**: These indexes use a hash of the field value and are primarily used for shard key values in sharded clusters. They ensure an even distribution of data across shards.

  Example:
  ```
  db.products.createIndex({ "sku": "hashed" })
  ```

- **Geospatial Indexes**: These indexes support location-based queries, such as finding products near a specific location using latitude and longitude coordinates.

  Example:
  ```
  db.products.createIndex({ "location": "2dsphere" })
  ```

- **Wildcard Indexes**: These indexes allow indexing on all fields in a document, which is useful when you don't know in advance which fields will be queried.

  Example:
  ```
  db.products.createIndex({ "$**": 1 })
  ```

### How do you decide which indexes to create for a MongoDB collection?
 
When deciding which indexes to create, consider the following:

- **Query Patterns**: Analyze the queries most frequently run against the collection. Index fields that are frequently queried, especially those in the `WHERE` clause or for sorting (`ORDER BY`).
  
- **Performance**: Too many indexes can slow down write operations (insert, update, delete) because MongoDB has to update each index. Prioritize indexes that significantly speed up your read queries.

- **Cardinality of Fields**: Fields with high cardinality (e.g., unique or near-unique values like `sku` or `user_id`) are ideal candidates for indexing, as they can improve query performance significantly.

- **Compound Indexes**: If your queries filter on multiple fields, creating compound indexes on those fields can optimize the query performance.

- **Use MongoDB Profiler**: The MongoDB profiler can help identify slow queries. You can then decide which fields should be indexed based on these slow query logs.

- **Indexing for Sharding**: When sharding, choose a shard key carefully. It's best to use a field with a high cardinality that ensures data is evenly distributed across shards.

---

<div id="t4"></div>

## 4. Querying in MongoDB [↑](#topics-covered)

MongoDB provides various ways to query data from the database. Below are the different types of queries:

- **Basic Queries**: MongoDB allows you to query collections using a simple filter on fields. This returns all documents matching the filter criteria.

  Example:
  ```
  db.products.find({ "sku": "12345" })
  ```

  This query finds all documents in the `products` collection where the `sku` field is `12345`.

- **Equality Query**: You can also perform equality checks on multiple fields.

  Example:
  ```
  db.products.find({ "sku": "12345", "status": "available" })
  ```

  This query finds documents where both the `sku` is `12345` and the `status` is `available`.

- **Comparison Queries**: MongoDB allows you to perform comparisons such as `$gt`, `$lt`, `$gte`, and `$lte` to filter documents based on numeric or date values.

  Example:
  ```
  db.products.find({ "price": { "$gt": 100 } })
  ```

  This query returns products with a price greater than 100.

- **Logical Queries**: MongoDB allows logical operations using `$and`, `$or`, and `$nor` to combine multiple conditions.

  Example:
  ```
  db.products.find({
    "$or": [
      { "price": { "$lt": 50 } },
      { "sku": "12345" }
    ]
  })
  ```

  This query returns products with a price less than 50 or an `sku` of `12345`.

- **Array Queries**: MongoDB allows you to query documents that contain arrays. You can match an element in an array or check for array length.

  Example:
  ```
  db.products.find({ "tags": "sale" })
  ```

  This query finds products that have `sale` in the `tags` array.

- **Projection**: In MongoDB, projection allows you to select specific fields to return in the result, excluding the ones you don't need.

  Example:
  ```
  db.products.find({ "sku": "12345" }, { "price": 1, "name": 1 })
  ```

  This query retrieves only the `price` and `name` fields of the documents that match the `sku`.

- **Sorting**: MongoDB allows you to sort the query results based on a field, either in ascending or descending order.

  Example:
  ```
  db.products.find({}).sort({ "price": -1 })
  ```

  This query returns all products sorted by `price` in descending order.

- **Limit and Skip**: MongoDB allows you to limit the number of results returned and skip a certain number of results, useful for pagination.

  Example:
  ```
  db.products.find({}).limit(10).skip(20)
  ```

  This query retrieves 10 products, skipping the first 20 results.

- **Text Search Queries**: If you have a text index, MongoDB allows you to perform full-text search on string fields.

  Example:
  ```
  db.products.find({ "$text": { "$search": "laptop" } })
  ```

  This query finds documents containing the word `laptop` in indexed text fields.

- **Geospatial Queries**: If you have a geospatial index, you can query documents based on location (latitude and longitude).

  Example:
  ```
  db.products.find({
    "location": {
      "$near": {
        "$geometry": { "type": "Point", "coordinates": [ -73.9667, 40.78 ] },
        "$maxDistance": 5000
      }
    }
  })
  ```

  This query finds products within a 5-kilometer radius of the given point.

---

<div id="t5"></div>

## 5. Aggregation [↑](#topics-covered)
 
Aggregation in MongoDB is the process of transforming and combining data from a collection to get meaningful insights. It is used for operations like filtering, grouping, sorting, and transforming data.

The aggregation framework uses a pipeline of stages where each stage performs an operation on the data. Each stage passes its results to the next stage in the pipeline. Some of the most commonly used stages in the aggregation pipeline include:

- **$match**: Filters the documents based on specified criteria. This is similar to the `find` operation but more powerful since it can include complex queries and logical operations.

  Example:
  ```
  db.products.aggregate([
    { "$match": { "category": "electronics" } }
  ])
  ```

  This query returns all products where the `category` is `electronics`.

- **$group**: Groups documents by a specified field, performing an aggregation operation on the grouped data, such as summing or averaging values.

  Example:
  ```
  db.products.aggregate([
    { "$group": { "_id": "$category", "total_sales": { "$sum": "$sales" } } }
  ])
  ```

  This query groups products by `category` and calculates the total sales for each category.

- **$project**: Reshapes the documents by including, excluding, or adding new fields.

  Example:
  ```
  db.products.aggregate([
    { "$project": { "name": 1, "price": 1, "category": 1 } }
  ])
  ```

  This query returns only the `name`, `price`, and `category` fields of each document.

- **$sort**: Sorts the documents by specified fields, either ascending or descending.

  Example:
  ```
  db.products.aggregate([
    { "$sort": { "price": -1 } }
  ])
  ```

  This query sorts products in descending order of price.

- **$limit**: Limits the number of documents returned.

  Example:
  ```
  db.products.aggregate([
    { "$limit": 5 }
  ])
  ```

  This query limits the result to the first 5 documents.

- **$skip**: Skips a specified number of documents.

  Example:
  ```
  db.products.aggregate([
    { "$skip": 10 }
  ])
  ```

  This query skips the first 10 documents.

- **$unwind**: Deconstructs an array field from the input documents to output one document for each element in the array.

  Example:
  ```
  db.products.aggregate([
    { "$unwind": "$tags" }
  ])
  ```

  This query unwinds the `tags` array and outputs one document per tag.

- **$lookup**: Performs a left outer join to another collection. This is useful when you need to combine data from two collections.

  Example:
  ```
  db.orders.aggregate([
    { "$lookup": {
      "from": "products",
      "localField": "product_id",
      "foreignField": "sku",
      "as": "product_details"
    } }
  ])
  ```

  This query performs a join between the `orders` collection and the `products` collection, linking `product_id` in `orders` with `sku` in `products`, and returns the `product_details`.

- **$addFields**: Adds new fields to the documents. This is useful for adding computed values to the aggregation pipeline.

  Example:
  ```
  db.products.aggregate([
    { "$addFields": { "discounted_price": { "$multiply": ["$price", 0.9] } } }
  ])
  ```

  This query adds a `discounted_price` field to each document, which is 90% of the `price`.

- **$count**: Counts the number of documents passed through the pipeline.

  Example:
  ```
  db.products.aggregate([
    { "$count": "total_products" }
  ])
  ```

  This query counts the total number of documents in the `products` collection.

---

<div id="t6"></div>

## 6. Performance Optimization [↑](#topics-covered)


Performance optimization in MongoDB involves strategies to improve query speed, indexing, and overall system efficiency. Below are some key practices for optimizing MongoDB performance:

- **Indexing**: Indexes are essential for fast query performance. By creating appropriate indexes, MongoDB can quickly locate the documents without scanning the entire collection.

  Example:
  ```
  db.products.createIndex({ "sku": 1 })
  ```

  This creates an index on the `sku` field, which speeds up queries that search for products by `sku`.

- **Use Covered Queries**: A covered query is one where all the fields in the query are included in the index. This allows MongoDB to return the results directly from the index, without scanning the actual documents.

  Example:
  ```
  db.products.createIndex({ "sku": 1, "price": 1 })
  db.products.find({ "sku": "12345", "price": { "$gt": 100 } })
  ```

  The query uses the index to fetch the results directly, making it faster than scanning the entire collection.

- **Use Projection to Limit Fields**: When querying, avoid returning unnecessary fields. Use projection to limit the fields returned to only the ones you need.

  Example:
  ```
  db.products.find({ "sku": "12345" }, { "name": 1, "price": 1 })
  ```

  This query returns only the `name` and `price` fields, improving performance by not fetching other fields unnecessarily.

- **Optimize Aggregation Pipelines**: When using the aggregation framework, ensure the pipeline stages are ordered optimally. For example, place `$match` and `$sort` as early as possible to reduce the amount of data passed through the pipeline.

  Example:
  ```
  db.products.aggregate([
    { "$match": { "category": "electronics" } },
    { "$sort": { "price": -1 } },
    { "$limit": 10 }
  ])
  ```

  This query first filters the documents by `category`, sorts them by `price`, and limits the result to the top 10, minimizing the data processed in each stage.

- **Use Indexes with Aggregation**: When performing aggregations, use indexes to speed up the `$match` stage and other stages that can benefit from them.

  Example:
  ```
  db.products.createIndex({ "category": 1 })
  db.products.aggregate([
    { "$match": { "category": "electronics" } },
    { "$group": { "_id": "$category", "total_sales": { "$sum": "$sales" } } }
  ])
  ```

  This query uses the index on `category` to quickly filter the documents before performing the `$group` stage.

- **Avoid Large Joins in `$lookup`**: The `$lookup` stage can be expensive, especially when joining large collections. To optimize performance, limit the fields returned by the joined collection and use indexes on the join fields.

  Example:
  ```
  db.orders.aggregate([
    { "$lookup": {
      "from": "products",
      "localField": "product_id",
      "foreignField": "sku",
      "as": "product_details"
    }},
    { "$project": { "product_details.name": 1, "product_details.price": 1 } }
  ])
  ```

  This query limits the fields returned from the `products` collection to only the necessary ones, improving performance.

- **Sharding**: Sharding is a method of distributing data across multiple servers to ensure that a large dataset can be handled efficiently. Sharding is particularly useful when you have large datasets and high traffic.

  Example:
  ```
  db.products.createIndex({ "sku": 1 })
  db.products.shardCollection("products", { "sku": 1 })
  ```

  This shards the `products` collection based on the `sku` field, allowing MongoDB to distribute the data across multiple shards, improving performance for large datasets.

- **Write Concern and Journaling**: If write performance is a concern, you can reduce the write concern to `1` (acknowledged by the primary only), but be aware that this increases the risk of data loss. Similarly, turning off journaling may improve performance but at the cost of data durability.

  Example:
  ```
  db.products.insertOne({ "sku": "12345", "name": "Product 1" }, { writeConcern: { w: 1 } })
  ```

  This reduces the write concern to improve insert performance, but it may result in data loss if the primary node fails.

- **Connection Pooling**: Use connection pooling to reuse database connections rather than creating new ones for each request. This reduces the overhead of establishing new connections and improves overall performance.

  Example:
  ```
  const client = new MongoClient(uri, { useUnifiedTopology: true, poolSize: 10 });
  ```

  This creates a connection pool with a maximum of 10 connections.

- **Limit the Use of `$unwind` and `$group`**: The `$unwind` and `$group` stages are powerful but can be slow when processing large datasets. Use them wisely and avoid them when possible.

  Example:
  ```
  db.products.aggregate([
    { "$unwind": "$tags" },
    { "$group": { "_id": "$tags", "count": { "$sum": 1 } } }
  ])
  ```

  This query uses both `$unwind` and `$group`, which could be slow for large collections, so it should be optimized for performance.

---

<div id="t7"></div>

## 7. Replication and Sharding [↑](#topics-covered)


- **Replication** in MongoDB involves copying data from one server (the primary node) to one or more other servers (secondary nodes). This ensures data redundancy and high availability. If the primary node fails, one of the secondary nodes can be promoted to the primary, ensuring continued availability.

  - **Setting up a Replica Set**: A replica set is a group of MongoDB servers that maintain the same data. A replica set consists of one primary node and multiple secondary nodes. The primary node receives all write operations, and the secondaries replicate the data.

  Example:
  ```
  rs.initiate()
  rs.add("secondary1:27017")
  rs.add("secondary2:27017")
  ```

  This command initializes a replica set and adds two secondary nodes to it.

- **Replication Reads**: By default, read operations are sent to the primary node. However, you can configure MongoDB to read from secondary nodes to distribute the read load.

  Example:
  ```
  db.getMongo().setReadPref("secondary")
  ```

  This command configures the application to read from a secondary node in the replica set.

- **Sharding**: Sharding in MongoDB is a method of distributing data across multiple servers to ensure that a large dataset can be handled efficiently. It allows MongoDB to horizontally scale by splitting data into smaller, manageable chunks and distributing those chunks across different servers (shards).

  - **Sharding Key**: When you shard a collection, you choose a shard key. The shard key determines how the data is distributed across shards. It's crucial to choose an appropriate shard key to balance the data evenly across shards.

  Example:
  ```
  db.products.createIndex({ "category": 1 })
  db.products.shardCollection("products", { "category": 1 })
  ```

  This command creates a shard key on the `category` field and shards the `products` collection based on this key.

- **Config Servers**: MongoDB uses config servers to store metadata about the sharded cluster. These servers track the chunks of data and their distribution across the shards. A sharded cluster requires at least three config servers for redundancy.

  Example:
  ```
  mongod --configsvr --replSet configReplSet --port 27019 --dbpath /data/config
  ```

  This command starts a config server for the sharded cluster.

- **Mongos**: `mongos` is the query router that acts as an interface between the application and the sharded cluster. It directs queries to the appropriate shard based on the shard key.

  Example:
  ```
  mongos --configdb configReplSet/localhost:27019
  ```

  This command starts a `mongos` query router that connects to the config servers in the sharded cluster.

- **Sharding Data**: When data is inserted into a sharded collection, MongoDB uses the shard key to determine the appropriate shard for that data. MongoDB automatically balances the data across shards as necessary to distribute the load evenly.

  - **Balancing**: MongoDB continuously balances data across shards to ensure even distribution. The balancer moves chunks of data between shards to avoid overloading any single shard.

  Example:
  ```
  sh.status()
  ```

  This command displays the current status of the sharded cluster, including the distribution of chunks across shards.

- **Failover and Recovery in Replication**: In case the primary node fails, one of the secondary nodes is automatically promoted to become the new primary. This process is called automatic failover and ensures high availability for the application.

  Example:
  ```
  rs.stepDown()
  ```

  This command forces the current primary to step down and triggers an automatic failover in the replica set.

---

<div id="t8"></div>

## 8. Transactions [↑](#topics-covered)


MongoDB supports multi-document transactions, which allow you to execute multiple operations across different documents or collections in a way that guarantees atomicity, consistency, isolation, and durability (ACID properties). Transactions are essential for applications that require operations involving more than one document or collection to be executed atomically.

- **Starting a Transaction**: To start a transaction, you need to begin a session and use the `startTransaction` method. The session allows you to perform multiple operations within the same transaction.

  Example:
  ```
  const session = client.startSession();
  session.startTransaction();
  ```

  This starts a new session and begins a transaction.

- **Performing Operations within a Transaction**: After starting a transaction, you can execute multiple operations (like inserts, updates, or deletes) on one or more collections. These operations will not be visible to other clients until the transaction is committed.

  Example:
  ```
  const productsCollection = client.db("ecommerce").collection("products");
  productsCollection.updateOne({ "sku": "12345" }, { "$set": { "price": 200 } }, { session });
  productsCollection.insertOne({ "sku": "67890", "name": "New Product" }, { session });
  ```

  This example performs two operations: updating a product and inserting a new product, both within the same transaction.

- **Committing a Transaction**: Once all the operations have been executed successfully, you can commit the transaction to make the changes permanent.

  Example:
  ```
  session.commitTransaction();
  session.endSession();
  ```

  This commits the transaction and ends the session. The changes are now permanent.

- **Rolling Back a Transaction**: If an error occurs or if the operations need to be undone, you can roll back the transaction. This ensures that no partial updates are saved to the database.

  Example:
  ```
  session.abortTransaction();
  session.endSession();
  ```

  This rolls back the transaction, discarding all the changes made during the transaction.

- **Error Handling in Transactions**: In practice, it's important to handle errors during transactions to ensure that the transaction can be either committed or rolled back properly.

  Example:
  ```
  try {
    session.startTransaction();
    productsCollection.updateOne({ "sku": "12345" }, { "$set": { "price": 200 } }, { session });
    productsCollection.insertOne({ "sku": "67890", "name": "New Product" }, { session });
    session.commitTransaction();
  } catch (error) {
    console.error("Transaction failed: ", error);
    session.abortTransaction();
  } finally {
    session.endSession();
  }
  ```

  This example includes proper error handling within a transaction. If an error occurs, the transaction is rolled back.

- **Transactions Across Multiple Shards**: MongoDB supports multi-shard transactions in sharded clusters. When performing a transaction that spans multiple shards, MongoDB ensures that all shards involved in the transaction are committed or rolled back together atomically.

  Example:
  ```
  const ordersCollection = client.db("ecommerce").collection("orders");
  session.startTransaction();
  ordersCollection.updateOne({ "orderId": "abc123" }, { "$set": { "status": "shipped" } }, { session });
  session.commitTransaction();
  session.endSession();
  ```

  This transaction ensures that the order status update is performed atomically across the involved shards.

- **Transaction Read Concern and Write Concern**: You can set read and write concerns for transactions to specify the level of consistency and durability required for the operations within the transaction.

  Example:
  ```
  const options = { readConcern: { level: "majority" }, writeConcern: { w: "majority" } };
  productsCollection.updateOne({ "sku": "12345" }, { "$set": { "price": 200 } }, options);
  ```

  This ensures that the read and write operations within the transaction meet the majority consistency level.

---

<div id="t9"></div>

## 9. Data Integrity and Consistency [↑](#topics-covered)


MongoDB provides mechanisms to maintain data integrity and consistency in various scenarios, ensuring the accuracy and reliability of the stored data. These mechanisms include transactions, validation rules, and atomic operations.

- **Atomic Operations**: MongoDB supports atomic operations on a single document, meaning that all changes to a document are completed successfully or not at all. This ensures the consistency of data at the document level.

  Example:
  ```
  db.products.updateOne({ "sku": "12345" }, { "$inc": { "stock": -1 } })
  ```

  This operation ensures that the stock for a product is decreased atomically.

- **Transactions for Multi-Document Consistency**: For operations spanning multiple documents or collections, MongoDB provides transactions. Transactions allow multiple operations to be grouped together in an atomic unit, ensuring that changes are either committed or rolled back as a whole.

  Example:
  ```
  const session = client.startSession();
  session.startTransaction();
  const ordersCollection = client.db("ecommerce").collection("orders");
  const productsCollection = client.db("ecommerce").collection("products");
  
  ordersCollection.insertOne({ "orderId": "abc123", "productId": "12345", "quantity": 2 }, { session });
  productsCollection.updateOne({ "sku": "12345" }, { "$inc": { "stock": -2 } }, { session });
  
  session.commitTransaction();
  session.endSession();
  ```

  This ensures that an order and stock update are committed together or not at all, maintaining consistency between the orders and products collections.

- **Validation Rules**: MongoDB provides schema validation features that help enforce data integrity by ensuring that documents adhere to specified rules before they are inserted or updated. You can define validation rules for specific fields, such as types, ranges, or presence.

  Example:
  ```
  db.createCollection("products", {
    validator: {
      $jsonSchema: {
        bsonType: "object",
        required: ["sku", "name", "price"],
        properties: {
          sku: { bsonType: "string" },
          name: { bsonType: "string" },
          price: { bsonType: "double" },
        }
      }
    }
  })
  ```

  This enforces that all documents in the `products` collection must have the `sku`, `name`, and `price` fields, with the appropriate data types.

- **Write Concern**: Write concern defines the level of acknowledgment requested from MongoDB for write operations. It ensures that data is written to the specified number of nodes (e.g., primary, secondary) before returning acknowledgment, improving data consistency.

  Example:
  ```
  db.products.insertOne({ "sku": "12345", "name": "Product A", "price": 100 }, { writeConcern: { w: "majority" } })
  ```

  This ensures that the write operation is acknowledged by a majority of the replica set members, improving data durability and consistency.

- **Read Concern**: Read concern defines the consistency level for read operations. MongoDB provides different read concern levels, such as `local`, `majority`, and `linearizable`, which control the visibility of data based on its replication state.

  Example:
  ```
  db.products.find({ "sku": "12345" }).readConcern("majority")
  ```

  This ensures that the read operation reflects data that has been acknowledged by a majority of the replica set members, providing stronger consistency guarantees.

- **Data Integrity in Sharded Clusters**: In sharded clusters, MongoDB ensures data consistency across shards. The sharding key plays a crucial role in maintaining consistency by ensuring that related data is grouped together on the same shard. Additionally, transactions can span multiple shards to maintain consistency across distributed data.

  Example:
  ```
  db.products.createIndex({ "category": 1 })
  db.products.shardCollection("products", { "category": 1 })
  ```

  This example creates a shard key on the `category` field and ensures that related product data is distributed across shards while maintaining data integrity.

- **Optimistic Concurrency Control**: MongoDB uses an optimistic concurrency control model to handle concurrent updates to documents. If two operations try to modify the same document at the same time, MongoDB detects the conflict and throws an error, allowing the application to handle the conflict.

  Example:
  ```
  db.products.updateOne({ "sku": "12345", "stock": { "$gte": 1 } }, { "$inc": { "stock": -1 } })
  ```

  This update operation ensures that the stock field is only decremented if the value is greater than or equal to 1, preventing the stock from going negative and ensuring integrity during concurrent updates.

- **Handling Data Corruption**: In case of data corruption, MongoDB offers tools like `mongorestore` and `fsync` to ensure data consistency during backups and recovery processes.

  Example:
  ```
  mongorestore --drop --dir /data/backup
  ```

  This command restores a backup and drops any existing data in the target database to ensure consistency with the backup.

---

<div id="t10"></div>

## 10. Backup and Restore [↑](#topics-covered)


MongoDB provides several tools and methods for backing up and restoring data. The two main tools for this are `mongodump` for backing up data and `mongorestore` for restoring data. These tools can help you safeguard your data and recover it when needed.

- **mongodump**: The `mongodump` tool is used to create a binary backup of MongoDB data. It can be used to back up the entire database or specific collections.

  Example:
  ```
  mongodump --uri="mongodb://localhost:27017" --out=/data/backup
  ```

  This command creates a backup of the entire MongoDB instance at `localhost:27017` and stores it in the `/data/backup` directory.

- **Backing up a Specific Database**: If you want to back up a specific database, you can use the `--db` option with `mongodump`.

  Example:
  ```
  mongodump --uri="mongodb://localhost:27017" --db=ecommerce --out=/data/backup
  ```

  This command backs up the `ecommerce` database and stores the backup in the `/data/backup` directory.

- **Backing up a Specific Collection**: You can also back up individual collections using the `--collection` option.

  Example:
  ```
  mongodump --uri="mongodb://localhost:27017" --db=ecommerce --collection=products --out=/data/backup
  ```

  This command backs up only the `products` collection from the `ecommerce` database.

- **mongorestore**: The `mongorestore` tool is used to restore data from a backup created with `mongodump`. It can restore the entire backup or specific collections from the backup.

  Example:
  ```
  mongorestore --uri="mongodb://localhost:27017" --dir=/data/backup
  ```

  This command restores the entire backup from the `/data/backup` directory to the MongoDB instance at `localhost:27017`.

- **Restoring a Specific Database**: If you want to restore a specific database from a backup, use the `--db` option with `mongorestore`.

  Example:
  ```
  mongorestore --uri="mongodb://localhost:27017" --db=ecommerce --dir=/data/backup/ecommerce
  ```

  This command restores the `ecommerce` database from the backup stored in `/data/backup/ecommerce`.

- **Restoring a Specific Collection**: You can restore an individual collection from the backup using the `--collection` option.

  Example:
  ```
  mongorestore --uri="mongodb://localhost:27017" --db=ecommerce --collection=products --dir=/data/backup/ecommerce/products.bson
  ```

  This command restores the `products` collection from the backup stored in `/data/backup/ecommerce/products.bson`.

- **Backup with Authentication**: If your MongoDB instance is secured with authentication, you will need to provide the appropriate credentials when backing up the data.

  Example:
  ```
  mongodump --uri="mongodb://username:password@localhost:27017" --out=/data/backup
  ```

  This command creates a backup of the MongoDB instance while using authentication to log in.

- **Restore with Authentication**: Similarly, when restoring data to a secured MongoDB instance, you need to supply authentication credentials.

  Example:
  ```
  mongorestore --uri="mongodb://username:password@localhost:27017" --dir=/data/backup
  ```

  This command restores the backup to the MongoDB instance while using authentication.

- **Incremental Backup and Restore**: For more advanced backup strategies, you can use the `--oplog` option with `mongodump` to create incremental backups by capturing the operation log (oplog). This is particularly useful for backing up data in a replica set.

  Example:
  ```
  mongodump --uri="mongodb://localhost:27017" --oplog --out=/data/backup
  ```

  This command creates an incremental backup, including the oplog, to allow for point-in-time recovery.

- **Point-in-Time Restore**: After creating an oplog-based backup, you can restore the data up to a specific point in time.

  Example:
  ```
  mongorestore --uri="mongodb://localhost:27017" --oplogReplay --dir=/data/backup
  ```

  This command restores the data to a specific point in time using the oplog.

- **Backup and Restore with Compression**: You can also compress the backup files to save storage space by using the `--gzip` option with both `mongodump` and `mongorestore`.

  Example:
  ```
  mongodump --uri="mongodb://localhost:27017" --out=/data/backup --gzip
  ```

  This command creates a compressed backup of the MongoDB instance.

  Example:
  ```
  mongorestore --uri="mongodb://localhost:27017" --dir=/data/backup --gzip
  ```

  This command restores a compressed backup.

---

<div id="t11"></div>

## 11. Security [↑](#topics-covered)

Securing a MongoDB instance is crucial to protect sensitive data. MongoDB provides several security features such as authentication, authorization, encryption, and auditing to ensure data is protected.

- **Authentication**: MongoDB supports various authentication mechanisms such as SCRAM (Salted Challenge Response Authentication Mechanism), LDAP (Lightweight Directory Access Protocol), and X.509 certificates.

  Example: Enabling authentication with SCRAM.
  ```
  # Edit the mongod.conf file and set authorization to "enabled"
  security:
    authorization: "enabled"
  ```

  After enabling authentication, users must authenticate before accessing the database.

- **Creating Users**: You can create users with specific roles and permissions to limit access to certain resources within MongoDB.

  Example:
  ```
  db.createUser({
    user: "admin",
    pwd: "password123",
    roles: [{ role: "root", db: "admin" }]
  })
  ```

  This creates an `admin` user with `root` privileges.

- **Role-Based Access Control (RBAC)**: MongoDB uses RBAC to control access. You can assign specific roles to users, granting them only the permissions they need.

  Example: Creating a read-only user.
  ```
  db.createUser({
    user: "readonlyUser",
    pwd: "readonlyPassword",
    roles: [{ role: "read", db: "ecommerce" }]
  })
  ```

  This creates a user with read-only access to the `ecommerce` database.

- **Encryption**: MongoDB supports encryption at rest and in transit. Encryption at rest ensures that stored data is encrypted, and encryption in transit protects data during communication between the client and the server.

  - **Encryption at Rest**: You can enable encryption at rest by configuring the `--encryptionKeyFile` parameter.

    Example:
    ```
    mongod --encryptionKeyFile /path/to/encryptionKey --dbpath /data/db
    ```

  - **Encryption in Transit**: You can enable TLS/SSL encryption for data in transit by configuring `--sslMode` and `--sslPEMKeyFile`.

    Example:
    ```
    mongod --sslMode requireSSL --sslPEMKeyFile /path/to/server.pem
    ```

- **Auditing**: MongoDB provides auditing features to track user actions on the database. You can enable auditing to monitor activities such as user authentication, data access, and changes to the database.

  Example:
  ```
  auditLog:
    destination: file
    path: /var/log/mongodb/audit.log
  ```

  This configuration enables auditing and stores the logs in a file.

- **IP Whitelisting**: You can configure MongoDB to allow connections only from specific IP addresses using the `bindIp` option.

  Example:
  ```
  net:
    bindIp: 127.0.0.1,192.168.1.10
  ```

  This restricts access to MongoDB from localhost (`127.0.0.1`) and a specific IP (`192.168.1.10`).

---

<div id="t12" ></div>

## 12. Monitoring and Debugging [↑](#topics-covered)


MongoDB provides a variety of tools and techniques for monitoring the performance of the database, as well as debugging any issues that arise.

- **Monitoring MongoDB**: MongoDB provides several built-in tools for monitoring the health and performance of your MongoDB deployment, including the `mongostat`, `mongotop`, and the MongoDB Atlas monitoring features.

  - **mongostat**: This tool provides statistics about the MongoDB server, including operation counts, memory usage, and network activity.

    Example:
    ```
    mongostat --host localhost:27017
    ```

    This command displays real-time statistics for the MongoDB instance.

  - **mongotop**: This tool provides insights into the time spent on various database operations and collections.

    Example:
    ```
    mongotop --host localhost:27017
    ```

    This command shows the amount of time spent on operations by each collection.

  - **MongoDB Atlas Monitoring**: If you are using MongoDB Atlas, you can access the cloud-based monitoring dashboard, which provides real-time metrics, alerts, and performance diagnostics.

- **Database Profiler**: MongoDB provides the database profiler to track slow queries and database operations. You can use it to log performance issues.

  Example:
  ```
  db.setProfilingLevel(2)
  db.system.profile.find()
  ```

  This enables profiling at level 2 (captures all operations) and queries the `system.profile` collection to retrieve the logs.

- **Logs and Error Handling**: MongoDB logs events to its log files. You can check the logs to diagnose issues related to performance, failures, or errors.

  Example:
  ```
  tail -f /var/log/mongodb/mongod.log
  ```

  This command continuously monitors the MongoDB log file for new entries.

- **Profiler for Specific Query**: You can use the profiler to capture slow queries for specific collections or operations. This is helpful for identifying performance bottlenecks.

  Example:
  ```
  db.setProfilingLevel(1, { slowms: 100 })
  db.system.profile.find({ millis: { $gt: 100 } })
  ```

  This configuration sets the profiling level to log queries that take longer than 100ms.

- **Using Logs for Debugging**: MongoDB logs errors and events to help diagnose issues. By reading the logs, you can identify configuration errors, slow queries, or replication issues.

  Example:
  ```
  tail -f /var/log/mongodb/mongod.log | grep "error"
  ```

  This command filters the MongoDB logs for any errors.

- **Query Debugging with `explain()`**: You can use the `explain()` method to get detailed information about how MongoDB executes queries. This is helpful for optimizing queries.

  Example:
  ```
  db.products.find({ sku: "abc123" }).explain("executionStats")
  ```

  This command explains how MongoDB executes the query to retrieve products with the SKU `abc123`.

----








