# React + Material-UI (MUI) Component Audit Template

## Framework Information

**Framework**: React
**Component Library**: Material-UI (MUI) v5+
**Theme System**: `@mui/material/styles` with `createTheme()`
**Documentation**: https://mui.com/material-ui/

## Common Components Checklist

### Buttons & Actions
- ✅ Button (contained, outlined, text variants)
- ✅ IconButton
- ✅ Fab (Floating Action Button)
- ✅ ToggleButton / ToggleButtonGroup
- ✅ LoadingButton (@mui/lab)
- ✅ ButtonGroup

### Form Inputs
- ✅ TextField (standard, filled, outlined)
- ✅ Select
- ✅ Autocomplete
- ✅ Checkbox
- ✅ Radio / RadioGroup
- ✅ Switch
- ✅ Slider
- ✅ Rating
- ✅ DatePicker / DateTimePicker / TimePicker (@mui/x-date-pickers)

### Navigation
- ✅ AppBar
- ✅ Drawer (permanent, persistent, temporary)
- ✅ Menu / MenuItem
- ✅ Tabs / Tab
- ✅ Breadcrumbs
- ✅ Pagination
- ✅ BottomNavigation
- ✅ Stepper / Step

### Layout
- ✅ Container (fixed, fluid)
- ✅ Grid (v1: Grid, v2: Grid2)
- ✅ Stack (flexbox utility)
- ✅ Box (generic container)
- ✅ Card / CardContent / CardActions
- ✅ Paper
- ✅ Divider
- ✅ Accordion / AccordionSummary / AccordionDetails

### Feedback
- ✅ Alert / AlertTitle
- ✅ Snackbar
- ✅ Dialog / DialogTitle / DialogContent / DialogActions
- ✅ Tooltip
- ✅ CircularProgress
- ✅ LinearProgress
- ✅ Skeleton
- ✅ Backdrop
- ✅ Badge

### Data Display
- ✅ Table / TableHead / TableBody / TableRow / TableCell
- ✅ DataGrid (@mui/x-data-grid)
- ✅ List / ListItem / ListItemText / ListItemIcon
- ✅ Chip
- ✅ Avatar / AvatarGroup
- ✅ Typography
- ✅ Icon (@mui/icons-material)

### Overlays
- ✅ Modal
- ✅ Popover
- ✅ Drawer (also in Navigation)
- ✅ Backdrop (also in Feedback)

## Design Token Checks

### Colors
**Theme References** (CORRECT):
- `theme.palette.primary.main` / `.light` / `.dark`
- `theme.palette.secondary.main` / `.light` / `.dark`
- `theme.palette.error.main` / `.light` / `.dark`
- `theme.palette.warning.main` / `.light` / `.dark`
- `theme.palette.info.main` / `.light` / `.dark`
- `theme.palette.success.main` / `.light` / `.dark`
- `theme.palette.background.default` / `.paper`
- `theme.palette.text.primary` / `.secondary` / `.disabled`
- `theme.palette.grey[500]` (50-900 scale)

**Hardcoded Values** (INCORRECT):
- `#1976d2`, `#dc004e`, `rgb(25, 118, 210)`, etc.

### Spacing
**Theme References** (CORRECT):
- `theme.spacing(1)` = 8px (default scale)
- `theme.spacing(2)` = 16px
- `theme.spacing(3)` = 24px
- `theme.spacing(0.5)` = 4px
- `sx={{ p: 2, m: 1 }}` (shorthand in sx prop)

**Hardcoded Values** (INCORRECT):
- `padding: '16px'`, `margin: '8px'`, `gap: '24px'`

### Typography
**Theme References** (CORRECT):
- `theme.typography.h1` through `h6`
- `theme.typography.subtitle1` / `subtitle2`
- `theme.typography.body1` / `body2`
- `theme.typography.button`
- `theme.typography.caption`
- `theme.typography.overline`
- `<Typography variant="h1">` (component variant)

**Hardcoded Values** (INCORRECT):
- `font-size: 24px`, `font-weight: 700`, `line-height: 1.5`

### Shadows
**Theme References** (CORRECT):
- `theme.shadows[0]` through `theme.shadows[24]`
- `elevation={2}` (on Paper, Card)

**Hardcoded Values** (INCORRECT):
- `box-shadow: 0 2px 4px rgba(0,0,0,0.1)`

### Breakpoints
**Theme References** (CORRECT):
- `theme.breakpoints.up('sm')` (600px+)
- `theme.breakpoints.down('md')` (<900px)
- `theme.breakpoints.between('sm', 'lg')`
- `sx={{ display: { xs: 'none', md: 'block' } }}` (responsive sx)

**Hardcoded Values** (INCORRECT):
- `@media (max-width: 768px)`, `@media (min-width: 1024px)`

## Audit Questions

### 1. Component Usage
**Question**: Does the UI-SPEC specify MUI components by name (e.g., "TextField", "Button", "DataGrid") or describe custom implementations?

**Look for**:
- ✅ GOOD: "Use MUI TextField with outlined variant"
- ✅ GOOD: "MUI DataGrid with sortable columns"
- ❌ BAD: "Custom input field with blue border and rounded corners"
- ⚠️ INCONSISTENT: Mix of MUI Button (some screens) and custom <button> (other screens)

### 2. Design Token Adherence
**Question**: Are theme references specified or hardcoded values used?

**Look for**:
- ✅ GOOD: "Primary color (theme.palette.primary.main)"
- ✅ GOOD: "Spacing: theme.spacing(2) between elements"
- ❌ BAD: "Color: #1976d2", "Padding: 16px"
- ❌ BAD: "Font size: 24px, weight: 700"

### 3. Component Props Consistency
**Question**: Are component props consistent with MUI documentation?

**Look for**:
- ✅ GOOD: "Button variant='contained' color='primary'"
- ✅ GOOD: "TextField error={true} helperText='Invalid input'"
- ❌ BAD: "Button style='filled'" (MUI uses 'contained', not 'filled')
- ❌ BAD: "TextField errorMessage='...'" (MUI uses 'helperText', not 'errorMessage')

### 4. Responsive Design
**Question**: Is responsive behavior specified using MUI breakpoints?

**Look for**:
- ✅ GOOD: "Grid xs={12} md={6} (full width on mobile, half on desktop)"
- ✅ GOOD: "Hide drawer on mobile (display: { xs: 'none', md: 'block' })"
- ❌ BAD: "Desktop layout at 1024px, mobile below 768px" (not using MUI breakpoints)

### 5. Accessibility
**Question**: Are accessibility props specified per MUI guidance?

**Look for**:
- ✅ GOOD: "IconButton aria-label='Delete item'"
- ✅ GOOD: "TextField inputProps={{ 'aria-describedby': 'helper-text' }}"
- ❌ BAD: IconButton without aria-label
- ❌ BAD: Custom form fields without ARIA attributes

### 6. Theme Customization
**Question**: Is theme customization specified or are inline styles used?

**Look for**:
- ✅ GOOD: "Customize theme.palette.primary to match brand colors"
- ✅ GOOD: "Override MUI Button styles via theme.components.MuiButton"
- ❌ BAD: "Apply inline styles to every Button instance"
- ⚠️ WARNING: Excessive use of sx prop for global styles (should use theme)

## Component Reuse Calculation

```
Component Reuse Score = (MUI Library Components / Total Components) × 100%

Example:
- MUI Components: Button (15), TextField (10), Card (8), DataGrid (3) = 36 components
- Custom Components: BlueButton (5), CustomTable (2) = 7 components
- Total: 43 components

Score = (36 / 43) × 100% = 83.7% (Excellent)
```

**Thresholds**:
- 🟢 **Excellent**: ≥80% (high library usage, minimal custom components)
- 🟡 **Good**: 60-79% (mostly library, some custom where justified)
- 🔴 **Needs Improvement**: <60% (too many custom reimplementations)

## Output Format

```markdown
## Component System Audit

**Framework Detected**: React + Material-UI (MUI) v5+

**Component Consistency**:
- ✅ **Used Correctly**: Button (15), TextField (10), Card (8), DataGrid (3), Select (5) = 41 components
- ❌ **Custom Reimplementation**:
  - "BlueButton" duplicates MUI Button with hardcoded color
  - "CustomDatePicker" reimplements MUI DatePicker functionality
- ⚠️ **Inconsistent Usage**:
  - Mix of MUI Button (12) and custom <button> (3) for primary actions
  - Mix of MUI DataGrid (2) and custom <table> (1) for data tables

**Design Token Adherence**:
- ✅ **Colors**: 85% using theme.palette references
- ❌ **Colors**: 15% hardcoded (#1976d2, #dc004e) - should use theme.palette
- ✅ **Spacing**: 90% using theme.spacing() or sx prop
- ⚠️ **Typography**: 70% using Typography component (30% custom font sizes)

**Recommendations**:
1. **Replace BlueButton with MUI Button**: Use `<Button variant="contained" color="primary">` instead of custom component
   - Impact: Reduces maintenance, ensures theme consistency
   - Effort: 1-2 hours (find/replace across 5 screens)
2. **Standardize on MUI DataGrid**: Remove custom table implementations
   - Impact: Adds sorting, filtering, pagination out-of-box
   - Effort: 1-2 days (migrate custom table to DataGrid)
3. **Replace hardcoded colors with theme tokens**:
   - Change `color: '#1976d2'` → `theme.palette.primary.main`
   - Change `color: '#dc004e'` → `theme.palette.error.main`
   - Impact: Enables theme switching (light/dark mode)
   - Effort: 1 day (find/replace + testing)
4. **Use Typography component consistently**: Replace `<span style={{ fontSize: 24 }}>` with `<Typography variant="h5">`
   - Impact: Consistent type scale, responsive typography
   - Effort: 4-6 hours

**Component Reuse Score**: 83.7% (🟢 Excellent)
```

## MCP Integration

**Not available for MUI** (no MCP server at this time)

If MCP were available, queries would look like:
```
Tool: mcp__mui-assistant__mui_assistant (hypothetical)
Query: "What are the required props and accessibility features for DataGrid?"
Component: "DataGrid"
```

For now, refer to official MUI documentation: https://mui.com/material-ui/

---

**Template Version**: 2.2.0
**Last Updated**: 2025-01-16
**Compatibility**: SpecKit v2.2+
