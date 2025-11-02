# Mistake Log: [FEATURE_NUM]-[FEATURE_NAME]

**Date**: [YYYY-MM-DD]
**Feature**: `specs/[FEATURE_NUM]-[FEATURE_NAME]/`
**Phase**: [PHASE_WHERE_ERROR_OCCURRED]

---

## What Happened (現象)

[Brief description of the error or mistake]

**Examples**:
- Plan validation gate failed: Anti-Abstraction Gate
- Tests failed due to missing error handling
- Spec ambiguity led to incorrect implementation

---

## Root Cause (根本原因)

[Deep analysis of why it happened]

**Examples**:
- Created custom auth wrapper instead of using framework directly (violated Article VIII)
- Didn't define error scenarios in spec phase
- Assumed default behavior without clarifying

---

## Why Missed (なぜ見逃したか)

[Why didn't we catch this earlier?]

**Examples**:
- Didn't review constitution Article VIII before planning
- Skipped clarify command assuming requirements were clear
- No security review before implementation

---

## Impact

**Severity**: [Critical/High/Medium/Low]

**Cost**:
- Time lost: [hours/days]
- Rework extent: [% of feature affected]
- Downstream effects: [what else broke]

---

## Fix Applied (修正内容)

[What we did to resolve it]

**Examples**:
- Removed custom wrapper, used framework authentication directly
- Added error handling section to spec, regenerated plan
- Ran /speckit.clarify to resolve ambiguity

---

## Prevention Checklist (防止策)

How to avoid this in future:

- [ ] [PREVENTION_STEP_1]
- [ ] [PREVENTION_STEP_2]
- [ ] [PREVENTION_STEP_3]

**Examples**:
- [ ] Review constitutional gates before planning phase
- [ ] Run /speckit.clarify even if only 1-2 ambiguities present
- [ ] Add security review checklist item before implementation

---

## Lesson Learned (教訓)

[Key insight for future features]

**Examples**:
- Constitutional gates exist for a reason - check them early, not after plan fails
- Invest 5 minutes in clarification to save hours of rework
- "Obvious" requirements aren't obvious - always specify explicitly

---

## Pattern Recommendation

Should this become a pattern? (Yes/No)

**If Yes**: What pattern should be added to patterns_learned.jsonl?

```json
{
  "pattern_id": "...",
  "pattern_name": "...",
  "trigger": "...",
  "questions": [...],
  "notes": "Learned from mistake in feature [FEATURE_NUM]"
}
```

---

## Related Features

Features with similar issues:
- [FEATURE_NUM]: [Brief description]

---

## Follow-up Actions

- [ ] Update documentation
- [ ] Add to pattern library
- [ ] Share lesson with team
- [ ] Update command templates

---

**Confidence in Prevention**: [Low/Medium/High]

**Next Review Date**: [YYYY-MM-DD] (reassess if pattern prevents recurrence)
