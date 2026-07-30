# Query with $search

The `$search` aggregation stage performs **full-text search** on fields covered by a search index. It returns documents ordered by relevance, with the most relevant document returned first.

## Before you begin

- `mongot` is running and connected to your Percona Search for MongoDB deployment.
- A search index exists for the collection.
- The index includes the fields you want to search.

## General syntax for $search

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
      returnStoredSource: true | false,
      searchNodePreference: {
        key: "<preference-string>"
      }
    }
  }
  ```

The `$search` stage accepts the following fields:

| Field  | Type    | Requirement | Description |
| -------| ------- | ----------- | ------------|
| `index`| String  | Optional    | Specifies the Search index. The default value is `default`.|
| `<operator-name>`| Object  | Conditional | Specifies an operator such as `text`, `phrase`, or `compound`. Either an operator or a collector is required.|
| `<collector-name>`     | Object  | Conditional | Specifies a collector and its options. Either a collector or an operator is required.|
| `highlight`            | Object  | Optional    | Returns matching search terms in their original context.|
| `concurrent`           | Boolean | Optional    | Runs the search across index segments concurrently. The default value is `false`.|
| `count`                | Object  | Optional    | Returns count metadata for the matching documents.|
| `searchAfter`   | String  | Optional    | Returns results after the specified pagination token. It cannot be combined with `searchBefore`.|
| `searchBefore`         | String  | Optional    | Returns results before the specified pagination token. It cannot be combined with `searchAfter`. |
| `scoreDetails`         | Boolean | Optional    | Returns detailed scoring information. The default value is `false`.|
| `sort`| Object  | Optional    | Sorts results by score or supported indexed fields.|
| `returnScope`          | Object  | Optional    | Sets the query context to an embedded document field. Requires `returnStoredSource: true`. |
| `returnStoredSource`   | Boolean | Conditional | Returns stored source fields directly from the Search index. The default value is `false`.|
| `searchNodePreference` | Object  | Optional    | Routes repeated queries to a preferred Search node when possible. This is an upstream Atlas-specific option whose PSMDB support should be verified. |


??? example "Search a text field"
    
    This $search stage uses the `products_text_idx` index to search the `name` field in the `products` collection for documents matching either `athletic` or `footwear`:

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

??? example "Search multiple indexed fields"
  
    To search multiple indexed fields, specify an array of field names in path. The following query searches the name and description fields:

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

For the complete syntax and available options, see the [upstream MongoDB documentation :octicons-link-external-16:](https://www.mongodb.com/docs/search/query/aggregation-stages/search/){:target="_blank"} for the $search aggregation stage.