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
