# DEK Integration Guide

Version 1.0

---

## Overview

This guide explains how to wrap an LLM, service, or autonomous system in DEK
to gain execution integrity, context binding, licensing, and policy enforcement.

---

## Prerequisites

- fardrun installed and on PATH
- DEK repository cloned or packages available

---

## Step 1: Choose your integration depth

DEK is composable. You can adopt any subset of the stack:

**Minimal** (encoding + CID only)
Gives you: stable content addresses for any value.
Packages needed: kernel-canon

**Context integrity** (+ context binding)
Gives you: truncation detection for LLM context windows.
Packages needed: kernel-canon, kernel-context, kernel-persist

**Full execution integrity** (+ exec + receipt)
Gives you: cryptographic witness for every execution.
Packages needed: kernel-canon, kernel-bundle, kernel-witness, kernel-exec

**Licensed execution** (+ license + policy)
Gives you: the full verification predicate.
Packages needed: all packages

---

## Step 2: Wrap your evaluator

The DEK engine takes an evaluator function:

    fn my_eval(program, input, config) {
      // your logic here
      { ok: result_canonical_value }
      // or
      { err: { code: "...", message: "..." } }
    }

The program, input, and config arguments are whatever you put in the bundle.
The result must be a canonical value (use types.make_text, make_int, etc.)

---

## Step 3: Build a bundle

    let bundle = {
      program: { t: "text", v: "my_program_v1" },
      input: types.make_record([
        { k: "query", v: types.make_text("user query here") }
      ]),
      trace_mode: "full"
    }

For LLM use, set input to a canonical context:

    let bundle = {
      program: { t: "text", v: "my_llm_program" },
      input: ctx.context_to_bundle_input(conversation),
      trace_mode: "full"
    }

---

## Step 4: Execute

    let result = engine.exec_bundle(bundle, my_eval)

    result.ok.receipt_cid   -- the execution receipt CID
    result.ok.exit          -- "ok" | "err" | "abort"
    result.ok.result        -- the canonical output value
    result.ok.receipt       -- the full canonical receipt

---

## Step 5: Add context integrity (LLM)

    let conversation = ctx.make_context(messages, "my-model", turn_count)
    let original_cid = ctx.context_cid(conversation).ok
    persist.save_context(conversation, "contexts/session_001.json")

    // Later, check for truncation:
    let check = ctx.truncation_check(original_cid, current_conversation)
    // { ok: { verified: true } } or { err: { code: "CONTEXT_MODIFIED" } }

---

## Step 6: Add oracle witnessing

Wrap every non-deterministic call:

    // Instead of: let t = time.now()
    let t = oracle.oracle_now(seq)
    // t.ok.value  -- the actual timestamp
    // t.ok.event  -- the canonical oracle event
    // t.ok.cid    -- the event CID

    // Instead of: let resp = http.get(url)
    let resp = oracle.oracle_http_get(url, seq)

    // For LLM calls:
    let resp = oracle.oracle_model_call(model, prompt, response, seq)

Build a snapshot after execution:

    let snapshot = oracle.make_snapshot([e1.ok.event, e2.ok.event, ...])
    let snapshot_cid = oracle.snapshot_cid(snapshot).ok

---

## Step 7: Add licensing

Publisher side:

    let registry = license.make_registry(["sha256:...", "sha256:..."])
    persist.save_registry(registry, "registries/v1.json")
    // Distribute: registry_cid = license.registry_cid(registry).ok

Customer side:

    let r = persist.load_registry("registries/v1.json")
    let licensed = license.is_licensed(program_cid, r.ok.registry)

---

## Step 8: Add policy enforcement

    let my_policy = policy.make_policy(
      allowed_programs,   // list of CIDs or null
      allowed_oracles,    // list of kinds or null
      allowed_exits,      // list of codes or null
      max_step_count,     // int or null
      expires_at          // unix ms or null
    )

    let check = policy.enforce(
      my_policy, program_cid, exit_code, oracle_kinds, step_count, now_ms
    )
    // { ok: true } or { ok: false, code, message, phase }

---

## Step 9: Verify

    // Verify a receipt exists and is valid:
    let v = verify.verify_receipt(run_id)

    // Verify the full chain:
    let c = verify.verify_chain(run_id)

    // Remote verification: set FARD_REGISTRY_URL environment variable

---

## Common patterns

**Pattern: Always-on context integrity**
Pin context CID at session start. Check before every LLM call.

**Pattern: Audit trail**
Save receipts and context snapshots to a versioned directory after each run.

**Pattern: Policy gate**
Enforce policy before returning results to the caller. Reject non-compliant runs.

**Pattern: Determinism check**
Use replay_check after any critical execution to confirm determinism.

---

## Error reference

All DEK errors follow the shape:
    { code: "...", message: "...", phase: "...", details: {...} }

Common codes:
    MISSING_FIELD        required bundle field absent
    INVALID_FIELD        field present but illegal value
    RECEIPT_NOT_FOUND    run_id not in local store or registry
    CHAIN_BROKEN         derived_from chain has a missing node
    NOT_LICENSED         program CID not in registry
    POLICY_REVOKED       policy has been revoked
    POLICY_EXPIRED       policy expiry has passed
    PROGRAM_NOT_ALLOWED  program not in allowed_programs
    ORACLE_NOT_ALLOWED   oracle kind not in allowed_oracles
    EXIT_NOT_ALLOWED     exit code not in allowed_exits
    STEP_CAP_EXCEEDED    step count exceeds max_step_count
    CONTEXT_MODIFIED     context CID does not match original
