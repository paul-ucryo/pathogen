# Bit-Physics Control Model (Summary)

## Core idea
Program **physics over bits**, then layer **semantics** cleanly on top.

This is a bad description of constraint based programming, with differential type system. but the idea is seperate the process into high level regions (ok, error, warning, idle, processing) and then construct your type system so that you encode behavior in relative position instead of global position. then you cen measure the clock energy needed for any given path. it doesn't need to increase complexity or process, the constraints can reduce to imparitive structures, but the differential type system lets you encode behavior in power spectra for storage, access, and decision.

There is only:
- a bitstream
- a cursor (relative bit position)

Everything else is a *typed interpretation* of bits plus a control loop that lets the system settle.

---

## Types (no magic)
Types are just **bit accounting**.

- Bit = 1
- Byte = 8
- Struct = sum of field sizes
- Array = count × element size
- Enum = tag bits + payload

Types do not store data — they **define how bits are folded**.

---

## Phase
Phase is simply **relative bit offset**.
For nested structures, phase becomes a sequence of offsets (one per fold).

Alignment and grouping cost energy.

---

## Source vs Sink
Source and sink are **orthogonal projections of the same bit geometry**.

### Source
- Active projection
- Low inertia
- Easy to rephase
- Represents *what is being worked on now*

### Sink
- Integrated projection
- High inertia
- Represents *what has settled*

They bind through shared structure, not containment.

---

## Sink.head (the bridge)
`sink.head` is a **typed contract + transaction frame**.

It:
- declares what values it accepts (e.g. up to 4 axis objects)
- binds to source geometry to capture values
- binds elsewhere to negotiate and settle them

When settled, values fall into:
- `sink.ok` (ready / busy)
- or `sink.error`

---

## Behavior = shaping space
Programs do not issue commands.
They **shape the energy landscape**:

- define valid groupings
- enforce constraints (e.g. C xor D)
- bias toward stable configurations

The system moves locally until it settles.

---

## Control loop
1. Bind sink to source and capture values
2. Fold bits and score fit (correlator)
3. Move cursor / regroup bits locally
4. Detect convergence
5. Commit to ok or error

This *is* the runtime.

---

## Why this works
- Big structures move slowly (many bits)
- Small structures move quickly (few bits)
- Stability emerges without freezing
- Debugging reduces to bit counts and offsets

---

## One-line summary
Computation is physics over bits: types are bit-length folds, semantics are named projections, and behavior emerges from a control loop that settles shared bit geometry into stable states.
