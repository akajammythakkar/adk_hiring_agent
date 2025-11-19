# Codelab Changelog

This file tracks updates made to the codelabs to keep them in sync with code changes.

## Latest Update - GitHub Validator Function Tool

**Date:** Current

**Changes Made:**

### 1. **Step 5 - Resume and GitHub Evaluation**
   - Added explanation that `github_validator` function is preferred over agent approach
   - Highlighted benefits: efficiency, deterministic behavior, faster, cost-effective
   - Kept agent approach as "Option B" for reference

### 2. **Step 7 - Root Orchestrator**
   - Updated imports to include `FunctionTool` from `google.adk.tools`
   - Changed `github_validator_agent` import to `github_validator` (function)
   - Updated tool wrapping to use `FunctionTool(func=github_validator)` instead of `AgentTool`
   - Added explanation of `FunctionTool` vs `AgentTool` differences
   - Added section on "Function Tool Parameters" explaining username parameter requirement
   - Updated example usage to show parameter passing

### 3. **Step 2 - Environment Setup**
   - Added `__init__.py` file content and explanation
   - Showed how package exports work for cleaner imports
   - Updated project structure diagram to clarify `__init__.py` purpose

### 4. **Step 8 - Testing**
   - Added comment clarifying `github_validator` is a function, not an agent
   - Updated test naming to reflect "Function" approach

## Key Architecture Change

**Before:**
```python
github_validator_agent = LlmAgent(...)  # Agent that calls API via code execution
github_validate_tool = AgentTool(agent=github_validator_agent)
```

**After:**
```python
def github_validator(username: str) -> dict:  # Direct Python function
    # ... calls GitHub API ...
    
github_validate_tool = FunctionTool(func=github_validator)  # Wrap function as tool
```

**Benefits:**
- ✅ More efficient (no LLM call for validation logic)
- ✅ Deterministic results
- ✅ Faster execution
- ✅ Lower token costs

**Important:** The root orchestrator must pass the `username` parameter when calling the function tool.

## Files Updated

- `codelabs/step_2.md` - Added __init__.py documentation
- `codelabs/step_5.md` - Added function vs agent comparison
- `codelabs/step_7.md` - Updated tool wrapping and imports
- `codelabs/step_8.md` - Updated test comments
- `codelabs/CHANGELOG.md` - Created this changelog

## Compatibility Notes

The tutorial now matches the actual implementation in:
- `agent.py` (line 6, 12, 24)
- `tools_agents.py` (github_validator function at line 192)
- `__init__.py` (package exports)

All code examples in the tutorial are now consistent with the working codebase.

