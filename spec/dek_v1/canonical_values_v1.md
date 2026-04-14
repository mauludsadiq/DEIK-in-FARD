# DEK Canonical Values v1

Status: FROZEN
Version: 1.0.0

## Overview

A canonical value is a typed record used as input to the DEK encoder.
Every canonical value has a `t` field identifying its type.
The same canonical value always encodes to the same bytes.

## Type Definitions

### Primitives

    { t: "null" }
    { t: "bool", v: true }
    { t: "bool", v: false }
    { t: "int", v: <int64> }
    { t: "float", bits_hex: "<16 hex chars>" }
    { t: "text", v: "<utf8 string>" }
    { t: "bytes", v: <Bytes> }

### Collections

    { t: "list", v: [<canon_val>, ...] }
    { t: "record", v: [{ k: "<key>", v: <canon_val> }, ...] }
    { t: "map", v: [{ k: <canon_val>, v: <canon_val> }, ...] }
    { t: "set", v: [<canon_val>, ...] }

### Wrappers

    { t: "option_none" }
    { t: "option_some", v: <canon_val> }
    { t: "result_ok", v: <canon_val> }
    { t: "result_err", v: <canon_val> }

### References

    { t: "cid", v: "<sha256:hex>" }

### Extended

    { t: "bigint", sign: "zero"|"positive"|"negative", mag_hex: "<hex>" }
    { t: "error", code: "<text>", message: "<text>", data: <canon_val>, phase: "<text>" }

### Kernel Types

    { t: "receipt", v: <canon_val> }
    { t: "trace_frame", v: <canon_val> }
    { t: "oracle_event", v: <canon_val> }
    { t: "bundle", v: <canon_val> }
    { t: "digest", v: <canon_val> }
    { t: "program_ref", v: <canon_val> }
    { t: "dep_list", v: <canon_val> }
    { t: "sensor_snapshot", v: <canon_val> }
    { t: "timestamp", v: <canon_val> }

## Constructors

Available via `packages/kernel-canon/src/types.fard`:

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

## Invariants

- Field order in record entries is fixed at construction time
- Record keys are text strings
- Float values are specified as IEEE 754 big-endian hex, not as floats
- Bigint magnitude is big-endian hex with no leading zeros (except zero itself)
- CID values are strings of the form "sha256:<64 hex chars>"
