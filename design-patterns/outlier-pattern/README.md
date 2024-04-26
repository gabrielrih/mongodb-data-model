# Outlier pattern
- Motivation: Sometimes we can model a document and put an array in that because this array will unlikely increase that much. But, what if we have an outlier document that starts to have many and many elements in this same array? See that our modeling decision fits almost 100% of the cases, there is just some outlier that doesn't fit this model. So, what to do?
- This pattern allows us to treat documents with unusual characteristics differently.
- Use cases:
    - You can use it when some documents are different and optimizing for all the documents would degrade the performance.
    - This is very common when handling popular itens in a eCommerce for example, which can have millions of comments.

## Example
Let's suppose we want to treat has outlier every book document which reachs the number of 1.000 comments
```json
{
  "rating": 1,
  "userId": 318,
  "productId": 34538756,
  "review_text": "Lorem ipsum dolor sit amet...",
  "comments": [{...}, {...}, ... ]
}
```

In this case you can first mark this document has outlier. An way to do that is to include a field called _outlier_ using the value _true_. After that you can decide how to treat the data based on your application: you can create the next documents in a separete collection, you can use the bucket pattern to split this comments in more then one document within the same collection. It all depends on your application.

> Note here that your application must treat it.

[MongoDB Blog Reference](https://www.mongodb.com/blog/post/building-with-patterns-the-outlier-pattern)
