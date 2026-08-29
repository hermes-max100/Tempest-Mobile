# Tempest ↔ HSE Governed Execution

Tempest is the execution and mobile-operator surface. HSE remains the policy, authorization, signing, replay-control, redaction, and evidence authority.

## Trust boundary

Tempest may request a test. Tempest may not authorize itself.

HSE owns:

- signed policy validation,
- scope and prohibited-path enforcement,
- grant lifetime and request-count limits,
- Ed25519 signing,
- replay state,
- evidence redaction,
- evidence hashing and receipt signing.

Tempest owns:

- presenting the requested synthetic assertion,
- receiving and verifying the public execution grant,
- executing only the exact grant fields,
- collecting result fingerprints and non-secret metadata,
- returning the evidence bundle to HSE,
- displaying the HSE evidence receipt.

The Tempest client must never receive an HSE private signing key, seed, policy bypass token, or credential that would allow it to widen grant scope.

## Protocol flow

1. Tempest builds a `grantRequest` matching `protocol/tempest-hse-v1.schema.json`.
2. HSE validates its signed policy and evaluates the exact target through the HSE policy engine.
3. If allowed, HSE returns an Ed25519-signed `executionGrant` bound to:
   - protocol version,
   - grant/request IDs,
   - policy ID/version/hash,
   - exact target URL,
   - source and target identities,
   - resource type and read action,
   - expected result/status,
   - rate ceiling,
   - maximum request count,
   - issue/expiry timestamps,
   - nonce and request hash.
4. Tempest verifies the grant signature with HSE's public verification key and refuses any locally modified grant.
5. Tempest executes only within the signed grant. It does not follow an unsigned redirect into a new target or change the method, identities, resource, rate, or request cap.
6. Tempest returns an `evidenceBundle` containing result/fingerprint data and metadata.
7. HSE verifies the grant again, rejects expired/mismatched/replayed executions, runs structured redaction, hashes the redacted bundle, and returns a signed `evidenceReceipt`.

## Current v1 safety scope

The current HSE baseline is intentionally narrow:

- authorized synthetic targets only,
- HTTP targets already admitted by the signed HSE policy,
- `GET` / `read` assertions only,
- BOLA/IDOR-style authorization verification,
- maximum grant lifetime of 300 seconds,
- maximum 20 requests per grant,
- no destructive actions,
- no exploit-chain automation,
- no credential harvesting,
- no device-side grant signing.

Expanding beyond this scope requires a new HSE policy/protocol revision rather than a Tempest-only client change.

## Mobile implementation rule

When the actual mobile runtime is added, its security adapter should be a thin client around this protocol. The UI can recommend or prepare a test, but execution must remain impossible until a valid HSE grant has been verified.
