# Session and protocol map

This is a partial map of Apple's Mac implementation, recorded on 14 October 2026. It is not a complete wire specification. Sources below are curated public observations; full local evidence is not part of this repository.

## Observed session sequence

| Phase | Established behavior | Unresolved requirement |
|---|---|---|
| Host UI and device state | The app declares imports of ScreenContinuityUI, ScreenContinuityServices and ScreenSharingKit. A companion-device type exposes remote-screen support and pairing state. | Remote provenance of capabilities and eligibility policy |
| Local authentication | The native app checks local authentication, waits for completion, then retains an authorization for later remote unlock. | Material creation API, bytes and reuse policy |
| Discovery | MediaTransportClientSession discovery begins; production metadata declares RPRemoteDisplayDiscovery. | Advertisements, filtering and secure endpoint handoff |
| Control activation | The stream `com.apple.oneness.sessionAndHIDMessages` activates and the client reports protocol version 6. | Full version/capability handshake and framing |
| Remote authentication | After startup configuration, the app revalidates local authorization; Sharing logs authentication start/completion. | Enrollment, key origin, host identity and iPhone verification |
| Media | Audio/video streams activate and final display readiness is reported. | Codec, negotiation, packetization and protection |
| Input | The native window accepts app-launch clicks, Home navigation and text entry. | Corresponding HID/remote-text encoding |
| Interruption/reconnect | iPhone use ends mirroring. Locking the phone again and completing Mac authentication restores a live session. | Teardown messages, trust reuse and automatic reconnect |

See [selected session events](../evidence/session-events.json), [reference observations](../evidence/reference-session.json) and [static summaries](../evidence/static-observations.json).

“Unlock” in the logs is an internal operation name. It does not establish that the user-facing lock screen or physical display was unlocked. Physical locked/screen-off state was user-confirmed before the selected reconnect, without an independent physical recording.

## Components and execution sides

The Mac UI imports ScreenSharingSession activation/state operations from ScreenSharingKit. The media-backed session vendor references ScreenSharingSession. Mac-side LocalAuthenticationManager exposes an async authentication result of Foundation.Data; native logs corroborate local approval but do not expose that material.

SharingBackedAuthenticationPrimitives owns SFAuthenticationManager and pairing/unlock/approval continuations. Its constructor calls `initWithQueue:` and `setDelegate:`. A common async helper calls `listCandidateDevicesForType:completionHandler:`. That is a candidate-device query, not proof of a pairing exchange. The iPhone-side server and trust verifier have not been analyzed.

Mac entitlements declare private pairing, authentication and transport access. They do not independently prove an iPhone requirement for Apple host certificates, a Secure Enclave operation or a particular host platform.

## Transport

One native log event reports an IPv6 link-local connection on `awdl0`, explicitly labeled AWDL. Its port numbers are transient observations, not a fixed service contract. Production metadata contains RPRemoteDisplaySession, RPStreamSession and a WiFiP2P traffic reservation. Whether ordinary LAN transport can support a compatible session remains unresolved.

No packet capture, decrypted payload or complete Rapport channel/authentication implementation was obtained. Discovery bytes, encryption, channel binding, reliability and fallback remain unknown.

## Control representation

Source-cache code establishes a PropertyListEncoder.encode call at `0x2673f4798`, in an async body starting at `0x2673f4688`, and a PropertyListDecoder.decode call at `0x2673f33c8`, in a function starting at `0x2673f3254`. This establishes encoder/decoder use in the Mac control implementation, not a complete sender-to-receiver path or XML/binary plist settings.

The following are declared typed fields and cases, rather than verified wire layouts:

| Type | Fields or cases |
|---|---|
| ControlMessage | draw, system, hidMessage, status, accessibility, dragAndDropEvent, rtiMessage |
| MediaControlTransportMessage | state: MediaControlEvent; streamID: String |
| SystemGestureEvent | homeButton, appSwitcher, spotlight, rotateScreen, launchPayload |
| HIDMessage | updateInputDevices, report, resetGestureState |
| HIDReportMessage | eventID: UInt64?; sender: EventReportSenderID?; reportData: Data |
| HIDUpdateInputDevicesMessage | deviceID: UUID; inputDevices: array of SSKHIDServiceProperties |
| RTIMessage | sessionID: UUID; sessionState: RTISessionState |
| RTISessionState | begin/end, text action, document change, pause/unpause, session death |

The [static summaries](../evidence/static-observations.json) identify source locations. HID bytes, pointer/button mapping, scrolling units, key codes, orientation transforms and text-action payloads remain unresolved. UI behavior does not identify which representation carried it.

## Video

MediaTransportVideoStream declares AVCMediaStreamNegotiator, AVCRemoteVideoClient, AVCVideoStream, RPStreamSession and negotiation/running/teardown state. A source method beginning at `0x26744db30` calls `initWithNetworkSockets:options:error:` on one branch and `initWithNWConnectionClientID:options:error:` on another. The observed branch and negotiated parameters were not established.

AVConference symbols and RTCP delegate cases do not alone establish RTP framing, an SRTP cipher or a video codec. Native audio/video activation and a live display corroborate media delivery on the Mac reference only.

Authentication and transport remain the prerequisites for the [next experiment](windows-compatibility.md).
