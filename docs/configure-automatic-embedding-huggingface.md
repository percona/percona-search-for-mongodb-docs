# Configure automatic embedding with the Hugging Face Inference API

You can use embedding models hosted by Hugging Face with Percona Search for MongoDB through the `HUGGINGFACE_INFERENCE` provider. You don't need to run your own embedding server, and the monthly credits included with every Hugging Face account let you get started at no cost.

`mongot` sends document and query text to the Hugging Face [Inference Providers :octicons-link-external-16:](https://huggingface.co/docs/inference-providers/index){:target="_blank"} router, which forwards it to the `hf-inference` feature-extraction pipeline:

```text
https://router.huggingface.co/hf-inference/models/<model-id>/pipeline/feature-extraction
```

Hugging Face returns one vector for each input. `mongot` uses these vectors for indexing and for vector search queries.

!!! note "`HUGGINGFACE_INFERENCE` or `OPENAI_COMPATIBLE`?"

    Use `HUGGINGFACE_INFERENCE` for the hosted Hugging Face Inference API, for a dedicated Hugging Face Inference Endpoint, or for the native `/embed` route of a self-hosted Text Embeddings Inference (TEI) server.

    If you run TEI yourself and prefer its OpenAI-compatible `/v1/embeddings` route, you can use the `OPENAI_COMPATIBLE` provider instead. See [Automatic embedding with OpenAI-compatible providers](automatic-embedding-with-OpenAI-providers.md).

## Before you begin

Make sure that:

- Percona Server for MongoDB and Percona Search for MongoDB are configured and running.
- The `mongot` host has network access to `https://router.huggingface.co`.
- You have a [Hugging Face account :octicons-link-external-16:](https://huggingface.co/join){:target="_blank"}.
- You picked an embedding model that the `hf-inference` provider serves for feature extraction.
- You know the native output dimension of the model.

The examples on this page use [`BAAI/bge-small-en-v1.5` :octicons-link-external-16:](https://huggingface.co/BAAI/bge-small-en-v1.5){:target="_blank"}, which produces 384-dimensional vectors.

### Choose a model

The `hf-inference` provider serves a curated set of CPU models. Most of them are popular sentence-transformers models, such as `BAAI/bge-small-en-v1.5` or `sentence-transformers/all-MiniLM-L6-v2`. It doesn't serve every embedding model on the Hugging Face Hub.

To see which models are available, browse the [feature-extraction models served by hf-inference :octicons-link-external-16:](https://huggingface.co/models?inference_provider=hf-inference&pipeline_tag=feature-extraction){:target="_blank"}.

Choose a model that meets these requirements:

- **It is a sentence embedding model.** The model must have a pooling layer so that it returns one vector for each input, not one vector for each token.
- **You know its native dimension.** Look for `hidden_size` in the model's `config.json` file on the Hub. For example, `BAAI/bge-small-en-v1.5` has `hidden_size: 384`.

### Free tier and billing

Inference Providers is billed pay-as-you-go, and every account gets a monthly allowance of included credits. Free accounts get a small allowance. PRO, Team, and Enterprise accounts get a larger one. `hf-inference` bills by compute time multiplied by the hardware price.

Embedding requests to small CPU models are inexpensive. However, the initial sync of a large collection sends every document to the model and can use up a free allowance quickly.

For current limits and prices, see [Inference Providers pricing :octicons-link-external-16:](https://huggingface.co/docs/inference-providers/pricing){:target="_blank"}. You can track your usage on your [Hugging Face billing page :octicons-link-external-16:](https://huggingface.co/settings/billing){:target="_blank"}.

When your credits run out, Hugging Face rejects requests with `HTTP 402 (Payment Required)`. `mongot` doesn't retry these requests. Add credits or upgrade your plan to resume embedding.

## Procedure

To configure automatic embedding with the Hugging Face Inference API, do the following:
{.power-number}

1. Create a Hugging Face access token.

    In your Hugging Face account, go to [Access Tokens :octicons-link-external-16:](https://huggingface.co/settings/tokens){:target="_blank"} and create a **fine-grained** token with the **Make calls to Inference Providers** permission.

    Verify that the endpoint responds, that your token works, and that `hf-inference` serves your model:

    ```sh
    curl -s https://router.huggingface.co/hf-inference/models/BAAI/bge-small-en-v1.5/pipeline/feature-extraction \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer <your-hugging-face-access-token>" \
      -d '{
        "inputs": ["hello"]
      }'
    ```

    A successful response is a JSON array that contains one vector of 384 numbers. If you receive a nested array with one vector per token, the model has no pooling layer. Choose a sentence embedding model instead.

2. Enable automatic embedding in `mongot`.

    Add an `embedding` section to the active `mongot` configuration file. For the documented systemd installation, edit `/etc/mongot/config.yml`. The presence of this section activates automatic embedding:

    ```yaml
    embedding:
      isAutoEmbeddingViewWriter: true
    ```

    !!! info "Important"

        Set `isAutoEmbeddingViewWriter: true` on exactly one `mongot` node. That node writes the embedding materialized view. If several `mongot` instances serve the same data, configure only one of them as the writer.

3. Configure the model in the catalog.

    Add an entry to `embedding-service-configs.yml` with the model and your access token. The default catalog contains a commented version of this example:

    ```yaml
    configs:
      - modelName: bge-small-en-v1.5
        embeddingProvider: HUGGINGFACE_INFERENCE
        config:
          modelConfig:
            # Exact, case-sensitive Hugging Face Hub repository ID
            modelId: BAAI/bge-small-en-v1.5
            batchSize: 32
            batchTokenLimit: 120000
            outputDimensions: 384
            quantization: float

          errorHandlingConfig:
            maxRetries: 10
            initialRetryWaitMs: 500
            maxRetryWaitMs: 30000
            jitter: 0.1

          credentials:
            apiToken: "<your-hugging-face-access-token>"
    ```

    The entry uses two different model names:

    - `modelName` is the name that you reference in `autoEmbed` index definitions. `mongot` converts it to lowercase.
    - `modelConfig.modelId` is the exact Hugging Face Hub repository ID, which is case-sensitive. Set it whenever the repository ID includes an organization prefix, such as `BAAI/`, or uppercase letters. If you omit it, `mongot` uses `modelName`.

    The `modelConfig`, `errorHandlingConfig`, and `credentials` sections are required. If you omit `batchSize` or `batchTokenLimit`, `mongot` uses `32` and `120000`, the values shown in the example.

    !!! info "Important"

        The catalog now holds a secret. Restrict the file so that only the account running `mongot` can read it, and keep it out of version control.

4. Set the vector dimension and quantization.

    The Hugging Face Inference API always returns vectors in the model's native dimension. Set `outputDimensions` to that value:

    ```yaml
    modelConfig:
      outputDimensions: 384
      quantization: float
    ```

    `mongot` uses `outputDimensions` to size the index. Always set it: if you omit it, `mongot` uses `1024`. If `numDimensions` is set in the index definition, it takes precedence over `outputDimensions`. If the model returns vectors of a different size, embedding fails with an error such as `returned N-dimensional vectors but the index expects M`.

    The `HUGGINGFACE_INFERENCE` provider supports only `float` vectors. Don't set a different quantization in the catalog or in the index definition.

5. (Optional) Configure model-specific settings.

    Depending on the model, you can add the following settings to `modelConfig`:

    - `truncate`: Defaults to `true`. Hugging Face truncates inputs that are longer than the model's maximum sequence length, instead of failing the whole batch. For example, the maximum sequence length of `bge-small-en-v1.5` is 512 tokens.
    - `normalize`: `mongot` sends this setting only when you set it. Otherwise, the server default applies. `autoEmbed` indexes with `float` vectors use dot-product similarity by default, which expects normalized vectors. If you set `normalize: false`, also set `similarity` to `cosine` or `euclidean` in the index definition.
    - `queryPrefix` and `documentPrefix`: Text that `mongot` adds before query input and document input. Asymmetric models need these. For example, e5 models expect `"query: "` and `"passage: "`. Include the separator in the value.

6. Restart `mongot` after you change the catalog:
    ```sh
    systemctl restart mongot
    ```

    Confirm that your Hugging Face model appears in the loaded model count. Warnings about skipped Voyage models are expected if you haven't configured Voyage credentials.

## Use a dedicated Inference Endpoint or a self-hosted TEI server

By default, `mongot` sends requests to the shared Hugging Face router. To send them to a different server, set `providerEndpoint` in the catalog entry. `mongot` uses this URL exactly as you specify it.

Dedicated Hugging Face Inference Endpoints that run Text Embeddings Inference (TEI), and self-hosted TEI servers, use the same request and response format on their `/embed` route.

=== "Dedicated Inference Endpoint"

    Use the `/embed` route of your endpoint and keep your access token in `credentials.apiToken`:

    ```yaml
    configs:
      - modelName: bge-small-en-v1.5
        embeddingProvider: HUGGINGFACE_INFERENCE
        config:
          providerEndpoint: https://<your-endpoint>.endpoints.huggingface.cloud/embed
          modelConfig:
            modelId: BAAI/bge-small-en-v1.5
            outputDimensions: 384
            quantization: float
          errorHandlingConfig:
            maxRetries: 10
            initialRetryWaitMs: 500
            maxRetryWaitMs: 30000
            jitter: 0.1
          credentials:
            apiToken: "<your-hugging-face-access-token>"
    ```

=== "Self-hosted TEI"

    Start a TEI server that serves your model:

    ```sh
    docker run -p 8080:80 ghcr.io/huggingface/text-embeddings-inference:cpu-1.8 \
      --model-id BAAI/bge-small-en-v1.5
    ```

    Point `providerEndpoint` at the `/embed` route. A token is optional. If you omit `credentials`, `mongot` loads the model and doesn't send an `Authorization` header:

    ```yaml
    configs:
      - modelName: bge-small-en-v1.5
        embeddingProvider: HUGGINGFACE_INFERENCE
        config:
          providerEndpoint: http://<tei-host>:8080/embed
          modelConfig:
            modelId: BAAI/bge-small-en-v1.5
            outputDimensions: 384
            quantization: float
          errorHandlingConfig:
            maxRetries: 10
            initialRetryWaitMs: 500
            maxRetryWaitMs: 30000
            jitter: 0.1
    ```

!!! note

    The global `embedding.providerEndpoint` setting in the `mongot` configuration file applies to `VOYAGE` models only. It doesn't affect `HUGGINGFACE_INFERENCE` models.

## Troubleshooting

| **Symptom** | **Cause and fix** |
| --- | --- |
| `Skipping Hugging Face embedding model '…': no credentials.apiToken configured` at startup | The catalog entry has no token, or the token is empty, and no `providerEndpoint` is set. Add `credentials.apiToken`. For a keyless self-hosted TEI server, set `providerEndpoint` instead. |
| `Authentication failed (HTTP 401/403)` | The token is invalid or lacks the required permission. Use a fine-grained token with the **Make calls to Inference Providers** permission. If you set a custom `providerEndpoint`, the server might require a token that the entry doesn't have. |
| `Payment required (HTTP 402)` | Your monthly Inference Providers credits are used up. Add credits or upgrade your plan. |
| `Got client error (HTTP 404)` | `hf-inference` doesn't serve this model for feature extraction, or `modelId` has the wrong case or organization prefix. |
| `Got client error (HTTP 413)` | The request is too large. Lower `batchSize` or `batchTokenLimit`. |
| `Got invalid request, fail fast and give up retries` (HTTP 400 or 422) | The server rejected the batch. `mongot` doesn't retry it, and the affected documents get no embedding. Check the response body in the message for details. |
| `Rate limit exceeded (HTTP 429)` | Hugging Face is throttling requests. `mongot` retries these requests with backoff, as defined in `errorHandlingConfig`. |
| `Got non OK status (HTTP 5xx)`, such as HTTP 503 | The server is overloaded, or a cold model is still loading. `mongot` retries these requests with backoff, as defined in `errorHandlingConfig`. `mongot` also retries timeouts. Each request times out after 60 seconds. |
| `HUGGINGFACE_INFERENCE provider supports only float embeddings` | The catalog entry or the index definition uses a quantization other than `float`. Set `quantization: float`. |
| `Model returned token-level embeddings` | The model has no pooling layer. Choose a sentence embedding model. |
| `returned N-dimensional vectors but the index expects M` | `outputDimensions`, or `numDimensions` in the index definition, doesn't match the model's native dimension. |

To check embedding traffic for this provider, filter the `mongot` metrics by the provider tag:

```sh
curl -s localhost:9946/metrics | grep 'provider="HUGGINGFACE_INFERENCE"'
```

The Hugging Face API doesn't report token usage, so the `inputTokenDistribution` metric isn't populated for this provider.

For other automatic embedding issues, see [Troubleshoot automatic embedding](troubleshooting.md#automated-embedding).

## Next steps

Reference the catalog `modelName` from an `autoEmbed` field, for example `model: "bge-small-en-v1.5"`, and query the index with `$vectorSearch`.

[Create and query an autoEmbed index :material-arrow-right:](autoembed-index.md){.md-button}
