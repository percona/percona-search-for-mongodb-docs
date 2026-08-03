Percona Search for MongoDB documentation

Percona Search for MongoDB adds full-text and vector search capabilities to self-managed Percona Server for MongoDB deployments. Use it to build semantic search, keyword search, hybrid search, recommendations, and retrieval-augmented generation (RAG) applications.

The `mongot` process maintains Lucene-based search indexes and executes queries submitted through the $search, $searchMeta, and $vectorSearch aggregation stages.

!!! tip ""Applications connect to `mongod` in a replica set or to `mongos` in a sharded cluster. They never connect directly to `mongot`. Percona Server for MongoDB sends search requests to `mongot` and returns the retrieved documents to the application.

[Compatibility and limitations :material-arrow-right:](vector-search-compatibility.md){ .md-button }

<style>
  .search-home-cards > ul > li {
    display: flex;
    min-height: 16rem;
    flex-direction: column;
  }

  .search-home-cards > ul > li > p:last-child {
    margin-top: auto;
    margin-bottom: 0;
  }

  .search-home-cards .md-button {
    white-space: nowrap;
  }

  @media screen and (max-width: 44.9844em) {
    .search-home-cards > ul > li {
      min-height: auto;
    }
  }
</style>

<div class="grid cards search-home-cards" markdown>

:material-progress-download: Get started

Review the requirements, start the mongot service, configure its connection to Percona Server for MongoDB, and verify that the components can communicate.

Install and configure mongot :material-arrow-right:{ .md-button }

:material-sitemap: Understand architecture and deployment

Learn how Percona Server for MongoDB, mongot, and the search index work together. Review component placement, communication flow, topologies, and deployment requirements.

Explore the architecture :material-arrow-right:{ .md-button }

:material-database-search: Create and manage search indexes

Create, inspect, update, and delete full-text and vector search indexes. Learn how mongot synchronizes indexes with changes to the source collections.

Search index overview :material-arrow-right:{ .md-button }

:material-magnify: Build search experiences

Search by keywords, semantic meaning, or both. Use manual embeddings for application-level control or automatic embeddings when supported by your deployment.

Explore vector search :material-arrow-right:{ .md-button }

</div>