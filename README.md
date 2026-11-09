# custom-karabiner-windows

A [Karabiner-Elements](https://karabiner-elements.pqrs.org/) configuration that makes a Mac behave like Windows/Linux for keyboard shortcuts, plus a small set of VS Code keybindings that keep the integrated terminal usable with it. Assumes a **US layout**, a Windows keyboard when docked and an Apple keyboard on its own.

| File | What it is |
| --- | --- |
| `karabiner.json` | The complete Karabiner-Elements config (one profile, 38 rules). |
| `vscode-terminal-keybindings.jsonc` | Entries to paste into VS Code's `keybindings.json`. |

## 1. Karabiner config (`karabiner.json`)

### Install

1. Install Karabiner-Elements.
2. **Back up your current config**: `cp ~/.config/karabiner/karabiner.json ~/karabiner.backup.json`. This file replaces the whole thing, including any other profiles, devices and simple modifications.
3. Copy this file over it: `cp karabiner.json ~/.config/karabiner/karabiner.json`. Karabiner reloads on save.
4. Grant Karabiner the Input Monitoring / Accessibility permissions when macOS asks.

### Make an Apple keyboard match a Windows layout

A Windows keyboard sits Ctrl, Win, Alt; an Apple one sits Ctrl, Option, Cmd. The rules assume the Windows order, so swap the keys on the Apple keyboard only (your Windows keyboard needs nothing):

Karabiner-Elements → **Devices** → select the Apple keyboard → **Modifier keys**:

| From | To |
| --- | --- |
| `left_option` | `left_command` |
| `left_command` | `left_option` |
| `right_command` | `right_option` |

This is done in the GUI because it needs the keyboard's product ID and has to apply before the rules in this file. On a MacBook's built-in keyboard, Ctrl is the second key from the left; swapping `fn` and `left_control` is possible but can interfere with the F-key row and media keys.

### How it works

Ctrl acts as Cmd for application shortcuts (Ctrl+C/V/X/Z/Y/A/S/N/F/W/T/…), Alt+Tab switches apps, Alt+F4 quits, Home/End/Print Screen work like on a PC, and the Win key keeps its Windows jobs (Win+E Finder, Win+L lock, Win+Shift+S screenshot, …).

Keys pass through untouched in these apps:

- **RustDesk** (`com.carriez.rustdesk`): every rule is disabled, so shortcuts go to the remote machine.
- Other remote-desktop clients and virtual machines: the PC-style shortcuts are disabled.
- **Terminals** (Terminal, iTerm2, Hyper, Alacritty, kitty, Ghostty, WezTerm, Tabby, Warp) and Emacs: the Ctrl→Cmd shortcuts are disabled, so Ctrl+C still cancels a process.

Rules marked `[Mix]` were added on top of the standard PC-style set:

- **Terminals:** Linux-style Ctrl+Shift+C / V (copy / paste) and Ctrl+Shift+N / W / F.
- **Finder:** F2 rename, Delete = Move to Trash, Alt+Enter = Get Info, Alt+Up / Left / Right = up / back / forward, Ctrl+L = Go to Folder.
- **Win key:** Win+Tab = Mission Control, Win+D = Show Desktop (needs macOS's default F11 Show Desktop shortcut).
- **Apple keyboards only:** Fn+Left / Right act as Home / End (Fn+Ctrl = start / end of document).

### Customising

- **Add an app to the exclusions** (for example another terminal): get its bundle ID with `osascript -e 'id of app "AppName"'`, then add `^bundle\\.id$` to the `frontmost_application_unless` lists. The terminal list appears many times, so search for `ghostty` to find every copy.
- **Keep rule order in mind.** Karabiner uses the first rule that matches, so two rules on the same key will shadow each other.
- **Don't bulk find-and-replace `"type": "basic"`** to add a condition. Many manipulators already have `conditions`, and the replace creates a second `conditions` key that silently overrides the first.

## 2. VS Code terminal keybindings (`vscode-terminal-keybindings.jsonc`)

Karabiner can't tell VS Code's editor from its integrated terminal, so Ctrl+C becomes Cmd+C in both and you can no longer cancel a process. These keybindings turn the Cmd chords back into real control characters, but **only while the terminal has focus**; the editor is unaffected.

### Install

1. In VS Code press **Cmd+Shift+P** → *Preferences: Open Keyboard Shortcuts (JSON)*.
2. Paste the entries from `vscode-terminal-keybindings.jsonc` **inside the existing `[ ... ]`**. Don't replace the whole file.

### Result in the terminal

- **Ctrl+C** copies if text is selected, otherwise sends SIGINT.
- **Ctrl+V** pastes. **Ctrl+Shift+C / Ctrl+Shift+V** copy / paste.
- **Ctrl+D / Z / A / B / F / N / P / T / U / W** act as normal shell and TUI control keys (EOF, suspend, line start, tmux prefix, history, swap characters, kill line, delete word).

Trade-offs: Ctrl+W deletes a word instead of closing the tab while the terminal is focused, and Ctrl+A doesn't select all there. To cover another key, add one line in the same pattern.

## Caveats

- Developed and checked by static analysis (JSON validity and a rule-shadowing check); not verified on every macOS version.
- The `[Mix]` Fn+arrow rules assume Karabiner reports Fn+Arrow as an arrow key with the `fn` modifier.
