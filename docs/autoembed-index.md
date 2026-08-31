# Create and query an autoEmbed index

Once an embedding engine is connected, the workflow is the same whichever engine you chose. You create a Vector Search index with an `autoEmbed` field, wait for the initial build, and query with plain text.

!!! note

    This page uses the `nomic-embed-text` model from the [Ollama setup](configure-automatic-embedding-ollama.md). Substitute your own model name if you connected a different engine.

## Before you begin

Make sure that:

- Automatic embedding is enabled in `Percona Search for MongoDB`, and the embedding engine is reachable from the `mongot` host.
- The model you plan to use exists in `embedding-service-configs.yml`.
- The collection contains the text field you want to search.

If `mongot` skipped your model at startup, fix that before you create an index. See [Troubleshoot automatic embedding](troubleshooting.md#automated-embedding).

## Procedure
To create and query an `autoEmbed` index, do the following:
{.power-number}

1. Create sample data.

    ??? example "movies collection"

        ```javascript
        use mydb

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

    | Placeholder | Value |
    |---|---|
    | `<collection>` | The collection holding the documents you want to search. |
    | `<index-name>` | A name for the index. You reference it later in `$vectorSearch`. |
    | `<field-to-embed>` | The text field that `mongot` embeds. Use dot notation for a nested field. |
    | `<model-name>` | A `modelName` from the model catalog. |

    !!! note

        The `model` value must match a `modelName` defined in `embedding-service-configs.yml`. You don't need to set `numDimensions`, because `mongot` resolves the dimension from the model's `outputDimensions` in the catalog.

    ??? example "Vector search index for the plot field"

        ```javascript
        db.movies.createSearchIndex(
          "plot_semantic",
          "vectorSearch",
          {
            fields: [
              {
                type: "autoEmbed",
                modality: "text",
                path: "plot",
                model: "nomic-embed-text"
              }
            ]
          }
        );
        ```

    To filter results before the vector search runs, add a `filter` field alongside the `autoEmbed` field. For the full set of index fields, see [How to Index Fields for Vector Search :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/index/vector-search-type/){:target="_blank"} in the MongoDB documentation.

3. Check the index status.

    ```javascript
    db.movies.getSearchIndexes("plot_semantic");
    ```

    The index stays in a build state while embeddings are generated, moving from `BUILDING` to `READY`. On a large collection this takes time, and the work happens on your embedding engine.

    Wait until the index is ready before you run vector search queries.

    ??? info "What happens during the initial build"

        `mongot`:
        {.power-number}

        1. Scans documents that contain the indexed field.
        2. Sends the field value to the configured embedding provider.
        3. Receives the generated vector.
        4. Stores the generated vectors in a dedicated internal database on the cluster.
        5. Builds the Vector Search index.

4. Run a semantic query.

    Pass plain text to `$vectorSearch`. `mongot` embeds the query text with the same model named in the index:

    ```javascript
    db.<collection>.aggregate([
      {
        $vectorSearch: {
          index: "<index-name>",
          path: "<field-to-embed>",
          query: {
            text: "<search-text>"
          },
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

    | Placeholder | Value |
    |---|---|
    | `<collection>` | The collection holding the documents you want to search. |
    | `<index-name>` | The name of the vector search index you created. |
    | `<field-to-embed>` | The text field that `mongot` embeds. Use dot notation for a nested field. |
    | `<search-text>` | The plain text query to search for. |
    | `<number-of-candidates>` | The number of candidates to consider during the search. Set it higher than `<number-of-results>`. |
    | `<number-of-results>` | The number of top results to return. |
    | `<field1>`, `<field2>` | The fields to include in the query result. |

    ??? example "Semantic query with plain text"

        ```javascript
        db.movies.aggregate([
          {
            $vectorSearch: {
              index: "plot_semantic",
              path: "plot",
              query: {
                text: "space exploration survival mission"
              },
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
        ```

        The output is similar to the following:

        ```json
        [
          { "title": "The Martian", "score": 0.82 }
        ]
        ```

    ??? info "What happens at query time"

        `mongot` generates an embedding for the query text, applying the model's `queryPrefix` if one is configured, then compares that vector against the vectors in the index to find the closest matches.

        The score comes from `{ $meta: "vectorSearchScore" }` and is calculated at query time. A higher score means a closer match under the configured similarity function.

    If you don't specify `model` in `$vectorSearch`, `mongot` uses the model named in the index. You can override it, but the model you name must be compatible with the one used at index time. Embeddings from unrelated models aren't comparable, so an incompatible override returns results that look plausible and aren't.

5. Insert new documents.

    `mongot` embeds new and changed documents as they are written, using change streams, so there is no reindexing to schedule.

    ??? example "Embedding a newly inserted document"

        ```javascript
        db.movies.insertOne({
          title: "Apollo 13",
          category: "drama",
          plot: "Astronauts work with mission control to survive a damaged spacecraft and return safely to Earth"
        });
        ```

    When the indexed text changes, `mongot` generates a new embedding. When a document is deleted, its generated embedding is removed.

6. Monitor embedding requests.

    Use the `mongot` metrics endpoint to watch embedding traffic by provider. For OpenAI-compatible engines, filter on the `OPENAI_COMPATIBLE` provider:

    ```sh
    curl -s localhost:9946/metrics | grep 'provider="OPENAI_COMPATIBLE"'
    ```

    The following metrics are useful for monitoring embedding requests:

    - `mongot_embeddingClient_inputTokenDistribution_*` tracks input token usage by model and by workload. The workload is one of `COLLECTION_SCAN`, `CHANGE_STREAM`, or `QUERY`.
    - `mongot_embeddingClient_invalidRequestCounter` counts embedding requests rejected as invalid.

    These metrics can help you monitor embedding usage, understand which workloads generate the most traffic, and identify rejected requests.

## Learn more

- [Create a Vector Search Index :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/tutorials/quick-start/?deployment-type=self&embedding=auto&interface=mongosh){:target="_blank"}
- [Run a Vector Search Query :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/tutorials/quick-start/?deployment-type=self&embedding=auto&interface=mongosh#run-a-vector-search-query){:target="_blank"}
- [Automated Embedding overview :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/crud-embeddings/automated-embedding/){:target="_blank"}