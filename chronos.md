# The Shell as Timeline Projection

## The Primitive

Correlation is atemporal. The relational structure of a system — what exists, what connects to what, what capabilities are bound to what identities — has no inherent order. It simply is. All bindings coexist. There is no before or after in the structure itself.

The shell is what produces local time.

A shell serializes correlation into a causal chain. By traversing the relational structure in a specific order, it creates a timeline — a local sequence of before and after that didn't exist in the structure itself. This isn't a side effect of execution. It's what a shell fundamentally does. Outside the shell there is only correlation. Inside the shell correlation becomes causation because traversal order matters.

Parallel processes are not an exception to this. They are parallel serializations of the same relational structure — multiple causal chains running simultaneously, each seeded by a fork and terminated by a join. The parallelism lives in the relational structure. The causality is imposed by the shells traversing it. The join is where independent timelines re-correlate.

The shell doesn't run inside time. It produces time as a local phenomenon.

---

## The Shell as Projection Axis

The shell defines local. Not geographically, not topologically in the network sense — local is whatever is reachable from the current execution context along the current projection axis.

Environment variables are the materialized projection state. `$USER`, `$HOME`, `$PATH` are not convenience variables. They are what the shell knows about its current position in the relational structure. The environment is the rendered local view. Sourcing a file is resolving a binding into the current context — extending the projection axis to include more of the relational structure.

This means a distributed resource becomes local the moment the shell's projection axis includes it. A user sitting at a library computer with access to a remote capability set is not accessing something remote — from inside that shell, those resources are local. The machine is just where the axis is anchored. It is not what defines locality.

---

## The Boot Sequence as Context Expansion

Every boot stage is a shell with a narrower or wider projection axis:

- Firmware — sees hardware registers, nothing else
- Bootloader — sees storage and a minimal filesystem
- Kernel — sees devices, memory, processes
- Init — sees services and dependencies
- Login shell — sees user context, network, the full relational structure

Each stage is a shell handing off to a shell with a wider projection axis. Each one establishes enough context for the next shell to exist. Booting is not a special process separate from normal operation. It is a sequence of context expansions, each one making the next resolution possible. The same operation all the way down.

Even the BIOS is a shell. It has an extremely narrow projection axis and almost no environment, but it is still a context. It still has a here and a what's reachable from here.

---

## The Monad Structure

The shell is the monad primitive. The type signature is:

```
shell -> operation -> shell
```

Every operation takes a context and returns a context. Composition is just chaining — each operation receives the context established by the previous one and returns a potentially richer or narrower context for the next. The pipeline is a sequence of context transformations.

Failure is a shell with a narrower projection axis than you started with. Not a broken state — a more constrained context. The system doesn't crash, it contracts.

The monad describes the implementation structure of the primitive. The primitive is timeline serialization. The monad is what it looks like when you build something that preserves that property through composition.

---

## User Space: dna.toml and bind.sh

Once the shell reaches user space it needs to know what's available to serialize — what resources exist, what identities are present, what capabilities are bound to this position in the network.

Two things are defined for this purpose.

`dna.toml` is the atemporal relational structure. It holds the system's description of itself — identities, capabilities, routes — as a set of coexisting bindings with no inherent order. It is not a database and not a configuration file in the traditional sense. It is the relational structure that shells traverse. It does not change when you run a shell against it. The shell changes.

`bind.sh` is a policy router. It takes the current shell context, projects the relevant slice of `dna.toml` into it, and returns an extended context. It is not a configuration tool. It is a lens — it changes what correlation is available to be serialized by subsequent operations.

The toml is outside the timeline. bind.sh is what brings a slice of it inside.

---

## The Policy Engine and Routing Syntax

The policy engine needs to express directed relationships between composed contexts. A proposed syntax:

```
{composable.src} / {composable.dst}  guard0  guard1
```

Dot composition narrows context from left to right. The slash separates source and destination frames of reference. Guards are filters applied to the traversal.

The asymmetry of the slash is not a limitation — it is what makes the routing table meaningful. `user.paul / system.smb` means something different than `system.smb / user.paul` because the projection direction matters. The left side is the frame of reference. The right side is what is being seen from that frame. Order encodes causality.

Within each side of the slash, dot composition can be symmetric where the data warrants it. `user.permissions` and `permissions.user` might resolve identically — not because the operator is commutative but because that particular relationship happens to be symmetric. That symmetry is a discoverable property of the data, not an assumption of the algebra. Finding it tells you something real about the structure.

This keeps the state space plastic. Because the algebra does not force symmetry but allows it where it exists, the same relational structure can be navigated from multiple directions. There is no canonical traversal. You can approach any binding from either side and the algebra tells you what you get from that position. The system remains navigable rather than rigid.

---

## Mapping to LMDB

The dot composition maps naturally onto an ordered btree keyspace. `user.paul.acs` is a key prefix — all bindings under that context are reachable by prefix scan without a separate query planner. The slash boundary becomes the separator between two btree lookups. Guards are additional key components that narrow the scan.

The policy engine is then a btree traversal with the composition rules defining the key structure. No query planner needed. The key is the query.

Versioning falls out naturally if a commit or context component is included in the key — making every binding addressable not just by identity and capability but by point in time. The full proposed key syntax:

```
{domain}.{context}.{commit} / {resource}.{tag}.{guard}
```

This is the same operation at every level — prefix scan, narrow by composition, project into the current shell context.

---

## Summary

The shell is not a tool running inside a system. It is the thing that produces local time by serializing correlation. The relational structure is atemporal. The shell is what makes it experienceable as a sequence.

Every layer of the stack — firmware, bootloader, kernel, init, login shell, policy router — is a shell in this sense. Each one establishes a projection axis, serializes what it can reach, and hands a richer context to the next. The boot sequence is context expansion. The distributed system is a projection axis that reaches across the network. The local machine is just where the current axis is anchored.

`dna.toml` is the structure outside the timeline. `bind.sh` is the lens that brings a slice of it inside. The policy engine is the algebra that makes the traversal composable. And the monad — shell -> operation -> shell — is what it looks like when you build all of this correctly.
