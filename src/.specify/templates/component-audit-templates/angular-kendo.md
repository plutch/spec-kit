# Angular + Kendo UI Component Audit Template

## Framework Information

**Framework**: Angular
**Component Library**: Kendo UI for Angular
**Theme System**: Kendo Theme (Default, Bootstrap, Material)
**Documentation**: https://www.telerik.com/kendo-angular-ui/components/
**MCP Support**: ✅ YES (kendo-angular-assistant)

## Common Components Checklist

### Buttons & Actions
- ✅ Button (primary, secondary, tertiary, flat)
- ✅ ButtonGroup
- ✅ DropDownButton
- ✅ SplitButton
- ✅ FloatingActionButton
- ✅ Chip / ChipList

### Form Inputs
- ✅ TextBox
- ✅ TextArea
- ✅ NumericTextBox
- ✅ MaskedTextBox
- ✅ ColorPicker
- ✅ DatePicker / TimePicker / DateTimePicker / DateRangePicker
- ✅ DropDownList
- ✅ ComboBox
- ✅ AutoComplete
- ✅ MultiSelect
- ✅ Switch
- ✅ Checkbox
- ✅ RadioButton
- ✅ Slider / RangeSlider
- ✅ Rating

### Navigation
- ✅ AppBar
- ✅ Drawer
- ✅ Menu / ContextMenu
- ✅ TabStrip
- ✅ Breadcrumb
- ✅ Pager
- ✅ BottomNavigation
- ✅ Stepper

### Layout
- ✅ Card
- ✅ PanelBar
- ✅ Splitter
- ✅ TileLayout
- ✅ StackLayout
- ✅ GridLayout

### Feedback
- ✅ Dialog
- ✅ Window
- ✅ Notification
- ✅ Tooltip
- ✅ Popover
- ✅ Loader (linear, converging, infinite)
- ✅ ProgressBar
- ✅ Skeleton

### Data Display
- ✅ Grid (most powerful component)
- ✅ TreeList
- ✅ ListView
- ✅ Gantt
- ✅ Scheduler
- ✅ Avatar
- ✅ Badge
- ✅ Label
- ✅ Chart (extensive charting library)

### Overlays
- ✅ Dialog (also in Feedback)
- ✅ Window (also in Feedback)
- ✅ Popup
- ✅ Popover (also in Feedback)

## Design Token Checks

### Colors
**Kendo Theme Variables** (CORRECT):
- `$kendo-color-primary`
- `$kendo-color-secondary`
- `$kendo-color-tertiary`
- `$kendo-color-info`, `$kendo-color-success`, `$kendo-color-warning`, `$kendo-color-error`
- Component props: `themeColor="primary"`, `themeColor="secondary"`

**Hardcoded Values** (INCORRECT):
- `#ff6358`, `#28b4c8`, `rgb(255, 99, 88)`

### Spacing
**Kendo Theme Variables** (CORRECT):
- `$kendo-spacing-1` through `$kendo-spacing-24` (4px base scale)
- Component props with spacing utilities

**Hardcoded Values** (INCORRECT):
- `padding: 16px`, `margin: 8px`

### Typography
**Kendo Theme Variables** (CORRECT):
- `$kendo-font-size-xs` through `$kendo-font-size-3xl`
- `$kendo-font-weight-light`, `$kendo-font-weight-normal`, `$kendo-font-weight-bold`
- Component props: `size="small"`, `size="medium"`, `size="large"`

**Hardcoded Values** (INCORRECT):
- `font-size: 14px`, `font-weight: 600`

### Shadows
**Kendo Theme Variables** (CORRECT):
- `$kendo-elevation-1` through `$kendo-elevation-9`

**Hardcoded Values** (INCORRECT):
- `box-shadow: 0 2px 4px rgba(0,0,0,0.2)`

### Breakpoints
**Kendo Responsive Utilities** (CORRECT):
- Use Kendo Grid's responsive column definitions
- Use `@media` queries with documented breakpoints

## Audit Questions

### 1. Component Usage
Does the UI-SPEC specify Kendo UI components by name or describe custom implementations?

**Look for**:
- ✅ GOOD: "Use Kendo Grid with sorting and filtering"
- ✅ GOOD: "Kendo DatePicker with min/max range validation"
- ❌ BAD: "Custom data table with sort functionality"
- ⚠️ INCONSISTENT: Mix of Kendo Grid (some screens) and custom <table> (other screens)

### 2. Design Token Adherence
Are Kendo theme variables specified or hardcoded values used?

**Look for**:
- ✅ GOOD: "Primary color ($kendo-color-primary)"
- ✅ GOOD: "Button themeColor='primary'"
- ❌ BAD: "Color: #ff6358", "Padding: 16px"

### 3. Component Props Consistency
Are component props consistent with Kendo documentation?

**Look for**:
- ✅ GOOD: "Grid [data]='gridData' [pageable]='true' [sortable]='true'"
- ✅ GOOD: "DatePicker [format]='dd/MM/yyyy' [min]='minDate'"
- ❌ BAD: Incorrect prop names or unsupported configurations

### 4. Responsive Design
Is responsive behavior specified using Kendo Grid responsive features?

**Look for**:
- ✅ GOOD: "Grid columns with [hidden]='isHidden' based on viewport"
- ✅ GOOD: "Drawer with [mode]='mode' (overlay on mobile, push on desktop)"
- ❌ BAD: Custom media queries not aligned with Kendo responsive patterns

### 5. Accessibility
Are accessibility features specified per Kendo guidance?

**Look for**:
- ✅ GOOD: Proper ARIA labels on Kendo components
- ✅ GOOD: Keyboard navigation specified for Grid, Menu, TabStrip
- ❌ BAD: Missing accessibility attributes

### 6. Grid Usage Patterns
(Kendo Grid is the most powerful and commonly used component)

**Look for**:
- ✅ GOOD: Proper [data] binding with async data
- ✅ GOOD: Built-in features: [pageable], [sortable], [filterable], [groupable], [reorderable]
- ✅ GOOD: Virtual scrolling for large datasets: [scrollable]='virtual'
- ❌ BAD: Custom pagination/sorting/filtering instead of built-in Grid features

## Component Reuse Calculation

```
Component Reuse Score = (Kendo Components / Total Components) × 100%

Thresholds:
- 🟢 Excellent: ≥80%
- 🟡 Good: 60-79%
- 🔴 Needs Improvement: <60%
```

## Output Format

```markdown
## Component System Audit

**Framework Detected**: Angular + Kendo UI for Angular

**Component Consistency**:
- ✅ **Used Correctly**: Grid (5), DatePicker (8), Button (15), DropDownList (6) = 34 components
- ❌ **Custom Reimplementation**:
  - "CustomDataTable" duplicates Kendo Grid functionality
  - "CustomCalendar" reimplements DatePicker
- ⚠️ **Inconsistent Usage**:
  - Mix of Kendo Grid (3) and custom <table> (2) for data display
  - Mix of Kendo Button (12) and Angular Material Button (3)

**Design Token Adherence**:
- ✅ **Colors**: [X]% using $kendo-color-* variables or themeColor prop
- ❌ **Colors**: [Y]% hardcoded (#ff6358, #28b4c8) - should use theme
- ✅ **Spacing**: [X]% using $kendo-spacing-* variables
- ⚠️ **Typography**: [X]% using size props (small/medium/large)

**Recommendations** (Enhanced by MCP):
1. **Replace CustomDataTable with Kendo Grid**:
   [MCP Query: "What are the required props and best practices for Grid with sorting and filtering?"]
   [MCP Response: Official Grid documentation with [data], [sortable], [filterable] props]
   - Impact: Adds built-in sorting, filtering, paging, Excel export
   - Effort: 2-3 days (migrate custom table to Grid)
   - Reference: https://www.telerik.com/kendo-angular-ui/components/grid/

2. **Standardize on Kendo DatePicker**:
   [MCP Query: "What are the accessibility features and validation patterns for DatePicker?"]
   [MCP Response: ARIA support, min/max validation, format options]
   - Impact: Consistent date handling, built-in validation
   - Effort: 1-2 days

3. **Replace hardcoded colors with theme variables**:
   - Change `color: '#ff6358'` → `$kendo-color-primary`
   - Change inline styles → `themeColor="primary"` prop
   - Impact: Enables theme switching (Default/Bootstrap/Material themes)
   - Effort: 1-2 days

**Component Reuse Score**: [X]% ([Excellent/Good/Needs Improvement])
```

## MCP Integration

**MCP Server**: `kendo-angular-assistant`
**Status**: ✅ Available

### How to Use MCP

When analyzing Kendo UI specifications, the `/speckit.analyze-ux` command can optionally query the Kendo MCP server for official documentation and best practices.

**Tool**: `mcp__kendo-angular-assistant__kendo_angular_assistant`

**Parameters**:
- `query` (required): Specific question about component usage, props, accessibility, or patterns
- `component` (required): Kendo component name (e.g., "Grid", "DatePicker", "Button")

**Example Queries**:

1. **Grid Component**:
```typescript
query: "What are the required props and accessibility features for Grid with sorting and filtering?"
component: "Grid"
```

2. **DatePicker Component**:
```typescript
query: "How to implement min/max date validation and custom formats?"
component: "DatePicker"
```

3. **Button Component**:
```typescript
query: "What are the themeColor options and accessibility requirements?"
component: "Button"
```

### Graceful Fallback

If MCP server is not available:
- ✅ Template-based audit still runs
- ✅ Recommendations generated from template checklist
- ℹ️ Note added: "MCP not available - refer to https://www.telerik.com/kendo-angular-ui/"

### MCP Benefits

When MCP is available:
- 🎯 **Official Documentation**: Direct access to Telerik's component docs
- ✅ **Validated Props**: Ensures prop names and values match current Kendo version
- ♿ **Accessibility Guidance**: ARIA attributes, keyboard navigation patterns
- 🔧 **Best Practices**: Recommended patterns from Kendo team
- 📚 **Code Examples**: Before/after examples with correct syntax

---

**Template Version**: 2.2.0
**Last Updated**: 2025-01-16
**Compatibility**: SpecKit v2.2+
**MCP Compatibility**: kendo-angular-assistant v1.0+
