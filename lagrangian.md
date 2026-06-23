# Dynamics in the Namespace

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

The tensor T_uv — still to be derived in this framework — will express how these dynamics couple across different distinctions in the namespace. Basically how the system updates in terms of relative changes up to the 4rth order derivative, because a non-zero fifth order darivative would result in a phase dependancy term in your change potential unit.
