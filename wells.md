# Complex Composition — Working Notes v2

*working format — not finalized*

---

## E — The Integration Function

`E` is a correlator. It takes a correlation function `psi` and integrates it over phase `t`, accumulating into the real component `r`. The ground state (`t=0`) returns `r` directly.

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

`E` is a numerical traversal of what the Lagrangian describes — it traces the path the Lagrangian prescribes for finding the minimization of the path integral between current and end state. For well-behaved `psi`, you could take the Laplacian and solve analytically in a simpler coordinate system. `E` is the numerical fallback for complex geometry.

- `E` → correlator
- `psi` → correlation geometry (the function passed to `E`)

---

## Correlation Functions

Every `psi` is built from the same primitive: `delta` — the binary correlation event. `if`, defined as an operator. The CPU is delta functions composed through `E` at increasing depth.

### delta

The primitive. Pure binary correlation — does this phase match or not. Not a thing in the field, it's the event that defines presence. The atomic write.

```javascript
function delta() { return (t) => t === 1 ? 1 : 0 }
```

No parameter. Carries no value. Just the selector. `E(delta())` gives you a constant — delta is the event, its integral is what persists.

Delta is a fixed address location. A read at a specific phase. The derivative of the step function.

### step

Delta integrated. The write that persists. Zero before the event, `a` from that point forward. A state change at an address that holds.

```javascript
function step(a) { return (t) => t >= 1 ? a : 0 }
```

`E(step(a))` gives you a ramp — `a*t`. Step is the resulting register state after the delta write event. You never see the delta directly; you see its integral held in the register.

- Delta is the transition. Step is what the transition leaves behind.
- Delta is a read at a specific address. Step is a write that holds.

### linear

Step integrated. Constant rate of change — `a` added each phase.

```javascript
function linear(a) { return (t) => a }
```

`E(linear(a))` gives you `a*t`. The slope of a potential well wall.

### quad

Linear integrated. Rate grows with phase.

```javascript
function quad(a) { return (t) => a * t }
```

`E(quad(a))` gives you `a*t²/2`. The curved well geometry — acceleration profile.

### exp

Self-referential growth. Each phase step adds a value proportional to itself.

```javascript
function exp(a) { return (t) => a**t }
```

`E(exp(a))` gives you `a**t / ln(a)`. `e` is the natural base — the specific value where `ln(a) = 1` and the function is exactly its own derivative. The natural unit of self-referential growth, the same way `pi` is the natural unit of rotational change. `pi` is the constant of perpendicular projection; `e` is the constant of parallel projection. Same property, different geometry.

### sine

Oscillation. `a` scales frequency.

```javascript
function sine(a) { return (t) => Math.sin(a * t) }
```

`E(sine(a))` gives you `-cos(a*t)/a`. Resonance, ringing, oscillation around a minimum.

---

## The Hierarchy

Each function is one differentiation above the next. Delta is the primitive; everything else is delta composed through `E` at increasing depth:

```
delta → step → linear → quad → ...
```

The entire potential field is built from binary correlation. The CPU is the same thing — `if` accumulated through phase into every higher level structure.

---

## p(s) and V(s) — The Fundamental Separation

Any system is described by two distinct objects:

- `p(s)` — free particles: positions and momenta. The system state. Never directly touched.
- `V(s)` — field geometry: the potential landscape. The design surface. What the user manipulates.

User input is always a perturbation to `V(s)`. Particles respond according to the field. Control is indirect by definition. This separation *is* the IO boundary.

---

## L — The IO Boundary

*#notation still imprecise*

`L = dV/ds` where `dV = V(s1) - V(s0)` — change in observable state between two points on the path.

`L` is how adding/removing potentials to different registers changes the system toward a stable state. The user's input is moving the well's position, not pushing the particle. The IO boundary is the well displacement.

---

## Potential Field Model

The system lives in a potential field. State is a particle in a well.

**To move to a new state:** shift the well position, not the particle. Inertia keeps the particle in place; now it sits on a slope in the new well geometry and rolls to the new minimum on its own.

**Well geometry defines dynamics:**
- Steep narrow well → fast response, hard stop
- Shallow wide well → slow smooth response  
- Asymmetric walls → damping
- Hard wall → latch point (switch)

**Cell position** = position of the well minimum  
**Acceleration/deceleration** = well geometry around the minimum

---

## Wheel and Switch — Example System

State variables: `[θ, moving, switch]`

Well geometry: a slope with a hard wall. The switch *is* the wall — not a separate system. Latching is the particle sitting at the bottom against the hard wall. Stable by geometry, no energy required to hold.

**Multi-position traversal:**

A staircase potential. Each intermediate cell has a wall with a release mechanism (open switch — triggered on contact). Ball hits wall, wall opens, rolls to next step. Only the target cell has a closed wall. Ball hits and stays. That's the dock latch.

Control: set target cell. Intermediate cells self-clear. Ball rolls to dock and stops. Staircase geometry handles sequencing automatically.

---

## Design Surface

The design artifact is a **2D potential field with N control handles**.

- Wells are draggable
- Well geometry defines motion profile
- Control handles are the IO boundary — the only thing the user touches
- N-body complexity represented in 2D

This is the holographic principle: the 2D field with N handles *is* the boundary. The full system behavior reads off it.

---

## Design Process

1. **Design the potential field** — draw the problem space as particle physics. Place balls, shape wells, set wall geometry, place switches and triggers.

2. **Translate to code registers** — each well minimum becomes a cell, each wall becomes a switch or latch, each slope becomes an `E` accumulation with the appropriate `psi`. Translation is mechanical.

Bugs are mismatches between the field and the code — the code resolves to a different minimum than the field specifies.

---

## General Analytical Framework

Given any domain of arbitrary complexity — computational, biological, economic, social — identify `p(s)` and `V(s)` and you get the physics for free:

- **Local minima** — stable states the system naturally falls into even if suboptimal
- **Global minimum** — deepest well, and the barrier height between here and there
- **Catalyst definition** — minimal perturbation to `V(s)` that destabilizes current well
- **Metastability** — shallow wells, one perturbation from collapse
- **State stability** — deep narrow well is robust; shallow well is fragile

The claim: **problem geometry determines register organization**. You don't design data structures and then solve the problem. The potential field tells you what the registers have to be. Arbitrary complexity in the problem space collapses to equivalent well structures — class equivalence defined by field geometry, not domain.

---

## Potential Field Editor — Validating Use Case

A JS-based visual design tool:

- Draw the potential landscape
- Place balls (particles/state objects)
- Shape well geometry by dragging — sets acceleration profiles
- Place switches and triggers as wall features
- Move control handles — the IO boundary
- Schema serializes the field → generates register definitions

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

---

*next: implement wheel-switch system in E formalism; build potential field editor as validating use case*
