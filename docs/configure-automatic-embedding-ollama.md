# Automatic embedding with Ollama

You can use Ollama to generate embeddings locally for Percona Search for MongoDB.

Ollama exposes an OpenAI-compatible `/v1/embeddings` endpoint. This allows mongot to use Ollama through the `OPENAI_COMPATIBLE` provider without requiring your application to generate embeddings.

## Before you begin

Make sure that:

- Percona Server for MongoDB and Percona Search for MongoDB are installed and configured.
- Percona Server for MongoDB is running as a replica set.
- [Ollama :octicons-link-external-16:](https://docs.ollama.com/quickstart){:target="_blank"} is installed on a host that `mongot` can reach.
- An embedding model is available in Ollama.
- You know the output dimensions of the model you plan to use.

The following example uses `nomic-embed-text`, which produces 768-dimensional vectors in the Percona model catalog.

## Procedure

To set up a fully local embedding pipeline, do the following:
{.power-number}

1. [Install Ollama :octicons-link-external-16:](https://docs.ollama.com/quickstart){:target="_blank"} and pull an embedding model.

    ```sh
    curl -fsSL https://ollama.com/install.sh | sh
    ollama pull nomic-embed-text
    ```

    - Verify that the OpenAI-compatible embeddings endpoint responds:

        ```sh
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

    ```sh
    mongod --replSet rs0 \
      --setParameter mongotHost=localhost:27028 \
      --setParameter searchIndexManagementHostAndPort=localhost:27028
    ```

    ```javascript
    use admin

    db.createUser({
      user: "searchUser",
      pwd: "<password>",
      roles: [{ role: "searchCoordinator", db: "admin" }]
    });
    ```

    The password must match the contents of the file that `mongot` references in `passwordFile`.

    In the `mongot` configuration file, `mongot.conf`, add the `embedding` section. Keyless local engines need no credentials:

    ```yaml
    syncSource:
      replicaSet:
        hostAndPort: 127.0.0.1:27017
        scramAuth:
          username: searchUser
          passwordFile: /etc/mongot/secrets/passwordFile
    storage:
      dataPath: /var/lib/mongot
    embedding:
      # exactly one mongot node writes the embedding materialized view
      isAutoEmbeddingViewWriter: true
    server:
      grpc:
        address: localhost:27028
        tls:
          mode: disabled
    metrics:
      enabled: true
      address: "localhost:9946"
    ```

3. Configure the Ollama model.

    The catalog `embedding-service-configs.yml` is installed next to the `mongot` binary and already contains two ready-to-use local models pointed at Ollama: `bge-m3` at 1024 dimensions and `nomic-embed-text` at 768 dimensions.

    ```yaml
    configs:
      - modelName: nomic-embed-text
        embeddingProvider: OPENAI_COMPATIBLE
        config:
          providerEndpoint: http://localhost:11434/v1/embeddings

          modelConfig:
            batchSize: 96
            batchTokenLimit: 120000
            outputDimensions: 768
            quantization: float
            # nomic-embed-text is an asymmetric model: queries and documents
            # must be embedded with different task-instruction prefixes
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
    - `embeddingProvider` set to `OPENAI_COMPATIBLE` tells `mongot` to use the OpenAI-compatible embedding client.
    - `providerEndpoint` points to Ollama.
    - `outputDimensions` must match the dimensions returned by the model.
    - `queryPrefix` and `documentPrefix` provide the task instructions required by `nomic-embed-text`.
    - `credentials: {}` means that no API key is sent.

4. Edit the model catalog to add new models or update existing endpoints. Restart `mongot` to apply the changes.

    If you want to keep the default catalog unchanged, create a separate copy and specify its path with `embedding.modelConfigFile`:

    ```yaml
    embedding:
      isAutoEmbeddingViewWriter: true
      modelConfigFile: /etc/mongot/embedding-service-configs.yml
    ```

5. Start `mongot` and verify:

    ```sh
    systemctl start mongot
    ```

    ??? example "Log"

        On startup, you may see log messages similar to the following:

        ```text
        WARN  Voyage API credential files not configured. Voyage models will be unavailable.
              Keyless OPENAI_COMPATIBLE models (Ollama/vLLM/TEI) remain active.
        INFO  Reading embedding configuration from on-disk catalog
        WARN  Skipping Voyage embedding model 'voyage-4-large': no Voyage API credentials configured
        INFO  Initialized auto-embedding with 2 models
        ```

        The Voyage warnings are expected if you haven't configured Voyage API credentials. They don't affect keyless `OPENAI_COMPATIBLE` models.

## Next steps

[Create and query an autoEmbed index :material-arrow-right:](autoembed-index.md){.md-button}
    






    









