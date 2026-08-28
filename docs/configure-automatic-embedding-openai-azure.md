# Configure automatic embedding with Azure OpenAI

Azure OpenAI uses the `OPENAI_COMPATIBLE` provider. It differs from the standard OpenAI endpoint in two ways:

{.power-number}

1. The endpoint is specific to an Azure OpenAI deployment and includes an `api-version` parameter.
2. Authentication uses the `api-key` header rather than `Authorization: Bearer`.

## Before you begin

Make sure that:

* Percona Server for MongoDB and `mongot` are configured and running.
* An embedding model is deployed to your Azure OpenAI resource.
* You know the Azure resource name.
* You know the deployment name.
* You have an Azure OpenAI API key.
* You know the API version supported by your deployment.
* The host running `mongot` can connect to Azure OpenAI.

## Procedure

To configure automatic embedding with Azure OpenAI, do the following:

{.power-number}

1. Verify the Azure endpoint.

   Azure OpenAI uses a deployment-specific embedding endpoint:

   ```text
   https://<resource>.openai.azure.com/openai/deployments/<deployment>/embeddings?api-version=<api-version>
   ```

   Test the endpoint before configuring `mongot`:

   ```bash
   curl "https://<resource>.openai.azure.com/openai/deployments/<deployment>/embeddings?api-version=<api-version>" \
     -H "Content-Type: application/json" \
     -H "api-key: <your-azure-openai-api-key>" \
     -d '{
       "model": "<deployment-name>",
       "input": "hello"
     }'
   ```

   A successful response contains an embedding vector.

2. Enable automatic embedding.

   Add the `embedding` section to `mongot.conf`:

   ```yaml
   embedding:
     isAutoEmbeddingViewWriter: true
   ```

   !!! important
   If multiple `mongot` instances process the same data, configure only one instance as the automatic embedding writer.

3. Configure the model in the catalog.

   Add an entry to `embedding-service-configs.yml` with the Azure OpenAI endpoint and API key:

   ```yaml
   configs:
     - modelName: text-embedding-3-small
       embeddingProvider: OPENAI_COMPATIBLE
       config:
         providerEndpoint: https://my-resource.openai.azure.com/openai/deployments/text-embedding-3-small/embeddings?api-version=2024-02-01

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

   !!! note
   Replace the resource name, deployment name, API version, and API key with values from your Azure OpenAI deployment.

4. Configure Azure authentication.

   Standard OpenAI authentication uses:

   ```text
   Authorization: Bearer <key>
   ```

   Azure OpenAI uses:

   ```text
   api-key: <key>
   ```

   Configure the credentials as follows:

   ```yaml
   credentials:
     apiKey: "<your-azure-openai-api-key>"
     authHeaderName: api-key
   ```

   When `authHeaderName` is set to `api-key`, `mongot` sends the API key directly in that header without adding the `Bearer` prefix.

5. Configure vector dimensions.

   If your Azure deployment uses a `text-embedding-3` model, you can enable:

   ```yaml
   forwardDimensions: true
   ```

   This allows `mongot` to send the resolved vector dimension using the OpenAI-compatible `dimensions` field.

   For models that don't support configurable dimensions, omit `forwardDimensions` or set it to `false`.

   !!! note
   Make sure `outputDimensions` matches the vector dimensions expected by the model and index configuration.

6. Start and verify `mongot`.

   Restart `mongot` after updating the model catalog:

   ```bash
   ./mongot --config mongot.conf
   ```

   Review the logs for configuration, connectivity, or authentication errors.

   If `mongot` cannot connect to Azure OpenAI, check the following:

   * **Authentication:** Verify that the API key is correct and that `authHeaderName` is set to `api-key`.
   * **Endpoint:** Confirm that `providerEndpoint` matches your Azure OpenAI resource and deployment.
   * **API version:** Verify that the `api-version` parameter is supported by your deployment.
   * **Deployment name:** Confirm that the deployment name matches the model deployment in Azure OpenAI.
   * **Vector dimensions:** Verify that `outputDimensions` is valid for the selected model. If you use `forwardDimensions`, make sure the model supports configurable dimensions.

## Next steps

[Create and query an `autoEmbed` index :material-arrow-right:](autoembed-index.md){.md-button}
