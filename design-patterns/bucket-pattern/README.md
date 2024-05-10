# Bucket Pattern
- Motivation:
    - It's a good option when embedding would result in very large documents and unbounded arrays and the reference option would result in poor performance.
    - When working with real data, for example the temperature from a sensor, you could want to create a new document for every single read made by the sensor. But imagine how many documents it would have if the measurement is taken every one minute. Besides, imagine if you want to create an index for the sensor_id and timestamp field, the cardinality is to high so the index would be too big.
    - To avoid this you can create and start_datetime and end_datetime on each document and put all the measurements between that time in that same document. For example:

```json
{
    sensor_id: 12345,
    start_date: ISODate("2019-01-31T10:00:00.000Z"),
    end_date: ISODate("2019-01-31T10:59:59.000Z"),
    measurements: [
       {
       timestamp: ISODate("2019-01-31T10:00:00.000Z"),
       temperature: 40
       },
       {
       timestamp: ISODate("2019-01-31T10:01:00.000Z"),
       temperature: 40
       }
    ],
   transaction_count: 2,
   avg_temperature: 40
} 
```
- Advantages:
    - You can precompute values using the computed pattern to reduce time in reading operations.
    - It lets our application scales in terms of data and index size.
    - It keeps document size predictable (avoiding unbounded arrays).
    - And you read only data you need.
    - Reduce number of documents in a collection.

- Use cases:
    - IoT / Time Series data
    - Real-Tiem Analytics data

## Example
Another example, looking for a book store, we may want to know the user views for each bucket. In this case, lets suppose our application wants to know the reviews by month. So, we can create a different document for each book and month, and create and array to save the user views.

```json
id: {
    book_id: 314500,
    month: ISODate("2024-04-01T00:00:00.000Z")
},
views: [
    {
        timestamp: ISODate("2024-04-10T15:30:01.000Z"),
        user_id: 100,
    },
    {
        timestamp: ISODate("2024-04-22T06:45:54.000Z"),
        user_id: 101,
    },
]
```

> Note here that we must have a predictable array size, so a strategy you can use in the application is to create a new document every time the number of views in the documents reach a certain size. This way you avoid unbounded arrays.

To analyse the quantity of views for a single book in a single month you could use this command here:
```sh
db.getCollection("views").find(
    {
        "id.book_id": 314500,
        "id.month": ISODate("2024-04-01T00:00:00.000Z")
    },
    {
        _id: 0,
        views_count: { $size: "$views" }
    }
)
```

[MongoDB Blog Reference](https://www.mongodb.com/blog/post/building-with-patterns-the-bucket-pattern)
