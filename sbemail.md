# UI as Polynomial State Composition

## The Model

Every state `s` is a word (register-width value) at some address.

```
s = r * e^(i*t)
```

- `r` — resolved/settled state (amplitude)
- `t` — active/functional bits (phase)
- `i` — the CPU: fetch-decode-execute, the operator that transforms t into r

All evolution is phase change. The CPU executes instructions; everything that happens is a change in the bit pattern.

---

## Composition

The smallest addressable unit is a word (64-bit on modern systems).

Two adjacent words compose by positional notation — the same way counting works in binary:

```
s0 at address n
s1 at address n + w^1   (w = word size)
s2 at address n + w^2
...
```

Any composite state is a polynomial:

```
S = r0*e^(it0) + r1*e^(it1)*w + r2*e^(it2)*w^2 + ... + rn*e^(itn)*w^n
```

**A particle, an object, a behavior — anything — is just a polynomial in w.**

Two particles relate by adding their polynomials. Relations between existing particles are not new particles — they are composition within the existing polynomial structure.

---

## Namespace

A namespace is a set of polynomials (particles) that always behave the same way regardless of context. Behavior is intrinsic to the polynomial — it composes with whatever it touches the way polynomials compose.

Each name in the namespace is a **distinction**: a state the system needs to be able to represent.

**Rules:**
- Add a distinction only when the system requires a new state
- If two things relate, they are not new distinctions — they are relations inside the existing structure
- The namespace accretes from need; it is never planned in advance

---

## Designing a UI

1. **Identify the target states** the user needs to be in
2. **Add distinctions** as the system requires new states
3. **Map constraints** — ask "what breaks the system?" — the answer defines which transitions are invalid
4. **The topology of valid transitions is the design**

The UI is navigation through the state space. Layout, interaction, and components all fall out of the topology.

**Invalid states do not exist in the address space.** If the polynomial cannot represent a state, it cannot be reached. Errors are impossible by construction, not by validation.

---

## Example: Detector Test Instrument

State space:
- `n` detector chips
- `m` filters  
- `l` apertures
- `connected ∈ {0, 1}` — whether a chip is plugged in

Constraint: changing chips while connected breaks the system.

Resolution: zero all states between chip_n and chip_n+1 where `connected = 1`. The invalid state (two chips simultaneously in connected state) has no address — it cannot be represented.

The constraint is a clear definition in the namespace, not a runtime check.

---

## Distributed Systems

A distributed resource extends the polynomial — it adds new terms (new addresses on remote machines). It only changes the design if it:

- Introduces new distinctions (new valid states), or
- Constrains existing transitions (new invalid states)

Otherwise it is an implementation detail; the topology is unchanged.

**Distributed coordination** reduces to: send the addresses you do or don't have at your local station for a given target state. Merge is union or intersection of address spaces depending on whether the resource adds or constrains. CRDT-style — reconciling which addresses exist, not which values win.
