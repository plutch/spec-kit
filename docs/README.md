# Spec-Kit Documentation

**Version 2.3.0** - Comprehensive guides for Spec-Driven Development with Claude Code

---

## Quick Navigation

### 🚀 Getting Started

| Guide | Description | Time to Read |
|-------|-------------|--------------|
| **[Main README](../README.md)** | Complete overview, features, and quick start | 10 minutes |
| **[Installation Guide](../INSTALL.md)** | 2-minute copy/paste setup | 2 minutes |

### 📚 New in v2.3.0

| Guide | Description | Time to Read |
|-------|-------------|--------------|
| **[EARS Quick Reference](EARS-QUICK-REFERENCE.md)** | How to write unambiguous requirements with 20+ examples | 15 minutes |
| **[v2.3.0 Migration Guide](v2.3.0-MIGRATION-GUIDE.md)** | Upgrade from v2.1/v2.2 with step-by-step instructions | 20 minutes |

---

## Documentation by Topic

### Requirements Writing

**[EARS Quick Reference](EARS-QUICK-REFERENCE.md)** - Complete guide to Easy Approach to Requirements Syntax

**What You'll Learn**:
- The 4 EARS patterns (WHEN/IF/WHILE/WHERE)
- How to write testable, unambiguous requirements
- Requirement ID format and traceability
- Common mistakes and how to fix them
- Full traceability from spec to tests

**Best For**: Everyone using Spec-Kit v2.3.0+

---

### Migration & Upgrade

**[v2.3.0 Migration Guide](v2.3.0-MIGRATION-GUIDE.md)** - Upgrade existing projects to v2.3.0

**What You'll Learn**:
- Two migration paths (Fresh Start vs. Gradual)
- State file conversion (state.json → spec-metadata.json)
- EARS requirements migration strategies
- Memory system setup
- Troubleshooting common issues

**Best For**: Users upgrading from v2.1 or v2.2

---

### Templates & References

| Template | Purpose | Location |
|----------|---------|----------|
| **EARS Requirements** | Detailed EARS syntax and examples | [src/.specify/templates/requirements-ears.md](../src/.specify/templates/requirements-ears.md) |
| **Feature Specification** | Complete spec.md template | [src/.specify/templates/spec-template.md](../src/.specify/templates/spec-template.md) |
| **Task Breakdown** | tasks.md format and conventions | [src/.specify/templates/tasks-template.md](../src/.specify/templates/tasks-template.md) |
| **Architecture Decision Record** | ADR template for technical amendments | [src/.specify/templates/adr-template.md](../src/.specify/templates/adr-template.md) |
| **Component Audit** | Framework-specific UI component checklists | [src/.specify/templates/component-audit-templates/](../src/.specify/templates/component-audit-templates/) |

---

### Memory System (v2.3.0)

Persistent AI context files that remember your project standards across sessions:

| Memory File | When Loaded | Purpose |
|-------------|-------------|---------|
| **[constitution.md](../src/.specify/memory/constitution.md)** | Always | Project principles, architecture decisions, coding standards |
| **[api-standards.md](../src/.specify/memory/api-standards.md)** | Auto (API work) | REST conventions, endpoint patterns, error handling |
| **[testing-approach.md](../src/.specify/memory/testing-approach.md)** | Auto (test work) | Test pyramid, coverage thresholds, TDD practices |
| **[deployment-runbook.md](../src/.specify/memory/deployment-runbook.md)** | Manual (`@deployment-runbook.md`) | Deployment procedures, rollback steps, monitoring |

**Initialize with**: `/speckit.memory`

---

## Learning Path

### For New Users (Never Used Spec-Kit)

1. **[Main README](../README.md)** - Understand what Spec-Kit is (10 min)
2. **[Installation Guide](../INSTALL.md)** - Set up in your project (2 min)
3. **[EARS Quick Reference](EARS-QUICK-REFERENCE.md)** - Learn to write requirements (15 min)
4. Run your first workflow: `/speckit.specify` → `/speckit.clarify` → `/speckit.plan`

**Total Time**: ~30 minutes to productivity

---

### For Existing Users (Upgrading to v2.3.0)

1. **[v2.3.0 Migration Guide](v2.3.0-MIGRATION-GUIDE.md)** - Plan your upgrade (20 min)
2. Backup and update framework files (5 min)
3. Initialize memory system: `/speckit.memory` (5 min)
4. **[EARS Quick Reference](EARS-QUICK-REFERENCE.md)** - Learn new requirements format (15 min)
5. Continue with existing features (automatic compatibility)
6. Start new features with v2.3.0 workflow

**Total Time**: ~45 minutes for full upgrade

---

### For Advanced Users (Customizing Spec-Kit)

1. Read command source in `.claude/commands/` - All commands are self-documented
2. Review templates in `src/.specify/templates/` - All customizable
3. Customize memory files in `src/.specify/memory/` - Adapt to your project
4. Add custom component audit templates for your UI library

---

## Command Reference

All 17 commands are documented in their source files:

**Location**: `.claude/commands/speckit.*.md`

**Format**: Each command file includes:
- Purpose and when to use
- Step-by-step execution protocol
- Input/output examples
- Integration with other commands
- Error handling guidance
- Version history and compatibility

**Quick Access**:
```bash
# Read any command documentation
cat .claude/commands/speckit.specify.md
cat .claude/commands/speckit.plan.md
cat .claude/commands/speckit.implement.md
# ... etc
```

---

## FAQ

### Q: Where do I start?
**A**: Read the [Main README](../README.md) for overview, then [INSTALL.md](../INSTALL.md) for setup. Use [EARS Quick Reference](EARS-QUICK-REFERENCE.md) when writing your first spec.

### Q: I'm upgrading from v2.1/v2.2. What should I read?
**A**: Start with [v2.3.0 Migration Guide](v2.3.0-MIGRATION-GUIDE.md) - it has step-by-step instructions and backward compatibility info.

### Q: How do I write good requirements?
**A**: Follow the [EARS Quick Reference](EARS-QUICK-REFERENCE.md) - it has 20+ examples and a quality checklist.

### Q: What's the difference between EARS template and Quick Reference?
**A**:
- **Quick Reference** ([docs/EARS-QUICK-REFERENCE.md](EARS-QUICK-REFERENCE.md)) - User-friendly guide with practical examples, best for learning
- **Template** ([src/.specify/templates/requirements-ears.md](../src/.specify/templates/requirements-ears.md)) - Comprehensive reference with full methodology details

### Q: Can I customize the templates?
**A**: Yes! All templates in `src/.specify/templates/` are designed to be customized. Copy and modify for your project needs.

### Q: How do I add custom memory files?
**A**: Create new `.md` files in `.specify/memory/` and reference them in your project constitution or use manual loading (`@filename.md`).

---

## Version History

| Version | Release Date | Key Changes | Migration Guide |
|---------|--------------|-------------|-----------------|
| **2.3.0** | 2025-11-05 | EARS requirements, unified state management, phase tracking | [v2.3.0 Migration Guide](v2.3.0-MIGRATION-GUIDE.md) |
| **2.2.0** | 2025-01-16 | Socratic validation, component system audit, quick wins | Included in README |
| **2.1.0** | 2025-01-15 | Risk assessment, UX framework, test strategy, quality gates | Included in README |
| **2.0.0** | 2025-01-01 | Hierarchical specs, reconciliation, strict validation | Breaking changes in README |

---

## Getting Help

### Resources

- **GitHub Issues**: [Report bugs or request features](https://github.com/anthropics/claude-code/issues)
- **Command Documentation**: All commands self-documented in `.claude/commands/`
- **Templates**: Customizable templates in `src/.specify/templates/`

### Common Issues

See [Troubleshooting section in Migration Guide](v2.3.0-MIGRATION-GUIDE.md#troubleshooting) for:
- State file not found errors
- Phase shows wrong value
- EARS requirements not generated
- Command version mismatches

---

## Contributing to Documentation

Documentation improvements welcome! Guidelines:

1. **Clarity First**: Write for beginners, not experts
2. **Examples Required**: Every concept needs a real example
3. **Quick Navigation**: Add table of contents for docs >1000 words
4. **Version Compatibility**: Note which versions a doc applies to
5. **Test Your Examples**: Verify all code examples actually work

Submit documentation PRs to the main repository.

---

**Spec-Kit Version**: 2.3.0
**Documentation Last Updated**: 2025-11-05
**License**: MIT
