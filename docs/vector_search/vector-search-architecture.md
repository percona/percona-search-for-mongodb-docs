# Percona Search for MongoDB architecture

The architecture describes the components that provide search and how data, search requests, and results move between them. It is independent of the number of replica set members or shards in the deployment.


## Core components

|Component|Purpose|
|---|---|
|Application|Connects to `mongod` in a replica set or to `mongos` in a sharded cluster. It never connects directly to `mongot`.|
|`mongod`|Stores application data, forwards search requests to `mongot`, and retrieves matching documents.|
|`mongos`|Routes search requests across shards and merges the results.|
|`mongot`|Maintains Lucene-based indexes and executes full-text and vector searches.|
|Search index storage|Stores search indexes separately from the database files managed by `mongod`.|
|Embedding model|Converts content into vectors for semantic search.|
|Generative LLM|Uses retrieved documents as context to generate responses in a RAG application.|

## Communication between mongod and mongot

Applications never communicate with `mongot` directly. They send database and search operations to `mongod`, or to `mongos` when the cluster is sharded.

Communication between Percona Server for MongoDB and Percona Search for MongoDB works in both directions:

- `mongod` to `mongot`: `mongod` forwards search queries and search index management commands over gRPC.

- `mongot` to `mongod`: `mongot` reads source documents and change events to build and refresh its indexes.

Use TLS for communication between the database and search processes. The mongot process must also authenticate to the database with the privileges required to read source data, monitor changes, and manage search indexes.

## Search query processing

A search query follows this general path:
{.power-number}

1. The application sends a `$search`, `$searchMeta`, or `$vectorSearch` aggregation to `mongod` or `mongos`.

2. A `mongod` member that owns the relevant data forwards the search stage to mongot over gRPC.

3. `mongot` searches its local Lucene-based index.

4. `mongot` returns matching document identifiers and relevance scores to `mongod`.

5. `mongod` loads the corresponding documents from the collection.

6. In a sharded cluster, mongos combines the partial results from the shards.

7. The application receives the final result set.

## Data synchronization
`mongot` does not store the primary copy of your data. Instead, it maintains vector indexes that are synchronized with the collections stored in `mongod`. Whenever documents are inserted, updated, or deleted, the corresponding vector indexes are updated automatically. This synchronization ensures that vector search queries operate on current data without requiring manual index maintenance.


