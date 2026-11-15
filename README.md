# Tiny Chess — V8.0 engine

A deterministic chess engine in a single packed Python file (`main.py`,
3,799 bytes), built for the make.fun tiny-chess-01 competition (4,096-byte
entrypoint cap). `main.py` is the output of the kit's official
`makefun compress` run on the readable source; the source travels with the
submission bundle under `packs/` and is byte-identical to the frozen V8.0
baseline (`v8_0.py`, sha256
`1309afae0ce87257df87324146b0fd835f7511bbb9a316cdd40e272e81b69b8f`).

## How it plays

- **Board**: 120-square mailbox (10x12) held as a string; make-move returns
  a new position tuple carrying the board, side to move, castling rights,
  en-passant square, halfmove clock, two incremental scores, and a phase
  counter.
- **Evaluation**: material plus piece-square tables in the PeSTO style, kept
  incrementally by make-move as separate middlegame and endgame scores and
  blended by the phase counter (starts at 24; knight/bishop 1, rook 2,
  queen 4 — removed on capture, added back on promotion). A bishop-pair
  bonus (+30) and, once the phase falls to 12 or below, a passed-pawn bonus
  that grows with the square of the pawn's advancement when no enemy pawn
  covers the files ahead of it.
- **Search**: iterative-deepening negamax with plain alpha-beta (no PVS)
  and a hard, deterministic cap of 75,000 nodes per move — no wall clock,
  no randomness, so a position plus game history always yields the same
  move.
  - Transposition table keyed on board, castling, en-passant and side to
    move, storing depth, score, bound flag and best move.
  - Aspiration windows (+/-45) from depth 5, with a full-window re-search
    on a fail high or low.
  - Null-move pruning (reduction 3) at depth >= 3 when not in check and
    the phase is >= 3.
  - Late-move reductions for later quiet moves, re-searched at full depth
    when they raise alpha.
  - Move ordering: transposition-table move first, captures scored by
    MVV-LVA with a static-exchange-evaluation split between winning and
    losing captures, promotions, two killer moves per ply, then a history
    heuristic.
  - Quiescence search with stand-pat: captures and promotions only, losing
    captures filtered by static exchange evaluation unless they give check,
    and full legal evasions when in check.
  - Repetition (game history and search path) and the fifty-move counter
    score as draws.
- **Protocol**: speaks the competition's HTTP JSON environment
  (`MAKEFUN_ENV_URL`: `/game`, `/move`, `/resign`), Python 3 standard
  library only. It resigns when the root score falls to -950 or below.

Local verification under the competition's practice protocol (fixed
75,000-node cap): three practice trials graded 1598 / 1530 / 1669, mean
1599, against a 1325-rated baseline.
