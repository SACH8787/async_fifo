# Parameterized Asynchronous Dual-Clock FIFO (CDC Logic)

A robust, synthesizable **Dual-Clock Asynchronous FIFO** implemented in pure Verilog HDL (IEEE 1364-2001) based on Clifford Cummings' industry-standard architecture. This module facilitates safe, multi-bit Clock Domain Crossing (CDC) between unsynchronized write (`wclk`) and read (`rclk`) clock domains without data corruption or metastability.

---

## Key Features

* **Safe Clock Domain Crossing (CDC):** Converts binary write and read pointers to Gray code ($G = B \oplus (B \gg 1)$) to ensure only 1 bit flips per clock cycle, eliminating multi-bit transition hazards across clock boundaries.
* **Metastability Mitigation:** Implements 2-stage D flip-flop synchronizer chains (`two_ff_sync`) across both clock domain boundaries.
* **Fully Parameterized Architecture:** Configurable data bus width (`DSIZE`) and address bus depth (`ASIZE`), where memory depth is $2^{\text{ASIZE}}$.
* **Dual-Port RAM Memory:** Synchronous write / asynchronous read dual-port memory array (`FIFO_memory`).
* **Precise Flag Logic:**
  * **Empty Flag (`rempty`):** Asserts in the read domain when the next Gray read pointer matches the synchronized write pointer.
  * **Full Flag (`wfull`):** Asserts in the write domain when the top two MSBs of the next Gray write pointer are inverted relative to the synchronized read pointer.

---

## Repository Structure

```text
async_fifo/
├── rtl/
│   ├── FIFO.v             # Top-Level Module Wrapper
│   ├── FIFO_memory.v      # Dual-Port Memory Array
│   ├── rptr_empty.v       # Read Pointer & Empty Flag Module
│   ├── wptr_full.v        # Write Pointer & Full Flag Module
│   └── two_ff_sync.v      # 2-Stage D-FF Synchronizer for CDC
└── tb/
    └── FIFO_tb.v          # Self-Checking Testbench
