# Query with $search

The `$search` aggregation stage performs full-text search on fields covered by a search index. It returns documents ordered by relevance, with the most relevant document returned first.

`$search` is available for Percona Search for MongoDB running alongside Percona Server for MongoDB 8.3 and later.

## Before you begin

- `mongot` is running and connected to your Percona Search for MongoDB deployment.
- A search index exists for the collection.
- The index includes the fields you want to search.

## Syntax

```javascript
{
  $search: {
    index: "<index-name>",
    "<operator-name>": {
      <operator-options>
    },
    highlight: {
      <highlight-options>
    },
    concurrent: true | false,
    count: {
      <count-options>
    },
    searchAfter: "<encoded-token>",
    scoreDetails: true | false,
    sort: {
      "<field>": 1 | -1
    },
    returnScope: {
      path: "<embedded-document-field>"
    },
    returnStoredSource: true | false
  }
}
```

The `$search` stage accepts the following fields:

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `index` | String | Optional | Specifies the search index. Defaults to `default`. |
| `<operator-name>` | Object | Conditional | Specifies an operator such as `text`, `phrase`, or `compound`. Either an operator or a collector is required. |
| `<collector-name>` | Object | Conditional | Specifies a collector and its options. Either a collector or an operator is required. |
| `highlight` | Object | Optional | Returns matching search terms in their original context. |
| `concurrent` | Boolean | Optional | Runs the search across index segments concurrently, on dedicated search nodes. If your deployment has no dedicated search nodes, this flag is ignored. Defaults to `false`. |
| `count` | Object | Optional | Requests count metadata for the matching documents. With `$search`, this metadata isn't returned directly — see [Metadata results](#metadata-results) below. |
| `searchAfter` | String | Optional | Returns results after the specified pagination token. Mutually exclusive with `searchBefore`. |
| `searchBefore` | String | Optional | Returns results before the specified pagination token. Mutually exclusive with `searchAfter`. |
| `scoreDetails` | Boolean | Optional | Returns a detailed breakdown of the relevance score. Defaults to `false`. To read the details, use the `$meta` expression in a `$project` stage after `$search`. |
| `sort` | Object | Optional | Sorts results by score or by supported indexed fields. |
| `returnScope` | Object | Optional | Sets the query context to an embedded document field. Requires `returnStoredSource: true`. |
| `returnStoredSource` | Boolean | Conditional | Returns stored source fields directly from the search index instead of performing a full document lookup. Defaults to `false`. Must be `true` if you specify `returnScope`. |

<!-- TBD-ENG: searchNodePreference was removed from this page. It's documented upstream as an Atlas-specific option for routing queries to the same dedicated search node for result consistency. If Percona Search for MongoDB adds equivalent support in the future, reintroduce it here with confirmed behavior. -->

## Metadata results

`$search` returns only the matching documents. Any metadata you request with `count` or a `facet` collector is stored separately, in the `$$SEARCH_META` aggregation variable, rather than added to the results themselves. To read it, reference `$$SEARCH_META` in a later stage, such as `$project` or `$facet`.

If you only need the metadata and not the documents, use [`$searchMeta`](searchmeta.md) instead — it's simpler and avoids the extra step of extracting `$$SEARCH_META`.

## Behavior

- `$search` must be the first stage in the aggregation pipeline.
- `$search` cannot be used in a view definition or inside a `$facet` pipeline stage.

## Operators

Search operators define the conditions that `$search` applies to indexed fields. Each operator supports a specific type of query, such as matching text, checking a value range, or combining several search conditions. The queried fields must be included in the search index with compatible field types.

??? example "Example: Search a text field"

    This `$search` stage uses the `products_text_idx` index to search the `name` field in the `products` collection for documents matching either `athletic` or `footwear`:

    ```javascript
    {
      $search: {
        index: "products_text_idx",
        text: {
          query: "athletic footwear",
          path: "name"
        }
      }
    }
    ```

    The `text` operator analyzes the query text and compares it with the indexed values in the specified field.

??? example "Example: Search multiple indexed fields"

    To search multiple indexed fields, specify an array of field names in `path`. The following query searches the `name` and `description` fields:

    ```javascript
    {
      $search: {
        index: "products_text_idx",
        text: {
          query: "athletic footwear",
          path: ["name", "description"]
        }
      }
    }
    ```

??? example "Example: Combine multiple operators"

    The `compound` operator combines multiple search conditions in a single query. Its clauses determine which conditions are required (`must`), which conditions are required but excluded from scoring (`filter`), which conditions are optional and boost the [relevance score :octicons-link-external-16:](https://www.mongodb.com/docs/search/query/score/overview/){:target="_blank"} (`should`), and which conditions must not match (`mustNot`).

    ```javascript
    {
      $search: {
        index: "products_text_idx",
        compound: {
          must: [
            {
              text: {
                query: "shoes",
                path: "name"
              }
            }
          ],
          should: [
            {
              text: {
                query: "waterproof",
                path: "description"
              }
            }
          ],
          filter: [
            {
              range: {
                path: "price",
                gte: 50,
                lte: 200
              }
            }
          ]
        }
      }
    }
    ```

    The clauses work as follows:

    - `must` requires the `name` field to match `shoes`. This match contributes to the relevance score.
    - `should` gives a higher score to products whose `description` contains `waterproof`. This condition is optional.
    - `filter` restricts the results to products priced from 50 through 200, inclusive. It does not affect the relevance score.

    The `products_text_idx` index must cover the `name`, `description`, and `price` fields with the appropriate field types.

For more information, see the upstream MongoDB documentation for the [compound operator :octicons-link-external-16:](https://www.mongodb.com/docs/search/query/operators-collectors/compound/){:target="_blank"} and [range operator :octicons-link-external-16:](https://www.mongodb.com/docs/search/query/operators-collectors/range/){:target="_blank"}.