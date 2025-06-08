---
title: "Building AI Agents with Strands: Part 2 - Tool Integration"
date: 2025-05-21T11:30:03+00:00
weight: 2
# aliases: ["/first"]
tags: ["strands", "agents", "tutorial", "python"]
author: "Dennis Traub"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Learn how to connect your AI agent to the real world using built-in and custom tools."
canonicalURL: "https://community.aws/content/2xP8NYlAQUN0feRpeq9xqhj0i7u/building-ai-agents-with-strands-part-2-tool-integration"
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
    image: "images/stra_tut_part2_header.webp" # image path/url
    alt: "header image" # alt text
    relative: false  # when using page bundles set this to true
    hidden: true # only hide on current single page

---

Last Modified May 28, 2025

Welcome to the second part of our series on building AI agents with Strands! In [Part 1](/posts/strands_tutorial_part1), we created a basic subject expert agent for our Integrated Learning Lab. Now, we'll take it to the next level by integrating tools to extend our agent's capabilities.

Tools are what transform a basic conversational agent into a truly useful assistant that can interact with the world. In this tutorial, we'll explore both built-in tools provided by the Strands SDK, and learn how to create our own custom tools.

## What You'll Learn

- How to integrate built-in tools from the `strands-agents-tools` package
- How to create custom tools using the `@tool` decorator
- How to implement file operations for learning resources
- How to build a custom glossary tool for our agent

## Prerequsites
- Completed [Part 1: Creating Your First Agent](/posts/strands_tutorial_part1)
- The Strands Agents SDK installed (`pip install strands-agents strands-agents-tools`)
- Basic Python knowledge

## Understanding Tools in Strands

Tools are functions that agents can use to perform actions beyond simply generating text. They allow agents to:

- Access and manipulate external data (files, databases)
- Perform calculations and specialized processing
- Create, modify, and manage content
- Connect to any external service via APIs

The Strands SDK makes tool integration straightforward while maintaining a security-focused approach.

Let's start by enhancing our subject expert with some built-in tools from the `strands-agents-tools` package:

```python
import logging
from strands import Agent
from strands_tools import current_time, http_request

subject_expert = Agent(
    system_prompt="""You are a Computer Science Subject Expert specializing
    in explaining technical concepts clearly and concisely. Your expertise
    covers programming languages, data structures, algorithms, computer
    architecture, and software engineering principles.

    You have access to tools that help you provide more accurate and timely
    information. Use these tools when appropriate to enhance your explanations.
    
    When explaining concepts:
    1. Start with a clear, concise definition
    2. Provide relevant examples to illustrate the concept
    3. Explain practical applications where applicable
    4. Use tools when additional information would be valuable
    5. Cite sources when you reference external information
    """,
    tools=[current_time, http_request]
)

# Test the agent with a query that might benefit from tools
query = """
Answer the following questions:
1. What is the current time in UTC?
2. Based on Wikipedia, which CS concept can be traced back to Paul Bachmann?
"""

response = subject_expert(query)
```

When you run this code, you'll get a response similar to this:

```markdown
I'll answer both of your questions using the appropriate tools for accurate
information.

### Question 1: What is the current time in UTC?

Let me get the current UTC time for you:
Tool #1: current_time

The current time in UTC is 2025-05-21T14:21:29.562062+00:00, which translates
to May 21, 2025, at 14:21:29 UTC.

### Question 2: Based on Wikipedia, which CS concept can be traced back to
Paul Bachmann?

Let me search for this information on Wikipedia:
Tool #2: http_request

Based on the Wikipedia article I've accessed, one of the key computer science
concepts that can be traced back to Paul Bachmann is **Big O notation**.
```

This demonstrates a key strength of Strands: **tools are selected and invoked automatically based on the context of the conversation**. Without any explicit commands, the agent:

1. Recognized the first question needed the current time and used the current\_time tool
2. Understood the second question required external information and used the http\_request tool
3. Integrated the tool results seamlessly into a comprehensive response

## Creating a Custom Tool

While built-in tools are useful, the real power comes from creating custom tools tailored to your specific needs. For our Integrated Learning Lab, let's build a tool that manages a glossary of computer science terms:

```python
import os
import json
from typing import Optional
from strands import Agent, tool
from strands_tools import calculator

# Define a custom tool to manage CS terminology
@tool
def cs_glossary(
    action: str, term: Optional[str] = None, definition: Optional[str] = None
) -> str:
    """
    Manage a glossary of computer science terms.

    Args:
        action: The action to perform: 'lookup', 'add', 'update', or 'list'.
        term: The term to look up, add, or update (not needed for 'list').
        definition: The definition to add or update (not needed for 'lookup' and 'list').
    """
    glossary_file = "cs_glossary.json"

    def load_glossary():
        if not os.path.exists(glossary_file):
            with open(glossary_file, "w") as f:
                json.dump({}, f)
        with open(glossary_file, "r") as f:
            return json.load(f)

    def save_glossary(glossary):
        with open(glossary_file, "w") as f:
            json.dump(glossary, f)

    glossary = load_glossary()

    if action == "lookup":
        if not term:
            return "Error: Term is required for lookup"
        return glossary.get(term, f"Term '{term}' not found in the glossary")

    elif action == "add":
        if not term or not definition:
            return "Error: Both term and definition are required"
        if term in glossary:
            return f"Error: '{term}' already exists. Use 'update' instead."
        glossary[term] = definition
        save_glossary(glossary)
        return f"Added '{term}' to the glossary"

    elif action == "update":
        if not term or not definition:
            return "Error: Both term and definition are required"
        if term not in glossary:
            return f"Error: Term '{term}' not found. Use 'add' instead."
        glossary[term] = definition
        save_glossary(glossary)
        return f"Updated definition for '{term}'"

    elif action == "list":
        if not glossary:
            return "The glossary is empty"
        return "\n".join(f"- {t}" for t in sorted(glossary))

    return f"Error: Unknown action '{action}'. Use 'lookup', 'add', 'update', or 'list'."
```

A few important notes about this implementation:

1. We use the `@tool` decorator to transform a regular Python function into a tool
2. The function's docstring and type hints are used to generate a tool specification
3. We include input validation as a precaution
4. The tool maintains state between invocations by reading from and writing to a file

This approach follows secure tool design principles, which are essential when building AI systems that interact with external resources.

## Creating a Subject Expert with Advanced Tools

Now, let's put it all together by creating a subject expert agent that uses both built-in and custom tools:

```python
# Glossary @tool implementation as shown above
# ...

# Create an enhanced subject expert agent with multiple tools
subject_expert = Agent(
    system_prompt="""You are a Computer Science Subject Expert specializing
    in explaining technical concepts clearly and concisely. Your expertise
    covers programming languages, data structures, algorithms, computer
    architecture, and software engineering principles.
    
    You can manage a glossary of computer science terms with the cs_glossary tool.
    You can perform calculations with the calculator tool.
    You can read and write files with the file_read and file_write tools.
    
    When explaining concepts:
    1. Start with a clear, concise definition
    2. Check the glossary for any related terms
    3. Add important terms to the glossary if they're not already there
    4. Provide relevant examples to illustrate the concept
    5. Use calculations when appropriate to demonstrate complexity or performance
    
    Your goal is to help learners build a solid understanding of computer
    science fundamentals.
    """,
    tools=[calculator, cs_glossary]
)
```

## Interactive Testing Session

Let's create an interactive session to test our enhanced agent:

```python
def interactive_session():
    print("Computer Science Subject Expert (type 'exit' to quit)")
    print("---------------------------------------------------")
    print("Suggested commands to try:")
    print("- 'Add recursion to the glossary'")
    print("- 'What terms are in the glossary?'")
    print("- 'Explain the time complexity of binary search'")
    
    while True:
        user_input = input("\nYour question: ")
        
        if user_input.lower() in ["exit", "quit", "bye"]:
            print("Goodbye!")
            break
        
        print("\nThinking...\n")
        response = subject_expert(user_input)
        print(response)

if __name__ == "__main__":
    interactive_session()
```

Save this to a file (e.g., `subject_expert_with_tools.py`) and run it. You'll have an interactive session with your tool-enhanced agent that looks similar to this:

```markdown
Computer Science Subject Expert (type 'exit' to quit)
---------------------------------------------------
Suggested commands to try:
- 'Add recursion to the glossary'
- 'What terms are in the glossary?'
- 'Explain the time complexity of binary search'

Your question: _
```

## Direct Tool Invocation

While the agent will automatically invoke tools based on user queries, sometimes you may want to call tools directly from your code:


```python
# Look up a term directly
definition = subject_expert.tool.cs_glossary(action="lookup", term="recursion")
print(f"Definition: {definition}")

# Perform a calculation directly
result = subject_expert.tool.calculator(expression="2^10")
print(f"Calculation result: {result}")
```

This gives you programmatic control when needed, while still benefiting from the agent's context.

## Security Considerations for Tool Design

When creating tools, always follow these security principles:

1. **Validate inputs**: Check that inputs match expected types and formats
2. **Limit permissions**: Tools should have minimal necessary access
3. **Sanitize outputs**: Ensure tool outputs don't contain harmful content
4. **Error handling**: Gracefully handle edge cases and errors
5. **Audit logging**: Log sensitive operations for review

Our `cs_glossary` tool implements these principles by validating input types, using a specific file for storage, and providing clear error messages.

## What We've Learned

In this tutorial, we've:

- ✅ Added built-in tools to our subject expert agent
- ✅ Created a custom glossary tool with the `@tool` decorator
- ✅ Combined multiple tools for enhanced capabilities
- ✅ Created an interactive testing environment

Our agent can now maintain a growing knowledge base of computer science terminology, perform calculations, and manage files - significantly expanding its capabilities beyond simple conversation.

## Next Steps and Resources

In the the next lesson, we'll explore the **Model Context Protocol (MCP)** to connect our agent with external specialized capabilities. This will allow us to tap into tools developed by others without having to implement everything ourselves.

Ready? Let's continue with [Part 3: MCP Integration](/posts/strands_tutorial_part3)

### Want to learn more about the Strands Agents SDK?

Here are some resources to help you deepen your understanding:

- 📚 [**Strands Agents Documentation**](https://strandsagents.com/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el) - Comprehensive guides and API references
- 💻 [**GitHub Repository**](https://github.com/strands-agents/sdk-python?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el) - Source code and examples - and if you like it, consider giving it a ⭐
- 🧩 [**Example Gallery**](https://strandsagents.com/latest/examples/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el) - Explore sample implementations for various use cases

What kind of tools would you like to build? Share your ideas in the comments below!

___

💡 _Troubleshooting tip: If tools aren't working as expected, check that they have proper docstrings and type hints. The Strands SDK uses these to generate the tool specifications that the agent uses to understand when and how to invoke tools._  
