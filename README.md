# Nutri-OS Agent Documentation

This repository contains all documentation and guidelines for AI assistants (Lovable and Cursor) working on the Nutri-OS project. It is designed to be used as a shareable submodule across multiple projects.

## 📁 Repository Structure

```
.agents/
├── README.md                    # This file - repository overview
├── PROJECT_GUIDELINES.md        # Main project guidelines
├── guidelines/                  # Area-specific guidelines
│   ├── ARCHITECTURE.md         # Code architecture and structure
│   ├── GIT_BRANCHING.md        # Git workflow and branching
│   ├── LOCAL_DEVELOPMENT.md    # Local development environment
│   ├── PRODUCTION.md           # Deployment and production
│   └── UI_GUIDELINES.md        # UI/UX patterns
└── troubleshooting/            # Solutions for known issues
    └── MODAL_FREEZE_FIX.md     # Modal freeze solution
```

## 🎯 Purpose

This repository serves as a **single source of truth** for:

- **Development guidelines** for AI assistants
- **Code patterns** and architecture
- **Workflows** and established processes
- **Solutions** for known technical issues
- **UI/UX patterns** and design system

## 🚀 How to Use

### For AI Assistants (Lovable/Cursor)

1. **Always consult** the relevant file before making changes
2. **Follow guidelines** specific to each area (UI, architecture, etc.)
3. **Maintain consistency** with established patterns
4. **Never make commits** without explicit user request

### For Developers

1. **Read** `PROJECT_GUIDELINES.md` for overview
2. **Consult** specific guidelines as needed
3. **Update** documentation when patterns change
4. **Report** inconsistencies or improvement suggestions

## 📋 Main Guidelines

### 🎨 UI/UX
- **ALWAYS** use shadcn/ui components
- **NEVER** use emojis in the interface
- Maintain accessibility standards
- Follow the established design system

### 💻 Development
- Use Supabase local for development
- Follow TypeScript patterns rigorously
- Implement proper error handling
- Keep components focused and reusable

### 🌿 Git & Versioning
- **ALL** Git operations in English
- **NEVER** make commits without explicit request
- Follow established branching strategy
- Use descriptive commit messages

### 🏗️ Architecture
- React + TypeScript + Supabase
- Functional components with hooks
- Global state with Zustand when needed
- Custom hooks for business logic

## 🔧 Technology Stack

### Frontend
- **React 18** with TypeScript
- **Vite** for build
- **Tailwind CSS** + **shadcn/ui** for UI
- **React Router** for navigation

### Backend
- **Supabase** (PostgreSQL + Edge Functions)
- **Row Level Security** for data protection
- **Real-time subscriptions** for live updates

### Tools
- **ESLint** for linting
- **Prettier** for formatting
- **TypeScript** for type safety

## 🐛 Troubleshooting

### Known Issues
- **Modal Freeze**: Solution documented in `troubleshooting/MODAL_FREEZE_FIX.md`
- **Focus Scope Conflicts**: Use `onSelect` with `setTimeout` for dropdowns that open modals
- **Expanded Content Corners**: Solution documented in `troubleshooting/EXPANDED_CORNERS_FIX.md`

### How to Report Issues
1. Check if a solution already exists in the `troubleshooting/` folder
2. Document the problem with specific details
3. Include steps to reproduce
4. Suggest solutions when possible

## 🔄 Maintenance

### Updating Guidelines
- Guidelines are living documents
- Update when patterns change
- Document new conventions
- Keep all files synchronized

### Versioning
- Track changes to guideline files
- Use descriptive commit messages
- Review changes before merging
- Keep guidelines synchronized with code

## 📚 Quick Reference

| Area | File | Description |
|------|------|-------------|
| **Overview** | `PROJECT_GUIDELINES.md` | Main guidelines and quick reference |
| **UI/UX** | `guidelines/UI_GUIDELINES.md` | Interface and design patterns |
| **Architecture** | `guidelines/ARCHITECTURE.md` | Code structure and patterns |
| **Development** | `guidelines/LOCAL_DEVELOPMENT.md` | Setup and local environment |
| **Git** | `guidelines/GIT_BRANCHING.md` | Workflow and branching |
| **Production** | `guidelines/PRODUCTION.md` | Deployment and production environment |
| **Issues** | `troubleshooting/` | Solutions for known problems |

## 🤝 Contributing

### For AI Assistants
- Always consult guidelines before making changes
- Maintain consistency with established patterns
- Document new solutions in the `troubleshooting/` folder
- Never make commits without explicit request

### For Developers
- Keep documentation updated
- Report inconsistencies or improvements
- Follow established patterns
- Contribute solutions for technical problems

## 📞 Support

### Questions
1. Consult the relevant guideline file
2. Look for examples in existing code
3. Ask for clarification when needed
4. Document new patterns as they emerge

### Issues
- Report guideline inconsistencies
- Suggest process improvements
- Update documentation as needed
- Maintain quality standards

---

**Last updated**: $(date)  
**Version**: 1.0.0  
**Maintainer**: Nutri-OS Team
