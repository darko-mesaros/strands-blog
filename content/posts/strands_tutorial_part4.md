---
title: "Building AI Agents with Strands: Part 4 - Alternative Model Providers"
date: 2025-05-27T11:30:03+00:00
weight: 4
# aliases: ["/first"]
tags: ["strands", "agents", "tutorial", "python", "mcp", "ollama", "openai"]
author: "Dennis Traub"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Learn how to use different AI models with the Strands Agents SDK, including models from Amazon Bedrock and OpenAI, as well as local models with Ollama."
canonicalURL: "https://community.aws/content/2xT246v0TpKQhlPjCCBcROjKvtp/building-ai-agents-with-strands-part-4-alternative-model-providers"
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
    image: "images/stra_tut_part4_header.webp" # image path/url
    alt: "header image" # alt text
    relative: false  # when using page bundles set this to true
    hidden: true # only hide on current single page

---
Last Modified May 28, 2025

Welcome to the fourth part of our series on building AI agents with Strands!

In the previous tutorials, we built a computer science subject expert agent ([Part 1](/posts/strands_tutorial_part1)), enhanced it with custom tools ([Part 2](/posts/strands_tutorial_part2)), and connected it to external services using MCP ([Part 3](/posts/strands_tutorial_part3)).

So far, we've been using the default model configuration with Amazon Bedrock. But one of Strands Agents' greatest strengths is its **model provider flexibility**. You can easily switch between different models and providers based on your specific needs, preferences, or performance requirements.

Today, we'll explore this flexibility through hands-on examples, starting simple and building up to more advanced scenarios.

## What You'll Learn

By the end of this tutorial, you'll know how to:

- Switch between different Bedrock models and AWS Regions
- Connect your agent to OpenAI and other providers
- Set up local model development with Ollama
- Understand when to use each approach
- Handle common issues when switching model providers

## Prerequisites

Before we begin, ensure you have:
- Completed [Part 1: Creating Your First Agent](/posts/strands_tutorial_part1)
- The Strands Agents SDK and tools installed (`pip install strands-agents`)

## Understanding Model Provider Flexibility

One of the most powerful aspects of Strands Agents is that your agent code remains almost identical regardless of which model provider you use. Let me show you what I mean.

Here's our basic agent from Part 1:

```python
from strands import Agent

# Default configuration - uses Claude 3.7 Sonnet on Amazon Bedrock
agent = Agent(
    system_prompt="You are a Computer Science Subject Expert..."
)

response = agent("Explain recursion")
```

You can switch to completely different models - and even model providers, by only adding a few lines of code. This flexibility lets you experiment with different models, optimize for cost or performance, and even switch between cloud and local deployment - all without rewriting your agent logic.

## Option 1: Different Bedrock Models and Regions

Let's start with the simplest change: using a different model on Amazon Bedrock. Maybe you want to try Amazon's Nova Pro model, or you need to deploy in a specific AWS Region.

```python
from strands import Agent
from strands.models import BedrockModel

# Configure Amazon Nova Pro in US West region
nova_pro = BedrockModel(
    model_id="us.amazon.nova-pro-v1:0",
    region_name="us-west-2",
    temperature=0.7
)

# Create your agent with the new model
subject_expert = Agent(
    model=nova_pro,
    system_prompt="""You are a Computer Science Subject Expert specializing
    in explaining technical concepts clearly and concisely..."""
)

# Test the new model
response = subject_expert("Explain quantum computing in simple terms")
```

For additional `BedrockModel` configuration parameters, check out the [Strands documentation](https://strandsagents.com/latest/user-guide/concepts/model-providers/amazon-bedrock/#configuration-options?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el).

**Important**: You'll need to [request access](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el) to specific models in your Bedrock console before using them.

## Option 2: Cloud-based Alternatives - OpenAI and Others

Strands supports several cloud model providers beyond Amazon Bedrock. Here's how to OpenAI's GPT:

```python
# First, install the OpenAI provider
# pip install "strands-agents[openai]"

from strands import Agent
from strands.models.openai import OpenAIModel

# Configure OpenAI GPT-4o
openai_model = OpenAIModel(
    client_args={
        "api_key": "your-openai-api-key",  # Replace with your actual API key
    },
    model_id="gpt-4o",
    params={
        "max_tokens": 1000,
        "temperature": 0.7,
    }
)

# Create your agent - same pattern as before
openai_agent = Agent(
    model=openai_model,
    system_prompt="You are a Computer Science Subject Expert...",
)

# Use exactly the same way
response = openai_agent("Explain object-oriented programming")
```

**Other supported providers:**

- [**Anthropic**](https://strandsagents.com/latest/user-guide/concepts/model-providers/anthropic/) - Direct API access to Claude models hosted by Anthropic
- [**Llama API**](https://strandsagents.com/latest/user-guide/concepts/model-providers/llamaapi/) - Access to Llama models on Meta's own API service
- [**LiteLLM**](https://strandsagents.com/latest/user-guide/concepts/model-providers/litellm/) - Aunified interface for multiple providers
    

## Option 3: Local Development with Ollama

Now for our main focus today: running models locally. This is incredibly valuable during development because it gives you:

- **Zero API costs** while you're experimenting
- **Offline capability** - work without internet connection
- **Faster iteration** - no API rate limits

Let's set this up step by step.

### Step 1: Install Ollama

Visit [ollama.ai](https://ollama.ai/) and follow the installation instructions for your platform. This typically involves downloading and running an installer.

### Step 2: Download a Model

Once Ollama is installed, download a model. We'll use Llama 3.2 with 3 billion parameters - it's capable but small enough to run on most development machines:

```bash
# This downloads the model (may take a few minutes)
ollama pull llama3.2:3b
```

### Step 3: Start the Ollama Server

Ollama runs as a server that your agent will connect to. Open a new terminal and start it:

```bash
# In a separate terminal window/tab
ollama serve
```

**Keep this terminal running!** You should see output like:

**Note:** If the port is in use, change it with the `OLLAMA_HOST` environment variable.

### Step 4: Install the Strands Ollama Provider

In your original terminal (not the one running `ollama serve`):

```bash
pip install "strands-agents[ollama]"
```

### Step 5: Create Your Local Agent

Now create a local version of our subject expert. Save this as `local_subject_expert.py`:

```python
from strands import Agent
from strands.models.ollama import OllamaModel

# Configure the local model
ollama_model = OllamaModel(
    host="http://localhost:11434",  # Default Ollama server address
    model_id="llama3.2:3b"          # The model we downloaded
)

# Create your local agent
local_agent = Agent(
    model=ollama_model,
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
    """
)

def interactive_session():
    print("🖥️  Local Computer Science Subject Expert (using Ollama)")
    print("=" * 55)
    print("This agent runs entirely on your local machine!")
    
    while True:
        user_input = input("\nYour question: ")
        
        if user_input.lower() in ["exit", "quit", "bye"]:
            print("👋 Goodbye!")
            break
        
        print("\n🤔 Thinking...\n")
        try:
            local_agent(user_input)
        except Exception as e:
            print(f"❌ Error: {e}")
            print("💡 Make sure Ollama is running with: ollama serve")

if __name__ == "__main__":
    interactive_session()
```

### Step 6: Test Your Local Agent

Run your agent:
```bash
python local_subject_expert.py
```

Now you have a fully functional agent running entirely on your local machine!

### Understanding Local Model Trade-offs

Local models are fantastic for development, but it's important to understand their limitations:

**Strengths:**

- ✅ No API costs during development
- ✅ No rate limits
- ✅ Works offline

**Limitations:**

- ⚠️ May struggle with complex agentic workflows compared to larger models
- ⚠️ May not follow complex multi-step instructions as reliably
- ⚠️ Require significant local hardware resources
- ⚠️ Can have difficulty with tool execution

This trade-off is often worthwhile for development and testing, but you may want to use cloud models for production agentic applications.

## Advanced: Custom Model Providers

For highly specialized needs, you can create custom model providers by extending Strands' base `Model` class. This might be useful for:

- Custom API endpoints
- Internal company models
- Specialized fine-tuned models
- Unique authentication requirements

## What We've Learned

In this tutorial, we've explored Strands Agents' model provider flexibility:

- ✅ **Switched Bedrock models and regions** for specific requirements
- ✅ **Connected to OpenAI and other cloud providers** for different capabilities
- ✅ **Set up local development with Ollama** for cost-free experimentation
- ✅ **Understood trade-offs** between local and cloud models
- ✅ **Learned when to use each approach** based on your needs

The key insight is that Strands abstracts away model provider complexity, letting you focus on building great agents rather than managing API differences.

## Next Steps & Resources

In the upcoming Part 5, we'll explore **Conversation Management & Memory**, showing you how to maintain state and context across conversations to create more engaging and personalized agent experiences.

While you wait, try these experiments:

- **Compare responses** from the same prompt across different models
- **Test tool usage** with local vs. hosted models to see the differences
- **Set up multiple local models** and switch between them based on query type
- **Benchmark performance** for your specific use cases

### 

Want to learn more about model providers?

Here are some resources to help you deepen your understanding:

- 📚 [**Strands Agents Documentation**](https://strandsagents.com/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el) - Comprehensive guides and API references
- 🖥️ [**Ollama**](https://ollama.com/) - For local model deployment

What's your preferred model provider for development vs. production? Share your experiences in the comments below!

___

## 💡Troubleshooting Tips

**Ollama Connection Issues:**

- Ensure `ollama serve` is running in a separate terminal before starting your agent
- Check that port 11434 isn't blocked by firewall settings
- If the port is in use, change it with the `OLLAMA_HOST` environment variable, e.g., `OLLAMA_HOST=127.0.0.1:8080`

**Model Access Problems:**

- For Bedrock models, confirm you've [requested access](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access-modify.html?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el) in the AWS console for the specific region(s)
- For other providers, verify your API key is valid and you haven't exceeded usage limits
- Double-check that model IDs match exactly what's supported by each provider

**Performance Issues:**

- Local models require significant RAM (8GB+ recommended for larger models)
- Start with smaller models like `llama3.2:1b` if you have limited resources
- Cloud models typically respond faster but have per-token costs
- If local models seem slow, try reducing the `max_tokens` parameter  

