# wallshuffle

[한국어](README.ko.md)

Shuffle macOS Aerial wallpapers on **any interval** — every 10 minutes, every 3 hours, whatever you like.

System Settings offers no Aerial shuffle interval shorter than "Every 12 hours".
wallshuffle adds a small background timer that switches to another Aerial you have downloaded, as often as you want.

- **Any interval** from 60 seconds up (`every 10m`, `every 1h`, `every 3h`, …)
- **Follows your choice in System Settings** — "Shuffle All" or a category such as "Shuffle Cityscape"
- **Uses every Aerial you downloaded**, including the ones inside sub-folders (e.g. Cityscape › San Francisco) that Apple's own shuffle skips
- **Unlocks Apple's hidden "Every 5 seconds" shuffle** if you want the built-in shuffle instead of a timer
- **One zsh file** — no sudo, no build step, nothing to install beyond what macOS ships (zsh, jq, plutil, osascript, launchd)
- **Backs up your wallpaper settings** on first use; `wallshuffle restore` puts them back

> Not affiliated with Apple. wallshuffle edits an undocumented settings file, so a macOS update may break it.

## Requirements

- **macOS 27 or later.** Tested on macOS 27.0.1 (Apple Silicon). Earlier versions store Aerials in a different place and are not supported.
- Your wallpaper must be an **Aerial** set to a **shuffle** (System Settings › Wallpaper).
- At least two Aerials downloaded. Aerials only rotate if their video is on disk.

## Install

1. Download [`wallshuffle`](wallshuffle) from this repository (or clone it).
2. In **System Settings › Wallpaper**, pick an Aerial shuffle, e.g. **Shuffle All** or **Shuffle Cityscape**, and download a few Aerials.
3. In Terminal, run it once with the interval you want:

   ```bash
   zsh ~/Downloads/wallshuffle every 1h
   ```

   This copies the script to `~/Library/Application Support/wallshuffle/`, starts the timer, and switches the wallpaper once right away.

### Shorter command (optional)

```bash
echo 'alias wallshuffle="$HOME/Library/Application Support/wallshuffle/wallshuffle"' >> ~/.zshrc
```

Open a new Terminal window, and from then on you can type `wallshuffle every 3h` and so on.
The rest of this README uses the short form.

## Usage

| Command | What it does |
|---|---|
| `wallshuffle status` | Show the current Aerial, the timer, and how many Aerials are in the pool |
| `wallshuffle every 3h` | Switch every 3 hours (`s` / `m` / `h`, minimum 60s). Use `24h` for daily |
| `wallshuffle next` | Switch to another Aerial right now |
| `wallshuffle off` | Remove the timer and go back to Apple's shuffle (every 12 hours) |
| `wallshuffle native 5s` | Use Apple's built-in shuffle at a given frequency instead of the timer (see below) |
| `wallshuffle restore` | Restore the wallpaper settings saved before the first change |
| `wallshuffle lang ko\|en\|auto` | Output language (default: your macOS language) |
| `wallshuffle help` | Help |

Running `every` again simply replaces the interval.

## Which Aerials does it pick?

It follows the shuffle you last picked in **System Settings › Wallpaper**:

- **Shuffle All** → any downloaded Aerial
- **Shuffle Cityscape** (or another category) → downloaded Aerials in that category

To change the pool, just pick another shuffle in System Settings; the next switch follows it (or run `wallshuffle next`).
Newly downloaded Aerials join the pool automatically on the next switch.
It never shows the same Aerial twice in a row, and it skips the Mac and Dynamic (light/dark) wallpapers.

## Apple's built-in frequencies (`native`)

If you'd rather use macOS's own shuffle instead of the timer:

| Value | Frequency |
|---|---|
| `5s` | Every 5 seconds — not in the System Settings menu (Apple hides it behind an internal feature flag) |
| `continuous` | Continuously — moves on whenever the current video ends |
| `12h` `1d` `2d` `1w` `1mo` | Every 12 hours / day / 2 days / week / month |

```bash
wallshuffle native 5s
```

`native` turns the timer off. Apple's own shuffle skips Aerials inside sub-folders.

## Good to know

- **While the timer runs, System Settings shows a single Aerial selected instead of "Shuffle".** That is expected — wallshuffle selects one Aerial at a time.
- **If you pick a single Aerial yourself** in System Settings, the timer leaves it alone until you pick a shuffle again.
- If your wallpaper is not an Aerial, the timer does nothing.
- Each switch restarts `WallpaperAgent`, so the desktop may flicker briefly.
- Time asleep or powered off counts. If the interval passed while the Mac was asleep, it switches within a minute of waking (once, not once per missed interval). Intervals count from the last switch, not from the top of the hour.

## How it works

macOS keeps wallpaper settings in `~/Library/Application Support/com.apple.wallpaper/Store/Index.plist`.
On each tick, wallshuffle:

1. reads Apple's Aerial catalog (`aerials/manifest/entries.json`) and keeps the Aerials in your pool whose video is downloaded,
2. picks one at random (never the current one),
3. writes its ID into `Index.plist` and restarts `WallpaperAgent`, then checks that the change stuck.

The timer is a per-user LaunchAgent (`launchd`) that checks once a minute whether the interval has passed on the clock, so sleep and restarts don't reset it.

### Files

| Path | Purpose |
|---|---|
| `~/Library/Application Support/wallshuffle/wallshuffle` | The installed script the timer runs |
| `~/Library/Application Support/wallshuffle/state` | Pool, last pick, language, interval, time of the last switch |
| `~/Library/Application Support/wallshuffle/error.log` | Errors from timer runs, if any |
| `~/Library/LaunchAgents/local.wallshuffle.plist` | The timer |
| `~/Library/Application Support/com.apple.wallpaper/Store/Index.plist.wallshuffle-backup` | Your original wallpaper settings (used by `restore`) |

## Uninstall

```bash
wallshuffle off
```
```bash
rm -rf "$HOME/Library/Application Support/wallshuffle"
```

`off` removes the timer and returns to Apple's shuffle. The second command deletes the script.
You can keep or delete the backup `Index.plist.wallshuffle-backup`, and remove the `alias wallshuffle=…` line from `~/.zshrc` if you added it.

## See also

Other projects that work with macOS Aerials:

- [apple-aerial-controller](https://github.com/JoeyJia02/apple-aerial-controller) — menu bar app with playlists, timed rotation and desktop video playback (Swift, build from source)
- [aerial-rotate](https://github.com/TylerJamesNz/aerial-rotate) — keeps only one Aerial on disk and rotates it daily (needs sudo)
- [macos-wallpaper-rotator](https://github.com/Icosahedral-Dice/macos-wallpaper-rotator) — switches Aerials by sunrise and sunset (Python)
- [tahoe-aerial-shuffle](https://github.com/adele-with-a-b/tahoe-aerial-shuffle) — lock screen Aerial screen saver that changes every minute

## License

[MIT](LICENSE). Apple's Aerial videos are not part of this project and belong to their owners.
