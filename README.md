# DorukStation Game Library

This repository is the official DorukStation Store catalog.

DorukStation should read [`catalog.json`](./catalog.json) as the authoritative list of Store entries and then load each listed `manifest.json`.

## Current Store entries

- **DorukCraft** — DorukStation-hosted game metadata. The install payload is expected at `Apps/DorukCraft/Inner.zip`.
- **DorukCraft Dungeons** — DorukStation-hosted game metadata. The install payload is expected at `Apps/DorukCraft-Dungeons/Inner.zip`.
- **Steam** — external official download. No Steam installer or application binary is stored in this repository.
- **Epic Games Launcher** — external official download. No Epic installer or application binary is stored in this repository.

## External applications

Steam and Epic Games Launcher use lightweight `Download.link` files. DorukStation resolves the current installer from the vendor's official download flow at install time instead of storing a copy on GitHub.

This keeps the repository small and means vendor updates do not require committing a new installer to the DorukStation Store.

### Steam

Official source: `https://store.steampowered.com/about/`

### Epic Games Launcher

Official source: `https://store.epicgames.com/download`

## Store layout

```text
catalog.json
Apps/
  DorukCraft/
    Outer/
      manifest.json
      description.txt
    Inner.zip                 # hosted game payload; must be supplied
  DorukCraft-Dungeons/
    Outer/
      manifest.json
      description.txt
    Inner.zip                 # hosted game payload; must be supplied
  Steam/
    Outer/
      manifest.json
      description.txt
      Download.link           # official vendor resolver only
  Epic-Games-Launcher/
    Outer/
      manifest.json
      description.txt
      Download.link           # official vendor resolver only
```

See `docs/superpowers/specs/2026-09-09-dorukstation-store-format-design.md` for the Store format design.
