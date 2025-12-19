<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Spec Claude Development (SCD)** is a portable and extensible framework for Spec-Driven Development. This is NOT an application, but a reusable meta-framework built on OpenSpec that provides an "army" of specialized AI agents and a structured workflow to assist in efficient and accurate code generation from formal specifications.

### Tech Stack
- **Core Standard**: OpenSpec CLI and workflow (`@fission-ai/openspec`)
- **Agent Framework**: Custom multi-agent system defined in `.claude/agents/` via Markdown
- **Slash Commands**: Custom commands in `.claude/commands/` for common workflows
- **Primary Language**: Shell scripts (bash/zsh) for tooling, Markdown for agent definitions and specs
- **Environment**: Unix-like environment with `node` and `npm` for OpenSpec CLI

## Common Commands

### OpenSpec Workflow
```bash
# List active changes and specifications
openspec list                  # List active changes
openspec list --specs          # List all specifications
openspec spec list --long      # List specs with details

# View details
openspec show [item]           # Display change or spec details
openspec show [change] --json --deltas-only  # Debug delta parsing

# Validation
openspec validate [change] --strict  # Validate changes comprehensively
openspec validate              # Bulk validation mode

# Project management
openspec init [path]           # Initialize OpenSpec in a project
openspec update [path]         # Update instruction files

# Archive completed changes
openspec archive <change-id> --yes  # Archive after deployment
```

### Slash Commands
```bash
# OpenSpec-related commands
/openspec:proposal  # Scaffold a new OpenSpec change proposal
/openspec:apply     # Implement an approved OpenSpec change
/openspec:archive   # Archive a deployed OpenSpec change

# Development workflow commands
/worktree           # Create git worktree for feature development
/worktree-from-issue  # Create worktree from GitHub issue
/issues             # Manage GitHub issues
/ideation           # Product ideation workflow
```

### Git Workflow
```bash
# Conventional commits (reference OpenSpec change)
git commit -m "feat(change-id): implement feature"
git commit -m "fix(change-id): resolve bug"

# Create feature branches
git worktree add ../feature-name -b feature-name
```

## Architecture

### SCD Agent Framework (`.claude/`)

The framework provides specialized AI agents, each with a specific role:

#### Available Agents
- **backend-developer**: Python backend development with hexagonal architecture
- **backend-test-engineer**: Backend unit testing with pytest
- **frontend-developer**: React frontend development with feature-based architecture
- **frontend-test-engineer**: Frontend unit testing with React Testing Library and Vitest
- **shadcn-ui-architect**: shadcn/ui component design and implementation
- **ui-ux-analyzer**: UI/UX review and improvement recommendations
- **qa-criteria-validator**: Acceptance criteria validation with Playwright
- **pydantic-ai-architect**: Pydantic AI agent system design and optimization

#### Agent Usage Pattern
Agents are invoked automatically by Claude Code based on task type. Each agent:
- Researches and proposes implementation plans
- Creates documentation in `.claude/doc/{feature_name}/{agent}.md`
- Reports findings and recommendations
- Parent agent (you) performs actual implementation based on their plans

### OpenSpec Workflow (`openspec/`)

Source of truth for all specifications, following a three-stage workflow:

#### Directory Structure
```
openspec/
├── project.md              # Project conventions and context
├── specs/                  # Current truth - what IS built
│   └── [capability]/       # Single focused capability
│       ├── spec.md         # Requirements and scenarios
│       └── design.md       # Technical patterns
├── changes/                # Proposals - what SHOULD change
│   ├── [change-id]/
│   │   ├── proposal.md     # Why, what, impact
│   │   ├── tasks.md        # Implementation checklist
│   │   ├── design.md       # Technical decisions (optional)
│   │   └── specs/          # Delta changes
│   │       └── [capability]/
│   │           └── spec.md # ADDED/MODIFIED/REMOVED
│   └── archive/            # Completed changes
```

#### Three-Stage Workflow

**Stage 1: Creating Changes** (Use `/openspec:proposal`)
1. Review `openspec/project.md` and `openspec list --specs`
2. Choose unique verb-led `change-id` (e.g., `add-user-auth`, `update-payment-flow`)
3. Scaffold `proposal.md`, `tasks.md`, optional `design.md`
4. Draft spec deltas using `## ADDED|MODIFIED|REMOVED Requirements`
5. Validate with `openspec validate <id> --strict`

**Stage 2: Implementing Changes** (Use `/openspec:apply`)
1. Read `proposal.md`, `design.md`, and `tasks.md`
2. Implement tasks sequentially
3. Update checklist after completion
4. Do NOT start until proposal is approved

**Stage 3: Archiving Changes** (Use `/openspec:archive`)
1. Move to `changes/archive/YYYY-MM-DD-[name]/`
2. Update `specs/` if capabilities changed
3. Validate with `openspec validate --strict`

## Development Guidelines

### Agent Workflow (CRITICAL)

#### Phase 1: Planning
- Create `.claude/sessions/context_session_{feature_name}.md` at start
- Consult relevant subagents (run in parallel when possible)
- Update context file with plan and agent recommendations

#### Phase 2: Implementation
- Read `.claude/sessions/context_session_{feature_name}.md` for full context
- Update context file after each phase
- Context file contains history, overall plan, and agent contributions

#### Phase 3: Validation
- Use `qa-criteria-validator` for final validation
- Review report and implement feedback
- Iterate until acceptance criteria pass

### OpenSpec Conventions

**When to Create Proposals:**
- New features or functionality
- Breaking changes (API, schema, architecture)
- Performance optimization changing behavior
- Security pattern updates

**Skip Proposals For:**
- Bug fixes (restore intended behavior)
- Typos, formatting, comments
- Non-breaking dependency updates
- Configuration changes

**Spec File Format:**
```markdown
## ADDED Requirements
### Requirement: Feature Name
The system SHALL provide...

#### Scenario: Success case
- **WHEN** user performs action
- **THEN** expected result

## MODIFIED Requirements
### Requirement: Existing Feature
[Complete modified requirement with all scenarios]

## REMOVED Requirements
### Requirement: Old Feature
**Reason**: [Why removing]
**Migration**: [How to handle]
```

**Critical Rules:**
- Every requirement MUST have at least one `#### Scenario:`
- Use `## ADDED|MODIFIED|REMOVED|RENAMED Requirements` headers
- MODIFIED requirements must include complete text (previous + new)
- Use `openspec validate --strict` before sharing proposals

### Code Style

**Shell Scripts:**
- Follow POSIX-compliance where possible
- Adhere to Google Shell Style Guide
- Comment complex logic
- Use proper error handling

**Markdown:**
- Follow CommonMark specification
- Clear, concise, parsable by language models
- Use proper formatting for agent definitions

### Git Workflow
- Use conventional commits: `type(scope): message`
- Scope should reference OpenSpec change name
- Examples:
  - `feat(add-user-auth): implement authentication flow`
  - `fix(update-ui): resolve button styling issue`
  - `docs(agents): update backend-developer agent description`

## Important Files & Directories

### Agent Definitions
- `.claude/agents/*.md`: Specialized AI agent definitions
- `.claude/commands/*.md`: Custom slash commands
- `.claude/doc/{feature}/`: Agent-generated implementation plans
- `.claude/sessions/`: Context files for feature development

### OpenSpec Files
- `openspec/project.md`: Project context and conventions
- `openspec/AGENTS.md`: Detailed OpenSpec workflow instructions
- `openspec/specs/`: Current specification truth
- `openspec/changes/`: Active change proposals

### Configuration
- `.mcp.json`: Model Context Protocol configuration

## Domain Context

This is a **meta-level programming** project focused on **AI-assisted software development**. Key concepts:

- **Spec-Driven Development**: Formal specifications drive implementation
- **Multi-Agent System**: Specialized agents collaborate on development
- **Specification Delta**: Incremental changes to specifications
- **Change Proposal**: Formal proposal for modifying capabilities
- **Archiving**: Recording completed changes

### Critical Distinctions
- **SCD Framework**: This repository (the meta-framework)
- **Target Project**: The codebase where SCD is applied

## Important Constraints

1. **Strict OpenSpec Lifecycle**: Must follow `propose` → `implement` → `archive`
2. **Agent Context Passing**: Use OpenSpec change folders, not session files
3. **No Direct Implementation**: Agents propose plans, parent agent implements
4. **Validation Required**: Always run `openspec validate --strict` before submission

## Subagent Management

You have access to 8 specialized subagents. When working on features:

1. **Consult relevant agents** based on task type
2. **Pass context file** (`.claude/sessions/context_session_{feature_name}.md`)
3. **Read agent documentation** before implementing
4. **Run agents in parallel** when possible

**Agent Selection Guide:**
- UI building → `shadcn-ui-architect`
- UI review → `ui-ux-analyzer`
- Frontend logic → `frontend-developer`
- Frontend tests → `frontend-test-engineer`
- Backend logic → `backend-developer`
- Backend tests → `backend-test-engineer`
- AI agents → `pydantic-ai-architect`
- Final validation → `qa-criteria-validator`

## External Dependencies

**Critical Dependency:**
- **OpenSpec CLI** (`@fission-ai/openspec`): Entire workflow orchestrated through `openspec` command

**Optional Tools:**
- Git worktree support for parallel development
- GitHub CLI (`gh`) for issue management
- Standard Unix tools (`rg`, `ls`, `cat`, etc.)
