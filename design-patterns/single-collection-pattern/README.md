# Single collection pattern
- Motivation:
    - Many-to-many or one-to-many relationship.
    - Avoid data duplication and embedding.
- This pattern groups related documents of different types into a single collection.
- Advantages:
    - Avoid multiple queries to read non embedded related documents.
    - Avoid $lookup operations.
- Use cases:
    - Many-to-many relationships (when embedding isn't a good option).
    - One-to-many relationships (catalog of products, shopping cart, etc).
- We have two implementation variantes:
    - Many-to-many: Use a docType field to specify the type of the document and a relatedTo array field (to put ids of the related documents).
    - One-to-many: Use a overloaded field.

## Example for many-to-many
Instead of three collections: Books, Reviews and Users.

You can create a single collection called "books_catalog" and use the fields "docType" and "relatedTo".

![](../../.docs/img/single-collection-pattern-1.png)

To search for documents you create a index on the "relatedTo" field and just run a query searching for a specific book_id on the relatedTo field. In this case, all the documents will be returned, no matter if it's a book, review or user.

## Example for one-to-many
Instead of two collections: Books and Reviews.

You can create a single collection called "books_catalog" and use a overload field (in this case the sc_id field).

![](../../.docs/img/single-collection-pattern-2.png)

You can query it used regex to get all the documents or just the review ones.

```
db.books_catalog.find({"sc_id": {"$regex": "^202367"}})
```