# Computed Pattern

- Motivation: Frequent computations can decrease performance.
- This pattern allows you to run expensive operations when the data changes and store the results for quick access.
- Common computations:
    - Mathematical
    - Roll-up

## Mathematical operations
- The goal is to pre-compute the result on writes instead of calculate it on every read.
- Imagine calculating the review rating from a book which has millions of reviews on every book read? This would be costly and probably slow. A better option is to calculate the average rating when a new review is created and stored this info on the book.

### Example
books collections
```json
{
    "title": "Practical MongoDB Aggregations",
    "description": "The complete book of MongoDB by its employees",
    "genres": ["Programming"],
    "publisher": "MongoDB Press",
    "price": 0.00,
    "authors": ["Paul Done"],
    "product_type": "ebook",
    "download_url": "https://fake-url/ebook",
    "rating": {
        "reviewCount": 354894,
        "avgRating": 4.79841
    }
}
```

In the example above, the rating subdocument is update on every new write on the review collection.
```
(avgRating * reviewCount + newRating) / (reviewCount + 1)
```

## Roll-Up operations
- Merge and group data.
- It allows us to view data as a whole.

### Example
The stakeholders wants to know the total number of books by product_type and the average number of authors.
Instead of compute this information every time it's requested, we can store this information in a new document and update it at specific intervals (daily for example).

This way we reduce the overload running a pipeline once a day, instead of running it on every write.

```
var roll_up_product_type_and_number_of_authors_pipeline = [
  {
    $group: {
      _id: "$product_type",
      count: {
        $sum: 1,
      },
      averageNumberOfAuthors: {
        $avg: {
          $size: "$authors",
        },
      },
    },
  },
]
```

```
db.books.aggregate(roll_up_product_type_and_number_of_authors_pipeline)
```

Result:
```
[
  { _id: 'audiobook', count: 1, averageNumberOfAuthors: 1 },
  { _id: 'ebook', count: 1, averageNumberOfAuthors: 3 },
  { _id: 'book', count: 1, averageNumberOfAuthors: 3 }
]
```