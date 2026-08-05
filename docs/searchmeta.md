# $searchMeta

Use the `$searchMeta` aggregation stage to retrieve metadata about a search query instead of the matching documents. This is useful when your application needs information such as the total number of matching documents or facet results for building filters and navigation, without returning the search results themselves. Unlike `$search`, which returns matching documents, `$searchMeta` returns a single metadata document and must be the first stage in the aggregation pipeline.

Typical use cases include:

* Returning the total number of documents that match a search query.
* Generating facet counts for fields such as category, brand, or tags.
* Retrieving search metadata to support pagination and search-driven user interfaces.

```javascript
db.<collection>.aggregate([
  {
    $searchMeta: {
      index: "<index-name>",
      <operator>: {
        <operator-specification>
      },
      count: {
        type: "<total-or-lowerBound>"
      },
      facet: {
        operator: { <operator-specification> },
        facets: { <facet-name>: { <facet-specification> } }
      }
    }
  }
])
```

`$searchMeta` accepts these fields:

| Field | Required | Description |
| ----- | -------- | ----------- |
| `index` | Optional | Name of the search index to query. Defaults to `default`. |
| `<operator>` | Required | One or more query operators, such as `text` or `range`, that define which documents are considered. |
| `count` | Optional | Returns the number of matching documents. Accepts `type` (`total` for an exact count, or `lowerBound` for a faster approximate count, with an optional `threshold`). |
| `facet` | Optional | Returns counts grouped by field values. Takes an `operator` to define the query and one or more named `facets`. |

## Count the number of matching documents

Use `$searchMeta` with the `count` option to return the total number of documents that match the search query.

??? example "Count movies matching 'adventure'"

    ```javascript
    db.movies.aggregate([
      {
        $searchMeta: {
          index: "default",
          text: {
            query: "adventure",
            path: "plot"
          },
          count: {
            type: "total"
          }
        }
      }
    ])
    ```

    Output:

    ```javascript
    [
      {
        count: { total: Long('128') }
      }
    ]
    ```

For the more details, see [$searchMeta (aggregation stage) :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/operator/aggregation/searchmeta/){:target="_blank"} and [Count MongoDB Search Results :octicons-link-external-16:](https://www.mongodb.com/docs/search/query/counting/){:target="_blank"} in the MongoDB documentation.