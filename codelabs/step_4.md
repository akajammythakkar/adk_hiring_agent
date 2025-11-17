# Step 4: Building the Rubric Builder Agent

**Level:** Intermediate

## Creating Your First Specialized Agent

In this step, we'll build the **RubricBuilder agent** - the first agent in our evaluation pipeline. This agent transforms a job description into a structured, objective evaluation rubric.

## Understanding the Rubric Builder

### Purpose and Workflow

The RubricBuilder agent:

1. **Reads** the job description from conversation history
2. **Extracts** key requirements (skills, experience, education)
3. **Generates** a structured rubric with clear scoring criteria
4. **Returns** a detailed evaluation framework

### Input and Output

**Input:** Job description (from conversation history)

```
We're hiring a Senior Python Developer with 5+ years of experience.
Must have: Python, Django, PostgreSQL, Docker, AWS
Nice to have: React, GraphQL
...
```

**Output:** Structured rubric with scoring guidelines

```markdown
## LEVEL 1: RESUME EVALUATION (10 points total)

**1. Technical Skills Match (4 points)**
- 4 points: All required skills present (Python, Django, PostgreSQL, Docker, AWS)
...
```

## Setting Up the Agent File

### Step 1: Create tools_agents.py

Open `tools_agents.py` and add the necessary imports:

```python
"""
Simplified sub-agents for conversational workflow (without template variables).
"""

from google.adk.agents import LlmAgent

import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()
GEMINI_MODEL = os.getenv("MODEL_NAME")
```

**Important components:**

- **LlmAgent**: The base class for creating AI agents
- **load_dotenv()**: Loads API keys from `.env` file
- **GEMINI_MODEL**: The model name from environment variables

## Building the RubricBuilder Agent

### Step 2: Define the Agent

Add the RubricBuilder agent to `tools_agents.py`:

```python
# Rubric Builder - takes job description directly from conversation
rubric_builder = LlmAgent(
    name="RubricBuilder",
    model=GEMINI_MODEL,
    description="Generates customized evaluation rubric from job description.",
    instruction="""
You are an expert HR assessment designer with deep experience in creating 
objective, measurable evaluation criteria for technical roles.

## YOUR TASK:
Analyze the job description from the conversation history and create a 
comprehensive, role-specific evaluation rubric that will be used to assess 
candidates.

## OUTPUT STRUCTURE:

### LEVEL 1: RESUME EVALUATION (10 points total)

**1. Technical Skills Match (4 points)**
- 4 points: All required technical skills present with strong evidence
  * List the SPECIFIC required skills from the JD
  * Define what "strong evidence" means for this role
- 3 points: Most required skills present (75%+), some with depth
- 2 points: Some required skills present (50-75%), limited depth
- 1 point: Few required skills (<50%), mostly surface-level mentions
- 0 points: No technical skill alignment

**2. Experience Level (3 points)**
- 3 points: Meets or exceeds required years with progressive responsibility
- 2 points: Within 1 year of requirement with relevant experience
- 1 point: 2+ years below requirement but compensating factors present
- 0 points: Significantly below requirements with no compensating factors

**3. Education & Qualifications (2 points)**
- 2 points: Meets stated education requirements
  * OR equivalent experience/certifications if role allows
- 1 point: Related education or significant self-taught evidence
- 0 points: Does not meet requirements

**4. Role Relevance & Domain Fit (1 point)**
- 1 point: Previous roles highly relevant to this position
  * Specify domain relevance
  * Similar role levels
- 0 points: Different domain or significantly different role type

### LEVEL 2: GITHUB EVALUATION (10 points total)

**1. Repository Quality (3 points)**
- 3 points: Multiple well-architected projects demonstrating:
  * Clean code organization and structure
  * Design patterns and best practices
  * Complex problem-solving
- 2 points: Some quality projects with good practices
- 1 point: Basic projects or limited complexity
- 0 points: Poor code quality or trivial projects only

**2. Technology Stack Alignment (3 points)**
- 3 points: Strong evidence of required tech stack
  * Multiple projects using core technologies
  * Depth (not just Hello World examples)
- 2 points: Some relevant technologies, moderate depth
- 1 point: Limited evidence of required tech stack
- 0 points: No technology alignment

**3. Activity & Consistency (2 points)**
- 2 points: Active contributions in past 6 months, consistent patterns
- 1 point: Some activity in past 6-12 months or sporadic contributions
- 0 points: No recent activity (12+ months) or very infrequent

**4. Documentation & Testing (2 points)**
- 2 points: Good READMEs, code comments, visible test coverage
- 1 point: Basic documentation present
- 0 points: No documentation or tests

## IMPORTANT GUIDELINES:

1. **Be Specific**: Reference exact technologies and requirements from the JD
2. **Be Fair**: Scoring should be objective and measurable
3. **Be Practical**: Focus on criteria that predict job performance
4. **Be Clear**: Evaluators should know exactly what earns each score

## OUTPUT FORMAT:
- Use markdown formatting (##, **, -, bullet points)
- Do NOT use code fences (````)
- Start directly with the heading
- Include specific required skills from the JD in criteria descriptions

Return ONLY the rubric, no preamble text.
""",
)
```

### Step 3: Understanding the Agent Components

**name**: Identifier for the agent

```python
name="RubricBuilder"
```

**model**: Gemini model to use (from environment)

```python
model=GEMINI_MODEL  # e.g., "gemini-2.0-flash-exp"
```

**description**: Brief summary of agent's purpose

```python
description="Generates customized evaluation rubric from job description."
```

This description helps the **root orchestrator** understand when to call this agent.

**instruction**: Detailed prompt that defines agent behavior

The instruction is the most important component. Let's break it down:

## Anatomy of an Effective Agent Instruction

### 1. Role Definition

```
You are an expert HR assessment designer with deep experience in creating 
objective, measurable evaluation criteria for technical roles.
```

**Why this matters:**
- Sets the agent's persona and expertise level
- Influences the tone and approach of responses
- Establishes credibility for the task

### 2. Task Definition

```
## YOUR TASK:
Analyze the job description from the conversation history and create a 
comprehensive, role-specific evaluation rubric.
```

**Why this matters:**
- Clear, actionable objective
- Specifies input source (conversation history)
- Defines expected output (evaluation rubric)

### 3. Output Structure

```
## OUTPUT STRUCTURE:

### LEVEL 1: RESUME EVALUATION (10 points total)

**1. Technical Skills Match (4 points)**
- 4 points: All required technical skills present...
- 3 points: Most required skills present (75%+)...
```

**Why this matters:**
- Provides exact template for agent to follow
- Ensures consistency across different job descriptions
- Makes output predictable and parseable
- Shows agents the level of detail expected

### 4. Guidelines and Rules

```
## IMPORTANT GUIDELINES:

1. **Be Specific**: Reference exact technologies from the JD
2. **Be Fair**: Scoring should be objective
3. **Be Practical**: Focus on job performance predictors
```

**Why this matters:**
- Sets quality standards
- Prevents common mistakes
- Aligns output with business needs
- Emphasizes key principles

### 5. Format Specifications

```
## OUTPUT FORMAT:
- Use markdown formatting (##, **, -, bullet points)
- Do NOT use code fences (````)
- Return ONLY the rubric, no preamble text.
```

**Why this matters:**
- Ensures clean, presentable output
- Prevents formatting issues
- Makes output directly usable

## Rubric Structure Explained

### Two-Level Evaluation System

**Level 1: Resume Evaluation (10 points)**

This is the **initial screen** - fast and efficient:

- **Technical Skills Match (4 points)**: Most important for technical roles
- **Experience Level (3 points)**: Seniority alignment
- **Education & Qualifications (2 points)**: Baseline requirements
- **Role Relevance (1 point)**: Domain and role type fit

**Level 2: GitHub Evaluation (10 points)**

This is the **technical deep dive** - validates claims:

- **Repository Quality (3 points)**: Code quality and architecture
- **Technology Stack Alignment (3 points)**: Hands-on skill verification
- **Activity & Consistency (2 points)**: Ongoing learning and contribution
- **Documentation & Testing (2 points)**: Professional practices

### Why This Point Distribution?

**Resume Evaluation (10 points):**
- Technical skills get most weight (4 pts) - primary job requirement
- Experience level is critical (3 pts) - impacts performance
- Education matters less (2 pts) - can be compensated
- Domain fit is a bonus (1 pt) - helpful but not mandatory

**GitHub Evaluation (10 points):**
- Quality and tech stack are equal priority (3 pts each)
- Professional practices are secondary (2+2 pts)
- Focuses on what's actually in the portfolio

## Testing the RubricBuilder

### Step 4: Create a Test Script

Create a new file `test_rubric_builder.py`:

```python
"""
Test script for RubricBuilder agent.
"""

from tools_agents import rubric_builder

# Sample job description
job_description = """
**Position:** Senior Backend Engineer

**Requirements:**
- 5+ years of Python development experience
- Strong expertise in Django and FastAPI
- PostgreSQL database design and optimization
- Docker and Kubernetes experience
- AWS cloud infrastructure (EC2, S3, RDS)
- RESTful API design

**Nice to have:**
- GraphQL experience
- CI/CD pipeline setup
- React or Vue.js knowledge

**Education:**
- Bachelor's degree in Computer Science or equivalent experience

**About the role:**
We're building a high-traffic SaaS platform. You'll architect backend 
services, optimize database queries, and mentor junior developers.
"""

# Run the agent
print("Testing RubricBuilder agent...\n")
print("="*60)

# The agent will read the job_description from conversation context
# For testing, we can simulate this by passing it directly
result = rubric_builder.run(
    f"Job Description:\n\n{job_description}\n\nPlease generate an evaluation rubric."
)

print(result)
```

### Step 5: Run the Test

```bash
python test_rubric_builder.py
```

**Expected output** (example):

```markdown
## LEVEL 1: RESUME EVALUATION (10 points total)

**1. Technical Skills Match (4 points)**
- 4 points: All required technical skills present with strong evidence
  * Python (5+ years demonstrated experience)
  * Django AND FastAPI (multiple projects or significant depth)
  * PostgreSQL (query optimization, schema design examples)
  * Docker & Kubernetes (containerization experience)
  * AWS (EC2, S3, RDS specifically mentioned)
  * RESTful API design (architecture examples)
- 3 points: Most required skills present (5 of 6 core skills), depth in 3+ areas
- 2 points: Some required skills present (4 of 6), depth in 1-2 areas
- 1 point: Few required skills (<4 of 6), mostly surface mentions
- 0 points: Fewer than 3 core skills or no Python experience

**2. Experience Level (3 points)**
- 3 points: 5+ years Python/backend development with progressive responsibility
- 2 points: 4+ years with relevant backend experience
- 1 point: 3 years but strong compensating skills or domain expertise
- 0 points: Less than 3 years backend development experience
...
```

## Common Customizations

### Adjusting for Different Roles

**For junior roles**, decrease experience requirements:

```python
**2. Experience Level (3 points)**
- 3 points: 0-2 years with relevant internships or projects
- 2 points: Fresh graduate with strong academic projects
- 1 point: Bootcamp graduate with portfolio
- 0 points: No relevant experience or education
```

**For senior/lead roles**, emphasize leadership:

```python
**2. Experience Level (3 points)**
- 3 points: 8+ years with 2+ years in technical leadership
- 2 points: 7+ years with team mentoring experience
- 1 point: 5-6 years strong individual contributor
- 0 points: Less than 5 years or no leadership indicators
```

### Adding Custom Criteria

Add domain-specific criteria if needed:

```python
**5. Security Experience (bonus 2 points)**
- 2 points: Demonstrated security expertise (OWASP, penetration testing)
- 1 point: Basic security awareness mentioned
- 0 points: No security experience
```

## Summary

In this step, you:

- ✓ Created the **RubricBuilder agent** with structured instructions
- ✓ Learned how to write **effective agent instructions**
- ✓ Understood the **two-level evaluation rubric** structure
- ✓ Implemented **role, task, format, and guidelines** in prompts
- ✓ Tested the agent with a sample job description

## What's Next?

Now that we have a rubric generator, we'll build the **ResumeReviewer** and **GitHub evaluation agents** that use this rubric to assess candidates!

---

**Previous:** [← Step 3 - Understanding Sub-Agents](step_3.md) | **Next:** [Step 5 - Implementing Resume and GitHub Evaluation →](step_5.md)

