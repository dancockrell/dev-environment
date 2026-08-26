# Development environment

Every tool this machine's work depends on, with versions, locations, and how
to reinstall. Verified against the live machine on 26 August 2026.

**Why this exists:** during a cleanup, `Ruby4Lich5` was removed as apparent
game clutter. It is a hard dependency of `dr-companion`. Nothing on the
machine recorded that. This file is that record.

---

## Core runtimes

| Tool | Version | Location | Install |
|---|---|---|---|
| Node.js | 24.19.0 | `C:\Program Files\nodejs` | `winget install OpenJS.NodeJS.LTS` |
| npm | 11.17.0 | bundled with Node | — |
| Python | 3.13.15 | `%LOCALAPPDATA%\Programs\Python\Python313` | `winget install Python.Python.3.13` |
| Rust | 1.98.0 | `%USERPROFILE%\.cargo` | rustup.rs |
| cargo | 1.98.0 | `%USERPROFILE%\.cargo\bin` | with rustup |
| .NET | 8.0.302 | system | Visual Studio / SDK installer |
| Git | 2.55.0 | `C:\Program Files\Git` | git-scm.com |

**Python trap:** `python` on PATH may resolve to the Microsoft Store stub at
`%LOCALAPPDATA%\Microsoft\WindowsApps\python` and fail with *permission
denied*. Use the full Python313 path, or disable the App Execution Alias in
Settings → Apps → Advanced app settings.

**Rust trap:** `~/.cargo/bin` is not always on PATH in Git Bash. Tauri builds
need it.

---

## Command-line tools

| Tool | Version | Purpose | Install |
|---|---|---|---|
| GitHub CLI | 2.98.0 | repo creation, API queries | `winget install GitHub.cli` |
| uv | 0.12.5 | Python envs and packages | `winget install astral-sh.uv` |
| ffmpeg | 9.0 | audio/video encoding, frame sequences | `winget install Gyan.FFmpeg` |
| pandoc | 3.10.2 | document conversion (docx/pptx to html/md) | `winget install JohnMacFarlane.Pandoc` |
| gitleaks | latest | pre-commit secret scanning | `winget install Gitleaks.Gitleaks` |
| osv-scanner | latest | dependency CVE scanning | `winget install Google.OSVScanner` |
| butler | — | itch.io publishing | `C:\Users\Admin\bin\butler.exe` |

**winget trap:** installed binaries do not resolve until a *new* terminal is
opened. They live under
`%LOCALAPPDATA%\Microsoft\WinGet\Packages\<Publisher.Name>_<hash>\`.

`gitleaks` also runs automatically as a pre-commit hook in every repo, via
`git config --global init.templateDir C:\Users\Admin\.git-templates`.

---

## Projects and their dependencies

| Project | Stack | Manifest |
|---|---|---|
| `dr-companion` | Tauri (Rust + web) | `package.json`, `src-tauri/Cargo.toml` |
| `react-lab` | Vite + React 19 | `package.json` |
| `visual-rig` | Python | `pyproject.toml` |
| `whisper` | Python venv | see below |
| `tts` | Python venv | see below |
| `chatterbox` | Python venv | see below |

### dr-companion — external dependencies

This is the one that bit us. `dr-companion` automates **Lich**, which runs on
**Ruby**. None of it is captured in `package.json` or `Cargo.toml`.

| Dependency | Location | Source |
|---|---|---|
| Ruby4Lich5 | `C:\Ruby4Lich5` | [lich-5 releases](https://github.com/elanthia-online/lich-5/releases) → `Ruby4Lich5.exe` |
| Lich5 scripts | `C:\Ruby4Lich5\Lich5` | same installer |
| Genie client | `C:\Genie4` | Simutronics |

dr-companion ships its own dependency installer, so a clean machine is a valid
test of it — but do not remove these assuming they are game clutter.

---

## Python environments

Created with `uv venv`, one per tool. All Python 3.13.15.

| Env | Key packages | Purpose |
|---|---|---|
| `dev\whisper` | faster-whisper, av, nvidia-cublas-cu12, nvidia-cudnn-cu12 | speech to word-level WebVTT |
| `dev\tts` | kokoro-onnx, soundfile, espeakng_loader | local text to speech |
| `dev\chatterbox` | torch, torchaudio | voice cloning (torch installed CPU-only) |
| `dev\visual-rig` | mss, PIL, imageio_ffmpeg | screen capture |

**CUDA trap:** pip's `nvidia-*` wheels put DLLs in
`site-packages/nvidia/*/bin`, which nothing searches. `os.add_dll_directory()`
is **not** sufficient — CTranslate2 resolves cuBLAS through its own
`LoadLibrary`. Prepend those directories to `os.environ["PATH"]` *before*
importing `faster_whisper`. Symptom: `Library cublas64_12.dll is not found`.

**uv trap:** `uv venv --python 3.12` fails with a spurious "missing expected
target directory" error. Pass the interpreter path directly instead.

---

## AI model stack

ComfyUI models at
`%LOCALAPPDATA%\Comfy-Desktop\ComfyUI-Shared\models`. Every model below is
licensed for commercial use and fits 12 GB VRAM.

| Model | Size | Folder | Licence |
|---|---|---|---|
| FLUX.1-schnell fp8 | 16.05 GB | `checkpoints/` | Apache-2.0 |
| Wan 2.2 TI2V-5B fp16 | 9.31 GB | `diffusion_models/` | Apache-2.0 |
| SDXL base 1.0 | 6.46 GB | `checkpoints/` | OpenRAIL++-M |
| umT5-XXL fp8 encoder | 6.27 GB | `text_encoders/` | Apache-2.0 |
| CLIP-ViT-H-14 | 2.35 GB | `clip_vision/` | MIT |
| ControlNet Union SDXL | 2.34 GB | `controlnet/` | Apache-2.0 |
| Wan 2.2 VAE | 1.31 GB | `vae/` | Apache-2.0 |
| IP-Adapter Plus SDXL | 0.79 GB | `ipadapter/` | Apache-2.0 |
| RealESRGAN x4plus | 0.06 GB | `upscale_models/` | BSD-3-Clause |

**Custom nodes** (`ComfyUI-Installs\ComfyUI\ComfyUI\custom_nodes`):
`ComfyUI-Manager`, `ComfyUI_IPAdapter_plus`. IP-Adapter model files are
useless without the node.

**Rejected on licence grounds — do not reinstall:**
FLUX.1-dev (non-commercial), 4x-UltraSharp (CC BY-NC-SA). Most popular
ComfyUI upscalers are non-commercial. Check before downloading.

**Removed as unusable:** MiniMax H3 stack (41 GB). Needed ~34 GB of VRAM
against 12 GB available. Wan 2.2 covers the same ground and fits.

---

## Structured memory database

**SQLite is a workflow dependency**, not just a storage option. It carries the
accumulating, queryable state that markdown memory files handle badly.

| | |
|---|---|
| Engine | SQLite 3.50.4, bundled with Python 3.13 (no CLI needed) |
| Database | `C:\Users\Admin\dev\memory-db\memory.db` — **never committed** |
| Tracked artefact | `memory.sql` — deterministic dump, plus `memory.sha256` |
| Repo | `dancockrell/memory-db` — **private** |
| Backup | `backup.py`, hourly via scheduled task `ClaudeMemoryBackup` |

### How the backup works

Dump the database to SQL text, hash the dump, compare against
`memory.sha256`, and commit only when the hash changes. Then verify
`HEAD == origin/main` rather than trusting the push exit code.

Two deliberate choices:

**Text dump, not the binary `.db`.** Git stores a complete copy of a binary
blob on every commit. Twenty-four commits a day of a growing SQLite file
bloats the repository permanently and irreversibly. A `.sql` dump diffs,
compresses, and stays reviewable.

**Private repository.** This accumulates project context and working notes.
A commit cannot be un-published.

### Schema

| Table | Holds |
|---|---|
| `facts` | durable facts by category, unique on (category, key), with a confidence flag |
| `events` | append-only log of decisions, errors, milestones, removals, installs |
| `projects` | project state, paths, repos, stack |
| `dependencies` | **external dependencies no manifest captures** |

That last table is the point. It exists because `Ruby4Lich5` was removed as
apparent game clutter when it is a hard dependency of `dr-companion`, and
nothing anywhere recorded the relationship.

### Restoring

```bash
git clone https://github.com/dancockrell/memory-db
cd memory-db
python -c "import sqlite3,pathlib; sqlite3.connect('memory.db').executescript(pathlib.Path('memory.sql').read_text(encoding='utf-8'))"
```

### Manual run

```bash
cd C:\Users\Admin\dev\memory-db && python backup.py
```

---

## Applications

Kept deliberately, in use:

- **Comfy Desktop** 1.0.39 — image/video generation
- **VS Code** 1.133.0
- **Camtasia** (TechSmith) — screen recording, standard instructional-design tool
- **Scrivener** — long-form writing
- **CLIP STUDIO PAINT**, **Krita** — art
- **Audacity** — audio editing
- **LibreOffice**, **VLC**, **Notion**
- **VIPTeacher PC Client** — active side work. Its uninstaller is deliberately
  renamed `_KEEP_Uninstall VIPKIDT.exe` to survive cleanup sweeps.
- **Wacom Tablet** driver
- **Godot** — game engine (replaced Unity, MIT licensed)

---

## Machine

| | |
|---|---|
| GPU | RTX 4070, **12,012 MB VRAM** |
| CPU | i7-13700KF, 16 cores / 24 threads |
| RAM | 32 GB |
| OS | Windows 11 Home, build 26200 |

**VRAM trap:** `Win32_VideoController.AdapterRAM` reports 4 GB — a 32-bit
overflow bug. Use `nvidia-smi --query-gpu=memory.total`.

---

## Known environment traps

1. **PowerShell execution policy is `AllSigned`**, which blocks `npm.ps1`.
   npm works in Git Bash and cmd. Fix: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`
2. **Windows script files need CRLF.** Heredocs write LF; `cmd.exe` mis-parses
   LF-only `.cmd`/`.bat`/`.ps1` and reports misleading errors. Always
   `sed -i 's/$/\r/' file` and verify with `cat -A` (lines end `^M$`).
3. **Git must use the GitHub noreply email.** Pushes exposing the real address
   are rejected, and the error does not say so clearly.
4. **Never clear `%TEMP%` wholesale on a running system.** Installers and
   updaters stage there. Doing so broke a Chrome auto-update mid-flight and
   removed `chrome.exe` while leaving the other 268 files intact.
5. **Killing `ollama app` kills the server** — the desktop app supervises
   `ollama serve`.

---

## Rebuilding from scratch

```powershell
winget install Git.Git OpenJS.NodeJS.LTS Python.Python.3.13 GitHub.cli
winget install astral-sh.uv Gyan.FFmpeg JohnMacFarlane.Pandoc
winget install Gitleaks.Gitleaks Google.OSVScanner
# then: rustup from rustup.rs, Comfy Desktop, Godot, VS Code
```

Then open a **new terminal**, and:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
git config --global user.name "Dan Cockrell"
git config --global user.email "<id>+dancockrell@users.noreply.github.com"
git config --global init.defaultBranch main
git config --global core.longpaths true
git config --global init.templateDir C:\Users\Admin\.git-templates
```

Clone projects from [github.com/dancockrell](https://github.com/dancockrell),
reinstall Ruby4Lich5 for `dr-companion`, and re-download the ComfyUI models
listed above.
