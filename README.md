# DEK in FARD

**Deterministic Execution Kernel** — a canonical binary serialization and execution integrity system implemented in pure FARD.

DEK makes context, program identity, execution, licensing, and chain of custody content-addressable and locally verifiable. Non-determinism is never hidden — every oracle call is typed, sequenced, and witnessed.

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
    Phase 12 - Policy layer            kernel-policy (next)

## Test summary

    156 tests, 0 failures

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

## Execution pipeline

    exec_bundle(bundle, evaluator) -> { ok: { receipt, receipt_cid, exit, result } }
                                    | { err: { code, message, phase, details } }

Internal phases:

    1. validate_bundle(bundle)               reject malformed bundles early
    2. cid_of(program), cid_of(input)        content-address all inputs
    3. evaluator(program, input, config)     caller-supplied execution
    4. cid_of(result)                        content-address the output
    5. make_receipt(...)                     form cryptographic witness
    6. receipt_cid(receipt)                  content-address the receipt itself

## Oracle boundary

Every non-deterministic call produces a canonical oracle_event:

    oracle_now(seq)                          -> { ok: { value, event, cid } }
    oracle_random_int(lo, hi, seq)           -> { ok: { value, event, cid } }
    oracle_random_uuid(seq)                  -> { ok: { value, event, cid } }
    oracle_http_get(url, seq)               -> { ok: { value, event, cid } }
    oracle_sensor(name, data, seq)          -> { ok: { value, event, cid } }
    oracle_model_call(model, prompt, response, seq) -> { ok: { value, event, cid } }

Oracle events are sequenced, typed, and content-addressed.
A snapshot of all events from one execution is itself a canonical value.

    make_snapshot([event, ...]) -> canonical list
    snapshot_cid(snapshot)      -> { ok: "sha256:..." }

Non-determinism is never hidden. Every oracle call is explicit.

## Replay

    replay_check(program, input, evaluator)

Runs the same execution twice and verifies both receipt CIDs match.
Proves determinism without needing a stored receipt.

    { ok: { receipt_cid, deterministic: true, runs: 2 } }
    { err: { code: "NON_DETERMINISTIC", first, second } }

## Licensing

    let registry = license.make_registry(["sha256:...", ...])
    persist.save_registry(registry, "registries/v1.json")

    let r = persist.load_registry("registries/v1.json")
    let gate = license.verify_licensed_chain(run_id, program_cid, r.ok.registry)

    { ok: { run_id, program_cid, registry_cid, chain_depth } }
    { err: { code: "NOT_LICENSED", ... } }

Remote registry: set FARD_REGISTRY_URL for remote receipt lookup.

## Context integrity

    let context = ctx.make_context([msg.ok, ...], "claude-3", turn_count)
    let original_cid = ctx.context_cid(context).ok
    persist.save_context(context, "contexts/session_001.json")

    ctx.truncation_check(original_cid, current_context)
    -- { ok: { cid, verified: true } }
    -- { err: { code: "CONTEXT_MODIFIED", original, current } }

## Content addressing

    cid_of(canon_val) -> { ok: "sha256:..." }

Known stable CIDs:

    cid(null)         sha256:6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d
    cid(bool false)   sha256:4bf5122f344554c53bde2ebb8cd2b7e3d1600ad631c385a5d7cce23c7785459a
    cid(bool true)    sha256:dbc1b4c900ffe48d575b5da5c638040125f65db0fe3e24494b76ea986457d986

## Bundle format

Required fields:

    program     { t: "cid", v: "sha256:..." } | { t: "text", v: "<source>" }
    input       any canonical value
    trace_mode  "full" | "minimal" | "none"

Optional fields:

    deps        list of { name: text, cid: text } entries
    oracle      { t: "cid", v: "sha256:..." }
    config      record of runtime settings
    version     text

## Receipt format

    make_receipt(
      bundle_cid, program_cid, input_cid, result_cid,
      exit_code, kernel_version,
      deps_cid, oracle_cid, trace_cid, step_count, parent_digest
    )

Exit codes: "ok" | "err" | "abort"
Parent digest enables receipt chaining across execution steps.

## Encoding format

Every encoded value starts with a 1-byte tag.

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

## Canonical value constructors

    make_null()
    make_bool(b)
    make_int(n)
    make_float_bits(hex)
    make_text(s)
    make_bytes(bs)
    make_list(xs)
    make_record(entries)
    make_cid(cid)
    make_set(xs)
    make_map(entries)

## Known FARD language behaviour

Inline if/else expressions inside list literals evaluate both branches eagerly.
Extract conditional logic to helper functions:

    -- Crashes if x is null:
    { k: "field", v: if x == null then default_val else types.make_text(x) }

    -- Works correctly:
    fn opt_text(x) { if x == null then { t: "option_none" } else types.make_text(x) }
    { k: "field", v: opt_text(x) }

## Properties

- Deterministic: same input produces same bytes, always
- Self-describing: tag byte first, no schema required to parse length
- Content-addressable: sha256(encode(v)) is a stable CID for any value
- Evaluator-agnostic: engine accepts any fn(program, input, config) as evaluator
- Licensable: program CID registries are versioned, persistent, and verifiable
- Tamper-evident: context truncation and modification are detectable by CID
- Replayable: determinism is provable by running twice and comparing receipt CIDs
- Oracle-transparent: every non-deterministic call is typed, sequenced, and witnessed
- FARD-native: no external dependencies, runs under fardrun
