# R-NIMKE

EXPERIMENTAL - DO NOT USE

Reticulum Non-Interactive Multi-Party Key Exchange

Initial Draft v1 - 14 July 2026 - Ivan 
LXMF: f489752fbef161c64d65e385a4e9fc74 

No LLMs were used in the creation of this document or cryptographic protocol. It is considered experimental until verified and extensively tested.

---

Definition: R-NIMKE is an application-layer group cryptography layer for programs that use Reticulum and LXMF or other message formats. It is not part of the Reticulum Network Stack, not a destination type, and not a replacement for RNS transport encryption. It defines how group members share secrets, rekey, and encrypt group message payloads that then ride as opaque data over RNS, LXMF, propagation nodes, or paper messages.

Purpose: enable low-bandwidth, Reticulum Zen compatible encrypted group messaging for small and large groups, including delivery via LXMF propagation nodes and paper messages or other offline means.

## THREAT MODEL

The adversary may:

  - Read all traffic on intermediate links and relays.
  - Log all historical ciphertext indefinitely.
  - Inject, modify, reorder, replay, or drop packets.
  - Operate malicious peers or Propagation Nodes.
  - Serve old or alternate signed states if clients do not enforce
    monotonic state rules.
  - Compromise a member device at a point in time.
  - Compromise the administrator key (group is then burned).

The adversary is assumed NOT to break:

  - X25519 / Ed25519
  - ChaCha20-Poly1305 / XChaCha20-Poly1305
  - HKDF-SHA256 / SHA-256
  - Secure random number generation on honest devices

## CRYPTOGRAPHIC PRIMITIVES

| Use | Algorithm |
|-----|-----------|
| Group state signatures | Ed25519 |
| Sealed secret delivery | X25519 ECDH + HKDF-SHA256 + ChaCha20-Poly1305 |
| Message encryption (preferred) | ChaCha20-Poly1305 with 12-byte nonces |
| Message encryption (optional) | XChaCha20-Poly1305 with 24-byte nonces |
| KDF / hash | HKDF-SHA256, SHA-256 |
| Random secrets | 32 bytes from CSPRNG |

HKDF domain labels (ASCII):

- RNIMKE1-GRS
- RNIMKE1-EPOCH
- RNIMKE1-SENDER
- RNIMKE1-MSG
- RNIMKE1-SEAL
- RNIMKE1-STATE
- RNIMKE1-NONCE

# Roles and objects

R-NIMKE does not invent a parallel identity system. Peers are ordinary Reticulum Identities. An RNS Identity already carries a X25519 encryption key and a Ed25519 signing key (64-byte public key = 32-byte X25519 + 32-byte Ed25519). R-NIMKE uses those halves directly.

The administrator is a Reticulum Identity that creates the group, signs states with that identity's Ed25519 key, samples the Group Root Secret, and seals it to members.

A member is a Reticulum Identity that holds the current Group Root Secret and sends and receives group messages. Sealed delivery uses that identity's existing X25519 encryption key. No separate MemberPub keypair is generated.

Propagation Nodes are untrusted. They store and forward opaque data only.

Core values:

| Name | Size | Notes |
|------|------|--------|
| GroupID | 16 bytes | Random at creation. Fixed for the life of the group |
| AdminPub | 32 bytes | Ed25519 half of the admin RNS Identity public key |
| MemberID | 16 bytes | Unique in the group |
| MemberPub | 32 bytes | X25519 half of the member RNS Identity public key. Used for sealed GRS delivery |
| GRS_v | 32 bytes | Group Root Secret for state version v. Never sent in the clear |
| state_seq | uint32 | Monotonic. Starts at 1. Bumps on membership change or rekey |
| epoch t | uint32 | Local ratchet index |

---

EXPERIMENTAL - DO NOT USE

DRAFT