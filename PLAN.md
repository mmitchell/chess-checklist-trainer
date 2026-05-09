# Chess Checklist Trainer — Setup & Initial Build Plan

## Context

Personal project: a browser-based trainer that grades the user's *evaluation process* (a 6-step pre-move checklist) rather than just their final move. The pedagogical bet is that a 950-rated player improves fastest by building checklist habits, not by memorizing engine lines. App is fully client-side: Maia ONNX as the human-like opponent, Stockfish WASM as ground-truth checklist evaluator, chess.js for board logic, react-chessboard for UI, Next.js + TypeScript shell.

This plan covers (a) one-time machine setup, (b) project bootstrap, and (c) the first runnable milestone — a board with a stub bot and an empty checklist sidebar, so the architecture is in place before the engine work starts.

## Decisions (already made)

- **Location**: `~/projects/chess-checklist-trainer`
- **Node**: install via nvm (LTS)
- **Scaffold**: fresh `create-next-app`, pull Maia loading code in later
- **GitHub**: public repo via `gh repo create`
- **Repo name**: `chess-checklist-trainer`

## Phase 0 — Prerequisites (one-time, run in current shell)

You currently have: git ✓, Homebrew ✓, **no Node.js**.

```bash
# 1. Install nvm via Homebrew
brew install nvm
mkdir -p ~/.nvm

# 2. Add to ~/.zshrc (Homebrew prints exact lines after install — paste those)
export NVM_DIR="$HOME/.nvm"
[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"
[ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"

# 3. Reload shell, install Node LTS
source ~/.zshrc
nvm install --lts
nvm use --lts
node --version   # expect v20.x or v22.x

# 4. GitHub CLI (if not already installed)
brew install gh
gh auth status   # if not logged in: gh auth login
```

## Phase 1 — Scaffold + GitHub

```bash
mkdir -p ~/projects && cd ~/projects

# Next.js 14+ with TS, Tailwind, app router, src dir
npx create-next-app@latest chess-checklist-trainer \
  --typescript --tailwind --app --src-dir --eslint --import-alias "@/*"

cd chess-checklist-trainer

# Core runtime deps
npm install chess.js react-chessboard
npm install onnxruntime-web        # for Maia later
# Stockfish WASM: we'll vendor the build into /public/stockfish in Phase 3 — no npm install needed

# Initial commit (create-next-app already ran git init)
git add -A
git commit -m "Initial scaffold from create-next-app + chess deps"

# Public GitHub repo
gh repo create chess-checklist-trainer --public --source=. --remote=origin --push
```

## Phase 2 — Restart Claude Code in the new folder

**Important:** quit this Claude Code session and relaunch from inside `~/projects/chess-checklist-trainer`. The session's working directory is fixed at startup; reopening there gives you accurate file paths, scoped permissions, and a project-local CLAUDE.md if you add one.

## Phase 3 — Milestone 1: Board + stub bot + empty checklist sidebar

Goal: prove the architecture renders end-to-end before introducing engine complexity. No Maia, no Stockfish yet — just a board you can play moves on against a "random legal move" bot, with a sidebar showing the 6 checklist steps as static text.

**Files to create:**
- `src/app/page.tsx` — main route, two-column layout: board left, checklist right
- `src/components/Board.tsx` — wraps `react-chessboard`, owns a `Chess` instance from chess.js, handles user moves and triggers bot reply
- `src/components/ChecklistSidebar.tsx` — static rendering of the 6 steps with placeholder input slots
- `src/lib/bots/randomBot.ts` — exports `pickMove(chess: Chess): Move` returning a random legal move; will be swapped for Maia later behind the same interface
- `src/lib/types.ts` — `ChecklistStep`, `BotInterface`, etc.

**Verification for Milestone 1:**
- `npm run dev` → board renders at `http://localhost:3000`
- You can drag a piece, the bot replies with a random legal move
- Illegal moves snap back
- Sidebar shows steps 1–6 as static text
- No console errors

Commit at end of milestone, push to GitHub.

## Future milestones (sketched, not detailed yet)

| # | Milestone | Notes |
|---|-----------|-------|
| 2 | Checklist input UI | User types/clicks observations per step; stored in component state |
| 3 | Stockfish WASM in web worker | Vendor build into `/public/stockfish/`, message-passing via worker; produces ground-truth eval |
| 4 | Steps 1–2 engine: attack maps | `chess.js.isAttacked()` diff before/after opponent move; enumerate hanging pieces |
| 5 | Step 3: null-move trick | Stockfish eval at depth 1–3 in null-move position |
| 6 | Step 4: CCT enumeration | Filter legal moves to checks, captures, threat-creators for both sides |
| 7 | Step 5: post-move re-scan | Apply candidate move, re-run hanging-piece scan |
| 8 | Diff engine + template feedback | Mechanical comparison of user input vs. engine output, "You missed: …" strings |
| 9 | Swap stub bot → Maia ONNX | onnxruntime-web loads Maia 1100/1300 model, replaces `randomBot` |
| 10 | chess.com import | Public API → import own games for post-game review mode |

Each milestone is its own branch + PR for clean history; merge once verification passes.

## Critical paths / references

- Maia reference frontend: https://github.com/CSSLab/maia-platform-frontend (consult for ONNX loading code in milestone 9)
- chess.js: https://github.com/jhlywa/chess.js — `.isAttacked(square, color)`, `.moves({ verbose: true })`, `.move()`, `.fen()`
- react-chessboard: https://github.com/Clariity/react-chessboard
- Stockfish WASM: https://github.com/lichess-org/stockfish.wasm (the lichess build is the standard one for browsers)

## Overall verification

After Phase 1 you should be able to:
1. `cd ~/projects/chess-checklist-trainer && npm run dev` → Next.js welcome page loads at `localhost:3000`
2. `git log` shows the initial commit
3. `gh repo view --web` opens the public repo on GitHub

After Milestone 1 (Phase 3) you should be able to play random-bot games in the browser with the checklist sidebar visible.
