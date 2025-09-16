# UI Guidelines

## Component Library

### Mandatory: shadcn/ui Components
- **ALWAYS** use shadcn/ui components for all UI elements
- Do not create custom UI components unless absolutely necessary
- If a component doesn't exist in shadcn/ui, first check if it can be composed from existing components
- Only create custom components after consulting the shadcn/ui documentation and confirming the component doesn't exist

### Component Usage Rules
- Use the existing component variants and props before creating custom styles
- Maintain consistency with the established design system
- Follow the shadcn/ui component patterns and conventions
- Use Tailwind CSS classes for styling, following the existing configuration

### Available Components
The project includes these shadcn/ui components:
- Button, Input, Label, Textarea
- Dialog, Sheet, Popover, Tooltip
- Card, Badge, Avatar
- Table, Select, Checkbox, RadioGroup
- Form, Alert, Separator
- And many more in the `src/components/ui/` directory

### Customization
- Only customize shadcn/ui components through CSS variables and Tailwind classes
- Maintain the component's accessibility features
- Test customizations across different screen sizes

## Design Principles

### Layout
- Use responsive design patterns
- Follow mobile-first approach
- Maintain consistent spacing using Tailwind's spacing scale
- Use CSS Grid and Flexbox for layouts

### Typography
- Use the established typography scale
- Maintain proper heading hierarchy
- Ensure good contrast ratios for accessibility

### Color Scheme
- Use the defined color palette
- Follow semantic color usage (success, error, warning, info)
- Maintain dark/light mode compatibility where applicable

### Content Guidelines
- **NEVER use emojis** in the user interface
- Keep text content professional and clean
- Use appropriate icons from the icon library instead of emojis

### Accessibility
- Ensure all interactive elements are keyboard accessible
- Use proper ARIA labels and roles
- Maintain sufficient color contrast
- Test with screen readers when possible

## File Organization
- UI components should be placed in `src/components/ui/`
- Feature-specific components go in their respective feature directories
- Keep components focused and single-purpose
- Use TypeScript for all component definitions
