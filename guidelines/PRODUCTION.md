# Production Deployment Guidelines

## Environment Configuration

### Production Environment Files
- **`.env.production`**: Production configuration (committed to repository)
- **Environment Variables**:
  ```
  VITE_SUPABASE_URL=https://dbqbuhnksoykgdxlodjw.supabase.co
  VITE_SUPABASE_ANON_KEY=[production-anon-key]
  VITE_SUPABASE_PROJECT_ID="dbqbuhnksoykgdxlodjw"
  ```

### Build Process
- **Production Build**: `npm run build` (uses `.env.production`)
- **Build Output**: `dist/` directory
- **Environment Mode**: `--mode production`
- **Optimization**: Minified, tree-shaken, optimized for production

## Deployment Checklist

### Pre-Deployment
- [ ] Verify `.env.production` has correct production URLs
- [ ] Ensure all environment variables are set
- [ ] Run production build locally: `npm run build`
- [ ] Test production build: `npm run preview`
- [ ] Verify no development URLs in production build
- [ ] Check bundle size and performance

### Database
- [ ] Production Supabase instance is running
- [ ] Database migrations are applied
- [ ] Edge functions are deployed
- [ ] Database backups are configured
- [ ] Monitoring is set up

### Security
- [ ] Environment variables are properly secured
- [ ] API keys are production-ready
- [ ] CORS settings are configured
- [ ] Authentication is working
- [ ] HTTPS is enforced

## Build Scripts

### Available Commands
- **`npm run build`**: Production build with optimization
- **`npm run build:dev`**: Development build (for testing)
- **`npm run build:local`**: Local build (for local testing)
- **`npm run preview`**: Preview production build locally

### Build Configuration
- **Mode**: `production` (uses `.env.production`)
- **Output**: `dist/` directory
- **Source Maps**: Disabled in production
- **Minification**: Enabled
- **Tree Shaking**: Enabled

## Environment Variables Priority

### Vite Environment Loading Order
1. `.env.local` (highest priority - local overrides)
2. `.env.production` (when `--mode production`)
3. `.env.development` (when `--mode development`)
4. `.env` (lowest priority - safe defaults)

### Production-Specific Variables
- `VITE_SUPABASE_URL`: Production Supabase URL
- `VITE_SUPABASE_ANON_KEY`: Production anon key
- `VITE_SUPABASE_PROJECT_ID`: Production project ID
- `VITE_APP_ENV`: Set to `production`

## Deployment Platforms

### Vercel
- Automatic builds on git push
- Environment variables in Vercel dashboard
- Automatic HTTPS and CDN

### Netlify
- Build command: `npm run build`
- Publish directory: `dist`
- Environment variables in Netlify dashboard

### Manual Deployment
- Build locally: `npm run build`
- Upload `dist/` contents to web server
- Configure web server for SPA routing
- Set up HTTPS and security headers

## Monitoring & Maintenance

### Health Checks
- Monitor application uptime
- Check database connectivity
- Verify API endpoints
- Monitor error rates

### Performance
- Monitor Core Web Vitals
- Check bundle size
- Monitor API response times
- Track user experience metrics

### Security
- Regular security audits
- Monitor for vulnerabilities
- Update dependencies regularly
- Review access logs

## Rollback Strategy

### Quick Rollback
- Keep previous build artifacts
- Database migration rollback procedures
- Environment variable rollback
- DNS/CDN cache invalidation

### Emergency Procedures
- Contact information for team members
- Escalation procedures
- Communication plan
- Recovery time objectives

## Best Practices

### Code Quality
- All code must pass linting
- TypeScript strict mode enabled
- No console.log statements in production
- Proper error handling and logging

### Performance
- Optimize images and assets
- Use lazy loading where appropriate
- Implement proper caching strategies
- Monitor and optimize bundle size

### Security
- Never commit sensitive data
- Use environment variables for configuration
- Implement proper authentication
- Regular security updates