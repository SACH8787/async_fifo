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

async_fifo/
├── rtl/
│   ├── FIFO.v             # Top-Level Module Wrapper
│   ├── FIFO_memory.v      # Dual-Port Memory Array
│   ├── rptr_empty.v       # Read Pointer & Empty Flag Module
│   ├── wptr_full.v        # Write Pointer & Full Flag Module
│   └── two_ff_sync.v      # 2-Stage D-FF Synchronizer for CDC
└── tb/
└── FIFO_tb.v          # Self-Checking Testbench

## Mathematical & CDC Logic

### 1. Gray Code Conversion
Binary counters change multiple bits simultaneously (e.g., `011` to `100` changes 3 bits). To prevent sampling intermediate illegal states across asynchronous clocks, pointers are converted to Gray code prior to synchronization:

$$G[n] = B[n] \oplus B[n+1]$$

### 2. Empty Condition Logic
The FIFO is empty when the next Gray-coded read pointer matches the synchronized write pointer in the read domain:

$$\text{rempty\_val} = (\text{rgray\_next} == \text{rq2\_wptr})$$

### 3. Full Condition Logic
The FIFO is full when the write pointer catches up to the read pointer after wrapping around one full lap ($2^{\text{ASIZE}}$ words ahead). In Gray code, this condition corresponds to the top two MSBs being inverted while all lower bits match:

$$\text{wfull\_val} = (\text{wgray\_next} == \{\sim\text{wq2\_rptr}[\text{MSB}:\text{MSB}-1], \text{wq2\_rptr}[\text{MSB}-2:0]\})$$

---

## How to Run Simulation in Xilinx Vivado

1. Open **Xilinx Vivado** and create a new project named `async_fifo`.
2. Select **RTL Project** and choose target FPGA device (e.g., `xc7a35tcpg236-1`).
3. Add all Verilog design files from the `rtl/` directory (`FIFO.v`, `FIFO_memory.v`, `rptr_empty.v`, `wptr_full.v`, `two_ff_sync.v`).
4. Add the simulation testbench from the `tb/` directory (`FIFO_tb.v`).
5. Set `FIFO_tb` as the top-level testbench module.
6. Click **Run Simulation** $\rightarrow$ **Run Behavioral Simulation**.

---

## Verification Strategy

The testbench (`FIFO_tb.v`) verifies the design across three operational test cases:
1. **Basic Data Transfer:** Writes random data bytes at 100 MHz (`wclk`) and reads them back at 50 MHz (`rclk`).
2. **Overflow Protection (Full State):** Drives continuous write cycles past FIFO capacity ($\text{Depth} + 3$ words) to confirm `wfull` asserts and prevents memory corruption.
3. **Underflow Protection (Empty State):** Executes continuous read cycles on an empty FIFO to confirm `rempty` asserts and blocks invalid reads.

---

## Author

**Sachin Mohanty**  
B.Tech in Electronics and Communication Engineering  
*Indian Institute of Information Technology, Guwahati (IIITG)*  
* GitHub: [@SACH8787](https://github.com/SACH8787)
