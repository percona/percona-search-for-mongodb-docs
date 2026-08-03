# Create vector search index

Before running vector search queries, create a Vector Search index on the field that contains your embeddings.
You can use the `createSearchIndex()` method to create a **Vector Search index** on a collection, or the `createSearchIndexes()` method to create multiple indexes simultaneously.

## Create a single Vector Search index

Follow these steps to create and query a Vector Search index.
{.power-number}

1. Insert test data:

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

2. Create a Vector Search index named `vector_idx`:

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

    The index definition specifies that:

    - Embedding contains the vectors to index.
    - Each vector has three dimensions.
    - Vector similarity is calculated using cosine similarity.

3. Check the index status:

    ```javascript
    db.docs.getSearchIndexes()
    ```
    ??? example "Output"

        ```javascript
        [  
            {
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

    `$vectorSearch` accepts the following fields:

    * `index` is the name of the vector search index to query.
    * `path` is the document field that holds the embeddings. It must match the
    `path` in the index definition.
    * `queryVector` is the vector to search with. It must have the same number of
    dimensions as the indexed embeddings.
    * `numCandidates` is how many nearest neighbors the search gathers before it
    selects the results. A higher value improves accuracy and takes longer. Set it
    well above `limit`, typically 10 to 20 times higher.
    * `limit` is the number of documents to return.

    ??? info "What happens under the hood"
        **When you create the index**
        {.power-number}
        
        1. `mongod` receives the index definition and forwards it to `mongot`. The definition is stored as metadata in MongoDB. `mongot` owns the index data.
        2. `mongot` reads the `embedding` field from the documents in the collection and builds a Hierarchical Navigable Small World (HNSW) graph. This structure is what makes approximate nearest neighbor search fast.
        3. The build is asynchronous, so `createSearchIndex()` returns before it completes.
        4. After that, `mongot` watches the collection through a change stream and applies later inserts, updates, and deletes to the index.

        **When you run a query**
        {.power-number}

        1. `mongod` parses the aggregation pipeline and hands the `$vectorSearch` stage to`mongot`.

        2. `mongot` searches the HNSW graph and returns the identifiers of the matching documents, each with a score. It does not hold your documents, only the index.

        3. `mongod` then pairs those identifiers with the documents in the collection and passes the result through the rest of the pipeline. The search returns the documents whose embeddings are nearest to `queryVector`, closest match first.

        4. `$project` shapes that output. The `score` field comes from `$meta: "vectorSearchScore"`. Percona Search for MongoDB calculates the score during the exchange described above and does not store it in the document, which is why you read it as metadata rather than as a field. A higher score means a closer match.

[Update a search index :material-arrow-right:](../cupdate-search-index.md){.md-button}

[Delete a search index :material-arrow-right:](../delete-search-index.md){.md-button}