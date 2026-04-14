# DEK Policy Layer v1

Status: FROZEN
Version: 1.0.0

## Overview

A policy is a canonical record that declares what is permitted for a class
of executions. Policies are content-addressed so versions are pinnable,
auditable, and revocable.

The full verification predicate is:

    valid_receipt AND licensed AND policy_compliant

## Policy Structure

    { t: "record", v: [
      { k: "allowed_exits",    v: <list of text> | option_none },
      { k: "allowed_oracles",  v: <list of text> | option_none },
      { k: "allowed_programs", v: <list of text> | option_none },
      { k: "expires_at",       v: <int unix_ms>  | option_none },
      { k: "max_step_count",   v: <int>          | option_none },
      { k: "revoked",          v: <bool> }
    ] }

null in any list field means "allow all values of that type".

## Constructor

    make_policy(
      allowed_programs,   -- list of sha256: strings, or null
      allowed_oracles,    -- list of oracle kind strings, or null
      allowed_exits,      -- list of "ok"|"err"|"abort", or null
      max_step_count,     -- int or null
      expires_at          -- unix ms int or null
    )

## Policy CID

    policy_cid(policy) -> { ok: "sha256:..." }

The policy CID uniquely identifies a specific version of a policy.

## enforce(policy, program_cid, exit_code, oracle_kinds, step_count, now_ms)

Checks all constraints in order:

    1. is_revoked(policy)
    2. check_expiry(policy, now_ms)
    3. check_program(policy, program_cid)
    4. check_exit(policy, exit_code)
    5. check_step_count(policy, step_count)
    6. check_oracle(policy, kind) for each kind in oracle_kinds

Returns:
    { ok: true }
    { ok: false, code, message, phase: "policy.enforce" }

## Error Codes

    POLICY_REVOKED       policy has been revoked
    POLICY_EXPIRED       policy expiry time has passed
    PROGRAM_NOT_ALLOWED  program CID not in allowed_programs
    EXIT_NOT_ALLOWED     exit code not in allowed_exits
    STEP_CAP_EXCEEDED    step_count exceeds max_step_count
    ORACLE_NOT_ALLOWED   oracle kind not in allowed_oracles

## Revocation

    revoke(policy)   -> new policy with revoked: true
    is_revoked(policy) -> bool

Revocation produces a new canonical value with a new CID.
The original policy CID remains valid as a historical record.

## Expiry

expires_at is a Unix millisecond timestamp.
A policy with expires_at = 1 is always expired.
A policy with expires_at = null never expires.

## Reference Implementation

    packages/kernel-policy/src/policy.fard
