# DEK in FARD

**Deterministic Execution Kernel** — a canonical binary serialization and execution integrity system implemented in pure FARD.

DEK makes context, program identity, execution, licensing, and chain of custody content-addressable and locally verifiable. Non-determinism is never hidden. Every oracle call is typed, sequenced, and witnessed. The full verification predicate is:

    valid_receipt AND licensed AND policy_compliant

## Quick start

    fardrun run --program main.fard --out out/run_main
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

## Architecture

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

    Next: compliance suite, reference demos, whitepaper

## Test summary

    183 tests, 0 failures

    tests/kernel/test_encode_smoke.fard      2 tests
    tests/kernel/test_encode_vectors.fard   11 tests
    tests/kernel/test_encode_types.fard     18 tests
    tests/kernel/test_cid.fard               7 tests
    tests/bundle/test_validate.fard         11 tests
    tests/witness/test_receipt.fard         13 tests
    tests/exec/test_engine.fard             11 tests
    tests/verify/test_verify.fard           12 tests
    tests/license/test_license.fard         17 tests
    tests/context/test_context.fard         16 tests
    tests/persist/test_persist.fard         14 tests
    tests/replay/test_replay.fard           13 tests
    tests/oracle/test_oracle.fard           22 tests
    tests/policy/test_policy.fard           27 tests

## Frozen spec

    spec/dek_v1/
      canonical_values_v1.md
      canonical_bytes_v1.md
      bundle_v1.md
      receipt_v1.md
      verify_v1.md
      license_registry_v1.md
      context_v1.md
      persist_v1.md
      oracle_v1.md
      policy_v1.md

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
    -> { ok: true }
    -> { ok: false, code, message, phase: "policy.enforce" }

Policy constraints (checked in order):

    revoked          policy has been revoked
    expires_at       unix ms expiry timestamp
    allowed_programs list of permitted program CIDs
    allowed_exits    list of permitted exit codes
    max_step_count   execution step cap
    allowed_oracles  list of permitted oracle kinds

null in any list field means allow all.

## Oracle boundary

    oracle_now(seq)
    oracle_random_int(lo, hi, seq)
    oracle_random_uuid(seq)
    oracle_http_get(url, seq)
    oracle_sensor(name, data, seq)
    oracle_model_call(model, prompt, response, seq)

Each returns { ok: { value, event, cid } }.
Oracle events are sequenced and content-addressed.
A snapshot of all events is itself a canonical value.

## Replay

    replay_check(program, input, evaluator)
    -> { ok: { receipt_cid, deterministic: true, runs: 2 } }

Runs execution twice and verifies receipt CIDs match.

## Licensing

    let registry = license.make_registry(["sha256:...", ...])
    persist.save_registry(registry, "registries/v1.json")
    let r = persist.load_registry("registries/v1.json")
    license.verify_licensed_chain(run_id, program_cid, r.ok.registry)

## Context integrity

    let context = ctx.make_context([msg.ok, ...], "claude-3", turn_count)
    let original_cid = ctx.context_cid(context).ok
    persist.save_context(context, "contexts/session_001.json")
    ctx.truncation_check(original_cid, current_context)

## Content addressing

Known stable CIDs:

    cid(null)        sha256:6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d
    cid(false)       sha256:4bf5122f344554c53bde2ebb8cd2b7e3d1600ad631c385a5d7cce23c7785459a
    cid(true)        sha256:dbc1b4c900ffe48d575b5da5c638040125f65db0fe3e24494b76ea986457d986

## Bundle format

Required: program, input, trace_mode
Optional: deps, oracle, config, version

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

## Known FARD language behaviour

Inline if/else inside list literals evaluates both branches eagerly.
Extract to helper functions:

    fn opt_text(x) { if x == null then { t: "option_none" } else types.make_text(x) }

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
- FARD-native: no external dependencies, runs under fardrun
