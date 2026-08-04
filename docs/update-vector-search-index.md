# Update vector search index

You can use the `updateSearchIndex()` method to update vector search index. 

!!! warning
    `updateSearchIndex()` replaces the index definition; it does not merge your changes into the existing one. Always submit the complete definition, including the fields you are not changing. Any field you omit is removed from the index.

You cannot rename an index with this method. To rename an index, drop it and create a new one with the desired name.

After you update an index, `mongot` rebuilds it in the background. The index continues to serve queries with the old definition until the rebuild completes. Use `getSearchIndexes()` to check the rebuild status.


## Update a Vector Search index

Suppose `products_vector_idx` currently indexes the `embedding` field. The following operation adds `category` as a filter field:

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

    1. `mongod` forwards the updated definition to mongot.
    3. `mongot` builds a new version of the index in the background. The existing index remains available for queries while the rebuild is in progress.
    3.  The new index contains the `embedding` vectors and the category values. During a filtered vector query, `category` narrows the documents considered for similarity search. It does not affect the vector similarity score.
    4. When the rebuild finishes, the new index replaces the previous version.

[Delete a search index :material-arrow-right:](../delete-search-index.md){.md-button}


