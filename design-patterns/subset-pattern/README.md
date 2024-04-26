# Subset Pattern
- Motivation:
    - The Working Set, which is the documents and indexes more used, should fit in the WiredTiger Internal Cache (Memory) to guarantee a good performance. When the working set exceeds the working set you can consider using the subset pattern.
- Advantages:
    - It reduces the document size increasing performance (relocating data that is not frequently accessed to other collection).
- Use Cases:
    - Large number of embedded documents, for example millions of reviews for a single book, but only a small subset of documents is actually used by application.

## Example
Imagine we have a "book" collection which has reviews has an array. In this case, we can create another collection to save the reviews but keep some of them in the array on the book collection. Example, you can keep the top ten most relevant reviews on the books collections.

By doing that, you guarantee that when you access the book document it will have 10 reviews to show in the same page (keeping data that is accessed together stored together).

This will lead to duplication of some reviews documents, but it's easy to manage updates because the content of those documents is unlikely to change often.

## Aggregation Framework
This is an example of using aggregation framework to apply the subset pattern in a collection.

Use this to copy data and send to another collection:
```sh
[
  {
    $unwind: {
      path: "$reviews"
    }
  },
  {
    $set: {
      "reviews.product_id": "$product_id"
    }
  },
  {
    $replaceRoot: {
      newRoot: "$reviews"
    }
  },
  {
    $out: "reviews"
  }
]
```

An use this to keep just the first 10 elements of the array in the original collection:
```sh
const update_reviews_array_pipeline = [
   {
       reviews: {
           $slice: ["$reviews", 3]
       }
   }
];
db.books.updateMany({}, update_reviews_array_pipeline);
```
