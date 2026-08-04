# Search overview

Percona Search for MongoDB adds full-text search and vector search to Percona Server for MongoDB (PSMDB) 8.3 and later. It runs `mongot`, a dedicated search service, alongside mongod to build search indexes on your collections and process search queries.

!!! warning "Technical Preview" 

    Percona Search for MongoDB is available as a technical preview. We recommend that early adopters use it for testing and evaluation only, and not in production environments.

You can create search indexes on your collections and use aggregation pipeline stages such as `$search`, `$searchMeta`, and `$vectorSearch `to retrieve relevant results.


## What is mongot?

[mongot :octicons-link-external-16:](https://www.mongodb.com/docs/manual/tutorial/mongot-sizing/advanced-guidance/architecture/){:target="_blank"} is a companion process that builds and maintains search indexes for your MongoDB collections. While `mongod` stores and manages your application data, `mongot` creates optimized search indexes and processes search queries.

The two services communicate internally during query execution:

- `mongod` stores documents and handles database operations.
- `mongot` maintains search indexes.
- Search queries are processed by `mongot`, while `mongod` retrieves the matching documents and returns the results to the client.

This architecture separates search workloads from core database operations while keeping the indexed data synchronized with the database.


## Search types

Percona Server for MongoDB supports the following search types. Choose the type that matches your query patterns:

| **Search type**  | **Query stages**| **Index type** | **Use for**|
|------------------|-----------------|----------------|------------|
| Full-text search | `$search`, `$searchMeta` | `search`| Relevance-ranked text queries, autocomplete, faceting, and highlighting |
| Vector search    | `$vectorSearch`| `vectorSearch` | Semantic similarity queries using machine learning embeddings|


### Full-text search

Full-text Search lets you search text stored in one or more fields using relevance-based ranking. It supports capabilities such as:

- Keyword and phrase searches
- Boolean operators
- Fuzzy matching
- Wildcard searches
- Field-specific searches
- Relevance scoring

Use cases include:

- Product catalog search
- Documentation search
- Blog and article search
- Customer support knowledge bases

## Vector search

Vector search finds documents based on the meaning behind your data, rather than exact keyword matches. It compares the vector embedding of your query against the embeddings stored in your collection, and returns the documents whose embeddings are closest in meaning. It supports capabilities such as:

- Approximate nearest neighbor (ANN) search, which uses an HNSW index to find close matches quickly across large datasets
- Exact nearest neighbor (ENN) search, which compares every indexed vector for guaranteed accuracy on smaller datasets
- Pre-filtering, so you can narrow the documents considered before the vector comparison runs
- Manual embeddings, so you can generate vector embeddings with an embedding model of your choice and store them alongside your documents

Use cases include:

- Semantic search over product descriptions or documentation
- Recommendation systems based on similarity
- Retrieval-augmented generation (RAG) for AI applications
- Image or audio similarity search, once the underlying content is converted to vector embeddings

