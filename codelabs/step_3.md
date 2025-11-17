# Step 3: Understanding Multi-Agent Architecture

**Level:** Intermediate

## Understanding Sub-Agents and Multi-Agent Systems

In this step, we'll explore the **multi-agent architecture** that powers our hiring system. You'll learn why we use multiple specialized agents instead of a single monolithic agent, and how they work together.

## What is a Multi-Agent System?

A **multi-agent system** consists of multiple AI agents, each specialized for a specific task, working together to solve complex problems.

### Single Agent vs Multi-Agent

**Single Agent Approach (Traditional):**

```
User → [One Large Agent] → Response
         ↓
    Does everything:
    - Parse job description
    - Evaluate resume
    - Check GitHub
    - Make decision
```

**Multi-Agent Approach (Our System):**

```
User → [Root Orchestrator]
         ↓
         ├→ [RubricBuilder] → Custom rubric
         ├→ [ResumeReviewer] → Resume score
         ├→ [GitHubValidator] → Account verification
         ├→ [GitHubReviewer] → Portfolio analysis
         └→ [VerdictSynthesizer] → Final decision
```

## Benefits of Multi-Agent Architecture

### 1. Separation of Concerns

Each agent has a **single, focused responsibility**:

```python
# Bad: One agent doing everything
mega_agent = LlmAgent(
    instruction="Evaluate candidates by checking resume, GitHub, 
                 and making hiring decisions..."
)

# Good: Specialized agents
rubric_builder = LlmAgent(
    name="RubricBuilder",
    instruction="Generate evaluation rubric from job description..."
)

resume_reviewer = LlmAgent(
    name="ResumeReviewer",
    instruction="Evaluate resume against rubric..."
)
```

### 2. Reusability

Agents can be used in different workflows:

- Use **RubricBuilder** for any role evaluation
- Use **ResumeReviewer** standalone or in batch processing
- Swap agents without affecting others

### 3. Easier Testing and Debugging

Test each agent independently:

```python
# Test RubricBuilder alone
rubric = rubric_builder.run("Senior Python Developer JD...")

# Test ResumeReviewer with known rubric
evaluation = resume_reviewer.run(rubric="...", resume="...")
```

### 4. Improved Prompt Quality

Specialized prompts are **more focused and effective**:

- Shorter instructions per agent
- Clear input/output contracts
- Easier to optimize each agent's behavior

### 5. Scalability

Add new evaluation criteria without touching existing agents:

```python
# Add new agent without modifying others
code_quality_agent = LlmAgent(
    name="CodeQualityAnalyzer",
    instruction="Analyze code style and best practices..."
)
```

## Agent Roles in Our System

### 1. RubricBuilder Agent

**Purpose:** Generate role-specific evaluation criteria

**Input:** Job description from conversation history

**Output:** Structured rubric with scoring guidelines

**Key Responsibilities:**
- Parse job requirements
- Create objective scoring criteria
- Define point allocations
- Specify what evidence to look for

**Example Output Structure:**

```markdown
## LEVEL 1: RESUME EVALUATION (10 points total)

**1. Technical Skills Match (4 points)**
- 4 points: All required skills (Python, Django, PostgreSQL)
- 3 points: Most required skills (75%+)
...
```

### 2. ResumeReviewer Agent

**Purpose:** Evaluate candidate resumes against the rubric

**Input:** 
- Rubric (from RubricBuilder)
- Resume text from conversation

**Output:** Detailed scored evaluation with evidence

**Key Responsibilities:**
- Extract candidate information
- Score each rubric criterion
- Cite specific resume evidence
- Identify strengths and gaps
- Extract GitHub information
- Provide PASS/FAIL recommendation

**Example Output:**

```markdown
**CANDIDATE:** John Doe
**SCORE: 8/10**

**1. Technical Skills Match: 4/4 points**
- Required skills found: Python (5 years), Django, PostgreSQL
- Evidence: "Led Django REST API development..."
```

### 3. GitHubValidatorAgent

**Purpose:** Verify GitHub account existence using REST API

**Input:** GitHub username/URL from conversation

**Output:** Validation report with account metrics

**Key Responsibilities:**
- Clean and parse GitHub username
- Call GitHub REST API in real-time
- Extract account metadata
- Assess account quality
- Provide proceed/skip recommendation

**Example API Call:**

```python
import requests

response = requests.get(
    f"https://api.github.com/users/{username}",
    headers={"Accept": "application/vnd.github.v3+json"}
)

data = response.json()
# Extract: public_repos, followers, created_at, etc.
```

### 4. GitHubReviewer Agent

**Purpose:** Analyze GitHub profile for technical capability

**Input:**
- Validation results
- Resume experience level
- Job requirements

**Output:** Technical portfolio assessment

**Key Responsibilities:**
- Evaluate repository quality
- Check technology stack alignment
- Assess activity and consistency
- Review documentation practices
- Provide technical PASS/FAIL recommendation

**Note:** Uses simulated analysis based on resume + validation data since actual repository inspection requires extensive API calls.

### 5. VerdictSynthesizer Agent

**Purpose:** Provide final hiring recommendation

**Input:** All evaluation data from conversation history

**Output:** Comprehensive hiring verdict

**Key Responsibilities:**
- Synthesize all evaluation scores
- Calculate composite score
- Identify compelling strengths
- Highlight critical concerns
- Provide HIRE/NO HIRE decision
- Recommend next steps (interviews, etc.)
- Specify confidence level

## Agent Communication Pattern

### How Agents Access Information

All agents have access to the **full conversation history**:

```python
# Agent doesn't need explicit parameters
rubric_builder = LlmAgent(
    name="RubricBuilder",
    instruction="""
    Analyze the job description from the conversation history
    and create a comprehensive evaluation rubric.
    """
)
```

When the root orchestrator calls an agent:

```python
# Root agent calls tool
rubric_tool = AgentTool(agent=rubric_builder)

# Agent automatically sees entire conversation
# No need to pass job description explicitly
```

### Conversation Context Flow

```
Step 1: User provides Job Description
  ↓ (conversation history)
Step 2: RubricBuilder reads JD from history → generates rubric
  ↓ (conversation history includes JD + rubric)
Step 3: User provides Resume
  ↓ (conversation history includes JD + rubric + resume)
Step 4: ResumeReviewer reads rubric + resume from history → scores resume
  ↓ (conversation history includes all above + evaluation)
Step 5: VerdictSynthesizer reads ALL history → final verdict
```

## Designing Agent Instructions

### Key Principles for Agent Prompts

**1. Clear Role Definition**

```python
instruction="""
You are an expert HR assessment designer with deep experience 
in creating objective, measurable evaluation criteria for 
technical roles.
"""
```

**2. Explicit Task Description**

```python
instruction="""
## YOUR TASK:
Analyze the job description from the conversation history and 
create a comprehensive, role-specific evaluation rubric.
"""
```

**3. Structured Output Format**

```python
instruction="""
## OUTPUT FORMAT:

**CANDIDATE:** [Extract full name]
**SCORE: X/10**

**DETAILED BREAKDOWN:**
...
"""
```

**4. Specific Guidelines and Rules**

```python
instruction="""
## CRITICAL RULES:

1. Only score based on what's in the resume
2. Cite evidence for every score
3. Calculate accurately
"""
```

**5. Examples (When Helpful)**

```python
instruction="""
**Example code you can execute:**

```python
import requests
response = requests.get(f"https://api.github.com/users/{username}")
```
"""
```

## Agent Tool Wrapping

### Converting Agents to Tools

In Google ADK, agents can be used as **tools** by other agents:

```python
from google.adk.tools import AgentTool

# Create sub-agent
rubric_builder = LlmAgent(
    name="RubricBuilder",
    model=GEMINI_MODEL,
    description="Generates customized evaluation rubric from job description.",
    instruction="..."
)

# Wrap as tool
rubric_tool = AgentTool(agent=rubric_builder)

# Root agent uses it as a tool
root_agent = LlmAgent(
    name="Orchestrator",
    tools=[rubric_tool, resume_tool, github_tool, ...]
)
```

### Why Use AgentTool?

**Benefits:**

- **Encapsulation**: Sub-agent logic is self-contained
- **Reusability**: Same agent can be tool in multiple workflows
- **Tool Calling Interface**: Root agent treats sub-agents like function calls
- **Conversation Context**: All agents share conversation history automatically

## Code Organization Structure

### File Organization Best Practice

**tools_agents.py** - All specialized sub-agents:

```python
# tools_agents.py

# Agent 1: Rubric generation
rubric_builder = LlmAgent(...)

# Agent 2: Resume evaluation
resume_reviewer = LlmAgent(...)

# Agent 3: GitHub validation
github_validator_agent = LlmAgent(...)

# Agent 4: GitHub analysis
github_reviewer = LlmAgent(...)

# Agent 5: Final verdict
verdict_synthesizer = LlmAgent(...)
```

**agent.py** - Root orchestrator:

```python
# agent.py
from .tools_agents import (
    rubric_builder,
    resume_reviewer,
    github_validator_agent,
    github_reviewer,
    verdict_synthesizer,
)

# Wrap as tools
rubric_tool = AgentTool(agent=rubric_builder)
resume_eval_tool = AgentTool(agent=resume_reviewer)
# ...

# Create root orchestrator
root_agent = LlmAgent(
    name="ConversationalHiringOrchestrator",
    tools=[rubric_tool, resume_eval_tool, ...]
)
```

## Summary

In this step, you learned:

- ✓ **Multi-agent architecture** separates concerns and improves maintainability
- ✓ **Five specialized agents** handle different evaluation tasks
- ✓ **Agents share conversation history** for context-aware processing
- ✓ **AgentTool wrapper** allows agents to be used as tools
- ✓ **Clear instructions** with role, task, format, and rules make agents effective
- ✓ **Structured file organization** keeps code clean and scalable

## What's Next?

Now that you understand the architecture, we'll start building the agents! In the next step, we'll create the **RubricBuilder agent** - the first agent in our evaluation pipeline.

---

**Previous:** [← Step 2 - Setting Up the Environment](step_2.md) | **Next:** [Step 4 - Building the Rubric Builder Agent →](step_4.md)

