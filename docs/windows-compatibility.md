# Direct Windows compatibility assessment

14 October 2026. **Outcome: candidate approach identified, with unresolved authentication and transport.** A direct Windows-origin session has not been demonstrated. No specific requirement has yet been proven to make such a session impossible.

The intended product connects directly from Windows to a locked, unmodified iPhone with its display off, streams real apps and accepts mouse/keyboard control. This investigation establishes a Mac reference and several implementation boundaries. It does not supply a finished Windows client or an implementable protocol specification. [Project goal](../README.md), [protocol map](protocol-map.md), [findings ledger](../evidence/findings.json).

## What the evidence changes

The reference session uses protocol version 6 and an observed AWDL route. Local Mac authentication produces an authorization that the app retains; after control-stream setup the app revalidates it and requests remote authentication before starting video. That makes **host trust and enrollment**, together with transport reachability, the next feasibility questions. A Windows screen receiver alone would not satisfy the goal. [Selected runtime events](../evidence/session-events.json).

Static analysis locates the Mac-side SFAuthenticationManager adapter, Rapport remote display types, property-list control serialization, HID reports and separate remote-text messages. These are concrete places to continue, while the credential origin, iPhone-side verification, channel setup and complete encodings are still missing. [Protocol map](protocol-map.md).

| Area | Established for this reference | Direct Windows status |
|---|---|---|
| Native behavior | Connected Mac window; Home navigation; Settings launch; text entered; interruption on physical iPhone use and subsequent reconnection | No Windows-origin test |
| Enrollment/trust | Mac-side pairing/unlock adapter and runtime remote authentication completion | Key origin, host identity, enrollment protocol and iPhone acceptance policy unresolved |
| Local approval | User completed Mac authentication; app logs authorization reuse for remote unlock | Whether an independently enrolled Windows host can create equivalent authorization unresolved |
| Discovery/transport | Rapport-related types; one runtime IPv6/AWDL connection on awdl0 | Reachability, authenticated transport, AWDL support and possible LAN fallback unresolved |
| Control | Protocol 6; observed stream ID; PropertyListEncoder/Decoder calls; typed HID/RTI structures | Full framing, object graph and report/text payloads unresolved |
| Video | Native session starts audio/video; AVConference transport-construction boundaries | Codec, negotiation, packetization and protection unresolved |
| Scroll/orientation | Related static interfaces exist | Scroll automation produced no visible movement; orientation not tested |

The iPhone reference is 16 Pro, iOS 27.0 (24A437); Mac/app identities and physical-lock confirmation are in [reference-session.json](../evidence/reference-session.json). Exact build was recovered after the first observations and corroborated in the reconnect log, so this is a retrospectively version-bound baseline. No controlled cross-version comparison was made.

## Smallest justified next experiment

**Question:** What produces the authorization passed to remote authentication, and what persistent host trust does SFAuthenticationManager use for this already-enrolled Mac?

1. Follow the concrete SFAuthenticationManager reference from SharingBackedAuthenticationPrimitives into the source-bound Sharing framework and its client/service interfaces. Preserve the same cache identity and hashes. Recover the `pairDeviceForMacUnlock` and `unlockWithAuthenticationToken` coroutine continuations, and LocalAuthenticationManager's authorization-producing path. Read mechanisms and metadata; credential values are unnecessary.
2. Identify the enrollment service operation, token input type, persistent key lookup and any device/account binding. Correlate operation names and completion states with one bounded reference reconnect. The existing log already supplies the order; extra captures need answer a named unknown.
3. Classify the result: a documented/recoverable host enrollment path justifies a minimal Windows authentication probe; a demonstrated non-exportable or platform-bound dependency establishes a blocker for that specific approach; ambiguous code or encrypted traffic remains unresolved.

**Prerequisites:** this Mac's existing working enrollment, the source-bound Sharing component, the same iPhone build, and accessible observations without changing OS protections. This experiment does not require a full Windows app. It should be limited to the authorization/enrollment chain rather than importing every Apple framework.

**Expected useful result:** an evidence-backed description of the authorization source, enrollment operation, required host identity and remote acceptance boundary. This is successful research even if it identifies a limitation; it is not evidence that Windows works.

**Failure interpretation:** a missing symbol, extraction failure, inaccessible service, or truncated decompilation means the check is incomplete. It must not be reported as rejection by the iPhone. A Mac entitlement or use of LocalAuthentication is not sufficient to claim Windows cannot authenticate.

## Gate for the first Windows probe

After the trust experiment resolves a viable enrollment path, use a Windows machine with a transport path that reaches the iPhone. The smallest client should discover/select that phone, perform fresh host enrollment, and request an authenticated remote display/control session. Instrument explicit success/failure stages; stop before video work if authentication is not accepted.

No Mac relay, transplant of this Mac's credentials, jailbreak or iPhone companion app satisfies this product's direct-connection criterion. Any approach that depends on one must be reported as a change of scope.

Acceptance requires a reproducible Windows-origin session that streams an app and accepts an input action while the user confirms the phone remains locked with its screen off. Then verify scroll, text and ordinary reconnect. Today's successful Mac connection cannot substitute for that test.

## Material limitations

- No Windows host is connected to this investigation and no Windows implementation ran.
- No iPhone server artifact or remote verifier has been analyzed.
- Extracted-image Ghidra has false no-return boundaries; source-cache disassembly mitigates selected functions, not the entire analysis.
- There is no packet capture, decrypted payload or recovered complete crypto/channel negotiation.
- Initial enrollment and a deliberately failed remote-authentication request were not tested; the observed failure was ordinary iPhone use interrupting a session.
- Scroll remains unverified through automation; no codec, HID bytes, text commit encoding or coordinate transform is ready for implementation.

The deliverables support a focused next investigation. They do not establish native Windows parity or universal impossibility.

Public evidence is a curated export. Referenced original local dossiers, screenshots and raw diagnostics are not included; source identities and selected observations are provided in the evidence directory.
