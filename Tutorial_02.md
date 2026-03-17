# Introduction to n8n & Foundational Models

**Objective:** Learn how to chain web services together to automate tasks. You will fetch data from a public API and dynamically pass it to a Large Language Model (LLM) using n8n's Advanced AI tools.

**Prerequisites:**
* Access to your n8n environment.
* Your personal OpenRouter API Key.

## Step 1: Start with a Manual Trigger
Every n8n workflow needs a starting point (a trigger). We will use a manual trigger so you can test the workflow whenever you want.
1. Open a new workflow in n8n.
2. Click **Add first step**.
3. Search for and select **Manual Trigger** (or "On clicking 'Execute'").

## Step 2: Fetch Public Data (The Data Source)
We need some data for the AI to process. We'll use a free API to fetch a random quote.
1. Click the **+** icon next to the Manual Trigger node.
2. Search for and select the **HTTP Request** node.
3. Configure the node as follows:
   * **Method:** `GET`
   * **URL:** `https://dummyjson.com/quotes/random`
4. Click **Execute Node**. In the output panel on the right, you should see a JSON response containing a `"quote"` and an `"author"`. 
5. Close the node settings.

## Step 3: Connect to the Foundational Model (Using Native AI Nodes)
Now, we will send this quote to OpenRouter using n8n's dedicated AI integration. 

1. Click the **+** icon next to your HTTP Request node.
2. Search for the **Basic LLM Chain** node and add it to the canvas. This node acts as the brain of the operation, taking your prompt and sending it to a model.
3. You will notice the Basic LLM Chain node has a required connection point on its left side labeled **Model**. Click the **+** icon attached to that specific input.
4. Search for and select the **OpenRouter Chat Model** node.
5. Configure the OpenRouter Chat Model node:
   * **Credential:** Click the dropdown, select **Create New Credential**, and paste your OpenRouter API Key. Save and close the credential window.
   * **Model:** Select a free model from the dropdown (for example, `openrouter/free`). *If it isn't listed, hover over the field, set it to "Expression", and paste the model name.*
6. Click back into the **Basic LLM Chain** node settings.
7. In the **Prompt** field, paste the following text. Notice how we use n8n's expression syntax (`{{ $json.quote }}`) to dynamically insert the data from the previous HTTP Request node:
   > You are a philosophical assistant. Please translate the following quote to Arabic, and then explain its core meaning in one simple sentence. The quote is: '{{ $json.quote }}' by {{ $json.author }}.

## Step 4: Execute and Observe
1. Close the node settings.
2. At the bottom of the n8n canvas, click the big **Execute Workflow** button.
3. Watch the data flow through the nodes. 
4. Click on the **Basic LLM Chain** node to view the final output. You will see the AI's Arabic translation and philosophical breakdown of the random quote.

---

## Why is this useful?
This simple lab demonstrates the core pattern of modern AI integration:
* **Connectivity:** n8n easily bridges the gap between disparate systems.
* **Dynamic Prompting:** You aren't just chatting with an AI; you are programmatically passing it live, dynamic data gathered from an external system.
* **Scalability:** Imagine replacing Step 2 with a node that reads newly submitted student feedback, and Step 3 with a prompt that extracts sentiment and categorizes complaints. You've just automated an entire analysis process.

---

**Additional Resource:**
[Ultimate Guide to the OpenRouter AI Node in n8n](https://www.youtube.com/watch?v=cB47meZF428) 
*This video provides a great visual walkthrough on setting up the native OpenRouter node and configuring the credentials in n8n.*
