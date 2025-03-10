<!-- What is vector search, and how can vector databases help? -->

# Vector Search

![](../../assets/building_blocks/vector_search/vector-search-cover-sketch.png)

Search systems that can retrieve information from structured and unstructured text data have been around since the [1970s](https://en.wikipedia.org/wiki/IBM_STAIRS). In the early days of the web, keyword-based search engine libraries like Lucene used inverted indexes to improve search relevance and speed when querying large collections of data. Inverted indexes are data structures that map keyword frequencies relative to the documents in which they appear. Because these methods search on the entire collection of texts at once, they're referred to as *full-text* search.

More recently, since the advent of deep learning models, search systems have evolved to capture the semantics of the underlying data. This is the basis of *vector* search, which is typically done by converting the data into a numerical representations that can be queried at scale.

## What are vectors?

In computer science, a vector is a one-dimensional array of numbers (typically floating point numbers).

```py
example_4d_vector = [0.1234, 0.3035, 0.9876]
```

The length of a vector is called its dimensionality, and in the context of machine learning, high-dimensional vectors play a critical role in representing data in a way that computers can better interpret. The example above shows a 3-D vector, but the vectors used in machine learning typically have hundreds (or thousands) of dimensions.

Vectors are generated by passing data through the layers of a trained neural network. For example, a transformer-based model like [SentenceBERT](https://www.sbert.net/docs/pretrained_models.html) can be used to convert a sentence/document into a a numerical representation that represents the semantic meaning of the sentence in vector space.

A toy example in 3-D space is shown below. Vectors for semantically similar sentences are closer together than those that represent dissimilar sentences or concepts.

![](../../assets/building_blocks/vector_search/vector-search-similarities.png)

The power of vectors lies in their multimodal nature -- they can just as well represent images or audio clips as they can text, depending on the model used to generate them. This makes them a powerful tool for building search systems that can handle multi-modal data.

:::hint{style="info"}
The terms *vector* and *embedding* are often used interchangeably, or sometimes even together, as *vector embedding*. In the context of vector search, they all mean the same thing.
:::

### Distance metrics

The similarity between two vectors is quantified via a distance metric. A few commonly used distance metrics are listed below.

* Euclidean (L2) distance
* Dot product similarity
* Cosine similarity

Cosine similarity, which considers the angle between two vectors, is commonly used in measuring the similarity between texts. Two vectors that are very dissimilar can be thought of as orthogonal to each other in vector space, with a cosine similarity of zero. On the other hand, vectors that are very similar to one another will have a cosine similarity close to 1, with a limiting value of 1 being the case where a given vector is always identical to itself.

## Vector search for unstructured data

Vector (semantic) search distinguishes itself from keyword (lexical) search by being able to return relevant results *even when the exact terms aren't present in the query*. Because of the general-purpose nature of vector embeddings, they can be used to represent almost any form of data, from unstructured data like text, images and audio. Additionally, multimodal embedding models like [CLIP](https://github.com/openai/CLIP) can generate a single embedding that simultaneously captures the semantics of multiple forms of data, such as images and their descriptions.


![](../../assets/building_blocks/vector_search/vector-search-embeddings.png)


### Context length for text embedding models

When vectorizing unstructured text data, it's important to bear in mind that embedding models have a maximum context length, which is the maximum token sequence length that can be embedded into a single vector. For many open-source embedding models, this is in the range of 384 to 1024 tokens (in [English](https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them), 100 tokens equal roughly 75 words).

Using an embedding model on long-form texts of the order of thousands of words would mean that the entire text cannot be embedded into a single vector, and would need to be split into smaller chunks, or require a windowing approach to capture the entire document in a single vector. Libraries like [LangChain](https://github.com/langchain-ai/langchain) and [LlamaIndex](https://github.com/jerryjliu/llama_index) can help with this.

## Vector search for structured data

A common misconception of vector search is that it's applicable only to unstructured data. This is not true! Vector embeddings have *always* been at the heart of deep learning models, and this section will highlight how structured data can be leveraged to gain new insights into existing data.

### Tabular data

Tabular data is typically stored in a relational database or data lake, and can be a powerful source of data for supervised learning systems. Neural networks that use gradient-based optimization have emerged as powerful tools to map numerical features, commonly seen in tabular data, to higher-dimensional vector representations.

Historically, gradient-boosted decision trees have outperformed baseline neural networks on tabular data, but [recent research](https://arxiv.org/abs/2203.05556) shows that a traditional MLP-like (Multilayer Perceptron) models, when equipped with embeddings from numerical features, can perform on par with computationally-heavy transformers on tabular data. Transforming tabular data into vector embeddings can leverage the features of the underlying data in a more meaningful way, allowing you to build better recommendation algorithms and fraud-detection systems.

### Time series

Time series analysis is a useful tool for tasks like anomaly detection, where, historically, distance-based techniques like kNN (k-Nearest Neighbors) are popular because they are unsupervised. However, kNN suffers from the "curse of dimensionality" as the datasets become large because it needs to exhaustively compare distances vs. all other points in the dataset to detect outliers. To deal with this limitation, autoencoders have historically been used to reduce the dimensionality of time series data.

Although there have been claims in [recent work](https://arxiv.org/abs/2205.13504) that transformers are not as effective as specialized linear models, at least for univariate time series, [further work](https://huggingface.co/blog/autoformer) by Hugging Face has shown that the embeddings learned in transformers are richer than linear models over covariate features. This means that the embeddings extracted from a time series transformer have the potential to describe and detect anomalies in time series data more effectively than traditional methods.

### Knowledge graphs

Graphs are connected data structures that can be used to represent the relationships between entities, and can be used to build a host of powerful applications that require access to factual information, such as recommender systems and question-answering engines. Just like NLP models can be used to generate vector embeddings for text, graph neural networks can be used to generate embeddings for the entities (nodes/edges) in a knowledge graph. [node2vec](https://arxiv.org/abs/1607.00653) and [GraphSAGE](https://arxiv.org/abs/1706.02216) are two popular methods to extract node embeddings from feature information and the overall graph structure.

Nodes that are similar to one another in the graph are closer to one another in the node embedding space. This means that these embeddings can be used to perform similarity search on the graph, and can be used to answer questions like "what are the most similar nodes to this one?" or "what could be additional nodes linked to this one?". When combined with faster ways to store and query these vectors (such as those offered by vector databases), graph embeddings can be used to build powerful applications that consider both the semantics *and* structure the of underlying data.

## What is a vector database?

A vector database is a purpose-built system designed to manage and query vector embeddings at scale. It is able to perform vector (semantic) search and retrieval by comparing the query vector with those stored in the database, returning the top-k most similar ones.

The figure below shows the key underlying components of a vector DB -- not all DB vendors may represent their internals this way, but the same principles generally apply.

![](../../assets/building_blocks/vector_search/vector-search-database.png)

### Vector databases vs. vector search libraries

What *makes* a vector DB? Why can't we just use a regular database and store vectors natively as arrays? The answer lies in the underlying data structures, indexing algorithms and other key features that enable vector DBs to perform similarity search at scale.

- A vector index allows fast and efficient retrieval of similar vectors
- A query engine that performs optimized similarity computations on the index
- Partitioning/sharding to enable horizontal scaling
- Replication and other reliability features
- An accessible API that allows for efficient vector CRUD operations

It's for these reasons that well-known open-source projects like [FAISS](https://github.com/facebookresearch/faiss), [ScaNN](https://github.com/google-research/google-research/tree/master/scann) and [Hnswlib](https://github.com/nmslib/hnswlib) are **not** categorized as vector DBs -- they are vector search *libraries* that offer only a portion of the capabilities required to build production-grade solutions.
 
## Breakdown of the vector DB landscape

Broadly speaking, vector DBs can be broken down to three categories:

- **Vector-native**: Purpose-built DBs that optimize for vector search while making vector embeddings first-class citizens
- **Hybrid**: Existing DBs that have added vector search capabilities
- **Search engines**: Systems that natively support search and relevance ranking on unstructured data via an underlying index

Although there is a strong overlap in capabilities across these categories, search engines are kept in their own category because they're typically not used as the primary database or for storing raw data. They're often used to index data that's already present in other databases or data lakes.

::::tabs
:::tab{title="Vector-native DBs"}
| Name | Built in | Open-source | Managed cloud | Self-hosting |
| ---------- | :-: | :-: | :-: | :-: |
| Weaviate | Go | ✅ | ✅ | ✅ |
| Qdrant | Rust | ✅ | ✅ | ✅ |
| Milvus | Go | ✅ | ✅ | ✅ |
| Zilliz | Go | ❌ | ✅ | ❌ |
| Chroma | C++ | ✅ | Upcoming | ✅ |
| LanceDB | Rust | ✅ | Upcoming | ✅ |
| Deeplake | C++ | ✅ | ❌ | ✅ |
| Pinecone | Rust | ❌ | ✅ | ❌ |
| MyScale | C++ | ❌ | ✅ | ❌ |
| AstraDB | Java | ❌ | ✅ | ❌ |
| Timescale | Rust | ❌ | ✅ | ❌ |
:::

:::tab{title="Hybrid DBs"}
| Name | Built in | Open-source | Managed cloud | Self-hosting |
| ---------- | :-: | :-: | :-: | :-: |
| Redis | C | ✅ | ✅ | ✅ |
| Postgres (pgvector) | C++ | ✅ | ❌ | ✅ |
| MongoDB | C++ | ✅ | ✅ | ❌ |
| ClickHouse | C++ | ✅ | ✅ | ✅ |
| Neo4j | Java | ✅ | ✅ | ✅ |
:::

:::tab{title="Search engines"}
| Name | Built in | Open-source | Managed cloud | Self-hosting |
| ---------- | :-: | :-: | :-: | :-: |
| Vespa | C++ | ✅ | ✅ | ✅ |
| Vald | Go | ✅ | ✅ | ✅ |
| ElasticSearch | Java | ❌ | ✅ | ✅ |
| OpenSearch | Java | ❌ | ✅ | ❌ |
| Meilisearch | Rust | ✅ | ✅ | ✅ |
| Typesense | C++ | ✅ | ✅ | ✅ |
:::
::::

There is some debate on the distinction between a search engine and a vector DB, but in general, a lot of these tools offer variations of the same functionality -- the ability to manage and query vectors at scale.

## Important considerations

In choosing a vector search solution for the optimal combination of cost, latency and quality, it's important to keep in mind the following factors.

### Performance

The performance of a vector search solution can be defined in many ways, but the most important metrics are:

* QPS (queries per second)
* Latency
* Recall (the fraction of relevant results returned)
* Indexing time

The [`ann-benchmarks`](https://github.com/erikbern/ann-benchmarks) repo was among the first to offer a standardized test bench for benchmarking vector search solutions. However, a lot of the tasks in this benchmark are simplistic and not representative of real-world datasets, and as such, it's recommended to test your own data on at least a few of the solutions you're considering.

In general, most modern vector DBs are expected to handle throughputs of hundreds of QPS, with latencies of under 100 ms per query, while also scaling to be able to serve many concurrent requests. Depending on the nature of the underlying index, the QPS generally increases as compression/quantization levels are increased, with a corresponding loss of recall as well.

### Scalability for a given cost

The scalability of a vector database depends not only on the throughput it can handle for a given amount of compute -- it also depends on how cost-effective the solution is for the given level of performance.

Obtaining exact cost estimates for the many solutions out there is challenging because the field evolves so rapidly, but in this section, we attempt to highlight some factors that can define what "scalability" means for a given cost budget.

> TODO: Need to add some details on monthly cost for the number of queries processed here.


### Hybrid search and re-ranking

To improve search relevance when an exact match between keywords in the query and the results are required, several vector DBs and search engines offer hybrid search options that combine top-k scores from keyword and vector search. Research from [Google](https://arxiv.org/abs/2201.10582) shows that using Reciprocal Rank Fusion (RRF) to re-rank search results using a combination of keyword and vector search can improve search relevance by up to 20%.

More sophisticated methods to improve relevance exist, such as re-ranking via [cross-encoders](https://www.sbert.net/examples/applications/cross-encoder/README.html). This approach uses a transformer-based model downstream of the vector DB that generates new scores for the top-k results returned by the DB. This approach is more computationally expensive, but can yield better results than RRF, at the expense of latency and added system complexity.

### Fine-tuning embedding models to address bias

Existing open source embedding models, whether in the image or text domain, can be biased towards certain concepts, or simply not model the distribution of unseen test data once their training is complete. For example, a model trained on a dataset of images of people may be biased towards certain skin tones, or a model trained on a dataset of text may be biased towards certain topics, which depends on the distribution of the training data. Additionally, embeddings from a model trained on a dataset of text from the web may not generalize well to text from a different domain, such as medical or legal text.

In cases like where the semantics of the underlying domain are really important, it may be necessary to fine-tune the embedding model prior to using them to encode data for vector search. Many open source frameworks like [Hugging Face](https://huggingface.co/blog/how-to-train-sentence-transformers#how-to-train-or-fine-tune-a-sentence-transformer-model), enable this.

## Conclusions

Building a production-grade search and retrieval solution requires specialized components, including embedding models, vector databases and re-ranking modules. There are a lot of conflicting viewpoints out there on how to effectively combine these tools for real use cases. Hopefully, reading this post has made you excited to learn more about vector search for innovative use cases on all kinds of data in your organization.

---
## Contributors

- [Prashanth Rao](https://www.linkedin.com/in/prrao87/)