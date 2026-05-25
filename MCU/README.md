# Virtualization on MCU-Based Systems

## Why Consolidate onto Fewer MCUs?

The traditional federated architecture in safety-critical domains (automotive, avionics, industrial)
has historically led to a proliferation of microcontrollers: each function runs on dedicated hardware,
with its own RTOS and firmware. This proliferation is motivated by organizational reasons
(separation of responsibilities among suppliers) and technical ones (fault isolation, certification
simplicity) [Obermaisser et al., 2009; Di Natale & Sangiovanni-Vincentelli, 2010].

However, as functional complexity grows, so does the number of microcontrollers — and with it
the complexity of interconnections (CAN, SPI, IIC) among them. This leads to increasing costs,
weight, reliability issues, and difficulty in orchestrating development across systems [Li et al., 2025;
Pan & Parmer, 2022]. The natural response is **consolidation**: reducing the number of physical
microcontrollers by aggregating multiple applications onto the same hardware.

## Why Does Consolidation Happen on MCUs (Without MMU)?

The key question is: why not migrate to more powerful processors equipped with an MMU, where
virtualization is simpler and more mature?

The answer is twofold.

### 1. Legacy Software Is Already Developed for MCUs

Safety-critical applications have traditionally run on microcontrollers — systems with an MPU
instead of an MMU, lightweight RTOSes, certified compilers, and proprietary toolchains. This is
not accidental: microcontrollers offer superior temporal determinism compared to general-purpose
processors, since the absence of an MMU eliminates the variability introduced by TLB misses.
On an MMU-based system, a single load instruction may cost 6 cycles (TLB hit) or over 400 cycles
(TLB miss), with eviction policies that are often opaque and unpredictable — making it difficult
to demonstrate tight WCET bounds [Pan & Parmer, 2019].

### 2. Changing Platform Is Prohibitive

Migrating legacy software from MCUs to MMU-based processors entails:

- **Loss of existing certifications**: rewriting the code invalidates previously obtained safety
  certifications (e.g., ISO 26262), requiring a new and costly certification process [Li et al., 2025].
- **RTOS and framework incompatibility**: legacy firmware depends on MCU-specific RTOSes and
  certified toolchains (e.g., IAR) that assume a bare-metal environment. These cannot be directly
  ported to MMU-based architectures [Li et al., 2025].
- **Hardware certification constraints**: ASIL-D functions require hardware with lockstep mode and
  specific certification. At the time of the relevant publications, no MMU-based processor certified
  for ASIL-D was available [Rajan et al., 2018; Kohn et al., 2017].

### Summary

Consolidation therefore happens on MCUs because **the software already exists on that platform**
and **cannot simply be moved elsewhere** without unsustainable costs and risks. Virtualization on
MCUs arises precisely to address this need: aggregating multiple legacy firmware instances onto the
same physical microcontroller, while preserving spatial and temporal isolation among them, without
requiring code rewrites or platform changes [Li et al., 2025; Rajan et al., 2018; Kohn et al., 2017].

---

## References

- **Li et al., 2025** — *FVM: Practical Feather-Weight Virtualization on Commodity Microcontrollers*, IEEE Transactions on Computers.
- **Pan & Parmer, 2022** — *SBIs: Application Access to Safe, Baremetal Interrupt Latencies*.
- **Pan & Parmer, 2019** — *MxU: Towards Predictable, Flexible, and Efficient Memory Access Control for the Secure IoT*, ACM TECS.
- **Rajan et al., 2018** — *Hypervisor for Consolidating Real-Time Automotive Control Units*, Journal of Systems Architecture.
- **Kohn et al., 2017** — *Timing Analysis for Hypervisor-based I/O Virtualization in Safety-Related Automotive Systems*, SAE.
- **Obermaisser et al., 2009** — *From a Federated to an Integrated Automotive Architecture*, IEEE TCAD.
- **Di Natale & Sangiovanni-Vincentelli, 2010** — *Moving From Federated to Integrated Architectures in Automotive*, Proceedings of the IEEE.
