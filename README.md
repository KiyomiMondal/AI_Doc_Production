# Automated AI Documentation Workflow

This repository includes a GitHub Actions workflow that automatically generates and updates project documentation whenever code is pushed to the `main` branch.

## Workflow Overview

```text
GitHub Push to main
│
▼
.github/workflows/docs.yml
│
├─ clones this repo into /tmp/engine
├─ copies:
│    ├─ scripts/
│    ├─ prompts/
│    ├─ templates/
│    └─ ai-docs/ (if missing)
│
├─ generates repository context
│    └─ scripts/generate-context.sh
│         ├─ scans repo structure
│         ├─ extracts dependency and stack hints
│         ├─ collects route/controller/auth/frontend/prisma slices
│         └─ writes context/*.txt
│
├─ generates progress doc
│    └─ scripts/generate-progress.sh
│         ├─ reads:
│         │    ├─ prompts/progress.txt
│         │    ├─ templates/progress.md
│         │    ├─ changes.diff
│         │    └─ commit.txt
│         ├─ optionally checks .llmignore
│         ├─ truncates diff
│         ├─ calls Groq API
│         ├─ merges AI-managed blocks
│         └─ writes ai-docs/progress.md
│
├─ generates architecture doc
│    └─ scripts/generate-architecture.sh
│         ├─ detects stack from repository files
│         ├─ invokes generate-context.sh
│         ├─ reads:
│         │    ├─ prompts/architecture.txt
│         │    ├─ templates/architecture.md
│         │    ├─ changes.diff
│         │    ├─ commit.txt
│         │    └─ context/*.txt
│         ├─ calls Groq API
│         ├─ merges AI-managed blocks
│         └─ writes ai-docs/architecture.md
│
├─ generates DFD doc
│    └─ scripts/generate-dfd.sh
│         ├─ checks if ai-docs/dfd.md exists
│         ├─ enables full analysis mode if missing
│         ├─ invokes generate-context.sh
│         ├─ reads:
│         │    ├─ prompts/dfd.txt
│         │    ├─ templates/dfd.md
│         │    ├─ changes.diff or broader repository snapshot
│         │    ├─ commit.txt
│         │    └─ context/*.txt
│         ├─ calls Groq API
│         ├─ merges AI-managed Mermaid blocks
│         └─ writes ai-docs/dfd.md
│
├─ generates TODOs doc
│    └─ scripts/generate-todos.sh
│         ├─ scans repository for:
│         │    ├─ TODO
│         │    ├─ FIXME
│         │    ├─ HACK
│         │    ├─ NOTE
│         │    ├─ XXX
│         │    └─ BUG
│         ├─ inspects changes.diff for new TODOs
│         ├─ reads:
│         │    ├─ prompts/todos.txt
│         │    ├─ templates/todos.md
│         │    ├─ changes.diff
│         │    └─ commit.txt
│         ├─ calls Groq API
│         ├─ merges AI-managed todo blocks
│         └─ writes ai-docs/todos.md
│
└─ commits if changed
     ├─ git add ai-docs/ changes.diff commit.txt
     ├─ skip commit if no staged diff
     └─ commit message includes [skip ci]
```

## Generated Documentation

The workflow automatically maintains the following files inside the `ai-docs/` directory:

| File              | Purpose                                                                   |
| ----------------- | ------------------------------------------------------------------------- |
| `progress.md`     | Tracks repository changes and development progress.                       |
| `architecture.md` | Documents system architecture, stack, and project structure.              |
| `dfd.md`          | Maintains Data Flow Diagrams and system interaction diagrams.             |
| `todos.md`        | Tracks TODO, FIXME, HACK, NOTE, XXX, and BUG items across the repository. |

## Context Generation

Before generating documentation, the workflow builds repository context by:

* Scanning the project structure.
* Detecting technologies and dependencies.
* Extracting routing and controller information.
* Analyzing authentication-related code.
* Collecting frontend implementation details.
* Extracting Prisma/database-related components.
* Writing reusable context files into `context/*.txt`.

## AI-Powered Documentation

Documentation generation scripts use:

* Repository context
* Commit information
* Git diffs
* Prompt templates
* Documentation templates

These inputs are sent to the Groq API to generate and update project documentation while preserving AI-managed sections and previously generated content where applicable.

## Commit Behavior

The workflow only creates a commit when documentation files change.

```bash
git add ai-docs/ changes.diff commit.txt
```

If no changes are detected, the commit step is skipped.

Generated commits include:

```text
[skip ci]
```

to prevent recursive workflow execution.
