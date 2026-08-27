# Automatic embedding configuration reference

This page lists all the settings for automatic embedding. The settings are split across two files: `mongot.conf` enables the subsystem and holds the values that apply across all models, and the [model catalog](automatic-embedding-model-catalog.md) defines the models themselves.

## The embedding section in mongot.conf

| Setting | Required | Description |
|---|---|---|
| `isAutoEmbeddingViewWriter` | No | Controls whether this `mongot` instance writes generated embedding data. Default is `false`. Configure one writer for the relevant deployment. |
| `modelConfigFile` | No | Path to a custom embedding model catalog. |
| `queryKeyFile` | Voyage only | File containing the Voyage API key used for queries. |
| `indexingKeyFile` | Voyage only | File containing the Voyage API key used during indexing. |
| `providerEndpoint` | No | Overrides the endpoint for `VOYAGE` models only. It doesn't affect `OPENAI_COMPATIBLE` models. |

## OPENAI_COMPATIBLE model settings

| Setting | Required | Description |
|---|---|---|
| `modelName` | Yes | Model identifier used by index definitions and sent in the embedding request. |
| `config.providerEndpoint` | Recommended | Full URL of the OpenAI-compatible `/v1/embeddings` endpoint. |
| `config.modelConfig.outputDimensions` | Yes | Vector dimensions produced by the model. |
| `config.modelConfig.batchSize` | No | Maximum number of inputs in one embedding request. Default is `96`. |
| `config.modelConfig.batchTokenLimit` | No | Approximate token budget for each request batch. Default is `120000`. |
| `config.modelConfig.quantization` | No | Embedding representation. Only `float` is currently supported. |
| `config.modelConfig.queryPrefix` | No | Prefix added to query input for asymmetric models. |
| `config.modelConfig.documentPrefix` | No | Prefix added to document input for asymmetric models. |
| `config.modelConfig.forwardDimensions` | No | Sends the resolved vector dimension through the OpenAI `dimensions` field. Default is `false`. |
| `config.errorHandlingConfig` | Yes | Retry configuration for transient failures. |
| `config.credentials.apiKey` | No | API key for the embedding endpoint. Omit it for a keyless local engine. |
| `config.credentials.authHeaderName` | No | Authentication header. Defaults to `Authorization`. Set it to `api-key` for Azure OpenAI. |

## Retry settings

`config.errorHandlingConfig` accepts the following:

| Setting | Description |
|---|---|
| `maxRetries` | Maximum number of retry attempts for a transient failure. |
| `initialRetryWaitMs` | Wait before the first retry, in milliseconds. |
| `maxRetryWaitMs` | Upper bound on the wait between retries, in milliseconds. |
| `jitter` | Randomization applied to the retry wait, to avoid synchronized retries. |

Authentication failures aren't treated as transient and aren't retried.