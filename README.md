# Village Provenance Ledger — off-box anchor

This private repository is the **off-box anchor** for the
provenance ledger of Andrew's autonomous village simulation
(v2). It holds one file of substance, `provenance_ledger.jsonl`
(plus `anchor_head.json`, the head snapshot from the latest
push).

## What the ledger is

Every time a villager's mind finishes a spawn in the village,
a small program called **the witness** — separate from the
process that runs the ticks — reads the spawn's recorded
output from the platform's session transcripts, hashes it,
and appends one line to a hash-chained ledger:

```
{"seq", "tick", "villager", "phase", "agent_id",
 "output_sha256", "prev_entry_hash", "entry_hash",
 "witnessed_at"}
```

`output_sha256` is the sha256 of the output's canonical form
(the same canonicalization the provenance gate uses).
`entry_hash` chains each line to its predecessor, so changing
any older line breaks every later link. Line 0 is the genesis
header. Genesis entry hash:
`0b1ac116dfeb26bb6c021aa534e39aeded67c3c6d8be320d476e0c3c090e819b`

The village's gate will only execute a villager's words when
they match the transcript **and** this chain holds a matching
hash for that spawn, witnessed before the words ran.

## Why this copy exists

Everything on the village machine runs as one root user, so a
determined writer on that machine could rewrite the local
ledger wholesale — chain and all — and the rewrite would
verify against itself. This repository is outside that
machine's reach. A separate anchor job pushes the ledger here
once per village day. After a push, the anchored past cannot
be silently rewritten: any later local rewrite differs from
this copy at or before the anchored head.

## How to check a local ledger against this anchor

With the village code (`sim2/anchor.py`), download
`provenance_ledger.jsonl` from this repo, then run:

```
python3 -m sim2.anchor check <downloaded_ledger_path>
```

It re-hashes both chains and compares entry by entry up to
the anchored head. A local ledger may run *ahead* of the
anchor (new witnessing since the push) — that is normal. It
may never *differ* at or below the anchored head; the first
differing seq is named. Exit code 2 means broken or diverged.

## What this does not prove

The chain proves a transcript has not changed **since the
witness hashed it**, and this anchor proves the ledger has
not changed **since it was pushed**. Neither proves that a
forgery slipped in *before* the witness's first pass was
genuine — the witness can only hash what it finds. Closing
that last gap would need the platform's own spawn records,
which the village's code cannot currently read.
