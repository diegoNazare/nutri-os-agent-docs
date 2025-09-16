# Nutri-OS Project Guidelines

## Overview
This document serves as the central hub for all project guidelines and development standards. These guidelines ensure consistency across development, design, and production environments for both Lovable and Cursor AI assistants.

## Quick Reference

### 🎨 UI Development
**File**: [`UI_GUIDELINES.md`](./guidelines/UI_GUIDELINES.md)
- Always use shadcn/ui components
- Follow established design patterns
- Maintain accessibility standards

### 💻 Local Development
**File**: [`LOCAL_DEVELOPMENT.md`](./guidelines/LOCAL_DEVELOPMENT.md)
- Always use Supabase local for development
- Follow local setup procedures
- Maintain environment consistency

### 🌿 Git & Branching
**File**: [`GIT_BRANCHING.md`](./guidelines/GIT_BRANCHING.md)
- All Git operations in English only
- Never commit without explicit user request
- Follow established branching strategy

### 🏗️ Architecture
**File**: [`ARCHITECTURE.md`](./guidelines/ARCHITECTURE.md)
- React + TypeScript + Supabase stack
- Follow component architecture patterns
- Implement proper state management

### 🚀 Production
**File**: [`PRODUCTION.md`](./guidelines/PRODUCTION.md)
- Environment separation strategy
- Security and monitoring guidelines
- Performance optimization standards

## Core Principles

### 1. Consistency
- Follow established patterns and conventions
- Maintain code style across the project
- Use consistent naming and structure

### 2. Quality
- Write clean, maintainable code
- Implement proper error handling
- Follow security best practices

### 3. Documentation
- Keep documentation up to date
- Use English for all documentation
- Document decisions and rationale

### 4. Testing
- Test changes thoroughly before deployment
- Maintain test coverage for critical functionality
- Use local environment for testing

## Technology Stack

### Frontend
- **React 18** with TypeScript
- **Vite** for build tooling
- **Tailwind CSS** for styling
- **shadcn/ui** for component library
- **React Router** for navigation

### Backend
- **Supabase** (PostgreSQL + Edge Functions)
- **Row Level Security** for data protection
- **Real-time subscriptions** for live updates

### Development Tools
- **ESLint** for code linting
- **Prettier** for code formatting
- **TypeScript** for type safety
- **Git** for version control

## Usage Instructions for AI Assistants

### For Lovable
When working with this project:
1. Reference the appropriate guideline file for the task
2. Follow the established patterns and conventions
3. Use the specified technology stack
4. Maintain consistency with existing code

### For Cursor
When making changes:
1. Check the relevant guideline file first
2. Follow the architectural patterns
3. Use the correct development environment
4. Never make commits without explicit request

## File Organization

```
.agents/
├── PROJECT_GUIDELINES.md          # This file - main overview
└── guidelines/
    ├── UI_GUIDELINES.md           # UI/UX standards
    ├── LOCAL_DEVELOPMENT.md       # Development environment
    ├── GIT_BRANCHING.md          # Git workflow
    ├── ARCHITECTURE.md           # Code structure
    └── PRODUCTION.md             # Production deployment
```

## Getting Started

### New Developers
1. Read this overview document
2. Review the specific guideline files
3. Set up local development environment
4. Follow the established workflow

### AI Assistants
1. Identify the task type (UI, development, architecture, etc.)
2. Reference the appropriate guideline file
3. Follow the specified patterns and conventions
4. Maintain consistency with existing code

## Maintenance

### Updating Guidelines
- Guidelines are living documents
- Update when patterns change
- Document new conventions
- Keep all files synchronized

### Version Control
- Track changes to guideline files
- Use descriptive commit messages
- Review changes before merging
- Keep guidelines in sync with code

## Support

### Questions
- Check the relevant guideline file first
- Look for examples in existing code
- Ask for clarification when needed
- Document new patterns as they emerge

### Issues
- Report guideline inconsistencies
- Suggest improvements to processes
- Update documentation as needed
- Maintain the quality standards
