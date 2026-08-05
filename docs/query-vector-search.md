# Query with `$vectorSearch`

Use the `$vectorSearch` aggregation stage to find documents whose vector embeddings are closest to a query vector. Percona Search for MongoDB returns the closest matches first.

## Before you begin

Make sure that:

- Percona Search for MongoDB is running and connected to Percona Server for MongoDB.
- The collection contains vector embeddings.
- A vector search index exists for the field you want to query.
- The query vector has the same number of dimensions as the vectors indexed in the collection.
- The query vector was generated with the same embedding model used for the stored embeddings.

To create an index, see [Create a vector search index](create-vector-search-index.md).

## Syntax

```javascript
{
  "$vectorSearch": {
    "exact": true | false,
    "filter": {<filter-specification>},
    "index": "<index-name>",
    "limit": <number-of-results>,
    "model": "<model-name>",
    "numCandidates": <number-of-candidates>,
    "path": "<field-to-search>",
    "query": {
      "text": "<query-text>"
    },
    "searchNodePreference": {
      "key": <preference-string>
    }      
  }
}
```

The `$vectorSearch` stage accepts the following fields:

| Field | Description |
|---|---|
| `index` | Name of the vector search index. |
| `path` | Document field that contains the indexed embeddings. |
| `queryVector` | Vector that represents the search query. |
| `numCandidates` | Number of candidate vectors to consider during an approximate nearest neighbor search. |
| `limit` | Maximum number of documents to return. |
| `filter` | Optional filter that limits which documents are considered. The filtered fields must be included in the index as `filter` fields. |
| `exact` | Set to `true` to run an exact nearest neighbor search. Omit this field for an approximate nearest neighbor search. |

## Run an approximate nearest neighbor query

Approximate nearest neighbor (ANN) search uses the HNSW index to find close matches without comparing the query vector with every indexed vector. This approach is suitable for larger datasets where query speed is important.

Assume that the `queryVector` variable contains the complete embedding generated for the search text. The following query searches the `embedding` field and returns the five closest matches:

```javascript
db.products.aggregate([
  {
    $vectorSearch: {
      index: "products_vector_idx",
      path: "embedding",
      queryVector: queryVector,
      numCandidates: 100,
      limit: 5
    }
  },
  {
    $project: {
      _id: 0,
      name: 1,
      description: 1,
      score: {
        $meta: "vectorSearchScore"
      }
    }
  }
]);
```

The query considers 100 candidate vectors and returns the five closest matches.

A higher `numCandidates` value can improve result accuracy because the search examines more possible matches. It also requires more processing time. As a starting point, set `numCandidates` to at least 20 times the value of `limit`, and then adjust it for your data and performance requirements.

## Check the similarity score

The [$project :octicons-link-external-16:](https://www.mongodb.com/docs/manual/core/aggregation-pipeline-optimization/#-project-stage-placement){:target="_blank"} stage adds the vector search score to each result:

```javascript
score: {
  $meta: "vectorSearchScore"
}
```

The score ranges from `0` to `1`. A higher score means that the document vector is closer to the query vector.

Percona Search for MongoDB calculates the score when the query runs. The score is not stored in the source document.

The example does not return the `embedding` field. Excluding large embedding arrays can reduce the amount of data returned by the query.

## Filter the search

Use the `filter` field to limit which documents are considered before the vector comparison.

The following query searches only products in the `footwear` category:

```javascript
db.products.aggregate([
  {
    $vectorSearch: {
      index: "products_vector_idx",
      path: "embedding",
      queryVector: queryVector,
      filter: {
        category: "footwear"
      },
      numCandidates: 100,
      limit: 5
    }
  },
  {
    $project: {
      _id: 0,
      name: 1,
      category: 1,
      score: {
        $meta: "vectorSearchScore"
      }
    }
  }
]);
```

For this query to work, the vector search index must include `category` as a `filter` field.

Keep filters broad enough to include relevant documents. A restrictive filter can exclude documents that are close to the query vector.

## Run an exact nearest neighbor query

Exact nearest neighbor (ENN) search compares the query vector with every indexed vector that meets the filter conditions. It returns the exact nearest matches but can take longer on large datasets.

Set `exact` to `true` and omit `numCandidates`:

```javascript
db.products.aggregate([
  {
    $vectorSearch: {
      index: "products_vector_idx",
      path: "embedding",
      queryVector: queryVector,
      exact: true,
      limit: 5
    }
  },
  {
    $project: {
      _id: 0,
      name: 1,
      description: 1,
      score: {
        $meta: "vectorSearchScore"
      }
    }
  }
]);
```

ENN search is useful when you need the exact closest matches or when you want to compare the accuracy of ANN results.

## Considerations

- `$vectorSearch` must be the first stage in the aggregation pipeline.
- You cannot run `$vectorSearch` inside a `$facet` stage or a `$lookup` subpipeline.
- You can add stages such as `$project`, `$match`, and `$limit` after `$vectorSearch` to process the results.
- Increasing `numCandidates` can improve ANN accuracy, but it can also increase query latency.
- Exclude the vector field from the returned documents unless your application needs it.

For reference, see [$vectorSearch (aggregation stage) :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/operator/aggregation/vectorsearch/){:target="_blank"} in the MongoDB documentation.