---
title: "Building AI Agents with Strands: Part 1 - Creating Your First Agent"
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
description: "Create your first AI agent with just a few lines of code using the Strands Agents SDK and Amazon Bedrock."
canonicalURL: "https://community.aws/content/2xP1AQ52ofPdLawEbDI5EEUhmhH/building-ai-agents-with-strands-part-1-creating-your-first-agent"
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
    image: "images/stra_tut_part1_header.webp" # image path/url
    alt: "header image" # alt text
    relative: false  # when using page bundles set this to true
    hidden: true # only hide on current single page

---

In this first tutorial, I'll walk you through creating a simple but functional AI agent using the [Strands Agents SDK](https://strandsagents.com/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el).

We'll set up our development environment, install the necessary packages, and create a subject expert agent - the first component of our Integrated Learning Lab project.

Let's dive in!

## Prerequisites

Before we begin, you'll need:
- [Python 3.10](https://www.python.org/downloads/) or higher installed
- An [AWS account](https://aws.amazon.com/free/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el)
- [Model access enabled](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access-modify.html?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el) for Claude 3.7 in Amazon Bedrock (the default model used by Strands Agents)
- AWS credentials configured with permissions to access [Amazon Bedrock](https://aws.amazon.com/bedrock/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el)

If you're not familiar with setting up AWS credentials, I recommend checking out the [AWS CLI configuration guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el).

## Setting Up Your Environment

First, let's create a virtual environment and install the required packages:
```python
# Create a virtual environment
python -m venv .venv

# Activate the environment
# On Windows
.venv\Scripts\activate
# On macOS/Linux
source .venv/bin/activate

# Install Strands Agents SDK and tools
pip install strands-agents strands-agents-tools
```


The `strands-agents` package provides the core SDK functionality, while `strands-agents-tools` includes a variety of built-in tools we can use to enhance our agents.

## Creating Your First Agent

Now, let's create a simple Subject Expert agent focused on computer science education. Create a new file called `subject_expert.py` with the following code:

```python
from strands import Agent

# Create a basic agent with a specialized system prompt
subject_expert = Agent(
    system_prompt="""You are a Computer Science Subject Expert specializing
    in explaining technical concepts clearly and concisely. Your expertise
    covers programming languages, data structures, algorithms, computer
    architecture, and software engineering principles.
    
    When explaining concepts:
    1. Start with a clear, concise definition
    2. Provide short, but relevant examples to illustrate the concept
    3. Explain practical applications where applicable
    4. Avoid unnecessary jargon, but introduce important terminology
    5. Consider the learner's perspective and make complex topics accessible
    
    Your goal is to help learners build a solid understanding of computer
    science fundamentals.
    """
)

# The response will be automatically printed by the Agent class
response = subject_expert("Explain the concept of recursion in programming.")
```
Save this file and run it:

```
python subject_expert.py

```

And just like that, you have a functional AI agent! Isn't it incredible how little code it took to create something so capable?

_**Cross-Region Inference note:** If you encounter permission errors despite configuring credentials correctly, you may need to enable model access in multiple AWS regions. Strands Agents uses inference profiles that can route requests to the region with the lowest latency. For US users, consider enabling Claude model access in all US regions where it's available (us-east-1, us-east-2, us-west-2), even if you're primarily working in just one region - similarly in the EU. See the_ [_Strands Model Providers documentation_](https://strandsagents.com/latest/user-guide/concepts/model-providers/amazon-bedrock/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el) _for more details on region configuration._

## Understanding What's Happening

Let's break down what's going on in our code:

### The Agent Code

```python
subject_expert = Agent(
    system_prompt="""You are a Computer Science Subject Expert..."""
)
```

This creates an `Agent` instance with a specialized system prompt. The system prompt is crucial as it defines the agent's personality, expertise, and behavior guidelines.

I've found that crafting effective system prompts is something of an art—it's about finding the right balance between specificity and flexibility. For our Subject Expert, I wanted to ensure it provides clear explanations with practical examples.

### Agent Invocation

```python
response = subject_expert("Explain the concept of recursion in programming.")
```

This line:

- Sends our query to the agent
- The agent processes it using the AI model
- The response is automatically printed to the console
- The response is also returned as a string for further use if needed

### The Default Model

By default, Strands Agents uses Amazon Bedrock with Claude 3.7. The SDK automatically handles:

1. Creating a secure connection to Amazon Bedrock
2. Formatting requests and responses for Claude 3.7
3. Managing token limits and other model-specific requirements

This abstraction lets us focus on building agent functionality rather than worrying about API details.

What's even better is that Strands Agents supports multiple model providers beyond Amazon Bedrock. The framework is designed to be provider-agnostic, so you can easily switch between Amazon Bedrock, Anthropic, LiteLLM, Ollama, Llama API, and build your own custom providers for specialized needs

This flexibility means you can start with one provider and easily switch to another later, or even use different providers for different agents in the same application. You can find more details in the [Model Providers documentation](https://strandsagents.com/latest/user-guide/concepts/model-providers/amazon-bedrock/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el).

In future tutorials, we'll explore how to use alternative model providers, but for now, we'll stick with the default Amazon Bedrock integration to keep things simple.

## Enabling Debug Logging

While developing, I often want to see what's happening "under the hood." Strands provides built-in logging to help with this. Let's modify our script to enable debug logging:

```python
import logging
from strands import Agent

# Enable debug logging for Strands
logging.getLogger("strands").setLevel(logging.DEBUG)
logging.basicConfig(
    format="%(levelname)s | %(name)s | %(message)s",
    handlers=[logging.StreamHandler()]
)

# Create the Subject Expert agent
subject_expert = Agent(
    system_prompt="""You are a Computer Science Subject Expert..."""
)

# Ask a question
response = subject_expert("Explain the concept of recursion in programming.")
```

With logging enabled, you'll see detailed information about what's happening behind the scenes:

- Model initialization
- Tool discovery and registration
- Event loop processing
- Request/response flow
- Conversation management
- Any errors or warnings

Here's a snippet of what the logging output may look like:

```log
DEBUG | strands.models.bedrock | config=<{'model_id': 'us.anthropic.claude-3-7-sonnet-20250219-v1:0'}> | initializing
DEBUG | strands.tools.registry | tool_modules=<[]> | discovered
DEBUG | strands.event_loop.streaming | model=<BedrockModel> | streaming messages
DEBUG | strands.types.models | formatting request
DEBUG | strands.types.models | invoking model
DEBUG | strands.types.models | got response from model
```

These logs have been invaluable for troubleshooting issues and understanding the inner workings of the agent as I've been experimenting with it.

## Building an Interactive Session

Let's enhance our script to create an interactive session with our subject expert agent:

```python
import logging
from strands import Agent

# Enable debug logging (optional)
# logging.getLogger("strands").setLevel(logging.DEBUG)
# logging.basicConfig(
#     format="%(levelname)s | %(name)s | %(message)s",
#     handlers=[logging.StreamHandler()]
# )

# Create the Subject Expert agent
subject_expert = Agent(
    system_prompt="""You are a Computer Science Subject Expert specializing
    in explaining technical concepts clearly and concisely. Your expertise
    covers programming languages, data structures, algorithms, computer
    architecture, and software engineering principles.
    
    When explaining concepts:
    1. Start with a clear, concise definition
    2. Provide relevant examples to illustrate the concept
    3. Explain practical applications where applicable
    4. Avoid unnecessary jargon, but introduce important terminology
    5. Consider the learner's perspective and make complex topics accessible
    
    Your goal is to help learners build a solid understanding of computer
    science fundamentals.
    """
)

def interactive_session():
    print("Computer Science Subject Expert Agent (type 'exit' to quit)")
    print("-----------------------------------------------------------")
    
    while True:
        # Get user input
        user_input = input("\nYour question: ")
        
        if user_input.lower() in ["exit", "quit", "bye"]:
            print("Goodbye!")
            break
        
        # Send the input to the agent, which will automatically print the response
        print("\nThinking...\n")
        subject_expert(user_input)

if __name__ == "__main__":
    interactive_session()
```

Run this script to start an interactive session:

```
python subject_expert.py
```

Now you can have a continuous conversation with your subject expert agent, asking various computer science questions.

## What We've Learned So Far

Creating this simple agent has taught us several things about the Strands Agents SDK:

1. The entry barrier is remarkably low—you can have a functional agent in just a few lines of code
2. System prompts are powerful for defining agent behavior and expertise
3. The SDK handles many complexities (authentication, request formatting, etc.) behind the scenes
4. Debug logging provides valuable insights into the agent's operation

While this is just the beginning, it's exciting to see how quickly we can create a functional AI agent. In the next tutorial, we'll explore how to enhance our agent with custom tools to extend its capabilities beyond conversation.

## Next Steps & Resources

In the next lesson, we'll learn how to add tools that allow our agent to:

- Access and manage learning resources
- Look up specific information
- Perform calculations and analysis

These capabilities will significantly enhance our subject expert's ability to deliver an interactive learning experience.
