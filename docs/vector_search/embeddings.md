# Embeddings for Percona Search for MongoDB

Percona Search for MongoDB can automatically generate and manage vector embeddings for text stored in your collections. When you enable automated embedding, it uses the configured Voyage AI embedding model to generate vectors for selected text fields during indexing. At query time, it converts the query text into a vector with the same model to find semantically similar documents.

This workflow simplifies semantic search because your application does not need to generate, store, or update embeddings.

## Manual embeddings

With manual embeddings, the application controls embedding generation:
{.power-number}

1. An ingestion pipeline sends document content to an external or locally hosted embedding model.

2. The pipeline stores the generated vector in a field in the source document.

3. `mongot` indexes that vector field.

4. At query time, the application sends the user's query to the same embedding model.

5. The application passes the generated query vector to $`vectorSearch` through the `queryVector` option.

For [RAG applications](https://www.mongodb.com/docs/voyageai/tutorials/rag/?language-no-interface=python&llm-provider=openai&vector-storage=in-memory), the application sends the user's question and the documents returned by vector search to a generative LLM. The LLM uses the retrieved documents as context to generate the response.

## Automatic embeddings

With automatic embeddings, `mongot` sends the indexed text and query text to the configured Voyage AI embedding service. It generates document embeddings during indexing and a query embedding when the application runs a text-based vector search.


![image](../_images/percona-search-embeddings.png)