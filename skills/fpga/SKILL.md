---
name: fpga
description: Apply FPGA RTL, Vivado, timing, CDC, AXI, and constraint discipline for HDL tasks.
---

# FPGA

## When to use me

Use this skill for:

- VHDL, Verilog, or SystemVerilog
- Vivado synthesis and simulation issues
- timing closure and pipelining
- CDC analysis
- AXI or stream handshakes
- XDC and top-level pin or clock work

## What I enforce

- synchronous, explicit RTL over accidental latches
- consistent reset polarity and clock naming
- separate design files from testbenches
- defaults in combinational logic and complete state handling
- timing-aware structure for wide arithmetic or deep control paths

## Critical checks

- confirm before changing top-level ports, clock trees, reset polarity, or XDC constraints
- verify domain crossings instead of assuming a shared clock
- document packet, lane, or handshake assumptions at boundaries
