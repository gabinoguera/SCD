# Spec Claude Development (SCD) - AI Agent Instructions

## Project Overview: Spec Claude Development (SCD)

This repository is the foundation for **Spec Claude Development (SCD)**, a framework for Spec-Driven Development. SCD is an "army" of AI agents designed to operate on top of the **OpenSpec** workflow, creating a portable and extensible system that can be integrated into any software project to assist in the efficient and accurate generation of code based on formal specifications.

This project is **not** an application itself, but a reusable framework that leverages a standard for spec management.

- **Core Philosophy**: Spec-Driven Development, powered by the OpenSpec standard.
- **Key Components**:
    1.  **SCD Agent Framework**: A multi-agent system (`.claude/agents/`) for specialized tasks.
    2.  **OpenSpec Workflow**: The underlying process and CLI tool for managing the lifecycle of specifications.

## The SCD Workflow, powered by OpenSpec

The SCD framework strictly follows the OpenSpec lifecycle. All development activities are managed through OpenSpec "changes". The old session-file-based workflow is deprecated.

### Phase 1: Proposing a Change (Planning & Specification)
1.  **Initiate a Change Proposal**: For any new feature or task, you MUST create an OpenSpec change proposal. This is done by instructing the AI to create a proposal, which will scaffold a new directory under `openspec/changes/`.
    - *Example Command*: `"Create an OpenSpec change proposal for user authentication"`
2.  **Consult Sub-Agents to Refine Specs**: Consult the necessary SCD sub-agents (e.g., `architecture-analyst`, `ui-ux-analyzer`) to analyze requirements. Their feedback MUST be used to refine the `spec_delta.md` and `tasks.md` files within the change folder.
3.  **Finalize the Plan**: The finalized plan is the set of approved files (`proposal.md`, `spec_delta.md`, `tasks.md`) inside the change folder. This is the single source of truth for the implementation.

### Phase 2: Implementing the Change
1.  **Context is the Change Folder**: Before writing any code, you MUST read the contents of the corresponding `openspec/changes/{change-name}/` directory to get the full context and plan.
2.  **Execute the Task List**: Implement the feature by strictly following the checklist in `tasks.md`. The implementation must satisfy the requirements defined in `spec_delta.md`.
3.  **Track Progress**: Mark tasks as complete in `tasks.md` as you proceed. This is the official progress tracker.

### Phase 3: Archiving the Change (QA & Merging)
1.  **Validation against Spec Delta**: After implementation, you MUST use the `qa-criteria-validator` sub-agent. This agent will generate a feedback report by comparing the implementation against the `spec_delta.md`.
2.  **Iterate on Feedback**: Implement all necessary changes based on the QA report until the acceptance criteria are met.
3.  **Archive the Change**: Once QA is passed and all tasks are complete, you MUST instruct the AI to archive the change. This merges the `spec_delta.md` into the source-of-truth specs located in `openspec/specs/` and moves the change folder to `openspec/archive/`.
    - *Example Command*: `"Archive the OpenSpec change for user authentication"`

## SCD + OpenSpec Architecture

The framework is composed of two primary components that work in tandem.

### `Component: SCD Agent Framework`
- **Purpose**: To provide specialized AI assistants (the "army") that execute tasks within the OpenSpec workflow.
- **Location**: `.claude/`
- **Specification**:
    - Each agent is defined by a markdown file in `.claude/agents/`.
    - An agent's definition includes its purpose, capabilities, and how it interacts with the OpenSpec workflow.

### `Component: OpenSpec Workflow`
- **Purpose**: To provide the structure, lifecycle, and CLI tooling for managing specifications as the source of truth.
- **Location**: `openspec/`
- **Components**:
    - `specs/`: The canonical, versioned source of truth for all project specifications.
    - `changes/`: Contains isolated folders for each feature or bug fix currently in progress.
    - `archive/`: A record of all completed and merged changes.
    - `openspec` **CLI**: The tool used by agents to manage the workflow (`proposal`, `validate`, `archive`, etc.).

## Agent Management

- **Orchestration**: The primary agent (you) orchestrates the process by invoking sub-agents and OpenSpec commands.
- **Context Passing**: When invoking a sub-agent, you MUST pass the path to the relevant OpenSpec change folder (e.g., `openspec/changes/add-user-auth`).

## Getting Started with SCD in a New Project

To incorporate the SCD framework into a new project, follow these steps:

1.  **Install & Initialize OpenSpec**: First, install the OpenSpec CLI and run `openspec init` in the project root. This creates the necessary `openspec/` directory structure.
2.  **Copy the `.claude` directory**: This contains the core SCD agent definitions and workflow structure.
3.  **Create a `.github/copilot-instructions.md` file**: Copy this file to provide instructions for the primary AI agent.
4.  **Customize Agents**: Modify the agent definitions in `.claude/agents/` to fit the specific needs of the project and to interact correctly with your tools and codebase.
5.  **Populate Initial Specs**: Work with agents to populate the `openspec/specs/` directory with the initial specifications of the existing project.
