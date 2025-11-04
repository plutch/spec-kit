# Component Audit Templates

## Purpose

This directory contains framework-specific templates for auditing component library usage in UI specifications. These templates are used by `/speckit.analyze-ux` when UI-SPEC.md is detected.

## Available Templates

| Framework | Component Library | Template File | MCP Support |
|-----------|------------------|---------------|-------------|
| React | Material-UI (MUI) | react-mui.md | No |
| React | Chakra UI | react-chakra.md | No |
| Angular | Kendo UI | angular-kendo.md | Yes (kendo-angular-assistant) |
| Angular | Angular Material | angular-material.md | No |
| Vue | Vuetify | vue-vuetify.md | No |
| Generic | Any | generic.md | No |

## Template Structure

Each template contains:

1. **Common Components Checklist**: Standard components organized by category (Buttons, Inputs, Navigation, Layout, Feedback, Data Display, Overlays)
2. **Design Token Checks**: Framework-specific theme system references (colors, spacing, typography, shadows, breakpoints)
3. **Audit Questions**: Framework-specific validation questions
4. **Component Reuse Calculation**: Formula for calculating library usage percentage
5. **Output Format**: Standardized reporting structure

## Usage

When `/speckit.analyze-ux` detects UI-SPEC.md:

1. **Framework Detection**: Prompts user to select framework + component library
2. **Template Loading**: Loads corresponding template (or generic.md if not found)
3. **Audit Execution**: Scans UI-SPEC.md against template checklist
4. **MCP Enhancement** (optional): If MCP available for framework, queries component documentation
5. **Report Generation**: Creates Component System Audit section in analysis output

## MCP Integration

### Kendo UI (Angular)

When `angular-kendo.md` template is used AND MCP server `kendo-angular-assistant` is available:

**Tool**: `mcp__kendo-angular-assistant__kendo_angular_assistant`

**Parameters**:
- `query` (required): Specific question about component usage
- `component` (required): Component name (e.g., "Grid", "DatePicker", "Button")

**Example Query**:
```
Query: "What are the required props and accessibility features?"
Component: "Grid"
```

**Graceful Fallback**: If MCP not available, template is used standalone without component doc queries.

## Adding New Templates

To add a new framework:

1. Create `[framework]-[library].md` file in this directory
2. Follow the structure in `generic.md`
3. Populate Common Components Checklist with library-specific components
4. Add Design Token Checks with framework theme system references
5. Define Audit Questions specific to that framework
6. Update this README.md with new template entry

## Version

**Template Version**: 2.2.0
**Last Updated**: 2025-01-16
**Compatibility**: SpecKit v2.2+
