# Configure automatic embedding with OpenAI

You can use OpenAI embedding models with Percona Search for MongoDB through the `OPENAI_COMPATIBLE` provider.

`mongot` sends document and query text to the OpenAI `/v1/embeddings` endpoint. OpenAI returns the generated vectors, which `mongot` uses for indexing and for vector search queries.

## Before you begin

Make sure that:

- Percona Server for MongoDB and Percona Search for MongoDB are configured and running.
- The `mongot` host has network access to the OpenAI API.
- You have an OpenAI API key.
- Your OpenAI account has access to the embedding model you plan to use.

The examples on this page use `text-embedding-3-small`, which produces 1536-dimensional vectors.

## Procedure

To configure automatic embedding with OpenAI, do the following:
{.power-number}

1. Create an API key in your OpenAI account.

    Verify that the embeddings endpoint responds and that your key works:

    ```sh
    curl -s https://api.openai.com/v1/embeddings \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer <your-openai-api-key>" \
      -d '{
        "model": "text-embedding-3-small",
        "input": ["hello"]
      }'
    ```

    A successful response contains an embedding vector. This confirms that the model is available to your account and that the `/v1/embeddings` endpoint is responding.

2. Enable automatic embedding in `mongot`.

    Add an `embedding` section to the active `mongot` configuration file. For the documented systemd installation, edit `/etc/mongot/config.yml`. The presence of this section activates automatic embedding:

    ```yaml
    embedding:
      isAutoEmbeddingViewWriter: true
    ```

    !!! info "Important"

        Set `isAutoEmbeddingViewWriter: true` on exactly one `mongot` node. That node writes the embedding materialized view. If several `mongot` instances serve the same data, configure only one of them as the writer.

3. Configure the model in the catalog.

    Add an entry to `embedding-service-configs.yml` with the OpenAI endpoint and your API key:

    ```yaml
    configs:
      - modelName: text-embedding-3-small
        embeddingProvider: OPENAI_COMPATIBLE
        config:
          providerEndpoint: https://api.openai.com/v1/embeddings

          modelConfig:
            batchSize: 96
            batchTokenLimit: 120000
            outputDimensions: 1536
            quantization: float
            # text-embedding-3 models support Matryoshka dimension reduction
            forwardDimensions: true

          errorHandlingConfig:
            maxRetries: 10
            initialRetryWaitMs: 200
            maxRetryWaitMs: 10000
            jitter: 0.1

          credentials:
            apiKey: "<your-openai-api-key>"
    ```

    Restart `mongot` after you change the catalog.

    !!! Info "Important"

        The catalog now holds a secret. Restrict the file so that only the account running `mongot` can read it, and keep it out of version control.

4. Authenticate with OpenAI.

    When `authHeaderName` isn't set, `mongot` uses the standard `Authorization` header and sends the key with the Bearer scheme:

    ```text
    Authorization: Bearer <key>
    ```

    This is what OpenAI expects, so you don't need to set `authHeaderName`. The key is redacted from `mongot` logs and error messages.

    Azure OpenAI uses a different header. See [Configure automatic embedding with Azure OpenAI](configure-automatic-embedding-openai-azure.md).

5. Choose a vector dimension.

    The `text-embedding-3` models support the OpenAI `dimensions` request field, so they can return vectors shorter than their native size. The catalog entry above enables this:

    ```yaml
    modelConfig:
      outputDimensions: 1536
      quantization: float
      forwardDimensions: true
    ```

    When `forwardDimensions` is enabled, `mongot` forwards the resolved index dimension to OpenAI. Shorter vectors use less storage and memory, at some cost to accuracy.

    !!! info "Important"

        Use `forwardDimensions` only with models that support the `dimensions` parameter. For a model with a fixed output size, leave the setting at its default or set it to `false`, and make sure `outputDimensions` matches the model's native dimension.

6. Start `mongot` and check the startup output:

    ```sh
    systemctl start mongot
    ```

    Confirm that your OpenAI model appears in the loaded model count. Warnings about skipped Voyage models are expected if you haven't configured Voyage credentials.

## Next steps

[Create and query an autoEmbed index :material-arrow-right:](autoembed-index.md){.md-button}