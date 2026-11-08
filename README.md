# 🌿 FocusFlow

**An offline-first daily planner built for an AuDHD brain.**
Low friction, low pressure, low glare. Nothing expires, nothing shouts, and unfinished things simply wait for you.

FocusFlow is a single-file desktop app written in Python (Tkinter + SQLite). There is nothing to install beyond Python itself, and your data stays in one plain database file that you own.

---

## Why it exists

Most planners are built around deadlines, streaks and guilt. FocusFlow is built around the opposite ideas:

| Principle | What it means in the app |
|---|---|
| **Zero-shame mechanics** | No overdue badges, no red alerts, no broken streaks. Unfinished tasks stay in your pool. Due dates are shown calmly, never as warnings. |
| **Low cognitive load** | One capture bar for everything. Type fast, tag inline, sort it out later. |
| **Sensory-friendly UI** | Dark, muted palette. No pure black or pure white, no animation, no sounds. Contrast meets WCAG AA. |
| **Context isolation** | Hyperfocus work (apps, side hustles, learning) is kept apart from routine upkeep (chores, money, admin). |
| **Energy-aware** | Tell the app how much energy you have today, and it adjusts how much it shows you. |

---

## Features

### Today
- **Energy selector:** `Low (Spoons Low)` · `Medium` · `High (Hyperfocus)`
  - *Low* shows only gentle maintenance tasks (max 3).
  - *Medium* shows up to 6.
  - *High* puts hyperfocus work first.
- **Anchors** (time-fixed commitments) are kept separate from the **Flexible Pool**.
- **Soft timer:** a shrinking colour bar (10 / 25 / 45 min). No ticking clock, no alarm sound.
- Hidden tasks are never counted against you; the app just says they are "resting quietly out of sight".

### Quick capture (brain dump)
Type into the capture bar and press Enter:

```
Reply to supplier emails #finance
Build login screen #app-hobby !important
Call dentist @14:30
Draft proposal #chromabridge !key
```

| Syntax | Meaning |
|---|---|
| `#tag` | Category (`daily`, `finance`, `app-hobby`, `side-hustle`, `tech-learning`, or an OSP sub-category such as `#chromabridge`) |
| `@HH:MM` | Makes it an anchor at that time |
| `!important` / `!key` | Priority (★) |

Global hotkey: **Win + Alt + Space** (requires the optional `pynput` package, see below). Inside the app, **Ctrl + Alt + Space** jumps to the capture bar.

### Task controls
Every task has **Done**, **Break down**, **Move ▾**, **Details**, and **×**.

- **Break down** creates three small steps. Each step has its own Done/Undo, edit (✎) and delete (×).
- **Details** edits title, category, priority, due date, anchor time and dependency.
- **Dependencies:** a task that is waiting on another is dimmed and sorted to the bottom, then returns to normal once the other task is done.
- **Double-click** a task title for a quick rename.

### Domain tabs
| Tab | Purpose |
|---|---|
| **OSP** | Sub-categories with their own task lists. Ships with *ChromaBridge* and *Nov5 Initiative*; add, rename or remove your own. |
| **App Hobbies** | Projects with progress bars and build logs / code links. |
| **Side Hustles** | Clients or projects with milestones. |
| **Life Admin** | Soft repeating routines ("roughly every 3 days"). Shows *around now*, never *overdue*. |
| **Finance** | Payday checklist, finance tasks, and the **Impulse Vault** (48-hour cooling-off timer before you decide on a purchase). |
| **Reading List** | Christian · Business · Personal Growth · Technology · Other. Mark books Read/Unread and move them between categories. |
| **Skills Development** | Skills with a level (Curious → Learning → Practicing → Confident) and a **+15 min** practice log. |

Use the **View** menu to show or hide any optional tab. Hiding **OSP** or **Finance** also removes their tasks from Today. The setting is saved in the database, so it follows you between computers.

### Editing everything
Every item has visible **✎** (edit) and **×** (delete) buttons. Edits open a dark, themed dialog: **Enter** saves, **Esc** cancels.

---

## Requirements

- **Python 3.9 or newer** (uses only the standard library: `tkinter`, `sqlite3`)
- Primarily developed for **Windows 10/11**. It should also run on macOS and Linux, but those have had less testing.
  - On some Linux distributions you may need: `sudo apt install python3-tk`

Optional:
- `pip install pynput` enables the global **Win + Alt + Space** quick-capture shortcut.

## Quick start

```bash
git clone https://github.com/<your-username>/focusflow.git
cd focusflow
python focusflow.py
```

On first run FocusFlow creates its database at `~/.focusflow/focusflow.db`.

---

## Configuration

The database location is resolved in this order:

1. The `FOCUSFLOW_DB` environment variable
2. `~/.focusflow/config.json` (`{"db": "path/to/focusflow.db"}`)
3. The default `~/.focusflow/focusflow.db`

You can inspect and change the location from **Help → Config → Change database location…**

---

## Using the same data on several computers (Google Drive, OneDrive, Dropbox)

FocusFlow keeps everything in one SQLite file, so you can place that file in a synced folder.

**Example (Google Drive mounted as `H:`):**

Command Prompt:
```bat
set FOCUSFLOW_DB=H:\My Drive\FocusFlow\focusflow.db
```

PowerShell:
```powershell
$env:FOCUSFLOW_DB="H:\My Drive\FocusFlow\focusflow.db"
```

Make it permanent on Windows: press the Windows key → *Edit the system environment variables* → *Environment Variables…* → under *User variables* choose *New…* → name `FOCUSFLOW_DB`, value `H:\My Drive\FocusFlow\focusflow.db`. Restart any open terminals or your IDE.

(The same instructions are available inside the app under **Help → Google Drive setup**.)

### Sync and conflict protocol

Sync tools copy files, they do not merge databases, so FocusFlow follows a simple one-at-a-time protocol:

1. **Keep FocusFlow open on one device at a time.** A lock file next to the database warns you if another device seems to have it open. A gentle, silent reminder appears every 10 minutes to close the copy you are not using.
2. **Before switching PC:** *Sync → Force sync now*, wait for your sync client to show "up to date", then close the app.
3. **On the next PC:** open FocusFlow. Use *Sync → Pull latest* if the sync client was still catching up.
4. **Conflicts:** if the sync client keeps a second copy (for example `focusflow (1).db`), FocusFlow detects it and offers:
   - **Merge (safe):** adds anything missing, deletes nothing.
   - **Replace current:** use that copy instead.
   - **Archive:** set it aside.
5. **Safety net:** a backup is taken before any merge or replace (`backups/`, last 20 kept). Handled copies move to `conflicts_resolved/`.

The **Sync** menu also has **Sync status**, which shows when the file was last saved, who holds the lock, and any conflict copies.

---

## Data and privacy

- Everything is stored locally in a single SQLite file. No account, no cloud service, no telemetry, no network calls.
- Tables: `tasks`, `steps`, and a flexible `items` table that holds hobbies, hustles, routines, books, skills, the payday list, the vault and your view preferences.
- The database uses `journal_mode=DELETE`, so it is always one plain file, which is safer for file-sync tools.

---

## Design notes

The look follows the **ChromaBridge brand palette**, adapted for low-glare dark mode:

| Role | Colour |
|---|---|
| Cards | Deep Industrial Blue `#003B5C` |
| Accent, active tab, progress | Bridge Teal `#00A8B5` |
| Hover and lines | Precision Indigo `#1B4980` |
| Cursor and checkmark only | Digital Spectrum Cyan `#00E5FF` |
| Text | Off-white `#E6EEF4` (never pure white or black) |

Contrast ratios are at or above 4.5:1 (WCAG AA). Fonts: Inter, then Plus Jakarta Sans, then Segoe UI, depending on what is installed.

---

## Known limitations

- Windows may reserve some Win-key combinations, so the global hotkey may not work on every system.
- There is no system tray icon yet.
- The Impulse Vault countdown updates when the screen refreshes (any action), not continuously.
- The one-at-a-time lock depends on how quickly your sync client uploads, so it is a safety net rather than a guarantee.
- Merge matches tasks by title and category, so a task renamed on both computers may appear twice after a merge.
- Windows draws its own menu bar and system pop-ups, so those may keep a light system style.

## Roadmap

- [ ] Tauri v2 desktop build with a true global shortcut and system tray
- [ ] Optional local or API-based AI task breakdown
- [ ] Sort and filter by due date or priority across all tasks
- [ ] Live-updating vault countdown
- [ ] Export / import as a portable file

## Optional: web prototype

An earlier single-file HTML prototype exists. Its **Backup & move to another device** code can be pasted into the Python app via **File → Import web backup code…** to bring your data across.

---

## Contributing

Issues and pull requests are welcome. Please keep the core philosophy intact: **no overdue badges, no alarm colours, no sounds, no streak penalties.** If a change adds pressure, it probably does not belong here.

## License

Add your preferred license here (for example MIT) and include a `LICENSE` file.
