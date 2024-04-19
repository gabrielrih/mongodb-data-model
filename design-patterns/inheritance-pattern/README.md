# Inheritance Pattern

- It provides to us a way to store polymorphic data in the same collection.
- Note that in this case the documents have some fields and common but there are differences between them.
- By using a field to track the shape of documents, applications can handle different kinds of documents within the same collection.
- The ideal is to apply this pattern in the modeling phase, but if you already has a collection, you can use the Aggregation Framework to apply the Inheritance Patten in this existente collection.
- Some use cases for this in when you store in the same collection the following content:
    - Books (ebooks, audiobooks and printed books).
    - Animals (cats, dogs and more).
    - Public servants (fireman, policeman, etc).

## Example

Remember that data that is accessed together should be keep together. So, imagine you have an application that sells books and the application access all kinds of books together. So, you must keep eBooks, printed books and audiobooks in the same collection. In this example, you can use the Inheritance Pattern to classify the book based on its type.

So, you have a _product_type_ field to define the type and then the application can handle correctly the data.

ebook
```json
{
    "title": "Practical MongoDB Aggregations",
    "description": "The complete book of MongoDB by its employees",
    "genres": ["Programming"],
    "publisher": "MongoDB Press",
    "price": 0.00,
    "authors": ["Paul Done"],
    "product_type": "ebook",
    "download_url": "https://fake-url/ebook"
}
```

audiobook
```json
{
    "title": "Practical MongoDB Aggregations",
    "description": "The complete book of MongoDB by its employees",
    "genres": ["Programming"],
    "publisher": "MongoDB Press",
    "price": 0.00,
    "authors": ["Paul Done"],
    "product_type": "audiobook",
    "download_url": "https://fake-url/ebook",
    "narrators": ["Paul Done"],
    "duration": "21 hours 8 minutes"
}
```

printer book
```json
{
    "title": "Practical MongoDB Aggregations",
    "description": "The complete book of MongoDB by its employees",
    "genres": ["Programming"],
    "publisher": "MongoDB Press",
    "price": 0.00,
    "authors": ["Paul Done"],
    "product_type": "printed_book",
    "stockLevel": 130,
    "deliveryTime": "2"
}
```