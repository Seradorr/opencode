---
name: vitis
description: Apply Xilinx Vitis and bare-metal rules for BSPs, DMA, interrupts, cache, and PS-PL coordination.
---

# Vitis

## When to use me

Use this skill for:

- Xilinx Vitis bare-metal C or C++
- BSP and driver integration
- DMA setup and ring/buffer handling
- interrupt controller and ISR work
- cache coherency and PS-PL interaction

## What I enforce

- standard Xilinx init and error-return patterns
- explicit cache flush and invalidate handling around DMA
- short ISRs and deferred processing when needed
- consistent device-ID and base-address handling
- careful linker, memory-map, and MMIO assumptions

## Critical checks

- verify cache coherency every time DMA touches shared buffers
- check interrupt enable, connect, and acknowledge order
- avoid widening scope into linker-script or startup changes unless required
