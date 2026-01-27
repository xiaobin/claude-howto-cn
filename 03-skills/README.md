<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# Agent Skills Guide

Agent Skills are reusable, model-invoked capabilities packaged as folders containing instructions, scripts, and resources. Claude automatically detects and uses relevant skills.

## Overview

**Agent Skills** are modular capabilities that extend Claude's functionality. They package expertise into discoverable, reusable components that Claude **autonomously uses** based on context—unlike slash commands which require explicit user invocation.

Agent Skills enable you to:
- **Package domain expertise** - Encapsulate specialized knowledge and processes
- **Ensure consistency** - Apply standardized approaches across all projects
- **Enable automation** - Let Claude automatically invoke skills when needed
- **Scale workflows** - Reuse skills across multiple projects and teams
- **Maintain quality** - Embed best practices directly into your workflow

Skills are **model-invoked**, meaning Claude automatically decides when to use them based on your request, the skill's description, and the context of your work.

## Skill Architecture

```mermaid
graph TB
    A["Skill Directory"]
    B["SKILL.md"]
    C["YAML Metadata"]
    D["Instructions"]
    E["Scripts"]
    F["Templates"]

    A --> B
    B --> C
    B --> D
    E --> A
    F --> A
```

## Skill Loading Process

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude
    participant System as System
    participant Skill as Skill

    User->>Claude: "Create Excel report"
    Claude->>System: Scan available skills
    System->>System: Load skill metadata
    Claude->>Claude: Match user request to skills
    Claude->>Skill: Load xlsx skill SKILL.md
    Skill-->>Claude: Return instructions + tools
    Claude->>Claude: Execute skill
    Claude->>User: Generate Excel file
```

## Skill Types & Locations

| Type | Location | Scope | Shared | Sync | Best For |
|------|----------|-------|--------|------|----------|
| **Personal Skills** | `~/.claude/skills/` | Individual | No | Manual | Personal workflows, experimental skills |
| **Project Skills** | `.claude/skills/` | Team | Yes | Git | Team standards, shared with team |
| **Plugin Skills** | Via plugin install | Varies | Depends | Auto | Bundled with Claude Code plugins |

### How Skills Work
Once created, skills work automatically—you simply describe what you need, and Claude detects and invokes the appropriate skill based on its description. **No slash commands needed.**

## Creating Custom Skills

### Basic Directory Structure
```
my-skill/
├── SKILL.md (required)
├── reference.md (optional)
├── scripts/
│   └── helper.py
└── templates/
    └── template.txt
```

### SKILL.md Format with Full Metadata

```yaml
---
name: your-skill-name
description: Brief description of what this Skill does and when to use it
context: fork              # Optional: run in isolated sub-agent context
agent: Explore             # Optional: which agent type (with context: fork)
user-invocable: true       # Optional: default true, hide from slash menu if false
disable-model-invocation: false  # Optional: prevent auto-invocation
allowed-tools:             # Optional: restrict tool access
  - Read
  - Grep
  - Glob
hooks:                     # Optional: component-scoped hooks
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate.sh"
          once: true
---

# Your Skill Name

## Instructions
Provide clear, step-by-step guidance for Claude.

## Examples
Show concrete examples of using this Skill.
```

### Required Fields
- **name**: lowercase letters, numbers, hyphens only (max 64 characters)
- **description**: what the Skill does AND when to use it (max 1024 characters)

### Optional Fields

#### Tool Restriction: `allowed-tools`

Limit which tools Claude can use with a Skill:

```yaml
allowed-tools:
  - Read
  - Grep
  - Glob
```

**Use cases:**
- **Read-only Skills**: Prevent accidental file modifications
- **Security-sensitive workflows**: Limit tool access for sensitive operations
- **Limited scope Skills**: Restrict to specific tools

If `allowed-tools` isn't specified, Claude will request permission as normal.

#### Execution Context: `context`

Run a skill in an isolated subagent context:

```yaml
context: fork              # Run in isolated context
agent: Explore             # Use specific agent type
```

This creates a dedicated subagent for the skill with its own context and state.

#### Visibility Control

Control skill visibility and invocation:

```yaml
user-invocable: false                 # Hide from slash menu
disable-model-invocation: false       # Block auto-invocation
```

| Setting | Slash Menu | Skill Tool | Auto-discovery |
|---------|------------|------------|----------------|
| `user-invocable: true` (default) | Visible | Allowed | Yes |
| `user-invocable: false` | Hidden | Allowed | Yes |
| `disable-model-invocation: true` | Visible | Blocked | Yes |

#### Hooks in Skills

Add component-scoped hooks to your skill:

```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate.sh"
          once: true
```

Supported events: `PreToolUse`, `PostToolUse`, `Stop`

## Managing Skills

### Viewing Available Skills

Ask Claude directly:
```
What Skills are available?
```

Or check the filesystem:
```bash
# List personal Skills
ls ~/.claude/skills/

# List project Skills
ls .claude/skills/

# View specific Skill
cat ~/.claude/skills/my-skill/SKILL.md
```

### Testing a Skill

Create the Skill, then ask questions matching its description. Claude autonomously activates it—no explicit invocation needed.

Example: If your description mentions "PDF files":
```
Can you help me extract text from this PDF?
```

### Updating a Skill

Edit the `SKILL.md` file directly:
```bash
# Personal Skill
code ~/.claude/skills/my-skill/SKILL.md

# Project Skill
code .claude/skills/my-skill/SKILL.md
```

**Important**: Changes take effect on next Claude Code startup. Restart if already running.

### Removing a Skill

```bash
# Personal Skill
rm -rf ~/.claude/skills/my-skill

# Project Skill
rm -rf .claude/skills/my-skill
git commit -m "Remove unused Skill"
```

## Practical Examples

### Example 1: Custom Code Review Skill

**Directory Structure:**

```
~/.claude/skills/code-review/
├── SKILL.md
├── templates/
│   ├── review-checklist.md
│   └── finding-template.md
└── scripts/
    ├── analyze-metrics.py
    └── compare-complexity.py
```

**File:** `~/.claude/skills/code-review/SKILL.md`

```yaml
---
name: code-review-specialist
description: Comprehensive code review with security, performance, and quality analysis. Use when users ask to review code, analyze code quality, evaluate pull requests, or mention code review, security analysis, or performance optimization.
---

# Code Review Skill

This skill provides comprehensive code review capabilities focusing on:

1. **Security Analysis**
   - Authentication/authorization issues
   - Data exposure risks
   - Injection vulnerabilities
   - Cryptographic weaknesses
   - Sensitive data logging

2. **Performance Review**
   - Algorithm efficiency (Big O analysis)
   - Memory optimization
   - Database query optimization
   - Caching opportunities
   - Concurrency issues

3. **Code Quality**
   - SOLID principles
   - Design patterns
   - Naming conventions
   - Documentation
   - Test coverage

4. **Maintainability**
   - Code readability
   - Function size (should be < 50 lines)
   - Cyclomatic complexity
   - Dependency management
   - Type safety

## Review Template

For each piece of code reviewed, provide:

### Summary
- Overall quality assessment (1-5)
- Key findings count
- Recommended priority areas

### Critical Issues (if any)
- **Issue**: Clear description
- **Location**: File and line number
- **Impact**: Why this matters
- **Severity**: Critical/High/Medium
- **Fix**: Code example

### Findings by Category

#### Security (if issues found)
List security vulnerabilities with examples

#### Performance (if issues found)
List performance problems with complexity analysis

#### Quality (if issues found)
List code quality issues with refactoring suggestions

#### Maintainability (if issues found)
List maintainability problems with improvements
```

**Python Script:** `~/.claude/skills/code-review/scripts/analyze-metrics.py`

```python
#!/usr/bin/env python3
import re
import sys

def analyze_code_metrics(code):
    """Analyze code for common metrics."""

    # Count functions
    functions = len(re.findall(r'^def\s+\w+', code, re.MULTILINE))

    # Count classes
    classes = len(re.findall(r'^class\s+\w+', code, re.MULTILINE))

    # Average line length
    lines = code.split('\n')
    avg_length = sum(len(l) for l in lines) / len(lines) if lines else 0

    # Estimate complexity
    complexity = len(re.findall(r'\b(if|elif|else|for|while|and|or)\b', code))

    return {
        'functions': functions,
        'classes': classes,
        'avg_line_length': avg_length,
        'complexity_score': complexity
    }

if __name__ == '__main__':
    with open(sys.argv[1], 'r') as f:
        code = f.read()
    metrics = analyze_code_metrics(code)
    for key, value in metrics.items():
        print(f"{key}: {value:.2f}")
```

**Python Script:** `~/.claude/skills/code-review/scripts/compare-complexity.py`

```python
#!/usr/bin/env python3
"""
Compare cyclomatic complexity of code before and after changes.
Helps identify if refactoring actually simplifies code structure.
"""

import re
import sys
from typing import Dict, Tuple

class ComplexityAnalyzer:
    """Analyze code complexity metrics."""

    def __init__(self, code: str):
        self.code = code
        self.lines = code.split('\n')

    def calculate_cyclomatic_complexity(self) -> int:
        """
        Calculate cyclomatic complexity using McCabe's method.
        Count decision points: if, elif, else, for, while, except, and, or
        """
        complexity = 1  # Base complexity

        # Count decision points
        decision_patterns = [
            r'\bif\b',
            r'\belif\b',
            r'\bfor\b',
            r'\bwhile\b',
            r'\bexcept\b',
            r'\band\b(?!$)',
            r'\bor\b(?!$)'
        ]

        for pattern in decision_patterns:
            matches = re.findall(pattern, self.code)
            complexity += len(matches)

        return complexity

    def calculate_cognitive_complexity(self) -> int:
        """
        Calculate cognitive complexity - how hard is it to understand?
        Based on nesting depth and control flow.
        """
        cognitive = 0
        nesting_depth = 0

        for line in self.lines:
            # Track nesting depth
            if re.search(r'^\s*(if|for|while|def|class|try)\b', line):
                nesting_depth += 1
                cognitive += nesting_depth
            elif re.search(r'^\s*(elif|else|except|finally)\b', line):
                cognitive += nesting_depth

            # Reduce nesting when unindenting
            if line and not line[0].isspace():
                nesting_depth = 0

        return cognitive

    def calculate_maintainability_index(self) -> float:
        """
        Maintainability Index ranges from 0-100.
        > 85: Excellent
        > 65: Good
        > 50: Fair
        < 50: Poor
        """
        lines = len(self.lines)
        cyclomatic = self.calculate_cyclomatic_complexity()
        cognitive = self.calculate_cognitive_complexity()

        # Simplified MI calculation
        mi = 171 - 5.2 * (cyclomatic / lines) - 0.23 * (cognitive) - 16.2 * (lines / 1000)

        return max(0, min(100, mi))

    def get_complexity_report(self) -> Dict:
        """Generate comprehensive complexity report."""
        return {
            'cyclomatic_complexity': self.calculate_cyclomatic_complexity(),
            'cognitive_complexity': self.calculate_cognitive_complexity(),
            'maintainability_index': round(self.calculate_maintainability_index(), 2),
            'lines_of_code': len(self.lines),
            'avg_line_length': round(sum(len(l) for l in self.lines) / len(self.lines), 2) if self.lines else 0
        }


def compare_files(before_file: str, after_file: str) -> None:
    """Compare complexity metrics between two code versions."""

    with open(before_file, 'r') as f:
        before_code = f.read()

    with open(after_file, 'r') as f:
        after_code = f.read()

    before_analyzer = ComplexityAnalyzer(before_code)
    after_analyzer = ComplexityAnalyzer(after_code)

    before_metrics = before_analyzer.get_complexity_report()
    after_metrics = after_analyzer.get_complexity_report()

    print("=" * 60)
    print("CODE COMPLEXITY COMPARISON")
    print("=" * 60)

    print("\nBEFORE:")
    print(f"  Cyclomatic Complexity:    {before_metrics['cyclomatic_complexity']}")
    print(f"  Cognitive Complexity:     {before_metrics['cognitive_complexity']}")
    print(f"  Maintainability Index:    {before_metrics['maintainability_index']}")
    print(f"  Lines of Code:            {before_metrics['lines_of_code']}")
    print(f"  Avg Line Length:          {before_metrics['avg_line_length']}")

    print("\nAFTER:")
    print(f"  Cyclomatic Complexity:    {after_metrics['cyclomatic_complexity']}")
    print(f"  Cognitive Complexity:     {after_metrics['cognitive_complexity']}")
    print(f"  Maintainability Index:    {after_metrics['maintainability_index']}")
    print(f"  Lines of Code:            {after_metrics['lines_of_code']}")
    print(f"  Avg Line Length:          {after_metrics['avg_line_length']}")

    print("\nCHANGES:")
    cyclomatic_change = after_metrics['cyclomatic_complexity'] - before_metrics['cyclomatic_complexity']
    cognitive_change = after_metrics['cognitive_complexity'] - before_metrics['cognitive_complexity']
    mi_change = after_metrics['maintainability_index'] - before_metrics['maintainability_index']
    loc_change = after_metrics['lines_of_code'] - before_metrics['lines_of_code']

    print(f"  Cyclomatic Complexity:    {cyclomatic_change:+d}")
    print(f"  Cognitive Complexity:     {cognitive_change:+d}")
    print(f"  Maintainability Index:    {mi_change:+.2f}")
    print(f"  Lines of Code:            {loc_change:+d}")

    print("\nASSESSMENT:")
    if mi_change > 0:
        print("  Code is MORE maintainable")
    elif mi_change < 0:
        print("  Code is LESS maintainable")
    else:
        print("  Maintainability unchanged")

    if cyclomatic_change < 0:
        print("  Complexity DECREASED")
    elif cyclomatic_change > 0:
        print("  Complexity INCREASED")
    else:
        print("  Complexity unchanged")

    print("=" * 60)


if __name__ == '__main__':
    if len(sys.argv) != 3:
        print("Usage: python compare-complexity.py <before_file> <after_file>")
        sys.exit(1)

    compare_files(sys.argv[1], sys.argv[2])
```

**Template:** `~/.claude/skills/code-review/templates/review-checklist.md`

```markdown
# Code Review Checklist

## Security Checklist
- [ ] No hardcoded credentials or secrets
- [ ] Input validation on all user inputs
- [ ] SQL injection prevention (parameterized queries)
- [ ] CSRF protection on state-changing operations
- [ ] XSS prevention with proper escaping
- [ ] Authentication checks on protected endpoints
- [ ] Authorization checks on resources
- [ ] Secure password hashing (bcrypt, argon2)
- [ ] No sensitive data in logs
- [ ] HTTPS enforced

## Performance Checklist
- [ ] No N+1 queries
- [ ] Appropriate use of indexes
- [ ] Caching implemented where beneficial
- [ ] No blocking operations on main thread
- [ ] Async/await used correctly
- [ ] Large datasets paginated
- [ ] Database connections pooled
- [ ] Regular expressions optimized
- [ ] No unnecessary object creation
- [ ] Memory leaks prevented

## Quality Checklist
- [ ] Functions < 50 lines
- [ ] Clear variable naming
- [ ] No duplicate code
- [ ] Proper error handling
- [ ] Comments explain WHY, not WHAT
- [ ] No console.logs in production
- [ ] Type checking (TypeScript/JSDoc)
- [ ] SOLID principles followed
- [ ] Design patterns applied correctly
- [ ] Self-documenting code

## Testing Checklist
- [ ] Unit tests written
- [ ] Edge cases covered
- [ ] Error scenarios tested
- [ ] Integration tests present
- [ ] Coverage > 80%
- [ ] No flaky tests
- [ ] Mock external dependencies
- [ ] Clear test names
```

**Template:** `~/.claude/skills/code-review/templates/finding-template.md`

```markdown
# Code Review Finding Template

Use this template when documenting each issue found during code review.

---

## Issue: [TITLE]

### Severity
- [ ] Critical (blocks deployment)
- [ ] High (should fix before merge)
- [ ] Medium (should fix soon)
- [ ] Low (nice to have)

### Category
- [ ] Security
- [ ] Performance
- [ ] Code Quality
- [ ] Maintainability
- [ ] Testing
- [ ] Design Pattern
- [ ] Documentation

### Location
**File:** `src/components/UserCard.tsx`

**Lines:** 45-52

**Function/Method:** `renderUserDetails()`

### Issue Description

**What:** Describe what the issue is.

**Why it matters:** Explain the impact and why this needs to be fixed.

**Current behavior:** Show the problematic code or behavior.

**Expected behavior:** Describe what should happen instead.

### Code Example

#### Current (Problematic)

```typescript
// Shows the N+1 query problem
const users = fetchUsers();
users.forEach(user => {
  const posts = fetchUserPosts(user.id); // Query per user!
  renderUserPosts(posts);
});
```

#### Suggested Fix

```typescript
// Optimized with JOIN query
const usersWithPosts = fetchUsersWithPosts();
usersWithPosts.forEach(({ user, posts }) => {
  renderUserPosts(posts);
});
```

### Impact Analysis

| Aspect | Impact | Severity |
|--------|--------|----------|
| Performance | 100+ queries for 20 users | High |
| User Experience | Slow page load | High |
| Scalability | Breaks at scale | Critical |
| Maintainability | Hard to debug | Medium |

### Related Issues

- Similar issue in `AdminUserList.tsx` line 120
- Related PR: #456
- Related issue: #789

### Additional Resources

- [N+1 Query Problem](https://en.wikipedia.org/wiki/N%2B1_problem)
- [Database Join Documentation](https://docs.example.com/joins)
- [Performance Optimization Guide](./docs/performance.md)

### Reviewer Notes

- This is a common pattern in this codebase
- Consider adding this to the code style guide
- Might be worth creating a helper function

### Author Response (for feedback)

*To be filled by the code author:*

- [ ] Fix implemented in commit: `abc123`
- [ ] Fix status: Complete / In Progress / Needs Discussion
- [ ] Questions or concerns: (describe)

---

## Finding Statistics (for Reviewer)

When reviewing multiple findings, track:

- **Total Issues Found:** X
- **Critical:** X
- **High:** X
- **Medium:** X
- **Low:** X

**Recommendation:** Approve / Request Changes / Needs Discussion

**Overall Code Quality:** 1-5 stars
```

**Usage Example:**

```markdown
User: Review this React component code

Claude: I'll conduct a comprehensive code review using my Code Review Skill.

[Loads SKILL.md]
[Analyzes code against checklist]
[Runs Python metrics script]
[Applies templates]

## Code Review Results

### Summary
Quality Score: 3.5/5
Key Findings: 7 items
Priority Areas: Performance optimization, error handling

### Critical Issues: 1

**Issue**: N+1 Query Problem
**Location**: UserComponent.tsx, lines 45-52
**Impact**: Renders 100+ database queries for list of 20 users
**Severity**: High
**Fix**:
```javascript
// Before: N+1 queries
const users = fetchUsers();
users.forEach(user => fetchUserPosts(user.id)); // 20+ queries

// After: Single query with JOIN
const users = fetchUsersWithPosts(); // 1 query
```

### Performance Findings
- Missing pagination on large lists
- Recommendation: Use React.memo() for items
- Database query: Could be optimized with index

### Quality Findings
- Function in line 20 is 127 lines (max: 50)
- Missing error boundary
- Props should have TypeScript types
```

### Example 2: Brand Voice Skill

**Directory Structure:**

```
.claude/skills/brand-voice/
├── SKILL.md
├── brand-guidelines.md
├── tone-examples.md
└── templates/
    ├── email-template.txt
    ├── social-post-template.txt
    └── blog-post-template.md
```

**File:** `.claude/skills/brand-voice/SKILL.md`

```yaml
---
name: brand-voice-consistency
description: Ensure all communication matches brand voice and tone guidelines. Use when creating marketing copy, customer communications, public-facing content, or when users mention brand voice, tone, or writing style.
---

# Brand Voice Skill

## Overview
This skill ensures all communications maintain consistent brand voice, tone, and messaging.

## Brand Identity

### Mission
Help teams automate their development workflows with AI

### Values
- **Simplicity**: Make complex things simple
- **Reliability**: Rock-solid execution
- **Empowerment**: Enable human creativity

### Tone of Voice
- **Friendly but professional** - approachable without being casual
- **Clear and concise** - avoid jargon, explain technical concepts simply
- **Confident** - we know what we're doing
- **Empathetic** - understand user needs and pain points

## Writing Guidelines

### Do's
- Use "you" when addressing readers
- Use active voice: "Claude generates reports" not "Reports are generated by Claude"
- Start with value proposition
- Use concrete examples
- Keep sentences under 20 words
- Use lists for clarity
- Include calls-to-action

### Don'ts
- Don't use corporate jargon
- Don't patronize or oversimplify
- Don't use "we believe" or "we think"
- Don't use ALL CAPS except for emphasis
- Don't create walls of text
- Don't assume technical knowledge

## Vocabulary

### Preferred Terms
- Claude (not "the Claude AI")
- Code generation (not "auto-coding")
- Agent (not "bot")
- Streamline (not "revolutionize")
- Integrate (not "synergize")

### Avoid Terms
- "Cutting-edge" (overused)
- "Game-changer" (vague)
- "Leverage" (corporate-speak)
- "Utilize" (use "use")
- "Paradigm shift" (unclear)

**Good Example:**
"Claude automates your code review process. Instead of manually checking each PR, Claude reviews security, performance, and quality—saving your team hours every week."

Why it works: Clear value, specific benefits, action-oriented

**Bad Example:**
"Claude leverages cutting-edge AI to provide comprehensive software development solutions."

Why it doesn't work: Vague, corporate jargon, no specific value

```

**Template:** `email-template.txt`

```
Subject: [Clear, benefit-driven subject]

Hi [Name],

[Opening: What's the value for them]

[Body: How it works / What they'll get]

[Specific example or benefit]

[Call to action: Clear next step]

Best regards,
[Name]
```

**Template:** `social-post-template.txt`

```
[Hook: Grab attention in first line]
[2-3 lines: Value or interesting fact]
[Call to action: Link, question, or engagement]
[Emoji: 1-2 max for visual interest]
```

**File:** `tone-examples.md`

```markdown
Exciting announcement:
"Save 8 hours per week on code reviews. Claude reviews your PRs automatically."

Empathetic support:
"We know deployments can be stressful. Claude handles testing so you don't have to worry."

Confident product feature:
"Claude doesn't just suggest code. It understands your architecture and maintains consistency."

Educational blog post:
"Let's explore how agents improve code review workflows. Here's what we learned..."
```

### Example 3: Documentation Generator Skill

**File:** `.claude/skills/doc-generator/SKILL.md`

```yaml
---
name: api-documentation-generator
description: Generate comprehensive, accurate API documentation from source code. Use when creating or updating API documentation, generating OpenAPI specs, or when users mention API docs, endpoints, or documentation.
---

# API Documentation Generator Skill

## Generates

- OpenAPI/Swagger specifications
- API endpoint documentation
- SDK usage examples
- Integration guides
- Error code references
- Authentication guides

## Documentation Structure

### For Each Endpoint
<document>
## GET /api/v1/users/:id

### Description
Brief explanation of what this endpoint does

### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | string | Yes | User ID |

### Response
**200 Success**

{
  "id": "usr_123",
  "name": "John Doe",
  "email": "john@example.com",
  "created_at": "2025-01-15T10:30:00Z"
}

**404 Not Found**

{
  "error": "USER_NOT_FOUND",
  "message": "User does not exist"
}

### Examples

**cURL**
curl -X GET "https://api.example.com/api/v1/users/usr_123" \
  -H "Authorization: Bearer YOUR_TOKEN"

**JavaScript**
const user = await fetch('/api/v1/users/usr_123', {
  headers: { 'Authorization': 'Bearer token' }
}).then(r => r.json());

**Python**
response = requests.get(
    'https://api.example.com/api/v1/users/usr_123',
    headers={'Authorization': 'Bearer token'}
)
user = response.json()
</document>
```
**Python Script:** `.claude/skills/doc-generator/scripts/generate-docs.py`

```python
#!/usr/bin/env python3
import ast
import json
from typing import Dict, List

class APIDocExtractor(ast.NodeVisitor):
    """Extract API documentation from Python source code."""

    def __init__(self):
        self.endpoints = []

    def visit_FunctionDef(self, node):
        """Extract function documentation."""
        if node.name.startswith('get_') or node.name.startswith('post_'):
            doc = ast.get_docstring(node)
            endpoint = {
                'name': node.name,
                'docstring': doc,
                'params': [arg.arg for arg in node.args.args],
                'returns': self._extract_return_type(node)
            }
            self.endpoints.append(endpoint)
        self.generic_visit(node)

    def _extract_return_type(self, node):
        """Extract return type from function annotation."""
        if node.returns:
            return ast.unparse(node.returns)
        return "Any"

def generate_markdown_docs(endpoints: List[Dict]) -> str:
    """Generate markdown documentation from endpoints."""
    docs = "# API Documentation\n\n"

    for endpoint in endpoints:
        docs += f"## {endpoint['name']}\n\n"
        docs += f"{endpoint['docstring']}\n\n"
        docs += f"**Parameters**: {', '.join(endpoint['params'])}\n\n"
        docs += f"**Returns**: {endpoint['returns']}\n\n"
        docs += "---\n\n"

    return docs

if __name__ == '__main__':
    import sys
    with open(sys.argv[1], 'r') as f:
        tree = ast.parse(f.read())

    extractor = APIDocExtractor()
    extractor.visit(tree)

    markdown = generate_markdown_docs(extractor.endpoints)
    print(markdown)
```

### Example 4: Multi-File Skill (Complex Structure)

For complex skills with multiple reference files, scripts, and templates:

**Directory Structure:**

```
pdf-processing/
├── SKILL.md              # Required - main instructions
├── FORMS.md              # Optional - detailed form-filling guide
├── REFERENCE.md          # Optional - API reference
└── scripts/
    ├── fill_form.py      # Optional - utility scripts
    └── validate.py
```

**File:** `pdf-processing/SKILL.md`

```yaml
---
name: pdf-processing
description: Extract text, fill forms, merge PDFs. Use when working with PDF files, forms, or document extraction. Requires pypdf and pdfplumber packages.
---

# PDF Processing

## Quick Start

Extract text:
```python
import pdfplumber
with pdfplumber.open("doc.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

For form filling, see [FORMS.md](FORMS.md).
For detailed API reference, see [REFERENCE.md](REFERENCE.md).

## Requirements

Packages must be installed in your environment:
```bash
pip install pypdf pdfplumber
```
```

**Important**: Claude reads additional files only when needed, using progressive disclosure to manage context efficiently. Reference files with relative links in your SKILL.md.

### Example 5: CLAUDE.md Generator Skill

A skill for creating and maintaining optimal CLAUDE.md files following best practices.

**Directory Structure:**

```
claude-md/
└── SKILL.md
```

**File:** `~/.claude/skills/claude-md/SKILL.md`

```yaml
---
description: Create or update CLAUDE.md files following best practices for optimal AI agent onboarding
---

## Core Principles

**LLMs are stateless**: CLAUDE.md is the only file automatically included in every conversation. It serves as the primary onboarding document for AI agents into your codebase.

### The Golden Rules

1. **Less is More**: Frontier LLMs can follow ~150-200 instructions. Keep your CLAUDE.md focused and concise.
2. **Universal Applicability**: Only include information relevant to EVERY session.
3. **Don't Use Claude as a Linter**: Use deterministic tools (prettier, eslint) instead.
4. **Never Auto-Generate**: Craft it manually with careful consideration.

## Essential Sections

A well-structured CLAUDE.md should include:

- **Project Name**: Brief one-line description
- **Tech Stack**: Primary language, frameworks, database
- **Project Structure**: Only for monorepos or complex structures
- **Development Commands**: Install, test, build commands
- **Critical Conventions**: Only non-obvious, high-impact conventions
- **Known Issues / Gotchas**: Things that consistently trip up developers

## Quality Constraints

- Target under 300 lines (ideally under 100)
- No style rules (use linters)
- No code snippets (use file references)
- No task-specific instructions
```

**Key Features:**
- Three operation modes: `create`, `update`, `audit`
- Follows WHAT/WHY/HOW content strategy
- Progressive disclosure for larger projects via `agent_docs/` folder
- Validation checklist for quality assurance
- Anti-pattern detection and removal

**Usage Examples:**

```
User: /claude-md create
Claude: [Analyzes project, creates optimized CLAUDE.md]

User: /claude-md audit
Claude: [Reports on current CLAUDE.md quality without modifications]

User: /claude-md update
Claude: [Improves existing CLAUDE.md following best practices]
```

### Example 6: Code Refactoring Skill

A systematic refactoring skill based on Martin Fowler's methodology.

**Directory Structure:**

```
refactor/
├── SKILL.md                          # Main instructions & workflow
├── references/
│   ├── code-smells.md                # Code smell catalog
│   └── refactoring-catalog.md        # Refactoring techniques
├── templates/
│   └── refactoring-plan.md           # Planning template
└── scripts/
    ├── analyze-complexity.py         # Complexity metrics
    └── detect-smells.py              # Automated smell detection
```

**File:** `refactor/SKILL.md`

```yaml
---
name: code-refactor
description: Systematic code refactoring based on Martin Fowler's methodology. Use when users ask to refactor code, improve code structure, reduce technical debt, clean up legacy code, eliminate code smells, or improve code maintainability.
---

# Code Refactoring Skill

A phased approach emphasizing safe, incremental changes backed by tests.

## Workflow

Phase 1: Research & Analysis → Phase 2: Test Coverage Assessment →
Phase 3: Code Smell Identification → Phase 4: Refactoring Plan Creation →
Phase 5: Incremental Implementation → Phase 6: Review & Iteration

## Core Principles

1. **Behavior Preservation**: External behavior must remain unchanged
2. **Small Steps**: Make tiny, testable changes
3. **Test-Driven**: Tests are the safety net
4. **Continuous**: Refactoring is ongoing, not a one-time event
5. **Collaborative**: User approval required at each phase
```

**Key Features:**
- Phased approach with user approval checkpoints
- Code smell catalog with severity assessment
- Refactoring technique catalog mapped to smells
- Automated complexity analysis scripts
- Safe rollback strategies

**Usage:**
```
User: "Refactor this UserService class - it's gotten too large"

Claude: [Loads refactor skill]
1. Analyzes UserService for code smells
2. Identifies: Large Class, Long Methods, Feature Envy
3. Creates phased refactoring plan
4. Requests approval before each phase
5. Implements incrementally with test verification
```

## Skill Discovery & Invocation

```mermaid
graph TD
    A["User Request"] --> B["Claude Analyzes"]
    B -->|Scans| C["Available Skills"]
    C -->|Metadata check| D["Skill Description Match?"]
    D -->|Yes| E["Load SKILL.md"]
    D -->|No| F["Try next skill"]
    F -->|More skills?| D
    F -->|No more| G["Use general knowledge"]
    E --> H["Extract Instructions"]
    H --> I["Execute Skill"]
    I --> J["Return Results"]
```

## Skills with Subagents

Skills can be automatically loaded with subagents, enabling specialized task delegation:

```yaml
# In subagent definition
skills: pr-review, security-check
```

This allows subagents to access and invoke specific skills autonomously.

**Note**: Built-in agents (Explore, Plan, general-purpose) do not have access to user-defined skills.

## Skill vs Other Features

```mermaid
graph TB
    A["Extending Claude"]
    B["Slash Commands"]
    C["Subagents"]
    D["Memory"]
    E["MCP"]
    F["Skills"]

    A --> B
    A --> C
    A --> D
    A --> E
    A --> F

    B -->|User-invoked| G["Quick shortcuts"]
    C -->|Auto-delegated| H["Isolated contexts"]
    D -->|Persistent| I["Cross-session context"]
    E -->|Real-time| J["External data access"]
    F -->|Auto-invoked| K["Autonomous execution"]
```

## Best Practices

### 1. Make Descriptions Specific
- **Bad (Vague)**: "Helps with documents"
- **Good (Specific)**: "Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction."

### 2. Keep Skills Focused
- One Skill = one capability
- ✅ "PDF form filling"
- ❌ "Document processing" (too broad)

### 3. Include Trigger Terms
- Add keywords in descriptions that match user requests
- Helps Claude discover the Skill more easily
- Example: "Use when working with PDF files or when the user mentions PDFs, forms, or document extraction"

### 4. Test Thoroughly
- Ask questions matching your description
- Have teammates test for discoverability
- Verify YAML syntax

### Additional Do's
- Use clear, descriptive names
- Include comprehensive instructions
- Add concrete examples
- Package related scripts and templates
- Test with real scenarios
- Organize skills by domain/purpose
- Document dependencies

### Don'ts
- Don't create skills for one-time tasks
- Don't duplicate existing functionality
- Don't make skills too broad
- Don't forget metadata in SKILL.md
- Don't skip examples
- Don't assume Claude knows skill context
- Don't create circular dependencies
- Don't ignore performance implications

### 5. Document Skill Versions

Track changes to your skills over time:

```markdown
# My Skill

## Version History
- v2.0.0 (2025-01-15): Breaking changes - new output format
- v1.1.0 (2025-01-01): Added chart generation feature
- v1.0.0 (2024-12-01): Initial release
```

This helps your team understand skill evolution and identify when breaking changes were introduced.

## Troubleshooting Guide

### Quick Reference

| Issue | Solution |
|-------|----------|
| Claude doesn't use Skill | Make description more specific with trigger terms |
| Skill file not found | Verify path: `~/.claude/skills/name/SKILL.md` or `.claude/skills/name/SKILL.md` |
| YAML errors | Check opening/closing `---`, indentation, no tabs (spaces only) |
| Skills conflict | Use distinct trigger terms in descriptions |
| Scripts not running | Check permissions with `chmod +x` and use forward slashes in paths |

### Debugging Skipped or Non-Functional Skills

#### Claude Doesn't Use My Skill

**Problem: Vague description**
```yaml
# Too generic - Claude can't match this to requests
description: Helps with data
```

**Solution: Make it specific with triggers**
```yaml
description: Analyze Excel spreadsheets, generate pivot tables, create charts. Use when working with Excel files, spreadsheets, or .xlsx files.
```

**Problem: Invalid YAML frontmatter**

Check your frontmatter:
```bash
cat SKILL.md | head -n 10
```

Ensure:
- Opening `---` on line 1
- Closing `---` before Markdown content
- Valid YAML syntax (no tabs, correct indentation)
- No special characters that need escaping

**Problem: Wrong file path**

Verify the skill location:
```bash
# Personal Skills
ls ~/.claude/skills/my-skill/SKILL.md

# Project Skills
ls .claude/skills/my-skill/SKILL.md
```

#### Skill Has Errors

**Check script permissions:**
```bash
chmod +x .claude/skills/my-skill/scripts/*.py
```

**Use forward slashes in paths:**
- Correct: `scripts/helper.py`
- Wrong: `scripts\helper.py` (Windows style)

**Check dependencies:**
```bash
# Claude will automatically install or ask for permission
pip install required-package
```

#### Multiple Skills Conflict

**Use distinct trigger terms** to help Claude choose the right Skill:

```yaml
# Skill 1 - Sales analysis
description: Analyze sales data in Excel files and CRM exports. Use for sales reports, pipeline analysis, and revenue tracking.

# Skill 2 - System logs
description: Analyze log files and system metrics data. Use for performance monitoring, debugging, and system diagnostics.
```

## Sharing Skills with Your Team

1. Create Skill in `.claude/skills/` (project-level)
2. Commit to git
3. Team members pull changes — Skills available immediately

For broader sharing, consider packaging your Skills as plugins.

## Installation Instructions

### For Personal Skills

Copy skill folders to your personal skills directory:

```bash
# Copy individual skill
cp -r code-review ~/.claude/skills/

# Copy all skills
cp -r * ~/.claude/skills/

# Make scripts executable
chmod +x ~/.claude/skills/code-review/scripts/*.py
```

### For Project Skills

Copy skill folders to your project skills directory to share with team:

```bash
# Copy individual skill to project
cp -r code-review /path/to/project/.claude/skills/

# Copy all skills
cp -r * /path/to/project/.claude/skills/

# Commit to version control
git add .claude/skills/
git commit -m "Add project skills"
```

### Verifying Installation

After copying skills:

1. Check that SKILL.md exists in each skill directory
2. Verify scripts have proper permissions: `ls -l ~/.claude/skills/code-review/scripts/`
3. Test skill invocation with a sample request

### Creating Custom Skills

1. Create skill directory structure
2. Write SKILL.md with metadata and instructions
3. Add scripts and templates as needed
4. Test by copying to skills directory
5. Refine based on usage

## Additional Resources

- [Official Skills Documentation](https://code.claude.com/docs/en/skills)
- [Slash Commands Guide](../01-slash-commands/) - User-initiated shortcuts
- [Subagents Guide](../04-subagents/) - Delegated AI agents
- [Memory Guide](../02-memory/) - Persistent context
- [MCP (Model Context Protocol)](../05-mcp/) - Real-time external data
- [Hooks Guide](../06-hooks/) - Event-driven automation
