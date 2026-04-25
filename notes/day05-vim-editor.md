# Day 5 — vi / vim Editor

## Why CLI Editors?

GUI editors (Gedit, gvim) work on desktop environments. But real servers usually have CLI only. You need `vi`/`vim` — you can't always open a GUI.

| Windows | GUI Linux | CLI Linux |
|---------|-----------|-----------|
| Notepad | Gedit | `vi` (RHEL 6) |
| Wordpad | gvim | `vim` (RHEL 7+) |

---

## vi vs vim

| Feature | vi | vim |
|---------|----|----|
| Full name | Visual Editor | Visual Editor Improved |
| Display | Black & white | Colorful (view mode); black & white when editing |
| Typo detection | Hard to spot | Easier — syntax highlighting helps |
| File awareness | Treats everything as plain text | Recognizes file type by extension |

**Prefer `vim`** — it's better and what RHEL 7+ uses by default.

---

## Switching Between CLI and GUI (RHEL)

```
RHEL 7:
Ctrl+Alt+F2 → GUI to CLI
Ctrl+Alt+F1 → CLI to GUI

RHEL 8:
Ctrl+Alt+F3 → GUI to CLI
Ctrl+Alt+F2 → CLI to GUI
```

Use `tty` to find which terminal you're currently on.

---

## vim Modes

vim has 3 modes — switch deliberately:

| Mode | How to Enter | Purpose |
|------|-------------|---------|
| **Command mode** | Default — press `Esc` to return here | Navigate, delete, copy, paste |
| **Insert mode** | `i`, `a`, `A`, `I`, `o`, `O` | Type and edit text |
| **Visual mode** | `v` | Select text (read-oriented) |
| **Extended command mode** | `:` from command mode | Save, quit, search, replace |

- `Esc` always returns you to Command mode
- `Backspace` also returns from Visual mode

---

## Insert Mode — Entry Options

| Key | Effect |
|-----|--------|
| `i` | Insert before cursor |
| `a` | Append after cursor (next character) |
| `A` | Append at end of line |
| `I` | Insert at beginning of line |
| `o` | Open new line **below** cursor |
| `O` | Open new line **above** cursor |

---

## Command Mode — Navigation & Editing

| Key | Action |
|-----|--------|
| `gg` | Go to start of file |
| `G` | Go to end of file |
| `w` | Move forward word by word |
| `nw` (e.g. `2w`, `4w`) | Move forward n words |
| `b` | Move backward word by word |
| `nb` (e.g. `2b`, `4b`) | Move backward n words |
| `dd` | Delete (cut) current line |
| `ndd` (e.g. `4dd`) | Delete n lines |
| `yy` | Copy (yank) current line |
| `nyy` (e.g. `4yy`) | Copy n lines |
| `p` | Paste (do NOT use Ctrl+C/V — they don't work in vim) |
| `u` | Undo |
| `U` | Undo entire line |
| `Ctrl+r` | Redo |
| `x` | Delete character under cursor |

---

## Extended Command Mode (`:`)

Press `:` from Command mode:

| Command | Action |
|---------|--------|
| `:w` | Save |
| `:q` | Quit |
| `:wq` | Save and quit |
| `:wq!` | Force save and quit |
| `:q!` | Quit without saving |
| `:x` | Save and quit (same as `:wq`) |
| `:se nu` | Show line numbers |
| `:se nonu` | Hide line numbers |
| `:20` | Jump to line 20 |
| `/word` | Search for "word" in file |
| `:s/Linux/Windows` | Replace first occurrence on current line |
| `:%s/Linux/Windows` | Replace all occurrences in entire file |
| `:X` | Encrypt the file (prompts for key) |

---

## Learning Resource

```bash
vimtutor    # built-in interactive vim tutorial — run this to practice
```

---

## Key Takeaway

In server environments, `vim` is your primary editor. Learn the modes cold: Command → Insert (`i`) → back to Command (`Esc`) → Extended (`:wq`). That's the core loop for 90% of editing tasks.
