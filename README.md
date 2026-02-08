# Chessar Engine Index

**Chessar Engine Index** is a public, versioned index of **UCI chess engine binaries** used by the **Chessar platform** and its companion CLI, **`chessar-engine-manager`**.

This repository does **not** contain source code for engines.
Instead, it provides a **stable, machine-readable distribution index** that points to prebuilt engine binaries hosted as GitHub Release assets.

## Main Page Preview

![Chessar Engine Index Main Page](docs/assets/main-page.png)

---

## Goals

This repository exists to:

* Provide a **stable, canonical download index** for UCI engines
* Support **multiple operating systems**, CPU architectures, and instruction-set variants (AVX2, AVX-512, generic)
* Allow **deterministic, automated installation** via CLI or CI
* Decouple engine distribution from the Chessar application runtime
* Scale cleanly to **hundreds of engines and versions**

---

## How It Works (High Level)

1. Each engine version has a **directory** in this repository
2. That directory contains an `index.html` file
3. The `index.html` links to **GitHub Release assets** that contain engine binaries
4. The Chessar Engine Manager:

   * Fetches the index page
   * Selects the **best binary for the current OS + CPU**
   * Downloads and installs the engine
   * Records metadata in a local `manifest.json`

This design avoids scraping GitHub APIs for every resolution and allows static hosting via **GitHub Pages**.

---

## Repository Structure

```
chessar-engine-index/
├── stockfish/
│   ├── 17.1/
│   │   └── index.html
│   ├── 16/
│   │   └── index.html
│   └── README.md
├── berserk/
│   └── 1/
│       └── index.html
├── lc0/
│   └── 0.31/
│       └── index.html
├── README.md
```

### Key Rules

* **One directory per engine**
* **One directory per engine version**
* Versions may be **major-only** (`17`) or **major.minor** (`17.1`)
* Each version directory **must contain `index.html`**

---

## Versioning Strategy

### Supported Version Formats

The index supports both:

* **Major versions**: `17`
* **Minor versions**: `17.1`

How they are resolved depends on your tooling configuration:

| User Input | Index Used                               |
| ---------- | ---------------------------------------- |
| `17`       | `/stockfish/17/`                         |
| `17.1`     | `/stockfish/17.1/` *or* `/stockfish/17/` |

> The Chessar Engine Manager supports both **exact** and **major-fallback** resolution.

### Recommendation

* Use **major directories** unless binaries differ by minor version
* Use **minor directories** only when required

---

## `index.html` Format

The `index.html` file is intentionally simple and static.

### Example: Stockfish 17

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Stockfish 17</title>
</head>
<body>

<h1>Stockfish 17</h1>

<ul>
  <li>
    <a href="https://github.com/official-stockfish/Stockfish/releases/download/sf_17/stockfish-windows-x86-64-avx2.zip">
      stockfish-windows-x86-64-avx2.zip
    </a>
  </li>
  <li>
    <a href="https://github.com/official-stockfish/Stockfish/releases/download/sf_17/stockfish-windows-x86-64.zip">
      stockfish-windows-x86-64.zip
    </a>
  </li>
  <li>
    <a href="https://github.com/official-stockfish/Stockfish/releases/download/sf_17/stockfish-linux-x86-64-avx2.tar.gz">
      stockfish-linux-x86-64-avx2.tar.gz
    </a>
  </li>
  <li>
    <a href="https://github.com/official-stockfish/Stockfish/releases/download/sf_17/stockfish-linux-x86-64.tar.gz">
      stockfish-linux-x86-64.tar.gz
    </a>
  </li>
</ul>

</body>
</html>
```

### Requirements

* Links **must be absolute URLs**
* Links **must point to GitHub Release assets**
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
3. Path: `/` (root)

### Resulting URLs

```
https://<org>.github.io/chessar-engine-index/stockfish/17/
https://<org>.github.io/chessar-engine-index/berserk/1/
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
chessar-engine-manager resolve stockfish --version 17
chessar-engine-manager install stockfish --version 17.1
```

---

## Security Considerations

* Only link to **trusted GitHub repositories**
* Prefer official engine releases
* Avoid mutable assets (do not overwrite release files)
* Checksums may be added in the future

---

## Adding a New Engine (Checklist)

1. Create engine directory: `engine-name/`
2. Create version directory: `engine-name/<version>/`
3. Add `index.html` with download links
4. Ensure binaries exist as GitHub Release assets
5. Write a meaningful release description
6. Commit and push
7. Verify Pages URL loads correctly

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

---

If you want, I can also provide:

* A **template `index.html` generator**
* Validation tooling for index consistency
* A CI check to ensure all index pages resolve
* A short “engine submission guide” for contributors
