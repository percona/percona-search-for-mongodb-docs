# Percona Search for MongoDB documentation

Percona Search for MongoDB supports full-text and vector search for Percona Server for MongoDB deployments. Use it to build semantic search, keyword search, hybrid search, recommendations, and retrieval-augmented generation (RAG) applications without moving source documents to a separate database.

The `mongot` process maintains Lucene-based search indexes and executes search queries. Applications query Percona Server for MongoDB with `$search`, `$searchMeta`, and `$vectorSearch` aggregation stages.


!!! tip "New to Percona Search for MongoDB?"
    Review the requirements, install and configure `mongot`, create an index, and run a query.

    [Get started :material-arrow-right:](install-mongot.md){ .md-button .md-button--primary }

## :material-rocket-launch: Get started { .title }

### :material-sitemap: Plan your deployment { .title }

Understand the components, supported topologies, resource requirements, network connectivity, and current limitations before installing the search service.

[Architecture :material-arrow-right:](vector-search-architecture.md){ .md-button }
[Deployment :material-arrow-right:](vector-search-deployment.md){ .md-button }
[Compatibility and limitations :material-arrow-right:](vector-search-compatibility.md){ .md-button }

### :material-download: Install and configure `mongot` { .title }

Start the `mongot` service, connect it to Percona Server for MongoDB, and verify communication between the database and search components.

[Install and configure `mongot` :material-arrow-right:](install-mongot.md){ .md-button }

### :material-database-search: Create an index and run a query { .title }

Create the index required by your search type, wait until it becomes queryable, and run the corresponding aggregation pipeline.

[Create a search index :material-arrow-right:](create-search-index.md){ .md-button }
[Query with `$vectorSearch` :material-arrow-right:](query-vector-search.md){ .md-button }
[Query with `$search` :material-arrow-right:](query-search.md){ .md-button }

## :material-sign-direction: Choose the right search path { .title }

| What you want to do | Start with |
|---|---|
| Find documents by meaning or intent | [Vector search overview](vector-search-overview.md) and [`$vectorSearch`](query-vector-search.md) |
| Find words, phrases, or fuzzy matches | [Full-text search overview](overview-search.md) and [`$search`](query-search.md) |
| Return counts, facets, or other search metadata | [`$searchMeta`](query-searchmeta.md) |
| Combine semantic and keyword relevance | [Hybrid search](hybrid-search.md) |
| Build advanced full-text conditions | [Operators and collectors](operators.md) |
| Understand or tune result ranking | [Search scoring](scoring.md) |

## :material-creation: Choose an embedding approach { .title }

Vector search requires embeddings for indexed content and queries. Choose the workflow that fits your application and operational requirements.

| Approach | Responsibility | Where document embeddings are stored | Start here |
|---|---|---|---|
| Manual embeddings | The application calls an external or local embedding model, stores document vectors, generates query vectors, and submits `$vectorSearch` queries. | In application documents in Percona Server for MongoDB | [Manual embeddings](manual-embeddings.md) |
| Automatic embeddings with Voyage AI | `mongot` generates embeddings at index time and query time and keeps them synchronized as indexed text changes. | In an internal generated-embeddings collection managed by the search service | [Automatic embeddings](automatic-embeddings.md) |

!!! note
    Automatic embedding availability depends on the Percona Server for MongoDB and `mongot` versions in your deployment. Review [compatibility and limitations](vector-search-compatibility.md) before choosing this approach.

## :material-database-cog: Manage and operate search { .title }

### Manage indexes

Create, inspect, update, and delete full-text and vector search indexes. Index definitions are managed through Percona Server for MongoDB, while `mongot` maintains the corresponding search index data.

[Search index overview :material-arrow-right:](search-index-overview.md){ .md-button }
[Update an index :material-arrow-right:](update-search-index.md){ .md-button }
[Delete an index :material-arrow-right:](delete-search-index.md){ .md-button }

### Validate behavior and performance

Use representative data and queries to evaluate index readiness, relevance, filtering, latency, memory consumption, and result quality before moving a workload to production.

[Query and index guidance :material-arrow-right:](vector-search-best-practices.md){ .md-button }

## :material-book-open-page-variant: Upstream references { .title }

For MongoDB-compatible concepts and aggregation-stage syntax, see:

- [MongoDB Vector Search overview :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/){:target="_blank"}
- [Automated Embedding overview :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/crud-embeddings/automated-embedding/){:target="_blank"}
- [`$vectorSearch` aggregation stage :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/operator/aggregation/vectorsearch/){:target="_blank"}

