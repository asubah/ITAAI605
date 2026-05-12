# Tutorial 7: Evaluating and Tuning RAG Pipelines

**Objective:** Learn how to quantitatively evaluate an existing RAG system using n8n's native evaluation tools. You will build an automated testing loop that pulls questions from a datatable, processes them through your existing RAG workflow, and scores the accuracy of the answers using local AI models.

**Prerequisites:**

* A functional n8n RAG sub-workflow (like the one built in Tutorial 5).
* **The Class GitHub Repository:** Clone or pull the latest changes from the class repository. You will need the files located in the [data/Tutorial_07](data/Tutorial_07/) directory.
* Access to local LLMs via Ollama hosted in The Benefit Advanced AI and Computing Lab.

---

## Part 0: Data Prep & Ingestion

Before evaluating the system, you must ensure your AI has the right documents and that your testing data is loaded into n8n.

1. **Ingest the PDFs:** Use your "Load Data" pipeline (from Tutorial 5) to upload the 4 UOB Policy PDFs located in the [data/Tutorial_07](data/Tutorial_07/) folder of the repository into your Vector Store.
2. **Upload the Evaluation Dataset:** * Open your n8n workspace and navigate to **Data Tables**.
* Create a new table (e.g., `uob_eval_data`).
* Upload the `uob_policy_eval.csv` file, located at `data/Tutorial_07/uob_policy_eval.csv`, to populate this table with the 20 test questions and their ground truths.



---

## Part 1: The Automated Evaluation Loop

In this phase, we bypass the standard chat interface and use n8n's built-in testing framework to push all 20 questions through your RAG system automatically.

### Step 1: The Evaluation Trigger

1. Create a new workflow and add the **On Evaluation Testing** trigger node.
2. Configure this trigger to pull data from your newly created `uob_eval_data` Data Table.
3. Map the columns: ensure the `question` column acts as your input, and the `ground_truth` column acts as the expected output.
4. *What it does:* This node automatically iterates through your dataset, launching the workflow once for every question.

### Step 2: Connecting the RAG Chain

1. Add an **Execute Workflow** node and connect it to the trigger.
2. Select your existing RAG workflow from the dropdown.
3. Pass the `question` from the trigger into the sub-workflow.
4. *What it does:* This sends the test question directly into your RAG pipeline, bypassing the need for a human to type it in a chat box.

### Step 3: The Evaluator Node

1. Add an **Evaluation** node and connect it to the output of the Execute Workflow node.
2. Configure the metrics to compare the **Actual Answer** (the output from your RAG chain) against the **Expected Answer** (the `ground_truth` from the datatable).
3. Set the grading metric to assess "Factual Accuracy" or "Answer Relevancy."
4. *What it does:* This node uses an LLM to read the AI's generated response, compare it to the absolute truth, and assign a pass/fail score.

---

## Part 2: Establishing the Baseline

Before making any changes, you must know how your system currently performs.

### Step 1: The Baseline Run

1. Ensure your RAG workflow is using standard, default settings:
* **Chunking:** Fixed-size (e.g., 500 tokens).
* **Prompt:** *"You are a helpful assistant. Answer based on the context."*
* **Model:** A standard local model via the Ollama node (e.g., `llama3:8b` or `qwen3:8b`).


2. Go to your Evaluation workflow and click **Execute Evaluation**.
3. *What it does:* n8n will run all 20 rows and generate a report. Record this baseline accuracy score.

---

## Part 3: Iteration and Tuning

Now, conduct three distinct experiments. For each experiment, change **only one variable** in your RAG sub-workflow, run the evaluation again, and record the new score.

### Experiment 1: Semantic Chunking

* **The Change:** Purge your Vector Store. Reconfigure your Default Data Loader to split the text semantically (e.g., by the word "المادة" or by markdown headers) instead of using a fixed character count. Re-insert the 4 PDFs from `data/Tutorial_07/`.
* **The Goal:** Does preserving the structural integrity of the UOB legal articles improve the retrieval precision?

### Experiment 2: Strict Prompt Engineering

* **The Change:** Open the AI Agent in your RAG workflow and update the System Message to:
> *"You are an official academic advisor at the University of Bahrain. Answer strictly using the provided context. If the answer is not in the context, output exactly: 'Not found in official policy.' Always cite the specific Article (المادة)."*


* **The Goal:** Does a strict persona reduce hallucinations and improve the evaluation score?

### Experiment 3: Model Scaling

* **The Change:** Change the **Ollama Chat Model** node in your RAG workflow to a heavyweight model (e.g., a 70B parameter model).
* **The Goal:** Does a massive increase in model parameters improve comprehension of complex Arabic legal phrasing? Monitor the inference speed and VRAM usage—does the accuracy gain justify the slower performance?

---

## Phase 4: Execution and Results

1. After running all three experiments, open the **Evaluation Runs** log in n8n.
2. Compare the pass/fail rates across the baseline and your three experimental runs.
3. Identify which specific `evaluation_type` (e.g., Factual vs. Complex Reasoning) caused the most failures.
4. You now have quantitative proof of which pipeline configuration yields the most reliable AI advisor!
