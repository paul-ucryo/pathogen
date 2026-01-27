# foldr

**foldr** is an experimental framework for treating computation as **difference processing** and programming as **intent decomposition into change vectors**.

Rather than modeling systems as a collection of imperative commands or static state transitions, foldr models systems as fields of **differences**, where behavior emerges from how those differences compose, propagate, and settle under constraints.

The goal is not automation for its own sake, but **legibility and control**: making it explicit how inputs, information, and interaction patterns influence outcomes.

---

## Core Idea

At its core, foldr treats all computation as:

> **Intent → Δ (difference) → composition → normalization → settled state**

- **Intent** is what a user or system wants to achieve.
- **Differences (Δ)** are the only primitive operations.
- **Composition rules** define how differences combine.
- **Constraints** define what outcomes are admissible.
- **Settling** resolves competing differences into a stable configuration.

There are no direct commands.  
Only changes, and rules for how changes interact.

---

## Why Difference-Based Computation?

Traditional systems conflate:
- *what is true*,
- *what is desired*,
- and *how change is applied*.

This leads to opaque behavior, fragile control logic, and unintended feedback loops.

foldr separates these concerns explicitly:

- **State** — what is currently true
- **Reference (Intent)** — what is desired
- **Constraints / Policy** — what is allowed
- **Control** — how differences are applied
- **Energy / Excitation** — how strongly change is driven

This mirrors how physical systems, control systems, and even hardware actually work: only **differences propagate**.

---

## Programming as Intent Decomposition

In foldr, programming is not writing step-by-step procedures.

Instead, programming is:
- identifying the **minimal distinctions** required to express behavior
- defining **typed change vectors** over those distinctions
- specifying **composition laws** and **normal forms**

High-level intent is decomposed into lower-level deltas, which are then resolved by the system.

This allows:
- multiple inputs to coexist without explicit ordering
- graceful handling of conflict and overload
- reversible, inspectable behavior

---

## The Intent Database

A central concept in foldr is the **intent database**.

Rather than treating content, events, or actions as raw data streams, foldr treats them as **intent-bearing signals** that can be cataloged, filtered, and composed.

An intent database:
- stores **explicit representations of desired outcomes**
- associates content and actions with **behavioral effects**
- allows systems to select and filter information based on **intended behavioral impact**, not just relevance or popularity

This makes it possible to ask questions like:
- *What information increases focus rather than engagement?*
- *Which inputs reduce volatility or cognitive load?*
- *How does this content shift behavior over time?*

---

## Behavioral Nudging (Explicit, Measurable, Non-Coercive)

foldr does not attempt to manipulate behavior implicitly.

Instead, it supports **explicit behavioral nudging** with clear metrics:

- nudges are declared as **intent vectors**
- effects are measured as **changes in system state or energy**
- users can inspect, adjust, or disable nudges
- outcomes are evaluated against stated goals, not hidden objectives

This enables:
- personalized information filtering based on desired outcomes
- transparent tradeoffs between engagement, stability, learning, or rest
- experimentation without lock-in or dark patterns

---

## Metrics and Control

Because foldr is difference-first, metrics are not an afterthought.

Typical metrics include:
- energy accumulation and decay
- volatility and oscillation
- convergence time to stable states
- conflict frequency between intents
- option-space expansion or contraction

These metrics allow systems to:
- tune behavior without guesswork
- detect instability early
- make tradeoffs explicit rather than implicit

---

## What foldr Is (and Is Not)

foldr is:
- a **conceptual and architectural framework**
- a way to **author behavior** rather than scripts
- a tool for **reasoning about change, not enforcing outcomes**

foldr is not:
- a replacement for existing languages
- a social network
- an AI agent that decides for you
- a universal solution to coordination

It is a **lens and toolkit** for building systems that are easier to reason about, audit, and adjust.

---

## Design Principles

- Differences are primary; values are derived.
- Intent is explicit and inspectable.
- Constraints are first-class.
- Composition beats sequencing.
- Stability is designed, not assumed.
- Control should be legible and reversible.

---

## Status

foldr is a work in progress.

The current focus is:
- defining a minimal difference algebra
- building a behavior/intent catalog
- experimenting with small, local systems (UI, workflows, coordination tools)

Nothing here is claimed to be final.
The goal is to make the system **understandable before it is powerful**.

---

## Open Questions

- What are the minimal distinctions required to express common behaviors?
- How should intent be represented across domains?
- Which metrics best capture healthy informational intake?
- How do we prevent overfitting or false precision?

These questions are part of the project, not problems to be hidden.

---

## Summary

foldr explores a simple but underused idea:

> If we can represent intent and difference explicitly,  
> we can reason about behavior before it escalates into instability.

The rest is engineering.
