# Sage Owl Chess — a deterministic 4 KB engine

An original tiny chess engine written for the make.fun
**tiny-chess-01** competition. The whole engine is `main.py`:
a single Python file, standard library only, that talks to the
competition environment at `MAKEFUN_ENV_URL` and plays every game
of the gauntlet it is served.

## Design

- **Board**: 0x88 mailbox board in a flat 128-slot list; pieces are
  small integers (type in the low bits, colour in the high bit).
- **Move generation**: pseudo-legal generation with offset tables
  per piece type, sliding rays for bishop/rook/queen, pawn pushes,
  captures, en passant, promotions and both castles; legality is
  confirmed by making the move and testing the own king.
- **Search**: iterative-deepening negamax with alpha-beta windows,
  a Zobrist transposition table, null-move pruning, late-move
  reductions and killer-move ordering. The quiescence search only
  extends captures. Everything is budgeted in **nodes, never
  time**, so the same position always produces the same move.
- **Evaluation**: material plus formula-generated piece-square
  tables, and an endgame king table blended in as non-pawn
  material disappears.

## This variant

Sage Owl keeps the family's reference material values and changes
almost everything around them. Its pawn table grows
quadratically in rank squared (2r² + 2r), its rook table pays
more for central files, its queen table is steeper, and its
endgame king blend completes over only 5,000 points of non-pawn
material — the earliest king activation in the family. Like the
crisp owl it orders quiet moves with a **history table** fed by
depth-squared cutoff credits, but it pairs that with the deepest
node schedule of all: 58k / 31k / 15.5k across the game. There is
no bishop-pair term; the sage owl would rather have the tempi.
Zobrist keys come from its own seeded generator. In the
family's tactical suite it converts the pawn endgame along a
different first move than any sibling.

## Files

- `main.py` — the engine (the readable source is packed into the
  file as an lzma + base85 payload under a two-line unpacker, a
  format the rules explicitly allow; both parts count toward the
  4,096-byte cap).
- `LICENSE` — MIT.
- `.makefun/owner.json` — the entrant wallet for this entry.

## Running

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:PORT python3 main.py
```

The bot loops on `POST /game` and answers `POST /move` until the
environment reports the trial done.
