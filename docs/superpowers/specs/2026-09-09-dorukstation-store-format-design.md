# DorukStation Store Repository Format Design

Date: 2026-09-09

## Goal

Define the official DorukStation Store repository format used by DorukStation to discover, display, download, install, and launch verified games and applications.

The Store must support both:

1. DorukStation-hosted payloads, such as DorukCraft and DorukCraft Dungeons.
2. External vendor downloads, such as Steam and Epic Games Launcher, where DorukStation presents a native Store experience while resolving and downloading the latest official installer from the vendor.

## Repository

The official catalog repository is:

`asildorukatik/DorukStation-Game-Library`

DorukStation should fetch `catalog.json` first and use it as the authoritative index of Store items.

## Directory Layout

```text
DorukStation-Game-Library/
├── catalog.json
├── README.md
├── docs/
└── Apps/
    ├── DorukCraft/
    │   ├── Outer/
    │   │   ├── manifest.json
    │   │   ├── description.txt
    │   │   ├── logo.png
    │   │   ├── banner.png
    │   │   ├── Screenshots/
    │   │   └── Videos/
    │   └── Inner.zip
    ├── DorukCraft-Dungeons/
    │   ├── Outer/
    │   │   ├── manifest.json
    │   │   ├── description.txt
    │   │   ├── logo.png
    │   │   ├── Screenshots/
    │   │   └── Videos/
    │   └── Inner.zip
    ├── Steam/
    │   └── Outer/
    │       ├── manifest.json
    │       ├── description.txt
    │       ├── logo.png
    │       ├── Screenshots/
    │       └── Download.link
    └── Epic-Games-Launcher/
        └── Outer/
            ├── manifest.json
            ├── description.txt
            ├── logo.png
            ├── Screenshots/
            └── Download.link
```

## Outer vs Inner

### Outer

`Outer/` contains Store-facing metadata and media only:

- `manifest.json`
- `description.txt`
- `logo.png`
- optional `banner.png`
- optional `Screenshots/`
- optional `Videos/`
- optional `Download.link` for externally-hosted apps

DorukStation may fetch these independently without downloading the application payload.

### Inner

`Inner.zip` contains the actual hosted game or application payload.

For hosted DorukStation games, `Inner.zip` is the installable payload.

For external vendor applications, there is no `Inner.zip`; the Store resolves the official installer from `Download.link`.

## catalog.json

Example:

```json
{
  "format": 1,
  "items": [
    "Apps/DorukCraft/Outer/manifest.json",
    "Apps/DorukCraft-Dungeons/Outer/manifest.json",
    "Apps/Steam/Outer/manifest.json",
    "Apps/Epic-Games-Launcher/Outer/manifest.json"
  ]
}
```

DorukStation should not crawl the repository tree. It should load this index and then fetch only the listed manifests.

## Manifest Schema

Every verified Store app has a `manifest.json`.

Required fields:

```json
{
  "format": 1,
  "id": "example-app",
  "name": "Example App",
  "publisher": "Example Publisher",
  "version": "1.0.0",
  "packageType": "hosted",
  "descriptionFile": "description.txt",
  "payload": {
    "fileType": "web-pwa",
    "source": "../Inner.zip"
  },
  "runtime": {
    "handler": "dorukstation-web"
  }
}
```

Optional Store metadata may include:

- age rating
- player count
- genres
- supported languages
- VR support
- install size
- download size
- screenshots
- videos
- banner
- tags
- developer/publisher metadata

## packageType

Supported values initially:

- `hosted` — payload is stored in the DorukStation Store repository.
- `external` — payload is obtained from an official vendor source using `Download.link`.

## Verified fileType

The manifest contains an explicit payload type so DorukStation does not guess which runtime or installer path to use.

Initial values:

- `windows-exe`
- `windows-msi`
- `linux-exe`
- `appimage`
- `deb`
- `rpm`
- `flatpak`
- `web-pwa`
- `html`
- `ps1-rom`
- `ps2-rom`
- `ps3-game`
- `psp-rom`
- `nes-rom`
- `snes-rom`
- `gba-rom`

This list is extensible.

## Runtime Handling

Examples:

```json
"runtime": { "handler": "native" }
```

```json
"runtime": { "handler": "dorukstation-web" }
```

```json
"runtime": { "handler": "windows-compat" }
```

```json
"runtime": { "handler": "ps3-emulator" }
```

DorukStation decides which built-in emulator or compatibility layer is used for the declared payload type.

For Windows software on Linux, the runtime is a compatibility layer such as Wine/Proton, even if DorukStation groups it together with its emulator subsystem in the user interface.

## Verified Store Trust Model

The manifest must not contain a self-declared field such as:

```json
"verified": true
```

Verification is assigned by DorukStation itself based on the source of the manifest.

A manifest loaded through the official trusted DorukStation catalog is considered Store Verified. Only Store Verified manifests may directly instruct DorukStation which `fileType` and runtime handler to use.

For sideloaded or untrusted packages:

- DorukStation must not trust a manifest-supplied runtime/fileType automatically.
- DorukStation should inspect the real payload.
- DorukStation may ask the user before running an inferred handler.

## External Download.link

`Download.link` is a UTF-8 text file using a custom `.link` extension.

Example:

```text
DORUKSTATION-LINK/1
mode=official-latest
provider=steam
page=https://store.steampowered.com/about/
platform=auto
```

Epic example:

```text
DORUKSTATION-LINK/1
mode=official-latest
provider=epic
page=https://store.epicgames.com/download
platform=windows
```

`mode=official-latest` tells DorukStation to resolve the current official installer rather than storing a version-specific installer URL that may become stale.

## External Resolver Security

The external resolver must only accept downloads and redirects from an approved domain allowlist for the selected provider.

Examples:

- Steam resolver: approved Valve/Steam-controlled download domains only.
- Epic resolver: approved Epic-controlled download domains only.

If the final resolved URL leaves the allowed provider domain set, the download must be rejected.

The Store should also verify that the actual downloaded file matches the expected `fileType` before execution.

## Steam Behavior

Steam has native Linux and Windows installers.

Preferred behavior:

- Linux DorukStation: resolve the native Linux Steam package.
- Windows DorukStation: resolve `SteamSetup.exe`.
- Other supported DorukStation environments may choose a compatible payload according to their runtime support.

Steam may therefore use platform-specific entries under `payloads` instead of one fixed payload.

## Epic Games Launcher Behavior

Epic Games Launcher currently uses the Windows installer for this Store integration.

On Linux DorukStation:

1. Resolve the official latest Epic Windows installer.
2. Download the `.exe`.
3. Use the manifest's trusted `windows-exe` file type.
4. Launch through DorukStation's Windows compatibility runtime.

On Windows DorukStation, run the installer natively.

## Initial Store Entries

The first catalog contains:

1. DorukCraft — hosted DorukStation package.
2. DorukCraft Dungeons — hosted DorukStation package.
3. Steam — external official-latest package.
4. Epic Games Launcher — external official-latest Windows package.

## DorukCraft Packaging

The newest supplied DorukCraft build is packaged as `Inner.zip` without unpacking its Store metadata into the payload folder.

The Store-facing logo and screenshots remain under `Outer/`.

The DorukCraft manifest identifies its file type as `web-pwa` and runtime as `dorukstation-web`.

## DorukCraft Dungeons Packaging

DorukCraft Dungeons is packaged as a hosted payload in `Inner.zip`.

The existing HTML build is retained as the game content and the manifest identifies it as an HTML/web runtime payload.

## Update Behavior

DorukStation compares the Store manifest version with the locally installed version.

For hosted apps:

- a newer manifest version points to the updated `Inner.zip`.

For `official-latest` external apps:

- DorukStation resolves the vendor's current installer at install/update time.
- the Store does not need to update every time the vendor changes a version-specific URL.

## Error Handling

DorukStation should fail safely when:

- `catalog.json` is malformed.
- a listed manifest is missing or malformed.
- `Inner.zip` is missing.
- a payload's actual type conflicts with the trusted `fileType`.
- an external resolver leaves the provider's approved domain allowlist.
- a download is incomplete.
- an unsupported runtime handler is requested.

A broken Store item should not make the entire Store unavailable.

## Compatibility

`format` fields are versioned so future DorukStation releases can introduce new schema versions without silently misinterpreting old manifests.

Unknown optional fields should be ignored. Unsupported required format versions should be rejected with a clear compatibility message.
