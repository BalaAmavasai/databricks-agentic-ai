# Notebook 3: Building a Simple Agentic Application on Databricks

Welcome to Notebook 3! So far, we've covered the foundational concepts of agentic AI, OpenAI models, setting up your Databricks environment, and the core components like the agent loop, prompt engineering, memory, and tool use. Now, it's time to bring these elements together to build a simple, functional agentic application.

**Goals for this Notebook:**
*   Define a clear goal, persona, and capabilities for a simple AI agent.
*   Implement the agent's logic using OpenAI's GPT models as the core reasoning engine.
*   Integrate basic memory and a simple tool (for information retrieval) into the agent.
*   Introduce and implement a basic version of Retrieval Augmented Generation (RAG) to allow the agent to answer questions based on a provided text document.
*   Discuss basic strategies for testing and iterating on your agent.

Let's build an agent!

## Module 3.1: Defining Your Agent's Goal, Persona, and Capabilities

Before writing any code for an agent, it's crucial to clearly define what you want it to do.

**For our workshop, let's build a "Document Q&A Agent."**

*   **Goal:** The agent's primary goal is to answer user questions based *only* on the content of a specific text document provided to it. It should not use its general knowledge from its pre-training if the answer can be found in the document.
*   **Persona:** "You are a helpful and precise Q&A assistant. Your knowledge is strictly limited to the document provided. If the answer is not in the document, clearly state that the information is not available in the provided text."
*   **Capabilities (and Tools):**
    1.  **Understand User Questions:** Process natural language questions from the user.
    2.  **Retrieve Relevant Information:** Access and search through a given text document to find parts relevant to the user's question. This will be our agent's primary "tool."
    3.  **Generate Answers:** Synthesize an answer based on the retrieved information using an OpenAI LLM.
    4.  **Maintain Short-Term Memory:** Remember the immediate context of the conversation (though for a simple Q&A bot, this might be less critical than for a multi-turn dialogue agent, we'll keep the structure from Notebook 2).
    5.  **Handle Out-of-Scope Questions:** Politely inform the user if the answer cannot be found in the document.

**The Document:**
For this exercise, we'll use a short, simple text document. Let's create one about a fictional planet.

```python
# Create a dummy text file in Databricks File System (DBFS) or locally for the agent to use.
# We'll write this to a file that our agent can then read.

fictional_planet_text = """
Planet Xylar: A Quick Overview

Planet Xylar is a fictional planet located in the Andromeda galaxy, approximately 2.5 million light-years from Earth. 
It was first conceptualized by science fiction author Dr. Aris Thorne in his 2023 novel, 'Echoes of Xylar'.
Xylar is known for its unique bioluminescent flora and fauna, which illuminate the planet's surface during its long nights. 
The atmosphere of Xylar is rich in oxygen and nitrogen, similar to Earth, but also contains trace amounts of a unique noble gas called "Xylium," which is believed to contribute to the bioluminescence.

The sentient inhabitants of Xylar are known as the Xylarians. They are a peaceful, technologically advanced species with a deep connection to their planet's ecosystem. 
Xylarians communicate through a combination of melodic vocalizations and light patterns emitted from their skin. 
Their primary energy source is geothermal, harnessing the planet's active volcanic core. 
Key exports from Xylar (in fictional trade agreements) include Xylium gas and intricately woven light-fiber textiles.
There is no mention of Xylar having any moons in Dr. Thorne's novel.
Average surface temperature on Xylar is a comfortable 25 degrees Celsius.
"""

# In a Databricks notebook, you can write this to DBFS:
# dbutils.fs.put("/tmp/xylar_document.txt", fictional_planet_text, True)

# For local execution (if not in Databricks or for easier testing initially):
with open("xylar_document.txt", "w") as f:
    f.write(fictional_planet_text)

print("Fictional planet document created as xylar_document.txt")
```

Run the cell above to create the `xylar_document.txt` file in your current working directory (or DBFS if you use the `dbutils` command in a Databricks notebook).

## Module 3.2: Implementing the Agent Logic with OpenAI

Now, let's start building the agent. We'll use the OpenAI GPT-3.5-Turbo model as its reasoning engine.

**Core Agent Structure:**

Our agent will follow these general steps for each user query:
1.  Receive user question.
2.  **Retrieve relevant context:** Use a simple retrieval mechanism (our "tool") to find parts of `xylar_document.txt` that are relevant to the question.
3.  **Construct a prompt:** Create a prompt for the LLM that includes:
    *   The agent's persona and instructions.
    *   The retrieved context from the document.
    *   The user's question.
4.  **Call OpenAI API:** Send the prompt to the LLM.
5.  **Return Answer:** Present the LLM's response to the user.

This process is a basic form of **Retrieval Augmented Generation (RAG)**.

## Module 3.3: Introduction to Basic RAG (Retrieval Augmented Generation)

**Retrieval Augmented Generation (RAG)** is a technique to improve the responses of LLMs by providing them with relevant information retrieved from an external knowledge base *at inference time*.

**Why RAG?**

*   **Reduces Hallucinations:** LLMs sometimes make up facts. Grounding their responses in factual, retrieved documents makes them more accurate.
*   **Access to Current/Proprietary Data:** LLMs are trained on a fixed dataset. RAG allows them to answer questions about information not in their training data (e.g., your company's internal documents, very recent news).
*   **Transparency and Verifiability:** By knowing which documents were used to generate an answer, users can often verify the information.

**Simple RAG Implementation:**

For our beginner-friendly agent, our RAG will consist of:

1.  **Knowledge Base:** The `xylar_document.txt` file.
2.  **Retriever:** A very simple keyword-based retriever. It will split the document into sentences (or small chunks) and find those that contain keywords from the user's question. More advanced RAG systems use semantic search with embeddings, but that adds complexity we'll avoid for now.
3.  **Generator:** The OpenAI LLM, which takes the user's question and the retrieved context to generate an answer.

**Let's implement the retriever first.**

```python
# Simple Retriever Function

def load_document(filepath="xylar_document.txt"):
    try:
        with open(filepath, "r") as f:
            return f.read()
    except FileNotFoundError:
        print(f"Error: Document not found at {filepath}")
        return None

def simple_retriever(query, document_text, num_chunks=2):
    """A very basic keyword-based retriever.
    Splits document into sentences and returns sentences containing keywords from the query.
    """
    if not document_text:
        return ""
    
    # Simple sentence splitting (can be improved with NLTK or spaCy for more complex texts)
    sentences = [s.strip() for s in document_text.split(".") if s.strip()]
    query_keywords = set(query.lower().split())
    
    relevant_sentences = []
    for sentence in sentences:
        sentence_words = set(sentence.lower().split())
        # Check for overlap in keywords; more sophisticated methods exist
        if any(keyword in sentence_words for keyword in query_keywords):
            relevant_sentences.append(sentence)
            
    # For simplicity, just join the top N relevant sentences found
    # A real retriever would rank them by relevance.
    return ". ".join(relevant_sentences[:num_chunks]) + ("." if relevant_sentences else "")

# Test the retriever
doc_content = load_document()
if doc_content:
    test_query = "What is Xylar known for?"
    retrieved_context = simple_retriever(test_query, doc_content)
    print(f"Query: {test_query}")
    print(f"Retrieved Context:\n{retrieved_context}")

    test_query_2 = "Does Xylar have moons?"
    retrieved_context_2 = simple_retriever(test_query_2, doc_content)
    print(f"\nQuery: {test_query_2}")
    print(f"Retrieved Context 2:\n{retrieved_context_2}")
else:
    print("Document content not loaded, skipping retriever test.")

```

This retriever is very basic. For instance, it doesn't handle synonyms well and might miss relevant information or pick up irrelevant sentences. Production-grade RAG systems use vector embeddings and semantic search for much more accurate retrieval.

**Now, let's integrate this into our agent.**

```python
import openai
import os

# --- API Key Setup (ensure this is done as in Notebook 1 & 2) ---
if 'dbutils' in globals():
    openai_api_key = dbutils.secrets.get(scope="agentic-ai-workshop", key="openai_api_key")
    print("Using Databricks secrets for API key.")
elif os.environ.get("OPENAI_API_KEY"):
    openai_api_key = os.environ.get("OPENAI_API_KEY")
    print("Using OPENAI_API_KEY environment variable.")
else:
    openai_api_key = None # Or prompt user, or handle error
    print("OpenAI API key not found. Please set it up.")

if openai_api_key:
    openai.api_key = openai_api_key

# Load the document content once
document_content = load_document()

conversation_history = [] # We can maintain a history if we want multi-turn Q&A

def document_qa_agent(user_question):
    global conversation_history # Allow modification of global history
    global document_content

    if not openai_api_key:
        return "Error: OpenAI API key not configured."
    if not document_content:
        return "Error: Document content for Q&A is not loaded."

    print(f"\nUser Question: {user_question}")

    # 1. Retrieve relevant context (Our RAG step)
    retrieved_context = simple_retriever(user_question, document_content, num_chunks=3)
    print(f"Retrieved Context Snippet: '{retrieved_context[:100]}...'" if retrieved_context else "No context retrieved.")

    # 2. Construct the prompt for the LLM
    system_prompt = (
        "You are a helpful and precise Q&A assistant. "
        "Your knowledge is strictly limited to the provided context from the document. "
        "Answer the user's question based ONLY on this context. "
        "If the answer is not found in the provided context, clearly state that the information is not available in the document. "
        "Do not use any external knowledge."
    )
    
    # Prepare messages for the API, including conversation history if desired
    # For a simple Q&A, history might be just the system prompt + current context + question
    # For a more conversational agent, you'd append to a longer history list.
    
    current_messages = [
        {"role": "system", "content": system_prompt}
    ]
    
    # Add previous turns if you want to maintain conversation context
    # For this simple RAG Q&A, we might simplify and not use extensive history for each query,
    # focusing instead on the retrieved context for *this specific question*.
    # However, let's keep the structure for potential extension.
    # current_messages.extend(conversation_history) 

    prompt_for_llm = (
        f"Context from the document:\n---\n{retrieved_context if retrieved_context else 'No relevant context found in the document.'}\n---\nUser Question: {user_question}"
    )
    
    current_messages.append({"role": "user", "content": prompt_for_llm})

    try:
        # 3. Call OpenAI API
        response = openai.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=current_messages,
            temperature=0.2, # Lower temperature for more factual, less creative answers
            max_tokens=250
        )
        assistant_response = response.choices[0].message.content.strip()

        # Optional: Add this interaction to conversation history for multi-turn
        # conversation_history.append({"role": "user", "content": user_question}) # Store original simple question
        # conversation_history.append({"role": "assistant", "content": assistant_response})
        
        # Limit history size to avoid excessive token usage (simple approach)
        # MAX_HISTORY_TURNS = 3 # 3 pairs of user/assistant messages
        # if len(conversation_history) > MAX_HISTORY_TURNS * 2:
        #     conversation_history = conversation_history[-(MAX_HISTORY_TURNS*2):]

    except Exception as e:
        assistant_response = f"An error occurred while contacting OpenAI: {e}"
    
    print(f"Agent Response: {assistant_response}")
    return assistant_response

# --- Hands-on: Test the Document Q&A Agent ---
if openai_api_key and document_content:
    document_qa_agent("What is Planet Xylar known for?")
    document_qa_agent("Who are the inhabitants of Xylar and how do they communicate?")
    document_qa_agent("What is Xylar's primary energy source?")
    document_qa_agent("Does Xylar have any moons?")
    document_qa_agent("What is the capital of France?") # Test out-of-scope question
    document_qa_agent("Tell me about Dr. Aris Thorne.")
else:
    print("Skipping agent tests due to missing API key or document content.")

```

**Running the Agent:**

When you run the tests above, observe:
*   How the agent uses the retrieved context to answer questions about Xylar.
*   How it responds when the answer is explicitly mentioned as not being in the document (e.g., about moons).
*   How it responds to an out-of-scope question (e.g., "What is the capital of France?"). It should state that the information is not in the provided document.

**Further Exploration (Optional, if time permits and complexity can be managed for beginners):**

*   **OpenAI Embeddings for Semantic Search:** Instead of keyword matching, you could:
    1.  Chunk the document into smaller pieces.
    2.  Use an OpenAI embedding model (e.g., `text-embedding-3-small`) to create a numerical vector (embedding) for each chunk.
    3.  When a user asks a question, embed the question as well.
    4.  Calculate the cosine similarity (or other distance metric) between the question embedding and all chunk embeddings.
    5.  Retrieve the chunks with the highest similarity as context.
    This approach is much more robust for finding semantically relevant information, even if the exact keywords don't match. It typically requires libraries like `numpy` or `scikit-learn` for vector operations, or a vector database for larger scale.

## Module 3.4: Testing, Evaluating, and Iterating on Your Agent

Building an agent is an iterative process. Your first version is unlikely to be perfect.

**Strategies for Testing:**

*   **Define Test Cases:** Create a list of questions you expect your agent to answer correctly, questions it should identify as out-of-scope, and edge cases.
    *   For our Xylar agent: questions about flora, fauna, inhabitants, energy, Xylium, Dr. Thorne, moons, temperature. Also, questions completely unrelated to Xylar.
*   **Qualitative Evaluation:** Read the agent's responses. Are they accurate? Fluent? Do they follow the persona? Do they correctly use the retrieved context?
*   **Error Analysis:** When the agent fails, try to understand why. Was the retrieval poor? Was the prompt to the LLM unclear? Did the LLM hallucinate despite the context?

**Debugging Common Issues:**

*   **Incorrect Responses:** Check the retrieved context. If it's bad, improve your retriever. If the context is good but the LLM answer is bad, refine your system prompt or the way you present the context and question to the LLM.
*   **Failure to Use Tools (or RAG context):** Ensure your prompts clearly instruct the agent to use the provided context. For explicit tool use (as in Notebook 2), check the tool descriptions and the model's decision-making process.
*   **Poor Contextual Understanding (in multi-turn):** If you extend to multi-turn RAG, ensure your conversation history management is effective and doesn't exceed token limits.

**Tips for Iterative Improvement:**

*   **Prompt Refinement:** This is often the first place to look. Small changes to the system prompt or the structure of the user prompt (with context) can have a big impact.
*   **Retriever Improvement:** If using RAG, the quality of retrieved context is paramount. Experiment with chunking strategies, different retrieval algorithms (keyword, semantic search), and the number of chunks to retrieve.
*   **Memory Enhancements:** For more complex agents, explore better ways to manage short-term and long-term memory.
*   **Tool Adjustments:** If your agent uses other tools, ensure they are reliable and the agent knows when and how to use them effectively.

Building robust and reliable agents requires continuous testing and refinement.

---

**Congratulations!** You have now designed and built a simple agentic application – a Document Q&A Agent – using OpenAI on Databricks. You've implemented a basic RAG pipeline, allowing your agent to answer questions based on provided information.

Next up: **Notebook 4: Next Steps and Expanding Your Agentic AI Horizons**, where we'll discuss how to take your agent development skills further.

