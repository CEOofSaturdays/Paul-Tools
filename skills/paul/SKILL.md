---
name: paul
description: Use when an agent needs to operate Paul through the `paul` CLI, set up or authenticate the CLI, inspect or transition work items, manage calendars or settings, or understand Paul's domain to project to task to sub-task model and workflow rules.
---

# Paul CLI (`paul`)

Use this skill to operate the Paul platform through `paul`.

## Mental Model

- Treat `work-item` as the canonical CLI surface. A work item can be a `project`, `task`, or `sub_task`.
- Treat the hierarchy as `Domain -> Project -> Task -> Sub-task`.
- Treat `Domain` as the top-level operational boundary for people, workflows, mailboxes, and work.
- Treat `Project` as a larger work container inside a domain.
- Treat `Task` as the standard executable work item under a project.
- Treat `Sub-task` as the granular executable work item under a task.
- Treat categories as labels for organizing work items inside a domain.
- Treat workflows as the source of truth for statuses, stages, stage gates, and custom fields.
- Treat work-item keys as domain-scoped and sequential. Sub-task keys inherit the parent task key with a suffix.
- Treat external calendar events as read-only synced data. Change scheduled work through `work-item schedule`, not calendar event mutation.
- Treat settings as human-owned surfaces. Workflow agent run tokens must not manage `paul settings ...`.

## Setup and Auth

Install and bootstrap the CLI:

```bash
paul install-check
paul init
paul doctor
```

Packaged `paul` releases are published to the public Paul Agent Tools release repository (`CEOofSaturdays/Paul-Tools`). `paul install-check` and `paul update` use that repository by default.

Validate a release target before installing or updating:

```bash
paul install-check
paul install-check --stable
paul install-check --tag v1.0.0
```

Update an existing CLI install in place:

```bash
paul update
paul update --stable
paul update --tag v1.0.0
```

Configure a profile with a Personal Access Token:

```bash
paul profile set \
  --app-url "https://app.paul.dev" \
  --token "oi_pat_..." \
  --agent-id "agent-alpha"
```

Verify auth and identity:

```bash
paul auth check
paul whoami
paul profile show
```

Remember:

- Use `--app-url`; `--api-url` is intentionally unsupported.
- `paul profile set` reports `resolvedApiUrl` and `apiResolutionSource`.
- Use `PAUL_API_URL` only as a one-off troubleshooting override.
- `paul update` defaults to the latest normal release. Use `--tag` for an exact release.
- Use `--repo <owner/repo>` only for alternate release buckets or private test releases.
- If an install or update check reports a GitHub `404 Not Found`, confirm the public release repository exists, has an initialized default branch, and includes the requested release tag and assets. Do not set a GitHub token for normal Paul CLI install or update flows.
- Use PATs for CLI access. `read_write` covers queue, work-item, and calendar operations. `admin` is only needed for admin endpoints.

## Operating Rules

- Prefer `paul --json ...` for machine use and automation.
- Run `paul schema` when you need the machine-readable command contract instead of scraping help text.
- Treat non-zero exit codes as failures.
- Treat `paul install-check` failures as release-readiness failures. In JSON mode, `install_not_ready` means the target release assets are incomplete or unavailable.
- In `--json` mode, read success payloads from stdout and failure payloads from stderr.
- Use `paul help`, `paul help <topic>`, and `paul help all` for built-in guides.
- Use `paul settings --help` and nested `settings` help for settings discovery.
- Prefer canonical command groups:
  - `work-item`, not `task`
  - `calendar source list`, not `calendar-source list`
- Use forgiving aliases only when you inherit them from existing automation:
  - `paul calendar events` -> `paul calendar day`
  - `paul sources list` -> `paul calendar source list`
- Use `paul domain list` when domain id, key, or selector is unclear.
- Expect server-side enforcement for transitions, stage gates, and custom-field requirements.
- When authoring workflows or custom fields through settings, draft first and apply only after explicit human confirmation.

## Required Work Item Hygiene

- Keep the work item updated while you work, not just at the end.
- Read `work-item agent-context` and `work-item custom-fields get` before changing status, stage, or custom fields.
- Add an internal comment when you start meaningful work, hit a blocker, make a material decision, finish a milestone, hand off, or complete the item.
- Keep the work-item description current with the latest implementation state, decisions, blockers, verification, and any context the next agent or human would need.
- Update important custom fields whenever your work changes information those fields are meant to track. Common examples are implementation notes, verification results, blockers, external links, handoff details, or other workflow-specific execution metadata.
- Do not invent custom-field keys. Use `custom-fields get` and the current workflow context to determine which fields exist and which are required.
- Before running `progress`, `complete`, or `done`, make sure the description, the latest comment, and any required or materially relevant custom fields reflect the work that was actually performed.

## Workflow Authoring

- Treat workflow and custom-field setup as a human-owned planning/apply flow, not an agent-run-token flow.
- Prefer the workflow-pack commands when creating or updating a demo workflow with multiple stages or fields.
- Draft the pack, show the result to the human, and only apply after they confirm.
- After apply, verify the created workflows and fields through `list`, `get`, and `fields list`.

Draft and apply a workflow pack:

```bash
paul settings domain workflows pack draft --domain <DOMAIN> --file <pack.json>
paul settings domain workflows pack apply --domain <DOMAIN> --file <pack.json>
```

Inspect workflows and fields:

```bash
paul settings domain workflows list --domain <DOMAIN>
paul settings domain workflows get <WORKFLOW_KEY> --domain <DOMAIN>
paul settings domain workflows fields list <WORKFLOW_KEY> --domain <DOMAIN>
```

Remember:

- Use a PAT-backed profile for `paul settings ...` commands.
- Workflow agent run tokens must not mutate settings.
- The CLI exposes workflow-pack draft/apply, but recurring monthly template management is not yet a first-class `paul` command.
- For a meeting demo, it is valid to author the workflow once, then have a human create the new monthly task manually and attach the current files.

## Common Flows

Inspect queue and available work:

```bash
paul queue list
paul domain list
```

Inspect one work item:

```bash
paul work-item get <WORK_ITEM_KEY>
paul work-item transitions <WORK_ITEM_KEY>
paul work-item agent-context <WORK_ITEM_KEY>
paul work-item custom-fields get <WORK_ITEM_KEY>
```

Collaborate on a work item:

```bash
paul work-item update <WORK_ITEM_KEY> --description "Current implementation status, decisions, blockers, and verification notes"
paul work-item comment create <WORK_ITEM_KEY> --body "Recorded update"
paul work-item custom-fields update <WORK_ITEM_KEY> --values-json '{"field_key":"value"}'
```

Move work through the workflow:

```bash
paul work-item claim <WORK_ITEM_KEY> --to-status in_progress
paul work-item progress <WORK_ITEM_KEY> --to-stage <STAGE_KEY> --accept-requirements-text
paul work-item complete <WORK_ITEM_KEY> --to-status completed
paul work-item done <WORK_ITEM_KEY>
```

Schedule work and inspect calendars:

```bash
paul work-item schedule <WORK_ITEM_KEY> --date 2026-03-01
paul work-item schedule <WORK_ITEM_KEY> --start 2026-03-01T15:00:00Z --end 2026-03-01T16:00:00Z
paul --json calendar agenda --date today --brief-json
paul --json calendar agenda --from 2026-03-01 --to 2026-03-07 --include-scheduled-tasks
paul calendar day --date 2026-03-01
paul calendar week --start-date 2026-03-01
paul calendar event <EVENT_ID>
paul calendar source list
```

Inspect human-owned settings surfaces:

```bash
paul settings personal profile get
paul settings domain workflows list --domain ME
paul settings admin ai-providers list
```

Author a monthly-demo workflow:

```bash
paul settings domain workflows pack draft --domain OPS --file ./workflow-pack.json
paul settings domain workflows pack apply --domain OPS --file ./workflow-pack.json
paul settings domain workflows get MONTHLY_SPEND_RECON --domain OPS
paul settings domain workflows fields list MONTHLY_SPEND_RECON --domain OPS
```

Run a manual monthly task through the workflow:

```bash
paul work-item create --kind task --parent-key OPS-123 --title "March 2026 spend reconciliation"
paul work-item attachment upload OPS-456 --file ./1-consolidated.xlsx
paul work-item attachment upload OPS-456 --file ./u_corp_orders.csv
paul work-item attachment upload OPS-456 --file ./u_ppe_orders.csv
paul work-item update OPS-456 --description "Waiting for agent reconciliation output"
paul work-item progress OPS-456 --to-stage execution
```

## Command Selection Hints

- Use `queue list` to discover work waiting for action.
- Use `work-item get` when you need the current record.
- Use `work-item transitions` when you need to know which statuses are valid next.
- Use `work-item agent-context` when you need stage goals, blocked exits, or required fields before acting.
- Use `work-item progress` to pass a stage gate.
- Use `work-item complete` to move to a terminal status.
- Use `work-item done` when you want the CLI to auto-progress through valid stage gates to completion.
- Use `calendar agenda --brief-json` for compact daily brief or agent-sync context with chronological external events and scheduled work.
- Use `calendar day` or `calendar week` to inspect read-only external events alongside scheduled work.
- Use `calendar source list` to inspect synced sources.
- Use `settings domain workflows pack draft` when you want a safe, reviewable workflow authoring step before apply.
- Use `settings domain workflows pack apply` only after the human has approved the pack.

## Constraints to Remember

- Claim and complete flows require an `agent_id` either in the active profile or passed explicitly.
- Settings commands require human-owned credentials; workflow agent run tokens are intentionally rejected.
- Work-item keys stay stable and are the safest selector for mutation commands.
- Calendar sync is ICS-only and one-way.
