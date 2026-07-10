# APNG Studio

An interactive [GitHub Copilot CLI](https://github.com/github/copilot-cli) **canvas extension** for building [Animated PNG (APNG)](https://wiki.mozilla.org/APNG_Specification) files from frames — draw or upload frames, tune the full practical APNG spec surface, preview live, and export an `.apng`.

The canvas renders in a side panel; the agent can also drive it through callable actions.

## Features

- **Frames** — upload images or draw them on a built‑in canvas (pen/eraser, fill, onion‑skin, "start from last frame"). Reorder, duplicate, and delete frames.
- **Per‑frame timing** — set the delay as an exact `numerator / denominator` fraction with a live `= N ms · N fps` readout.
- **Per‑frame compositing** — `dispose_op` (None / Background / Previous) and `blend_op` (Source / Over) dropdowns, straight from the APNG spec.
- **Apply to all** — set every frame's delay (ms), snap to an exact frame rate (fps), or apply dispose + blend in one click.
- **Loop count** — `0` = infinite, or a fixed number of plays.
- **Hidden first frame** — mark frame 1 as a static fallback: shown by non‑APNG viewers, excluded from the animation loop (encoded as a default image with no leading `fcTL`, `num_frames = N‑1`).
- **Live preview** — a real `/preview.apng` is assembled on every change; a **Reload** button re‑syncs state and rebuilds the preview.
- **Export** — writes a valid `.apng` to disk and returns its path.

## Install

### From Copilot CLI (recommended)

Ask Copilot to install it straight from this repo folder:

```
Install the canvas extension at
https://github.com/octobooth-1/apng-studio
```

…or use the `install_extension` tool / command palette with this repository URL. Choose the scope you want:

- **User** — `$COPILOT_HOME/extensions/apng-studio/` (defaults to `~/.copilot/extensions/`), available in every project.
- **Project** — `.github/extensions/apng-studio/` inside a repo, committed and shared with your team.
- **Session** — loaded only for the current session.

### Manual

Copy the source files into one of the extension directories above, keeping the layout:

```
apng-studio/
├── extension.mjs      # entry point (required name)
├── apng.mjs           # APNG codec + RGBA→PNG encoder
└── web/               # canvas iframe renderer
    ├── index.html
    ├── app.js
    └── styles.css
```

Then reload extensions. The `@github/copilot-sdk` import is resolved by the CLI — **do not** add a `package.json` or `node_modules` for it.

## Open the canvas

Once installed, open the **APNG Studio** canvas from Copilot. Optional open input:

| field       | type   | description                                    |
| ----------- | ------ | ---------------------------------------------- |
| `projectId` | string | Animation project id (defaults to `default`).  |
| `name`      | string | Optional display name for the animation.       |

Each project's frames persist on disk under `artifacts/<projectId>/`, so they survive reloads and are shared between every open panel and the agent actions. That folder is local user data and is **git‑ignored**.

## Agent actions

The extension exposes these callable actions on the `apng-studio` canvas:

| action            | what it does                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| `get_state`       | Return project settings + per‑frame timing/compositing and total duration.                              |
| `set_settings`    | Update `width`/`height` (only with 0 frames), `loops`, and `hiddenFirst`.                               |
| `add_color_frame` | Append a solid‑color frame; accepts `delayNum`/`delayDen`/`disposeOp`/`blendOp`.                         |
| `set_frame`       | Update one frame (`frameId`) or all (`all: true`): timing via `delayMs`/`fps`/`delayNum`+`delayDen`, plus `disposeOp`/`blendOp`. |
| `clear_frames`    | Remove every frame.                                                                                      |
| `export`          | Assemble and write the `.apng` to disk; returns the absolute path.                                       |

All actions accept an optional `projectId` to target a specific animation.

## How it works

- **`extension.mjs`** — one loopback HTTP server per open canvas instance serves the renderer, JSON state, per‑frame PNGs, the live `/preview.apng`, and mutation endpoints. Server‑Sent Events (`/events`) push a `changed` signal so every open panel and the preview stay in sync.
- **`apng.mjs`** — assembles the APNG chunk stream (`IHDR` / `acTL` / `fcTL` / `IDAT` / `fdAT` / `IEND`) with contiguous sequence numbers, plus a minimal RGBA→PNG encoder.
- **`web/`** — the iframe UI. It talks to its server over plain HTTP; there is no privileged host bridge.

## License

[MIT](./LICENSE)
