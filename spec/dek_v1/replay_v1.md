# DEK Replay v1

Status: FROZEN
Version: 1.0.0

## Overview

Replay is the mechanism by which DEK proves that an execution is deterministic.
For any deterministic execution, running the same program with the same input
through the same kernel version produces the same receipt CID.

The replay invariant:

    Replay(Receipt) = Receipt

for deterministic (oracle-free) computations.

For oracle-using computations:

    Replay(Receipt, OracleSnapshot) = Receipt

where the snapshot supplies the same non-deterministic values as the original run.

## replay_check(program, input, evaluator)

Runs the same execution twice and compares receipt CIDs.
Proves determinism without requiring a stored receipt.

Returns:
    { ok: { receipt_cid, deterministic: true, runs: 2 } }
    { err: { code: "REPLAY_EXEC_FAILED", message, details } }

If both runs produce the same receipt_cid, the execution is deterministic.
If they differ, the execution is non-deterministic and the error carries both CIDs.

## replay(run_id, program, input, evaluator)

Given a stored receipt run_id, re-executes the program with the same input
and returns the new receipt CID alongside confirmation that the original
receipt was found.

Returns:
    { ok: { run_id, replayed_receipt_cid, exit, deterministic: true } }
    { err: { code: "RECEIPT_NOT_FOUND", message, run_id } }
    { err: { code: "REPLAY_EXEC_FAILED", message, details } }

## Determinism scope

Determinism holds for executions that do not call oracle functions.
For executions using oracle functions, determinism requires supplying
the same OracleSnapshot as input so the same values are returned.

A failing evaluator (one that returns { err: ... }) is still deterministic
if it fails the same way every time. The receipt records the failure.

## Error Codes

    REPLAY_EXEC_FAILED    execution failed during replay
    RECEIPT_NOT_FOUND     stored receipt not found for run_id
    NON_DETERMINISTIC     two runs produced different receipt CIDs

## Reference Implementation

    packages/kernel-replay/src/replay.fard
