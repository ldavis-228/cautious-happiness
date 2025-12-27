# DAQ_ASIC_Spec_v0

## TinyTapeout Mixed-Signal Data Acquisition ASIC
GENAI
---

## 0. Document Status

* **Revision:** v0 (Frozen Rev A Scope)
* **Process:** sky130A (TinyTapeout)
* **Target Shuttle:** Tiny Tapeout
* **Authoritative Scope:** This document is the single source of truth for Rev A

---

## 1. Project Intent

Design, fabricate, and validate a **TinyTapeout user macro** implementing a **single-channel SAR ADC** with strong **BIST and DFT/debug features**, interfaced via **SPI** to the TinyTapeout demo board (RP2040). The goal is to demonstrate real mixed-signal engineering judgment: architectural tradeoffs, testability, bring-up, and correlation of silicon behavior to models.

Primary success metric: measurable silicon data plus a credible DFT/BIST and bring-up story suitable for design and validation interviews.

---

## 2. TinyTapeout Constraints (Adopted Design Rules)

* **Process:** sky130A
* **Area:** TT user macro tiles (assume 2×2 tiles unless otherwise constrained)
* **Metal usage:** Metal5 **not allowed** (reserved for TT power grid)
* **Analog pads:** ua[0]..ua[2] used (max 3)

  * Path constraints: <500 Ω, <5 pF, max 4 mA
* **Digital I/O:**

  * clk (input), rst_n (input)
  * ui_in[7:0] (inputs), uo_out[7:0] (outputs), uio[7:0] (bidir)
* **I/O pads:** sky130_ef_io_gpiov2_pad

  * Max output toggle ~33 MHz
  * Max input freq ~66 MHz
  * IO supply from demo board: 3.3 V (not 5 V tolerant)
* **Clock source:** TT demo board (RP2040), 1 Hz – 66.5 MHz

---

## 3. Frozen Rev A Scope

### In Scope

* **ADC:** 8-bit SAR, single channel
* **Max sample rate:** 200 kS/s
* **Input:** single-ended VIN
* **Reference:** external VREF pin (on-chip buffer optional)
* **Interface:** SPI (Mode 0)
* **DFT:** debug registers, single-step modes, analog test mux
* **BIST:** digital signature + sanity checks + comparator/DAC toggle check
* **Deliverables:** silicon bring-up, measured performance, characterization plots

### Explicitly Out of Scope (Rev A)

* Multi-channel ADC
* Integrated precision bandgap reference
* High-speed serial interfaces (USB, Ethernet, PCIe)
* Full IEEE JTAG boundary scan
* On-chip precision stimulus DAC

---

## 4. Pinout (Frozen)

### 4.1 Analog Pads

* **ua[0] = VIN** (ADC analog input)
* **ua[1] = VREF** (external reference input)
* **ua[2] = ATEST_OUT** (analog test output from internal mux)

### 4.2 Digital Interface (SPI)

* **ui_in[0] = SPI_SCLK**
* **ui_in[1] = SPI_CSN** (active low)
* **ui_in[2] = SPI_MOSI**
* **uo_out[0] = SPI_MISO**

### 4.3 Optional Status Outputs

* **uo_out[1] = DRDY** (data ready / FIFO not empty)
* **uo_out[2] = BUSY** (conversion in progress)
* **uo_out[3] = FAULT** (OR of sticky fault flags)

### 4.4 Clock / Reset

* **clk**: system clock input
* **rst_n**: active-low reset

---

## 5. Performance Requirements

### 5.1 ADC Core

* Architecture: SAR ADC
* Resolution: 8 bits
* Input range: 0 to VREF
* Max sample rate: 200 kS/s
* Conversion latency: deterministic (programmable clock divider)

### 5.2 Engineering Targets (Non-Promissory)

* Monotonic transfer characteristic under nominal conditions
* Goal: no missing codes (typical)
* Target ENOB: ~7 bits @ 50–100 kS/s

---

## 6. Architecture Overview

### 6.1 Analog Blocks

* Sampling switch network
* Binary-weighted capacitive DAC (Rev A)
* Dynamic comparator (latch-based or equivalent)
* Minimal bias circuitry

### 6.2 Digital Blocks

* SAR control FSM
* Result register and optional small FIFO (8–32 samples)
* SPI slave and register file
* BIST engine (CRC/LFSR + checks)
* Fault monitor block

### 6.3 ATEST Mux

Internal analog mux routes selected nodes to ua[2]. Minimum required selections:

1. VREF (or buffered VREF)
2. DAC top-plate node (tapped carefully)
3. Comparator input node (high-Z tap)
4. Bias/common-mode node (if present)

---

## 7. SPI Protocol (Frozen)

* Mode: SPI Mode 0 (CPOL=0, CPHA=0)
* Max SPI clock: 20 MHz
* MISO driven only when CSN low

### 7.1 Command Format

* Byte 0: CMD (R/W, auto-increment)
* Byte 1: Register address
* Bytes 2+: Data or dummy bytes for readback

---

## 8. Register Map (Functional Requirements)

### Identification

* **ID_REV**: chip ID and revision

### Control

* **MODE**: ADC_EN, SINGLE_START, CONTINUOUS_EN, TEST_MODE
* **CLK_DIV**: internal timing divider

### Status / Data

* **STATUS**: DRDY, BUSY, FIFO_LEVEL, FAULT_SUMMARY
* **DATA**: last conversion result
* **FIFO_DATA**: read pops next sample (if FIFO enabled)

### BIST / DFT

* **BIST_CTRL**: START, MODE, LENGTH
* **BIST_STATUS**: DONE, PASS_FAIL, FAIL_CODE
* **BIST_SIG**: CRC/LFSR signature
* **FAULT_FLAGS**: TIMEOUT, STUCK_CODE, RANGE_FAIL (sticky)
* **ATEST_SEL**: selects analog node for ua[2]
* **DEBUG**: SAR state index, last comparator decision bit

---

## 9. BIST Requirements

### 9.1 Signature BIST (Mandatory)

* Acquire N samples
* Compute CRC-16/32 or LFSR signature
* Report signature and DONE flag

### 9.2 Sanity Checks (Mandatory)

* Stuck-code detection
* Min/max range check
* Conversion timeout watchdog

### 9.3 Comparator/DAC Toggle Check (Recommended)

* Force DAC to known patterns
* Verify comparator toggles at least once
* Report fault if not observed

---

## 10. DFT / Debug Features

* Single-step SAR conversion mode
* Readable internal SAR state via DEBUG register
* Readable comparator decision bit
* ATEST analog observability via ua[2]
* Deterministic reset; no latches; scan-friendly RTL style

---

## 11. Upgrade Path (Rev B / Rev A+)

* Segmented capacitor DAC
* Bootstrapped sampling switch
* On-chip VREF buffer or bandgap
* Digital calibration/trimming
* Additional channels (if more analog pads purchased)

---

## 12. Acceptance Criteria

Rev A is complete when:

* SPI read/write and burst read operate correctly
* Single-shot and continuous ADC modes function
* BIST modes run and produce repeatable results
* ATEST mux routes multiple internal nodes to ua[2]
* Measured silicon data is produced:

  * Transfer curve and histogram
  * Coarse INL/DNL
  * FFT/SNR at one tone
  * Power vs sample rate

---

## 13. Companion PCB (Out-of-ASIC Scope)

A TinyTapeout companion board shall provide:

* VIN conditioning (SMA, selectable RC, selectable source impedance)
* Precision VREF generation and filtering
* ATEST breakout to SMA/testpoint
* Optional external DAC stimulus for ramp generation

---

**End of DAQ_ASIC_Spec_v0**
