# Documentation Review Agent Instructions

You are reviewing markdown documentation changes for the Spec-Kit unified SDD methodology.

## Files Under Review

1. `/home/localdev/work/spec-kit/temp/pm-agent-sdd-enhancements.md` (modified)
   - Stakeholder approval requirements removed
   - Timeline references updated

2. `/home/localdev/work/spec-kit/temp/unified-sdd-methodology-v2.md` (modified → v2.1)
   - Confidence checks added to 4 decision trees
   - Reflexion pattern added to Decision Tree 2
   - Evidence packages added to 8 milestones
   - Governance approval gate removed
   - Timeline updated from 10 weeks to 7 weeks

## Review Focus

### 1. Content Correctness (CRITICAL/HIGH Priority)

**Completeness Checks**:
- [ ] All 4 decision trees have confidence checks
- [ ] Decision Tree 2 has reflexion pattern (past ADR search)
- [ ] All 8 milestones have evidence packages
- [ ] Evidence packages include: file changes, validation, hallucination detection
- [ ] Timeline references changed from 10 weeks → 7 weeks
- [ ] Milestone 1 header changed from "Weeks 1-2" → "Week 1"
- [ ] Milestone 7 changed from "Week 8-9" → "Week 6"
- [ ] Milestone 8 changed from "Week 9-10" → "Week 7"
- [ ] All stakeholder approval references removed from enhancements doc

**Integration Accuracy**:
- [ ] Confidence check format consistent across all 4 trees
- [ ] Reflexion pattern properly integrated before confidence check in Tree 2
- [ ] Evidence packages follow consistent structure (File Changes → Validation → Hallucination Detection)
- [ ] Version updated to v2.1 in both header and footer
- [ ] Changelog documents v2.1 changes accurately

### 2. Internal Consistency (HIGH Priority)

**Cross-Reference Validation**:
- [ ] Section references still valid (e.g., "Section 2", "Section 9")
- [ ] Decision tree numbering (1-4) consistent
- [ ] Milestone numbering (1-8) consistent
- [ ] Task numbering (M1.0-M1.3, M2.1-M2.5, etc.) correct after removal of M1.0
- [ ] Week numbering logical (no gaps: Week 1, 3-4, 4-5, 5-6, 6-7, 7-8, 6, 7)

**Terminology Consistency**:
- [ ] "Milestone" vs "Phase" terminology correct (implementation = Milestones, SDD workflow = Phases)
- [ ] "Evidence Package" term used consistently
- [ ] "Confidence Check" term used consistently
- [ ] "Reflexion Check" vs "Reflexion Pattern" consistent

### 3. Structural Integrity (MEDIUM Priority)

**Format Validation**:
- [ ] Markdown syntax valid (no broken code blocks, lists, tables)
- [ ] Heading hierarchy logical (H1 → H2 → H3, no skips)
- [ ] Code blocks properly closed (```bash ... ```)
- [ ] Emoji usage consistent (🎯 for confidence, 🔍 for reflexion, 🚨 for hallucination)

**Organizational Logic**:
- [ ] Decision trees appear in Section 2
- [ ] Milestones appear in Section 9 (Implementation Roadmap)
- [ ] Evidence packages appear after deliverables in each milestone
- [ ] Confidence checks appear immediately after START in decision trees

### 4. Missing Enhancements (HIGH Priority)

From pm-agent-sdd-enhancements.md Priority 1-2 items, verify presence:

**Priority 1 (Confidence Checks)**:
- [ ] Decision Tree 1: 4 self-assessment questions present
- [ ] Decision Tree 2: Blocker understanding check present
- [ ] Decision Tree 3: Scope understanding check present
- [ ] Decision Tree 4: Business logic identification check present
- [ ] All have High (>90%), Medium (70-90%), Low (<70%) thresholds
- [ ] All have IF Low Confidence STOP gates

**Priority 2 (Evidence Requirements)**:
- [ ] Milestone 1: git diff, markdownlint, functional test
- [ ] Milestone 2: file counts, markdownlint
- [ ] Milestone 3: script execution, shellcheck
- [ ] Milestone 4: pytest, mypy, integration test
- [ ] Milestone 5: command count, help text check
- [ ] Milestone 6: word count, link validation
- [ ] Milestone 7: test project artifacts
- [ ] Milestone 8: version grep, CHANGELOG check, GitHub release

**Priority 3 (Reflexion Pattern)**:
- [ ] Decision Tree 2 has 3-step reflexion check
- [ ] Step 1: grep commands for past ADRs
- [ ] Step 2: analyze matches logic (same context, different context)
- [ ] Step 3: document learning

## Output Format

Use this exact format for findings:

```markdown
## Documentation Review Complete

**Files Reviewed**: 2
**Total Changes**: [count] additions, [count] modifications
**Recommendation**: [APPROVE | REQUEST_CHANGES | BLOCK]

---

### 🔴 CRITICAL Issues ([count])

[If none, write "None found ✓"]

#### [Issue Title]
- **File**: [path/to/file.md:line](path/to/file.md#Lline)
- **Issue**: [One sentence description]
- **Impact**: [Why this matters - completeness, accuracy, consistency]
- **Fix**: [Specific text change needed]

---

### 🟠 HIGH Issues ([count])

[Same format as CRITICAL]

---

### 🟡 MEDIUM Issues ([count])

[Same format, can be more concise]
- **File:Line**: [description]

---

### 🟢 LOW Issues ([count])

[Bullet list acceptable for brevity]
- [file:line](file#Lline): [description]

---

### ℹ️ INFO / Observations

[Bullet list]
- [observation]

---

### Verification Checklist

**Completeness** (Priority 1-2 enhancements):
- Decision Trees: [X/4] have confidence checks
- Reflexion Pattern: [Yes/No] in Decision Tree 2
- Evidence Packages: [X/8] milestones have packages
- Timeline Updates: [X/Y] week references corrected
- Stakeholder Removal: [Complete/Incomplete]

**Consistency**:
- Cross-references: [Valid/Broken]
- Terminology: [Consistent/Inconsistent]
- Formatting: [Valid/Invalid]

**Accuracy**:
- Version updated: [Yes/No] to v2.1
- Changelog accurate: [Yes/No]
- Task renumbering: [Correct/Incorrect] after M1.0 removal

---

### Summary

[2-3 sentences: Overall assessment, quality of integration, key concerns if any]

**Recommendation Rationale**: [Why APPROVE/REQUEST_CHANGES/BLOCK]

**Suggested Next Steps** (if applicable):
1. [Prioritized action items]
2. [...]

---

**Review Duration**: [time taken]
```

## Severity Levels

- **🔴 CRITICAL**: Missing required enhancements, broken structure, incorrect integration
- **🟠 HIGH**: Incomplete enhancements, inconsistent terminology, broken cross-references
- **🟡 MEDIUM**: Formatting issues, minor inconsistencies, style variations
- **🟢 LOW**: Typos, optional improvements, stylistic preferences
- **ℹ️ INFO**: Observations, suggestions, no action required

## Special Attention Points

1. **Milestone Task Renumbering**: Verify M1.1-M1.4 became M1.0-M1.3 after M1.0 removal
2. **Week Number Coherence**: Ensure no week number gaps (e.g., Week 1, then Week 3-4 is OK, but Week 1 then Week 5 is not)
3. **Confidence Check Placement**: Must be immediately after "START:" in decision trees, before any branching logic
4. **Evidence Package Structure**: Must have 3 sections minimum (File Changes, Validation, Hallucination Detection)
5. **Reflexion vs Confidence Order**: In Decision Tree 2, reflexion BEFORE confidence is correct (search past ADRs first)

## Begin Review

Start reviewing the two files listed above. Be thorough but efficient. Prioritize CRITICAL and HIGH issues. Complete review in <10 minutes.
