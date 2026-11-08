# Individual Tech Seal — Core Technical Specification

**Version:** 1.0.0
**Status:** Initial Release
**License:** CC0 1.0

## 1. Purpose
ITS defines a stateless execution environment for digital works. Every state transition (usage, duplication, derivation) is bound to a single-use tuple of:
- time window,
- device identity derived from hardware (TEE/PUF),
- a volatile counter vector (Z1-Z6),
- the creator's Biography Fragment.

No persistent authorization state is stored. The license rules are enforced through cryptographic commitments and physical hardware binding.

## 2. Core principles
1. **Zero Trust** — no trust in any user or network.
2. **Blindness** — the system does not evaluate identity. It only checks the Z6 capability vector.
3. **Ephemeral Unlinkability** — no persistent identifier survives a single state transition.
4. **Creator-Independent Operation** — no central authority can alter the rules.
5. **Automatic Penalization** — Any unauthorized attempt triggers an immediate smart contract penalty and mandates Machine Unlearning.
6. **Aura-Binding** — The Work is cryptographically bound to the Creator's Biography Fragment, rendering it useless for unauthorized AI parsing.
7. **Non-Transferability of Core Intent** — The copyright may be sold, but the Z6 Vector and Biography Fragment cannot.

## 3. The six counters (Z1–Z6)

| Counter | Role | Trigger |
|---------|------|---------|
| Z1 | Action counter | Incremented on each legal action |
| Z2 | Intrusion counter | Incremented on any validation failure |
| Z3 | Audit shield | Incremented on unauthorized audit attempt |
| Z4 | Generation | Incremented on node rebirth |
| Z5 | Device heartbeat | TEE-only monotonic counter |
| Z6 | **License Vector** | Hardware-enforced allowed instructions. Checks for the Biography Fragment. Fires an alarm and penalty on violation. |

All counters are derived from the PUF/TEE, incremented after each action, never persistently stored, and immediately zeroized. Physical violations trigger a WORM/eFuse burn-in.

## 4. Dual-Chain Architecture

### 4.1 Operative Chain (Volatile)
Holds current generation, seed, commitment, timestamp, and the active Biography Fragment. Destroyed on zeroization. Validates Z6 against the requested action. If a violation occurs, it triggers the Storage Chain to execute a penalty transaction.

### 4.2 Storage Chain (Persistent)
Append-only archive. Holds only cryptographic commitments and signatures. Never holds plaintext. Records royalty transactions, usage proofs, penalty receipts, and permanent violation marks.

## 5. Formal Model (TLA+ Excerpt)

```tla
---- MODULE ITSMonitor ----
EXTENDS Naturals, TLC
CONSTANTS TimeWindow, ActionInterval

VARIABLES state, born, Z1, Z2, Z5, Z6, seed, trace_out, penalty_triggered, unlearning_mandated, aura_valid

Init ==
  /\ state = "DEAD"
  /\ born = FALSE
  /\ Z1 = 0 /\ Z2 = 0
  /\ Z5 = 0 /\ Z6 = 0
  /\ penalty_triggered = FALSE
  /\ unlearning_mandated = FALSE
  /\ aura_valid = FALSE

Validate(request) ==
  /\ state = "IDLE"
  /\ born = TRUE
  /\ LET cmdAllowed = request.cmd \in Z6
         Z1match    = request.Z1 = Z1
         auraMatch  = (request.biography_fragment == seed)
     IN  IF cmdAllowed /\ Z1match /\ auraMatch
         THEN /\ state' = "EXECUTE"
              /\ penalty_triggered' = FALSE
         ELSE /\ state' = "ZEROIZE"
              /\ Z2' = Z2 + 1
              /\ penalty_triggered' = TRUE
              /\ unlearning_mandated' = (request.cmd = "ai_training")

Zeroize ==
  /\ state = "ZEROIZE"
  /\ born' = FALSE
  /\ Z1' = 0 /\ Z2' = 0
  /\ Z5' = 0 /\ Z6' = 0
  /\ state' = "DEAD"

Safety == [](state = "DEAD" => born = FALSE)
====
