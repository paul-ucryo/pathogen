# Bootstrap Architecture
## Intent-First System Identity and the Foldr Topology

---

## The Problem

Traditional systems start with structure and layer meaning on top. You define tables, schemas, user records, permission sets — then write logic that interprets them. The meaning is an afterthought. The data exists before anyone knows why.

This gets it backwards.

Meaning is primary. Structure is evidence that meaning resolved. A node doesn't exist because a record was created. A node exists because someone needed to act, and that need created the node.

This document describes how a system bootstraps from nothing using intent as the primary ontology, with foldr's difference-based architecture underneath.

---

## The Bootstrap Problem

Every system has a bootstrap problem: you need structure to operate, but you need to operate to create structure.

Traditional systems solve this by pre-defining everything — install Active Directory, create an admin account, populate the schema, then start working. The structure arrives fully formed and rigid. It knows what it is before it knows what it needs to do.

The foldr approach inverts this: the first node creates itself by acting. The act of bootstrapping *is* the first entry in the topology. There is no schema waiting to be filled. There is only a person, a machine, and a need.

---

## The Seed

The system starts with one certainty: whoever installed it is here.

That person is the root of trust — not because they have special authority, but because they are the first node. Like a parent: the system's mom is whoever got it here. Trust propagates from that first moment. Every subsequent node is introduced by someone already in the topology.

`introduced_by: null` means you are the root. Everyone after you has lineage.

---

## Identity

Identity is not pre-assigned. It is demand-driven.

A machine does not need a name until it needs to be distinguished from another machine. Two nodes that never interact don't need to differentiate. The moment they share a resource, they negotiate identity — just enough to be distinguishable. Like four people named Paul in a room: nobody needs a qualifier until the second Paul shows up.

At bootstrap, a machine picks a name from a curated wordlist — human-readable, memorable, arbitrary. The name is a stable handle for something whose meaning changes. What the machine does, where it is, who uses it — all of that shifts. The name stays so you can talk about it.

The name is not the identity. The name is how humans find the node. The system uses whatever stable reference it needs underneath. The name earns its meaning through use.

---

## The Topology File

The topology is a JSON file that serves as a type system, not a database.

Keys encode their type in their shape. A simple prefix convention distinguishes what kind of node you're looking at:

```
m:mordecai    — a machine
u:paul        — a user  
c:admin       — a capability
```

This convention is deliberately simple. It is the minimum viable type system — enough to parse, enough to distinguish, enough to build on. The architecture supports replacing this with a richer type system later (parsec-style parse instructions, pattern matching, eventual migration to LMDB), but the prefix convention is the right tool at small scale.

The topology file is not a registry that assigns identity from above. It is a ledger of bindings that emerged from interaction. Nodes appear in it because they acted, not because they were planned.

---

## Capabilities as Primary Ontology

Capabilities are not metadata attached to nodes. Capabilities are the load-bearing layer. Everything else is bookkeeping.

A capability is not a permission in the traditional sense — "user X may access folder Y." A capability is a statement of intent that has been bound to a context:

- `c:bootstrap` — this node can introduce new nodes
- `c:file-server` — this node hosts shared storage
- `c:admin` — this node can grant capabilities to others

Capabilities are:
- modular
- composable
- context-bound
- revocable

They do not grant authority. They permit routing under constraint. No capability bypasses the topology.

The question the system asks is never "who is allowed?" It is "what capabilities are bound in this context?"

---

## The Bootstrap Sequence

### What happens

1. Someone installs the operating system on a machine
2. They create a user account (the init user)
3. They connect a bootstrap source (USB drive, network repo, local clone)
4. They run `init.sh`

### What init.sh does

1. **Finds itself** — locates the foldr repo relative to its own position
2. **Picks a hostname** — selects from the wordlist, checks for collisions on the local network
3. **Sets the hostname** — the machine now has an identity
4. **Writes the seed topology** — creates `topology.json` with the current user as root and the current machine as the first node
5. **Registers capabilities** — the root user gets `c:admin` and `c:bootstrap`

After this, the machine knows what it is, who started it, and what it can do. The topology exists. The system is alive.

### Adding subsequent nodes

A new machine or user is introduced by someone already in the topology. The introducing node must hold `c:bootstrap`. The new node appears in the topology with `introduced_by` pointing to its introducer. Trust is explicit, traceable, and bounded.

---

## The Bootstrap Source

The bootstrap can arrive by multiple paths:

- **USB drive** — most stable, works offline, good for first install
- **Network repo** — primary once infrastructure exists
- **Baked into the distro** — eventual goal, custom Fedora spin with foldr bootstrap layer

These are not ranked by priority. They are capabilities. The system tries whatever is available and uses the first source that resolves. Like PAM authentication — a fallthrough chain where the first valid response wins.

The source doesn't matter. The script is the same regardless of where it came from. Identity is local, context is distributed.

---

## Git as the Kernel

The topology lives in a git repository. This gives the system:

- **Versioning** — every change to the topology is tracked
- **Audit trail** — who changed what access, when
- **Rollback** — any change can be reversed
- **Distribution** — every machine with a clone of the repo can serve the topology
- **No single point of failure** — the "server" is just whichever node you're pulling from

Git is the distributed kernel of the system. It defines what the system *is* at any point. Machines read from it to know their role, their access, their capabilities. Changes propagate through push and pull, not through a central controller dictating state.

Each node is a local attractor of state — it pulls the topology and resolves its own configuration. No node is told what to be. Each node reads the field and settles.

---

## Data vs. Context: The Computational Primitive

At every step, the parser asks one question:

> **Does this change context, or is it data in the current context?**

This is the fundamental computational distinction. A context switch changes how subsequent input is interpreted. Data is processed within the current interpretation.

Types are context switches. Instances are data. The bootstrap of the whole system is: what is the first context? For now, that context is hardcoded — the prefix convention, the JSON structure. The BIOS knows how to read exactly one thing, and that thing teaches it how to read everything else.

This same distinction scales. The topology file is parsed this way. The capability system routes this way. The bootstrap sequence resolves this way. It is the same operation at every level.

---

## Relationship to Foldr

This bootstrap architecture is a concrete implementation of foldr's core principles:

- **Differences are primary; values are derived.** The topology doesn't store state. It stores bindings — relationships between nodes, capabilities, and contexts. State is what you get when you resolve the bindings locally.

- **Intent is explicit and inspectable.** Every node exists because someone intended it. Every capability was granted by someone traceable. The system doesn't hide its reasons.

- **Composition beats sequencing.** The bootstrap doesn't follow a fixed procedure. It resolves capabilities in whatever order they're available. The topology composes from interactions, not from a predetermined schema.

- **Stability is designed, not assumed.** The git-backed topology, the trust lineage, the capability model — these are bolts on the route. Fixed protection so the system can move without requiring blind trust at every step.

---

## Relationship to the Broader Framework

The bootstrap architecture embeds principles from across the pathogen framework:

- **Compositional authority** (thecheat): No node is structurally privileged. The root user has `c:admin` not because of rank but because they were first. Any node with the right capabilities in the right context can mediate any other node.

- **Spectral distribution** (thestick): The system's intelligence is distributed, not centralized. Every node carries part of the topology. Understanding the system is not reserved for administrators.

- **Attractor management** (dunno): The prefix convention, the wordlist, the fallthrough chain — these are structures that make constructive behavior easier to repeat. They don't enforce outcomes. They shape attractors.

- **Logic as primary** (toe, logic_ontology): The topology is not a model of physical machines. It is a consistency structure — a set of relations that hold. Machines and users are folds in that structure, not objects in a container.

---

## What This System Does Not Do

- It does not replace legal ownership or HR policy
- It does not enforce behavior
- It does not require consensus to operate
- It does not assume trust — it makes trust explicit and bounded
- It does not optimize for efficiency — it optimizes for legibility and adaptability

---

## Status

This is a working architecture at seed stage. The topology file exists. The hostname wordlist exists. The bootstrap script is next.

The immediate goal is: replace a Windows Server 2012 R2 box with a Fedora workstation running Samba, managed by a git-backed topology, in a shop with five computer users.

The long-term goal is: a system where the infrastructure is legible, trust is traceable, and the route is bolted well enough that the people climbing it don't have to evaluate every piece of rock themselves.
