# Identity and Capability Vault

## Purpose

The Identity and Capability Vault is a shared security module for the game and, eventually, the wider chao_0x system. It joins four concerns that applications otherwise implement badly and repeatedly:

- human and agent identity;
- protected credentials and account connections;
- narrow permission grants;
- signed, inspectable use records.

Nostr signing authorizes access to the vault. It does not expose a user's Nostr private key, and the Nostr key is not reused as the encryption key for every stored secret.

## Core invariant

> Secrets are capabilities an agent may exercise, not values an agent may read.

An application, model prompt, or autonomous worker never receives a raw provider key, refresh token, relay credential, storage password, or Nostr private key. It receives a temporary capability grant. The vault's connection broker performs the approved operation and returns only the permitted result.

For example, an agent receives permission equivalent to:

```json
{
  "capability": "model.generate",
  "provider_connection": "openai:bre-default",
  "subject": "instance:07-F",
  "run": "run:post-extinction-001",
  "expires_at": "2026-07-31T18:30:00Z",
  "budget": {
    "currency": "USD",
    "maximum": 0.75
  },
  "approval": "preauthorized"
}
```

It never receives `OPENAI_API_KEY=...`.

## Responsibilities

The module owns:

- connection and disconnection of external accounts;
- local, remote, and delegated signer adapters;
- encrypted storage of API keys and login tokens;
- issuance, attenuation, expiry, and revocation of capability grants;
- execution of secret-backed provider operations;
- approval prompts for sensitive or expensive actions;
- budget and rate enforcement;
- redaction of secrets from errors, prompts, telemetry, and logs;
- append-only audit events;
- export and recovery metadata that does not expose credentials.

It does not decide game rules, model behavior, or whether an observation becomes canon.

## Identity adapters

The first identity family is Nostr:

- NIP-07 browser signers for desktop web clients;
- NIP-46 remote signers for cross-device and OAuth-like connection flows;
- NIP-55 Android signer applications;
- optional local key storage only inside an explicitly selected local vault.

The module may later support passkeys, email magic links, and conventional OAuth identities. These are additional ways to authenticate a person; they do not replace the user's Nostr identity when a signed public identity is required.

Authentication proves who is requesting an operation. Authorization determines what that identity may do.

## Vault modes

### Local vault

Secrets are encrypted on the user's device and secret-backed requests execute there. This minimizes server trust, but the device must be reachable whenever an autonomous agent needs a capability.

### Remote vault

Encrypted secrets live behind the game service and may be used while the user is offline. The server-side broker can unwrap only the credential needed for the approved operation. This enables persistent autonomous runs but creates a stronger operational security requirement.

### Delegated provider connection

Where a provider offers OAuth or another scoped authorization mechanism, the vault stores the delegated token instead of asking for a permanent raw API key.

The user chooses the mode per connection. A capability grant does not silently move a connection between modes.

## Secret storage

Each secret record receives a random data-encryption key. The ciphertext and wrapped key are stored separately from the identity and capability records when practical.

Required properties:

- encryption at rest and in transit;
- authenticated encryption with versioned metadata;
- rotation without changing public identity;
- no credentials in source control;
- no credentials in static-site assets, browser bundles, model context, or Nostr events;
- aggressive redaction of provider responses and stack traces;
- deletion and revocation that are distinguishable in the audit log;
- recovery procedures that never publish recovery material.

A Nostr signature may approve an unlock or capability request. Signing must not be treated as deterministic key derivation for vault encryption.

## Capability grants

A grant is an unforgeable reference with a policy envelope. At minimum it identifies:

- issuer;
- authenticated owner;
- subject app, user, or agent;
- permitted operation;
- provider connection reference;
- world and run scope when applicable;
- creation and expiry times;
- maximum calls, tokens, storage, or money;
- whether each use needs interactive approval;
- revocation state;
- parent grant when the permission was attenuated.

Grants are narrow by default. An agent allowed to call one model for one run is not thereby allowed to list account data, publish to Nostr, access storage, or delegate that permission to another agent.

## Connection broker

The connection broker is the only component permitted to unwrap provider credentials. It:

1. receives a capability reference and structured operation request;
2. verifies identity, subject, scope, expiry, budget, and approval policy;
3. unwraps the required credential outside model-visible memory;
4. calls the provider adapter;
5. strips sensitive response material;
6. records cost and outcome;
7. returns the allowed result;
8. discards transient plaintext credential material.

Provider adapters expose operations such as `model.generate`, `archive.read`, `storage.write`, or `nostr.publish`. They do not expose generic secret-reading operations.

## Audit ledger

Every security-relevant transition creates an append-only event:

- identity connected or disconnected;
- secret added, rotated, or deleted;
- capability granted, used, denied, expired, or revoked;
- interactive approval accepted or rejected;
- provider operation completed or failed;
- budget threshold reached.

Audit events identify the secret by opaque connection reference. They never contain the secret. Private audit state remains in the ledger; selected proof events may be signed and published to Nostr.

## User interface

The vault should feel like a connection and permissions panel rather than a password database. It shows:

- connected identities and providers;
- where each connection executes: local, remote, or delegated;
- which apps and agents currently have access;
- operation, time, and budget limits;
- pending approvals;
- recent usage and cost;
- revoke, rotate, disconnect, and delete controls;
- a human-readable explanation before every expanded grant.

The live agent interface can display `07-F used model.generate` and the resulting cost while truthfully displaying `secret disclosed: no`.

## Game integration

The static or nsite client may request identities and capabilities, but it contains no permanent credentials. The game coordinator receives capability references, not keys. Model adapters call the broker. The gamemaster and AI instances see tool results, never authentication material.

The vault supports three initial participation modes:

1. anonymous observation with no capability;
2. authenticated participation using a Nostr or conventional login;
3. model-funded participation using a narrowly scoped house or user connection.

A house-funded grant can enforce a tiny per-run allowance. A user-funded grant can be restricted to one provider, world, instance, duration, and maximum spend.

## First vertical slice

The first implementation needs:

- NIP-07 identity connection;
- one optional fallback login method;
- one model-provider connection type;
- session-only secret storage before persistent remote storage;
- capability issuance for `model.generate`;
- per-run expiry and spending ceiling;
- brokered model requests;
- revoke control;
- visible audit feed;
- tests proving secrets never enter prompts, logs, client responses, or Nostr events.

Persistent remote credentials, multiple providers, NIP-46, NIP-55, passkeys, recovery, and cross-app chao_0x integration follow after the capability boundary is proven.

