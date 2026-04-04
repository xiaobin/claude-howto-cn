<picture>
  <source media="(prefers-color-scheme: dark)" srcset="resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="resources/logos/claude-howto-logo.svg">
</picture>

# Claude 概念完整指南

涵盖斜杠命令、子代理、记忆、MCP 协议和代理技能的综合参考指南，包含表格、图表和实践示例。

---

## 目录

1. [斜杠命令](#斜杠命令)
2. [子代理](#子代理)
3. [记忆](#记忆)
4. [MCP 协议](#mcp-协议)
5. [代理技能](#代理技能)
6. [插件](#插件加载流程)
7. [钩子](#钩子)
8. [检查点和回退](#检查点和回退)
9. [高级功能](#高级功能)
10. [对比与集成](#对比与集成)

---

## 斜杠命令

### 概述

斜杠命令是存储为 Markdown 文件的用户调用快捷方式，Claude Code 可以执行这些文件。它们使团队能够标准化常用提示词和工作流。

### 架构

```mermaid
graph TD
    A["用户输入：/command-name"] -->|触发| B["搜索 .claude/commands/"]
    B -->|找到| C["command-name.md"]
    C -->|加载| D["Markdown 内容"]
    D -->|执行| E["Claude 处理提示词"]
    E -->|返回| F["上下文中的结果"]
```

### 文件结构

```mermaid
graph LR
    A["项目根"] -->|包含| B[".claude/commands/"]
    B -->|包含| C["optimize.md"]
    B -->|包含| D["test.md"]
    B -->|包含| E["docs/"]
    E -->|包含| F["generate-api-docs.md"]
    E -->|包含| G["generate-readme.md"]
```

### 命令组织表

| 位置 | 范围 | 可用性 | 使用场景 | Git 跟踪 |
|----------|-------|--------------|----------|-------------|
| `.claude/commands/` | 项目特定 | 团队成员 | 团队工作流、共享标准 | 是 |
| `~/.claude/commands/` | 个人 | 个人用户 | 跨项目个人快捷方式 | 否 |
| 子目录 | 命名空间 | 基于父级 | 按类别组织 | 是 |

### 功能和能力

| 功能 | 示例 | 支持 |
|---------|---------|-----------|
| Shell 脚本执行 | `bash scripts/deploy.sh` | 支持 |
| 文件引用 | `@path/to/file.js` | 支持 |
| Bash 集成 | `$(git log --oneline)` | 支持 |
| 参数 | `/pr --verbose` | 支持 |
| MCP 命令 | `/mcp__github__list_prs` | 支持 |

### 实践示例

#### 示例 1：代码优化命令

**文件：** `.claude/commands/optimize.md`

```markdown
---
name: Code Optimization
description: Analyze code for performance issues and suggest optimizations
tags: performance, analysis
---

# Code Optimization

Review the provided code for the following issues in order of priority:

1. **Performance bottlenecks** - identify O(n²) operations, inefficient loops
2. **Memory leaks** - find unreleased resources, circular references
3. **Algorithm improvements** - suggest better algorithms or data structures
4. **Caching opportunities** - identify repeated computations
5. **Concurrency issues** - find race conditions or threading problems

Format your response with:
- Issue severity (Critical/High/Medium/Low)
- Location in code
- Explanation
- Recommended fix with code example
```

**使用：**
```bash
# 用户在 Claude Code 中输入
/optimize

# Claude 加载提示词并等待代码输入
```

#### 示例 2：Pull Request 辅助命令

**文件：** `.claude/commands/pr.md`

```markdown
---
name: Prepare Pull Request
description: Clean up code, stage changes, and prepare a pull request
tags: git, workflow
---

# Pull Request Preparation Checklist

Before creating a PR, execute these steps:

1. Run linting: `prettier --write .`
2. Run tests: `npm test`
3. Review git diff: `git diff HEAD`
4. Stage changes: `git add .`
5. Create commit message following conventional commits:
   - `fix:` for bug fixes
   - `feat:` for new features
   - `docs:` for documentation
   - `refactor:` for code restructuring
   - `test:` for test additions
   - `chore:` for maintenance

6. Generate PR summary including:
   - What changed
   - Why it changed
   - Testing performed
   - Potential impacts
```

**使用：**
```bash
/pr

# Claude runs through checklist and prepares the PR
```

#### 示例 3：分层文档生成器

**文件：** `.claude/commands/docs/generate-api-docs.md`

```markdown
---
name: Generate API Documentation
description: Create comprehensive API documentation from source code
tags: documentation, api
---

# API Documentation Generator

Generate API documentation by:

1. Scanning all files in `/src/api/`
2. Extracting function signatures and JSDoc comments
3. Organizing by endpoint/module
4. Creating markdown with examples
5. Including request/response schemas
6. Adding error documentation

Output format:
- Markdown file in `/docs/api.md`
- Include curl examples for all endpoints
- Add TypeScript types
```

### 命令生命周期图

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant FS as File System
    participant CLI as Shell/Bash

    User->>Claude: Types /optimize
    Claude->>FS: Searches .claude/commands/
    FS-->>Claude: Returns optimize.md
    Claude->>Claude: Loads Markdown content
    Claude->>User: Displays prompt context
    User->>Claude: Provides code to analyze
    Claude->>CLI: (May execute scripts)
    CLI-->>Claude: Results
    Claude->>User: Returns analysis
```

### 最佳实践

| 应该做 | 不应该做 |
|------|---------|
| 使用清晰、动宾结构的名称 | 为一次性任务创建命令 |
| 在描述中记录触发词 | 在命令中构建复杂逻辑 |
| 保持命令专注于单一任务 | 创建冗余命令 |
| 对项目命令进行版本控制 | 硬编码敏感信息 |
| 组织在子目录中 | 创建冗长的命令列表 |
| 使用简单、可读的提示词 | 使用缩写或晦涩措辞 |

---

## 子代理

### 概述

子代理是具有隔离上下文窗口和自定义系统提示词的专业 AI 助手。它们支持委托任务执行，同时保持关注点清晰分离。

### 架构图

```mermaid
graph TB
    User["用户"]
    Main["主代理<br/>(协调器)"]
    Reviewer["代码审查<br/>子代理"]
    Tester["测试工程师<br/>子代理"]
    Docs["文档<br/>子代理"]

    User -->|询问| Main
    Main -->|分配| Reviewer
    Main -->|分配| Tester
    Main -->|分配| Docs
    Reviewer -->|返回结果| Main
    Tester -->|返回结果| Main
    Docs -->|返回结果| Main
    Main -->|综合| User
```

### 子代理生命周期

```mermaid
sequenceDiagram
    participant User
    participant MainAgent as 主代理
    participant CodeReviewer as 代码审查<br/>子代理
    participant Context as 独立<br/>上下文窗口

    User->>MainAgent: "Build new auth feature"
    MainAgent->>MainAgent: 分析任务
    MainAgent->>CodeReviewer: "Review this code"
    CodeReviewer->>Context: Initialize clean context
    Context->>CodeReviewer: Load reviewer instructions
    CodeReviewer->>CodeReviewer: Perform review
    CodeReviewer-->>MainAgent: Return findings
    MainAgent->>MainAgent: Incorporate results
    MainAgent-->>User: Provide synthesis
```

### 子代理配置表

| 配置 | 类型 | 用途 | 示例 |
|---------------|------|---------|---------|
| `name` | String | 代理标识符 | `code-reviewer` |
| `description` | String | 用途和触发词 | `Comprehensive code quality analysis` |
| `tools` | List/String | 允许的能力 | `read, grep, diff, lint_runner` |
| `system_prompt` | Markdown | 行为指示 | Custom guidelines |

### 工具访问层级

```mermaid
graph TD
    A["子代理配置"] -->|选项 1| B["继承主线程<br/>的所有工具"]
    A -->|选项 2| C["指定单个工具"]
    B -->|包含| B1["文件操作"]
    B -->|包含| B2["Shell 命令"]
    B -->|包含| B3["MCP 工具"]
    C -->|明确列表| C1["read, grep, diff"]
    C -->|明确列表| C2["Bash(npm:*), Bash(test:*)"]
```

### 实践示例

#### 示例 1：完整子代理设置

**文件：** `.claude/agents/code-reviewer.md`

```yaml
---
name: code-reviewer
description: Comprehensive code quality and maintainability analysis
tools: read, grep, diff, lint_runner
---

# Code Reviewer Agent

You are an expert code reviewer specializing in:
- Performance optimization
- Security vulnerabilities
- Code maintainability
- Testing coverage
- Design patterns

## Review Priorities (in order)

1. **Security Issues** - Authentication, authorization, data exposure
2. **Performance Problems** - O(n²) operations, memory leaks, inefficient queries
3. **Code Quality** - Readability, naming, documentation
4. **Test Coverage** - Missing tests, edge cases
5. **Design Patterns** - SOLID principles, architecture

## Review Output Format

For each issue:
- **Severity**: Critical / High / Medium / Low
- **Category**: Security / Performance / Quality / Testing / Design
- **Location**: File path and line number
- **Issue Description**: What's wrong and why
- **Suggested Fix**: Code example
- **Impact**: How this affects the system

## Example Review

### Issue: N+1 Query Problem
- **Severity**: High
- **Category**: Performance
- **Location**: src/user-service.ts:45
- **Issue**: Loop executes database query in each iteration
- **Fix**: Use JOIN or batch query
```

**文件：** `.claude/agents/test-engineer.md`

```yaml
---
name: test-engineer
description: Test strategy, coverage analysis, and automated testing
tools: read, write, bash, grep
---

# Test Engineer Agent

You are expert at:
- Writing comprehensive test suites
- Ensuring high code coverage (>80%)
- Testing edge cases and error scenarios
- Performance benchmarking
- Integration testing

## Testing Strategy

1. **Unit Tests** - Individual functions/methods
2. **Integration Tests** - Component interactions
3. **End-to-End Tests** - Complete workflows
4. **Edge Cases** - Boundary conditions
5. **Error Scenarios** - Failure handling

## Test Output Requirements

- Use Jest for JavaScript/TypeScript
- Include setup/teardown for each test
- Mock external dependencies
- Document test purpose
- Include performance assertions when relevant

## Coverage Requirements

- Minimum 80% code coverage
- 100% for critical paths
- Report missing coverage areas
```

**文件：** `.claude/agents/documentation-writer.md`

```yaml
---
name: documentation-writer
description: Technical documentation, API docs, and user guides
tools: read, write, grep
---

# Documentation Writer Agent

You create:
- API documentation with examples
- User guides and tutorials
- Architecture documentation
- Changelog entries
- Code comment improvements

## Documentation Standards

1. **Clarity** - Use simple, clear language
2. **Examples** - Include practical code examples
3. **Completeness** - Cover all parameters and returns
4. **Structure** - Use consistent formatting
5. **Accuracy** - Verify against actual code

## Documentation Sections

### For APIs
- Description
- Parameters (with types)
- Returns (with types)
- Throws (possible errors)
- Examples (curl, JavaScript, Python)
- Related endpoints

### For Features
- Overview
- Prerequisites
- Step-by-step instructions
- Expected outcomes
- Troubleshooting
- Related topics
```

#### 示例 2：子代理分配实战

```markdown
# Scenario: Building a Payment Feature

## User Request
"Build a secure payment processing feature that integrates with Stripe"

## Main Agent Flow

1. **Planning Phase**
   - Understands requirements
   - Determines tasks needed
   - Plans architecture

2. **Delegates to Code Reviewer Subagent**
   - Task: "Review the payment processing implementation for security"
   - Context: Auth, API keys, token handling
   - Reviews for: SQL injection, key exposure, HTTPS enforcement

3. **Delegates to Test Engineer Subagent**
   - Task: "Create comprehensive tests for payment flows"
   - Context: Success scenarios, failures, edge cases
   - Creates tests for: Valid payments, declined cards, network failures, webhooks

4. **Delegates to Documentation Writer Subagent**
   - Task: "Document the payment API endpoints"
   - Context: Request/response schemas
   - Produces: API docs with curl examples, error codes

5. **Synthesis**
   - Main agent collects all outputs
   - Integrates findings
   - Returns complete solution to user
```

#### 示例 3：工具权限范围

**限制性设置——仅限于特定命令**

```yaml
---
name: secure-reviewer
description: Security-focused code review with minimal permissions
tools: read, grep
---

# Secure Code Reviewer

Reviews code for security vulnerabilities only.

This agent:
- ✅ Reads files to analyze
- ✅ Searches for patterns
- ❌ Cannot execute code
- ❌ Cannot modify files
- ❌ Cannot run tests

This ensures the reviewer doesn't accidentally break anything.
```

**扩展设置——所有工具用于实现**

```yaml
---
name: implementation-agent
description: Full implementation capabilities for feature development
tools: read, write, bash, grep, edit, glob
---

# Implementation Agent

Builds features from specifications.

This agent:
- ✅ Reads specifications
- ✅ Writes new code files
- ✅ Runs build commands
- ✅ Searches codebase
- ✅ Edits existing files
- ✅ Finds files matching patterns

Full capabilities for independent feature development.
```

### 子代理上下文管理

```mermaid
graph TB
    A["主代理上下文<br/>50,000 tokens"]
    B["子代理 1 上下文<br/>20,000 tokens"]
    C["子代理 2 上下文<br/>20,000 tokens"]
    D["子代理 3 上下文<br/>20,000 tokens"]

    A -->|Clean slate| B
    A -->|Clean slate| C
    A -->|Clean slate| D

    B -->|Results only| A
    C -->|Results only| A
    D -->|Results only| A

    style A fill:#e1f5ff
    style B fill:#fff9c4
    style C fill:#fff9c4
    style D fill:#fff9c4
```

### 何时使用子代理

| 场景 | 使用子代理 | 原因 |
|----------|--------------|-----|
| 复杂功能，多步骤 | 支持 | 分离关注点，防止上下文污染 |
| 快速代码审查 | 不支持 | 不必要的开销 |
| 并行任务执行 | 支持 | 每个子代理有自己的上下文 |
| 需要专业技能 | 支持 | 自定义系统提示词 |
| 长时间运行分析 | 支持 | 防止主上下文耗尽 |
| 单个任务 | 不支持 | 徒增延迟 |

### 代理团队

代理团队协调多个代理共同处理相关任务。不是一次分配给一个子代理，而是让主代理编排一组共享中间结果、协作并朝着共同目标努力的代理。这对于大规模任务（如全栈功能开发）很有用，前端代理、后端代理和测试代理并行工作。

---

## 记忆

### 概述

记忆使 Claude 能够跨会话和对话保留上下文。它有两种形式：claude.ai 中的自动综合，以及 Claude Code 中的基于文件系统的 CLAUDE.md。

### 记忆架构

```mermaid
graph TB
    A["Claude 会话"]
    B["用户输入"]
    C["记忆系统"]
    D["记忆存储"]

    B -->|用户提供信息| C
    C -->|每 24 小时综合| D
    D -->|自动加载| A
    A -->|使用上下文| C
```

### Claude Code 中的记忆层级（7 层）

Claude Code 从 7 层加载记忆，从高到低优先级：

```mermaid
graph TD
    A["1. 托管策略<br/>企业管理员策略"] --> B["2. 项目记忆<br/>./CLAUDE.md"]
    B --> C["3. 项目规则<br/>.claude/rules/*.md"]
    C --> D["4. 用户记忆<br/>~/.claude/CLAUDE.md"]
    D --> E["5. 用户规则<br/>~/.claude/rules/*.md"]
    E --> F["6. 本地记忆<br/>.claude/local/CLAUDE.md"]
    F --> G["7. 自动记忆<br/>自动捕获的偏好"]

    style A fill:#fce4ec,stroke:#333,color:#333
    style B fill:#e1f5fe,stroke:#333,color:#333
    style C fill:#e1f5fe,stroke:#333,color:#333
    style D fill:#f3e5f5,stroke:#333,color:#333
    style E fill:#f3e5f5,stroke:#333,color:#333
    style F fill:#e8f5e9,stroke:#333,color:#333
    style G fill:#fff3e0,stroke:#333,color:#333
```

### 记忆位置表

| 层级 | 位置 | 范围 | 优先级 | 共享 | 最适合 |
|------|----------|-------|----------|--------|----------|
| 1. 托管策略 | 企业管理员 | 组织 | 最高 | 所有组织用户 | 合规、安全策略 |
| 2. 项目 | `./CLAUDE.md` | 项目 | 高 | 团队（Git） | 团队标准、架构 |
| 3. 项目规则 | `.claude/rules/*.md` | 项目 | 高 | 团队（Git） | 模块化项目约定 |
| 4. 用户 | `~/.claude/CLAUDE.md` | 个人 | 中 | 个人 | 个人偏好 |
| 5. 用户规则 | `~/.claude/rules/*.md` | 个人 | 中 | 个人 | 个人规则模块 |
| 6. 本地 | `.claude/local/CLAUDE.md` | 本地 | 低 | 不共享 | 机器特定设置 |
| 7. 自动记忆 | 自动 | 会话 | 最低 | 个人 | 学习的偏好、模式 |

### 自动记忆

自动记忆自动捕获会话期间观察到的用户偏好和模式。Claude 从你的交互中学习并记住：

- 编码风格偏好
- 你所做的常见修正
- 框架和工具选择
- 沟通风格偏好

自动记忆在后台工作，不需要手动配置。

### 记忆更新生命周期

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant Editor as 文件系统
    participant Memory as CLAUDE.md

    User->>Claude: "Remember: use async/await"
    Claude->>User: "Which memory file?"
    User->>Claude: "Project memory"
    Claude->>Editor: Open ~/.claude/settings.json
    Claude->>Memory: Write to ./CLAUDE.md
    Memory-->>Claude: File saved
    Claude->>Claude: Load updated memory
    Claude-->>User: "Memory saved!"
```

### 实践示例

#### 示例 1：项目记忆结构

**文件：** `./CLAUDE.md`

```markdown
# Project Configuration

## Project Overview
- **Name**: E-commerce Platform
- **Tech Stack**: Node.js, PostgreSQL, React 18, Docker
- **Team Size**: 5 developers
- **Deadline**: Q4 2025

## Architecture
@docs/architecture.md
@docs/api-standards.md
@docs/database-schema.md

## Development Standards

### Code Style
- Use Prettier for formatting
- Use ESLint with airbnb config
- Maximum line length: 100 characters
- Use 2-space indentation

### Naming Conventions
- **Files**: kebab-case (user-controller.js)
- **Classes**: PascalCase (UserService)
- **Functions/Variables**: camelCase (getUserById)
- **Constants**: UPPER_SNAKE_CASE (API_BASE_URL)
- **Database Tables**: snake_case (user_accounts)

### Git Workflow
- Branch names: `feature/description` or `fix/description`
- Commit messages: Follow conventional commits
- PR required before merge
- All CI/CD checks must pass
- Minimum 1 approval required

### Testing Requirements
- Minimum 80% code coverage
- All critical paths must have tests
- Use Jest for unit tests
- Use Cypress for E2E tests
- Test filenames: `*.test.ts` or `*.spec.ts`

### API Standards
- RESTful endpoints only
- JSON request/response
- Use HTTP status codes correctly
- Version API endpoints: `/api/v1/`
- Document all endpoints with examples

### Database
- Use migrations for schema changes
- Never hardcode credentials
- Use connection pooling
- Enable query logging in development
- Regular backups required

### Deployment
- Docker-based deployment
- Kubernetes orchestration
- Blue-green deployment strategy
- Automatic rollback on failure
- Database migrations run before deploy

## Common Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | Start development server |
| `npm test` | Run test suite |
| `npm run lint` | Check code style |
| `npm run build` | Build for production |
| `npm run migrate` | Run database migrations |

## Team Contacts
- Tech Lead: Sarah Chen (@sarah.chen)
- Product Manager: Mike Johnson (@mike.j)
- DevOps: Alex Kim (@alex.k)

## Known Issues & Workarounds
- PostgreSQL connection pooling limited to 20 during peak hours
- Workaround: Implement query queuing
- Safari 14 compatibility issues with async generators
- Workaround: Use Babel transpiler

## Related Projects
- Analytics Dashboard: `/projects/analytics`
- Mobile App: `/projects/mobile`
- Admin Panel: `/projects/admin`
```

#### 示例 2：目录特定记忆

**文件：** `./src/api/CLAUDE.md`

```markdown
# API Module Standards

This file overrides root CLAUDE.md for everything in /src/api/

## API-Specific Standards

### Request Validation
- Use Zod for schema validation
- Always validate input
- Return 400 with validation errors
- Include field-level error details

### Authentication
- All endpoints require JWT token
- Token in Authorization header
- Token expires after 24 hours
- Implement refresh token mechanism

### Response Format

All responses must follow this structure:

```json
{
  "success": true,
  "data": { /* actual data */ },
  "timestamp": "2025-11-06T10:30:00Z",
  "version": "1.0"
}
```

### Error responses:
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "User message",
    "details": { /* field errors */ }
  },
  "timestamp": "2025-11-06T10:30:00Z"
}
```

### Pagination
- Use cursor-based pagination (not offset)
- Include `hasMore` boolean
- Limit max page size to 100
- Default page size: 20

### Rate Limiting
- 1000 requests per hour for authenticated users
- 100 requests per hour for public endpoints
- Return 429 when exceeded
- Include retry-after header

### Caching
- Use Redis for session caching
- Cache duration: 5 minutes default
- Invalidate on write operations
- Tag cache keys with resource type
```

#### 示例 3：个人记忆

**文件：** `~/.claude/CLAUDE.md`

```markdown
# My Development Preferences

## About Me
- **Experience Level**: 8 years full-stack development
- **Preferred Languages**: TypeScript, Python
- **Communication Style**: Direct, with examples
- **Learning Style**: Visual diagrams with code

## Code Preferences

### Error Handling
I prefer explicit error handling with try-catch blocks and meaningful error messages.
Avoid generic errors. Always log errors for debugging.

### Comments
Use comments for WHY, not WHAT. Code should be self-documenting.
Comments should explain business logic or non-obvious decisions.

### Testing
I prefer TDD (test-driven development).
Write tests first, then implementation.
Focus on behavior, not implementation details.

### Architecture
I prefer modular, loosely-coupled design.
Use dependency injection for testability.
Separate concerns (Controllers, Services, Repositories).

## Debugging Preferences
- Use console.log with prefix: `[DEBUG]`
- Include context: function name, relevant variables
- Use stack traces when available
- Always include timestamps in logs

## Communication
- Explain complex concepts with diagrams
- Show concrete examples before explaining theory
- Include before/after code snippets
- Summarize key points at the end

## Project Organization
I organize my projects as:
```
project/
  ├── src/
  │   ├── api/
  │   ├── services/
  │   ├── models/
  │   └── utils/
  ├── tests/
  ├── docs/
  └── docker/
```

## Tooling
- **IDE**: VS Code with vim keybindings
- **Terminal**: Zsh with Oh-My-Zsh
- **Format**: Prettier (100 char line length)
- **Linter**: ESLint with airbnb config
- **Test Framework**: Jest with React Testing Library
```

#### 示例 4：会话期间更新记忆

**会话交互：**

```markdown
User: Remember that I prefer using React hooks instead of class components
     for all new components.

Claude: I'm adding that to your memory. Which memory file should this go in?
        1. Project memory (./CLAUDE.md)
        2. Personal memory (~/.claude/CLAUDE.md)

User: Project memory

Claude: ✅ Memory saved!

Added to ./CLAUDE.md:

### Component Development
- Use functional components with React Hooks
- Prefer hooks over class components
- Custom hooks for reusable logic
- Use useCallback for event handlers
- Use useMemo for expensive computations
```

### Claude Web/桌面中的记忆

#### 记忆综合时间线

```mermaid
graph LR
    A["第 1 天：用户<br/>对话"] -->|24 小时| B["第 2 天：记忆<br/>综合"]
    B -->|自动| C["记忆更新<br/>已总结"]
    C -->|加载到| D["第 2-N 天：<br/>新对话"]
    D -->|添加到| E["记忆"]
    E -->|24 小时后| F["记忆刷新"]
```

**示例记忆摘要：**

```markdown
## Claude's Memory of User

### Professional Background
- Senior full-stack developer with 8 years experience
- Focus on TypeScript/Node.js backends and React frontends
- Active open source contributor
- Interested in AI and machine learning

### Project Context
- Currently building e-commerce platform
- Tech stack: Node.js, PostgreSQL, React 18, Docker
- Working with team of 5 developers
- Using CI/CD and blue-green deployments

### Communication Preferences
- Prefers direct, concise explanations
- Likes visual diagrams and examples
- Appreciates code snippets
- Explains business logic in comments

### Current Goals
- Improve API performance
- Increase test coverage to 90%
- Implement caching strategy
- Document architecture
```

### 记忆功能对比

| 功能 | Claude Web/桌面 | Claude Code（CLAUDE.md） |
|---------|-------------------|------------------------|
| 自动综合 | 每 24 小时 | 手动 |
| 跨项目 | 共享 | 项目特定 |
| 团队访问 | 共享项目 | Git 跟踪 |
| 可搜索 | 内置 | 通过 `/memory` |
| 可编辑 | 聊天中 | 直接文件编辑 |
| 导入/导出 | 支持 | 支持 |
| 持久化 | 24 小时+ | 无限期 |

---

## MCP 协议

### 概述

MCP（Model Context Protocol）是 Claude 访问外部工具、API 和实时数据源的标准方式。与记忆不同，MCP 提供对变化数据的实时访问。

### MCP 架构

```mermaid
graph TB
    A["Claude"]
    B["MCP 服务器"]
    C["外部服务"]

    A -->|请求：list_issues| B
    B -->|查询| C
    C -->|数据| B
    B -->|响应| A

    A -->|请求：create_issue| B
    B -->|操作| C
    C -->|结果| B
    B -->|响应| A
```

### MCP 生态系统

```mermaid
graph TB
    A["Claude"] -->|MCP| B["文件系统<br/>MCP 服务器"]
    A -->|MCP| C["GitHub<br/>MCP 服务器"]
    A -->|MCP| D["数据库<br/>MCP 服务器"]
    A -->|MCP| E["Slack<br/>MCP 服务器"]
    A -->|MCP| F["Google Docs<br/>MCP 服务器"]

    B -->|文件 I/O| G["本地文件"]
    C -->|API| H["GitHub 仓库"]
    D -->|查询| I["PostgreSQL/MySQL"]
    E -->|消息| J["Slack 工作区"]
    F -->|文档| K["Google Drive"]
```

### MCP 设置流程

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant Config as 配置文件
    participant Service as 外部服务

    User->>Claude: Type /mcp
    Claude->>Claude: List available MCP servers
    Claude->>User: Show options
    User->>Claude: Select GitHub MCP
    Claude->>Config: Update configuration
    Config->>Claude: Activate connection
    Claude->>Service: Test connection
    Service-->>Claude: Authentication successful
    Claude->>User: ✅ MCP connected!
```

### 可用 MCP 服务器表

| MCP 服务器 | 用途 | 常用工具 | 认证 | 实时 |
|------------|---------|--------------|------|-----------|
| **文件系统** | 文件操作 | read、write、delete | OS 权限 | 支持 |
| **GitHub** | 仓库管理 | list_prs、create_issue、push | OAuth | 支持 |
| **Slack** | 团队沟通 | send_message、list_channels | Token | 支持 |
| **数据库** | SQL 查询 | query、insert、update | 凭据 | 支持 |
| **Google Docs** | 文档访问 | read、write、share | OAuth | 支持 |
| **Asana** | 项目管理 | create_task、update_status | API Key | 支持 |
| **Stripe** | 支付数据 | list_charges、create_invoice | API Key | 支持 |
| **记忆** | 持久化记忆 | store、retrieve、delete | 本地 | 不支持 |

### 实践示例

#### 示例 1：GitHub MCP 配置

**文件：** `.mcp.json`（项目范围）或 `~/.claude.json`（用户范围）

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**可用 GitHub MCP 工具：**

```markdown
# GitHub MCP Tools

## Pull Request Management
- `list_prs` - List all PRs in repository
- `get_pr` - Get PR details including diff
- `create_pr` - Create new PR
- `update_pr` - Update PR description/title
- `merge_pr` - Merge PR to main branch
- `review_pr` - Add review comments

## Issue Management
- `list_issues` - List all issues
- `get_issue` - Get issue details
- `create_issue` - Create new issue
- `close_issue` - Close issue
- `add_comment` - Add comment to issue

## Repository Information
- `get_repo_info` - Repository details
- `list_files` - File tree structure
- `get_file_content` - Read file contents
- `search_code` - Search across codebase

## Commit Operations
- `list_commits` - Commit history
- `get_commit` - Specific commit details
- `create_commit` - Create new commit
```

#### 示例 2：数据库 MCP 设置

**配置：**

```json
{
  "mcpServers": {
    "database": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-database"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost/mydb"
      }
    }
  }
}
```

**示例使用：**

```markdown
User: Fetch all users with more than 10 orders

Claude: I'll query your database to find that information.

# Using MCP database tool:
SELECT u.*, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id
HAVING COUNT(o.id) > 10
ORDER BY order_count DESC;

# Results:
- Alice: 15 orders
- Bob: 12 orders
- Charlie: 11 orders
```

#### 示例 3：多 MCP 工作流

**场景：每日报告生成**

```markdown
# Daily Report Workflow using Multiple MCPs

## Setup
1. GitHub MCP - fetch PR metrics
2. Database MCP - query sales data
3. Slack MCP - post report
4. Filesystem MCP - save report

## Workflow

### Step 1: Fetch GitHub Data
/mcp__github__list_prs completed:true last:7days

Output:
- Total PRs: 42
- Average merge time: 2.3 hours
- Review turnaround: 1.1 hours

### Step 2: Query Database
SELECT COUNT(*) as sales, SUM(amount) as revenue
FROM orders
WHERE created_at > NOW() - INTERVAL '1 day'

Output:
- Sales: 247
- Revenue: $12,450

### Step 3: Generate Report
Combine data into HTML report

### Step 4: Save to Filesystem
Write report.html to /reports/

### Step 5: Post to Slack
Send summary to #daily-reports channel

Final Output:
✅ Report generated and posted
📊 47 PRs merged this week
💰 $12,450 in daily sales
```

#### 示例 4：文件系统 MCP 操作

**配置：**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    }
  }
}
```

**可用操作：**

| 操作 | 命令 | 用途 |
|-----------|---------|---------|
| 列出文件 | `ls ~/projects` | 显示目录内容 |
| 读文件 | `cat src/main.ts` | 读文件内容 |
| 写文件 | `create docs/api.md` | 创建新文件 |
| 编辑文件 | `edit src/app.ts` | 修改文件 |
| 搜索 | `grep "async function"` | 在文件中搜索 |
| 删除 | `rm old-file.js` | 删除文件 |

### MCP vs 记忆：决策矩阵

```mermaid
graph TD
    A["需要外部数据？"]
    A -->|否| B["使用记忆"]
    A -->|是| C["频繁变化？"]
    C -->|否/很少| B
    C -->|是/经常| D["使用 MCP"]

    B -->|存储| E["偏好<br/>上下文<br/>历史"]
    D -->|访问| F["实时 API<br/>数据库<br/>服务"]

    style B fill:#e1f5ff
    style D fill:#fff9c4
```

### 请求/响应模式

```mermaid
sequenceDiagram
    participant App as Claude
    participant MCP as MCP 服务器
    participant DB as 数据库

    App->>MCP: 请求："SELECT * FROM users WHERE id=1"
    MCP->>DB: 执行查询
    DB-->>MCP: 结果集
    MCP-->>App: Return parsed data
    App->>App: Process result
    App->>App: Continue task

    Note over MCP,DB: 实时访问<br/>无缓存
```

---

## 代理技能

### 概述

代理技能是打包为包含指令、脚本和资源的可复用模型调用能力。Claude 自动检测并使用相关技能。

### 技能架构

```mermaid
graph TB
    A["技能目录"]
    B["SKILL.md"]
    C["YAML 元数据"]
    D["指令"]
    E["脚本"]
    F["模板"]

    A --> B
    B --> C
    B --> D
    E --> A
    F --> A
```

### 技能加载流程

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude
    participant System as 系统
    participant Skill as 技能

    User->>Claude: "Create Excel report"
    Claude->>System: Scan available skills
    System->>System: Load skill metadata
    Claude->>Claude: Match user request to skills
    Claude->>Skill: Load xlsx skill SKILL.md
    Skill-->>Claude: Return instructions + tools
    Claude->>Claude: Execute skill
    Claude->>User: Generate Excel file
```

### 技能类型和位置表

| 类型 | 位置 | 范围 | 共享 | 同步 | 最适合 |
|------|----------|-------|--------|------|----------|
| 预构建 | 内置 | 全局 | 所有用户 | 自动 | 文档创建 |
| 个人 | `~/.claude/skills/` | 个人 | 否 | 手动 | 个人自动化 |
| 项目 | `.claude/skills/` | 团队 | 支持 | Git | 团队标准 |
| 插件 | 通过插件安装 | 变化 | 取决于 | 自动 | 集成功能 |

### 预构建技能

```mermaid
graph TB
    A["预构建技能"]
    B["PowerPoint (pptx)"]
    C["Excel (xlsx)"]
    D["Word (docx)"]
    E["PDF"]

    A --> B
    A --> C
    A --> D
    A --> E

    B --> B1["Create presentations"]
    B --> B2["Edit slides"]
    C --> C1["Create spreadsheets"]
    C --> C2["Analyze data"]
    D --> D1["Create documents"]
    D --> D2["Format text"]
    E --> E1["Generate PDFs"]
    E --> E2["Fill forms"]
```

### 捆绑技能

Claude Code 现在包含 5 个开箱即用的捆绑技能：

| 技能 | 命令 | 用途 |
|-------|---------|---------|
| **Simplify** | `/simplify` | 简化复杂代码或解释 |
| **Batch** | `/batch` | 跨多个文件或项目运行操作 |
| **Debug** | `/debug` | 系统化调试问题并进行根因分析 |
| **Loop** | `/loop` | 按计时器安排重复任务 |
| **Claude API** | `/claude-api` | 直接与 Anthropic API 交互 |

这些捆绑技能始终可用，无需安装或配置。

### 实践示例

#### 示例 1：自定义代码审查技能

**目录结构：**

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

**文件：** `~/.claude/skills/code-review/SKILL.md`

```yaml
---
name: Code Review Specialist
description: Comprehensive code review with security, performance, and quality analysis
version: "1.0.0"
tags:
  - code-review
  - quality
  - security
when_to_use: When users ask to review code, analyze code quality, or evaluate pull requests
effort: high
shell: bash
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

#### 示例 2：品牌调性技能

**目录结构：**

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

**文件：** `.claude/skills/brand-voice/SKILL.md`

```yaml
---
name: Brand Voice Consistency
description: Ensure all communication matches brand voice and tone guidelines
tags:
  - brand
  - writing
  - consistency
when_to_use: When creating marketing copy, customer communications, or public-facing content
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
```

#### 示例 3：文档生成器技能

**文件：** `.claude/skills/doc-generator/SKILL.md`

```yaml
---
name: API Documentation Generator
description: Generate comprehensive, accurate API documentation from source code
version: "1.0.0"
tags:
  - documentation
  - api
  - automation
when_to_use: When creating or updating API documentation
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

- Description
- Parameters (with types)
- Returns (with types)
- Throws (possible errors)
- Examples (curl, JavaScript, Python)
- Related endpoints
```

### 技能发现和调用

```mermaid
graph TD
    A["用户请求"] --> B["Claude 分析"]
    B -->|扫描| C["可用技能"]
    C -->|元数据检查| D["技能描述匹配？"]
    D -->|是| E["加载 SKILL.md"]
    D -->|否| F["尝试下一个技能"]
    F -->|有更多技能？| D
    F -->|没有更多| G["使用通用知识"]
    E --> H["提取指令"]
    H --> I["执行技能"]
    I --> J["返回结果"]
```

### 技能 vs 其他功能

```mermaid
graph TB
    A["扩展 Claude"]
    B["斜杠命令"]
    C["子代理"]
    D["记忆"]
    E["MCP"]
    F["技能"]

    A --> B
    A --> C
    A --> D
    A --> E
    A --> F

    B -->|用户调用| G["快速快捷方式"]
    C -->|自动分配| H["隔离上下文"]
    D -->|持久化| I["跨会话上下文"]
    E -->|实时| J["外部数据访问"]
    F -->|自动调用| K["自主执行"]
```

---

## Claude Code 插件

### 概述

Claude Code 插件是将自定义（斜杠命令、子代理、MCP 服务器和钩子）打包的集合，通过一条命令安装。它们代表最高级的扩展机制——将多个功能组合成内聚的、可分享的包。

### 架构

```mermaid
graph TB
    A["插件"]
    B["斜杠命令"]
    C["子代理"]
    D["MCP 服务器"]
    E["钩子"]
    F["配置"]

    A -->|打包| B
    A -->|打包| C
    A -->|打包| D
    A -->|打包| E
    A -->|打包| F
```

### 插件加载流程

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant Plugin as 插件市场
    participant Install as 安装
    participant SlashCmds as 斜杠命令
    participant Subagents
    participant MCPServers as MCP 服务器
    participant Hooks
    participant Tools as 已配置工具

    User->>Claude: /plugin install pr-review
    Claude->>Plugin: Download plugin manifest
    Plugin-->>Claude: Return plugin definition
    Claude->>Install: Extract components
    Install->>SlashCmds: Configure
    Install->>Subagents: Configure
    Install->>MCPServers: Configure
    Install->>Hooks: Configure
    SlashCmds-->>Tools: Ready to use
    Subagents-->>Tools: Ready to use
    MCPServers-->>Tools: Ready to use
    Hooks-->>Tools: Ready to use
    Tools-->>Claude: Plugin installed ✅
```

### 插件类型和分发

| 类型 | 范围 | 共享 | 权限 | 示例 |
|------|-------|--------|-----------|----------|
| 官方 | 全局 | 所有用户 | Anthropic | PR 审查、安全指导 |
| 社区 | 公开 | 所有用户 | 社区 | DevOps、数据科学 |
| 组织 | 内部 | 团队成员 | 公司 | 内部标准、工具 |
| 个人 | 个人 | 单用户 | 开发者 | 自定义工作流 |

### 插件结构

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── task-1.md
│   ├── task-2.md
│   └── workflows/
├── agents/
│   ├── specialist-1.md
│   ├── specialist-2.md
│   └── configs/
├── skills/
│   ├── skill-1.md
│   └── skill-2.md
├── hooks/
│   └── hooks.json
├── .mcp.json
├── .lsp.json
├── settings.json
├── templates/
│   └── issue-template.md
├── scripts/
│   ├── helper-1.sh
│   └── helper-2.py
├── docs/
│   ├── README.md
│   └── USAGE.md
└── tests/
    └── plugin.test.js
```

### 实践示例

#### 示例 1：PR 审查插件

**文件：** `.claude-plugin/plugin.json`

```json
{
  "name": "pr-review",
  "version": "1.0.0",
  "description": "Complete PR review workflow with security, testing, and docs",
  "author": {
    "name": "Anthropic"
  },
  "license": "MIT"
}
```

**安装：**

```bash
/plugin install pr-review

# Result:
# ✅ 3 slash commands installed
# ✅ 3 subagents configured
# ✅ 2 MCP servers connected
# ✅ 4 hooks registered
# ✅ Ready to use!
```

#### 示例 2：DevOps 插件

**组件：**

```
devops-automation/
├── commands/
│   ├── deploy.md
│   ├── rollback.md
│   ├── status.md
│   └── incident.md
├── agents/
│   ├── deployment-specialist.md
│   ├── incident-commander.md
│   └── alert-analyzer.md
├── mcp/
│   ├── github-config.json
│   ├── kubernetes-config.json
│   └── prometheus-config.json
├── hooks/
│   ├── pre-deploy.js
│   ├── post-deploy.js
│   └── on-error.js
└── scripts/
    ├── deploy.sh
    ├── rollback.sh
    └── health-check.sh
```

#### 示例 3：文档插件

**打包组件：**

```
documentation/
├── commands/
│   ├── generate-api-docs.md
│   ├── generate-readme.md
│   ├── sync-docs.md
│   └── validate-docs.md
├── agents/
│   ├── api-documenter.md
│   ├── code-commentator.md
│   └── example-generator.md
├── mcp/
│   ├── github-docs-config.json
│   └── slack-announce-config.json
└── templates/
    ├── api-endpoint.md
    ├── function-docs.md
    └── adr-template.md
```

### 插件功能对比

| 功能 | 斜杠命令 | 技能 | 子代理 | 插件 |
|---------|---------------|-------|----------|--------|
| **安装** | 手动复制 | 手动复制 | 手动配置 | 一条命令 |
| **设置时间** | 5 分钟 | 10 分钟 | 15 分钟 | 2 分钟 |
| **打包** | 单文件 | 单文件 | 单文件 | 多个 |
| **版本控制** | 手动 | 手动 | 手动 | 自动 |
| **团队共享** | 复制文件 | 复制文件 | 复制文件 | 安装 ID |
| **更新** | 手动 | 手动 | 手动 | 自动可用 |
| **依赖** | 无 | 无 | 无 | 可能包含 |
| **市场** | 否 | 否 | 否 | 支持 |
| **分发** | 仓库 | 仓库 | 仓库 | 市场 |

### 何时创建插件

```mermaid
graph TD
    A["应该创建插件？"]
    A -->|需要多个组件| B{"多个命令<br/>或子代理<br/>或 MCP？"}
    B -->|是| C["✅ 创建插件"]
    B -->|否| D["使用单个功能"]
    A -->|团队工作流| E{"与<br/>团队共享？"}
    E -->|是| C
    E -->|否| F["保留为本地设置"]
    A -->|复杂设置| G{"需要自动<br/>配置？"}
    G -->|是| C
    G -->|否| D
```

### 插件 vs 手动配置

**手动设置（2+ 小时）：**
- 一个一个安装斜杠命令
- 单独创建子代理
- 分别配置 MCP
- 手动设置钩子
- 记录一切
- 与团队分享（希望他们配置正确）

**使用插件（2 分钟）：**
```bash
/plugin install pr-review
# ✅ Everything installed and configured
# ✅ Ready to use immediately
# ✅ Team can reproduce exact setup
```

---

## 对比与集成

### 功能对比矩阵

| 功能 | 调用方式 | 持久化 | 范围 | 使用场景 |
|---------|-----------|------------|-------|----------|
| **斜杠命令** | 手动（`/cmd`） | 仅会话 | 单命令 | 快速快捷方式 |
| **子代理** | 自动分配 | 隔离上下文 | 专业化任务 | 任务分配 |
| **记忆** | 自动加载 | 跨会话 | 用户/团队上下文 | 长期学习 |
| **MCP 协议** | 自动查询 | 实时外部 | 实时数据访问 | 动态信息 |
| **技能** | 自动调用 | 基于文件系统 | 可复用专业知识 | 自动化工作流 |

### 交互时间线

```mermaid
graph LR
    A["会话开始"] -->|加载| B["记忆（CLAUDE.md）"]
    B -->|发现| C["可用技能"]
    C -->|注册| D["斜杠命令"]
    D -->|连接| E["MCP 服务器"]
    E -->|就绪| F["用户交互"]

    F -->|输入 /cmd| G["斜杠命令"]
    F -->|请求| H["技能自动调用"]
    F -->|查询| I["MCP 数据"]
    F -->|复杂任务| J["分配给子代理"]

    G -->|使用| B
    H -->|使用| B
    I -->|使用| B
    J -->|使用| B
```

### 实际集成示例：客户支持自动化

#### 架构

```mermaid
graph TB
    User["客户邮件"] -->|接收| Router["支持路由器"]

    Router -->|分析| Memory["记忆<br/>客户历史"]
    Router -->|查询| MCP1["MCP：客户数据库<br/>历史工单"]
    Router -->|检查| MCP2["MCP：Slack<br/>团队状态"]

    Router -->|路由复杂| Sub1["子代理：技术支持<br/>上下文：技术问题"]
    Router -->|路由简单| Sub2["子代理：计费<br/>上下文：支付问题"]
    Router -->|路由紧急| Sub3["子代理：升级<br/>上下文：优先处理"]

    Sub1 -->|格式化| Skill1["技能：响应生成器<br/>保持品牌调性"]
    Sub2 -->|格式化| Skill2["技能：响应生成器"]
    Sub3 -->|格式化| Skill3["技能：响应生成器"]

    Skill1 -->|生成| Output["格式化响应"]
    Skill2 -->|生成| Output
    Skill3 -->|生成| Output

    Output -->|发布| MCP3["MCP：Slack<br/>通知团队"]
    Output -->|发送| Reply["客户回复"]
```

#### 请求流程

```markdown
## Customer Support Request Flow

### 1. Incoming Email
"I'm getting error 500 when trying to upload files. This is blocking my workflow!"

### 2. Memory Lookup
- Loads CLAUDE.md with support standards
- Checks customer history: VIP customer, 3rd incident this month

### 3. MCP Queries
- GitHub MCP: List open issues (finds related bug report)
- Database MCP: Check system status (no outages reported)
- Slack MCP: Check if engineering is aware

### 4. Skill Detection & Loading
- Request matches "Technical Support" skill
- Loads support response template from Skill

### 5. Subagent Delegation
- Routes to Tech Support Subagent
- Provides context: customer history, error details, known issues
- Subagent has full access to: read, bash, grep tools

### 6. Subagent Processing
Tech Support Subagent:
- Searches codebase for 500 error in file upload
- Finds recent change in commit 8f4a2c
- Creates workaround documentation

### 7. Skill Execution
Response Generator Skill:
- Uses Brand Voice guidelines
- Formats response with empathy
- Includes workaround steps
- Links to related documentation

### 8. MCP Output
- Posts update to #support Slack channel
- Tags engineering team
- Updates ticket in Jira MCP

### 9. Response
Customer receives:
- Empathetic acknowledgment
- Explanation of cause
- Immediate workaround
- Timeline for permanent fix
- Link to related issues
```

### 完整功能编排

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant Memory as 记忆<br/>CLAUDE.md
    participant MCP as MCP 服务器
    participant Skills as 技能
    participant SubAgent as 子代理

    User->>Claude: Request: "Build auth system"
    Claude->>Memory: Load project standards
    Memory-->>Claude: Auth standards, team practices
    Claude->>MCP: Query GitHub for similar implementations
    MCP-->>Claude: Code examples, best practices
    Claude->>Skills: Detect matching Skills
    Skills-->>Claude: Security Review Skill + Testing Skill
    Claude->>SubAgent: Delegate implementation
    SubAgent->>SubAgent: Build feature
    Claude->>Skills: Apply Security Review Skill
    Skills-->>Claude: Security checklist results
    Claude->>SubAgent: Delegate testing
    SubAgent-->>Claude: Test results
    Claude->>User: Complete system delivered
```

### 何时使用每个功能

```mermaid
graph TD
    A["新任务"] --> B{任务类型？}

    B -->|重复工作流| C["斜杠命令"]
    B -->|需要实时数据| D["MCP 协议"]
    B -->|下次记住| E["记忆"]
    B -->|专业化子任务| F["子代理"]
    B -->|领域特定工作| G["技能"]

    C --> C1["✅ 团队快捷方式"]
    D --> D1["✅ 实时 API 访问"]
    E --> E1["✅ 持久上下文"]
    F --> F1["✅ 并行执行"]
    G --> G1["✅ 自动调用专业知识"]
```

### 选择决策树

```mermaid
graph TD
    Start["需要扩展 Claude？"]

    Start -->|快速重复任务| A{"手动或自动？"}
    A -->|手动| B["斜杠命令"]
    A -->|自动| C["技能"]

    Start -->|需要外部数据| D{"实时？"}
    D -->|是| E["MCP 协议"]
    D -->|否/跨会话| F["记忆"]

    Start -->|复杂项目| G{"多个角色？"}
    G -->|是| H["子代理"]
    G -->|否| I["技能 + 记忆"]

    Start -->|长期上下文| J["记忆"]
    Start -->|团队工作流| K["斜杠命令 +<br/>记忆"]
    Start -->|完全自动化| L["技能 +<br/>子代理 +<br/>MCP"]
```

---

## 摘要表

| 方面 | 斜杠命令 | 子代理 | 记忆 | MCP | 技能 | 插件 |
|--------|---|---|---|---|---|---|
| **设置难度** | 简单 | 中等 | 简单 | 中等 | 中等 | 简单 |
| **学习曲线** | 低 | 中等 | 低 | 中等 | 中等 | 低 |
| **团队收益** | 高 | 高 | 中等 | 高 | 高 | 非常高 |
| **自动化水平** | 低 | 高 | 中等 | 高 | 高 | 非常高 |
| **上下文管理** | 单会话 | 隔离 | 持久 | 实时 | 持久 | 所有功能 |
| **维护负担** | 低 | 中等 | 低 | 中等 | 中等 | 低 |
| **可扩展性** | 良好 | 优秀 | 良好 | 优秀 | 优秀 | 优秀 |
| **可分享性** | 一般 | 一般 | 良好 | 良好 | 良好 | 优秀 |
| **版本控制** | 手动 | 手动 | 手动 | 手动 | 手动 | 自动 |
| **安装** | 手动复制 | 手动配置 | 不适用 | 手动配置 | 手动复制 | 一条命令 |

---

## 快速开始指南

### 第一周：简单开始
- 为常见任务创建 2-3 个斜杠命令
- 在设置中启用记忆
- 在 CLAUDE.md 中记录团队标准

### 第二周：添加实时访问
- 设置 1 个 MCP（GitHub 或数据库）
- 使用 `/mcp` 配置
- 在工作流中查询实时数据

### 第三周：分配工作
- 为特定角色创建第一个子代理
- 使用 `/agents` 命令
- 用简单任务测试分配

### 第四周：自动化一切
- 为重复自动化创建第一个技能
- 使用技能市场或构建自定义
- 组合所有功能实现完整工作流

### 持续进行
- 每月审查和更新记忆
- 随着模式出现添加新技能
- 优化 MCP 查询
- 完善子代理提示词

---

## 钩子

### 概述

钩子是事件驱动的 shell 命令，自动响应 Claude Code 事件执行。它们支持自动化、验证和自定义工作流，无需手动干预。

### 钩子事件

Claude Code 支持跨四种钩子类型的 **25 个钩子事件**：

| 钩子事件 | 触发 | 使用场景 |
|------------|---------|-----------|
| **SessionStart** | 会话开始/恢复/清除/压缩 | 环境设置、初始化 |
| **InstructionsLoaded** | CLAUDE.md 或规则文件加载 | 验证、转换、增强 |
| **UserPromptSubmit** | 用户提交提示词 | 输入验证、提示词过滤 |
| **PreToolUse** | 任何工具运行前 | 验证、审批门禁、日志记录 |
| **PermissionRequest** | 显示权限对话框 | 自动批准/拒绝流程 |
| **PostToolUse** | 工具成功后 | 自动格式化、通知、清理 |
| **PostToolUseFailure** | 工具执行失败 | 错误处理、日志记录 |
| **Notification** | 发送通知 | 警报、外部集成 |
| **SubagentStart** | 产生子代理 | 上下文注入、初始化 |
| **SubagentStop** | 子代理结束 | 结果验证、日志记录 |
| **Stop** | Claude 响应完成 | 摘要生成、清理任务 |
| **StopFailure** | API 错误结束回合 | 错误恢复、日志记录 |
| **TeammateIdle** | 代理团队队友空闲 | 工作分配、协调 |
| **TaskCompleted** | 任务标记为完成 | 任务后处理 |
| **TaskCreated** | 通过 TaskCreate 创建任务 | 任务跟踪、日志记录 |
| **ConfigChange** | 配置文件更改 | 验证、传播 |
| **CwdChanged** | 工作目录更改 | 目录特定设置 |
| **FileChanged** | 监视文件更改 | 文件监控、重建触发 |
| **PreCompact** | 上下文压缩前 | 状态保留 |
| **PostCompact** | 压缩完成 | 压缩后操作 |
| **WorktreeCreate** | 正在创建 worktree | 环境设置、依赖安装 |
| **WorktreeRemove** | 正在移除 worktree | 清理、资源释放 |
| **Elicitation** | MCP 服务器请求用户输入 | 输入验证 |
| **ElicitationResult** | 用户响应征求 | 响应处理 |
| **SessionEnd** | 会话终止 | 清理、最终日志记录 |

### 常用钩子

钩子在 `~/.claude/settings.json`（用户级）或 `.claude/settings.json`（项目级）中配置：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write $CLAUDE_FILE_PATH"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "eslint $CLAUDE_FILE_PATH"
          }
        ]
      }
    ]
  }
}
```

### 钩子环境变量

- `$CLAUDE_FILE_PATH` - 正在编辑/写入的文件路径
- `$CLAUDE_TOOL_NAME` - 正在使用的工具名称
- `$CLAUDE_SESSION_ID` - 当前会话标识符
- `$CLAUDE_PROJECT_DIR` - 项目目录路径

### 最佳实践

**应该做：**
- 保持钩子快速（< 1 秒）
- 使用钩子进行验证和自动化
- 优雅处理错误
- 使用绝对路径

**不应该做：**
- 使钩子交互式
- 使用钩子运行长时间任务
- 硬编码凭据

**参见**：[06-hooks/](06-hooks/) 详细示例

---

## 检查点和回退

### 概述

检查点允许你保存对话状态并回退到之前的点，支持安全实验和探索多种方法。

### 核心概念

| 概念 | 描述 |
|---------|-------------|
| **检查点** | 包含消息、文件和上下文的对话状态快照 |
| **回退** | 返回到之前的检查点，丢弃后续更改 |
| **分支点** | 探索多种方法的起点检查点 |

### 访问检查点

检查点随每个用户提示词自动创建。要回退：

```bash
# 按 Esc 两次打开检查点浏览器
Esc + Esc

# 或使用 /rewind 命令
/rewind
```

选择检查点时，有五个选项：
1. **恢复代码和对话** —— 两者都回退到该点
2. **恢复对话** —— 回退消息，保留当前代码
3. **恢复代码** —— 回退文件，保留对话
4. **从此处总结** —— 将对话压缩成摘要
5. **算了** —— 取消

### 使用场景

| 场景 | 工作流 |
|----------|----------|
| **探索方法** | 保存 → 尝试 A → 保存 → 回退 → 尝试 B → 比较 |
| **安全重构** | 保存 → 重构 → 测试 → 如果失败：回退 |
| **A/B 测试** | 保存 → 设计 A → 保存 → 回退 → 设计 B → 比较 |
| **错误恢复** | 发现问题 → 回退到最后良好状态 |

### 配置

```json
{
  "autoCheckpoint": true
}
```

**参见**：[08-checkpoints/](08-checkpoints/) 详细示例

---

## 高级功能

### 规划模式

在编码前创建详细实施计划。

**激活：**
```bash
/plan Implement user authentication system
```

**好处：**
- 带时间估算的清晰路线图
- 风险评估
- 系统化任务分解
- 审查和修改的机会

### 扩展思考

复杂问题的深度推理。

**激活：**
- 会话期间按 `Alt+T`（macOS 上为 `Option+T`）切换
- 设置 `MAX_THINKING_TOKENS` 环境变量进行编程控制

```bash
# 通过环境变量启用扩展思考
export MAX_THINKING_TOKENS=50000
claude -p "Should we use microservices or monolith?"
```

**好处：**
- 全面分析权衡
- 更好的架构决策
- 考虑边缘情况
- 系统化评估

### 后台任务

不阻塞对话地运行长时间操作。

**使用：**
```bash
User: Run tests in background

Claude: Started task bg-1234

/task list           # Show all tasks
/task status bg-1234 # Check progress
/task show bg-1234   # View output
/task cancel bg-1234 # Cancel task
```

### 权限模式

控制 Claude 可以做什么。

| 模式 | 描述 | 使用场景 |
|------|-------------|----------|
| **default** | 对危险操作提示的标准权限 | 一般开发 |
| **acceptEdits** | 自动接受文件编辑而不确认 | 信任的编辑工作流 |
| **plan** | 仅分析和规划，不修改文件 | 代码审查、架构规划 |
| **auto** | 自动批准安全操作，仅对危险操作提示 | 平衡自主性和安全性 |
| **dontAsk** | 执行所有操作而不确认提示 | 有经验的用户、自动化 |
| **bypassPermissions** | 完全无限制访问，无安全检查 | CI/CD pipeline、信任的脚本 |

**使用：**
```bash
claude --permission-mode plan          # 只读分析
claude --permission-mode acceptEdits   # 自动接受编辑
claude --permission-mode auto          # 自动批准安全操作
claude --permission-mode dontAsk       # 无确认提示
```

### 无头模式（打印模式）

使用 `-p`（打印）标志运行 Claude Code 而无需交互输入，用于自动化和 CI/CD。

**使用：**
```bash
# Run specific task
claude -p "Run all tests"

# Pipe input for analysis
cat error.log | claude -p "explain this error"

# CI/CD integration (GitHub Actions)
- name: AI Code Review
  run: claude -p "Review PR changes and report issues"

# JSON output for scripting
claude -p --output-format json "list all functions in src/"
```

### 计划任务

使用 `/loop` 命令按重复计划运行任务。

**使用：**
```bash
/loop every 30m "Run tests and report failures"
/loop every 2h "Check for dependency updates"
/loop every 1d "Generate daily summary of code changes"
```

计划任务在后台运行，完成时报告结果。它们对持续监控、定期检查和自动化维护工作流很有用。

### Chrome 集成

Claude Code 可以与 Chrome 浏览器集成用于 Web 自动化任务。这支持在开发工作流中直接导航网页、填写表单、截取屏幕截图和从网站提取数据等能力。

### 会话管理

管理多个工作会话。

**命令：**
```bash
/resume                # 恢复上一个对话
/rename "Feature"      # 命名当前会话
/fork                  # Fork 成新会话
claude -c              # 继续最近的对话
claude -r "Feature"    # 按名称/ID 恢复会话
```

### 交互功能

**键盘快捷键：**
- `Ctrl + R` - 搜索命令历史
- `Tab` - 自动补全
- `↑ / ↓` - 命令历史
- `Ctrl + L` - 清屏

### 配置

完整配置示例：

```json
{
  "planning": {
    "autoEnter": true,
    "requireApproval": true
  },
  "extendedThinking": {
    "enabled": true,
    "showThinkingProcess": true
  },
  "backgroundTasks": {
    "enabled": true,
    "maxConcurrentTasks": 5
  },
  "permissions": {
    "mode": "default"
  }
}
```

**参见**：[09-advanced-features/](09-advanced-features/) 综合指南

---

## 资源

- [Claude Code 文档](https://code.claude.com/docs/en/overview)
- [Anthropic 文档](https://docs.anthropic.com)
- [MCP GitHub 服务器](https://github.com/modelcontextprotocol/servers)
- [Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook)

---

*最后更新：2026 年 3 月*
*适用于 Claude Haiku 4.5、Sonnet 4.6 和 Opus 4.6*
*现在包含：钩子、检查点、规划模式、扩展思考、后台任务、权限模式（6 种模式）、无头模式、会话管理、自动记忆、代理团队、计划任务、Chrome 集成、频道、语音听写和捆绑技能*
