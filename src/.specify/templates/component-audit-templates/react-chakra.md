# React + Chakra UI Component Audit Template

## Framework Information

**Framework**: React
**Component Library**: Chakra UI v2+
**Theme System**: `@chakra-ui/react` with `extendTheme()`
**Documentation**: https://chakra-ui.com/

## Common Components Checklist

### Buttons & Actions
- ✅ Button (solid, outline, ghost, link variants)
- ✅ IconButton
- ✅ CloseButton
- ✅ ButtonGroup

### Form Inputs
- ✅ Input
- ✅ Textarea
- ✅ Select
- ✅ Checkbox / CheckboxGroup
- ✅ Radio / RadioGroup
- ✅ Switch
- ✅ Slider
- ✅ PinInput
- ✅ NumberInput
- ✅ Editable

### Navigation
- ✅ Breadcrumb / BreadcrumbItem
- ✅ Link
- ✅ Tabs / TabList / TabPanels / Tab / TabPanel
- ✅ Stepper / Step

### Layout
- ✅ Box (generic container)
- ✅ Container
- ✅ Flex
- ✅ Grid / GridItem
- ✅ Stack / HStack / VStack
- ✅ Wrap / WrapItem
- ✅ Center / Square / Circle
- ✅ Spacer
- ✅ Divider
- ✅ Card / CardHeader / CardBody / CardFooter

### Feedback
- ✅ Alert / AlertIcon / AlertTitle / AlertDescription
- ✅ Toast (via useToast hook)
- ✅ Modal / ModalOverlay / ModalContent / ModalHeader / ModalBody / ModalFooter
- ✅ Drawer / DrawerOverlay / DrawerContent / DrawerHeader / DrawerBody / DrawerFooter
- ✅ Popover / PopoverTrigger / PopoverContent / PopoverHeader / PopoverBody
- ✅ Tooltip
- ✅ Progress (linear)
- ✅ CircularProgress
- ✅ Spinner
- ✅ Skeleton / SkeletonText / SkeletonCircle

### Data Display
- ✅ Table / Thead / Tbody / Tr / Th / Td
- ✅ List / ListItem / ListIcon
- ✅ Tag / TagLabel / TagCloseButton
- ✅ Badge
- ✅ Avatar / AvatarGroup
- ✅ Image
- ✅ Icon
- ✅ Code / CodeBlock
- ✅ Kbd (keyboard key)
- ✅ Stat / StatLabel / StatNumber / StatHelpText

### Overlays
- ✅ Modal (also in Feedback)
- ✅ Drawer (also in Feedback)
- ✅ Popover (also in Feedback)
- ✅ Menu / MenuButton / MenuList / MenuItem

## Design Token Checks

### Colors
**Theme References** (CORRECT):
- `theme.colors.blue.500` (50-900 scale)
- `theme.colors.red.500`, `theme.colors.green.500`, etc.
- `theme.colors.gray.100` through `gray.900`
- Shorthand in props: `color="blue.500"`, `bg="red.50"`

**Hardcoded Values** (INCORRECT):
- `#3182ce`, `#e53e3e`, `rgb(49, 130, 206)`

### Spacing
**Theme References** (CORRECT):
- `theme.space[4]` = 1rem (16px default)
- `theme.space[2]` = 0.5rem (8px)
- Shorthand in props: `p={4}`, `m={2}`, `gap={3}`

**Hardcoded Values** (INCORRECT):
- `padding: '16px'`, `margin: '8px'`

### Typography
**Theme References** (CORRECT):
- `theme.fontSizes.xs` through `4xl`
- `theme.fontWeights.normal`, `.medium`, `.bold`, `.extrabold`
- Shorthand in props: `fontSize="lg"`, `fontWeight="bold"`

**Hardcoded Values** (INCORRECT):
- `font-size: 18px`, `font-weight: 700`

### Shadows
**Theme References** (CORRECT):
- `theme.shadows.sm`, `.md`, `.lg`, `.xl`, `.2xl`
- Shorthand in props: `boxShadow="md"`

**Hardcoded Values** (INCORRECT):
- `box-shadow: 0 2px 4px rgba(0,0,0,0.1)`

### Breakpoints
**Theme References** (CORRECT):
- `theme.breakpoints.sm` (30em / 480px)
- `theme.breakpoints.md` (48em / 768px)
- `theme.breakpoints.lg` (62em / 992px)
- Responsive props: `<Box display={{ base: 'none', md: 'block' }}>`

**Hardcoded Values** (INCORRECT):
- `@media (max-width: 768px)`

## Audit Questions

### 1. Component Usage
Does the UI-SPEC specify Chakra UI components by name or describe custom implementations?

**Look for**:
- ✅ GOOD: "Use Chakra Input with outline variant"
- ✅ GOOD: "Chakra Table with hover effects"
- ❌ BAD: "Custom input with blue border"
- ⚠️ INCONSISTENT: Mix of Chakra Button and custom <button>

### 2. Design Token Adherence
Are theme references specified or hardcoded values used?

**Look for**:
- ✅ GOOD: "Color: blue.500", "Spacing: space[4]"
- ❌ BAD: "Color: #3182ce", "Padding: 16px"

### 3. Component Props Consistency
Are component props consistent with Chakra documentation?

**Look for**:
- ✅ GOOD: "Button variant='solid' colorScheme='blue'"
- ❌ BAD: "Button style='filled'" (Chakra uses 'solid', not 'filled')

### 4. Responsive Design
Is responsive behavior specified using Chakra breakpoints?

**Look for**:
- ✅ GOOD: "Stack direction={{ base: 'column', md: 'row' }}"
- ❌ BAD: "Desktop layout at 1024px" (not using Chakra breakpoints)

### 5. Accessibility
Are accessibility props specified per Chakra guidance?

**Look for**:
- ✅ GOOD: "IconButton aria-label='Delete item'"
- ❌ BAD: IconButton without aria-label

## Component Reuse Calculation

```
Component Reuse Score = (Chakra Components / Total Components) × 100%

Thresholds:
- 🟢 Excellent: ≥80%
- 🟡 Good: 60-79%
- 🔴 Needs Improvement: <60%
```

## Output Format

```markdown
## Component System Audit

**Framework Detected**: React + Chakra UI v2+

**Component Consistency**:
- ✅ **Used Correctly**: Button (12), Input (8), Card (6), Table (2) = 28 components
- ❌ **Custom Reimplementation**: [List custom components duplicating Chakra functionality]
- ⚠️ **Inconsistent Usage**: [List inconsistencies]

**Design Token Adherence**:
- ✅ **Colors**: [X]% using theme.colors references
- ✅ **Spacing**: [X]% using theme.space
- ⚠️ **Typography**: [X]% using theme tokens

**Recommendations**:
1. [Replace custom X with Chakra Y]
2. [Standardize on Chakra Z]
3. [Replace hardcoded values with tokens]

**Component Reuse Score**: [X]% ([Excellent/Good/Needs Improvement])
```

## MCP Integration

**Not available for Chakra UI** (no MCP server at this time)

Refer to official Chakra UI documentation: https://chakra-ui.com/

---

**Template Version**: 2.2.0
**Last Updated**: 2025-01-16
**Compatibility**: SpecKit v2.2+
