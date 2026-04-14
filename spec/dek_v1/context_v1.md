# DEK Context v1

Status: FROZEN
Version: 1.0.0

## Overview

A context is a canonical value representing an LLM conversation window.
Contexts are content-addressed so truncation and modification are detectable.

## Message Structure

    { t: "record", v: [
      { k: "content", v: { t: "text", v: "<content>" } },
      { k: "role",    v: { t: "text", v: "<role>" } }
    ] }

Valid roles: "system" | "user" | "assistant"

## Context Structure

    { t: "record", v: [
      { k: "messages",   v: { t: "list", v: [<message>, ...] } },
      { k: "model",      v: { t: "text", v: "<model>" } },
      { k: "turn_count", v: { t: "int",  v: <n> } }
    ] }

## Context CID

    context_cid = cid(context_canonical_value)

The context CID pins the exact content of the conversation at a point in time.

## Truncation Detection

    truncation_check(original_cid, current_context)

Recomputes the CID of current_context and compares to original_cid.

Returns:
    { ok: { cid, verified: true } }
    { err: { code: "CONTEXT_MODIFIED", original, current } }

## Error Codes

    INVALID_ROLE        role not in allowed set
    CONTEXT_MODIFIED    current context CID does not match original
    CID_FAILED          CID computation failed

## Reference Implementation

    packages/kernel-context/src/context.fard
