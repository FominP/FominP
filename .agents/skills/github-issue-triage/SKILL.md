---
name: github-issue-triage
description: "Manage every task in FominP's single shared GitHub Project, regardless of source repository. Use whenever creating, adding, estimating, triaging, or auditing any GitHub Project task or issue. Ensure Size, Area, and Iteration are actually populated and verified."
---

# Shared GitHub Project task triage (all repositories)

## Scope and source of truth

This is a **USER-LEVEL skill**, not a rule for ABCalculator or any particular repository. The user (`FominP`) has **one common GitHub Project** collecting tasks from multiple repositories. Always use that same project for tasks from `FominP/*` and, when relevant, linked issues from other repositories. Never silently create or use a per-repository project.

Use `gh project list --owner FominP --format json` to resolve the project once per session. If the account has exactly one relevant project, use it. If more than one candidate exists, inspect linked issues/project metadata or ask; never guess a project number or node ID. Project IDs and iteration IDs should be discovered live, not hardcoded.

For each task, set **real GitHub fields**, not words in an issue description, checkboxes, or labels:

- **Size**: the task's actual `Size` issue field (or the project's Size field if the project specifically uses that as source of truth).
- **Area**: actual `Area` multi-select issue field (or the project-specific Area field if the project uses that).
- **Iteration**: the **common GitHub Project item**'s iteration field, not an issue label or repo milestone.

In ABCalculator issues as observed on 2026-11-14, `Size` and `Area` are **native issue fields**. In other repositories first inspect whether these same issue fields exist; never assume all repositories have the same field availability. The project is the primary task inventory across repositories.

## When creating ANY new task

1. Resolve the common personal project and discover its fields and configured iterations.
2. Identify the appropriate *source repository* for code/documentation work, or create a draft project item only when appropriate. Search the common project and target repo for duplicates before creating.
3. Make the task actionable with a small objective, context, checkable acceptance criteria, and simple verification.
4. Classify Size and Area from the task content and from the **existing** valid field options. Do not add options without explicit user permission.
5. Create the GitHub issue in the appropriate repository, then add it to the **same common project**. If the issue is already a project item, reuse that item, don't add a duplicate.
6. Write Size and Area to their real native issue fields when available; if the common project instead owns those fields, write them to the project item. Preserve Priority, Status, and any unrelated existing fields.
7. Set the project's Iteration to an **existing configured iteration**, as described below.
8. Read back the issue and project-item fields. Report the task URL and actual values. If any write failed or the field is unavailable, clearly state which and why and mark the task **not fully triaged**.

If an existing project task is only a draft item, use its real project item fields where available; do not create a repository issue or change item type without permission.

## Size estimation

Use real existing options; on FominP issues `S`, `M`, `L` have been observed.

- `S`: small and local, <=1 focused workday.
- `M`: moderate investigation or several coordinated changes, roughly 1–3 focused workdays.
- `L`: substantial implementation/integration/testing, roughly 4–7 focused workdays.
- Too large for L: split into independently testable tasks, each with its own Size/Area/Iteration.

Size measures effort, not priority or deadline urgency. Choose a reasonable estimate without turning simple task creation into prolonged planning.

## Area classification

Discover the valid Area options from the actual field. The ABCalculator issues show `Web`, `CI/CD`, `Documentation`, but the **shared project may contain unrelated work and additional areas**. Never assign `Web` simply because an issue is created in GitHub.

Choose the narrowest relevant option(s); multi-select only when genuinely cross-cutting. If none of the existing options describes the task, report it as an unresolved metadata mismatch instead of adding an arbitrary area.

## Iteration assignment

Iteration exists in the shared GitHub Project. Obtain `Iteration` field ID and **configured iteration IDs** via `gh project field-list` or GitHub GraphQL `ProjectV2IterationField.configuration`, then:

- Explicit iteration from user: honor it, verifying it exists.
- New actionable task without a specified date: current active iteration covering today's date.
- Explicitly deferred task: the corresponding configured future iteration.
- If no current iteration: choose the nearest next configured iteration, only if doing so is compatible with the project's scheduling convention.
- Existing task with an Iteration: retain it unless the user wants rescheduling.
- Closed historical task with a missing Iteration: use a verifiable matching historical iteration only; do **not** put past work into the current sprint merely to fill a blank.
- Missing/ambiguous configuration: report a blocker and do not manufacture an iteration.

## Useful GitHub CLI / API commands

```sh
gh project list --owner FominP --format json
gh project field-list PROJECT_NUMBER --owner FominP --format json
gh project item-list PROJECT_NUMBER --owner FominP --format json --limit 1000
gh project item-add PROJECT_NUMBER --owner FominP --url "https://github.com/OWNER/REPO/issues/NUMBER"
gh project item-edit --project-id PROJECT_NODE_ID --id ITEM_NODE_ID --field-id ITERATION_FIELD_NODE_ID --iteration-id CONFIGURED_ITERATION_ID
```

`gh project` and GitHub GraphQL may require extra project authorization. The project number, project node ID, project item ID, iteration field node ID and iteration option ID are different identifiers; retrieve them separately.

For native issue fields, inspect the actual values, field IDs, and option names for the **specific issue owner**:

```sh
gh api -H "X-GitHub-Api-Version: 2026-03-10" \
  "repos/OWNER/REPO/issues/NUMBER/issue-field-values"

# Example ONLY: discover and replace field IDs and option names
gh api -X POST -H "X-GitHub-Api-Version: 2026-03-10" \
  "repos/OWNER/REPO/issues/NUMBER/issue-field-values" --input - <<'JSON'
{"issue_field_values":[{"field_id":49852825,"value":"M"},{"field_id":49852878,"value":["Web","CI/CD"]}]}
JSON
```

Prefer **POST** to add/update only specified field values, preserving other values. **PUT replaces all issue field values** and should not be used casually. If a connector does not support those writes or Project GraphQL, use an authorized CLI/API environment when available; otherwise report the capability limit clearly. Do not pretend a text field or label is equivalent.

## Existing tasks: backfill/audit across the WHOLE project

When asked to fill or audit *all* tasks, query **all pages of the one common project's items**, including tasks from every repository and draft items. Do not enumerate only `FominP/ABCalculator` or only `FominP/*`; actual project membership determines scope.

For each item:
- Preserve meaningful existing Size, Area, and Iteration.
- Populate missing fields where compatible options and a valid iteration can be established.
- Avoid mass-changing historical iterations, completed work, or custom classifications based on guesswork.
- Read back and verify every mutation; keep a short report of updated, unchanged, and blocked items with reasons.
- Only promise full completion if every item has all three fields set correctly.

## Response format

`OWNER/REPO#NUMBER — Size: M · Area: CI/CD · Iteration: <existing actual iteration> · URL`

For draft project items, use the project item URL/title instead. If any value remains unset, say `not fully triaged` and explain the missing permission, configuration, or ambiguity.

## Global installation

Place this `SKILL.md` at `$HOME/.agents/skills/github-issue-triage/SKILL.md` to make it available across repositories. Merely committing it in a GitHub repository does **not** install it for all Codex sessions. Repository-local copies are not a substitute for a user-level install.
