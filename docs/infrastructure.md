# Infrastructure requirements

## First local test

The first automated run can live entirely on a Debian laptop while the machine is awake.

Required locally:

- Node.js or another coordinator runtime;
- Docker or native service processes;
- SQLite;
- a local Nostr relay;
- the static web client;
- a gamemaster worker;
- at least one model API key, local model endpoint, or manual copy/paste bridge.

It does not initially require a domain, public server, owned Blossom server, public nsite gateway, or GPU.

## Persistent public alpha

A small virtual private server (VPS) is an always-online rented Linux computer. It runs the game when the development laptop is asleep.

Recommended starting capacity for a low-traffic alpha that does not perform local model inference:

- 1 shared CPU;
- 2 GB RAM;
- approximately 40–50 GB SSD;
- Debian;
- Docker Compose;
- persistent backups.

The VPS runs:

- reverse proxy and TLS termination;
- game coordinator and API;
- gamemaster job worker;
- SQLite state ledger;
- one Nostr relay;
- optional Blossom server;
- scheduled backup and health-check jobs.

The actual language models normally run elsewhere through provider APIs. A cheap CPU VPS coordinates inference but is not suitable for running a large local model.

## Relay plan

The first public deployment needs one owned game relay. Publicly released events should also be copied to several independent archival relays.

An existing relay such as [nostr-rs-relay](https://github.com/scsibug/nostr-rs-relay) is sufficient for early public events. The coordinator retains private state and enforces sealing. A later custom relay can apply authenticated per-instance read policies directly.

## Blossom and gateway plan

The project does not need to own either service initially.

- Static nsite files may be uploaded to existing Blossom servers.
- Existing public nsite gateways may resolve and serve the signed manifest.
- The same VPS may later run a [Blossom server](https://github.com/hzrd149/blossom-server).
- A separate nsite gateway is useful for redundancy but is not required to begin.

Blossom stores interface assets. It does not execute the game or hold private world state.

## What runs the gamemaster

The gamemaster worker is code running on the laptop or VPS. It assembles the permitted rules and state, then calls an interchangeable language-model endpoint.

The language model proposes structured observations and state changes. The local world kernel validates them before anything becomes canon.

Possible inference sources include:

- hosted model APIs;
- an Ollama-compatible local endpoint;
- an Ollama cloud endpoint;
- manual transfer between model chats during early tests.

Consumer chat subscriptions do not automatically provide programmatic API usage. Automated operation requires separate API access or a local model.

## Minimum persistent bill of materials

- one small VPS;
- one domain or stable HTTPS endpoint for the game API;
- one owned Nostr relay on that VPS;
- one SQLite database and backup destination;
- one or more model API accounts or a local inference endpoint;
- public relays for replication;
- public Blossom storage and nsite gateways initially.

The most variable operating cost is model inference. Hosting, relay storage, and the static nsite can remain comparatively small.

