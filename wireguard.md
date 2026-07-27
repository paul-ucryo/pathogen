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

## UI Model — The Card System

### The Card as Primitive

The card is the UI primitive, analogous to a punch card — a defined set of data and relationships that different rendering contexts interpret differently. Like Tk's widgets or the DOM's elements, the card is what the rendering system reasons over. The rendering context determines what the card looks like and how it behaves, not the card itself.

A card is a reference to a data group with enough metadata to present it meaningfully:

```
card.name
card.description
card.target        # the artifact — a file, URL, structured data entry, snapshot
card.children[]    # sub-items, drill-down
card.siblings[]    # same concern level
card.parent        # what this belongs to
card.state         # collapsed, preview, expanded, focused
card.drag()
card.drop()
```

The target can be anything — a Word document, a web URL, a SolidWorks file, a btrfs snapshot, a structured data entry. The card wrapper is stable, the target is whatever makes sense for that concern. The rendering system doesn't need to know what's inside the target, only how to present the card metadata and link to it.

Drag and drop is just reassigning relationships — moving a card changes its parent or sibling, which is a symlink change in the filesystem. The visual behavior is the renderer's concern, the data change is just routing.

### The Deck

The OS UI is a deck organizer. Cards lay out next to each other with varying degrees of expand and collapse. The deck is the working surface — you expand what you need, collapse what you don't. Layout reflects current attention, not a fixed hierarchy.

Natural card states:
- **Collapsed** — name, status indicator
- **Preview** — name, description, key metadata, sibling count
- **Expanded** — full content, children visible, relationships navigable
- **Focused** — full screen, children laid out as their own deck

Navigation is expansion and collapse, not movement through folders. A domain card expands to show its concerns. A concern card expands to show its items. An item card expands to show content and relationships. The hierarchy is there, zoom level is how you traverse it.

The deck state itself is lightweight — which cards are expanded and how they're arranged. That state lives on the client volume. The data lives wherever it lives.

### Rendering Contexts

The same card primitive renders differently depending on context. The card API is platform agnostic — the primitives are defined once, rendering is pluggable:

- **Web** — cards render as HTML, a parts order is a form, a catalog item is a product listing
- **Kanban** — cards grouped by concern or status, drag between columns reassigns routing
- **3D/CAD** — cards interpreted spatially, children are sub-assemblies, siblings are adjacent components, position and geometry are properties the 3D renderer handles
- **Game engine** — card relationships interpreted as spatial and temporal, physics and animation are renderer properties
- **Terminal** — ncurses-like representation of the same card graph

A CAD model is a card whose target is geometry data. In a parts ordering context the same card renders as a line item with quantity and price. Same data, different rendering context.

The menu system inside a CAD environment runs the same code as accessing data in an order form on a website. A button is a card with a target that's an action. A dropdown is a card whose children are options. UI chrome and the data it operates on are the same primitive — cards all the way down.

### Relationship to the Volume System

Because cards map directly onto the filesystem structure there is no translation layer. A card is a directory entry with metadata. Children are symlinks. The deck is a graphical traversal of what already exists in the volume structure.

New artifact types are just new card targets. New views are just new rendering contexts. The card API and the volume structure are stable — complexity lives in the renderers, not in the data model.

### Concerns as First-Class Cards

The concern taxonomy maps naturally onto the card system. For an operational shop system the initial concerns to implement as card collections are:

- **catalog** — what parts, materials, and services exist and their specs
- **inventory** — quantities of catalog items on hand
- **budget** — spending authority and allocation
- **order** — acquiring catalog items against a budget, updating inventory on receipt
- **issue** — discrepancies, problems, open questions referencing any of the above

An order card references catalog cards for line items, inventory cards for stock levels, budget cards for spending authority. An issue card references whatever it concerns. The relationships are symlinks, the display is the renderer reasoning over those relationships.

The initial UI is a deck of these concern collections. Cards within a concern are laid out as siblings. Expanding an order shows its referenced catalog items, its budget draw, its effect on inventory. The data model is already implied by the concern relationships — the UI just makes it navigable.

---

## Summary

The system is a dependency graph of LUKS volumes connected by symlinks, routed by CTXT/ROUTE environment variables, synced by btrfs send/recv, and networked by Wireguard. Each layer uses the same key primitive. Security is cryptographic at every boundary. Complexity matches actual coordination need — a solo user has one volume and no overhead, a large distributed team has many volumes and rich routing, the mechanism is identical throughout.
