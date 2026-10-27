# Floresta GUI releases

Signed `florestad` builds that [Floresta GUI](https://github.com/cristiansolutchak) installs automatically. Floresta publishes no prebuilt binaries for v0.9.1, so these are built here from the official, PGP-signed source and signed for the app.

Nothing in this repository is a fork or a modified Floresta: every binary is the upstream release tag, built unchanged.

## Contents

| Path | What it is |
|---|---|
| `florestad/manifest.json` | The signed list of builds the app reads (Ed25519 signature over `payload.json`). |
| `florestad/payload.json` | The exact bytes that were signed: version, upstream tag and commit, and each build's URL, SHA-256, size and minimum OS. |
| `florestad/florestad-v0.9.1-<os>-<arch>` | The binaries: macOS arm64/x64, Linux arm64/x64. Windows is added after it is built and tested on Windows. |
| `florestad/*.build-info.json` | How each binary was built: Rust version, source archive hash, minimum OS. |
| `florestad/SHA256SUMS` | Checksums of the binaries. |
| `KEYS` | The public keys the app trusts for `manifest.json`. |
| `LICENSE-MIT`, `LICENSE-APACHE` | Floresta's licence (MIT OR Apache-2.0), which covers these binaries. |

## How each binary was made

- Source: [getfloresta/Floresta](https://github.com/getfloresta/Floresta) tag `v0.9.1` at commit `bc2db8d07e72651f9981ce589c5688f4d575dc7a`. The tag's PGP signature is checked against Davidson Souza's key (primary `2C8E0F836FD7DBBBB9E9B2EF89964EC3AB22B2E3`) before building.
- Built from a `git archive` export with `cargo build --release --locked -p florestad`, Rust 1.95.0: macOS natively (minimum macOS 11.0), Linux in Debian 12 containers (glibc 2.34 or newer).
- Before listing, each binary must have the expected architecture, only system libraries, and `--version` printing exactly `florestad v0.9.1`.

The build scripts are in the Floresta GUI repository: `tool/release/build_florestad_artifact.sh` and `tool/release/build_florestad_linux_in_container.sh`.

## How the app checks them

Floresta GUI downloads `florestad/manifest.json` and accepts it only if all of these hold:
- it is signed by a key in `KEYS`;
- it has not expired;
- its serial is not older than one already seen.

It then downloads the build for the computer it runs on. That build must:
- match the manifest's size and SHA-256;
- run and report the manifest's version.

Only then is it installed.

To check a download yourself:

```bash
cd florestad && shasum -a 256 -c SHA256SUMS
```

## Expiry

`manifest.json` expires 180 days after signing (this one: 2027-04-25). The app refuses an expired manifest, so a new one must be signed before then, even with no new binaries.
