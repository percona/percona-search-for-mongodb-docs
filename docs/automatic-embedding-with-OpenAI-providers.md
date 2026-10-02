# Automatic embedding with OpenAI-Compatible Providers

With the `OPENAI_COMPATIBLE` provider, `mongot` can generate embeddings using any service that exposes an OpenAI-compatible `/v1/embeddings` endpoint, whether it is a hosted service or an embedding server that you run yourself.

This page covers what is specific to OpenAI-compatible providers. For the list of all supported providers, how automatic embedding works, and the general configuration procedure, see [Automatic embedding configuration](configure-automatic-embedding-openai.md).

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

Before configuring an OpenAI-compatible provider, review these settings and requirements:

| **Setting**  | **Supported values and requirements** |
| ---------| ----------------------------------|
| `numDimensions`              | Supported values are `256`, `512`, `1024`, and `2048`. |
| OpenAI `dimensions` field    | Local engines commonly return vectors with a fixed dimension and may reject requests that include the OpenAI `dimensions` field.         |
| `embedding.providerEndpoint` | The global override applies to **VOYAGE** models only. Each `OPENAI_COMPATIBLE` model defines its own `providerEndpoint` in the catalog. |
| API key                      | Local engines can run without an API key when authentication isn't configured.|

For details about these settings and their defaults, see the [Automatic embedding configuration reference](automatic-embedding-configuration-reference.md).

## Next steps

Choose the engine you want to connect:

[Configure automatic embedding with Ollama :material-arrow-right:](configure-automatic-embedding-ollama.md){.md-button} 

[Configure automatic embedding with OpenAI :material-arrow-right:](configure-automatic-embedding-openai.md#procedure){.md-button} 

[Configure automatic embedding with Azure OpenAI :material-arrow-right:](configure-automatic-embedding-openai-azure.md){.md-button}

To use models hosted by Hugging Face without an OpenAI-compatible endpoint, use the dedicated `HUGGINGFACE_INFERENCE` provider:

[Configure automatic embedding with the Hugging Face Inference API :material-arrow-right:](configure-automatic-embedding-huggingface.md){.md-button}

## Learn more

- [Automated Embedding :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/crud-embeddings/automated-embedding/){:target="_blank"}

- [How Automated Embedding Works :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/crud-embeddings/automated-embedding/overview/){:target="_blank"}

- [How to Index Fields for Vector Search :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/index/vector-search-type/){:target="_blank"}