# Vector search overview

## What is a vector database?

A vector database stores, indexes, and searches vector embeddings. A vector embedding is an array of numbers that represents the characteristics or semantic meaning of data, such as text, images, or audio. An embedding model generates these vectors. The database stores and indexes them for retrieval.

When you run a vector search, the application converts the search input into a query vector. The database compares this vector with the indexed embeddings and returns the documents whose vectors are closest to it. A smaller distance generally indicates greater semantic similarity.

For more information, see the [MongoDB Vector Search overview :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/){:target="_blank"}.

## How do vector databases work?

Vector search determines similarity by measuring the distance between a query vector and the vectors stored in your data. Vectors that are closer together are considered more similar.

A **k-nearest neighbors (kNN)** search returns the `k` vectors closest to the query vector. For example, if `k` is `5`, the search returns the five most similar results. Vector search can use one of the following approaches:

- **Exact nearest neighbor (ENN)**

    Compares the query vector with every indexed vector to identify the exact nearest neighbors. ENN provides the most accurate results but can require more processing time for large datasets.

- **Approximate nearest neighbor (ANN)**

    Uses a specialized vector index to identify likely nearest neighbors without comparing every vector. ANN provides faster and more scalable searches, although it might not always return the exact closest vectors.

Percona Search for MongoDB uses a specialized vector index to improve approximate nearest neighbor search performance. The index organizes vector embeddings so the search can locate likely nearest neighbors without comparing the query vector with every indexed vector.

- **Hierarchical Navigable Small World (HNSW)**

Organizes vectors as a multilayer graph and connects nearby vectors to one another. During a search, the algorithm navigates these connections to quickly find vectors that are likely to be closest to the query vector.

For more information, see [MongoDB Vector Search queries :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/#mongodb-vector-search-queries){:target="_blank"}.


## Vector search in Percona Search for MongoDB

Percona Search for MongoDB supports vector search, which retrieves results based on the underlying meanings of the data rather than relying on exact keyword matches. Unlike traditional full-text search that looks for matching terms, [vector search :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/#what-is-vector-search){:target="_blank"} identifies vectors positioned near your query in a multi-dimensional space. The closer two vectors are in this space, the more closely their meanings align.

??? example "Semantic search example"

    A keyword search for `payment failure` typically prioritizes documents containing those terms. Vector search can also retrieve records describing a `declined transaction`, `failed card authorization`, or `incomplete payment`, even when the exact phrase `payment failure` is absent.


