# DEK Bundle v1

Status: FROZEN
Version: 1.0.0

## Overview

A bundle is a FARD record that packages a program, its input, and execution
configuration. Bundles are the unit of work submitted to the DEK engine.

## Required Fields

    program     { t: "cid", v: "sha256:..." } | { t: "text", v: "<source>" }
    input       any canonical value
    trace_mode  "full" | "minimal" | "none"

## Optional Fields

    deps        list of { name: <text>, cid: <text> } entries
    oracle      { t: "cid", v: "sha256:..." }
    config      record of runtime settings
    version     text

## Validation Rules

- `program` must have t = "cid" or t = "text"
- `trace_mode` must be exactly "full", "minimal", or "none"
- Each entry in `deps` must have both `name` and `cid` fields
- Unknown fields are permitted and ignored

## Error Codes

    MISSING_FIELD    required field absent
    INVALID_FIELD    field present but value illegal

All errors include `phase: "bundle.validate"`.

## Reference Implementation

    packages/kernel-bundle/src/validate.fard
