# iPhone Mirroring on Windows Research

Research into a direct Windows client for Apple's native iPhone Mirroring: wireless screen streaming and mouse/keyboard control while an unmodified iPhone remains locked with its display off.

**Status: early investigation. A Windows-origin session has not been demonstrated.** The first phase maps selected parts of the working Mac implementation. Authentication, fresh host enrollment and transport remain unresolved prerequisites.

The intended client would connect directly from Windows to the iPhone, without a Mac relay, jailbreak or iPhone companion app.

## Initial findings

The reference session reports control protocol version **6** and stream ID `com.apple.oneness.sessionAndHIDMessages`. One observed transport connection uses IPv6 link-local addresses over **AWDL**, on `awdl0`; this does not establish that AWDL is mandatory for every session.

The native logs show this order:

1. Local Mac authentication produces an authorization retained for later use.
2. Device discovery and control-stream activation complete.
3. Startup configuration is exchanged and local authorization is revalidated.
4. Remote authentication completes through the Sharing subsystem.
5. Audio/video streams activate and the display becomes ready.

The Mac-side remote-authentication adapter uses `SFAuthenticationManager`. Transport types include Rapport's `RPRemoteDisplaySession` and `RPStreamSession`. The Mac control implementation calls `PropertyListEncoder.encode` and `PropertyListDecoder.decode`; the complete object graph, format settings and outer framing still need recovery. Input types distinguish system gestures, HID reports and remote-text session messages.

These observations identify places to investigate. They do not yet supply an implementable wire specification.

## Reference and methods

The initial reference is an iPhone 16 Pro on **iOS 27.0 (24A437)**, with iPhone Mirroring **1.6 (98.5)** on **macOS 26.6.2 (25G83)**. Native Mac mirroring already works. App launch by mouse, Home navigation, short text entry and reconnection were observed. Physical lock/display state was user-confirmed; automated scrolling did not visibly move the list and remains unverified.

The investigation used AI-assisted tooling through REA 6.4.0 and Ghidra 12.1.4, ipsw 3.1.737, source-cache disassembly and native session logs. Some extracted-image imports were mislabeled by Ghidra, producing false no-return boundaries. Selected code paths were checked against the original cache rather than relying on truncated pseudocode.

## Documentation

- [Session and protocol map](docs/protocol-map.md)
- [Windows compatibility and next experiment](docs/windows-compatibility.md)
- [Enrollment and authorization follow-up](docs/authentication.md)
- [Trusted keys and attestation investigation](docs/key-attestation.md)
- [Secure Enclave options and iPhone peer acquisition](docs/secure-enclave-options.md)
- [Current iPhone enrollment stack](docs/current-enrollment.md)
- [Methods and evidence limitations](docs/methodology.md)
- [Research log](docs/research-log.md)
- [Findings ledger](evidence/findings.json)
- [Selected native session events](evidence/session-events.json)
- [Static observation summaries](evidence/static-observations.json)
- [Artifact identities](evidence/artifact-identities.json)
- [Reference session](evidence/reference-session.json)

Public evidence is a curated export of observation summaries, selected redacted events and artifact identities. Full local dossiers, raw diagnostics and Apple binaries are not included. Derived exports are labeled separately from original REA Evidence records.

## Next experiment

The [authentication follow-up](docs/authentication.md) now identifies the local authorization representation, separate pairing/authentication requests, the Sharing service boundary and selected long-term-key/AppleKeyStore session operations. The [key investigation](docs/key-attestation.md) identifies a selected pairing path that requests attestation and an attested generator using Apple activation DCRT/AKS parameters. The [exact iPhone pairing daemon](docs/secure-enclave-options.md) is now acquired, with selected legacy attestation boundaries identified. The [current iPhone framework](docs/current-enrollment.md) now corroborates the type-5 attestation policy and exposes a newer executor alongside compatibility paths. Live flow selection, the full verifier and any permitted alternative enrollment remain unverified. A viable path would justify a minimal Windows authentication probe. Video codec, framing, protection, HID bytes, coordinate transforms and text payloads remain open questions.

Pointers to prior work on Mac-unlock pairing, Rapport remote-display enrollment and AWDL transport are welcome through [GitHub discussions in issues](https://github.com/BlueWhale87/iphone-mirroring-windows/issues). See [contribution guidance](CONTRIBUTING.md).
