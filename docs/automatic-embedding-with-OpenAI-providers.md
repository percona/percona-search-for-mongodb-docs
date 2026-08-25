# Automatic Embedding with OpenAI-Compatible Providers

Percona Search for MongoDB can now generate vector embeddings using any embedding server that implements the OpenAI `/v1/embeddings` API. This is enabled by the new `OPENAI_COMPATIBLE` embedding provider.

## Why this matters

Until now, automatic embedding (autoEmbed vector search indexes) required a Voyage AI API key, which meant relying on a third-party cloud service with per-token costs.

With the `OPENAI_COMPATIBLE` provider, you have more flexibility in how and where embeddings are generated. You can:

- Run embedding models locally to keep data within your infrastructure and avoid usage-based API costs.

- Use hosted services such as OpenAI or Azure OpenAI.

## OpenAI-compatible embedding providers

The `OPENAI_COMPATIBLE` provider works with embedding services that implement the OpenAI embeddings API.


!!! note
    The following table shows common examples. Ports are typical defaults. Check the configuration of your embedding server before using them.

| Engine| Typical endpoint | Authentication|
| ------| -----------------| --------------|
| [Ollama :octicons-link-external-16:](https://ollama.com/){:target="_blank"}               | `http://localhost:11434/v1/embeddings` | Not required by default       |
| vLLM                                         | `http://localhost:8000/v1/embeddings`  | Not required by default       |
| llama.cpp server                             | `http://localhost:8080/v1/embeddings`  | Not required by default       |
| LM Studio                                    | `http://localhost:1234/v1/embeddings`  | Not required by default       |
| LocalAI                                      | `http://localhost:8080/v1/embeddings`  | Not required by default       |
| Hugging Face Text Embeddings Inference (TEI) | `http://localhost:8080/v1/embeddings`  | Not required by default       |
| OpenAI                                       | `https://api.openai.com/v1/embeddings` | `Authorization: Bearer <key>` |
| Azure OpenAI                                 | Deployment-specific endpoint           | `api-key: <key>`              |

