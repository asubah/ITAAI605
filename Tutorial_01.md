# LLM Deployment and Consumption Tutorial

Welcome to the ITAAI605 tutorial on deploying, managing, and consuming Large Language Models (LLMs). In this guide, we will cover how to access the university's High-Performance Computing (HPC) cluster, deploy a local model using Apptainer and Ollama, expose the API to your local machine, and consume cloud-hosted models as an alternative.

---

## 1. Accessing the Hayrat Cluster

You will first need to log in to our HPC cluster at `hayrat.uob.edu.bh`. You can do this using either a standard SSH client or the web-based Open OnDemand dashboard.

**Option A: Open OnDemand Dashboard**
1. Open your web browser and navigate to `https://hayrat.uob.edu.bh`.
2. Log in with your university credentials.
3. In the top navigation menu, click on **Clusters** -> **Hayrat Shell Access**. This will open a terminal directly in your browser.

**Option B: SSH**

Alternatively, you can access the cluster via your terminal:
```bash
ssh <your_username>@hayrat.uob.edu.bh

```

---

## 2. Setting Up Your Workspace

The default home directory on the cluster is strictly limited to 5GB, which is not enough to store large model weights and datasets. You must create and navigate to a dedicated workspace on the data drive.

```bash
# Create a working directory using your username
mkdir -p /data/datasets/$USER

# Navigate to your new workspace
cd /data/datasets/$USER

```

---

## 3. Allocating Compute Resources

We use Slurm as our resource manager. To execute heavy workloads like running an LLM, you must allocate a compute node rather than running processes on the login node.

Request an interactive session on a compute node for 3 hours:

```bash
srun -t 3:00:00 -p compute --pty /bin/bash

```

---

## 4. Deploying Local Models with Apptainer and Ollama

Once you are on a compute node, you will pull the Ollama container and set up your local model environment.

**Step 4.1: Pull the Container and Prepare Directories**

```bash
# Pull the latest Ollama container image
apptainer pull docker://ollama/ollama:latest

# Create a local directory to store downloaded model weights
mkdir -p models

```

**Step 4.2: Start the Ollama Instance**
We will start the Ollama instance in the background, binding our local `models` directory to ensure the large files are saved to our expanded workspace.

```bash
apptainer instance run --nv --bind models:/models --env "OLLAMA_MODELS=/models" ollama_latest.sif ollama start

```

**Step 4.3: Pull and Run the Model**
For this tutorial, we are using the `qwen3:14b` model.

```bash
# Pull the model weights
apptainer run --nv instance://ollama pull qwen3:14b

# Serve the model (ensures the API is actively listening)
apptainer run --nv instance://ollama serve qwen3:14b

# Alternatively, to run the model interactively in the terminal:
apptainer run --nv instance://ollama run qwen3:14b

```

---

## 5. Exposing the API for Local Consumption (Ngrok)

By default, the Ollama API runs on `localhost:11434` on the compute node. To interact with this API from your personal laptop, you can use `ngrok` to create a secure, public URL forwarding to that port.

First, download and extract the ngrok executable, then configure it with your authentication token (you will need to log into your free ngrok account to retrieve your token):

```bash
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
tar -xzf ngrok-v3-stable-linux-amd64.tgz
./ngrok config add-authtoken <your_authtoken_here>

```

Once configured, run the following command to expose the Ollama port:

```bash
./ngrok http 11434

```

Ngrok will output a Forwarding URL (e.g., `https://<random-string>.ngrok-free.app`). **Copy this URL**, as you will need it for your Python script on your laptop.

---

## 6. Consuming the Local API via Python

Now, open a terminal on your **local laptop** to create a Python environment and write a script to interact with your hosted model.

**Step 6.1: Setup Virtual Environment**

```bash
# Create a virtual environment
python3 -m venv llm_env

# Activate the environment (Linux/macOS)
source llm_env/bin/activate
# (For Windows: llm_env\Scripts\activate)

# Install required packages
pip install requests

```

**Step 6.2: Python Demo Script**
Create a file named `demo.py` and paste the following code. Make sure to replace `YOUR_NGROK_URL` with the forwarding URL you generated in Step 5.

```python
import requests
import json

# Replace with your actual Ngrok forwarding URL
BASE_URL = "YOUR_NGROK_URL" 
API_URL = f"{BASE_URL}/api/generate"

def query_ollama(prompt):
    payload = {
        "model": "qwen3:14b",
        "prompt": prompt,
        "stream": False
    }
    
    headers = {
        "Content-Type": "application/json"
    }

    try:
        response = requests.post(API_URL, data=json.dumps(payload), headers=headers)
        response.raise_for_status()
        result = response.json()
        print("Model Response:\n")
        print(result.get("response", "No response text found."))
    except requests.exceptions.RequestException as e:
        print(f"Error connecting to the API: {e}")

if __name__ == "__main__":
    user_prompt = "Explain the importance of High-Performance Computing in modern AI."
    print(f"Prompting qwen3:14b: '{user_prompt}'\n")
    query_ollama(user_prompt)

```

Run the script using `python demo.py`.

---

## 7. Alternative: Using Cloud-Hosted Models (OpenRouter)

If you prefer not to host the model yourself, or want access to a wider variety of proprietary and open-source models, you can use a cloud aggregator like OpenRouter.

**How to set up OpenRouter:**

1. Visit [OpenRouter.ai](https://openrouter.ai/) and click **Create Account**.
2. Once logged in, navigate to **Settings** from the main dashboard.
3. Go to the **API Keys** section.
4. Click **Create Key**, give it a name (e.g., "ITAAI605 Project"), and securely copy the generated key. *Note: You will not be able to see this key again once you close the window.*

You can now use this API key with the standard OpenAI Python library to route requests to virtually any model available on their platform.
