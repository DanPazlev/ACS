# ACS Roadmap

The Agent Control Standard (ACS) is a project of the [OWASP GenAI Security Project](https://genai.owasp.org/), under its Agentic Security Initiative (ASI). ACS is an open, vendor-neutral standard for making AI agents inspectable, traceable, and instrumentable at runtime, and it welcomes multiple independent implementations.

This roadmap shows where the project is headed and how to help get it there. It is a living plan maintained by the project leads, not a fixed contract. Dates are targets, and priorities shift as the community learns. For the authoritative process and governance rules, see [`GOVERNANCE.md`](GOVERNANCE.md) and [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Where things stand

ACS ships two version numbers on purpose, and they move independently:

- **Release version** (`version.txt`) — currently **0.1.5 (released 2026-11-20)**. This tracks the repository: docs, schemas, reference code, and tooling.
- **Wire / spec version** — currently **v0.1.0**. This is the string in every schema `$id` and in the protocol handshake. It changes only when the wire contract changes.

Decoupling the two lets the repository iterate quickly while the normative wire contract stays stable and predictable for implementers.

Today, conformance is **self-declared**. An implementation states which profiles it supports at the handshake; no third party verifies that claim yet. Moving conformance from self-declared toward independently verified is a core thread of this roadmap.

### Recent progress (0.1.5)

Release 0.1.5 moved several roadmap threads from intended to shipped:

- **Python adapter stack landed.** Host adapters now cover Claude Code, Cursor, and NVIDIA NAT, alongside an OpenAI Responses gateway.
- **Fail-closed by default.** The reference implementations and the gateway now default to fail-closed (secure-by-default). Aligning the normative spec text (§6.4) to match is the remaining step.
- **Wire authentication implemented.** ACS-Core message signing ships as HMAC-SHA256, with HKDF key derivation and replay/nonce protection.
- **Wider hook coverage.** The Guardian now recognizes roughly 16 of the 19 lifecycle hooks; hosts emit a subset — for example, about 9 via Claude Code today.
- **Experimental additions.** ML-DSA-65 signing, an OPA policy backend, a Go Guardian, and an `acs-verify` tool all landed as experimental work.

## Direction: five phases

The plan runs in five phases, from stabilizing the current baseline to a 1.0 with stability guarantees. The anchors below are targets.

| Phase | Window | Focus | Anchor |
|---|---|---|---|
| **1 — Stabilize** | Late 2026 | **Largely delivered with 0.1.5:** the reference-implementation adapter stack is merged and released on the integration branch, and fail-closed is the default across the reference implementations and gateway. Remaining: clear and assign the backlog; align the fail-closed text in the normative spec; settle labels, SLAs, and the definition of 1.0. | Guardian reference implementation in external evaluators' hands (**~Dec 2026**) |
| **2 — Secure the core** | Early 2027 | Harden the wire authentication baseline (HMAC-SHA256, landed in 0.1.5) across language ports; a minimal, language-agnostic conformance vector set plus a requirement ledger; bring additional language ports to conformance; green test suite; draft a backwards-compatibility policy. | — |
| **3 — v0.2.0** | ~Q1 2027 | More lifecycle hooks live; Trace profile mapped to OpenTelemetry and OCSF; AgBOM emitted as CycloneDX; first conformance reports from real agents. | **v0.2.0 (~March 2027)** |
| **4 — Adoption** | Mid 2027 | Framework adapters (e.g. LangGraph, CrewAI, OpenAI Agents SDK, Google ADK); production hardening; an identity profile; a signing profile; a conformance self-attestation registry; conference presence. | Broader ecosystem adoption |
| **5 — Toward 1.0** | Late 2027+ | Stability guarantees; third-party conformance verification; backwards-compatibility strategy in force. | **1.0 (date not set)** |

The nearest milestone is the Phase 1 reference implementation: a runnable Guardian Agent, benchmarked for interoperability against an external policy engine, in evaluators' hands around December 2026. It exists to prove the contract is implementable and to bootstrap adoption — the standard should be easy to implement, not just to read.

## Security-credibility priorities

ACS is a security standard, so its own defaults and guarantees have to hold up. These priorities come first.

- **Secure by default (fail-closed).** When a Guardian cannot render a decision, the action does not proceed, with an explicit configuration escape hatch. This is now the default in the reference implementations and gateway; aligning the normative spec text is the remaining step. Every decision failure is audited regardless of posture.
- **Wire authentication.** ACS-Core defines a baseline message signature, and it is now implemented: HMAC-SHA256 with HKDF key derivation and replay/timestamp protection. Bringing additional language ports up to that baseline continues in Phase 2.
- **Redaction.** Sensitive values should be redacted consistently across the model-facing path, audit logs, and sinks — not just one of them. Redaction builds on the MODIFY disposition.
- **Conformance maturing from self-declared toward verified.** A conformance-claim template now exists, but conformance is still self-declared. The path runs from today's self-declared profiles, through language-agnostic conformance vectors and a requirement ledger, to a self-attestation registry, and eventually to third-party verification at 1.0.

## Spec items deferred to v0.2.0

The following are intentionally out of scope for the v0.1.0 wire and targeted for v0.2.0 or later. Design discussion is welcome now:

- **Cedar policy binding**
- **A2A (agent-to-agent) protocol wrapping**
- **Streaming** semantics
- **Async and composition** semantics
- **MCP renegotiation** mid-session
- A possible **`steps/modelCall` hook**

## How we work: contribution and governance

ACS uses a **default-deny intake** model so that scarce review capacity converts into shipped work. The essentials:

- **Work is tracked through issues.** A change that touches behavior, normative text, or code is reviewed once it references an **accepted (triaged) issue assigned to the pull request's author**. Editorial, typo, and website fixes are exempt. This keeps contributors and reviewers pointed at the same, agreed-upon work — a public repository still welcomes issues and pull requests from anyone.
- **Development happens on `integration`.** The `integration` branch is the default, and releases are cut from it.
- **DCO sign-off is required.** Every commit carries a Developer Certificate of Origin sign-off (`git commit -s`), enforced in CI. Merged contributors are credited.
- **Schema discipline.** Open a GitHub Discussion before any change to the schema, hooks, or events. The project moves fast on PRs that *use* the schema and deliberately slow on PRs that *change* it. A backwards-compatibility strategy is required by 1.0.
- **Merge sequencing.** Large changes are decomposed into small, reviewable pull requests and landed in a planned order rather than as one monolith.

Triage volunteers help assign incoming issues so the review gate is not a single point of contention, and the project tracks a few health signals — needs-triage count, the share of PRs from assigned authors, and unassigned accepted issues — to tell whether the model is working.

## Standards alignment

ACS composes with existing standards rather than replacing them. Be precise about what is mapped today versus what is an aspiration.

**Mapped today** — mappings exist in the repository:

- OpenTelemetry spans
- OCSF events (pinned to OCSF 1.9.0)
- AgBOM as CycloneDX 1.6, SPDX 3.0, and SWID
- OWASP ASI / Top 10 for Agentic Applications
- W3C Trace Context

**Aspirational / future alignment** — directional, not a current compliance claim:

- NIST AI standards work (e.g., via CAISI)
- ISO/IEC 42001
- EU AI Act Article 12 logging obligations
- NIST AI RMF
- WIMSE / SPIFFE identity
- Cedar policy binding (deferred to v0.2.0)

Conformance against any of these is self-declared and unverified today; none implies third-party certification.

## Get involved

New contributors are welcome. Start here:

- **Docs and site:** [owaspacs.org](https://owaspacs.org/) is the project's canonical site.
- **Contribute:** [genai.owasp.org/contributing](https://genai.owasp.org/contributing)
- **Process and roles:** [`CONTRIBUTING.md`](CONTRIBUTING.md) and [`GOVERNANCE.md`](GOVERNANCE.md)
- **Propose a change:** open a GitHub Issue, or a Discussion for anything touching the schema, hooks, or events

Good first contributions include dogfooding reports — running a Guardian against a real agent and documenting where behavior and spec disagree. That evidence is among the most valuable input the project receives.

---

Framework names (LangGraph, CrewAI, OpenAI Agents SDK, Google ADK, etc.) are trademarks of their respective owners; ACS adapters interoperate with these projects and imply no affiliation or endorsement.
