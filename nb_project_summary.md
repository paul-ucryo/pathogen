# .nb Project Summary

## The Core Idea

A unified document/notebook format that is:
- A live config file with backend access
- A project management tool (tasks, issues, log, SOPs)
- A composable context pipeline
- The interface layer over arbitrary data backends

The file is always readable as plain text. Tooling enhances but never owns the data.

---

## Syntax Sketch

### Blocks
```
[block-id] /flag:value /flag2:value2
content
block-id]

[[reference]]   -- nested or referenced block
```

### Tables / Schematic Data
```
[[table-name]]:csv /ui:spreadsheet /shader:schematic
a,b,c
1,2,3
:csv
```

### Templates (lazy, partial application)
```
greeting = s'hello {name}, system is {status}'
greeting(name='alice')        -- partial, status still open
greeting(name='alice', n=3)   -- fully resolved
greeting()                    -- renders with current scope, holes visible
```

Combines f-string legibility with deferred evaluation. Inspired by Haskell range syntax `1..4` for inline iteration.

### Key/Value Query DSL
```
{domain}.{context}.{commit}:{resource}.{tag}.{with}.{as}.{fn_composition} {value}
```

The key IS the query. Ordered btree means prefix scans are free — narrow from coarse to fine by adding dots. No separate query planner needed. The commit component makes everything versioned by default.

### Rendering Flags
```
/ui:spreadsheet|schematic|vtxt|flowchart
/shader:custom_render_fn
/out:vtxt|file|winID
/backend:lmdb|http|sql
/overflow:scroll|top|bottom
```

Same data, different rendering context. The shader is just another template in the same system.

---

## Architecture

### Storage Stack
```
identity
  └── context pipeline (.nb composition)
        └── persistence policy (btrfs/ZFS/bcachefs)
              └── LMDB keyvalue btree
                    └── write multiplexer (single-writer queue)
```

### Filesystem as Diff Graph
Everything is accumulated broken symmetry. A file is a diff sequence, not a stable object. Different views (customer, developer, submodule) are different traversals of the same diff graph — different packaging of the same underlying btree objects.

Git's internal model is the closest existing thing but git is opaque — diffs are blobs, not queryable AST nodes. The .nb + LMDB model makes diffs first-class addressable objects.

### Context Pipeline
`.nb` files compose top-down like a monad pipeline:

- `/etc/nb/00-default` — global base (optional)
- Walk path toward target, apply each `.nb` found
- Each narrows or overloads the context
- Exit behavior: `map` (transform and continue), `short` (handle and stop), `fallback` (try next, return if unhandled)

No config required at root. A single `00-default.nb` in the project folder bootstraps a working system.

### Domain / Project Model
No folders as hard boundaries. Everything is a node. Project and domain are just tags — filter dimensions over a flat keyspace.

```
file: server-config.nb
tags: [infrastructure, file-server, vendor-acme]
```

"Show me the file-server project" = filter on tag `file-server`.
Access control = another filter dimension based on identity tags.

---

## VCS Integration

### Git Scripts
```bash
# save.sh — snapshot everything, no decisions required
git add -A && git commit -m "$(date '+%Y-%m-%d %H:%M')"

# checkpoint.sh — save with a note
echo "note: " && read note
git add -A
git commit -m "$(date '+%Y-%m-%d %H:%M') - $note"
HASH=$(git rev-parse --short HEAD)
# append to .nb log: time, note, hash
```

### .nb Log Entry
```json
{
  "log": [
    {"time": "2024-03-14 15:32", "note": "split rotation working", "hash": "a3f2c1"}
  ]
}
```

Git is backup. The `.nb` file is the legible surface. The hash links them.

---

## SVG / Vector Graphics as CSV

SVG serializes naturally into CSV. The visual complexity is a rendering concern — the data is just coordinates and relationships.

```
[[wiring]] /ui:schematic /shader:firewall
type,id,x,y,w,h,connects_to
rect,firewall1,0,0,100,50,router1
line,conn1,100,25,200,25,switch1
:csv
```

Same CSV renders as a diagram, a wiring table, a firewall policy summary, or a legal decision tree — just different shaders. The shader is a template, composable and swappable. The file never changes.

---

## Neovim Integration (2ui in terminal)

The `.nb` file is 2ui running in vim. Same paradigm, different renderer.

### Block syntax for execution
```
@block1 /o:value /target:winID
#code
block1@

@winID /overflow:scroll
winID@
```

`@block@` — source block, runs code, pipes to target  
`#winID#` — output block, displays result  
`/out:vtxt` — render output as virtual text inline instead of writing to tile

### Plugin structure (TBD — `nb.nvim` or `foldr.nvim`)
```
plugin/
  nb.lua           -- entry, registers commands and filetype
lua/
  nb/
    parser.lua     -- reads/writes .nb format
    layout.lua     -- manages split/tile structure
    execute.lua    -- runs blocks, pipes output
    bindings.lua   -- input binding config
ftplugin/
  nb.lua           -- filetype behavior
syntax/
  nb.vim           -- highlighting
```

Shortest path to useful: syntax highlighting + concealment + virtual text for output. Feels close to Mathcad without full rendering infrastructure.

---

## License / Policy Framework (TBD)

**This is an open design problem.** The composable context pipeline suggests a natural extension into access policy — but the right model is not yet defined.

The intuition:

Just as `.nb` files compose context top-down (global → domain → project → file), a license or policy framework could compose the same way. Each layer narrows or overloads the permissions of the layer above. A resource's effective policy is the result of walking the full composition chain to that resource.

Key properties the framework should have:

- **Composable** — policies combine algebraically, no special cases
- **Auditable** — the full policy chain for any resource is always traceable
- **Identity-scoped** — customer, vendor, consultant, internal all have different views of the same resource graph, defined by which tags and domains their identity resolves to
- **Late-bound** — policy is resolved at access time against current context, not baked into the resource at creation
- **No privileged nodes** — no single policy owner; policy is a property of the relationship between identity and resource, not of either alone

The `.nb` block syntax already supports this directionally — `/backend:identity` could attach an identity store to the file's scope, and the tag system provides the filter dimensions for access control.

A git-style model where access policy is defined per-domain and inherited/overridden by projects (like gitignore walking up the tree, but for permissions) is probably the right starting point — with the monad exit behavior (`short`, `map`, `fallback`) determining how conflicts resolve.

**This needs real use before it can be designed correctly.** The workflow will reveal the right primitives. Don't design it upfront.

---

## Immediate Next Steps

1. **Git scripts** — `save.sh` and `checkpoint.sh` working and habitual. This is the safety net that lets the format evolve without fear.
2. **First `.nb` file** — syntax loose, just useful. One file for the most pressing current project.
3. **Format stabilizes through use** — don't design it, grow it.
4. **Neovim syntax file** — just highlighting and concealment, get the Mathcad feeling cheaply.
5. **Everything else** — when the foundation is stable.

---

## Current Storage Recommendation

- **Now**: btrfs on Fedora — stable, COW, subvolumes enough to prototype
- **Scale**: ZFS — production hardened, send/receive for distributed backup
- **Future**: bcachefs — btree objects natively, most aligned with the diff-as-primitive model; wait for maturity (kernel 6.7+)

Migration is concept-preserving at each step. Nothing thrown away.
