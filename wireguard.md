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

Every card bottoms out at a file path on a volume. That is the ground truth — no further abstraction below it. You can always ask "where does this actually live" and get a concrete answer: `/home/fs/<domain>/<concern>/<hash>`. That path tells you who owns the data, what kind of data it is, and what specific resource it is. The entire card system, all rendering contexts, all routing and relationship logic — it all resolves to file paths on LUKS volumes.

A card is a reference to a data group with enough metadata to present it meaningfully:

```
card.name
card.description
card.target        # file path on a volume — the ground truth
card.children[]    # list of cards, or inline card data
card.siblings[]    # same concern level
card.parent        # what this belongs to
card.state         # collapsed, preview, expanded, focused
card.drag()
card.drop()
```

The target can be anything — a Word document, a web URL, a SolidWorks file, a btrfs snapshot, a structured data entry. The card wrapper is stable, the target is whatever makes sense for that concern. The rendering system doesn't need to know what's inside the target, only how to present the card metadata and link to it.

A directory is just a card that contains a list of other cards. A file can itself be a list of cards with their data inline. The card is the unit at every level.

### The Deck

A deck is a volume. One to one, no exceptions. There are no nested decks — only cards referencing data in another volume. Volumes don't nest, they reference each other through symlinks, and card references between decks are exactly that.

- Volume = deck
- File or directory in that volume = card
- Reference to another volume = card reference to another deck

A deck is self-describing. The volume contains the data, the card relationships, and the rendering hints. Mount the volume anywhere and the rendering context comes with it. Another system mounts it and knows how to display it without external configuration.

The OS UI is a deck organizer. Cards lay out next to each other with varying degrees of expand and collapse. The deck is the working surface — you expand what you need, collapse what you don't. Layout reflects current attention, not a fixed hierarchy.

Natural card states:
- **Collapsed** — name, status indicator
- **Preview** — name, description, key metadata, sibling count
- **Expanded** — full content, children visible, relationships navigable
- **Focused** — full screen, children laid out as their own deck

Navigation is expansion and collapse, not movement through folders. The deck state itself is lightweight — which cards are expanded and how they're arranged. That state lives on the client volume. The data lives wherever it lives.

The only thing that crosses deck boundaries is references — a card pointing to a card in another deck. That reference is a path to a file on another volume, which is a symlink. The rendering system follows it if the target deck is mounted, shows a dead reference if it isn't.

### Primitive Operations

Data management across the entire card system reduces to a small set of primitive operations:

- **create** — new card in a deck
- **move** — reassign parent/sibling, which is a symlink change
- **reference** — point to a card in another deck
- **expand/collapse** — change card state
- **drag/drop** — move or reference depending on context
- **follow** — traverse a reference to another deck

That is essentially it. Everything else is the rendering context interpreting those operations differently. Drag and drop in a kanban reassigns a concern. The same operation in a CAD environment repositions a sub-assembly. In a game engine it means picking up an object. The operation is identical, the rendering context gives it meaning.

Button presses in a game are card interactions at high frequency. A keypress follows a reference to an action card, executes it, updates state. The game loop is the rendering context cycling through card states very fast. Input handling, data management, and rendering all reason over the same primitive set. There is no special game input system, no special CAD interaction model, no special form handling for web. Just card operations interpreted by the active rendering context.

The complexity ceiling of what you can build is how sophisticated your rendering context is, not how complex your data primitives are. The primitives stay simple all the way up.

### Rendering Contexts

The same card primitive renders differently depending on context. The card API is platform agnostic — the primitives are defined once, rendering is pluggable:

- **Web** — cards render as HTML, a parts order is a form, a catalog item is a product listing
- **Kanban** — cards grouped by concern or status, drag between columns reassigns routing
- **3D/CAD** — cards interpreted spatially, children are sub-assemblies, siblings are adjacent components, position and geometry are properties the 3D renderer handles
- **Game engine** — card relationships interpreted as spatial and temporal, physics and animation are renderer properties
- **Terminal** — ncurses-like representation of the same card graph

A CAD model is a card whose target is geometry data. In a parts ordering context the same card renders as a line item with quantity and price. Same data, different rendering context.

The menu system inside a CAD environment runs the same code as accessing data in an order form on a website. A button is a card with a target that is an action. A dropdown is a card whose children are options. UI chrome and the data it operates on are the same primitive — cards all the way down.

### Relationship to the Volume System

Because cards map directly onto the filesystem structure there is no translation layer. A card is a directory entry with metadata. Children are symlinks. The deck is a graphical traversal of what already exists in the volume structure.

The system is inspectable at every level without the UI being present. If you can mount the volume you can see exactly what is there and where everything points. The UI is a convenience, not a requirement for understanding what the system contains.

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

## Rendering, Input, and Semantic Space

### Hit Detection as Universal Input Primitive

Hit detection is the universal input primitive across all rendering contexts. Whether it is:

- A mouse click on a button — did this coordinate intersect this card's bounding region
- A key press — did this input intersect the focused card's input handler
- A cursor in a text field — where in the card's text does this position land
- A bullet in a game — did this trajectory intersect this card's collision volume
- A tool path in CAD — did this geometry intersect this feature

It is all the same operation. A ray or point tested against a card's defined region. The card defines its hit volume, the rendering context defines what coordinate space that is in — 2D screen space, 3D world space, text buffer space — and the hit test is uniform across all of them.

Overflow handling is equally universal — what happens when a card's content exceeds its defined region. Scroll, clip, wrap, collapse. The same four options whether handling text overflow in an input field, geometry clipping in a CAD viewport, or a game object leaving the play area.

The rendering system's core responsibilities reduce to:

- Maintain card regions in the active coordinate space
- Test inputs against those regions
- Handle overflow at region boundaries
- Update card state based on results

Everything else — game physics, text editing, button clicks, CAD interaction — is the rendering context defining what coordinate space and what overflow means for that context. The mechanism is the same throughout.

### Search as Hit Detection in Semantic Space

Search is just hit detection in semantic space. Instead of a ray in 3D world space or a point in screen space, a query is cast through the graph of semantic relationships and tested for intersection.

A search query is a trajectory with direction and weight — it hits some cards more solidly than others based on how well their semantic region overlaps the query vector. Relevance is hit strength. Filtering is defining the collision volume more narrowly.

Because the semantic graph is just card relationships — the same symlinks and references that define the UI structure — search traverses the same graph the renderer traverses, just with different hit geometry. A card's semantic volume is defined by its name, description, relationships, and concern context. The query intersects that volume the same way a mouse click intersects a bounding box.

Faceted search falls out naturally — just adding more constraints to the hit geometry. Cards that intersect both "aluminum" and "order" and have a child that intersects "overdue" is a compound hit test across the semantic graph.

Results are cards, same as everything else. Search output is a deck of cards that passed the hit test, rendered however the active context renders a deck. No special search results UI — just another view over card relationships.

### Phase Diagrams and Projection

The semantic space can be visualized as a phase diagram — cards occupy regions that overlap like phases in a material diagram. A search trajectory passing through the diagram intersects whatever phases it crosses. The diagram makes the semantic structure legible: which concerns cluster together, which cards sit at phase boundaries, which regions are dense with related content.

Changing the filter is rotating the projection angle — looking at the same semantic graph from a different direction so different phases come into view. A budget query projects from one angle and intersects financial cards. The same data projected from a project angle shows the same cards grouped differently. Same graph, different cut through the phase space.

Bubbling is where cards near the boundary of a projection get partially hit — relevant but not central. Strong hits are deep in the intersected phase, weak hits are at the phase boundary. The relevance gradient comes from the geometry without a separate ranking algorithm.

The phase diagram is not a separate search index. It is the card relationship graph rendered in semantic coordinate space instead of screen space. It updates live as cards are added or relationships change because it is just a view of the volume structure.

### 3D Scene Graph and Dimensional Projection

The semantic graph can be rendered as a 3D scene graph — like Blender or a CAD viewport — where card placement and overlap represent information relationships. Each axis is an information dimension: concern type, time, ownership, relevance, phase, or whatever is meaningful for the current task. Three dimensions are chosen to project into viewable space and navigation happens from there.

Card placement in that space defines relationships visually. Two cards that overlap are semantically close in those three dimensions. A card positioned between two others is a mediator — information passes through it. This makes filter and transform layers spatial and inspectable.

Directionality matters. A ray from an action card through a filter card to a target card means "this action is mediated by this filter before reaching this target." Reverse the ray and it means "changes to this target are viewed through this lens." Same cards, same positions, different traversal direction, different meaning. Like a lens that behaves differently depending on which side the light enters.

The full semantic graph may be n-dimensional but only three dimensions are legible at a time. Changing which three to project onto is rotating the basis vectors — the cards do not move in the full semantic space, you are choosing a different slice to make visible. Switch from concern/time/owner to concern/relevance/phase and the same cards rearrange into a different meaningful layout. The constraint of three dimensions is a feature not a limitation — it forces a choice about what information matters right now, and that choice is itself navigable.

You can shape cards such that their overlaps represent areas of concern:
- A card positioned in the path of an action filters that action before it reaches its target
- A card positioned between a change and an observer mediates what that observer sees
- The cartesian axes are different information paths
- Rotating the projection changes which paths are foregrounded

The scene graph is a navigation tool for the semantic space, not a separate data structure. The same card relationships, always — just a choice of which three dimensions to surface at any moment.

---

## Automation

Automation can be constraint driven. Drawing 'potential wells' inside something like 'simulink' gives you a modeling environment for behavior primitives. You can add switches, step functions, catalysts/triggers. Those primitives describe a vast collection of automation topologies currently viewed as 'separate' but actually behave as equivalence classes. You can view your system (say you want to automate key lookup and mounting) as a constraint topology. Linking card handles (callbacks) to geometries of other conditions (password submission acts like a latch for example).

### Potential Wells

---

## Summary

The system is a dependency graph of LUKS volumes connected by symlinks, routed by CTXT/ROUTE environment variables, synced by btrfs send/recv, and networked by Wireguard. Each layer uses the same key primitive. Security is cryptographic at every boundary. Complexity matches actual coordination need — a solo user has one volume and no overhead, a large distributed team has many volumes and rich routing, the mechanism is identical throughout.
