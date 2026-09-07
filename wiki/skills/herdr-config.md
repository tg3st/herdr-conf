---
type: Skill
id: "skill:herdr-config"
title: herdr-config
description: Procedural Oh-My-Pi skill wrapped as an OKF concept.
tags: [skill, procedural-knowledge, omp]
status: stable
generated:
  by: omp-framework/0.2
  at: 2026-08-15T00:00:00Z
sources:
  - id: "skill:herdr-config"
    resource: .omp/skills/herdr-config/SKILL.md
    title: herdr-config SKILL.md
---

---
name: herdr-config
description: Edits the Herdr (terminal workspace manager) config.toml safely and cheaply — keybindings, UI, theme, terminal, update settings. Use when the user asks to modify herdr/herder config, change herdr keybindings or prefix actions, or when the working directory is ~/.config/herdr and the task touches config.toml. Do NOT use for controlling panes/agents/workspaces from inside a Herdr pane (use the herdr skill), for herdr server crash debugging, or for herdr CLI/socket API automation.
---

# Herdr Config

Edit Herdr's `config.toml` without re-deriving its schema: never invent keys — dump the authoritative commented default config, mirror its exact key names, validate, then live-reload the running server.

## When to Use

- "Change the herdr keybinding so prefix+X does Y"
- "Modify my herder config" / "edit herdr config.toml"
- Working directory is `~/.config/herdr` and the task touches `config.toml` (keybindings, sidebar, theme, shell, sounds, update channel)

## When NOT to Use

- Driving Herdr from inside a Herdr pane (split panes, run commands, read output) — that is the `herdr` skill / `herdr` CLI
- Debugging a crashed herdr server or socket API failures
- Editing anything under `~/.config/herdr` that is not config (logs, session.json)

## Instructions

1. **Locate the config.** `~/.config/herdr/config.toml` on macOS/Linux (`%APPDATA%\herdr\config.toml` on Windows). It may not exist yet — Herdr runs fine without one; create it with only the keys being changed.
2. **Dump the authoritative defaults.** Run `herdr --default-config`. It prints the full commented config with every valid key and its default. Grep the relevant section instead of reading it whole:
   - Keybindings → `[keys]` (plus `[[keys.command]]`, `[keys.indexed]`)
   - Sidebar/UI → `[ui]` (subsections `[ui.sidebar]`, `[ui.sound]`)
   - Shell/CWD → `[terminal]` (`shell_mode`, `new_cwd`)
   - Update channel → `[update]`
3. **Mirror exact key names and value syntax.** Binding values are strings like `"prefix+w"` (requires prefix) or `"ctrl+alt+k"` (direct, terminal mode). Navigate-mode locals (`navigate_workspace_*`, `navigate_pane_*`, `navigate_resize_*`) MUST NOT use `prefix+`, `esc`, `enter`, `tab`, or `1..9`. Setting a key replaces its default (arrows lose their binding when rebound to `j`/`k`).
4. **Edit minimally.** Only uncomment/override the keys being changed; leave the shipped file otherwise untouched so future `herdr` versions stay compatible.
5. **Validate.** Run `herdr config check` — must print `config: ok`.
6. **Apply live.** Run `herdr server reload-config`; require `"status":"applied"` and empty `diagnostics` in the JSON reply. If no server is running, the config applies on next launch.
7. **Verify behavior.** Confirm against the actual surface (e.g. the rebound key works in a Herdr client) before reporting done.

## CLI

- `herdr --default-config` — full commented default config; the source of truth for key names. Do NOT invent config keys.
- `herdr config check` — validate config.toml, print diagnostics.
- `herdr server reload-config` — apply edits to the running server without restarting.
- `herdr config reset-keys` — back up config.toml and remove custom keybindings (rollback path).
- `herdr --version` — e.g. `0.8.2`. Brew/managed installs update via the package manager, not `herdr update`.

## Reference

- Config reference: https://herdr.dev/docs/configuration/
- Keyboard guide (safe chords, prefix vs direct): https://herdr.dev/docs/keyboard/
- Agent guide: https://herdr.dev/agent-guide.md

## Example

Rebind workspace-picker navigation to j/k (arrows replaced):

```toml
[keys]
navigate_workspace_up = "k"
navigate_workspace_down = "j"
```

Then `herdr config check && herdr server reload-config` → `config: ok`, `"status":"applied"`.

## Related

- `skill://herdr` — operating Herdr from inside a pane (may not be installed; check before referencing)
