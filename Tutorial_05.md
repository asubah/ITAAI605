# Tutorial 5: Building a Custom AI Knowledge Base (RAG)

**Objective:** Learn how to teach an AI about your own private documents. You will build a system that reads an uploaded file, converts the text into a mathematical format the AI can understand, and allows users to chat with the document using an AI Agent.

**Prerequisites:**
* Access to your n8n environment.
* An OpenAI API Key.
* A sample document (like a PDF syllabus or a text file about an interesting topic).

---

## Core Concepts: What is RAG?
Large Language Models (LLMs) are like smart students who have read the whole internet, but they haven't read *your* specific, private files. **RAG (Retrieval-Augmented Generation)** is the process of giving the AI an "open-book exam." Instead of relying on its memory, we give it a search engine to look up answers in your documents before it speaks.

To make this work, we need to understand three new concepts:
1. **Embeddings:** Computers don't understand words; they understand numbers. An "Embedding" translates a sentence into a long list of numbers (a vector). Sentences with similar meanings will have similar numbers.
2. **Vector Store:** A special kind of database designed to store these number lists (vectors). Instead of searching for exact keyword matches (like `SELECT * WHERE word='apple'`), a Vector Store searches for *mathematical closeness* (e.g., finding that "fruit" is near "apple").
3. **AI Agent:** An LLM equipped with tools. Instead of just chatting, the Agent can decide to use a "Search Tool" to look inside the Vector Store when a user asks a question.

---

## Part 1: The "Load Data" Pipeline (Ingestion)
This first half of the workflow is responsible for taking a human-readable file and shoving it into the AI's mathematical database.

### Step 1: The Trigger (Form Upload)
1. Add a **n8n Form Trigger** node.
2. This creates a simple, public web page where a user can upload a file. 
3. In the node settings, you will see a field configured to accept file types like `.pdf` and `.csv`. When a file is uploaded, the workflow starts.

### Step 2: The Default Data Loader
The uploaded file is currently just raw binary data. 
1. Add a **Default Data Loader** node and connect it to the Form Trigger.
2. This node acts as a translator. It reads the binary PDF or CSV and extracts the raw text. It also splits the text into smaller "chunks" (like paragraphs) so the AI doesn't get overwhelmed reading the whole book at once.

### Step 3: Embeddings OpenAI
1. Add an **Embeddings OpenAI** node to your canvas.
2. Connect your OpenAI credentials.
3. *What it does:* This node takes the text chunks from the Data Loader and converts them into vectors (those lists of numbers representing semantic meaning).

### Step 4: Insert Data to Store (Vector Store In-Memory)
1. Add a **Vector Store In-Memory** node. 
2. Set the mode to **Insert**.
3. **The Connections:** You will notice this node requires *two* inputs. 
   * Connect the **Default Data Loader** to the `Document` input (the text).
   * Connect the **Embeddings OpenAI** node to the `Embedding` input (the math).
4. *What it does:* This creates a temporary database holding your text and its mathematical representation. *(Note: Because it is "In-Memory," this database resets if n8n restarts. In a production environment, you would swap this for a permanent database like Pinecone or Qdrant).*

---

## Part 2: The "Retriever" Pipeline (Chatting with the Data)
Now that our document is loaded into the database, we need to build the chat interface so the user can ask questions about it.

### Step 1: The Chat Trigger
1. Add an **On chat message** trigger node to a separate, disconnected part of your canvas.
2. This provides the chat interface panel on the right side of your screen.

### Step 2: The AI Agent & Language Model
1. Add an **AI Agent** node and connect it to the Chat Trigger.
2. Attach an **OpenAI Chat Model** node (using a fast model like `gpt-4o-mini`) to the `Model` input of the Agent. 
3. *What it does:* The Agent acts as the brain. It receives the user's chat message and decides what to do next.

### Step 3: The Query Data Tool (Connecting the Database to the Brain)
This is the most critical step. We need to give the AI Agent permission to search the database we built in Part 1.
1. Add another **Vector Store In-Memory** node.
2. Set the mode to **Retrieve-as-tool**. 
3. Name the tool `knowledge_base` and give it a description like: *"Use this knowledge base to answer questions from the user based on the uploaded document."*
4. Connect this node to the `Tools` input of the **AI Agent**.
5. **The Golden Rule of Embeddings:** You must connect the exact same **Embeddings OpenAI** node from Part 1 into the `Embedding` input of this Query node. 
   * *Why?* If the text was translated into math using OpenAI, the search query must also be translated into math using OpenAI. If you use different translators, the math won't match, and the search will fail!

---

## Phase 3: Execution and Testing
1. Click **Execute Workflow**.
2. First, click the URL provided by the **Form Trigger** to open the web form. Upload a PDF (like a course syllabus or a research paper on parallel computing).
3. Wait a few seconds for the data to process through the Load pipeline.
4. Next, open the **Chat** panel in n8n.
5. Ask a highly specific question about the document you just uploaded (e.g., *"What is the grading rubric for the final project according to this syllabus?"*).
6. Watch the AI Agent call the `knowledge_base` tool, retrieve the mathematical matches, and generate a perfect, document-specific answer!
