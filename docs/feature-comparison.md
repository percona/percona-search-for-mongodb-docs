# Feature comparison

Percona Search for MongoDB is currently available as a **technical preview**. 

The following table compares the current capabilities of Percona Search for MongoDB with MongoDB Community Edition, MongoDB Enterprise Advanced, and MongoDB Atlas.

| Capability | Percona Search for MongoDB | MongoDB Search (Community) | MongoDB Search (Enterprise) | Atlas Search |
|------------|----------------------------|----------------------------|-----------------------------|--------------|
| Deployment model | Self-managed | Self-managed | Self-managed | Fully managed |
| Kubernetes Operator support | Technical Preview | Technical Preview | Technical Preview | N/A |
| Number of Search nodes | 1 | any | any | any |
| Full-text search | Yes | Yes | Yes | Yes |
| Vector search | Yes | Yes | Yes | Yes |
| Hybrid Search | Yes | Yes | Yes | Yes |
| Manual embeddings | Yes | Yes | Yes | Yes |
| Automatic embeddings | open model choice| locked to Voyage AI | locked to Voyage AI | locked to Voyage AI |
| Self-hosted / Open Embedding Models | Ollama, vLLM, llama.cpp, TEI, etc. | No | No | No |
| Backup and Restore integration | No | No | No | Yes (index definitions only) |
| Reranking | Planed (open cross-encoder) | No | No | Yes |
| Native LLM answer generation | Integrate an external LLM | Integrate an external LLM | Integrate an external LLM | Integrate an external LLM |
| Product status | Technical Preview | Generally available | Generally available | Generally available |

!!! note
    - Percona Search for MongoDB currently supports **manual embeddings**. Automatic embedding generation is planned for a future release.

## Next steps

[Full-text search overview :material-arrow-right:](full-text-search-overview.md){.md-button}

[Vector search overview :material-arrow-right:](vector-search-overview.md){.md-button}
