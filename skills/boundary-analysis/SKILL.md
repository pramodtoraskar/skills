---
name: boundary-analysis
description: >-
  Tests input limits to find off-by-one errors and edge cases by exercising minimum/maximum boundaries for lengths, numbers, dates, and file sizes. Use when forms show explicit limits, counters, or "max N characters", or when users mention "edge cases", "limits", "boundaries", "just over the limit", or testing at exact thresholds.
compatibility: claude, cursor
---

# Boundary-Analysis

## When to Apply

Use boundary analysis when you encounter:
- Form fields with visible character/length limits
- Numeric inputs with min/max constraints
- File upload size restrictions
- Date ranges or calendar controls
- Counter displays showing limits
- API documentation specifying bounds
- User requests to "test edge cases" or "check limits"

## Procedure

1. **Extract all limits** from the interface, code, or documentation:
   - Character limits (maxlength attributes, counter text)
   - Numeric ranges (min/max values, step increments)
   - File size restrictions
   - Date boundaries (earliest/latest allowed dates)

2. **Identify boundary pairs** for each limit:
   - Minimum: `min`, `min-1` 
   - Maximum: `max`, `max+1`
   - Zero: `0`, `-1` (for numeric fields)
   - Empty: `""`, `null` (for text fields)

3. **Generate test values** systematically:
   - **Length boundaries**: If max is 50 chars, test 49, 50, 51
   - **Numeric boundaries**: If range is 1-100, test 0, 1, 100, 101
   - **Date boundaries**: First/last day of month, Feb 29th, invalid dates like Feb 30th
   - **File boundaries**: Exactly at size limit, 1 byte over

4. **Execute tests** and record outcomes:
   - Note whether each value is accepted or rejected
   - Capture exact error messages
   - Check for silent truncation or rounding

5. **Document findings** with expected vs actual behavior

## Checklist

- [ ] All visible limits extracted from UI elements
- [ ] Hidden limits found in HTML attributes or network requests  
- [ ] Boundary pairs defined for each limit type
- [ ] Test values generated for min-1, min, max, max+1
- [ ] Special cases covered (zero, negative, empty, null)
- [ ] Date edge cases tested (leap years, month boundaries, invalid dates)
- [ ] File size boundaries tested where applicable
- [ ] Results documented with expected behavior noted
- [ ] Off-by-one errors identified and reported- [ ] Save output to md files

## Gotchas

- **Client vs server validation**: Frontend may accept values that backend rejects, or vice versa
- **Silent truncation**: Some systems cut off excess characters without error messages
- **Unicode considerations**: Multi-byte characters may count differently than expected
- **Floating point precision**: Decimal boundaries may behave unexpectedly due to rounding
- **Timezone effects**: Date boundaries may shift based on user timezone vs server timezone

## Output Format

```
## Boundary Analysis Results

### [Field/Component Name]
**Stated Limit**: [limit description]
**Test Values**:
- min-1 ([value]): [PASS/FAIL] - [behavior/error]
- min ([value]): [PASS/FAIL] - [behavior/error] 
- max ([value]): [PASS/FAIL] - [behavior/error]
- max+1 ([value]): [PASS/FAIL] - [behavior/error]

**Issues Found**: [off-by-one errors, unexpected behavior]
**Risk Level**: [Low/Medium/High]
```

## References

Link to detailed boundary value analysis techniques and common edge case patterns in your project's testing documentation.