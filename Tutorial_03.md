# Tutorial 3: Building an AI-Powered ETL Pipeline (Extract, Transform, Load)

**Objective:** Learn how to build a stateful automation pipeline. You will extract live news data, use a locally hosted Large Language Model (LLM) to transform and analyze the text into structured JSON, and load the results into a persistent database using n8n Data Tables.

**Prerequisites:**
* Access to your n8n environment.
* A running instance of **Ollama** with a model pulled (e.g., `llama3.2:latest`).
* A free API key from [NewsAPI.org](https://newsapi.org/) (takes 1 minute to generate).

## Step 1: Create the Destination (Data Table)
Before fetching data, we need a place to store it. We will use n8n's built-in database feature.
1. In the left-hand menu of n8n, click on **Data Tables**.
2. Click **Create Data Table** and name it `HPC_News_Vault`.
3. Add the following columns by clicking **Add Column**:
   * **Title** (Type: String)
   * **Summary** (Type: String)
   * **Sentiment** (Type: String)
   * **URL** (Type: String)

## Step 2: Extract (Fetch Live News Data)
We will pull the latest articles about High-Performance Computing.
1. Open a new workflow and add a **Manual Trigger** node.
2. Click the **+** icon and add an **HTTP Request** node.
3. Configure the node:
   * **Method:** `GET`
   * **URL:** `https://newsapi.org/v2/everything?q="High-Performance Computing"&pageSize=10&apiKey=YOUR_NEWS_API_KEY` *(Replace with your actual key)*.
4. Click **Execute Node**. The output will contain an `articles` array containing 10 recent news items.
5. Add a **Split Out** node directly after the HTTP Request. 
   * **Field To Split Out:** Type `articles`.
   * This ensures n8n processes each of the 10 news articles as a separate, individual item moving forward.

## Step 3: Transform (AI Processing & Structured Output)
Now, we instruct the AI to read each article. Unlike previous tutorials where we hoped the AI would format its text nicely, we will now *force* it to output structured JSON data so our database can read it cleanly.

1. Add a **Basic LLM Chain** node after your Split Out node.
2. **Connect the Model:** Click the **+** icon on the `Model` input of the LLM Chain and attach an **Ollama Chat Model**. 
   * Configure your Ollama credentials.
   * **Model:** `llama3.2:latest` (or whichever model you have pulled).
   * **Options:** Click Add Option, select **Format**, and choose `json`.
   3. **Connect the Output Parser:** Click the **+** icon on the `Output Parser` input (bottom left of the LLM Chain) and attach a **Structured Output Parser** node. 
   * In the parser's settings, paste the following JSON Schema Example. This acts as a strict template for the LLM:
     ```json
     {
       "URL": "https://www.example.com",
       "Title": "News Article Title",
       "Summary": "One sentence article summary",
       "Sentiment": "Positive"
     }
     ```
   4. **Configure the Prompt:** Click back into the **Basic LLM Chain** node settings. Notice how we no longer need to manually beg the AI to format its text. We just define the task:
   > Analyze the following news article:
   > - Provide a concise 1-sentence summary.
   > - Determine the sentiment of the news (Positive, Neutral, or Negative).
   > 
   > Title: '{{ $json.title }}'. Description: '{{ $json.description }}'. URL: {{ $json.url }}.

## Step 4: Load (Persist Data)
Finally, we save the perfectly structured AI analysis to our table.
1. Add an **n8n Data Table** node after the LLM Chain.
2. Configure it as follows:
   * **Operation:** `Insert row` (or `Upsert` to prevent duplicates on multiple runs).
   * **Data Table:** Select `HPC_News_Vault`.
   * **Columns -> Mapping Mode:** Select `Define Below`.
3. Because we used the Structured Output Parser, the AI's response is neatly tucked inside an `output` object. Map the fields exactly like this using expressions:
   * **URL:** `{{ $json.output.URL }}`
   * **Title:** `{{ $json.output.Title }}`
   * **Summary:** `{{ $json.output.Summary }}`
   * **Sentiment:** `{{ $json.output.Sentiment }}`
4. Click **Execute Workflow** and check your Data Table in the left menu to see the populated records!
