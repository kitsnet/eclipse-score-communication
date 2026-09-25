# Compact change record

Suggested location: `<component>/maintenance/changes/<date>-<slug>.md`.
Use an existing equivalent record/PR when it preserves the same information.
Keep current status in one place; retain material decisions, not a step-by-step chat log.

```markdown
# <Change>

Status: proposed | in progress | ready for review | accepted | paused | blocked
Baseline: <commit; component; platform; relevant tooling version>
Trigger: <request/defect/changed need/drift/retirement; source>
Scope: <requested outcome; relevant exclusions>

## Decisions and uncertainties
<Authored contract vs observed behavior; proposal; actual human decisions if given.
For paused work, preserve why and what would permit resumption.>

## Impact
| Artifact / ID | Reason and semantic change or re-pin | Old → new | Dependents / check |
|---|---|---|---|

## Validation
| Command or review | Configuration / included inputs | Result | Warnings / exclusions |
|---|---|---|---|

## Outcome and handoff
<Final delta; durable knowledge updated; open items with stable IDs;
remaining acceptance/validation; next action if any.>
```

Use `not applicable` only with a reason. Use `not run` or `blocked` for missing
execution. Record acceptance and its source separately from technical completion.
Attach large evidence only when needed; preserve its location/revision in the record.
