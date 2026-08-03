# Update Index

You can use the `updateSearchIndex()` method to update a Search or Vector Search index. 

!!! warning
    `updateSearchIndex()` replaces the index definition; it does not merge your changes into the existing one. Always submit the complete definition, including the fields you are not changing. Any field you omit is removed from the index.

You cannot rename an index with this method. To rename an index, drop it and create a new one with the desired name.

After you update an index, `mongot` rebuilds it in the background. The index continues to serve queries with the old definition until the rebuild completes. Use `getSearchIndexes()` to check the rebuild status.

## Update single Search Index

The following example updates `search_idx` to index only the text field:
{.power-number}

1. Update the search index:

    ```javascript
    db.docs.updateSearchIndex(
      "search_idx",
      {
        mappings: {
          dynamic: false,
          fields: {
            text: {
              type: "string"
            }
          }
        }
      }
    )
    ```

2. Check the index status:

```javascript
db.docs.getSearchIndexes()
[
  {
    id: '6a5f4e46f961b92133ad8c73',
    name: 'search_idx',
    status: 'READY',
    queryable: true,
    latestDefinitionVersion: { version: 1, createdAt: ISODate('2026-07-21T10:48:12.000Z') },
    latestDefinition: { mappings: { dynamic: false, fields: { text: { type: 'string' } } } },
    statusDetail: [
      {
        hostname: '6a57ac86331e2c0925871f72',
        status: 'READY',
        queryable: true,
        mainIndex: {
          status: 'READY',
          queryable: true,
          definitionVersion: { version: 1, createdAt: ISODate('2026-07-21T10:48:12.000Z') },
          definition: {
            mappings: {
              dynamic: false,
              fields: {
                text: {
                  type: 'string',
                  indexOptions: 'offsets',
                  store: true,
                  norms: 'include'
                }
              }
            }
          }
        }
      },
      {
        hostname: '6a57ac8646c3f951ee8ce224',
        status: 'READY',
        queryable: true,
        mainIndex: {
          status: 'READY',
          queryable: true,
          definitionVersion: { version: 1, createdAt: ISODate('2026-07-21T10:48:12.000Z') },
          definition: {
            mappings: {
              dynamic: false,
              fields: {
                text: {
                  type: 'string',
                  indexOptions: 'offsets',
                  store: true,
                  norms: 'include'
                }
              }
            }
          }
        }
      }
    ]
  }
]
```

[Delete a search index :material-arrow-right:](../delete-search-index.md){.md-button}


