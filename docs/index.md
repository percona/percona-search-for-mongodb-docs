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

[Search index overview :material-arrow-right:](search-indexes.md){ .md-button }

</div><div data-banner markdown>

## :material-magnify: Build search experiences { .title }

Search by keywords, semantic meaning, or both. Use manual embeddings for application-level control or automatic embeddings when supported by your deployment.

[Explore vector search :material-arrow-right:](vector-search-overview.md){ .md-button }

</div>
</div>