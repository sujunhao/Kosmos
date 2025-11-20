# Kosmos E2E Testing Plan

**Status:** 🟢 Testing Framework Operational
**Provider:** DeepSeek API
**Budget:** $20 (~140-200 test runs)
**Last Updated:** 2025-11-20

---

## ✅ Completed Tests

### Test 1: Biology Research Workflow
- **Status:** ✅ PASSED (21.95s)
- **Question:** "How does temperature affect enzyme activity in metabolic pathways?"
- **Result:** Successfully generated research plan and initialized workflow
- **Cost:** ~$0.10-0.15
- **Commit:** 097fa5f

---

## 🎯 Next Tests to Run (Priority Order)

### Phase 1: Domain Validation (Run Next)

#### Test 2: Neuroscience Research Workflow
```bash
pytest tests/e2e/test_full_research_workflow.py::TestNeuroscienceResearchWorkflow::test_full_neuroscience_workflow -v -s --no-cov
```
- **Question:** "What neural pathways are involved in memory consolidation?"
- **Expected:** Similar to biology test (research_started, hypothesis generation)
- **Budget:** ~$0.10-0.15

#### Test 3: Multi-Iteration Research Cycle
**TODO:** Enhance test to run 2-3 full iterations
```python
# Add to test_full_research_workflow.py
def test_multi_iteration_biology_workflow():
    """Test complete research cycle with multiple iterations."""
    config = {"max_iterations": 3}
    # Execute multiple step() calls to advance workflow
    for i in range(3):
        result = director.execute({"action": "step"})
        assert result["status"] == "step_executed"
```
- **Expected:** Workflow progresses through states (hypothesis → design → execute → analyze)
- **Budget:** ~$0.30-0.50

---

### Phase 2: Integration Testing

#### Test 4: CLI Workflow Integration
```bash
kosmos run "Does caffeine improve cognitive performance?" --domain neuroscience --max-iterations 2
```
- **Expected:** CLI successfully orchestrates full workflow
- **Validates:** CLI → Agent → LLM integration
- **Budget:** ~$0.20-0.30

#### Test 5: Database Persistence
**TODO:** Verify results are stored and retrievable
```bash
# After running research
kosmos history  # Should show previous run
kosmos status <run-id>  # Should show detailed status
```

---

### Phase 3: Performance Testing

#### Test 6: Cache Effectiveness
```bash
# Run same question twice
pytest tests/e2e/test_full_research_workflow.py::TestBiologyResearchWorkflow::test_full_biology_workflow -v --count=2
```
- **Expected:** Second run should be faster + cheaper (cache hits)
- **Validates:** Prompt caching works

#### Test 7: Parallel Execution
**TODO:** Test concurrent experiment execution
- Enable `enable_concurrent_operations: true`
- Measure speedup vs sequential

---

## 📊 Budget Tracking

| Test | Status | Cost | Time | Notes |
|------|--------|------|------|-------|
| Biology Workflow | ✅ | $0.10-0.15 | 21.95s | First successful E2E |
| Neuroscience Workflow | ⏳ | ~$0.10-0.15 | ~20s | Next to run |
| Multi-Iteration | 📋 | ~$0.30-0.50 | ~60s | Needs enhancement |
| CLI Integration | 📋 | ~$0.20-0.30 | ~30s | Manual test |
| **Total Used** | - | **~$0.12** | **22s** | - |
| **Remaining Budget** | - | **~$19.88** | - | ~130-165 runs left |

---

## 🐛 Known Issues & Workarounds

### Issue 1: Workflow state case sensitivity
- **Status:** ✅ Fixed in 097fa5f
- **Solution:** Test now handles both uppercase and lowercase states

### Issue 2: Provider fallback to Anthropic
- **Status:** ✅ Fixed in 097fa5f
- **Solution:** OpenAI provider now handles both dict and Pydantic config

### Issue 3: Neo4j authentication failure
- **Status:** ⚠️ Warning (non-blocking)
- **Impact:** Graph persistence disabled, but workflow continues
- **Workaround:** Research director falls back to in-memory storage
- **Fix Needed:** Update Neo4j credentials in .env or disable graph features

---

## 🔧 Quick Commands

### Run All E2E Tests
```bash
pytest tests/e2e/ -v -s --no-cov -m e2e
```

### Run Only Fast E2E Tests
```bash
pytest tests/e2e/ -v -s --no-cov -m "e2e and not slow"
```

### Run Single Test with Debug Output
```bash
pytest tests/e2e/test_full_research_workflow.py::TestBiologyResearchWorkflow -v -s --no-cov --tb=short
```

### Check DeepSeek Usage
Visit: https://platform.deepseek.com/usage

---

## 🎯 Success Criteria

### Minimum Viable E2E Coverage
- ✅ 1 biology workflow test passes
- ⏳ 1 neuroscience workflow test passes
- ⏳ 1 multi-iteration test passes
- ⏳ CLI integration works

### Stretch Goals
- Performance benchmarks documented
- Cache effectiveness validated
- All 6 research domains tested
- Docker deployment tested

---

## 🚨 Troubleshooting Guide

### Test Skipped (No API Key)
```bash
# Check .env is loaded
grep OPENAI_API_KEY .env

# Verify pytest loads environment
pytest tests/e2e/ --collect-only | grep "Loaded environment"
```

### DeepSeek API Errors
```python
# Test connection manually
python3 -c "
from kosmos.config import get_config
from kosmos.core.providers.openai import OpenAIProvider
provider = OpenAIProvider(get_config().openai)
print(provider.generate('Hello', max_tokens=5))
"
```

### Workflow Stuck/Hanging
- Check logs: `tail -f logs/kosmos.log`
- Reduce iterations: Set `max_iterations: 1` in test config
- Disable concurrent ops: Set `enable_concurrent_operations: false`

---

## 📝 Next Session Checklist

Before starting next E2E testing session:

1. ✅ Verify DeepSeek balance: https://platform.deepseek.com/usage
2. ✅ Check .env file has correct API key
3. ✅ Git status clean (no uncommitted changes)
4. ✅ Review this plan and pick next test
5. ✅ Run the test!

---

## 🔄 Update History

- **2025-11-20:** Initial plan created after first successful E2E test
- Biology workflow test passed with DeepSeek API
- Documented known issues and workarounds
- Established budget tracking system
