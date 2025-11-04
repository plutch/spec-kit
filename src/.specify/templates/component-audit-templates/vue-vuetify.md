# Vue + Vuetify Component Audit Template

## Framework Information

**Framework**: Vue 3
**Component Library**: Vuetify 3
**Theme System**: Vuetify theming with SCSS variables
**Documentation**: https://vuetifyjs.com/

## Common Components Checklist

### Buttons & Actions
- ✅ VBtn (button variants: elevated, flat, tonal, outlined, text, plain)
- ✅ VBtnToggle
- ✅ VFab (floating action button)
- ✅ VIcon

### Form Inputs
- ✅ VTextField
- ✅ VTextarea
- ✅ VSelect
- ✅ VAutocomplete
- ✅ VCombobox
- ✅ VCheckbox / VCheckboxBtn
- ✅ VRadio / VRadioGroup
- ✅ VSwitch
- ✅ VSlider / VRangeSlider
- ✅ VFileInput
- ✅ VRating
- ✅ VOtp (one-time password input)

### Navigation
- ✅ VAppBar
- ✅ VNavigationDrawer
- ✅ VBottomNavigation
- ✅ VBreadcrumbs
- ✅ VTabs / VTab
- ✅ VMenu
- ✅ VPagination
- ✅ VStepper

### Layout
- ✅ VApp (root application component)
- ✅ VMain (main content area)
- ✅ VContainer
- ✅ VRow / VCol (12-column grid)
- ✅ VCard / VCardTitle / VCardText / VCardActions
- ✅ VSheet (generic surface)
- ✅ VDivider
- ✅ VSpacer
- ✅ VExpansionPanels / VExpansionPanel

### Feedback
- ✅ VAlert
- ✅ VSnackbar
- ✅ VDialog
- ✅ VTooltip
- ✅ VProgressLinear
- ✅ VProgressCircular
- ✅ VSkeletonLoader
- ✅ VOverlay
- ✅ VBanner

### Data Display
- ✅ VTable (simple tables)
- ✅ VDataTable (advanced tables with sorting, filtering, pagination)
- ✅ VList / VListItem / VListItemTitle / VListItemSubtitle
- ✅ VChip / VChipGroup
- ✅ VBadge
- ✅ VAvatar
- ✅ VImg (optimized image component)
- ✅ VTimeline / VTimelineItem

### Overlays
- ✅ VDialog (also in Feedback)
- ✅ VMenu (also in Navigation)
- ✅ VOverlay (also in Feedback)
- ✅ VBottomSheet

## Design Token Checks

### Colors
**Vuetify Theme Variables** (CORRECT):
- `theme.colors.primary`, `.secondary`, `.accent`, `.error`, `.warning`, `.info`, `.success`
- Component props: `color="primary"`, `bg-color="secondary"`
- CSS: `rgb(var(--v-theme-primary))`

**Hardcoded Values** (INCORRECT):
- `#1867c0`, `#fb8c00`, `rgb(24, 103, 192)`

### Spacing
**Vuetify Spacing** (CORRECT):
- Spacing scale 0-16 (4px base unit)
- Classes: `pa-4` (padding all), `ma-2` (margin all), `px-3` (padding x-axis)
- Inline: `:style="{ padding: '4px' }"` (use spacing scale values)

**Hardcoded Values** (INCORRECT):
- Random spacing like 12px, 18px, 22px (not on 4px grid)

### Typography
**Vuetify Typography** (CORRECT):
- Typography classes: `.text-h1` through `.text-h6`, `.text-subtitle-1`, `.text-body-1`, `.text-caption`
- Component props: `<VBtn size="large">`, `<VTextField density="compact">`

**Hardcoded Values** (INCORRECT):
- `font-size: 16px`, `font-weight: 500`

### Elevation (Shadows)
**Vuetify Elevation** (CORRECT):
- Elevation classes: `.elevation-0` through `.elevation-24`
- Component props: `elevation="4"`

**Hardcoded Values** (INCORRECT):
- `box-shadow: 0 2px 4px rgba(0,0,0,0.2)`

### Breakpoints
**Vuetify Breakpoints** (CORRECT):
- Breakpoints: `xs` (<600px), `sm` (600-960px), `md` (960-1264px), `lg` (1264-1904px), `xl` (>1904px)
- Grid props: `<VCol cols="12" md="6">` (full width mobile, half on desktop)
- Display utilities: `d-none d-md-block` (hide on mobile, show on desktop)

**Hardcoded Values** (INCORRECT):
- `@media (max-width: 768px)`

## Audit Questions

### 1. Component Usage
Does the UI-SPEC specify Vuetify components by name or describe custom implementations?

**Look for**:
- ✅ GOOD: "Use VDataTable with sortable and filterable columns"
- ✅ GOOD: "VTextField with validation rules"
- ❌ BAD: "Custom input field with validation"
- ⚠️ INCONSISTENT: Mix of VBtn and custom <button>

### 2. Design Token Adherence
Are Vuetify theme variables specified or hardcoded values used?

**Look for**:
- ✅ GOOD: "Primary color (color='primary')"
- ✅ GOOD: "Spacing: pa-4 (16px padding)"
- ❌ BAD: "Color: #1867c0", "Padding: 18px"

### 3. Component Props Consistency
Are component props consistent with Vuetify documentation?

**Look for**:
- ✅ GOOD: "VBtn variant='elevated' color='primary'"
- ❌ BAD: "VBtn type='raised'" (Vuetify uses 'variant', not 'type')

### 4. Responsive Design
Is responsive behavior specified using Vuetify breakpoints?

**Look for**:
- ✅ GOOD: "VCol cols='12' md='6' (full width mobile, half desktop)"
- ✅ GOOD: "d-none d-md-block (hide mobile, show desktop)"
- ❌ BAD: Custom media queries not aligned with Vuetify breakpoints

### 5. Accessibility
Are accessibility features specified per Vuetify guidance?

**Look for**:
- ✅ GOOD: "VBtn aria-label='Delete item'"
- ✅ GOOD: "VTextField label='Email' hint='Enter your email'"
- ❌ BAD: VBtn without aria-label (when icon-only)

## Component Reuse Calculation

```
Component Reuse Score = (Vuetify Components / Total Components) × 100%

Thresholds:
- 🟢 Excellent: ≥80%
- 🟡 Good: 60-79%
- 🔴 Needs Improvement: <60%
```

## Output Format

```markdown
## Component System Audit

**Framework Detected**: Vue 3 + Vuetify 3

**Component Consistency**:
- ✅ **Used Correctly**: VBtn (12), VTextField (8), VCard (6), VDataTable (2) = 28 components
- ❌ **Custom Reimplementation**:
  - "CustomButton" duplicates VBtn functionality
  - "CustomTable" reimplements VDataTable
- ⚠️ **Inconsistent Usage**:
  - Mix of VBtn (10) and custom <button> (2)
  - Mix of VDataTable (1) and VTable (1) for similar data

**Design Token Adherence**:
- ✅ **Colors**: [X]% using color="primary/secondary/etc"
- ❌ **Colors**: [Y]% hardcoded (#1867c0, #fb8c00) - should use theme
- ✅ **Spacing**: [X]% using Vuetify spacing classes (pa-*, ma-*)
- ⚠️ **Typography**: [X]% using .text-* classes

**Recommendations**:
1. **Replace CustomButton with VBtn**: Use `<VBtn variant="elevated" color="primary">` instead
   - Impact: Consistent Material Design ripple, elevation states
   - Effort: 1-2 hours
2. **Standardize on VDataTable**: Remove custom table implementations
   - Impact: Adds sorting, filtering, pagination, selection out-of-box
   - Effort: 1-2 days
3. **Replace hardcoded colors with theme**: Change `color: '#1867c0'` → `color="primary"`
   - Impact: Enables theme switching (light/dark mode)
   - Effort: 1 day

**Component Reuse Score**: [X]% ([Excellent/Good/Needs Improvement])
```

## MCP Integration

**Not available for Vuetify** (no MCP server at this time)

Refer to official Vuetify documentation: https://vuetifyjs.com/

---

**Template Version**: 2.2.0
**Last Updated**: 2025-01-16
**Compatibility**: SpecKit v2.2+
