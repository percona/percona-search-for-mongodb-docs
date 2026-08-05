# Delete vector search index

Use the `dropSearchIndex()` method to delete a vector search index.

**Syntax**

```javascript
db.<collection>.dropSearchIndex(<name>)
```

`dropSearchIndex()` accepts this field:

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `name` | `string` | Required | Name of the search index to delete. |

??? example "Delete vector search index named `vector_idx`"

    ```javascript
    db.products.dropSearchIndex("vector_idx");
    ```

Deleting an index is irreversible. To restore search functionality, create the index again with `createSearchIndex()`, and wait for the initial sync to
complete before running queries against it.

!!! warning
    `$vectorSearch` queries that reference an index that does not exist return an empty result set rather than an error. If your application starts receiving empty search results after an index change, verify that the index exists and check its status with `getSearchIndexes()`.

For the complete `dropSearchIndex()` reference, see [db.collection.dropSearchIndex() :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/method/db.collection.dropSearchIndex/){:target="_blank"} in the MongoDB documentation.