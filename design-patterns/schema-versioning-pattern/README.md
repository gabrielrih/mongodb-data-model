# Schema versioning pattern
- Motivation: Data schema changes and dealing with these changes can be stressful. To help us and to be possible to apply new schema without downtime, the schema versioning pattern was created.
- Advantages:
    - Apply schema changes with no downtime.
    - Documents with different schema version can exist in the same collection.
    - We have the flexiblity to decide when and how to apply schema updates on the documents.
- How it works?
    - You add a new field in each document to represent the schema version. This new field is __schema_version__. The absence of a schema version field is an implicit version 1.
    - Then your application should be prepare to handle multiple schemas.

## How to update existing documents?
We have some options:
- Let the application update the shape when the document is accessed.
- Have a background task performing the updates on all documents.

The timeline migration for the background task could be:
```
Upgrade the application version > Run the background task to migrate schema version on all documents > Remove code to handle old schema version
```