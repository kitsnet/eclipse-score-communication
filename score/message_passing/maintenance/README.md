# Message passing maintenance knowledge

Reviewed 2026-09-25 against commit `b8bbca3218f171d07e6b7577e4005bea26aa1479`.
This is a concise orientation index, not a requirement set or a safety approval.
It replaces the need to load the historical `research/` and copied chat memory for
ordinary changes. Those sources are still present during review of this extraction.

Read this page first; read [decisions](decisions.md) for safety/requirement intent,
[open items](open-items.md) for remaining work, and the
[repository conventions](../../../docs/engineering/seooc-maintenance.md) for tooling.
Re-read the actual affected artifacts before editing. Code describes current behavior;
TRLC describes the authored contract; a disagreement is an issue to resolve, not permission
to silently make either one agree with the other.

## Scope and source map

- Same-host, connection-oriented IPC with one public API and Unix-domain-socket and
  QNX-native-dispatch implementations. The intended safety qualification is QNX/ASIL B;
  the non-QNX implementation is QM. These are scope/requirement statements, not evidence
  that qualification has been achieved. See
  [assumed requirements](../dependability/assumed_system/assumed_system_requirements.trlc)
  and [feature requirements](../dependability/requirements/feature_requirements.trlc).
- [Dependable element](../dependability/BUILD): `dependable_element_message_passing`,
  `integrity_level = "B"`, `maturity = "development"`, Linux-compatible documentation
  target. A successful host build does not validate QNX execution.
- Public contracts: [client connection](../i_client_connection.h),
  [client factory](../i_client_factory.h), [server](../i_server.h),
  [server factory](../i_server_factory.h), [server connection](../i_server_connection.h),
  [handler](../i_connection_handler.h), [protocol configuration](../service_protocol_config.h).
  Concrete platform aliases also live in [engine](../engine.h),
  [client_factory](../client_factory.h), and [server_factory](../server_factory.h).
- [Architecture](../dependability/software_architectural_design/client-server.md),
  [diagram wiring](../dependability/software_architectural_design/BUILD),
  [component requirements](../dependability/requirements/component_requirements.trlc),
  [safety analysis](../dependability/safety_analysis/BUILD), and
  [implementation/test wiring](../BUILD) remain the working sources. Do not duplicate
  their complete record or method inventories into these notes.
- [LoLa messaging](../../mw/com/impl/bindings/lola/messaging/) is one concrete consumer.
  Check its usage when relevant; keep the public contract consumer-independent and do
  not assume this is the only consumer.

## Architecture facts that prevented incorrect edits

`ClientConnection` is shared across platforms through `ISharedResourceEngine`.
The backend servers and their nested server connections use their concrete engine
types; there is no equivalent shared server implementation through that interface.
The `server_connection` Bazel unit represents common headers, not a third concrete
server implementation. Check [client_connection.h](../client_connection.h),
[Unix-domain server](../unix_domain/unix_domain_server.h),
[QNX server](../qnx_dispatch/qnx_dispatch_server.h), and [BUILD](../BUILD).

`score::os` wrappers are the external OS abstraction. `ISharedResourceEngine` is an
internal client-side transport abstraction; they must not be modeled as the same
interface. The OS diagram currently selects the QNX engine's `OsResources` wrappers;
thread/synchronization primitives were deliberately left out for brevity.

`Send` is one-way, but may call the transport inline and block. It queues when
`truly_async`, or when `fully_ordered` and a reply is pending. Queue capacity alone
does not select this behavior. OS acceptance, peer receipt, and handler completion
are different guarantees; Linux buffering must not inherit QNX-specific completion
claims. `Notify` has a separate asynchronous feature. `SendWithCallback` is
request/reply despite its callback-based return path; current parent traceability
still needs correction. See [Send implementation](../client_connection.cpp) and
[open items](open-items.md), MP-01/MP-02.

`UserData` has non-owning/value alternatives (`void*`, `std::uintptr_t`) and an owned handler alternative
(`score::cpp::pmr::unique_ptr<IConnectionHandler>`); distinguish ownership and callback
dispatch. Client identity is transport-dependent: QNX native dispatch and Linux UDS
obtain identity from the OS; UDS compiled on QNX returns zero placeholders. See
[server_types.h](../server_types.h) and
[Unix-domain implementation](../unix_domain/unix_domain_server.cpp).

## Current work state

No historical cycle is automatically resumed by loading these notes. The FTA redo is
explicitly paused/deprioritized, pending a new request. Its candidate reasoning is
not approved TRLC content. Two other cycles have implementation/evidence records but
no unambiguous final acceptance recorded: the September 22 notification/platform
requirements change and September 24 `ClientInterface` change. The
[review](../../../docs/engineering/rules-score-update-review.md) preserves the cycle
status inventory and source provenance.

The September 24 diagram reconciliation is recorded as accepted, but its zero-finding
result includes method-validation exclusions and leaves semantic discrepancies found
in this review. Load only the affected open items; do not reopen all old findings.

## Keeping this knowledge useful

Update these notes only when a durable decision, fact, or open-item status changes.
Keep one current status per issue. Store future change rationale and checks in a
small change record when needed; retain historical detail in Git rather than growing
another transcript. Keep process history out of normative TRLC/PlantUML content.
