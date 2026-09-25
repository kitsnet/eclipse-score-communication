# Message passing open items

Reviewed 2026-09-25 at `b8bbca3218f171d07e6b7577e4005bea26aa1479`.
This is an issue inventory, not authorization or a priority decision. “Observed”
means source inspection; no fresh Bazel/TRLC/QNX execution was possible on this host.
Keep identifiers stable and replace status when resolved.

## Newly identified while reviewing the result

| ID | Evidence and discrepancy | Next bounded investigation |
|---|---|---|
| MP-01 | [Component requirements](../dependability/requirements/component_requirements.trlc), `IClientConnectionSendWithCallbackAPI`, pins `AsynchronousUnidirectionalCommunication@2`; the [feature](../dependability/requirements/feature_requirements.trlc) is @3 and now explicitly one-way `Send`. `ClientConnectionSendWithCallbackFailsWhenStopped` pins @3 but retains the same questionable parent. | Decide the appropriate asynchronous request/reply parent before fixing pins. Search the full cascade, including Reply-related requirements; a numeric re-pin alone does not resolve the semantic mismatch. |
| MP-02 | [Happy-path sequence](../dependability/software_architectural_design/server_client_sequence.puml) calls `Send` non-blocking/returning immediately and makes a similar claim for `SendWithCallback`. [Implementation](../client_connection.cpp), `Send` and `SendWithCallback`, can invoke transport inline. The revised one-way feature already admits blocking. | Reconcile the diagrams with the configuration/state-dependent transport paths on both platforms. |
| MP-03 | [QNX server](../qnx_dispatch/qnx_dispatch_server.cpp), `ServerConnection::RequestDisconnect`, is an empty TODO. The happy-path diagram says it triggers disconnect and [IPC-unavailable FTA](../dependability/safety_analysis/fta_ipc_channel_unavailable.puml) includes an event treating it as dropping a live channel. | Decide intended platform behavior and trace affected contract, diagrams, FTA, implementation, and tests. |
| MP-04 | [Control measures](../dependability/safety_analysis/control_measures.trlc), `OsIpcFaultHandling`, promises checking every OS return. The QNX server explicitly ignores `MsgDeliverEvent` and `MsgReplyv` results. | Establish per-call error handling and safety effect. Do not merely weaken the control measure to match code. |

## Unresolved findings recovered from the earlier work

| ID | Current state and evidence | What must be preserved |
|---|---|---|
| MP-05 | [AoUs](../dependability/assumed_system/aous.trlc) contain only `ExampleAoU`; four [controls](../dependability/safety_analysis/control_measures.trlc) address `MessageNotDeliveredCorrectly`, while eight failure modes/trees exist. | FTA redo remains paused. Preserve [safety intent](decisions.md), investigate actual code paths and potential overlap, and author justified controls/AoUs; do not mechanically generate seven sets of controls. |
| MP-06 | Server `pre_alloc_connections` and `max_queued_sends` are declared but not consumed by backend implementations, contrary to `ServerPreallocatesConnectionObjects` / `ServerRingBufferQueueSizeConfigurable`. | Decide implementation versus contract correction before deriving a memory model. Full prior evidence remains in [memory/size findings](../dependability/software_architectural_design/memory_and_size_limits_findings.md). |
| MP-07 | Identifier validation differs: UDS can truncate to its socket-path capacity; QNX rejects empty/over-limit identifiers (limit 256). | Define cross-backend identity/length behavior. SSO thresholds from past notes are implementation-specific tuning observations, not portable API limits. |
| MP-08 | No agreed per-connection/configuration memory model or allocator-containment test; see the same memory findings document. | Measure after MP-06 decisions. Shared receive buffers are per engine, not necessarily per connection. Preserve the proposed counting-resource/poisoned-default-resource test idea as an unimplemented candidate. |
| MP-09 | [External component requirements](../dependability/requirements/external_component_requirements.trlc) are omitted from [requirements BUILD](../dependability/requirements/BUILD) and element requirements. | Decide ownership/allocation and wire the file before claiming automated verification. Linux transport being QM is already decided, not an open question. |
| MP-10 | [Client state diagram](../dependability/software_architectural_design/client_connection_activity_diagram.puml) is not listed in [architecture BUILD](../dependability/software_architectural_design/BUILD); past parser attempt rejected its state syntax. | Check the pinned grammar and convert/register it if needed. Do not claim it was validated with the other diagrams. |
| MP-11 | [Unit design](../dependability/software_unit_design/BUILD) has no authored unit-design content. | Scope actual unit-design work when requested; its existence as a directory does not establish coverage. |
| MP-12 | Candidate integration obligations were recorded, not formalized: destruction outside `Stopped`, `sync_first_connect` inside a callback, server-connection use after disconnect callback, missing `Reply`, invalid handler/user-data contracts, and UDS-on-QNX zero identity. | Evaluate each against code and feasible internal mitigation before adding an AoU. Partial mitigations may coexist with obligations. |
| MP-13 | Architecture checks exclude some public-method labels via outer-parentheses workaround; both component declarations in [dependability BUILD](../dependability/BUILD) have `tests = []` while the element lists unit tests. | Establish exact traced coverage, coverage-lock ownership, generated checks and platform coverage. Do not equate passing host tests or zero diagram findings with complete requirement coverage. |

## Superseded historical tasks

The `ClientInterface` feature now exists and the seven intended children reference it;
only final cycle acceptance remains unconfirmed in the records. The 54 mechanical
diagram findings are recorded as resolved in a separate accepted cycle. The old
`SafeState` record is removed. Identity/UserData prose TODOs were addressed by the
September 1 cycle. Do not restore these as open implementation tasks from older memory.

## Optional ideas, not defects or approved work

- Shared-memory handle passing; watchdog arm/disarm notification callbacks; larger
  shared thread pools. Reassess requirements and architecture if selected.
- Model one-way `Send` as request/reply with an empty reply to obtain a portable
  handler-completion guarantee. This is a potential API redesign, explicitly not part
  of the September 22 wording correction.
- An informal system-level FTA, kept separate from the formal FMEA graph.

Historical provenance: `score/message_passing/research/backlog.md`,
`nice_to_haves.md`, `safety_concept_notes.md`, and the change directories listed in the
[review](../../../docs/engineering/rules-score-update-review.md), all at the commit above.
The memory/size findings document remains outside `research/` and is still available.
