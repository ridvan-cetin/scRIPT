# IPython Notebook Comparison Review

## Files Reviewed
1. `scLTy_002_part001_FIXED.ipynb`
2. `scLTy_002_part001_STREAMLINED (10) (3).ipynb`
3. `scLTy_002_part001_new_version.ipynb`

Excluded: `D457_res1_Q50u_filtered_all_001.ipynb`

## Key Discovery

**STREAMLINED and new_version are identical** — they produce zero diff. The real comparison is FIXED vs STREAMLINED/new_version.

## Verdict

**`scLTy_002_part001_FIXED.ipynb` is the most correct notebook.**

It has proper parameterization, better reproducibility guarantees, and fixes multiple logical bugs present in the other two. It has one indentation bug in cell 122 (edge-case only).

## Detailed Differences

### 1. Hardcoded `0.05` vs Parameterized `alpha` (BUG in STREAMLINED/new_version)

STREAMLINED/new_version hardcode `0.05` in 8 places for significance thresholds (Sections 8c, 8c2, 8d, 8d2, 10b). FIXED correctly uses `alpha = params['alpha']` everywhere. If the significance level is changed in parameters, STREAMLINED/new_version silently use the wrong threshold.

### 2. Jaccard Distance Zero-Vector Handling (BUG in STREAMLINED/new_version)

STREAMLINED/new_version call `scipy.spatial.distance.jaccard(u, v)` directly. When both vectors are all-zero, this can return `nan`. FIXED adds an explicit guard:

```python
if not (u.any() or v.any()):
    jac = 0.0  # both fates absent across all clones
else:
    jac = 1.0 - distance.jaccard(u, v)
```

### 3. SeedSequence Reproducibility (BUG in STREAMLINED/new_version)

STREAMLINED/new_version use the SeedSequence directly from `rng_registry`. Since `.spawn()` is stateful, re-running the permutation cell produces different results. FIXED creates a fresh copy to preserve cell-level reproducibility.

### 4. `try/finally` for Safe Resource Cleanup

FIXED wraps registry-modifying operations in `try/finally` blocks (Sections 8b-verify and 8_sens), guaranteeing `rng_registry` restoration on failure. STREAMLINED/new_version leave the registry corrupted if an error occurs.

### 5. `PSEUDOCOUNT` Variable vs Hardcoded `1e-10`

FIXED uses the `PSEUDOCOUNT` parameter variable. STREAMLINED/new_version hardcode `1e-10`.

### 6. `MIN_BARCODES_PRIMARY` Parameterization

FIXED reads from params and uses the variable. STREAMLINED/new_version hardcode `"Primary (min_barcodes=5)"`.

### 7. Memory Estimate

FIXED: `10,000 x 11^2 x 8 x 3 = 29 MB` (correct — 3 coupling methods).
STREAMLINED/new_version: `10,000 x 11^2 x 8 x 5 = 48 MB` (incorrect — only 3 methods).

### 8. One-Sided Test Documentation

FIXED includes methodological notes about the one-sided upper-tail test. STREAMLINED/new_version omit these.

### 9. Cross-Reference Style

FIXED uses stable section-based references ("Section 7b"). STREAMLINED/new_version use fragile cell-number references ("Cell 90").

### 10. Stratification Interpretation

FIXED provides nuanced biological interpretation. STREAMLINED/new_version use overly alarmist language.

## Shared Bug Across All Three Notebooks

### Cell 151 `load_results` uses wrong tuple element (commented-out code)

The commented-out reload code in cell 151 (Section 11b) uses:
```python
expected_methods=[k for k, *_ in coupling_analyses]
```
This extracts method names (`'SW'`, `'Jaccard'`, `'Weinreb'`) instead of the dict keys (`'SW_weighted'`, `'Jaccard_binary'`, `'Weinreb_weighted'`). If uncommented, `validate_results()` would raise a `ValueError`. The correct expression should be:
```python
expected_methods=[k for _, _, k, _ in coupling_analyses]
```
This bug is present in **all three notebooks**.

## Issues in FIXED

**Cell 122 indentation bug**: The `try:` keyword is at column 0 instead of 4-space indent, breaking out of the `else:` block. Only manifests when fewer than 3 cell types survive the stringent filter (edge case).

## Summary Table

| Category | FIXED | STREAMLINED/new_version |
|---|---|---|
| Parameterized alpha | `alpha` variable | Hardcoded `0.05` (8 places) |
| Jaccard zero-vector | Safe guard | Potential `nan` |
| SeedSequence reuse | Fresh copy | Mutates original |
| try/finally safety | Yes (2 places) | No |
| PSEUDOCOUNT param | Variable | Hardcoded |
| Memory estimate | Correct (3 methods) | Wrong (says 5) |
| Test documentation | Complete | Missing |
| Cross-references | Section-based | Cell-number-based |
| Indentation (cell 122) | Bug: `try:` at col 0 | Correct indentation |
