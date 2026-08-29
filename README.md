# Tempest Mobile

Tempest is the mobile operator and execution surface for governed AI-assisted security testing.

## HSE-governed execution

Tempest does **not** authorize its own security tests. It requests short-lived execution grants from HSE (HSE Enterprise BOLA Assurance), verifies those grants with HSE's public key, executes only the exact signed scope, and returns evidence to HSE for redaction, hashing, replay checks, and a signed evidence receipt.

Current protocol: `tempest-hse-v1`

- Wire schema: `protocol/tempest-hse-v1.schema.json`
- Architecture and trust boundary: `docs/HSE_INTEGRATION.md`

### Current v1 scope

- policy-approved synthetic targets only
- BOLA/IDOR-style read assertions
- GET-only execution under the existing HSE baseline
- signed target/identity/resource binding
- short-lived grants (maximum 300 seconds)
- explicit request caps (maximum 20 per grant)
- no destructive actions
- no private HSE signing keys on the mobile device

The mobile runtime should remain a thin client around this contract. Any future capability expansion must happen through an explicit HSE policy/protocol revision rather than an unsigned Tempest-side override.
