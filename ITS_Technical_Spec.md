# Individual Tech Seal — Core Technical Specification

**Version:** 1.2.0
**Status:** Release
**License:** CC0 1.0

## 1. Purpose
ITS defines a stateless execution environment for digital works. Every state transition (usage, duplication, derivation) is bound to a single-use tuple of:
- time window,
- device identity derived from hardware (TEE/PUF),
- a volatile counter vector (Z1-Z7),
- the creator's Biography Fragment.

No persistent authorization state is stored. The license rules are enforced through cryptographic commitments and physical hardware binding.

## 2. Core Principles

1. **Zero Trust** — no trust in any user or network.
2. **Blindness** — the system does not evaluate identity. It only checks the Z6 capability vector.
3. **Ephemeral Unlinkability** — no persistent identifier survives a single state transition.
4. **Creator-Independent Operation** — no central authority can alter the rules.
5. **Automatic Penalization** — Any unauthorized attempt triggers an immediate smart contract penalty and mandates Machine Unlearning.
6. **Aura-Binding** — The Work is cryptographically bound to the Creator's Biography Fragment, rendering it useless for unauthorized AI parsing.
7. **Non-Transferability of Core Intent** — The copyright may be sold, but the Z6 Vector and Biography Fragment cannot. Acquisitions under duress are void.
8. **Cryptographic Visibility Control** — The Work is stored encrypted. The Biography Fragment acts as the decryption key. Without the key, the Work is verifiable on the blockchain (the hash exists) but cannot be displayed or read by unauthorized humans or machines. The owner decides who can view it.

## 3. The Seven Counters (Z1–Z7)

| Counter | Role | Trigger |
|---------|------|---------|
| Z1 | Action counter | Incremented on each legal action. |
| Z2 | Intrusion counter | Incremented on any validation failure. |
| Z3 | Audit shield | Incremented on unauthorized audit attempt or Aura brute-force. |
| Z4 | Generation | Incremented on node rebirth. |
| Z5 | Device heartbeat | TEE-only monotonic counter. |
| Z6 | **License Vector** | Hardware-enforced allowed instructions. Checks for the Biography Fragment. Fires an alarm and penalty on violation. |
| Z7 | **Attribution Integrity** | Validates the dynamic `NOTICE` ledger. Acts as a helper for honest mistakes. Triggers Z6 on malicious attribution removal. |

All counters are derived from the PUF/TEE, incremented after each action, never persistently stored, and immediately zeroized. Physical violations trigger a WORM/eFuse burn-in.

## 4. Dual-Chain Architecture

### 4.1 Operative Chain (Volatile)
Holds current generation, seed, commitment, timestamp, and the active Biography Fragment. Destroyed on zeroization. Validates Z6 against the requested action. If a violation occurs, it triggers the Storage Chain to execute a penalty transaction.

### 4.2 Storage Chain (Persistent)
Append-only archive. Holds only cryptographic commitments and signatures. Never holds plaintext. Records royalty transactions, usage proofs, penalty receipts, and permanent violation marks.

## 5. Cryptographic Visibility & The Forensic Trap

**The Mechanism (Visibility Control):**
The Work is encrypted using the Creator's Biography Fragment as the private key. It is stored on the decentralized network in its encrypted form. 
*   **For Humans (Authorized):** The Creator can grant a "Viewing Token" to specific individuals. The ITS layer uses this token to decrypt the Work locally on the viewer's device.
*   **For AI (Unauthorized):** An AI scraper ingesting the data sees only cryptographic noise. Even if the AI bypasses the encryption, the Z6 Vector blocks the rendering of the data because the hardware lacks the valid Biography Fragment signature.

**The Forensic Trap (Traceable Theft):**
If a malicious actor attempts to extract, brute-force, or steal the Biography Fragment to decrypt the Work, the system acts as a trap:
1.  **Z3 Counter Activation:** The Z3 (Audit Shield) detects the unauthorized access pattern (e.g., a script attempting to brute-force the key or a memory dump).
2.  **Z6 Hardware Lockdown:** The Z6 Vector freezes the execution of the decryption process and locks the hardware.
3.  **Immutable Proof:** A cryptographic hash of the stolen/attempted key and the attacker's metadata is burned into the Storage Chain (WORM/eFuse).
4.  **Smart Contract Penalty:** The Smart Contract triggers an immediate financial penalty against the attacker's registered wallet (if applicable) or flags the attacker's hardware ID on the TGO Blacklist.

The Creator is instantly notified: *"Someone attempted to steal your Aura."* The proof is mathematically undeniable.

## 6. Hardware Enforcement & The NOTICE System

**The Z6 Vector:**
The Z6 Vector is the hardware-enforced capability vector that defines permitted actions. It is bound to the chip. If an AI system attempts to scrape the Work, Z6 blocks the access. The attempt itself is the violation.

**The Z7 Vector (Attribution Integrity & Helper Mode):**
The Z7 counter validates the dynamic `NOTICE` file against the actual execution state.
*   **Helper Mode:** If a developer forgets to attribute a dependency, Z7 provides an automated error report to help them fix the omission. It does not kill the node. It acts as a supportive tool.
*   **Punitive Mode (Z6 Trigger):** If the system detects a malicious removal of attribution (Attribution Laundering) intended to steal ownership, Z6 triggers a Hard Lockdown and zeroizes the node.

## 7. The Constitutional Amendment Protocol (Swarm & Dongle)
To prevent a single instance (state or corporation) from breaking the license while allowing the society to adapt it:
*   **Threshold Consensus:** Changes require an 80% supermajority of hardware-bound nodes (PUF/Dongle). Voting power is decentralized among actual individuals (Proof of Personhood).
*   **Time-Lock:** Any approved amendment is subject to a mandatory 6-month time-lock before taking effect.
*   **Forking:** The minority rejecting the change can fork the system and continue under the old rules. No one is forced, but no one can buy the system.

## 8. Reference Implementation (Rust Excerpt)

```rust
// Individual Tech Seal — reference implementation
// IMPORTANT: Illustrative only. Do not use in production.

use sha2::{Sha256, Digest};
use zeroize::Zeroize;

#[derive(Default)]
struct NodeState {
    z6_license_vector: u64,
    z3_audit_shield: u64, // Added for Forensic Trap
    seed: [u8; 32],
    biography_fragment: [u8; 32], // The Cryptographic Aura
    action_executed: bool,
}

impl Zeroize for NodeState {
    fn zeroize(&mut self) {
        self.z6_license_vector.zeroize();
        self.z3_audit_shield.zeroize();
        self.seed.zeroize();
        self.biography_fragment.zeroize();
        self.action_executed = false;
    }
}

pub struct ITSNode {
    state: NodeState,
    alive: bool,
}

impl ITSNode {
    /// Checks if the requested action is allowed by the Z6 license vector.
    /// Requires the matching Biography Fragment.
    pub fn execute_action(&mut self, action_type: u64, payload: &[u8], provided_aura: &[u8; 32]) -> Result<(), &'static str> {
        if !self.alive {
            return Err("node is dead");
        }

        // Z6 Validation and Aura Check (Hardware enforced)
        if (self.state.z6_license_vector & action_type) == 0 || self.state.biography_fragment != *provided_aura {
            self.trigger_penalty_and_zeroize(action_type);
            return Err("Z6 license violation or invalid Aura Fragment. Action not permitted.");
        }

        // Execute (e.g., trigger smart contract for royalty)
        let _transaction_hash = Sha256::digest(payload);
        
        self.state.action_executed = true;
        self.zeroize();
        Ok(())
    }

    /// Attempts to decrypt and display the Work (Visibility Control).
    /// Requires the matching Biography Fragment (Aura).
    pub fn decrypt_and_view(&mut self, encrypted_payload: &[u8], provided_aura: &[u8; 32]) -> Result<Vec<u8>, &'static str> {
        if !self.alive {
            return Err("node is dead");
        }

        // Z6 Validation: Is the provided Aura the correct one?
        if self.state.biography_fragment != *provided_aura {
            // FORENSIC TRAP: Unauthorized decryption attempt detected!
            self.state.z3_audit_shield += 1; // Increment audit shield
            self.state.z6_license_vector = 0; // Revoke all permissions immediately
            self.zeroize(); // Kill the node state
            
            println!("[ALARM] UNAUTHORIZED AURA ACCESS! Theft attempt logged. Z3 Shield activated.");
            return Err("Z6 Security Violation: Invalid Aura. Access Denied. Theft Logged.");
        }

        // Authorized: Decrypt the payload (pseudo-code for demonstration)
        let decrypted_data = encrypted_payload.to_vec(); // Placeholder for actual decryption
        println!("[SUCCESS] Work decrypted. Displaying to authorized user.");
        
        Ok(decrypted_data)
    }

    fn trigger_penalty_and_zeroize(&mut self, action_type: u64) {
        if action_type == 0b100 { // Example bit for AI Training
            println!("[ALARM] AI Scraping detected! Machine Unlearning Mandate issued.");
        } else {
            println!("[ALARM] Unauthorized access detected! Smart Contract Penalty executed.");
        }
        self.zeroize();
    }

    pub fn zeroize(&mut self) {
        self.state.zeroize();
        self.alive = false;
    }
}
