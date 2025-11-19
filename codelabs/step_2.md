# Step 2: Setting Up the Environment

**Level:** Beginner

## Environment Setup and Dependencies

In this step, we'll set up your development environment, install necessary dependencies, and configure API access for the AI hiring agent system.

## Installing Python and Dependencies

### Step 1: Verify Python Installation

Ensure you have Python 3.8 or higher installed:

```bash
python --version
# or
python3 --version
```

If Python is not installed, download it from [python.org](https://www.python.org/downloads/).

### Step 2: Create Project Directory

Create a new directory for your project:

```bash
mkdir conversational_hiring
cd conversational_hiring
```

### Step 3: Create a Virtual Environment (Recommended)

Using a virtual environment isolates your project dependencies:

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On macOS/Linux:
source venv/bin/activate

# On Windows:
venv\Scripts\activate
```

Your terminal prompt should now show `(venv)` indicating the virtual environment is active.

### Step 4: Create Requirements File

Create a `requirements.txt` file with the following dependencies:

```txt
google-genai
python-dotenv
requests>=2.31.0
```

**Dependency breakdown:**

- **google-genai**: Google's ADK (Agent Development Kit) for building AI agents
- **python-dotenv**: Loads environment variables from `.env` files
- **requests**: HTTP library for making REST API calls (GitHub validation)

### Step 5: Install Dependencies

Install all required packages:

```bash
pip install -r requirements.txt
```

You should see output indicating successful installation of each package.

## Configuring API Access

### Step 1: Obtain Gemini API Key

You need a Gemini API key to use Google's generative AI models:

1. Visit [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Sign in with your Google account
3. Click "Create API Key"
4. Copy your API key (keep it secure!)

**Alternative:** If using Google Cloud, you can also authenticate via service accounts.

### Step 2: (Optional) Obtain GitHub Token

While not required, a GitHub token increases API rate limits:

1. Go to [GitHub Settings → Developer Settings → Personal Access Tokens](https://github.com/settings/tokens)
2. Click "Generate new token (classic)"
3. Give it a descriptive name (e.g., "Hiring Agent Validator")
4. Select scopes: **public_repo** (or leave all unchecked for public data only)
5. Click "Generate token"
6. Copy the token immediately (you won't see it again!)

### Step 3: Create Environment Variables File

Create a `.env` file in your project root:

```bash
touch .env
```

Add your API credentials:

```env
# Required: Gemini API Configuration
MODEL_NAME=gemini-2.0-flash-exp
GOOGLE_API_KEY=your_gemini_api_key_here

# Optional: GitHub Token (for higher rate limits)
GITHUB_TOKEN=your_github_token_here
```

**Important:** Replace `your_gemini_api_key_here` with your actual API key!

### Step 4: Secure Your Credentials

Add `.env` to `.gitignore` to prevent accidentally committing secrets:

```bash
echo ".env" >> .gitignore
echo "venv/" >> .gitignore
echo "__pycache__/" >> .gitignore
```

## Project Structure Setup

### Create Initial Project Files

Create the basic file structure:

```bash
# Create main Python files
touch __init__.py
touch agent.py
touch tools_agents.py

# Verify structure
ls -la
```

Create the `__init__.py` file with package exports:

```python
"""
Conversational Hiring Agent Package
"""

from .agent import root_agent

__all__ = ["root_agent"]
```

This allows you to import the root agent easily:

```python
# Can now do:
from conversational_hiring import root_agent

# Instead of:
from conversational_hiring.agent import root_agent
```

Your project directory should now look like this:

```
conversational_hiring/
├── .env                  # API keys (not committed to git)
├── .gitignore           # Files to ignore in git
├── __init__.py          # Python package exports
├── agent.py             # Root orchestrator agent
├── tools_agents.py      # Specialized sub-agents
├── requirements.txt     # Python dependencies
└── venv/               # Virtual environment (if created)
```

## Verifying Installation

### Test Google ADK Import

Create a test file to verify installation:

```python
# test_setup.py
from google.adk.agents import LlmAgent
from dotenv import load_dotenv
import os
import requests

# Load environment variables
load_dotenv()

# Check if API key is loaded
api_key = os.getenv("GOOGLE_API_KEY")
model_name = os.getenv("MODEL_NAME")

print("✓ google-genai imported successfully")
print("✓ python-dotenv imported successfully")
print("✓ requests imported successfully")

if api_key:
    print(f"✓ GOOGLE_API_KEY loaded (length: {len(api_key)})")
else:
    print("✗ GOOGLE_API_KEY not found in .env")

if model_name:
    print(f"✓ MODEL_NAME: {model_name}")
else:
    print("⚠ MODEL_NAME not set, will need to specify manually")

github_token = os.getenv("GITHUB_TOKEN")
if github_token:
    print(f"✓ GITHUB_TOKEN loaded (optional)")
else:
    print("⚠ GITHUB_TOKEN not set (optional, but recommended)")

print("\n✓ Environment setup complete!")
```

Run the test:

```bash
python test_setup.py
```

Expected output:

```
✓ google-genai imported successfully
✓ python-dotenv imported successfully
✓ requests imported successfully
✓ GOOGLE_API_KEY loaded (length: 39)
✓ MODEL_NAME: gemini-2.0-flash-exp
✓ GITHUB_TOKEN loaded (optional)

✓ Environment setup complete!
```

## Understanding the Model Selection

### Gemini Model Options

The `MODEL_NAME` can be set to various Gemini models:

- **gemini-2.0-flash-exp**: Fast, efficient, good for production
- **gemini-1.5-pro**: More powerful, slower, better for complex reasoning
- **gemini-1.5-flash**: Balanced performance and cost

For this tutorial, we use **gemini-2.0-flash-exp** for optimal performance.

## Troubleshooting Common Issues

### Issue: "ModuleNotFoundError: No module named 'google.adk'"

**Solution:** Ensure you've activated your virtual environment and installed dependencies:

```bash
source venv/bin/activate  # Activate venv
pip install -r requirements.txt
```

### Issue: "API key not found"

**Solution:** Check your `.env` file:

1. File is in the project root directory
2. Variable names match exactly: `GOOGLE_API_KEY`, `MODEL_NAME`
3. No spaces around `=` signs
4. No quotes around values (unless part of the key)

### Issue: "Permission denied" when running Python

**Solution:** Ensure Python has execute permissions:

```bash
chmod +x venv/bin/python  # macOS/Linux
```

## Summary

You've successfully:

- ✓ Installed Python and created a virtual environment
- ✓ Installed all required dependencies (google-genai, python-dotenv, requests)
- ✓ Obtained and configured API keys (Gemini, optionally GitHub)
- ✓ Created the project structure
- ✓ Verified the installation

## What's Next?

In the next step, we'll dive into **multi-agent architecture concepts** and understand how specialized agents work together to create a powerful hiring evaluation system.

---

**Previous:** [← Step 1 - Introduction](step_1.md) | **Next:** [Step 3 - Understanding Sub-Agents →](step_3.md)

