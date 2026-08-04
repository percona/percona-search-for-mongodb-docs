## Initial feature comparison

|**Capability**|**Percona Search for MongoDB**|**MongoDB Community|MongoDB Enterprise Advanced**|**MongoDB Atlas**|
|---|---|---|---|---|
|Operating model|Self-managed|Self-managed (tarball, Docker, or package manager); Kubernetes Operator support available as a Preview feature|`mongot` managed through the MongoDB Controllers for Kubernetes Operator|Fully managed|
|Full-text search|Yes|Yes|Yes|Yes|
|Vector search|Yes|Yes|Yes|Yes|
|`$search`, `$searchMeta`, `$vectorSearch`|Yes|Yes|Yes|Yes|
|Manual embeddings|Yes|Yes|Yes|Yes|
|Automated embeddings with Voyage AI|Yes|Yes (Preview, via Kubernetes Operator deployments)|Not yet available|Yes|
|Replica sets|Yes|Yes|Yes|Yes|
|Sharded clusters|Yes|Yes|Yes|Yes|
|Direct client connection to `mongot`|No|No|No|No|
|Search index management through MongoDB commands|Yes|Yes|Yes|Yes|
|Native LLM answer generation|No, integrate an external LLM|No, integrate an external LLM|No, integrate an external LLM|No, integrate an external LLM|
|Product status|Technical preview|`mongot` for Community is a public preview capability|Kubernetes Operator search deployment is a Preview feature|Generally available managed service|