# Secure Enclave: what can we investigate next?

14 October 2026. The exact iOS 27.0 (24A437) iPhone pairing daemon is now acquired. **A direct Windows host is still unverified.**

## Progress after storage cleanup

About 10.9 GB of regenerable download, compiler and IDE caches were removed. Installed analysis tools and unique research evidence were retained. The filesystem component was retrieved from Apple's exact-build restore image, decrypted and mounted read-only, and `/usr/libexec/sharingd` copied for offline analysis. The mounted image's system-version metadata is 27.0 / 24A437 and source/copy bytes match.

Verification initially compared the encrypted carrier's SHA384 with the manifest OS field and found a mismatch. Independent ZIP size/CRC32 checks matched. The decrypted image's SHA384 then matched the OS manifest exactly. For this component, that manifest field covers plaintext image bytes; no whole-carrier manifest match is claimed.

See [curated peer evidence](../evidence/peer-enrollment.json) for artifact identities and selected locations. Large distribution working images can be reacquired from the recorded source; selected binaries, hashes and raw inspection records are retained locally.

## Selected iPhone pairing boundary

In this daemon, SDAutoUnlockPairingLockSession's `useAttestedProtocol` at `0x10006b808` returns true. Its `handleDeviceAuthenticationSetupRequest` calls the AKS pairing constructor at `0x10006d57c`, passing `requiresAttestation=1` and `originator=0`.

The called constructor at `0x1001226e8` chooses `localAttestedLTK` when attestation is requested and takes a missing-key error path rather than falling back to an ordinary local key. Original instruction words and selector references are checked against the acquired executable.

These are selected **legacy pairing lock-session** boundaries. Their reachability from current mirroring is not established. Two direct branch call sites do not exhaust modern or indirect dispatch, and this does not recover the iPhone's certificate-verifier internals.

## What remains possible to test

We can trace the current iPhone enrollment route, recover message/certificate representations and determine which attestation checks it enforces. This work can proceed through OS daemons and key-service interfaces without decrypting SEP application code. A permitted alternative enrollment path, if one exists, is the relevant lead for a software host.

Emulating firmware is separate from reproducing a trusted identity. Apple documents hardware-bound root keys and engine keys that software cannot extract. Learning the protocol or obtaining code does not establish possession of those keys. This is a general constraint, not proof that a particular mirroring signing key has a particular derivation. [Secure Enclave architecture](https://support.apple.com/guide/security/the-secure-enclave-sec59b0b31ff/web).

The general remote-unlock framework also documents an older kernel endpoint for Macs without Secure Enclaves. That is a research lead, not a demonstrated non-attested mirroring mode; current supported Mirroring hosts require Apple silicon or T2. [Remote-unlock architecture](https://support.apple.com/guide/security/automatically-unlock-apple-devices-sec6ab47ebfc/web), [Mirroring requirements](https://support.apple.com/en-us/120421).

If current peer policy permits software-held keys, direct Windows interoperability may be possible after reconstructing enrollment and the session protocol. If it requires an Apple-backed hardware identity and has no compatible alternative, producing a self-signed key or emulating the protocol alone will not satisfy that requirement. Neither condition has been established by a Windows-origin test.

## Next check

Trace modern/current mirroring enrollment into the peer's long-term-key acceptance and attestation failure handling. The encrypted SEP payload and complete verifier remain unrecovered. No new pairing, private-key export or device-protection change was performed.
