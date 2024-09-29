## Best Practices

Below are some of the techniques that could improve the RAG pipeline:
* Small-to-Big chunk retrieval
* Leveraging document metadata
* Hybrid search
* User query rewriting
* Document reranking

Useful Links:
* Slides: [pdf](llm-zoomcamp-best-practicies.pdf)
* Five Techniques for Improving RAG Chatbots - Nikita Kozodoi: [Video](https://www.youtube.com/watch?v=xPYmClWk5O8)
* Survey on RAG techniques: [Article](https://arxiv.org/abs/2312.10997)

> Note: These codes and notebooks are yet to be executed and rechecked.

### Hybrid Search

Useful Links:
* Hybrid Search and Reranking in ElasticSearch: [Notebook](hybrid-search-and-reranking-es.ipynb)
* Hybrid search: [Elasticsearch Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/knn-search.html#_combine_approximate_knn_with_other_features)
* Hybrid search: [Tutorial](https://www.elastic.co/search-labs/tutorials/search-tutorial/vector-search/hybrid-search)


### Document Reranking

Pull and run a docker container with Elasticsearch 8.9.0 or higher in order to use reranking based on RRF algorithm:

```bash
docker run -it \
    --rm \
    --name elasticsearch \
    -m 4GB \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    docker.elastic.co/elasticsearch/elasticsearch:8.9.0
```

Useful Links:
* Reciprocal Rank Fusion (RRF) method: [Elasticsearch Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/rrf.html)
* RRF method: [Article](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf)
* Elasticsearch subscription plans: [Link](https://www.elastic.co/subscriptions)


### Hybrid search with LangChain

```bash
pip install -qU langchain langchain-elasticsearch langchain-huggingface
```

Links:
* Langchain Hybrid Search Implementation: [Notebook](hybrid-search-langchain.ipynb)
* Chatbot Implementation: [Tutorial](https://www.elastic.co/search-labs/tutorials/chatbot-tutorial/implementation)
* ElasticsearchRetriever: [link](https://python.langchain.com/v0.2/docs/integrations/retrievers/elasticsearch_retriever/)