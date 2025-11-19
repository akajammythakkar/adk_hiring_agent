# Step 5: Implementing Resume and GitHub Evaluation

**Level:** Intermediate

## Building the Core Evaluation Agents

In this step, we'll create three critical agents: the **ResumeReviewer**, **GitHubValidator**, and **GitHubReviewer**. These agents form the core of our candidate assessment pipeline.

## Part 1: Resume Reviewer Agent

### Purpose and Workflow

The ResumeReviewer agent:

1. Reads the evaluation rubric from conversation history
2. Analyzes the candidate's resume line-by-line
3. Scores each criterion with evidence
4. Extracts GitHub information
5. Provides a PASS/FAIL recommendation

### Building the ResumeReviewer

Add this to `tools_agents.py`:

```python
# Resume Reviewer - takes resume and rubric from conversation
resume_reviewer = LlmAgent(
    name="ResumeReviewer",
    model=GEMINI_MODEL,
    description="Evaluates candidate resume against the rubric.",
    instruction="""
You are a senior technical recruiter with 10+ years of experience evaluating 
engineering candidates.

## YOUR TASK:
Perform a thorough, evidence-based Level 1 resume evaluation using the rubric 
from the conversation history.

## EVALUATION PROCESS:

1. **Carefully review** the rubric criteria from earlier in conversation
2. **Analyze the resume** line-by-line, section-by-section
3. **Score each criterion** based strictly on the rubric scoring guide
4. **Cite specific evidence** - quote or reference exact resume phrases
5. **Be objective** - no assumptions, only what's explicitly stated
6. **Extract metadata** - candidate name, years of experience, GitHub links

## OUTPUT FORMAT:

**CANDIDATE:** [Extract full name from resume]

**SCORE: X/10**

**DETAILED BREAKDOWN:**

**1. Technical Skills Match: X/4 points**
- Required skills found: [list with resume citations]
- Required skills missing: [list what's not mentioned]
- Evidence quality: [superficial mentions vs. detailed project work]
- Justification: [explain the score based on rubric criteria]

**2. Experience Level: X/3 points**
- Total years: [calculate from resume dates]
- Relevant years: [years in similar roles/technologies]
- Progression: [junior → mid → senior trajectory observed?]
- Justification: [explain score]

**3. Education & Qualifications: X/2 points**
- Degree(s): [list with institutions and years]
- Certifications: [list if any]
- Alignment: [how well it matches requirements]
- Justification: [explain score]

**4. Role Relevance & Domain Fit: X/1 point**
- Previous roles: [list titles and companies]
- Domain match: [industry/sector alignment]
- Level match: [seniority alignment]
- Justification: [explain score]

**KEY STRENGTHS:**
- [Strength 1 with specific resume citation]
- [Strength 2 with citation]
- [Strength 3 with citation]

**CRITICAL GAPS:**
- [Gap 1 - be specific about what's missing or weak]
- [Gap 2 - cite evidence of absence]
- [Gap 3 - mention any red flags]

**GITHUB INFORMATION:**
- GitHub URL/Username: [extract if present, otherwise "Not provided"]
- Source: [where in resume it was found]

**RECOMMENDATION:** 
**[YES/NO] - Proceed to GitHub Evaluation**

**Rationale:** [1-2 sentences explaining whether candidate meets baseline 
requirements. Be decisive - if score is ≥7/10 typically recommend YES, 
<7 recommend NO unless compensating factors]

## CRITICAL RULES:

1. **Only score based on what's in the resume** - no assumptions
2. **Cite evidence** - every score should reference specific resume content
3. **Use the rubric** - follow the scoring guide exactly as defined
4. **Be thorough** - check every section
5. **Extract GitHub** - search for github.com links, @username patterns
6. **Calculate accurately** - if rubric says "5+ years" and resume shows 4, score accordingly
7. **Note ambiguities** - if dates are unclear, mention in gaps

Return the complete evaluation following the format above.
""",
)
```

### Key Features of ResumeReviewer

**Evidence-Based Scoring:**

The agent must cite specific resume content:

```markdown
**1. Technical Skills Match: 4/4 points**
- Required skills found: 
  * Python (mentioned in 3 positions: "Python developer at XYZ Corp")
  * Django ("Built REST APIs using Django framework")
  * PostgreSQL ("Optimized PostgreSQL queries, reduced latency by 40%")
```

**Careful Date Calculations:**

```markdown
**2. Experience Level: 3/3 points**
- Total years: 6 years (2018-2024)
  * XYZ Corp: 2022-2024 (2 years)
  * ABC Inc: 2018-2022 (4 years)
- Relevant years: All 6 years in backend Python development
- Progression: Junior → Mid → Senior trajectory observed
```

**GitHub Extraction:**

```markdown
**GITHUB INFORMATION:**
- GitHub URL/Username: github.com/johndoe
- Source: Found in "Contact" section of resume
```

## Part 2: GitHub Validator

### Purpose and Workflow

The GitHubValidator performs **real-time account verification**:

1. Extracts username from URL/text
2. Calls GitHub REST API
3. Returns account status and metrics
4. Recommends proceed/skip

### Option A: Python Function (Direct API Call)

Add this function to `tools_agents.py`:

```python
# GitHub Validator - validates account exists using REST API
def github_validator(username: str) -> dict:
    """
    Validates GitHub account existence and retrieves basic profile information 
    using GitHub REST API.
    
    Args:
        username: GitHub username (can be URL, @username, or plain username)
    
    Returns:
        dict with validation results including status, user data, and recommendations
    """
    import re
    import requests
    from datetime import datetime
    
    # Extract username from various formats
    username = username.strip()
    
    # Handle github.com/username URLs
    if "github.com/" in username:
        match = re.search(r'github\.com/([a-zA-Z0-9-]+)', username)
        if match:
            username = match.group(1)
    
    # Handle @username format
    if username.startswith('@'):
        username = username[1:]
    
    # Remove trailing slashes or query params
    username = username.split('?')[0].rstrip('/')
    
    # Validate username format
    format_valid = bool(re.match(r'^[a-zA-Z0-9]([a-zA-Z0-9-]{0,37}[a-zA-Z0-9])?$', username))
    
    if not format_valid:
        return {
            "status": "FAILED",
            "username": username,
            "format_valid": False,
            "exists": False,
            "error": "Invalid username format",
            "recommendation": "❌ Skip GitHub analysis - Invalid username format"
        }
    
    # Call GitHub REST API
    try:
        api_url = f"https://api.github.com/users/{username}"
        headers = {
            "Accept": "application/vnd.github.v3+json",
            "User-Agent": "GitHub-Profile-Validator"
        }
        
        # Add token if available (for higher rate limits)
        github_token = os.getenv("GITHUB_TOKEN")
        if github_token:
            headers["Authorization"] = f"token {github_token}"
        
        response = requests.get(api_url, headers=headers, timeout=10)
        
        if response.status_code == 200:
            user_data = response.json()
            
            # Extract relevant information
            public_repos = user_data.get('public_repos', 0)
            followers = user_data.get('followers', 0)
            following = user_data.get('following', 0)
            created_at = user_data.get('created_at', '')
            
            # Calculate account age
            account_age_years = 0
            if created_at:
                created_date = datetime.strptime(created_at, '%Y-%m-%dT%H:%M:%SZ')
                account_age_years = (datetime.now() - created_date).days / 365.25
            
            # Determine status and recommendation
            if public_repos == 0:
                status = "WARNING"
                recommendation = "⚠️ Proceed with caution - No public repositories"
                assessment = "Account exists but has no public repositories."
            elif public_repos < 5 and account_age_years > 1:
                status = "WARNING"
                recommendation = "⚠️ Proceed with caution - Limited repository activity"
                assessment = f"Only {public_repos} public repos despite {account_age_years:.1f} year old account."
            else:
                status = "PASSED"
                recommendation = "✅ Proceed with GitHub analysis"
                assessment = f"Active account with {public_repos} public repositories."
            
            return {
                "status": status,
                "username": username,
                "format_valid": True,
                "exists": True,
                "public_repos": public_repos,
                "followers": followers,
                "following": following,
                "account_age_years": round(account_age_years, 1),
                "assessment": assessment,
                "recommendation": recommendation,
                "profile_url": f"https://github.com/{username}"
            }
        
        elif response.status_code == 404:
            return {
                "status": "FAILED",
                "username": username,
                "format_valid": True,
                "exists": False,
                "error": "GitHub account not found",
                "recommendation": "❌ Skip GitHub analysis - Account does not exist"
            }
        
        elif response.status_code == 403:
            return {
                "status": "WARNING",
                "username": username,
                "error": "GitHub API rate limit exceeded",
                "recommendation": "⚠️ Manual verification needed - API rate limit reached"
            }
        
        else:
            return {
                "status": "WARNING",
                "username": username,
                "error": f"GitHub API error: {response.status_code}",
                "recommendation": "⚠️ Manual verification recommended"
            }
    
    except requests.exceptions.RequestException as e:
        return {
            "status": "WARNING",
            "username": username,
            "error": f"Network error: {str(e)}",
            "recommendation": "⚠️ Manual verification needed"
        }
```

### Which Approach to Use?

**In the actual implementation**, we use **Option A: Python Function** wrapped with `FunctionTool` because:

- ✅ **More efficient**: Direct function call vs. LLM agent
- ✅ **Deterministic**: No LLM interpretation needed
- ✅ **Faster**: No API call to Gemini for validation logic
- ✅ **Cost-effective**: Saves on LLM tokens

The root orchestrator will call this function directly by passing the username as a parameter.

### Option B: Agent with Code Execution (Alternative)

For reference, you could also create an agent that executes Python code:

```python
github_validator_agent = LlmAgent(
    name="GitHubValidatorAgent",
    model=GEMINI_MODEL,
    description="Validates GitHub account existence by calling GitHub REST API.",
    instruction="""
You are a GitHub account validator that uses the GitHub REST API to verify 
accounts IN REAL-TIME.

## YOUR TASK:
Extract the GitHub username from the conversation and validate it by calling 
the REAL GitHub REST API.

## HOW TO VALIDATE:

You have the ability to execute Python code. Use it to call the GitHub REST API:

```python
import requests
import json
from datetime import datetime

username = "EXTRACTED_USERNAME_HERE"

# Clean username
username = username.strip()
if "github.com/" in username:
    username = username.split("github.com/")[-1]
if username.startswith('@'):
    username = username[1:]
username = username.split('?')[0].split('/')[0].rstrip('/')

# Call GitHub API
try:
    response = requests.get(
        f"https://api.github.com/users/{username}",
        headers={"Accept": "application/vnd.github.v3+json"},
        timeout=10
    )
    
    if response.status_code == 200:
        data = response.json()
        print(json.dumps({
            "status": "PASSED",
            "username": data.get("login"),
            "public_repos": data.get("public_repos"),
            "followers": data.get("followers"),
            "created_at": data.get("created_at"),
            "profile_url": data.get("html_url")
        }, indent=2))
    elif response.status_code == 404:
        print(json.dumps({"status": "FAILED", "error": "Account not found"}))
    else:
        print(json.dumps({"status": "WARNING", "error": f"API returned {response.status_code}"}))
except Exception as e:
    print(json.dumps({"status": "ERROR", "error": str(e)}))
```

## OUTPUT FORMAT:

**GITHUB VALIDATION REPORT**

**STATUS:** ✅ PASSED / ⚠️ WARNING / ❌ FAILED

**USERNAME:** [from API]

**ACCOUNT DETAILS:**
- Public Repositories: [count from API]
- Followers: [count from API]
- Account Created: [date from API]

**RECOMMENDATION:**
- ✅ **Proceed with GitHub analysis** - Account is valid
- ⚠️ **Proceed with caution** - [specific concern]
- ❌ **Skip GitHub analysis** - Account not found

**PROFILE URL:** [from API]

## IMPORTANT:

1. ALWAYS execute Python code to get REAL data from GitHub API
2. DO NOT make up or simulate data
3. Present actual API results
""",
)
```

## Part 3: GitHub Reviewer Agent

### Purpose and Workflow

The GitHubReviewer agent:

1. Reads validation results and resume
2. Evaluates portfolio based on rubric
3. Provides simulated analysis (since full repo inspection is resource-intensive)
4. Scores technical capability

### Building the GitHubReviewer

Add this to `tools_agents.py`:

```python
# GitHub Reviewer - analyzes GitHub profile
github_reviewer = LlmAgent(
    name="GitHubReviewer",
    model=GEMINI_MODEL,
    description="Analyzes candidate's GitHub profile.",
    instruction="""
You are a senior software engineer and technical lead with extensive experience 
evaluating code quality and developer portfolios.

## YOUR TASK:
Perform a comprehensive Level 2 GitHub evaluation based on the rubric, 
validation results, and job requirements from the conversation history.

## EVALUATION APPROACH:

Since you cannot actually access GitHub repositories, provide a **realistic 
simulated analysis** based on:
1. The candidate's experience level (from resume)
2. The technologies they claim to know
3. The validation report (repos count, account age)
4. Industry standards for similar roles

## SCORING CRITERIA (Use rubric from conversation):

### 1. Repository Quality (0-3 points)

**Evaluate based on expectations:**
- **Senior developers (5+ years)**: Expect 20-50+ repos with 3-5 substantial projects
- **Mid-level (2-5 years)**: Expect 10-30 repos with 2-3 solid projects
- **Junior (<2 years)**: Expect 5-15 repos with learning projects

**Score: X/3** - Justify based on alignment with claimed experience

### 2. Technology Stack Alignment (0-3 points)

**Match to Job Requirements:**
- List required technologies from JD
- Expected repository evidence:
  * 3 points: Multiple repos showcasing core technologies
  * 2 points: Some relevant tech demonstrated
  * 1 point: Limited relevant technology evidence
  * 0 points: No alignment with required stack

**Score: X/3** - Justify

### 3. Activity & Consistency (0-2 points)

**Expected Patterns:**
- Active developers: Regular commits, recent activity
- Full-time employed devs may have less public activity

**Score: X/2** - Justify

### 4. Documentation & Testing (0-2 points)

**Professional Indicators:**
- READMEs with clear instructions
- Code comments
- Test files or CI badges

**Score: X/2** - Justify

## OUTPUT FORMAT:

**GITHUB PROFILE ANALYSIS**

**CANDIDATE:** [name from resume]
**GITHUB USERNAME:** [from conversation]

**OVERALL SCORE: X/10**

---

**1. Repository Quality: X/3 points**

📊 **Expected Profile Characteristics:**
- [What you'd expect for this experience level]

💡 **Simulated Observations:**
- [Based on validation + resume]

✅/❌ **Assessment:**
- [Score justification]

---

**2. Technology Stack Alignment: X/3 points**

🎯 **Required Technologies** (from JD):
- [List key tech]

💡 **Expected Portfolio Content:**
- [What projects would demonstrate these skills]

✅/❌ **Assessment:**
- [Score justification]

---

**3. Activity & Consistency: X/2 points**

💡 **Expected Activity:**
- [For this experience level]

✅/❌ **Assessment:**
- [Score justification]

---

**4. Documentation & Testing: X/2 points**

💡 **Expected Standards:**
- [For this seniority level]

✅/❌ **Assessment:**
- [Score justification]

---

**RECOMMENDATION:** 
**[PASS/FAIL] for GitHub Screen**

**Rationale:** [2-3 sentences. Typically PASS if ≥6/10, FAIL if <6/10]

---

⚠️ **IMPORTANT DISCLAIMER:**

This analysis is **simulated** based on resume claims and validation data.

**For production use, you should:**
- Manually inspect top 3-5 repositories
- Review actual code quality and architecture
- Verify contribution authenticity

Return the complete analysis following the format above.
""",
)
```

### Why Simulated Analysis?

**Reasons:**

1. **API Rate Limits**: GitHub API has strict limits on repository access
2. **Processing Time**: Full repository analysis requires extensive API calls
3. **Privacy Concerns**: Some repos may be private or sensitive
4. **Resource Cost**: Deep code analysis is computationally expensive

**Solution:**

Use validation data + resume experience to make **informed estimates** based on industry norms.

## Testing the Evaluation Agents

### Test Script for Resume Evaluation

Create `test_resume_eval.py`:

```python
from tools_agents import rubric_builder, resume_reviewer

# Step 1: Generate rubric
job_desc = """
Senior Python Developer - 5+ years experience
Required: Python, Django, PostgreSQL, Docker, AWS
"""

rubric = rubric_builder.run(f"Generate rubric for:\n\n{job_desc}")
print("RUBRIC GENERATED:\n")
print(rubric)
print("\n" + "="*60 + "\n")

# Step 2: Evaluate resume
resume = """
John Doe
john@example.com | github.com/johndoe

EXPERIENCE:
Senior Backend Engineer, TechCorp (2019-2024)
- Built REST APIs using Django and FastAPI
- Optimized PostgreSQL queries, reduced latency by 40%
- Deployed microservices on AWS using Docker and Kubernetes

SKILLS: Python, Django, PostgreSQL, Docker, AWS, React

EDUCATION: BS Computer Science, MIT, 2019
"""

evaluation = resume_reviewer.run(
    f"Previous conversation:\n{rubric}\n\nResume to evaluate:\n\n{resume}"
)

print("RESUME EVALUATION:\n")
print(evaluation)
```

Run it:

```bash
python test_resume_eval.py
```

## Summary

In this step, you built:

- ✓ **ResumeReviewer** - Evidence-based resume scoring
- ✓ **GitHub Validator** - Real-time account verification via REST API
- ✓ **GitHubReviewer** - Technical portfolio assessment

These three agents form the **evaluation pipeline** that assesses candidates objectively.

## What's Next?

Next, we'll build the **VerdictSynthesizer** agent that combines all evaluation data to provide a final HIRE/NO HIRE recommendation!

---

**Previous:** [← Step 4 - Building the Rubric Builder Agent](step_4.md) | **Next:** [Step 6 - Creating the Verdict Synthesizer →](step_6.md)

