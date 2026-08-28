# Configure automatic embedding with OpenAI

You can use OpenAI embedding models with Percona Search for MongoDB through the `OPENAI_COMPATIBLE` provider.

`mongot` sends document and query text to the OpenAI `/v1/embeddings` endpoint. OpenAI returns the generated vectors, which `mongot` uses for indexing and vector search queries.


## Before you begin

Make sure that:

- Percona Server for MongoDB and `mongot` are properly configured and operational.
- The `mongot` host has network access to the OpenAI API.
- An OpenAI API key is available.
- Your OpenAI account has access to the desired embedding model.
- Make sure the `text-embedding-3-small` model is available before creating an autoEmbed index that references it.

The examples on this page use `text-embedding-3-small`, which produces 1536-dimensional vectors.

## Procedure

To configure automatic embedding with OpenAI, do the following:
{.power-number}

1. Create an API key in your OpenAI account.

    - Verify that the embeddings endpoint responds and that your key works:

    ```bash
    curl -s https://api.openai.com/v1/embeddings \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer <your-openai-api-key>" \
      -d '{
       "model": "text-embedding-3-small",
       "input": ["hello"]
    }'
    ```

     - A successful response contains an embedding vector. This confirms that the model is available and the `/v1/embeddings` endpoint is responding.

2. Configure the model in the catalog:

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
3. Authenticate with OpenAI.

    - When `authHeaderName` isn't specified, `mongot` uses the standard Authorization header.

    - The request uses:

      Authorization: Bearer <key>

    - You don't need to set `authHeaderName` for OpenAI.

4. Choose a vector dimension:

    `text-embedding-3 `models support the OpenAI dimensions request field.

    The default entry uses:

    ```sh
    modelConfig:
        outputDimensions: 1536
        quantization: float
        forwardDimensions: true
    ```

    - When `forwardDimensions` is enabled, `mongot` can forward the resolved index dimension to OpenAI.

    - Use `forwardDimensions` only with models that support the dimensions parameter.

    - For models with a fixed output size, keep `forwardDimensions `disabled.

5. Configure a custom model catalog:

    If you maintain your own catalog, configure its location:

    ```sh
    embedding:
        isAutoEmbeddingViewWriter: true
        modelConfigFile: /etc/mongot/embedding-service-configs.yml
    ```

    Restart `mongot` after changing the catalog.

6. Start and verify `mongot`:

    ```sh
    ./mongot --config mongot.conf
    ```

    Check the logs for configuration or authentication errors.

## Next steps

[Create and query an autoEmbed index :material-arrow-right:](autoembed-index.md){.md-button}


