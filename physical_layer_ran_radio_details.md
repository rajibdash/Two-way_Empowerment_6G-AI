# Physical Layer (Layer 1) Communication Details: RAN & Radio Perspectives

The **Physical Layer (Layer 1 or L1)** in cellular networks (such as 5G NR and 4G LTE) is responsible for converting digital data into electromagnetic radio waves and transmitting them over the air interface between the base station (gNodeB/eNodeB) and the User Equipment (UE).

In modern disaggregated architectures (such as Open RAN / 5G Split 7.2x), Physical Layer responsibilities are divided between the **Distributed Unit (DU)** (handling baseband digital processing) and the **Radio Unit (RU)** (handling low-PHY digital functions, RF conversion, and over-the-air transmission).

---

## 1. End-to-End L1 Signal Chain Overview

```text
[MAC Layer (L2)] 
       │ (Transport Blocks via DL-SCH / UL-SCH)
       ▼
┌─────────────────────────────────────────────────────────┐
│ RAN PERSPECTIVE: Baseband / Distributed Unit (O-DU)     │
│  1. CRC Attachment & Segmentation                       │
│  2. Channel Coding (LDPC / Polar) & Rate Matching       │
│  3. Scrambling & Modulation (QPSK up to 1024-QAM)       │
│  4. Layer Mapping & MIMO Precoding                      │
│  5. Resource Element (RE) Mapping                       │
│  6. OFDM Generation (IFFT + Cyclic Prefix)              │
└──────────────────────────┬──────────────────────────────┘
                           │ (Fronthaul: eCPRI / I/Q Samples)
                           ▼
┌─────────────────────────────────────────────────────────┐
│ RADIO PERSPECTIVE: RF / Radio Unit (O-RU)               │
│  7. Digital Pre-Distortion (DPD) & Crest Factor Red.    │
│  8. Digital-to-Analog Conversion (DAC)                  │
│  9. Frequency Upconversion (IF/Baseband -> RF Carrier)  │
│ 10. RF Power Amplification (PA) & Bandpass Filtering    │
│ 11. Analog Beamforming & Antenna Radiation              │
└──────────────────────────┬──────────────────────────────┘
                           │ (Over-the-Air Propagation)
                           ▼
                   [User Equipment (UE)]
```

---

## 2. The RAN Perspective (Baseband & Protocol Processing)

From the Radio Access Network (RAN) side—specifically within the Baseband Unit (BBU) or Distributed Unit (DU)—the Physical Layer executes deterministic, real-time digital signal processing (DSP) under strict timing budgets.

### 2.1 Transport & Physical Channel Mapping
* **Transport Channels:** Interfaces with the Medium Access Control (MAC) sublayer. Examples include Downlink Shared Channel (`DL-SCH`), Uplink Shared Channel (`UL-SCH`), Broadcast Channel (`BCH`), and Random Access Channel (`RACH`).
* **Physical Channels:** Carry user payload and L1/L2 control signaling over the air:
  * **Downlink:** `PDSCH` (user data), `PDCCH` (scheduling grants/DCI), `PBCH` (Master Information Block).
  * **Uplink:** `PUSCH` (user data), `PUCCH` (HARQ ACKs, CSI feedback, scheduling requests), `PRACH` (initial access preambles).
* **Physical Reference Signals:** Known sequences inserted into the time-frequency grid for channel estimation, synchronization, and beam management:
  * `DMRS` (Demodulation Reference Signal), `CSI-RS` (Channel State Information), `SRS` (Sounding Reference Signal), `PTRS` (Phase Tracking Reference Signal), and `SSB` (Synchronization Signal Block: PSS/SSS).

### 2.2 Error Detection & Forward Error Correction (FEC)
* **CRC Attachment:** Appends Cyclic Redundancy Check bits (e.g., 16-bit or 24-bit) to transport blocks and code blocks so the receiver can verify data integrity and trigger Hybrid Automatic Repeat Request (`HARQ`) retransmissions.
* **Code Block Segmentation:** Splits large transport blocks into smaller code blocks that fit the maximum block size of the channel encoder.
* **Channel Coding:** Adds mathematical redundancy to recover bits corrupted by noise and interference:
  * **LDPC (Low-Density Parity-Check):** Used for high-throughput data channels (`PDSCH`, `PUSCH`) in 5G NR due to low decoding latency at high data rates.
  * **Polar Codes:** Used for control channels (`PDCCH`, `PUCCH`, `PBCH`) in 5G NR due to superior error-correction performance on short block lengths.
  * *(Note: 4G LTE primarily uses Turbo Codes for data and Tail-Biting Convolutional Codes for control.)*

### 2.3 Rate Matching & Scrambling
* **Rate Matching:** Selects, punctures, or repeats coded bits from a circular buffer to match the exact number of bits that can fit into the scheduled physical resource allocation, supporting Incremental Redundancy (`IR-HARQ`).
* **Scrambling:** Multiplies the bit stream with a cell-specific and UE-specific pseudo-random Gold sequence. This breaks up long strings of zeros or ones and randomizes inter-cell interference across neighboring base stations.

### 2.4 Modulation
* Maps groups of scrambled bits into complex-valued IQ modulation symbols ($I + jQ$):
  * **$\pi/2$-BPSK:** 1 bit/symbol (used in uplink DFT-s-OFDM for low Peak-to-Average Power Ratio).
  * **QPSK:** 2 bits/symbol (robust against poor Signal-to-Noise Ratio / cell edge).
  * **16-QAM:** 4 bits/symbol.
  * **64-QAM:** 6 bits/symbol.
  * **256-QAM:** 8 bits/symbol.
  * **1024-QAM:** 10 bits/symbol (requires ultra-high SINR, typically indoor/small-cell environments).

### 2.5 Layer Mapping & MIMO Precoding
* **Layer Mapping:** Distributes modulated symbols across one or more spatial layers (up to 8 layers in downlink SU-MIMO, up to 16 layers in MU-MIMO).
* **Digital Precoding:** Applies a complex precoding matrix to map spatial layers onto logical antenna ports. Uses Channel State Information (`CSI`) or uplink Sounding Reference Signals (`SRS`) to maximize spatial multiplexing or digitally steer beams toward targeted UEs.

### 2.6 Resource Grid & OFDM Waveform Generation
* **Time-Frequency Grid:** Organizes transmissions into Frames (10 ms), Subframes (1 ms), Slots (14 OFDM symbols), Resource Blocks (`RB` = 12 subcarriers), and Resource Elements (`RE` = 1 subcarrier $	imes$ 1 OFDM symbol).
* **Flexible Numerology ($\mu$):** 5G NR supports scalable Subcarrier Spacing (`SCS`): $15 	imes 2^\mu	ext{ kHz}$ (15 kHz, 30 kHz, 60 kHz, 120 kHz, 240 kHz).
* **IFFT & Cyclic Prefix (CP):** Applies an Inverse Fast Fourier Transform to convert frequency-domain Resource Elements into a time-domain OFDM waveform. Prepends a Cyclic Prefix (copy of the end of the symbol) to eliminate Inter-Symbol Interference (`ISI`) caused by multipath delay spread.

---

## 3. The Radio Perspective (RF, Fronthaul & Over-the-Air)

From the Radio Unit (RU) and Radio Frequency (RF) engineering perspective, L1 bridges the gap between discrete digital samples and continuous electromagnetic waves radiating into physical space.

### 3.1 Fronthaul Interface (eCPRI & Functional Splits)
* **Open RAN Split 7.2x:** Separates High-PHY (in the O-DU) from Low-PHY (in the O-RU). In downlink, the O-DU performs up to RE mapping and precoding, then sends frequency-domain I/Q samples to the O-RU. The O-RU performs IFFT, CP addition, and beamforming.
* **eCPRI (Enhanced Common Public Radio Interface):** Transports I/Q user-plane data, real-time control-plane beamforming weights, and synchronization messages over high-speed Ethernet (10G/25G/100G fiber).
* **I/Q Compression:** Uses algorithms like Block Floating Point (`BFP`) to reduce fronthaul bandwidth requirements without degrading Error Vector Magnitude (`EVM`).
* **Synchronization (IEEE 1588v2 PTP & SyncE):** Maintains sub-microsecond phase and time alignment across radios to prevent frame collisions in TDD networks and enable carrier aggregation.

### 3.2 Digital-to-Analog Conversion & Linearization
* **Crest Factor Reduction (`CFR`):** Clips high peaks in the OFDM signal to reduce its Peak-to-Average Power Ratio (`PAPR`).
* **Digital Pre-Distortion (`DPD`):** Applies an inverse non-linear distortion model to the digital baseband signal so that when it passes through the non-linear Power Amplifier, the resulting RF output is linear and clean.
* **DAC / ADC:** High-speed Digital-to-Analog Converters (transmit path) and Analog-to-Digital Converters (receive path) translate between digital sample streams and analog intermediate-frequency (IF) or baseband signals.

### 3.3 Frequency Upconversion & Downconversion
* **Local Oscillators (`LO`) & Mixers:** Mixes the analog baseband/IF signal with a high-frequency carrier generated by a Phase-Locked Loop (`PLL`) synthesizer to shift the signal to the licensed spectrum band:
  * **FR1 (Sub-6 GHz):** Low-band (< 1 GHz, e.g., 700 MHz) and Mid-band (1–7.125 GHz, e.g., 3.5 GHz C-Band).
  * **FR2 (mmWave):** High-band (24.25–71 GHz, e.g., 28 GHz, 39 GHz).
* **Phase Noise Control:** Ensures oscillator stability; excessive phase noise causes Common Phase Error (`CPE`) and Inter-Carrier Interference (`ICI`), especially at high frequencies (mmWave) and high modulation orders (256/1024-QAM).

### 3.4 RF Amplification, Duplexing & Filtering
* **Power Amplifier (`PA`):** Boosts transmit signal power (e.g., 40W to 320W total conducted power for macro cells) using Gallium Nitride (`GaN`) or LDMOS transistors.
* **Low-Noise Amplifier (`LNA`):** Placed right after the antenna on the receive path to amplify weak incoming UE signals while adding minimal thermal noise (low Noise Figure).
* **Duplexing (TDD vs. FDD):**
  * **FDD (Frequency Division Duplex):** Uses simultaneous uplink and downlink on paired spectrum separated by a guard band, relying on duplex filters to isolate Tx from Rx.
  * **TDD (Time Division Duplex):** Shares a single frequency band by rapidly switching high-power RF switches between transmit and receive time slots, requiring strict guard periods (`GP`) to account for propagation delay.
* **RF Filtering:** Cavity or BAW/SAW filters suppress out-of-band spurious emissions and Adjacent Channel Leakage Ratio (`ACLR`) to meet regulatory masks (3GPP / FCC / ETSI).

### 3.5 Antenna Arrays & Analog/Hybrid Beamforming
* **Massive MIMO Arrays:** Uses 32T32R or 64T64R cross-polarized antenna elements integrated into Active Antenna Units (`AAUs`).
* **Beamforming Architectures:**
  * **Digital Beamforming:** Weights applied in baseband per RF chain (flexible multi-beam, high power/cost).
  * **Analog Beamforming:** Phase shifters and variable gain amplifiers applied at RF (common in mmWave FR2 to minimize transceiver chains).
  * **Hybrid Beamforming:** Combines digital baseband precoding with analog RF phase shifting.

### 3.6 Over-the-Air Channel Impairments
Once radiated from the antenna, the RF wave interacts with the physical environment before reaching the UE receiver:
* **Free-Space Path Loss & Shadowing:** Signal attenuation proportional to distance squared and carrier frequency squared ($L \propto d^2 f^2$), plus obstruction losses from buildings, foliage, and atmospheric absorption (oxygen/rain attenuation at mmWave).
* **Multipath Fading & Delay Spread:** Reflections, diffraction, and scattering create multiple delayed copies of the signal arriving at the receiver, causing frequency-selective fading.
* **Doppler Shift & Spread:** Relative motion between the gNodeB and UE shifts the apparent carrier frequency ($f_d = rac{v}{c} f_c$), shrinking channel coherence time and requiring denser `DMRS` symbols for accurate channel tracking.

---

## 4. Summary Comparison: RAN vs. Radio Responsibilities

| Attribute | RAN Perspective (Baseband / O-DU) | Radio Perspective (RF / O-RU) |
| :--- | :--- | :--- |
| **Primary Domain** | Digital bits, modulation symbols, and linear algebra | Electromagnetic waves, analog circuits, and RF power |
| **Core Hardware** | General-purpose CPUs (x86/ARM), inline accelerators (ASIC/FPGA/GPU) | Transceivers (DAC/ADC), GaN Power Amplifiers, LNAs, filters, antenna arrays |
| **Key Functions** | CRC, LDPC/Polar coding, scrambling, QAM mapping, MIMO precoding, scheduling | eCPRI framing, IFFT/FFT, CFR, DPD, frequency mixing, amplification, beamforming |
| **Resource Units** | Subframes, Slots, OFDM Symbols, Resource Blocks (`RB`), Resource Elements (`RE`) | Carrier frequency (MHz/GHz), Bandwidth (MHz), EIRP / Transmit Power (dBm/Watts) |
| **Key Metrics / KPIs** | BLER (Block Error Rate), Throughput (Mbps/Gbps), HARQ RTT, CPU/Accelerator load | EVM (Error Vector Magnitude), ACLR, PA Efficiency, Noise Figure, VSWR |
| **Primary Challenges** | Microsecond real-time deadlines, channel estimation accuracy, MU-MIMO pairing | Thermal dissipation, PA non-linearity, phase noise, TDD switching transients, fading |
