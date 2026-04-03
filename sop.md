# Roles as Legible Structure

## The Problem with Documentation

Most systems are documented after the fact. Someone builds a thing, then writes a manual explaining what they built. The manual is immediately stale because it describes the system at a moment, not the system as it lives. Worse, it describes the *what* without the *why* — here's what this setting does, but not why it's this value and not another.

The result is that documentation becomes red tape. Nobody reads it because it doesn't track reality. Nobody updates it because updating it is separate work from the actual work. The system drifts from its description and the next person inherits an artifact they can't trust.

The alternative is a system where the design is the documentation. Not comments explaining code — structure that is self-describing because the relationships are legible in the structure itself. The product is its own manual because the manual is the product.

## Four Terms

The vocabulary has four terms in two pairs:

**Potential** is a well in the environment — energy that would move if there were a path. Something needs to happen. A file needs to be accessible. An identity needs to resolve. A material needs to become a different shape. The potential exists whether or not anything addresses it.

**Capability** is a specific transform that resolves a potential. SMB resolves file access. DNS resolves identity. A CNC mill resolves material removal. The capability is the path through which energy transitions from one state to another.

**Field** is a region of the landscape where related potentials cluster. A workplace, a home network, a government agency, an industry sector. Fields have character — the same potential (say, trust) manifests differently in different fields. Trust at home is a Wi-Fi password. Trust in defense is ITAR clearance. Same potential, different impedance.

**Role** is a group of capabilities that together form a functional shape. A role isn't a job title — it's the set of transforms a node can perform. The `design` role groups the capabilities needed to take a problem from undefined to specified. The `fabrication` role groups the capabilities needed to take a specified shape and make it physical.

## Where Roles Come From

Roles don't come from org charts or job descriptions. They derive from potentials.

The environment presents potentials — wells where energy accumulates. Each potential implies a capability that would resolve it. Capabilities that get exercised together in response to the same field cluster naturally into roles. The role is emergent, not designed. You discover it by looking at which capabilities consistently fire together.

This means roles are descriptive, not prescriptive. You don't assign someone a role and then list what they're allowed to do. You observe what potentials they resolve and the role is the name for that pattern. If the pattern changes — if new potentials appear or old ones drain — the role evolves without anyone rewriting a policy document.

In system terms: Paul resolves design, fabrication, production, and poc potentials. That's not because someone assigned those roles — it's because those are the capabilities that fire when the environment presents energy in those wells. The role list is a snapshot of the binding pattern, not a permission set.

## The Potential/Capability Skeleton

Here's the current map. Each potential names a well. Each capability names what resolves it. The pairing is the reason the capability exists — remove the potential and the capability is bloat.

### Asset Management

These potentials exist because an organism that doesn't track its own resources can't allocate energy intentionally.

**budget** — the potential for resource misallocation. Energy needs to flow to the right places. Without legibility on where money is and where it's going, allocation is reactive. Capability: accounting systems (QuickBooks, spreadsheets, whatever makes the ATP flow visible).

**catalog** — the potential for unnamed assets. Things exist in the organism that aren't legible — a tool nobody knows you have, a material sitting on a shelf, a capability that isn't expressed because it isn't listed. Capability: inventory/catalog systems that make the genome visible to its own nodes.

**inventory** — the potential for unknown state. Different from catalog — catalog is *what exists*, inventory is *how much and where*. The organism needs to know not just that it has aluminum stock but how much, what alloy, what condition. Capability: tracking systems that maintain current state.

### System

These potentials exist because a computing system — like any organism — must maintain the conditions for its own operation before it can do anything useful.

**health** — the potential for undetected dysfunction. The system can be failing in ways that no individual component reports. Health is the field measurement — is the whole thing alive, is signal propagating, are bindings resolving? Capability: a user. A person who reads the state, feels the shape, notices when something is off that no alert would catch. This cannot be fully automated because automation only checks what it's been told to check. Health requires the kind of pattern recognition that notices what's missing.

**audit** — the potential for incoherent process. Transforms executed without traceability. Decisions made without legible rationale. The system works but nobody can verify that it worked correctly or reconstruct why it did what it did. Capability: logging, traceability, standards (ISO). Audit answers "did the process execute as specified." It is internal coherence — looking inward at the mechanism.

The distinction between health and audit: a system can pass every audit and still be dying. Audit checks the rules. Health checks whether the rules are the right ones. Audit is necessary but not sufficient.

**access** — the potential for unreachable resources. Data exists but the node that needs it can't get to it. A design file on one machine that the machinist on another machine needs. Capability: SMB, NFS, whatever protocol makes resources reachable across the topology.

**identity** — the potential for ambiguous nodes. The system can't route if it can't distinguish who is asking, which machine is responding, what domain a request belongs to. Capability: DNS, LDAP, LLDAP — anything that resolves a name to a specific entity.

**trust** — the potential for unverified claims. Identity says who you are. Trust says: prove it, and here's what you're allowed to do once proven. Without trust, identity is just a label anyone can wear. Capability: PAM (authentication decisions), WireGuard (encrypted transport that verifies both ends), certificates, kerberos — the stack that makes identity consequential.

**transform** — the potential for unexecuted intent. A user has something they want to do. The system needs to accept that intent and execute it. Capability: the shell. The most fundamental capability — a place where a human's intent meets the machine's state and produces a delta.

**persist** — the potential for lost state. Without persistence, every transform is temporary. The system does work and the work disappears. Capability: LMDB, BTRFS, any storage layer that makes state survive across time. Distinct from backup (which is persist's resilience layer — persist keeps state, backup keeps persist).

### Communication

These potentials exist because a node that can't send or receive signal is functionally dead to the network, even if it's internally healthy.

**messaging** — the potential for undelivered signal between nodes. One person knows something another person needs to know. Without a channel, the signal dies at the source. Capability: email, Slack, Matrix — whatever carries intentional communication between nodes.

**presence** — the potential for invisible organisms. The company exists, has capabilities, could bind to potentials in the environment — but the environment can't see it. The receptor site isn't expressed. Capability: website, public DNS, any surface that makes the organism's binding potential visible to external fields.

**alert** — the potential for unnoticed state changes. Something changed and someone needs to know. Different from messaging — messaging is intentional communication, alerting is the system itself surfacing a state change that a node needs to respond to. Capability: monitoring systems, CRM alerts (Salesforce), any system that converts a state change into a signal routed to the right node.

## How Roles Compose

A role is a group of capabilities. But roles also compose with each other through dependency.

The `employee` role requires `shell` access — you can't work without being able to transform state. It suggests `smb` — file access makes most work possible but isn't strictly required for every function. It implies `identity` and `trust` underneath, because shell access without authentication is an incoherent binding.

The `design` role requires `employee` (which brings shell, identity, trust) plus `access` (needs to reach and modify design files). It implies `persist` because design work that doesn't survive is wasted energy.

The `fabrication` role requires `employee` plus access to machine state and design files. It implies `audit` because fabricated parts without traceability are unverifiable.

The `poc` (point of contact) role requires `messaging`, `presence`, and `alert` — because the poc is the node at the membrane between the organism and the environment. They need to send signal outward (presence), receive signal inward (messaging), and notice state changes (alert).

These dependencies aren't policies imposed from above. They're the natural implication of what each role actually does. If you removed `trust` from under `employee`, the role stops functioning — not because a rule says so but because the transforms can't execute without verified identity. The dependency is structural, not bureaucratic.

## Self-Describing Systems

When the structure is legible — when potentials name the *why*, capabilities name the *what*, and roles name the *who* — the system documents itself.

A new person joining the team doesn't read a manual. They read the dna file. They see: here are the potentials the system addresses, here are the capabilities that resolve them, here's which roles bind to which capabilities, and here's how the dependencies chain. They understand not just what the system does but why each piece is there.

When something breaks, the troubleshooting path is in the structure. Access isn't working? Walk the dependency chain: access depends on identity depends on trust depends on persist. Which binding failed? The answer is in the topology, not in a separate troubleshooting guide.

When someone asks "can we remove this?" — the answer is in the potential. Is the potential still active? Is energy still accumulating in that well? If yes, removing the capability creates an unresolved potential. If no, the capability is bloat and should go. The justification for every component's existence is its named potential.

When the environment changes — a new field appears, an old one drains, potentials shift — the roles evolve because the binding pattern changes. The file updates to reflect the new reality, and the update *is* the documentation. There's no separate step of "update the docs." The system's description is the system's structure, and changing the structure changes the description.

This is what "the product is its own manual" means. Not that the product doesn't need documentation — but that the documentation isn't a separate artifact maintained by separate effort. It's the same file that configures the system, describes the system, and justifies the system. One source of truth that serves the machine and the human reading it.

## What This Doesn't Cover Yet

**Fields** need more development. The current candidates — work, home, city.gov — are placeholders for a richer description of the thermodynamic landscape. Each field should characterize: what potentials are active in this region, what impedance exists between the field and the organism's capabilities, and what surfaces express the organism's binding potential into this field.

**Impedance** between potentials and capabilities needs articulation. What stands between a potential well and the capability that would resolve it? For `trust`, the impedance might be certificate infrastructure you haven't built yet. For `presence`, it might be a website that doesn't express your actual binding potential. Impedance is where the real work of system design lives — not choosing capabilities but reducing the barriers between them and the potentials they serve.

**Health metrics** as field measurements orthogonal to the flow need a concrete implementation. What do you actually look at to know the system is alive versus just passing audit? This might not be solvable in advance — it might require running the system and discovering what the meaningful signals are. But the slot should exist so the question stays visible.

**Scaling** from server to business. The vocabulary works at both scales. But the specific potentials and capabilities at the business scale — where fields map to industries, potentials map to market needs, and capabilities map to the company's functional capacity — need their own enumeration. The structure is the same. The content is different.
