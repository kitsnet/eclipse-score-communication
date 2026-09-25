# Message passing decisions and unresolved intent

Extracted 2026-09-25. Historical evidence is recoverable from commit
`b8bbca3218f171d07e6b7577e4005bea26aa1479` using the repository-relative paths below
with `git show <commit>:<path>`. This preserves provenance after `research/` removal.
These notes record decisions; they do not certify implementation or override the
current contract without a reviewed change.

## Requirement level and platform scope

Assumed-system requirements describe capabilities an integrator assesses from outside
the component: communication model, one-way and request/reply interaction, detecting
production errors and integration misconfiguration, platform support, and access
control. Method names, queues, and internal failure handling belong below that level.
Safety scope is represented separately by the QNX qualification requirement. This
convention was accepted after several rewrites on September 19 and is also stated in
[dependability/README.md](../dependability/README.md).

`SystemMessagingProtocol` and the old `Mitigation SafeState` were removed by that
rewrite. Do not resurrect an action item to rewrite `SafeState`. The surviving
lesson is to use precise behavior descriptions and avoid presenting “safe-silent”
as an established standards term without a source. No standards-compliance claim
is established by these notes.

QNX native dispatch is the safety-relevant implementation; Linux/non-QNX is QM.
UDS-on-QNX exists as code but its zero identity placeholders must not become an
authentication guarantee. Formalizing that restriction as an AoU remains open.

Sources: `score/message_passing/research/changes/2026-09-19-assumed-system-requirements-rewrite/`
and `2026-09-01-client-identity-and-userdata-docs/` under the same `changes/` directory;
current [assumed requirements](../dependability/assumed_system/assumed_system_requirements.trlc)
and [external requirements](../dependability/requirements/external_component_requirements.trlc).

## Public API allocation decisions

`ClientInterface` now provides the parent for seven client lifecycle/state requirements.
Factory creation stays under `OSIndependentAPI`; factory engine-sharing requirements
remain under `SingletonFreeImplementation`. The September 23 API review judged
`GetDefaultOsResources` an implementation helper without an independent requirement.
Do not turn every diagrammed helper into a new obligation. Keep QNX-specific engine
resource injection distinct from the shared client's `ISharedResourceEngine` injection.
Source: `score/message_passing/research/changes/2026-09-23-public-api-diagram-requirements-review/evidence_bundle.md`
and the September 24 `client-interface-feature-requirement` cycle at the same revision.

## Safety intent to preserve for the paused FTA redo

The September 17 human-provided safety notes explicitly challenged the correctness
of the existing safety artifacts. Their nine principles, including the September 24
addition, are preserved here as design intent awaiting reconciliation:

1. Report detected errors through the API's return/error channel when available.
2. Where no reporting path exists, some errors are absorbed; identify the actual
   behavior at each failure point instead of inventing a universal reporting promise.
3. Preserve ordering subject to the explicitly configured ordering rules.
4. For send failures that cannot be reported, the intended fallback may be silence
   on the affected channel until disconnect/reconnect; verify each path before
   expressing this as an implemented control measure.
5. Isolate a connection's failure from other connections.
6. Termination can be justified when isolation cannot be guaranteed, but the later
   clarification below limits any blanket fail-stop interpretation.
7. Mitigate same-process caller misuse where reasonable; impose AoUs for real
   integration obligations, not every imaginable misuse.
8. Contain IPC peer misuse and document the specific chosen response.
9. An AoU and a real partial mitigation can coexist. A timing obligation on the
   integrator does not erase the partial benefit of a queue/background dispatch.

The termination clarification is essential: the design is more willing to terminate
at startup when resources cannot be obtained. During operation, new-activity resource
failures should leave existing activities running where possible. Allocation from an
exhausted configured memory resource can still terminate. The precise boundaries must
be established per failure point. There is no agreed AoU requiring an external
supervisor to terminate on the library's behalf. Preallocation is intent, not proof
that all server preallocation requirements are implemented (MP-06).

Peer containment is deliberately unresolved: protocol errors can drop a connection;
an unhandled notification can be ignored; other paths assume callbacks exist. Do not
replace this with an invented uniform rule. Likewise, the notes' suggested controls
must be checked against the code's ignored OS errors (MP-04).

Source: `score/message_passing/research/safety_concept_notes.md`, including its
“Follow-up answers” and ninth principle. Its final status paragraph is stale about
`SafeState` and the number of principles; those statements were not carried forward.

## FTA method chosen for this component

The optional [tiered-FTA skill](../../../.github/skills/rules-score-safety-analysis-tiered-fta/SKILL.md)
was explicitly selected for this component. The system-level tree is an optional,
informal reasoning aid; it was not started and must not be wired into FMEA as though
its roots were `FailureMode` records. Formal micro-FTAs remain rooted in the
tooling's failure-mode graph.

The paused redo must derive candidate causes from the corresponding implementation,
not just rename existing events based on headers. Example: `ClientConnection::TryConnect`
retries `EAGAIN`/`ECONNREFUSED`/`ENOENT` with a capped retry delay, while `EACCES` and
other terminal errors produce distinguishable stop reasons. The existing
`ServerHealthCheck` and `ClientRetryPolicy` events should not be treated as proof of
two independent implemented mechanisms. A capped delay does not bound the number
of retries. `Restart()` is `void` and returns without restarting when not stopped;
do not preserve the old scratchpad claim that it returns `EINVAL`.

Resume only on an explicit new request; reconstruct the current requirement and
architecture baseline before continuing. The old next-step list's `SafeState`
rewrite and 54-diagram-findings task are superseded. Content reconciliation of all
eight failure modes remains open. Original-author clarification may still be needed
where intent cannot be recovered from code.

Sources: `score/message_passing/research/changes/2026-09-17-fta-redo-grounded-in-architecture/`,
`score/message_passing/research/references.md`, and
[client_connection.cpp](../client_connection.cpp).

## Unresolved FTA dispositions to revisit, not approved edits

The paused change request's thirteen questions include revised and erroneous premises.
Preserve their decision content without treating the old proposed answers as accepted:

| Decision | Current boundary for resumption |
|---|---|
| Timing supervision | Decide integrator timing obligation and real partial mitigation; no built-in watchdog guarantee was established. |
| Duplicate disconnect events | Consider one shared obligation only after resolving QNX's no-op `RequestDisconnect` (MP-03). |
| Server queue configuration event | The referenced configuration is unused; decide retirement together with the requirement/implementation discrepancy (MP-06). |
| Health check / retry | Re-derive the single actual retry/error-classification path; the original proposal for two controls was superseded. |
| Missing notification callback | Scope any obligation to a protocol that expects notifications; unhandled-notification behavior is not a universal callback policy. |
| Handler failure / missing reply | Distinguish one-way delivery from request/reply and decide whether the two reply obligations really coincide. |
| Call-flow / lifecycle | Separate internally checked misuse from real residual integrator obligations; inspect server lifecycle too. `Restart` does not return `EINVAL`. |
| Invalid connect-callback data | Check value/handler alternatives and actual failure behavior before merging the two candidate causes. |
| Absent required callbacks | Resolve the actual `score::cpp::callback` empty-invocation behavior in the selected dependency; graceful no-op was not established. |
| Notification queue exhaustion | Re-ground ASIL-B controls in the QNX pool/transport path. UDS uses a different send path; do not extrapolate one backend's containment to the other. |

Source: the paused cycle's `change_request.md` questions 1–13 and latest platform-scope
correction in `next_steps.md`. Their dispositions still need engineering review.
