# Create and query an `autoEmbed` index

Once an embedding engine is connected, the workflow is the same regardless of which engine you chose. You create a Vector Search index with an `autoEmbed` field, wait for the initial build, and query with plain text.

!!! tip "Info"
    This page uses the `nomic-embed-text` model from the [Ollama setup](configure-automatic-embedding-ollama.md). Substitute your own model name if you connected a different engine.

## Before you begin

- Ensure that automatic embedding is enabled in `mongot`.
- Verify that the embedding provider is configured.
- Confirm that the model exists in `embedding-service-configs.yml`.
- Check that the embedding endpoint is reachable.
- Ensure the collection contains a text field you want to search.

## Procedure

To create an `autoEmbed` index, follow these steps:
{.power-number}

1. Create sample data.

    ??? example "movies collection"
        ```yaml
        use mydb
        ```

        ```javascript
        db.movies.insertMany([
            {
                title: "The Martian",
                plot: "An astronaut becomes stranded on Mars and must use his ingenuity to survive until rescue"
            },
            {
                title: "Finding Nemo",
                plot: "A clownfish father crosses the ocean to find his son who was captured by a scuba diver"
            },
            {
                title: "The Godfather",
                plot: "The aging patriarch of an organized crime dynasty transfers control of his empire to his reluctant son"
            }
    ]);
        ```

2. Create the index.

    Use the following syntax:

    ```javascript
    db.<collection>.createSearchIndex(
      "<index-name>",
      "vectorSearch",
     {
       fields: [
         {
            type: "autoEmbed",
            modality: "text",
            path: "<field-to-embed>",
            model: "<model-name>"
         }
       ]
      }
    );
    ``` 
    
    Replace the placeholders with your own values:

    | **Placeholder**| **Value**|
    |-----------------|------|
    | `<collection>`    | The collection holding the documents you want to search.             |
    | `<index-name>`    | A name for the index. You reference it later in `$vectorSearch`.      |
    | `<field-to-embed>`| The text field that `mongot` embeds. Use dot notation for a nested field. |
    | `<model-name>`    | A `modelName` from the model catalog. 
    
    !!! note
        - Ensure the `model` value matches a `modelName` defined in `embedding-service-configs.yml`.
        - The `numDimensions` parameter is not required. `mongot` automatically determines the dimension based on the `outputDimensions` setting specified in the model catalog.

    ??? example "Example: Vector search index"

        Create a vector search index for the `plot` field:

        ```javascript
        db.movies.createSearchIndex(
        "plot_semantic",
        "vectorSearch",
        {
            fields: [
            {
                type: "autoEmbed",
                path: "plot",
                model: "nomic-embed-text",
                modality: "text",
                similarity: "cosine"
            }
            ]
        }
        );
        ```

        ??? info "What happens under the hood"

            During the initial build, `mongot`:
            {.power-number}

            1. Scans documents that contain the indexed field.
            2. Sends the field value to the configured embedding provider.
            3. Receives the generated vector.
            4. Stores the generated embedding data.
            5. Builds the Vector Search index.

3. Check the index status.

     ??? example "Index status"

        ```sh
        db.movies.getSearchIndexes("plot_semantic")   // status: "BUILDING" -> "READY"
        ```

    The index can remain in a build state while embeddings are being generated.

    Wait until the index is ready before running vector search queries.


4. Run a semantic query.
    
    Pass plain text to `$vectorSearch`. The query text is automatically embedded using the same model specified in the index:

    ```javascript
    db.<collection>.aggregate([
        {
            $vectorSearch: {
                index: "<index-name>",
                query: "<search-text>",
                path: "<field-to-embed>",
                numCandidates: <number-of-candidates>,
                limit: <number-of-results>
            }
        },
        {
            $project: {
                _id: 0,
                <field1>: 1,
                <field2>: 1,
                score: { $meta: "vectorSearchScore" }
            }
        }
    ]);
    ```

    Replace the placeholders with your own values:

    | **Placeholder**         | **Value**                                                                 |
    |--------------------------|---------------------------------------------------------------------------|
    | `<collection>`           | The collection holding the documents you want to search.                 |
    | `<index-name>`           | The name of the vector search index you created.                         |
    | `<search-text>`          | The plain text query to search for.                                      |
    | `<field-to-embed>`       | The text field that `mongot` embeds. Use dot notation for a nested field.|
    | `<number-of-candidates>` | The number of candidate documents to consider during the search.         |
    | `<number-of-results>`    | The number of top results to return.                                     |
    | `<field1>`, `<field2>`   | The fields to include in the query result.                               |
    ```

??? example "Example: Semantic query with plain text"

    ```javascript
    db.movies.aggregate([
        { 
            $vectorSearch: {
                index: "plot_semantic",
                path: "plot",
                query: "space exploration survival mission",
                numCandidates: 10,
                limit: 3
            }
        },
        { 
            $project: { 
                _id: 0, 
                title: 1, 
                score: { $meta: "vectorSearchScore" } 
            } 
        }
    ]);

    ```bash
    [
        { "title": "The Martian", "score": 0.82 },
        ...
    ]
    ```

    ?? info "What happens under the hood"
    - `mongot` automatically generates an embedding for the query text.
    - The query vector is compared with the vectors in the index to find the closest matches.
    - The score is calculated at query time using `{ $meta: "vectorSearchScore" }`.
    - A higher score indicates a closer match based on the configured similarity function.


5. Insert new documents.

    Newly added or updated documents are automatically embedded in real-time through change streams. This process removes the need for manual reindexing, ensuring the index remains up-to-date seamlessly.

    ??? example "Example: Real-time embedding for new documents"

        ```javascript
        db.movies.insertOne({
            title: "Apollo 13",
            category: "drama",
            plot: "Astronauts work with mission control to survive a damaged spacecraft and return safely to Earth"
        });
        ```

6. Monitor embedding requests:

    You can use the `mongot` metrics endpoint to monitor embedding traffic by provider. For OpenAI-compatible embedding servers, filter metrics by the `OPENAI_COMPATIBLE `provider:\

    ```bash
    curl -s localhost:9946/metrics | grep 'provider="OPENAI_COMPATIBLE"'
    ```
    
    The following metrics are useful for monitoring embedding requests:

    - `mongot_embeddingClient_inputTokenDistribution_*`: Tracks input token usage by embedding model and workload. Workloads include `COLLECTION_SCAN`, `CHANGE_STREAM`, and `QUERY`.
    - `mongot_embeddingClient_invalidRequestCounter`: Tracks embedding requests rejected as invalid.

    These metrics can help you monitor embedding usage, understand which workloads generate the most traffic, and identify rejected requests.




## Learn more

[$vectorSearch aggregation stage :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/operator/aggregation/vectorSearch/){:target="_blank"}















