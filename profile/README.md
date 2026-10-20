# Getulio

> **Swiss-grade personal financial organizer engineered with zero-compromise Privacy-First Design (PFD) and end-to-end zero-knowledge cryptography.**

Getulio is an offline-capable, privacy-first personal financial management platform. It is engineered to maintain complete confidentiality and mathematical data protection even in the event of a catastrophic server, CDN, or database compromise.

---

## Architectural Pillars

Getulio treats user privacy not as an access-control policy, but as an inviolable cryptographic boundary.

* **Zero-Knowledge Architecture (ZKE):** All master keys (`data_key` AES-256 and Ed25519 signing pairs) are generated client-side. The backend server never receives, stores, or processes unencrypted financial payloads or private keys.
* **Opaque Data Model (ODM):** Database tables (e.g., `getulio_ledger`) use fully obfuscated schemas (`f0` through `f6`). Financial domain entities reside exclusively inside authenticated AES-GCM encrypted payloads (`f6`).
* **Per-Account Order-Preserving Encryption (OPE):** Sorting indices (such as dates in `f1` and amounts in `f2`) use distinct, per-account derived OPE keys ($K_{\text{OPE}} = \text{HKDF}(\text{data\_key}, \text{account\_id}, \text{salt})$), neutralizing global statistical pattern matching and frequency analysis across the database.
* **Isolated Cryptographic Boundary (ICB):** Decryption and signing routines are quarantined inside a dedicated **Browser Extension**. The Web Application presentation layer never holds raw master keys in scope, neutralizing supply-chain and malicious frontend script injection attacks.
* **Zero Password Knowledge (ZPK):** No passwords, recovery emails, or centralized credentials exist on the server. Access and writes are authorized purely through asymmetric signature verification (Ed25519) by a blind "Bouncer" backend.
* **Zero Personally Identifiable Information (Z-PII):** Accounts are indexed exclusively by anonymous client-generated identifiers (`u_id`). Profile settings and metadata are stored fully encrypted inside state payloads.
* **Volatile Sessions & Instant Key Eviction:** Decryption keys exist only in volatile memory (RAM). The Browser Extension binds keys strictly to the active Session ID (`sid`). Any session mismatch or tab closure immediately flushes keys from memory, requiring a fresh mobile handshake.
* **Sovereign Recovery:** Zero custodial key escrow. Backups rely on self-custodial encrypted vault exports (PBKDF2 + AES-GCM via SVG QR codes or deterministic recovery sheets).

---

## Topology & Core Roles

```
 ┌─────────────────┐       P2P Handshake (QR + nacl.box)       ┌───────────────────────┐
 │   Mobile App    │ ◄───────────────────────────────────────► │   Browser Extension   │
 │ (Auth & Viewer) │                                           │ (Key Custody & Crypto)│
 └─────────────────┘                                           └───────────┬───────────┘
                                                                           │ Ephemeral IPC
                                                                           │ (sid-validated)
                                                                           ▼
 ┌─────────────────┐       Blind Encrypted Blobs & Signatures  ┌───────────────────────┐
 │ Backend Server  │ ◄──────────────────────────────────────── │    Web Application    │
 │ ("The Bouncer") │                                           │  (Presentation Layer) │
 └─────────────────┘                                           └───────────────────────┘

```

1. **Mobile Application (Sovereign Authenticator & Read-Only Viewer):** Holds the master seed securely. Acts as the authenticator for desktop sessions via dynamic QR codes (`nacl.box` ephemeral P2P handshake). Provides a read-only dashboard for quick reference.
2. **Browser Extension (Cryptographic Boundary & Signer):** Terminates the P2P handshake, holds keys in ephemeral RAM, exposes a permissioned crypto-provider IPC API, and signs all outgoing ledger mutations.
3. **Web Application (Presentation Layer & Sole Writer):** Pure presentation UI (Svelte / SvelteKit). Requests decryption/signing through the extension. Single source of mutation for transactions and ledger state changes.
4. **Backend Server (Blind Bouncer):** Verifies Ed25519 request signatures against stored public keys, verifies write authorizations, and persists encrypted rows without the ability to inspect payload contents.

---

## Functional Scope

* **Double-Entry Account Ledger:** Centralized tracking for liquid accounts, savings, and custom currency buckets with atomic transfer linkage.
* **Credit Card Management:** Full lifecycle tracking from transaction posting through statement closing cuts and automatic settlement commitments.
* **Active Merge Bank Reconciliation:** Bi-directional matching between imported/executed transactions and forecasted plans via an isolated Quarantine state (`State 03`) before atomic promotion to Reconciled (`State 01`).
* **Dynamic Cash Flow Engine (Column H):** High-performance in-memory linked list computing continuous rolling balances across past, present, and projected horizons.
* **Multi-Layer Scenario Simulation ("What-If"):** Ephemeral projection layers (`State 51` Ghost Lines) evaluated in-RAM against sandbox parameters (`State 50`) without mutating live ledger data.
* **Budgeting & Variance Tracking:** Envelopes, spending limits, and automated drift detection against scheduled commitments.

---

## Privacy-Preserving AI Financial Advisory

Getulio integrates advanced LLM reasoning without exposing financial data, true identities, or network footprints:

* **Direct Extension Egress:** AI requests dispatch directly from the Browser Extension background worker, completely bypassing the Getulio backend.
* **Bring-Your-Own-Key (BYOK):** The user provides their personal LLM API key, stored only within the extension's local encrypted storage.
* **Ephemeral Data Masking (In-RAM Transformation):**
* **Value Perturbation:** Amounts are scaled by a session-secret random multiplier ($V'_i = V_i \times S_k$) or converted into relative basis points.
* **Temporal Shifting:** Real calendar dates are shifted by a uniform random delta ($\Delta t$) or abstracted into relative cycle offsets.
* **Entity Obfuscation:** Bank names, merchant tags, and notes are replaced with synthetic aliases (`Vendor_Groceries_01`, `Account_Primary_Liquid`).


* **RAM Denormalization:** The LLM delivers reasoning strictly on masked ratios. The extension reverses the scaling and alias mapping in RAM before rendering recommendations to the view, discarding perturbation keys immediately.

