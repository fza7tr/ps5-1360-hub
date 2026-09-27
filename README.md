# PS5 13.60 Hub

[اقرأ هذا الملف بالعربية](README.ar.md)

A free Windows toolkit for tracking PS5 firmware 13.60 research, finding original homebrew releases, browsing PS5 game information, and using local console services when they are already running.

## What is included

- An English/Arabic console cockpit with full right-to-left layout, a language switch, and an English/Arabic Windows installer. The app follows the Windows language on first launch.
- A source-linked capability matrix that distinguishes a complete public jailbreak from individual SDK and payload updates.
- Bilingual guides and a curated homebrew library with favorites, category and compatibility filters, Arabic search, publisher release assets, and review dates.
- Download progress, cancellation, and safe resumption when the publisher supports ranges and validators. SHA-256 is always calculated; when GitHub supplies the publisher digest, the app verifies it before replacing any existing file.
- PS5 commercial game search through RAWG. Commercial game entries provide information and links, not game files or firmware compatibility claims.
- Local service checks, an ELF sender for an already-running `elfldr` service, and FTP upload to homebrew paths.
- SQLite-backed settings, favorites, downloaded-file history, release cache, and a bundled bilingual offline catalog. Existing version 0.4 settings, favorites, local PKGs, and downloads are retained on upgrade.
- An NSIS Windows installer and signed app-update capability.
- A live source check on launch, each hour while running in the window or tray, and on **Refresh all now**. It checks Sony's official firmware notes, the selected catalog feed, and publisher releases in sequence. New release posts appear as activity, not automatically confirmed compatibility.
- A per-source health table shows the last attempt, last success, cache age, failure, and next retry. Windows notifications announce only newly observed official firmware and reviewed jailbreak status changes after an initial baseline.
- Full RAWG game detail pages show artwork, screenshots, genre, publisher, release facts, and sourced firmware evidence. Commercial search is delayed briefly while typing, cached locally, and retains recent searches and favorites.
- The app can start with Windows when enabled in Settings. Closing its window keeps hourly checks running in the tray; the tray has Open, Refresh, and Exit actions.
- Latest official PS5 firmware beside the 13.60 focus. Minimum firmware and documented tested versions appear only when the catalog includes primary evidence; otherwise they show **Unknown**.
- A local PKG library for your own base, update, and DLC files. It records file paths and SHA-256 hashes on this PC and flags missing files. Publisher PKG assets appear only when listed in original publisher releases.
- Sony PlayStation Store links and sourced Arabic screen/voice information for selected Saudi listings. Choose another Store region in Settings; unmatched listings remain **Unknown**.
- An hourly regional-manifest check shows the reported `force_update` minimum, latest manifest version, regional coverage, source, and check time. This value does not establish PS Store access. Separate dated, regional Store access reports show **Mixed**, **Stale**, or **Unknown** when the evidence warrants it.
- An hourly local SQLite index of the public `pkg.games` PS5 game pages. A commercial game gets a direct third-party page link only for an unambiguous normalized title match; edition and release year are checked when the source provides them. PSPKG checked pages may be supplied in the curated feed; unmatched titles get a PSPKG site search. HowLongToBeat is a search link, without embedded playtime estimates.

## Get a free RAWG key

1. Create a personal/hobby key at [RAWG API Docs](https://rawg.io/apidocs).
2. Launch the app and paste it into **Settings → RAWG API key**.
3. Save Settings. The key is encrypted for your Windows user account with DPAPI before it is stored in SQLite; it is not embedded in the EXE. A key saved by an older version is migrated on first launch. Do not commit it or paste it into an issue.

No GitHub API key or OpenAI key is needed to run the app. Public GitHub release lookups are rate limited; the app caches them and uses conditional requests during hourly and manual checks.

## Build locally

Requirements: Windows x64, Node.js, Rust (MSVC target), Microsoft C++ Build Tools, and WebView2. Tauri's [Windows prerequisites](https://v2.tauri.app/start/prerequisites/) describe the free installations.

```powershell
npm ci
npm run build
npm run tauri dev
```

To produce a signed Windows setup EXE, generate an update signing key **once**, then keep its private file safe:

```powershell
npm run tauri signer generate -- --ci --password '' --write-keys .\signing.key
$env:TAURI_SIGNING_PRIVATE_KEY = (Resolve-Path .\signing.key).Path
$env:PS5_HUB_UPDATER_PUBKEY = (Get-Content .\signing.key.pub -Raw).Trim()
npm run tauri build
```

`signing.key` is ignored by Git. Back it up securely; losing it prevents the installed app from trusting future updates. The setup EXE is written under `src-tauri/target/release/bundle/nsis/`. This update-signing key is free and is separate from optional Windows Authenticode signing.

## GitHub updates and curated feed

The default catalog owner is `fza7tr`. Keep `catalog.json` at the root of the public `ps5-1360-hub` repository on `main`. The app's **Refresh catalog** action reads `https://raw.githubusercontent.com/fza7tr/ps5-1360-hub/main/catalog.json`; if unavailable or invalid, the last valid local catalog remains usable. The signed app-update channel always uses `fza7tr/ps5-1360-hub`, regardless of a custom catalog source.

The included GitHub Actions workflow builds on a `v*` tag and publishes an NSIS installer with `latest.json`. Set repository secret `TAURI_SIGNING_PRIVATE_KEY` to the private key's complete contents and repository variable `PS5_HUB_UPDATER_PUBKEY` to the public key's complete contents before pushing a release tag. Do not change the signing key between releases. Public release assets are required for unauthenticated update checks.

## Catalog editing

`catalog.json` uses schema version 4 and requires English and Arabic text for every curated claim, guide, and homebrew note. Existing version 2 and 3 catalogs migrate locally with favorites, downloads, local PKG records, and settings preserved. Every firmware claim has a status, source, and review date. Add or change claims only after checking the publisher's primary source. A payload claiming firmware offsets does not imply a usable public jailbreak. The app validates incoming JSON before replacing the cached copy. Publisher names, game titles, and release notes remain in their original language.

`storeAccessReports` entries require a firmware version, `works` or `blocked` outcome, region, HTTPS source URL, and ISO observation time. Reports older than 48 hours are stale; opposing fresh outcomes are mixed. `checkedThirdPartyPages` accepts reviewed PSPKG page URLs with a title, check time, and optional release year. The bundled arrays are empty because no recent Store report or PSPKG page passed the current review. The app does not automate PSPKG's anti-bot challenge.

## Console checks

Enter the PS5's private IPv4 address in **Console**, then run a service check. The ELF sender only enables when port 9021 responds, an `.elf` is selected, and you confirm that the PS5's ELF loader is running there. FTP upload requires an FTP greeting on a common PS5 FTP port and accepts homebrew destination paths. These service checks do not determine the firmware or verify jailbreak success. Real-device behavior should be checked on the owner's PS5 before relying on transfers.

## Current limitations

- The bundled catalog is a reviewed starting set, not every homebrew project in the scene. Game ports such as DevilutionX, EDuke32, LakeSnes, and ScummVM may require original game data and a compatible console environment.
- RAWG needs the user's free key and an internet connection for new searches; cached searches remain available after a successful lookup.
- The 13.60 end-to-end jailbreak status is marked unverified until a public upstream release is checked.
- The unsigned Windows installer may show a Windows reputation prompt; the Tauri update bundle still has a cryptographic signature.
- The PKG library indexes local files; it does not obtain commercial game packages. Commercial results link to RAWG and the official PlayStation Store.
- `pkg.games` and PSPKG are labeled third-party page listings. Their package files and compatibility claims are not verified or downloaded by this app. A unique title match is only a navigation aid.
