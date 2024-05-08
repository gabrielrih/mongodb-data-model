# Schema Lifecycle Management

## Index
- [Schema validation](#schema-validation)
- [Schema evolution](#schema-evolution)
- [Schema migration](#schema-migration)

## Schema validation
- You can use schema validation to force an schema on a collection.
- Concepts: Validation rules, validation levels and validation actions.

### Validation rules
- Here you can specify for example the required fields in a document or even the data type for a given field.    
- We can create the validation rules when creating a new collection. MongoDB will validate the schema every time a document is inserted or updated.
- If you add validation rules on an existing collection, only new inserted documents are checked for validation. MongoDB also check the schema in the updates here.
- To define validation rules you can use _Query Operators_ or the _JSONSchema_ standard (using the ```$jsonSchema``` operator).
- You can see an example above using query operators to define that a certain field cannot be less than another field and you we also use the $jsonSchema to define a type for a certain field.

```js
db.createCollection("sales", {
    validator: {
      "$and": [
        {
          "$expr": {
            "$lt": ["$items.discountedPrice", "$items.price"]
          }
        },
        {
          "$jsonSchema": {
            "properties": {
              "items": { "bsonType": "array" }
            }
           }
         }
       ]
     }
   }
 )
```

### Validations levels
- What happens to existing documents in the collection after we define schema validation rules? That's why the validation levels exist.
- Validation level controls the enforcement of a schema.
- We have two levels: STRICT and MODERATE.
- __Strict (Default)__: It applies rules to all inserted and updated documents.
- __Moderate__: It only applies rules to new and existing valid documents. In this case, invalid documents that existed before the validation was set are not checked for validity when they are updated. 
- If you want to identify the documents that are not complaince with the schema, you can use this command:

```js
db.collection.find( { $nor: [ validator ] })
```

### Validation actions
- Here you define what happens when a document fails validation.
- The options are: ERROR and WARM.
- __Error (Default)__: Reject update or insert operation.
- __Warm__: Allows operation to proceed by a validation failure is recorded on the log.

### Putting all together
Example putting all pieces together

```js
const bookstore_reviews_default = {
    bsonType: "object",
    required: ["_id", "review_id", "user_id", "timestamp", "review", "rating"],
    additionalProperties: false,
    properties: {
        _id: { bsonType: "objectId" },
        review_id: { bsonType: "string" },
        user_id: { bsonType: "string" },
        timestamp: { bsonType: "date" },
        review: { bsonType: "string" },
        rating: {
            bsonType: "int",
            minimum: 0,
            maximum: 5,
        },
        comments: {
            bsonType: "array",
            maxItems: 3,
            items: {
                bsonType: "object",
            },
        },
    },
};
```

```js
const schema_validation = { $jsonSchema: bookstore_reviews_default }

db.runCommand({
    collMod: "reviews",
    validator: schema_validation,
    validationLevel: "strict",
    validationAction: "error",
});
```

## Schema evolution
- The structure of a database will change over time. This process is know as schema evolution.
- We can gradually evolve the schema. This is possible because documents with differents shapes can coexist within the same collection. In this cases you can use schema validation to check schema consistency in new documents.
- Monitoring schema evolution is important to identify schema design issues, improve data quality and optimizing performance.
- Monitoring the logs is an essential element of any strategy, especially if the validation action is set to warm. But, it's no easy to handle. MongoDB Atlas has Schema Suggestions.

## Schema migration
- MongoDB allows to multiple schema versions to coexist within the same collection.
- Versioning is a common strategy to enable controlled migrations.
- Working with a large number of versions increases complexity. So, you should limit the number of schema versions to the minimum required.
- To migrate to the new version of our schema, there are several strategies that we can use:
  - __Eager__: all at once.
  - __Lazy__: changes are implemented as data is used.
  - __Incremental__: where we take small steps to implement changes.
  - __Predictive__: updates the schema based on predictions for future data usage.
- To validate schema using multiples schemas we can use the __oneOf__ in the schema validation rules. For example,

```js
const schema_migration_validation = {
  $jsonSchema: {
    oneOf: [
      bookstore_reviews_default,
      bookstore_reviews_international
    ]
  }
}

db.runCommand({
  collMod: "reviews",
  validator: schema_migration_validation,
  validationLevel: "strict",
  validationAction: "error"
});

```

> In this case, the variables bookstore_reviews_default and bookstore_reviews_international represents schema validation for each schema version.
