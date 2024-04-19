# Extended Reference Pattern
- Motivation: Data modeling sometimes involves multiple documents within different collections. In relational databases, we use JOIN to merge data from different tables. On MongoDB, we have the $lookup aggregation operator. However, using this can be resource intensive, much like a join. To avoid this, we can apply the Extended Reference Pattern to our data.
- An "Extended Reference" is a reference that is rich enough to include all that is needed so we can avoid lookups. You do that embedding relevant data from multiple documents and different collections into the main document.
- Advantages:
    - Reduce or avoid lookups (joins).
    - Reduce the latency of read.
    - Increase performance.
- Down sides:
    - Duplication

## Example
Imagine we have three collections - users, books and reviews - and we want to add a new feature allowing to filter the reviews by user, product_type or book title.

Using normalization, the product_type and book title would be on the books collections, and the username would be on the users collections. In this case, we would need to use the $lookup operator to merge the data.

To avoid this, we can add all this data in the review document:

```json
{
    "product": {
        "product_id": "SKU-88727401",
        "product_type": "ebook",
        "title": "Practical MongoDB Aggregations"
    },
    "reviewer": {
        "userId": "MDBU020894",
        "username": "Sue Smith"
    },
    "review": {
        "title": "Amazing story!",
        "date": "2024-01-01T10:45:34-0500",
        "stars": 5,
        "reviewBody": "Such an amazing book"
    }
}
```

Note that in this case we can apply the filter using just fields on the documents from the reviews collection. 

However, it introduce __data duplication__. To reduce this issue, try to minimize duplication selecting fields that do not change often. Also, try to also bring fields you really need. Another important thing is that you need to consider how to __keep duplicate data in sync__ with the source.

To manage duplication when a source field is updated first identify what is the list of dependent extended references.
- What is the list of dependent extended references? What other fields need to be updated once the source gets updated? The application must know this references.
- Do the extended references need to be updated immediately or can they be updated at a later time? A scheduled task can be configured to update the document with extended references periodically instead of doing it every time the original referenced document is updated.

## Implementing this pattern on an existente dataset
- Aggregation framework.

Example of bringing the product data to the review collection.
```
var reshape_review_docs_pipeline = [
  {
    $lookup: {
      from: "books",
      localField: "product_id",
      foreignField: "product_id",
      as: "product_info",
    },
  },
  {
    $unwind: {
      path: "$product_info",
      includeArrayIndex: "string",
      preserveNullAndEmptyArrays: false,
    },
  },
  {
    $project: {
      _id: "$_id",
      "product.product_id": "$product_id",
      "product.product_type":
        "$product_info.product_type",
      "product.title": "$product_info.title",
      "review.user_id": "$user_id",
      "review.reviewTitle": "$reviewTitle",
      "review.reviewBody": "$reviewBody",
      "review.date": "$date",
      "review.stars": "$stars",
    },
  },
  {
    $merge: {
      into: "reviews",
      on: "_id",
      whenMatched: "replace",
      whenNotMatched: "discard",
    },
  },
]

db.reviews.aggregate(reshape_review_docs_pipeline)
```