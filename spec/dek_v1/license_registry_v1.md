# DEK License Registry v1

Status: FROZEN
Version: 1.0.0

## Overview

A license registry is a canonical set of approved program CIDs. The registry
is itself content-addressed so license versions are pinnable and auditable.

## Registry Structure

    { t: "set", v: [{ t: "text", v: "sha256:..." }, ...] }

## Registry CID

    registry_cid = cid(registry_canonical_value)

The registry CID uniquely identifies a specific version of an allowlist.

## verify_licensed(run_id, program_cid, registry)

Verifies:
1. The receipt exists and is valid
2. The program_cid is in the registry

Returns:
    { ok: { run_id, program_cid, registry_cid } }
    { err: { code: "NOT_LICENSED", message, run_id, program_cid } }
    { err: { code: "RECEIPT_NOT_FOUND", message, run_id } }

## verify_licensed_chain(run_id, program_cid, registry)

Same as verify_licensed but also walks the full receipt chain.

Returns:
    { ok: { run_id, program_cid, registry_cid, chain_depth } }
    { err: ... }

## Error Codes

    NOT_LICENSED         program CID not in registry
    RECEIPT_NOT_FOUND    receipt not found
    CHAIN_BROKEN         receipt chain verification failed

## Reference Implementation

    packages/kernel-license/src/license.fard
