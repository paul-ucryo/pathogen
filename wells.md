# Complex Composition — Working Notes

*working format — not finalized*

---

## E Monad

`E` wraps a function `fn` into an imaginary object. Resolution integrates `fn` over phase `t` in steps of `dt`, accumulating into the real component `r`. The ground state (`t=0`) returns `r` directly.

```javascript
const hbar = 1.0545718e-34
const E = function(fn){ 
    const I = function(r,t,dt=1){
        if (t<=0) return r;
        return I(r+fn(t)*dt,t-dt);
    };
    return function(t){
        return I(0,t);
    };
}
```

`E` is a numerical traversal of what the Lagrangian describes — it traces the path the Lagrangian prescribes for finding the minimization of the path integral between current and end state.

---

## L — The IO Boundary

`L = dV/ds` where: *#pretty sure this is wrong/imprecise.*

- `V` is observable state — what you can measure
- `s` is the system path
- `dV = V(s1) - V(s0)` — change in observable state between two points

`L` is the IO boundary: how adding/removing potentials to different registers changes the system toward a stable state. The IO boundary of any system is the well displacement — the user's input is moving the well's position, not pushing the object.

---

## p(s) and V(s) — The Fundamental Separation

Any system is described by two distinct objects:

- `p(s)` — free particles: their positions and momenta. The system state. Never directly touched by the user.
- `V(s)` — field geometry: the potential landscape. The design surface. What the user manipulates.

User input is always a perturbation to `V(s)`. Particles respond according to the field. Control is indirect by definition. This separation *is* the IO boundary — `V(s)` is the interface, `p(s)` is the interior.

---

## Potential Field Model

The system lives in a potential field. State is a particle in a well.

**Key operations:**

- To move to a new state: shift the well position, not the particle
- Inertia keeps the particle in place; now it sits on a slope in the new well geometry
- The particle rolls to the new minimum on its own
- The user only ever moves wells — never touches the particle directly

**Well geometry defines dynamics:**
- Steep narrow well → fast response, hard stop
- Shallow wide well → slow smooth response
- Asymmetric walls → damping
- Hard wall → latch point (switch)

**Cell position** = position of the well minimum  
**Acceleration/deceleration** = well geometry around the minimum

---

## Wheel and Switch — Example System

A wheel with a position switch.

State variables: `[θ, moving, switch]`

Well geometry: a slope with a hard wall. The switch is the wall — not a separate system. Latching is the particle sitting at the bottom against the hard wall. Stable by geometry, no energy required to hold.

To move the wheel: shift the well. The wheel stays by inertia, now sits on the slope, rolls to the new wall.

**Multi-position traversal (4 cells):**

A staircase potential. Each intermediate cell has a wall with a release mechanism (open switch — weight triggered on collision). The ball hits the wall, wall opens, rolls to next step. Only the target cell has a closed wall — no release. Ball hits and stays. That's the dock latch.

Control: set target cell. All intermediate cells self-clear. Ball rolls to dock and stops. Staircase geometry handles sequencing automatically.

---

## Design Surface

The design artifact is a **2D potential field with N control handles**.

- Wells are draggable
- Well geometry (shape) defines motion profile
- Control handles are the IO boundary — the only thing the user touches
- N-body complexity represented in 2D — holographic principle

This is the holographic principle: every 3D environment composed of a particle field with N degrees of control. The 2D field with N handles *is* the boundary. The 3D system behavior is what you read off it.

---

## General Analytical Framework

This is not just a design tool — it is a general physics map of any information structure.

Given any domain of arbitrary complexity — computational, biological, economic, social — if you can identify `p(s)` (the free agents, the things that move) and `V(s)` (the incentive structure, the environmental constraints), you get the physics for free:

- **Local minima** — stable states the system naturally falls into even if they are not optimal
- **Global minimum** — where the system would go given enough energy, and how deep the barrier is between here and there
- **Catalyst definition** — the minimal perturbation to `V(s)` that destabilizes the current well and lets the system roll to a better configuration
- **Metastability** — shallow wells, one perturbation away from collapse
- **State stability analysis** — stability is well geometry: deep and narrow is robust, shallow is fragile

Any given information structure's state stability is readable directly from the field geometry.

---

## Connection to Physics

- Multibody dynamics solvers (CAD constraint solvers) are executing the Lagrangian's prescription — the iterative search is the path integral being traced
- Physics, information, and language/meaning are the same structure at different levels of description
- The brain is a CAD kernel — continuously solving constraint satisfaction, finding minimum action path between current and target state given the full capability map
- Every model of every problem anyone tries to solve is the same process: Lagrangian minimization with a dynamically chosen ground state

---

## Design Process

Design happens in two stages:

1. **Design the potential field** — draw the problem space as particle physics. Place balls, shape wells, set wall geometry, place switches and triggers. This is the system. It can be drawn on paper or in a visual editor.

2. **Translate to code registers** — each well minimum becomes a cell, each wall becomes a switch or latch, each slope becomes an `E` accumulation with the appropriate forcing function. The code is the field written in registers. Translation is mechanical.

Bugs are mismatches between the field and the code — the code is resolving to a different minimum than the field specifies.

---

## Potential Field Editor — Validating Use Case

A JS-based visual design tool:

- Draw the potential landscape
- Place balls (particles/state objects)
- Shape well geometry by dragging — sets acceleration profiles
- Place switches and triggers as wall features
- Move control handles — these are the IO boundary
- Schema serializes the field → generates register definitions

The editor generates the low-level code. You never write registers directly — you design the physics and the translation is mechanical.

Complexity scales without changing the representation. More balls, more wells, more handles — same 2D surface.

**JS schema sketch:**

```javascript
{
  wells: [
    { id: 'cell_0', position: 0, geometry: { slope: 1.0, wall: 'hard', release: null } },
    { id: 'cell_1', position: 10, geometry: { slope: 1.0, wall: 'hard', release: 'on_contact' } },
    { id: 'cell_target', position: 20, geometry: { slope: 1.0, wall: 'hard', release: null } }
  ],
  balls: [
    { id: 'wheel', position: 0, well: 'cell_0' }
  ],
  handles: [
    { id: 'dock', controls: 'cell_target.position' }
  ]
}
```

*next: implement wheel-switch system in E formalism; build potential field editor as validating use case*
