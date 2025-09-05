# 📘 VI/VIM Commands Dictionary

This practical guide lists the most useful commands for the `vi`/`vim` editor, organized by categories to facilitate your use during labs.

---

## 📝 Edit Mode

| Command | Action                      |
| ------- | --------------------------- |
| `i`     | Insert before cursor        |
| `I`     | Insert at beginning of line |
| `a`     | Insert after cursor         |
| `A`     | Insert at end of line       |
| `o`     | New line below              |
| `O`     | New line above              |
| `Esc`   | Exit insert mode            |

---

## 💾 Save and Exit

| Command | Action             |
| ------- | ------------------ |
| `:w`    | Save               |
| `:q`    | Exit               |
| `:wq`   | Save and exit      |
| `:q!`   | Exit without save  |
| `ZZ`    | Save and exit (shortcut) |

---

## 🔍 Navigation

| Command  | Action                    |
| -------- | ------------------------- |
| `h`, `l` | Move cursor ← →           |
| `j`, `k` | Move cursor ↓ ↑           |
| `w`      | Move forward one word     |
| `b`      | Move back one word        |
| `0`      | Beginning of line         |
| `^`      | First character of line   |
| `$`      | End of line               |
| `G`      | Go to last line           |
| `gg`     | Go to first line          |
| `:n`     | Go to line `n`            |

---

## ✂️ Copy, Paste and Delete

| Command  | Action            |
| -------- | ----------------- |
| `yy`     | Copy line         |
| `dd`     | Delete line       |
| `p`      | Paste below       |
| `P`      | Paste above       |
| `x`      | Delete character  |
| `u`      | Undo              |
| `Ctrl+r` | Redo              |

---

## 🔄 Search and Replace

| Command            | Action                                        |
| ------------------ | --------------------------------------------- |
| `/text`            | Search "text" forward                         |
| `?text`            | Search "text" backward                        |
| `n`, `N`           | Next / previous occurrence                    |
| `:%s/old/new/g`    | Replace "old" with "new" in entire file      |
| `:s/old/new/g`     | Replace in current line                       |

---

With these commands, you can already navigate, edit and save files in `vi/vim` during labs efficiently. If you want to go further, I recommend exploring visual modes (`v`, `V`, `Ctrl+v`) and macros.
