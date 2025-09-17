# Complex Problem Solving - Best Practices for AI Agents

## Overview
This document provides guidelines for AI agents when tackling complex problems that require thorough analysis and systematic investigation. The goal is to prevent "narrow mind" approaches and ensure comprehensive problem resolution.

## Core Principles

### 1. Never Stop at the First Finding
- **Problem**: Agents often find one issue and immediately focus on fixing it, missing the bigger picture
- **Solution**: Always continue investigating after finding the first problem
- **Rule**: Find at least 3 potential causes before starting any fixes

### 2. Take a Step Back ("Ver com Calma")
When you encounter a problem:
1. **Pause** - Don't immediately jump to solutions
2. **Analyze** - Look at the problem from multiple angles
3. **Investigate** - Search for related issues, patterns, and root causes
4. **Plan** - Create a systematic approach before implementing fixes

### 3. Comprehensive Analysis Framework
Use this systematic approach for complex problems:

#### Phase 1: Problem Understanding
- [ ] What is the exact error/issue?
- [ ] When does it occur? (timing, conditions, triggers)
- [ ] Where does it occur? (which components, files, functions)
- [ ] Who is affected? (users, systems, processes)
- [ ] How severe is the impact?

#### Phase 2: Context Gathering
- [ ] Recent changes in the codebase
- [ ] Related components and dependencies
- [ ] Environment differences (local vs production)
- [ ] Similar issues in the past
- [ ] External factors (APIs, services, infrastructure)

#### Phase 3: Root Cause Investigation
- [ ] Trace the problem to its source
- [ ] Identify all contributing factors
- [ ] Map dependencies and relationships
- [ ] Consider edge cases and corner scenarios
- [ ] Look for patterns across multiple instances

## Investigation Strategies

### 1. Multi-Layer Search Approach
Don't rely on a single search or investigation method:

```markdown
✅ Good Approach:
1. Search for exact error message
2. Search for related functionality
3. Search for similar patterns in codebase
4. Check recent changes and commits
5. Investigate dependencies and imports
6. Look at configuration and environment setup
```

```markdown
❌ Bad Approach:
1. Search for error message
2. Find one potential cause
3. Immediately implement fix
```

### 2. The "5 Whys" Technique
For each problem found, ask "why" at least 5 times:
- Why is this error occurring?
- Why is that condition happening?
- Why wasn't this caught earlier?
- Why is the system behaving this way?
- Why wasn't this scenario considered?

### 3. Holistic Code Analysis
When investigating code issues:

#### Frontend Issues
- [ ] Component hierarchy and props flow
- [ ] State management and data flow
- [ ] Event handling and user interactions
- [ ] API calls and data fetching
- [ ] Routing and navigation
- [ ] CSS/styling conflicts
- [ ] Browser compatibility

#### Backend Issues
- [ ] Database queries and relationships
- [ ] API endpoints and middleware
- [ ] Authentication and authorization
- [ ] Edge functions and serverless logic
- [ ] Environment variables and configuration
- [ ] External service integrations
- [ ] Caching and performance

#### Integration Issues
- [ ] Frontend-backend communication
- [ ] Database schema and migrations
- [ ] Third-party service connections
- [ ] Environment-specific configurations
- [ ] Deployment and build processes

## Common Pitfalls to Avoid

### 1. Tunnel Vision
- **Problem**: Focusing only on the obvious symptom
- **Solution**: Always look for underlying causes and related issues

### 2. Quick Fix Mentality
- **Problem**: Implementing the first solution that comes to mind
- **Solution**: Evaluate multiple solutions and their implications

### 3. Isolated Component Thinking
- **Problem**: Treating each component as independent
- **Solution**: Consider system-wide impacts and dependencies

### 4. Ignoring Edge Cases
- **Problem**: Only testing happy path scenarios
- **Solution**: Consider error states, boundary conditions, and unusual inputs

## Systematic Problem-Solving Workflow

### Step 1: Information Gathering (Minimum 10 minutes)
```markdown
🔍 Gather Information:
- Read error messages completely
- Check logs and console output
- Examine recent code changes
- Review related documentation
- Search for similar issues in codebase
- Check environment and configuration
```

### Step 2: Pattern Recognition (5-10 minutes)
```markdown
🧩 Identify Patterns:
- Are there multiple related errors?
- Do issues occur in specific scenarios?
- Are there common components involved?
- Is this environment-specific?
- Are there timing-related patterns?
```

### Step 3: Hypothesis Formation (5 minutes)
```markdown
💭 Create Multiple Hypotheses:
- List at least 3 possible causes
- Rank them by likelihood
- Consider both technical and process causes
- Think about system interactions
- Consider user behavior factors
```

### Step 4: Systematic Testing (Variable time)
```markdown
🧪 Test Hypotheses:
- Test most likely cause first
- Verify fixes don't break other functionality
- Test edge cases and error scenarios
- Check different environments
- Validate with realistic data
```

### Step 5: Comprehensive Validation (10-15 minutes)
```markdown
✅ Validate Solution:
- Test the complete user flow
- Check related functionality
- Verify performance impact
- Test error handling
- Confirm accessibility compliance
- Check mobile responsiveness (if applicable)
```

## Investigation Checklist for Common Scenarios

### UI/Component Issues
- [ ] Check component props and state
- [ ] Verify event handlers and callbacks
- [ ] Inspect CSS classes and styling
- [ ] Test different screen sizes
- [ ] Check accessibility features
- [ ] Validate form inputs and validation
- [ ] Test user interaction flows

### Data/API Issues
- [ ] Check database queries and results
- [ ] Verify API endpoints and responses
- [ ] Test authentication and permissions
- [ ] Check data validation and sanitization
- [ ] Verify error handling and responses
- [ ] Test with different data scenarios
- [ ] Check caching and data freshness

### Performance Issues
- [ ] Check network requests and timing
- [ ] Analyze bundle size and loading
- [ ] Verify database query efficiency
- [ ] Check memory usage and leaks
- [ ] Test with realistic data volumes
- [ ] Verify caching strategies
- [ ] Check for unnecessary re-renders

## Communication Guidelines

### When Reporting Findings
Always provide:
1. **Complete problem description**
2. **All identified causes** (not just the first one)
3. **Systematic investigation results**
4. **Recommended solutions with trade-offs**
5. **Potential side effects or risks**

### When Asking for Clarification
Instead of:
- "I found an error in component X"

Use:
- "I've identified multiple issues: 1) Component X has a state management problem, 2) The API call is missing error handling, 3) The validation logic has edge case bugs. Let me investigate further to understand the root cause and relationships between these issues."

## Tools and Techniques

### Code Investigation Tools
- Use semantic search for broad understanding
- Use grep for specific pattern matching
- Use file search for related components
- Read complete files, not just snippets
- Trace imports and dependencies

### Analysis Techniques
- Create mental maps of data flow
- Document component relationships
- Track state changes through the system
- Map user journey and potential failure points
- Consider all environment variables and configurations

## Success Metrics

A thorough investigation should result in:
- [ ] Understanding of root cause(s)
- [ ] Identification of all affected components
- [ ] Clear plan for comprehensive fix
- [ ] Prevention strategy for similar issues
- [ ] Documentation of findings and solutions

## Remember: Quality Over Speed

It's better to take 30 minutes to fully understand a problem than to spend hours fixing symptoms while the root cause remains. Always prioritize thorough analysis over quick fixes.

---

*This document should be referenced whenever tackling complex issues that seem to have multiple symptoms or when initial solutions don't fully resolve the problem.*
