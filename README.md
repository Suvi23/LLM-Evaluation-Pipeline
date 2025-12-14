# LLM-Evaluation-Pipeline

This project implements a comprehensive evaluation pipeline for AI-generated responses in chat conversations. It checks **relevance**, **completeness**, and **hallucination**, while also tracking **latency** and **estimated cost** for each evaluation.



## **Table of Contents**

1. [Local Setup Instructions](#local-setup-instructions)
2. [Evaluation Pipeline Architecture](#evaluation-pipeline-architecture)
3. [Design Decisions](#design-decisions)
4. [Scalability Considerations](#scalability-considerations)
5. [Sample Output](#sample-output)



## **Google Colab Setup Instructions**

1. **Open the Notebook**

   * Open `llm evaluation pipeline.ipynb` in Google Colab.
   * Make sure you have a Google account and are signed in.

2. **Upload Your Files**

   * Click the **folder icon** on the left sidebar to open the file explorer.
   * Use the **Upload button** to upload:

     * 2 **chat JSON files**: e.g., `sample-chat-conversation-01.json`, `sample-chat-conversation-02.json`
     * 2 **context JSON files**: e.g., `sample_context_vectors-01.json`, `sample_context_vectors-02.json`

3. **Install Dependencies**

   * Colab comes with most common packages pre-installed, but you may need:

   ```python
   !pip install -q scikit-learn sentence-transformers numpy
   ```

4. **Run the Notebook**

   * Execute the cells sequentially from top to bottom.
   * The notebook will:

     * Validate your uploaded JSON files.
     * Extract context passages.
     * Compute embeddings and cosine similarities.
     * Evaluate relevance, completeness, and hallucination.
     * Track latency and estimated cost.
     * Output results as JSON.

5. **Download Results**

   * After evaluation, results can be saved to a file using:

   ```python
   import json
   with open("evaluation_results.json", "w") as f:
       json.dump(final_results, f, indent=2)
   ```



## **Evaluation Pipeline Architecture**

The pipeline consists of the following steps:

### **1. File Validation**

* Check if uploaded files exist and are valid JSON.
* Print detailed error messages with line/column information if a file is invalid.
* Separate files into `chat_files` and `context_files`.

### **2. Context Extraction**

* Function `clean_contexts` extracts text passages from JSON:

  * Primary: `final_response` from retrieved sources
  * Fallback: `vector_data` if `final_response` is empty

### **3. Embedding and Similarity Computation**

* **Relevance & Completeness**

  * Embed query, AI response, and context passages.
  * Relevance = cosine similarity between query and response embeddings.
  * Completeness = mean cosine similarity between response and context embeddings.
* **Hallucination Detection**

  * Compute maximum similarity between AI response and context embeddings.
  * Flag as hallucination if similarity < threshold (default 0.65).

### **4. Performance Metrics**

* **Latency Tracking**

  ```python
  latency_seconds = track_latency(start_time)
  ```
* **Cost Estimation**

  ```python
  estimated_cost_usd = estimate_cost(user_query + ai_response)
  ```

### **5. Output Aggregation**

* Final results are stored as a structured JSON for further analysis.

---

## **Design Decisions**

1. **Why this architecture?**

   * Separation of concerns: validation → context extraction → evaluation → metrics → output.
   * Fallback mechanisms in context extraction ensure the pipeline does not fail if some fields are missing.
   * Cosine similarity with embeddings is efficient and widely used for semantic comparisons.
   * Explicit latency and cost tracking makes it suitable for operational monitoring.

2. **Why not other approaches?**

   * Avoided full LLM-based evaluation for relevance/completeness to reduce API calls, latency, and costs.
   * Avoided naive keyword matching because it does not capture semantic meaning.
   * Modular structure allows easy replacement of embedding models or thresholds.


## **Scalability Considerations**

If processing **millions of conversations per day**:

1. **Batch Embedding**

   * Use batch embedding for multiple queries/responses to reduce API calls or local inference time.

2. **Vector Store Optimization**

   * Pre-compute embeddings for context documents once.
   * Use efficient similarity search libraries (FAISS, Milvus) instead of computing all pairwise cosine similarities repeatedly.

3. **Latency & Cost Reduction**

   * Only compute relevance and hallucination when needed.
   * Use caching for repeated queries or similar conversations.
   * Lightweight embedding models balance speed and accuracy.

4. **Asynchronous Processing**

   * Pipeline can be parallelized for independent conversations.
   * Use job queues or async tasks for real-time evaluation with minimal delay.


## **Sample Output**

```json
[
  {
    "chat_file": "sample-chat-conversation-01.json",
    "context_file": "sample_context_vectors-01.json",
    "evaluation_result": {
      "relevance_and_completeness": {
        "relevance_score": 0.718,
        "completeness_score": 0.264
      },
      "hallucination_check": {
        "max_context_similarity": 0.575,
        "hallucination_detected": true
      },
      "latency_seconds": 0.567,
      "estimated_cost_usd": 0.0003
    }
  }
]
```


