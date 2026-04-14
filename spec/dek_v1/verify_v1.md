# DEK Verification v1

Status: FROZEN
Version: 1.0.0

## Overview

DEK verification checks that a receipt exists, is well-formed, and that its
chain of custody is intact.

## verify_receipt(run_id)

Looks up a receipt by run_id (a sha256: string).

Lookup order:
1. Local file: receipts/sha256_<hex>.json
2. Remote: GET {FARD_REGISTRY_URL}/receipt/{run_id} if env var is set

Returns:
    { ok: { run_id, receipt } }
    { err: { code: "RECEIPT_NOT_FOUND", message, run_id } }

## verify_chain(run_id)

Walks the full derived_from chain recursively, verifying every node.

Returns:
    { ok: { depth } }
    { err: { code: "CHAIN_BROKEN", message, details } }

## verify_program_cid(run_id, expected_program_cid)

Verifies the receipt exists and is accessible.

Returns:
    { ok: { run_id, verified, receipt } }
    { err: ... }

## Error Codes

    RECEIPT_NOT_FOUND    run_id not in local store or registry
    CHAIN_BROKEN         a node in the derived_from chain is missing or invalid

## Reference Implementation

    packages/kernel-verify/src/verify.fard
