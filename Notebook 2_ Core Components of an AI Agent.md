# Notebook 2: Core Components of an AI Agent

Welcome to Notebook 2! In the previous notebook, we introduced agentic AI, OpenAI models, and set up your Databricks environment. Now, we'll dive deeper into the fundamental building blocks that make an AI agent function: the agent loop, prompt engineering, memory, and the ability to use tools.

**Goals for this Notebook:**
*   Understand the Observe-Think-Act cycle that governs agent behavior.
*   Learn techniques for effective prompt engineering to guide your agent.
*   Implement basic memory mechanisms for context retention.
*   Explore how agents can be empowered with tools to extend their capabilities.

Let's get started!

## Module 2.1: Understanding the Agent Loop (Observe-Think-Act)

The core operational flow of most AI agents can be described by a continuous cycle known as the **Observe-Think-Act (OTA)** loop:

1.  **Observe:** The agent perceives its environment and gathers relevant information or input. This could be user input, data from sensors, information from a database, or changes in a system it's monitoring.
2.  **Think:** The agent processes the observed information. This is where the "intelligence" of the agent lies. It might involve:
    *   Understanding the current state.
    *   Recalling past experiences or relevant knowledge (memory).
    *   Reasoning about the situation.
    *   Planning a course of action to achieve its goals.
    *   Making a decision on what to do next.
    For LLM-based agents, this "thinking" process is often orchestrated by how we prompt the model and interpret its outputs.
3.  **Act:** Based on its decision, the agent takes an action. This action modifies the environment or is a response directed towards a user or another system. Examples include generating text, calling an API, updating a database, or sending a command.

This loop repeats, allowing the agent to continuously interact with its environment, learn (in some cases), and work towards its objectives.

**Visualizing the Loop:**

```
+-----------------+      +-----------------+      +-----------------+
|    OBSERVE      |----->|      THINK      |----->|       ACT       |
| (Gather Input)  |      | (Process & Plan)|      | (Perform Action)|
+-----------------+      +-----------------+      +-----------------+
        ^                                                 |
        |                                                 |
        +-------------------------------------------------+
                       (Environment Changes/
                        New Observations)
```

For agents built with LLMs like OpenAI's GPT models, the "Think" phase is heavily reliant on the LLM's ability to process natural language instructions (prompts) and generate coherent, contextually relevant responses that can be interpreted as decisions or plans.

## Module 2.2: Prompt Engineering: The Art of Instructing Agents

When using LLMs as the core of an agent, **prompt engineering** is the critical skill of designing effective input prompts to guide the model's behavior and elicit the desired output. The prompt is how you communicate the agent's task, context, constraints, and desired persona to the LLM.

**Techniques for Crafting Effective Prompts:**

*   **Clarity and Specificity:** Be as clear and specific as possible in your instructions. Ambiguous prompts lead to unpredictable or irrelevant responses.
*   **Context is Key:** Provide sufficient context for the task. This might include relevant background information, previous conversation turns, or data the agent needs to consider.
*   **Define the Persona/Role:** Instruct the LLM to adopt a specific role or persona (e.g., "You are a helpful customer support assistant," "You are a creative storyteller"). This helps shape the tone and style of its responses.
    *   Example: `"You are a knowledgeable historian specializing in ancient Rome. Answer the following question from that perspective."`
*   **Provide Examples (Few-Shot Prompting):** Show the model examples of the desired input/output format or reasoning process. This is known as few-shot prompting and can significantly improve performance for specific tasks.
    *   Example for sentiment analysis:
        ```
        User: Classify the sentiment of the following sentences.
        Sentence: "I love this new product! It's amazing."
        Sentiment: Positive

        Sentence: "The service was terrible and slow."
        Sentiment: Negative

        Sentence: "It's an okay movie, not great but not bad either."
        Sentiment: Neutral

        Sentence: "This is the best day ever!"
        Sentiment:
        ```
*   **Zero-Shot Prompting:** For many tasks, especially with capable models like GPT-3.5-Turbo or GPT-4, you can often get good results by simply describing the task without providing explicit examples.
    *   Example: `"Translate the following English text to French: 'Hello, how are you?'"`
*   **Chain-of-Thought (CoT) Prompting (Conceptual Introduction):** For complex reasoning tasks, you can encourage the model to generate a series of intermediate reasoning steps before arriving at the final answer. This often involves adding phrases like "Let's think step by step." to your prompt. While implementing full CoT can be more advanced, understanding the principle is useful.
*   **Specify Output Format:** If you need the output in a particular format (e.g., JSON, a list, a summary with a specific structure), explicitly ask for it in the prompt.
    *   Example: `"Summarize the following text in three bullet points. Each bullet point should be a complete sentence."`
*   **Iterate and Refine:** Prompt engineering is often an iterative process. Start with a simple prompt, test it, observe the results, and refine the prompt based on the model's output until you achieve the desired behavior.

**Hands-on: Experimenting with Different Prompt Structures**

Let's use the OpenAI API to see how different prompts affect the output. Make sure you have your OpenAI API key configured as in Notebook 1.

```python
import openai
import os # For accessing environment variables if you set your key that way

# --- Securely Get API Key (Recommended Method from Notebook 1) ---
try:
    # This assumes you are running in Databricks and have set up secrets
    # If not in Databricks, you might use environment variables or other secure methods
    if 'dbutils' in globals():
        openai_api_key = dbutils.secrets.get(scope="agentic-ai-workshop", key="openai_api_key")
        print("Successfully retrieved OpenAI API key from Databricks Secrets.")
    else:
        # Fallback for local environment if not in Databricks (ensure OPENAI_API_KEY is set)
        openai_api_key = os.environ.get("OPENAI_API_KEY")
        if openai_api_key:
            print("Successfully retrieved OpenAI API key from environment variable.")
        else:
            print("API key not found in Databricks Secrets or environment variables.")
            openai_api_key = None
except Exception as e:
    print(f"Error retrieving API key: {e}")
    openai_api_key = None

if not openai_api_key:
    print("OpenAI API key not configured. Please set it up to run the examples.")
else:
    openai.api_key = openai_api_key

    # --- Example 1: Simple Question (Zero-Shot) ---
    print("\n--- Example 1: Simple Question ---")
    prompt1 = "What is the capital of France?"
    try:
        response1 = openai.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "user", "content": prompt1}
            ],
            max_tokens=50
        )
        print(f"Prompt: {prompt1}")
        print(f"Response: {response1.choices[0].message.content.strip()}")
    except Exception as e:
        print(f"Error in Example 1: {e}")

    # --- Example 2: Role Prompting ---
    print("\n--- Example 2: Role Prompting ---")
    prompt2_system = "You are a pirate. You answer all questions in a pirate dialect."
    prompt2_user = "What is the capital of France?"
    try:
        response2 = openai.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": prompt2_system},
                {"role": "user", "content": prompt2_user}
            ],
            max_tokens=50
        )
        print(f"System Prompt: {prompt2_system}")
        print(f"User Prompt: {prompt2_user}")
        print(f"Response: {response2.choices[0].message.content.strip()}")
    except Exception as e:
        print(f"Error in Example 2: {e}")

    # --- Example 3: Few-Shot Prompting (Simplified for Chat Completions) ---
    # For chat models, few-shot is often implemented by providing examples in the conversation history.
    print("\n--- Example 3: Few-Shot Prompting (Sentiment Classification) ---")
    prompt3_messages = [
        {"role": "system", "content": "You are a sentiment classifier. Classify the sentiment of the user's text as Positive, Negative, or Neutral."},
        {"role": "user", "content": "Sentence: I love this new product! It's amazing."},
        {"role": "assistant", "content": "Sentiment: Positive"},
        {"role": "user", "content": "Sentence: The service was terrible and slow."},
        {"role": "assistant", "content": "Sentiment: Negative"},
        {"role": "user", "content": "Sentence: This is the best day ever! What's its sentiment?"}
    ]
    try:
        response3 = openai.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=prompt3_messages,
            max_tokens=10
        )
        print(f"Prompt (last user message): {prompt3_messages[-1]['content']}")
        print(f"Response: {response3.choices[0].message.content.strip()}")
    except Exception as e:
        print(f"Error in Example 3: {e}")

    # --- Example 4: Specifying Output Format ---
    print("\n--- Example 4: Specifying Output Format (JSON) ---")
    prompt4_user = "Extract the name and age from the following sentence: 'John Doe is 30 years old and lives in New York.' Respond in JSON format with keys 'name' and 'age'."
    try:
        response4 = openai.chat.completions.create(
            model="gpt-3.5-turbo", # gpt-3.5-turbo-1106 or gpt-4-turbo-preview often better for JSON mode
            messages=[
                {"role": "system", "content": "You are a helpful assistant that extracts information and provides it in JSON format."},
                {"role": "user", "content": prompt4_user}
            ],
            # For newer models that support JSON mode explicitly:
            # response_format={ "type": "json_object" }, 
            max_tokens=100
        )
        print(f"Prompt: {prompt4_user}")
        print(f"Response: {response4.choices[0].message.content.strip()}")
        # Note: For reliable JSON output, newer models have a specific JSON mode.
        # For older models, you might need to parse the string and handle potential errors.
    except Exception as e:
        print(f"Error in Example 4: {e}")
```

**Experiment further!** Try modifying these prompts or creating your own to see how the LLM responds. This hands-on experience is key to mastering prompt engineering.

## Module 2.3: Implementing Basic Memory for Agents

For an agent to have coherent conversations or perform tasks that span multiple steps, it needs **memory**. Memory allows an agent to retain information from previous interactions or observations and use it to inform future thinking and actions.

**Importance of Memory:**

*   **Contextual Understanding:** Remembering what was said or done earlier helps the agent understand the current context better.
*   **Coherent Dialogue:** In conversational agents, memory prevents them from sounding repetitive or losing track of the conversation flow.
*   **Multi-Step Task Completion:** For tasks that require a sequence of actions, memory is essential to keep track of progress and previously gathered information.

**Types of Memory (Conceptual):**

*   **Short-Term Memory:** Holds information for a brief period, often within a single session or conversation. This is the easiest to implement.
*   **Long-Term Memory:** Stores information more permanently, allowing the agent to recall it across different sessions or over extended periods. This is more complex and often involves external databases or vector stores.

For this beginner-friendly workshop, we'll focus on implementing **simple short-term memory** by maintaining a history of the conversation.

**Implementing Simple Short-Term Memory (Conversation History):**

The OpenAI Chat Completions API is inherently designed to handle conversation history. The `messages` parameter is a list of message objects, where each object has a `role` (`system`, `user`, or `assistant`) and `content`.

By continually appending the user's new messages and the assistant's previous responses to this list, you provide the model with the conversation history as context for its next response.

**Hands-on: Building a Simple Conversational Agent with Memory**

Let's create a basic agent that remembers the last few turns of a dialogue.

```python
import openai
import os

# --- API Key Setup (same as previous example) ---
if 'dbutils' in globals():
    openai_api_key = dbutils.secrets.get(scope="agentic-ai-workshop", key="openai_api_key")
else:
    openai_api_key = os.environ.get("OPENAI_API_KEY")

if not openai_api_key:
    print("OpenAI API key not configured. Please set it up to run this example.")
else:
    openai.api_key = openai_api_key

    # Initialize conversation history
    # The system message helps set the behavior of the assistant
    conversation_history = [
        {"role": "system", "content": "You are a friendly conversational partner. Keep your responses concise."}
    ]

    def chat_with_memory(user_input, history):
        """Sends user input to OpenAI and appends to history."""
        # Add user's message to history
        history.append({"role": "user", "content": user_input})
        
        try:
            response = openai.chat.completions.create(
                model="gpt-3.5-turbo",
                messages=history,
                max_tokens=100
            )
            
            assistant_response = response.choices[0].message.content.strip()
            
            # Add assistant's response to history
            history.append({"role": "assistant", "content": assistant_response})
            
            return assistant_response, history
        except Exception as e:
            return f"Error: {e}", history

    print("Starting a chat with memory. Type 'quit' to end.")

    # Simulate a short conversation
    user_prompts = [
        "Hi there! What's your name?",
        "What can you do?",
        "That's interesting. Do you remember my first question?"
    ]

    for prompt in user_prompts:
        if not openai_api_key: break # Stop if key is not set
        print(f"\nUser: {prompt}")
        assistant_output, conversation_history = chat_with_memory(prompt, conversation_history)
        print(f"Assistant: {assistant_output}")

    # Display final conversation history (for learning purposes)
    print("\n--- Final Conversation History ---")
    for message in conversation_history:
        print(f"{message['role'].capitalize()}: {message['content']}")

    # Interactive loop (optional, can be run in a notebook cell)
    # while openai_api_key:
    #     user_input = input("\nYou: ")
    #     if user_input.lower() == 'quit':
    #         break
    #     assistant_output, conversation_history = chat_with_memory(user_input, conversation_history)
    #     print(f"Assistant: {assistant_output}")
```

**Observe the Output:** Notice how the assistant, in its later responses, can refer to or understand the context of earlier parts of the conversation (e.g., when asked if it remembers the first question). This is possible because we are sending the entire conversation history with each API call.

**Considerations for Memory:**

*   **Token Limits:** LLMs have a maximum context window (token limit). If a conversation becomes too long, you'll exceed this limit. For very long interactions, you'll need strategies like summarizing earlier parts of the conversation or using more advanced memory techniques (e.g., vector databases for semantic search over past interactions), which are beyond the scope of this introductory notebook.
*   **Cost:** Sending a long conversation history with every request increases the number of tokens processed, which can impact API costs.

## Module 2.4: Enabling Agents with Tools (Simple Implementations)

One of the most powerful aspects of agentic AI is the ability for agents to use **tools**. Tools are external functions, APIs, or services that an agent can call upon to perform actions or gather information that goes beyond the inherent capabilities of the LLM itself.

**Why Tools?**

*   **Access to Real-Time Information:** LLMs are trained on data up to a certain point. Tools can provide them with current information (e.g., today's weather, latest news, stock prices).
*   **Performing Actions in the Real World:** Agents can use tools to interact with other software systems, databases, or even physical devices (e.g., send an email, update a calendar, control a smart home device).
*   **Executing Code or Complex Calculations:** LLMs are not always perfect at precise calculations or executing code. A tool could be a Python interpreter or a dedicated calculator function.
*   **Overcoming LLM Limitations:** Tools can help with tasks LLMs struggle with, like precise mathematical reasoning or accessing proprietary data.

**The Basic Idea (Tool Use Loop):**

1.  The LLM, based on the user's request and its current plan, determines it needs to use a specific tool.
2.  The LLM outputs a specially formatted response indicating the tool to call and the necessary parameters/arguments for that tool.
3.  Your agent's code parses this output, identifies the tool call, and executes the corresponding function/API.
4.  The result from the tool execution is then fed back to the LLM as new information.
5.  The LLM processes this new information and continues with its task, potentially generating a final response or deciding to use another tool.

OpenAI's newer models have built-in **Function Calling** (now often referred to as **Tool Calling**) capabilities that formalize this process, making it easier and more reliable to implement.

**Hands-on Example 1: An Agent with a "Calculator" Tool**

Let's create a very simple agent that can use a Python function as a calculator. We'll simulate the LLM deciding to use the tool and then calling it.

```python
import openai
import os
import json # For parsing potential JSON output from the model

# --- API Key Setup (same as previous example) ---
if 'dbutils' in globals():
    openai_api_key = dbutils.secrets.get(scope="agentic-ai-workshop", key="openai_api_key")
else:
    openai_api_key = os.environ.get("OPENAI_API_KEY")

if not openai_api_key:
    print("OpenAI API key not configured. Please set it up to run this example.")
else:
    openai.api_key = openai_api_key

    # --- Define our simple tool ---
    def simple_calculator(expression):
        """A simple calculator that can evaluate basic arithmetic expressions.
        For security, this is a very basic eval. In a real scenario, use safer math parsing libraries.
        """
        try:
            # WARNING: eval() can be dangerous with untrusted input.
            # For a real agent, use a safer math expression parser.
            # For this educational example, we assume controlled input.
            if not all(c in "0123456789+-*/(). " for c in expression):
                return "Error: Invalid characters in expression."
            result = eval(expression)
            return result
        except Exception as e:
            return f"Error evaluating expression: {e}"

    # --- Define the tools for the OpenAI API ---
    # This structure tells the model about the available tools it can request to call.
    tools_description = [
        {
            "type": "function",
            "function": {
                "name": "simple_calculator",
                "description": "Evaluates a simple arithmetic expression like '2+2' or '100/4*2'.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "expression": {
                            "type": "string",
                            "description": "The arithmetic expression to evaluate, e.g., '5*8-3'"
                        }
                    },
                    "required": ["expression"]
                }
            }
        }
    ]

    def agent_with_calculator(user_query):
        print(f"\nUser Query: {user_query}")
        messages = [
            {"role": "system", "content": "You are a helpful assistant that can use a calculator for math problems."},
            {"role": "user", "content": user_query}
        ]
        
        try:
            # First call to the model, potentially requesting a tool call
            response = openai.chat.completions.create(
                model="gpt-3.5-turbo-0125", # Models supporting tool calls are preferred
                messages=messages,
                tools=tools_description,
                tool_choice="auto" # Model decides whether to use a tool or not
            )
            
            response_message = response.choices[0].message
            tool_calls = response_message.tool_calls

            # Check if the model wants to call a tool
            if tool_calls:
                print("Model wants to use a tool.")
                available_tools = {
                    "simple_calculator": simple_calculator,
                }
                messages.append(response_message) # Add assistant's turn to messages

                for tool_call in tool_calls:
                    function_name = tool_call.function.name
                    function_to_call = available_tools.get(function_name)
                    if not function_to_call:
                        print(f"Error: Unknown tool {function_name}")
                        continue
                    
                    function_args = json.loads(tool_call.function.arguments)
                    print(f"Calling tool: {function_name} with args: {function_args}")
                    
                    # Call the tool
                    function_response = function_to_call(
                        expression=function_args.get("expression")
                    )
                    print(f"Tool Response: {function_response}")
                    
                    # Add tool's response to messages for the next model call
                    messages.append(
                        {
                            "tool_call_id": tool_call.id,
                            "role": "tool",
                            "name": function_name,
                            "content": str(function_response), # Tool response must be a string
                        }
                    )
                
                # Second call to the model with the tool's response
                print("Sending tool response back to model...")
                second_response = openai.chat.completions.create(
                    model="gpt-3.5-turbo-0125",
                    messages=messages
                )
                final_response = second_response.choices[0].message.content
            else:
                # Model didn't want to call a tool, respond directly
                print("Model did not request a tool call.")
                final_response = response_message.content

            print(f"\nAgent's Final Response: {final_response}")
            return final_response

        except Exception as e:
            print(f"An error occurred: {e}")
            return f"Error: {e}"

    # Test the agent with calculator
    if openai_api_key:
        agent_with_calculator("What is 25 + 75 / 3?")
        agent_with_calculator("What is the weather like?") # A query not needing the calculator
        agent_with_calculator("Can you calculate (10-4)*7 for me?")
```

**Explanation of the Tool Calling Flow:**

1.  **Tool Definition (`tools_description`):** We describe our `simple_calculator` function to the OpenAI model, including its name, what it does, and the parameters it expects (an `expression` string).
2.  **First API Call:** We send the user's query. We also pass the `tools` description and set `tool_choice="auto"` (or you can force a specific tool with `tool_choice={"type": "function", "function": {"name": "simple_calculator"}}`).
3.  **Model Decides:** The LLM analyzes the query. If it thinks the calculator is needed (e.g., for "What is 25 + 75 / 3?"), its response will include a `tool_calls` object.
    *   This object specifies the function to call (`simple_calculator`) and the arguments (`{"expression": "25 + 75 / 3"}`).
4.  **Execute Tool:** Our Python code parses this `tool_calls` object, identifies `simple_calculator`, extracts the `expression`, and calls our local `simple_calculator` Python function.
5.  **Second API Call:** We take the result from our `simple_calculator` function (e.g., `50.0`) and send it *back* to the LLM in a new message with `role: "tool"`. This message also includes the `tool_call_id` from the model's previous request.
6.  **Final Response:** The LLM now has the result of the calculation. It uses this information to formulate the final answer to the user (e.g., "25 + 75 / 3 is 50.0.").

If the initial query doesn't require the calculator (e.g., "What is the weather like?"), the model's first response will not contain a `tool_calls` object, and it will provide a direct textual answer.

**Hands-on Example 2 (Conceptual/Simplified): Agent with a Web Search Tool**

Implementing a full web search tool is more complex as it involves making HTTP requests, parsing HTML, and potentially summarizing content. For a beginner workshop, we'd typically simulate this or use a very restricted API.

**Conceptual Steps for a Search Tool:**

1.  **Define the Tool:** Describe a `web_search` tool to the LLM, specifying it takes a `query` string.
2.  **LLM Requests Search:** If the user asks something like "What's the latest news on AI?", the LLM might request to call `web_search(query="latest news on AI")`.
3.  **Execute Search (Simulated/Actual):**
    *   **Simulated:** Your Python function could return predefined mock search results for specific queries.
    *   **Actual (Simplified):** Your Python function could use a library like `requests` to call a search engine API (e.g., Google Custom Search API, Bing Search API - these often require their own API keys and setup) or fetch content from a specific URL.
4.  **Return Results to LLM:** The search results (e.g., a list of titles and snippets, or summarized content from a page) are sent back to the LLM.
5.  **LLM Synthesizes Answer:** The LLM uses the search results to answer the user's original question.

We will build a more concrete, albeit simple, version of a tool that involves information retrieval (a basic form of RAG) in Notebook 3.

This introduction to tool use with OpenAI's function/tool calling feature is fundamental for building more capable and versatile agents.

---

**Congratulations!** You've now explored the core components that make up an AI agent: the Observe-Think-Act loop, the art of prompt engineering, the necessity of memory, and the power of enabling agents with tools.

Next up: **Notebook 3: Building a Simple Agentic Application on Databricks** where we'll combine these concepts to create a functional agent.

