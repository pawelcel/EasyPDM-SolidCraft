# Working on EasyPDM for SolidCraft

Conventions that are easy to miss. The EasyPDM server and web app live in
[pawelcel/EasyPDM](https://github.com/pawelcel/EasyPDM), with their own CLAUDE.md; the FreeCAD
and SolidWorks integrations in pawelcel/EasyPDM-FreeCAD and pawelcel/EasyPDM-SolidWorks.

## Layout

- `plugin/` — the SolidCraft plugin, Rust compiled to a WebAssembly component
  (`craft:plugin@0.1.0`, `wit/plugin.wit`). Plugins have no network: its menu entries only
  leave requests in the granted *outbox* folder. `plugin/build.sh` builds `easypdm.zip` (needs
  the `wasm32-unknown-unknown` target and `wasm-tools`); the zip is not committed.
- `companion/` — the EasyPDM SolidCraft companion (.NET 10, single-file, self-contained in
  releases): logs in to the server, serves the outbox requests, drives SolidCraft through
  `solidcraft-cli` and its local control channel. The README's "Tested so far" records what has
  been seen working in SolidCraft itself.

## SolidCraft gotchas

- Needs SolidCraft 0.43 or later: 0.42 cannot grant a plugin a command group in its window.
- The plugin menu slot's `name` is the command's title in the Plugins menu; 0.43 moves a slot
  named after the File or Help menu into that menu. Name slots after the command.
- SolidCraft voids a whole plugin call when one of its reads fails, even a handled one: never
  read a file that may not exist.
- The Flatpak build on the NVIDIA 470 driver under Wayland needs `WGPU_BACKEND=gl`.

## Testing

No automated test of SolidCraft behaviour. CI builds the companion (warnings are errors) and
the plugin, and checks that their versions agree. Check a change in SolidCraft itself; the
user runs it, and never install over the user's SolidCraft or open windows on their desktop.

## Commits, changelog, releases

- Commit messages in English, explaining why the change is right.
- CHANGELOG entries on one line each: the release workflow cuts the section into the GitHub
  release, which renders single newlines as line breaks.
- A release is a `vX.Y.Z` tag equal to the version in `plugin/plugin.json`, `plugin/Cargo.toml`
  and `companion/EasyPDM.Companion.csproj` (the workflow refuses otherwise). It opens a draft
  with `EasyPDM_SolidCraft_Companion_<win-x64|linux-x64>_vX.Y.Z.zip`; publishing stays manual.
