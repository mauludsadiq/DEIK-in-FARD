# DEK Licensing Guide

Version 1.0

---

## Overview

This guide explains how a publisher issues DEK license registries and how a
consumer verifies that an execution used a licensed program.

---

## Concepts

**Program CID**
The sha256 CID of a program's canonical representation (source, bytecode, or
model weights). This is the stable identifier for a specific version of a program.

**License Registry**
A canonical set of approved program CIDs. The registry is itself
content-addressed so every version is pinnable and auditable.

**Registry CID**
The CID of a registry canonical value. This is the license anchor.
Distributing a registry CID is equivalent to issuing a license for all
programs it contains.

**Policy**
A canonical record declaring what is permitted: which programs, which oracle
kinds, which exit codes, step caps, and expiry. Policies are also
content-addressed and can be revoked.

---

## Publisher workflow

### 1. Determine program CIDs

For a FARD program:

    let program_source = "my program source code"
    let program_canon = types.make_text(program_source)
    let program_cid = cid.cid_of(program_canon).ok

For a model or binary, encode it as a canonical bytes value and CID it.
The exact method must be documented and stable for verification to work.

### 2. Build a registry

    let registry = license.make_registry([
      "sha256:aaa...",  // program v1.0
      "sha256:bbb...",  // program v1.1
      "sha256:ccc..."   // plugin module
    ])

    let registry_cid = license.registry_cid(registry).ok

### 3. Save and distribute

    persist.save_registry(registry, "dist/registry_v1.json")

Distribute to customers:
- the registry JSON file
- the registry CID (for independent verification)
- the registry version label

### 4. Build a policy

    let policy_v1 = policy.make_policy(
      ["sha256:aaa...", "sha256:bbb..."],  // allowed programs
      null,                                 // any oracle
      ["ok"],                              // only successful exits
      10000,                               // 10k step cap
      1800000000000                        // expires Jan 2027
    )

    let policy_cid = policy.policy_cid(policy_v1).ok

Distribute the policy CID alongside the registry CID.

### 5. Revocation

To revoke a policy:

    let revoked = policy.revoke(policy_v1)
    persist.save_registry(revoked, "dist/policy_v1_revoked.json")

Publish the revoked policy. Customers loading it will see revoked: true.
The original policy CID remains valid as a historical record.

To issue a new policy version:

    let policy_v2 = policy.make_policy(
      ["sha256:ddd..."],  // updated program list
      null, ["ok"], 10000, 1830000000000
    )

    let policy_v2_cid = policy.policy_cid(policy_v2).ok

---

## Consumer workflow

### 1. Receive license artifacts

From the publisher:
- registry_v1.json (or a URL)
- registry_cid: "sha256:..."
- policy_v1.json (or a URL)
- policy_cid: "sha256:..."

### 2. Load and verify registry

    let r = persist.load_registry("dist/registry_v1.json")

    // Verify the CID matches what the publisher distributed
    let cid_match = r.ok.registry_cid == "sha256:..."  // publisher's registry CID

### 3. Run a program

    let result = engine.exec_bundle(bundle, my_eval)

### 4. Verify licensing

    let licensed = license.is_licensed(program_cid, r.ok.registry)

### 5. Verify policy compliance

    let compliant = policy.enforce(
      loaded_policy,
      program_cid,
      result.ok.exit,
      oracle_kinds_used,
      step_count,
      now_ms
    )

### 6. Full verification

    let verified = licensed && compliant.ok == true

Only accept the result if verified is true.

---

## Registry versioning

Registries are immutable once published. To update:

1. Create a new registry with the updated program list
2. Compute and publish the new registry CID
3. Distribute the new registry file
4. Old registry CIDs remain valid for historical verification
5. Customers must explicitly update their registry anchor

---

## Remote verification

Set FARD_REGISTRY_URL to enable remote receipt lookup:

    export FARD_REGISTRY_URL=https://your-registry-server.com

verify.verify_receipt will fall back to this URL if the receipt is not found
locally. The registry server must respond to:

    GET /receipt/{run_id}

with the receipt JSON.

---

## Security considerations

- Registry CIDs are the ground truth. Always verify CIDs match.
- Policies should have expiry timestamps for time-limited licenses.
- Revocation produces a new canonical value. Distribute the revoked version.
- Program CIDs must be derived from a stable, documented encoding.
- The DEK kernel does not verify the identity of the executor.
  Use DEK receipts as inputs to your existing identity and audit systems.
