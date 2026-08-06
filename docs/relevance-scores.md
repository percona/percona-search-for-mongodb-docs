# Relevance scores

Relevance scores in MongoDB are numerical values that indicate how well a document matches a search query. These scores are particularly useful in search applications, where ranking results by relevance is critical for providing meaningful and accurate responses to user queries. The score is influenced by factors such as the frequency and position of the search terms, the configured [analyzer :octicons-link-external-16:](https://www.mongodb.com/docs/search/index/analyzers/overview/){:target="_blank"}, and the search operator. Higher scores indicate stronger matches, and results are returned from the highest score to the lowest.

The following query searches the `name` and `description` fields in the `products` collection for documents matching `athletic footwear`. It returns up to `ten` results and includes their relevance scores:

```javascript
db.products.aggregate([
  {
    $search: {
      index: "products_text_idx",
      text: {
        query: "athletic footwear",
        path: ["name", "description"]
      }
    }
  },
  {
    $limit: 10
  },
  {
    $project: {
      _id: 0,
      name: 1,
      description: 1,
      score: {
        $meta: "searchScore"
      }
    }
  }
])
```

The `score` field contains the relevance score assigned to each result. The documents with the strongest matches appear first.

## Learn more

[Score the documents in the results :octicons-link-external-16:](https://www.mongodb.com/docs/search/query/score/overview/?utm_source=chatgpt.com){:target="_blank"}