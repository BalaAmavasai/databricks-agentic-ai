# Notebook 1: Introduction to Agentic AI and Your Databricks Environment

Welcome to the first notebook in our workshop on building agentic AI systems on Databricks with OpenAI! This notebook will introduce you to the fundamental concepts of agentic AI, the OpenAI models that can power them, and how to set up your Databricks environment for development.

**Goals for this Notebook:**
*   Understand what agentic AI is and its key characteristics.
*   Get acquainted with relevant OpenAI models and how to interact with them.
*   Prepare your Databricks workspace for building AI agents, including secure API key management.
*   Make your first successful call to the OpenAI API from Databricks.

Let's begin!

## Module 1.1: What is Agentic AI?

Agentic AI refers to artificial intelligence systems, often called **AI agents**, that can perceive their environment, make decisions, and take actions autonomously to achieve specific goals. Think of them as intelligent entities that can operate independently or with minimal human intervention.

**Key Concepts of AI Agents:**

*   **Autonomy:** Agents can operate without direct human control over every decision and action. They have a degree of self-governance.
*   **Proactiveness (or Goal-Directedness):** Agents don't just react to their environment; they can take initiative to achieve their defined goals. They exhibit goal-oriented behavior.
*   **Reactivity (or Perception):** Agents can perceive their environment (which could be digital, like a database, or physical, through sensors) and respond to changes or events within it in a timely fashion.
*   **Social Ability (or Interaction):** Agents can interact with other agents or humans, often through some form of communication language or protocol.

**Simple Examples of Agentic AI Systems:**

*   **Advanced Chatbots:** Beyond simple Q&A, agentic chatbots can understand context, remember past interactions, and even perform tasks on behalf of the user (e.g., booking an appointment).
*   **Personalized Recommendation Systems with Agency:** Imagine a movie recommender that not only suggests films but also learns your evolving preferences, proactively finds new releases you might like, and perhaps even manages a watchlist for you.
*   **Simple Task Automation Bots:** Software robots (like those in Robotic Process Automation - RPA) that can perform routine digital tasks, such as data entry, report generation, or system monitoring, based on predefined rules and goals.
*   **Smart Home Devices:** Devices that learn your habits and adjust settings (like temperature or lighting) proactively to optimize comfort and energy efficiency.

**Why Build Agentic Systems? Benefits and Use Cases:**

Developing agentic AI systems offers several advantages:

*   **Automation of Complex Tasks:** Agents can handle intricate, multi-step processes that would be time-consuming or difficult for humans.
*   **Personalized Experiences:** Agents can adapt to individual user needs and preferences, providing tailored services and interactions.
*   **Increased Efficiency and Productivity:** By automating tasks and making intelligent decisions, agents can free up human resources for more strategic work.
*   **Scalability:** Agentic systems can often be scaled to handle a large volume of tasks or interactions.
*   **Continuous Operation:** Agents can operate 24/7 without fatigue.

**Illustrative Use Cases:**

*   **Customer Service:** AI agents handling customer queries, resolving issues, and guiding users through processes.
*   **Supply Chain Management:** Agents optimizing logistics, predicting demand, and managing inventory autonomously.
*   **Personal Assistants:** Digital assistants that manage schedules, answer questions, and perform tasks for individuals.
*   **Scientific Research:** Agents that can design experiments, analyze data, and even propose new hypotheses.
*   **Financial Trading:** Algorithmic trading bots that analyze market data and execute trades based on predefined strategies.

## Module 1.2: Introduction to OpenAI Models for Agents

OpenAI provides a suite of powerful Large Language Models (LLMs) that are exceptionally well-suited for serving as the "brain" or core reasoning engine of AI agents. These models can understand and generate human-like text, making them ideal for tasks involving natural language processing, decision-making, and content creation.

**Overview of Relevant OpenAI Models:**

While OpenAI offers various models, some are particularly relevant for agentic tasks:

*   **GPT-4 and GPT-4 Turbo:** OpenAI's most capable models. They excel at tasks requiring complex reasoning, deep understanding, and creativity. GPT-4 Turbo offers a larger context window and is optimized for performance.
*   **GPT-3.5-Turbo:** A highly capable and cost-effective model. It's excellent for a wide range of applications, including dialogue, summarization, and code generation. It's often a great starting point for many agentic systems due to its balance of performance and speed.
*   **Text Embedding Models (e.g., `text-embedding-ada-002`, `text-embedding-3-small`, `text-embedding-3-large`):** These models convert text into numerical representations (embeddings) that capture semantic meaning. Embeddings are crucial for tasks like semantic search, clustering, and recommendation, which are often components of agentic systems (e.g., for Retrieval Augmented Generation - RAG).

**Understanding API Access and Basic Request/Response Structure:**

To use OpenAI models, you'll interact with their API. This typically involves:

1.  **Authentication:** You'll need an API key to authenticate your requests.
2.  **Making a Request:** You send a request to a specific model endpoint (e.g., the chat completions endpoint for GPT-3.5-Turbo or GPT-4). This request includes your input (the "prompt"), model parameters (like `temperature` for creativity, `max_tokens` for response length), and the model you want to use.
3.  **Receiving a Response:** The API processes your request and sends back a response, usually in JSON format, containing the model's output (e.g., the generated text).

**Managing API Keys:**

Your OpenAI API key is sensitive information. It's crucial to keep it secure and **never embed it directly in your code, especially if you plan to share your notebook or commit it to version control.** We'll cover how to manage this securely in Databricks in the next module.

**Core Principles of Ethical Considerations and Responsible AI:**

When developing with LLMs, it's vital to be mindful of ethical implications and strive for responsible AI practices:

*   **Bias:** LLMs are trained on vast amounts of text data from the internet, which can contain societal biases. Be aware that models might generate biased or unfair outputs. Test thoroughly and consider mitigation strategies.
*   **Misinformation:** LLMs can sometimes generate plausible-sounding but incorrect or misleading information ("hallucinations"). Always critically evaluate model outputs, especially for factual claims.
*   **Transparency:** Be clear about when users are interacting with an AI agent versus a human.
*   **Privacy:** Be cautious with sensitive user data. Ensure you have appropriate consent and data handling practices if your agent processes personal information.
*   **Intended Use:** Design your agents for beneficial purposes and consider potential misuse.

OpenAI provides usage guidelines and safety best practices that you should familiarize yourself with.

## Module 1.3: Setting up Your Databricks Environment for Agent Development

Databricks provides a powerful and collaborative platform for data science, machine learning, and AI development. Let's get your environment ready.

**Brief Overview of the Databricks Workspace:**

The Databricks workspace is a web-based interface where you can:
*   Create and manage notebooks (like this one!) for Python, Scala, SQL, and R.
*   Manage clusters (the compute resources that run your code).
*   Access data (e.g., from Delta Lake, cloud storage).
*   Use tools like MLflow for experiment tracking and model management.

For this workshop, we'll primarily be working within Databricks notebooks.

**Guidance on Installing Necessary Python Libraries:**

To interact with the OpenAI API, you'll need the `openai` Python library. You can install libraries in Databricks at the notebook scope or cluster scope.

For notebook-scoped installation (recommended for quick setup and specific notebook dependencies), you can use `%pip` in a notebook cell:

```python
# In a new cell in your Databricks notebook, run:
%pip install openai
```

After running this, you might need to detach and reattach the notebook to the cluster for the library to become available, or simply restart the Python kernel if prompted.

**Best Practices for Storing API Keys Securely Using Databricks Secrets:**

As mentioned, never hardcode your API keys. Databricks offers a secure way to store and access secrets called **Databricks Secrets**.

**Steps to Create a Secret Scope and Store Your API Key:**

1.  **Install Databricks CLI (if you haven't already):** You'll typically do this on your local machine, not within the Databricks notebook itself. Follow the official Databricks documentation to install and configure the Databricks CLI.
    *   `pip install databricks-cli`
    *   Configure it with your Databricks host and a personal access token: `databricks configure --token`

2.  **Create a Secret Scope:** A secret scope is a collection of secrets. It's good practice to create a scope for your project or team.
    *   From your local terminal (with Databricks CLI configured): `databricks secrets create-scope --scope agentic-ai-workshop`
    *   (You can choose any scope name. If you get an error that it already exists, you can list existing scopes with `databricks secrets list-scopes` and use an existing one or choose a new name.)

3.  **Add Your OpenAI API Key as a Secret:**
    *   From your local terminal: `databricks secrets put --scope agentic-ai-workshop --key openai_api_key`
    *   This command will open a simple text editor. Paste your OpenAI API key into the editor, save, and close it. Your key is now securely stored.

**Accessing the Secret in Your Notebook:**

Once the secret is stored, you can access it in your Databricks notebook using the `dbutils.secrets.get()` utility:

```python
# In a Databricks notebook cell:
# Ensure you have created the secret scope "agentic-ai-workshop" 
# and the secret key "openai_api_key" as described above.

# openai_api_key = dbutils.secrets.get(scope="agentic-ai-workshop", key="openai_api_key")

# For the purpose of this example, if you haven't set up secrets yet,
# you can temporarily assign your key here for testing, BUT REMEMBER TO REMOVE IT.
# **NEVER COMMIT A NOTEBOOK WITH A HARDCODED API KEY.**
# openai_api_key = "YOUR_API_KEY_HERE" # <-- Replace and then remove after testing if not using secrets
```

**Important Security Note:** The line `openai_api_key = "YOUR_API_KEY_HERE"` is **ONLY for initial, temporary testing if you haven't set up Databricks Secrets yet.** Always prioritize using Databricks Secrets. If you do use it temporarily, **delete the key from the cell immediately after testing and before saving or sharing the notebook.**

**Hands-on: Your First OpenAI API Call from a Databricks Notebook**

Now, let's put it all together and make a simple call to the OpenAI API.

```python
import openai

# --- Securely Get API Key (Recommended Method) ---
try:
    openai_api_key = dbutils.secrets.get(scope="agentic-ai-workshop", key="openai_api_key")
    print("Successfully retrieved OpenAI API key from Databricks Secrets.")
except Exception as e:
    print(f"Error retrieving API key from Databricks Secrets: {e}")
    print("Please ensure you have created a secret scope named 'agentic-ai-workshop' and a secret named 'openai_api_key' with your OpenAI API key.")
    print("For temporary testing ONLY, you can manually set the API key in the next cell, but this is NOT recommended for production or shared notebooks.")
    openai_api_key = None # Set to None if not found

# --- OR: Temporary Manual API Key (NOT RECOMMENDED - REMOVE AFTER USE) ---
# if openai_api_key is None:
#     manual_key_input = input("Enter your OpenAI API key for temporary use (press Enter to skip): ")
#     if manual_key_input:
#         openai_api_key = manual_key_input
#         print("Temporary API key set. REMEMBER TO REMOVE THIS FROM YOUR NOTEBOOK.")
#     else:
#         print("API key not set. API call will likely fail.")

# Configure the OpenAI library with your API key
if openai_api_key:
    openai.api_key = openai_api_key

    try:
        # Let's try a simple chat completion
        response = openai.chat.completions.create(
            model="gpt-3.5-turbo",  # You can also try "gpt-4" if you have access
            messages=[
                {"role": "system", "content": "You are a helpful assistant."},
                {"role": "user", "content": "What is the weather like in San Francisco today?"} # A simple test query
            ],
            max_tokens=50 # Limit the response length for this test
        )
        
        # Print the response from the assistant
        # The actual message content is in response.choices[0].message.content
        print("\nAssistant's Response:")
        print(response.choices[0].message.content)

    except openai.APIError as e:
        print(f"An OpenAI API error occurred: {e}")
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
else:
    print("\nOpenAI API key not configured. Skipping API call.")

```

**Expected Output (will vary):**

You should see a response from the assistant, something like:

```
Successfully retrieved OpenAI API key from Databricks Secrets.

Assistant's Response:
I am an AI and do not have real-time access to weather information. To get the current weather in San Francisco, please check a reliable weather website or app!
```

(Note: GPT models don't have real-time internet access by default, so their weather answers will be generic like this. The goal here is to confirm the API call works.)

**Congratulations!** You've successfully set up your environment and made your first call to the OpenAI API from a Databricks notebook. You're now ready to dive deeper into building the core components of an AI agent in the next notebook.

---

Next up: **Notebook 2: Core Components of an AI Agent**

