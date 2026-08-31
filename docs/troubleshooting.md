# Troubleshooting

Use this page to identify and resolve common issues with Percona Search for MongoDB.

Start with the symptom you see in the logs, index status, or query response. Check the suggested cause and fix before changing multiple settings at once. This makes it easier to isolate the problem.

## Automated embedding

The following issues apply when you use automated embedding with Voyage AI or an OpenAI-compatible embedding provider.
{.power-number}

1. Voyage embedding model is skipped during startup

    mongot logs a message similar to:

    ```sh
    Skipping Voyage embedding model '...'
    ```

    **Cause**

    This is expected when Voyage AI credentials are not configured. `mongot` skips Voyage models if `queryKeyFil`e and `indexingKeyFile` are not set.

    **Resolution**

    No action is required if you are not using Voyage AI.

    If you want to use Voyage AI models, configure both `queryKeyFile` and `indexingKeyFile`.

    **Verify**

     Restart `mongot` and check that the Voyage model is loaded without the skip message.

2. 
    


    

### Before you troubleshoot further

Check the following before changing the Percona Search configuration:

* Confirm that the configured model exists on the embedding provider.
* Test the embedding endpoint from the same host or container where `mongot` runs.
* Check the `mongot` log for the first provider error rather than only the final index status.
* Verify the model name, endpoint, authentication settings, and model catalog together.
* Avoid using `localhost` in `providerEndpoint` unless the embedding server is reachable from the same network namespace as `mongot`.
* If you use Voyage AI, configure both `queryKeyFile` and `indexingKeyFile`.

## References

For related configuration and provider-specific information, see:

* [Configure `mongot`](https://www.mongodb.com/docs/search/self-managed/current/configuration/reference/)
* [Configure `mongot` for Automated Embedding](https://www.mongodb.com/docs/search/self-managed/current/configuration/automated-embedding/)
* [How Automated Embedding Works](https://www.mongodb.com/docs/vector-search/crud-embeddings/automated-embedding/overview/)
* [Ollama FAQ](https://docs.ollama.com/faq)
* [Azure OpenAI embeddings REST reference](https://learn.microsoft.com/en-us/rest/api/microsoft-foundry/azureopenai/embeddings)
