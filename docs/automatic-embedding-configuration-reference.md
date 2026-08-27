# Automatic embedding configuration reference

This page lists all the settings for automatic embedding. The settings are split across two files: `mongot.conf` enables the subsystem and holds the values that apply across all models, and the [model catalog](automatic-embedding-model-catalog.md) defines the models themselves.

## Embedding section in `mongot.conf`

| **Setting** | **Required** | **Description** |
|---|---|---|
| `isAutoEmbeddingViewWriter` | No | Controls whether this `mongot` instance writes generated embedding data. Default is `false`. Configure one writer for the relevant deployment. |
| `modelConfigFile` | No | Path to a custom embedding model catalog. |
| `queryKeyFile` | Voyage only | File containing the Voyage API key used for queries. |
| `indexingKeyFile` | Voyage only | File containing the Voyage API key used during indexing. |
| `providerEndpoint` | No | Overrides the endpoint for `VOYAGE` models only. It doesn't affect `OPENAI_COMPATIBLE` models. |

## OPENAI_COMPATIBLE model settings

| **Setting** | **Required** | **Description** |
| --- | --- | --- |
| `modelName` | Yes | Name of the embedding model. `mongot` sends this value in the request `model` field and uses it to match the model referenced in an `autoEmbed` index definition. |
| `config.providerEndpoint` | No. Default: `https://api.openai.com/v1/embeddings` | Full URL of the OpenAI-compatible `/v1/embeddings` endpoint. Configure this value for local engines or hosted providers that use a different endpoint. |
| `config.modelConfig.outputDimensions` | Yes | Number of dimensions in the vectors returned by the model. This value determines the vector dimensions used by the index. It is sent to the provider only when `forwardDimensions` is set to `true`. |
| `config.modelConfig.batchSize` | No. Default: `96` | Maximum number of inputs that `mongot` can include in a single embedding request. |
| `config.modelConfig.batchTokenLimit` | No. Default: `120000` | Approximate maximum number of input tokens that `mongot` can include in one embedding request batch. |
| `config.modelConfig.quantization` | No. Default: `float` | Vector output format. Only `float` is currently supported for `OPENAI_COMPATIBLE` models. |
| `config.modelConfig.queryPrefix` | No | Text added before query input for models that use different instructions for queries and documents. Include any required separator, such as `"search_query: "`, in the value. |
| `config.modelConfig.documentPrefix` | No | Text added before document input for asymmetric embedding models. Include any required separator, such as `"search_document: "`, in the value. |
| `config.modelConfig.forwardDimensions` | No. Default: `false` | When set to `true`, sends the resolved vector dimension to the provider using the OpenAI `dimensions` request field. Use this only with models that support configurable dimensions, such as OpenAI and Azure OpenAI `text-embedding-3` models. |
| `config.errorHandlingConfig` | Yes | Defines how `mongot` handles transient embedding request failures. Configure `maxRetries`, `initialRetryWaitMs`, `maxRetryWaitMs`, and `jitter`. |
| `config.credentials.apiKey` | No | API key used to authenticate with the embedding provider. Omit this setting, or use `credentials: {}`, for local engines that don't require authentication. |
| `config.credentials.authHeaderName` | No. Default: `Authorization` | HTTP header used to send the API key. With the default `Authorization` header, `mongot` uses the `Bearer <key>` scheme. Set this to `api-key` for Azure OpenAI. |

## Retry settings

`config.errorHandlingConfig` accepts the following:

| **Setting** | **Description** |
|---|---|
| `maxRetries` | Maximum number of retry attempts for a transient failure. |
| `initialRetryWaitMs` | Wait before the first retry, in milliseconds. |
| `maxRetryWaitMs` | Upper bound on the wait between retries, in milliseconds. |
| `jitter` | Randomization applied to the retry wait, to avoid synchronized retries. |

Authentication failures aren't treated as transient and aren't retried.