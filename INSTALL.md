# Spec-Kit for Claude Code - Installation Guide

**Installation time: 2 minutes**

---

## Quick Install

### Step 1: Navigate to Your Project

```bash
cd /path/to/your-project/
```

### Step 2: Copy the Framework

```bash
cp -r /path/to/spec-kit-claude/src/.claude ./
cp -r /path/to/spec-kit-claude/src/.specify ./
```

### Step 3: Verify Installation

Open Claude Code in your project directory. You should see the following commands available:

- `/speckit.specify`
- `/speckit.clarify`
- `/speckit.plan`
- `/speckit.tasks`
- `/speckit.implement`
- `/speckit.constitution`
- `/speckit.supplement`
- `/speckit.reconcile`
- `/speckit.validate-hierarchy`

**Done!** You're ready to start using Spec-Driven Development.

---

## What Gets Installed?

After installation, your project will contain:

```
your-project/
├── .claude/
│   └── commands/              # 9 self-contained command files (196KB)
└── .specify/
    ├── config.example.yml     # Configuration template (8KB)
    ├── business-rules/        # Business rule validation templates
    ├── memory/                # Project constitution templates
    │   └── constitution.md
    ├── templates/             # All SDD templates (~250KB)
    └── docs/                  # Methodology guides (~100KB)
```

**Total size**: ~550KB

---

## Installation Options

### Option 1: Full Installation (Recommended)

Copy both `.claude/` and `.specify/` for complete functionality:

```bash
cp -r /path/to/spec-kit-claude/src/{.claude,.specify} ./
```

**Includes:**
- All 9 commands
- All templates
- Configuration examples
- Methodology documentation

### Option 2: Minimal Installation

Copy only `.claude/` for basic commands:

```bash
cp -r /path/to/spec-kit-claude/src/.claude ./
```

**Includes:**
- All 9 commands (self-contained)

**Missing:**
- Templates (commands still work, but no customization templates)
- Configuration examples
- Documentation

**Recommendation**: Use full installation unless you have specific size constraints.

---

## Post-Installation Configuration

### 1. Create Project Constitution (Recommended)

```bash
# In Claude Code
/speckit.constitution
```

This creates `.specify/memory/constitution.md` with your project's guiding principles.

### 2. Configure Settings (Optional)

```bash
cp .specify/config.example.yml .specify/config.yml

# Edit .specify/config.yml to customize:
# - Supplementary spec patterns
# - Validation strictness
# - Auto-discovery settings
```

---

## Verification

### Test Basic Workflow

```bash
# In Claude Code, run these commands in sequence:

# 1. Create specification
/speckit.specify "Build a simple task manager"

# You should see specs/ directory created with spec.md

# 2. Generate plan
/speckit.plan

# You should see plan.md generated

# 3. Break down tasks
/speckit.tasks

# You should see tasks.md generated
```

**If all commands work**, installation is successful!

---

## Updating the Framework

To update to a newer version of Spec-Kit:

```bash
# 1. Backup your existing specs
cp -r specs/ specs.backup/

# 2. Remove old framework files
rm -rf .claude/ .specify/

# 3. Copy new version
cp -r /path/to/spec-kit-claude-new/src/{.claude,.specify} ./

# 4. Restore your specs
# (Your specs/ directory should not be affected, but backup is safe)
```

**Note**: Your existing specifications in `specs/` directory are NOT touched during updates.

---

## Uninstallation

To remove Spec-Kit from your project:

```bash
# Remove framework files
rm -rf .claude/ .specify/

# (Optional) Remove generated specifications
rm -rf specs/
```

**Warning**: This will delete all your specifications. Backup first!

---

## Troubleshooting

### Commands Not Appearing in Claude Code

**Problem**: Commands don't show up when typing `/speckit.`

**Solutions**:
1. Restart Claude Code
2. Verify `.claude/commands/` contains 9 `.md` files
3. Check file permissions: `ls -la .claude/commands/`

### Permission Errors

**Problem**: Cannot copy files due to permissions

**Solution**:
```bash
# Make sure you have write permissions in target directory
ls -ld .

# If needed, fix permissions
chmod u+w .
```

### Missing Templates

**Problem**: Commands work but can't find templates

**Solution**:
```bash
# Verify .specify/ exists
ls -la .specify/

# If missing, copy it
cp -r /path/to/spec-kit-claude/src/.specify ./
```

---

## Installation Locations

### Multiple Projects

Spec-Kit can be installed in multiple projects independently:

```bash
# Project 1
cd ~/projects/project1/
cp -r /path/to/spec-kit-claude/src/{.claude,.specify} ./

# Project 2
cd ~/projects/project2/
cp -r /path/to/spec-kit-claude/src/{.claude,.specify} ./
```

Each project has its own isolated Spec-Kit installation.

### Global Templates (Advanced)

You can share templates across projects by symlinking:

```bash
# Install Spec-Kit in a central location
mkdir -p ~/.spec-kit/
cp -r /path/to/spec-kit-claude/src/.specify ~/.spec-kit/

# In each project, create commands and symlink templates
cd ~/projects/project1/
cp -r /path/to/spec-kit-claude/src/.claude ./
ln -s ~/.spec-kit/.specify .specify
```

**Warning**: Changes to templates will affect all projects using the symlink.

---

## Requirements

### System Requirements
- **OS**: Linux, macOS, or Windows
- **Claude Code**: Version 0.7.0 or later
- **Disk Space**: ~1MB per project

### No Dependencies Required
- ❌ No Python
- ❌ No Node.js
- ❌ No package managers (npm, pip, etc.)
- ❌ No build tools
- ❌ No configuration files to edit

Just copy and use!

---

## Next Steps

After installation:

1. **Read the methodology**: See [README.md](README.md) for complete overview
2. **Create your first spec**: Run `/speckit.specify` with a feature description
3. **Explore documentation**: Check `.specify/docs/` for detailed guides

---

## Getting Help

### Documentation
- **README.md**: Framework overview and features
- **CLAUDE.md**: Working with this repository (for contributors)
- **.specify/docs/**: Detailed methodology guides

### Support
- **GitHub Issues**: [https://github.com/github/spec-kit/issues](https://github.com/github/spec-kit/issues)
- **Documentation**: [https://github.github.io/spec-kit/](https://github.github.io/spec-kit/)

---

## FAQ

### Q: Do I need to run any installation scripts?

**A:** No! Spec-Kit uses pure copy/paste installation. Just copy the directories and start using the commands.

### Q: Can I customize the commands?

**A:** Yes! All commands in `.claude/commands/` are editable markdown files. Customize them to your needs.

### Q: Will updates overwrite my specifications?

**A:** No. Your specifications in `specs/` directory are never touched during framework updates. Only `.claude/` and `.specify/` are replaced.

### Q: Can I use Spec-Kit without the `.specify/` directory?

**A:** Yes, but you'll lose access to templates and documentation. Commands in `.claude/` are self-contained and will still work.

### Q: How do I share Spec-Kit with my team?

**A:** Commit `.claude/` and `.specify/` to your project's Git repository. Your team will get the framework when they clone the repo.

### Q: Can I use Spec-Kit with other AI assistants?

**A:** Spec-Kit v2.0 is optimized exclusively for Claude Code. For multi-LLM support, see the original spec-kit repository.

---

**Installation Complete!** Start with `/speckit.specify` to create your first specification.

**Spec-Kit Version**: 2.0.0
**Target**: Claude Code 0.7.0+
**License**: MIT
