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

## Component interaction

Applications send aggregation pipelines containing `$search`, `$searchMeta`, or `$vectorSearch` stages to `mongod`, or to `mongos` in a sharded cluster.

`mongod` forwards the search stage to `mongot`, which executes the search against its local indexes and returns the identifiers of the matching documents together with their relevance scores. `mongod` retrieves the corresponding documents from the collection, applies any remaining aggregation stages, and returns the results to the application.

In a sharded deployment, each shard performs the search independently. `mongos` merges the results from all shards before returning the final response.
