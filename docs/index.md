# Percona Search for MongoDB documentation

!!! warning "Technical Preview"
    Percona Search for MongoDB 1.70.3-1 is available as a technical preview.

    We recommend that early adopters use this release for testing purposes only and not in production environments.

Percona Search for MongoDB adds full-text and vector search to self-managed Percona Server for MongoDB deployments. Use it to build keyword, semantic, and hybrid search, along with recommendation and retrieval-augmented generation (RAG) applications.

Search runs in a separate process called `mongot`. This process maintains Apache Lucene-based search indexes and executes the queries that you submit through the `$search`, `$searchMeta`, and `$vectorSearch` aggregation stages.

[What's new in Percona Search for MongoDB {{release}}](release_notes/{{release}}.md){ .md-button .md-button }

<div data-grid markdown><div data-banner markdown>

## :material-progress-download: Get started { .title }

Check the requirements, start the `mongot` service, connect it to Percona Server for MongoDB, and verify that the two components can communicate.

[Install and configure `mongot` :material-arrow-right:](install-mongot.md){ .md-button }

</div><div data-banner markdown>

## :material-sitemap: Understand the architecture { .title }

See how `mongod`, `mongot`, and your search indexes work together, from a single query's path through the system to how a sharded cluster splits the work.

[Explore the architecture :material-arrow-right:](vector-search-architecture.md){ .md-button }

</div><div data-banner markdown>

## :material-database-search: Create and manage search indexes { .title }

Create, check the status of, update, and delete full-text and vector search indexes as your data and query needs change.

[Search index overview :material-arrow-right:](search-indexes.md){ .md-button }

</div><div data-banner markdown>

## :material-magnify: Build search experiences { .title }

Query by keyword, by semantic meaning, or combine both in a single pipeline. Percona Search for MongoDB currently works with embeddings you generate yourself or automatically generated, giving your application full control over the embedding model and workflow.

[Explore vector search :material-arrow-right:](vector-search-overview.md){ .md-button }

</div>
</div>