# quick-launch-app

Dev Tool Launcher & Cheatsheet — Agent Build Brief (2025 Edition)

> Goal: A zero‑backend, browser app that lets users launch tools and copy/paste commands fast — across macOS, Windows, Linux, and WSL — with simple organization, import/export, and precise control over how each launch behaves.




---

1) Scope & Outcomes

In scope

Add/Edit/Delete tools.

Add commands per tool with OS applicability.

Quick Launch: open URLs/protocols or copy commands — never execute.

Launch Actions per tool (split‑button): pick what happens on click (e.g., WSL terminal, Windows Terminal, VS Code WSL Remote, Cursor CLI, URL, etc.).

Install path (optional per OS) with Reveal (best‑effort file://) + Copy reveal command fallback.

Search, tags, categories, favorites, OS filter, sort & reorder.

Import/Export JSON; all data stays local (localStorage).


Out of scope

Authentication, multi‑user collaboration, cloud sync, remote command execution.


Success criteria

User can find a tool and copy/run what they need in ≤2 clicks.



---

2) Core UX

2.1 Tool list

Header: Search, OS filter (All/macOS/Windows/Linux), Sort (Manual/Name/Category), Group by category, Favorites only.

Cards show: name, description, category, tags, website/docs links, favorite toggle, Quick Launch split‑button, commands (filtered by OS), optional install path block.


2.2 Quick Launch split‑button

Primary: runs the tool’s Default Action.

Caret menu: other Actions (e.g., Open in WSL terminal, Open in VS Code (WSL), Open in Cursor, Open URL).

Behavior remains safe: either open URL/protocol or copy text.


2.3 Manage tools (CRUD + order)

Add/Edit dialog with: name, description, category, website, docs, tags (CSV), Launch strings (per‑OS + custom), Launch profiles (per‑OS), Actions (list), Install paths (per‑OS), commands (sortable), notes.

Reorder tools: drag handle + keyboard (Alt+↑/↓, Home/End).

Reorder commands inside a tool similarly.

Manual order persists and survives sort toggles.


2.4 Install path & Reveal

Optional path per OS displayed on the card when available.

Buttons: Copy path, Reveal (best‑effort file://), Copy reveal cmd (terminal helper: mac open -R, Win explorer.exe, Linux xdg-open).



---

3) Launch semantics (simple & explicit)

3.1 Launch string

One Launch string per OS (plus optional Custom). Examples: https://…, vscode://…, code ., docker ps -a.


3.2 Launch profiles (per‑OS default)

auto (default): if value looks like scheme:// → open; else copy as plain text.

url: always open as URL/protocol.

copy: always copy text.

wsl: copy wsl.exe -d <distro> -- bash -lc "<Launch string>".

wt: copy wt.exe <Launch string>.

ide-cli: copy <cli> <Launch string> (default code; can be cursor, windsurf, etc.).


3.3 Actions (choose what happens on click)

Each tool may define multiple Actions (label + profile + optional params + optional per‑OS Launch override). One is Default.

Preset: VS Code — WSL Remote → copies code --remote wsl+<distro> <path>.

Parameters (optional): WSL distro (default Ubuntu), IDE CLI name (default code).


3.4 What we never do

The app never executes commands on the user’s machine. It only opens URLs/protocols or copies exact text.



---

4) Data model (storage shape)

> Stored as JSON in localStorage (key toolpad.v2). Export/Import uses the same shape.



4.1 Tool

id (string, uid)

name (string)

description (string?)

category (string?)

website (url?)

docs (url?)

tags (string[]?)

launch (map: mac|windows|linux|custom → string)

launchProfile (map: mac|windows|linux|custom → enum auto|url|copy|wsl|wt|ide-cli)

actions (LaunchAction[]?)

installPath (map: mac|windows|linux → string?)

wslDistro (string?) — default Ubuntu

ideCliName (string?) — default code

commands (Command[])

favorite (bool?)

notes (string?)

sortIndex (number?) — manual order


4.2 Command

id (string)

label (string)

snippet (string)

os (enum all|mac|windows|linux)

sortIndex (number?)


4.3 LaunchAction

id (string)

label (string)

profile (enum auto|url|copy|wsl|wt|ide-cli|vscode-remote-wsl)

distro (string?) — WSL

ideCliName (string?)

launchOverride (map per OS?) — optional

sortIndex (number?)

isDefault (bool)


Validation on import

Coerce strings; drop unknown fields; ensure OS enums; add missing IDs; normalize sortIndex to 0..n-1.



---

5) UI behaviors

URL detection: treat as URL/protocol when ^[a-zA-Z]+:\/\/ matches.

Copy uses the Clipboard API; show toast on success/failure.

Split‑button: primary runs Default Action; caret opens menu (keyboard accessible).

Reordering available only in Manual sort mode; store sortIndex.

Group by category honors manual order within each category.

OS filtering: show commands whose os matches filter or all.

Best‑effort file:// for Reveal; always show Copy reveal cmd.



---

6) Technology & versions (stable, compatible)

> Pin these (no carets) for reproducible builds.



Node.js: 22.x LTS

Vite: 7.1.x (React + TS template)

React: 19.1.x

TypeScript: 5.9.x

Tailwind CSS: 4.1.x

UI primitives: shadcn/ui (generator) + radix-ui mono package

Icons: lucide-react 0.542.x

Animations: Motion (prev. Framer Motion) 12.x

Drag & drop: @dnd-kit/core 6.3.x, @dnd-kit/sortable 10.0.x, @dnd-kit/modifiers 6.x


Browsers: Chrome/Edge ≥ 117, Firefox ≥ 115, Safari ≥ 16.4. Clipboard API required for best UX.


---

7) Architecture

Single‑page React app; no router; state stored in components + useLocalStorage.

Components (high level):

App: search/filters/sort, list rendering, toasts.

ToolCard: details, split‑button, commands, install path block.

AddEditDialog: full form with Actions editor and sortable command list.


DnD: @dnd-kit (pointer + keyboard sensors, vertical list strategy).



---

8) A11y & i18n

Use Radix primitives (Dialog/Menu) for focus‑traps & roles.

Keyboard: all interactive controls operable (Tab/Shift+Tab; Enter/Space). Reorder with Alt+↑/↓, Home/End.

aria-live region announces reorder moves (e.g., “Moved Docker to position 2”).

Labels and title for icon‑only buttons (favorite, drag handle).



---

9) Security & privacy

No execution of arbitrary commands — only open URL/protocol or copy text.

No network persistence; only localStorage + optional file import/export.

External links: target="_blank" rel="noreferrer noopener".

file:// opens may be blocked by the browser; feature remains best‑effort.



---

10) Setup (minimal)

Create Vite React‑TS app; add Tailwind v4; run shadcn/ui generator & add used components; install radix-ui, lucide-react, motion, and optional @dnd-kit/*.

Configure path alias @/* for imports.

Persist JSON under toolpad.v2.



---

11) QA & acceptance

Functional

CRUD: create/edit/delete a tool; data persists after reload.

Quick Launch Actions: Default runs via primary; caret lists alternates.

Profiles:

auto: URL opens; non‑URL copies.

url: always opens.

copy: always copies.

wsl: copies wsl.exe -d <distro> -- bash -lc "<cmd>".

wt: copies wt.exe <cmd>.

ide-cli: copies <cli> <cmd>.

vscode-remote-wsl: copies code --remote wsl+<distro> <path>.


Install path: Copy path; Reveal best‑effort; Copy reveal cmd correct per OS.

Reorder tools & commands by drag and keyboard; order persists.

OS filter shows correct commands; Search hits name/desc/tags/commands.

Import/Export round‑trip preserves schema, order, and defaults.


Non‑functional

Lighthouse a11y ≥ 90; no console errors; works on latest Chrome/Edge/Firefox/Safari.



---

12) Risks & mitigations

Browser blocks file:// → always provide Copy reveal cmd fallback.

IDE protocol/CLI variance (Cursor/Windsurf) → treat as generic ide-cli; user supplies CLI name; provide examples in sample data.

Windows Terminal wt alias not available in WSL → provide plain text copies; users can adapt (e.g., cmd.exe /c wt.exe …).

WSL distros vary → allow per‑tool wslDistro and per‑action override.

Tailwind v4 changes → pin Tailwind 4.1.x and rely on shadcn/ui latest templates.



---

13) Deliverables

1. Vite React‑TS project, Tailwind v4, shadcn/ui + radix‑ui, Motion, lucide‑react, optional dnd‑kit.


2. Implementation of the split Quick Launch with Actions & profiles.


3. Add/Edit dialog with Actions editor and sortable commands.


4. Install path block with Copy/Reveal/Copy‑reveal‑cmd.


5. Import/Export using the schema above; sample dev-tools.json.


6. README with setup and version pins.




---

14) Version matrix (pin these)

Area	Package	Version

Runtime	Node.js	22.x LTS
Bundler	Vite	7.1.x
Framework	React	19.1.x
Language	TypeScript	5.9.x
CSS	Tailwind CSS	4.1.x
UI primitives	radix-ui	1.4.x (mono)
UI generator	shadcn/ui	latest (2025)
Icons	lucide-react	0.542.x
Animation	Motion (prev Framer Motion)	12.x
DnD	@dnd-kit/core	6.3.x
DnD	@dnd-kit/sortable	10.0.x
DnD	@dnd-kit/modifiers	6.x



---

15) Sample action presets (for seed data)

Open URL → auto/url profile with https://… or vscode://… URI.

Open in VS Code (WSL — Ubuntu) → vscode-remote-wsl + distro=Ubuntu; path=..

Open in Cursor (CLI) → ide-cli + ideCliName=cursor, launch string: .

Open in Windows Terminal → wt, launch string: npm run dev

Run in WSL (Ubuntu) → wsl, launch string: docker ps -a, distro=Ubuntu


> Keep this brief as seed examples only; users can add their own variants.




---

15A) Preconfigured Tool — Claude Code CLI (Cursor & VS Code, Windows/WSL)

Goal: Provide a ready-to-import tool with Actions that match your workflow.

Assumptions

Your WSL distro is Ubuntu (change if needed).

Claude Code CLI command is available as claude (update to your exact binary if different).

Linux install path is /home/kngpn/.claude (you mentioned ~/home/kngpn/.claude; on Linux the canonical path is /home/kngpn/.claude).

VS Code + Remote WSL extension installed. Cursor is available on PATH as cursor.


Configured Actions

Default: Open in VS Code (WSL — Ubuntu) → copies code --remote wsl+Ubuntu .

Run in WSL terminal (Ubuntu) → copies wsl.exe -d Ubuntu -- bash -lc "claude"

Run in Windows Terminal → copies wt.exe claude

Open in Cursor (CLI) → copies cursor .


Minimal seed JSON (copy → Import)

[
  {
    "id": "seed-claude-cli",
    "name": "Claude Code CLI",
    "category": "CLI",
    "description": "AI coding assistant CLI. Runs in Windows, WSL, and IDE terminals.",
    "tags": ["ai", "cli"],
    "launch": {
      "windows": "claude",
      "linux": "claude",
      "mac": "claude"
    },
    "launchProfile": {
      "windows": "copy",
      "linux": "copy",
      "mac": "copy"
    },
    "installPath": { "linux": "/home/kngpn/.claude" },
    "wslDistro": "Ubuntu",
    "ideCliName": "code",
    "actions": [
      {
        "id": "act-default-vscode-wsl",
        "label": "Open in VS Code (WSL — Ubuntu)",
        "profile": "vscode-remote-wsl",
        "distro": "Ubuntu",
        "launchOverride": { "windows": "." },
        "isDefault": true,
        "sortIndex": 0
      },
      {
        "id": "act-wsl-run",
        "label": "Run in WSL terminal (Ubuntu)",
        "profile": "wsl",
        "distro": "Ubuntu",
        "launchOverride": { "windows": "claude" },
        "sortIndex": 1
      },
      {
        "id": "act-wt-run",
        "label": "Run in Windows Terminal",
        "profile": "wt",
        "launchOverride": { "windows": "claude" },
        "sortIndex": 2
      },
      {
        "id": "act-cursor-ide",
        "label": "Open in Cursor (CLI)",
        "profile": "ide-cli",
        "ideCliName": "cursor",
        "launchOverride": { "windows": "." },
        "sortIndex": 3
      }
    ],
    "commands": [
      { "id": "c-help", "label": "Help", "snippet": "claude --help", "os": "all", "sortIndex": 0 }
    ],
    "favorite": true,
    "sortIndex": 0
  }
]

Notes

Adjust the CLI command name (claude) to your actual binary (e.g., claude-code).

If you keep ~/home/kngpn/.claude as written, the app will display it verbatim; for the Reveal helper to work reliably on Linux, prefer the full path /home/kngpn/.claude.

To open VS Code with a specific workspace, change the VS Code action path from . to the target path in WSL.



---

16) Implementation guidance (light)

Action resolver (concept):

1. Determine OS → get Launch string (tool.launch[os] or custom or action launchOverride[os]).


2. Determine profile (from action; else from tool.launchProfile[os]; default auto).


3. Build output: { mode: open|copy, value: string }.


4. If open → window.open(value, '_blank'); if copy → clipboard + toast.



Order persistence: after any move, re‑index sortIndex to 0..n-1 and save.

LocalStorage key: toolpad.v2 (migrates v1 if present by mapping fields and setting defaults).



---

17) Backlog (post‑MVP)

PWA install + offline cache.

Variable placeholders in commands (with quick‑fill UI).

Cloud sync (GitHub Gist/Drive) behind explicit opt‑in.

Collections/folders; bulk edit.

Theming.


