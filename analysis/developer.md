========================================================
TECHNICAL PROPOSAL – “Minesweeper : Plants vs Zombies Edition”
========================================================

1. High-level Architecture (keep-it-simple)
-------------------------------------------
Web first (HTML5).  
Single-page application that runs 100 % on the client; a tiny Node/Express service is optional and only needed if we later want to persist scores or configs.

```
 ┌───────────────┐
 │   Browser     │
 │  (React SPA)  │
 └──────┬────────┘
        │ static assets (JS/CSS/images)
 ┌──────▼────────┐
 │  CDN / S3     │ ← automatic deploy via CI
 └──────┬────────┘
        │ (opt.) REST/HTTPS
 ┌──────▼────────┐
 │ Node/Express  │ optional – scores, configs
 │  + SQLite     │
 └───────────────┘
```

Why no server logic now?  
• Classic Minesweeper is deterministic and single-player.  
• Removes infra & security overhead.  
• Still scalable (everything is static).  
The backend stub keeps the solution extensible but is not a blocker for v1.

2. Required Components
----------------------
Frontend
1. GameEngine.ts – pure functions for board generation, reveal, flood-fill, flag, win/lose check. 100 % unit tested.
2. UI Components
   • `Board` – flex/grid that renders cells.  
   • `Cell` – handles left/right/touch events, draws sprite/number/state.  
   • `HUD` – mine counter (= zombies left), timer, new-game button, difficulty.  
   • `ThemeProvider` – central palette + sprite atlas swap (PvZ or placeholders).  
3. AssetLoader – preloads png/webp + audio, shows splash if > 2 s.
4. State management – React Context (small), no Redux needed.

Optional Backend
5. Express API with three endpoints (see item 4). SQLite file DB (or Dynamo/Firestore if serverless). Very small footprint, dockerised.

Tooling / Ops
6. Vite + React + TypeScript.  
7. Jest + React Testing Library for logic & component tests.  
8. ESLint/Prettier + Husky pre-commit.  
9. GitHub Actions: build → unit tests → deploy to Netlify/Vercel or S3+CloudFront.

3. Suggested Data Model
-----------------------
Pure front-end objects

```
Cell {
  x: number
  y: number
  hasMine: boolean      // zombie?
  isRevealed: boolean
  isFlagged: boolean    // plant?
  adjacentMines: number
}

GameState {
  board: Cell[][]
  rows: number
  cols: number
  mines: number
  startedAt: Date | null
  endedAt: Date | null
  status: 'idle' | 'playing' | 'won' | 'lost'
  flagsLeft: number
}
```

Backend (optional)

```
Score {
  id          INTEGER PK
  playerName  TEXT
  boardSize   TEXT   // e.g. 9x9, 16x16
  mineCount   INT
  timeMs      INT
  createdAt   DATETIME
}
```

4. Suggested REST End-points
----------------------------
(all under /api)

GET /config  
→ `{ difficulties: [{id:'easy', rows:9, cols:9, mines:10},…] }`  
(Static JSON if no backend.)

POST /score  
Body: `{ name, boardSize, mines, timeMs }`  
Returns created object or 201.

GET /score?boardSize=9x9&limit=20  
→ Top N scores list.

No auth for v1. CORS open only for front-end origin.

5. Implementation Plan & Milestones
-----------------------------------
Week 0 – Pre-work  
• Confirm open questions (platform OK, board sizes, IP licence).  
• Decide: real PvZ assets now or placeholder pack.

Week 1 – Project skeleton  
• Repo, CI, deploy pipeline, basic Vite/React scaffold.  
• Jest, prettier, eslint.

Week 2 – Core Game Engine (TDD)  
• Board generator (first-click safe).  
• openCell / flagCell / win-checker.  
• 95 % code coverage.

Week 3 – Basic UI (placeholder theme)  
• Board & Cell components wired to engine.  
• HUD (timer, counters, restart).  
• Responsive grid (desktop & mobile).

Week 4 – PvZ Theme integration  
• Import sprite atlas or placeholder plants/zombies.  
• ThemeProvider & CSS vars.  
• Basic animations (CSS keyframes – losing zombie “bite”, win confetti).

Week 5 – QA & Accessibility  
• Keyboard nav, aria-labels (numbers, mine) + screen-reader test.  
• Cross-browser smoke (Chrome / Edge / Safari / Firefox, Mobile Safari/Chrome).  
• Lighthouse – perf & a11y ≥ 90.

Week 6 – (Optional) Backend & Leaderboard  
• Express + SQLite, Dockerfile, small admin script.  
• Integrate fetch in HUD if scores feature desired.

Week 7 – Final hardening & hand-off  
• Production build, deploy, write README (build, run, deploy).  
• Collect licences for any third-party assets.  
• Sign-off session with stakeholders.

6. Technical Risks & Mitigations
--------------------------------
1. IP infringement (highest).  
   • Mitigation: use only custom/placeholder art until written approval by EA/PopCap.  
2. Asset weight/perf.  
   • Compress sprites to WebP/PNG-8, lazy-load audio, use Vite “preload” hints.  
3. Different input devices (touch vs right-click).  
   • Detect pointer type: long-press on touch = flag, right-click on desktop = flag.  
4. State update race conditions (fast click spam).  
   • Engine functions are pure & synchronous; use React `useReducer` to serialize updates.  
5. Scope-creep (online features, new mechanics).  
   • PoC first, backlog gatekeeping.  
6. Browser compatibility for audio autoplay policies.  
   • Audio is optional & user-triggered; no auto-play on load.  
7. Security (if backend).  
   • Input validation, rate-limit score submits, no PII stored, run behind HTTPS only.

========================================================
Deliverable
-----------  
• Git repository with documented code.  
• Static build ready for CDN hosting.  
• (Optional) Docker image for API.  
• Test report & lighthouse score.  
• Asset attribution & licence file.