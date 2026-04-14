# DEK v1 Release

**Deterministic Execution Kernel — Version 1.0**
Released: April 2026
Repository: https://github.com/mauludsadiq/DEIK-in-FARD

---

## What DEK v1 is

DEK is a deterministic execution and governance kernel implemented in pure FARD.
It makes context, program identity, execution, licensing, and chain of custody
content-addressable and locally verifiable.

The full verification predicate is:

    valid_receipt AND licensed AND policy_compliant

No blockchain. No network. No trusted third party required.

---

## What is included in v1

### Kernel packages (12)

    kernel-canon      Deterministic serialization and content addressing
    kernel-bundle     Bundle validation and admission
    kernel-witness    Receipt formation and CID chaining
    kernel-exec       Execution pipeline
    kernel-verify     Receipt and chain verification
    kernel-license    Program CID registry and licensing gate
    kernel-context    Context binding and truncation detection
    kernel-persist    Durable storage with CID stability
    kernel-replay     Determinism proof
    kernel-oracle     Non-determinism boundary
    kernel-policy     Policy enforcement

### Service boundary

    service/dek_service.fard

Six stable endpoints: exec, verify, replay, license_check, context_check, policy_check.
Unified dispatch via svc.dispatch({ cmd: "...", ... }).

### Frozen v1 spec (10 documents)

    spec/dek_v1/canonical_values_v1.md
    spec/dek_v1/canonical_bytes_v1.md
    spec/dek_v1/bundle_v1.md
    spec/dek_v1/receipt_v1.md
    spec/dek_v1/verify_v1.md
    spec/dek_v1/license_registry_v1.md
    spec/dek_v1/context_v1.md
    spec/dek_v1/persist_v1.md
    spec/dek_v1/oracle_v1.md
    spec/dek_v1/policy_v1.md

### Compliance suite (10 vector files, 118 vectors)

    compliance/vectors/comp_canonical_bytes.fard   25 vectors
    compliance/vectors/comp_cid.fard               12 vectors
    compliance/vectors/comp_bundle.fard            12 vectors
    compliance/vectors/comp_receipt.fard           10 vectors
    compliance/vectors/comp_policy.fard            12 vectors
    compliance/vectors/comp_context.fard           11 vectors
    compliance/vectors/comp_license.fard            8 vectors
    compliance/vectors/comp_replay.fard             7 vectors
    compliance/vectors/comp_oracle.fard            20 vectors
    compliance/vectors/comp_persist.fard           13 vectors

### Reference demos (3)

    demos/llm/demo_llm.fard               LLM context pinning and truncation detection
    demos/drone/demo_drone.fard           Sensor canonicalization and safety policy
    demos/enterprise/demo_enterprise.fard Registry distribution and full verification

### Documentation (3)

    docs/whitepaper.md        What DEK guarantees and what it does not claim
    docs/integration_guide.md How to wrap a system in DEK
    docs/licensing_guide.md   How to issue and verify licenses

---

## Normative specifications

The following documents are normative for DEK v1.
Any conforming implementation must satisfy all of them.

    canonical_values_v1.md   Defines the canonical value type system
    canonical_bytes_v1.md    Defines the byte encoding and tag table
    bundle_v1.md             Defines the bundle schema and validation rules
    receipt_v1.md            Defines the receipt structure and chaining
    verify_v1.md             Defines the verification contract
    license_registry_v1.md   Defines the license registry schema
    context_v1.md            Defines context binding and truncation detection
    persist_v1.md            Defines the persistence contract
    oracle_v1.md             Defines the oracle boundary and event schema
    policy_v1.md             Defines the policy enforcement contract

---

## Conformance

An implementation is conformant with DEK v1 if and only if it produces
identical results for all compliance vectors in compliance/vectors/.

The FARD implementation in this repository is the reference implementation.

Stable CIDs that every conformant implementation must reproduce:

    cid(null)        sha256:6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d
    cid(false)       sha256:4bf5122f344554c53bde2ebb8cd2b7e3d1600ad631c385a5d7cce23c7785459a
    cid(true)        sha256:dbc1b4c900ffe48d575b5da5c638040125f65db0fe3e24494b76ea986457d986
    cid(int 0)       sha256:dc4c8669df128318c5790c414c870cc76c585268552851e78d3ee8604dbec0e3
    cid(text "")     sha256:6c449f91c1adbf3945ad078f5f875c0c1f133f246c4588668faffbe23a3c195f
    cid(list [])     sha256:7e2e8b49f93a4f1fcd3d8c53db08bcd2fb714f1d91d4db7ed0b8786c572f9164
    cid(record [])   sha256:92c0a3cd8b571ac5634aa066fcc63d7b62155c36126a29231d97692ff2944875
    cid(option_none) sha256:01ba4719c80b6fe911b091a7c05124b64eeece964e09c058ef8f9805daca546b

---

## Test counts

    183 unit tests          0 failures
    118 compliance vectors  0 failures
     29 service tests       0 failures
    ────────────────────────────────
    330 total               0 failures

---

## What DEK v1 guarantees

**Determinism**
The same canonical input always produces the same canonical output and the
same receipt CID.

**Content addressing**
Every value, context, program, result, oracle snapshot, registry, policy,
and receipt has a stable SHA-256 CID.

**Oracle transparency**
Every non-deterministic call (time, random, HTTP, sensor, model, FFI) is
typed, sequenced, and content-addressed. Non-determinism is never hidden.

**Receipt formation**
Every execution produces a cryptographic receipt binding program, input,
result, oracle history, exit code, and kernel version.

**Verifiability**
Receipts are locally verifiable without contacting any external service.
Chain verification walks the full derived_from ancestry.

**Replay**
For deterministic executions: Replay(Receipt) = Receipt.

**Context integrity**
Context window truncation and modification are detectable by CID comparison.

**Licensing**
Program CID registries are content-addressed, persistent, and locally verifiable.

**Policy enforcement**
Executions can be gated on allowed programs, oracle kinds, exit codes,
step counts, expiry, and revocation.

---

## What DEK v1 does not guarantee

- The correctness of the evaluator logic supplied by the caller
- The identity of the party running the kernel
- Consensus or distributed agreement between parties
- Prevention of tampering after execution
- Cryptographic signing of outputs (use DEK receipts as inputs to signing)

---

## Out of scope for v1

- A network registry server implementation
- Versioned registry signatures and publisher identity
- On-chain anchoring
- Cross-language SDK implementations
- GUI or dashboard tooling

These are v2 candidates.

---

## Running DEK v1

Requirements: fardrun on PATH.

    # Run a demo
    fardrun run --program demos/llm/demo_llm.fard --out out/llm

    # Run all tests
    fardrun test --program tests/kernel/test_encode_smoke.fard
    fardrun test --program tests/policy/test_policy.fard
    fardrun test --program service/test_service.fard

    # Run compliance suite
    fardrun test --program compliance/vectors/comp_canonical_bytes.fard
    fardrun test --program compliance/vectors/comp_cid.fard

    # Use the service boundary
    import("./service/dek_service") as svc
    svc.dispatch({ cmd: "exec", bundle: my_bundle, evaluator: my_eval })

---

## Runtime bug fixed in this release

codec.hex_decode in FARD v0.5 was returning Val::Text(String::from_utf8(bytes)?)
which crashed on any byte >= 0x80. Fixed to return Val::Bytes(bytes).
This fix is required for float and bigint encoding to work correctly.
Commit: FARD_v0.5 main branch.
