# WA Pulse Releases

Official APK releases and updates for **WA Pulse** - an Xposed module that enhances
WhatsApp and WhatsApp Business.

## Download

Get the latest APKs from the [Releases](../../releases) page, or use the built-in updater
inside the app.

> **Note:** releases made after the signing-key rotation are signed with a new key. If
> installation over an older build fails with a signature mismatch
> (`INSTALL_FAILED_UPDATE_INCOMPATIBLE`), uninstall the old app once and install the new APK.

## How builds work

This repository builds and publishes the APKs; the source of truth lives in the private
`AltafYafai/WA-Pulse` repository, which never runs Gradle itself:

- Every push to `master` dispatches a `build` event here. `build.yml` checks out that exact
  commit, compiles WhatsApp + Business APKs, signs them and publishes a release tagged
  `v<version>-<short-sha>` (default version `1.0.0`).
- Pushing a `v*` tag (or dispatching manually) sends a `release_build` event with the
  version and the changelog, producing a release tagged `v<version>`.

The private repo needs a `DISPATCH_TOKEN` secret (a PAT with write access here) to send the
dispatch events (`RELEASE_TOKEN` also works).

### Required repository secrets

Builds fail early (with a clear error) when these are missing. Configure them in
**Settings -> Secrets and variables -> Actions**:

| Secret | Purpose |
|---|---|
| `KEYSTORE_BASE64` | base64 of the release keystore (`keystore/release.jks` of the source repo) |
| `KEYSTORE_PASSWORD` | keystore store password |
| `KEY_ALIAS` | keystore key alias (e.g. `wapulse`) |
| `KEY_PASSWORD` | keystore key password |
| `SOURCE_REPO_TOKEN` | PAT with read access to the private `AltafYafai/WA-Pulse` repo (used to check out the source). Optional when a read-only deploy key is provided as `SOURCE_SSH_KEY`; legacy names `PAT`, `SOURCE_TOKEN` and `GH_PAT` are also accepted |

Optional: `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` for build/release notifications.

The keystore is never committed; `build.yml` restores it from `KEYSTORE_BASE64` before
building, so every APK is signed with the same release key and updates install cleanly over
previous releases. See `keystore/README.md` in the source repository for details.

## Workflows

| Workflow | Trigger | Result |
|---|---|---|
| `build.yml` | `build` dispatch (every push to the source repo) or manual dispatch | signed APK artifacts + a GitHub Release |
| `build.yml` | `release_build` dispatch (version tags / manual) | GitHub Release with the dispatched version and changelog |

## OTA supported versions

`supported-versions.xml` is fetched by installed apps so new WhatsApp beta versions can be
marked as supported without shipping an app update. Add verified versions there as
`<item>2.xx.yy.xx</item>` entries.
