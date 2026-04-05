---
name: c-systems
description: Guide generic C and C++ systems work with strong rules for memory, errors, interfaces, and low-level correctness.
---

# C Systems

## When to use me

Use this skill for:

- generic C or C++ systems code
- embedded-adjacent code that is not specifically Vitis/BSP work
- buffers, parsers, protocols, drivers, and bit-level logic
- portability, ABI, layout, endianness, and ownership concerns

## What I enforce

- explicit ownership and lifetime rules
- checked return values and meaningful error paths
- bounded buffer handling and no silent truncation
- careful use of `volatile`, atomics, and interrupt-shared state
- clear separation of public headers and private implementation details

## Critical checks

- verify integer widths and signedness at hardware or protocol boundaries
- avoid hidden allocations in hot paths or ISR-adjacent code
- prefer fixed-width types for wire formats and MMIO
- document alignment, packing, and endian assumptions when they matter
