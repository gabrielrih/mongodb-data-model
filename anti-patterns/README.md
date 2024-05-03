# Anti Patterns

## Index
- [Unbounded arrays](#unbounded-arrays)
- [Bloated Documents](#bloated-documents)
- [Massiver Number of collections](#massive-number-of-collections)
- [Unnecessary indexes](#unnecessary-indexes)
- [Data Normalization](#data-normalization)
- [Case-Sensitivity](#case-sensitivity)

### Unbounded arrays
- An unbounded array is a large, growing array with an unlimited number of elements.
- What to avoid it?
    - We have the risk to exceed the BSON document size limit of 16MB.
    - We can also have a decrease in the index performance.
- What to consider:
    - Only store data together if it is queried together.
    - An array should not grow without bound.
    - High cardinality arrays should not be embedded.
- Design patterns that can help us to solve this: __Extended Reference__ and the __Subset__ patterns.

### Bloated Documents
- When you starts to add mode fields in the document instead of refactoring the schema, we can end up with a bloat document antio pattern.
- When all data related to an entity is stored together in a single document, despite its access pattern, we are bloating the document. So, bloated documents store data that is accessed separately in a single document.
- It's a problem because it increases the size of the working set and eventually impact performance.
- How to reduce performance impact?
    - Increase cluster memory: To fit more data in the WiredTiger internal cache.
    - Estimate the working set size and update the data model to use less memory.
- Example: let's suppose we have a Books collections where each document represents all the data from a book. In addition, our app has to pages: a homepage that loads just the book name and author, and another page that displays the details of a single book. In this case, it doesn't make sense to store all the books details in a single collections because some data is access in the home page (top 10 books for example) and the other details are accessed in the details web page. This way we can split this documents in two different collections: Summary and Details.

### Massive Number of Collections
- In theory, the recommended limit for the number of collections in a replica set is 10,000. In real world, it depends on your workload and database resources.
- When you have many collections it may impact the overall performance. This is because WiredTiger stores a separate file for each collection and each index. 
- On MongoDB Atlas, using a Replica Set, you can start to have performance issue when you have more than:
    - 5,000 collections on M10.
    - 10,000 collections on M20/M30.
- When using Shard, it can handle up to 10,000 collections per shard.
- How to avoid this problem?
    - Drop old and unused collections frequently (we recommend to monitor the number of collections).
    - Update schema design.
    - Sharding your database (increase the number of collections handles by each shard).

### Unnecessary Indexes
- We recommend to create just the necessary indexes.
- Indexes impact write performance.
- Indexes use more memory and require more disk space.
- You can use this pipeline to check the usage of an index:

```js
var sortIndexesByAccessOpsPipeline =  [
    { $indexStats: {} },
    { $project: { 
        _id: 0,
        name: "$name",
        ops: "$accesses.ops",
        since: "$accesses.since" }
    },
    { $sort: { ops: 1 } }
]
```

- Before dropping an index we recommend you to hide this index. It's easy to enable if you need rollback.

### Data Normalization
- It happens when you separate data that is accessed together into different collections.
- You'll need to use the ```$lookup``` operator and it can be costly.
- How to avoid this problem? By using the [Subset pattern](../design-patterns/subset-pattern/README.md) and/or the [Extended Reference pattern](../design-patterns/extended-reference-pattern/README.md).

### Case-Sensitivity
- When the upper and the lower letters are treated differently, this is known as case sensitivity.
- By default, MongoDB queries are case sensitive.
- When you use the default settings of MongoDB, but expect search terms to ignore case, it's known as case sensitivity antipattern.
- Which problems it may result?
    - Unexpected results.
    - Reduce performance.
- How do I configure MongoDB to support case insensitive queries? Using __collations__.
- A collation has locale and strength (from 1 to 5). The default strength value is 3, which makes it case sensitive. __To make the comparison case insensitive we need to set eh strength to 1 or 2__.
- How to implement it?
    - You can build an index with a given collation. And then, you specify this same collation in the query.
    - You can also assign a default collation to a collection when you create it (the collation cannot be changed). The queries triggered through this collection will also use this same collation.
- What not to do?
    - Using the ```$regex``` operator. It's well supported on MongoDb when used for exact matches, but not for case insensitive.
- Example:

Create an index specifying a collation:
```js
db.Books.createIndex(
   { author: 1 },
   {
       collation: {
           locale: 'en',
           strength: 2
       }
   }
)
```

.. and then you run the query using this same collation:
```js
db.Books.find({ author: 'Paul Done' }).collation({locale: 'en', strength: 2})
```

> IMPORTANT: Remember that if your query is case insensitive does should be your index. Otherwise, MongoDB will performs a COLLSCAN in the collection.
