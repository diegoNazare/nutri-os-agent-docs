# Local Development Guidelines

## Database & Backend

### Supabase Local Development
- **MANDATORY**: Always use Supabase local for development
- Never connect to production Supabase instance during development
- Use `supabase start` to run local Supabase stack
- Ensure local database is running before starting the frontend

### Local Setup Commands
```bash
# Start Supabase local stack
supabase start

# Reset local database if needed
supabase db reset

# Generate types from local database
supabase gen types typescript --local > src/integrations/supabase/types.ts
```

### Environment Variables
- **Environment File Priority**: `.env.local` > `.env.production` > `.env`
- Use `.env.local` for local development overrides
- Never commit sensitive environment variables
- **Default Configuration** (`.env`):
  ```
  VITE_SUPABASE_URL=http://127.0.0.1:54321
  VITE_SUPABASE_ANON_KEY=[local-anon-key]
  VITE_SUPABASE_PROJECT_ID="dbqbuhnksoykgdxlodjw"
  ```
- **Local Override** (`.env.local`):
  ```
  VITE_SUPABASE_URL=http://127.0.0.1:54321
  VITE_SUPABASE_ANON_KEY=[local-anon-key]
  VITE_OPENAI_API_KEY=your_openai_api_key_here
  VITE_ASAAS_API_KEY=your_asaas_api_key_here
  VITE_APP_ENV=development
  ```

## Frontend Development

### Development Server
- Use `npm run dev` to start development server
- **Default port**: 8080 (configured in vite.config.ts)
- **URLs**: 
  - Local: http://localhost:8080
  - Network: http://192.168.1.66:8080
- Enable hot reload for efficient development
- Server automatically restarts when environment files change

### Build Scripts
- **`npm run dev`**: Development server (uses `.env.local`)
- **`npm run build`**: Production build (uses `.env.production`)
- **`npm run build:dev`**: Development build (uses `.env`)
- **`npm run build:local`**: Local build (uses `.env`)
- **`npm run build:preview`**: Build + preview production build locally (recommended for testing builds)
- **`npm run preview`**: Preview existing production build locally

### Environment Management
- **`.env`**: Safe development defaults (points to local Supabase)
- **`.env.local`**: Local overrides (highest priority, not committed)
- **`.env.production`**: Production configuration (used for production builds)
- **Never commit**: `.env.local` (contains personal API keys)
- **Always commit**: `.env` and `.env.production` (safe defaults)

### Code Quality
- Use ESLint for code linting
- Follow TypeScript strict mode
- Use Prettier for code formatting
- Run type checking before committing

### Testing
- Write unit tests for utility functions
- Test components with React Testing Library
- Use local Supabase for integration tests

## Development Workflow

### Before Starting Development
1. Ensure Supabase local is running
2. Pull latest changes from main branch
3. Install dependencies with `npm install`
4. Verify local environment is working

### During Development
1. Make small, focused changes
2. Test changes locally before committing
3. Use descriptive commit messages
4. Follow the established code patterns

### Code Organization
- Keep components in appropriate directories
- Use custom hooks for business logic
- Follow the established folder structure
- Maintain separation of concerns

## Database Migrations
- Create migrations for schema changes
- Test migrations on local database first
- Use descriptive migration names
- Never modify existing migrations

## API Development
- Use Supabase Edge Functions for serverless functions
- Test functions locally with `supabase functions serve`
- Follow RESTful API conventions
- Document API endpoints

## Troubleshooting

### Common Issues
- **Supabase won't start**: `supabase stop && supabase start`
- **Types are outdated**: regenerate from local database
- **Functions don't work**: check local function deployment
- **Wrong Supabase instance**: verify `.env.local` points to `http://127.0.0.1:54321`
- **Environment not loading**: restart dev server after changing `.env` files
- **Production data in dev**: check that `.env` has local Supabase URL, not production

### Performance
- Use React DevTools for performance debugging
- Monitor bundle size with build tools
- Optimize images and assets
- Use lazy loading where appropriate
