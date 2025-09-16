# Architecture & Code Structure Guidelines

## Project Structure

### Frontend Architecture
- **Framework**: React with TypeScript
- **Build Tool**: Vite
- **Styling**: Tailwind CSS + shadcn/ui
- **State Management**: React hooks + Zustand for complex state
- **Routing**: React Router
- **Backend**: Supabase (PostgreSQL + Edge Functions)

### Directory Organization
```
src/
├── components/          # Reusable UI components
│   ├── ui/             # shadcn/ui components
│   ├── forms/          # Form-related components
│   ├── layout/         # Layout components
│   └── [feature]/      # Feature-specific components
├── hooks/              # Custom React hooks
├── pages/              # Route components
├── stores/             # State management
├── types/              # TypeScript type definitions
├── utils/              # Utility functions
├── lib/                # Third-party library configurations
├── contexts/           # React contexts
└── integrations/       # External service integrations
```

## Component Architecture

### Component Types
1. **UI Components** (`src/components/ui/`)
   - Pure presentation components
   - Use shadcn/ui components
   - No business logic

2. **Feature Components** (`src/components/[feature]/`)
   - Feature-specific components
   - May contain business logic
   - Connect to data sources

3. **Page Components** (`src/pages/`)
   - Route-level components
   - Orchestrate feature components
   - Handle routing logic

### Component Guidelines
- Use functional components with hooks
- Implement proper TypeScript typing
- Follow single responsibility principle
- Keep components focused and reusable
- Use proper prop interfaces

## State Management

### React State
- Use `useState` for local component state
- Use `useReducer` for complex local state
- Use `useContext` for shared state across components

### Global State
- Use Zustand for complex global state
- Keep state minimal and focused
- Use custom hooks to access global state
- Implement proper state updates

### Data Fetching
- Use custom hooks for API calls
- Implement proper loading and error states
- Use React Query or SWR for caching (if needed)
- Handle offline scenarios gracefully

## API Integration

### Supabase Integration
- Use Supabase client for database operations
- Implement Row Level Security (RLS) policies
- Use Edge Functions for server-side logic
- Handle authentication properly

### Custom Hooks Pattern
```typescript
// Example custom hook
export function usePatients() {
  const [patients, setPatients] = useState<Patient[]>([]);
  const [loading, setLoading] = useState(false);
  
  // Implementation...
  
  return { patients, loading, refetch };
}
```

## TypeScript Guidelines

### Type Definitions
- Define interfaces for all data structures
- Use proper generic types
- Avoid `any` type usage
- Export types from `src/types/`

### Component Props
- Define explicit prop interfaces
- Use optional props when appropriate
- Provide default values where needed
- Document complex prop types

### API Types
- Generate types from Supabase schema
- Keep types in sync with database
- Use discriminated unions for variants
- Implement proper error types

## Error Handling

### Error Boundaries
- Implement error boundaries for critical sections
- Provide fallback UI for errors
- Log errors appropriately
- Handle network errors gracefully

### Validation
- Validate data at component boundaries
- Use TypeScript for compile-time validation
- Implement runtime validation for external data
- Provide clear error messages

## Performance Guidelines

### Code Splitting
- Use dynamic imports for large components
- Implement route-based code splitting
- Lazy load non-critical components
- Optimize bundle size

### Rendering Optimization
- Use React.memo for expensive components
- Implement proper dependency arrays
- Avoid unnecessary re-renders
- Use useMemo and useCallback appropriately

### Asset Optimization
- Optimize images and assets
- Use proper image formats
- Implement lazy loading for images
- Minimize bundle size

## Security Guidelines

### Authentication
- Use Supabase Auth for authentication
- Implement proper session management
- Handle token refresh automatically
- Protect sensitive routes

### Data Validation
- Validate all user inputs
- Sanitize data before database operations
- Implement proper CORS policies
- Use HTTPS in production

### Environment Variables
- Never expose sensitive data in frontend
- Use environment variables for configuration
- Implement proper secret management
- Keep API keys secure
