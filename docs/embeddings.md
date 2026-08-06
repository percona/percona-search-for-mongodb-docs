# Embeddings for Percona Search for MongoDB

Percona Search for MongoDB supports vector search based on embeddings that you generate and manage yourself or on automatically generated ones. You can generate embeddings for your data using an embedding model of your choice, store the resulting vectors in your MongoDB documents, and query them with the `$vectorSearch` aggregation stage to find documents with similar meaning.

Once an embedding is stored, it can be reused for future searches. Each search query requires its own embedding, generated from the query text using the same embedding model that was used to generate the stored document embeddings.

![image](../_images/percona-search-embeddings.png)

## Manual embeddings

With manual embeddings, the application controls embedding generation:
{.power-number}

1. An ingestion pipeline sends document content to an external or locally hosted embedding model.

2. The pipeline stores the generated vector in a field in the source document.

3. `mongot` indexes that vector field.

4. At query time, the application sends the user's query to the same embedding model.

5. The application passes the generated query vector to $`vectorSearch` through the `queryVector` option.

For [RAG applications :octicons-link-external-16:](https://www.mongodb.com/docs/voyageai/tutorials/rag/?language-no-interface=python&llm-provider=openai&vector-storage=in-memory){:target="_blank"}, the application sends the user's question and the documents returned by vector search to a generative LLM. The LLM uses the retrieved documents as context to generate the response.

## Automatic embeddings

With [automatic embeddings :octicons-link-external-16:](https://www.mongodb.com/docs/vector-search/crud-embeddings/automated-embedding/){:target="_blank"}, `mongot` sends the indexed text and query text to the configured Voyage AI embedding service. It generates document embeddings during indexing and a query embedding when the application runs a text-based vector search.

## Next steps

[`$vectorSearch` :material-arrow-right:](query-with-vectorsearch.md){.md-button}

[Create a vector search index :material-arrow-right:](create-vector-search-index.md){.md-button}




