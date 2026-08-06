# Full-text search

## Overview

Percona Search for MongoDB provides integrated full-text search for data stored in Percona Server for MongoDB. It enables you to build scalable, relevance-based search features without deploying and maintaining a separate search platform. 

Full-text search performs lexical matching on indexed text. It supports:

- Relevance-based ranking

- Autocomplete

- Filtering

- Highlighting

- Faceting

For semantic similarity search, use [vector search with embeddings](embeddings.md), which uses vector embeddings to find documents based on meaning rather than exact keyword matches.

## How full-text search works

`mongot` builds and maintains search indexes.
{.power-number}

1. `mongod` forwards the index definition to mongot.

2. `mongot` builds the search index asynchronously.

3. After the index reaches the `READY` state, it becomes queryable.

4. `mongot` automatically synchronizes the index with document inserts, updates, and deletes.

## Next steps

[Create a search index :material-arrow-right:](create-search-index.md){.md-button}