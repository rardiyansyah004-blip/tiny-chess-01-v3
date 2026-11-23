# tiny-chess-01-v3

[![make.fun](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/rardiyansyah004-blip/tiny-chess-01-v3/rank.svg)](https://make.fun/bounty/tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/rardiyansyah004-blip/tiny-chess-01-v3/reward.svg)](https://make.fun/bounty/tiny-chess-01)

A chess engine in 4 KB of Python, built for the [make.fun](https://make.fun/bounty/tiny-chess-01) bounty "Build the strongest chess engine in 4 KB of Python". This is v3, a structural revision of the v2 engine (practice Elo 1364) chasing a payable position — and it is a large step up: **practice Elo 1654**.

`main.py` is the packed entry (4,075 bytes, at most 4,096), produced by the kit's `compress` from `engine3.py`, the readable source kept beside it in this folder (uncommitted — it is 10,739 bytes, over the 4,096 per-file cap, and is sent as the pack source at bundle time).

## Engine (own work, not a sunfish fork)

0x88 board, principal-variation search (PVS) with late-move reductions (LMR) and iterative deepening under a FIXED node budget (140,000 nodes/move — no clock, no unseeded randomness), quiescence over captures/promotions/check replies with delta pruning, transposition table, killer moves, MVV-LVA ordering, tapered PeSTO evaluation (midgame/endgame piece-square tables + piece values, phased by material, with separate endgame pawn and king tables), null-move pruning, and repetition avoidance replayed from the game's history. Deterministic: every move derives from the position, the game history and the fixed node budget.

## What changed vs v2 — a structural leap, not micro-tweaks

Profiling showed the real bottleneck is **nodes per second** (~25k under the profiler; the engine is node-starved, not depth-starved): move generation is 30% of time, attack detection 21%, make/unmake 23%. A micro-tweaked v3 (bigger node cap + check extension) actually REGRESSED to Elo 1279 and lost a game on time. So v3 instead:

- **Faster move generation:** iterates a precomputed list of the 64 valid 0x88 squares instead of scanning all 128 squares and skipping off-board ones.
- **More aggressive LMR:** reductions start at move index 2 (was 3) and the double reduction starts at index 6 (was 8), so each node reaches deeper.
- **Check extension REMOVED:** it cost time (a timeout loss) and Elo; the deeper LMR search covers tactics without it.
- **Node cap 140,000** (was 120,000): the most nodes that provably never time out — a 160,000-cap variant was tested and lost two games on time (Elo 1451).
- Tempo bonus +10 and max depth 14 kept from the earlier v3 draft.

## Local practice results

Kit env server + scorer, practice/trial-0 (6 games vs the house gauntlet), v3 final (140k cap):

- Game 1 (depth-2, white): **win** (checkmate)
- Game 2 (depth-3, black): **win** (checkmate)
- Game 3 (depth-4, white): **win** (checkmate)
- Game 4 (nodes-100k, black): **win** (checkmate) — v2 drew this, the first v3 draft lost it
- Game 5 (nodes-250k, white): loss (checkmate)
- Game 6 (nodes-500k, black): **win** (checkmate) — v2 and the first v3 draft both lost this

**5 won, 0 drawn, 1 lost — practice Elo 1654** (v2: 1364; v1: 1280; first v3 draft: 1279; 160k-cap variant: 1451). Wall 747 s, spare plies 634. Well above the sunfish baseline (~1365), so a paid place (top 20 + beat baseline) is now in reach.

## Files

- `main.py` — the packed entry (4,075 bytes, at most 4,096)
- `engine3.py` — the readable source `main.py` was packed from (uncommitted; sent as pack source)
- `LICENSE` — MIT
