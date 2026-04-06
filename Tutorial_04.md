# Tutorial 4: Building an Autonomous Data Analyst Agent

**Objective:** Transition from linear, step-by-step workflows to Agentic AI. You will build a chatbot that autonomously decides when to fetch open government data and how to calculate answers to user questions using specialized tools.

**Prerequisites:**
* Access to your n8n environment.
* Your personal OpenRouter API Key.

## Step 1: The Conversational Trigger
Instead of a manual button, this workflow reacts to chat messages.
1. Open a new workflow.
2. Click **Add first step** and select the **On chat message** trigger node. 
3. This creates a built-in chat interface panel on the right side of your n8n editor.

## Step 2: The Agentic Brain
We need an orchestrator to manage the logic and memory.
1. Add an **AI Agent** node next to the trigger.
2. Set the **Agent Type** to `Tools Agent`.
3. **Connect a Model:** Click the **+** icon on the `Model` input of the AI Agent and attach an **OpenRouter Chat Model**. Configure it with your credentials.
4. **Connect Memory:** Click the **+** icon on the `Memory` input and attach a **Window Buffer Memory** node. Set the window size to `5` so the agent remembers the context of the conversation.

## Step 3: Forging the Tools (The Data Engine)
LLMs cannot browse live data or do reliable math on their own. We must provide tools.
1. Click the **+** icon on the `Tools` input of the AI Agent and attach an **HTTP Request Tool**.
2. Configure the HTTP Request Tool:
   * **Name:** `Fetch_Bahrain_Air_Quality` *(Note: No spaces allowed)*.
   * **Description:** `Call this tool to get real-time environmental and air quality data in Bahrain. It returns JSON data containing metrics like pm10 and so2 from various stations.`
   * **Method:** `GET`
   * **URL:** `https://www.data.gov.bh/api/explore/v2.1/catalog/datasets/02-air-quality-by-month-station/records?limit=20`
3. Click the **+** icon on the `Tools` input again and attach a **Calculator Tool**. The default description is perfect.

## Step 4: System Prompt & Execution
1. Open the **AI Agent** node settings.
2. In the **System Message** (or Prompt) field, define the agent's rules:
   > You are an expert Data Scientist. You have access to real-time air quality data from Bahrain's open data portal via your tools. 
   > When a user asks a question:
   > 1. Fetch the data using your HTTP tool.
   > 2. Use your calculator tool to find exact averages or maximums to avoid hallucinating math.
   > 3. Provide a clear, analytical answer.
3. Open the **Chat** panel at the bottom right of the screen.
4. Type: *"What is the average PM10 pollution level across the stations right now?"*
5. Watch the node execution! You will see the agent autonomously decide to trigger the HTTP tool, read the JSON, trigger the Calculator tool, and formulate your final answer.
