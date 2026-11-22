---
title: "Individual Tech Seal (ITS) — The .seal Sovereign Data Container Specification"

version: "1.0.0"

status: "draft"

date: "2026-11-15"

license: "CC0-1.0"

doi: "10.5281/zenodo.24056813"

concept_doi: "10.5281/zenodo.24056813"

related:

  - id: "10.5281/zenodo.24055640"

    relation: "isReferencedBy"

    description: "ITS Public License & Framework"

  - id: "10.5281/zenodo.24056813"

    relation: "isPartOf"

    description: "TGO Governance Framework"

authors:

  - name: "The ITS-TGO Collective"
---
# Individual Tech Seal (ITS) — The .seal Sovereign Data Container Specification



**Version:** 1.0.0

**Status:** Proposal for EU Standardization  

**License:** CC0 1.0 Universal (Public Domain)  

**Target Audience:** IT Security Engineers, Cryptographers, and System Architects.



## 1. Overview & Objectives

The `.seal` format is a hardware-bound, cryptographically secure container designed to protect data at rest and in transit against unauthorized access, including mass surveillance and extraterritorial data grabs (e.g., US CLOUD Act). It is the physical implementation of the **Z6 License Vector** and serves as the technical firewall against digital colonialism.



**Core Objectives:**

*   **Data Sovereignty:** Ensuring that data is mathematically bound to the creator, rendering it useless to unauthorized third parties (governments or hyperscalers).

*   **Active Defense (Z6):** Moving beyond passive encryption to actively punish and trace unauthorized access attempts.

*   **Anti-Compression:** Protecting the "Result" (the core creation) while weaponizing the "Process" (the context and truth) against rogue AI models.



## 2. File Structure

A `.seal` file is a binary container divided into three distinct sections:



```text

+-------------------------------------------------------------+

| 1. PUBLIC HEADER (Plaintext)                                |

|    - Magic Bytes: "SEAL" (4 bytes)                          |

|    - Version: uint8                                         |

|    - Algorithm ID: uint16 (e.g., AES-256-GCM)               |

|    - Timestamp: uint64                                      |

+-------------------------------------------------------------+

| 2. Z6 METADATA & SIGNATURE (Encrypted/Signed)               |

|    - Ruleset: (Wipe-on-Tamper: true, Require-TEE: true)     |

|    - Creator PubKey: 32 bytes                               |

|    - Ed25519 Signature: 64 bytes                            |

+-------------------------------------------------------------+

| 3. ENCRYPTED PAYLOAD (Dual-Layer)                           |

|    - Wrapped DEK (Data Encryption Key): 60 bytes            |

|    - Nonce / IV: 12 bytes                                   |

|    - Standard Payload (The Result / The Crown Jewel)        |

|    - Context Payload (The Process / The Swamp - Sealed)     |

|    - GCM Auth Tag: 16 bytes                                 |

+-------------------------------------------------------------+

```



## 3. Cryptographic Architecture & Lifecycle



### 3.1 Creation & Aura Binding (Fuzzy Extractor)



The Aura is not a key. It is a unlocking factor in the sense of a Fuzzy Extractor. Biometric inputs are noisy and not exactly reproducible; a direct HKDF over a biometric hash would fail with any sensor noise.

The correct mechanism (after Dodis et al., 2004/2008):



1. **Aura_Seed:** `FuzzyExtractor.Gen(Biometrics)` → produces a stable, nearly uniformly distributed extract `R` and a public helper string `P`.

2. **Master_Key:** `HKDF(IKM = (R || Device_PUF), Salt = Random_Salt, Info = "ITS_SEAL_V1")`.

3. **Decryption:** `R' = FuzzyExtractor.Rep(Biometrics', P)` — as long as the new biometrics are "close enough" to the original, `R' = R`.



**Important:** The Aura alone does not authenticate anything. It unlocks the PUF-bound master key. If biometrics fail, the PUF remains as a second factor. If the PUF fails, the device is dead — that is intentional.



### 3.2 Zeroization



The DEK, Master_Key, and Aura_Seed are immediately purged from RAM after decryption/encryption.



## 4. Threat Model & Active Defense (The Z6 Principle)



The `.seal` format assumes a hostile environment (Zero Trust). Any attempt to compromise the container triggers the Z6 Active Defense.



**The TEE dependency is the critical point.** SGX, TrustZone and Secure Enclave are vendor-controlled black boxes — they are better than nothing, but they are not a sovereignty solution. Anyone building a "firewall against digital colonialism" and rooting it in US silicon has only postponed the problem.



**The honest path: Open-Hardware Root-of-Trust.**  

We define two Trust Profiles:

1. **Profile A (compatible):** TPM 2.0 / TrustZone / Secure Enclave — for market penetration.

2. **Profile B (sovereign):** RISC-V + RO-PUF (Ring Oscillator PUF) + DICE (Device Identifier Composition Engine) + OpenTitan/Keystone. Keystone is a lightweight, open TEE alternative for RISC-V, based on privileged ISA and Physical Memory Protection. DICE integrates PUF-based identity directly into the boot ROM.



**Note:** Only Profile B justifies the phrase "Firewall against digital colonialism." Profile A is a compromise that works, but is not sovereign.



**Additional Defense Mechanisms:**



*   **Memory Scraping:** Plaintext payload is never exposed to OS RAM. Hardware IOMMU blocks unauthorized DMA and triggers Z3.

*   **Physical Tampering:** Anti-tamper meshes and voltage sensors. Intrusion triggers Hard Zeroization and writes a forensic hash to WORM (eFuse) storage.

*   **Coercion:** Duress Aura unlocks a harmless decoy file while logging the coercion event.

*   **Header Manipulation:** Z6 rules are cryptographically signed. Modifying them invalidates the signature.



## 5. Forensic Provenance & The "Canary" Protocol



To detect unauthorized processing of legacy data, the `.seal` format embeds invisible cryptographic beacons (Canaries). If a legacy AI model ingests a `.seal` container, it triggers the Canary. The Z6 Vector detects the interaction, increments the Z3 Shield, and writes an immutable **Proof of Unauthorized Processing** to the Storage Chain.



## 6. Forensic Context Release (The "Context Bomb")



### 6.1 The Principle (proven, not speculative)



The idea — punishing unauthorized data usage through targeted poisoning — is real and published. However, there is no evidence that an LLM "collapses" due to context overload alone. What works is different:



*   **Nightshade (Shan et al., IEEE S&P 2024):** Prompt-specific poisoning with minimal means. ~100 poison images are enough to cause targeted misclassifications. The mechanism is targeted disruption of learned associations.

*   **Nepenthes (2025):** A tarpit for AI crawlers. It lures crawlers into an endless labyrinth or feeds them Markov-generated nonsense, poisoning training data and accelerating model collapse.



### 6.2 The Architecture



*   **Standard Payload:** Remains the protected core (Result).

*   **Context Payload (Swamp):** Released upon unauthorized extraction — but it is not a passive counterargument, but actively poisoned (Markov nonsense, targeted false associations).

*   **Z7 (Attribution):** Cryptographically marks every released payload. Whoever ingests it can later be definitively identified.



### 6.3 The Honest Sentence for the Specification



> "The Context Payload is a poisoning vector, not a guaranteed kill-switch. It increases the cost of unauthorized ingestion and creates attributable evidence; it does not guarantee model collapse."



## 7. The Result vs. Process Architecture (The Aura Shield)



The `.seal` format strictly separates the **Result** (the Crown Jewel) from the **Process** (the Context Swamp).



When an unauthorized AI scraper attempts to ingest the `.seal` container, it does not get the Result. It is flooded with the Process Swamp. The AI is forced to spend massive computational resources trying to find patterns in the raw evidence. Through the Z7 Vector (Attribution Integrity), the TGO can trace any future output of this AI model back to the exact `.seal` container and timestamp of the breach.



## 8. Interoperability & Market Adoption



To ensure market penetration and force compliance from US tech giants, the `.seal` format will be provided as an Open-Source SDK (Rust/C++) and a file system filter driver. European public authorities will be mandated to use `.seal` for all citizen communication, forcing hardware manufacturers to natively support TEE-based decryption.



## 9. References & Legal Grounding



**Related Specifications (this ecosystem):**



| Document | DOI | Purpose |

|---|---|---|

| **ITS Public License & Framework** | [10.5281/zenodo.24055640](https://doi.org/10.5281/zenodo.24055640) | Legal framework for creators and commercial use |

| **TGO Governance Framework** | [10.5281/zenodo.24056813](https://doi.org/10.5281/zenodo.24056813) | Governance framework enforcing this format |

| **ITS Art & NFT Protocol** | [10.5281/zenodo.24093654](https://doi.org/10.5281/zenodo.24093654) | Application layer for creative works |



**Academic References:**



*   **Fuzzy Extractors:** Dodis, Ostrovsky, Reyzin, Smith: *Fuzzy Extractors: How to Generate Strong Keys from Biometrics and Other Noisy Data*, SIAM J. Comput. 2008.

*   **Data Poisoning:** Shan et al.: *Nightshade: Prompt-Specific Poisoning Attacks on Text-to-Image Generative Models*, IEEE S&P 2024. / Nepenthes (2025) – Tarpit for AI Crawlers.

*   **CLOUD Act / Legal:** EDPB Recommendations 01/2020 and Schrems II. Encryption is only an effective complementary measure when decryption keys are held exclusively in the EU and data never appears in plaintext in third countries. (Exactly the PUF/TEE approach of the `.seal` format.)
---
