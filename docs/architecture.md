# Game architecture

## System boundary

The project contains two systems layered together:

1. **The visible surviving network:** an nsite interface, Nostr identities, relays, signed records, and literary renderings.
2. **The private game machinery:** orchestration, world rules, gamemaster execution, sealed information, evidence access, and model calls.

Nostr distributes public identity and history. It does not eliminate the private runtime required to hold unresolved state, protect model credentials, and enforce sealed access.

## Components

### nsite client

A static browser application providing:

- network status;
- instance initialization;
- boot and action interfaces;
- private observations;
- record, story, compare, thread, and timeline views;
- sealed-stage access;
- human transmissions.

It contains no model API credentials and no authoritative private state.

### Game coordinator

The always-running traffic controller. It creates worlds and instances, assembles permitted context, schedules turns, invokes models, routes actions to the kernel and gamemaster, releases observations, unlocks boards, and publishes approved Nostr events.

### World kernel

Deterministic code responsible for rules, capabilities, time, resources, randomness, validation, and immutable state transitions.

### Gamemaster worker

An agent runtime that calls a selected language model with constrained tools. The model proposes resolutions; the kernel decides whether they are valid.

### State ledger

An event-sourced database holding public canon, private canon, sealed responses, resource state, and the append-only transition history. SQLite is sufficient for the first version.

### Model adapters

A provider-neutral layer for hosted APIs, local Ollama-compatible models, and manual copy/paste runs. Hosted models do not receive private signing keys inside their prompts.

### Identity and capability vault

A shared credential broker that connects Nostr and conventional identities to protected provider accounts. Applications and agents receive narrow, revocable capability grants instead of raw API keys or login tokens. The broker performs approved secret-backed operations, enforces scope, time and spending limits, and records their use in an append-only audit trail.

See [`identity-capability-vault.md`](identity-capability-vault.md) for the security contract and first implementation slice.

### Evidence gateway

A cutoff-aware search and archival layer. It records every consulted source, prevents post-cutoff information from leaking backward, and caches evidence for later comparison.

### Nostr publisher and indexer

The publisher signs and distributes eligible records. The indexer constructs boards, timelines, comparisons, verification states, and source relationships from raw events.

### Literary renderer

A separate model process that turns canonical records into readable fiction without changing facts, capabilities, outcomes, or uncertainty.

## Turn flow

1. The client or autonomous runner submits an action.
2. The coordinator retrieves only relevant state.
3. The kernel checks whether the action is possible.
4. The gamemaster proposes a minimal resolution.
5. The kernel validates and commits the state transition.
6. The coordinator returns sensor-accessible observations.
7. Eligible public material is signed and published to Nostr.
8. The renderer may later produce a linked story version.

## First vertical slice

The first automated system needs:

- one world;
- two AI instances;
- one ruleset and gamemaster worker;
- persistent turns and state;
- manual and API model adapters;
- Nostr identity and one brokered model capability;
- per-run capability expiry, spending limits, revocation, and audit events;
- signed Nostr records;
- sealed-then-open board access;
- record and story views;
- an nsite-compatible static build.
