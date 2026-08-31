# Automatic embedding with OpenAI-Compatible Providers

Percona Search for MongoDB supports automatic embedding with Voyage AI and OpenAI-compatible embedding providers. With the `OPENAI_COMPATIBLE` provider, `mongot` can generate embeddings using services that expose an OpenAI-compatible `/v1/embeddings` endpoint.

## OpenAI-compatible embedding providers

!!! note
    The following table lists common examples. The ports are typical defaults. Check the configuration of your embedding server before using them.

| **Engine**| **Default endpoint** | **Authentication**|
| ------| -----------------| --------------|
| [Ollama :octicons-link-external-16:](https://ollama.com/){:target="_blank"}               | `http://localhost:11434/v1/embeddings` | Not required by default       |
| vLLM                                         | `http://localhost:8000/v1/embeddings`  | Not required by default       |
| llama.cpp server                             | `http://localhost:8080/v1/embeddings`  | Not required by default       |
| LM Studio                                    | `http://localhost:1234/v1/embeddings`  | Not required by default       |
| LocalAI                                      | `http://localhost:8080/v1/embeddings`  | Not required by default       |
| Hugging Face Text Embeddings Inference (TEI) | `http://localhost:8080/v1/embeddings`  | Not required by default       |
| OpenAI                                       | `https://api.openai.com/v1/embeddings` | `Authorization: Bearer <key>` |
| Azure OpenAI                                 | Deployment-specific endpoint           | `api-key: <key>`              |

## What to know before you start

Automatic embedding uses two configuration files. The embedding section in `mongot.conf` defines settings shared across all models. The model catalog, `embedding-service-configs.yml`, defines each embedding model and contains most model-specific settings.

Before configuring automatic embedding, review these settings and provider requirements:

- Only **float** vector output is currently supported. `int8` and binary quantization aren't supported.
- `outputDimensions` in the catalog must match the dimensions returned by the model, unless you set `forwardDimensions: true`.
- Local engines commonly return vectors with a fixed dimension. They can reject requests that contain the OpenAI `dimensions` field.
- The global `embedding.providerEndpoint` override applies to **VOYAGE models only**. Each `OPENAI_COMPATIBLE` model carries its own `providerEndpoint` in the catalog.
- Local engines can run without an API key when authentication isn't configured.

For details about these settings and their defaults, see the [Automatic embedding configuration reference](automatic-embedding-configuration-reference.md).

## How automatic embedding works

### Overview

With manual embedding, your application is responsible for generating embeddings before storing or querying data. However, automatic embedding moves this work to `mongot`.

When you create a vector search index with an `autoEmbed` field, `mongot` embeds the indexed text during the initial collection scan and then keeps embedding it as documents change, using change streams. At query time, the text you pass to `$vectorSearch` goes through the same model, with the model's query prefix applied if one is configured, and is matched against the stored vectors.

![Embedding request flow between PSMDB, mongot, and the engine](_images/autoembed-with-OpenAI-compatible-providers.png)

### How `mongot` selects an embedding provider

The embedding provider isn't selected globally in `mongot.conf`. It is configured for each model in `embedding-service-configs.yml`.

![Model name resolving through the catalog to an embedding client](_images/embedding-client-flow.png)

`mongot` resolves the embedding provider and settings from the model name:
{.power-number}

1. The index definition names a model, for example `{ type: "autoEmbed", model: "nomic-embed-text", ... }`.
2. `mongot` finds the catalog entry whose `modelName` matches.
3. The entry's `embeddingProvider` field, either `VOYAGE` or `OPENAI_COMPATIBLE`, decides which client handles the traffic. Everything else the client needs, including `providerEndpoint`, credentials, prefixes, and batching, comes from the same entry.

!!! note
    Both provider types can be configured on the same `mongot` instance. When you create an `autoEmbed` index, the model name determines which provider and model configuration `mongot` uses. The embedding section in `mongot.conf` contains settings shared across the automatic embedding setup, such as the model catalog path and Voyage-specific configuration.


## Next steps

Choose the engine you want to connect:

[Configure automatic embedding with Ollama :material-arrow-right:](configure-automatic-embedding-ollama.md){.md-button} 

[Configure automatic embedding with OpenAI :material-arrow-right:](configure-automatic-embedding-openai.md){.md-button} 

[Configure automatic embedding with Azure OpenAI :material-arrow-right:](configure-automatic-embedding-openai-azure.md){.md-button}

## Learn more

- [Automated Embedding :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/crud-embeddings/automated-embedding/){:target="_blank"}

- [How Automated Embedding Works :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/crud-embeddings/automated-embedding/overview/){:target="_blank"}

- [How to Index Fields for Vector Search :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/index/vector-search-type/){:target="_blank"}









