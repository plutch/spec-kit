# Angular + Angular Material Component Audit Template

## Framework Information

**Framework**: Angular
**Component Library**: Angular Material
**Theme System**: Material Design theming with SCSS
**Documentation**: https://material.angular.io/

## Common Components Checklist

### Buttons & Actions
- ✅ Button (mat-button, mat-raised-button, mat-flat-button, mat-stroked-button, mat-icon-button, mat-fab, mat-mini-fab)
- ✅ ButtonToggle / ButtonToggleGroup

### Form Inputs
- ✅ Input (matInput directive)
- ✅ Select
- ✅ Autocomplete
- ✅ Checkbox
- ✅ Radio / RadioGroup
- ✅ Slide Toggle (switch)
- ✅ Slider
- ✅ Datepicker

### Navigation
- ✅ Toolbar
- ✅ Sidenav / SidenavContainer
- ✅ Menu
- ✅ Tabs / TabGroup
- ✅ Stepper / Step

### Layout
- ✅ Card / CardHeader / CardContent / CardActions
- ✅ Divider
- ✅ Expansion Panel / Accordion
- ✅ Grid List
- ✅ List / NavList / ActionList / SelectionList

### Feedback
- ✅ Dialog
- ✅ Snackbar
- ✅ Tooltip
- ✅ Progress Bar
- ✅ Progress Spinner
- ✅ Badge

### Data Display
- ✅ Table / Sort / Paginator
- ✅ List (also in Layout)
- ✅ Chips / ChipList
- ✅ Icon (mat-icon)
- ✅ Tree

### Overlays
- ✅ Dialog (also in Feedback)
- ✅ Bottom Sheet
- ✅ Overlay (CDK primitive)

## Design Token Checks

### Colors
**Material Theme Variables** (CORRECT):
- `$theme-primary` / `$theme-accent` / `$theme-warn`
- `mat-color($theme-primary)` / `mat-color($theme-accent)`
- Component color attributes: `color="primary"`, `color="accent"`, `color="warn"`

**Hardcoded Values** (INCORRECT):
- `#3f51b5`, `#ff4081`, `rgb(63, 81, 181)`

### Spacing
**Material Spacing** (CORRECT):
- Material Design 8dp grid system
- Common spacing: 8px, 16px, 24px, 32px, 48px, 64px

**Hardcoded Values** (INCORRECT):
- Non-standard spacing like 12px, 18px, 22px

### Typography
**Material Typography** (CORRECT):
- Material typography scale (display-4 through caption)
- `mat-typography-level($config, headline)`
- Component typography: `<h1 mat-title>`, `<p mat-body-1>`

**Hardcoded Values** (INCORRECT):
- `font-size: 14px`, `font-weight: 500`

### Elevation (Shadows)
**Material Elevation** (CORRECT):
- Material elevation levels 0-24
- Component elevation: `[elevation]="4"` (not all components support)

**Hardcoded Values** (INCORRECT):
- `box-shadow: 0 2px 4px rgba(0,0,0,0.2)`

### Breakpoints
**Angular CDK Breakpoints** (CORRECT):
- `Breakpoints.Handset`, `Breakpoints.Tablet`, `Breakpoints.Web`
- BreakpointObserver service

**Hardcoded Values** (INCORRECT):
- `@media (max-width: 768px)`

## Audit Questions

### 1. Component Usage
Does the UI-SPEC specify Angular Material components by name or describe custom implementations?

**Look for**:
- ✅ GOOD: "Use mat-table with matSort and mat-paginator"
- ✅ GOOD: "mat-datepicker with min/max validation"
- ❌ BAD: "Custom table with sort functionality"
- ⚠️ INCONSISTENT: Mix of mat-button and custom <button>

### 2. Design Token Adherence
Are Material theme variables specified or hardcoded values used?

**Look for**:
- ✅ GOOD: "Primary color (color='primary')"
- ✅ GOOD: "Spacing: 16px (Material 8dp grid)"
- ❌ BAD: "Color: #3f51b5", "Padding: 12px"

### 3. Component Props Consistency
Are component attributes consistent with Angular Material documentation?

**Look for**:
- ✅ GOOD: "mat-raised-button color='primary'"
- ❌ BAD: "mat-button variant='raised'" (Angular Material uses separate directives, not variant prop)

### 4. Responsive Design
Is responsive behavior specified using Angular CDK BreakpointObserver?

**Look for**:
- ✅ GOOD: "Use BreakpointObserver to detect Handset/Tablet/Web"
- ✅ GOOD: "Sidenav mode='side' on desktop, mode='over' on mobile"
- ❌ BAD: Custom media queries not aligned with Material breakpoints

### 5. Accessibility
Are accessibility features specified per Material guidance?

**Look for**:
- ✅ GOOD: "mat-icon-button aria-label='Delete item'"
- ✅ GOOD: "matInput aria-describedby='email-error'"
- ❌ BAD: mat-icon-button without aria-label

## Component Reuse Calculation

```
Component Reuse Score = (Material Components / Total Components) × 100%

Thresholds:
- 🟢 Excellent: ≥80%
- 🟡 Good: 60-79%
- 🔴 Needs Improvement: <60%
```

## Output Format

```markdown
## Component System Audit

**Framework Detected**: Angular + Angular Material

**Component Consistency**:
- ✅ **Used Correctly**: mat-button (15), mat-input (10), mat-card (8), mat-table (3) = 36 components
- ❌ **Custom Reimplementation**:
  - "CustomButton" duplicates mat-button functionality
  - "CustomDatepicker" reimplements mat-datepicker
- ⚠️ **Inconsistent Usage**:
  - Mix of mat-button (12) and custom <button> (3)
  - Mix of mat-table (2) and custom <table> (1)

**Design Token Adherence**:
- ✅ **Colors**: [X]% using color="primary/accent/warn"
- ❌ **Colors**: [Y]% hardcoded (#3f51b5, #ff4081) - should use theme
- ✅ **Spacing**: [X]% using Material 8dp grid
- ⚠️ **Typography**: [X]% using mat-typography classes

**Recommendations**:
1. **Replace CustomButton with mat-button**: Use `<button mat-raised-button color="primary">` instead
   - Impact: Consistent Material Design ripple, focus states
   - Effort: 1-2 hours
2. **Standardize on mat-table**: Remove custom table implementations
   - Impact: Adds matSort, mat-paginator, accessibility out-of-box
   - Effort: 1-2 days
3. **Replace hardcoded colors with theme**: Change `color: '#3f51b5'` → `color="primary"`
   - Impact: Enables theme switching
   - Effort: 1 day

**Component Reuse Score**: [X]% ([Excellent/Good/Needs Improvement])
```

## MCP Integration

**Not available for Angular Material** (no MCP server at this time)

Refer to official Angular Material documentation: https://material.angular.io/

---

**Template Version**: 2.2.0
**Last Updated**: 2025-01-16
**Compatibility**: SpecKit v2.2+
