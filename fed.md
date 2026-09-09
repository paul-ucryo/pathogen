# /fed — A Cryptographic Namespace

## Core Idea

A federated system where every domain is a cryptographic identity, and the filesystem is the interface. There is no location, only address. The address and the content are part of the same description.

## Structure

```
/
  .a1b2c3d4          ← encrypted volume (LUKS/ZFS), named by identity
  fed/               ← federation gateway, mounted from .a1b2c3d4
    .b5e6f7g8        ← domain volume
    myservice/       ← human readable label, mounted from .b5e6f7g8
      bind/
        nginx        ← symlink into local service environment
        dns          ← symlink into local service environment
        fs           ← symlink into local service environment
```

## Identity

Each domain is an ed25519 keypair. The public key is hashed to produce the domain ID:

```bash
id0=$(openssl pkey -in pubkey.pem -pubin -outform DER | sha256sum | cut -c1-8)
```

The private key unlocks the LUKS volume. Proving identity and opening the domain are the same operation.

## Storage

Each domain is a LUKS2 encrypted volume over a sparse file, with ZFS on top:

```
/.a1b2c3d4           ← sparse file (truncate -s 10G)
/dev/loop0           ← loopback device
/dev/mapper/fed      ← LUKS decrypted device
/fed                 ← ZFS pool, mounted at label
```

Domains are independent LUKS channels through device-mapper — not nested encryption, parallel channels. Each domain has its own key, its own mapper device, its own ZFS pool.

## Namespace

Labels are mounts of cryptographic containers, in the Plan 9 tradition:

```
/fed/.a1b2c3d4    ← cryptographic identity
/fed/myservice    ← label, mounted from .a1b2c3d4
```

The label is stable. The identity underneath it can be swapped by unmounting and remounting a different container. Services point at labels, not identities — so they are automatically decoupled from the cryptographic layer.

## Service Binding

Bindings are symlinks from the local service environment into the domain:

```bash
ln -s /fed/myservice/bind/nginx /etc/nginx/sites-available/myservice
```

The domain carries its service definitions. The local environment points at the domain. When a domain is mounted, services pick up its config automatically. When it is unmounted, the config disappears. No service configuration changes hands — only the mount changes.

This is monadic composition at the filesystem level. Each binding extends what the domain is, not just what it runs. The domain's capabilities are the product of its bindings.

## Distribution

Transport security is redundant. The domain is encrypted at rest by LUKS. A delta is just encrypted bytes — opaque without the private key. The transport needs to be reliable, not secure.

Sync is a delta query against the cryptographic identity:

```bash
zfs snapshot /fed/myservice@v2
zfs send /fed/myservice@v1 /fed/myservice@v2 | curl -X POST http://othernode/fed/myservice
```

Deltas can be stored on untrusted infrastructure — S3, bittorrent, sneakernet. Anyone can host them. Nobody can read them without the key.

Distribution is constraint and query, not sync:

- who has this identity mounted
- who can satisfy its bindings
- route to whoever answers

There is no central authority. There is no location dependency. There is no config drift.

## Addressing

Location is a runtime convenience, a hint about where to find an address. The address is the thing.

The cryptographic ID is not a location — it is a description. The content at that address is part of what the address means. You cannot have different content at the same address because the address is derived from the identity of the content.

The domain exists independently of where it is mounted or who is hosting it. The `bind/` directory makes this concrete — what a domain is includes what it binds. The address describes not just content but capability.

## Bootstrap

```bash
newDomain() {
  local base=${1:-/fed}
  local name=${2:-}
  local keystore=/etc/keys/fed
  mkdir -p $keystore

  openssl genpkey -algorithm ed25519 -out $$.tmp.0
  openssl pkey -in $$.tmp.0 -pubout -out $$.tmp.1
  local id0=$(openssl pkey -in $$.tmp.1 -pubin -outform DER | sha256sum | cut -c1-8)

  mv $$.tmp.0 $keystore/$id0.pem
  mv $$.tmp.1 $keystore/$id0.pub

  local mount=${name:-$id0}
  truncate -s 10G $base/.$id0
  local LOOP=$(losetup -f --show $base/.$id0)
  cryptsetup luksFormat --type luks2 $LOOP --key-file $keystore/$id0.pem
  cryptsetup open $LOOP $mount --key-file $keystore/$id0.pem
  zpool create -m $base/$mount $mount /dev/mapper/$mount

  echo $id0
}

# create federation
if ! zpool list fed > /dev/null 2>&1; then
  newDomain / fed
fi

# create domain inside federation
newDomain /fed
```

## Mount / Unmount

```bash
# mount existing domain
LOOP=$(losetup -f --show /fed/.$id0)
cryptsetup open $LOOP $id0 --key-file /etc/keys/fed/$id0.pem
zpool import -m /fed/$name $id0

# unmount
zpool export $id0
cryptsetup close $id0
losetup -d $LOOP
```

Once unmounted the domain is cryptographically sealed. The encrypted volume can be moved, copied, or hosted anywhere.

## UI — Four Region Layout

### Regions

The UI is four named regions:

```
┌─────────────────────────────┐
│ env                         │  ← environment header (url, title, geometry)
├──────────────┬──────────────┤
│ im           │ re           │  ← node editor / rendered output
│ (graph)      │ (wysiwyg)    │
├──────────────┴──────────────┤
│ bind                        │  ← binding footer (status, active mounts)
└─────────────────────────────┘
```

- **re** — the rendered content. Whatever the current domain produces as output — a webpage, a document, a data view. This is the wysiwyg face.
- **im** — a node editor over the graph that generates the rendered state. The raw domain graph, editable as nodes and edges. Changes here propagate to `re` in real time.
- **env** — the environment header. Defines setup geometry — url bar, document title, active identity, mount state. The context in which the current domain is being viewed.
- **bind** — the binding footer. Status of active bindings — which services are mounted, which snapshots are current, which branches are open.

### Layout as a Domain

The four regions are just named slots. A style page tells the system the layout independently of content:

```json
{
  "layout": {
    "env": { "region": "header", "height": "2rem" },
    "im":  { "region": "left",   "width": "50%" },
    "re":  { "region": "right",  "width": "50%" },
    "bind":{ "region": "footer", "height": "2rem" }
  }
}
```

The layout is a domain descriptor like any other. Swap the style page, the same content reflows into a different geometry. A mobile layout, a presentation layout, a terminal layout — all just different style descriptors over the same four regions.

### Simultaneous Editor and WYSIWYG

`im` and `re` are two views of the same graph state. `im` is the data editor — the graph in its native form, nodes and edges, addresses and bindings. `re` is the rendered projection of that graph through whatever driver the domain declares.

Any data type with a driver into the four panels becomes simultaneously editable and renderable:

- a markdown document — `im` is the source graph, `re` is the rendered html
- a website — `im` is the dom/config graph, `re` is the live page
- a database — `im` is the schema and record graph, `re` is a table or form view
- a codebase — `im` is the ast or file graph, `re` is the syntax highlighted source
- a federation — `im` is the domain graph, `re` is the namespace view

The driver is just a function from graph to rendered output. Writing a driver for a new data type brings it into the editor for free.

### Everything is a Driver

The regions are not special — they are just named mount points in the UI namespace. A driver populates them by binding content into the slots. The same binding mechanism that mounts services into `/fed/domain/bind/` mounts content into `re`, `im`, `env`, `bind`.

So the UI is itself a domain. The style page is a domain descriptor. The content is a domain. The editor is a domain. The whole thing composes the same way the filesystem does — names over cryptographic identities, bindings as the composition mechanism, the rendered state as the projection of the graph through its drivers.

## VZFS — Vector(delta) ZFS Strategy

### Computation as a Distributed Delta System

Computation is not different from storage — it is just another domain capability. A local station, a remote service, a blog, a project — all are domains. The distinction is in what they bind and how they federate.

### Domain Subclassing

Domains are subclassable by what they contain and what they bind:

- **federation** — a domain devoted to coordinating other domains. Holds references to member domains, manages routing and access rules, acts as the coordination layer.
- **station** — a domain representing a local runtime instance. Describes the physical and logical resources of a machine: cpu, storage, network gateway.
- **project / site** — a domain subscribed to one or more federated networks. Carries its own content and service bindings.
- **journal / blog** — a domain subscribing to federated content networks. Content is just a snapshot stream.

A station is described as a domain graph. For example:

```json
{
  "_": "fs:/",
  "label": "work",
  "dev": {
    "cpu.0": "",
    "fs.0": "sda"
  },
  "gw": {
    "wan.dns": "www.sample.fed",
    "http": "nginx",
    "bind.0": "fs:/home/work/bind"
  }
}
```

`fs:/` is the local filesystem root as a domain address. The station describes its devices, its gateway, and its bind paths — the folders of domain subscriptions it caches locally.

### Subscriptions and Caching

A subscription is a signed directive telling the local station:

- where to mount the domain locally (`fs:/home/{label}`)
- what its non-local gateway address is (the public address requests come from/to)
- who is allowed to create a cache and where (access rules)

A local domain cache with a public gateway means any request for that domain may be answered by the local instance. The distribution system routes based on who has the identity mounted and who can satisfy the request — like any routing mechanism, it needs seed addresses, broadcast rules, and access control.

### VCS as Snapshot Graph

Version control is just a snapshot store. Each ZFS volume is copy-on-write by nature — snapshots are cheap and cryptographically addressed. The VCS structure is a linked list of snapshot hashes:

```json
{
  "_.": "sha256:current",
  "._": "sha256:previous",
  "next.0": "sha256:branch0",
  "next.1": "sha256:branch1",
  "prev.0": "sha256:branch0-parent"
}
```

`_.` is the current snapshot hash. `._` is the previous. `next.0`, `next.1` address branches forward. `prev.0` addresses the branch point backward. Both directions of any branch are addressable.

Sync is requesting domain deltas between snapshot hashes:

```bash
zfs send $current@prev $current@next | curl -X POST http://othernode/fed/$id
```

The cache hash value is the sync primitive. Merge conflict resolution is matching snapshot timelines and presenting the remaining diff — accept all left, reject all right, or arbitrary granularity down to individual nodes or script logic.

### Volume Descriptors

Each ZFS dataset in the graph carries a descriptor:

```json
{
  "label": "/",
  "acs.0": "sha256",
  "gw.pub": "www.example.com",
  "_": "fs:/mount/location",
  "_.": "sha256:current",
  "._": "sha256:previous",
  "gw.bind": "sha256:",
  "bind": {
    "bash": "zfs create $(lookup '_.' $current)"
  },
  "rel": {
    "bash": "zfs send $(lookup '_.' $current)"
  }
}
```

`bind` is how you instantiate the volume locally. `rel` is how you release or sync it to another node. Both are just scripts keyed to the snapshot address.

### Sync Targets

Different content types have natural sync strategies:

```json
{
  "json": "rfc 6902",
  "fs": "rsync",
  "sql": "",
  "lmdb": ""
}
```

JSON patches (RFC 6902) for structured data, rsync for filesystem trees, native replication for databases. The domain carries its own sync strategy as part of its description.

### Station Init

```bash
{
  "0": "export lib='/fed'",
  "1": "mkdir -p ${lib:-.}",
  "2": "truncate -s 10G /fed/..."
}
```

The init sequence is just a domain descriptor — an ordered map of shell expressions. `_.` and `._` as stdin/stdout conventions mean each step is a composable function in the shell environment. The station bootstraps by walking its own descriptor and executing what isn't already present.

### Summary

VZFS treats the filesystem as a distributed computation graph:

- every resource is a domain with a cryptographic address
- every station is a domain describing its local runtime
- subscriptions are signed cache directives, not config files
- VCS is a snapshot graph, addressable in both directions on any branch
- sync is delta query, not replication
- merge is diff presentation, not automatic resolution
- the transport is untrusted, the identity is the trust
