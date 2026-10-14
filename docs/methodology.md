# Methods and evidence limitations

The initial investigation combines static Mac artifact analysis with selected observations from a working native Mac–iPhone session. It uses REA 6.4.0, Ghidra 12.1.4, Java 21.0.10 and ipsw 3.1.737. AI-assisted analysis is checked against artifact identity, original instructions and available runtime observations.

## Artifact provenance

The installed app executable and bundled ScreenContinuityUI are universal binaries. Verified arm64e slices were derived with lipo and compared byte-for-byte with their original FAT ranges. Ghidra reports its effective architecture as arm64; the selected artifact headers and parent-range verification establish the actual arm64e input.

ScreenSharingKit and ScreenContinuityServices were extracted from the system's arm64e dyld shared cache using ipsw with slide/Objective-C fixups. Image UUIDs match the source cache inventory. Original cache files were read in place; derived image digests and source identity are recorded in [artifact identities](../evidence/artifact-identities.json).

## Decompiler limitation

In extracted ScreenSharingKit, Ghidra labels several imported stubs as `___swift_memcpy16_8` and marks them no-return. Original-cache observations identify real targets such as objc_allocWithZone, swift_allocObject, swift_bridgeObjectRetain, swift_once and swift_task_switch. The incorrect flags truncate some pseudocode and function bodies.

Re-extraction with stub metadata produced the same bytes and did not resolve the labeling problem. Selected paths were therefore read through cache-aware source disassembly and checked against the extracted executable sections. Complete reliable decompilation of the framework is not claimed.

Swift metadata includes mock/demo types as well as production types. Published conclusions use identified production types. Metadata declarations are distinguished from executed paths and wire layouts.

## Native reference

The selected window was Apple's native iPhone Mirroring app. The user performed Mac authentication directly and confirmed that the nearby iPhone was locked with its display off. Native UI observations establish app launch by mouse, Home navigation and a short search query. Scrolling did not visibly move the list through automation and remains unverified.

The exact iOS build was recovered after initial observations by tapping the version card; the reconnect log independently reports the same build. This is a retrospectively version-bound baseline, not a controlled cross-version experiment.

Scoped native unified logs corroborate selected client-side phases and one transport route. They do not provide a debugger-observed call sequence, server-side policy, decrypted payload or initial fresh host enrollment. Internal “unlock” operation names are not independent physical lock-state telemetry.

## Public evidence export

The repository publishes redacted event selections, observation summaries, findings and artifact identities. These are derived exports, not the complete original REA records. Original Evidence IDs and local source filenames serve as traceability references where available; referenced raw files are not shipped.

Raw diagnostics, credentials, transient personal/device/network identifiers, screenshots with incidental personal content, Apple binaries, extracted frameworks and tool installations remain outside the public export.

The first-phase local validation checked source links, Evidence IDs, analyzed target digests, parent/slice byte equality and selected instruction words. A separate reviewer checked the principal code/log claims and found no critical or important overclaim. This verifies the research record's selected evidence; it is not a test of Windows compatibility.
