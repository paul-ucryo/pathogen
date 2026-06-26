# Complex Composition — Working Notes v2

## Correctness

Physics is the study of what remains correct.

A system has a current state and a future tendency. The current state is real — localized, present, bound to existing structure. The future tendency is imaginary — distributed across the whole system as potential, because the structure it refers to doesn't exist in the present projection. You can't localize a future state on objects that will look different later. Locality is a property of the present configuration.

This is not a mathematical trick. It's what complex numbers actually are. The real component is what the system is. The imaginary component is what it is committed to becoming, held non-locally because the future structure is incoherent in the present. Uncertainty isn't epistemic — it's ontological. The future state isn't there to be localized into.

The Schrödinger equation tracks the invariant across that evolution. It enforces the relationship between current state and future tendency — what must remain consistent as the imaginary component unfolds. It is the equation of non-change, written in the language of state evolution.

Observation is correctness. Not disturbance, not extraction of a hidden fact. When you observe, you provide the structure the future tendency needed to resolve into. The imaginary component becomes real because the future state has arrived. Collapse isn't mysterious — it's what correctness looks like when the future becomes present.

The measurement problem was a category error. Observation was treated as something external interrupting the system. It isn't. It's internal to the structure — the moment distributed potential localizes because locality is now available.

This reframes everything that follows.

---

## The Lagrangian

A physical system has a state. Call the current state V0 and a future state V1.

V captures everything about the system at a moment — positions, configurations, potentials. In quantum mechanics we stop talking about positions and talk about states directly; positions are just one way to describe a state.

T is what changed between states: V1 − V0. It is a function of velocities — not the state itself, but how the state is moving.

## The Lagrangian

The Lagrangian is:

**L = T − V = (V1 − V0) − V0**

This is what *didn't* change. You subtract the current state from what changed, leaving the remainder — the part of the transition that is conserved.

## The Action

The action S is the integral of L over time:

**S = ∫ L dt**

This accumulates what didn't change along the entire path from V0 to V1 as action, the new location of what didn't change.

## What the Principle of Stationary Action Actually Says

The physical path is the one where S is stationary — where small perturbations to the path don't change the total.

Read forwards, this sounds abstract. Read backwards, it's a tautology:

> *The sum of what didn't change is what didn't change.*

The system takes the path of maximum change — the most direct route through state space — and so the total of what is conserved along that path is stationary. Any other path would involve the system being somewhere it wasn't.

This is causality. The shortest path between two states is adjacency.

## Why It's Coordinate Independent

The Lagrangian rotates the system into a conserved coordinate frame — the frame where what remains constant is explicit. You solve for what must not change, then rotate back to find what did. Because the conserved frame is the working frame, the coordinates you started in don't matter.

This is not a coincidence. It is built into the structure of L = T − V.

---

## What the Euler-Lagrange Equation Does

The Euler-Lagrange equation defines the coordinate metric produced by the Lagrangian. 

By seperating what doesn't change and what does, then writing a differential equation to select for the largest 'bit' that remains invariant you are finding the largest invariant in any arbitrary encodable system as defined by its own behavior.

This is not a chosen coordinate system. Not an externally imposed metric. It is a structure that resolves by what the system actually conserves.

## The Metric Falls Out

The relationship between what changes and what doesn't defines your coordinate system. The metric of the system is just the collection of all its invariants — everything that must remain the same as the system evolves.

You are not choosing coordinates and imposing them on the system. You are letting the system tell you what its natural coordinates are, based purely on separating what does and doesn't change.

## Why This Is General

This is why the framework works for any system, in any coordinates. You are not doing physics in a particular frame — you are rotating into the frame of no change, solving there, and rotating back. The metric that results is the largest thing that remains invariant under the system's evolution.

*you keep repeating that. clean it up*

The Euler-Lagrange equation is the machinery that makes that separation exact.

---

## Incorrectness

Incorrectness is not error. It is the gaps in a diffraction pattern.

If your antenna doesn't capture the full field, what you see looks like noise — missing signal, wrong values, incoherent output. But the field is complete. The incorrectness is in the receiver, not the system. You are measuring a correct process through an incomplete aperture.

This reframes debugging and error correction entirely. You are not fixing a broken system. You are extending the antenna — expanding the set of psi's until the aperture is wide enough to capture what the field is actually doing. The gaps close not because you corrected the error but because you finished the process.

Incorrectness is correctness distributed on an unfinished process.

---

## Self-Organization as a General Algorithm

This gives you a system for self-organization in any arbitrary distributed system.

You need three things: a current state V0, a target state V1, and a set of operators defined over your working space — your psi's. The Lagrangian then tells you how to evolve most efficiently. You don't design the path. You don't impose the dynamics. The path falls out from the separation of what changes and what doesn't, and the system finds its own most efficient route through the potential field.

Self-organization isn't a special property of biological or complex systems. It's what any system does when you give it a V1 and let the Lagrangian run. You're not designing behavior — you're defining the space and the target, and correctness does the rest.

The hard part is defining the right psi's. The operators have to actually span your working space. Get them wrong and the Lagrangian optimizes through a subspace that doesn't cover the real dynamics. Get them right and you have a general engine for directed self-organization in any encodable system — computational, biological, economic, social.

The E formalism below is the machinery that makes this concrete.

---

## E — The Integration Function

`E` is a correlator. It takes a correlation function `psi` and integrates it over phase `t`, accumulating into the real component `r`. Real objects (energy/reg values) are inserted into the field via E(), which adds them as pure imaginary, phase t=0.

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

- `E` → correlator
- `psi` → correlation geometry (the function passed to `E`)

---

## Correlation Functions

Every `psi` is built from the same primitive: `delta` — the binary correlation event. `if`, defined as an operator. The CPU is delta functions composed through `E` at increasing depth.

### delta

The primitive. Pure binary correlation — does this phase match or not. Not a thing in the field, it's the event that defines presence. The atomic write.

```javascript
function delta(a) { return (t) => a-t === 1 ? 1 : 0 }
```

Just the selector. `E(delta(a))` gives you a constant — delta is the event, its integral is what persists.

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

## p(s) and V(s) — The Fundamental Separation

It is useful to decompose your system into two distinct object classes, particles (p) and enviroment (V). The difference is functional and largely defined by stability. Their seperation is notional, momentum largely is an accounting structure tied to having made that seperation.

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
