# AI-Powered Conversational Hiring Agent

Welcome to the Codelab for building an **AI-Powered Conversational Hiring Agent System** using Google's ADK (Agent Development Kit) and Gemini 3 models!

![final photoo adk](https://github.com/user-attachments/assets/1afcac0b-6a36-4b74-80b6-217c29d80e13)

> Made with ❤️ for the open source

Follow the Codelab: [codelab](https://imjt.dev/adk_hiring_agent)

## Overview

This Codelab teaches you how to build a sophisticated multi-agent system that automates technical candidate evaluation through a conversational interface. The system uses specialized AI agents to evaluate resumes, validate GitHub profiles, and provide comprehensive hiring recommendations.

### What You'll Build

A complete hiring automation system featuring:

- **5 Specialized AI Agents**: RubricBuilder, ResumeReviewer, GitHubValidator, GitHubReviewer, VerdictSynthesizer
- **1 Root Orchestrator**: Conversational coordinator that manages the evaluation workflow
- **Real-time GitHub API Integration**: Live account validation and metrics
- **Structured Evaluation Framework**: Objective, measurable criteria with scoring
- **End-to-End Workflow**: From job description to final HIRE/NO HIRE decision

### Technologies Used

- **Google ADK (Agent Development Kit)** - Agent framework
- **Gemini 3 Model** - LLM for agent intelligence
- **Python 3.8+** - Programming language
- **GitHub REST API** - Account validation
- **python-dotenv** - Environment management
- **requests** - HTTP client library

### Key Concepts You'll Learn

- **Multi-Agent Systems**: Coordinating multiple specialized agents
- **Prompt Engineering**: Crafting effective agent instructions
- **Tool Calling**: Using agents as tools for other agents
- **Conversation Management**: State tracking across multiple turns
- **API Integration**: Real-time external data fetching
- **Structured Evaluation**: Building objective assessment frameworks
- **System Design**: Architecting scalable AI applications

## Getting Started

### Prerequisites

- Python 3.8+
- Google Cloud account (for Gemini API)
- Basic understanding of Python and REST APIs
- GitHub account (optional, for testing)

### Quick Start

1. **Clone or navigate to the project directory:**
   ```bash
   git clone https://github.com/akajammythakkar/adk-hiring-agent.git
   cd adk-hiring-agent
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Set up environment variables:**
   - Create a `.env` file in the project root
   - Add your API keys:
     ```
     GEMINI_API_KEY=your_gemini_api_key
     GITHUB_TOKEN=your_github_token  # Optional
     ```

4. **Follow the Codelab:**
   - Start with [https://gradus.dev/labs/building-a-hiring-agent-with-adk-gemini-3](https://gradus.dev/labs/building-a-hiring-agent-with-adk-gemini-3)

## 🔗 Codelab Link

📚 **Gradus Codelab**: [https://gradus.dev/labs/building-a-hiring-agent-with-adk-gemini-3](https://gradus.dev/labs/building-a-hiring-agent-with-adk-gemini-3)

## 👥 Contributors

This Codelab was created with ❤️ by:

- **Jay** - [@akajammythakkar](https://github.com/akajammythakkar)
- **Vrijraj** - [@vrijraj](https://github.com/vrijraj)

## Contributing

Found an error? Have a suggestion? Want to share your implementation?

- **Report issues**: Create an issue with detailed description
- **Submit improvements**: Pull requests welcome!
- **Share your work**: Show us what you've built

**Good luck, and happy building!** 🚀

Google Cloud credits are provided for this project.
