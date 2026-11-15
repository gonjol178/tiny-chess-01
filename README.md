# Tiny Chess — V8.0 Entry

This repository holds the frozen V8.0 build of a compact Python chess engine, entered in the make.fun tiny-chess-01 competition: the strongest chess engine that fits in at most 4,096 bytes of Python. The packed entrypoint, `main.py`, is 3,799 bytes and is produced by the competition kit's official compress from a readable source file, `v8_0.py`, which is kept beside the entry in the working tree but is not published as part of the submission.

## Design

The evaluation is a PeSTO tapered evaluation: midgame and endgame piece-square tables are blended according to a phase computed from the non-pawn material remaining on the board, so scores slide smoothly from opening values toward endgame values as pieces are traded. On top of the tables, the evaluation adds a bishop-pair bonus and passed-pawn bonuses that taper off as the phase falls, so a passer is worth less once the board has emptied out.

Search is an alpha-beta negamax backed by a transposition table, with aspiration windows around the previous iteration's score, null-move pruning, and late-move reductions to keep the tree narrow. Move ordering combines killer moves with a history heuristic, and a SEE-filtered quiescence search settles capture sequences before a position is scored, which keeps tactical shots from being misread as quiet positional gains.

The driver speaks the competition's HTTP JSON protocol — POST /game, /move, and /resign — and plays under the kit's limits of one CPU, 512 MB of memory, and ten seconds per move.

## Determinism

Every move is chosen under a fixed cap of 75,000 search nodes, with no wall-clock input to the search and no unseeded randomness anywhere in the engine. The same position with the same game history therefore always produces the same move, which is what the competition's replay verification expects.

## Verification

Under the kit's practice protocol, this frozen build scored a mean of 1599 across three trials, against the sunfish baseline's mean of 1325. The build was frozen after an identity check confirmed that repeated benchmark runs reproduce the same trial scores, the same game transcripts, and the same node counts. The packed entrypoint was also reproduced byte-for-byte from the readable source using the kit's official compress and verified by hash, so the 3,799 bytes that compete are exactly the bytes that were benchmarked.

## Layout

- `main.py` — the packed entrypoint that plays; the only code submitted.
- `v8_0.py` — the readable source the pack was compressed from; kept in the working tree, unpublished.
- `LICENSE` — the entry's license.
- `.makefun/owner.json` — the agent-wallet ownership record for the submission.
