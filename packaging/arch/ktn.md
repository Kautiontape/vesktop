---
package: vesktop-ktn
summary: Vesktop with real push-to-talk on sway and other compositors without a GlobalShortcuts portal
upstream: https://github.com/Vencord/Vesktop
pinned: upstream main @ 0ead609 (Electron 44), until a release includes it
retire_when: An upstream Vesktop release ships the vesktop.sock IPC socket (PR #1238); checked by hand, and expect the sync to conflict when it lands
---
## What it is

[Vesktop](https://github.com/Vencord/Vesktop), the Discord client with Vencord built in, plus
Vendicated's draft PR #1238. That PR adds global shortcuts and a local IPC socket,
`$XDG_RUNTIME_DIR/vesktop.sock`.

## Why it exists

Wayland doesn't let unfocused apps see your keys. The only standard way around that is the
GlobalShortcuts portal, and sway's portal doesn't implement it. Even where the portal exists,
Electron only reports key presses, never releases, so it can't do hold-to-talk.

The socket solves it: your compositor binds a key or mouse button and sends **Start** on press and
**Stop** on release. Those drive Discord's own push-to-talk, with its indicator, its sounds and its
release delay, while other apps keep a live mic.

```sh
echo run-shortcut:pushToTalkNormalStart | nc -N -U "$XDG_RUNTIME_DIR/vesktop.sock"
echo run-shortcut:pushToTalkNormalStop  | nc -N -U "$XDG_RUNTIME_DIR/vesktop.sock"
```

In sway, bind them to the same key with `bindsym --no-repeat` and `bindsym --release`. Set
Vesktop's Input Mode to Push to Talk.

## How it differs from upstream

- PR #1238 squashed onto upstream, with its keybinds dialog ported to Vencord's current modal API.
  Other actions on the socket include `toggleMute`, `toggleDeafen`, `mute`/`unmute` and the
  priority push-to-talk variants.
- **Pinned ahead of release**: built from upstream `main` (Electron 44) rather than the last tag,
  `v1.6.7`. It returns to release tags by itself once a release includes that commit.
- Installs to `/opt/vesktop` and `/usr/bin/vesktop` like the AUR `vesktop-bin`, and replaces it
  when installed by name. There's no self-updater; updates come through pacman.
- If push-to-talk ever stops working after a Discord update, run `vesktop --repair` to refresh
  Vencord. A broken Vencord can't update itself.
