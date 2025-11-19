# Step 8: Testing the Complete System

**Level:** Advanced

## End-to-End Testing and Validation

In this step, we'll test the complete AI hiring agent system, validate all components work together, and learn how to debug common issues.

## Testing Strategy

### Three Levels of Testing

**1. Unit Testing** - Test each agent individually

**2. Integration Testing** - Test agents working together

**3. End-to-End Testing** - Test the complete user workflow

## Unit Testing Individual Agents

### Test 1: Rubric Builder

Create `tests/test_rubric_builder.py`:

```python
"""
Unit test for RubricBuilder agent.
"""

from tools_agents import rubric_builder

def test_rubric_generation():
    """Test rubric generation from job description."""
    
    job_desc = """
    Senior Backend Engineer
    Requirements:
    - 5+ years Python experience
    - Django and FastAPI expertise
    - PostgreSQL database design
    - Docker and AWS deployment
    
    Education: BS Computer Science or equivalent
    """
    
    print("Testing RubricBuilder...")
    result = rubric_builder.run(
        f"Generate evaluation rubric for this role:\n\n{job_desc}"
    )
    
    # Verify rubric structure
    assert "LEVEL 1: RESUME EVALUATION" in result
    assert "LEVEL 2: GITHUB EVALUATION" in result
    assert "Technical Skills Match" in result
    assert "Experience Level" in result
    
    # Verify role-specific technologies mentioned
    assert "Python" in result
    assert "Django" in result or "FastAPI" in result
    assert "PostgreSQL" in result
    
    print("✅ RubricBuilder test PASSED")
    print(f"\nGenerated rubric (first 300 chars):\n{result[:300]}...")
    
    return result

if __name__ == "__main__":
    test_rubric_generation()
```

Run it:

```bash
python tests/test_rubric_builder.py
```

**Expected output:**

```
Testing RubricBuilder...
✅ RubricBuilder test PASSED

Generated rubric (first 300 chars):
## LEVEL 1: RESUME EVALUATION (10 points total)

**1. Technical Skills Match (4 points)**
- 4 points: All required technical skills present (Python 5+ years, Django, FastAPI, PostgreSQL, Docker, AWS)
...
```

### Test 2: Resume Reviewer

Create `tests/test_resume_reviewer.py`:

```python
"""
Unit test for ResumeReviewer agent.
"""

from tools_agents import rubric_builder, resume_reviewer

def test_resume_evaluation():
    """Test resume evaluation with rubric."""
    
    # Step 1: Generate rubric
    job_desc = "Senior Python Developer - 5+ years, Django, PostgreSQL, AWS"
    rubric = rubric_builder.run(f"Generate rubric:\n\n{job_desc}")
    
    # Step 2: Create test resume
    resume = """
    John Doe
    Email: john@example.com
    GitHub: github.com/johndoe
    
    EXPERIENCE:
    Senior Backend Engineer, TechCorp (2019-2024) - 5 years
    - Built REST APIs using Django and FastAPI
    - Optimized PostgreSQL queries, reduced latency by 40%
    - Deployed microservices on AWS using Docker
    
    SKILLS: Python, Django, PostgreSQL, Docker, AWS, React
    
    EDUCATION:
    BS Computer Science, MIT, 2019
    """
    
    # Step 3: Evaluate resume
    print("Testing ResumeReviewer...")
    conversation = f"Rubric:\n{rubric}\n\nResume to evaluate:\n\n{resume}"
    result = resume_reviewer.run(conversation)
    
    # Verify evaluation structure
    assert "CANDIDATE:" in result
    assert "SCORE:" in result
    assert "Technical Skills Match:" in result
    assert "Experience Level:" in result
    assert "GITHUB INFORMATION:" in result
    assert "github.com/johndoe" in result.lower()
    assert "RECOMMENDATION:" in result
    
    print("✅ ResumeReviewer test PASSED")
    print(f"\nEvaluation summary (first 500 chars):\n{result[:500]}...")
    
    return result

if __name__ == "__main__":
    test_resume_evaluation()
```

Run it:

```bash
python tests/test_resume_reviewer.py
```

### Test 3: GitHub Validator Function

Create `tests/test_github_validator.py`:

```python
"""
Unit test for GitHub validator function.
"""

from tools_agents import github_validator

def test_github_validation():
    """Test GitHub account validation - direct function call."""
    
    # Test 1: Valid username
    print("Test 1: Valid GitHub username")
    result1 = github_validator("torvalds")  # Linus Torvalds' account
    assert result1["status"] in ["PASSED", "WARNING"]
    assert result1["format_valid"] == True
    assert result1["exists"] == True
    print(f"✅ Valid username test PASSED - Status: {result1['status']}")
    print(f"   Repos: {result1.get('public_repos', 'N/A')}")
    
    # Test 2: Invalid username (doesn't exist)
    print("\nTest 2: Non-existent username")
    result2 = github_validator("thisuserdoesnotexist12345xyz")
    assert result2["status"] == "FAILED"
    assert result2["exists"] == False
    print(f"✅ Non-existent username test PASSED - correctly detected as non-existent")
    
    # Test 3: URL format
    print("\nTest 3: GitHub URL format")
    result3 = github_validator("https://github.com/torvalds")
    assert result3["username"] == "torvalds"
    print(f"✅ URL parsing test PASSED - extracted username: {result3['username']}")
    
    # Test 4: @username format
    print("\nTest 4: @username format")
    result4 = github_validator("@torvalds")
    assert result4["username"] == "torvalds"
    print(f"✅ @username parsing test PASSED")
    
    print("\n🎉 All GitHub validator tests PASSED!")

if __name__ == "__main__":
    test_github_validation()
```

Run it:

```bash
python tests/test_github_validator.py
```

**Note:** This test makes **real API calls** to GitHub. Ensure you have internet connection.

## Integration Testing

### Test Agent Chain

Create `tests/test_agent_chain.py`:

```python
"""
Integration test for agent chain.
"""

from tools_agents import (
    rubric_builder,
    resume_reviewer,
    github_validator,  # This is a function, not an agent
    github_reviewer,
    verdict_synthesizer
)

def test_complete_evaluation_flow():
    """Test the complete evaluation pipeline."""
    
    print("="*60)
    print("INTEGRATION TEST: Complete Evaluation Flow")
    print("="*60)
    
    # Step 1: Generate Rubric
    print("\n[Step 1/5] Generating rubric...")
    job_desc = """
    Senior Python Developer
    Required: 5+ years Python, Django, PostgreSQL, Docker, AWS
    """
    rubric = rubric_builder.run(f"Generate rubric:\n\n{job_desc}")
    assert len(rubric) > 500  # Rubric should be detailed
    print("✅ Rubric generated")
    
    # Step 2: Evaluate Resume
    print("\n[Step 2/5] Evaluating resume...")
    resume = """
    Jane Smith
    jane@example.com | github.com/janesmith
    
    EXPERIENCE:
    Senior Backend Developer, TechCorp (2018-2024) - 6 years
    - Architected Django REST APIs serving 1M+ users
    - PostgreSQL database optimization and scaling
    - AWS infrastructure management (EC2, RDS, S3)
    - Docker containerization and Kubernetes deployment
    
    SKILLS: Python (expert), Django, FastAPI, PostgreSQL, Docker, AWS
    EDUCATION: MS Computer Science, Stanford, 2018
    """
    
    conversation = f"Rubric:\n{rubric}\n\nResume:\n{resume}"
    resume_eval = resume_reviewer.run(conversation)
    assert "SCORE:" in resume_eval
    assert "github.com/janesmith" in resume_eval.lower()
    print("✅ Resume evaluated")
    
    # Step 3: Validate GitHub
    print("\n[Step 3/5] Validating GitHub account...")
    github_result = github_validator("janesmith")  # May or may not exist
    assert "status" in github_result
    assert "username" in github_result
    print(f"✅ GitHub validated - Status: {github_result['status']}")
    
    # Step 4: Analyze GitHub (simulated)
    if github_result["status"] == "PASSED":
        print("\n[Step 4/5] Analyzing GitHub profile...")
        full_conversation = f"{conversation}\n\nResume Eval:\n{resume_eval}\n\nGitHub:\n{github_result}"
        github_eval = github_reviewer.run(full_conversation)
        assert "GITHUB PROFILE ANALYSIS" in github_eval
        print("✅ GitHub analyzed")
    else:
        print(f"\n[Step 4/5] Skipping GitHub analysis - Status: {github_result['status']}")
        github_eval = "GitHub analysis skipped"
    
    # Step 5: Generate Verdict
    print("\n[Step 5/5] Generating final verdict...")
    verdict_conversation = f"{conversation}\n\nResume Eval:\n{resume_eval}\n\nGitHub:\n{github_result}\n\nGitHub Eval:\n{github_eval}"
    verdict = verdict_synthesizer.run(verdict_conversation)
    assert "FINAL HIRING VERDICT" in verdict
    assert ("HIRE" in verdict or "NO HIRE" in verdict)
    print("✅ Verdict generated")
    
    print("\n" + "="*60)
    print("🎉 INTEGRATION TEST PASSED - All agents work together!")
    print("="*60)
    
    return {
        "rubric": rubric,
        "resume_eval": resume_eval,
        "github_result": github_result,
        "github_eval": github_eval,
        "verdict": verdict
    }

if __name__ == "__main__":
    result = test_complete_evaluation_flow()
    
    # Print final verdict
    print("\n" + "="*60)
    print("FINAL VERDICT:")
    print("="*60)
    print(result["verdict"][:1000] + "...")
```

Run it:

```bash
python tests/test_agent_chain.py
```

## End-to-End Testing

### Test the Root Orchestrator

Create `tests/test_root_orchestrator.py`:

```python
"""
End-to-end test for root orchestrator.
"""

from agent import root_agent

def test_conversational_flow():
    """Test the conversational orchestrator."""
    
    print("="*60)
    print("E2E TEST: Root Orchestrator Conversational Flow")
    print("="*60)
    
    # Turn 1: Greeting
    print("\n[Turn 1] User: Hello")
    response1 = root_agent.run("Hello")
    assert "hiring" in response1.lower() or "assistant" in response1.lower()
    assert "job description" in response1.lower()
    print(f"✅ Greeting response received ({len(response1)} chars)")
    
    # Turn 2: Provide Job Description
    print("\n[Turn 2] User provides job description")
    jd = """
    We're hiring a Senior Python Developer.
    Requirements:
    - 5+ years Python development
    - Django framework expertise
    - PostgreSQL database experience
    - Docker and AWS skills
    Education: BS Computer Science
    """
    response2 = root_agent.run(jd)
    assert "rubric" in response2.lower()
    print(f"✅ JD processed, rubric generated ({len(response2)} chars)")
    
    # Turn 3: Provide Resume
    print("\n[Turn 3] User provides resume")
    resume = """
    Alex Johnson
    alex@email.com
    
    EXPERIENCE:
    Senior Developer, StartupCo (2018-2024) - 6 years
    - Built Django applications
    - PostgreSQL database design
    - Deployed on AWS with Docker
    
    SKILLS: Python, Django, PostgreSQL, Docker, AWS
    EDUCATION: BS CS, UC Berkeley, 2018
    """
    response3 = root_agent.run(resume)
    assert "score" in response3.lower() or "evaluation" in response3.lower()
    print(f"✅ Resume evaluated ({len(response3)} chars)")
    
    # Turn 4: Request Verdict
    print("\n[Turn 4] User requests final verdict")
    response4 = root_agent.run("Please generate a final hiring verdict")
    assert "verdict" in response4.lower() or "hire" in response4.lower()
    print(f"✅ Verdict generated ({len(response4)} chars)")
    
    print("\n" + "="*60)
    print("🎉 E2E TEST PASSED - Complete workflow successful!")
    print("="*60)

if __name__ == "__main__":
    test_conversational_flow()
```

Run it:

```bash
python tests/test_root_orchestrator.py
```

## Common Issues and Debugging

### Issue 1: API Key Not Found

**Error:**

```
KeyError: 'GOOGLE_API_KEY'
```

**Solution:**

```bash
# Check if .env file exists
ls -la .env

# Verify contents
cat .env

# Should contain:
GOOGLE_API_KEY=your_key_here
MODEL_NAME=gemini-2.0-flash-exp

# Reload environment
source venv/bin/activate
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print(os.getenv('GOOGLE_API_KEY'))"
```

### Issue 2: Module Import Errors

**Error:**

```
ModuleNotFoundError: No module named 'google.adk'
```

**Solution:**

```bash
# Ensure virtual environment is activated
source venv/bin/activate

# Reinstall dependencies
pip install -r requirements.txt

# Verify installation
python -c "from google.adk.agents import LlmAgent; print('✓ ADK installed')"
```

### Issue 3: Agent Not Calling Tools

**Symptom:** Root agent responds but doesn't call sub-agents

**Debug steps:**

```python
# Add debug logging to agent.py
import logging
logging.basicConfig(level=logging.DEBUG)

# Check tool registration
print(f"Root agent has {len(root_agent.tools)} tools")
for tool in root_agent.tools:
    print(f"  - {tool.name}")

# Test tool call directly
from agent import rubric_tool
result = rubric_tool.agent.run("Generate rubric for Python Developer role")
print(result)
```

### Issue 4: GitHub API Rate Limit

**Error:**

```
{"status": "WARNING", "error": "GitHub API rate limit exceeded"}
```

**Solution:**

```bash
# Add GitHub token to .env
echo "GITHUB_TOKEN=your_github_token" >> .env

# Verify rate limit status
curl -H "Authorization: token YOUR_TOKEN" https://api.github.com/rate_limit

# Without token: 60 requests/hour
# With token: 5000 requests/hour
```

### Issue 5: Agent Output Not Formatted

**Symptom:** Agents return plain text instead of structured markdown

**Fix:** Check agent instructions for output format specifications:

```python
instruction="""
...
## OUTPUT FORMAT:
- Use markdown formatting (##, **, -)
- Do NOT use code fences (````)
- Start directly with heading
...
"""
```

## Performance Testing

### Measure Agent Response Times

Create `tests/test_performance.py`:

```python
"""
Performance testing for agents.
"""

import time
from tools_agents import rubric_builder, resume_reviewer

def measure_agent_performance():
    """Measure agent response times."""
    
    job_desc = "Senior Python Developer - 5+ years, Django, PostgreSQL"
    
    # Test RubricBuilder
    start = time.time()
    rubric = rubric_builder.run(f"Generate rubric:\n\n{job_desc}")
    rubric_time = time.time() - start
    
    print(f"RubricBuilder: {rubric_time:.2f}s")
    
    # Test ResumeReviewer
    resume = "Sample resume with Python, Django experience..."
    conversation = f"Rubric:\n{rubric}\n\nResume:\n{resume}"
    
    start = time.time()
    evaluation = resume_reviewer.run(conversation)
    resume_time = time.time() - start
    
    print(f"ResumeReviewer: {resume_time:.2f}s")
    print(f"\nTotal: {rubric_time + resume_time:.2f}s")
    
    # Performance thresholds
    assert rubric_time < 30, "RubricBuilder too slow"
    assert resume_time < 30, "ResumeReviewer too slow"
    
    print("\n✅ Performance within acceptable limits")

if __name__ == "__main__":
    measure_agent_performance()
```

**Expected times:**

- RubricBuilder: 5-15 seconds
- ResumeReviewer: 10-20 seconds
- Total pipeline: 30-60 seconds

## Summary

In this step, you:

- ✓ Created **unit tests** for individual agents
- ✓ Built **integration tests** for agent chains
- ✓ Implemented **end-to-end tests** for the orchestrator
- ✓ Learned **debugging techniques** for common issues
- ✓ Measured **performance** of the system

## What's Next?

In the final step, we'll wrap up the tutorial with a conclusion, discuss deployment strategies, and explore potential enhancements!

---

**Previous:** [← Step 7 - Building the Root Orchestrator](step_7.md) | **Next:** [Step 9 - Conclusion →](step_9.md)

