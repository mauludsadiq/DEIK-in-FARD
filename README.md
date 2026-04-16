# DEK in FARD

**Deterministic Execution Kernel** — a canonical binary serialization and execution integrity system implemented in pure FARD.

DEK makes context, program identity, execution, licensing, and chain of custody content-addressable and locally verifiable. Non-determinism is never hidden. Every oracle call is typed, sequenced, and witnessed. The full verification predicate is:

    valid_receipt AND licensed AND policy_compliant

## Quick start

    fardrun run --program main.fard --out out/run_main

    # Unit tests
    fardrun test --program tests/kernel/test_encode_smoke.fard
    fardrun test --program tests/kernel/test_encode_vectors.fard
    fardrun test --program tests/kernel/test_encode_types.fard
    fardrun test --program tests/kernel/test_cid.fard
    fardrun test --program tests/bundle/test_validate.fard
    fardrun test --program tests/witness/test_receipt.fard
    fardrun test --program tests/exec/test_engine.fard
    fardrun test --program tests/verify/test_verify.fard
    fardrun test --program tests/license/test_license.fard
    fardrun test --program tests/context/test_context.fard
    fardrun test --program tests/persist/test_persist.fard
    fardrun test --program tests/replay/test_replay.fard
    fardrun test --program tests/oracle/test_oracle.fard
    fardrun test --program tests/policy/test_policy.fard

    # Compliance vectors
    fardrun test --program compliance/vectors/comp_canonical_bytes.fard
    fardrun test --program compliance/vectors/comp_cid.fard
    fardrun test --program compliance/vectors/comp_bundle.fard
    fardrun test --program compliance/vectors/comp_receipt.fard
    fardrun test --program compliance/vectors/comp_policy.fard
    fardrun test --program compliance/vectors/comp_context.fard
    fardrun test --program compliance/vectors/comp_license.fard
    fardrun test --program compliance/vectors/comp_replay.fard

## Architecture

    docs/
      Whitepaper.md

    demos/
      drone/demo_drone.fard       -- autonomous systems: sensor canonicalization, safety policy, replay proof
      enterprise/demo_enterprise.fard  -- licensing chain: registry, policy, verification, revocation
      llm/demo_llm.fard           -- LLM integrity: context pinning, oracle witnessing, truncation detection

    packages/
      kernel-canon/src/
        encode.fard       -- deterministic serializer (27 tag types)
        types.fard        -- canonical value constructors
        cid.fard          -- content addressing: sha256(encode(v))
      kernel-bundle/src/
        validate.fard     -- bundle admission gatekeeper
      kernel-witness/src/
        receipt.fard      -- execution witness with CID chaining
      kernel-exec/src/
        engine.fard       -- full exec pipeline: validate+cid+eval+receipt
      kernel-verify/src/
        verify.fard       -- receipt and chain verification
      kernel-license/src/
        license.fard      -- program CID registry and licensing gate
      kernel-context/src/
        context.fard      -- canonical context binding and truncation detection
      kernel-persist/src/
        persist.fard      -- registry and context save/load with CID stability
      kernel-replay/src/
        replay.fard       -- determinism proof via receipt comparison
      kernel-oracle/src/
        oracle.fard       -- canonical oracle boundary: time/random/http/sensor/model/ffi
      kernel-policy/src/
        policy.fard       -- policy enforcement: programs/oracles/exits/steps/expiry/revocation

## Phase ladder

    Phase 1  - Canon             done  encode.fard, types.fard
    Phase 2  - Addressing        done  cid.fard
    Phase 3  - Bundle admission  done  validate.fard
    Phase 4  - Receipt formation done  receipt.fard
    Phase 5  - Execution         done  engine.fard
    Phase 6  - Verification      done  verify.fard
    Phase 7  - Licensing         done  license.fard
    Phase 8  - Context binding   done  context.fard
    Phase 9  - Persistence       done  persist.fard
    Phase 10 - Replay            done  replay.fard
    Phase 11 - Oracle boundary   done  oracle.fard
    Phase 12 - Policy layer      done  policy.fard

    Next: service boundary

## Test and compliance summary

    183 unit tests,  0 failures
     85 compliance vectors, 0 failures

    Unit tests:
      tests/kernel/test_encode_smoke.fard      2
      tests/kernel/test_encode_vectors.fard   11
      tests/kernel/test_encode_types.fard     18
      tests/kernel/test_cid.fard               7
      tests/bundle/test_validate.fard         11
      tests/witness/test_receipt.fard         13
      tests/exec/test_engine.fard             11
      tests/verify/test_verify.fard           12
      tests/license/test_license.fard         17
      tests/context/test_context.fard         16
      tests/persist/test_persist.fard         14
      tests/replay/test_replay.fard           13
      tests/oracle/test_oracle.fard           22
      tests/policy/test_policy.fard           27

    Compliance vectors:
      compliance/vectors/comp_canonical_bytes.fard   25
      compliance/vectors/comp_cid.fard               12
      compliance/vectors/comp_bundle.fard            12
      compliance/vectors/comp_receipt.fard           10
      compliance/vectors/comp_policy.fard            12
      compliance/vectors/comp_context.fard           11
      compliance/vectors/comp_license.fard            8
      compliance/vectors/comp_replay.fard             7

## Compliance

The compliance suite contains normative test vectors derived from the frozen
DEK v1 spec. Any conforming implementation in any language must produce
identical results for all vectors.

Stable CIDs that every implementation must reproduce:

    cid(null)         sha256:6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d
    cid(bool false)   sha256:4bf5122f344554c53bde2ebb8cd2b7e3d1600ad631c385a5d7cce23c7785459a
    cid(bool true)    sha256:dbc1b4c900ffe48d575b5da5c638040125f65db0fe3e24494b76ea986457d986
    cid(int 0)        sha256:dc4c8669df128318c5790c414c870cc76c585268552851e78d3ee8604dbec0e3
    cid(text "")      sha256:6c449f91c1adbf3945ad078f5f875c0c1f133f246c4588668faffbe23a3c195f
    cid(list [])      sha256:7e2e8b49f93a4f1fcd3d8c53db08bcd2fb714f1d91d4db7ed0b8786c572f9164
    cid(record [])    sha256:92c0a3cd8b571ac5634aa066fcc63d7b62155c36126a29231d97692ff2944875
    cid(option_none)  sha256:01ba4719c80b6fe911b091a7c05124b64eeece964e09c058ef8f9805daca546b

## Frozen spec

    spec/dek_v1/
      canonical_values_v1.md    -- type definitions and constructors
      canonical_bytes_v1.md     -- byte encoding format and tag table
      bundle_v1.md              -- bundle schema and validation rules
      receipt_v1.md             -- receipt structure and CID chaining
      verify_v1.md              -- verification contract
      license_registry_v1.md   -- license registry schema
      context_v1.md             -- context binding and truncation detection
      persist_v1.md             -- persistence contract
      oracle_v1.md              -- oracle boundary and event schema
      policy_v1.md              -- policy enforcement contract

## Verification

The complete verification predicate:

    valid_receipt(run_id)
    AND licensed(run_id, program_cid, registry)
    AND policy_compliant(policy, program_cid, exit_code, oracle_kinds, step_count, now_ms)

Each gate is independent and composable.

## Execution pipeline

    exec_bundle(bundle, evaluator) -> { ok: { receipt, receipt_cid, exit, result } }
                                    | { err: { code, message, phase, details } }

Internal phases:

    1. validate_bundle(bundle)
    2. cid_of(program), cid_of(input)
    3. evaluator(program, input, config)
    4. cid_of(result)
    5. make_receipt(...)
    6. receipt_cid(receipt)

## Policy enforcement

    enforce(policy, program_cid, exit_code, oracle_kinds, step_count, now_ms)

Constraints checked in order:
    revoked, expires_at, allowed_programs, allowed_exits, max_step_count, allowed_oracles

null in any list field means allow all.

## Oracle boundary

    oracle_now(seq)
    oracle_random_int(lo, hi, seq)
    oracle_random_uuid(seq)
    oracle_http_get(url, seq)
    oracle_sensor(name, data, seq)
    oracle_model_call(model, prompt, response, seq)

Each returns { ok: { value, event, cid } }.
A snapshot of all events is a canonical value with its own CID.

## Replay

    replay_check(program, input, evaluator)
    -> { ok: { receipt_cid, deterministic: true, runs: 2 } }

## Licensing

    let registry = license.make_registry(["sha256:...", ...])
    persist.save_registry(registry, "registries/v1.json")
    let r = persist.load_registry("registries/v1.json")
    license.verify_licensed_chain(run_id, program_cid, r.ok.registry)

## Context integrity

    let context = ctx.make_context([msg.ok, ...], "model-name", turn_count)
    let original_cid = ctx.context_cid(context).ok
    persist.save_context(context, "contexts/session.json")
    ctx.truncation_check(original_cid, current_context)
    -- { ok: { cid, verified: true } }
    -- { err: { code: "CONTEXT_MODIFIED", original, current } }

## Encoding format

| Tag | Hex  | Type            | Payload                                              |
|-----|------|-----------------|------------------------------------------------------|
| 0   | 0x00 | null            | (none)                                               |
| 1   | 0x01 | bool false      | (none)                                               |
| 2   | 0x02 | bool true       | (none)                                               |
| 3   | 0x03 | int64           | 8 bytes big-endian two's complement                  |
| 4   | 0x04 | float64         | 8 bytes IEEE 754 big-endian                          |
| 5   | 0x05 | text            | u64_be(len) ++ utf8_bytes                            |
| 6   | 0x06 | bytes           | u64_be(len) ++ raw_bytes                             |
| 7   | 0x07 | list            | u64_be(count) ++ encode(item)*                       |
| 8   | 0x08 | record          | u64_be(count) ++ (encode_text(key) ++ encode(val))*  |
| 9   | 0x09 | error           | encode(record{code,data,message,phase})               |
| 10  | 0x0a | option_none     | (none)                                               |
| 11  | 0x0b | option_some     | encode(inner)                                        |
| 12  | 0x0c | result_ok       | encode(inner)                                        |
| 13  | 0x0d | result_err      | encode(inner)                                        |
| 14  | 0x0e | cid_ref         | u64_be(len) ++ utf8_bytes                            |
| 15  | 0x0f | trace_frame     | encode(record v)                                     |
| 16  | 0x10 | oracle_event    | encode(record v)                                     |
| 17  | 0x11 | bundle          | encode(record v)                                     |
| 18  | 0x12 | digest          | encode(record v)                                     |
| 19  | 0x13 | program_ref     | encode(record v)                                     |
| 20  | 0x14 | dep_list        | encode(record v)                                     |
| 21  | 0x15 | sensor_snapshot | encode(record v)                                     |
| 22  | 0x16 | timestamp       | encode(record v)                                     |
| 23  | 0x17 | bigint          | sign_byte ++ u64_be(len) ++ mag_bytes                |
| 24  | 0x18 | map             | u64_be(count) ++ (encode(key) ++ encode(val))*       |
| 25  | 0x19 | set             | u64_be(count) ++ encode(item)*                       |
| 26  | 0x1a | receipt         | encode(record v)                                     |

## FARD language notes

Inline if/else inside record literals inside list literals inside function bodies
previously required extraction to helper functions due to a VM stack bug.
This bug has been fixed in FARD v0.5 (commit 2b705ce).
Inline conditional values now work correctly:

    { k: "field", v: if x == null then { t: "option_none" } else types.make_text(x) }

The workaround functions opt_text/opt_int have been removed from receipt.fard.

## Properties

- Deterministic: same input produces same bytes, always
- Self-describing: tag byte first, no schema required
- Content-addressable: sha256(encode(v)) is a stable CID
- Evaluator-agnostic: engine accepts any fn(program, input, config)
- Licensable: program CID registries are versioned and verifiable
- Tamper-evident: context truncation detectable by CID
- Replayable: determinism provable by running twice
- Oracle-transparent: every non-deterministic call is witnessed
- Policy-enforced: programs/oracles/exits/steps/expiry/revocation
- Conformance-testable: 85 normative compliance vectors
- FARD-native: no external dependencies, runs under fardrun
