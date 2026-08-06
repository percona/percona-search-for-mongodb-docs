## Create a search index

Before you can run full-text search queries, create a search index on the collection.

Use the [db.collection.createSearchIndex() :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/method/db.collection.createsearchindex/#mongodb-method-db.collection.createSearchIndex){:target="_blank"} method to create a search index.

Follow these steps:
{.power-number}

1. Create a search index.

    ```javascript
    db.<collection>.createSearchIndex(
      "<name>",
      "<type>",
      {
        <definition>
      }
    )
    ```

    `createSearchIndex()` accepts the following parameters:

    | Field | Type | Required | Description |
    |-------|------|----------|-------------|
    | `name` | `string` | No | Name of the index. Defaults to `default` if omitted. |
    | `type` | `string` | No | Index type. Specify `search` or `vectorSearch`. The default is `search`. |
    | `definition` | `document` | Yes | Defines the index configuration. |

    ??? example "Example: Create a search index"

        ```javascript
        db.docs.createSearchIndex({
          name: "search_idx",
          definition: {
            mappings: {
              dynamic: true
            }
          }
        })
        ```

        **Output**

        ```text
        search_idx
        ```

2. Verify that the index is ready.

    ```javascript
    db.docs.getSearchIndexes()
    ```

    Index creation runs asynchronously. Wait until the index status is `READY` before running search queries.

    **Output**

    ```javascript
    [
      {
        name: "search_idx",
        status: "READY",
        queryable: true,
        ...
      }
    ]
    ```

3. Run a full-text search.

    ??? example "Search the collection"

    ```javascript
    db.docs.aggregate([
      {
        $search: {
          index: "search_idx",
          text: {
            query: "future",
            path: "text"
          }
        }
      }
    ])
    ```

    **Output**

    ```javascript
    [
      {
        text: "Vector search is the future"
      }
    ]
    ```

## Next steps

[Update a search index :material-arrow-right:](update-search-index.md){.md-button}

[Delete a search index :material-arrow-right:](delete-search-index.md){.md-button}

## Learn more

[$vectorSearch aggregation stage :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/operator/aggregation/vectorSearch/){:target="_blank"}