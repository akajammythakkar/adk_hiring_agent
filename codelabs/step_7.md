# Step 7: Building the Root Orchestrator

**Level:** Advanced

## Creating the Conversational Coordinator

In this step, we'll build the **Root Orchestrator Agent** - the "brain" of our system that coordinates all specialized agents and provides a conversational user interface.

## Understanding the Root Orchestrator

### Purpose and Workflow

The Root Orchestrator:

1. **Manages conversation flow** - guides users through evaluation steps
2. **Calls specialized agents** as tools at appropriate times
3. **Maintains context** - tracks what's been collected and completed
4. **Presents results** - displays agent outputs clearly
5. **Handles edge cases** - manages missing data, errors, retries

### What Makes This Different?

Unlike specialized agents that focus on **one task**, the orchestrator:

- Manages **multi-step workflows**
- Makes **decisions about when to call which agent**
- Provides **user-friendly explanations**
- Handles **conversation state**

## Setting Up the Root Agent File

### Create agent.py

Open `agent.py` and add the foundation:

```python
"""
Conversational multi-agent orchestrator that uses specialized sub-agents as tools.
"""

from google.adk.agents import LlmAgent
from google.adk.tools import AgentTool, FunctionTool

# Import the simplified sub-agents (no template variables)
from .tools_agents import (
    rubric_builder,
    resume_reviewer,
    github_validator,  # Import the function, not the agent
    github_reviewer,
    verdict_synthesizer,
)
import os
from dotenv import load_dotenv

load_dotenv()
MODEL_NAME = os.getenv("MODEL_NAME")
```

**Key imports:**

- **LlmAgent**: Base class for creating agents
- **AgentTool**: Wrapper to convert agents into tools
- **All sub-agents**: Import from tools_agents.py

## Wrapping Sub-Agents as Tools

### Step 1: Create AgentTool Wrappers

Add this to `agent.py`:

```python
# Create AgentTools that wrap the sub-agents
rubric_tool = AgentTool(agent=rubric_builder)
resume_eval_tool = AgentTool(agent=resume_reviewer)
github_validate_tool = FunctionTool(func=github_validator)  # Use FunctionTool for direct function call
github_eval_tool = AgentTool(agent=github_reviewer)
verdict_tool = AgentTool(agent=verdict_synthesizer)
```

**What do AgentTool and FunctionTool do?**

**AgentTool** - Wraps an LLM agent as a tool:

```
Sub-Agent (LlmAgent) → AgentTool → Tool for Root Agent
   ↓                      ↓               ↓
Has instructions    Wraps agent    Root can "call" it
```

**FunctionTool** - Wraps a Python function as a tool:

```
Python Function → FunctionTool → Tool for Root Agent
   ↓                 ↓               ↓
Direct logic    Wraps function   Root calls with params
```

**Why use FunctionTool for GitHub validation?**

- ✅ **More efficient**: Direct function call (no LLM needed for API logic)
- ✅ **Deterministic**: Consistent results every time
- ✅ **Faster**: No extra LLM API call
- ✅ **Cost-effective**: Saves on token usage

The root agent treats these as **function calls**:

```python
# Root agent thinks like this:
"User provided resume → I should call resume_eval_tool"
"GitHub URL found → I should call github_validate_tool(username='...')"
```

## Building the Root Orchestrator Agent

### Step 2: Define the Root Agent  

**(Note: Due to length constraints, the full instruction prompt is in the actual `agent.py` file. Below is an overview.)**

```python
root_agent = LlmAgent(
    name="ConversationalHiringOrchestrator",
    model=MODEL_NAME,
    description=(
        "An interactive hiring assistant that orchestrates specialized sub-agents "
        "to evaluate candidates step-by-step through conversation."
    ),
    tools=[
        rubric_tool,
        resume_eval_tool,
        github_validate_tool,
        github_eval_tool,
        verdict_tool,
    ],
    instruction="""
    [Detailed workflow instructions defining:
    - Conversational flow (7 steps)
    - Tool calling logic
    - State tracking
    - Error handling
    - User experience principles]
    """,
)
```

## Understanding the Orchestrator's Logic

### Automatic vs. Manual Transitions

**Automatic Transitions** (no user confirmation):

```
User provides JD → Auto-generate rubric
User provides resume → Auto-evaluate resume
GitHub found → Auto-validate → Auto-analyze (if valid)
```

**Manual Transitions** (ask user):

```
Evaluation complete → Ask: "Generate final verdict?"
GitHub failed → Ask: "Provide new URL or skip?"
```

**Why this design?**

- **Automatic**: Clear next action, saves time
- **Manual**: Decision point, user input needed

### Tool Calling Strategy

The orchestrator decides **when to call which tool**:

```python
# Pseudo-logic inside the orchestrator
if user_message_contains_job_description:
    call rubric_tool
    
elif user_message_contains_resume and rubric_exists:
    call resume_eval_tool
    
elif github_url_mentioned:
    call github_validate_tool
    if validation_passed:
        call github_eval_tool
        
elif user_requests_verdict and evaluations_exist:
    call verdict_tool
```

The orchestrator's LLM handles this logic **automatically** based on conversation context.

### Important: Function Tool Parameters

When calling the `github_validate_tool`, the orchestrator must pass the `username` parameter:

```python
# Correct usage
github_validate_tool(username="johndoe")
github_validate_tool(username="github.com/johndoe")

# The root agent's instructions tell it to pass parameters:
# "call github_validator(username='EXTRACTED_USERNAME')"
```

This is different from AgentTools which don't require explicit parameters - they get context from conversation history.

### Conversation State Management

The orchestrator tracks progress **implicitly** through conversation history:

```
Turn 1: JD provided → Rubric generated ✓
Turn 2: Resume provided → Resume evaluated ✓
Turn 3: GitHub validated ✓ → GitHub evaluated ✓
Turn 4: User asks for verdict → Verdict generated ✓
```

If user asks "status", orchestrator reviews history and reports:

```markdown
**Current Status:**
✓ Job description collected
✓ Rubric generated
✓ Resume evaluated (Score: 8/10)
✓ GitHub validated and evaluated (Score: 7/10)
✗ Final verdict (not yet generated)

**Next Action:** Would you like me to generate a final verdict?
```

## Testing the Complete System

### Interactive Test

Create `test_interactive.py`:

```python
"""
Interactive test for the complete hiring agent system.
"""

from agent import root_agent

def run_interactive_session():
    print("="*60)
    print("AI HIRING ASSISTANT - INTERACTIVE TEST")
    print("="*60)
    print("\nType 'quit' to exit\n")
    
    # Start with greeting
    print("Assistant: ", end="")
    response = root_agent.run("Hello")
    print(response)
    print("\n" + "-"*60 + "\n")
    
    while True:
        # Get user input
        user_input = input("You: ")
        
        if user_input.lower() in ['quit', 'exit', 'bye']:
            print("\nAssistant: Thank you for using the AI Hiring Assistant!")
            break
        
        # Get agent response
        print("\nAssistant: ", end="")
        response = root_agent.run(user_input)
        print(response)
        print("\n" + "-"*60 + "\n")

if __name__ == "__main__":
    run_interactive_session()
```

Run it:

```bash
python test_interactive.py
```

**Example interaction:**

```
You: Hello