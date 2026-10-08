# local-service-green-package

> Package local always-on services (new-api.exe, gateway/service executables, etc.) into "unzip and double-click" portable bundles you can hand to a friend or copy to another machine — complete with a ready-to-use AI deployment prompt.

[中文 README](README.md)

## Introduction

This is an AI Skill repository for WorkBuddy / Claude Code. It solves a very specific problem: you have a useful local service running on your own machine (for example, an AI gateway like new-api), and you want to share it with a friend or move it to another computer — but the recipient has no development environment, can't install dependencies, and can't troubleshoot port conflicts.

The skill guides the AI to package any self-hosted tool of the form "one exe + one config/data file + needs to stay resident in the background" into a standardized portable zip: the recipient unzips it, double-clicks the launcher, and everything just works. The bundle also ships a self-contained `AI Deployment Prompt.md` — the recipient drops it into their own AI assistant, and the AI walks them through deployment step by step.

Who it's for:

- Indie developers who want to share a local AI gateway or tool with friends
- People who need to copy a self-hosted service to multiple Windows machines with zero installation
- Power users who let an AI assistant automate packaging and deployment

## Trigger Phrases

"Package this for a friend", "portable version", "no-install bundle", "unzip and run", "package XXX for someone else", "copy to another computer", "one-click deployment prompt", "set someone up with a copy".

## Features

- **Standardized bundle layout**: defines a uniform structure of `Launcher.exe + ServiceMain.exe + config/data files + UserGuide.txt + AI Deployment Prompt.md`, wrapped in a top-level folder so files don't scatter on extraction.
- **Seven-step packaging workflow**: clarify requirements → produce a clean data snapshot → write the launcher → PyInstaller build → dual-scenario final verification → write both documents → zip it and test from a fresh extraction. Every step has concrete instructions.
- **Clean data snapshots**: uses sqlite3's `backup()` API (handles WAL automatically) to snapshot databases, then wipes runtime data while keeping configuration — on a copy, never touching the user's live database.
- **Smart launcher**: a tkinter GUI panel plus a CLI mode. Its job: "pick a free port → spawn the backend → wait for readiness → open the browser."
- **8 hard-won rules**: covers real pitfalls such as the Windows port-probing trap, missing tkinter in managed Python builds, sandbox child-process reaping, process identification by full path, bcrypt password resets, document encoding, and registry-based autostart.
- **Final acceptance checklist**: 9 explicit checkboxes so "it worked once on the happy path" never counts as done.

## How It Works / Tech Stack

This is a documentation-only skill (no code). The methodology lives in `SKILL.md` and touches on:

- **PyInstaller**: `--onefile --noconsole` to build the Python launcher into a single GUI exe.
- **tkinter**: the launcher GUI (the build Python *must* ship tkinter, or you get "double-click and nothing happens").
- **sqlite3 backup API**: safe database snapshots including WAL.
- **netstat -ano -p TCP**: the only reliable way to detect port usage on Windows (Windows allows duplicate binds, so probing with `bind()` gives false "free" results).
- **CreateToolhelp32Snapshot + QueryFullProcessImageNameW**: identify processes by full path — only ever kill the backend in your own directory, never `taskkill /IM` by name and risk killing another copy the user is running.
- **bcrypt**: reset account passwords for the recipient (hashes can't be reversed — only reset).
- **HKCU Run registry key**: autostart without admin rights.
- **UTF-8 BOM + CRLF**: keeps `UserGuide.txt` readable in legacy Notepad.

### Key Technical Rule (Example)

Windows port probing: Windows allows duplicate binds — even if `0.0.0.0:3000` is already listening, binding the same port again succeeds. The correct approach is parsing the LISTENING lines of `netstat -ano -p TCP` (with a 2-second cache), and "startup succeeded" only counts when all three hold: this directory's backend process is alive + that process is listening on the port + the API responds.

## Installation & Usage

This is an AI skill, not standalone software. To install, copy this repository's folder into your skills directory, keeping the folder name `local-service-green-package`:

- WorkBuddy / CodeBuddy: `~/.workbuddy/skills/local-service-green-package/`
- Claude Code: `~/.claude/skills/local-service-green-package/`

Restart your session, then use a trigger phrase (e.g. "package this service as a portable bundle for my friend") and the AI will follow the standard workflow in SKILL.md.

## Project Structure

```
local-service-green-package/
├── SKILL.md      ← Skill body: bundle structure, 7-step workflow, 8 rules, acceptance checklist
├── README.md     ← Chinese README (detailed)
├── README_EN.md  ← This file (English)
└── .gitignore
```

## Notes & Limitations

- The methodology primarily targets **Windows** (registry autostart, netstat parsing, PyInstaller exe packaging are all Windows-specific).
- When building the GUI launcher, you must use a Python that ships tkinter (the system Python on Windows usually does; managed/isolated Pythons often don't).
- In sandboxed shells, child processes are reaped when a command finishes — "start → verify → stop" must happen within a single command or script.
- Always operate on copies of databases; never touch the user's live service or data.
- Final verification must be done by extracting the finished zip into a fresh directory and running it there — verifying in the original directory is not enough.

## License

MIT

## Author

sheen945
