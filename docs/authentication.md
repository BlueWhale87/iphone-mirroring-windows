# Enrollment and authorization follow-up

14 October 2026. This pass traces the Mac-local authorization and service boundaries further than the initial session map. **A fresh Windows host has not enrolled or authenticated.**

The app obtains an externalized LocalAuthentication context, places authorization data in Sharing options, and uses separate pairing/session-authentication requests. Sharing forwards those operations through a local XPC service. The daemon contains concrete AppleKeyStore session creation, token and step boundaries. This maps useful implementation locations without supplying the complete peer protocol or proving Windows acceptance.

See the [curated follow-up evidence](../evidence/authentication-followup.json) for artifact digests, source locations, confidence and remaining gaps. Raw local dossiers, binary images and account credentials are not part of this export.

## App authorization and request types

In ScreenSharingKit, LocalAuthenticationManager's authentication-success handler calls `externalizedContext` at `0x26739cdb0` and `0x26739cf24`. The returned object is bridged into Foundation.Data and delivered through a continuation on the success paths. This is a local authorization representation; it is not established as a reusable peer credential for Windows.

The pairing adapter calls `requestEnablementForType:withIDSDeviceID:` at `0x2673a1498`, passing authentication type **5**. The unlock adapter constructs SFAuthenticationOptions, sets `authorizationToken` at `0x2673a23bc`, sets `idsDeviceID` at `0x2673a23ec`, and calls `authenticateForType:withOptions:` at `0x2673a25d0`, passing authentication type **6**.

Sharing's `SFAuthenticationTypeToString` function at `0x19613f454` indexes a string table. Its source-bound entries identify:

| Authentication value | Description |
|---|---|
| 5 | SFAuthenticationTypeMacUnlockPhonePairing |
| 6 | SFAuthenticationTypeMacUnlockPhone |

These are **authentication types**, separate from the previously observed control-protocol version 6 and from AppleKeyStore's own session-type arguments.

The exact creation/evaluation policy of the LocalAuthentication context and every intervening asynchronous call remain incompletely traced. Correlating these static boundaries with the earlier successful native log is not equivalent to a debugger-observed dataflow.

## Local service boundary

SFCompanionXPCManager connects to `com.apple.sharingd.nsxpc`, obtains an unlock-manager proxy and dispatches:

- `requestEnablementUsingClientProxy:authenticationType:device:sessionID:` for enrollment.
- `authenticateUsingClientProxy:type:sessionID:options:` for authentication.

In sharingd, SDUnlockXPCSession handles those requests. Its authentication entry is at `0x10000a07c`; its pairing entry is at `0x1000099ec`. Both invoke a local entitlement check before forwarding work to SDAuthenticationManager. The check at `0x10000b160` queries the current XPC connection for `com.apple.private.sharing.unlock-manager`.

This is a **Mac-local access gate**. It is not evidence that the iPhone checks a Windows executable for a Mac entitlement. Replacing the desktop UI would not replace the service's enrollment and key-exchange responsibilities.

## Key and cryptographic session boundaries

SDAutoUnlockAKSManager's authorization constructor at `0x1000beb28` selects a local or local-attested long-term key, looks up a remote long-term key by device ID, and passes an external ACM context in selected session-construction branches. A pairing constructor at `0x1000be064` likewise selects a local/attested key and obtains a remote key, using an initializer without that context. These are concrete code boundaries; no key contents were read.

The AKS session implementation exposes the following selected paths:

| Path | Established call boundary |
|---|---|
| Authorization session setup | setupAuthorizationSession at `0x1000e7df4` reaches a native creation helper at `0x1001650d4`, which calls `aks_remote_session` |
| Generate session token | generateSessionToken at `0x1000e8454` calls a helper at `0x100165c3c`, which calls `aks_remote_session_token` |
| Step session | stepSessionWithData at `0x1000e8200` calls a helper at `0x100165e60`, which calls `aks_remote_session_step` |

Imported targets are resolved through the daemon's authenticated stubs and chained-fixup bindings. Original instruction words are checked against the source-bound executable.

Production Swift metadata declares MacUnlockPhone flow contexts with AKS managers, request/response/token/confirmation messages and an authorization-token input. It also declares operations involving long-term keys and AKS tokens. Some metadata formatting is defective, so only corroborated names/fields serve as search seeds. Full operation ordering, payload schemas and the complete route from the selected type-6 request into these helpers are not established.

There are explicit `requiresAttestation` and `attestation` inputs. Inspected call sites derive them from session/type/role state, and constructors select attested keys conditionally. The exact policy for a fresh mirroring enrollment and the iPhone's validation of it remain unresolved. The presence of an attestation argument alone does not prove a universal hardware requirement.

## Documented architecture

Apple documents a mutually authenticated STS foundation with long-term enrollment keys and per-session keys. For iPhone Mirroring, Mac authentication policy is enforced by its Secure Enclave and remote access unlocks the iPhone keybag while the visible phone remains locked. The guide's general framework also notes kernel termination on Macs without Secure Enclaves. This describes Apple's implementation; it does not by itself prove rejection of every software peer. [Apple Platform Security](https://support.apple.com/guide/security/automatically-unlock-apple-devices-sec6ab47ebfc/web).

Notification identity pinning is documented separately from remote unlock. Its description must not be substituted for the remote-unlock enrollment/verifier without a demonstrated connection.

## Implication for Windows

The hypothesis to test is now more specific: **can an independently enrolled software host satisfy the current iPhone's long-term-key and attestation policy and complete a compatible remote-unlock exchange?**

The next pass should trace creation/signing/registration of the long-term key for authentication type 5, establish its attestation/certificate requirements, and find the corresponding peer-side verification boundary. A software host cannot be called compatible merely because it can serialize options or generate a generic STS exchange. No Mac credential transplantation or relay satisfies the direct-client goal.

The local analysis does not prove universal impossibility. It also does not justify a full Windows mirroring implementation yet. Transport, control framing and media negotiation remain separate unresolved work.

## Analysis limits

- Ghidra's Sharing import/procedure search exceeded the configured 180-second provider startup budget. That error was retained; the identical import was not repeated.
- REA Mach-O metadata and native source disassembly supplied the selected boundaries. Reliable whole-framework pseudocode is not claimed.
- Verbose daemon metadata contains invalid text and suspicious Swift ivar offsets. Those offsets are not used as evidence.
- No new live session, initial enrollment, credential extraction or Windows-origin probe ran in this pass.
