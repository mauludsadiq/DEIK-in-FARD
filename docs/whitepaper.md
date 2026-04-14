# DEK Whitepaper

**Deterministic Execution Kernel**
Version 1.0

---

## Abstract

DEK is a deterministic execution kernel that makes context, program identity,
execution, licensing, and chain of custody content-addressable and locally
verifiable. It is implemented in pure FARD and requires no external
dependencies. Any execution wrapped in DEK produces a cryptographic receipt
that proves what ran, what it consumed, and what it produced. The same inputs
always produce the same receipt. Non-determinism is never hidden.

---

## 1. The Problem

Modern AI systems, autonomous controllers, and enterprise software share a
common structural weakness: their outputs cannot be independently verified.

When an LLM produces an answer, there is no proof of:
- what context it received
- whether that context was complete
- which version of the model ran
- whether the output was policy-compliant

When a drone executes a control step, there is no proof of:
- what sensor state triggered the command
- which software version made the decision
- whether the decision was deterministic

When enterprise software runs a licensed program, there is no proof of:
- whether the program was actually licensed
- whether it ran within policy constraints
- whether the output can be replicated

DEK addresses all of these gaps with a single mechanism: deterministic
content-addressed execution.

---

## 2. Core Invariants

DEK provides the following invariants that hold for every wrapped execution:

**Invariant 1: Determinism**
The same canonical input always produces the same canonical output.
Same inputs => same bytes => same CIDs => same receipt.

**Invariant 2: Content Addressing**
Every value, context, program, result, and receipt has a stable CID.
CID = sha256(encode(value)) where encode is the DEK canonical serializer.

**Invariant 3: Oracle Transparency**
Non-determinism is never hidden. Every call to time, random, HTTP, sensor,
model, or FFI produces a typed, sequenced, content-addressed oracle event.
An execution's complete oracle history is itself a canonical value.

**Invariant 4: Receipt Formation**
Every execution produces a receipt binding:
- bundle_cid: what was submitted
- program_cid: what ran
- input_cid: what it consumed
- result_cid: what it produced
- oracle_cid: what non-determinism occurred
- exit: how it completed
- kernel_version: which DEK ran it

**Invariant 5: Verifiability**
Receipts are locally verifiable without contacting any external service.
Chain verification walks the full derived_from ancestry.

**Invariant 6: Replay**
For deterministic executions, Replay(Receipt) = Receipt.
The same program with the same input always produces the same receipt.

---

## 3. Architecture

DEK is organized as a stack of 12 composable packages:

    kernel-canon     Deterministic serialization (27 typed tags)
    kernel-bundle    Bundle validation and admission
    kernel-witness   Receipt formation and CID chaining
    kernel-exec      Execution pipeline
    kernel-verify    Receipt and chain verification
    kernel-license   Program CID registry and licensing gate
    kernel-context   Context binding and truncation detection
    kernel-persist   Durable storage with CID stability
    kernel-replay    Determinism proof
    kernel-oracle    Non-determinism boundary
    kernel-policy    Policy enforcement
    (compliance)     Normative test vectors

Each package is independent and composable. A system can adopt any subset.

---

## 4. The Canonical Byte Format

Every DEK value is encoded as a self-describing byte sequence starting with
a 1-byte tag. The format is defined in canonical_bytes_v1.md.

Key properties:
- No schema required to parse length or type
- All integers big-endian
- All lengths 8 bytes (u64)
- Floats as IEEE 754 hex bits
- Bigints as sign byte + magnitude

The same canonical value always produces the same bytes on every platform,
runtime, and implementation.

---

## 5. Content Addressing

CID derivation:

    cid(v) = "sha256:" ++ hex(sha256(encode(v)))

Stable reference CIDs:

    cid(null)        sha256:6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d
    cid(false)       sha256:4bf5122f344554c53bde2ebb8cd2b7e3d1600ad631c385a5d7cce23c7785459a
    cid(true)        sha256:dbc1b4c900ffe48d575b5da5c638040125f65db0fe3e24494b76ea986457d986
    cid(int 0)       sha256:dc4c8669df128318c5790c414c870cc76c585268552851e78d3ee8604dbec0e3
    cid(text "")     sha256:6c449f91c1adbf3945ad078f5f875c0c1f133f246c4588668faffbe23a3c195f
    cid(list [])     sha256:7e2e8b49f93a4f1fcd3d8c53db08bcd2fb714f1d91d4db7ed0b8786c572f9164
    cid(record [])   sha256:92c0a3cd8b571ac5634aa066fcc63d7b62155c36126a29231d97692ff2944875

---

## 6. Execution Pipeline

    exec_bundle(bundle, evaluator)
    -> { ok: { receipt, receipt_cid, exit, result } }
    -> { err: { code, message, phase, details } }

Six internal phases:

    1. validate_bundle     reject malformed bundles early
    2. cid_of(inputs)      content-address program and input
    3. evaluator(...)      caller-supplied execution logic
    4. cid_of(result)      content-address the output
    5. make_receipt(...)   form cryptographic witness
    6. receipt_cid(...)    content-address the receipt itself

The evaluator is injected by the caller. Any function with signature
fn(program, input, config) -> { ok: result } | { err: ... } is valid.
This keeps the kernel pure and the execution logic replaceable.

---

## 7. The Verification Predicate

The full DEK verification predicate is:

    valid_receipt(run_id)
    AND licensed(program_cid, registry)
    AND policy_compliant(policy, program_cid, exit, oracles, steps, now)

Each gate is independent and composable. A system can enforce any subset.

**valid_receipt**: receipt exists and is well-formed
**licensed**: program CID appears in a content-addressed registry
**policy_compliant**: all policy constraints satisfied

Policy constraints:
- allowed program CIDs
- allowed oracle kinds
- allowed exit codes
- maximum step count
- expiry timestamp
- revocation flag

---

## 8. Oracle Boundary

Every non-deterministic call must go through the oracle layer:

    oracle_now(seq)                              time
    oracle_random_int(lo, hi, seq)               random integer
    oracle_random_uuid(seq)                      UUID
    oracle_http_get(url, seq)                    HTTP response
    oracle_sensor(name, data, seq)               physical sensor
    oracle_model_call(model, prompt, resp, seq)  LLM/ML model

Each call produces a canonical oracle_event with:
- kind: the type of non-determinism
- input: what was requested
- output: what was returned
- seq: sequence number within the execution

A snapshot of all oracle events is itself a canonical value.
Its CID can be stored in the receipt's oracle_cid field.

The oracle contract: non-determinism is never hidden.
Determinism is complete only for oracle-free computations.
For oracle-using computations, replay requires the same snapshot.

---

## 9. Context Integrity

For LLM applications, DEK provides context window integrity:

    context = make_context(messages, model, turn_count)
    original_cid = context_cid(context)

At any point:

    truncation_check(original_cid, current_context)
    -> { ok: { verified: true } }
    -> { err: { code: "CONTEXT_MODIFIED", original, current } }

If the context was truncated or modified, the CID changes.
The truncation is detected and the error carries both CIDs as evidence.

---

## 10. What DEK Does Not Claim

DEK does not claim to:
- prevent all forms of tampering after execution
- guarantee the correctness of the evaluator logic
- provide consensus or distributed agreement
- replace cryptographic signing of outputs
- verify the identity of the party running the kernel

DEK claims only that: given the same inputs, the same kernel version will
produce the same outputs, and that this fact is locally verifiable by anyone
with the receipt.

---

## 11. Conformance

Any implementation in any language is conformant with DEK v1 if and only if
it produces identical results for all compliance vectors defined in:

    compliance/vectors/

The compliance vectors are normative. The FARD implementation is the
reference implementation.

---

## 12. Reference Implementations and Demos

Three reference demos are provided:

**LLM Demo** (demos/llm/demo_llm.fard)
Context pinning, oracle witnessing, truncation detection, policy compliance.

**Drone Demo** (demos/drone/demo_drone.fard)
Sensor canonicalization, safety policy, deterministic control, audit trail.

**Enterprise Demo** (demos/enterprise/demo_enterprise.fard)
Registry distribution, full verification, unlicensed rejection, revocation.

---

## 13. Conclusion

DEK provides a minimal, composable, locally-verifiable execution integrity
layer. It does not require a blockchain, a network, or a trusted third party.
It requires only that the same value always encodes to the same bytes.

That invariant, combined with SHA-256 content addressing and a structured
receipt format, is sufficient to make any computation auditable, replayable,
licensable, and policy-enforceable.

The kernel is complete. The spec is frozen. The compliance suite is public.
