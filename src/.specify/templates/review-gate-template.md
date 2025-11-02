# [PHASE_NAME] Review Gate Template

**Purpose**: Validate [phase] completeness and readiness to proceed to [next phase].

---

## MANDATORY: The Four [Phase] Questions

### ❓ Question 1: [Validation Focus Area 1]

**Action Required**:
- [What agent must verify]
- [Where to check (file paths, commands)]
- [What counts as evidence]

**Expected Evidence Format**:
```yaml
[Structure for evidence presentation]
- Field 1: [Value]
- Field 2: [Value]
- Status: [Pass/Fail with criteria]
```

**Readiness Determination**:
```yaml
IF [condition]:
  → ✅ [Status]
IF [condition]:
  → ⚠️ [Status]
IF [condition]:
  → ❌ [Status]
```

---

### ❓ Question 2: [Validation Focus Area 2]
[Same structure as Question 1]

---

### ❓ Question 3: [Validation Focus Area 3]
[Same structure as Question 1]

---

### ❓ Question 4: [Evidence of Completeness]
**Required Evidence (ALL must be provided)**:
1. **File Existence Check (MANDATORY)**:
   ```bash
   Run: ls -la [TARGET_DIR]
   Output: [Expected files]
   ```

2. **Content Validation (MANDATORY)**:
   ```bash
   Run: [grep/validation commands]
   Output: [Expected results]
   ```

3. **Traceability (MANDATORY)**:
   [Map from previous phase to current phase]
   Coverage: [X]% (target: [Y]%)

**IF any evidence is MISSING**:
```
❌ CANNOT report completion
→ Gather missing evidence first
→ Re-run this step with complete evidence
```

---

## Hallucination Prevention (7 Red Flags for [Phase])

```yaml
Detect and BLOCK these patterns:

🚨 "[Phase] complete" WITHOUT [specific evidence]
   → Self-correction: "Wait, I need to [verification action]"

🚨 "All [artifacts] generated" WITHOUT showing file list
   → Self-correction: "Let me verify files actually exist"

🚨 "[Quality claim]" WITH [contradictory evidence]
   → Self-correction: "[Issue description]"

🚨 Claiming [completeness] WITHOUT [traceability]
   → Self-correction: "Need to map [source] to [destination]"

🚨 Skipping [critical check]
   → Self-correction: "I need to verify [requirement]"

🚨 Hiding [errors/failures]
   → Self-correction: "Must report [issues] honestly"

🚨 "[Ready]" statements WITHOUT evidence
   → Self-correction: "Need to verify completeness criteria first"

IF detected: STOP → Gather evidence → Report honestly
```

---

## Determine Status

### ✅ **READY for [Next Phase]**

**Criteria (ALL must be met)**:
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]
- [ ] [Criterion 4]

**IF ALL criteria met**:
→ Proceed to [next step] (present completion report)

---

### ⚠️ **CAN PROCEED** (with risks noted)

**Criteria**:
- [Partial completion condition]
- [Minor gaps documented]
- [Non-blocking issues]

**IF criteria met**:
→ Present risks to user
→ Ask: "Minor gaps remain. Proceed to [next phase] or address gaps?"
→ Wait for user decision before proceeding

---

### ❌ **NOT READY** (more [phase work] needed)

**Criteria (ANY triggers NOT READY)**:
- [Critical gap 1]
- [Critical gap 2]
- [Blocking issue]

**IF NOT READY**:
→ Present issues to user with evidence
→ Recommend specific actions:
  - [Action 1]
  - [Action 2]
  - [Action 3]
→ STOP (do not proceed to next phase)

---

## Output Format

**Present to User - ONLY if evidence provided**:

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[EMOJI] [Phase Name] Review Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: [✅ READY | ⚠️ CAN PROCEED | ❌ NOT READY]

Branch: [branch-name]
[Primary Artifact]: [path/to/artifact]

## [Validation Area 1]

[Evidence presentation - scores, counts, status]

## [Validation Area 2]

[Evidence presentation]

## [Validation Area 3]

[Evidence presentation]

## [Evidence Section]

[Actual command outputs, file listings, validation results]

## Readiness Assessment

[IF ✅ READY:]
✅ [All criteria met statement]
✅ Ready to proceed to [next phase] ([next command])

[IF ⚠️ CAN PROCEED:]
⚠️ Minor gaps detected:
   - [Gap 1]: [Why low impact]
   - [Gap 2]: [Why low impact]

Risk: [Description of proceeding with gaps]

[IF ❌ NOT READY:]
❌ Critical gaps prevent progression:
   - [Gap 1]: [HIGH impact] - [Why blocks]
   - [Gap 2]: [HIGH impact] - [Why blocks]

Required Actions:
1. [Specific fix 1]
2. [Specific fix 2]
3. [Specific fix 3]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## Next Steps
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[IF ✅ READY:]
1. Proceed to [next phase]:
   [next command]

2. Or review artifacts:
   [artifact paths]

[IF ⚠️ CAN PROCEED:]
Choose one:
1. Proceed with noted risks:
   [next command]

2. Address minor gaps:
   [specific actions]

3. Review artifacts:
   [artifact paths]

[IF ❌ NOT READY:]
REQUIRED: Address critical gaps before [next phase]:

1. For [gap category]:
   - [Action 1]
   - [Action 2]

2. For [gap category]:
   - [Action 1]
   - [Action 2]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Ready to proceed? (yes/review artifacts/address gaps/[other option])
```

---

## User Interaction

```yaml
IF user says "yes" or "proceed":
  IF status = READY:
    → Suggest: [next command]
  IF status = CAN PROCEED:
    → Warn about risks, then suggest: [next command]
  IF status = NOT READY:
    → Block: "Cannot proceed - critical gaps remain"
    → Suggest specific remediation actions

IF user says "review" or "review artifacts":
  → Show list of artifacts with paths
  → Offer to display specific artifact contents

IF user says "address gaps":
  → List specific gaps with:
     - Why they're critical (impact)
     - Suggested resolution steps
     - Which artifacts need updating
  → Ask: "Which gap to address first?"

IF user says [phase-specific option]:
  → [Phase-specific handling]

IF EVIDENCE MISSING:
  ❌ "Cannot complete [phase] review without evidence."
  → List missing evidence
  → Gather evidence automatically where possible
  → Re-run review gate
```

---

## Benefits

**From PM Agent Reflexion pattern**:
- ✅ Prevents premature phase progression with incomplete artifacts
- ✅ Evidence-based completeness validation
- ✅ No false "ready" claims without verification
- ✅ Transparent validation of [phase] effectiveness
- ✅ User confidence in [phase output] quality
