# Automatic embedding with Ollama

You can use Ollama to generate embeddings locally for Percona Search for MongoDB.

Ollama exposes an OpenAI-compatible `/v1/embeddings` endpoint. This allows mongot to use Ollama through the `OPENAI_COMPATIBLE` provider without requiring your application to generate embeddings.

## Before you begin

Make sure that:
{.power-number}

- Percona Server for MongoDB and mongot are installed and configured.
- Percona Server for MongoDB is running as a replica set.
- [Ollama :octicons-link-external-16:](https://docs.ollama.com/quickstart){:target="_blank"} is installed on a host that `mongot` can reach.
- An embedding model is available in Ollama.
- You know the output dimensions of the model you plan to use.

The following example uses `nomic-embed-text`, which produces 768-dimensional vectors in the Percona model catalog.

## Procedure

The following walkthrough sets up a fully local pipeline: 
{.power-number}

1. [Install Ollama :octicons-link-external-16:](https://docs.ollama.com/quickstart){:target="_blank"} and pull an embedding model:

    ```sh
    curl -fsSL https://ollama.com/install.sh | sh
    ollama pull nomic-embed-text
    ```
    - Verify that the OpenAI-compatible embeddings endpoint responds:

    ```bash
    curl -s http://localhost:11434/v1/embeddings \
       -H "Content-Type: application/json" \
       -d '{
         "model": "nomic-embed-text",
         "input": ["hello"]
       }'
    ```

    - A successful response contains an embedding vector. This confirms that the model is available and the `/v1/embeddings` endpoint is responding.

    !!! note
        If Ollama runs on another host, replace `localhost` with an address that `mongot` can reach.

2. Configure PSMDB and `mongot`.

    Follow the [Install and configure mongot](install-mongot.md) procedure to set up Percona Search for MongoDB. Configure PSMDB as a replica set, point the search parameters to `mongot`, and create a user with the `searchCoordinator` role.

    ```bash
    mongod --replSet rs0 \
       --setParameter mongotHost=localhost:27028 \
       --setParameter searchIndexManagementHostAndPort=localhost:27028 ...
    ```

    ```bash
    use admin
    db.createUser({
       user: "searchUser",
       pwd: "<password>",           // must match mongot's passwordFile content
      roles: [{ role: "searchCoordinator", db: "admin" }]
    })

    In the mongot configuration file (`mongot.conf`), add the embedding section. For keyless local engines no credentials are required:

    ```bash
    syncSource:
      replicaSet:
        hostAndPort: 127.0.0.1:27017
        scramAuth:
         username: searchUser
         passwordFile: /etc/mongot/secrets/passwordFile
    storage:
      dataPath: /var/lib/mongot
    embedding:
      isAutoEmbeddingViewWriter: true   # exactly one mongot node writes the embedding materialized view
    server:
      grpc:
        address: localhost:27028
        tls:
          mode: disabled
    metrics:
      enabled: true
      address: "localhost:9946"
    ```
3. Configure the Ollama model:

    The catalog `embedding-service-configs.yml` is installed next to the `mongot` binary and already contains two ready-to-use local models pointed at Ollama — `bge-m3` (1024 dimensions) and `nomic-embed-text` (768 dimensions):

    ```bash
      - modelName: nomic-embed-text
        embeddingProvider: OPENAI_COMPATIBLE
        config:
          providerEndpoint: <http://localhost:11434/v1/embeddings>
          modelConfig:
            batchSize: 96
            batchTokenLimit: 120000
            outputDimensions: 768
            quantization: float
        # nomic-embed-text is an asymmetric model: queries and documents
        # must be embedded with different task-instruction prefixes.
            queryPrefix: "search_query: "
            documentPrefix: "search_document: "
          errorHandlingConfig:
            maxRetries: 10
            initialRetryWaitMs: 200
            maxRetryWaitMs: 10000
            jitter: 0.1
          credentials: {}
    ```

    Where:

    - `modelName` is the name you use in the `autoEmbed` index.
    - `embeddingProvider`: `OPENAI_COMPATIBLE` tells `mongot` to use the OpenAI-compatible embedding client.
    - `providerEndpoint` points to Ollama.
    - `outputDimensions` must match the dimensions returned by the model.
    - `queryPrefix` and `documentPrefix` provide the task instructions required by `nomic-embed-text`.
    - `credentials: `{} means that no API key is sent.
        
4. Edit the model catalog to add new models or update existing endpoints. Restart `mongot` to apply the changes. If you want to keep the default catalog unchanged, create a separate copy and specify its path with `embedding.modelConfigFile`.

    ```bash
    embedding: isAutoEmbeddingViewWriter: true      
    modelConfigFile: /etc/mongot/embedding-service-configs.yml
    ```

5. Start `mongot` and verify:

    ```sh
    ./mongot --config mongot.conf
    ```

    ??? example "Log"
        On startup, you may see log messages similar to the following:

        ```bash
        WARN  ... Voyage API credential files not configured. Voyage models will be unavailable.
          Keyless OPENAI_COMPATIBLE models (Ollama/vLLM/TEI) remain active. ...

        INFO  ... Reading embedding configuration from on-disk catalog

        WARN  ... Skipping Voyage embedding model 'voyage-4-large': no Voyage API credentials configured ...

        INFO  ... Initialized auto-embedding with 2 models
        ```
    The Voyage warnings are expected if you haven't configured Voyage API credentials. They don't affect keyless `OPENAI_COMPATIBLE` models.

        
5. Create an auto-embedding vector search index:

    ```bash
    use mydb
    db.movies.insertMany([
      { title: "The Martian", plot: "An astronaut becomes stranded on Mars and must use his ingenuity to survive until rescue" },
      { title: "Finding Nemo", plot: "A clownfish father crosses the ocean to find his son who was captured by a scuba diver" },
      { title: "The Godfather", plot: "The aging patriarch of an organized crime dynasty transfers control of his empire to his reluctant son" }
    ])

    db.movies.createSearchIndex("plot_semantic", "vectorSearch", {
       fields: [{
         type: "autoEmbed",
         path: "plot",                  // the text field to embed
        model: "nomic-embed-text",     // modelName from the catalog
        modality: "text",
        similarity: "cosine"           // cosine | dotProduct | euclidean
       }]
    })

## Next steps

[Create and query an autoEmbed index :material-arrow-right:](autoembed-index.md){.md-button}
    






    









