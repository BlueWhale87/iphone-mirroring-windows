# Can Windows enroll its own trusted keys?

14 October 2026. **An ordinary Windows software key does not reproduce the selected native enrollment path. Stock iPhone acceptance remains untested.** The new evidence narrows the problem to attested key generation and peer enrollment, rather than just implementing an encryption handshake. A [subsequent peer acquisition](secure-enclave-options.md) obtained exact-build iPhone sharingd after storage cleanup; the acquisition limits below describe this earlier pass.

## Selected pairing policy

The enrollment adapter forwards external authentication type 5 through the helper at `0x1001ec924`. Its source table at `0x100771310` maps 5 to internal 5; session-authentication type 6 maps to internal 6.

In SDAuthenticationAKSPairingSession, the named `type` field is linked through original Objective-C metadata and chained-fixup pointers to the global field-offset variable `0x1008e57c0`. Pairing creation reads that field, checks `type < 21` and tests mask `0x1800e0` before calling `aksPairingSessionForDeviceID:originator:errorCode:requiresAttestation:requiresEscrow:` at `0x1003de78c`. Types 5 and 6 set the attestation bit. The called constructor selects `localAttestedLTK` when that bit is set and takes an error path if the local key is missing; it has no ordinary-key fallback within that constructor.

This establishes policy in the selected class. It does not establish that every currently reachable enrollment route uses that class, or that the iPhone has no alternative. The actual field displacement is initialized at runtime; interpreting zero-fill file offset 0 as a stored displacement produces bogus values and is excluded.

## What the attested generator does

The native helper at `0x1001648ec` distinguishes ordinary and attested generation. The ordinary branch calls `aks_create_signing_key`. The attested branch:

1. Calls `MAECopyDCRTMacOSWithError` to obtain DCRT data. Missing data takes an error path in this helper.
2. Adds those bytes as AKS parameter `0x15`, and numeric value 4 as parameter `0x14`. Parameter IDs are observed; their formal enum names are not recovered.
3. Serializes the parameters and calls `aks_create_signing_key_with_params` at `0x100164c20`.

A conditional callback path can refresh DCRT through `MAEIssueDCRTMacOSWithCallback`. Its completion handler reports activation errors and invokes the key-generation block on success. No activation request was executed and no certificate or private key contents were read.

The generation block stores the resulting key representation in the attested or ordinary local-key slot. Selected remote-key signing/storage and `aks_remote_peer_setup` boundaries are also located. The peer-setup wrapper in AppleKeyStore selects operation `0x2e` and reaches `IOConnectCallMethod`. This is a Mac driver boundary, not the recovered iPhone certificate verifier.

## Windows assessment

These observations are evidence against a straightforward port that substitutes an ordinary software-generated key for Apple's attested key. A Windows key's ability to sign data would not demonstrate compatibility with the selected DCRT/AKS enrollment path.

The decisive remaining question is whether the current iPhone validates an Apple-backed attestation chain for this relationship, and whether any permitted alternative enrollment exists. That requires the peer verifier or an independently enrolled software-host test. Neither has been obtained. This report therefore makes no universal impossibility claim and reports no successful Windows session.

## Exact iPhone build and acquisition boundary

Apple's restore metadata identifies iPhone 16 Pro (`iPhone17,1`), iOS 27.0 build `24A437`. Its small AppOS component was retrieved and verified against the manifest SHA-384. That component is not the mirroring daemon. The attempt to search firmware files stopped at AppOS mounting without obtaining sharingd.

The encrypted filesystem component is about 8.9 GB, with additional space needed for decryption and extraction. Acquisition remains pending additional disk headroom and a working filesystem extraction route. No personal files were removed.

A smaller acquisition route recovered the exact-build kernel and AppleSEPKeyStore driver. The original kernel carrier matches the manifest SHA-384, independent decompression reproduces the analyzed kernel, and every extracted driver text byte matches its source fileset. C++ metadata locates AppleKeyStoreUserClient; unnamed methods and remote-session entitlement strings remain search seeds, without a claimed peer protocol role.

The exact-build SEP firmware was also acquired and verified against the manifest. Its `sepi` payload is reported encrypted. Extracting the raw payload did not recover plaintext application code; its public wrapped keybag fields are not plaintext decryption keys. The layout-parser failures are analysis limits, not iPhone rejection evidence.

The next concrete step is to obtain source-bound iPhone sharingd, follow its enrollment route into these key-service boundaries, and establish peer-key acceptance, certificate checks and failure branches. The underlying SEP verifier remains unrecovered. A generic self-signed certificate would not be an informative compatibility test.

## Evidence and limits

See [the curated ledger](../evidence/key-attestation.json) for locations, identities and confidence. Original artifacts and disassembly stay local. The new AppleKeyStore image is bound to the same verified cache UUID; its extracted identity is recorded in the ledger. Source instruction checks are performed against the original executable images.

Ghidra's daemon dossier request timed out at the 60-second provider budget. It was preserved and not retried identically. Native disassembly and source metadata supplied these findings. There was no new runtime pairing, activation call, credential export, protection change or Windows-origin probe.
