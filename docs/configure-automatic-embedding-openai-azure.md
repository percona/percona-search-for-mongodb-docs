# Configure automatic embedding with Azure OpenAI

Azure OpenAI also uses the `OPENAI_COMPATIBLE` provider. It differs from the standard OpenAI endpoint in two ways:
{.power-number}

1. The endpoint is specific to an Azure OpenAI deployment and includes an `api-version` parameter.
2. Authentication uses the `api-key` header rather than `Authorization: Bearer`.

## Before you begin

- Ensure that Percona Server for MongoDB and `mongot` are properly configured and operational.
- Deploy an embedding model to the Azure resource.
- Obtain the Azure resource name.
- Acquire the deployment name.
- Generate an API key.
- Verify the API version supported by your deployment.
- Confirm network connectivity from `mongot` to Azure OpenAI.


## Procedure

To configure automatic embedding with OpenAI, do the following:
{.power-number}

1. Verify the Azure endpoint

    The Percona configuration can use an Azure deployment-specific embedding URL such as: 
    https://<resource>.openai.azure.com/openai/deployments/<deployment>/embeddings?api-version=<api-version>

    Test the endpoint before configuring `mongot`.
    - Verify that the embeddings endpoint responds and that your key works:

    ```bash
    curl "https://<resource>.openai.azure.com/openai/deployments/<deployment>/embeddings?api-version=<api-version>" \ 
    -H "Content-Type: application/json" \ 
    -H "api-key: <your-azure-openai-api-key>" \ 
    -d '{ "model": "<deployment-name>", "input": "hello" }'
    ```

     - A successful response contains an embedding vector.

3. Enable automatic embedding:

    Add the embedding section to `mongot.conf`:

   
        embedding:
            isAutoEmbeddingViewWriter: true
    

    Configure only one automatic embedding writer when multiple `mongot` instances process the same data.`

2. Configure the model in the catalog:

    Add an entry to `embedding-service-configs.yml` with the OpenAI endpoint and your API key:

    ```yaml
    configs:
      - modelName: text-embedding-3-small
        embeddingProvider: OPENAI_COMPATIBLE
        config:
          providerEndpoint: <https://my-resource.openai.azure.com/openai/deployments/text-embedding-3-small/embeddings?api-version=2024-02-01>

          modelConfig:
            batchSize: 96
            batchTokenLimit: 120000
            outputDimensions: 1536
            quantization: float
            forwardDimensions: true
          errorHandlingConfig:
              maxRetries: 10
              initialRetryWaitMs: 200
              maxRetryWaitMs: 10000
              jitter: 0.1
          credentials:
             apiKey: "<your-azure-openai-api-key>"
             authHeaderName: api-key
    ```
3. Configure Azure authentication:

    - OpenAI normally uses:

        `Authorization: Bearer <key>`

        For the Azure API key configuration shown here, use:

        `api-key: <key>`

    - Set:

        ```bash
        credentials: 
            apiKey: "<your-azure-openai-api-key>" 
            authHeaderName: api-key
        ```

    - When `authHeaderName` is set to `api-key`, `mongot` sends the raw key in that header instead of adding a Bearer prefix.

4. Configure dimensions:

    - If the Azure deployment uses a text-embedding-3 model, you can enable:

    `forwardDimensions: true`

    This allows the requested vector dimension to be sent using the OpenAI-compatible dimensions field.

    - For models that don't support configurable dimensions, leave this setting unset or set it to `false`.

5. Start and verify `mongot`:

    Restart `mongot` after adding the model:

    ```sh
    ./mongot --config mongot.conf
    ```

    Check the logs for potential issues:

    - Check for authentication errors:
        - Ensure the API key is correct and has the necessary permissions.
        - Verify that the `api-key` header is properly configured.

    - Validate the endpoint:
        - Confirm that the Azure OpenAI endpoint is accurate and matches your deployment.
        - Test the endpoint using `curl` or similar tools to ensure it is reachable.

    - Verify the API version:
        - Ensure the `api-version` parameter matches the version supported by your Azure deployment.

    - Check the deployment configuration:
        - Confirm that the deployment name is correct and matches the one in your Azure resource.

    - Resolve dimension errors:
        - Verify that the `outputDimensions` setting matches the model's expected dimensions.
        - If using `forwardDimensions`, ensure the model supports configurable dimensions.

## Next steps

[Create and query an autoEmbed index :material-arrow-right:](autoembed-index.md){.md-button}


