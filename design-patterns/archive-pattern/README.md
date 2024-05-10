# Archive Pattern
- Motivation: Sometimes we have old data that is infrequently access. If you keep this the same way we keep hot data, probably we'll impact the database performance. So, we need a strategy to keep this old data but without impacting the overall performance.
- Use cases:
    - Data complaince that I need to keep for some years.
    - IoT measurements
    - Old logs
- Why to do that?
    - A collection with a large number of inactive documents can impact performance.
    - Large indexes can waste memory.
- The Archive Pattern moves inactive documents out of main database.

## Example
We want to archive reviews that are not frequently used. A way to do that is to archive reviews that are older than 1 year and that haven't received any votes from other users.

Review document:
```json
{
    "date": "",
    "votes": "",
    "reviewRating": "",
    "reviewTitle": "",
    "reviewBody": "",
    "bookId": "",
    "userId": ""
}
```

Some way to archive the document:
- Simply copy the source document to archive: It may be complicate if this document contain reference, because you need to recreate all the hierachy. To solve this, you can use the Extended Reference Pattern to enhance archive document with referenced data.

Then, you must select the store solution to archive data. It depends on your needs, but, MongoDB recommends the use of Online Archive service (on Atlas cluster).

And to finish, the final step is to define a schedule for archiving and deleting documents.

> It's a good idea to archive and delete documents frequently, to avoid performance issues.
