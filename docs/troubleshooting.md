# Troubleshooting

Use this page to identify and resolve common issues with Percona Search for MongoDB.

Begin with the information available in the logs, index status, or query response. Review the suggested cause and resolution before altering multiple settings simultaneously. This approach simplifies the process of isolating the issue.

## Before you troubleshoot

Check the following before changing the Percona Search configuration:

* Confirm that the configured model exists on the embedding provider.
* Test the embedding endpoint from the same host or container where `mongot` runs.
* Check the `mongot` log for the first provider error rather than only the final index status.
* Verify the model name, endpoint, authentication settings, and model catalog together.
* Avoid using `localhost` in `providerEndpoint` unless the embedding server is reachable from the same network namespace as `mongot`.
* If you use Voyage AI, configure both `queryKeyFile` and `indexingKeyFile`.

## Automated embedding

The following issues apply when you use automated embedding with Voyage AI or an OpenAI-compatible embedding provider.
{.power-number}

1. Voyage embedding model is skipped during startup

    `mongot` logs a message similar to:

    ```text
    Skipping Voyage embedding model '...'
    ```

    **Cause**

    This is expected when Voyage AI credentials are not configured. `mongot` skips Voyage models if `queryKeyFile` and `indexingKeyFile` are not set.

    **Resolution**

    No action is required if you are not using Voyage AI.

    If you want to use Voyage AI models, configure both `queryKeyFile` and `indexingKeyFile`.

    **Verify**

    Restart `mongot` and check that the Voyage model is loaded without the skip message.

2. `mongot` falls back to the bundled model catalog

    `mongot` reports that it could not load the configured model catalog and uses the bundled catalog.

    **Cause**

    The file specified in `modelConfigFile` might:

    * Not exist
    * Contain invalid syntax
    * Be unreadable by the `mongot` process
    * Point to the wrong location

    **Resolution**

    * Check the `modelConfigFile` path and validate the catalog file.
    * Ensure the `mongot` process has read permissions for the file.
    * Fix any issues in the file and restart `mongot`.

    **Verify**

    Check the startup logs to confirm that `mongot` loads the configured catalog without falling back to the bundled version.

3. An index remains in `PENDING` or `BUILDING` state

    `mongot` logs a message similar to:

    ```text
    Failed embedding call in retry time: N, retrying
    ```

    **Cause**

    `mongot` cannot successfully generate embeddings. Common causes include:

    * The embedding endpoint is unreachable.
    * The configured model does not exist.
    * The model has not been pulled in Ollama.
    * The embedding service is temporarily unavailable.

    **Resolution**

    * Check that the embedding service is running and that the configured model is available.
    * Test the endpoint from the host or container where `mongot` runs.
    * `mongot` retries failed embedding requests according to `errorHandlingConfig`. After the provider becomes available, indexing can continue automatically.

    **Verify**

    Check the index status and confirm that it progresses from `PENDING` or `BUILDING` to `READY`.

4. Embedding requests return HTTP `401` or `403`

    `mongot` reports an authentication failure.

    **Cause**

    The API key is missing, invalid, or sent using the wrong HTTP header.

    **Resolution**

    * Check the configured API key and `authHeaderName`.

    * For Azure OpenAI deployments that use API key authentication, configure:

        ```yaml
        authHeaderName: api-key
        ```

    * Authentication errors are not retried. Correct the authentication configuration before retrying the request.

    **Verify**

    Retry the operation and confirm that the embedding provider accepts the request.

5. `OPENAI_COMPATIBLE` reports unsupported embeddings

    `mongot` reports:

    ```text
    OPENAI_COMPATIBLE provider currently supports only float embeddings
    ```

    **Cause**

    The index definition requests an embedding representation that the provider does not support.

    **Resolution**

    Configure:

    ```yaml
    quantization: float
    ```

    You can also omit the setting if `float` is used by default.

    **Verify**

    Recreate or update the index and confirm that embedding generation starts successfully.

6. Search relevance is poor with some embedding models

    Queries complete successfully, but the results are noticeably less relevant than expected.

    **Cause**

    Some embedding models use different prefixes for documents and queries.

    For example, models such as `nomic-embed-text` can require `documentPrefix` and `queryPrefix`. Without these prefixes, indexing and queries can still succeed, but the vectors may not be comparable in the way the model expects.

    **Resolution**

    * Check the embedding model documentation and configure the required `documentPrefix` and `queryPrefix` values in the model catalog.
    * Rebuild the affected index if required.

    **Verify**

    Run the same representative queries again and compare the returned results.

7. A remote `mongot` instance cannot connect to Ollama

    Ollama works locally, but `mongot` running on another host cannot reach it.

    **Cause**

    Ollama listens on `127.0.0.1` by default. This allows connections only from the local host.

    **Resolution**

    * Configure Ollama to listen on an address reachable by `mongot`.

        For example:

        ```sh
        OLLAMA_HOST=0.0.0.0:11434 ollama serve
        ```

    * Configure `providerEndpoint` with the hostname or IP address that `mongot` can reach.

    * Do not use `localhost` unless Ollama and `mongot` run on the same host or network namespace.

    **Verify**

    From the host or container where `mongot` runs, test the Ollama endpoint before retrying the index build.

## Learn more

* [Configure `mongot`](https://www.mongodb.com/docs/search/self-managed/current/configuration/reference/)
* [Configure `mongot` for Automated Embedding](https://www.mongodb.com/docs/search/self-managed/current/configuration/automated-embedding/)
* [How Automated Embedding Works](https://www.mongodb.com/docs/vector-search/crud-embeddings/automated-embedding/overview/)
* [Ollama FAQ](https://docs.ollama.com/faq)
* [Azure OpenAI embeddings REST reference](https://learn.microsoft.com/en-us/rest/api/microsoft-foundry/azureopenai/embeddings)
