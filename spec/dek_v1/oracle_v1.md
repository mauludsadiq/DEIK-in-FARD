# DEK Oracle Boundary v1

Status: FROZEN
Version: 1.0.0

## Overview

An oracle is any source of non-determinism in a DEK execution.
The oracle boundary ensures that non-determinism is never hidden.
Every oracle call produces a canonical oracle_event value that is
typed, sequenced, and content-addressed.

## Oracle Event Structure

    { t: "oracle_event",
      v: { t: "record", v: [
        { k: "input",  v: <canon_val> },
        { k: "kind",   v: { t: "text", v: "<kind>" } },
        { k: "output", v: <canon_val> },
        { k: "seq",    v: { t: "int",  v: <n> } }
      ] }
    }

## Oracle Kinds

    "time"        wall clock and timestamp calls
    "random"      random number and UUID generation
    "http"        external HTTP responses
    "sensor"      physical sensor readings
    "model_call"  external LLM or ML model responses
    "ffi"         foreign function interface calls

## Sequence Numbers

Every oracle event carries a seq integer. Sequence numbers are
assigned by the caller and must be strictly increasing within
one execution. Two events with different seq values produce
different CIDs even if their inputs and outputs are identical.

## Oracle Snapshot

A snapshot is a canonical list of oracle events from one execution.

    make_snapshot([event, ...]) -> { t: "list", v: [event, ...] }
    snapshot_cid(snapshot)      -> { ok: "sha256:..." }

The snapshot CID can be stored in the receipt's oracle_cid field,
binding the complete oracle history to the execution witness.

## Oracle Functions

### time

    oracle_now(seq)
    input:  { t: "text", v: "now" }
    output: { t: "int", v: <unix_ms> }

### random

    oracle_random_int(lo, hi, seq)
    input:  { t: "record", v: [{ k: "hi", ... }, { k: "kind", ... }, { k: "lo", ... }] }
    output: { t: "int", v: <n> }

    oracle_random_uuid(seq)
    input:  { t: "text", v: "uuid_v4" }
    output: { t: "text", v: "<uuid>" }

### http

    oracle_http_get(url, seq)
    input:  { t: "record", v: [{ k: "method", v: "GET" }, { k: "url", v: <url> }] }
    output: { t: "record", v: [{ k: "body", v: <text> }, { k: "status", v: <int> }] }

### sensor

    oracle_sensor(name, data, seq)
    input:  { t: "text", v: <sensor_name> }
    output: <any canonical value>

### model_call

    oracle_model_call(model, prompt, response, seq)
    input:  { t: "record", v: [{ k: "model", ... }, { k: "prompt", ... }] }
    output: { t: "text", v: <response> }

### ffi

    make_event("ffi", input_canon, output_canon, seq)
    input:  caller-defined canonical value
    output: caller-defined canonical value

## Replay with Oracles

For deterministic replay of oracle-using executions, the caller must
supply the same oracle snapshot as input. The snapshot replaces live
oracle calls during replay, ensuring the same values are returned.

Without oracle replay, only closed (oracle-free) computations satisfy:
    Replay(Receipt) = Receipt

With oracle replay:
    Replay(Receipt, Snapshot) = Receipt

## Error Codes

    CID_FAILED    CID computation failed on event or snapshot

## Reference Implementation

    packages/kernel-oracle/src/oracle.fard
