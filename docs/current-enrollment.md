# Current iPhone enrollment stack

14 October 2026. **Direct Windows enrollment is still unverified.** This pass recovered the exact iOS 27.0 / 24A437 framework that the iPhone daemon actually calls, then examined its selected pairing and key-service boundaries.

## Source-bound progress

The client-facing enablement block calls the daemon's authentication manager. Its adapter queues a closure invoking `AutoUnlockDaemon.SDAuthenticationManager.enable`. That imported implementation is now acquired. This connects selected static code boundaries; it is not a capture of a live remote Mirroring request.

The framework code lives in the separate `Cryptex1,SystemOS` component, not the root filesystem containing daemon code and framework resources. The decrypted SystemOS SHA384 matches its manifest digest. Its cache UUID is `E0B5BDE5-0A07-3924-B762-8E411A50080C`. Selected executable sections in AutoUnlockDaemon, AppleKeyStore, AutoUnlock and SharingCore match the original cache bytes. Artifact identities and locations are in the [curated evidence](../evidence/current-enrollment.json).

Standalone metadata inspection encountered pointers outside the extracted image. Original-cache inspection recovered the relevant Swift and Objective-C metadata. Global stub-island symbol labels added during extraction are search aids, not proof that those functions belong to the selected image. Source instruction words and selected selector stubs are separately checked.

## What type 5 does

The selected current enable implementation converts external authentication type 5 to internal 5. `SDAuthenticationType.requiresAttestation` uses mask `0x1800f8`, whose bits for internal types 5 and 6 are set. This corroborates the [earlier Mac-side type mapping](key-attestation.md) with exact-build iPhone code. It does not recover the complete verifier policy.

This build contains both an executor route and compatibility AKS pairing classes. Type 5's `supportsExecutor` policy returns true, and `SDAuthentication.Feature.requiresAttestation` returns true unconditionally. The compatibility exchange method also derives its attestation argument from the named type field before calling the AKS manager pairing constructor.

These routes must be kept distinct. Actual current Mirroring flow selection and complete value flow through the newer executor are still unresolved. A compatibility class alone cannot establish the live route.

## Keys and the acceptance boundary

The selected current `AKSUtilities.generateLocalLTK` implementation has ordinary and attested branches. The attested branch obtains DCRT via a helper calling `MAECopyDCRTWithOptions`, constructs `AKSRefKeyParams`, and invokes `aks_create_signing_key_with_params`. An ordinary branch in a general-purpose utility is not a demonstrated Mirroring enrollment fallback.

Selected current framework registration/session code calls `aks_remote_peer_setup`. The exact iPhone AppleKeyStore wrapper selects operation `0x2e`, enters `_remote_session_operate`, and reaches an outlined helper ending at an `IOConnectCallMethod` stub. This puts an acceptance boundary in the key-service/driver stack; it does not reveal its full certificate checks or SEP policy.

`validatePeerKey` was also located, but its name must not be read as a recovered certificate verifier: the selected implementation checks peer state through another helper. No Windows-created key was submitted or accepted in this pass.

## Implication for Windows

The inspected current policies and attested generation path strengthen the evidence for an attestation dependency. They do not demonstrate a permitted software-host route, and they do not prove every possible Windows route is rejected.

The next meaningful test needs the actual Mirroring request-to-executor route, complete enrollment message representations, and a way to distinguish account, transport and attestation failures. Only then would a fresh Windows-origin enrollment probe produce interpretable evidence. No keys were exported, no pairings reset and no device protections changed.
