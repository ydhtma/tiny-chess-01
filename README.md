# tiny-chess-01 — 4KB chess engine

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/ydhtma/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/ydhtma/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A 0x88-board negamax engine in 4,094 bytes (MIT). Pure Python 3 stdlib,
no other repo file read, no model calls, deterministic: search runs to a
fixed node budget (100,000 nodes/move), never a wall clock, so a replay
produces the same moves.

## Engine

- 0x88 board, incremental Zobrist hash + incremental piece-square eval.
- Negamax alpha-beta + quiescence (captures / check evasions), iterative
  deepening on the fixed node budget, transposition table (capped 200k
  entries), MVV-LVA + transposition-move ordering, check extension.
- House-style piece-square tables (pawn advance/centre, knight/bishop
  centrality, rook 7th rank, queen centrality, king middlegame table).
- Driver: polls the env `/game`, plays `/move`, retries transient HTTP
  errors, falls back to the first legal move if the search fails.

## Layout

- `main.py` — the whole engine (lzma + base85 payload, 4,094 bytes).
- `dev/` — readable source (`main_v5.py`), selftests, trial harness,
  build script (NOT part of the submitted snapshot; see below).

## Size rule (verified)

- `main.py` 4,094 bytes ≤ 4,096. Repo snapshot (main.py + LICENSE +
  README.md) well under 8,192 bytes; no file over 4,096.
- `dev/main_v5.py` is the readable source; `dev/build.py` minifies,
  renames, lzma-9 + base85 packs it into `main.py`, and verifies the
  round-trip compiles and the selftests pass on the minified source.

## Verification

- Selftests (`dev/verify_min.py`): perft 1–4 startpos, Kiwipete
  perft(3), hash/eval/piece-list consistency over 30 random games ×
  60 plies, eval sanity, UCI sanity — all pass.
- Practice trials @100k nodes (kit env, `--driver process`):
  trial-0 Elo 1191 (2/1/3), trial-1 Elo 876 (0/2/4), trial-2 Elo 997
  (1/1/4). Mean 1021 — below the 1154 practice floor; see BUILD-LOG.md
  for the full per-game lines and the size/strength tradeoffs made.
- Max move time 8.38s (trial-0) / 6.70s (trial-1) under normal load;
  trial-2 saw 2 timeout losses at 10.59s under heavy VM contention.
