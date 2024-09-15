# Module 4: Evaluation and Monitoring

In this module, we will learn how to evaluate and monitor our LLM and RAG system.

In the evaluation part, we assess the quality of our entire RAG system before it goes live.

In the monitoring part, we collect, store and visualize metrics to assess the answer quality of a deployed LLM. We also
collect chat history and user feedback.

> Note: Retrieval will have its own evaluation(discussed in previous module), and generation from the rag system will have a separate evaluation(discussed here).

## Introduction

**Monitoring answer quality of LLMs**:
- Compute Different types of quality metrics:
    - LLM-as-a-judge to assess quality of LLM Answer
    - Similarity score between expected and LLM answer
    - [optional] LLM-as-a-judge to compute toxicity of LLM Answer
- Store computed metrics in relational database
    - Store chat sessions and collect user feedback in database
- Use Grafana to visualize metrics over time
    - Use Grafana to visualize user feedback and corresponding chat sessions
- What else to monitor
    - System metrics: latency, traffic, errors, saturation.
    - Cost of used infrastructure, i.e. vector store and LLM API

## Offline RAG Evaluation

In previous module, we have generated ground-truth (GT) dataset that contains `question, course, document_id` columns.

Now, we get an answer for a query using our RAG pipeline, and evaluate it. For ground truth answer, we take `text from documents[document_id]`.

### Cosine Similarity

We an use answers to generate questions and then use these generated questions to generate RAG answers. These two sets of answers are then the ground truth and generated ones that can be used to calculate similarity scores.

```python
v_llm = embedding_model.encode(answer_llm) # normalized
v_orig = embedding_model.encode(answer_orig)

v_llm.dot(v_orig) # between 0 and 1 similarity score
```

Using the code above, we can calculate the average `cosine_similarity` over the test dataset, plot the histogram of similarity scores, etc.

**Limitations (gpt4 answer):**
- `Lack of Context Understanding and Semantic Similarity:` Two responses might be semantically similar (in terms of topics) but differ significantly in the accuracy or relevance of the information provided.
- `Binary Nature of Scoring:` Cosine similarity scores range between -1 and 1, but in practice, for normalized vectors, they range between 0 and 1. This binary scoring can be simplistic and may not provide a detailed gradient of correctness or relevance, which is often needed in nuanced LLM evaluations.

**Alternatives (gpt4 answer):**
- ROUGE/BLEU Scores

### LLM as a judge

As name says, we ask an LLM to evaluate the answer given the ground truth answer.

```python
# very high-level example 
prompt = '''
You're expert evaluator...

Here is data for evaluation:
Original answer: {orig_answer}
Generated question: {gen_question}
Generated answer: {gen_answer}

Please analyze the content and context, and answer in format ... 

{
    "relevance": "NON_RELEVANT" | "PARTLY_RELEVANT" | "RELEVANT",
    "explanation": "[provide a brief explanation]"
}
'''
# constuct prompt for a record
...

# pass to llm
evaluation = llm(prompt)
```

We run the code for whole test dataset, and collect all the evaluation done by LLM.
We can count the values for `relevance` column, etc.


## Offline RAG Evaluation: Generating data

Links:

* Code Notebook used to generate data files required for evaluation : [notebook](offline-rag-evaluation.ipynb)
* [results-gpt4o.csv](data/results-gpt4o.csv) (answers from GPT-4o)
* [results-gpt35.csv](data/results-gpt35.csv) (answers from GPT-3.5-Turbo)


### Cosine Similarity

Links:

* [results-gpt4o-cosine.csv](data/results-gpt4o-cosine.csv) (answers with cosine calculated from GPT-4o)
* [results-gpt35-cosine.csv](data/results-gpt35-cosine.csv) (answers with cosine calculated from GPT-3.5-Turbo)
* [results-gpt4o-mini.csv](data/results-gpt4o-mini.csv) (answers from GPT-4o-mini)
* [results-gpt4o-mini-cosine.csv](data/results-gpt4o-mini-cosine.csv) (answers with cosine calculated from GPT-4o-mini)


### LLM as a Judge

Links:

* [evaluations-aqa.csv](data/evaluations-aqa.csv) (A->Q->A' evaluation results)
* [evaluations-qa.csv](data/evaluations-qa.csv) (Q->A evaluation results)


## Online RAG Evaluation

> The intermediate prompts and the outputs from claude used to develop the application can be seen [here](code.md)

Overall, following steps are required to set up the online evaluation:
* Adding +1 and -1 buttons
* Setting up a postgres database
```bash
pip install pgcli
pgcli -h localhost -U your_username -d course_assistant -W
```
* Putting everything in docker compose
* adding vector search and OpenAI
* Setting up Grafana
* Tokens and costs
* QA relevance
* User feedback
* Other metrics


### Capturing User Feedback

> Check the final code of built [application](app/)



The app has Streamlit UI, and Postgres DB. The pipeline is simple:

1. Create DB
2. Set up the UI, LLM Assistant, and LLM Judge
3. UI should have `+1` (positive) and `-1` (negative) feedback buttons.
4. User asks a question through the UI, and the following code executes under the hood:
```python
search_results = search(query)
prompt = build_prompt(query, search_results)
answer, tokens, response_time = llm(prompt, model_choice) # llm assistant

relevance, explanation, eval_tokens = evaluate_relevance(query, answer) # llm judge
```
5. The answer is displayed on the UI, and the user can press the feedback button.
6. Insert the conversation and feedback information into the DB.
```python
cur.execute(
    """
    INSERT INTO conversations 
    (id, question, answer, course, model_used, response_time, relevance, 
    relevance_explanation, prompt_tokens, completion_tokens, total_tokens, 
    eval_prompt_tokens, eval_completion_tokens, eval_total_tokens, openai_cost, timestamp)
    VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, COALESCE(%s, CURRENT_TIMESTAMP))
""",
... # values
)

cur.execute(
    "INSERT INTO feedback (conversation_id, feedback, timestamp) VALUES (%s, %s, COALESCE(%s, CURRENT_TIMESTAMP))",
    (conversation_id, feedback, timestamp),
)
```

### Monitoring

**Grafana:**
- Setting up:
    - In `docker-compose.yaml`, we pull grafana image
- Special Variables:
    - `$__timeFrom()` and `$__timeTo()`: Start and end of the selected time range
    - `$__timeGroup(timestamp, $__interval)`: Groups results by time intervals automatically calculated by Grafana
- Then, on the Grafana UI:
    1. Add data source $\rightarrow$ Choose PostgreSQL
    2. Create new dashboard $\rightarrow$ Add new panel $\rightarrow$ Write SQL Queries
    ```sql
    SELECT
    $__timeGroup(timestamp, $__interval) AS time,
    AVG(total_tokens) AS avg_tokens
    FROM conversations
    WHERE timestamp BETWEEN $__timeFrom() AND $__timeTo()
    GROUP BY 1
    ORDER BY 1
    ```

* SQL queries for Grafana: [here](grafana.md)
* Grafana dashboard: [here](dashboard.json)



### Overview of the module

![image](https://github.com/user-attachments/assets/0c5e932b-4fca-4d51-8b1c-93f4600861dc)
