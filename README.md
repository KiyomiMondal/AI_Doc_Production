GitHub push to main
        │
        ▼
.github/workflows/docs.yml
        │
        ├─ clones this repo into /tmp/engine
        ├─ copies:
        │    - scripts/
        │    - prompts/
        │    - templates/
        │    - ai-docs/ (if missing)
        │
        ├─ generates repository context
        │    └─ scripts/generate-context.sh
        │         ├─ scans repo structure
        │         ├─ extracts dependency/stack hints
        │         ├─ collects route/controller/auth/frontend/prisma slices
        │         └─ writes context/*.txt
        │
        ├─ generates progress doc
        │    └─ scripts/generate-progress.sh
        │         ├─ reads:
        │         │    - prompts/progress.txt
        │         │    - templates/progress.md
        │         │    - changes.diff
        │         │    - commit.txt
        │         ├─ optionally checks .llmignore
        │         ├─ truncates diff
        │         ├─ calls Groq API
        │         ├─ merges AI-managed blocks
        │         └─ writes ai-docs/progress.md
        │
        ├─ waits 30s
        │
        ├─ generates architecture doc
        │    └─ scripts/generate-architecture.sh
        │         ├─ detects stack from repo files
        │         ├─ invokes generate-context.sh
        │         ├─ reads:
        │         │    - prompts/architecture.txt
        │         │    - templates/architecture.md
        │         │    - changes.diff
        │         │    - commit.txt
        │         │    - context/*.txt
        │         ├─ calls Groq API
        │         ├─ merges AI-managed blocks
        │         └─ writes ai-docs/architecture.md
        │
        ├─ waits 30s
        │
        ├─ generates dfd doc
        │    └─ scripts/generate-dfd.sh
        │         ├─ checks if ai-docs/dfd.md exists
        │         ├─ enables “full analysis mode” if missing
        │         ├─ invokes generate-context.sh
        │         ├─ reads:
        │         │    - prompts/dfd.txt
        │         │    - templates/dfd.md
        │         │    - changes.diff or broader repo snapshot
        │         │    - commit.txt
        │         │    - context/*.txt
        │         ├─ calls Groq API
        │         ├─ merges AI-managed Mermaid blocks
        │         └─ writes ai-docs/dfd.md
        │
        ├─ waits 30s
        │
        ├─ generates todos doc
        │    └─ scripts/generate-todos.sh
        │         ├─ scans repo for TODO/FIXME/HACK/NOTE/XXX/BUG
        │         ├─ inspects changes.diff for new TODOs
        │         ├─ reads:
        │         │    - prompts/todos.txt
        │         │    - templates/todos.md
        │         │    - changes.diff
        │         │    - commit.txt
        │         ├─ calls Groq API
        │         ├─ merges AI-managed todo blocks
        │         └─ writes ai-docs/todos.md
        │
        └─ commits if changed
             ├─ git add ai-docs/ changes.diff commit.txt
             ├─ skip commit if no staged diff
             └─ commit message includes [skip ci]
