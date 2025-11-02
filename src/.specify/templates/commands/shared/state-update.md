# State Update Helper Module

**Purpose**: Standard instructions for updating feature state after command execution.

## Usage

Include this module at the end of each `/speckit.*` command with:

```markdown
@include shared/state-update.md#CommandName_StateUpdate
```

---

## Specify_StateUpdate

After `/speckit.specify` completes successfully:

1. **Create state file** at `.specify/memory/features/SPEC_DIR_NAME/state.json`:

   ```json
   {
     "feature_num": "FEATURE_NUM",
     "feature_name": "SPEC_DIR_NAME",
     "branch_name": "BRANCH_NAME",
     "spec_dir": "SPEC_DIR_NAME",
     "phase": "SPECIFYING",
     "progress": 15,
     "created_at": "[ISO-8601-NOW]",
     "updated_at": "[ISO-8601-NOW]",
     "gates_passed": ["spec_created"],
     "gates_failed": [],
     "commands_executed": [{"command": "specify", "timestamp": "[ISO-8601-NOW]", "success": true}],
     "clarifications_count": 0,
     "blockers": [],
     "notes": "Spec created. Ready for clarification."
   }
   ```

2. **Create/update** `.specify/memory/next_actions.md`:
   ```markdown
   ## Immediate
   - `/speckit.clarify` - Resolve ambiguities before planning
   ```

---

## Clarify_StateUpdate

After `/speckit.clarify` completes successfully:

1. **Update state.json**:
   ```json
   {
     "phase": "CLARIFYING",
     "progress": 30,
     "updated_at": "[ISO-8601-NOW]",
     "gates_passed": ["spec_created", "clarify_complete"],
     "commands_executed": [..., {"command": "clarify", "timestamp": "[ISO-8601-NOW]", "success": true, "clarifications_added": COUNT}],
     "clarifications_count": COUNT,
     "notes": "Clarifications complete. Ready for planning."
   }
   ```

2. **Update next_actions.md**:
   ```markdown
   ## Immediate
   - `/speckit.plan` - Create technical implementation plan
   ```

---

## Plan_StateUpdate

After `/speckit.plan` completes successfully:

1. **Update state.json**:
   ```json
   {
     "phase": "PLANNING",
     "progress": 50,
     "updated_at": "[ISO-8601-NOW]",
     "gates_passed": ["spec_created", "clarify_complete", "plan_complete", "simplicity_gate", "anti_abstraction_gate"],
     "commands_executed": [..., {"command": "plan", "timestamp": "[ISO-8601-NOW]", "success": true}],
     "notes": "Plan complete. Ready for task breakdown."
   }
   ```

2. **Update next_actions.md**:
   ```markdown
   ## Immediate
   - `/speckit.tasks` - Generate task breakdown from plan
   ```

---

## Tasks_StateUpdate

After `/speckit.tasks` completes successfully:

1. **Update state.json**:
   ```json
   {
     "phase": "TASKING",
     "progress": 70,
     "updated_at": "[ISO-8601-NOW]",
     "gates_passed": ["spec_created", "clarify_complete", "plan_complete", "tasks_complete"],
     "commands_executed": [..., {"command": "tasks", "timestamp": "[ISO-8601-NOW]", "success": true, "tasks_generated": COUNT}],
     "notes": "Tasks defined. Ready for implementation."
   }
   ```

2. **Update next_actions.md**:
   ```markdown
   ## Immediate
   - `/speckit.implement` - Execute tasks and build feature
   ```

---

## Implement_StateUpdate

After `/speckit.implement` completes successfully:

1. **Update state.json**:
   ```json
   {
     "phase": "IMPLEMENTING",
     "progress": 90,
     "updated_at": "[ISO-8601-NOW]",
     "gates_passed": ["spec_created", "clarify_complete", "plan_complete", "tasks_complete", "impl_complete"],
     "commands_executed": [..., {"command": "implement", "timestamp": "[ISO-8601-NOW]", "success": true}],
     "notes": "Implementation complete. Ready for validation."
   }
   ```

2. **Update next_actions.md**:
   ```markdown
   ## Immediate
   - Run tests and validate functionality
   - Create pull request when ready
   ```

---

## Error Handling

If command fails:

```json
{
  "gates_failed": ["command_name_gate"],
  "blockers": ["Specific error message"],
  "commands_executed": [..., {"command": "command_name", "timestamp": "[ISO-8601-NOW]", "success": false, "error": "Error description"}]
}
```

---

## Helper Functions

**Read Current State**:
```bash
cat .specify/memory/features/$(git rev-parse --abbrev-ref HEAD | sed 's|feature/||')/state.json
```

**Update Progress**:
- Atomic update: Read → Modify → Write
- Always update `updated_at` timestamp
- Append to `commands_executed` array (don't replace)

**Create Memory Structure** (if missing):
```bash
mkdir -p .specify/memory/features/
```
