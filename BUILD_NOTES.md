# tiny-chess-01 — build notes

## Rules
- `main.py` ≤ 4096 B, stdlib only, deterministic. Repo ≤ 8192 B, no file > 4096. MIT.
- Trial: 8 games (practice 6) vs depth-2, depth-3, depth-4, nodes-100k, nodes-250k, nodes-500k, nodes-1M, nodes-2M. Elo 0–2112.
- 10 s/move, ~3 s avg, 1800 s wall/trial. No clock, no randomness, no model calls. Practice Elo ≥ 1154 for final; prizes need beating sunfish-4k (~1365). Deadline Wed 2026-11-25 00:00 WIB. $500, 20 places.

## Design
- 0x88 board, negamax + alpha-beta, iterative deepening to a fixed 48,000-node budget/move (no clock), quiescence over captures/promotions and all replies to check, TT (120k entries, persists across moves), killer moves, MVV-LVA ordering, material + piece-square eval, incremental eval, root-history repetition avoidance.
- `main.py` is an lzma/b85 packed payload (allowed) of a minified engine: 3847 B packed, 7138 B unpacked. Readable source kept outside repo (would break 8192 B cap).

## Local tests
- Perft exact: startpos 20/400/8902, kiwipete 48/2039/97862, EP 14/191/2812 (d1–d3). 300 random make/unmake cycles restore board+eval exactly. Mate-in-1 found.
- `makefun check --worktree`: size/license/entrypoint pass; only `transcripts_present` fails (bundled at submit).
- `grade-local` can't run here (no Docker); trials run directly vs kit env + scorer.

## Practice trial-0 (local, real kit env + scorer)
- T1 (PST flipped): Elo 876, 1W 0D 5L — beat depth-2 only.
- T2 (PST fixed): Elo 997, 1W 1D 4L — beat depth-3, drew depth-2.
- T3 (qs stand-pat-in-check fix; Rf5 had walked into mate-in-3): Elo 1191, 2W 1D 3L — beat depth-2/3, drew nodes-500k; above 1154 floor.
- T4 (80k nodes): Elo 1099, 2W 0D 4L — no gain over T3, slower; reverted to 48k.

## Known gaps
- No null-move pruning, no LMR, one PST set (no tapered/endgame king table), no bishop-pair term. 48k nodes/move ≈ depth 4–5 middlegame; sunfish baseline not guaranteed.
- Mate scores never stored in TT; repetition state can stale TT entries.
