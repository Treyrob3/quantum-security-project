# Getting Started with Quantum Security
 
A zero-to-hero learning path for quantum security: what it is, why it is urgent, and how to become useful in it. No physics background required.
 
Quantum security has two halves, and most people arrive knowing about one:
 
- **Post-quantum cryptography (PQC) and migration** — quantum computers will break the public-key cryptography protecting the internet today. This half is about replacing it, across estates that take years to change.
- **Quantum platform security** — quantum computers are already sold as cloud services. Securing the platforms themselves, their toolchains, and the workloads running on them is a live discipline with published attacks.
This guide covers both. Work through the levels in order, or jump to the track that matches your job.
 
| Level | For |
|---|---|
| [Level 0](#level-0--orientation) | Anyone. What is happening and why it matters now.
| [Level 1](#level-1--foundations) | Everyone continuing further. The concepts everything else assumes.
| [Level 2A](#level-2a--the-migration-track) | Security engineers, architects, GRC, anyone with an estate to migrate.
| [Level 2B](#level-2b--the-quantum-platform-track) | Anyone securing or using quantum computing platforms.
| [Level 3](#level-3--depth-and-contribution) | Contributors, researchers, people going deep. | Ongoing |
 
---
 
## Level 0 — Orientation
 
**The one-paragraph version.** A sufficiently large quantum computer running Shor's algorithm would break RSA, Diffie-Hellman, and elliptic-curve cryptography — which is nearly all public-key cryptography deployed today, covering TLS, VPNs, code signing, and PKI. No such machine exists yet. It still matters now, for three reasons: encrypted data stolen today can be decrypted later once one exists; migrating a large estate takes five to fifteen years; and regulators have already set deadlines that do not depend on when the machine arrives. Separately and in parallel, today's small, noisy quantum computers are commercially available through the cloud, and securing *them* is its own emerging field.
 
**Do one thing.** Visit [Cloudflare's post-quantum test endpoint](https://pq.cloudflareresearch.com/). It tells you whether your own browser just negotiated a post-quantum key exchange. Five minutes, and the abstraction becomes something happening on your machine right now.
 
**Then read one thing.** The joint CISA/NSA/NIST [Quantum-Readiness fact sheet](https://www.cisa.gov/resources-tools/resources/quantum-readiness-migration-post-quantum-cryptography) — short, official, and the basis of most national migration guidance.
 
---
 
## Level 1 — Foundations
 
Everything later assumes this material.
 
### The two algorithms
 
**Shor's algorithm** (1994) solves integer factoring and discrete logarithms in polynomial time on a sufficiently large quantum computer. Those two problems are the entire security basis of RSA, Diffie-Hellman, and elliptic-curve cryptography, including the signature schemes built on them (ECDSA, EdDSA, DSA). When a large enough machine exists, mainstream public-key cryptography fails — not gradually, but as a class.
 
**Grover's algorithm** gives a quadratic speedup for brute-force search. Against symmetric cryptography it effectively halves the security level: AES-128 behaves like roughly 64-bit security. The remedy is larger parameters (AES-256, SHA-384), not new algorithm families. This is why quantum is primarily a *public-key* problem — though symmetric parameters, MAC lengths and KDFs still need review.
 
### Two acronyms you will see constantly
 
- **CRQC** — Cryptographically Relevant Quantum Computer: large and reliable enough to actually run Shor's against real key sizes. It does not exist. Government planning horizons cluster around 2030–2035; nobody knows.
- **NISQ** — Noisy Intermediate-Scale Quantum: today's machines. Small, error-prone, nowhere near cryptographically relevant — and already available to rent by the second. This is why platform security (Level 2B) is a present-tense discipline, not a future one.
### Why it is urgent before any CRQC exists
 
**Harvest-Now-Decrypt-Later (HNDL).** An adversary records encrypted traffic or exfiltrates encrypted archives *today*, stores them, and decrypts them once a CRQC exists. The capture half requires no quantum computer — only storage and patience. Every TLS session whose key exchange used RSA or ECDH is future-readable to whoever captured it.
 
**Mosca's inequality.** The planning rule: if **X** (how long your data must stay confidential) plus **Y** (how long your migration takes) exceeds **Z** (time until a CRQC), you are exposed *today*. A hospital with 30-year records fails this inequality on any plausible Z, no matter how fast it migrates. The consequence that surprises people: *data shelf-life*, not sensitivity alone, drives migration priority. A medium-sensitivity record kept 30 years outranks a high-sensitivity record kept two.
 
### The replacement: post-quantum cryptography
 
PQC is **classical cryptography built on problems with no known quantum speedup** — mostly lattices and hash functions. It runs on ordinary computers. No quantum hardware is involved in the defence.
 
NIST finalized the first standards in August 2024:
 
| Standard | Algorithm | Replaces | Notes |
|---|---|---|---|
| [FIPS 203](https://csrc.nist.gov/pubs/fips/203/final) | ML-KEM (Kyber) | RSA/ECDH key exchange | Parameter sets 512/768/1024 |
| [FIPS 204](https://csrc.nist.gov/pubs/fips/204/final) | ML-DSA (Dilithium) | RSA/ECDSA signatures | Keys 2–4 KB, signatures 2.4–4.5 KB |
| [FIPS 205](https://csrc.nist.gov/pubs/fips/205/final) | SLH-DSA (SPHINCS+) | Long-lived signatures | Conservative hash-based assumptions |
 
Note the sizes — PQC signatures run roughly ten times larger than classical ones, which is why constrained hardware (smart cards, TPMs, HSMs) is a recurring problem rather than a footnote.
 
**Hybrid cryptography** is the transition pattern: combine a classical and a PQC algorithm so an attacker must break *both*. Done correctly, the session key derives through a KDF over both inputs. Done incorrectly — key derived from one input, or silent fallback to classical when PQC negotiation fails — hybrid provides false comfort rather than defence in depth.
 
### The deadlines are regulatory
 
They apply regardless of when a CRQC actually arrives:
 
- **[NIST IR 8547](https://csrc.nist.gov/pubs/ir/8547/ipd)** — RSA/ECC deprecated by 2030, all quantum-vulnerable public-key cryptography disallowed by 2035
- **NSA CNSA 2.0** — quantum-resistant software and firmware signing by 2030
- **EU roadmap** — standalone classical public-key crypto prohibited for high-risk uses after 2030
### Level 1 hands-on
 
- **[Practical Introduction to Quantum Safe Cryptography](https://quantum.cloud.ibm.com/learning/en/courses/quantum-safe-cryptography)** (IBM Quantum Learning, free, ~10 hours) — the most on-topic structured course available. If you do one thing at this level, do this.
- **[Cloudflare's post-quantum blog series](https://blog.cloudflare.com/tag/post-quantum/)** — what deploying PQC at internet scale actually involved, including what broke.
---
 
## Level 2A — The migration track
 
For anyone with an estate to move. The consistent lesson across every published migration programme: **migration fails on engineering and inventory realities, not on algorithm choice.**
 
### The pipeline
 
1. **Inventory.** You cannot migrate what you cannot see. Where does your estate use which algorithms — in code, protocols, certificates, hardware, and vendor products? Most organisations do not know. Discovery tooling and cryptographic bills of materials (CBOMs) serve this step, and it is step one in every government playbook.
2. **Prioritise by data lifetime.** Mosca's inequality applied per dataset.
3. **Build crypto-agility.** The property of swapping an algorithm or parameter set *without rebuilding the system*. Hard-coded algorithm identifiers, bespoke protocols that cannot negotiate, and firmware that cannot be updated all mean replacement rather than migration. Agility outlives this transition — PQC parameter sets will themselves be revised.
4. **Migrate, anchors first.** Signature and trust-anchor migration has the longest lead time and widest blast radius, because forgery is an *active* attack: fake updates, forged certificates, fraudulent transactions. Hardware roots of trust may be physically unreplaceable on relevant timescales.
5. **Do hybrid correctly, and treat it as transitional.** Test that PQC failure does not silently fall back to classical. Plan hybrid's replacement with pure PQC ahead of the deadlines.
### Where the hard parts are
 
- **Hardware roots of trust** — TPMs, UEFI Secure Boot keys, smart cards, HSMs, signed firmware in vehicles and industrial controllers. Tamper-resistant by design means unupgradable by design, on 15–20 year lifecycles. A controller shipped today is still in service past every CRQC planning horizon. Procurement matters more than patching here.
- **Downgrade and fallback** — an active attacker stripping PQC options from negotiation defeats hybrid entirely if the system fails open.
- **The PQC libraries themselves** — implementation bugs in the stacks used for migration are a live risk, not a theoretical one. See the [knowledge base](README.md) for tracked examples.
### Level 2A resources
 
- **[NCCoE Migration to Post-Quantum Cryptography](https://www.nccoe.nist.gov/applied-cryptography/migration-to-pqc)** (NIST) — a live project with 50+ industry collaborators, organised around cryptographic discovery and interoperability testing. Its [documentation and FAQ site](https://pages.nist.gov/nccoe-migration-post-quantum-cryptography/) is the most practical migration reading available.
- **[Awesome Post-Quantum Cryptography list](../awesomelist/README.md)** — this project's curated resource list: inventory tooling, PQC implementations, standards, test servers.
- **[Open Quantum Safe test servers](https://test.openquantumsafe.org/)** — per-algorithm TLS endpoints for testing client support.
- **[IETF PQUIP working group](https://datatracker.ietf.org/wg/pquip/about/)** — migration patterns and documented failure modes.
---
 
## Level 2B — The quantum platform track
 
The less populated half of the field, and the one where this project actively seeks contributors. Quantum computers are cloud services people already pay to use; securing them is a discipline with real, published attacks.
 
### The architecture you need in your head
 
You never touch a QPU directly. A job flows:
 
```
your circuit (gate level)
   → framework (Qiskit, Cirq, PennyLane)
   → transpiler / optimising compiler
   → pulse-level scheduler + calibration data
   → classical control electronics (FPGAs, signal generators)
   → QPU, usually shared with other tenants
   → results, as probability distributions
```
 
Every layer is a trust decision.
 
### Three facts that drive nearly every platform risk
 
1. **There is no quantum memory.** Your input data enters as constants hardcoded inside the circuit itself. Stealing the circuit steals the algorithm *and* its data; observing the control plane observes the workload.
2. **What you write is not what runs.** Gates are abstractions; hardware executes calibrated microwave pulses. Most SDKs never validate that a custom gate's pulse implementation matches what its gate-level description declares — the quantum equivalent of source versus compiled binary.
3. **Results are probability distributions.** A tampered, degraded, or misrouted execution can return a plausible-looking result indistinguishable from ordinary NISQ noise — and the workloads with the strongest commercial case for quantum are often those whose answers cannot be cheaply checked.
### What has actually been demonstrated
 
These are peer-reviewed results, not speculation:
 
- **Cross-tenant attacks** — crosstalk from an adjacent tenant's circuit degrading a victim's computation (NDSS 2025); residual qubit state surviving standard reset operations and readable by the next tenant (CCS 2023).
- **Toolchain attacks** — circuit theft via compromised compilers (HASP 2021); backdoors triggered through compiler configuration files disguised as routine calibration (ICASSP 2023); pulse-level attacks abusing the unvalidated gate/pulse gap, with most current SDKs found vulnerable (IEEE S&P 2025).
- **Side channels** — reset-operation timing revealing program structure (CCS 2022); power traces from classical controllers reconstructing gate-level circuits (CCS 2023).
- **Untrusted providers** — adversarial tampering with executions modelled and experimentally evaluated, with shot-splitting across providers proposed as runtime detection (HASP 2022; Frontiers in Computer Science 2024).
Each is cited with full references in the corresponding [Top 10 entry](../quantum-top-10/).
 
### Level 2B hands-on
 
Reading about QPUs is a poor substitute for submitting a job to one.
 
- **[Use a quantum computer today](https://quantum.cloud.ibm.com/learning/en/courses/use-a-qc-today)** (IBM, free, ~3 hours) — shortest path from zero to having run something on real hardware.
- **[IBM Quantum Platform](https://quantum.cloud.ibm.com/)** — free-tier accounts can submit jobs to real QPUs (usage limits apply). Worth doing once. Watch your circuit queue, transpile, and return a distribution — and notice that you receive no independent evidence of what physically executed.
- **[PennyLane Codebook](https://codebook.xanadu.ai/)** (Xanadu) — exercise-based, runs entirely in the browser, no account or install. Good if you learn by writing code.
- **[A Primer on Security of Quantum Computing Hardware](https://arxiv.org/abs/2305.02505)** — survey of the hardware attack surface.
---
 
## Level 3 — Depth and contribution
 
### Read the primary sources
 
The fastest route to real depth is to stop reading summaries. Pick a [Top 10 entry](../quantum-top-10/) and read its cited papers — the platform-surface ones especially are short, concrete, and readable. Standards to know: FIPS 203/204/205, NIST IR 8547, the CISA/NSA/NIST fact sheet, and the IETF hybrid TLS drafts.
 
### Go deeper on the theory (optional)
 
- **[Basics of Quantum Information](https://quantum.cloud.ibm.com/learning/en/courses/basics-of-quantum-information)** (IBM / John Watrous, free, ~15 hours) — university-level and genuinely rigorous. Not required for security work; excellent if you want the real thing rather than analogies.
### Keep current
 
This field moves. The [knowledge base](README.md) in this repository logs disclosed vulnerabilities and cryptanalysis findings as they land, mapped to the relevant risks — a good way to watch the field without drinking from the firehose. New attacks appear at NDSS, CCS, USENIX Security, IEEE S&P, and QCE each year.
 
### Contribute
 
You do not need to be a physicist or a cryptographer. The project needs reviewers, evidence-checkers, and writers as much as researchers. Newcomer-sized ways in:
 
- Verify that an entry's reference links resolve and that the cited papers are characterised accurately
- Add a resource to the [awesome list](../awesomelist/README.md), or an entry to the [knowledge base](README.md)
- Review an open pull request against the project's evidence convention (demonstrated / emerging / theoretical)
- Take on the platform surface, where contributors are most needed and the ground is least crowded
See [How to Contribute](../README.md#how-to-contribute) and the [sprint plan](../plans/) for where the work currently stands.
 
### Community
 
`#project-quantum-security` on the [OWASP Slack](https://owasp.org/slack/invite), and fortnightly community calls — details in the [README](../README.md#community-and-contact).
 
---
 
> **A note on vendor material.** IBM, Cloudflare, Xanadu and others produce genuinely good free education, listed here on merit. Read it aware that it sits alongside commercial offerings. The standards bodies (NIST, NCCoE, IETF) and the peer-reviewed papers cited in each Top 10 entry remain the neutral references.
 
*Part of the [OWASP Quantum Security Project](https://github.com/OWASP/quantum-security-project). Licensed CC BY-SA 4.0. Corrections and additions welcome by pull request.*
