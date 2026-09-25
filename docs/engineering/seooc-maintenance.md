# SEooC maintenance in this repository

This note records repository conventions and tooling limitations observed while reviewing
`message_passing` on 2026-09-25 at revision
`b8bbca3218f171d07e6b7577e4005bea26aa1479`. SEooC means **Safety Element out of
Context** here; the reviewed skills and artifacts do not constitute a cybersecurity workflow.
Component contracts and outstanding engineering decisions belong with the component.

## Dependency and source locations

[MODULE.bazel](../../MODULE.bazel) declares `score_tooling` **2.2.2**, without an override
for that module, and TRLC **3.0.1**. [.bazelversion](../../.bazelversion) selects Bazel
**8.7.0**. [MODULE.bazel.lock](../../MODULE.bazel.lock) contains the corresponding
`score_tooling/2.2.2` registry entries. These are source configuration facts, not proof
that this workstation has those tools installed.

The four artifact skills contain inherited links to `bazel/rules/rules_score/...`,
including schemas, examples and validator specifications. Those paths belong to
`@score_tooling`; they are absent from this repository. Resolve the actual external
repository before relying on those links. Once Bazel is available, inspect
`bazel mod show_repo score_tooling` and its resolved source directory; confirm the
version and any local overrides. Do not substitute an arbitrary tooling checkout.

The [local requirement model](../../third_party/score_requirement_model/score_requirements_model.rsl)
is a separate, smaller schema: it lacks the safety-analysis types used by this component.
The component BUILD files load rules from `@score_tooling`; their default schema must
be checked there. In particular, do not validate the safety records against the local
copy or promote a copied memory's enum inventory into a schema specification.

At review time, neither `bazel` nor `trlc` was on PATH. The repository's Bazel symlinks
pointed into a nonexistent `/var/cache/bazel/...`; `~/.cache/bazel` and a sibling tooling
checkout were also absent. No new build or parser result was obtained in this review.

## Repository validation entry points

Run from the repository root after restoring the declared build environment. These
commands correspond to current BUILD declarations and historical successful runs;
discover generated targets with `bazel query` before depending on generated names.

```sh
bazel query '//score/message_passing/dependability/...'
bazel test //score/message_passing/dependability/requirements/...
bazel build //score/message_passing/dependability/software_architectural_design:message_passing_architectural_design
bazel build //score/message_passing/dependability:dependable_element_message_passing
bazel test //score/message_passing:unit_tests
```

Check the result of each relevant validator and the documentation rendering logs.
Both the architecture and dependable-element targets currently use
`maturity = "development"`; historical successful commands included validator warnings
before reconciliation. A successful process exit alone cannot establish a clean report.

The [dependability BUILD](../../score/message_passing/dependability/BUILD) uses `manual`
tags. Explicitly request the relevant targets; do not describe `bazel test //...` as
proof that every manual validation ran. The only declared requirements AI target is
`//score/message_passing/dependability/requirements:feature_requirements_ai_check`, also
manual. Run it separately when in scope and its configured service is available.

There is currently no `test_case_coverage_lock` attribute in this component's BUILD
files and no coverage lock file. Do not invoke an assumed `.update` target or report a
coverage-drift check as passed. [score-testing](../../.github/skills/score-testing/SKILL.md)
describes a workflow that first requires that wiring.

The [unit-test suite](../../score/message_passing/BUILD) excludes `qnx_dispatch_test`.
The QNX unit declaration also has empty implementation/test lists in the dependability
graph. Host tests and the Linux-compatible dependable-element build therefore do not
establish QNX runtime coverage. The [test macro](../../quality/unit_testing/unit_testing.bzl)
creates platform-specific targets and a public selector; verify the QNX configuration
and execution prerequisites in [.bazelrc](../../.bazelrc) before requesting QNX evidence.

Do not follow the older safety skill's instruction to ignore tooling-RSL union syntax
errors as a passing validation. Such a mismatch leaves the affected validation blocked
until a compatible parser and the resolved schema can run together.

## Recorded parser limitations and exclusions

The September 24 reconciliation records the following workarounds. The inspected
revision pins tooling 2.2.2, but this review could not reproduce the historical binary
or establish the exact affected version range. Recheck after a tooling upgrade.

- The custom `puml_cli` pipeline rejected state diagrams, block `skinparam` syntax,
  lost-message arrows and narrative `...` lines. Use the accepted syntax demonstrated
  by existing diagrams; rendering successfully in PlantUML is insufficient.
- Mixing a nested single-line note inside an `alt`/`loop` with a separate multiline
  note triggered a false “unterminated sequence group” resolver error. Existing
  sequence diagrams use colon-style notes on one physical line, with literal `\n`
  for visual line breaks.
- Component-diagram interfaces rejected method bodies. Method declarations belong
  in the class-grammar public/private API diagrams.
- Calls involving `ExternalEndpoint` still underwent internal-API method checking.
  Existing labels such as `(Create(...))` deliberately make method extraction empty
  and **skip that check**. Preserve a manual comparison with public headers/API
  diagrams for these calls; zero findings does not prove their method coverage.
- Real PlantUML, used by Sphinx separately from `puml_cli`, required the first message
  after `create X` to target `X`. Check both validation and rendered documentation.

## Provenance and portable memory

The copied [documentation memory](../../score/message_passing/chat_memory/score-documentation-conventions.md)
and [architecture memory](../../score/message_passing/chat_memory/eclipse-score-communication-architecture.md)
preserve discoveries but mix old summaries with later corrections. Their original
`/memories/repo/` paths and other-workstation paths are not portable dependencies.
Their older schema examples, architecture assumptions and resolved findings should
not be copied unchanged into new instructions.

Historical reports remain retrievable without retaining `research/` in the working
tree: use `git show <revision>:<path>` with the revision above and these paths:

- `score/message_passing/research/changes/2026-09-24-diagram-reconciliation-54-findings/evidence_bundle.md`
  — reported 54→0 findings, parser workarounds, component build and 6/6 host tests.
- `score/message_passing/research/changes/2026-09-24-client-interface-feature-requirement/evidence_bundle.md`
  — reported 2/2 requirements tests and component build; AI check explicitly omitted.

These are historical reports, not new verification results. Keep portable decisions,
unresolved work and tooling exceptions in maintained repository documents, with links
to actual artifacts rather than requiring access to an editor's private memory.

## Other copied-memory material

The architecture memory's service-discovery-daemon extension ideas are proposals, not
an adopted architecture. Its integration-test tips about combining daemon/application
packages and avoiding QNX `WrappedProcess` teardown via `target.execute_async` are
historical recollections. The current
[bigdata BUILD](../../score/mw/com/test/bigdata/integration_test/BUILD),
[async API test](../../score/mw/com/test/basic_rust_api/consumer_async_apis/integration_test/com_api_async_api_test.py),
and [read-only-slot test](../../score/mw/com/test/data_slots_read_only/integration_test/test_data_slots_read_only.py)
do not contain the described daemon setup. Retain these as investigation leads for a
reproduction on the applicable branch/environment, not as general teardown guidance.
The memory's `message_passing/test/integration_test/stress_test.py` path is absent in
this checkout. These differences are another reason to validate copied memory against
current sources before using it.
