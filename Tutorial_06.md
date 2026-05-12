# Tutorial 6: Automating AI Evaluation and Accuracy Testing

**Objective:** Learn how to move beyond manual testing by building an automated evaluation loop. You will learn how to pull questions from a dataset, have your AI Agent categorize them using your RAG knowledge base, and automatically compare the AI's answers against a "ground truth" to measure performance.

**Prerequisites:**

* Completion of **Tutorial 5: Building a Custom AI Knowledge Base (RAG)**.
* An n8n Data Table populated with test questions (e.g., `Evaluation_Dataset`).
* Local Ollama instance running `qwen3:8b` and `qwen3-embedding`.

---

## Core Concepts: Why Evaluate?

Building a RAG system is only the first step. To ensure the AI is reliable, you must test it against many examples. **Automated Evaluation** allows you to:

1. **Benchmark Accuracy:** Instead of asking one question at a time, you can test 100 questions automatically.
2. **Identify Hallucinations:** Determine if the AI is misclassifying information or making up facts.
3. **Structured Output:** Forcing the AI to provide specific categories (like "Plagiarism" or "AI Use") makes it easier to track success rates mathematically.

---

## Part 1: The "Evaluation Trigger" (Batch Processing)

Instead of waiting for a human to type a message, this workflow pulls questions directly from a database.

### Step 1: Fetching the Dataset

1. Add a **When fetching a dataset row** node.
2. Select your `Evaluation_Dataset` table.
3. *What it does:* This node acts as a loop. It triggers the workflow for every single row in your table, passing the `Question` and the correct `Class` (the "ground truth") into the workflow.

### Step 2: Mapping Fields

1. Add an **Edit Fields (Set)** node.
2. Map the data so the AI Agent understands it. Use an expression like `{{ $json.Question }}` to set a field called `chatInput`.

---

## Part 2: The "Classification" Agent

In this workflow, the Agent isn't just chatting; it is acting as a specialized classifier for University of Bahrain (UOB) regulations.

### Step 1: The Specialized System Prompt

1. In the **AI Agent** node, update the **System Message**.
2. Instruct the agent to act as UOB admin staff and classify questions into the 4 specific Arabic categories:
* "نظام الدراسة والاختبارات" (Exams/Study)
* "نظام الدراسات العليا" (Graduate Studies)
* "الانتحال الأكاديمي" (Plagiarism)
* "استخدام الذكاء الاصطناعي" (AI Use).



### Step 2: Structured Output Parser

1. Add a **Structured Output Parser** node and connect it to the Agent.
2. Define a schema (e.g., `category`).
3. *What it does:* It forces the AI to respond with *only* the category name in a JSON format. This prevents the AI from adding conversational fluff like "Sure, I can help with that!" which would break your evaluation math.

### Step 3: Local LLM with Ollama

1. Attach the **Ollama Chat Model** (`qwen3:8b`) and **Embeddings Ollama** (`qwen3-embedding`) to the Agent and Vector Store.
2. *Note:* Using local models allows you to run hundreds of evaluation tests without incurring API costs.

---

## Part 3: The "Comparison & Logging" Pipeline

This is where we determine if the AI got the answer right or wrong.

### Step 1: The Evaluation Node

1. Add an **Evaluation** node.
2. Configure **Set Metrics**:
* **Actual Answer:** The output from the AI Agent (`{{ $json.output.category }}`).
* **Expected Answer:** The correct class from your original dataset (`{{ $('When fetching a dataset row').item.json.Class }}`).


3. *What it does:* It compares the two strings. If they match exactly, it logs a "Success."

### Step 2: Updating the Data Table

1. Add a **Data Table (Update)** node.
2. Configure it to find the row that matches the current question ID.
3. Map the AI's result (`ModelClass`) back into the table.
4. *What it does:* This creates a permanent record of the AI's performance, allowing you to see exactly which questions the AI is struggling with.

---

## Phase 4: Execution and Results

1. Click **Execute Workflow**.
2. Watch as the workflow cycles through your dataset.
3. Once finished, open your **n8n Data Table**.
4. Compare the **Class** column (Correct Answer) with the **ModelClass** column (AI Answer). You now have a clear accuracy percentage for your RAG system!
