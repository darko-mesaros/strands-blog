---
title: "Building AI Agents with Strands: Part 3 - MCP Integration"
date: 2025-05-22T11:30:03+00:00
weight: 3
# aliases: ["/first"]
tags: ["strands", "agents", "tutorial", "python", "mcp"]
author: "Dennis Traub"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Learn how to build and integrate MCP servers with the Strands Agents SDK. Connect your AI agents to external tools and services using the Model Context Protocol."
canonicalURL: "https://community.aws/content/2xSxbxfYzWnaE7eBfvFsKQ62hcv/building-ai-agents-with-strands-part-3-mcp-integration"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "/images/stra_tut_part3_header.webp" # image path/url
    alt: "header image" # alt text
    relative: false  # when using page bundles set this to true
    hidden: true # only hide on current single page

---

Last Modified May 27, 2025

Welcome to the third part of our series on building AI agents with Strands!

In [Part 1: Creating Your First Agent](/posts/strands_tutorial_part1), we created a simple agent that acts as a computer science expert, and in [Part 2: Tool Integration](/posts/strands_tutorial_part2), we enhanced it with custom tools, like a glossary of terms and the ability to directly access the web.

Now, we'll integrate the Model Context Protocol (MCP) to connect our agent with external specialized services. You'll learn how to expand your agent's capabilities by connecting to any MCP server with just a few lines of code.

**Our use case**: Connect your agent to a specialized quiz service - perhaps a platform provided by a university, or by a commercial vendor. For this tutorial, we'll build a simple quiz service to show the integration patterns, but in practice you'll often connect to an existing service by a third-party provider.

## Prerequisites

- Completed [Part 2: Tool Integration](/posts/strands_tutorial_part2)
- Strands Agents SDK and tools installed (`pip install strands-agents strands-agents-tools`)

## What is the Model Context Protocol (MCP)?

Before we dive into the code, let's briefly talk about MCP:

The [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction "https://modelcontextprotocol.io/introduction") is an open protocol standardizing how AI agents connect to external services, like databases, APIs, legacy systems, or third-party tools. Instead of building custom integrations for each service, MCP provides one standard interface for all external connections - somewhat like REST, but for AI agents.

Manual MCP implementation involves a lot of work: managing handshakes, connection state, message parsing, schema validation, etc.

With Strands, on the other hand, it's really just a few lines of code:

```python
mcp_client = MCPClient(lambda: streamablehttp_client("http://example-service.com/mcp"))
with mcp_client:
    tools = mcp_client.list_tools_sync()
    agent = Agent(tools=tools)
```

The Strands SDK handles all the protocol complexity, letting you focus on agent functionality rather than integration details.

## Building Our Quiz MCP Server

To demonstrate MCP integration, we'll create a simple quiz server in a new file, called `quiz_mcp_server.py`:

```python
# Strands already includes MCP, no additional install required
from mcp.server import FastMCP

# Create an MCP server
mcp = FastMCP(
    name="Computer Science Quiz Service",
    host="0.0.0.0",
    port=8080
)

# Quiz database
QUIZ_CATALOG = {
    "python_basics": {
        "title": "Python Programming Fundamentals",
        "questions": [
            {
                "question": "What keyword is used to define a function in Python?",
                "options": ["func", "def", "function", "define"],
                "correct_answer": "def"
            },
            {
                "question": "Which of these creates a list in Python?",
                "options": ["(1, 2, 3)", "{1, 2, 3}", "[1, 2, 3]", "<1, 2, 3>"],
                "correct_answer": "[1, 2, 3]"
            }
        ]
    },
    "data_structures": {
        "title": "Data Structures Essentials",
        "questions": [
            {
                "question": "What is the time complexity of accessing an element in an array by index?",
                "options": ["O(n)", "O(log n)", "O(1)", "O(n²)"],
                "correct_answer": "O(1)"
            }
        ]
    }
}

@mcp.tool()
def list_quiz_topics() -> dict:
    """List all available quiz topics."""
    topics = {}
    for topic_id, quiz_data in QUIZ_CATALOG.items():
        topics[topic_id] = {
            "title": quiz_data["title"],
            "question_count": len(quiz_data["questions"])
        }
    return {"available_topics": topics}

@mcp.tool()
def get_quiz_for_topic(topic: str) -> dict:
    """Retrieve a quiz for a specific topic."""
    if topic.lower() not in QUIZ_CATALOG:
        return {
            "error": f"Topic '{topic}' not found",
            "available_topics": list(QUIZ_CATALOG.keys())
        }
    
    return QUIZ_CATALOG[topic.lower()]

# Start the MCP server
if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```
### Running the MCP Server

Once you've create `quiz_mcp_server.py` with the code above, start it in its own terminal:

```bash
# Activate your virtual environment
source .venv/bin/activate  # On macOS/Linux
# or
.venv\Scripts\activate     # On Windows

# Start the quiz MCP server
python quiz_mcp_server.py
```

Now you should see something similar to this:

```
INFO:     Started server process
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8080 (Press CTRL+C to quit)
```

**Important**: Keep this terminal running! The MCP server needs to stay active for your agent to connect to it.

## Connecting to the MCP Server

Now let's integrate our subject expert agent with the quiz service. Create `subject_expert_with_mcp.py`:

```python
from strands import Agent
from strands.tools.mcp import MCPClient
from mcp.client.streamable_http import streamablehttp_client

def main():
    # Connect to the quiz MCP server
    print("\nConnecting to MCP Server...")
    mcp_quiz_server = MCPClient(lambda: streamablehttp_client("http://localhost:8080/mcp"))

    try:
        with mcp_quiz_server:

            # Create the subject expert agent with a system prompt
            subject_expert = Agent(
                system_prompt="""You are a Computer Science Subject Expert with access to 
                an external quiz service. You can list available quiz topics, retrieve 
                quizzes for students, ask the user to take a quiz, and check their answers.

                When a student requests a quiz:
                1. Show available topics if they ask what's available
                2. Retrieve the specific quiz they want
                3. Present questions clearly, one at a time, with numbered options
                5. After they have provided all answers, check their responses against the
                   correct answers
                6. Once done with the quiz, give encouraging feedback and explanations

                Rules:
                - You must use the tools provided to you by the MCP server.
                - You must NOT make up your own quiz topics or questions.
                - The quiz data includes correct answers, so you can grade responses yourself.
                """
            )

            # List the tools available on the MCP server...
            mcp_tools = mcp_quiz_server.list_tools_sync()
            print(f"Available tools: {[tool.tool_name for tool in mcp_tools]}")

            # ... and add them to the agent
            subject_expert.tool_registry.process_tools(mcp_tools)

            # Start an interactive learning session
            print("\n👨‍💻 CS Subject Expert with MCP Integration")
            print("=" * 50)
            print("\n📋 Try: 'What quiz topics are available?' or 'Give me a Python quiz'")

            while True:
                user_input = input("\n🎯 Your request: ")
                
                if user_input.lower() in ["exit", "quit", "bye"]:
                    print("👋 Happy learning!")
                    break
                
                print("\n🤔 Processing...\n")
                subject_expert(user_input)
               
    except Exception as e:
        print(f"❌ Connection failed: {e}")
        print("💡 Make sure the quiz service is running: python quiz_mcp_server.py")

if __name__ == "__main__":
    main()
```
### Connecting Your Agent

Open a **second terminal** and activate your virtual environment there too:

```bash
# In a NEW terminal window/tab
source .venv/bin/activate  # On macOS/Linux
# or
.venv\Scripts\activate     # On Windows

# Run your agent (make sure the server is still running in the other terminal)
python subject_expert_with_mcp.py
```

You should see:

```markdown
Connecting to MCP Server...
Available tools: ['list_quiz_topics', 'get_quiz_for_topic']

👨‍💻 CS Subject Expert with MCP Integration
==================================================

📋 Try: 'What quiz topics are available?' or 'Give me a Python quiz'
```

## Testing the Integration

Now you can interact with your agent, which seamlessly combines local tools with external services:

**Discovering available content:**

```markdown
🎯 Your request: What quiz topics are available?

🤔 Processing...

I'd be happy to help you find out what quiz topics are available!
Let me check that for you.

Tool #1: list_quiz_topics

Here are the available quiz topics:

1. **Python Programming Fundamentals** (2 questions)
2. **Data Structures Essentials** (1 question)

Would you like to take a quiz on one of these topics?
```

**Taking a quiz:**

```markdown
🎯 Your request: I'd like to try the Python basics quiz

🤔 Processing...

Great choice! Let me retrieve the Python Programming Fundamentals quiz for you.
Tool #2: get_quiz_for_topic

# Python Programming Fundamentals Quiz

I'll present one question at a time. Please provide your answer by selecting
one of the options.

## Question 1:
What keyword is used to define a function in Python?
1. func
2. def
3. function
4. define

Please answer with the number (1-4).
```

**Getting feedback:**

Once you've answered all questions, the agent will show you the results.

```markdown
🙌 Congratulations! You've completed the Python Programming Fundamentals quiz
with a perfect score of 2/2! 

Would you like to try another quiz topic, or do you have any questions about
the material we covered?
```
Now try experimenting with correct and incorrect answers. You could also ask the agent for more detailed explanations to help you learn the concepts.

## Direct Tool Calling

While the agent automatically selects tools based on conversation, you can also call MCP tools directly:

```python
# Example of direct tool usage
with mcp_quiz_server:
    mcp_tools = mcp_quiz_server.list_tools_sync()
    agent = Agent(tools=mcp_tools)
    
    # Direct tool call via MCP
    topics = agent.tool.list_quiz_topics()
    print(f"Available topics:\n{topics}")
```

This gives you direct control when needed, while still benefiting from the agent's natural language interface.

## Understanding Strands' MCP Integration

This integration demonstrates several key advantages of the MCP approach:

- **Service Abstraction**: Your agent doesn't need to know the internal implementation of the quiz service. It could be a simple JSON file, a complex database, or even an AI-powered agent itself - the MCP interface remains the same.
- **Technology Independence**: The quiz service could be rewritten in Java, hosted anywhere on the internet, or replaced with a completely different provider - your agent code doesn't change.
- **Scalability**: You can easily connect to multiple services, and even mix them with your own custom or built-in tools:
```python
# Connect to external MCP servers
quiz_service = MCPClient(lambda: streamablehttp_client("http://quiz-provider.com/mcp"))
library_service = MCPClient(lambda: streamablehttp_client("http://cs-library.edu/mcp"))

with quiz_service, library_server:
    # Combine all tools - they all work the same way!
    tools = (
        quiz_service.list_tools_sync() +      # Tools from the external quiz server
        library_service.list_tools_sync() +   # Tools from the external library server
        [http_request] +                      # Built-in tools from the Strands SDK
        [cs_glossary, ...]                    # Your custom tools
    )
    
    # Create agent with all tools
    agent = Agent(tools=tools)
```


## Production Considerations

Some important considerations when connecting to real MCP servers in production are...

📊 **Monitoring**: Track service health and performance.

⚠️ **Error Handling**: Implement robust fallbacks for service unavailability.

🔐 **Authentication**: Many commercial MCP servers may require API keys or OAuth.

Here is an example with a custom timeout, an authorization header, and a local fallback in case the MCP server is unavailable:

```python
mcp_client = MCPClient(lambda: streamablehttp_client(
    "https://example.com/mcp",
    timeout=timedelta(seconds=10),
    headers={"Authorization": "Bearer <token>"}, 
))

try:
    tools = mcp_client.list_tools_sync()
except Exception as e:
    print(f"Service unavailable: {e}")
    # Fall back to local tools only
    tools = [cs_glossary]
```

## What We've Learned

In this tutorial, we've:

- ✅ Built a simple MCP server to demonstrate external integration
- ✅ Connected our agent to the MCP server with minimal code
- ✅ Accomplished seamless tool integration through natural language
- ✅ Understood how the Strands Agents SDK abstracts MCP complexity
- ✅ Explored advanced patterns for scaling, security, and error handling

Our subject expert agent can now use external services, opening up endless possibilities to integrate with all kinds of specialized platforms and tools.

## Next Steps & Resources

In Part 4, we'll explore [**Alternative Model Providers**](/posts/strands_tutorial_part4), showing you how to set up local model deployment for development and testing.

### Want to learn more about the Strands Agents SDK?

Here are some resources to help you deepen your understanding:

- 📚 [**Strands Agents Documentation**](https://strandsagents.com/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el) - Comprehensive guides and API references
- 💻 [**GitHub Repository**](https://github.com/strands-agents/sdk-python?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el) - Source code and examples - and if you like it, consider giving it a ⭐
- 🧩 [**Example Gallery**](https://strandsagents.com/latest/examples/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el) - Explore sample implementations for various use cases

What kind of MCP servers would you like to use - or even build yourself? Share your ideas in the comments below!

___

## 💡Troubleshooting Tips

**Connection Issues:**
- Ensure the MCP server is running before starting your agent
- Verify the URL includes the `/mcp` path: `http://localhost:8080/mcp`
- Check firewall settings if running on different machines

**Service Discovery Problems:**

- Restart both server and client if tools aren't discovered
- Check the MCP server terminal for error messages

**Virtual Environment Issues:**

- Make sure both terminals have the virtual environment activated before running the server and client
- If you see import errors, verify that `strands-agents` and `strand-agents-tools` are installed in your active environment with `pip list | grep strands`  
