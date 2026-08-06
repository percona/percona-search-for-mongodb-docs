# Percona Search for MongoDB architecture

Percona Search for MongoDB extends Percona Server for MongoDB with integrated full-text and vector search capabilities. Search indexes are maintained by `mongot`, while application data remains stored in `mongod`.

Applications continue to connect to MongoDB in the same way they do for other database operations. Search requests are routed internally to `mongot`, and the matching documents are returned through `mongod`.

## Architecture components

| Component | Description |
|-----------|-------------|
| Application | Connects to `mongod` in a replica set or to `mongos` in a sharded cluster. Applications never connect directly to `mongot`. |
| `mongod` | Stores application data, coordinates search requests, and retrieves matching documents from the collection. |
| `mongos` | Routes search requests to the appropriate shards and merges the results before returning them to the client. Available only in sharded clusters. |
| `mongot` | Builds and maintains Lucene-based search indexes and executes full-text and vector search queries. |
| Search indexes | Stored separately from the data files managed by `mongod` and maintained automatically by `mongot`. |
| Embedding model | Generates embeddings for semantic search. Applications are responsible for generating embeddings in the current release. |

## Component interaction in a sharded cluster

In a replica set, `mongod` forwards a `$search`, `$searchMeta`, or `$vectorSearch` stage to `mongot` and assembles the final result, as described in [Search overview](search-overview.md). Sharded clusters work the same way at each shard, with `mongos` handling the extra step of combining results across shards.

Each shard runs its own `mongot`, which indexes only the data on that shard. When a query reaches `mongos`, it forwards the search stage to every shard, and each shard's `mongod` and `mongot` execute the search locally. `mongos` then merges the results from all shards into a single response before returning it to the application.

If you add a search index to a collection that's already sharded, or add a new shard to a collection that already has a search index, you may see incomplete search results for that shard until its `mongot` finishes an initial sync.

