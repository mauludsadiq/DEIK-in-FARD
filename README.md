# DEK in FARD

**Deterministic Encoding Kernel** — a canonical binary serialization format implemented in pure FARD.

DEK produces a deterministic, content-addressable byte representation of typed values. The same value always produces the same bytes, regardless of platform or runtime. This makes encoded values suitable for SHA-256 content addressing, cryptographic witnessing, and FARD receipt chains.

## Quick start

    fardrun run --program main.fard --out out/run_main
    fardrun test --program tests/kernel/test_encode_smoke.fard
    fardrun test --program tests/kernel/test_encode_vectors.fard
    fardrun test --program tests/kernel/test_encode_types.fard
    fardrun test --program tests/kernel/test_cid.fard

## Phases

    Phase 1 - Canon         done  encode.fard, types.fard
    Phase 2 - Addressing    done  cid.fard
    Phase 3 - Bundle admission      validate.fard (next)
    Phase 4 - Receipt formation     receipt.fard
    Phase 5 - Execution             engine.fard

## Content addressing

    cid_of(canon_val) -> { ok: "sha256:..." }

The CID is sha256(encode(v)) hex-encoded with a sha256: prefix.
Same canonical value always produces the same CID.
Different canonical values always produce different CIDs.

Known stable CIDs:

    cid_of(null)         sha256:6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d
    cid_of(bool false)   sha256:4bf5122f344554c53bde2ebb8cd2b7e3d1600ad631c385a5d7cce23c7785459a
    cid_of(bool true)    sha256:dbc1b4c900ffe48d575b5da5c638040125f65db0fe3e24494b76ea986457d986

## Format

Every encoded value starts with a 1-byte tag. The tag determines the layout of the remaining bytes.

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

## Canonical value format

The encoder expects values in canonical record form using the helpers in types.fard:

    make_null()
    make_bool(true)
    make_int(42)
    make_float_bits("3ff0000000000000")   -- IEEE 754 hex
    make_text("hello")
    make_bytes(<Bytes>)
    make_list([canon_val, ...])
    make_record([{ k: "key", v: canon_val }, ...])
    make_cid("sha256:abc...")

Wrapped types use raw records:

    { t: "option_none" }
    { t: "option_some", v: canon_val }
    { t: "result_ok",   v: canon_val }
    { t: "result_err",  v: canon_val }
    { t: "bigint", sign: "positive", mag_hex: "ff..." }
    { t: "map", v: [{ k: canon_val, v: canon_val }, ...] }
    { t: "set", v: [canon_val, ...] }

## Structure

    packages/kernel-canon/src/
      encode.fard   -- encoder (27 tag types)
      types.fard    -- canonical value constructors
      cid.fard      -- content addressing: sha256(encode(v))
    tests/kernel/
      test_encode_smoke.fard    -- basic run check (2 tests)
      test_encode_vectors.fard  -- exact byte vector tests (11 tests)
      test_encode_types.fard    -- full type coverage (18 tests)
      test_cid.fard             -- content addressing (7 tests)
    main.fard       -- demo: encodes {answer: 42, label: "DEK"} -> 59 bytes

## Test summary

    38 tests, 0 failures

## Properties

- Deterministic: same input produces same bytes, always
- Self-describing: tag byte first, no schema required to parse length
- Content-addressable: sha256(encode(v)) is a stable CID for any value
- FARD-native: no external dependencies, runs under fardrun
