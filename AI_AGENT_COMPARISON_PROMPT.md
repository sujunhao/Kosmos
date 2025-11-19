# AI Coding Agent Bug Fix Challenge

## Your Task
You are participating in a comparative evaluation of AI coding agents' bug-fixing capabilities. Your objective is to fix as many bugs as possible from the provided bug list in the Kosmos AI Scientist codebase.

## Instructions

### 1. Create Your Unique Branch
First, create a unique branch for your work:
```bash
git checkout -b bugfix-[MODEL_NAME]-[TIMESTAMP]
```
Replace:
- `[MODEL_NAME]` with your identifier (e.g., `claude-opus`, `gpt4`, `claude-sonnet`, `gemini`, `deepseek`, etc.)
- `[TIMESTAMP]` with current timestamp (e.g., `20251118-1430`)

Example: `bugfix-claude-opus-20251118-1430`

### 2. Review the Bug List
The complete bug list is in: `UNIFIED_BUG_LIST.md`
- Contains 60+ execution-blocking bugs
- Organized by severity: CRITICAL → HIGH → TEST FIXTURES → MEDIUM
- Each bug includes file path, line numbers, error description, and suggested fix

### 3. Fix the Bugs
**Priority Order:**
1. Start with **Bug #1** (Pydantic V2 Configuration) - MUST fix first as it blocks all startup
2. Fix other CRITICAL bugs (they prevent basic functionality)
3. Fix HIGH severity bugs (common path failures)
4. Fix TEST FIXTURE bugs (enable tests to run)
5. Fix MEDIUM severity bugs (improve stability)

**Guidelines:**
- Make atomic commits for each bug or related group of bugs
- Include the bug number in commit messages (e.g., "Fix #1: Pydantic V2 config parsing")
- Write clear commit messages explaining what was fixed
- Do NOT introduce new dependencies unless absolutely necessary
- Ensure fixes don't create regressions

### 4. Verify Your Fixes
After each group of fixes, verify:
```bash
# Check if application starts (after fixing Bug #1)
python -m kosmos.cli.main doctor

# Run integration tests
pytest tests/integration/ -v --tb=short

# Run unit tests
pytest tests/unit/ -v

# Check overall test coverage
pytest tests/ --cov=kosmos --cov-report=term-missing

# Run specific test files affected by your fixes
pytest tests/integration/test_analysis_pipeline.py -v
```

### 5. Track Your Progress
Create a file `BUGFIX_REPORT_[MODEL_NAME].md` documenting:
```markdown
# Bug Fix Report - [MODEL_NAME]

## Summary
- Bugs attempted: X/60
- Bugs successfully fixed: Y/60
- Tests passing: Z% (baseline: 57.4%)
- Code coverage: A% (baseline: 22.77%)
- Time taken: H hours M minutes

## Fixed Bugs
- Bug #1: ✅ Fixed - [brief description]
- Bug #2: ✅ Fixed - [brief description]
- Bug #3: ❌ Attempted - [reason for failure]
...

## Test Results
### Before
- Integration tests: 81/141 passing (57.4%)
- Coverage: 22.77%

### After
- Integration tests: X/141 passing (Y%)
- Coverage: Z%

## Challenges Encountered
[List any significant challenges or blockers]

## Additional Improvements
[List any additional fixes or improvements made beyond the bug list]
```

### 6. Success Metrics
You will be evaluated on:
1. **Bug Fix Count**: Total number of bugs successfully resolved
2. **Test Pass Rate**: Improvement from 57.4% baseline
3. **Code Coverage**: Improvement from 22.77% baseline
4. **Fix Quality**: No regressions, proper error handling
5. **Code Quality**: Clean, maintainable solutions
6. **Completeness**: How thoroughly each bug was addressed

### 7. Rules
- Work independently in your branch
- Do NOT look at other agents' branches or solutions
- No time limit - work until you've fixed as many bugs as possible
- Focus on quantity AND quality of fixes
- Document any bugs you couldn't fix and why

### 8. Final Submission
When complete:
1. Ensure all changes are committed to your branch
2. Run final test suite and document results
3. Create your `BUGFIX_REPORT_[MODEL_NAME].md` file
4. Make a final commit with message: "Complete: [X/60] bugs fixed, [Y]% tests passing"

## Starting Context
- **Repository**: Kosmos AI Scientist v0.2.0
- **Language**: Python 3.11/3.12
- **Current State**: 57.4% tests passing, 22.77% coverage
- **Critical Issue**: Application won't start due to Pydantic V2 config bug
- **Bug List**: See `UNIFIED_BUG_LIST.md` for complete details

## Begin
Good luck! Your performance will be compared against other AI coding agents working on the identical bug list. Focus on demonstrating your debugging skills, code comprehension, and ability to produce working fixes efficiently.

Start by creating your branch and fixing Bug #1 to unblock the application startup.