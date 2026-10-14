# Research log

## 14 October 2026

Completed the first investigation phase against a working native Mac reference. The goal remains direct Windows mirroring and control of a locked, unmodified iPhone with its display off.

Established selected app/framework identities, session phase order, control protocol version 6, one AWDL route, property-list encoder/decoder calls, and production HID/remote-text type declarations. Native app launch, Home navigation, short text entry and reconnect were observed. Scroll remains unverified.

Extracted-image decompilation has imported-stub/no-return defects. Selected original-cache disassemblies provide corroboration; this is not a complete recovered protocol.

Outcome: a candidate reconstruction path with unresolved enrollment, authentication and transport. No Windows-origin session has run and no platform incompatibility has been proven.

Next: trace the SFAuthenticationManager/Sharing authorization and enrollment chain before designing a minimal Windows probe. See the [compatibility assessment](windows-compatibility.md).

### Enrollment and authorization follow-up

Traced selected externalized LocalAuthentication context calls, authentication types 5/6, and the adapter's authorization-token/device-ID options. Sharing forwards enrollment/authentication through `com.apple.sharingd.nsxpc`. The daemon checks a Mac-local entitlement before dispatch; that check is not an iPhone platform requirement.

Located conditional local/attested long-term-key selection and concrete AppleKeyStore session creation, token and step calls. The full current-flow route, attestation policy, enrollment transcript and iPhone verifier remain incomplete. New findings include explicit confidence and interpretation limits.

Next: current iPhone long-term-key/attestation acceptance for an independently enrolled software host. See the [follow-up report](authentication.md).
