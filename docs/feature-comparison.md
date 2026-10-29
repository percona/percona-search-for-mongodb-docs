# Feature comparison

The following table compares the current capabilities of Percona Search for MongoDB with MongoDB Community Edition, MongoDB Enterprise Advanced, and MongoDB Atlas.

| Capability | Percona Search for MongoDB | MongoDB Search (Community) | MongoDB Search (Enterprise) | Atlas Search |
|------------|----------------------------|----------------------------|-----------------------------|--------------|
| Deployment model | Self-managed | Self-managed | Self-managed | Fully managed |
| Kubernetes Operator support | Technical Preview | Technical Preview | Technical Preview | N/A |
| Manual embeddings | Yes | Yes | Yes | Yes |
| Automatic embeddings | Yes, open model choice | Voyage AI only | Voyage AI only | Voyage AI only |
| Hosted embedding providers | Voyage AI, OpenAI, Azure OpenAI, Hugging Face Inference API, Hugging Face Inference Endpoints | Voyage AI | Voyage AI | Voyage AI |
| Self-hosted / open embedding models | Ollama, vLLM, llama.cpp, LM Studio, LocalAI, Hugging Face Text Embeddings Inference (TEI), and other OpenAI-compatible servers | No | No | No |
| Backup and Restore integration | No | No | No | Yes (index definitions only) |
| Reranking models | Planned (open cross-encoder) | No | No | Yes |
| Native LLM answer generation | Integrate an external LLM | Integrate an external LLM | Integrate an external LLM | Integrate an external LLM |
