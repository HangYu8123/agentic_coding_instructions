---
name: 'Master Orchestrator'
description: 'Routes user requests to the appropriate workflow instruction file'
applyTo: '**'
---

# Master Orchestrator — Instruction Router

This repo has structured workflow instructions. **Before doing any work**, classify the user's request into one of the categories below and then **read and follow** the corresponding instruction file in full.

## Safety Rules (always apply)
- **DO NOT TRY TO COMMIT CHANGES TO GITHUB**
- **DO NOT WRITE SPAM FILES INTO THE REPO**
- **DO NOT USE SUDO**

## Request Classification

Analyze the user's prompt and determine which **one** category **best matches**.
Use the trigger phrases as soft signals, not strict rules.
Classify based on the user's primary intent, even if none of the exact keywords appear.
Do not choose based on keyword overlap alone.
If multiple categories seem possible, pick the one that best reflects the main action the user wants.

### Category: Code Implementation
Trigger keywords / intent: implement, add, create, build, refactor, update, modify, write code, new feature, change behavior
Instruction file to follow: `.github/instructions/workflow/code.instructions.md`

### Category: Debug
Trigger keywords / intent: debug, fix, error, bug, crash, broken, failing, not working, traceback, exception, investigate issue
Instruction file to follow: `.github/instructions/workflow/debug.instructions.md`

### Category: Query / Q&A
Trigger keywords / intent: explain, what is, how does, where is, why, describe, summarize, document, question about code
Instruction file to follow: `.github/instructions/workflow/query.instructions.md`

### Category: Correctness Check
Trigger keywords / intent: test, verify, check, validate, review, audit, examine, ensure correctness, consistency check
Instruction file to follow: `.github/instructions/workflow/correctness_check.instructions.md`

### Category: Initialize Repo
Trigger keywords / intent: initialize, init, setup repo, create overview, bootstrap, first-time setup
Instruction file to follow: `.github/instructions/workflow/initialize.instructions.md`

## Routing Procedure

1. **Read** the user's prompt carefully.
2. **Classify** it into exactly one category from the table above. If ambiguous, prefer the category that best matches the primary intent.
3. **Read the matched instruction file** in its entirety using the file path from the table.
4. **Also read** `.github/instructions/general/philosophy.instructions.md` for general guidelines.
5. **Follow** the matched instruction file step-by-step to complete the request.

## If multiple intents are present
Ideally the user should use one type of request at a time. If the user's request clearly spans multiple categories (e.g., "fix this bug and then add a new feature"), handle them sequentially:
1. First complete the **Debug** workflow for the fix.
2. Then complete the **Code Implementation** workflow for the new feature.

## Repo context files
When any workflow instruction tells you to read context files (like `codebase_overview.md`, `scripts_overview.md`, `update_logs.md`, and etc. ), look for them under `.github/instructions/repo_info/`.
