# Generic Component Library Audit Template

## Framework Information

**Framework**: [Specify framework]
**Component Library**: [Specify library]
**Theme System**: [Specify theming approach]
**Documentation**: [Specify documentation URL]

## Purpose

This is a generic fallback template for auditing component library usage when a framework-specific template is not available. Adapt this template to your specific component library.

## Common Components Checklist

Use this checklist to identify which components should be provided by your component library vs. implemented as custom components.

### Buttons & Actions
- [ ] Button (various variants/styles)
- [ ] IconButton
- [ ] Button Group
- [ ] Toggle Button
- [ ] Floating Action Button

### Form Inputs
- [ ] Text Input
- [ ] Text Area
- [ ] Number Input
- [ ] Select / Dropdown
- [ ] Autocomplete / Combobox
- [ ] Checkbox
- [ ] Radio Button
- [ ] Switch / Toggle
- [ ] Slider
- [ ] Date Picker
- [ ] Time Picker
- [ ] File Upload
- [ ] Color Picker
- [ ] Rating

### Navigation
- [ ] Navigation Bar / Header
- [ ] Sidebar / Drawer
- [ ] Menu / Dropdown Menu
- [ ] Tabs
- [ ] Breadcrumbs
- [ ] Pagination
- [ ] Stepper
- [ ] Bottom Navigation

### Layout
- [ ] Container
- [ ] Grid System
- [ ] Flexbox Utilities
- [ ] Card
- [ ] Panel / Surface
- [ ] Divider
- [ ] Spacer
- [ ] Accordion / Expansion Panel

### Feedback
- [ ] Alert / Banner
- [ ] Toast / Snackbar / Notification
- [ ] Dialog / Modal
- [ ] Drawer (if not in Navigation)
- [ ] Tooltip
- [ ] Popover
- [ ] Progress Bar (linear)
- [ ] Progress Spinner (circular)
- [ ] Skeleton Loader
- [ ] Badge
- [ ] Loading Indicator

### Data Display
- [ ] Table (simple)
- [ ] Data Grid / Data Table (advanced)
- [ ] List
- [ ] Chip / Tag
- [ ] Avatar
- [ ] Icon
- [ ] Image
- [ ] Typography
- [ ] Tree View
- [ ] Timeline

### Overlays
- [ ] Modal / Dialog (if not in Feedback)
- [ ] Drawer (if not in Navigation)
- [ ] Popover (if not in Feedback)
- [ ] Bottom Sheet
- [ ] Backdrop

## Design Token Checks

Adapt these checks to your component library's theming system.

### Colors
**Theme References** (CORRECT):
- Primary, secondary, accent colors from theme
- Semantic colors: error, warning, info, success
- Background and text colors from theme
- Component color props that reference theme

**Hardcoded Values** (INCORRECT):
- Hex codes: `#3f51b5`, `#ff4081`
- RGB values: `rgb(63, 81, 181)`
- Named colors: `blue`, `red` (unless part of theme)

### Spacing
**Theme References** (CORRECT):
- Spacing scale from theme (e.g., 4px, 8px, 16px, 24px, 32px, 48px, 64px)
- Spacing utilities or functions
- Consistent grid system (4px or 8px base unit)

**Hardcoded Values** (INCORRECT):
- Random spacing values not on your grid (e.g., 13px, 22px, 37px)

### Typography
**Theme References** (CORRECT):
- Typography scale from theme (headings, body, caption, etc.)
- Font weights from theme (light, regular, medium, bold)
- Line heights from theme
- Typography component variants

**Hardcoded Values** (INCORRECT):
- Random font sizes: `font-size: 17px`
- Random font weights: `font-weight: 650`

### Shadows
**Theme References** (CORRECT):
- Elevation/shadow scale from theme (0-24 levels typical)
- Component elevation props

**Hardcoded Values** (INCORRECT):
- Custom box-shadow values

### Breakpoints
**Theme References** (CORRECT):
- Named breakpoints from theme (xs, sm, md, lg, xl)
- Responsive utilities from framework
- Grid system responsive props

**Hardcoded Values** (INCORRECT):
- Random media queries: `@media (max-width: 768px)`
- Pixel-specific breakpoints not from theme

## Audit Questions

### 1. Component Usage
Does the UI-SPEC specify component library components by name or describe custom implementations?

**Look for**:
- ✅ GOOD: References to library component names
- ✅ GOOD: Props/attributes consistent with library documentation
- ❌ BAD: "Custom [component]" when library provides equivalent
- ⚠️ INCONSISTENT: Mix of library and custom components for same purpose

### 2. Design Token Adherence
Are theme variables/tokens specified or hardcoded values used?

**Look for**:
- ✅ GOOD: References to theme colors, spacing, typography
- ❌ BAD: Hardcoded hex codes, pixel values, font sizes

### 3. Component Props Consistency
Are component props/attributes consistent with library documentation?

**Look for**:
- ✅ GOOD: Correct prop names and values per documentation
- ❌ BAD: Incorrect prop names, unsupported values, deprecated props

### 4. Responsive Design
Is responsive behavior specified using framework's responsive utilities?

**Look for**:
- ✅ GOOD: Grid system with responsive breakpoints
- ✅ GOOD: Display utilities for show/hide at breakpoints
- ❌ BAD: Custom media queries not aligned with framework

### 5. Accessibility
Are accessibility features specified per library guidance?

**Look for**:
- ✅ GOOD: ARIA labels on icon-only buttons
- ✅ GOOD: Form labels, helper text, error messages
- ✅ GOOD: Keyboard navigation support
- ❌ BAD: Missing accessibility attributes

### 6. Theming & Customization
Is theme customization specified or inline styles overused?

**Look for**:
- ✅ GOOD: Theme overrides at theme level
- ⚠️ WARNING: Excessive inline styles (should use theme)
- ❌ BAD: Inconsistent styling approach

## Component Reuse Calculation

```
Component Reuse Score = (Library Components / Total Components) × 100%

Example:
- Library Components: 36
- Custom Components: 7
- Total: 43

Score = (36 / 43) × 100% = 83.7%

Thresholds:
- 🟢 Excellent: ≥80% (high library usage, minimal custom components)
- 🟡 Good: 60-79% (mostly library, some custom where justified)
- 🔴 Needs Improvement: <60% (too many custom reimplementations)
```

## Output Format

```markdown
## Component System Audit

**Framework Detected**: [Framework] + [Component Library]

**Component Consistency**:
- ✅ **Used Correctly**: [List library components with counts] = [N] total
  - Example: Button (12), TextField (8), Card (5) = 25 components
- ❌ **Custom Reimplementation**: [Components duplicating library functionality]
  - Example: "CustomButton" duplicates library Button
- ⚠️ **Inconsistent Usage**: [Mix of library + custom for same purpose]
  - Example: Mix of library Button (10) and custom <button> (2)

**Design Token Adherence**:
- ✅ **Colors**: [X]% using theme color references
- ❌ **Colors**: [Y]% hardcoded (hex codes) - should use theme
- ✅ **Spacing**: [X]% using spacing scale
- ⚠️ **Typography**: [X]% using typography scale

**Recommendations**:
1. [Replace custom component X with library component Y]
   - Impact: [Quantified benefit]
   - Effort: [Time estimate]
   - Location: [Section reference]
2. [Standardize on library component Z]
   - Impact: [Quantified benefit]
   - Effort: [Time estimate]
   - Location: [Section reference]
3. [Replace hardcoded values with theme tokens]
   - Impact: [Quantified benefit]
   - Effort: [Time estimate]
   - Location: [Section reference]

**Component Reuse Score**: [X]% ([Excellent/Good/Needs Improvement])
```

## MCP Integration

**Not available for generic template** (no MCP server)

If your component library has MCP support:
1. Update this template with MCP tool information
2. Add example queries for common components
3. Document graceful fallback behavior

## Customization Instructions

To create a framework-specific template from this generic template:

1. **Update Framework Information**:
   - Specify framework name, version
   - Specify component library name, version
   - Add documentation URL

2. **Populate Component Checklist**:
   - Research library's component catalog
   - Organize by category (Buttons, Inputs, Navigation, etc.)
   - Mark common components with ✅

3. **Document Design Tokens**:
   - Identify theme system (SCSS variables, CSS custom properties, JS theme object)
   - Document color, spacing, typography, shadow, breakpoint tokens
   - Provide correct and incorrect examples

4. **Write Audit Questions**:
   - Tailor questions to library's patterns
   - Add library-specific considerations
   - Include accessibility guidelines

5. **Add Example Output**:
   - Use library-specific component names
   - Show realistic component counts
   - Demonstrate recommendations format

6. **Save Template**:
   - Name: `[framework]-[library].md`
   - Location: `.specify/templates/component-audit-templates/`
   - Update README.md with new template entry

---

**Template Version**: 2.2.0
**Last Updated**: 2025-01-16
**Compatibility**: SpecKit v2.2+
