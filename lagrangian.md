# Dynamics in the Namespace
This is wrong, but points in the right direction. I can almost see, you can define the system constraints as how you want your io state to vary d/dt*L(t). So you can define a namespace
name = particle/behavior
that value is read out by evaluating E(t), t is sort of the address of the register holding E.
Then you can define how L is allowed to change and how E(t0) effects E(t1). This lets you define how the registers will compose when the system runs.

## Core Objects

- **V(s)** — the value at an address, as a function of system state s. This is potential, system state phase shifted into an address relative to reference (s).
- **L** — the raw system change. s1 - s0. Think of this as the io boundary.
- **T** — the internal dynamics of V. How V changes given L (io change).

```
dV/ds = L + V
```

A given register updates by accumulating the io (L) on top of its current value V. The register has memory — it carries itself forward.

The Lagrangian:

```
L = dV/ds - V
```

The difference between how V is changing and what V is. Dynamics minus state is the io boundary.

---

## Kinetic and Potential

- **Potential (V)** — A register is observer state phase shifted to a different spacetime coordinate. You never observe potential directly — observation is itself a phase shift, so you only ever see the derivative.
  
- **Kinetic (T)** — the internal dynamics of the potential. How V behaves in its own local coordinates. Not the observer's frame — V's own frame. The imaginary component: how the behavior of V defines its future state.

L = T - V is the observer's own rate of change. The Lagrangian is the observer's dynamics.

---

## Stationary Action

Given two states s0 and s1, we want the function V(s) that takes the shortest path between them.

The shortest path between 2 states is only a straight line in euclidean geometry. The path is only shortest when your metric is constant throughout the state space. Your bit doesn't change meaning/weight depending on where you are in state space. No step does more or less work than any other.

**Stationary condition**: δS = 0, where S = ∫ L ds.

This is not df/dx = 0 (pointwise). It is optimization over the space of functions — which function V(s) minimizes the total accumulated L along the path?

The path is the shortest path between s0 and s1 if and only if each incremental step is uniform — the update is consistent across the whole path.

The **Euler-Lagrange equation** falls out of this condition:

```
∂L/∂V - d/ds(∂L/∂(dV/ds)) = 0
```

This is the equation of motion for V — the rule that governs how V evolves along the stationary path.

---

## What This Means for the Namespace

Every dynamic in the namespace is a choice of which path to take through state space. The stationary condition says: the valid paths are the ones where each step is consistent — no bit changes meaning mid-path.

The Lagrangian L = dV/ds - V is the bookkeeping: what changed (dV/ds), what was already there (V), and the difference is what the system is actually doing.

# State, Phase, and the IO Tensor

## First Principles

### The Register

The smallest object in the system is a word — a fixed-width value at a memory address. Call it:

```
E(t)
```

Where `t` is phase — not time, not sequence, but the coordinate that offsets the observer state. The register holds a potential: the stored capacity to cause an effect on the system.

### System State

The system state is:

```
S(t) = S(t=0) + E(t)
```

The current state is the reference state plus whatever the register holds at phase t. The register is the phase-shifted displacement from the reference.

### The IO Boundary

```
L(t) = S(t) - S(t=0) = E(t)
```

L(t) is the accumulated difference — the io boundary. It is not computed and not stored. It resolves as the system evolves. L(t) is the desired io effect at phase t: what the system should produce when it reaches that phase.

The register E(t) holds the potential to cause L(t). The register is the cause; L is the effect, not yet resolved.

---

## Phase is Not Sequence

t is not a causal chain. It is a phase offset. The system can hold multiple states at different phases simultaneously:

```
E(t0) = ''
E(t1) = 'happy birthday'
E(t2) = 'hello world'
```

And the io boundary can project any phase onto any effect:

```
L(t0) = E(t0)
L(t1) = E(t2)
L(t2) = E(t0)
L(t3) = E(t1)
```

L(t) is not a readout of E(t) — it is a projection. L picks out register values at different phase offsets and resolves them as effects. The system does not execute sequentially; it resolves projections.

---

## The Taylor Expansion of E

A register value E can be expanded in phase t:

```
E(t) = E₀ + E₁t + E₂t²/2! + E₃t³/3! + E₄t⁴/4!
```

Split into coefficients and basis:

```
A = [E₀, dE/dt, d²E/dt², d³E/dt³, d⁴E/dt⁴]   — coefficient vector
X = [1, t, t²/2!, t³/3!, t⁴/4!]ᵀ               — phase basis
```

```
E(t) = A · X
```

The 4th derivative d⁴E/dt⁴ is a constant — it does not vary with phase. The first three non-trivial derivatives vary with t: three independent pattern spaces. The 4th is a fixed proportion. A 5th order term would require the bit weight itself to vary with phase — which is outside the word. The word bounds the expansion at 4th order.

---

## The IO Tensor T_uv

E(t) is distance from the current state to an address. L(t) is defined as constraints that move addresses in phase such that L(t=now) resolves to the desired state given E(t).

T_uv is the cross-correlation between phase offsets of E and phase offsets of L — how the register value at phase u projects onto the io effect at phase v.

```
T_uv = projection of E(u) onto L(v)
```

T_uv is not a computation. It is the topology of the projections — the map of which phase offsets resolve to which io effects. It tells you, for any given scalar energy value E, how the expansion changes across correlation projections.

The tensor has 3 independent pattern spaces (orders 1–3, which vary with phase) and 1 constant proportion (order 4). This gives the 4x4 structure of T_uv.

---

## What This Means for the Namespace

Every register in the namespace is an E(t) — a potential at a phase offset.

Every distinction in the namespace is a phase coordinate where the system needs to resolve a specific io effect.

The dynamics of the namespace are defined by T_uv — the map of projections between phase offsets. You do not define behavior sequentially. You define which phases project onto which effects, and the tensor holds that topology.

**Adding a distinction** = adding a phase coordinate and defining its projection in T_uv.

**Relating two distinctions** = defining a non-zero entry in T_uv between their phase coordinates.

**Invalid states** = phase coordinates with no defined projection — they exist in the address space but resolve to nothing.

The namespace is the topology of T_uv. The UI is its rendering.

The tensor T_uv — still to be derived in this framework — will express how these dynamics couple across different distinctions in the namespace. Basically how the system updates in terms of relative changes up to the 4rth order derivative, because a non-zero fifth order darivative would result in a phase dependancy term in your change potential unit.
