# Virtualization on MCU-Based Systems

## Why Consolidate onto Fewer MCUs?

The traditional federated architecture in safety-critical domains (automotive, avionics, industrial)
has historically led to a proliferation of microcontrollers: each function runs on dedicated hardware,
with its own RTOS and firmware. This proliferation is motivated by organizational reasons
(separation of responsibilities among suppliers) and technical ones (fault isolation, certification
simplicity) [1, 2].

However, as functional complexity grows, so does the number of microcontrollers — and with it
the complexity of interconnections (CAN, SPI, IIC) among them. This leads to increasing costs,
weight, reliability issues, and difficulty in orchestrating development across systems [3, 4]. The
natural response is **consolidation**: reducing the number of physical microcontrollers by
aggregating multiple applications onto the same hardware.

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
to demonstrate tight WCET bounds [5].

### 2. Changing Platform Is Prohibitive

Migrating legacy software from MCUs to MMU-based processors entails:

- **Loss of existing certifications**: rewriting the code invalidates previously obtained safety
  certifications (e.g., ISO 26262), requiring a new and costly certification process [3].
- **RTOS and framework incompatibility**: legacy firmware depends on MCU-specific RTOSes and
  certified toolchains (e.g., IAR) that assume a bare-metal environment. These cannot be directly
  ported to MMU-based architectures [3].
- **Hardware certification constraints**: ASIL-D functions require hardware with lockstep mode and
  specific certification. At the time of the relevant publications, no MMU-based processor certified
  for ASIL-D was available [6, 7].

### Summary

Consolidation therefore happens on MCUs because **the software already exists on that platform**
and **cannot simply be moved elsewhere** without unsustainable costs and risks. Virtualization on
MCUs arises precisely to address this need: aggregating multiple legacy firmware instances onto the
same physical microcontroller, while preserving spatial and temporal isolation among them, without
requiring code rewrites or platform changes [3, 6, 7].

## The Mixed-Criticality Challenge

In the federated architecture, each application runs on its own dedicated hardware, so the
coexistence of different ASIL levels was never an issue — each MCU simply had its own criticality
level and was certified independently. The problem arises precisely **when consolidating**: bringing
together applications with different criticality levels onto the same hardware platform introduces
the risk of interference between them, which is the defining challenge of **Mixed-Criticality
Systems (MCS)** [8, 9].

### Why Not Just Consolidate Apps of the Same Criticality Level?

A natural objection is: why not avoid the problem by consolidating only applications with the
same ASIL level onto the same MCU? The literature identifies two fundamental reasons why this
approach is insufficient.

**1. Resource waste.** High-criticality applications (e.g., ASIL-D) are certified using very
pessimistic WCET estimates obtained via static analysis. As a result, the processor is significantly
underutilized. Dedicating an ASIL-D MCU — expensive, with lockstep hardware — exclusively to
high-criticality tasks means paying for hardware that runs at a fraction of its capacity. As Jiang
et al. note: *"complete isolation leads to huge resource waste as the WCET estimations used in the
certification of high-criticality tasks is very pessimistic"* [8].

**2. Loss of consolidation benefits.** Grouping by criticality level still results in as many MCUs
as there are ASIL levels, merely replacing one form of proliferation with another. The reduction
in hardware count — and thus in cost, weight, and interconnection complexity — would be
marginal at best.

**3. Intrinsic interdependency across criticality levels.** Modern safety-critical functions are
inherently mixed-criticality by nature. In an autonomous driving system, for instance, braking
(ASIL-D) and infotainment (non-safety) components must communicate and coordinate. Physically
separating them introduces latency and communication complexity that undermines system
efficiency [9].

The dilemma is therefore unavoidable: if resource sharing between high- and low-criticality
tasks is allowed, then low-criticality tasks must be certified at the high-criticality level due to
potential interference — which is prohibitively expensive. If sharing is forbidden, resource
utilization collapses [8]. The solution lies in **virtualization with strong spatial and temporal
isolation**, which allows mixed-criticality applications to coexist on the same MCU while
preventing any cross-criticality interference.

---

## References

- [1] R. Obermaisser, C. El Salloum, B. Huber, and H. Kopetz, "From a Federated to an Integrated Automotive Architecture," *IEEE Transactions on Computer-Aided Design of Integrated Circuits and Systems*, vol. 28, no. 7, pp. 956–965, 2009.
- [2] M. Di Natale and A. L. Sangiovanni-Vincentelli, "Moving From Federated to Integrated Architectures in Automotive: The Role of Standards, Methods and Tools," *Proceedings of the IEEE*, vol. 98, no. 4, pp. 603–620, 2010.
- [3] J. Li, R. Hou, G. Shang, H. Zhang, X. Cheng, and R. Pan, "FVM: Practical Feather-Weight Virtualization on Commodity Microcontrollers," *IEEE Transactions on Computers*, vol. 74, no. 7, pp. 2389–2402, 2025.
- [4] R. Pan and G. Parmer, "SBIs: Application Access to Safe, Baremetal Interrupt Latencies," 2022.
- [5] R. Pan and G. Parmer, "MxU: Towards Predictable, Flexible, and Efficient Memory Access Control for the Secure IoT," *ACM Transactions on Embedded Computing Systems*, vol. 18, no. 5s, Article 103, 2019.
- [6] A. K. Sundar Rajan et al., "Hypervisor for Consolidating Real-Time Automotive Control Units: Its Procedure, Implications and Hidden Pitfalls," *Journal of Systems Architecture*, vol. 82, pp. 37–48, 2018.
- [7] A. Kohn, K. Schmidt, J. Decker, M. Sebastian, A. Züpke, and A. Herkersdorf, "Timing Analysis for Hypervisor-based I/O Virtualization in Safety-Related Automotive Systems," *SAE Int. J. Passeng. Cars – Electron. Electr. Syst.*, vol. 10, no. 2, 2017.
- [8] Z. Jiang, N. Audsley, P. Dong, N. Guan, X. Dai, and L. Wei, "MCS-IOV: Real-Time I/O Virtualization for Mixed-Criticality Systems," in *Proc. IEEE Real-Time Systems Symposium (RTSS)*, 2019.
- [9] S. Pinto, H. Araujo, D. Oliveira, J. Martins, and A. Tavares, "Virtualization on TrustZone-enabled Microcontrollers? Voilà!" in *Proc. IEEE Real-Time Embedded Technology and Applications Symposium (RTAS)*, 2019.
