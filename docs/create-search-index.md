## Create a search index

Before you can run full-text search queries, create a search index on the collection.

Use the `createSearchIndex()` method to create a search index.

Follow these steps:
{.power-number}

1. Insert sample documents into the collection from `mongosh`.

    ```javascript
    db.<collection>.insertMany(
      [
        <document1>,
        <document2>,
        ...
      ],
      {
        <options>
      }
    )
    ```

    `insertMany()` accepts the following parameters:

    | Field | Type | Required | Description |
    |-------|------|----------|-------------|
    | `documents` | Array of documents | Yes | Documents to insert into the collection. |
    | `options` | Document | No | Additional options, such as `ordered` and `writeConcern`. |

    ??? example "Example: Insert sample documents"

        ```javascript
        use test

        db.docs.insertMany([
          { text: "MongoDB search is powerful" },
          { text: "Vector search is the future" },
          { text: "Full text search with mongot" }
        ])
        ```

        **Output**

        ```javascript
        {
          acknowledged: true,
          insertedIds: {
            "0": ObjectId(...),
            "1": ObjectId(...),
            "2": ObjectId(...)
          }
        }
        ```

2. Create a search index.

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

3. Verify that the index is ready.

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

4. Run a full-text search.

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

    [Update a search index](update-search-index.md){.md-button}

    [Delete a search index](delete-search-index.md){.md-button}