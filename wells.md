# Preamble

This is a working model, not a formal theory. The goal is to express a current understanding clearly enough to think with and build on. Gaps and imprecision are known and expected. The question driving it is: how does nature make choice and how do complex structures make 'correct' choice. All points should be argued over and contested. The usefulness of the framing should not wait for the formal definitions. The focus on 'whats the next step' should come from implementing better decision not formalizing physics terms.

Known next steps:

-> Define Y as a container to hold complex values.

# Complex Composition — Working Notes v2

## Core claim for Review: On Selection and Correctness

Let's consider every behavior, even passive observation, is an action. Action can be seen as a selection or choice amongst a set. The following suggests a general strategy for selection meant to minimize prediction error. Effectively a strategy for defining 'correctness' amongst a set of choices.

I would like to ground the selection in physics terminology, so let's consider a *state* as a selection amongst a set. This definition can be argued, but the following useful formulation does not depend on its precision and can defer to future work. As such a dynamic system would be one in which the selection changes amongst the set; the state changes. We want to develop a formal strategy around state changes, prediction. Prediction allows us to consider both expected and observed selection and compare them. 

In this formulation we always start with an inital state. I can't see a way to define a system outside a relation to a state that already exists. This naturally gives us 2 states, the reference or initial condition and observation or final state. Our goal is to be able to produce the final state from the initial condition. So let's say, the initial condition is presumed known and the final state is our unknown.

Define the set S1-S0, the set of states which are different between final and initial state (ds). I would call this set, the behavior. It defines the changes that initial state will undergo. As such we can define our final state:

`S1 = S0+ds`

The addition opperator in this context is not what we think of as scalar addition, ds represents changes to the selection of S0, not sequential adding. In order to indicate that the addition is non-standard, we are going to introduce an opperator on ds that transforms it into coordinates that are coherent to S0. Call this operator `i`. In this context, `i` is selecting which parts of ds should be applied to which parts of S0 (coordinate transform). i selects not just which parts of ds apply to which parts of S0, but resolves them into a common basis — it is the operation that makes future structure legible in present coordinates.

This is common in mathamatics and engineering. It is the function of complex numbers. The real component is what the system is, the current selection. The imaginary component is what it is committed to becoming, held non-locally because the future structure is incoherent in the present. Uncertainty isn't epistemic — it's ontological. The future state is a correlation across the localized pattern.

Observation is resolution of prediction (correctness). Not disturbance, not extraction of a hidden fact. When you observe, the initial selection moves to the future selection. The imaginary component becomes real because the future state has arrived. Collapse isn't mysterious — it's what correct prediction looks like when the future becomes present.

---

## The Lagrangian

The previous section motivated the notation we use in common physics. At this time I'm going to change to more common symbols. We can formalize the gap at a later date. The following does not need the above to be useful.

A physical system has a state. Call the current state V0 and a future state V1. A more generalized version of V may be possible, but we will start with physical states.

V captures everything about the system at a moment — positions, configurations, potentials. In quantum mechanics we stop talking about positions and talk about states directly; positions are just one way to describe a state.

Momentum (T) in this frame is what changed between states: V1 − V0 (our ds, imaginary component). It is a function of velocities — not the state itself, rather how the state is evolving.

In Lagrangian mechanics we define:

**L = T − V**

Which our formulation means:

**L = ds-V0 = (V1 − V0) − V0**

Consider, this is what *didn't* change. You subtract the current state from what changed, leaving the remainder — the selection invariants.

## The Action
Standard physics like to define a quantity *S*, the sum of L as the system evolves. In our framing this is sum of the inverients over an evoloving system. In a a well behaved smooth system this appears as the integral:

**S = ∫ L dt**

This accumulates what didn't change along the entire path from V0 to V1. Why do we call this action? Because it answers the question, where did what I had, get moved to. AKA,the set of actions applied to the invarients.

## What the Principle of Stationary Action Says in This Frame

Traditionally stationary action is an observation, the physical path is the one where S is stationary — where small perturbations to the path don't change the total.

Read this way, this sounds abstract. Read backwards, it's a tautology:

> *The sum over what didn't change remains unchanged*

By working with the set of invariants, you naturally measure the set of maximum change, or rather you measure all of the change in one metric, and the invariants remain stationary.

I would suggest this as a working definition for causality. The shortest path between two states is adjacency.

---

## Differentiation: What the Euler-Lagrange Equation Gives 

Once you have seperation of change from invariants, we can inspect our change structure against the invariant. We look through our change structure for the largest invariant reference applicable to every change. This is our 'bit', the largest object we can use to define any given change.

This is not a chosen coordinate system. Not an externally imposed metric. It is a structure that resolves by what the system actually conserves.

The Euler-Lagrange equation is the differnetial equation describing that mechanism.

---

## The Fundamental Separation

It is useful to decompose your system into two distinct object classes, environment and observables. In modeling terms this is what you know or control and your target. In computer modeling this is functionally input vs output. This sets up your system give you control handles where you have them and readouts on what you need to respond to.


---

## Potential Field Model

Consider a particle in a potential well, where the well is a ramp on one side and a wall on the other.

**To move the particle to a new state:** shift the well position, not the particle. Now the partilcle sits on the top of the slope and rolls to the new minimum on its own. With this simple setup we can expand to complex topologies and optimize automating their resolution.

**Well geometry defines dynamics:**
- Steep narrow well → fast response, hard stop
- Shallow wide well → slow smooth response  
- Asymmetric walls → damping
- Removable wall → latch point (switch)

**Cell position** = position of the well minimum  
**Acceleration/deceleration** = well geometry around the minimum

---

## Example System - Wheel and Switch

Well geometry: a slope with a hard wall. The switch *is* the wall — not a separate system. Latching is the particle sitting at the bottom against the hard wall. Stable by geometry, no energy required to hold.

**Multi-position traversal:**

A staircase potential. Each intermediate cell has a wall with a release mechanism (open switch — triggered on contact). Ball hits wall, wall opens, rolls to next step. Only the target cell has a closed wall. Ball hits and stays. That's the dock latch.

Control: set target cell. Intermediate cells self-clear. Ball rolls to dock and stops. Staircase geometry handles sequencing automatically.

If your state is a wheel, the geometry resets, the user brings you back to the top of the slide, which looks like the potential wells resetting to the starting state. If user input isn't involved, then the wells look like harmonics in a signal.

---

## Design Surface

The design artifact is a **2D potential field with N control handles**.

- Wells are draggable
- Well geometry defines motion profile
- Control handles are the IO boundary — the only thing the user touches
- N-body complexity represented in 2D

Is this the holographic principle: the 2D field with N handles *is* the boundary. The full system behavior reads off it.

---

## Design Process

1. **Design the potential field** — draw the problem space as particle physics. Place balls, shape wells, set wall geometry, place switches and triggers.

2. **Translate to code registers** — each well minimum becomes a cell, each wall becomes a switch or latch, each slope becomes an `E` accumulation with the appropriate `p(t)`. Translation is mechanical (not trivial but well defined).

---

## General Analytical Framework

Given any domain of arbitrary complexity — computational, biological, economic, social — identify `p(s)` and `V(s)` and you get predictive autonomy/decision as a procedure:

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

---

## Digital to Analog Converter

The following is not a computation of physical behavior. It is physical behavior instantiated on a digital substrate.
The distinction matters. A simulation maintains a model and an approximation of reality — there is always a gap between the two. What follows has no such gap. The accumulation structure is the physics running. The system doesn't compute what a physical system would do. It does it.
This is possible because the monad carries the Lagrangian, the Euler-Lagrange condition, and the action integral simultaneously as a single accumulator. In standard analytical field theory these are three separate steps — define L, derive the metric, run the field equations. Numerically, they share the same accumulation structure. The monad runs all three in one pass, updating together as data arrives.
dt is not a parameter you set. It is the natural phase period that emerges as the correlators find coherence — the same way a physical system finds its natural frequency. You don't calculate it. The system settles into it.
E is the correlator. It takes a correlation function p and integrates it over phase, accumulating simultaneously into L, the Euler-Lagrange condition, and S. The system is self-calibrating by construction.
You should be able to measure E as the heat from the cpu. 

## Append: On What The Physics Means

This document is not an interpretation of standard physics. It is a statement of what the standard physics formalism already means, with the accumulated interpretive residue removed. Each equation is not being reframed — it is being read plainly, shut up and calculate without the shut up.

### On L

L = T − V is not a quantity that happens to satisfy a clever variational condition discovered after the fact. Lagrange constructed it specifically so that the arrangement of the problem — separating the rate of change of state from the state itself — would make the constraint structure legible. The equation is not found in nature and then verified. The arrangement *is* the condition. The meaning of L is the operation itself: group what is changing and correlate what it is changing against. What comes out of that separation, when integrated and varied, is not a derived result checked against Newton — it is a grouping strategy.

L is a statement about state geometry. T and V are not energy quantities that happen to have this geometric structure — the geometric structure is what they are. Energy is one instantiation of that structure, the same way 2 grains of rice is one instantiation of 2. The arithmetic doesn't depend on the rice.

### On The Lagrangian Formalism and QM

Quantum mechanics is not hard because it contradicts classical mechanics. It appears hard because people read it against classical mechanics — trying to find the balls and positions underneath. There are none. QM is already a state-only formalism. Energy and momentum are not properties of massive objects — they are geometries of state change, operators on state. The difficulty dissolves when you stop looking for the classical objects underneath and read the formalism as what it actually is: a complete description at the level of state, where positions are one possible description among many, not the ground floor.

### On Non-Locality

The constancy of c — the same in every reference frame regardless of the motion of source or observer — is already a non-local fact, prior to and independent of quantum mechanics or Bell's theorem. A medium gives you a preferred frame; c has none. This means the structure that propagates isn't moving *through* spacetime the way sound moves through air. c is not a speed through a pre-existing container. It is a network update property — a fact about the maximum rate at which one part of the structure can be reflected in another. Spacetime geometry is what you read off the update graph, not the substrate the updates happen inside.

This is speculative and unworked-out, but the observation is: you don't need Bell's theorem to see non-locality. You only need c and what its invariance requires about the structure that has that property.

### On Y

The eigenvalue problem `H*Y = E*Y` is not a technique for extracting special solutions from a system. It is a projection of a self referential problem. The differential equation is the system viewed as its own instantaneous self-reference. The eigenvalue framing is that same system viewed as the set of constraints on the problem projected onto a stable set of values within the problem space.

Y becomes the set of solutions that satisfy the constraint. Not a member of that set selected by some criterion and not a probablitly distribution but the full set of coherent solutions.

For systems simple enough to characterize the solution set by inspection or enumeration, Y can be stated outright — no eigenvalue solve required. The `H(p)` accumulator is the tool for producing Y when direct solution is unavailable. It is not Y itself.

### What Y Looks Like As A Bit Structure

Since Y is a set of solutions, it is a complex bit structure: one concrete solution in the real part, and the set of transforms that reach every other valid solution in the imaginary part.

The clearest model is git. A commit is a fully resolved state — what the repository actually is right now. Other valid states aren't stored as separate full snapshots; they're stored as diffs against a common reference. The full solution space isn't "all snapshots expanded out" — it's one real state plus the transforms that stay within the valid solution space.

Producing Y means: find one valid solution (real part), then characterize the transforms between that solution and the others H admits (imaginary part). H has the constraint geometry. Y lives inside it — one real state plus the branch structure of everything else the constraint permits.

This is precisely what a monad is in catagory theory. It is a structure which correlates change against some future input. It is helpful for me to think of it as a phase shift, moving data to a position until it is needed at some other time. The time parameter is the correlation between the input state and its update.

---

# sandbox

## E = mc²: Rest Energy of a Register

- **hbar** — size of the address space, total nodes
- **G** — network effect of flipping 1 bit
- **m** — number of set bits in the object, node weight
- **c** — propagation rate: the number of network nodes affected by moving one bit in the network
- **c²** — cascade rate: one change affects c nodes, each of which affects c more
- **E** — rest energy: total network effect when this object changes state

## Statement

**E = mc²**

The rest energy of an object in the network is its size times the square of the propagation rate. This is how much of the network has to update when this object changes state — before any dynamics, just from its structural position.

## Interpretation

In a fully connected network, every node relates to every other node. Bit position defines relation; bit value defines weight. One bit update changes the weighted value of that bit's relation to all other bits in the network. c is the rate that one change propagates. c² is the cascade.

Larger objects (more bits) with higher propagation rates (more connections) have higher rest energy — they move more of the network when they change. This is why massive objects in a load-balanced system gravitate toward each other: the update cost of coordinating highly connected objects makes co-location efficient.

## Relation to Address Space

c is not the clock speed — objects in the registers do not observe the clock speed. c is defined by the relational geometry of the address space: how many other bits a given bit update affects. This is invariant across the system, defined by the architecture, not the contents.

---

## Incorrectness

Incorrectness is this frame is not error. It is the gaps in a diffraction pattern.

If your antenna doesn't capture the full field, what you see looks like discontinuity — missing signal and jumps. But those are component parts of a continous signal.The incorrectness is in the antenna, not the receiver. You are measuring a correct process through an incomplete aperture and assuming locality confuses the behavior.

Prediction error is then better seen as accurate prediction on a sub manifold that does not map to desired state. 

---

## Self-Organization as Self Selection

Given three things: a current state V0, a target state V1, and a set of operators defined over your working space — your psi's, the Lagrangian tells you how to evolve most efficiently to your target state. You don't design the path. You don't impose the dynamics. The path falls out from the separation of what changes and what doesn't, and the system finds its own most efficient route through the potential field.

Self-organization isn't a special property of biological or complex systems. It's what any system does when you give it a V1 and let the Lagrangian run. And I would consider that systems which work toward self organization are largely sub optimal paths through the Lagrangian. 

In a design problem, defining psi is not trivial. The operators should form the eigenbasis of the your desired outcome. Get them wrong and the Lagrangian optimizes through a subspace that doesn't cover the real dynamics. Get them right and you have a general engine for directed self-organization irrespective of domain — computational, biological, economic, social up to the resolution/coherence of your invarient.

## H — The Potential Monad

`H` is a correlator. It takes a correlation function `p` and integrates potential position over phase `t`, accumulating constraint to possible positions (r). Real objects (potential/register values) are inserted into the field via E(), encoding the behavior or momentum as p(t=0) for cross correlation against a real value at some later phase t!=0.

```javascript
//const hbar = 1.0545718e-34
//dt=hbar is natural physics?

const H = function(p){ 
    const i = function(r,t,dt){
        if (t<=0) return r;
        return i(r+p(t)*dt,t-dt);
    };
    return function(t){
        return i(0,t);
    };
}
```

- `H` -> Constraint matrix/geometry
- `p` -> potential/constraint
- `Y` -> possible configurations as initial (real) and imaginary component tells you how to iterate through the configurations. This gives you the possible bit patterns that result in constrained behavior. The imaginary component lets you run through different equivalent patterns. I'm not 100% sure that's 100% right.


H*Y = E*Y -> eigenvalue problem

- `m` -> mass is a constraint, these contigous bit values travel together. As a type system, this is constant, where the word size is compressed to minimum set.

range = function (start,end){
    return function (arg){
        if arg<start: return None
        if arg>end: return None
        return arg
    }
}

H(range(0,10))()+H(range(4,6))()

const = function (dt){
    return function(arg){
        return dt
    }
}


xxx = function (dt){
    return function(arg){
        return arg+dt
    }
}

---

### delta

The primitive. Pure binary correlation — does this phase match or not. Not a thing in the field, it's the event that defines presence (observation?).

```javascript
function delta(a) { return (t) => a-t === 1 ? 1 : 0 }
```

`E(delta(a))` gives you a constant — delta is the event, its integral is what persists (step function).

Delta is a fixed address location. A compare at a specific phase (address). 

### step

Delta integrated. The write that persists. Zero before the event, `a` from that point forward. A state change at an address that holds.

```javascript
function step(a) { return (t) => t >= 1 ? a : 0 }
```

`E(step(a))` gives you a ramp — `a*t`. The step is the resulting register state after the delta write event. You never see the delta directly; you see its integral held in the register.

- Delta is the transition. Step is what the transition leaves behind.
- Delta is a compare at a specific address. Step remembers the event, effect of the event which persists across the context.

### linear

Constant rate of change — `a` added each phase.

```javascript
function linear(a) { return (t) => a }
```

`E(linear(a))` gives you curvature `a/2*t**2`. The ramp defines how multiple events accumulate across the phase. 

### quad

Linear integrated gives curvature, how the system responds to events. `E(quad(a))` -> `a*t²/2`. 

Higher order integrations repeat this at different scales.

### exp

Self-referential growth. Patterns where each unit copies itself in the next step.

```javascript
function exp(a) { return (t) => a**t }
```

`E(exp(a))` gives you `a**t / ln(a)` -> the pattern of a pattern that repeats itself, is the pattern repeating itself.

`e` is the natural base — the specific value where `ln(a) = 1` and the function is exactly its own derivative. The natural unit of self-referential growth, the same way `pi` is the natural unit of rotational change. `pi` is the constant of perpendicular projection; `e` is the constant of parallel projection. Same property, different geometry. (right?)

Oscillation or pattern recognition.

```javascript
function sine(a) { return (t) => Math.sin(a * t) }
```

`E(sine(a))` gives you `-cos(a*t)/a`. The pattern of a pattern that identifies itself, is a pattern that identifies itself.

This gives a strong motivation for the euler relation. Exponential is a repeating pattern, and sin is pattern identification. In the euler formula, you are looking at the relationship of a pattern of self identifications. This isn't precisely correct, but I think its close to a real intuitive understanding of e^it = cos(t)+isin(t) and the general relationship between exponentiation and periodic behavior.

---

3: atom trap
: 4x checkpoints
: nasa shroud
