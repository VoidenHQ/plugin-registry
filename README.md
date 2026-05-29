# Voiden Plugin Registry

This repository is the unified registry for all Voiden plugins — both **core** plugins (maintained by the Voiden team) and **community** plugins (built and published by third-party developers).

The registry is a single flat JSON array in [`extensions.json`](./extensions.json). The Voiden app fetches this file on startup to discover available plugins, check for updates, and populate the Extension Browser.

---

## File Structure

```
extensions.json   ← unified plugin registry (flat JSON array)
README.md         ← this file
```

---

## Registry Format

`extensions.json` is a flat JSON array. Each entry is either a **core** entry or a **community** entry, distinguished by the `type` field.

### Common Fields (all plugins)

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | ✓ | Unique identifier for the plugin (e.g. `"voiden-rest-api"`). Must be URL-safe and globally unique. |
| `type` | `"core"` \| `"community"` | ✓ | Distinguishes core (Voiden-maintained) from community plugins. |
| `name` | string | ✓ | Human-readable display name shown in the Extension Browser. |
| `description` | string | ✓ | Short description of what the plugin does. |
| `author` | string | ✓ | Plugin author or organization name. |
| `version` | string | ✓ | Current published version (semver, e.g. `"1.0.0"`). |
| `repo` | string | ✓ | GitHub repository in `owner/repo` format (e.g. `"VoidenHQ/plugin-md-preview"`). Used to fetch releases, README, changelog, and manifest. |
| `voidenVersion` | string | ✓ | Semver range specifying compatible Voiden versions (e.g. `">=1.0.0"`). The app uses this to determine update compatibility. |
| `icon` | string | — | URL or icon identifier for the plugin. Displayed in the Extension Browser. |

### Core Plugin Fields

Core plugins are distributed via GitHub Releases and loaded dynamically (OTA updates). They may be bundled into the app or downloaded on first use.

| Field | Type | Required | Description |
|---|---|---|---|
| `bundled` | boolean | ✓ | `true` if the plugin is pre-bundled with the app (always available offline). `false` if it must be downloaded on first enable. |
| `priority` | number | ✓ | Load order priority. Lower numbers load first. Used to ensure foundational plugins (e.g. `voiden-rest-api` at 10) are ready before dependent plugins. |
| `mainProcess` | boolean | — | `true` if the plugin runs in the Electron main process. Defaults to `false` (renderer/UI only). |
| `capabilities` | object | — | Structured description of what the plugin registers (blocks, slash commands, sidebar tabs, pipeline hooks, etc.). See existing entries for shape examples. |
| `features` | string[] | — | Human-readable list of features shown in the Extension Browser detail panel. |

### Community Plugin Fields

Community plugins are installed as full plugin directories under `userData/plugins/community/<id>/`. Their README, changelog, and manifest are fetched on-demand when the user opens the detail panel.

| Field | Type | Required | Description |
|---|---|---|---|
| `icon` | string | — | URL to the plugin icon image. |

> Community plugins do **not** include `bundled`, `priority`, `mainProcess`, `capabilities`, or `features` in the registry — those are read from the installed plugin's `manifest.json` at runtime or fetched from the GitHub release on demand.

---

## Adding a Core Plugin

Add an entry to `extensions.json` with `"type": "core"`:

```json
{
  "type": "core",
  "id": "my-plugin",
  "repo": "VoidenHQ/plugin-my-plugin",
  "name": "My Plugin",
  "description": "What this plugin does.",
  "author": "Voiden Team",
  "version": "1.0.0",
  "voidenVersion": ">=1.0.0",
  "priority": 30,
  "bundled": false,
  "mainProcess": false,
  "capabilities": {},
  "features": [
    "Feature one",
    "Feature two"
  ]
}
```

The app also reads the plugin's `manifest.json` from its GitHub Release (`releases/latest/download/manifest.json`) for runtime metadata.

---

## Adding a Community Plugin

Add an entry with `"type": "community"`:

```json
{
  "type": "community",
  "id": "my-community-plugin",
  "repo": "yourname/voiden-plugin-my-plugin",
  "name": "My Community Plugin",
  "description": "A community-built plugin for Voiden.",
  "author": "Your Name",
  "version": "1.0.0",
  "voidenVersion": ">=1.0.0",
  "icon": "https://example.com/icon.png"
}
```

The plugin's `README.md`, `changelog.json`, and `manifest.json` are fetched on-demand from the GitHub repository when the user opens the detail panel.

---

## Update Compatibility

The `voidenVersion` field controls whether an update is shown as available or flagged as incompatible:

- If the running Voiden version satisfies `voidenVersion`, the update is offered normally.
- If not, the Extension Browser shows an amber warning: **"vX.Y.Z — needs Voiden >=X.Y.Z"** instead of a standard update prompt.

---

## How the App Fetches This Registry

The Voiden app fetches `extensions.json` from:

```
https://raw.githubusercontent.com/VoidenHQ/plugin-registry/main/extensions.json
```

- Core plugins are filtered by `type === "core"` and checked against the local build-time snapshot for update detection.
- Community plugins are filtered by `type === "community"` and displayed in the Extension Browser for discovery and install.
