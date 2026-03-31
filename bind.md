# bind & dna.toml
## Identity, Intent, and Convolution

---

## The Three Primitives

The system has three primitives. Everything else is composition.

**Domain** — identity context. Who you are and where you are. A domain is not an organization chart. It's the set of things that are mutually legible from a given position. `ucryo` is a domain. `gxi` is a domain. A person, a machine, and a project are all domain nodes. They exist because someone introduced them. They carry lineage.

**Role** — intent vector. What you are trying to do. A role is not a job title. It's a direction of action. `design` means you intend to create or modify geometry. `cam` means you intend to produce toolpaths. `accts` means you intend to work with financial records. Roles don't belong to people. People carry roles into contexts.

**Bind** — the convolution rule. Given a source (domain + role) and a destination (domain + resource), bind resolves whether the intent reaches the resource. If the composition resolves, the action happens. If it doesn't, the traversal stops. Denial isn't a decision. It's a failure to compose.

---

## The Key Syntax

Everything is expressed as a single key:

```
{domain}.{context}.{commit} / {resource}.{path}
```

Left of `/` — where the intent comes from. Identity, context, version.

Right of `/` — where the intent is going. The resource.

`/` separates source from destination. It is not a directory separator. Dots compose context on both sides. A directory is not a container. It is a context node that returns correlated data when resolved.

The same resource can be reachable through multiple context chains without duplication. `design.bracket` and `cam.bracket` may resolve to the same underlying data — the difference is the traversal, not the storage.

---

## Firewall Model

Every access rule is a firewall rule with four fields:

- **src** — source domain and role
- **dst** — destination domain and resource
- **content filter** — conditions on the traversal
- **action** — what falls out of a successful composition

There is no separate rule engine. Each field is a context that composes with the next. The traversal is the permission check. If the composition resolves, the action happens.

---

## Roles

Roles define intent, not hierarchy. They are flat — boundaries in the topology, not layers in a stack.

Current role vocabulary:

| Role | Intent |
|------|--------|
| public | No authentication. The website. |
| guest | Authenticated. Shell access, internet. No company data. |
| user | Employee baseline. Company-wide data. |
| design | Create and modify geometry. |
| cam | Produce toolpaths and gcode from geometry. |
| accts | Financial records and bookkeeping. |
| install | Field installation, site documentation. |
| delegate | Assign roles to others. |

Roles compose. A person carrying `design + cam` can operate on both sides of that boundary. The composition is the capability set.

---

## Lock States as Folder Structure

Resources move through lock states. The folder structure is the state machine.

- `/design/` — **unlocked**. Active work. Design has read/write.
- `/cam/` — **soft lock**. Design can still write (override). Cam reads design files via SolidWorks references, writes toolpaths and setups in this space.
- `/nc/` — **hard lock**. Frozen record of what was actually run. Read-only for everyone. History accumulates here.

Files don't move between folders. A cam assembly in `/cam/` references parts in `/design/`. The reference is the soft lock — it makes downstream impact visible without preventing upstream changes.

A cam user with design capability can modify source geometry to add manufacturing features. The role composition gives access to both sides of the boundary.

---

## Capabilities

Capabilities are not permissions. They are boundary definitions — distinctions that describe where a projection becomes discontinuous.

| Capability | Boundary |
|------------|----------|
| transport | Can bits move at all. Wifi, ethernet, SATA, USB, COM. |
| dns | Can things be found by name. |
| token | Does the system know who you are. |
| trust / acs | Can you see through the encryption. Applies to network and storage equally. |
| shell | Can you interact with the system. |
| smb | Can you access files over the network. |
| pam | Authentication interface. |
| session | Active shell instance. |

These are flat. They don't have hierarchy. They intersect differently for different identities. Topology is the study of what remains unknowable regardless of how you rearrange what you do know. The capabilities define where knowledge stops.

---

## Domain Convolution

Both source and destination have domains. Bind convolves them.

```
paul.ucryo.design / gxi.cam.bracket
```

- Source domain: `ucryo`, identity: `paul`, role: `design`
- Destination domain: `gxi`, resource: `cam.bracket`

Two domain contexts meeting at `/`. Bind resolves whether the source domain's intent can project onto the destination domain's resource space.

Domain is not a category in the file. It is a property of both sides. The roles have domains. The resources have domains. The convolution is where they meet.

---

## RPM Model for Dependencies

Capabilities follow the RPM provides/requires pattern:

- **Provides** — what a node offers. A service provides `file-access`. A machine provides `name-resolution`.
- **Requires** — what a node needs to function. A user requires `shell`. Cam requires `design` files to exist.
- **Conflicts** — what can't coexist.

Dependencies are expressed in terms of capabilities, not implementations. A user requires `file-access`, not `samba`. The resolver determines which service satisfies the requirement.

---

## dna.toml

The topology file. Describes what exists, what boundaries matter, and what rules govern traversal. It is not a database. It is a type system expressed as data — keys encode their type in their shape.

```toml
[ucryo/]

[.domain]
# identity contexts — who and where

[.role]
# intent vectors — what you're trying to do

[.capability]
# boundary vocabulary — where knowledge stops

[.bind]
# convolution rules — src/dst = content.filter -> context
```

The file is both the plan and the ledger. Git tracks changes. The commit history is the audit trail. Automation reads the file and translates it to Linux primitives — groups, ACLs, smb.conf, PAM rules. Today, the router is you. Tomorrow, the router is `bind`.

---

## What bind Does

`bind` lives at the `/`. It evaluates the convolution of source and destination contexts. It is a firewall, a PAM stack, an ACL resolver, and a namespace constructor — all the same operation expressed through the key syntax.

The implementation maps to existing Linux infrastructure. Nothing is replaced. The filesystem, PAM, groups, permissions — they already do context composition. They just aren't legible about it. `dna.toml` makes the structure explicit. `bind` translates intent to mechanism.

---

## Status

This is a working architecture at seed stage. The first implementation is:

1. A `dna.toml` file describing the real system — machines, users, roles, capabilities, rules
2. Manual execution of the rules — you are the router
3. Git tracking changes to the topology
4. Shell scripts generating Linux configs from the topology when the manual process gets tedious

The immediate goal: replace a Windows Server 2012 R2 file share with a Fedora box running Samba, managed by a legible topology, in a shop with four users and five machines.

The long-term goal: a system where infrastructure is legible, trust is traceable, and the route is bolted well enough that you don't have to evaluate every piece of rock yourself.
