# Kosmos Codebase - Comprehensive Code Review Report

**Date:** 2025-11-18
**Reviewer:** Claude Code (Sonnet 4.5)
**Focus:** Execution-blocking bugs

---

## Executive Summary

This code review identified **26 bugs** across the Kosmos codebase that would prevent execution. The issues range from critical import errors to field mismatches between models and their usage in tests. Most issues are concentrated in the integration tests and the interaction between models.

---

## CRITICAL SEVERITY - Would crash immediately on import/startup

### 1. Missing Required Parameters in Database Operation Call

**File:** `kosmos/execution/result_collector.py`
**Line:** 441-448

```python
db_result = db_ops.create_result(
    experiment_id=result.experiment_id,
    data=result_dict,
    statistical_tests={test.test_name: test.model_dump() for test in result.statistical_tests},
    p_value=result.primary_p_value,
    effect_size=result.primary_effect_size,
    supports_hypothesis=result.supports_hypothesis,
    interpretation=result.interpretation
)
```

**Error:** `TypeError: create_result() missing 2 required positional arguments: 'session' and 'id'`

**Why:** The `create_result()` function in `kosmos/db/operations.py` (line 317-328) requires `session: Session` and `id: str` as the first two positional arguments, but the caller doesn't pass them.

---

### 2. Accessing Non-Existent Attribute `is_primary` on StatisticalTestResult

**File:** `kosmos/analysis/summarizer.py`
**Line:** 189

```python
if not test.is_primary:
```

**Error:** `AttributeError: 'StatisticalTestResult' object has no attribute 'is_primary'`

**Why:** The `StatisticalTestResult` model in `kosmos/models/result.py` does not have an `is_primary` field. This attribute does not exist anywhere in the model definition (lines 59-97).

---

### 3. Accessing Non-Existent Attributes `primary_ci_lower` and `primary_ci_upper`

**File:** `kosmos/analysis/summarizer.py`
**Line:** 280

```python
if result.primary_ci_lower is None or result.primary_ci_upper is None:
```

**Error:** `AttributeError: 'ExperimentResult' object has no attribute 'primary_ci_lower'`

**Why:** The `ExperimentResult` model does not have `primary_ci_lower` or `primary_ci_upper` fields. These fields are not defined in `kosmos/models/result.py`.

---

### 4. Calling `.lower()` on Enum Object Instead of String

**File:** `kosmos/execution/code_generator.py`
**Lines:** 65, 139, 154

```python
if 't_test' in test.test_type.lower() or 't-test' in test.test_type.lower():
```

**Error:** `AttributeError: 'StatisticalTest' object has no attribute 'lower'`

**Why:** `test.test_type` is a `StatisticalTest` enum (defined in `kosmos/models/experiment.py` line 217), not a string. Should use `test.test_type.value.lower()`.

---

## HIGH SEVERITY - Would fail on basic operations

### 5. Test Fixture Uses Non-Existent `research_question_id` Field on Hypothesis

**File:** `tests/integration/test_analysis_pipeline.py`
**Line:** 117

```python
return Hypothesis(
    id="hyp-001",
    research_question_id="rq-001",  # This field doesn't exist
    ...
)
```

**Error:** `pydantic.error_wrappers.ValidationError: 1 validation error for Hypothesis - research_question_id - extra fields not permitted`

**Why:** The `Hypothesis` model in `kosmos/models/hypothesis.py` uses `research_question` (line 50), not `research_question_id`.

---

### 6. Test Fixture Uses Non-Existent `experiment_type` String Value

**File:** `tests/integration/test_analysis_pipeline.py`
**Line:** 122

```python
experiment_type="comparative",
```

**Error:** `pydantic.error_wrappers.ValidationError: value is not a valid enumeration member`

**Why:** The `Hypothesis` model does not have an `experiment_type` field. This field belongs to `ExperimentProtocol`, and even there it expects an `ExperimentType` enum, not a string.

---

### 7. Test Fixture Uses Non-Existent `feasibility_score` Field on Hypothesis

**File:** `tests/integration/test_analysis_pipeline.py`
**Line:** 125

```python
feasibility_score=0.8,
```

**Error:** `pydantic.error_wrappers.ValidationError: extra fields not permitted`

**Why:** The `Hypothesis` model does not have a `feasibility_score` field. The existing score fields are `testability_score`, `novelty_score`, `confidence_score`, and `priority_score` (lines 58-61 of hypothesis.py).

---

### 8. Test Fixture Uses Non-Existent `variables` Field on Hypothesis

**File:** `tests/integration/test_analysis_pipeline.py`
**Line:** 126

```python
variables=["treatment", "control", "outcome_Y"],
```

**Error:** `pydantic.error_wrappers.ValidationError: extra fields not permitted`

**Why:** The `Hypothesis` model does not have a `variables` list field. It has `related_papers` and `similar_work` as list fields, but not `variables`.

---

### 9. Test Fixture Uses Non-Existent Fields on ExperimentResult

**File:** `tests/integration/test_analysis_pipeline.py`
**Lines:** 46-47, 106

```python
primary_ci_lower=0.2,
primary_ci_upper=1.1,
...
plots_generated=[],
```

**Error:** `pydantic.error_wrappers.ValidationError: extra fields not permitted`

**Why:** The `ExperimentResult` model does not have `primary_ci_lower`, `primary_ci_upper`, or `plots_generated` fields. The model uses `generated_files` instead of `plots_generated`.

---

### 10. Test Fixture Uses Non-Existent `q1` and `q3` Fields on VariableResult

**File:** `tests/integration/test_analysis_pipeline.py`
**Lines:** 76-77, 89-90

```python
q1=9.1,
q3=11.9,
```

**Error:** `pydantic.error_wrappers.ValidationError: extra fields not permitted`

**Why:** The `VariableResult` model in `kosmos/models/result.py` (lines 100-119) does not have `q1` or `q3` fields. It has `mean`, `median`, `std`, `min`, `max`, `n_samples`, `n_missing`, but no quartile fields.

---

### 11. Test Fixture Uses String for `test_type` Instead of Enum

**File:** `tests/integration/test_execution_pipeline.py`
**Line:** 36

```python
StatisticalTestSpec(
    test_type="t_test",  # Should be StatisticalTest.T_TEST
    ...
)
```

**Error:** `pydantic.error_wrappers.ValidationError: value is not a valid enumeration member`

**Why:** The `StatisticalTestSpec` model expects `test_type: StatisticalTest` (an enum), not a string. The enum is defined at line 24-34 of experiment.py.

---

### 12. Test Fixture Uses Non-Existent Fields on ResourceRequirements

**File:** `tests/integration/test_execution_pipeline.py`
**Lines:** 55-60

```python
resource_requirements=ResourceRequirements(
    estimated_runtime_seconds=300,  # Not a valid field
    cpu_cores=1,                    # Not a valid field
    memory_gb=1,
    storage_gb=0.1                  # Not a valid field
)
```

**Error:** `pydantic.error_wrappers.ValidationError: extra fields not permitted`

**Why:** The `ResourceRequirements` model does not have `estimated_runtime_seconds`, `cpu_cores`, or `storage_gb` fields. It has:
- `compute_hours` (not estimated_runtime_seconds)
- `memory_gb` (exists)
- `data_size_gb` (not storage_gb)
- No cpu_cores field at all

---

### 13. Test Fixture Uses Non-Existent Fields on ExperimentProtocol

**File:** `tests/integration/test_execution_pipeline.py`
**Lines:** 30, 61-63

```python
ExperimentProtocol(
    ...
    title="Integration Test - T-test",              # Not a valid field
    data_requirements={"format": "csv", ...},       # Not a valid field
    expected_duration_minutes=5                     # Not a valid field
)
```

**Error:** `pydantic.error_wrappers.ValidationError: extra fields not permitted`

**Why:** The `ExperimentProtocol` model does not have `title`, `data_requirements`, or `expected_duration_minutes` as top-level fields. The `title` exists only on `ProtocolStep` (line 129), and `expected_duration_minutes` exists only on `ProtocolStep` (line 144).

---

## MEDIUM SEVERITY - Would fail on common use cases

### 14. Same Test Fixture Issues in Unit Tests

**File:** `tests/unit/agents/test_data_analyst.py`
**Lines:** 52-53, 79-80, 92-93, 106, 117, 121, 124, 126

Same issues as items 5-10 but in unit tests:
- `primary_ci_lower`, `primary_ci_upper` (lines 52-53)
- `q1`, `q3` on VariableResult (lines 79-80, 92-93)
- `plots_generated` (line 106)
- `research_question_id` (line 117)
- `experiment_type="comparative"` (line 121)
- `feasibility_score` (line 124)
- `variables=[]` on Hypothesis (line 126)

---

### 15. Same ResourceRequirements Issues in Other Integration Tests

**File:** `tests/integration/test_iterative_loop.py`
**Lines:** 107-112

```python
resource_requirements=ResourceRequirements(
    estimated_runtime_seconds=300,
    cpu_cores=1,
    memory_gb=1,
    storage_gb=0.1
)
```

Same issue as item 12 - these fields don't exist in the model.

---

### 16. Status Comparison Using String Instead of Enum

**File:** `kosmos/analysis/summarizer.py`
**Line:** 269

```python
if result.status != "success":
```

**Potential Error:** This works but is inconsistent - should compare with `ResultStatus.SUCCESS`

**Why:** `result.status` is a `ResultStatus` enum. While string comparison may work due to Python's enum behavior, it's inconsistent and error-prone. Should be `if result.status != ResultStatus.SUCCESS:`.

---

### 17. Potential AttributeError When Accessing `expected_outcome`

**File:** `kosmos/agents/data_analyst.py`
**Line:** 336

```python
Expected Outcome: {getattr(hypothesis, 'expected_outcome', 'Not specified')}
```

**Why:** This is handled with `getattr()` with a default, but it indicates the code expects a field that doesn't exist. The `Hypothesis` model doesn't have an `expected_outcome` field.

---

## LOW SEVERITY - Would fail on less common paths

### 18. ExperimentType Import Location Mismatch

**File:** `kosmos/models/experiment.py`
**Line:** 13

```python
from kosmos.models.hypothesis import ExperimentType
```

**Why:** This creates a dependency from experiment.py to hypothesis.py. While not a bug per se, it's unusual that `ExperimentType` is defined in hypothesis.py but used primarily in experiment.py. Could cause circular import issues if hypothesis.py ever imports from experiment.py.

---

### 19. Potential None Dereference in ResultExport.export_markdown()

**File:** `kosmos/models/result.py`
**Lines:** 337-342

```python
lines.append(
    f"| {var.variable_name} | "
    f"{var.mean:.2f} | {var.median:.2f} | {var.std:.2f} | "
    f"{var.min:.2f} | {var.max:.2f} | {var.n_samples} |"
)
```

**Error:** `TypeError: unsupported format string passed to NoneType.__format__`

**Why:** All these fields (`mean`, `median`, `std`, `min`, `max`) are `Optional[float]` in the `VariableResult` model. If any are None, the `.2f` format will fail.

---

### 20. Validation Error in Hypothesis Statement Check

**File:** `kosmos/models/hypothesis.py`
**Lines:** 85-102

The validator rejects statements ending with `?` but this check may be too strict - some valid hypothesis statements in different languages/formats might end with punctuation that looks like a question mark.

---

### 21. Missing Validation for `test_type` Enum Conversion in Template Matching

**File:** `kosmos/execution/code_generator.py`
**Line:** 65

```python
if 't_test' in test.test_type.lower() or 't-test' in test.test_type.lower():
```

Should be:
```python
if 't_test' in test.test_type.value.lower() or 't-test' in test.test_type.value.lower():
```

---

## Summary of Issues by Category

| Category | Count |
|----------|-------|
| Missing/Wrong Model Fields | 14 |
| Enum vs String Type Errors | 3 |
| Missing Function Arguments | 1 |
| Missing Attribute Access | 3 |
| Potential None Dereference | 1 |
| Import/Dependency Issues | 1 |
| Inconsistent Comparisons | 1 |
| Other | 2 |
| **Total** | **26** |

---

## Recommended Fix Priority

### Phase 1: Critical Source Code Fixes (4 files)
1. `kosmos/execution/result_collector.py` - Add missing session/id arguments
2. `kosmos/analysis/summarizer.py` - Fix `is_primary` and `primary_ci_lower/upper` attribute access
3. `kosmos/execution/code_generator.py` - Add `.value` to enum access (3 locations)

### Phase 2: Test Fixture Alignment (4 test files)
1. Align `Hypothesis` creation with actual model fields
2. Align `ExperimentResult` creation with actual model fields
3. Use enum values instead of strings for `test_type`
4. Fix `ResourceRequirements` and `ExperimentProtocol` fields

### Phase 3: Defensive Improvements
1. Add null checks for optional field formatting in `result.py`
2. Use consistent enum comparisons throughout codebase
