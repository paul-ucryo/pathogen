# E = mc²: Rest Energy of a Register

## Definitions

- **m** — number of bits in the object (register size)
- **c** — propagation rate: the number of network nodes affected by one bit update
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
