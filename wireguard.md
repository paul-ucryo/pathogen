# Distributed Domain Storage System

## Core Concept

Wireguard for persistent storage. Identity is a key, ownership is a volume, access is a mount. The same model that makes Wireguard clean for networking applied uniformly to storage.

---

## Primitives

### LUKS Volumes as Domains
- Each domain is a LUKS volume. The key is the capability — if you hold the key you can mount it, if you can mount it you can access it.
- Ownership is not a policy, it is cryptographic. No ACL, no trust in software, just the math.
- Keys can be derived from multiple factors (TPM + measured boot + user key), binding a volume to specific hardware and software. Data physically cannot be decrypted outside the defined environment.

### Concern Taxonomy
Every domain has the same set of subvolumes representing information types:

```
/home/fs/<domain>/issue
/home/fs/<domain>/design
/home/fs/<domain>/catalog
/home/fs/<domain>/inventory
/home/fs/<domain>/budget
/home/fs/<domain>/order
/home/fs/<domain>/run
/home/fs/<domain>/acs
/home/fs/<domain>/gateway
/home/fs/<domain>/bus
/home/fs/<domain>/fs
```

Each concern is a separate COW subvolume. Access is granted at the concern level, not the domain level. A role may mount `catalog` and `inventory` without touching `budget`.

### Symlinks as Routing
- `/home/fs/` — raw LUKS volume mount points (the storage layer)
- `/home/<user>/` — symlinks into mounted volumes (the navigation layer)

The directory structure is a view, not the data. Ownership and meaning are decoupled. The store owns the bytes, the symlinks own the meaning. When a session ends, the symlinks are gone. The data remains where it lives.

---

## Routing

### CTXT and ROUTE
- `CTXT` — an ordered list of active contexts/roles (e.g. `[designer, machinist]`)
- `ROUTE` — the resolved routing table populated by CTXT, mapping concerns to volume paths in priority order

CTXT is configuration. ROUTE is the resolved state the system actually consults. Like a shell sources a config to set PATH — the config expresses intent, PATH is what runs.

A write defaults to whatever ROUTE resolves for the current CTXT. A specific CTXT can be specified per operation without changing the live environment — the operation forks the shell with the modified CTXT, runs, and exits. The live shell is unchanged.

### Fallback and Root
- If ROUTE cannot resolve a concern, data falls back to the root volume — an inbox for unclassified data.
- `root` — provenance, origin, owner. Where data lives before classification.
- `bind` — composition. Where concerns are assembled into documents and views for human consumption.

---

## Bind Layer

Bind is the document and interface layer. It composes data from multiple concerns into something meaningful for a specific audience or purpose:

- A statement of work pulls from `design`, `order`, `catalog`
- A shop traveler pulls from `design`, `run`, `inventory`
- A financial report pulls from `budget`, `order`, `inventory`

The accountant sees a document, not a concern taxonomy. The underlying structure is invisible until someone wants to see it.

Adoption is gradual. A Word doc is a valid bind artifact. An HTML file that queries live concern data is a more capable bind artifact. Both coexist. The structured version replaces the manual one incrementally as tooling matures, with no flag day required.

---

## Workflow Model

`run` is execution of a `design`. In a machine shop, run is manufacturing. In software, run is deployment. The concern taxonomy is domain-agnostic.

### Issue Tracking
- `run.issue` — deviation during execution (scrap, broken tool, missed feature)
- `design.issue` — flaw in the plan itself (wrong cutting strategy, bad program)

A program review before a job runs is a `design.issue` opened proactively. It sits open until reviewed. The feedback lives on the issue as a record of craft knowledge attached to real work.

### Continuous Improvement
Production metrics map directly to concern structure:
- **Cycle time** → `run` performance, speeds/feeds/cutting strategy
- **Tool life** → `run` parameters, setup rigidity, coolant
- **Scrap rate** → `design` and `run` attention, setup verification

A PIP is just a `design` attached to an `issue`. No separate HR system. The same issue/design/run loop applied to skill development. Improvement goals are expectation values — shop average, industry standard, job requirement, or personal target — compared against personal trend data. Anonymized aggregate comparison gives signal without exposing individuals.

Work allocation follows demonstrated capability naturally. Lower risk parts go to people still building their metrics. The data suggests allocation, the conversation makes it productive. The system grounds the conversation in shared observable facts rather than judgment.

---

## Distribution

### Local Instance Model
The system gives each user a key and a local instance of a domain volume rather than managing user contexts internally. Isolation is physical, not logical.

- Local instance: your working copy, mounted and fully accessible
- Remote instance: the authoritative store
- Delta audit: the remote reviews btrfs send/recv deltas before committing

You work freely in your local instance. The remote only sees what you send, and only commits what passes audit. Privilege escalation has no surface — your instance is yours, the boundary is the delta.

### Sync via btrfs send/recv
- COW snapshots provide a precise delta stream — only changes move, not whole volumes
- Deltas stay within LUKS ownership boundaries — changes to your data, not your key
- Offline systems are just systems with an old last-sync snapshot — no special handling, just a longer delta when they reconnect
- Conflict is visible as divergent snapshots, not silent overwrite — effectively CRDT at the kernel level

### Wireguard Integration
- Live access: Wireguard connection, data stays on trusted hardware, only routed remotely
- Offline sync: btrfs send/recv delta stream between LUKS instances
- Identity is uniform across both — the same key that identifies a network peer identifies a storage volume

Local and remote storage are the same concept. Network access and storage access are the same concept. Identity across both is the same primitive. The whole system is keys, routes, and volumes whether local or distributed.

---

## Security Model

- **Mounting is the exposure boundary** — you only expose what you mount
- **Symlinks are the capability** — you only navigate what has a link
- **Purpose-specific volumes** — for sensitive sharing, create a volume containing only what the target is allowed to see
- **Wireguard for high trust** — data stays on your hardware, only routed to the target, never at rest there
- **Hardware binding** — TPM + measured boot + user key means data cannot be decrypted outside the defined hardware/software environment

The shared system sees routing metadata — who has what, where things resolve to — without needing access to the content. The routing table is public. The data behind it is private. A new collaborator can understand the shape of the system before being granted any keys.

Trust is incremental: share the routing entry first, share the key when ready.

---

## Summary

The system is a dependency graph of LUKS volumes connected by symlinks, routed by CTXT/ROUTE environment variables, synced by btrfs send/recv, and networked by Wireguard. Each layer uses the same key primitive. Security is cryptographic at every boundary. Complexity matches actual coordination need — a solo user has one volume and no overhead, a large distributed team has many volumes and rich routing, the mechanism is identical throughout.
