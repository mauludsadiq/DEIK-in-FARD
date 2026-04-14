# DEK Receipt v1

Status: FROZEN
Version: 1.0.0

## Overview

A receipt is a canonical value that cryptographically witnesses a completed
execution. It binds together the CIDs of all inputs and outputs so the
execution can be verified and replayed.

## Structure

A receipt is a canonical value of type "receipt" whose v field is a canonical
list of single-entry canonical records, sorted alphabetically by key:

    bundle_cid      text   CID of the validated bundle
    deps_cid        text | option_none
    exit            text   "ok" | "err" | "abort"
    input_cid       text   CID of the input value
    kernel_version  text   e.g. "fard-0.5"
    oracle_cid      text | option_none
    parent_digest   text | option_none   CID of parent receipt
    program_cid     text   CID of the program
    result_cid      text   CID of the output value
    step_count      int  | option_none
    trace_cid       text | option_none

## Exit Codes

    "ok"     execution completed successfully
    "err"    execution completed with an error
    "abort"  execution was aborted

## CID Derivation

    receipt_cid = cid(receipt_canonical_value)

## Chaining

Parent digest enables receipt chaining. A chain of receipts forms an auditable
execution history where each step references the previous.

## Reference Implementation

    packages/kernel-witness/src/receipt.fard
