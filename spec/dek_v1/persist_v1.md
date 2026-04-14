# DEK Persistence v1

Status: FROZEN
Version: 1.0.0

## Overview

DEK persistence provides save/load operations for canonical values with CID
verification. Every load recomputes the CID and returns it alongside the value
so callers can verify integrity after loading.

## save_registry(registry, path)

Encodes registry as JSON and writes to path.

Returns:
    { ok: { path, registry_cid } }
    { err: { code: "WRITE_FAILED", message, path } }
    { err: { code: "CID_FAILED", message } }

## load_registry(path)

Reads and decodes registry from path, recomputes CID.

Returns:
    { ok: { registry, registry_cid } }
    { err: { code: "READ_FAILED", message, path } }
    { err: { code: "DECODE_FAILED", message, path } }
    { err: { code: "CID_FAILED", message } }

## save_context(context, path)

Encodes context as JSON and writes to path.

Returns:
    { ok: { path, context_cid } }
    { err: { code: "WRITE_FAILED", message, path } }
    { err: { code: "CID_FAILED", message } }

## load_context(path)

Reads and decodes context from path, recomputes CID.

Returns:
    { ok: { context, context_cid } }
    { err: { code: "READ_FAILED", message, path } }
    { err: { code: "DECODE_FAILED", message, path } }
    { err: { code: "CID_FAILED", message } }

## Roundtrip Invariant

For any canonical value v saved to path p:

    load(p).ok.cid == save(v, p).ok.cid

## Reference Implementation

    packages/kernel-persist/src/persist.fard
