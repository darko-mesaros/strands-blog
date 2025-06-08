---
title: "Building AI Agents with Strands: A Hands-On Tutorial Series"
date: 2025-05-21T11:30:03+00:00
weight: 2
# aliases: ["/first"]
tags: ["strands", "agents", "tutorial"]
author: "Dennis Traub"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Follow this hands-on journey and learn how to build AI agents using the open source Strands Agents SDK."
canonicalURL: "https://community.aws/content/2xOwSRPn2OwV2nj6ZYyTxn6P7gH/building-ai-agents-with-strands-an-introduction-to-the-series"
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
    image: "stra_tut_intro_header.jpeg" # image path/url
    alt: "header image" # alt text
    relative: true  # when using page bundles set this to true
    hidden: true # only hide on current single page

---
Last Modified May 27, 2025

Welcome to this hands-on tutorial series on building AI agents with the [Strands Agents SDK](https://strandsagents.com/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el "https://strandsagents.com/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el")!

Artificial Intelligence is rapidly evolving, and AI Agents are at the forefront of this revolution. These intelligent systems can understand context, make decisions, use tools, and collaborate to perform complex tasks, moving beyond simple question-answering systems.

If you're a developer interested in creating intelligent agents powered by foundation models like Claude, GPT, or Llama, you've come to the right place.

## What We're Building: The Integrated Learning Lab

Throughout this series, we'll progressively build an **Integrated Learning Lab** – a practical application that demonstrates key Strands Agents concepts and features in a cohesive project.

I'll be using [Amazon Bedrock](https://aws.amazon.com/bedrock/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el "https://aws.amazon.com/bedrock/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el") as our primary foundation model provider, though one of the strengths of Strands Agents is its flexibility to work with various providers including Anthropic, LiteLLM, Ollama, and Llama API. We'll explore some of these alternatives in later tutorials.

___

🧑‍💻 No time for explanations? Want to jump right in? Here are the parts that I've already published:
1. 🚀 Foundations: Creating Your First Agent
2. ⚙️ Tool Integration: Enhancing Agent Capabilities
3. 🔌 Model Context Protocol: Integrate Your Agent with MCP
4. 🖥️ Alternative Model Providers and Local Development with Ollama
5. And there's more to come!
___

## What is the Strands Agents SDK?

The [Strands Agents SDK](https://strandsagents.com/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el "https://strandsagents.com/?trk=2af10798-ef42-4f17-9c3b-3ea26b3c37db&sc_channel=el") is a lightweight, code-first framework that simplifies the process of building intelligent agents. It provides:

- A straightforward agent loop that handles complex interactions
- Support for Amazon Bedrock, Anthropic, Llama, Ollama, and custom providers
- A robust system for building and integrating tools to extend agent capabilities
- Native support for Model Context Protocol (MCP) servers, enabling access to thousands of pre-built tools
- Advanced patterns like multi-agent systems and autonomous workflows
- Built-in security capabilities for responsible AI development

This last point is particularly important – as AI systems become more powerful, ensuring they operate safely and responsibly is increasingly critical. The Strands Agents SDK includes features for guardrails, prompt security, and responsible tool design that we'll explore throughout this series.

## My Learning Path

While I have a general roadmap for this series, it may evolve as I continue experimenting with the SDK. Here's my current plan for the topics we'll cover:

🚀 **[Foundations: Creating Your First Agent](/posts/strands_tutorial_part1)** \
Setting up the SDK and building a simple agent that will be a subject expert for our learning lab.

⚙️ **[Tool Integration: Enhancing Agent Capabilities](/posts/strands_tutorial_part2)** \
Creating custom tools that extend our agent's abilities beyond just simple conversation.

🔌 **[Model Context Protocol: Integrate Your Agent with MCP](/posts/strands_tutorial_part3)** \
Exploring the Model Context Protocol (MCP) to connect our agent with external capabilities.

🖥️ **[Alternative Model Providers and Local Development with Ollama](/posts/strands_tutorial_part4)**  \
Setting up local model deployment for development and testing environments.

🧠 **Conversation Management & Memory** (upcoming)   
Implementing state tracking and persistence for contextual awareness.

🛡️ **Security Deep Dive** (upcoming)  
Exploring core security principles for AI agents, including guardrails and prompt security.

🎭 **Simple Multi-Agent Patterns: Agents as Tools** (upcoming)  
Creating a multi-agent system with an orchestrator coordinating specialized agents.

🌐 **Advanced Multi-Agent Systems: Swarms and Graphs** (upcoming)  
Diving into agent swarms and graphs for more sophisticated applications.

📊 **Observability, Testing & Evaluation** (upcoming)  
Adding logging, tracing, metrics, and testing to understand and improve performance.

☁️ **Deployment Options** (upcoming)  
Deploying agents to production using AWS Lambda, Fargate, or other environments.

👥 **Human-in-the-Loop Integration** (upcoming)  
Implementing review mechanisms and feedback loops combining AI with human oversight.

## Who is this series for?

This series is designed for developers who:

- Have basic familiarity with Python programming
- Are interested in practical applications of LLMs and generative AI
- Want to learn through hands-on, code-focused examples
- Prefer building real systems over theoretical discussions

## Getting Started

In our first tutorial, we'll set up the Strands Agents SDK and create a simple Subject Expert agent. You'll see how quickly we can go from zero to a functional AI agent with remarkably few lines of code.

___

_Note: This series focuses on practical implementation. If you're interested in the theoretical concepts behind LLMs and agent systems, I'll include suggested resources at the end of each post._  
