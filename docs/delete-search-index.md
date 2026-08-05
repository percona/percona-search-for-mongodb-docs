# Delete search index

Use the `dropSearchIndex()` method to delete a full-text search index.

```javascript
db.<collection>.dropSearchIndex(<name>)
```

`dropSearchIndex()` accepts this field:

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `name` | `string` | Required | Name of the search index to delete. |

**Example**

```javascript
db.products.dropSearchIndex("products_text_idx");
```

Deleting an index is irreversible. To restore search functionality, create the index again with `createSearchIndex()`, and wait for the initial sync to
complete before running queries against it.

!!! warning
    `$search` queries that reference an index that does not exist return an empty result set rather than an error. If your application starts receiving empty search results after an index change, verify that the index exists and check its status with `getSearchIndexes()`.