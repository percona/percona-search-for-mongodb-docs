# Troubleshoot Percona Search for MongoDB

This page collects the problems you're most likely to hit with Percona Search for MongoDB, grouped by the area they affect. Each entry names the symptom you see, explains what causes it, and tells you what to change.

## Before you start

Most problems are diagnosed from three places. Have them to hand before you start changing configuration.

| Source | What it tells you | How to check |
|---|---|---|
| The `mongot` log | Whether `mongot` started cleanly, which models and indexes it loaded, and what it is retrying | Read the output from startup onward |
| Index status | Whether an index is building, ready, or stuck | [`db.collection.getSearchIndexes()` :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/method/db.collection.getsearchindexes/){:target="_blank"} |
| The metrics endpoint | Request volume and rejection counts, broken down by provider and workload | `curl -s localhost:9946/metrics`, if `metrics.enabled` is `true` |

!!! tip

    Check the `mongot` startup log before anything else. A configuration problem almost always announces itself there, and it saves you debugging a layer that was never the cause.

## Automatic embedding

Automatic embedding involves three moving parts: `mongot`, the model catalog, and the embedding engine. Most problems come down to one of them disagreeing with the other two, so work out which layer is at fault before you edit anything.

### Narrow down the layer

{.power-number}

1. **Did the model load?** Check the `mongot` startup log for the count of initialized models. If your model isn't in that count, the problem is the catalog or the configuration, and nothing downstream will work.
2. **Is the engine reachable from the `mongot` host?** Call the endpoint directly, from the machine `mongot` runs on:

    ```sh
    curl -s http://localhost:11434/v1/embeddings \
      -H "Content-Type: application/json" \
      -d '{"model": "nomic-embed-text", "input": ["hello"]}'
    ```

    A vector in the response means the engine and the model are fine, and the problem sits between `mongot` and the engine. No response means you have an engine or network problem, and no amount of catalog editing will help.

3. **Is the index building or stuck?** An index that never leaves `BUILDING` is usually waiting on embedding calls that keep failing.
4. **Are results wrong rather than absent?** That is a different class of problem. See [Search results are poor](#search-results-are-poor).

!!! tip

    Step 2 is the one people skip, and the one that saves the most time. `mongot` and the engine often run on different hosts or in different containers, so an endpoint that works from your laptop may be unreachable from `mongot`.

### Voyage models are skipped at startup

The log shows a message similar to:

```text
Skipping Voyage embedding model '<model>': no Voyage API credentials configured
```

This is expected if you haven't configured Voyage credentials, and it isn't an error. `mongot` skips models whose credentials are missing rather than refusing to start, so keyless `OPENAI_COMPATIBLE` models stay available.

Configure `queryKeyFile` and `indexingKeyFile` only if you need Voyage models.

### Automatic embedding is inactive

If nothing embeds at all and no embedding activity appears in the log, check these in order:

- **The `embedding` section is missing from `mongot.conf`.** Its presence is what activates the subsystem.
- **No instance is set as the writer.** `isAutoEmbeddingViewWriter` defaults to `false`. If you run several `mongot` instances and set it on none of them, no embedding data gets written. Set it to `true` on exactly one.
- **The catalog wasn't reloaded.** `mongot` reads the catalog at startup, so editing `embedding-service-configs.yml` has no effect until you restart.

### The custom catalog isn't being used

If you set `modelConfigFile` and the file is missing, unreadable, or invalid, automatic embedding stays inactive. `mongot` doesn't fall back to another catalog. This is deliberate: silently loading a different catalog could send your data to an unintended endpoint under a familiar model name.

Check that:

- The path is correct and readable by the account running `mongot`.
- The file is valid YAML with a top-level `configs` key. A common mistake is copying an entry out of the documentation and losing the `configs:` line above it, which leaves a bare list.
- Indentation is consistent. YAML that parses but nests a key one level too shallow produces an entry that looks right and doesn't work.

<!-- TBD-ENG (Radek): an earlier draft said a missing or malformed modelConfigFile makes mongot
     "fall back to the bundled catalog" with a warning. The catalog and reference pages both say
     the opposite: it fails closed with no fallback. These cannot both be true, and they lead to
     opposite debugging paths. I have documented fail-closed, since it appears in two places and
     has a stated rationale, but this needs confirming before publish. -->

### The index stays in PENDING or BUILDING

The log shows retries:

```text
Failed embedding call in retry time: N, retrying
```

`mongot` is reaching the point of calling the engine and failing. It retries according to `errorHandlingConfig`, so the index resumes on its own once the cause is fixed. You don't need to drop and recreate it.

For more detail than `getSearchIndexes()` gives you, run the [`$listSearchIndexes` :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/operator/aggregation/listSearchIndexes/){:target="_blank"} aggregation stage. Its `statusDetail` field reports status per `mongot` instance, which is what you want when only one node is failing.

!!! note

    A delay between creating an index and it becoming queryable is normal, and upstream documents it as expected behavior. Give the build time before treating it as stuck, particularly on a large collection where every document has to be embedded.

Common causes:

- The embedding engine isn't running.
- `providerEndpoint` points somewhere `mongot` can't reach. Confirm with the `curl` test above, run from the `mongot` host.
- The model isn't available on the engine. With Ollama this usually means it was never pulled.
- The model named in the index doesn't match any `modelName` in the catalog.

!!! note

    A build that retries is behaving correctly. A build that fails immediately is a different signal, because authentication and quantization errors fail fast rather than retrying. If you see no retry messages, look at the two sections below instead.

### Authentication fails

```text
Authentication failed (HTTP 401/403): check the API key and authHeaderName
```

Authentication errors aren't treated as transient, so `mongot` doesn't retry them. Failing immediately rather than after `maxRetries` attempts is itself useful: it tells you the problem is credentials, not connectivity.

Check that:

- `apiKey` is correct and valid for the resource.
- The auth scheme matches the provider. Azure OpenAI needs `authHeaderName: api-key`. Without it, `mongot` sends `Authorization: Bearer`, which Azure rejects.
- For Azure, the endpoint's `api-version` is one your deployment supports, and the deployment name is correct and cased correctly.

### Only float embeddings are supported

```text
OPENAI_COMPATIBLE provider currently supports only float embeddings
```

The index is asking for `int8` or binary quantization. The `OPENAI_COMPATIBLE` provider supports `float` only, which is narrower than upstream MongoDB, where Voyage-backed [Automated Embedding :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/crud-embeddings/automated-embedding/){:target="_blank"} also supports scalar and binary quantization.

Set `quantization: float`, or omit the setting to take the default.

### The engine rejects the request dimensions

Local engines generally serve one fixed vector dimension and reject requests that carry the OpenAI `dimensions` field.

Set `forwardDimensions` to `false`, or omit it, and make `outputDimensions` match the dimension the model actually returns. The `curl` test tells you that number: count the values in the response vector.

Use `forwardDimensions: true` only with models that accept the field, such as the OpenAI and Azure OpenAI `text-embedding-3` models.

### Search results are poor

The index built, queries return documents, and the matches are wrong. Nothing looks broken, which is what makes this one expensive to diagnose.

Check the asymmetric model prefixes first. Some models, `nomic-embed-text` among them, expect queries and documents to carry different task-instruction prefixes before they're embedded. Without `queryPrefix` and `documentPrefix` in the catalog entry, queries and documents land in different regions of the vector space and retrieval quality drops. The index still builds and queries still return results, so there is no error to find.

Consult the model's own documentation to confirm whether it needs prefixes. Include the separator in the value: `"search_query: "` is a prefix, `"search_query"` is not.

Also check that you haven't overridden the model at query time with an incompatible one. Upstream requires that a model named in the [`$vectorSearch` :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/query/aggregation-stages/vector-search-stage/){:target="_blank"} stage be compatible with the one used at index time. Embeddings from unrelated models aren't comparable, and the results look plausible rather than obviously wrong.

!!! important

    Changing `queryPrefix` or `documentPrefix` changes how documents are embedded. Existing embeddings were generated with the old setting, so the index needs rebuilding before the change applies to data that is already indexed.

### mongot can't reach Ollama on another host

Ollama binds to `127.0.0.1` by default, so it accepts connections only from its own machine. A `mongot` instance anywhere else, including another container on the same host, can't reach it.

Start Ollama with a wider bind address:

```sh
OLLAMA_HOST=0.0.0.0 ollama serve
```

Then set `providerEndpoint` in the catalog to an address `mongot` can reach, rather than `localhost`, and restart `mongot`.

## Related pages

- [Automatic embedding configuration reference](automatic-embedding-reference.md) lists every setting mentioned here.
- [Embedding model catalog](automatic-embedding-model-catalog.md) explains how a model name resolves to a provider.
- [Create and query an autoEmbed index](autoembed-index.md) covers the normal workflow.

## Learn more

Percona Search for MongoDB follows upstream MongoDB for index definitions, index status, and query syntax. The `OPENAI_COMPATIBLE` provider and the model catalog are Percona extensions, so upstream doesn't document them.

- [Automated Embedding :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/crud-embeddings/automated-embedding/){:target="_blank"}
- [How to Index Fields for Vector Search :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/index/vector-search-type/){:target="_blank"}
- [`$vectorSearch` aggregation stage :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/query/aggregation-stages/vector-search-stage/){:target="_blank"}
- [`$listSearchIndexes` aggregation stage :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/operator/aggregation/listSearchIndexes/){:target="_blank"}
- [`db.collection.getSearchIndexes()` :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/method/db.collection.getsearchindexes/){:target="_blank"}

