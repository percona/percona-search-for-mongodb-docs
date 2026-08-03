# Create vector search index

Before running vector search queries, create a Vector Search index on the field that contains your embeddings.
You can use the `createSearchIndex()` method to create a **Vector Search index** on a collection, or the `createSearchIndexes()` method to create multiple indexes simultaneously.

## Create a single Vector Search index

Follow these steps to create and query a Vector Search index.
{.power-number}

1. Create a Vector Search index named `vector_idx`:

    ```javascript
    db.docs.createSearchIndex({
    name: "vector_idx",
    type: "vectorSearch",
    definition: {
        fields: [
        {
            type: "vector",
            path: "embedding",
            numDimensions: 3,
            similarity: "cosine"
        }
        ]
    }
    })
    ```

2. Check the status:

    ```javascript
    db.docs.getSearchIndexes()
    [  {
        id: '69ebbabd651bce4d10f57be5',
        name: 'vector_index',
        status: 'READY',
        queryable: true,
        latestDefinitionVersion: { version: 0, createdAt: ISODate('2026-04-24T18:47:25.000Z') },
        latestDefinition: {
        fields: [  {
            type: 'vector',
            path: 'text',
            numDimensions: 2048,
            similarity: 'dotProduct',
            quantization: 'scalar'
            }   ]    },
        statusDetail: [
        {
            hostname: '69eb5fc573906b6bfb8cefe7',
            status: 'READY',
            queryable: true,
            mainIndex: {
            status: 'READY',
            queryable: true,
            definitionVersion: { version: 0, createdAt: ISODate('2026-04-24T18:47:25.000Z') },
            definition: {
                fields: [
                {
                    type: 'vector',
                    path: 'text',
                    numDimensions: 2048,
                    similarity: 'dotProduct',
                    quantization: 'scalar'
                }]}} }] }]
    ```

3. Insert test data:

    ```javascript
    db.docs.insertOne({
    text: "AI embeddings example",
    embedding: [0.1, 0.2, 0.3]
    })

    db.docs.insertOne({
    text: "AI embeddings example 2",
    embedding: [0.5, 0.5, 0.6]
    })

    db.docs.insertOne({
    text: "AI embeddings example 3",
    embedding: [0.8, 0.8, 0.5]
    })
    ```

4. Perform a vector search:

    ```javascript
    db.docs.aggregate([
    {
        $vectorSearch: {
        index: "vector_index",		// always required!!
        queryVector: [0.3, 0.2, 0.3],
        path: "embedding",
        numCandidates: 10,
        limit: 2
        }
    },
    {
        "$project": {
        "_id": 0,
        "text": 1,
        "embedding": 1,
        "score": { $meta: "vectorSearchScore" }
        }
    }])
    ```

## Create full-text search and vector search indexes together

You can include full-text search and vector search index definitions in the same command.

```javascript
db.docs.createSearchIndexes([
  {
    name: "search_idx",
    type: "search",
    definition: {
      mappings: {
        dynamic: false,
        fields: {
          text: {
            type: "string"
          }
        }
      }
    }
  },
  {
    name: "vector_idx",
    type: "vectorSearch",
    definition: {
      fields: [
        {
          type: "vector",
          path: "embedding",
          numDimensions: 3,
          similarity: "cosine"
        }
      ]
    }
  }
])
[ 'search_idx', 'vector_idx' ]
```






[Update a search index :material-arrow-right:](../cupdate-search-index.md){.md-button}

[Delete a search index :material-arrow-right:](../delete-search-index.md){.md-button}