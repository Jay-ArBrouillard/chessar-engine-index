# Chessar Engine Index

**Chessar Engine Index** is the public catalog for current **UCI engine builds** supported by Chessar calibration and used by **`chessar-engine-manager`**.

This repository does **not** contain source code for engines.
Instead, it provides a **stable, machine-readable distribution index** that points to prebuilt engine binaries hosted as GitHub Release assets.

[Browse the current engine index](https://jayarbrouillard.com/chessar-engine-index/)

---

## Goals

This repository exists to:

* Provide a **stable, canonical download index** for UCI engines
* Support **multiple operating systems**, CPU architectures, and instruction-set variants (AVX2, AVX-512, generic)
* Allow **deterministic, automated installation** via CLI or CI
* Decouple engine distribution from the Chessar application runtime
* Publish one current certified version per engine family

---

## How It Works (High Level)

1. Each engine version has a **directory** in this repository
2. That directory contains an `index.html` file
3. The `index.html` links to **GitHub Release assets** or to the engine's upstream distribution and manual installation instructions
4. The Chessar Engine Manager:

   * Fetches the index page
   * Selects the **best binary for the current OS + CPU**
   * Downloads and installs the engine
   * Records metadata in a local `manifest.json`

This design avoids scraping GitHub APIs for every resolution and allows static hosting via **GitHub Pages**.

---

## Repository Structure

- `docs/index.html` lists the current calibration roster.
- `docs/<engine>/<version>/index.html` lists downloads or setup instructions for that build.
- Use the exact version shown in the index. When calibration adopts a new version, replace the previous version's listing and page.

## `index.html` Format

The `index.html` file is intentionally simple and static.

### Stockfish 18 example

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Stockfish 18 Downloads</title>
  </head>
  <body>
    <h1>Stockfish 18</h1>

    <ul>
      <li>
        <a href="https://github.com/Jay-ArBrouillard/chessar-engine-index/releases/download/stockfish-18/stockfish-windows-x86-64-avx2.zip">
          stockfish-windows-x86-64-avx2.zip
        </a>
      </li>

      <li>
        <a href="https://github.com/Jay-ArBrouillard/chessar-engine-index/releases/download/stockfish-18/stockfish-ubuntu-x86-64-avx2.tar">
          stockfish-ubuntu-x86-64-avx2.tar
        </a>
      </li>
    </ul>

  </body>
</html>
```

### Requirements

* Links **must be absolute URLs**
* Prefer GitHub Release assets for automated downloads.
  * When an engine comes from an authorized third-party bundle, the version page can link to its files for manual installation.
  * For an upstream-only release, link directly to the publisher's download. For Komodo 14.1, follow the manual installation and Chessar worker instructions on its version page.
* Include the corresponding source and license when redistributing binaries that require them.
* File names should clearly encode:

  * OS
  * architecture
  * instruction-set variant (if applicable)

---

## Variant Selection (AVX, Generic, etc.)

The Chessar Engine Manager automatically selects the **best compatible variant**.

### Preference Order

| Platform | Preference Order         |
| -------- | ------------------------ |
| Linux    | AVX-512 → AVX2 → Generic |
| Windows  | AVX2 → Generic           |

### Variant Naming Conventions

Use consistent naming in asset filenames:

| Variant | Expected Token         |
| ------- | ---------------------- |
| AVX-512 | `avx512` or `avx-512`  |
| AVX2    | `avx2`                 |
| Generic | no suffix or `generic` |

---

## Engine Profile Attribution

Chessar imports [`docs/engine-attribution.json`](docs/engine-attribution.json) for public managed engine profiles.

- Use `schemaVersion: 1` and one entry in `projects` per Chessar engine family. Use the site's family key, such as `stockfish` or `black marlin`.
- Set `authors` to the public names or handles credited by the project. Write a short `description` in your own words and use `projectUrl` for the original project's public site or repository. Use `creditsUrl` for a separate credits page or a preserved publisher document.
- Add records under `builds` using the exact version Chessar displays. Set `sourceUrl` to the corresponding upstream release or source, `license` to the license for that build, and `licenseUrl` to the versioned license or publisher terms.
- Keep previous build records so game history can still link to those releases. Leave a field out when its attribution or source is uncertain.
- After publishing an update, select **Refresh attribution** in Chessar's **Engine admin → Engines** workspace.

## Release Descriptions

Each engine version **should have a GitHub Release** containing:

* Binary assets
* A **release description** (body text)

### Why this matters

The Chessar Engine Manager:

* Fetches the release description
* Stores it in the local `manifest.json`
* Makes it available to UIs and APIs

### Best Practice

Write the release description as:

* Plain text or Markdown
* First paragraph = short engine overview
* Remaining content = release notes

---

## Hosting via GitHub Pages

This repository is designed to be hosted via **GitHub Pages**.

### Setup

1. Enable GitHub Pages
2. Source: `main` branch
3. Path: `/docs`

### Resulting URLs

```
https://jayarbrouillard.com/chessar-engine-index/stockfish/18/
https://jayarbrouillard.com/chessar-engine-index/berserk/13/
```

These URLs are consumed directly by the engine manager.

---

## Integration with Chessar Engine Manager

The companion tool **`chessar-engine-manager`** uses this index to:

* Resolve engine versions
* Select optimal binaries
* Download and install engines
* Generate a local `manifest.json`

Example:

```bash
chessar-engine-manager resolve stockfish --version 18
chessar-engine-manager install stockfish --version 18
```

---

## Security Considerations

* Only link to **trusted GitHub repositories**
* Prefer official engine releases
* Avoid mutable assets (do not overwrite release files)
* Checksums may be added in the future

---

## Adding a New Engine (Checklist)

1. Confirm the exact engine and version in the [Chessar ranked engine roster](https://chessar.org/api/v1/engines/ranked).
2. Add or update `docs/<engine>/<version>/index.html` with download links or manual installation instructions.
3. Update `docs/index.html` and keep one page for the current build of each family.
4. Verify the index, version page, and download links after publishing.

---

## Scope and Non-Goals

This repository intentionally does **not**:

* Build engines from source
* Host binaries directly
* Provide runtime configuration
* Track ELO ratings or benchmarks

Those responsibilities belong to **Chessar** itself.

---

## License

This repository contains **index metadata only**.

All chess engines linked from this repository are subject to their **own licenses**.
Refer to the upstream engine repositories for licensing details.

---

## Questions / Contributions

If you are contributing engines or index updates:

* Follow existing naming and structure
* Keep index pages simple
* Prefer clarity over cleverness

For platform integration questions, refer to the Chessar project documentation.
