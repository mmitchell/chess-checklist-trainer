<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Chess Checklist Trainer

Browser-based trainer that grades the user's *evaluation process* (a 6-step pre-move checklist) rather than just the final move choice. Maia ONNX bot as opponent, Stockfish WASM as ground-truth checklist evaluator. Fully client-side, no backend, works offline.

## User context

Owner is ~950 ELO chess.com rapid (10-minute). Calibrate chess explanations and feature suggestions to that level — focus on tactical fundamentals (hanging pieces, checks/captures/threats) over deep positional theory. The 6 checklist steps are designed to catch ~85-90% of game-deciding errors at this level; plateau ceiling for the checklist alone is roughly 1200-1400.

## Tech stack

- **Framework**: Next.js 16 + TypeScript + Tailwind, app router, `src/` layout
- **Board**: `chess.js` (logic), `react-chessboard` (UI)
- **Bot**: `onnxruntime-web` will load Maia ONNX models (1100-1900 available); not wired yet
- **Engine**: Stockfish WASM in a web worker; will be vendored into `/public/stockfish/` when needed
- **Storage**: `localStorage` for session state — no database

## Current state

Phases 0 (machine setup) and 1 (scaffold + GitHub) are **done**. Repo: https://github.com/mmitchell/chess-checklist-trainer (public).

**Next up: Milestone 1** — board + stub random-move bot + static checklist sidebar. See `PLAN.md` § Phase 3 for file list and verification steps.

## Plan

Full design doc and milestone roadmap in [PLAN.md](./PLAN.md). Update PLAN.md as architecture decisions evolve; update this file's "Current state" section as milestones complete.

## Conventions

- Each milestone gets its own branch + PR; merge once verification (manual browser check) passes
- Bots implement a shared `BotInterface` so swapping random → Maia is a one-line change later
- Engine analysis (steps 1-5) runs in a web worker, never on the main thread
