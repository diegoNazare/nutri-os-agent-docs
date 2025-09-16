# Git & Branching Guidelines

## Language Policy
- **MANDATORY**: All Git operations must be in English
- Commit messages must be in English
- Branch names must be in English
- Pull request titles and descriptions must be in English
- Documentation must be in English

## Branching Strategy

### Branch Naming Convention
- Use descriptive, English branch names
- Follow the pattern: `type/description`
- Types: `feature/`, `bugfix/`, `hotfix/`, `refactor/`, `docs/`
- Examples:
  - `feature/user-authentication`
  - `bugfix/login-validation`
  - `refactor/meal-plan-components`

### Main Branches
- `main`: Production-ready code
- `develop`: Integration branch for features (if using Git Flow)

### Feature Development
1. Create feature branch from `main`
2. Make commits with descriptive messages
3. Push branch and create pull request
4. Request code review
5. Merge after approval

## Commit Guidelines

### Commit Message Format
```
type(scope): description

[optional body]

[optional footer]
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

### Examples
```
feat(auth): add Google OAuth integration
fix(forms): resolve validation error on submit
docs(api): update authentication endpoints
refactor(ui): extract common button component
```

### Commit Rules
- **NEVER make commits without explicit user request**
- Each commit should represent a single logical change
- Keep commits small and focused
- Test changes before committing
- Use present tense ("add feature" not "added feature")

## Pull Request Guidelines

### PR Title
- Use descriptive, English titles
- Include the type of change
- Example: "feat: Add meal plan export functionality"

### PR Description
- Describe what changes were made
- Explain why the changes were necessary
- Include any breaking changes
- Add screenshots for UI changes
- Reference related issues

### Review Process
- Request review from team members
- Address all review comments
- Ensure CI/CD checks pass
- Test the changes locally before merging

## Git Workflow

### Before Starting Work
1. Pull latest changes from main
2. Create new branch from main
3. Ensure local environment is set up

### During Development
1. Make small, frequent commits
2. Push changes regularly
3. Keep branch up to date with main
4. Test changes thoroughly

### Before Merging
1. Squash commits if necessary
2. Update documentation
3. Ensure all tests pass
4. Get code review approval

## File Management

### .gitignore
- Keep .gitignore up to date
- Include common patterns for Node.js, TypeScript, and Supabase
- Never commit sensitive files (API keys, passwords)

### Large Files
- Use Git LFS for large files if needed
- Avoid committing binary files when possible
- Keep repository size manageable

## Emergency Procedures

### Hotfixes
- Create hotfix branch from main
- Fix the issue quickly
- Test thoroughly
- Merge directly to main
- Tag the release

### Rollbacks
- Use git revert for safe rollbacks
- Document the reason for rollback
- Notify team of rollback
- Create follow-up issue if needed
