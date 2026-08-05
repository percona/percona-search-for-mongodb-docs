# Update vector search index

You can use the [db.collection.updateSearchIndex() :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/method/db.collection.updateSearchIndex/){:target="_blank"} method to update a vector search index.

!!! warning
    `updateSearchIndex()` replaces the index definition; it does not merge your changes into the existing one. Always submit the complete definition, including the fields you are not changing. Any field you omit is removed from the index.

You cannot rename an index with this method. To rename an index, drop it and create a new one with the desired name.

After you update an index, `mongot` rebuilds it in the background. The index continues to serve queries with the old definition until the rebuild completes. Use `getSearchIndexes()` to check the rebuild status.

## Update a vector search index

```javascript
db.<collection>.updateSearchIndex(
  <name>,
  {
    <definition>
  }
)
```

`updateSearchIndex()` accepts these fields:

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `name` | `string` | Required | Name of the index to update. |
| `definition` | `document` | Required | The complete replacement definition for the index. This replaces the existing definition; it is not merged with it. |

??? example "Add `category` as a filter field to `products_vector_idx`"

    Suppose `products_vector_idx` currently indexes the `embedding` field. The following operation adds `category` as a filter field.

    ```javascript
    db.products.updateSearchIndex(
      "products_vector_idx",
      {
        fields: [
          {
            type: "vector",
            path: "embedding",
            numDimensions: 768,
            similarity: "cosine"
          },
          {
            type: "filter",
            path: "category"
          }
        ]
      }
    );
    ```

??? info "What happens under the hood"
    **When you update the index**
    {.power-number}

    1. `mongod` forwards the updated definition to `mongot`.
    2. `mongot` builds a new version of the index in the background. The existing index remains available for queries while the rebuild is in progress.
    3. The new index contains the `embedding` vectors and the `category` values. During a filtered vector query, `category` narrows the documents considered for similarity search. It does not affect the vector similarity score.
    4. When the rebuild finishes, the new index replaces the previous version.


## Next steps

[Delete a search index :material-arrow-right:](delete-vector-search-index.md){.md-button}