# Percona Search for MongoDB documentation

Percona Search for MongoDB adds full-text and vector search to self-managed Percona Server for MongoDB deployments. Use it to build keyword, semantic, and hybrid search, along with recommendation and retrieval-augmented generation (RAG) applications.

Search runs in a separate process called `mongot`. This process maintains Apache Lucene-based search indexes and executes the queries that you submit through the `$search`, `$searchMeta`, and `$vectorSearch` aggregation stages.

<div data-grid markdown><div data-banner markdown>

## :material-progress-download: Get started { .title }

Check the requirements, start the `mongot` service, connect it to Percona Server for MongoDB, and verify that the two components can communicate.

[Install and configure `mongot` :material-arrow-right:](install-mongot.md){ .md-button }

</div><div data-banner markdown>

## :material-sitemap: Understand the architecture { .title }

See how Percona Server for MongoDB, `mongot`, and the search indexes work together. Review component placement, communication flow, supported topologies, and deployment requirements.

[Explore the architecture :material-arrow-right:](vector-search-architecture.md){ .md-button }

</div><div data-banner markdown>

## :material-database-search: Create and manage search indexes { .title }

Create, inspect, update, and delete full-text and vector search indexes. Learn how `mongot` keeps an index synchronized with changes to its source collection.

[Search index overview :material-arrow-right:](search-index-overview.md){ .md-button }

</div><div data-banner markdown>

## :material-text-search: Run full-text searches { .title }

Query text fields with `$search` to match on keywords, phrases, and fuzzy terms. Use `$searchMeta` to return facets, counts, and other result metadata.

[Explore full-text search :material-arrow-right:](search-overview.md){ .md-button }

<!-- TBD-DOCS: confirm the filename of the full-text search overview page. -->

</div><div data-banner markdown>

## :material-vector-point: Run vector searches { .title }

Query embeddings with `$vectorSearch` to match on meaning rather than wording. Generate embeddings in your application for full control, or use automatic embeddings where your deployment supports them.

[Explore vector search :material-arrow-right:](vector-search-overview.md){ .md-button }

</div><div data-banner markdown>

## :material-clipboard-check: Check compatibility and limitations { .title }

Review the supported Percona Server for MongoDB versions and deployment types, and the search features, operators, and options that are not yet available.

[Compatibility and limitations :material-arrow-right:](vector-search-compatibility.md){ .md-button }

</div>
</div>