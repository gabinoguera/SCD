# Project Context: Spec Claude Development (SCD)

## Purpose
This repository contains **Spec Claude Development (SCD)**, a portable and extensible framework for Spec-Driven Development. SCD provides an "army" of AI agents and a structured workflow, built on top of the OpenSpec standard, to assist in the efficient and accurate generation of code from formal specifications. This project is not an application, but a reusable framework intended to be incorporated into other software projects.

## Tech Stack
- **Core Standard**: OpenSpec CLI and workflow (`@fission-ai/openspec`).
- **Agent Framework**: A custom multi-agent system managed through Markdown files.
- **Primary Language**: Shell scripts (`bash`, `zsh`) for tooling, and Markdown for agent definitions and specifications.
- **Environment**: Assumes a Unix-like environment with `node` and `npm` available for the OpenSpec CLI.

## Project Conventions

### Code Style
- Shell scripts should be POSIX-compliant where possible and follow the Google Shell Style Guide.
- Markdown files must adhere to CommonMark specification. Agent definitions and specs should be clear, concise, and easily parsable by language models.

### Architecture Patterns
The SCD framework is built on two primary components:
1.  **SCD Agent Framework (`.claude/`)**: Contains the definitions of all specialized AI agents. Each agent has a specific role and interacts with the OpenSpec workflow.
2.  **OpenSpec Workflow (`openspec/`)**: The source-of-truth for all project specifications, managed by the OpenSpec CLI. It contains the canonical specs (`specs/`), active changes (`changes/`), and a record of completed work (`archive/`).

### Testing Strategy
- The framework itself is validated through usage. Each successful feature implementation in a project using SCD serves as a test case.
- The `qa-criteria-validator` agent is a core part of the workflow, acting as the primary testing mechanism by comparing implementation against the `spec_delta.md`.

### Git Workflow
- The state of the project is managed through OpenSpec changes, not directly through Git branches for feature development.
- Commits should be conventional (`type(scope): message`). The scope should often reference the OpenSpec change name (e.g., `feat(auth): implement add-user-auth change`).

## Domain Context
- The primary domain is **AI-assisted software development** and **meta-level programming**.
- Agents must understand the distinction between the **SCD Framework** (this repository) and the **Target Project** (the codebase where SCD is being applied).
- Key concepts: `Spec-Driven Development`, `Multi-Agent System`, `Specification Delta`, `Change Proposal`, `Archiving`.

## Important Constraints
- The workflow **must** strictly adhere to the OpenSpec lifecycle (`propose` -> `implement` -> `archive`). The old session-file-based workflow is deprecated and must not be used.
- All agent interaction and context passing must be done through the files within an OpenSpec change folder.

## External Dependencies
- **OpenSpec CLI**: This is the only critical external dependency. The entire workflow is orchestrated through calls to the `openspec` command.
