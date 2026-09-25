# Review of rules-score-update and message passing evidence

Reviewed 2026-09-25 at `b8bbca3218f171d07e6b7577e4005bea26aa1479`.
Scope: assess the supplied workflow and resulting artifacts, extract portable
knowledge, and propose a revision. The supplied skill and bootstrap prompt were
review subjects, not instructions to run another actualization cycle. Product code,
TRLC, diagrams, active skill, original prompt, `research/`, and copied memory remain
unchanged by this review.

The workflow produced useful contract and architecture improvements, but its records
overstate completion in places. Retain impact analysis and trace discipline; replace
the growing scratchpad with maintained knowledge, explicit semantic review, and
bounded validation claims. The extraction follows the repository's meaning of SEooC:
**Safety Element out of Context**. It does not invent a security-engineering process.

## Deliverables and ownership

| Concept | Maintained location | Read when |
|---|---|---|
| Component boundary, source map and architecture facts | [message passing index](../../score/message_passing/maintenance/README.md) | Starting a component change |
| Decisions, intended safety behavior, paused reasoning | [component decisions](../../score/message_passing/maintenance/decisions.md) | Changing requirement level, safety, or failure behavior |
| Current defects, gaps, proposals and superseded tasks | [component open items](../../score/message_passing/maintenance/open-items.md) | Selecting or scoping work |
| Repository dependency paths, commands, and observed tool limitations | [repository conventions](seooc-maintenance.md) | Running or interpreting this repository's tooling |
| General SEooC update reasoning and validation semantics | [tooling reference](../../.github/skills/rules-score-update/references/tooling.md) | Tracing or validating an incremental update |
| Orchestration and a small evidence record | [proposed skill](../../.github/skills/rules-score-update/SKILL.proposed.md), [record template](../../.github/skills/rules-score-update/references/change-record.md) | Reviewing/adopting the new workflow |

The draft is deliberately named `SKILL.proposed.md`; the installed `SKILL.md` is
unchanged. No duplicate discoverable skill or new Codex-only metadata was added.
Component facts are outside the generic skill. Tool-version workarounds live in the
repository note, not as universal SEooC rules. This review is an audit artifact;
future runs need not load it or the old transcripts by default.

## Access and evidence limits

Accessible: both supplied files, all **45 research Markdown files** (33,754 words),
both copied memory files (5,957 words), eight change directories, seven evidence
bundles, local companion skills, current dependability artifacts, implementation,
tests, BUILD/module configuration, and the named LoLa consumer. The original working
tree was clean. All research files are already tracked at the revision above.

Unavailable here: the original editor's private `/memories/repo/` store and full chat
transcripts beyond the supplied copies; the referenced separate `copybara-export`
workspace; the historical run outputs beyond prose in evidence bundles; the resolved
`@score_tooling` source/cache. Neither `bazel` nor `trlc` is on PATH, and the Bazel
symlinks point to a missing cache. The pinned tooling is 2.2.2 in source configuration,
but that does not establish the binary used for every historical experiment. The
local vendored requirement schema is a different, incomplete substitute.

The linked tooling `latest/user_guide/general.html` page and attempted pinned raw
schema URL were not retrievable through the web tool. The public tooling repository
was accessible, but its current default branch is not evidence for the pinned
validator. Consequently, schema/parser claims below are attributed to source
configuration or historical reports, not a new execution. No QNX runtime evidence,
certification assessment, or complete correctness proof is claimed.

Current [VS Code skill documentation](https://code.visualstudio.com/docs/agent-customization/agent-skills)
supports repository skills under `.github/skills`, explicit relative references,
and on-demand loading of supporting files. The draft keeps that layout and the
supported `argument-hint` field. This review does not test the user's original
VS Code/model combination or attribute failures to a model version.

## Review findings

### 1. The result still has substantive discrepancies

These were checked against current sources, not merely copied from the backlog:

| Finding | Current evidence | Consequence |
|---|---|---|
| Missed re-pin and wrong semantic parent | [component requirements](../../score/message_passing/dependability/requirements/component_requirements.trlc), line 112: `IClientConnectionSendWithCallbackAPI → AsynchronousUnidirectionalCommunication@2`; [feature requirements](../../score/message_passing/dependability/requirements/feature_requirements.trlc), lines 86–90: current @3 explicitly describes one-way `Send` | Fixing only the number would leave request/reply under a one-way parent. The stopped-state child at line 284 already uses @3 but has the same semantic issue. |
| Corrected requirement meaning did not propagate to diagrams | [happy-path sequence](../../score/message_passing/dependability/software_architectural_design/server_client_sequence.puml), lines 91/125, says immediate non-blocking return; [client implementation](../../score/message_passing/client_connection.cpp), lines 118–135/253–269, can call transport inline | Impact closure must include behavioral prose and diagrams, not just identifier references. |
| QNX disconnect claim is not implemented | [QNX server](../../score/message_passing/qnx_dispatch/qnx_dispatch_server.cpp), lines 139–142, is a no-op TODO; sequence line 150 and [IPC-unavailable FTA](../../score/message_passing/dependability/safety_analysis/fta_ipc_channel_unavailable.puml), line 30, assume disconnect behavior | A diagram without validator findings can still describe nonexistent behavior. |
| Safety control overclaims implementation | [control measures](../../score/message_passing/dependability/safety_analysis/control_measures.trlc), `OsIpcFaultHandling`, promises checking every OS call; QNX server lines 93/114/134/255 ignore results | Re-derive controls from actual failure points before claiming mitigation. |

A lexical audit of all seven component TRLC files found **78 records and 58 versioned
`MessagePassing` references: 57 matching, one stale, no missing target**. The one stale
pin is above. This includes the unwired external-requirements file and is not TRLC
verification. Repository-wide TRLC search found no additional versioned consumers.
The [open-items inventory](../../score/message_passing/maintenance/open-items.md)
preserves these findings and the previously unresolved work; no product fix was made.

### 2. Build wiring and check exclusions changed what “passed” meant

The September 17 architecture cycle added sequence diagrams under `static` and
reported success. September 23 moved them to `dynamic`, and `private_api.puml` to
`internal_api`, exposing **54 findings**. Correct inclusion was a prerequisite to
meaningful checking, not a cosmetic BUILD correction.

The September 24 reconciliation reports **54 → 0 findings**, but public API labels
such as `(Create(...))` intentionally produce an empty extracted method name and skip
method checking. Its “None outstanding” conclusion needs that qualification.
Real PlantUML/Sphinx rendering also exposed problems that custom-parser checks did
not. Preserve both pipelines and the exact excluded checks in future evidence.

Historical sources: under `score/message_passing/research/changes/`, September 17
`architecture-completeness-for-fta-redo/evidence_bundle.md` (lines 14/41), September 23
`public-api-diagram-requirements-review/evidence_bundle.md` (line 11), and September 24
`diagram-reconciliation-54-findings/evidence_bundle.md` (lines 50–85). Use the complete
dated directory names in the status table below.

### 3. Human review supplied missing engineering judgment

The assumed-system rewrite took six drafting rounds to settle capability-level
requirements. The public-API cycle needed a follow-up `ClientInterface` feature after
new children had been placed under the broad OS-independence requirement. The
diagram rewrite initially invented a shared server-side engine abstraction; the
human corrected it. The FTA work was paused after header-derived reasoning and then
the wrong platform emphasis proved insufficient.

These are reasons to front-load requirement-level examples, parent sufficiency,
platform/safety boundaries, and source evidence. They do not establish that more
serial permission checkpoints improve routine edits. The bounded `ClientInterface`
change was handled in one session. The draft keeps decisions with the human when
intent is uncertain, while using authorization already given for mechanical work.

### 4. The knowledge store has conflicting authority and status

The old skill calls the baseline trusted. `dependability/README.md` says it wins over
research, while `research/safety_concept_notes.md` declares later human intent that
supersedes conflicting safety wording. A useful workflow must represent this
disagreement, not pick whichever file was read last.

Copied memory retains stale `SafeState` content, incorrect initial BUILD examples,
old machine paths, and contradictory schema summaries alongside later corrections.
The safety notes and paused next-step list still refer to removed `SafeState` and
already-resolved diagram tasks. Two cycles say “closed” while awaiting acceptance.
The extraction preserves intent and current unresolved work, with provenance and
one status per item; it does not promote the latest paragraph wholesale into truth.

### 5. The written procedure is heavier and less precise than the useful practice

Five files per cycle repeat trigger, scope, status, and evidence; root snapshots and
chat memory repeat them again. The active skill is about 2,447 words and asks for
layer-by-layer checkpoints plus `bazel test //...` and all gates. Actual successful
cycles used narrower explicit targets; external TRLC stayed unwired, AI checks were
omitted, and host tests did not run the QNX path.

The proposed skill retains upward/downward impact analysis but uses one current
change record, small durable documents, relevant mechanical skills, and validation
dimensions selected from the change. It clarifies same-parent pin-only updates versus
own-content/parent-identity changes and avoids inventing diagram version fields.
It also permits early validation instead of forbidding it until all edits finish.

## Historical cycle state to preserve

Source directory prefix: `score/message_passing/research/changes/` at the review revision.
The `next_steps.md` line 3 in each directory is the primary status evidence.

| Dated directory | State supported by the records |
|---|---|
| `2026-09-01-client-identity-and-userdata-docs` | Reported closed; scope decisions recorded, separate final acceptance not explicit |
| `2026-09-17-architecture-completeness-for-fta-redo` | Reported closed; scope decisions recorded, separate final acceptance not explicit |
| `2026-09-17-fta-redo-grounded-in-architecture` | Re-paused at impact analysis; explicitly deprioritized; no evidence bundle |
| `2026-09-19-assumed-system-requirements-rewrite` | Explicit acceptance recorded |
| `2026-09-22-feature-req-notify-split-and-crossplatform-qm` | Changes/evidence present; final acceptance pending |
| `2026-09-23-public-api-diagram-requirements-review` | Explicit acceptance recorded; deferred diagram work later handled separately |
| `2026-09-24-client-interface-feature-requirement` | Changes/evidence present; final acceptance pending despite “closed” wording |
| `2026-09-24-diagram-reconciliation-54-findings` | Explicit acceptance recorded, subject to the validation limits above |

Overall: five reported closed (three explicitly accepted), two awaiting acceptance,
one paused. Do not infer acceptance or resume priority from the existence of an
evidence bundle. The safety redo needs current impact analysis before resumption.

## Removal and adoption proposal

The extraction is ready for review; deletion and activation were not performed.
The active skill still mandates `research/`, so deletion alone would recreate it on
the next bootstrap. Adopt the new paths and instructions in the same change:

1. Keep the source revision above reachable in project Git history, or retain an
   archive if a shallow/exported checkout cannot access it. For example:
   `git show b8bbca3218f171d07e6b7577e4005bea26aa1479:score/message_passing/research/safety_concept_notes.md`.
   Raw protocol detail is intentionally left in that history, not duplicated here.
2. Adopt `SKILL.proposed.md` as `SKILL.md`, remove its review-draft paragraph, and keep
   the two supporting references. Update conflicting companion guidance at adoption:
   `score-requirements` version wording/source paths, `score-safety-analysis` advice
   about ignoring parser errors, and tiered-FTA numeric “Core Principle 3”/research
   pointers. Keep `rules-score`'s fresh-lifecycle convention separate.
3. Replace the old bootstrap prompt with a short routing prompt, for example:

   > Use rules-score-update on score/message_passing for the user's concrete request.
   > Start with maintenance/README.md and docs/engineering/seooc-maintenance.md;
   > read decisions/open-items only as relevant. Verify affected current sources.
   > Preserve the paused safety work and recorded scope decisions. If no change is
   > specified, provide orientation and request the trigger; do not create research/.

4. Update `score/message_passing/dependability/README.md` to point process material at
   `maintenance/` and acknowledge unreconciled decisions instead of asserting blanket
   precedence. Remove the `research/backlog.md` process link embedded in
   `server_client_internal_fault_sequence.puml`; keep the technical scope distinction.
   Search all remaining active instructions and artifacts for `research/`, old
   `/memories/` paths, and numbered principles. Historical citations may remain as
   Git object paths rather than working-tree dependencies.
5. Review this extraction, especially the unresolved FTA dispositions and recorded
   acceptance gaps. Then remove `research/` in the adoption change. Keep `chat_memory/`
   as historical input until separately retired; no new workflow should require it.

Migration map: baseline snapshot/references → component index; safety concept and
accepted engineering choices → decisions; backlog/nice-to-haves/remaining questions
→ open items and decisions; cycle status/evidence provenance → this review; repo and
parser facts from memory → repository conventions; general method → skill references;
routine work logs and superseded drafts → Git history. The standalone memory/size
findings document already lives outside `research/` and is retained by reference.

## Evaluation limits and next trial

The proposed packaging and links are checked separately from behavior. A fresh-agent,
read-only trial exercises the draft against a current traceability review; its result
is recorded below. This cannot replace trying it in the original
VS Code environment. For that trial, use a bounded requirement change, an
architecture-only change with rendering, and a paused-safety resume request. Observe
whether the agent reads the correct sources, finds semantic dependents, preserves
scope/pauses, reports excluded checks, and avoids unnecessary checkpoints. No claim
of measured token savings or improved Sonnet performance is made from this review.

## Checks performed on this review package

- All relative Markdown links across the eight new files resolved at review time.
- The proposed skill has a 345-character description and valid VS Code header fields.
  The bundled Codex skill validator rejects the existing VS Code `argument-hint`
  extension; a temporary copy omitting only that field passed its remaining checks.
  The actual draft retains the field supported by VS Code. This is not a claim that
  the unmodified draft passed that narrower validator.
- Existing tracked files were unchanged; additions are limited to the review,
  knowledge extraction and skill proposal. No build/test execution or product fix
  was represented as completed.

## Independent read-only trial result

A fresh agent received only the draft, a request to review
`IClientConnectionSendWithCallbackAPI`, and current artifacts. Historical research,
copied memory, the extracted component notes and this review were excluded. It found
both the stale pin and the sibling's semantic misparenting, checked the implementation
and wiring, distinguished callback-based reply from guaranteed non-blocking transport,
and reported missing test traces and inaccessible tooling. It made no edits or
product-approval requests and did not create tracking files for a read-only review.

It proposed two bounded alternatives: reparent the two component records directly to
`RequestReplyInteractionCapability@1`, or introduce a dedicated callback request/reply
feature. The companion guidance permits the former, but the agent correctly qualified
that actual pinned-schema compatibility was not parser-verified. This exposed a useful
clarification: the draft now explicitly says not to invent an intermediate feature
solely to complete a hierarchy. The behavioral discrepancy remains a separate decision
rather than being silently resolved during a pin repair.

This is one successful local reasoning trial, not a comparative model benchmark or
end-to-end validation of the revised workflow in VS Code.
